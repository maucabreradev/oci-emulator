# ADR 0004: Transloco with a static translation loader

- Status: Accepted
- Date: 2026-11-16
- Deciders: Project maintainer
- Supersedes: none
- Superseded by: none

## Context

The web console must support English and Spanish with runtime language switching. Server-rendered pages must resolve the same language as the client, and static pages must not depend on a network request to render.

Angular built-in i18n requires one build per locale and does not support runtime switching. Transloco supports runtime locales but defaults to an HTTP loader, which introduces a request during SSR and a possible hydration mismatch.

## Decision

We will use Transloco 8.4.0 with a static loader that bundles both translation files into the application. The active locale is resolved from the `emu.locale` cookie, then a query parameter, then `Accept-Language`, with English as the fallback.

## Consequences

### Positive

- Runtime language switching without a rebuild.
- Deterministic SSR and SSG, with no translation fetch.
- No hydration mismatch caused by a late translation load.
- Translation keys can be checked statically.

### Negative

- All translations are part of the bundle.
- Adding a language increases the initial payload.
- Translation files must be kept in sync manually, enforced by a check script.

### Neutral

- The `lang` attribute updates on locale change.
- Dates and numbers use `Intl` through a `FormatService`.
- SSG pages render in English and switch after hydration.

## Alternatives considered

| Alternative               | Why it was rejected                            |
| ------------------------- | ---------------------------------------------- |
| Angular built-in i18n     | One build per locale, no runtime switching     |
| Transloco HTTP loader     | Extra request, SSR race, hydration risk        |
| ngx-translate             | Smaller ecosystem, no advantage over Transloco |
| Custom dictionary service | Reinvents plural and interpolation handling    |

## References

- [ARCHITECTURE.md](../../ARCHITECTURE.md)
- [DESIGN.md](../../DESIGN.md)
- [docs/UI-FLOWS.md](../UI-FLOWS.md)
