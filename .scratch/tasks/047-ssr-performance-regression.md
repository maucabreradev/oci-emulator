# 047 - SSR and performance regression suite

- Phase: 6
- Status: Pending
- Depends on: 046
- Blocks: 048

## Goal

Prevent regressions in rendering, hydration, and performance.

## Scope

- SSR HTML checks for every SSR route.
- Hydration warning detection.
- SSG build checks.
- Redirect checks.
- Bundle size budget.
- LCP and CLS checks.
- Language switch layout stability.
- JavaScript-disabled checks.

## Deliverables

- Playwright SSR suite.
- Bundle size script.
- Performance budget report.
- CI integration.

## Acceptance criteria

- [ ] SSR routes return meaningful HTML without JavaScript.
- [ ] Hydration produces no console warnings on covered routes.
- [ ] SSG routes build without API calls.
- [ ] Unauthenticated and authenticated redirects are covered.
- [ ] Initial JavaScript stays under 500 KB gzip.
- [ ] LCP stays under 1.5 seconds locally.
- [ ] CLS stays under 0.1.
- [ ] Language switching causes no layout shift.
- [ ] Regressions fail CI.

## Verification

```bash
make e2e
cd console && pnpm build && pnpm budget
```

## Out of scope

- Load testing.
- Multi-user concurrency.
- CDN behavior.
