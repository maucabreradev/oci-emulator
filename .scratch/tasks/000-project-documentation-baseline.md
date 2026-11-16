# 000 - Project documentation baseline

- Phase: 0
- Status: Done
- Depends on: none
- Blocks: 001

## Goal

Establish the written contracts that the implementation must follow.

## Scope

- Product and UX design.
- Technical architecture.
- User and developer guide.
- Compatibility, protocol, and data model contracts.
- Contribution, security, UI flow, lab, and release guides.
- Initial architecture decision records.
- Request and response examples.

## Deliverables

- `README.md`
- `DESIGN.md`
- `ARCHITECTURE.md`
- `CONTRIBUTING.md`
- `SECURITY.md`
- `docs/COMPATIBILITY.md`
- `docs/PROTOCOL.md`
- `docs/DATA-MODEL.md`
- `docs/UI-FLOWS.md`
- `docs/LABS.md`
- `docs/RELEASE.md`
- `docs/adr/0000-template.md` through `docs/adr/0006-simulated-login.md`
- `docs/examples/` with VCN, instance, and bucket examples

## Acceptance criteria

- [x] All documents are in English and contain no em dashes.
- [x] Internal links and anchors resolve.
- [x] No Oracle logos, fonts, icons, or screenshots.
- [x] Phase scope and rendering decisions are consistent across documents.
- [x] JSON examples are valid.

## Verification

```bash
node -e "for (const f of require('fs').readdirSync('docs/examples')) if (f.endsWith('.json')) JSON.parse(require('fs').readFileSync('docs/examples/' + f, 'utf8'))"
find . -name '*.md' -not -path './.git/*' -exec perl -ne 'print if /\x{2014}/' {} +
```

## Out of scope

- OpenAPI specifications.
- Separate operations, roadmap, glossary, or changelog documents.
