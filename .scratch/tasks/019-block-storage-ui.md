# 019 - Block Storage UI

- Phase: 1
- Status: Pending
- Depends on: 015, 017
- Blocks: 020

## Goal

Expose volumes, attachments, and backups in the console.

## Scope

- Volume list, create, and detail.
- Attach and detach flows.
- Resize flow.
- Backup create, list, restore, and delete.
- Volume tab on the instance detail page.

## Deliverables

- `features/storage/volumes`
- `features/storage/backups`
- Attachment dialog.
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The volume flow in `docs/UI-FLOWS.md` passes.
- [ ] Size validation matches the backend.
- [ ] The attach dialog lists only compatible instances.
- [ ] Attach and detach show a work request.
- [ ] A volume cannot be deleted while attached without explicit confirmation.
- [ ] Backups appear on the volume detail page.
- [ ] The instance detail page lists attached volumes.
- [ ] Every page has loading, empty, and error states.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Real block devices.
- Volume performance metrics.
- Cross-region backups.
