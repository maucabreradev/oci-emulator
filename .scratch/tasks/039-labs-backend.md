# 039 - Labs backend and catalog

- Phase: 5
- Status: Pending
- Depends on: 035
- Blocks: 040

## Goal

Implement lab definitions, validation, and reset.

## Scope

- Lab JSON format and loader.
- Catalog endpoint.
- Validation endpoint.
- Reset endpoint scoped by lab tag.
- Four initial labs from `docs/LABS.md`.
- Validation result shape.
- Tests for each lab.

## Deliverables

- `src/main/resources/labs/*.json`
- `LabService`
- `/_emu/labs`, `/_emu/labs/{id}/validate`, `/_emu/labs/{id}/reset`
- Validator for each supported resource type.
- Backend tests.

## Acceptance criteria

- [ ] The four initial labs load and validate.
- [ ] Validation is read-only.
- [ ] Failed checks report expected and found values.
- [ ] Reset removes only resources tagged with the lab ID.
- [ ] Reset respects dependency order.
- [ ] Validation does not depend on request logs.
- [ ] Unknown check types fail with a development warning.
- [ ] Lab definitions are deterministic and require no internet access.

## Verification

```bash
curl -sS http://localhost:4599/_emu/labs | jq .
curl -sS -X POST http://localhost:4599/_emu/labs/vcn-public-instance/validate | jq .
make test
```

## Out of scope

- Lab UI.
- Lab sharing.
- Completion history on the server.
