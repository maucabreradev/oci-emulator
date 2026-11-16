# 007 - Internationalization foundation

- Phase: 0
- Status: Pending
- Depends on: 005
- Blocks: 008

## Goal

Provide complete runtime English and Spanish localization with deterministic SSR.

## Scope

- Transloco with a static loader.
- English and Spanish translation files.
- Locale resolution from cookie, query, and `Accept-Language`.
- Language selector.
- Locale-aware date, number, and plural formatting.
- Missing-key check script.

## Deliverables

- `console/public/i18n/en.json`
- `console/public/i18n/es.json`
- `LocaleService`
- `StaticTranslocoLoader`
- `FormatService`
- `scripts/check-i18n.mjs`
- `pnpm i18n:check` script

## Acceptance criteria

- [ ] Both translation files have the same key set.
- [ ] The language selector switches the UI without a reload.
- [ ] The `emu.locale` cookie is set and honored by the server.
- [ ] The `<html lang>` attribute updates.
- [ ] Dates and numbers use the active locale through `Intl`.
- [ ] Missing keys fall back to English and warn in development.
- [ ] `pnpm i18n:check` fails on missing or extra keys.
- [ ] SSR and client render the same language for the same cookie.

## Verification

```bash
cd console && pnpm i18n:check && pnpm test
curl -sS -H 'Cookie: emu.locale=es' http://localhost:4599/login | grep -o 'lang="[^"]*"'
```

## Out of scope

- Additional languages.
- Localized URLs.
- Translation management tooling.
