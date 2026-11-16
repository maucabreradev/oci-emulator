# 045 - Reset and seed hardening

- Phase: 6
- Status: Pending
- Depends on: 044
- Blocks: 048

## Goal

Make reset and seed deterministic and safe.

## Scope

- Reset behavior while the server is running.
- Seed idempotency or reset-then-seed.
- Seed coverage across all supported services.
- CLI and UI actions.
- Confirmation requirements.
- Tests against a fresh and a seeded environment.

## Deliverables

- Hardened `/_emu/reset` and `/_emu/seed`.
- Deterministic seed data.
- UI confirmations.
- `make seed` and `make reset` behavior.
- Tests.

## Acceptance criteria

- [ ] Seed creates the documented resources from `docs/DATA-MODEL.md`.
- [ ] Running seed twice follows the documented behavior without duplicates.
- [ ] Reset removes all resources and recreates the local tenancy.
- [ ] Reset warns when the server is running and writes are possible.
- [ ] Seed data is deterministic across runs.
- [ ] UI actions require confirmation.
- [ ] CLI actions are safe when the server is stopped.
- [ ] Tests cover fresh, seeded, reset, and reseeded states.

## Verification

```bash
make reset
make dev
make seed
curl -sS http://localhost:4599/_emu/dashboard | jq .
```

## Out of scope

- Partial tenant reset.
- Scheduled reset.
- Seed snapshots per lab.
