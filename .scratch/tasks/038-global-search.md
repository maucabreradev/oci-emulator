# 038 - Global search

- Phase: 5
- Status: Pending
- Depends on: 035
- Blocks: 042

## Goal

Search resources and documentation from anywhere in the console.

## Scope

- Search endpoint across enabled services.
- Resource matching by name and OCID.
- Documentation entries.
- Quick actions.
- `Ctrl+K` dialog.
- Keyboard navigation.
- Recent searches.

## Deliverables

- `SearchService` and `/_emu/search`.
- Global search dialog.
- Search result grouping.
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] `Ctrl+K` opens the dialog from any console page.
- [ ] Results are grouped by service and show name, OCID, state, and compartment.
- [ ] Documentation and quick actions are included.
- [ ] Arrow keys and Enter navigate results.
- [ ] Escape closes the dialog and restores focus.
- [ ] Empty results suggest removing filters or searching by OCID.
- [ ] Search returns at most 50 results.
- [ ] The dialog is accessible by keyboard and screen readers.
- [ ] Search does not block the main thread on large datasets.

## Verification

```bash
make test-ui
make e2e
curl -sS 'http://localhost:4599/_emu/search?q=demo' | jq .
```

## Out of scope

- Full-text search across object contents.
- Saved searches.
- Search analytics.
