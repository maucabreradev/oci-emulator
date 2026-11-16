# 035 - Phase 4 verification and documentation sync

- Phase: 4
- Status: Pending
- Depends on: 033, 034
- Blocks: 036, 037, 038, 039

## Goal

Close databases with green gates and updated contracts.

## Scope

- Run `make verify`, `make compat`, and `make e2e`.
- Fix defects in ADB and MySQL.
- Update compatibility rows.
- Update `STATE.md`.
- Record deviations.

## Deliverables

- Green suites.
- Updated `docs/COMPATIBILITY.md`.
- Updated `STATE.md`.
- Phase summary.

## Acceptance criteria

- [ ] Every Phase 4 operation is verified with the official CLI.
- [ ] ADB and MySQL lifecycle flows pass end to end.
- [ ] Wallet download and backup flows pass.
- [ ] Compartment deletion is blocked by each new resource type.
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
- Database data plane.
