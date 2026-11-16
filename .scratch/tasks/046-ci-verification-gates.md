# 046 - CI pipeline and verification gates

- Phase: 6
- Status: Pending
- Depends on: 043
- Blocks: 047

## Goal

Enforce the quality gates on every change.

## Scope

- CI workflow for backend, frontend, CLI, and e2e.
- Java and Node caching.
- Artifact retention.
- Format, lint, typecheck, test, and i18n checks.
- CLI compatibility job with a fresh emulator.
- Failure reporting.

## Deliverables

- CI configuration.
- Makefile targets used by CI.
- Cached dependency setup.
- Test reports as artifacts.
- Branch protection notes in `CONTRIBUTING.md`.

## Acceptance criteria

- [ ] Every pull request runs format, lint, backend tests, frontend tests, i18n check, and build.
- [ ] CLI compatibility runs on a fresh emulator.
- [ ] E2E runs on a fresh data directory.
- [ ] Failures publish logs and reports.
- [ ] The pipeline completes within a reasonable local-CI budget.
- [ ] Secrets are not required for the default pipeline.
- [ ] A red pipeline blocks merge.

## Verification

```bash
make verify
make compat
make e2e
```

## Out of scope

- Release publishing automation.
- Cross-platform runners beyond Linux.
- Performance benchmarking infrastructure.
