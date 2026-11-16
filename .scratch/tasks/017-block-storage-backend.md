# 017 - Block Storage backend

- Phase: 1
- Status: Pending
- Depends on: 016
- Blocks: 019

## Goal

Implement volumes, attachments, and backups.

## Scope

- Volume create, list, get, update, and delete.
- Volume resize.
- Attachment create, list, get, and delete.
- Backup create, list, get, and delete.
- State transitions and delete constraints.
- Compartment usage.

## Deliverables

- Block Storage service and controller.
- Stored models and state machine.
- Protocol tests.
- CLI compatibility scripts.
- Seed updates.

## Acceptance criteria

- [ ] Volume size is validated between 50 and 32768 GB.
- [ ] Size cannot be decreased.
- [ ] A volume can have only one attachment at a time.
- [ ] A volume cannot be deleted while attached unless force is used.
- [ ] Attachment and detach operations update the volume state.
- [ ] Backups create and delete without affecting the source volume.
- [ ] The official CLI performs the full lifecycle.
- [ ] Compartment usage blocks deletion while volumes exist.

## Verification

```bash
bin/ocilocal bv volume create \
  --compartment-id ocid1.compartment.oc1..demo \
  --availability-domain AD-1 \
  --display-name demo-volume \
  --size-in-gbs 50
make compat
```

## Out of scope

- Real block devices.
- iSCSI sessions.
- Cross-region replication.
