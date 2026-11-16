# 029 - API Gateway backend and UI

- Phase: 3
- Status: Pending
- Depends on: 013, 026
- Blocks: 031

## Goal

Implement and expose API Gateway resources.

## Scope

- Gateway create, list, get, update, and delete.
- Deployments.
- Routes and methods.
- Work requests.
- Console list, create wizard, and detail.
- Translations and tests.

## Deliverables

- API Gateway service and controller under `/20190501`.
- `features/api-gateway`
- Protocol tests.
- CLI compatibility scripts.
- Playwright coverage.

## Acceptance criteria

- [ ] A gateway requires a subnet and a public endpoint type.
- [ ] Deployments belong to a gateway and have a path prefix.
- [ ] Routes validate paths, methods, and backend definitions.
- [ ] A gateway with deployments cannot be deleted.
- [ ] Creation reaches `ACTIVE` through a work request.
- [ ] The UI shows routes grouped by deployment.
- [ ] The UI states that no requests are routed.
- [ ] Every page has loading, empty, and error states.
- [ ] The official CLI performs the full lifecycle.

## Verification

```bash
make compat
make e2e
```

## Out of scope

- Request routing.
- Authentication and authorization at the gateway.
- Usage plans.
