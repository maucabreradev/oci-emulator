# ADR 0006: Simulated login with a cosmetic session cookie

- Status: Accepted
- Date: 2026-11-16
- Deciders: Project maintainer
- Supersedes: none
- Superseded by: none

## Context

The web console needs a recognizable entry experience. Users expect a sign-in page before a cloud console, and the project needs a session concept for redirects, the profile menu, and the selected region.

Implementing real authentication would require an identity provider, password storage, token signing, and a security model that the project explicitly does not provide.

The OCI API must stay usable by the CLI and SDKs without a browser session.

## Decision

We will implement a simulated login:

- `POST /_emu/session/login` accepts any non-empty tenancy, user, and password.
- The password is ignored and never stored.
- The server sets the `emu_session` cookie with base64 JSON profile data.
- The cookie is unsigned and not `HttpOnly`.
- Angular guards and the SSR server use the cookie only to decide which pages to show.
- The OCI API accepts signed and unsigned requests and does not require a session.

## Consequences

### Positive

- A familiar entry flow with no identity infrastructure.
- The CLI works without a browser session.
- No credential handling or password storage.
- Easy reset by clearing site data.

### Negative

- The login is not a security control.
- The cookie is not signed and can be edited by the user.
- The emulator must never be exposed to an untrusted network.
- Documentation must state this clearly to avoid false expectations.

### Neutral

- The session payload includes tenancy, user, display name, and region.
- The session TTL defaults to 8 hours.
- The SSR server redirects unauthenticated `/console/**` requests and authenticated `/login` requests.

## Alternatives considered

| Alternative        | Why it was rejected                                                |
| ------------------ | ------------------------------------------------------------------ |
| No login page      | Loses the console entry experience                                 |
| Real OAuth or OIDC | Out of scope, requires external infrastructure                     |
| Signed JWT session | Implies a security boundary the project does not provide           |
| HTTP Basic auth    | Does not match the OCI console flow and still has no real identity |

## References

- [ARCHITECTURE.md](../../ARCHITECTURE.md)
- [SECURITY.md](../../SECURITY.md)
- [docs/UI-FLOWS.md](../UI-FLOWS.md)
