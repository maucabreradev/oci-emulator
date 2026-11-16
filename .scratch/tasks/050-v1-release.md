# 050 - Version 1.0.0 release

- Phase: 6
- Status: Pending
- Depends on: 049
- Blocks: none

## Goal

Publish the first stable release.

## Scope

- Final version bump.
- Tag creation.
- GitHub release.
- Distribution and checksum upload.
- Changelog publication.
- Post-release state update.

## Deliverables

- `v1.0.0` tag.
- GitHub release with artifacts and notes.
- Updated `CHANGELOG.md`.
- Updated `STATE.md`.
- Post-release verification.

## Acceptance criteria

- [ ] The working tree is clean before tagging.
- [ ] `make verify`, `make compat`, and `make e2e` pass on the release commit.
- [ ] The tag points to the release commit.
- [ ] The GitHub release includes the ZIP and checksums.
- [ ] Release notes match the changelog.
- [ ] Download links work.
- [ ] The release can be started from the ZIP.
- [ ] `STATE.md` records the release as the last completed task.
- [ ] No follow-up task is required for the release itself.

## Verification

```bash
git status --short
git tag --list 'v1.0.0'
make verify
```

## Out of scope

- Version 1.1 planning.
- Long-term support branches.
- Marketplace publication.
