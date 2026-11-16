# Agent Guide

Operational rules for any coding agent working in this repository. Read this file first, then the linked documents. Do not duplicate their content here; link to them.

## 0. Session start

Before writing or editing code, do this in order:

1. Read `STATE.md`. Identify the active task, the current phase, and any blocker or open question.
2. Open the active task file under `.scratch/tasks/`. Read its Goal, Scope, Acceptance criteria, and Verification before planning.
3. Read the contracts the task touches: `docs/COMPATIBILITY.md`, `docs/PROTOCOL.md`, `docs/DATA-MODEL.md`, `docs/UI-FLOWS.md`, `DESIGN.md`, or `ARCHITECTURE.md` as relevant. Do not code against an outdated model.
4. If the change is non-trivial, present a plan and wait for approval before implementing. Use Plan mode for multi-file or architectural work.
5. Confirm the definition of done from the task file and from `CONTRIBUTING.md` section 7 before starting.

Before declaring a task done:

1. Run the verification block of the task file and the checks in section 10.
2. Update the task status and the counters in `STATE.md` following its section 6.
3. Move the completed task to `STATE.md` section 4 with a short summary.
4. Set the next task in `STATE.md` header and section 5.
5. Record any new decision as an ADR in `docs/adr/` and link it from `ARCHITECTURE.md` section 18.
6. Do not commit or push unless the user explicitly asks.

## 1. Project identity

- OCI Emulator is a local emulator of the OCI control plane with a web console, OCI CLI support, and SDK support.
- Backend: Java 25 + Quarkus 3.40.1.1 on top of floci-oci 0.7.0 with the floci persistent storage backend.
- Frontend: Angular 22.2.6 with SSR, SSG, and CSR per route.
- One public port `4599`. Node SSR runs internally on `4600` behind a Quarkus UI proxy.
- Control plane only. No data plane, no TLS, no multi-tenant isolation.

## 2. Stack and versions

| Component              | Version         |
| ---------------------- | --------------- |
| Java (Temurin)         | 25.0.4          |
| Quarkus                | 3.40.1.1        |
| Maven                  | 3.9.9           |
| floci-oci              | 0.7.0           |
| Angular / CLI          | 22.2.6 / 22.2.7 |
| TypeScript             | 6.0.3           |
| Node.js                | 24.19.0         |
| pnpm                   | 12.14.0         |
| Transloco              | 8.4.0           |
| ng2-charts / Chart.js  | 11 / 4.6.1      |
| Angular Material / CDK | 22.2.7          |
| Vitest                 | 5.1.0           |
| Playwright             | 1.65.1          |
| Quinoa                 | 2.9.3           |

Do not upgrade versions or add dependencies without asking the user first.

## 3. Commands

The `Makefile` and source trees do not exist yet. Task 001 creates them. Until then, treat these as planned:

- `make setup`: tool checks and dependency installation.
- `make dev`: Quarkus dev plus Angular dev server on `4200`.
- `make run`: full build and start on `4599`.
- `make run-lite`: no-Node fallback serving the prerendered and client-rendered build.
- `make test`: backend, frontend, and integration tests.
- `make test-ui`: frontend unit tests.
- `make e2e`: Playwright end-to-end tests.
- `make compat`: official OCI CLI compatibility suite.
- `make verify`: typecheck, lint, tests, and build.
- `make fmt`, `make lint`: format and lint the backend and frontend.
- `make clean`, `make reset`, `make seed`, `make package`, `make budget`.
- `mvn -q package`: backend build.
- `cd console && pnpm budget`: bundle budget check.

Only document a command as available after its task is `Done`. Do not invent new targets.

## 4. Repository layout

- Root: `pom.xml`, `Makefile`, `.gitignore`.
- Backend: `src/main/java/io/floci/oci/`, `src/main/resources/`, `src/test/java/io/floci/oci/`.
- Frontend: `console/` with the Angular workspace under `console/src/app/`.
- Labs: `src/main/resources/labs/`.
- Local data: `./data`, ignored by Git.
- Documentation: `docs/`, `docs/adr/`, `docs/examples/`.
- Tasks: `.scratch/tasks/` (`000` through `050`).
- Progress board: `STATE.md`.

## 5. Backend conventions

