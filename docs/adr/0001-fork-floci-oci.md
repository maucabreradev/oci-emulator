# ADR 0001: Fork floci-oci as the backend base

- Status: Accepted
- Date: 2026-11-16
- Deciders: Project maintainer
- Supersedes: none
- Superseded by: none

## Context

OCI Emulator needs an OCI-compatible control plane with persistent local storage, work requests, OCID generation, pagination, ETags, and several services already modeled: IAM, Object Storage, Core Networking, Compute, and Block Storage.

Building that from scratch would take months before the first end-to-end flow works. Several existing emulators were evaluated. Most are incomplete, unmaintained, or written in a stack that does not fit the project.

`floci-oci` is a Java 25 and Quarkus implementation with:

- OCI wire primitives.
- A storage abstraction with persistent mode.
- IAM, Object Storage, and Core services.
- A service pattern designed for extension.
- An MIT license.

## Decision

We will fork `floci-oci` 0.7.0 and keep the `io.floci.oci` package namespace. Emulator-specific code will live under `io.floci.oci.emulator`, and shared primitives will not be renamed.

## Consequences

### Positive

- Working OCI protocol behavior from day one.
- Reuse of storage, pagination, ETags, work requests, and compartment usage.
- A tested CLI path earlier.
- Upstream fixes can be merged.

### Negative

- The project inherits upstream design constraints.
- Upstream changes may conflict with emulator decisions.
- The Java and Quarkus versions are tied to upstream compatibility.
- Some upstream services depend on Docker and must be disabled.

### Neutral

- The repository keeps an upstream remote and a documented merge procedure.
- New services follow the upstream pattern instead of inventing a new one.

## Alternatives considered

| Alternative                                     | Why it was rejected                                                |
| ----------------------------------------------- | ------------------------------------------------------------------ |
| Build from scratch in Go with SQLite            | Too much protocol and service work before the first usable release |
| Use LocalStack-style Python emulator            | Different runtime, weaker OCI coverage                             |
| Mock only the CLI needed for demos              | No persistent control plane, no web console data model             |
| Use the real OCI SDK against a stub HTTP server | No resource model, no state transitions                            |

## References

- [ARCHITECTURE.md](../../ARCHITECTURE.md)
- [CONTRIBUTING.md](../../CONTRIBUTING.md)
- [floci-oci](https://github.com/floci-io/floci-oci)
