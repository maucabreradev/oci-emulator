# 015 - Networking UI

- Phase: 1
- Status: Pending
- Depends on: 013, 014
- Blocks: 018, 019

## Goal

Expose the complete networking model in the console.

## Scope

- VCN detail tabs.
- Subnet list, create, and detail.
- Internet, NAT, and service gateway pages.
- Route table editor.
- Security list and NSG rule editors.
- DHCP options pages.
- Private and public IP pages.

## Deliverables

- Feature routes for networking resources.
- List, create, and detail pages.
- Rule editor components.
- Navigation and breadcrumb metadata.
- Translations.
- Playwright coverage.

## Acceptance criteria

- [ ] The subnet flow in `docs/UI-FLOWS.md` passes.
- [ ] Gateway and route rule flows pass.
- [ ] The VCN detail page shows related subnets, gateways, and route tables.
- [ ] Default resources show a protected state and no delete action.
- [ ] Rule editors validate input before submit.
- [ ] Every page has loading, empty, and error states.
- [ ] Every create flow shows the CLI equivalent and JSON preview.
- [ ] Keyboard navigation and focus states work.

## Verification

```bash
make test-ui
make e2e
```

## Out of scope

- Bulk rule editing.
- Topology visualization.
- Import and export.
