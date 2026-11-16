# Project State

Canonical progress board for OCI Emulator. Every iteration updates this file. Task scope and acceptance criteria live in [`.scratch/tasks/`](.scratch/tasks/).

- Last updated: 2026-11-17
- Current phase: Phase 0 - Foundation
- Last completed task: [000 - Project documentation baseline](.scratch/tasks/000-project-documentation-baseline.md)
- Next task: [001 - Repository scaffold and build guardrails](.scratch/tasks/001-repository-scaffold.md)
- Overall progress: 1 of 51 tasks complete (2%)
- Status: On track. Documentation baseline complete. Implementation not started.

---

## 1. Status summary

| Field            | Value                                          |
| ---------------- | ---------------------------------------------- |
| Product version  | 0.1.0-SNAPSHOT                                 |
| Backend          | Java 25, Quarkus 3.40.1.1, floci-oci 0.7.0     |
| Frontend         | Angular 22.2.6, Transloco 8.4.0, ng2-charts 11 |
| Public port      | 4599                                           |
| SSR port         | 4600                                           |
| Data directory   | `./data`                                       |
| Default locale   | English                                        |
| Secondary locale | Spanish                                        |

---

## 2. Phase progress

| Phase                               | Tasks  | Done  | Pending | Progress |
| ----------------------------------- | ------ | ----- | ------- | -------- |
| 0 - Foundation                      | 13     | 1     | 12      | 8%       |
| 1 - Core Services                   | 8      | 0     | 8       | 0%       |
| 2 - IAM and Object Storage UI       | 6      | 0     | 6       | 0%       |
| 3 - Load Balancer, API Gateway, DNS | 5      | 0     | 5       | 0%       |
| 4 - Autonomous Database and MySQL   | 4      | 0     | 4       | 0%       |
| 5 - Emulator features               | 8      | 0     | 8       | 0%       |
| 6 - Hardening and release           | 7      | 0     | 7       | 0%       |
| **Total**                           | **51** | **1** | **50**  | **2%**   |

---

## 3. Task index

### Phase 0 - Foundation

| Task                                                        | Title                                           | Status  |
| ----------------------------------------------------------- | ----------------------------------------------- | ------- |
| [000](.scratch/tasks/000-project-documentation-baseline.md) | Project documentation baseline                  | Done    |
| [001](.scratch/tasks/001-repository-scaffold.md)            | Repository scaffold and build guardrails        | Pending |
| [002](.scratch/tasks/002-backend-config-storage.md)         | Backend configuration and storage baseline      | Pending |
| [003](.scratch/tasks/003-protocol-core-inspector.md)        | OCI protocol core and request inspector backend | Pending |
| [004](.scratch/tasks/004-cli-bootstrap.md)                  | CLI bootstrap and smoke profile                 | Pending |
| [005](.scratch/tasks/005-angular-quinoa.md)                 | Angular workspace and Quinoa integration        | Pending |
| [006](.scratch/tasks/006-design-system-shell.md)            | Design system and application shell             | Pending |
| [007](.scratch/tasks/007-i18n-foundation.md)                | Internationalization foundation                 | Pending |
| [008](.scratch/tasks/008-simulated-session-login.md)        | Simulated session and login page                | Pending |
| [009](.scratch/tasks/009-hybrid-rendering.md)               | Hybrid rendering and data transfer              | Pending |
| [010](.scratch/tasks/010-phase-0-backend-slice.md)          | Phase 0 backend slice                           | Pending |
| [011](.scratch/tasks/011-phase-0-ui-slice.md)               | Phase 0 UI slice                                | Pending |
| [012](.scratch/tasks/012-phase-0-verification.md)           | Phase 0 verification and documentation sync     | Pending |

### Phase 1 - Core Services

| Task                                                             | Title                                                     | Status  |
| ---------------------------------------------------------------- | --------------------------------------------------------- | ------- |
| [013](.scratch/tasks/013-networking-subnets-gateways.md)         | Networking backend: subnets and gateways                  | Pending |
| [014](.scratch/tasks/014-networking-tables-security-dhcp-ips.md) | Networking backend: route tables, security, DHCP, and IPs | Pending |
| [015](.scratch/tasks/015-networking-ui.md)                       | Networking UI                                             | Pending |
| [016](.scratch/tasks/016-compute-backend.md)                     | Compute backend                                           | Pending |
| [017](.scratch/tasks/017-block-storage-backend.md)               | Block Storage backend                                     | Pending |
| [018](.scratch/tasks/018-compute-ui.md)                          | Compute UI                                                | Pending |
| [019](.scratch/tasks/019-block-storage-ui.md)                    | Block Storage UI                                          | Pending |
| [020](.scratch/tasks/020-phase-1-verification.md)                | Phase 1 verification and documentation sync               | Pending |

