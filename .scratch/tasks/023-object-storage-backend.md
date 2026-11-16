# 023 - Object Storage backend completion

- Phase: 2
- Status: Pending
- Depends on: 020
- Blocks: 024, 025

## Goal

Complete the object storage control plane and data path.

## Scope

- Bucket update and delete rules.
- Object put, get, head, list, and delete.
- Prefix and delimiter listing.
- Multipart upload create, upload part, complete, and abort.
- Pre-authenticated requests.
- Lifecycle policies.

## Deliverables

- Object Storage services and controllers.
- Stored metadata and byte storage.
- Multipart assembly and cleanup.
- PAR access handling.
- Protocol tests.
- CLI compatibility scripts.

## Acceptance criteria

- [ ] Bucket names follow the documented rules and are unique in the namespace.
- [ ] A non-empty bucket cannot be deleted.
- [ ] Object bytes round-trip without modification.
- [ ] Prefix and delimiter listing match OCI behavior.
- [ ] Multipart uploads assemble into the final object.
- [ ] Aborted uploads leave no orphan parts.
- [ ] PARs support read, read/write, and any object read.
- [ ] Expired or revoked PARs are rejected.
- [ ] Lifecycle policies are stored and displayed, not executed.
- [ ] The official CLI performs the full lifecycle.

## Verification

```bash
bin/ocilocal os object put --bucket-name demo-bucket --file README.md --name README.md
bin/ocilocal os object get --bucket-name demo-bucket --name README.md --file downloaded.md
cmp README.md downloaded.md
make compat
```

## Out of scope

- Replication.
- Versioning.
- Archive tier execution.
- Object Lock.
