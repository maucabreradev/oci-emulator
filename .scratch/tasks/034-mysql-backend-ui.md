# 034 - MySQL backend and UI

- Phase: 4
- Status: Pending
- Depends on: 013, 031
- Blocks: 035

## Goal

Implement and expose MySQL DB systems.

## Scope

- DB system create, list, get, update, and delete.
- Start, stop, and restart.
- Endpoints and connection details.
- Backups.
- Console list, create wizard, and detail.
- Work requests, translations, and tests.

## Deliverables

- MySQL service and controller under `/20190415`.
- `features/database/mysql`
- Stored models and state machine.
- Protocol tests.
- CLI compatibility scripts.
- Playwright coverage.

## Acceptance criteria

- [ ] A DB system requires a shape, subnet, and storage size.
- [ ] The subnet must belong to the selected compartment.
- [ ] Admin username and password meet the documented rules.
- [ ] The password is never returned.
- [ ] Creation reaches `ACTIVE` through a work request.
- [ ] Start, stop, and restart follow the documented state machine.
- [ ] Endpoints and connection details are labeled simulated.
- [ ] Backups can be created, listed, and deleted.
- [ ] The UI states that no database engine runs.
- [ ] The official CLI performs the full lifecycle.

## Verification

```bash
bin/ocilocal mysql db-system list --compartment-id ocid1.compartment.oc1..demo
make compat
make e2e
```

## Out of scope

- Database engine.
- Replicas.
- HeatWave.
- Real endpoints.
