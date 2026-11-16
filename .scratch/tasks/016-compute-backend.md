# 016 - Compute backend

- Phase: 1
- Status: Pending
- Depends on: 013
- Blocks: 017, 018

## Goal

Implement the compute control plane.

## Scope

- Shapes and images as seeded read-only catalogs.
- Instance launch, list, get, update, and terminate.
- Instance actions: start, stop, reboot, reset.
- VNICs and console history.
- Work requests and lifecycle transitions.
- Compartment usage.

## Deliverables

- Compute service and controller.
- Stored models and state machine.
- Seeded shapes and images.
- Protocol tests.
- CLI compatibility scripts.

## Acceptance criteria

- [ ] Launch returns `202` with `opc-work-request-id`.
- [ ] The instance transitions from `PROVISIONING` to `RUNNING`.
- [ ] `--wait-for-state RUNNING` succeeds with the official CLI.
- [ ] Invalid actions return `Conflict` with a clear message.
- [ ] Terminate detaches volumes and releases ephemeral public IPs.
- [ ] VNICs are created in the selected subnet.
- [ ] Shapes and images cannot be modified or deleted.
- [ ] Compartment usage blocks deletion while instances exist.

## Verification

```bash
bin/ocilocal compute instance launch \
  --compartment-id ocid1.compartment.oc1..demo \
  --availability-domain AD-1 \
  --shape VM.Standard.E4.Flex \
  --subnet-id "$SUBNET_ID" \
  --image-id "$IMAGE_ID" \
  --display-name demo-instance \
  --wait-for-state RUNNING
make compat
```

## Out of scope

- Real virtualization.
- SSH access.
- Boot volumes beyond the control plane.