### Phase 2 - IAM and Object Storage UI

| Task                                                   | Title                                       | Status  |
| ------------------------------------------------------ | ------------------------------------------- | ------- |
| [021](.scratch/tasks/021-iam-backend-completion.md)    | IAM backend completion                      | Pending |
| [022](.scratch/tasks/022-iam-ui.md)                    | IAM UI                                      | Pending |
| [023](.scratch/tasks/023-object-storage-backend.md)    | Object Storage backend completion           | Pending |
| [024](.scratch/tasks/024-object-browser-ui.md)         | Object browser UI                           | Pending |
| [025](.scratch/tasks/025-par-lifecycle-metadata-ui.md) | PAR, lifecycle, and object metadata UI      | Pending |
| [026](.scratch/tasks/026-phase-2-verification.md)      | Phase 2 verification and documentation sync | Pending |

### Phase 3 - Load Balancer, API Gateway, DNS

| Task                                                | Title                                       | Status  |
| --------------------------------------------------- | ------------------------------------------- | ------- |
| [027](.scratch/tasks/027-load-balancer-backend.md)  | Load Balancer backend                       | Pending |
| [028](.scratch/tasks/028-load-balancer-ui.md)       | Load Balancer UI                            | Pending |
| [029](.scratch/tasks/029-api-gateway-backend-ui.md) | API Gateway backend and UI                  | Pending |
| [030](.scratch/tasks/030-dns-backend-ui.md)         | DNS backend and UI                          | Pending |
| [031](.scratch/tasks/031-phase-3-verification.md)   | Phase 3 verification and documentation sync | Pending |

### Phase 4 - Autonomous Database and MySQL

| Task                                              | Title                                       | Status  |
| ------------------------------------------------- | ------------------------------------------- | ------- |
| [032](.scratch/tasks/032-adb-backend.md)          | Autonomous Database backend                 | Pending |
| [033](.scratch/tasks/033-adb-ui.md)               | Autonomous Database UI                      | Pending |
| [034](.scratch/tasks/034-mysql-backend-ui.md)     | MySQL backend and UI                        | Pending |
| [035](.scratch/tasks/035-phase-4-verification.md) | Phase 4 verification and documentation sync | Pending |

### Phase 5 - Emulator features

| Task                                                          | Title                                                   | Status  |
| ------------------------------------------------------------- | ------------------------------------------------------- | ------- |
| [036](.scratch/tasks/036-dashboard-charts.md)                 | Dashboard aggregation and charts                        | Pending |
| [037](.scratch/tasks/037-request-inspector-ui.md)             | Request inspector UI                                    | Pending |
| [038](.scratch/tasks/038-global-search.md)                    | Global search                                           | Pending |
| [039](.scratch/tasks/039-labs-backend.md)                     | Labs backend and catalog                                | Pending |
| [040](.scratch/tasks/040-labs-ui.md)                          | Labs UI                                                 | Pending |
| [041](.scratch/tasks/041-json-cli-tags-filters.md)            | JSON view, CLI equivalent, tags, and filters            | Pending |
| [042](.scratch/tasks/042-a11y-responsive-i18n-performance.md) | Accessibility, responsive, i18n, and performance polish | Pending |
| [043](.scratch/tasks/043-phase-5-verification.md)             | Phase 5 verification and documentation sync             | Pending |

### Phase 6 - Hardening and release

| Task                                                    | Title                                     | Status  |
| ------------------------------------------------------- | ----------------------------------------- | ------- |
| [044](.scratch/tasks/044-export-import.md)              | Export and import                         | Pending |
| [045](.scratch/tasks/045-reset-seed-hardening.md)       | Reset and seed hardening                  | Pending |
| [046](.scratch/tasks/046-ci-verification-gates.md)      | CI pipeline and verification gates        | Pending |
| [047](.scratch/tasks/047-ssr-performance-regression.md) | SSR and performance regression suite      | Pending |
| [048](.scratch/tasks/048-release-packaging.md)          | Release packaging and changelog           | Pending |
| [049](.scratch/tasks/049-final-documentation-rc.md)     | Final documentation and release candidate | Pending |
| [050](.scratch/tasks/050-v1-release.md)                 | Version 1.0.0 release                     | Pending |

---

## 4. Previous iteration

Iteration date: 2026-11-17.

Completed:

- Defined the product, architecture, and stack decisions.
- Created the documentation baseline:
  - `README.md`, `DESIGN.md`, `ARCHITECTURE.md`
  - `CONTRIBUTING.md`, `SECURITY.md`
  - `docs/COMPATIBILITY.md`, `docs/PROTOCOL.md`, `docs/DATA-MODEL.md`
  - `docs/UI-FLOWS.md`, `docs/LABS.md`, `docs/RELEASE.md`
  - ADR template and ADRs 0001 through 0006
  - Request and response examples for VCN, instance, and bucket
