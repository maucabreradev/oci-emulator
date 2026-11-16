# Contributing to OCI Emulator

This document defines how to work on OCI Emulator: setup, branch conventions, test gates, code patterns, and the definition of done.

For end-user instructions, see [README.md](README.md). For architecture, see [ARCHITECTURE.md](ARCHITECTURE.md). For product and UX rules, see [DESIGN.md](DESIGN.md).

---

## 1. Before you start

### 1.1 Read the contracts

Every change must respect these documents:

| Document                                       | What it fixes                                                |
| ---------------------------------------------- | ------------------------------------------------------------ |
| [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md) | Which operations are supported and at what status            |
| [docs/PROTOCOL.md](docs/PROTOCOL.md)           | Wire-level rules: headers, pagination, errors, work requests |
| [docs/DATA-MODEL.md](docs/DATA-MODEL.md)       | Resource fields, relationships, states, delete constraints   |
| [DESIGN.md](DESIGN.md)                         | UX, visual identity, i18n, accessibility                     |

If a change requires a new rule, update the document in the same pull request. Do not leave the contracts behind the code.

### 1.2 Environment

Follow the setup steps in [README.md](README.md):

```bash
make setup
make dev
```

Required versions:

- Java 25.
- Maven 3.9.9.
- Node.js 24.19.0.
- pnpm 12.14.0.
- uv.
- Python 3.12 (installed by uv).
- OpenSSL 3.x.
- make and git.

The repository pins the Java version in `.java-version`. The version is the requirement; use any version manager that reads it, or install Temurin 25.0.4 manually.

### 1.3 Scope check

Before writing code, confirm:

1. The operation is listed in `docs/COMPATIBILITY.md`.
2. The resource model is described in `docs/DATA-MODEL.md`.
3. The UI flow has acceptance criteria in `docs/UI-FLOWS.md`, or you add them.
4. The task exists in `.scratch/tasks/`, or you add it.

If any of these is missing, stop and discuss the change first.

### 1.4 Local tooling and secrets

Project configuration for coding tools is committed when it contains no secrets:

- `opencode.jsonc` declares the project MCP servers and is versioned.
- `AGENTS.md` is the agent guide and is versioned.
- `.scratch/tasks/`, `STATE.md`, and `docs/` are part of the project record and are versioned.
- `.java-version` pins the Java version and is versioned.

Rules:

- Never put tokens, API keys, passwords, or private keys in any tracked file.
- Use `{env:NAME}` substitution in `opencode.jsonc` and keep the value in your shell or a local `.env` file.
- Document every required environment variable in `.env.example` without values.
- Add new local-only files to `.gitignore` in the same change.
- If a tool configuration must hold a secret, ignore the real file and commit a `.example` version instead.

---

## 2. Branch and commit conventions

### 2.1 Branches

Create branches from `main`:

```text
feat/<short-description>
fix/<short-description>
refactor/<short-description>
docs/<short-description>
test/<short-description>
chore/<short-description>
```

Never push directly to `main`. Never force-push a shared branch.

### 2.2 Commits

Use Conventional Commits:

```text
feat(network): add VCN create and list endpoints
fix(object-storage): return Conflict when deleting a non-empty bucket
refactor(console): move instance list state into a feature store
docs(protocol): document work request error envelope
test(cli): cover compartment delete blocking
chore(deps): upgrade Angular to 22.2.7
```

Rules:

- One logical change per commit.
- Imperative mood in the subject.
- Include a body when the reason is not obvious.
- Reference the task file or issue when applicable.
- Do not commit generated output, credentials, or local data.

### 2.3 Pull requests

A pull request should include:

- A short description of the behavior change.
- The task or contract it implements.
- Test evidence, including the commands you ran.
- Screenshots only for UI changes, using the project's own branding. Do not include Oracle assets.
- Notes about deviations from OCI, if any.

---

## 3. Code patterns

Backend, frontend, rendering, and i18n patterns are documented in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md). Read it before adding code.

Non-negotiables:

- Keep the `io.floci.oci` package root.
- Put emulator-only code under `io.floci.oci.emulator` or a service package.
- Use constructor injection, thin controllers, and services that own validation and state.
- Use Angular standalone components, signals, Signal Forms, and Transloco.
- Route every user-visible string through Transloco.
- Do not touch browser globals during SSR.
- Add keys to both `console/public/i18n/en.json` and `console/public/i18n/es.json`.

---

## 4. Adding a backend service

Follow the checklist in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) section 6. Before starting, add the coverage rows to [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md) and the data model to [docs/DATA-MODEL.md](docs/DATA-MODEL.md).

Acceptance for a new service:

