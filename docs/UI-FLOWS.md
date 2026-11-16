# UI Flows and Acceptance Criteria

This document defines the acceptance criteria for the main web console flows. It is the reference for frontend implementation, Playwright tests, accessibility checks, and reviews.

For visual and interaction rules, see [DESIGN.md](../DESIGN.md). For protocol behavior, see [PROTOCOL.md](PROTOCOL.md). For resource states, see [DATA-MODEL.md](DATA-MODEL.md).

---

## 1. How to use this document

Each flow includes:

- **Goal**: what the user wants to accomplish.
- **Preconditions**: required state and permissions.
- **Steps**: the expected interaction path.
- **Acceptance**: checkable outcomes.
- **Edge cases**: behavior that must not be forgotten.
- **API**: endpoints exercised by the flow.
- **Tests**: the automated coverage expected.

A flow is implemented when all acceptance items are covered by unit or end-to-end tests.

---

## 2. Global flows

### 2.1 First start and sign-in

**Goal**: open the console and reach the dashboard without real credentials.

**Preconditions**: the emulator is running and the browser has no `emu_session` cookie.

**Steps**:

1. Open `http://localhost:4599`.
2. The SSR server redirects to `/login`.
3. Enter any non-empty tenancy, user, and password, or click **Use local profile**.
4. Submit.
5. The server sets `emu_session` and redirects to `/`.

**Acceptance**:

- Unauthenticated `/console/**` requests redirect to `/login` with a `302`.
- `/login` renders with SSR and the selected language.
- Empty fields show inline errors and do not submit.
- Any non-empty values create a session.
- The dashboard loads after sign-in.
- The profile menu shows the local user and the sign-out action.
- The password is never stored, logged, or sent to the API.
- The cookie attributes are `Path=/`, `SameSite=Lax`, `Max-Age=28800`.
- The page states that credentials are local and not sent to Oracle.

**Edge cases**:

- Already authenticated and visiting `/login` redirects to `/`.
- A malformed cookie is treated as unauthenticated.
- `localhost` and `127.0.0.1` are separate cookie stores.
- `make run-lite` relies on the client guard and still works.

**API**: `POST /_emu/session/login`, `GET /_emu/session`, `POST /_emu/session/logout`.

**Tests**: Playwright login and logout, guard unit tests, SSR redirect test.

### 2.2 Sign-out

**Acceptance**:

- Sign-out clears the cookie, redirects to `/login`, and clears profile-specific UI state.
- The emulator data is not deleted.
- Going back in the browser does not restore the console.

### 2.3 Language switching

**Acceptance**:

- The language selector changes the UI immediately without a reload.
- The `emu.locale` cookie is updated.
- The `lang` attribute on `<html>` changes.
- The choice survives reloads and applies to SSR pages.
- English and Spanish keys are complete.
- No layout shift or clipped labels after switching.
- Dates, numbers, and plurals use `Intl`.

**Edge cases**:

- A missing translation falls back to English and logs a development warning.
- The browser `Accept-Language` is used only when no cookie exists.

### 2.4 Region and compartment switching

**Acceptance**:

- The top bar shows the active region and compartment.
- Changing either updates the URL query parameters.
- Lists and dashboard data refresh for the new scope.
- The selection is stored in cookies and survives reloads.
- Invalid region or compartment values fall back to the default and show a non-blocking notice.

### 2.5 Global search

**Goal**: find any resource by name or OCID.

**Acceptance**:

- `Ctrl+K` opens the dialog from any page.
- Typing filters after a short debounce.
- Results are grouped by service and show name, OCID, state, and compartment.
- Arrow keys and Enter navigate results.
- Escape closes the dialog and restores focus.
- Empty results suggest removing filters or searching by OCID.
- The dialog is reachable by keyboard and has an accessible name.

**API**: `GET /_emu/search?q=...`.

### 2.6 Notifications and work requests

**Acceptance**:

- A successful asynchronous action opens the work request tray.
- The tray polls every two seconds while the request is active.
- Polling stops when the state is terminal.
- The tray shows status, operation, affected resources, and a link to detail.
- A failed request shows the error and the request ID.
- The badge is not color-only.

**API**: `GET /20160918/workRequests`, `GET /20160918/workRequests/{id}`.

---

## 3. Networking flows

### 3.1 Create a VCN

**Goal**: create a VCN with a valid IPv4 CIDR.

**Preconditions**: signed in and a compartment is selected.

**Steps**:

1. Open **Networking > VCNs**.
2. Click **Create VCN**.
3. Enter a display name and a CIDR block.
4. Review the summary, the CLI equivalent, and the JSON preview.
5. Create.

**Acceptance**:

- The name is required and limited to 255 characters.
- The CIDR is required and must be valid IPv4.
- A CIDR that overlaps another VCN in the compartment is rejected with a field error.
- The review step shows the equivalent `bin/ocilocal network vcn create` command.
- The review step shows a JSON preview that matches the request body.
- A successful create redirects to the VCN detail page.
- The VCN is `AVAILABLE` and has a default route table, security list, and DHCP options.
- The list page shows the new VCN without a manual refresh.
- The CLI command in the review step works when copied.

**Edge cases**:

- Duplicate display names are allowed.
- A `/32` CIDR is rejected as too small.
- Network errors keep the form state and show a retry action.

**API**: `POST /20160918/vcns`, `GET /20160918/vcns`.

**Tests**: Vitest for CIDR validation, Playwright for create and detail.

### 3.2 Create a subnet

**Acceptance**:

- The VCN selector lists only VCNs in the active compartment.
- The subnet CIDR must be inside the VCN CIDR.
- The subnet CIDR cannot overlap another subnet in the VCN.
- Route table, security list, and DHCP options selectors only show resources from the same VCN.
- The subnet appears on the VCN detail page.
- A VCN with subnets cannot be deleted.

**API**: `POST /20160918/subnets`, `GET /20160918/subnets`.

### 3.3 Manage gateways and route rules

**Acceptance**:

- Internet, NAT, and service gateways can be created from the VCN detail page.
- The route table editor validates that destinations are CIDRs or service values.
- The target must be a gateway in the same VCN.
- Security list rules validate protocol and port ranges.
- Default route tables, security lists, and DHCP options cannot be deleted.
- Changes are reflected in the VCN detail tabs without a full reload.

---

## 4. Compute flows

### 4.1 Launch an instance

**Goal**: launch an instance into a subnet and watch it reach `RUNNING`.

**Preconditions**: a VCN with a subnet exists.

**Steps**:

1. Open **Compute > Instances**.
2. Click **Create instance**.
3. Enter a name, choose a compartment and availability domain.
4. Choose a shape and an image.
5. Choose a VCN and subnet.
6. Optionally attach a block volume.
7. Review and create.
8. The work request tray opens.
9. The instance reaches `RUNNING`.

**Acceptance**:

- Shape and image are required.
- Flex shapes validate OCPUs and memory against the shape limits.
- The subnet must exist and be `AVAILABLE`.
- The review shows the CLI equivalent and JSON preview.
- The create response includes `opc-work-request-id`.
- The detail page shows the work request, VNIC, private IP, and optional public IP.
- `--wait-for-state RUNNING` succeeds with the CLI.
- The list page supports start, stop, reboot, and terminate actions based on state.

**Edge cases**:

- Starting an instance that is already running returns a clear conflict.
- Terminating an instance detaches volumes after confirmation.
- Terminating an instance releases ephemeral public IPs.

**API**: `POST /20160918/instances`, `GET /20160918/instances/{id}`, `POST /20160918/instances/{id}/action`.

### 4.2 Instance actions

**Acceptance**:

- Actions are enabled only for valid states.
- Destructive actions require confirmation.
- State transitions are visible through the work request and the lifecycle badge.
- Buttons show a pending state during the request.
- A failed action shows the OCI error and the request ID.

---

## 5. Storage flows

