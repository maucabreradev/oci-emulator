# 004 - CLI bootstrap and smoke profile

- Phase: 0
- Status: Pending
- Depends on: 002
- Blocks: 010

## Goal

Make the official OCI CLI usable against the emulator in one setup step.

## Scope

- Install Python 3.12 and the OCI CLI with uv into `.venv`.
- Generate a local RSA key pair and `.oci/config`.
- Add a `bin/ocilocal` wrapper.
- Add a CLI smoke script.
- Document the profile and endpoint.

## Deliverables

- `scripts/setup-cli.sh`
- `bin/ocilocal`
- `.oci/config` generation
- `scripts/compat/smoke.sh`
- README updates if the generated paths differ

## Acceptance criteria

- [ ] `make setup` creates `.venv`, `.oci/config`, and `bin/ocilocal`.
- [ ] `bin/ocilocal iam region list` returns data.
- [ ] `bin/ocilocal os ns get` returns data.
- [ ] The profile uses `OCIEMU`, the local tenancy, and `http://localhost:4599`.
- [ ] `.oci/` and `.venv/` are ignored by Git.
- [ ] The wrapper works from the repository root without extra exports.
- [ ] Re-running setup regenerates missing files without breaking a working config.

## Verification

```bash
make setup
bin/ocilocal iam region list
bin/ocilocal os ns get
```

## Out of scope

- Full compatibility suite.
- Global `~/.oci/config` changes.
- Windows-native shell support.
