# 002 - Backend configuration and storage baseline

- Phase: 0
- Status: Pending
- Depends on: 001
- Blocks: 003, 004, 005

## Goal

Make runtime configuration and storage predictable in development, test, and production.

## Scope

- Define `application.yml` with host, port, storage, emulator, and service toggles.
- Support environment variable overrides with the `FLOCI_OCI_*` prefix.
- Support persistent and in-memory storage modes.
- Seed the local tenancy on first start.
- Disable services that require Docker by default.

## Deliverables

- `src/main/resources/application.yml`
- Configuration mapping classes.
- Storage bootstrap for `./data`.
- Local tenancy seed.
- Tests for configuration overrides and persistence.

## Acceptance criteria

- [ ] Defaults bind to `127.0.0.1:4599` and store data in `./data`.
- [ ] `FLOCI_OCI_PORT`, `FLOCI_OCI_STORAGE_MODE`, and `FLOCI_OCI_STORAGE_DIR` override the defaults.
- [ ] Resources survive a restart in persistent mode.
- [ ] Tests use an isolated temporary directory or in-memory mode.
- [ ] Disabled services are not registered and return an OCI-shaped error.
- [ ] First start creates the local tenancy `ocid1.tenancy.oc1..emulator`.

## Verification

```bash
FLOCI_OCI_PORT=4699 mvn quarkus:dev
curl -sS http://localhost:4699/_emu/config
```

## Out of scope

- UI settings.
- Export and import.
- Multi-region seeding beyond the default region.
