# 003 - OCI protocol core and request inspector backend

- Phase: 0
- Status: Pending
- Depends on: 002
- Blocks: 010, 037

## Goal

Implement the shared wire behavior used by every service.

## Scope

- Request ID filter.
- OCI signature parser and auth context.
- Region resolution.
- OCI error envelope and exception mapper.
- Pagination helper.
- ETag helper.
- Request inspector ring buffer and endpoints.

## Deliverables

- `RequestIdFilter`
- `OciSignatureParser`
- `OciContext`
- `OciException` and `OciExceptionMapper`
- `OciPage` integration
- `RequestInspector` and `/_emu/requests`
- Protocol tests

## Acceptance criteria

- [ ] Every response includes `opc-request-id`, including errors.
- [ ] Signed and unsigned requests both work.
- [ ] The region resolves from `x-emu-region`, then the signature, then the default.
- [ ] Errors use the OCI envelope with `code`, `message`, `status`, and `opc-request-id`.
- [ ] Invalid page tokens return `InvalidParameter`.
- [ ] A mismatched `if-match` returns `412`.
- [ ] The inspector keeps the last 500 requests and truncates bodies at 64 KB.
- [ ] The inspector excludes its own endpoints and static assets.

## Verification

```bash
make test
curl -sS -i http://localhost:4599/20160918/unknown | head -20
curl -sS http://localhost:4599/_emu/requests
```

## Out of scope

- Inspector UI.
- Persistence of inspector history.
- Signature verification.
