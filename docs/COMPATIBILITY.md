# OCI Compatibility Matrix

This document defines what "supported" means for OCI Emulator. It is the contract between the backend, the web console, the CLI compatibility suite, and the documentation.

For protocol rules, see [PROTOCOL.md](PROTOCOL.md). For resource relationships and state machines, see [DATA-MODEL.md](DATA-MODEL.md).

---

## 1. Principles

1. The official OCI CLI is the primary compatibility target.
2. OCI SDKs are supported when they can override the service endpoint.
3. The web console uses the same OCI endpoints as the CLI.
4. Emulator-only features live under `/_emu`.
5. Unsupported behavior must return a clear OCI-shaped error, not a silent success.
6. A row is only marked `verified` when an automated CLI test covers it.

---

## 2. Status legend

| Status        | Meaning                                                         |
| ------------- | --------------------------------------------------------------- |
| `verified`    | Covered by an automated CLI compatibility test and passing      |
| `partial`     | Implemented but missing fields, lifecycle states, or edge cases |
| `planned`     | In scope and scheduled, not implemented                         |
| `not planned` | Explicitly out of scope for version 1                           |

---

## 3. How to update this document

- Add a row before implementing a new operation.
- Link the row to the task that implements it.
- Mark the row `partial` when the first implementation lands.
- Mark the row `verified` only after the CLI compatibility test passes.
- Add a note for every known deviation from OCI.
- Do not add endpoints that are not used by the CLI, the web console, or the compatibility suite without a concrete use case.

---

## 4. Phase 0 coverage

Phase 0 proves the full stack with a small, complete slice.

### 4.1 Identity

| Operation          | CLI command                  | Method and path                      | Status    | Notes                         |
| ------------------ | ---------------------------- | ------------------------------------ | --------- | ----------------------------- |
| List regions       | `oci iam region list`        | `GET /20160918/regions`              | `planned` | Static region list            |
| List compartments  | `oci iam compartment list`   | `GET /20160918/compartments`         | `planned` | Paginated                     |
| Create compartment | `oci iam compartment create` | `POST /20160918/compartments`        | `planned` | Requires name and description |
| Get compartment    | `oci iam compartment get`    | `GET /20160918/compartments/{id}`    | `planned` |                               |
| Delete compartment | `oci iam compartment delete` | `DELETE /20160918/compartments/{id}` | `planned` | Blocked when non-empty        |

### 4.2 Object Storage

| Operation     | CLI command            | Method and path                               | Status    | Notes                       |
| ------------- | ---------------------- | --------------------------------------------- | --------- | --------------------------- |
| Get namespace | `oci os ns get`        | `GET /n`                                      | `planned` | Returns the local namespace |
| Create bucket | `oci os bucket create` | `POST /n/{namespace}/b`                       | `planned` |                             |
| List buckets  | `oci os bucket list`   | `GET /n/{namespace}/b`                        | `planned` |                             |
| Get bucket    | `oci os bucket get`    | `GET /n/{namespace}/b/{bucket}`               | `planned` |                             |
| Delete bucket | `oci os bucket delete` | `DELETE /n/{namespace}/b/{bucket}`            | `planned` | Blocked when non-empty      |
| Put object    | `oci os object put`    | `PUT /n/{namespace}/b/{bucket}/o/{object}`    | `planned` | Stores real bytes           |
| List objects  | `oci os object list`   | `GET /n/{namespace}/b/{bucket}/o`             | `planned` | Prefix and delimiter        |
| Get object    | `oci os object get`    | `GET /n/{namespace}/b/{bucket}/o/{object}`    | `planned` |                             |
| Delete object | `oci os object delete` | `DELETE /n/{namespace}/b/{bucket}/o/{object}` | `planned` |                             |

### 4.3 Core Networking

| Operation  | CLI command              | Method and path              | Status    | Notes                      |
| ---------- | ------------------------ | ---------------------------- | --------- | -------------------------- |
| Create VCN | `oci network vcn create` | `POST /20160918/vcns`        | `planned` | CIDR required              |
| List VCNs  | `oci network vcn list`   | `GET /20160918/vcns`         | `planned` | Compartment filter         |
| Get VCN    | `oci network vcn get`    | `GET /20160918/vcns/{id}`    | `planned` |                            |
| Update VCN | `oci network vcn update` | `PUT /20160918/vcns/{id}`    | `planned` | Display name and tags      |
| Delete VCN | `oci network vcn delete` | `DELETE /20160918/vcns/{id}` | `planned` | Blocked when subnets exist |

