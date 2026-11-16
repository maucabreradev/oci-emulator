# 012 - Phase 0 verification and documentation sync

- Phase: 0
- Status: Pending
- Depends on: 011
- Blocks: 013

## Goal

Close Phase 0 with green gates and updated contracts.

## Scope

- Run the full verification suite.
- Fix defects found by CLI, e2e, SSR, and accessibility checks.
- Update `docs/COMPATIBILITY.md` statuses.
- Update `STATE.md` with the completed phase.
- Record deviations discovered during implementation.

## Deliverables

- Passing `make verify`, `make compat`, and `make e2e`.
- Updated compatibility matrix.
- Updated `STATE.md`.
- A short phase summary in `STATE.md`.

## Acceptance criteria

- [ ] `make verify` passes.
- [ ] `make compat` passes.
- [ ] `make e2e` passes on a fresh data directory.
- [ ] Phase 0 rows are `verified` or documented as `partial` with a reason.
- [ ] The login, VCN, and work request flows pass with JavaScript disabled where applicable.
- [ ] No critical accessibility violations on the covered pages.
- [ ] `STATE.md` points to the next task.

## Verification

```bash
make verify
make compat
make e2e
```

## Out of scope

- New services.
- Performance tuning beyond the documented budget.
