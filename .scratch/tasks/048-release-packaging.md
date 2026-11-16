# 048 - Release packaging and changelog

- Phase: 6
- Status: Pending
- Depends on: 045, 047
- Blocks: 049

## Goal

Produce a reproducible distribution and release documentation.

## Scope

- Version update across Maven, pnpm, and `STATE.md`.
- `make package` target.
- Distribution ZIP with backend, UI, docs, and scripts.
- Checksums.
- Changelog entry.
- Release notes template.
- Pre-release checklist execution.

## Deliverables

- `make package`.
- `target/oci-emulator-<version>.zip`.
- `target/checksums.txt`.
- `CHANGELOG.md`.
- Release notes draft.
- Updated `docs/RELEASE.md` if commands changed.

## Acceptance criteria

- [ ] `make package` produces the distribution ZIP and checksums.
- [ ] The ZIP contains no `.oci`, `.venv`, `data`, or local configuration.
- [ ] The packaged build starts on a clean machine with the documented prerequisites.
- [ ] The CLI smoke commands pass against the packaged build.
- [ ] The UI loads from the packaged build.
- [ ] The version is visible in About and `/_emu/config`.
- [ ] The changelog matches the actual changes.
- [ ] The release notes include known issues and verification evidence.

## Verification

```bash
make clean
make verify
make package
unzip -l target/oci-emulator-*.zip | head -40
```

## Out of scope

- GitHub release publication.
- Code signing.
- Installers.
