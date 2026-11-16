# ADR 0003: Node SSR server behind a Quarkus UI proxy

- Status: Accepted
- Date: 2026-11-16
- Deciders: Project maintainer
- Supersedes: none
- Superseded by: none

## Context

The backend is Quarkus and serves the OCI API. The frontend is Angular 22 with SSR. Angular SSR requires a Node process in production.

The project needs one public origin so that:

- The `emu_session` cookie works without cross-origin rules.
- API calls from the browser are same-origin.
- Users open one URL, not two.
- The CLI and the web console share the same host and port.

## Decision

In production, Quarkus listens on the public port `4599` and forwards UI requests to a Node SSR server on `127.0.0.1:4600`. API routes and `/_emu` routes are handled directly by Quarkus and are never proxied. The proxy is a Vert.x HTTP proxy registered as a fallback route.

A `make run-lite` mode disables the proxy and serves the prerendered browser bundle directly from Quarkus for environments without Node.

## Consequences

### Positive

- One public origin and one cookie scope.
- The API and the UI share the same port.
- Node can be omitted in lite mode.
- The proxy can be tested independently.

### Negative

- Production requires two processes.
- The proxy adds a network hop.
- Header forwarding, cookies, and binary assets need explicit handling.
- A crashed SSR server produces a `502` unless the error page is implemented.

### Neutral

- Development uses Quinoa instead of the proxy.
- The proxy is enabled only in the `prod` profile.

## Alternatives considered

| Alternative                                 | Why it was rejected                                             |
| ------------------------------------------- | --------------------------------------------------------------- |
| Separate UI port                            | Cookie and CORS complexity, two URLs for users                  |
| Serve the Angular SSR bundle inside Quarkus | Not supported by Angular SSR                                    |
| Nginx or Caddy in front                     | Extra dependency, violates the no-Docker and minimal setup goal |
| SSG only in production                      | No authenticated SSR data                                       |

## References

- [ARCHITECTURE.md](../../ARCHITECTURE.md)
- [README.md](../../README.md)