- The official CLI performs create, list, get, update, and delete.
- Headers, pagination, and error envelopes match [docs/PROTOCOL.md](docs/PROTOCOL.md).
- Compartment deletion is blocked when the service has live resources.
- The compatibility suite passes.

---

## 5. Adding a frontend feature

Follow the checklist in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) section 7. Add acceptance criteria to [docs/UI-FLOWS.md](docs/UI-FLOWS.md) and a page inventory entry to [DESIGN.md](DESIGN.md) if it is a new page type.

Acceptance for a new feature:

- List, create, detail, and destructive actions work.
- Empty, loading, and error states are present.
- Keyboard navigation works.
- The page renders with SSR and hydrates without warnings.
- English and Spanish are complete.
- Initial JS budget is respected.

---

## 6. Test gates

### 6.1 Commands

| Command        | Scope                                                         |
| -------------- | ------------------------------------------------------------- |
| `make test`    | Backend unit and integration tests                            |
| `make test-ui` | Frontend unit tests                                           |
| `make e2e`     | Playwright end-to-end tests                                   |
| `make compat`  | Official OCI CLI compatibility suite                          |
| `make verify`  | Format check, lint, backend tests, frontend tests, i18n check |

`make verify` is the minimum gate before requesting review. Test layers, coverage, accessibility, and performance budgets are documented in [docs/TESTING.md](docs/TESTING.md).

### 6.2 What to run for each change

| Change                        | Required checks                   |
| ----------------------------- | --------------------------------- |
| Backend service or controller | `make test`, `make compat`        |
| Frontend component or store   | `make test-ui`                    |
| SSR, routing, or guards       | `make test-ui`, `make e2e`        |
| Translations                  | `make test-ui`, `pnpm i18n:check` |
| Documentation only            | Link check and Markdown review    |
| Configuration or build        | `make verify`                     |

### 6.3 Test rules

- Every bug fix includes a regression test.
- Every new endpoint has a protocol test.
- Every new page has at least one Playwright scenario.
- Tests must not depend on the developer's local data directory.
- E2E tests reset state before each scenario.
- CLI compatibility tests use the local profile and a dedicated tenancy.

---

## 7. Definition of done

A task is done when all of the following are true:

- Behavior matches the task file and the contracts.
- Backend and frontend tests pass.
- The CLI compatibility suite passes for the affected operations.
- `make verify` passes.
- English and Spanish translations are complete.
- Accessibility checks pass for new pages.
- Documentation is updated in the same pull request.
- No secrets, local data, or generated files are staged.
- The task file is updated and referenced in `STATE.md`.

A task is not done when:

- It works only in the browser or only with the CLI.
- It relies on undocumented deviations.
- It leaves TODOs in user-visible behavior.
- It breaks SSR or hydration.
- It adds untranslated strings.

---

## 8. Upstream policy

OCI Emulator is a fork of [floci-oci](https://github.com/floci-io/floci-oci).

To keep merges feasible:

- Keep the `io.floci.oci` package namespace.
- Put emulator-specific code under `io.floci.oci.emulator`.
- Do not rename shared primitives.
- Do not reformat upstream files without a functional change.
- Keep emulator configuration under the `floci-oci.emulator` prefix.
- When an upstream change conflicts with an emulator decision, document the decision in an ADR.

Upstream sync procedure:

1. Fetch `upstream`.
2. Create a branch from `main`.
3. Merge or rebase `upstream/main`.
4. Resolve conflicts in favor of the documented emulator contracts.
5. Run `make verify` and `make compat`.
6. Open a pull request with a summary of upstream changes and conflict resolutions.

---

## 9. Prohibited changes

Do not commit:

- `.oci/`
- `.venv/`
- `data/`
- `.env` files
- private keys or certificates
- generated build output
- IDE-specific files
- Oracle logos, fonts, icons, screenshots, or other proprietary assets
- secrets, tokens, or credentials in `opencode.jsonc` or any other tracked configuration

The authoritative ignore rules live in [.gitignore](.gitignore). Update that file and this list together when you add a new local-only artifact.

Do not:

- Add dependencies without discussion.
- Change CI, release, or infrastructure configuration without review.
- Expose the emulator on a public interface.
- Reuse real OCI credentials.
- Bypass a permission prompt or a deny rule.

---

## 10. Reporting problems

- Functional bugs: open an issue with steps, expected behavior, actual behavior, and the `opc-request-id` when available.
- Security concerns: follow [SECURITY.md](SECURITY.md). Do not open a public issue.
- Compatibility gaps: update `docs/COMPATIBILITY.md` with the failing command and observed response, then open an issue.

---

## 11. License

By contributing, you agree that your contributions are licensed under the MIT License. See [LICENSE](LICENSE).
