# 037 - Request inspector UI

- Phase: 5
- Status: Pending
- Depends on: 003, 035
- Blocks: 042

## Goal

Make the request inspector usable for learning and debugging.

## Scope

- Request table with filters.
- Live tail toggle.
- Detail drawer with headers and bodies.
- Copy as CLI and copy as curl.
- Request ID display.
- Truncation and binary body notices.

## Deliverables

- `features/emulator/inspector`
- Filters, table, and drawer components.
- CLI and curl generators.
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The inspector lists the last 500 requests, newest first.
- [ ] Filters cover service, method, status, and free text.
- [ ] Live tail follows new requests and can be paused.
- [ ] The drawer shows method, path, status, duration, headers, bodies, and `opc-request-id`.
- [ ] Truncated and binary bodies are clearly indicated.
- [ ] Copy as CLI and copy as curl produce usable commands.
- [ ] Inspector requests and static assets are excluded.
- [ ] Polling stops when the tab is hidden.
- [ ] The page works with CSR only.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Request replay.
- Persistent history.
- HAR export.
