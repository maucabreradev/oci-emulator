# 021 - IAM backend completion

- Phase: 2
- Status: Pending
- Depends on: 020
- Blocks: 022

## Goal

Complete the identity model beyond compartments.

## Scope

- Users and user lifecycle.
- Groups and memberships.
- Policies.
- Dynamic groups.
- API key metadata for users.
- Compartment usage and delete rules.

## Deliverables

- IAM services and controllers.
- Stored models.
- Protocol tests.
- CLI compatibility scripts.

## Acceptance criteria

- [ ] Users support create, list, get, update, deactivate, and delete.
- [ ] Group memberships can be added and removed.
- [ ] Removing a membership does not delete the user.
- [ ] Policies store statements and a version date.
- [ ] Policies are stored and displayed but not evaluated.
- [ ] Dynamic groups store a matching rule.
- [ ] Names are unique where `docs/DATA-MODEL.md` requires it.
- [ ] The official CLI performs the full lifecycle.
- [ ] Compartment deletion accounts for identity resources.

## Verification

```bash
bin/ocilocal iam user list --compartment-id ocid1.tenancy.oc1..emulator
bin/ocilocal iam policy list --compartment-id ocid1.tenancy.oc1..emulator
make compat
```

## Out of scope

- Policy evaluation.
- Password management.
- Real API key authentication.
- Federation.
