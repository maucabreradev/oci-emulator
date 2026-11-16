# 001 - Repository scaffold and build guardrails

- Phase: 0
- Status: Pending
- Depends on: 000
- Blocks: 002

## Goal

Create a reproducible Java and Angular repository rooted in the floci-oci fork.

## Scope

- Import floci-oci 0.7.0 sources under the `io.floci.oci` package.
- Add the root Maven build and the Makefile skeleton.
- Add `.gitignore` entries for local state and credentials.
- Add version checks for Java, Maven, Node, pnpm, uv, and OpenSSL.
- Add a minimal health endpoint.

## Deliverables

- `pom.xml` with Java 25, Quarkus 3.40.1.1, and the floci dependencies.
- `Makefile` with `setup`, `dev`, `run`, `run-lite`, `test`, `clean`, and `verify` targets.
- `.gitignore` covering `.oci/`, `.venv/`, `data/`, build output, and IDE files.
- `src/main/java/io/floci/oci/` package root.
- A health endpoint at `/health`.

## Acceptance criteria

- [ ] `mvn -q package` succeeds on a clean checkout.
- [ ] `make setup` fails with a clear message when a required tool is missing.
- [ ] `make setup` succeeds when all tools are present.
- [ ] `GET /health` returns `200`.
- [ ] No local credentials or data files are tracked by Git.
- [ ] The package root remains `io.floci.oci`.

## Verification

```bash
mvn -q package
make setup
curl -sS http://localhost:4599/health
git status --short
```

## Out of scope

- Business services.
- Frontend workspace.
- CI configuration.
