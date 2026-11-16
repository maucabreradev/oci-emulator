# 010 - Phase 0 backend slice

- Phase: 0
- Status: Pending
- Depends on: 003, 004
- Blocks: 011

## Goal

Prove the backend with a small, complete OCI slice.

## Scope

- IAM regions and compartments.
- Object Storage namespace, buckets, and object metadata.
- VCN create, list, get, update, and delete.
- Work requests for asynchronous operations.
- Compartment usage protection.
- CLI compatibility scripts for the slice.

## Deliverables

- Controllers and services for regions, compartments, buckets, and VCNs.
- Stored models and protocol tests.
- `scripts/compat/identity.sh`
- `scripts/compat/object-storage.sh`
- `scripts/compat/network-vcn.sh`
- Compartment usage implementation for the slice.

## Acceptance criteria

- [ ] All Phase 0 rows in `docs/COMPATIBILITY.md` are verified with the official CLI.
- [ ] A VCN cannot be deleted while it has dependent resources.
- [ ] A compartment cannot be deleted while it contains resources.
- [ ] Bucket and VCN responses include OCI headers and correct error envelopes.
- [ ] Work requests are created and reach a terminal state.
- [ ] Pagination works for compartments, buckets, and VCNs.
- [ ] Restarting the emulator preserves resources.

## Verification

```bash
make compat
bin/ocilocal iam compartment list --compartment-id ocid1.tenancy.oc1..emulator
bin/ocilocal network vcn list --compartment-id ocid1.compartment.oc1..demo
```

## Out of scope

- Subnets and gateways.
- Multipart uploads.
- Compute.
