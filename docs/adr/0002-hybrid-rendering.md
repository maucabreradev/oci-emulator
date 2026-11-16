# ADR 0002: Hybrid rendering with SSR, SSG, and CSR per route

- Status: Accepted
- Date: 2026-11-16
- Deciders: Project maintainer
- Supersedes: none
- Superseded by: none

## Context

The web console has three different kinds of pages:

- Data-driven pages that benefit from server rendering: login, dashboard, resource lists, and detail pages.
- Static content that never changes at runtime: documentation, help, labs catalog, about, and 404.
- Browser-heavy pages that depend on the File API, live polling, or complex form state: wizards, object browser, and request inspector.

A single rendering mode forces a bad tradeoff. A pure SPA has a blank first paint and weak SEO for documentation. A pure SSR setup adds server cost and complexity to pages that do not need it. A pure SSG setup cannot render authenticated data.

Angular 22 supports route-level rendering modes through `app.routes.server.ts`.

## Decision

We will use hybrid rendering:

| Route type                                 | Mode |
| ------------------------------------------ | ---- |
| Login, dashboard, lists, details           | SSR  |
| Wizards, object browser, request inspector | CSR  |
| Docs, help, labs catalog, about, 404       | SSG  |

SSR pages use the API on the server and transfer the data to the client. SSG pages must not call the API at build time. CSR pages are rendered only in the browser.

## Consequences

### Positive

- Fast first paint for the shell and data pages.
- Static documentation without runtime data dependencies.
- Browser-only features stay simple and do not need server-safe code.
- Better perceived performance and clearer hydration rules.

### Negative

- Three rendering modes increase the test matrix.
- SSR code must avoid browser globals.
- Hydration mismatches are possible if language or session state differs between server and client.
- SSG pages are prerendered in English only.

### Neutral

- Every new route must declare its rendering mode.
- The team must test each SSR route with JavaScript disabled and with hydration enabled.

## Alternatives considered

| Alternative                   | Why it was rejected                                         |
| ----------------------------- | ----------------------------------------------------------- |
| SPA only                      | Blank first paint, poor documentation indexing, no SSR data |
| SSR for every route           | Unnecessary server work for static and browser-only pages   |
| SSG for every route           | Cannot render authenticated, data-driven pages              |
| Separate static site for docs | Extra deployment and navigation split                       |

## References

- [ARCHITECTURE.md](../../ARCHITECTURE.md)
- [DESIGN.md](../../DESIGN.md)
- [docs/UI-FLOWS.md](../UI-FLOWS.md)
