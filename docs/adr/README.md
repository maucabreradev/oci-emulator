# Architecture Decision Records

This directory records the architectural decisions for OCI Emulator.

| ADR                                     | Title                                             | Status   |
| --------------------------------------- | ------------------------------------------------- | -------- |
| [0000](0000-template.md)                | Template                                          | Template |
| [0001](0001-fork-floci-oci.md)          | Fork floci-oci as the backend base                | Accepted |
| [0002](0002-hybrid-rendering.md)        | Hybrid rendering with SSR, SSG, and CSR per route | Accepted |
| [0003](0003-node-ssr-quarkus-proxy.md)  | Node SSR server behind a Quarkus UI proxy         | Accepted |
| [0004](0004-transloco-static-loader.md) | Transloco with a static translation loader        | Accepted |
| [0005](0005-floci-storage.md)           | Use the floci persistent storage backend          | Accepted |
| [0006](0006-simulated-login.md)         | Simulated login with a cosmetic session cookie    | Accepted |

## Adding a decision

1. Copy `0000-template.md` to the next number, for example `0007-short-title.md`.
2. Fill in context, decision, consequences, and alternatives.
3. Keep the status `Proposed` until the decision is accepted.
4. Link the ADR from the decision log in [ARCHITECTURE.md](../../ARCHITECTURE.md).
5. Update this index.

Use an ADR when a decision is likely to be questioned, is expensive to reverse, or changes a cross-cutting rule. Do not duplicate decisions that are already stable and fully described in the architecture document.
