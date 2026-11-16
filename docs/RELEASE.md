# Release Guide

This document defines versioning, packaging, verification, and publication for OCI Emulator.

For setup and operation, see [README.md](../README.md). For the test matrix, see [CONTRIBUTING.md](../CONTRIBUTING.md).

---

## 1. Versioning

OCI Emulator uses [Semantic Versioning](https://semver.org/):

```text
MAJOR.MINOR.PATCH
```

| Change                                                       | Version |
| ------------------------------------------------------------ | ------- |
| Breaking API or data format change                           | MAJOR   |
| New service, new flow, new lab, backward-compatible endpoint | MINOR   |
| Bug fix, translation fix, documentation correction           | PATCH   |

Versioning rules:

- The API version prefixes are OCI dates. They are not the product version.
- The product version is exposed in `/_emu/config` and the About page.
- The version is stored in `pom.xml`, `console/package.json`, and `STATE.md`.
- Pre-releases use `-alpha.N`, `-beta.N`, or `-rc.N`.
- A release is immutable once published.

---

## 2. Release artifacts

| Artifact          | Path                                 | Contents                               |
| ----------------- | ------------------------------------ | -------------------------------------- |
| Backend JAR       | `target/quarkus-app/quarkus-run.jar` | Quarkus application                    |
| Full distribution | `target/oci-emulator-<version>.zip`  | Quarkus app, UI bundles, scripts, docs |
| Source archive    | GitHub generated                     | Source at the release tag              |
| Checksums         | `target/checksums.txt`               | SHA-256 for each artifact              |

The distribution contains:

```text
oci-emulator-<version>/
  bin/ocilocal
  console/dist/console/browser/
  console/dist/console/server/
  docs/
  scripts/
  target/quarkus-app/
  LICENSE
  README.md
  SECURITY.md
  CONTRIBUTING.md
```

The distribution does not contain `.oci/`, `.venv/`, `data/`, or any local configuration.

---

## 3. Pre-release checklist

### 3.1 Code and contracts

- [ ] All tasks for the milestone are merged.
- [ ] `docs/COMPATIBILITY.md` rows for the milestone are `verified`.
- [ ] `docs/DATA-MODEL.md` matches the stored models.
- [ ] `docs/PROTOCOL.md` matches the controllers.
- [ ] `docs/UI-FLOWS.md` acceptance criteria are covered.
- [ ] No TODOs in user-visible behavior.
- [ ] No unresolved `partial` rows without a documented note.

### 3.2 Verification

- [ ] `make verify` passes.
- [ ] `make compat` passes with the official OCI CLI.
- [ ] `make e2e` passes on a fresh data directory.
- [ ] Accessibility checks pass for the changed pages.
- [ ] SSR routes render with JavaScript disabled.
- [ ] Hydration produces no warnings.
- [ ] English and Spanish translations are complete.
- [ ] Language switching causes no layout shift.
- [ ] Initial JavaScript is under 500 KB gzip.

### 3.3 Manual smoke test

Run these against a fresh emulator:

```bash
make setup
make dev
```

Then:

- [ ] Sign in with any credentials.
- [ ] Seed demo data.
- [ ] Create a VCN and a subnet.
- [ ] Launch an instance and wait for `RUNNING`.
- [ ] Create a bucket, upload an object, and download it.
- [ ] Create a PAR and open it.
- [ ] Create a compartment and a policy.
- [ ] Create a load balancer and a DNS record.
- [ ] Create an Autonomous Database and stop it.
- [ ] Open the request inspector and copy a request as CLI.
- [ ] Complete a lab.
- [ ] Switch to Spanish and back.
- [ ] Reset the emulator.
- [ ] Run the CLI smoke commands from `docs/COMPATIBILITY.md`.

### 3.4 Documentation

- [ ] README commands match the Makefile.
- [ ] Configuration keys match `application.yml`.
- [ ] Screenshots, if any, use the project branding.
- [ ] No Oracle logos, fonts, icons, or screenshots.
- [ ] No em dashes in English documentation.
- [ ] Links and anchors are valid.

### 3.5 Repository

- [ ] `main` is green.
- [ ] The release branch is up to date.
- [ ] No secrets or local files are tracked.
- [ ] The version is updated in all files.
- [ ] The changelog entry is written.

---

## 4. Changelog

Use a `CHANGELOG.md` with this shape:

```markdown
# Changelog

All notable changes to this project are documented in this file.

## [Unreleased]

## [0.1.0] - 2026-12-01

### Added

- Phase 0 vertical slice: login, dashboard, VCN, compartments, buckets.
- English and Spanish UI.

### Changed

- None.

### Fixed

- None.

### Removed

- None.
```

Rules:

- Group by Added, Changed, Deprecated, Removed, Fixed, and Security.
- Write for users, not for commit history.
- Link to the pull request or task when useful.
- Do not include internal refactors unless they affect behavior.

---

## 5. Release procedure

### 5.1 Prepare

1. Create `release/<version>` from `main`.
2. Update the version in `pom.xml`, `console/package.json`, and `STATE.md`.
3. Update `CHANGELOG.md`.
4. Run the full pre-release checklist.
5. Commit with `chore(release): prepare <version>`.

### 5.2 Build

```bash
make clean
make verify
make compat
make e2e
make package
```

`make package` must produce:

- `target/quarkus-app/quarkus-run.jar`
- `target/oci-emulator-<version>.zip`
- `target/checksums.txt`

### 5.3 Tag

```bash
git tag -a v<version> -m "OCI Emulator <version>"
git push origin release/<version>
git push origin v<version>
```

Do not tag from a dirty working tree.

### 5.4 Publish

1. Create a GitHub release for the tag.
2. Attach the distribution ZIP and checksums.
3. Paste the changelog entry as the release notes.
4. Mark pre-releases as pre-release.
5. Verify the download links.

### 5.5 Post-release

1. Merge `release/<version>` back into `main`.
2. Bump `main` to the next `-SNAPSHOT` version if the project uses snapshots.
3. Update `STATE.md` with the release task as completed.
4. Announce the release in the repository discussions, if enabled.

---

## 6. Rollback

If a release is broken:

1. Mark the GitHub release as a pre-release or remove it.
2. Publish a patch release with the fix.
3. Do not delete the tag. Add a note to the release.
4. Document the failure and the fix in the changelog.

Data compatibility:

- Persistent state from an older minor version should load in a newer minor version.
- A major version may require a reset.
- Export files include a `manifest.json` with the schema version.
- Import rejects an incompatible schema version with a clear error.

---

## 7. Support policy

- Only the latest release is supported.
- Security fixes follow [SECURITY.md](../SECURITY.md).
- Compatibility fixes target the official OCI CLI version documented in the release notes.
- There is no long-term support branch.

---

## 8. Release notes template

```markdown
# OCI Emulator <version>

<One-paragraph summary.>

## Highlights

- Item.
- Item.

## Services

| Service | Status |
|---|---|
| Compute | ... |
| Networking | ... |

## Breaking changes

- None.

## Upgrade notes

- None.

## Known issues

- None.

## Verification

- `make verify`
- `make compat`
- `make e2e`
```

---

## 9. Definition of a releasable build

A build is releasable when:

- Every checklist item in section 3 is checked.
- The distribution starts on a clean machine with the documented prerequisites.
- The CLI smoke commands pass against the packaged build.
- The UI loads from the packaged build.
- The version is visible in the About page and in `/_emu/config`.
- The changelog matches the actual changes.
