# 043 - Phase 5 verification and documentation sync

- Phase: 5
- Status: Pending
- Depends on: 042
- Blocks: 044, 046

## Goal

Close emulator features with green gates and updated contracts.

## Scope

- Run `make verify`, `make compat`, and `make e2e`.
- Verify dashboard, inspector, search, labs, JSON, CLI, tags, and filters.
- Update `docs/UI-FLOWS.md` if behavior changed.
- Update `STATE.md`.
- Record deviations.

## Deliverables

- Green suites.
- Accessibility and performance report.
- Updated `STATE.md`.
- Phase summary.

## Acceptance criteria

- [ ] The four labs pass end to end.
- [ ] Dashboard aggregates match the resource counts.
- [ ] Search returns grouped results with keyboard navigation.
- [ ] Inspector copy actions produce usable commands.
- [ ] JSON and CLI equivalents exist on every applicable page.
- [ ] No critical accessibility violations.
- [ ] Performance budgets are met or documented with a plan.
- [ ] `STATE.md` points to the next task.

## Verification

```bash
make verify
make compat
make e2e
```

## Out of scope

- New services.
- Release packaging.
