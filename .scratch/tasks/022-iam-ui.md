# 022 - IAM UI

- Phase: 2
- Status: Pending
- Depends on: 021
- Blocks: 026

## Goal

Expose the identity model in the console.

## Scope

- Compartment tree and CRUD.
- User list, create, detail, and deactivate.
- Group list, create, detail, and memberships.
- Policy list, create, detail, and statement editor.
- Dynamic group pages.
- API key metadata display.

## Deliverables

- `features/iam/compartments`
- `features/iam/users`
- `features/iam/groups`
- `features/iam/policies`
- `features/iam/dynamic-groups`
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The compartment and policy flows in `docs/UI-FLOWS.md` pass.
- [ ] The compartment tree reflects parent and child relationships.
- [ ] Deleting a non-empty compartment lists the blocking resource types.
- [ ] The tenancy root shows no delete action.
- [ ] The policy editor validates statements and shows the not-evaluated notice.
- [ ] Memberships can be added and removed from the group detail page.
- [ ] Deactivated users are visually distinct and not selectable for memberships.
- [ ] Every page has loading, empty, and error states.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Policy simulation.
- Federation.
- Password reset.
