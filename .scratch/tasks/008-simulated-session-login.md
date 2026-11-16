# 008 - Simulated session and login page

- Phase: 0
- Status: Pending
- Depends on: 007
- Blocks: 009, 011

## Goal

Provide the OCI-like entry experience with a cosmetic local session.

## Scope

- Session endpoints under `/_emu/session`.
- `emu_session` cookie with base64 JSON.
- Login page with tenancy, user, password, language selector, and local profile shortcut.
- Angular auth and guest guards.
- SSR redirects.
- Profile menu and sign-out.

## Deliverables

- `SessionController`
- `SessionService` on the backend
- `console/src/app/features/auth/login`
- `console/src/app/core/auth/session.service.ts`
- `authGuard` and `guestGuard`
- SSR redirect logic in `server.ts`

## Acceptance criteria

- [ ] Any non-empty credentials create a session.
- [ ] Empty fields show inline errors and do not submit.
- [ ] The password is never stored, logged, or sent to the API.
- [ ] The cookie uses `Path=/`, `SameSite=Lax`, and `Max-Age=28800`.
- [ ] Unauthenticated `/console/**` requests redirect to `/login`.
- [ ] Authenticated `/login` requests redirect to `/`.
- [ ] Sign-out clears the cookie and returns to `/login`.
- [ ] The page states that credentials are local and not sent to Oracle.
- [ ] The login page is fully translated and keyboard accessible.

## Verification

```bash
curl -sS -i -X POST http://localhost:4599/_emu/session/login \
  -H 'Content-Type: application/json' \
  -d '{"tenancy":"ociemu","user":"admin","password":"emulator"}'
```

## Out of scope

- Real authentication.
- Password validation.
- OAuth or OIDC.