### 5.1 Create a bucket

**Acceptance**:

- Bucket names follow the naming rules in [DATA-MODEL.md](DATA-MODEL.md) and are unique in the namespace.
- The namespace is resolved automatically.
- The create response redirects to the bucket detail page.
- The bucket shows its storage tier and object count.
- An empty bucket can be deleted.
- A non-empty bucket returns `Conflict` with a clear message.

**API**: `POST /n/{namespace}/b`, `GET /n/{namespace}/b/{bucket}`.

### 5.2 Upload an object

**Goal**: upload a file and see it in the object browser.

**Steps**:

1. Open the bucket detail page.
2. Open the **Objects** tab.
3. Click **Upload** or drop files on the drop zone.
4. Select one or more files.
5. Watch progress.
6. The objects appear in the list.

**Acceptance**:

- Drag and drop highlights the drop zone.
- Uploads show per-file progress and can be cancelled.
- Files larger than the multipart threshold use multipart upload.
- The object list shows name, size, content type, ETag, and created time.
- The object detail drawer shows metadata and a download action.
- The download returns the original bytes.
- A failed upload shows an actionable error and can be retried.
- An empty folder shows an empty state, not an error.

**API**: `PUT /n/{namespace}/b/{bucket}/o/{object}`, `GET /n/{namespace}/b/{bucket}/o`.

### 5.3 Create a pre-authenticated request

**Acceptance**:

- The PAR form offers object read, object read/write, and any object read.
- The expiration must be in the future and within the allowed range.
- The generated URL is displayed once with a copy action.
- The PAR list shows access type, target, expiration, and state.
- Revoking a PAR requires confirmation and marks it `REVOKED`.
- An expired PAR shows `EXPIRED` and cannot be used.
- A read PAR returns the object without an OCI signature.

**API**: `POST /n/{namespace}/b/{bucket}/p`, `GET /n/{namespace}/b/{bucket}/p`.

### 5.4 Create and attach a block volume

**Acceptance**:

- Size is validated between 50 GB and 32768 GB.
- The availability domain must match the instance for attachment.
- A volume can have only one attachment.
- Attaching and detaching show a work request.
- A volume cannot be deleted while attached unless the user confirms a force delete.
- The instance detail page lists attached volumes.

**API**: `POST /20160918/volumes`, `POST /20160918/volumeAttachments`.

---

## 6. Identity flows

### 6.1 Create a compartment

**Acceptance**:

- Name and description are required.
- The parent is the selected compartment or the tenancy.
- The new compartment appears in the tree without a reload.
- A compartment with children or resources cannot be deleted.
- The delete dialog lists the blocking resource types.
- The tenancy root compartment cannot be deleted.

**API**: `POST /20160918/compartments`, `DELETE /20160918/compartments/{id}`.

### 6.2 Create a policy

**Acceptance**:

- Statements are added as rows.
- An empty statement is rejected.
- The policy is stored and displayed on the detail page.
- The UI states that policies are not evaluated.
- Editing a policy updates its version date.
- Deleting a policy requires confirmation.

**API**: `POST /20160918/policies`, `PUT /20160918/policies/{id}`.

### 6.3 Manage users, groups, and memberships

**Acceptance**:

- User names are unique in the tenancy.
- A user can be added to and removed from a group.
- Removing a membership does not delete the user.
- Deactivated users show `INACTIVE` and cannot be selected for new memberships.
- Dynamic groups show their matching rule and state.

---

## 7. Load Balancer and DNS flows

### 7.1 Create a load balancer

**Acceptance**:

- The wizard collects a name, shape, and one or two subnets.
- At least one listener is required.
- A listener requires a name, protocol, port, and backend set.
- A backend set requires a policy and at least one backend.
- Backend IPs and ports are validated.
- Creation shows a work request and ends in `ACTIVE`.
- The detail page shows listeners, backend sets, backends, and simulated health.
- The UI states that no traffic is forwarded.

**API**: `POST /20200501/loadBalancers`.

