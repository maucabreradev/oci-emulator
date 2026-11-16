# 018 - Compute UI

- Phase: 1
- Status: Pending
- Depends on: 015, 016
- Blocks: 020

## Goal

Expose the compute lifecycle in the console.

## Scope

- Instance list, create wizard, and detail page.
- Shape and image selection.
- VCN and subnet selection.
- Optional volume attachment.
- Start, stop, reboot, and terminate actions.
- Work request tracking.
- VNIC and console history tabs.

## Deliverables

- `features/compute/instances`
- `features/compute/images`
- `features/compute/shapes`
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The instance flow in `docs/UI-FLOWS.md` passes.
- [ ] Flex shape limits are validated.
- [ ] The review step shows the CLI equivalent and JSON preview.
- [ ] The work request tray opens and follows the launch.
- [ ] Actions are enabled only for valid states.
- [ ] Terminate requires confirmation.
- [ ] Detail tabs show networking, storage, work requests, and JSON.
- [ ] The list supports search, state filter, and pagination.
- [ ] Every page has loading, empty, and error states.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Console terminal access.
- Real metrics.
- Bulk actions.
