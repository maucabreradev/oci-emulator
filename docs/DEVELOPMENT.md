# Development

Patterns and conventions for adding backend and frontend code to OCI Emulator.

For the contribution workflow, branches, commits, and test gates, see [CONTRIBUTING.md](../CONTRIBUTING.md). For the runtime architecture, see [ARCHITECTURE.md](../ARCHITECTURE.md). For the contracts every change must respect, see [COMPATIBILITY.md](COMPATIBILITY.md), [PROTOCOL.md](PROTOCOL.md), and [DATA-MODEL.md](DATA-MODEL.md).

---

## 1. Project layout

```text
oci-emulator/
  pom.xml
  src/
    main/
      java/io/floci/oci/
        core/
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
      resources/
        application.yml
        labs/
    test/
      java/io/floci/oci/
  console/
    src/
      app/
        core/
        shared/
        features/
      server.ts
    public/
    angular.json
    package.json
  bin/
    ocilocal
  scripts/
    setup-cli.sh
    compat/
  data/
  docs/
  Makefile
  README.md
  DESIGN.md
  ARCHITECTURE.md
```

---

## 2. Backend pattern

Every service follows the same layers:

```text
Controller -> Service -> Stored model -> Storage -> CompartmentUsage -> ServiceDescriptor -> Config -> Tests
```

Guidelines:

- Controllers parse OCI REST input and return OCI responses.
- Services contain business logic and throw `OciException`.
- Stored models are Jackson-serializable and use `@RegisterForReflection` where required.
- New services use the existing `StorageBackend`, `OciPage`, `Ocids`, `Etags`, and `WorkRequestService`.
- Every service implements `CompartmentUsage` so Identity can block deletion of non-empty compartments.
- Every service registers a `ServiceDescriptor`.
- Every service has an enable flag in `application.yml`.
- New endpoints must preserve OCI wire formats. Do not invent convenience endpoints under a version prefix.
- Emulator-only endpoints belong under `/_emu`.

---

## 3. Frontend pattern

- Angular 22, standalone components, zoneless, OnPush by default.
- Feature folders own their routes, API clients, models, pages, and stores.
- State uses Angular signals and feature-local stores. There is no global NgRx store.
- Forms use Signal Forms.
- Reads use `httpResource`. Mutations use `HttpClient`.
- Tables use server-side pagination with `opc-next-page`.
- All user-visible strings go through Transloco.
- All new pages include loading, empty, and error states.
- All destructive actions require confirmation.
- Charts and browser-only features use `@defer` and platform checks.
- Every detail page exposes JSON and, where applicable, the CLI equivalent.

---

## 4. SSR rules

- Do not access `window`, `document`, `localStorage`, or `navigator` during SSR.
- Use `isPlatformBrowser` or `afterNextRender`.
- Use `API_BASE_URL` for API calls. On the server it points to `http://127.0.0.1:4599`; in the browser it is empty.
- SSG routes must not call the API at build time.
- SSR and client must resolve the same language and session state.
- Test every new SSR route with JavaScript disabled and with hydration enabled.

Rendering mode per route is defined in [ARCHITECTURE.md](../ARCHITECTURE.md) section 6.1.

---

## 5. i18n rules

- Add keys to both `public/i18n/en.json` and `public/i18n/es.json`.
- Use namespaced keys: `feature.section.key`.
- Do not translate OCIDs, API values, resource names, or JSON keys.
- Use ICU syntax for plurals and interpolation.
- Run `pnpm i18n:check` before committing.
- Keep English and Spanish strings close in length to avoid layout shifts.

---

## 6. Adding a service

1. Create the package under `src/main/java/io/floci/oci/services/<service>`.
2. Add the stored models.
3. Implement the service with storage and validation.
4. Add the controller with the correct API version prefix.
5. Implement `CompartmentUsage`.
6. Register a `ServiceDescriptor`.
7. Add the enable flag to `application.yml`.
8. Add backend tests.
9. Add a CLI compatibility script under `scripts/compat`.
10. Add the frontend feature with routes, API client, models, pages, and translations.
11. Add the service to the navigation and dashboard aggregation.
12. Add Playwright coverage for the main flow.

Before starting, add the coverage rows to [COMPATIBILITY.md](COMPATIBILITY.md) and the data model to [DATA-MODEL.md](DATA-MODEL.md).

---

## 7. Adding a frontend feature

1. Add the page inventory entry to [DESIGN.md](../DESIGN.md) if it is a new page type.
2. Add acceptance criteria to [UI-FLOWS.md](UI-FLOWS.md).
3. Create the feature folder under `console/src/app/features/<feature>`.
4. Add lazy routes.
5. Add the API client and models.
6. Add list, create, and detail pages.
7. Add translations to both languages.
8. Add navigation metadata and breadcrumbs.
9. Add Vitest unit tests.
10. Add Playwright coverage for the main flow.
11. Add axe-core checks for the new pages.

---

## 8. Code style

- Java: standard Quarkus style, constructor injection, small services.
- TypeScript: strict mode, no `any`, explicit return types on public APIs.
- Prettier for formatting.
- angular-eslint for frontend linting.
- Conventional Commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`.
- Work on feature branches. Do not push directly to `main`.

---

## 9. Useful commands

| Command         | Purpose                                                       |
| --------------- | ------------------------------------------------------------- |
| `make dev`      | Quarkus dev mode with Quinoa and Angular SSR                  |
| `make run`      | Production Quarkus and Node SSR                               |
| `make run-lite` | Production without Node SSR                                   |
| `make setup`    | Install dependencies and generate local OCI credentials       |
| `make test`     | Backend tests                                                 |
| `make test-ui`  | Frontend unit tests                                           |
| `make e2e`      | Playwright tests                                              |
| `make compat`   | OCI CLI compatibility suite                                   |
| `make verify`   | Format check, lint, backend tests, frontend tests, i18n check |
| `make fmt`      | Format backend and frontend                                   |
| `make lint`     | Lint backend and frontend                                     |
| `make seed`     | Seed demo data                                                |
| `make reset`    | Remove local data                                             |
| `make package`  | Build the release distribution and checksums                  |
| `make clean`    | Remove build outputs                                          |
| `make budget`   | Frontend bundle budget check                                  |

For test scope and gates, see [TESTING.md](TESTING.md) and [CONTRIBUTING.md](../CONTRIBUTING.md) section 6.
