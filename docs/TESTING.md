# Testing and Quality

Test layers, commands, compatibility coverage, accessibility, and performance budgets.

For the required checks per change type and the definition of done, see [CONTRIBUTING.md](../CONTRIBUTING.md). For protocol conformance, see [PROTOCOL.md](PROTOCOL.md). For release gates, see [RELEASE.md](RELEASE.md).

---

## 1. Test layers

| Layer                        | Tool                             | Scope                                                     |
| ---------------------------- | -------------------------------- | --------------------------------------------------------- |
| Backend unit and integration | JUnit 5, RestAssured             | Controllers, services, validation, errors, work requests  |
| CLI compatibility            | Shell scripts and `bin/ocilocal` | Real `oci` commands against the emulator                  |
| Frontend unit                | Vitest and TestBed               | Stores, API clients, validators, session, i18n            |
| End-to-end                   | Playwright                       | Login, VCN, instance, bucket, PAR, LB, ADB, locale, reset |
| Accessibility                | axe-core and Playwright          | Key pages, contrast, keyboard navigation                  |
| SSR and SSG                  | curl and Playwright              | Rendered HTML, hydration, redirects, locale cookie        |

---

## 2. Run the tests

```bash
make test
make test-ui
make e2e
make compat
make verify
```

`make verify` is the required check before merging. It runs format check, lint, backend tests, frontend tests, and the i18n check.

---

## 3. Compatibility suite

The compatibility suite creates, reads, updates, and deletes resources with the official CLI. It covers:

- IAM compartments, users, groups, and policies.
- Object Storage namespace, buckets, objects, and PARs.
- VCNs, subnets, gateways, route tables, security lists, and NSGs.
- Instances, volumes, attachments, and backups.
- Load balancers, API Gateways, and DNS zones.
- Autonomous Databases and MySQL DB systems.
- Work requests and pagination.

The supported operations and their status are tracked in [COMPATIBILITY.md](COMPATIBILITY.md).

---

## 4. Accessibility

Target: WCAG 2.2 AA.

Requirements:

- Full keyboard navigation.
- Visible focus states.
- Accessible names for controls.
- No color-only status indicators.
- Chart alternatives as data tables.
- Reduced motion support.
- Sufficient contrast in light and dark surfaces.
- Form errors associated with their fields.

---

## 5. Performance budgets

- Initial JavaScript: less than 500 KB gzip, excluding lazy chunks.
- LCP: less than 1.5 seconds locally.
- CLS: less than 0.1.
- No layout shift when switching language.
- Charts load only when visible.
- Tables paginate on the server.

Budgets are enforced with `pnpm budget` in the frontend workspace.