### 4.4 Work requests

| Operation          | CLI command                                     | Method and path                          | Status    | Notes |
| ------------------ | ----------------------------------------------- | ---------------------------------------- | --------- | ----- |
| List work requests | `oci work-requests work-request list`           | `GET /20160918/workRequests`             | `planned` |       |
| Get work request   | `oci work-requests work-request get`            | `GET /20160918/workRequests/{id}`        | `planned` |       |
| List errors        | `oci work-requests work-request-error list`     | `GET /20160918/workRequests/{id}/errors` | `planned` |       |
| List logs          | `oci work-requests work-request-log-entry list` | `GET /20160918/workRequests/{id}/logs`   | `planned` |       |

---

## 5. Phase 1 coverage: Core Services

### 5.1 Networking

| Operation                                                | CLI command                        | Status    | Notes                   |
| -------------------------------------------------------- | ---------------------------------- | --------- | ----------------------- |
| Subnet create, list, get, update, delete                 | `oci network subnet ...`           | `planned` | CIDR overlap validation |
| Internet gateway create, list, get, update, delete       | `oci network internet-gateway ...` | `planned` |                         |
| NAT gateway create, list, get, update, delete            | `oci network nat-gateway ...`      | `planned` |                         |
| Service gateway create, list, get, update, delete        | `oci network service-gateway ...`  | `planned` |                         |
| Route table create, list, get, update, delete            | `oci network route-table ...`      | `planned` | Rule editor             |
| Security list create, list, get, update, delete          | `oci network security-list ...`    | `planned` | Ingress and egress      |
| Network security group create, list, get, update, delete | `oci network nsg ...`              | `planned` | Rules                   |
| DHCP options create, list, get, update, delete           | `oci network dhcp-options ...`     | `planned` |                         |
| Private IP list, get, update                             | `oci network private-ip ...`       | `planned` |                         |
| Public IP list, get, create, update, delete              | `oci network public-ip ...`        | `planned` |                         |

### 5.2 Compute

| Operation                                     | CLI command                       | Status    | Notes                                |
| --------------------------------------------- | --------------------------------- | --------- | ------------------------------------ |
| Shape list                                    | `oci compute shape list`          | `planned` | Static catalog                       |
| Image list and get                            | `oci compute image list`, `get`   | `planned` | Seeded images                        |
| Instance launch, list, get, update, terminate | `oci compute instance ...`        | `planned` | Work request on launch and terminate |
| Instance actions                              | `oci compute instance action`     | `planned` | START, STOP, REBOOT, RESET           |
| VNIC list and get                             | `oci compute vnic ...`            | `planned` |                                      |
| Console history capture and get               | `oci compute console-history ...` | `planned` | Simulated output                     |

### 5.3 Block Storage

| Operation                                   | CLI command                          | Status    | Notes |
| ------------------------------------------- | ------------------------------------ | --------- | ----- |
| Volume create, list, get, update, delete    | `oci bv volume ...`                  | `planned` |       |
| Volume attachment create, list, get, delete | `oci compute volume-attachment ...`  | `planned` |       |
| Volume backup create, list, get, delete     | `oci bv backup ...`                  | `planned` |       |
| Volume resize                               | `oci bv volume update --size-in-gbs` | `planned` |       |

---

## 6. Phase 2 coverage: IAM and Object Storage UI

The backend for these services exists in floci-oci. Phase 2 completes the web console and the missing operations.

| Area           | Operations                                     | Status    | Notes                 |
| -------------- | ---------------------------------------------- | --------- | --------------------- |
| Compartments   | Tree, create, edit, delete                     | `planned` | UI and API            |
| Users          | List, create, get, update, delete              | `planned` | UI and API            |
| Groups         | List, create, get, update, delete, memberships | `planned` | UI and API            |
| Policies       | List, create, get, update, delete              | `planned` | Stored, not evaluated |
| Dynamic groups | List, create, get, update, delete              | `planned` | UI and API            |
| Buckets        | Create, list, get, update, delete              | `planned` | UI                    |
| Objects        | Upload, download, list, delete, metadata       | `planned` | UI                    |
| Multipart      | Create, upload part, complete, abort           | `planned` | UI                    |
| PARs           | Create, list, get, delete                      | `planned` | UI                    |
| Lifecycle      | Create, list, get, update, delete              | `planned` | UI                    |