### 7.2 Create a DNS zone and records

**Acceptance**:

- The zone name is a valid lowercase domain.
- A zone can be created as `PRIMARY`.
- Records support `A`, `AAAA`, `CNAME`, `TXT`, `MX`, and `NS`.
- A record value is validated against its type.
- Protected records cannot be deleted.
- The record list supports search and pagination.
- The UI states that records are not resolved.

**API**: `POST /20180115/zones`, `POST /20180115/records`.

---

## 8. Database flows

### 8.1 Create an Autonomous Database

**Acceptance**:

- Display name and DB name are required.
- DB name is uppercase alphanumeric, at most 30 characters, and follows the other rules in [DATA-MODEL.md](DATA-MODEL.md).
- OCPU count and storage follow the ranges in [DATA-MODEL.md](DATA-MODEL.md).
- The admin password meets the credential rules in [DATA-MODEL.md](DATA-MODEL.md) and is never returned.
- Creation shows a work request and ends in `AVAILABLE`.
- The detail page shows lifecycle actions, connection strings, and the wallet action.
- Start, stop, and restart are enabled only in valid states.
- The wallet download contains placeholder credentials and a warning.
- The UI states that no database engine runs.

**API**: `POST /20160918/autonomousDatabases`.

### 8.2 Create a MySQL DB system

**Acceptance**:

- Display name, shape, subnet, and storage are required.
- The admin username and password meet the rules in [DATA-MODEL.md](DATA-MODEL.md).
- The subnet must be in the selected compartment.
- Creation shows a work request and ends in `ACTIVE`.
- The detail page shows endpoints, lifecycle actions, and backups.
- The UI states that no database engine runs.

**API**: `POST /20190415/dbSystems`.

---

## 9. Emulator flows

### 9.1 Request inspector

**Acceptance**:

- The inspector lists the last 500 requests, newest first.
- Filters cover service, method, status, and free text.
- A live tail toggle follows new requests.
- The detail drawer shows method, path, status, duration, headers, bodies, and `opc-request-id`.
- Large bodies are truncated with a visible notice.
- Binary bodies show content type and size.
- Copy as CLI and copy as curl produce a usable command.
- Inspector requests and static assets are not listed.
- The page does not poll when the tab is hidden.

### 9.2 Labs

**Acceptance**:

- The catalog shows title, difficulty, duration, and completion state.
- A lab shows goal, prerequisites, steps, hints, and CLI equivalents.
- Validation shows pass and fail per check with an explanation.
- The lab is complete only when all checks pass.
- Completion is stored in local storage.
- Reset restores the lab prerequisites after confirmation.
- The lab does not claim to validate anything it does not check.

### 9.3 Seed, reset, export, and import

**Acceptance**:

- Seed and reset require confirmation.
- Reset explains that all local data will be removed.
- Import requires selecting a ZIP and confirming replacement.
- Export downloads a ZIP with a manifest.
- Errors from an invalid archive are actionable.
- The UI warns that import is not transactional.

### 9.4 Settings

**Acceptance**:

- Language, default region, and default compartment can be changed.
- The inspector capacity can be changed within the allowed range.
- The pricing table can be edited and reset.
- The about section shows the app version and the API version.
- Destructive actions require typed confirmation.

### 9.5 JSON view and CLI equivalent

**Acceptance**:

- Every detail page has a **View JSON** action.
- The JSON view is syntax highlighted and searchable.
- Copy and download work.
- The CLI equivalent is displayed for create and update flows.
- Placeholders in the CLI command are visually marked.
- Copying shows a toast.

---

## 10. Cross-cutting acceptance

Every flow must satisfy:

- Loading, empty, and error states.
- Keyboard-only operation.
- Visible focus.
- Accessible names for icon buttons.
- Form errors associated with fields.
- No color-only status.
- English and Spanish translations.
- No browser global access during SSR.
- Hydration without warnings.
- No layout shift when switching language.
- Request ID visible on API errors.
- Destructive actions confirmed.
