# 005 - Angular workspace and Quinoa integration

- Phase: 0
- Status: Pending
- Depends on: 002
- Blocks: 006, 007, 009

## Goal

Create the Angular 22 workspace and connect it to the Quarkus development loop.

## Scope

- Generate `console/` with Angular 22, SSR, SCSS, and strict TypeScript.
- Add Material, CDK, Transloco, ng2-charts, Chart.js, Vitest, and Playwright.
- Configure Quinoa with pnpm and the Angular dev server.
- Add formatting and lint scripts.
- Add the production browser and server builds.

## Deliverables

- `console/package.json`, `angular.json`, `tsconfig.json`
- `console/src/app/app.config.ts`
- `console/src/server.ts`
- Quinoa configuration in `application.yml`
- `pnpm` scripts for `build`, `test`, `lint`, `e2e`, and `i18n:check`

## Acceptance criteria

- [ ] `pnpm install` succeeds with the committed lockfile.
- [ ] `pnpm build` produces browser and server bundles.
- [ ] `make dev` starts Quarkus on `4599` and the Angular dev server through Quinoa.
- [ ] `http://localhost:4599` serves the Angular app.
- [ ] TypeScript strict mode is enabled and no `any` is allowed by lint.
- [ ] Vitest runs a sample test.
- [ ] Playwright can launch against the local app.

## Verification

```bash
cd console && pnpm install && pnpm build && pnpm test
make dev
```

## Out of scope

- Application shell.
- Feature pages.
- Production Node process.
