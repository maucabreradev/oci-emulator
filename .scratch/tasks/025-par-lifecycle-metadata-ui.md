# 025 - PAR, lifecycle, and object metadata UI

- Phase: 2
- Status: Pending
- Depends on: 023, 024
- Blocks: 026

## Goal

Expose sharing, lifecycle, and object metadata in the console.

## Scope

- PAR create, list, copy URL, and revoke.
- Lifecycle policy create, edit, and delete.
- Object metadata drawer.
- Object JSON view.
- Object tags.

## Deliverables

- PAR pages and dialogs.
- Lifecycle policy editor.
- Object detail drawer.
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The PAR flow in `docs/UI-FLOWS.md` passes.
- [ ] The PAR URL is shown once with a copy action.
- [ ] Revoking requires confirmation and updates the state.
- [ ] Expired PARs are visually distinct.
- [ ] Lifecycle policies validate action, amount, and unit.
- [ ] The object drawer shows size, content type, ETag, metadata, and storage tier.
- [ ] The JSON view matches the API response.
- [ ] The UI warns that lifecycle policies are not executed.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Object versioning.
- Cross-region replication.
- Lifecycle execution.
