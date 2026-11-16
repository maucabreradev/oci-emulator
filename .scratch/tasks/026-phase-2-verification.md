# 026 - Phase 2 verification and documentation sync

- Phase: 2
- Status: Pending
- Depends on: 022, 025
- Blocks: 027, 029, 030

## Goal

Close IAM and Object Storage with green gates and updated contracts.

## Scope

- Run `make verify`, `make compat`, and `make e2e`.
- Fix defects in identity and object storage.
- Update compatibility rows.
- Update `STATE.md`.
- Record deviations.

## Deliverables

- Green suites.
- Updated `docs/COMPATIBILITY.md`.
- Updated `STATE.md`.
- Phase summary.

## Acceptance criteria

- [ ] Every Phase 2 operation is verified with the official CLI.
- [ ] Object round-trip and multipart tests pass.
- [ ] PAR access and revocation tests pass.
- [ ] Compartment deletion is blocked by identity and storage resources.
- [ ] No critical accessibility violations on the covered pages.
- [ ] SSR and hydration checks pass for the new pages.
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
