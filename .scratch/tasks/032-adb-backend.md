# 032 - Autonomous Database backend

- Phase: 4
- Status: Pending
- Depends on: 031
- Blocks: 033

## Goal

Implement the Autonomous Database control plane.

## Scope

- Create, list, get, update, and delete.
- Start, stop, and restart.
- Connection strings.
- Wallet generation and download.
- Backups.
- Work requests and lifecycle states.

## Deliverables

- ADB service and controller under `/20160918`.
- Stored models and state machine.
- Wallet ZIP with placeholder credentials.
- Protocol tests.
- CLI compatibility scripts.
- Seed updates.

## Acceptance criteria

- [ ] DB name is uppercase alphanumeric and at most 30 characters.
- [ ] OCPU and storage values are positive.
- [ ] The admin password meets the documented rules and is never returned.
- [ ] Creation reaches `AVAILABLE` through a work request.
- [ ] Start, stop, and restart are valid only in the documented states.
- [ ] Connection strings are returned and clearly labeled simulated.
- [ ] The wallet ZIP contains placeholder files and a warning.
- [ ] Backups can be created, listed, and deleted.
- [ ] The official CLI performs the full lifecycle.
- [ ] No database engine runs.

## Verification

```bash
bin/ocilocal db autonomous-database create \
  --compartment-id ocid1.compartment.oc1..demo \
  --db-name DEMOADB \
  --display-name demo-adb \
  --cpu-core-count 1 \
  --data-storage-size-in-tbs 1 \
  --admin-password 'Emulator#12345'
make compat
```

## Out of scope

- Database engine.
- SQL endpoints.
- Data guard.
- Real wallets.
