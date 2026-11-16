# 044 - Export and import

- Phase: 6
- Status: Pending
- Depends on: 043
- Blocks: 045

## Goal

Provide reliable local backup and restore for emulator state.

## Scope

- Export endpoint producing a ZIP with data and a manifest.
- Import endpoint that validates the manifest and replaces state.
- Schema version checks.
- UI actions in Settings.
- Error handling for invalid or incompatible archives.
- Tests with real archives.

## Deliverables

- `/_emu/export` and `/_emu/import`
- Manifest schema and version.
- Settings UI for export and import.
- Backend and Playwright tests.

## Acceptance criteria

- [ ] Export produces a ZIP with a manifest and the data directory contents.
- [ ] Export never includes `.oci`, `.venv`, or logs.
- [ ] Import validates the manifest before writing.
- [ ] Import rejects an incompatible schema version with a clear error.
- [ ] Import replaces the current state and is not labeled as transactional.
- [ ] A failed import does not leave a partially written archive unnoticed.
- [ ] The UI requires confirmation before import.
- [ ] A round-trip export and import preserves resources and object bytes.

## Verification

```bash
curl -o export.zip http://localhost:4599/_emu/export
curl -X POST --data-binary @export.zip \
  -H 'Content-Type: application/zip' \
  http://localhost:4599/_emu/import
make test
```

## Out of scope

- Cloud backup.
- Incremental export.
- Encryption.
