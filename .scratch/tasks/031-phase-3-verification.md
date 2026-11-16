# 031 - Phase 3 verification and documentation sync

- Phase: 3
- Status: Pending
- Depends on: 028, 029, 030
- Blocks: 032, 034

## Goal

Close developer services with green gates and updated contracts.

## Scope

- Run `make verify`, `make compat`, and `make e2e`.
- Fix defects in load balancer, API Gateway, and DNS.
- Update compatibility rows.
- Update `STATE.md`.
- Record deviations.

## Deliverables

- Green suites.
- Updated `docs/COMPATIBILITY.md`.
- Updated `STATE.md`.
- Phase summary.

## Acceptance criteria

- [ ] Every Phase 3 operation is verified with the official CLI.
- [ ] Load balancer, API Gateway, and DNS flows pass end to end.
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
- Data plane work.
