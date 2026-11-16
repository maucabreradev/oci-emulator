# 049 - Final documentation and release candidate

- Phase: 6
- Status: Pending
- Depends on: 048
- Blocks: 050

## Goal

Make the documentation match the released behavior.

## Scope

- README command and configuration audit.
- Compatibility matrix audit.
- Protocol and data model audit.
- UI flow and lab audit.
- Troubleshooting updates from real defects.
- Link check and em dash check.
- Release candidate smoke test.

## Deliverables

- Updated `README.md`, `docs/`, and `CONTRIBUTING.md`.
- Final `docs/COMPATIBILITY.md` statuses.
- Release candidate checklist.
- Updated `STATE.md`.

## Acceptance criteria

- [ ] Every README command exists and works.
- [ ] Every configuration key matches `application.yml`.
- [ ] Compatibility rows are `verified`, `partial`, or `not planned` with reasons.
- [ ] Links and anchors resolve.
- [ ] No em dashes in English documentation.
- [ ] No Oracle assets.
- [ ] The release candidate passes the manual smoke test.
- [ ] Known issues are documented.
- [ ] `STATE.md` points to the release task.

## Verification

```bash
make verify
make compat
make e2e
```

## Out of scope

- New features.
- Refactors without a documentation reason.
