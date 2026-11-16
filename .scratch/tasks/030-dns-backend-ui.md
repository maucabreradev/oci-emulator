# 030 - DNS backend and UI

- Phase: 3
- Status: Pending
- Depends on: 026
- Blocks: 031

## Goal

Implement and expose DNS zones and records.

## Scope

- Zone create, list, get, update, and delete.
- Record create, list, get, update, and delete.
- Bulk record import from a zone file subset.
- Console zone list, create wizard, detail, and record editor.
- Translations and tests.

## Deliverables

- DNS service and controller under `/20180115`.
- `features/dns`
- Protocol tests.
- CLI compatibility scripts.
- Playwright coverage.

## Acceptance criteria

- [ ] Zone names are valid lowercase domains.
- [ ] Zones are created as `PRIMARY`.
- [ ] Records support `A`, `AAAA`, `CNAME`, `TXT`, `MX`, and `NS`.
- [ ] Record values are validated against their type.
- [ ] Protected records cannot be deleted.
- [ ] Record lists support search and pagination.
- [ ] The UI states that records are not resolved.
- [ ] Every page has loading, empty, and error states.
- [ ] The official CLI performs the full lifecycle.

## Verification

```bash
bin/ocilocal dns zone create \
  --compartment-id ocid1.compartment.oc1..demo \
  --name example.local \
  --zone-type PRIMARY
make compat
```

## Out of scope

- DNS resolution.
- Secondary zones.
- DNSSEC.
- Zone transfers.
