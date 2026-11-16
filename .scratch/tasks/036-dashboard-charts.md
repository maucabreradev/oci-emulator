# 036 - Dashboard aggregation and charts

- Phase: 5
- Status: Pending
- Depends on: 035
- Blocks: 041

## Goal

Turn the dashboard into the operational overview of the emulator.

## Scope

- Dashboard aggregation endpoint.
- KPI counts by service.
- Instances by state.
- Storage by bucket.
- Work requests over seven days.
- Simulated monthly cost.
- Charts with ng2-charts and Chart.js.
- Accessible data table alternatives.

## Deliverables

- `DashboardService` and `/_emu/dashboard`.
- KPI card components.
- Chart components with `@defer`.
- Pricing table configuration.
- Translations.
- Playwright and unit tests.

## Acceptance criteria

- [ ] The dashboard matches the hierarchy in `DESIGN.md`.
- [ ] KPI counts match the underlying resources.
- [ ] Charts load only in the browser.
- [ ] Chart containers keep a fixed height before hydration.
- [ ] Every chart has a data table alternative.
- [ ] The cost chart is labeled simulated.
- [ ] Empty state offers seed and create actions.
- [ ] Dashboard aggregation stays under 200 ms for 10,000 resources.
- [ ] No layout shift when charts hydrate.

## Verification

```bash
make test-ui
make e2e
curl -sS http://localhost:4599/_emu/dashboard | jq .
```

## Out of scope

- Real billing.
- Historical metrics beyond work requests.
- Custom dashboards.
