# 041 - JSON view, CLI equivalent, tags, and filters

- Phase: 5
- Status: Pending
- Depends on: 036
- Blocks: 042

## Goal

Make the API visible and the resource lists consistent across the console.

## Scope

- JSON view on every detail page.
- CLI equivalent on every create and update flow.
- Tags editor for freeform and defined tags.
- Consistent list filters and URL state.
- Row actions and bulk actions where supported.
- Copy actions and toasts.

## Deliverables

- Shared JSON viewer.
- Shared CLI equivalent component.
- Shared tag editor.
- Shared filter bar.
- Translations.
- Tests.

## Acceptance criteria

- [ ] Every detail page exposes the raw API response.
- [ ] JSON is syntax highlighted, searchable, copyable, and downloadable.
- [ ] Create and update flows show a CLI command using `bin/ocilocal`.
- [ ] Placeholders in CLI commands are visually marked.
- [ ] Tags can be added, edited, and removed.
- [ ] Active filters are shown as removable chips.
- [ ] Filters are reflected in the URL.
- [ ] Empty search results differ from empty resource lists.
- [ ] Copy actions show a toast.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Saved views.
- Advanced query language.
- Bulk actions for services that do not support them.
