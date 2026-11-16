# 033 - Autonomous Database UI

- Phase: 4
- Status: Pending
- Depends on: 032
- Blocks: 035

## Goal

Expose Autonomous Database lifecycle and connection information.

## Scope

- ADB list, create wizard, and detail.
- Start, stop, restart, clone, and terminate actions.
- Connection strings tab.
- Wallet download action.
- Backups tab.
- Work request tracking.

## Deliverables

- `features/database/autonomous`
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The ADB flow in `docs/UI-FLOWS.md` passes.
- [ ] The wizard validates DB name, OCPU, storage, and password.
- [ ] The review step shows the CLI equivalent and JSON preview.
- [ ] Lifecycle actions are enabled only in valid states.
- [ ] Connection strings are visible and labeled simulated.
- [ ] The wallet download works and explains its contents.
- [ ] Backups can be created and deleted from the detail page.
- [ ] The UI states that no database engine runs.
- [ ] Every page has loading, empty, and error states.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- SQL worksheet.
- Performance metrics.
- Data import.
