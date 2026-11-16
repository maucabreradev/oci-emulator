# ADR 0005: Use the floci persistent storage backend

- Status: Accepted
- Date: 2026-11-16
- Deciders: Project maintainer
- Supersedes: none
- Superseded by: none

## Context

The emulator needs to persist control plane resources and Object Storage bytes between restarts. It must also support an in-memory mode for tests, a reset operation, seed data, and export and import.

The floci fork already provides a `StorageBackend` abstraction with a persistent mode, tenancy and region scoping, and Object Storage byte handling. The alternative is a relational database with a schema per service.

## Decision

We will use the floci `StorageBackend` in persistent mode, with data under `./data` by default. Services access storage only through the shared abstraction, never through filesystem paths.

## Consequences

### Positive

- No database server or migrations.
- Object bytes and JSON metadata use the same local storage.
- In-memory mode for tests comes for free.
- Reset is a directory removal.
- Export and import can archive the data directory.

### Negative

- No transactions across services.
- No query language, so list filters and search are implemented in code.
- Single-writer assumption.
- Export and import are not atomic.

### Neutral

- Storage keys are scoped by tenancy and region.
- Compartment usage is computed by scanning service stores.
- Large datasets are limited to tens of thousands of resources.

## Alternatives considered

| Alternative                                 | Why it was rejected                                           |
| ------------------------------------------- | ------------------------------------------------------------- |
| SQLite                                      | New schema and migration layer, duplicate object byte storage |
| PostgreSQL                                  | External service, contradicts the no-Docker goal              |
| JSON files per resource without abstraction | No scoping, no in-memory mode, no shared reset                |
| Embedded H2                                 | Same schema cost as SQLite, no benefit for object bytes       |

## References

- [ARCHITECTURE.md](../../ARCHITECTURE.md)
- [docs/DATA-MODEL.md](../DATA-MODEL.md)