- Package root is `io.floci.oci`. Do not change it.
- Service pattern: Controller -> Service -> Stored model -> Storage -> CompartmentUsage -> ServiceDescriptor -> Config -> Tests.
- Every registrable service implements `ServiceDescriptor` and is toggled in `application.yml`.
- Disabled by default: Queue, Streaming, KMS, Vault, Organizations, Governance Rules, Functions, OKE, PostgreSQL. Do not implement them in version 1.
- Storage: floci persistent backend in `./data`. No SQLite. Supported modes are `persistent` and `in-memory`.
- OCIDs use `ocid1.<type>.<realm>..<unique>`. The emulator accepts the expected resource type without requiring a region in the unique part.
- Local tenancy: `ocid1.tenancy.oc1..emulator`. Local user: `ocid1.user.oc1..emulator`, shared by the CLI and the web console.
- Default region: `us-ashburn-1`.
- Endpoint prefixes: Core `/20160918`, Load Balancer `/20200501`, API Gateway `/20190501`, DNS `/20180115`, MySQL `/20190415`, Object Storage `/n/{namespace}/b/{bucket}/o/{object}`, emulator `/_emu`.
- Errors use the OCI envelope with `code`, `message`, and `opc-request-id`. Unknown routes return `NotAuthorizedOrNotFound`. Validation errors return `InvalidParameter` naming the field and the failed rule.
- Pagination defaults to limit 10, capped at 100.
- Request inspector keeps the last 500 requests and truncates bodies to 64 KB. It excludes its own requests and static assets.

## 6. Frontend conventions

- Angular 22 standalone components, signals, and no NgRx.
- Wizards use Signal Forms, not Reactive Forms.
- i18n uses Transloco with a static loader. Keys: `emu.locale`, `emu.region`, `emu.compartment`. English default, complete Spanish.
- Charts use ng2-charts and Chart.js.
- Components use Angular Material and CDK.
- Rendering by route: SSG for docs, help, labs, about, and 404. SSR for login, dashboard, lists, and details. CSR for wizards, object browser, and inspector.
- SSR rule: browser requests use relative URLs; server requests use `http://127.0.0.1:4599`.
- Development uses Quinoa with `ng serve` on `4200`. Production uses Node SSR on `4600` behind the Quarkus proxy.
- Budgets: initial JS under 500 KB gzip, LCP under 1.5 s, CLS under 0.1.
- Accessibility target: WCAG 2.2 AA, checked with axe-core.

## 7. Session and authentication

- Cookie `emu_session` holds base64 JSON. Attributes: `Path=/`, `SameSite=Lax`, `Max-Age=28800`, not `HttpOnly`, not signed.
- Login is at `/login`. Any non-empty credentials are accepted. Demo values: `ociemu` / `admin` / `emulator`. The CLI bypasses login.
- The cookie controls only the console experience. It is not a security token.

## 8. Hard rules

- No Oracle assets: logos, fonts, icons, screenshots, or trademarks.
- No SQLite. Use floci storage.
- No TLS, no multi-tenant isolation, no data plane, no IPv6 networking, no Terraform, no IAM policy evaluation.
- Do not edit generated files, `node_modules`, `vendor`, or lockfiles unless asked.
- Do not expose ports `4599` or `4600` to public interfaces.
- Disabled services must not register and must return an OCI-shaped error.

## 9. Task workflow

- Tasks are atomic files in `.scratch/tasks/NNN-<slug>.md` with Goal, Scope, Deliverables, Acceptance criteria, Verification, and Out of scope.
- One task is `In progress` at a time unless tasks are explicitly parallelizable.
- A task is `Done` only when its acceptance criteria and verification pass.
- Update `STATE.md` at the end of every iteration using the rules in its section 6.
- Record new decisions as an ADR in `docs/adr/` and link it from the decision log in `ARCHITECTURE.md` section 18.
- Record blockers in `STATE.md` section 4 and keep the task status.

## 10. Verification before done

- Backend: `mvn -q package` passes and relevant tests pass.
- Frontend: typecheck, lint, tests, build, and `pnpm budget` pass.
- Documentation: internal links and anchors resolve, JSON examples parse, no em dashes, no leftover names.
- Do not claim a task is done without running its verification block.

## 11. References

Do not duplicate these documents. Link to them:

- User and developer guide: `README.md`
- Product and UX design: `DESIGN.md`
- Technical architecture: `ARCHITECTURE.md`
- Web console guide: `docs/CONSOLE.md`
- OCI CLI guide: `docs/CLI.md`
- Configuration reference: `docs/CONFIGURATION.md`
- Development guide: `docs/DEVELOPMENT.md`
- Testing and quality: `docs/TESTING.md`
- Troubleshooting: `docs/TROUBLESHOOTING.md`
- Compatibility matrix: `docs/COMPATIBILITY.md`
- Wire protocol: `docs/PROTOCOL.md`
- Data model: `docs/DATA-MODEL.md`
- UI flows: `docs/UI-FLOWS.md`
- Labs: `docs/LABS.md`
- Release guide: `docs/RELEASE.md`
- Architecture decisions: `docs/adr/`
- Contribution guide: `CONTRIBUTING.md`
- Security policy: `SECURITY.md`
- Progress board: `STATE.md`
