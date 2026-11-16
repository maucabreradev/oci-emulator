# OCI Emulator Design

This document defines the product, interaction, and visual design of OCI Emulator. For installation and operation, see [README.md](README.md). For the technical architecture, see [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Table of contents

- [1. Purpose](#1-purpose)
- [2. Audience and scenarios](#2-audience-and-scenarios)
- [3. Design principles](#3-design-principles)
- [4. Terminology](#4-terminology)
- [5. User journeys](#5-user-journeys)
- [6. Information architecture](#6-information-architecture)
- [7. Visual identity](#7-visual-identity)
- [8. Layout system](#8-layout-system)
- [9. Screen designs](#9-screen-designs)
- [10. Page inventory](#10-page-inventory)
- [11. Interaction patterns](#11-interaction-patterns)
- [12. Dashboard and charts](#12-dashboard-and-charts)
- [13. Content and i18n](#13-content-and-i18n)
- [14. Accessibility](#14-accessibility)
- [15. Responsive behavior](#15-responsive-behavior)
- [16. Perceived performance](#16-perceived-performance)
- [17. Error and empty states](#17-error-and-empty-states)
- [18. Labs design](#18-labs-design)
- [19. Design QA checklist](#19-design-qa-checklist)
- [20. Future design work](#20-future-design-work)

---

## 1. Purpose

OCI Emulator provides a local, hands-on OCI environment for people who want to learn and practice without an Oracle Cloud account.

The design has two goals:

1. Make the web console feel immediately familiar to anyone who has used OCI Console.
2. Make the emulator easier to learn from than a real cloud by exposing requests, CLI equivalents, and guided labs.

The result is a functional imitation of OCI workflows with its own visual identity. It does not copy Oracle logos, fonts, icons, screenshots, or proprietary text.

---

## 2. Audience and scenarios

### 2.1 Primary audiences

| Audience                 | Need                                          |
| ------------------------ | --------------------------------------------- |
| Cloud beginners          | Learn OCI concepts without cost or risk       |
| Certification candidates | Practice resource creation and navigation     |
| Developers               | Test SDK and CLI integrations locally         |
| Platform engineers       | Reproduce flows and validate automation       |
| Instructors              | Build guided exercises with predictable state |

### 2.2 Main scenarios

1. Sign in and explore the dashboard.
2. Create a compartment and a VCN.
3. Launch an instance and attach a block volume.
4. Create a bucket and upload an object.
5. Generate a pre-authenticated request.
6. Create an IAM policy and a dynamic group.
7. Create a load balancer and a DNS zone.
8. Create an Autonomous Database and obtain its connection strings.
9. Compare a UI action with the equivalent CLI command.
10. Inspect the API request produced by the CLI.
11. Complete a guided lab and validate the result.
12. Reset or seed the environment.

### 2.3 Assumptions

- The user runs the emulator locally.
- The user has a browser and optionally a terminal.
- The user may have no prior OCI experience.
- The user may not have an Oracle Cloud account.
- The user is the only person using the instance.

---

## 3. Design principles

### 3.1 Familiar first

Navigation, page structure, terminology, and resource lifecycles follow OCI conventions. A user who knows OCI Console should recognize the layout immediately.

### 3.2 Own identity

The product uses its own name, logo, colors, typography, and icons. It does not use Oracle assets. The interface is inspired by OCI, not copied from it.

### 3.3 Show the API

The UI never hides the API. Every resource detail page exposes JSON, every create flow shows the equivalent CLI command, and the inspector records the exact request.

### 3.4 Learn by doing

Labs validate real state. Hints are progressive. Errors explain what to fix, not just what failed.

### 3.5 Safe by default

Destructive actions require confirmation. Reset and import are clearly marked. The emulator warns that data is local and not durable.

### 3.6 Calm density

The console is information-dense but readable. Tables, tabs, and cards are preferred over deep nested menus. Primary actions are visually distinct.

### 3.7 Predictable state

Lifecycle badges, work request status, and loading states use consistent language and color. The user should always know whether an operation is pending, running, or complete.

---

## 4. Terminology

Use OCI terminology consistently:

| Term                | Meaning in this project                |
| ------------------- | -------------------------------------- |
| Tenancy             | The local simulated tenancy            |
| Compartment         | Logical resource container             |
| Region              | Simulated region label                 |
| Availability domain | Simulated AD label                     |
| OCID                | OCI resource identifier                |
| Work request        | Asynchronous operation record          |
| VCN                 | Virtual Cloud Network                  |
| NSG                 | Network Security Group                 |
| PAR                 | Pre-authenticated request              |
| ADB                 | Autonomous Database                    |
| Console             | The web user interface, not a terminal |

Avoid calling the web UI a "terminal" or "shell". The OCI CLI is the terminal experience.

---

## 5. User journeys

### 5.1 First run

```text
Start emulator
  -> Open localhost:4599
  -> Login page
  -> Use local profile or enter any credentials
  -> Dashboard with empty state
  -> Seed demo data or create the first resource
```

Success criteria:

- The user understands that no real credentials are needed.
- The user can reach the dashboard in under one minute.
- The empty state offers a clear next action.

### 5.2 Create a VCN and a subnet

```text
Networking > VCNs
  -> Create VCN
  -> Name, compartment, CIDR
  -> Review
  -> Create
  -> VCN detail
  -> Create subnet
  -> Choose CIDR and route table
  -> Review
  -> Create
```

Success criteria:

- Validation prevents invalid CIDRs and overlaps.
- The UI shows the CLI equivalent.
- The detail page shows both resources and the route table association.

### 5.3 Launch an instance

```text
Compute > Instances
  -> Create instance
  -> Name, compartment, AD, shape, image
  -> Select VCN and subnet
  -> Optional block volume
  -> Review
  -> Create
  -> Work request panel
  -> Detail page with RUNNING state
```

Success criteria:

- The work request is visible.
- The state transition is clear.
- The instance detail shows networking and storage.

### 5.4 Upload an object

```text
Storage > Buckets
  -> Select bucket
  -> Objects tab
  -> Upload
  -> Drag and drop or choose files
  -> Progress
  -> Object appears in the list
  -> Download or create PAR
```

Success criteria:

- Large files use multipart upload.
- Errors are actionable.
- The object metadata page shows size, content type, ETag, and storage tier.

### 5.5 Learn from the CLI

```text
User runs a CLI command
  -> Request Inspector
  -> Find the request
  -> View method, path, headers, body, response
  -> Copy as CLI or curl
```

Success criteria:

- Requests are easy to filter.
- The inspector shows the exact request.
- The user can connect the terminal command to the API call.

### 5.6 Complete a lab

```text
Emulator > Labs
  -> Select lab
  -> Read goal and steps
  -> Create resources in the console or CLI
  -> Validate
  -> See which checks passed and which failed
```

Success criteria:

- Validation is deterministic.
- Failed checks explain what is missing.
- Reset returns the lab to a clean state.

---

## 6. Information architecture

### 6.1 Top-level navigation

```text
Dashboard
Compute
  Instances
  Images
  Shapes
  Instance configurations
Networking
  VCNs
  Subnets
  Route tables
  Security lists
  Network security groups
  Internet gateways
  NAT gateways
  Service gateways
  DHCP options
  Private IPs
  Public IPs
Storage
  Buckets
  Object browser
  Pre-authenticated requests
  Volumes
  Volume backups
Identity
  Compartments
  Users
  Groups
  Policies
  Dynamic groups
Databases
  Autonomous Databases
  MySQL DB systems
Developer services
  API Gateways
  Load Balancers
  DNS zones
Work requests
Emulator
  Dashboard
  Request inspector
  Labs
  Search
  Settings
  Import and export
Documentation
  Getting started
  CLI guide
  API compatibility
  Limitations
  Glossary
```

### 6.2 Navigation model

- The top bar is global: region, compartment, search, notifications, help, language, profile.
- The sidebar is the service catalog.
- Breadcrumbs show the path from service to resource.
- Tabs organize detail pages.
- The work request tray is global and can be opened from any page.

### 6.3 URL model

```text
/login
/
/console/dashboard
/console/compute/instances
/console/compute/instances/:id
/console/network/vcns
/console/network/vcns/:id
/console/storage/buckets/:id/objects
/console/iam/compartments
/console/database/autonomous
/console/work-requests/:id
/console/emulator/inspector
/console/emulator/labs
/console/emulator/settings
/docs
/help
/labs
/about
```

Region and compartment are query parameters so links are shareable:

```text
/console/compute/instances?region=us-ashburn-1&compartment=ocid1.compartment.oc1..demo
```

---

## 7. Visual identity

### 7.1 Name and voice

Name: **OCI Emulator**.

Tagline: **Local OCI practice environment**.

Voice:

- Direct and factual.
- No marketing language.
- Short sentences.
- Explain errors in terms of the user's action.
- Use OCI terminology, not invented synonyms.

### 7.2 Logo

- Simple geometric mark.
- Own SVG asset.
- No Oracle shapes, colors, or typography.
- Works at 24 px in the top bar and at 96 px on the login page.
- Monochrome and full-color variants.

### 7.3 Typography

| Role             | Font           | Fallback                |
| ---------------- | -------------- | ----------------------- |
| UI               | Inter          | system-ui, sans-serif   |
| Code, OCID, JSON | JetBrains Mono | ui-monospace, monospace |

Both fonts are self-hosted under the SIL Open Font License.

Type scale:

| Token     | Size  | Line height | Use           |
| --------- | ----- | ----------- | ------------- |
| `display` | 32 px | 40 px       | Login hero    |
| `h1`      | 24 px | 32 px       | Page title    |
| `h2`      | 20 px | 28 px       | Section title |
| `h3`      | 16 px | 24 px       | Card title    |
| `body`    | 14 px | 22 px       | Default text  |
| `small`   | 12 px | 18 px       | Metadata      |
| `code`    | 13 px | 20 px       | OCID and JSON |

### 7.4 Color palette

| Token                 | Value     | Use                        |
| --------------------- | --------- | -------------------------- |
| `--emu-bg`            | `#F5F7FA` | Page background            |
| `--emu-surface`       | `#FFFFFF` | Cards and tables           |
| `--emu-topbar`        | `#1B1F24` | Top bar                    |
| `--emu-text`          | `#1F2933` | Primary text               |
| `--emu-muted`         | `#5B6773` | Secondary text             |
| `--emu-accent`        | `#D95D39` | Primary actions            |
| `--emu-accent-strong` | `#B94726` | Hover and active           |
| `--emu-success`       | `#1F7A4D` | Running, active, succeeded |
| `--emu-warning`       | `#B7791F` | Pending, warning           |
| `--emu-danger`        | `#B42318` | Failed, destructive        |
| `--emu-info`          | `#2563A6` | Informational              |
| `--emu-border`        | `#D9E0E7` | Borders and dividers       |
| `--emu-focus`         | `#F59E0B` | Keyboard focus ring        |

Rules:

- Status is never communicated by color alone. Use a badge with text and an icon.
- Primary action uses the accent color. Secondary actions use neutral outlines.
- Destructive actions use the danger color and require confirmation.
- Charts use a color-blind safe sequence derived from the palette.

### 7.5 Spacing and shape

| Token       | Value |
| ----------- | ----- |
| `space-1`   | 4 px  |
| `space-2`   | 8 px  |
| `space-3`   | 12 px |
| `space-4`   | 16 px |
| `space-6`   | 24 px |
| `space-8`   | 32 px |
| `space-12`  | 48 px |
| `radius-sm` | 4 px  |
| `radius-md` | 8 px  |
| `radius-lg` | 12 px |

### 7.6 Elevation

- Top bar: no shadow, dark surface.
- Cards: 1 px border, subtle shadow only on hover.
- Dialogs: medium shadow.
- Dropdowns: medium shadow.
- Work request tray: strong shadow.

### 7.7 Iconography

- Lucide icons.
- 16 px for inline, 20 px for buttons, 24 px for navigation.
- Icons are decorative unless they carry meaning. Interactive icon buttons have an accessible name.
- No Oracle icon set.

---

## 8. Layout system

### 8.1 Desktop shell

```text
+--------------------------------------------------------------+
| Logo | Menu | Region | Compartment | Search | Help | Lang | Me |
+---------+----------------------------------------------------+
|         | Breadcrumbs                                        |
| Sidebar | Page title                          Primary action |
|         | Subtitle / metadata                                |
|         | Tabs                                               |
|         | Content                                            |
|         |                                                    |
+---------+----------------------------------------------------+
| Work request tray (collapsible)                              |
+--------------------------------------------------------------+
```

Measurements:

- Top bar: 56 px.
- Sidebar: 260 px expanded, 64 px collapsed.
- Content max width: 1440 px.
- Content padding: 24 px desktop, 16 px tablet.
- Table row height: 44 px.
- Form field height: 40 px.

### 8.2 Page header

Every page has:

- Breadcrumbs.
- Title.
- Optional description.
- Compartment and region context.
- Primary action.
- Secondary actions in an overflow menu.

### 8.3 Cards

Cards are used for:

- Summary metrics.
- Grouped details.
- Empty states.
- Chart containers.

Cards do not nest more than one level deep.

### 8.4 Tables

- Sticky header.
- Dense rows.
- Column alignment: text left, numbers right, status centered.
- Row click opens the detail page.
- Row actions are available from an overflow menu and from the detail page.
- Pagination is server-side.
- Filters are visible above the table.
- Column visibility can be configured for large tables.

---

## 9. Screen designs

### 9.1 Login

Purpose: provide a familiar entry experience without real authentication.

Layout:

- Split screen on desktop.
- Left panel: dark background, logo, product name, one-line description, local region, and a small “local only” notice.
- Right panel: light surface with the form.
- Mobile: single column with the brand block on top.

Fields:

- Cloud account or tenancy.
- User name.
- Password.
- Language selector.
- Sign in button.
- Use local profile link.
- Notice: any credentials work.

States:

- Idle.
- Validating.
- Error if fields are empty.
- Redirect if already signed in.

Microcopy:

- Title: “Sign in to OCI Emulator”.
- Subtitle: “Use any local credentials. Nothing is sent to Oracle.”
- Button: “Sign in”.
- Secondary: “Use local profile”.

### 9.2 Dashboard

Purpose: give a quick overview of the emulated environment.

Sections:

1. Page header with region, compartment, seed action.
2. KPI cards.
3. Charts.
4. Recent work requests.
5. Quick actions.
6. Recent resources.

KPI cards:

- Compartments.
- Running instances.
- VCNs.
- Buckets.
- Databases.
- Open work requests.

Each card shows the count, a short label, and a link to the filtered list.

Charts:

- Resources by service.
- Instances by state.
- Storage by bucket.
- Work requests over seven days.
- Simulated monthly cost.

Empty state:

- Title: “Your emulator is empty”.
- Body: “Seed demo data or create your first resource.”
- Primary: “Seed demo data”.
- Secondary: “Create a VCN”.

### 9.3 List page

Structure:

- Breadcrumbs.
- Title.
- Description.
- Primary create action.
- Filter bar: search, compartment, state, tags.
- Table.
- Pagination.
- Empty state.

Table columns vary by service, but always include:

- Name.
- OCID, truncated with copy.
- Compartment.
- Lifecycle state.
- Created.
- Actions.

### 9.4 Create wizard

Structure:

- Page title and description.
- Step indicator.
- Form area.
- Contextual help panel.
- Footer with Back, Next, and Create.

Steps:

1. Basic information.
2. Configuration.
3. Dependencies.
4. Review.

Review step:

- Summary of values.
- Validation summary.
- CLI equivalent in a code block with copy.
- JSON preview.
- Create button.

Behavior:

- The user can go back without losing data.
- Validation is per step and on submit.
- Errors are shown next to the field and summarized at the top.
- Leaving with unsaved changes prompts for confirmation.

### 9.5 Detail page

Structure:

- Breadcrumbs.
- Resource name and lifecycle badge.
- OCID with copy.
- Compartment and region.
- Primary actions.
- Secondary actions.
- Tabs.

Tabs:

- Overview.
- Configuration.
- Networking.
- Storage.
- Tags.
- Work requests.
- JSON.

Overview includes:

- Key metrics.
- Created and updated timestamps.
- Related resources.
- Recent activity.

### 9.6 Object browser

Layout:

- Bucket header.
- Breadcrumb path inside the bucket.
- Upload button.
- Search and filter.
- Table or grid view.
- Object detail drawer.

Actions:

- Upload files or folders.
- Create folder.
- Download.
- Copy object path.
- Delete.
- Create PAR.
- View metadata.

Behavior:

- Drag and drop highlights the drop zone.
- Uploads show progress and can be cancelled.
- Large files show multipart progress.
- Empty folders show a clear empty state.

### 9.7 Work request panel

Layout:

- Collapsible tray at the bottom.
- List of active and recent work requests.
- Status badge.
- Progress.
- Link to detail.

Detail view:

- Operation type.
- Status.
- Timestamps.
- Affected resources.
- Logs.
- Auto-refresh toggle.

### 9.8 Request inspector

Layout:

- Filter bar: service, method, status, text search.
- Live tail toggle.
- Request table.
- Detail drawer.

Table columns:

- Time.
- Method.
- Path.
- Status.
- Duration.
- Request ID.

Detail drawer:

- Request headers.
- Request body.
- Response headers.
- Response body.
- Copy as CLI.
- Copy as curl.

### 9.9 Labs

Layout:

- Catalog with cards: title, difficulty, duration, completion state.
- Lab detail with goal, prerequisites, steps, hints, CLI equivalents, validation, and reset.

Validation:

- Each check has a pass or fail state.
- Failed checks explain what is missing.
- The lab is complete only when all checks pass.

### 9.10 Settings

Sections:

- Appearance and language.
- Default region and compartment.
- Emulator behavior.
- Inspector capacity.
- Pricing table.
- Data: seed, reset, export, import.
- About and version.

Destructive actions:

- Require typed confirmation for reset and import.
- Explain exactly what will be removed.

### 9.11 Documentation

Public pages available without login:

- Getting started.
- CLI guide.
- API compatibility.
- Limitations.
- Glossary.
- About.

These pages are prerendered and searchable. They do not call the API.

### 9.12 Global search

Dialog opened with `Ctrl+K`.

Sections:

- Recent searches.
- Resources grouped by service.
- Documentation.
- Actions, such as “Create VCN”.

Keyboard:

- Arrow keys navigate.
- Enter opens.
- Escape closes.

### 9.13 Error pages

- 404: page not found, with links to dashboard and search.
- 500: unexpected error, with request ID and retry.
- 502: UI proxy cannot reach the SSR server, with instructions.
- 403: simulated authorization failure, with an explanation that policies are not enforced.

---

## 10. Page inventory

| Area       | Pages                               | Key actions                                                       |
| ---------- | ----------------------------------- | ----------------------------------------------------------------- |
| Auth       | Login                               | Sign in, local profile, language                                  |
| Dashboard  | Overview                            | KPIs, charts, quick actions, seed                                 |
| Compute    | Instances list, create, detail      | Launch, start, stop, reboot, terminate, attach volume, tags, JSON |
| Compute    | Images list, detail                 | View image metadata                                               |
| Compute    | Shapes list                         | Compare shapes                                                    |
| Compute    | Instance configurations             | List, create, detail                                              |
| Networking | VCNs list, create, detail           | Create, edit, delete, view subnets and gateways                   |
| Networking | Subnets list, create, detail        | Create, edit, delete, associate route table                       |
| Networking | Route tables list, create, detail   | Add, edit, delete rules                                           |
| Networking | Security lists list, create, detail | Add, edit, delete ingress and egress rules                        |
| Networking | NSGs list, create, detail           | Add, edit, delete rules                                           |
| Networking | Internet gateways                   | Create, enable, disable, delete                                   |
| Networking | NAT gateways                        | Create, delete                                                    |
| Networking | Service gateways                    | Create, delete                                                    |
| Networking | DHCP options                        | Create, edit, delete                                              |
| Networking | Private IPs                         | List, assign, unassign                                            |
| Networking | Public IPs                          | List, assign, release                                             |
| Storage    | Buckets list, create, detail        | Create, delete, view metrics                                      |
| Storage    | Object browser                      | Upload, download, delete, folder, metadata                        |
| Storage    | PARs                                | Create, list, revoke, copy URL                                    |
| Storage    | Lifecycle policies                  | Create, edit, delete                                              |
| Storage    | Volumes list, create, detail        | Create, attach, detach, resize, delete                            |
| Storage    | Volume backups                      | Create, restore, delete                                           |
| Identity   | Compartments                        | Create, edit, delete, move resources                              |
| Identity   | Users                               | Create, edit, deactivate, API keys                                |
| Identity   | Groups                              | Create, edit, memberships                                         |
| Identity   | Policies                            | Create, edit, delete, view statements                             |
| Identity   | Dynamic groups                      | Create, edit, delete                                              |
| Databases  | Autonomous Databases                | Create, start, stop, clone, connection strings, wallet, backups   |
| Databases  | MySQL DB systems                    | Create, start, stop, connection details, backups                  |
| Developer  | Load Balancers                      | Create wizard, listeners, backend sets, backends, health          |
| Developer  | API Gateways                        | Create, deploy, routes                                            |
| Developer  | DNS zones                           | Create, records, import                                           |
| Operations | Work requests                       | List, detail, logs, resources                                     |
| Emulator   | Dashboard                           | Overview and shortcuts                                            |
| Emulator   | Request inspector                   | Filter, inspect, copy as CLI or curl                              |
| Emulator   | Labs                                | Catalog, run, validate, reset                                     |
| Emulator   | Search                              | Global search                                                     |
| Emulator   | Settings                            | Preferences, pricing, data actions                                |
| Emulator   | Import and export                   | Export, import, reset                                             |
| Docs       | Getting started                     | Setup and first steps                                             |
| Docs       | CLI guide                           | Commands and profiles                                             |
| Docs       | API compatibility                   | Endpoints and limitations                                         |
| Docs       | Glossary                            | OCI terms                                                         |
| Global     | Notifications                       | Work request updates and errors                                   |
| Global     | Profile menu                        | Settings, sign out                                                |
| Global     | 404, 500, 502, 403                  | Recovery actions                                                  |

---

## 11. Interaction patterns

### 11.1 Primary actions

- One primary action per page.
- Primary action is placed in the page header, right aligned.
- In wizards, the primary action is in the footer.
- Destructive actions are never primary.

### 11.2 Confirmation

Use a confirmation dialog for:

- Delete.
- Terminate.
- Reset.
- Import.
- Revoke PAR.
- Detach volume.

The dialog states the resource name, the consequence, and whether the action can be undone. For irreversible actions, require typing the resource name.

### 11.3 Notifications

Use toasts for:

- Successful create, update, or delete.
- Copy actions.
- Non-blocking errors.

Use inline alerts for:

- Validation errors.
- Partial failures.
- Warnings that affect the current page.

Use the work request tray for:

- Long-running operations.
- Operations with multiple resources.

### 11.4 Loading

- Skeletons for page-level loads.
- Spinners inside buttons for submissions.
- Progress bars for uploads.
- The work request panel for asynchronous state transitions.
- Do not replace the whole page with a spinner if the shell is already visible.

### 11.5 Search and filters

- Search is debounced.
- Filters apply immediately.
- Active filters are shown as removable chips.
- Clear all resets to the default view.
- Filters are reflected in the URL when practical.

### 11.6 Tables

- Row click opens detail.
- Overflow menu contains actions.
- Selection enables bulk actions only when the service supports them.
- Pagination preserves filters.
- Empty search results differ from an empty resource list.

### 11.7 Forms

- Labels above fields.
- Required fields marked with an asterisk.
- Help text below the field.
- Errors below the field.
- Inline validation on blur and on submit.
- Review step before creation.
- Unsaved changes prompt.

### 11.8 OCIDs

- Show the first and last segments with a copy button.
- Full OCID available on hover or in the detail metadata.
- Copy action shows a toast.
- OCIDs use the monospace font.

### 11.9 Tags

- Freeform tags as key-value rows.
- Defined tags grouped by namespace.
- Tag editor supports add, edit, and remove.
- Tags are searchable from the list filters.

### 11.10 JSON view

- Syntax highlighted.
- Copy and download.
- Search inside the JSON.
- Collapsible objects and arrays.
- Clearly labeled as the raw API response.

### 11.11 CLI equivalent

- Shown in create and update flows.
- Monospace block with copy.
- Uses the local wrapper name `bin/ocilocal`.
- Warns when the command contains placeholders.

### 11.12 Keyboard shortcuts

| Shortcut     | Action                 |
| ------------ | ---------------------- |
| `Ctrl+K`     | Global search          |
| `Esc`        | Close dialog or drawer |
| `?`          | Shortcut help          |
| `G` then `D` | Go to dashboard        |
| `G` then `I` | Go to instances        |
| `G` then `N` | Go to VCNs             |

Shortcuts are disabled while typing in an input.

---

## 12. Dashboard and charts

### 12.1 Dashboard hierarchy

1. Context: region and compartment.
2. Health: open work requests and recent failures.
3. Inventory: KPI cards.
4. Trends: charts.
5. Activity: recent work requests and resources.
6. Actions: quick create.

### 12.2 KPI cards

Each card contains:

- Label.
- Count.
- Icon.
- Link to the filtered list.
- Optional secondary text, such as “2 provisioning”.

### 12.3 Charts

| Chart                | Type           | Data                              |
| -------------------- | -------------- | --------------------------------- |
| Resources by service | Doughnut       | Count per service                 |
| Instances by state   | Vertical bar   | Count per lifecycle state         |
| Storage by bucket    | Horizontal bar | Bytes per bucket                  |
| Work requests        | Line           | Requests per day, last seven days |
| Estimated cost       | Line           | Simulated monthly cost trend      |

Chart rules:

- Use the project palette.
- Label axes and units.
- Show a legend when there is more than one series.
- Provide a data table alternative.
- Use tooltips with exact values.
- Do not use 3D or decorative effects.
- Keep chart height stable between SSR and hydration.

### 12.4 Cost simulation

The cost chart is explicitly labeled “Simulated”. It uses a local price table editable in Settings. It is a learning aid, not a billing estimate.

### 12.5 Empty dashboard

When there are no resources:

- Show a short explanation.
- Offer “Seed demo data”.
- Offer “Create a VCN”.
- Keep the KPI cards visible with zero values.

---

## 13. Content and i18n

### 13.1 Languages

- English: default and source language.
- Spanish: complete translation.

Future languages must be added to the locale service, translation files, and language selector.

### 13.2 Translation rules

- All user-visible strings go through Transloco.
- Keys are namespaced by feature.
- OCIDs, resource names, API values, and JSON keys are not translated.
- Dates and numbers use `Intl` with the active locale.
- Plurals use ICU syntax.
- Missing keys fall back to English and log a development warning.

### 13.3 Content style

- Sentence case for headings and buttons.
- Active voice.
- Short sentences.
- No exclamation marks.
- No marketing language.
- Explain what happened and what to do next.
- Use “you” sparingly. Prefer direct instructions.

### 13.4 Error copy

Structure:

1. What failed.
2. Why.
3. What to do.

Example:

```text
The VCN could not be created.
The CIDR block overlaps with an existing subnet.
Choose a CIDR block that does not overlap with 10.0.1.0/24.
```

### 13.5 Empty state copy

Structure:

1. What this page shows.
2. Why it is empty.
3. The next action.

Example:

```text
No instances yet.
Instances run in a compartment and require a subnet.
Create an instance or seed demo data.
```

### 13.6 Dates and numbers

- Dates: locale-aware, with timezone label where relevant.
- Relative times: “2 minutes ago” in activity feeds, with an absolute timestamp tooltip.
- Bytes: binary units with a localized number.
- Currency: USD, labeled simulated.

---

## 14. Accessibility

Target: WCAG 2.2 AA.

### 14.1 Requirements

- All interactive elements are keyboard reachable.
- Focus is visible and follows a logical order.
- Dialogs trap focus and restore it on close.
- Icon-only buttons have accessible names.
- Form fields have labels and descriptions.
- Errors are associated with fields using `aria-describedby`.
- Status badges include text, not only color.
- Charts have data table alternatives.
- Tables have captions or accessible names.
- Reduced motion is respected.
- Contrast meets AA in all surfaces.
- Language changes update the `lang` attribute.

### 14.2 Testing

- axe-core in Playwright for key pages.
- Keyboard-only walkthrough for login, dashboard, list, wizard, and detail.
- Screen reader spot checks for dialogs and tables.
- Zoom to 200% without loss of function.

---

## 15. Responsive behavior

Priority: desktop first.

| Breakpoint        | Behavior                                                          |
| ----------------- | ----------------------------------------------------------------- |
| `>= 1280px`       | Full sidebar, multi-column dashboard, full tables                 |
| `1024px - 1279px` | Collapsible sidebar, two-column dashboard                         |
| `768px - 1023px`  | Overlay sidebar, stacked cards, horizontal table scroll           |
| `< 768px`         | Single column, condensed top bar, table scroll, simplified charts |

Mobile is supported for exploration and status checks. Complex wizards may require horizontal scrolling and are optimized for tablet and desktop.

---

## 16. Perceived performance

- Render the shell immediately with SSR.
- Use skeletons for content regions.
- Keep chart containers at a fixed height.
- Defer charts and heavy tabs.
- Prefetch likely next routes on hover.
- Keep filters and pagination responsive.
- Show optimistic feedback for actions that complete synchronously.
- Use the work request tray for asynchronous operations.
- Avoid full-page spinners after the shell is visible.

Budgets:

- Initial JS under 500 KB gzip.
- LCP under 1.5 seconds locally.
- CLS under 0.1.
- No layout shift when switching language.

---

## 17. Error and empty states

### 17.1 Empty resource list

- Explain what the resource is.
- Explain the prerequisites.
- Offer a primary create action.
- Offer a documentation link.

### 17.2 Empty search results

- Show the query.
- Suggest removing filters.
- Suggest searching by OCID.
- Offer global search.

### 17.3 API error

- Show the OCI error code.
- Show the message.
- Show the request ID.
- Offer retry.
- Link to the request inspector.

### 17.4 Validation error

- Inline message next to the field.
- Summary at the top of the form.
- Focus moves to the first invalid field on submit.

### 17.5 Partial failure

- Explain which resources succeeded and which failed.
- Link to the work request.
- Do not hide successful resources.

### 17.6 Proxy failure

When the SSR server is unreachable:

- Show a local error page.
- Explain that `make run` starts the SSR server.
- Mention `make run-lite` as an alternative.
- Do not show a blank page.

---

## 18. Labs design

### 18.1 Lab structure

- Title.
- Difficulty.
- Estimated duration.
- Goal.
- Prerequisites.
- Steps.
- Hints.
- CLI equivalents.
- Validation checks.
- Reset.

### 18.2 Difficulty levels

| Level        | Description                           |
| ------------ | ------------------------------------- |
| Beginner     | Single service, guided steps          |
| Intermediate | Multiple services, some decisions     |
| Advanced     | Full flow with validation and cleanup |

### 18.3 Validation feedback

Each check shows:

- Pass or fail.
- What was expected.
- What was found.
- A hint.

### 18.4 Lab list

- Available.
- In progress.
- Completed.
- Reset.

Completion is local and stored in browser local storage. The lab state itself is validated against the emulator.

### 18.5 Safety

Labs can create and delete resources. Before reset, explain what will be removed. Never reset the whole emulator without confirmation.

---

## 19. Design QA checklist

### 19.1 Layout

- Page header follows the standard structure.
- Primary action is unique and visible.
- Breadcrumbs match the URL.
- Sidebar state is preserved.
- Content width is within the grid.
- Tables scroll horizontally on narrow screens.

### 19.2 Content

- All strings are translated.
- No placeholder text remains.
- OCIDs use monospace and copy.
- Dates use the active locale.
- Empty states explain the next action.
- Errors include a recovery action.

### 19.3 Interaction

- Keyboard navigation works.
- Focus is visible.
- Dialogs trap focus.
- Destructive actions require confirmation.
- Loading states are present.
- Work requests are visible.
- Unsaved changes prompt appears.

### 19.4 Visual

- Colors use design tokens.
- No Oracle assets.
- Status is not color-only.
- Charts have legends and units.
- Contrast passes AA.
- Fonts are self-hosted.

### 19.5 Accessibility

- axe-core reports no critical issues.
- Form fields have labels.
- Icon buttons have names.
- Charts have table alternatives.
- Reduced motion is respected.
- Language attribute updates.

### 19.6 Performance

- Initial JS within budget.
- Charts deferred.
- Images optimized.
- No layout shift on language change.
- SSR and hydration produce no warnings.

---

## 20. Future design work

- Dark theme.
- Locale-prefixed prerendered documentation.
- Saved views for tables.
- Bulk actions for supported services.
- Guided tours for first-time users.
- Shareable lab snapshots.
- Import and export from the UI with progress.
- Policy simulator with a statement editor and evaluation results.
- Terraform compatibility explorer.
- Cost comparison reports with editable pricing.

For technical details, see [ARCHITECTURE.md](ARCHITECTURE.md). For setup and operation, see [README.md](README.md).