- Divided the approved plan into 51 atomic tasks across phases 0 through 6.
- Created this state board.
- Applied the documentation audit corrections:
  - Added database credential rules to the data model and protocol.
  - Unified the default local user OCID across the CLI and the web session.
  - Aligned the JSON examples with the data model fields.
  - Synced the seed list and the disabled services list.
  - Added bucket and Autonomous Database naming rules.
  - Added README cross-links and the ADR index.
  - Linked the ADR-backed decisions in the architecture decision log.
  - Reorganized the documentation:
    - Reduced `README.md` to a landing page with summaries and links.
    - Added `docs/CONSOLE.md`, `docs/CLI.md`, `docs/CONFIGURATION.md`, `docs/DEVELOPMENT.md`, `docs/TESTING.md`, and `docs/TROUBLESHOOTING.md`.
    - Removed duplicated patterns from `CONTRIBUTING.md` and linked to the development guide.

Documentation chores (2026-11-17):

- Aligned all markdown tables across the repository.
- Fixed the article in `CONTRIBUTING.md` ("an emulator decision").
- Reworded `DESIGN.md` section 9.2 to "emulated environment".
- Added the `.java-version` pin (25.0.4) and documented it in `README.md`, `CONTRIBUTING.md`, and `docs/TROUBLESHOOTING.md`.
- Synced the make target list: added `make test-ui`, `make e2e`, `make compat`, `make fmt`, and `make lint` to `AGENTS.md`, and documented `make budget` in `docs/DEVELOPMENT.md`.
- Pinned Maven to 3.9.9 in `README.md` and `CONTRIBUTING.md`.
- Made `opencode.jsonc` strict JSON by removing the dangling comma after `url`.

Not done yet:

- No backend or frontend source code exists.
- The Makefile, `src/`, `console/`, `bin/`, `scripts/`, and `data/` directories do not exist.
- The commands in `README.md` describe the target state and will not work until task 001 and later tasks are complete.

Known open questions:

- None blocking. Any new decision must be recorded in `docs/adr/`.

---

## 5. Next iteration

Next task: [001 - Repository scaffold and build guardrails](.scratch/tasks/001-repository-scaffold.md).

Planned outcome:

- Reproducible Maven and Quarkus build.
- Makefile skeleton and tool checks.
- `.gitignore` for local state and credentials.
- Health endpoint.
- Green `mvn package`.

After 001:

1. 002 - Backend configuration and storage baseline.
2. 003 - OCI protocol core and request inspector backend.
3. 004 - CLI bootstrap and smoke profile.
4. 005 - Angular workspace and Quinoa integration.

---

## 6. How to update this file

At the end of every iteration:

1. Open the completed task file and confirm every acceptance item is checked.
2. Change its status in section 3 from `Pending` to `Done`.
3. Update the phase count and percentage in section 2.
4. Update the overall count and percentage in the header.
5. Move the completed task to section 4 and summarize what changed.
6. Set the next task in the header and in section 5.
7. Add a short note for any blocker, deviation, or new decision.
8. Update the "Last updated" date.

Rules:

- One task is `In progress` at a time unless tasks are explicitly parallelizable.
- A task is `Done` only when its acceptance criteria and verification pass.
- A blocked task keeps its status and records the blocker in section 4.
- Do not mark a phase complete until its verification task is done.
- Link to the task file, not to a line number.

---

## 7. Status definitions

| Status        | Meaning                                             |
| ------------- | --------------------------------------------------- |
| `Pending`     | Not started                                         |
| `In progress` | Actively being implemented                          |
| `Blocked`     | Waiting on an external decision, dependency, or fix |
| `Done`        | Acceptance criteria and verification pass           |
| `Canceled`    | Removed from scope with a documented reason         |

---

## 8. Links

- Task directory: [`.scratch/tasks/`](.scratch/tasks/)
- User and developer guide: [README.md](README.md)
- Product and UX design: [DESIGN.md](DESIGN.md)
- Technical architecture: [ARCHITECTURE.md](ARCHITECTURE.md)
- Compatibility matrix: [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md)
- Wire protocol: [docs/PROTOCOL.md](docs/PROTOCOL.md)
- Data model: [docs/DATA-MODEL.md](docs/DATA-MODEL.md)
- UI flows: [docs/UI-FLOWS.md](docs/UI-FLOWS.md)
- Labs: [docs/LABS.md](docs/LABS.md)
- Release guide: [docs/RELEASE.md](docs/RELEASE.md)
- Architecture decisions: [docs/adr/](docs/adr/)
- Contribution guide: [CONTRIBUTING.md](CONTRIBUTING.md)
- Security policy: [SECURITY.md](SECURITY.md)
