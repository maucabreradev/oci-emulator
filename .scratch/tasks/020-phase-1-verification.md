# 020 - Phase 1 verification and documentation sync

- Phase: 1
- Status: Pending
- Depends on: 018, 019
- Blocks: 021, 023

## Goal

Close Core Services with green gates and updated contracts.

## Scope

- Run `make verify`, `make compat`, and `make e2e`.
- Fix defects across networking, compute, and block storage.
- Update compatibility rows to `verified`.
- Update `STATE.md`.
- Record deviations.

## Deliverables

- Green test suites.
- Updated `docs/COMPATIBILITY.md`.
- Updated `STATE.md`.
- Phase summary.

## Acceptance criteria

- [ ] Every Phase 1 operation is verified with the official CLI.
- [ ] `make verify` passes.
- [ ] `make e2e` passes on a fresh data directory.
- [ ] Compartment deletion is blocked by every Phase 1 resource type.
- [ ] No critical accessibility violations on the covered pages.
- [ ] SSR and hydration checks pass for list and detail pages.
- [ ] `STATE.md` points to the next task.

## Verification

```bash
make verify
make compat
make e2e
```

## Out of scope

- New services.
- Performance work.
