# 011 - Phase 0 UI slice

- Phase: 0
- Status: Pending
- Depends on: 006, 008, 009, 010
- Blocks: 012

## Goal

Deliver the first complete console experience on top of the Phase 0 backend.

## Scope

- Dashboard shell with KPI counts and empty state.
- VCN list, create wizard, and detail page.
- Compartment selector.
- Work request panel.
- JSON view and CLI equivalent for VCN.
- Loading, empty, and error states.

## Deliverables

- `features/dashboard`
- `features/network/vcns`
- Compartment selector in the top bar.
- Work request store and panel.
- JSON view component.
- CLI equivalent component.
- Playwright coverage for the flow.

## Acceptance criteria

- [ ] The VCN flow in `docs/UI-FLOWS.md` passes.
- [ ] The create wizard validates name and CIDR and shows the CLI equivalent.
- [ ] A created VCN appears in the list and on its detail page.
- [ ] The detail page shows default route table, security list, and DHCP options.
- [ ] API errors show the OCI code, message, and request ID.
- [ ] Empty states offer a primary next action.
- [ ] The dashboard shows zero values without errors.
- [ ] The UI works with SSR and hydrates without warnings.
- [ ] English and Spanish strings are complete.

## Verification

```bash
make e2e
```

## Out of scope

- Charts.
- Subnet creation.
- Bulk actions.
