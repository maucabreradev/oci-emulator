# 009 - Hybrid rendering and data transfer

- Phase: 0
- Status: Pending
- Depends on: 005, 008
- Blocks: 011

## Goal

Configure SSR, SSG, and CSR per route with correct API origins and hydration.

## Scope

- `app.routes.server.ts` with the rendering mode per route.
- `API_BASE_URL` factory for server and browser.
- HTTP transfer cache origin map.
- Client hydration with event replay and incremental hydration.
- Prerender configuration for docs, help, labs catalog, about, and 404.
- SSR redirect rules.

## Deliverables

- `console/src/app/app.routes.server.ts`
- `console/src/app/core/api/api-base-url.ts`
- `console/src/app/app.config.ts`
- `console/src/app/app.config.server.ts`
- Prerender route list

## Acceptance criteria

- [ ] Login, dashboard, lists, and details render with SSR.
- [ ] Wizards, object browser, and inspector render with CSR.
- [ ] Docs, help, labs catalog, about, and 404 are prerendered.
- [ ] SSG routes do not call the API at build time.
- [ ] Browser requests use relative URLs; server requests use `http://127.0.0.1:4599`.
- [ ] The transfer cache maps the server origin to the public origin.
- [ ] Hydration produces no warnings on covered routes.
- [ ] SSR pages render meaningful HTML with JavaScript disabled.

## Verification

```bash
cd console && pnpm build
curl -sS http://localhost:4599/console/dashboard | grep -o '<app-root[^>]*>'
```

## Out of scope

- Feature data loading.
- Static documentation content.
- Performance tuning.
