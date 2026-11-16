# 006 - Design system and application shell

- Phase: 0
- Status: Pending
- Depends on: 005
- Blocks: 011

## Goal

Implement the visual foundation and the persistent console shell.

## Scope

- Design tokens for color, spacing, typography, radius, and elevation.
- Self-hosted Inter and JetBrains Mono.
- Project logo and Lucide icons.
- Top bar, sidebar, breadcrumbs, page header, and content outlet.
- Shared empty, loading, and error components.
- Responsive behavior for desktop, tablet, and mobile.

## Deliverables

- `console/src/styles/` with tokens and base styles.
- `console/public/fonts/` with licensed font files.
- `console/public/logo.svg`
- `shared/layout/app-shell`
- `shared/ui/page-header`
- `shared/ui/empty-state`
- `shared/ui/loading-state`
- `shared/ui/error-state`

## Acceptance criteria

- [ ] The shell matches the layout measurements in `DESIGN.md`.
- [ ] The palette uses the documented tokens and no default Material purple.
- [ ] Fonts are self-hosted and load without external requests.
- [ ] Icons come from Lucide and icon-only buttons have accessible names.
- [ ] The shell is keyboard navigable and focus is visible.
- [ ] Status colors are never the only indicator.
- [ ] No Oracle assets are present.
- [ ] The layout works at 1440, 1024, 768, and 375 px.

## Verification

```bash
cd console && pnpm lint && pnpm test
make dev
```

## Out of scope

- Charts.
- Feature navigation data.
- Dark theme.
