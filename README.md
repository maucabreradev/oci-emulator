# OCI Emulator

A local Oracle Cloud Infrastructure emulator with a complete web console and compatibility with the official OCI CLI and SDKs.

OCI Emulator gives you a realistic OCI control plane on your machine. No Oracle Cloud account, subscription, API key, or credit card is required. The web console reproduces the information architecture and workflows of OCI Console with its own visual identity. The OCI CLI and SDKs can point at the same local API.

> **Disclaimer**
>
> OCI Emulator is an independent project. It is not affiliated with, endorsed by, or sponsored by Oracle Corporation. Oracle, Oracle Cloud, and OCI are trademarks of Oracle Corporation. This project does not include Oracle logos, fonts, icons, screenshots, or other proprietary assets.
>
> OCI Emulator is a development and learning tool. It is not intended for production workloads, and it does not provide security, isolation, or availability guarantees.

---

## Table of contents

- [1. What OCI Emulator is](#1-what-oci-emulator-is)
- [2. Features](#2-features)
- [3. What OCI Emulator is not](#3-what-oci-emulator-is-not)
- [4. Requirements](#4-requirements)
- [5. Quick start](#5-quick-start)
- [6. Web console guide](#6-web-console-guide)
- [7. OCI CLI guide](#7-oci-cli-guide)
- [8. Service coverage](#8-service-coverage)
- [9. API compatibility](#9-api-compatibility)
- [10. Configuration](#10-configuration)
- [11. Data and persistence](#11-data-and-persistence)
- [12. Emulator tools](#12-emulator-tools)
- [13. Development guide](#13-development-guide)
- [14. Testing and quality](#14-testing-and-quality)
- [15. Troubleshooting](#15-troubleshooting)
- [16. Limitations](#16-limitations)
- [17. Contributing](#17-contributing)
- [18. Further reading](#18-further-reading)
- [19. License and attribution](#19-license-and-attribution)

---

## 1. What OCI Emulator is

OCI Emulator is a Java and Angular application that emulates the OCI control plane locally.

It has three clients:

| Client      | Purpose                                       |
| ----------- | --------------------------------------------- |
| Web console | Visual OCI-like experience in the browser     |
| OCI CLI     | Official `oci` commands against the local API |
| OCI SDKs    | Programmatic access with a custom endpoint    |

The web console is not a terminal. It is a full web application served at `http://localhost:4599`. It includes a simulated sign-in page, dashboard, service navigation, resource lists, creation wizards, detail pages, charts, work requests, and learning tools.

The API is served by Quarkus on the same port. The CLI and the web console use the same backend and the same data.

```text
Browser  ->  Angular web console  ->  Quarkus API  ->  local storage
Terminal ->  OCI CLI              ->  Quarkus API  ->  local storage
```

---

## 2. Features

### Control plane services

- Compute: instances, images, shapes, VNICs, console history.
- Networking: VCNs, subnets, internet gateways, NAT gateways, service gateways, route tables, security lists, network security groups, DHCP options, private IPs, public IPs.
- Block Storage: volumes, attachments, backups.
- IAM: compartments, users, groups, memberships, policies, dynamic groups.
- Object Storage: buckets, objects, multipart uploads, pre-authenticated requests, lifecycle policies.
- Load Balancer: load balancers, listeners, backend sets, backends, health checks.
- API Gateway: gateways, deployments, routes.
- DNS: zones and records.
- Autonomous Database: lifecycle, connection strings, wallet, backups.
- MySQL: DB systems, endpoints, backups.
- Work requests with polling and state transitions.

### Web console

- Simulated sign-in with any non-empty credentials.
- Region and compartment selectors.
- Dashboard with resource counts and charts.
- Service menu, favorites, recent resources, and breadcrumbs.
- Resource lists with filters, pagination, status badges, and row actions.
- Creation wizards with validation, review, JSON preview, and the equivalent CLI command.
- Detail pages with tabs, tags, metadata, related resources, and raw JSON.
- Work request panel with status, logs, and affected resources.
- Global search with `Ctrl+K`.
- English and Spanish UI with runtime language switching.
- Accessible components and keyboard navigation.
- Responsive layout with desktop-first behavior.

### Learning tools

- Request inspector with method, path, status, duration, headers, bodies, and `opc-request-id`.
- Copy any inspected request as a CLI command or `curl` command.
- Guided labs with steps, hints, CLI equivalents, and automatic validation.
- Seed and reset demo data.
- Export and import local state.
- JSON view on every resource detail page.

### Developer experience

- Java 25 and Quarkus backend.
- Angular 22 frontend with SSR, SSG, and CSR per route.
- Persistent local storage in `./data`.
- Official OCI CLI installed in a project-local Python virtual environment.
- No Docker required.
- Test suites for backend, frontend, CLI compatibility, and browser flows.

---

## 3. What OCI Emulator is not

- It is not a real cloud.
- It does not run real compute workloads, containers, or databases.
- It does not provide SSH access to instances.
- It does not route network traffic.
- It does not resolve DNS records.
- It does not execute load balancer traffic.
- It does not evaluate IAM policies for authorization.
- It does not provide TLS.
- It does not provide tenant isolation or security boundaries.
- It does not support Terraform in version 1.
- It does not support IPv6 in version 1.
- It is not a pixel-perfect copy of OCI Console and does not use Oracle assets.

The simulated sign-in is cosmetic. It controls which screen the web console shows. The OCI API accepts unsigned requests so the CLI can work without a session.

---

## 4. Requirements

### Required

| Tool    | Version            | Notes                                                              |
| ------- | ------------------ | ------------------------------------------------------------------ |
| Java    | 25                 | Temurin 25.0.4 is the reference runtime, pinned by `.java-version` |
| Maven   | 3.9.9              | Backend build                                                      |
| Node.js | 24.19.0            | Angular requires `^22.22.3 || ^24.15.0 || >=26`                    |
| pnpm    | 12.14.0            | Frontend package manager                                           |
| uv      | Latest             | Installs Python and the OCI CLI locally                            |
| Python  | 3.12               | Installed by uv inside the project                                 |
| OpenSSL | 3.x                | Generates the local RSA key                                        |
| make    | Any recent version | Task runner                                                        |
| git     | Any recent version | Source control                                                     |

### Optional

| Tool                                 | Purpose                                   |
| ------------------------------------ | ----------------------------------------- |
| Playwright browsers                  | Browser end-to-end tests                  |
| IDE with Java and TypeScript support | Development                               |
| `jq`                                 | Inspecting JSON responses in the terminal |

Linux, macOS, and WSL2 on Windows are supported. Docker is not required and is not used by the default configuration. Internet access is required only to download dependencies during setup.

---

## 5. Quick start

### 5.1 Clone the repository

```bash
git clone git@github.com:maucabreradev/oci-emulator.git
cd oci-emulator
```

### 5.2 Bootstrap the project

```bash
make setup
```

`make setup` performs the following steps:

1. Checks Java, Maven, Node, pnpm, uv, and OpenSSL versions.
2. Installs frontend dependencies with pnpm.
3. Creates `.venv` and installs the official OCI CLI with uv.
4. Generates `.oci/key.pem`, `.oci/key_public.pem`, and `.oci/config`.
5. Builds the backend and runs the base test suite.

The generated `.oci` files are local development credentials. They are ignored by Git. The server does not verify the signature.

### 5.3 Start in development mode

```bash
make dev
```

This starts Quarkus on `http://localhost:4599` and Quinoa on `4200` with the Angular dev server. Quarkus proxies the UI. Open `http://localhost:4599`.

### 5.4 Start in production mode

```bash
make run
```

This builds the application and starts Quarkus on `4599` plus the Angular SSR Node server on `127.0.0.1:4600` behind the Quarkus UI proxy. Open `http://localhost:4599`.

If you do not want to run the Node SSR process, use:

```bash
make run-lite
```

This serves the prerendered and client-rendered build directly from Quarkus. It is useful for environments without Node, but SSR is disabled.

### 5.5 Sign in

Open `http://localhost:4599`. You will be redirected to `/login`.

Demo credentials:

```text
Tenancy:  ociemu
User:     admin
Password: emulator
```

You can use any non-empty values. Alternatively, click **Use local profile**. The emulator does not validate the password and does not store it.

### 5.6 Verify the CLI

In a second terminal:

```bash
bin/ocilocal iam region list
bin/ocilocal os ns get
```

If both commands return data, the CLI is configured correctly.

### 5.7 Stop the emulator

Press `Ctrl+C` in the terminal running `make dev` or `make run`.

---

## 6. Web console guide

The console includes a sign-in page, top bar with region and compartment selectors, grouped service menu, dashboard with charts, resource lists with filters and row actions, creation wizards, tabbed detail pages, a work request panel, a request inspector, guided labs, and global search with `Ctrl+K`.

Wizards show a review step with the equivalent OCI CLI command and a JSON preview. Every detail page exposes the raw JSON response. The dashboard and charts load only in the browser and use fixed-height skeletons during hydration.

The full guide, including every page and setting, is in **[docs/CONSOLE.md](docs/CONSOLE.md)**. Visual and UX rules are in [DESIGN.md](DESIGN.md). Per-flow acceptance criteria are in [docs/UI-FLOWS.md](docs/UI-FLOWS.md).

---

## 7. OCI CLI guide

`make setup` creates `.venv`, `.oci/config`, a local RSA key pair, and `bin/ocilocal`. The wrapper points the official CLI at `http://localhost:4599` with the `OCIEMU` profile.

```bash
bin/ocilocal <service> <command> [options]
```

Common starting points:

```bash
bin/ocilocal iam region list
bin/ocilocal os ns get
```

The full command reference, profile details, wait patterns, and CLI troubleshooting are in **[docs/CLI.md](docs/CLI.md)**.

---

## 8. Service coverage

| Service                        | Coverage                                                                       | Notes                                            |
| ------------------------------ | ------------------------------------------------------------------------------ | ------------------------------------------------ |
| Identity and Access Management | Compartments, users, groups, memberships, policies, dynamic groups             | Policies are stored and displayed, not evaluated |
| Object Storage                 | Buckets, objects, multipart uploads, PARs, lifecycle                           | Object bytes are stored locally                  |
| Core: Networking               | VCNs, subnets, gateways, route tables, security lists, NSGs, DHCP options, IPs | IPv4 only                                        |
| Core: Compute                  | Instances, images, shapes, VNICs, console history                              | Control plane only                               |
| Core: Block Storage            | Volumes, attachments, backups                                                  | No real block device                             |
| Load Balancer                  | Load balancers, listeners, backend sets, backends, health checks               | No traffic forwarding                            |
| API Gateway                    | Gateways, deployments, routes                                                  | No request routing                               |
| DNS                            | Zones, records                                                                 | No DNS resolution                                |
| Autonomous Database            | Lifecycle, connection strings, wallet, backups                                 | No database engine                               |
| MySQL                          | DB systems, endpoints, backups                                                 | No database engine                               |
| Work Requests                  | List, get, logs, resources                                                     | Polling and state transitions                    |

Services disabled by default: Queue, Streaming, KMS, Vault, Organizations, Governance Rules, Functions, OKE, and PostgreSQL. They require Docker sidecars or are outside the version 1 scope.

The operation-level matrix, known deviations, and out-of-scope list are in **[docs/COMPATIBILITY.md](docs/COMPATIBILITY.md)**. Service toggles are in [docs/CONFIGURATION.md](docs/CONFIGURATION.md) section 4.

---

## 9. API compatibility

| Service                      | Prefix                                 |
| ---------------------------- | -------------------------------------- |
| Core and Autonomous Database | `/20160918`                            |
| Load Balancer                | `/20200501`                            |
| API Gateway                  | `/20190501`                            |
| DNS                          | `/20180115`                            |
| MySQL                        | `/20190415`                            |
| Object Storage               | `/n/{namespace}/b/{bucket}/o/{object}` |
| Emulator                     | `/_emu`                                |

The CLI and SDKs add the service version prefix automatically when the endpoint is set to `http://localhost:4599`.

Common response headers are `opc-request-id`, `opc-next-page`, `opc-work-request-id`, and `etag`. Request headers include `if-match` and `if-none-match`.

Errors use the OCI shape:

```json
{
  "code": "NotAuthorizedOrNotFound",
  "message": "The requested resource was not found.",
  "status": 404,
  "opc-request-id": "20261116T180000Z-..."
}
```

The API accepts signed and unsigned requests. The signature is parsed but not verified. The web console calls the API without signing. The CLI works without the simulated web session. There is no TLS, and the server binds to `127.0.0.1` by default.

OCI SDKs that support custom service endpoints can point at the emulator. Terraform is not supported in version 1.

Wire-level rules, pagination, ETags, work requests, and the full error envelope are in **[docs/PROTOCOL.md](docs/PROTOCOL.md)**.

---

## 10. Configuration

Defaults live in `src/main/resources/application.yml`. The main settings are:

- `FLOCI_OCI_PORT`: public API and UI port, default `4599`.
- `FLOCI_OCI_STORAGE_MODE`: `persistent` or `in-memory`.
- `FLOCI_OCI_STORAGE_DIR`: data directory, default `./data`.
- `FLOCI_OCI_MAX_REQUEST_SIZE`: maximum request body in MB, default `64`.
- `FLOCI_OCI_EMULATOR_INSPECTOR_CAPACITY`: inspector ring buffer size, default `500`.
- `FLOCI_OCI_EMULATOR_SESSION_TTL`: simulated session lifetime, default `8h`.

Example:

```bash
FLOCI_OCI_PORT=4699 make dev
```

UI preferences use `emu.locale`, `emu.region`, `emu.compartment`, and `emu_session`. Clearing site data resets them and signs you out.

The full configuration reference, service toggles, Node SSR variables, profiles, and storage modes are in **[docs/CONFIGURATION.md](docs/CONFIGURATION.md)**.

---

## 11. Data and persistence

All emulator state lives in `./data` by default and is managed by the floci storage backend. Do not edit it by hand while the server is running.

Backup:

```bash
cp -a data data.backup
```

Reset (stop the emulator first):

```bash
make reset
```

Seed demo data:

```bash
make seed
```

Export and import:

```bash
curl -o oci-emulator-export.zip http://localhost:4599/_emu/export
curl -X POST --data-binary @oci-emulator-export.zip \
  -H 'Content-Type: application/zip' \
  http://localhost:4599/_emu/import
```

Export and import are intended for local backups and lab snapshots, not for production migration. The seed creates the root tenancy, `demo` and `sandbox` compartments, a VCN, an instance, a volume, a bucket with objects, a load balancer, an API Gateway, a DNS zone, an Autonomous Database, a MySQL DB system, IAM resources, and one completed work request per asynchronous operation.

The resource model, storage keys, and seed details are in **[docs/DATA-MODEL.md](docs/DATA-MODEL.md)**.

---

## 12. Emulator tools

- **Request inspector**: last 500 requests, method, path, status, duration, headers, bodies truncated to 64 KB, and `opc-request-id`. Copy any request as CLI or `curl`.
- **Global search**: `Ctrl+K` across resource names and OCIDs.
- **Labs**: guided scenarios validated against real emulator state.
- **CLI equivalent**: every create or update action shows the matching `oci` command.
- **JSON view**: raw API response on every detail page.
- **Seed and reset**: populate or clear the demo environment from Settings or the HTTP API.

Guided scenarios are defined in **[docs/LABS.md](docs/LABS.md)**. UI acceptance criteria are in [docs/UI-FLOWS.md](docs/UI-FLOWS.md).

---

## 13. Development guide

Backend code follows Controller -> Service -> Stored model -> Storage -> CompartmentUsage -> ServiceDescriptor -> Config -> Tests. Frontend code uses Angular standalone components, signals, feature folders, Signal Forms, Transloco, and server-side pagination.

Keep the `io.floci.oci` package namespace to ease upstream merges. Emulator-specific code goes under `io.floci.oci.emulator` or a service package. Every service implements `CompartmentUsage` and registers a `ServiceDescriptor`.

Project layout, backend and frontend patterns, SSR rules, i18n rules, how to add a service or feature, code style, and useful commands are in **[docs/DEVELOPMENT.md](docs/DEVELOPMENT.md)**. The contribution workflow is in [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 14. Testing and quality

```bash
make test      # backend
make test-ui   # frontend unit
make e2e       # Playwright
make compat    # OCI CLI compatibility
make verify    # format, lint, tests, i18n
```

`make verify` is the required check before merging. The suite covers backend unit and integration tests, CLI compatibility, frontend unit tests, end-to-end flows, accessibility with axe-core, and SSR and SSG rendering.

Targets: WCAG 2.2 AA, initial JavaScript under 500 KB gzip, LCP under 1.5 seconds locally, and CLS under 0.1.

Test layers, compatibility coverage, accessibility requirements, and performance budgets are in **[docs/TESTING.md](docs/TESTING.md)**.

---

## 15. Troubleshooting

Common issues:

- **Port `4599` busy**: stop the process or set `FLOCI_OCI_PORT`.
- **Wrong Java version**: `java -version` and `mvn -version` must report Java 25.
- **Quinoa does not start**: check Node and pnpm, then run `pnpm install` in `console/`.
- **Proxy error in production**: the Node SSR server is not running on `4600`; use `make run` or `make run-lite`.
- **Login redirect loop**: use one host, allow cookies, and clear `emu_session`.
- **CLI config errors**: use `bin/ocilocal` or set `OCI_CLI_CONFIG_FILE`, `OCI_CLI_PROFILE`, and `OCI_CLI_ENDPOINT`.

The full list, including WSL2, data permissions, and upload size limits, is in **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**.

---

## 16. Limitations

- Control plane only. No real compute, storage, network, DNS, or database engines.
- IAM policies are stored but not evaluated.
- No Terraform support in version 1.
- No IPv6.
- No TLS.
- No real authentication or tenant isolation.
- The simulated login is cosmetic.
- Some niche OCI endpoints are not implemented.
- Load balancer health checks and database states are simulated.
- Object Storage stores real bytes locally, but there is no replication or durability guarantee.
- The web console imitates OCI workflows but does not use Oracle assets and is not a pixel-perfect copy.

---

## 17. Contributing

Create a feature branch from `main`, keep changes focused, add tests, update the documentation, run `make verify`, and use Conventional Commits. Do not commit `.oci/`, `.venv/`, `data/`, `.env` files, generated build output, or secrets.

The full workflow, branch and commit conventions, test gates, code patterns, and definition of done are in **[CONTRIBUTING.md](CONTRIBUTING.md)**.

---

## 18. Further reading

- [Web console guide](docs/CONSOLE.md)
- [OCI CLI guide](docs/CLI.md)
- [Configuration](docs/CONFIGURATION.md)
- [Development guide](docs/DEVELOPMENT.md)
- [Testing and quality](docs/TESTING.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Compatibility matrix](docs/COMPATIBILITY.md)
- [Wire protocol](docs/PROTOCOL.md)
- [Data model](docs/DATA-MODEL.md)
- [UI flows](docs/UI-FLOWS.md)
- [Labs](docs/LABS.md)
- [Release guide](docs/RELEASE.md)
- [Architecture decisions](docs/adr/)
- [Contributing](CONTRIBUTING.md)
- [Security policy](SECURITY.md)
- [Project state](STATE.md)
- [Product and UX design](DESIGN.md)
- [Technical architecture](ARCHITECTURE.md)

---

## 19. License and attribution

OCI Emulator is released under the MIT License. See [LICENSE](LICENSE).

The backend is based on [floci-oci](https://github.com/floci-io/floci-oci), also MIT licensed. See the upstream project for its architecture and service patterns.

Fonts:

- Inter, SIL Open Font License 1.1.
- JetBrains Mono, SIL Open Font License 1.1.

Icons:

- Lucide, ISC License.

Oracle, Oracle Cloud, and OCI are trademarks of Oracle Corporation. OCI Emulator is not affiliated with Oracle.
