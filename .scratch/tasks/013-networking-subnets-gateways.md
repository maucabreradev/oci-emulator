# 013 - Networking backend: subnets and gateways

- Phase: 1
- Status: Pending
- Depends on: 012
- Blocks: 014, 015, 016, 029, 034

## Goal

Complete subnet and gateway support for VCNs.

## Scope

- Subnet create, list, get, update, and delete.
- CIDR containment and overlap validation.
- Internet, NAT, and service gateways.
- Route table and security list associations.
- Compartment usage updates.

## Deliverables

- Subnet service and controller.
- Gateway services and controllers.
- Stored models and protocol tests.
- CLI compatibility scripts.
- Seed updates for a public subnet and internet gateway.

## Acceptance criteria

- [ ] A subnet CIDR must be inside the VCN CIDR.
- [ ] Overlapping subnets in the same VCN are rejected.
- [ ] Referenced route tables, security lists, and DHCP options must belong to the VCN.
- [ ] A subnet cannot be deleted while it has VNICs.
- [ ] A VCN with subnets or gateways cannot be deleted.
- [ ] Gateway enable and disable operations are supported.
- [ ] The official CLI performs the full lifecycle.
- [ ] Responses preserve OCI headers and errors.

## Verification

```bash
bin/ocilocal network subnet create \
  --compartment-id ocid1.compartment.oc1..demo \
  --vcn-id "$VCN_ID" \
  --cidr-block 10.0.1.0/24 \
  --display-name demo-subnet
make compat
```

## Out of scope

- Route rule evaluation.
- Packet forwarding.
- IPv6.
