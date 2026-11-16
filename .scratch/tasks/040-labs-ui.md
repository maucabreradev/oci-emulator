# 040 - Labs UI

- Phase: 5
- Status: Pending
- Depends on: 039
- Blocks: 042

## Goal

Provide the guided lab experience in the console.

## Scope

- Lab catalog with difficulty, duration, and completion state.
- Lab detail with goal, prerequisites, steps, hints, and CLI equivalents.
- Validate action and per-check results.
- Reset action with confirmation.
- Completion stored in local storage.

## Deliverables

- `features/emulator/labs`
- Catalog and detail pages.
- Check result component.
- Translations.
- Playwright coverage for one complete lab.

## Acceptance criteria

- [ ] The lab flow in `docs/UI-FLOWS.md` passes.
- [ ] Completion is stored in local storage.
- [ ] Failed checks explain what is missing.
- [ ] Reset requires confirmation and explains what will be removed.
- [ ] The lab does not claim to validate untested properties.
- [ ] CLI equivalents are copyable.
- [ ] The page works with CSR only.
- [ ] Every state has loading, empty, and error handling.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Server-side completion tracking.
- Lab authoring UI.
- Certificates.
