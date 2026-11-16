# 014 - Networking backend: route tables, security, DHCP, and IPs

- Phase: 1
- Status: Pending
- Depends on: 013
- Blocks: 015

## Goal

Complete the remaining VCN configuration resources.

## Scope

- Route tables and rules.
- Security lists and rules.
- Network security groups and rules.
- DHCP options.
- Private IPs and public IPs.

## Deliverables

- Services, controllers, and stored models.
- Validation for rules and references.
- Default resource protection.
- CLI compatibility scripts.
- Protocol tests.

## Acceptance criteria

- [ ] Default route tables, security lists, and DHCP options cannot be deleted.
- [ ] Route destinations accept CIDRs and service values.
- [ ] Route targets must belong to the same VCN.
- [ ] Security rule protocols and port ranges are validated.
- [ ] NSG rules are stored and listed.
- [ ] Private and public IP lifecycles follow `docs/DATA-MODEL.md`.
- [ ] The official CLI performs the full lifecycle.
- [ ] Compartment usage accounts for every new resource.

## Verification

```bash
bin/ocilocal network route-table list --compartment-id ocid1.compartment.oc1..demo
bin/ocilocal network security-list list --compartment-id ocid1.compartment.oc1..demo
make compat
```

## Out of scope

- Actual traffic filtering.
- Route evaluation.
- IPv6.
