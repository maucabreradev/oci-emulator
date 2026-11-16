# Web Console Guide

How to use the OCI Emulator web console: sign-in, navigation, lists, wizards, detail pages, work requests, and emulator tools.

For visual and UX rules, see [DESIGN.md](../DESIGN.md). For per-flow acceptance criteria, see [UI-FLOWS.md](UI-FLOWS.md). For lab scenarios, see [LABS.md](LABS.md).

---

## 1. Sign-in page

The sign-in page mirrors the entry experience of a cloud console:

- Tenancy or cloud account name.
- User name.
- Password.
- Language selector.
- Local profile shortcut.
- Local-only notice.

Any non-empty credentials create a local session cookie. The cookie is not a security token and contains only local profile information. See [ARCHITECTURE.md](../ARCHITECTURE.md) section 7.

---

## 2. Top bar

The top bar contains:

- Product logo and name.
- Service menu toggle.
- Region selector.
- Compartment selector.
- Global search.
- Notifications.
- Help.
- Language selector.
- Profile menu with Settings and Sign out.

---

## 3. Service menu

The left sidebar groups services by area:

- Compute.
- Networking.
- Storage.
- Identity.
- Databases.
- Developer services.
- Observability and management.
- Emulator tools.

You can pin services to favorites. Recent resources appear above the full catalog.

---

## 4. Dashboard

The dashboard shows:

- Resource totals by service.
- Running instances.
- Open work requests.
- Resources by service chart.
- Instances by lifecycle state chart.
- Storage by bucket chart.
- Work requests over the last seven days.
- Simulated monthly cost.
- Recent work requests.
- Quick actions.

Charts are loaded only in the browser. Server-rendered pages show fixed-height skeletons until hydration completes.

---

## 5. Lists

Every service has a list page with:

- Search by name or OCID.
- Compartment filter.
- Lifecycle state filter.
- Tag filter.
- Server-side pagination.
- Sortable columns where the API supports sorting.
- Row actions.
- Empty state with a primary create action.

---

## 6. Creation wizards

Wizards use Angular Signal Forms and validate each step before advancing. Typical steps:

1. Basic information.
2. Configuration.
3. Networking or dependencies.
4. Review.
5. Create.

The review step shows:

- A summary of the request.
- The equivalent OCI CLI command.
- A JSON preview.
- Validation warnings.

Long-running operations return a work request and open the work request panel.

---

## 7. Detail pages

Detail pages use tabs. Common tabs:

- Overview.
- Configuration.
- Networking.
- Storage.
- Tags.
- Work requests.
- JSON.

Actions are available in the page header. Destructive actions require confirmation.

---

## 8. Work requests

The work request panel shows:

- Status.
- Operation type.
- Created and finished timestamps.
- Affected resources.
- Logs.
- Refresh and auto-refresh controls.

The panel polls every two seconds while a work request is active.

---

## 9. JSON view

Every detail page has a **View JSON** action. It shows the raw API response with syntax highlighting, copy, and download.

---

## 10. CLI equivalent

Wizards and actions display the equivalent OCI CLI command. This is useful for learning how a UI action maps to the API.

Example:

```bash
bin/ocilocal network vcn create \
  --compartment-id ocid1.compartment.oc1..demo \
  --cidr-block 10.0.0.0/16 \
  --display-name demo-vcn
```

---

## 11. Request inspector

Open **Emulator > Request Inspector** to see the last API requests:

- Method and path.
- Service.
- Status code.
- Duration.
- Request and response headers.
- Request and response bodies, truncated to 64 KB.
- `opc-request-id`.
- Copy as CLI or `curl`.

The inspector keeps the last 500 requests by default. It excludes its own requests and static assets.

---

## 12. Labs

Open **Emulator > Labs** for guided scenarios:

1. Build a VCN with a public subnet and launch an instance.
2. Create a bucket, upload an object, and generate a pre-authenticated request.
3. Create a compartment and an IAM policy.
4. Create a load balancer, a DNS zone, and a record.

Each lab includes:

- Goal and prerequisites.
- Step-by-step instructions.
- Hints.
- Equivalent CLI commands.
- Automatic validation.
- Reset action.

---

## 13. Global search

Press `Ctrl+K` or click the search box. Results are grouped by service and include resource name, OCID, lifecycle state, and compartment.

---

## 14. Settings

Settings includes:

- UI language.
- Default region.
- Default compartment.
- Theme preferences.
- Inspector capacity.
- Simulated pricing table.
- Seed and reset actions.
- Export and import.
- About and version information.

UI preference storage is documented in [CONFIGURATION.md](CONFIGURATION.md) section 5.