---

## 7. Phase 3 coverage: Load Balancer, API Gateway, DNS

| Service       | Operations                        | Status    | Notes                                |
| ------------- | --------------------------------- | --------- | ------------------------------------ |
| Load Balancer | Create, list, get, update, delete | `planned` |                                      |
| Listeners     | Create, list, get, update, delete | `planned` |                                      |
| Backend sets  | Create, list, get, update, delete | `planned` |                                      |
| Backends      | Create, list, get, update, delete | `planned` | Health check simulated               |
| API Gateway   | Create, list, get, update, delete | `planned` |                                      |
| Deployments   | Create, list, get, update, delete | `planned` |                                      |
| Routes        | Create, list, get, update, delete | `planned` |                                      |
| DNS zones     | Create, list, get, update, delete | `planned` |                                      |
| DNS records   | Create, list, get, update, delete | `planned` | Bulk import; `AAAA` is metadata only |

---

## 8. Phase 4 coverage: Databases

| Service                  | Operations                        | Status    | Notes                      |
| ------------------------ | --------------------------------- | --------- | -------------------------- |
| Autonomous Database      | Create, list, get, update, delete | `planned` |                            |
| ADB lifecycle            | Start, stop, restart              | `planned` |                            |
| ADB connection strings   | Get                               | `planned` | Simulated                  |
| ADB wallet               | Generate and download             | `planned` | ZIP with placeholder files |
| ADB backups              | Create, list, get, delete         | `planned` |                            |
| MySQL DB system          | Create, list, get, update, delete | `planned` |                            |
| MySQL lifecycle          | Start, stop, restart              | `planned` |                            |
| MySQL connection details | Get                               | `planned` | Simulated                  |
| MySQL backups            | Create, list, get, delete         | `planned` |                            |

---

## 9. Phase 5 and 6 coverage

Phase 5 adds emulator features, not new OCI services:

- Dashboard aggregation.
- Global search.
- Request inspector UI.
- Labs and validation.
- JSON view and CLI equivalent.
- Tags and filters.
- Complete translations.
- Accessibility and responsive polish.

Phase 6 adds operational hardening:

- Export and import.
- Reset and seed hardening.
- CI and verification gates.
- SSR and performance regression tests.
- Release packaging.
- Final documentation.

---

## 10. Known deviations

| Area           | Deviation                                  | Impact                                  |
| -------------- | ------------------------------------------ | --------------------------------------- |
| Authentication | OCI signatures are parsed but not verified | Any local key works                     |
| IAM policies   | Stored and displayed, not evaluated        | Policy changes do not affect access     |
| Compute        | No real VM runtime                         | No SSH or console output from a real OS |
| Networking     | No packet forwarding                       | No connectivity tests                   |
| DNS            | No resolver                                | Records are metadata only               |
| Load Balancer  | No traffic plane                           | Health checks are simulated             |
| ADB and MySQL  | No database engine                         | Connection strings are placeholders     |
| Object Storage | Local filesystem storage                   | No replication or durability            |
| Terraform      | Not tested                                 | May work partially with host overrides  |
| IPv6           | Not supported                              | IPv4 only                               |

---

## 11. Verification commands

Phase 0 smoke test:

```bash
bin/ocilocal iam region list
bin/ocilocal iam compartment list --compartment-id ocid1.tenancy.oc1..emulator
bin/ocilocal os ns get
bin/ocilocal network vcn create \
  --compartment-id ocid1.compartment.oc1..demo \
  --cidr-block 10.0.0.0/16 \
  --display-name smoke-vcn
bin/ocilocal network vcn list --compartment-id ocid1.compartment.oc1..demo
```

A row is not complete until its command runs against a fresh emulator and the response is checked by the compatibility suite.

CLI setup and command examples are in [CLI.md](CLI.md). Console flows are in [CONSOLE.md](CONSOLE.md).

---

## 12. Out of scope for version 1

- Terraform compatibility.
- IAM policy evaluation.
- Real compute, network, DNS, load balancer, or database data planes.
- IPv6.
- TLS.
- Multi-tenant isolation.
- Queue, Streaming, KMS, Vault, Organizations, Governance Rules, Functions, OKE, and PostgreSQL.
- Endpoints that are not used by the CLI, the web console, or the compatibility suite.
