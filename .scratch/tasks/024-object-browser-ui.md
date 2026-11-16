# 024 - Object browser UI

- Phase: 2
- Status: Pending
- Depends on: 023
- Blocks: 025

## Goal

Provide a usable object browser for real files.

## Scope

- Bucket list, create, and detail.
- Object list with prefix navigation.
- Upload by file picker and drag and drop.
- Folder creation.
- Download.
- Delete.
- Multipart progress and cancellation.
- List and grid views.

## Deliverables

- `features/object-storage/buckets`
- `features/object-storage/objects`
- Upload queue component.
- Object browser table and grid.
- Translations.
- Playwright coverage with file fixtures.

## Acceptance criteria

- [ ] The upload flow in `docs/UI-FLOWS.md` passes.
- [ ] Drag and drop highlights the drop zone.
- [ ] Uploads show per-file progress and can be cancelled.
- [ ] Large files use multipart upload.
- [ ] Downloads return the original bytes.
- [ ] Prefix navigation updates the breadcrumb path.
- [ ] Empty folders show an empty state.
- [ ] Object names with slashes are handled correctly.
- [ ] The browser works with CSR and does not run during SSR.
- [ ] Every state has loading, empty, and error handling.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Object versioning.
- Archive restore.
- Bulk move.
