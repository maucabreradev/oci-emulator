# 028 - Load Balancer UI

- Phase: 3
- Status: Pending
- Depends on: 027
- Blocks: 031

## Goal

Expose load balancer configuration in the console.

## Scope

- Load balancer list, create wizard, and detail.
- Listener editor.
- Backend set editor.
- Backend editor.
- Simulated health display.
- Work request tracking.

## Deliverables

- `features/load-balancer`
- Wizard with steps for basics, subnets, listener, backend set, review.
- Detail tabs for listeners, backend sets, backends, and JSON.
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The load balancer flow in `docs/UI-FLOWS.md` passes.
- [ ] The wizard validates subnets, listener ports, and backend IPs.
- [ ] The review step shows the CLI equivalent and JSON preview.
- [ ] Creation shows a work request and ends in `ACTIVE`.
- [ ] The detail page shows listeners, backend sets, and simulated health.
- [ ] The UI states that no traffic is forwarded.
- [ ] Destructive actions require confirmation.
- [ ] Every page has loading, empty, and error states.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Traffic statistics.
- TLS certificates.
- WAF rules.
