# OCI Emulator Architecture

This document describes the technical architecture of OCI Emulator. For the product and UX design, see [DESIGN.md](DESIGN.md). For installation, operation, and development workflows, see [README.md](README.md).

---

## Table of contents

- [1. Context and goals](#1-context-and-goals)
- [2. System context](#2-system-context)
- [3. Runtime topology](#3-runtime-topology)
- [4. Backend architecture](#4-backend-architecture)
- [5. Frontend architecture](#5-frontend-architecture)
- [6. Rendering and hydration](#6-rendering-and-hydration)
- [7. Session and authentication model](#7-session-and-authentication-model)
- [8. API design](#8-api-design)
- [9. Data architecture](#9-data-architecture)
- [10. Emulator module](#10-emulator-module)
- [11. UI proxy](#11-ui-proxy)
- [12. Build and deployment](#12-build-and-deployment)
- [13. Security model](#13-security-model)
- [14. Performance](#14-performance)
- [15. Observability](#15-observability)
- [16. Testing architecture](#16-testing-architecture)
- [17. Extension points](#17-extension-points)
- [18. Decision log](#18-decision-log)
- [19. Known architectural limitations](#19-known-architectural-limitations)

---

## 1. Context and goals

OCI Emulator runs a local OCI control plane with three clients:

1. A browser-based web console.
2. The official OCI CLI.
3. OCI SDKs with custom endpoint support.

The architecture must provide:

- OCI-compatible REST behavior for the supported services.
- A complete web console with SSR, SSG, and CSR.
- Persistent local state without external infrastructure.
- A single public port for the API and the UI.
- A development workflow that does not require Docker.
- Predictable behavior for learning and testing.

The architecture is optimized for a single local user. It does not attempt to provide production-grade isolation, durability, or scalability.

### 1.1 Quality attributes

| Attribute              | Target                                                                |
| ---------------------- | --------------------------------------------------------------------- |
| Protocol compatibility | The official OCI CLI works for supported operations                   |
| Local setup            | One command after system prerequisites                                |
| UI completeness        | Full list, create, detail, and action coverage for supported services |
| Startup time           | Quarkus and UI ready in a few seconds in dev mode                     |
| Data safety            | No writes outside the project data directory and local config         |
| Operability            | One public port, clear logs, reset and seed commands                  |
| Maintainability        | Service pattern reused across backend and frontend                    |
| Offline operation      | No network access required after setup                                |

---

## 2. System context

```text
+----------------+        +-------------------------+
| Browser        |        | OCI CLI / SDKs          |
| Web console    |        | Official clients        |
+--------+-------+        +------------+------------+
         |                             |
         | HTTP :4599                  | HTTP :4599
         |                             |
         v                             v
+---------------------------------------------------+
| Quarkus                                           |
|                                                   |
|  /20160918   Core and Autonomous Database         |
|  /20200501   Load Balancer                        |
|  /20190501   API Gateway                          |
|  /20180115   DNS                                  |
|  /20190415   MySQL                                |
|  /n/...      Object Storage                       |
|  /_emu/*     Session, dashboard, inspector, labs  |
|  /*          UI proxy (production)                |
+------------------------+--------------------------+
                         |
                         v
              +-----------------------+
              | Local storage         |
              | ./data                |
              +-----------------------+
```

The Quarkus process is the public entry point. In production it also proxies UI requests to the Angular SSR server. The SSR server calls back into the Quarkus API for server-rendered data.

---

## 3. Runtime topology

### 3.1 Development

```text
make dev
  |
  +-- Quarkus :4599
  |     +-- OCI API
  |     +-- /_emu
  |     +-- Quinoa dev proxy
  |
  +-- Quinoa
        +-- ng serve :4200
              +-- Angular with SSR and HMR
```

Characteristics:

- One public port: `4599`.
- Quinoa manages the Angular dev server.
- Angular SSR is enabled in the dev server.
- API calls from the browser go to Quarkus on `4599`.
- Hot reload applies to Angular templates and styles.

### 3.2 Production

```text
make run
  |
  +-- Quarkus :4599 (public)
  |     +-- OCI API
  |     +-- /_emu
  |     +-- UiProxy fallback
  |
  +-- Node SSR :4600 (internal)
        +-- Angular server bundle
        +-- Prerendered pages
        +-- Static assets
        +-- Calls Quarkus API on 127.0.0.1:4599
```

Characteristics:

- One public port: `4599`.
- Quarkus serves API requests directly.
- Quarkus forwards UI requests to the Node SSR server.
- The SSR server forwards cookies and renders the authenticated shell.
- Static assets are served by the SSR server and streamed through Quarkus.

### 3.3 Lite production

```text
make run-lite
  |
  +-- Quarkus :4599 (public)
        +-- OCI API
        +-- /_emu
        +-- Prerendered and CSR build
```

Characteristics:

- No Node process.
- SSR is disabled.
- Login guards run in the browser only.
- Useful for constrained environments and CI smoke tests.

### 3.4 Ports

| Port   | Process            | Exposure                    |
| ------ | ------------------ | --------------------------- |
| `4599` | Quarkus            | Public on `127.0.0.1`       |
| `4600` | Node SSR           | Internal on `127.0.0.1`     |
| `4200` | Angular dev server | Internal, managed by Quinoa |

---

## 4. Backend architecture

### 4.1 Technology

- Java 25.
- Quarkus 3.40.1.1.
- RESTEasy Reactive with Jackson.
- Maven.
- JUnit 5 and RestAssured.
- floci-oci storage and protocol primitives.

The project keeps the `io.floci.oci` root package to reduce merge conflicts with upstream. Emulator-specific code lives under `io.floci.oci.emulator`.

### 4.2 Package structure

```text
io.floci.oci
  config/
    EmulatorConfig
  core/
    auth/
    common/
    storage/
    workrequest/
  services/
    identity/
    objectstorage/
    core/
    loadbalancer/
    apigateway/
    dns/
    database/
    mysql/
  emulator/
    EmulatorController
    SessionController
    DashboardService
    RequestInspector
    LabService
    SearchService
    UiProxy
```

### 4.3 Request lifecycle

```text
HTTP request
  |
  v
Request ID filter
  |
  v
Signature parser and auth context
  |
  v
JAX-RS controller
  |
  v
Service
  |
  +-- Validation
  +-- Storage access
  +-- Work request creation
  +-- Compartment usage
  |
  v
Response mapper
  |
  v
HTTP response with OCI headers
```

#### Request ID filter

Adds `opc-request-id` to every response. The value is also stored by the request inspector.

#### Signature parser

Parses the OCI Authorization header when present. It extracts:

- Tenancy OCID.
- User OCID.
- Key fingerprint.
- Region.
- Request target.

The signature is not verified. Unsigned requests are accepted for the web console and local tools.

#### Auth context

`OciContext` carries the current tenancy, user, and region. Services use it for storage scoping and multi-region behavior.

#### Controllers

Controllers parse OCI REST input and return OCI-shaped responses. They do not contain business logic. Emulator-only endpoints are separated under `/_emu`.

#### Services

Services implement validation, state transitions, and storage operations. They throw `OciException` with OCI error codes.

#### Error mapping

`OciExceptionMapper` converts exceptions to the OCI error envelope:

```json
{
  "code": "InvalidParameter",
  "message": "cidrBlock is required.",
  "status": 400,
  "opc-request-id": "..."
}
```

### 4.4 Service pattern

Every resource service follows the same structure:

```text
Controller
  -> Service
    -> Stored model
      -> StorageBackend
        -> CompartmentUsage
          -> ServiceDescriptor
            -> Configuration
              -> Tests
```

Rules:

- Controllers are thin.
- Services own validation and state transitions.
- Stored models are serializable and explicit.
- Storage access goes through the shared abstraction.
- Every service implements `CompartmentUsage`.
- Every service registers a `ServiceDescriptor`.
- Every service has an enable flag.
- Every service has protocol tests.

### 4.5 Shared primitives

| Primitive                    | Purpose                                      |
| ---------------------------- | -------------------------------------------- |
| `StorageBackend`             | Persistent and in-memory storage abstraction |
| `TenancyAwareStorageBackend` | Tenancy and region scoping                   |
| `OciPage`                    | OCI-style pagination                         |
| `Ocids`                      | OCID generation and parsing                  |
| `Etags`                      | Entity tag generation and comparison         |
| `CompartmentValidator`       | Compartment existence and state validation   |
| `CompartmentUsage`           | Non-empty compartment protection             |
| `WorkRequestService`         | Asynchronous operation records               |
| `ServiceDescriptor`          | Service registration and configuration       |
| `OciException`               | OCI error model                              |

### 4.6 Work requests

Work requests model long-running operations.

```text
Client request
  |
  v
Service creates resource in PROVISIONING
  |
  v
Work request created
  |
  v
202 Accepted + opc-work-request-id
  |
  v
Background state transition
  |
  v
Work request SUCCEEDED
  |
  v
Resource ACTIVE or RUNNING
```

The emulator mostly completes operations synchronously but records work requests so clients can poll. Instances, volumes, and databases use background transitions to support `--wait-for-state`.

Work request fields:

- `id`
- `compartmentId`
- `operationType`
- `status`
- `percentComplete`
- `timeAccepted`
- `timeStarted`
- `timeFinished`
- `resources`
- `logs`

### 4.7 Compartment usage

Identity cannot delete a compartment that contains resources. Every service implements `CompartmentUsage` and reports live resources in a compartment across regions. This avoids coupling Identity to individual service stores.

### 4.8 Multi-region behavior

The region comes from the signature or from the `x-emu-region` development header. Storage keys include tenancy and region. The web console keeps the selected region in a cookie and sends it with emulator requests.

### 4.9 Service enablement

Services are registered as enabled or disabled in configuration. Disabled services return OCI-shaped `NotAuthorizedOrNotFound` responses or are not registered, depending on the service. The default configuration disables services that require Docker sidecars.

---

## 5. Frontend architecture

### 5.1 Technology

- Angular 22.2.6.
- TypeScript 6.0.3.
- Angular Material and CDK 22.2.7.
- Transloco 8.4.0.
- ng2-charts 11.0.0 and Chart.js 4.6.1.
- Vitest 5.1.0.
- Playwright 1.65.1.

### 5.2 Application structure

```text
console/src/app/
  core/
    api/
      api-base-url.ts
      oci-error.ts
      pagination.ts
    auth/
      session.service.ts
      auth.guard.ts
      guest.guard.ts
    i18n/
      locale.service.ts
      static-transloco.loader.ts
      format.service.ts
    emulator/
      emulator-api.ts
    work-requests/
      work-request.store.ts
    charts/
      chart-theme.ts
  shared/
    layout/
    ui/
    charts/
    directives/
    pipes/
  features/
    auth/login/
    dashboard/
    compute/
    network/
    block-storage/
    iam/
    object-storage/
    load-balancer/
    api-gateway/
    dns/
    database/
    work-requests/
    emulator/
    docs/
  app.routes.ts
  app.routes.server.ts
  app.config.ts
  app.config.server.ts
```

### 5.3 Feature structure

```text
features/<feature>/
  routes.ts
  api/
    <feature>.api.ts
  models/
    <resource>.model.ts
  pages/
    list/
    create/
    detail/
  components/
  stores/
```

Each feature owns its API client, models, and state. There is no shared God service for OCI resources.

### 5.4 State management

- Angular signals for synchronous state.
- Feature-local stores for resource lists, filters, and selections.
- `SessionStore` for the simulated user, region, and compartment.
- `WorkRequestStore` for active work requests.
- No NgRx. The global state is small and does not justify the ceremony.

### 5.5 Forms

Creation and edit flows use Signal Forms.

Pattern:

```text
Form model (signals)
  -> per-step validation
  -> cross-field validation
  -> review step
  -> submit
  -> work request tracking
  -> redirect to detail
```

Validation rules mirror the backend. The backend remains authoritative.

### 5.6 HTTP layer

- `httpResource` for reactive reads.
- `HttpClient` for mutations.
- `API_BASE_URL` injection token.
- `ociErrorInterceptor` normalizes errors.
- `apiBaseUrlInterceptor` prefixes server-side requests.
- Transfer cache avoids duplicate fetches after hydration.
- Pagination uses `opc-next-page`.

### 5.7 Routing

Top-level routes:

```text
/login
/
/console/dashboard
/console/compute/**
/console/network/**
/console/storage/**
/console/iam/**
/console/object-storage/**
/console/load-balancer/**
/console/api-gateway/**
/console/dns/**
/console/database/**
/console/work-requests/**
/console/emulator/**
/docs/**
/help/**
/labs
/about
/**  -> 404
```

Feature routes are lazy loaded.

### 5.8 Guards

- `authGuard` protects `/console/**`.
- `guestGuard` protects `/login`.
- Both read the `emu_session` cookie through `SessionService`.
- The Node SSR server also performs an HTTP redirect for unauthenticated `/console/**` requests.

### 5.9 i18n

- Transloco with a static loader.
- English and Spanish.
- Locale resolution from cookie, query, and `Accept-Language`.
- Server and client read the same cookie to avoid hydration mismatch.
- `FormatService` uses `Intl` with the active locale for dates and numbers.
- Translation files are bundled, not fetched at runtime.

### 5.10 Theming

- Angular Material with a custom theme.
- SCSS design tokens.
- No default Material purple.
- Light theme in version 1.
- Chart colors read from CSS variables.
- Fonts are self-hosted.

### 5.11 Charts

- ng2-charts wraps Chart.js.
- Only the required Chart.js controllers and scales are registered.
- Charts are client-only.
- Dashboard charts use `@defer` and fixed-height skeletons.
- Each chart has an accessible data table alternative.

### 5.12 UI shell

- `AppShell` for instant navigation.
- Top bar, sidebar, breadcrumbs, and content outlet.
- Work request tray.
- Toast notifications.
- Global search dialog.
- Confirmation dialog.
- Page header component.
- Empty, loading, and error states.

---

## 6. Rendering and hydration

### 6.1 Rendering modes

| Route                                | Mode | Reason                               |
| ------------------------------------ | ---- | ------------------------------------ |
| `/login`                             | SSR  | Read the session cookie and redirect |
| `/` and dashboard                    | SSR  | Initial shell and KPI data           |
| Resource lists and details           | SSR  | Shell and initial data               |
| Create and edit wizards              | CSR  | Forms and browser interactions       |
| Object browser                       | CSR  | File API and downloads               |
| Request inspector                    | CSR  | Polling and live updates             |
| Docs, help, labs catalog, about, 404 | SSG  | Static content                       |
| Fallback                             | CSR  | Safe default                         |

### 6.2 Server rendering

The Angular server bundle is built with `@angular/ssr` and served by a Node process.

```text
Browser
  |
  | GET /console/network/vcns
  v
Quarkus :4599
  |
  | UiProxy
  v
Node SSR :4600
  |
  | HTTP GET http://127.0.0.1:4599/20160918/vcns
  v
Quarkus API
  |
  v
Rendered HTML + serialized transfer state
```

### 6.3 API origin

```ts
{
  provide: API_BASE_URL,
  useFactory: () =>
    isPlatformBrowser(inject(PLATFORM_ID))
      ? ''
      : 'http://127.0.0.1:4599',
}
```

The browser uses relative URLs. The server uses an absolute URL. The transfer cache maps the server origin to the public origin:

```ts
{
  provide: HTTP_TRANSFER_CACHE_ORIGIN_MAP,
  useValue: { 'http://127.0.0.1:4599': 'http://localhost:4599' },
}
```

### 6.4 Hydration

- `provideClientHydration(withEventReplay(), withIncrementalHydration())`.
- Event replay captures user events before hydration completes.
- Incremental hydration activates deferred blocks on demand.
- The app shell is rendered with `withAppShell`.
- Browser-only components use `@defer` and `isPlatformBrowser`.

### 6.5 SSG rules

- Prerendered routes must be data-independent.
- Docs, help, labs catalog, and about use static content.
- The default prerender locale is English.
- Language switching happens after hydration for static pages.
- SSR pages read the locale cookie and render the correct language.

### 6.6 Hydration safety rules

- Do not access browser globals during server rendering.
- Do not generate random values during render.
- Do not render dates relative to the current time without a stable server value.
- Use the same locale and session state on server and client.
- Keep chart placeholders at the same height as the hydrated chart.

---

## 7. Session and authentication model

### 7.1 Goals

- Provide an OCI-like sign-in experience.
- Keep the API usable by the CLI without a browser session.
- Avoid any real credential handling.

### 7.2 Session cookie

Cookie name: `emu_session`.

Payload:

```json
{
  "tenancyId": "ocid1.tenancy.oc1..emulator",
  "userId": "ocid1.user.oc1..emulator",
  "displayName": "Local Admin",
  "region": "us-ashburn-1"
}
```

Encoding: base64 JSON.

Attributes:

- `Path=/`
- `SameSite=Lax`
- `Max-Age=28800`
- Not `HttpOnly`, because the Angular client reads it.

The cookie is not signed and is not a security token. It only controls the web console experience.

The local OCI CLI profile uses the same default user OCID (`ocid1.user.oc1..emulator`). The web console and the CLI describe the same local identity.

### 7.3 Endpoints

| Endpoint                    | Behavior                                                                                |
| --------------------------- | --------------------------------------------------------------------------------------- |
| `POST /_emu/session/login`  | Accepts any non-empty tenancy, user, and password, sets the cookie, returns the session |
| `GET /_emu/session`         | Returns the current session or `401`                                                    |
| `POST /_emu/session/logout` | Clears the cookie                                                                       |

### 7.4 CLI access

The OCI API endpoints do not require a session. The CLI authenticates with an OCI signature that is parsed but not verified. This separation is intentional:

- Browser users get a realistic login flow.
- CLI users practice real commands without browser state.
- SDK users can call the API directly.

### 7.5 Redirects

- `server.ts` redirects unauthenticated `/console/**` requests to `/login`.
- `server.ts` redirects authenticated `/login` requests to `/`.
- Angular guards handle client-side navigation.
- `make run-lite` relies on client-side guards only.

---

## 8. API design

### 8.1 Compatibility principle

The emulator implements the OCI wire protocol for supported services. It does not expose a separate convenience API for the web console. The web console uses the same endpoints as the CLI.

Emulator-specific behavior is isolated under `/_emu`.

### 8.2 Version prefixes

| Service                      | Prefix                                 |
| ---------------------------- | -------------------------------------- |
| Core and Autonomous Database | `/20160918`                            |
| Load Balancer                | `/20200501`                            |
| API Gateway                  | `/20190501`                            |
| DNS                          | `/20180115`                            |
| MySQL                        | `/20190415`                            |
| Object Storage               | `/n/{namespace}/b/{bucket}/o/{object}` |

### 8.3 Response headers

- `opc-request-id`: unique request identifier.
- `opc-next-page`: pagination token.
- `opc-work-request-id`: asynchronous operation identifier.
- `etag`: resource version.
- `location`: created resource URL where applicable.

### 8.4 Pagination

Pagination uses OCI query parameters:

- `limit`
- `page`
- `sortBy`
- `sortOrder`

Responses include `opc-next-page` when more results exist. The frontend passes the token back unchanged.

### 8.5 Concurrency

ETags support optimistic concurrency for services that expose mutable resources. `if-match` is validated when present. The emulator follows OCI behavior for mismatches.

### 8.6 Error model

All errors use the OCI envelope:

```json
{
  "code": "Conflict",
  "message": "The resource is not in a deletable state.",
  "status": 409,
  "opc-request-id": "..."
}
```

Common codes:

- `NotAuthorizedOrNotFound`
- `InvalidParameter`
- `Conflict`
- `LimitExceeded`
- `InternalError`

### 8.7 CORS

The web console is served from the same origin as the API, so CORS is not required in the default topology. The backend keeps a permissive local CORS filter for development and SDK experiments.

---

## 9. Data architecture

### 9.1 Storage backend

The emulator uses the floci storage abstraction. The default mode is persistent, with data under `./data`.

Storage responsibilities:

- Serialize resources as JSON.
- Scope keys by tenancy and region.
- Store Object Storage bytes.
- Support reset and seed.
- Provide consistent reads and writes for a single local process.

### 9.2 Logical data model

```text
Tenancy
  +-- Region
        +-- Compartment
              +-- VCN
              |     +-- Subnet
              |     +-- Route table
              |     +-- Security list
              |     +-- Network security group
              |     +-- Gateways
              +-- Instance
              |     +-- VNIC
              |     +-- Volume attachment
              +-- Volume
              |     +-- Backup
              +-- Bucket
              |     +-- Object
              |     +-- PAR
              +-- Load balancer
              |     +-- Listener
              |     +-- Backend set
              |     +-- Backend
              +-- API Gateway
              |     +-- Deployment
              |     +-- Route
              +-- DNS zone
              |     +-- Record
              +-- Autonomous Database
              +-- MySQL DB system
              +-- Work request
```

### 9.3 Object Storage bytes

Object bytes are stored by the Object Storage backend, not in the JSON metadata store. Multipart uploads assemble parts before publishing the final object.

### 9.4 Reset and seed

Reset removes all resources and recreates the default local tenancy. Seed creates a demo topology with resources across all supported services.

Both operations are exposed through `/_emu` and are safe for local use only.

### 9.5 Export and import

Export produces a ZIP archive containing the persistent data and a manifest. Import validates the manifest and replaces the current state. Import is not transactional across services, so a failed import may leave partial state. The recommended workflow is to stop the server before import.

### 9.6 Concurrency

The emulator assumes a single local user and a single Quarkus process. Storage operations are serialized per resource. There is no distributed locking.

---

## 10. Emulator module

Configuration keys, environment variables, and service toggles for this module are documented in [docs/CONFIGURATION.md](docs/CONFIGURATION.md).

### 10.1 Endpoints

| Endpoint                        | Purpose                   |
| ------------------------------- | ------------------------- |
| `POST /_emu/session/login`      | Create a local session    |
| `GET /_emu/session`             | Read the current session  |
| `POST /_emu/session/logout`     | Clear the session         |
| `GET /_emu/config`              | Runtime configuration     |
| `GET /_emu/dashboard`           | Aggregated dashboard data |
| `GET /_emu/requests`            | Request inspector list    |
| `GET /_emu/requests/{id}`       | Request detail            |
| `GET /_emu/search`              | Global search             |
| `GET /_emu/labs`                | Lab catalog               |
| `POST /_emu/labs/{id}/validate` | Lab validation            |
| `POST /_emu/reset`              | Reset state               |
| `POST /_emu/seed`               | Seed demo data            |
| `GET /_emu/export`              | Export state              |
| `POST /_emu/import`             | Import state              |

### 10.2 Dashboard service

The dashboard service aggregates data from the storage backend. It does not call the OCI HTTP API, which avoids unnecessary network hops and keeps the aggregates consistent.

Aggregates:

- Resource counts by service.
- Instances by state.
- Storage bytes by bucket.
- Work requests per day.
- Estimated monthly cost from a local price table.

### 10.3 Request inspector

The inspector is implemented as a Quarkus filter with a bounded ring buffer.

Captured data:

- Request ID.
- Method.
- Path and query.
- Service and operation.
- Status.
- Duration.
- Request headers and body.
- Response headers and body.
- Error code.

Limits:

- 500 entries by default.
- 64 KB per body.
- Binary bodies are represented by size and content type.
- Inspector requests and static assets are excluded.

### 10.4 Search service

Search scans resource names and OCIDs across enabled services. It returns at most 50 results grouped by service. The implementation uses storage indexes where available and falls back to scans for small local datasets.

### 10.5 Lab service

Labs are defined as JSON resources under `src/main/resources/labs`. A lab contains:

- ID, title, and description.
- Prerequisites.
- Steps with instructions and hints.
- CLI equivalents.
- A validator with resource checks.
- A reset scope.

Validation runs against the current storage state, not against request logs.

---

## 11. UI proxy

### 11.1 Purpose

OCI Emulator exposes one public port. Quarkus handles API requests and proxies UI requests to the Node SSR server in production.

### 11.2 Implementation

- Vert.x HTTP proxy from the Quarkus BOM.
- Registered as a fallback route after REST routes.
- Target: `http://127.0.0.1:4600`.
- Active in the `prod` profile when `floci-oci.emulator.ui.proxy.enabled=true`.
- Disabled in dev mode, where Quinoa handles the UI.

### 11.3 Exclusions

The proxy must not intercept:

- `/20160918/**`
- `/20200501/**`
- `/20190501/**`
- `/20180115/**`
- `/20190415/**`
- `/n/**`
- `/_emu/**`
- `/q/**`
- `/health`

Requests to excluded paths are handled by Quarkus or return a Quarkus error.

### 11.4 Header handling

The proxy forwards:

- `Host` as the target host.
- `Cookie`.
- `Accept`.
- `Accept-Language`.
- `User-Agent`.
- `X-Forwarded-*`.
- `Set-Cookie` from the SSR server.

### 11.5 Failure behavior

If the SSR server is unreachable, Quarkus returns `502 Bad Gateway` with a local error page that explains how to start `make run` or use `make run-lite`.

### 11.6 Tests

- API paths are never proxied.
- UI paths are proxied.
- Cookies are forwarded in both directions.
- Binary assets are streamed without corruption.
- A missing SSR server returns a clear error.

---

## 12. Build and deployment

### 12.1 Development build

```text
make dev
  -> mvn quarkus:dev
  -> Quinoa detects console/angular.json
  -> Quinoa starts ng serve with SSR
  -> Quarkus proxies UI requests to 4200
```

### 12.2 Production build

```text
make run
  -> mvn package
       -> Quinoa runs pnpm install and ng build
       -> Angular produces browser and server bundles
  -> Quarkus starts on 4599
  -> Node starts dist/console/server/server.mjs on 4600
  -> Quarkus proxies UI requests to 4600
```

### 12.3 Artifacts

| Artifact               | Path                                     |
| ---------------------- | ---------------------------------------- |
| Quarkus application    | `target/quarkus-app/`                    |
| Angular browser bundle | `console/dist/console/browser/`          |
| Angular server bundle  | `console/dist/console/server/`           |
| SSR entry point        | `console/dist/console/server/server.mjs` |

### 12.4 Quinoa configuration

```yaml
quarkus:
  quinoa:
    ui-dir: console
    package-manager: pnpm
    build-dir: dist/console/browser
    dev-server:
      port: 4200
    enable-ssr-mode: true
```

In production, `just-build=true` builds the UI without serving it. Quinoa's SSR mode is used for dev proxying. The production UI is served by Node.

### 12.5 Lite build

`make run-lite` sets the UI proxy to disabled and serves the prerendered browser bundle directly from Quarkus. SSR routes fall back to the prerendered shell or CSR.

### 12.6 Native image

The backend can be built as a native executable. The UI and Node SSR server remain separate processes. Native image is optional and not part of the default workflow.

---

## 13. Security model

### 13.1 Scope

OCI Emulator is a local development tool. It is not a security boundary.

### 13.2 Controls

| Control           | Implementation                              |
| ----------------- | ------------------------------------------- |
| Bind address      | `127.0.0.1` by default                      |
| TLS               | Not provided                                |
| OCI signature     | Parsed, not verified                        |
| Web session       | Unsigned local cookie                       |
| Password handling | Ignored, never stored                       |
| Secrets           | None required                               |
| Data access       | Local filesystem only                       |
| Proxy             | Excluded API prefixes and local target only |
| Docker            | Not required                                |

### 13.3 Threats explicitly out of scope

- Remote attackers.
- Multi-tenant isolation.
- Credential theft.
- Privilege escalation.
- Data integrity under concurrent writers.
- Denial of service.
- Production compliance.

### 13.4 Recommendations

- Keep the bind address on `127.0.0.1`.
- Do not expose the emulator to a public network.
- Do not reuse real OCI credentials.
- Do not store sensitive data in the emulator.
- Use the project-local `.oci` profile, not a real profile.

---

## 14. Performance

### 14.1 Targets

| Metric              | Target                                   |
| ------------------- | ---------------------------------------- |
| Initial JS          | Less than 500 KB gzip                    |
| LCP                 | Less than 1.5 seconds locally            |
| CLS                 | Less than 0.1                            |
| API list response   | Less than 100 ms for local datasets      |
| Dashboard aggregate | Less than 200 ms for 10,000 resources    |
| SSR response        | Less than 300 ms for authenticated pages |

### 14.2 Techniques

- Route-level code splitting.
- `@defer` for charts and heavy tabs.
- Server-side pagination.
- Transfer cache for SSR data.
- OnPush change detection.
- Signal-based state to avoid broad re-renders.
- Bounded inspector buffer.
- Indexed storage for list operations.
- No polling when a work request is terminal.

### 14.3 Limits

The emulator is designed for local datasets. Tens of thousands of resources are acceptable. Millions are not. There is no background compaction or distributed indexing.

---

## 15. Observability

### 15.1 Logs

- Quarkus logs to stdout.
- The SSR server logs to stdout.
- Each request has an `opc-request-id`.
- Errors include the request ID.

### 15.2 Health

| Endpoint         | Purpose                           |
| ---------------- | --------------------------------- |
| `/health`        | Quarkus health                    |
| `/q/health`      | Quarkus management health         |
| `/_emu/config`   | Runtime configuration and version |
| `/_emu/requests` | Recent request history            |

### 15.3 Request inspector

The inspector is the primary debugging tool for protocol issues. It shows the exact request and response exchanged with the CLI or the web console.

### 15.4 Work requests

Work requests provide an audit trail for asynchronous operations, including timestamps, status, and affected resources.

---

## 16. Testing architecture

### 16.1 Backend tests

- Unit tests for validation and state transitions.
- RestAssured tests for HTTP behavior.
- Protocol tests for headers, pagination, ETags, and errors.
- Work request tests.
- Compartment usage tests.
- Storage tests with temporary directories.

### 16.2 CLI compatibility tests

Shell scripts run the official CLI against a fresh emulator instance. They cover create, list, get, update, and delete flows. The suite is the primary compatibility gate.

### 16.3 Frontend tests

- Vitest for stores, API clients, validators, and i18n.
- TestBed for components with Material dependencies.
- Mock HTTP with `HttpTestingController`.
- Session and guard tests.

### 16.4 End-to-end tests

Playwright covers:

- Login and logout.
- Redirects for unauthenticated routes.
- VCN creation.
- Instance launch and state transition.
- Bucket creation, upload, download, and PAR.
- Load balancer creation.
- Autonomous Database creation.
- Language switching.
- Reset and seed.
- SSR HTML and hydration checks.

### 16.5 Accessibility tests

axe-core runs on key pages in Playwright. Critical violations fail the build.

### 16.6 Test data

Tests use isolated data directories. E2E tests reset the emulator before each scenario. The CLI compatibility suite uses a dedicated profile and tenancy.

---

## 17. Extension points

Step-by-step checklists for adding services, features, labs, languages, and storage backends are in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md). The sections below summarize the architectural impact.

### 17.1 Adding a backend service

1. Create the service package.
2. Add stored models.
3. Implement the service and validation.
4. Add the controller with the correct version prefix.
5. Implement `CompartmentUsage`.
6. Register a `ServiceDescriptor`.
7. Add configuration.
8. Add tests.
9. Add the CLI compatibility script.
10. Add the dashboard aggregation.
11. Add search indexing.

### 17.2 Adding a frontend feature

1. Create the feature folder.
2. Add routes and lazy loading.
3. Add the API client and models.
4. Add list, create, and detail pages.
5. Add translations.
6. Add navigation metadata.
7. Add Playwright coverage.
8. Add accessibility coverage.

### 17.3 Adding a lab

1. Add a JSON lab definition under `resources/labs`.
2. Define prerequisites, steps, hints, and CLI equivalents.
3. Implement a validator using storage queries.
4. Add the lab to the catalog.
5. Add an E2E test.

### 17.4 Adding a language

1. Add the locale to `LocaleService`.
2. Add the translation JSON file.
3. Add the language selector entry.
4. Run `pnpm i18n:check`.
5. Test SSR and SSG rendering with the new locale.

### 17.5 Adding a storage backend

1. Implement the storage interface.
2. Add a configuration option.
3. Add tests for persistence and scoping.
4. Document the tradeoffs.

---

## 18. Decision log

The decisions with an Architecture Decision Record link to it. The full index is in [docs/adr/README.md](docs/adr/README.md).

| Decision                                                              | Alternatives             | Rationale                                          |
| --------------------------------------------------------------------- | ------------------------ | -------------------------------------------------- |
| [Fork floci-oci](docs/adr/0001-fork-floci-oci.md)                     | Build from scratch in Go | Reuse IAM, Object Storage, OCI protocol primitives |
| Angular 22                                                            | React                    | User preference                                    |
| [Hybrid rendering](docs/adr/0002-hybrid-rendering.md)                 | SPA only or SSR only     | Best fit per route                                 |
| [Node SSR in production](docs/adr/0003-node-ssr-quarkus-proxy.md)     | SSG only                 | Real SSR while keeping one public port             |
| Quarkus UI proxy                                                      | Separate UI port         | Single origin for cookies and API                  |
| Quinoa in dev                                                         | Manual dev proxy         | Faster setup and HMR                               |
| Transloco                                                             | Angular built-in i18n    | Runtime language switching                         |
| [Static translation loader](docs/adr/0004-transloco-static-loader.md) | HTTP loader              | Deterministic SSR and SSG                          |
| Angular Material                                                      | CDK + Tailwind, PrimeNG  | Accessible components and speed                    |
| Signal Forms                                                          | Reactive Forms           | Less boilerplate for wizards                       |
| Signals, no NgRx                                                      | NgRx                     | Small global state                                 |
| ng2-charts and Chart.js                                               | ECharts, ApexCharts      | Angular 22 support and bundle size                 |
| [floci storage](docs/adr/0005-floci-storage.md)                       | SQLite                   | Existing persistence and object bytes              |
| [Simulated login](docs/adr/0006-simulated-login.md)                   | OAuth or no login        | Realistic entry without identity infrastructure    |
| Control plane only                                                    | Data plane               | Scope and Docker constraints                       |
| Own visual identity                                                   | Oracle assets            | Legal and maintainability                          |
| English default, Spanish complete                                     | English only             | Localization practice and local use                |
| Hand-written TypeScript models                                        | Generated SDK            | Smaller surface and less coupling                  |

---

## 19. Known architectural limitations

- Single-process storage assumes one writer.
- The UI proxy adds a network hop in production.
- The Node SSR process is required for full production rendering.
- SSG pages are prerendered in English only.
- The login is cosmetic and not a security boundary.
- IAM policies are not evaluated.
- The inspector is in-memory and loses history on restart.
- Search is a local scan, not a distributed index.
- Export and import are not transactional.
- Terraform compatibility is not tested.
- IPv6 is not supported.
- No TLS.
- No real data plane.

For operational instructions, see [README.md](README.md). For product and UX decisions, see [DESIGN.md](DESIGN.md).
