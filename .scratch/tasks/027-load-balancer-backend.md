# 027 - Load Balancer backend

- Phase: 3
- Status: Pending
- Depends on: 026
- Blocks: 028

## Goal

Implement the load balancer control plane.

## Scope

- Load balancer create, list, get, update, and delete.
- Listeners.
- Backend sets.
- Backends.
- Health check configuration and simulated status.
- Work requests and state transitions.

## Deliverables

- Load Balancer service and controller under `/20200501`.
- Stored models.
- Protocol tests.
- CLI compatibility scripts.
- Seed updates.

## Acceptance criteria

- [ ] A load balancer requires one or two subnets.
- [ ] At least one listener is required at creation.
- [ ] Listener names are unique in the load balancer.
- [ ] Backend sets require a policy and a health checker.
- [ ] A backend set cannot be deleted while referenced by a listener.
- [ ] Health status is simulated and clearly labeled.
- [ ] Creation reaches `ACTIVE` through a work request.
- [ ] No traffic is forwarded.
- [ ] The official CLI performs the full lifecycle.

## Verification

```bash
bin/ocilocal lb load-balancer list --compartment-id ocid1.compartment.oc1..demo
make compat
```

## Out of scope

- Traffic plane.
- TLS termination.
- WAF.
- Real health checks.
