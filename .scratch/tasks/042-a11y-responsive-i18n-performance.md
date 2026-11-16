# 042 - Accessibility, responsive, i18n, and performance polish

- Phase: 5
- Status: Pending
- Depends on: 037, 038, 040, 041
- Blocks: 043

## Goal

Meet the accessibility, localization, responsive, and performance targets.

## Scope

- Keyboard navigation audit.
- Focus management for dialogs and drawers.
- Screen reader labels and table captions.
- Contrast and reduced motion.
- Responsive fixes for 1024, 768, and 375 px.
- Complete English and Spanish translations.
- Bundle size and layout shift fixes.
- axe-core checks.

## Deliverables

- Accessibility fixes across the console.
- Responsive fixes.
- Translation corrections.
- Performance budget report.
- axe-core test suite.

## Acceptance criteria

- [ ] Every interactive element is keyboard reachable.
- [ ] Dialogs trap focus and restore it on close.
- [ ] Icon-only buttons have accessible names.
- [ ] Form errors are associated with fields.
- [ ] Charts have data table alternatives.
- [ ] Reduced motion is respected.
- [ ] No critical axe-core violations on key pages.
- [ ] The layout works at 1280, 1024, 768, and 375 px.
- [ ] English and Spanish are complete with no missing keys.
- [ ] Initial JavaScript is under 500 KB gzip.
- [ ] LCP is under 1.5 seconds locally and CLS is under 0.1.
- [ ] Switching language causes no layout shift.

## Verification

```bash
cd console && pnpm i18n:check && pnpm test && pnpm e2e
make e2e
```

## Out of scope

- Dark theme.
- Additional languages.
- Mobile-specific wizards.
