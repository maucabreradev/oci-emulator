# OCI Wire Protocol Notes

This document defines the shared wire-level rules used by every emulated service. It is the reference for controllers, error mappers, the CLI compatibility suite, and the request inspector.

For service coverage, see [COMPATIBILITY.md](COMPATIBILITY.md). For resource models, see [DATA-MODEL.md](DATA-MODEL.md).

---

## 1. Goals

- Preserve the OCI request and response shapes.
- Keep behavior consistent across services.
- Avoid custom endpoint shapes.
- Make the official OCI CLI work without special cases.
- Make deviations explicit and testable.

---

## 2. Base URL and version prefixes

The emulator listens on `http://127.0.0.1:4599` by default.

| Service                      | Prefix                                 |
| ---------------------------- | -------------------------------------- |
| Core and Autonomous Database | `/20160918`                            |
| Load Balancer                | `/20200501`                            |
| API Gateway                  | `/20190501`                            |
| DNS                          | `/20180115`                            |
| MySQL                        | `/20190415`                            |
| Object Storage               | `/n/{namespace}/b/{bucket}/o/{object}` |
| Emulator                     | `/_emu`                                |

The CLI and SDKs append the service prefix when the endpoint is set to the base URL. Do not include the version prefix in `OCI_CLI_ENDPOINT`.

Examples:

```bash
OCI_CLI_ENDPOINT=http://localhost:4599 oci network vcn list
OCI_CLI_ENDPOINT=http://localhost:4599 oci os ns get
```

---

## 3. Request lifecycle

```text
HTTP request
  |
  v
Request ID filter
  |
  v
Signature parser
  |
  v
Auth context
  |
  v
JAX-RS controller
  |
  v
Service
  |
  v
Storage
  |
  v
Response mapper
  |
  v
HTTP response
```

Each step has one responsibility:

- The request ID filter assigns and returns `opc-request-id`.
- The signature parser extracts tenancy, user, fingerprint, and region.
- The auth context scopes storage access.
- The controller parses and validates the HTTP shape.
- The service applies business rules.
- The response mapper normalizes headers and errors.

---

## 4. Authentication

### 4.1 Signed requests

The CLI sends an OCI `Authorization` header with this shape:

```text
Signature version="1",keyId="<tenancy>/<user>/<fingerprint>",algorithm="rsa-sha256",headers="date (request-target) host content-length content-type x-content-sha256",signature="<base64>"
```

The emulator parses:

- `version`
- `keyId`
- `algorithm`
- `headers`
- `signature`
- `date`
- `host`
- `(request-target)`

The tenancy, user, and fingerprint are extracted from `keyId`.

The signature is not verified. The key material and signature bytes are ignored after parsing.

### 4.2 Unsigned requests

Unsigned requests are accepted. This is required for:

- The web console.
- Local scripts.
- The request inspector replay feature.

Unsigned requests use the default local tenancy and user.

### 4.3 Region

The region is resolved in this order:

1. `x-emu-region` header, for development and tests.
2. Region from the signature credential scope, when present.
3. Default region from configuration.

The web console sends `x-emu-region` on emulator and OCI requests.

### 4.4 Security note

Authentication is not a security boundary. The emulator must not be exposed to a network. See [SECURITY.md](../SECURITY.md).

---

## 5. Common request headers

| Header           | Required    | Meaning                                         |
| ---------------- | ----------- | ----------------------------------------------- |
| `Authorization`  | No          | OCI signature, parsed but not verified          |
| `Content-Type`   | For bodies  | `application/json` or binary for Object Storage |
| `Accept`         | No          | Usually `application/json`                      |
| `opc-request-id` | No          | Client-generated request ID                     |
| `if-match`       | Conditional | ETag for optimistic concurrency                 |
| `if-none-match`  | Conditional | Conditional read                                |
| `x-emu-region`   | No          | Local region override                           |

The emulator accepts the official OCI headers and ignores unknown `opc-*` headers unless a service needs them.

---

## 6. Common response headers

| Header                | Meaning                                           |
| --------------------- | ------------------------------------------------- |
| `opc-request-id`      | Unique request ID                                 |
| `opc-next-page`       | Token for the next page                           |
| `opc-work-request-id` | Asynchronous operation ID                         |
| `etag`                | Resource version                                  |
| `location`            | Created resource URL                              |
| `retry-after`         | Suggested retry delay for asynchronous operations |

`opc-request-id` must be present on every response, including errors.

---

## 7. Pagination

### 7.1 Request parameters

| Parameter       | Meaning                |
| --------------- | ---------------------- |
| `limit`         | Maximum items per page |
| `page`          | Opaque page token      |
| `sortBy`        | Field to sort by       |
| `sortOrder`     | `ASC` or `DESC`        |
| `compartmentId` | Compartment filter     |

### 7.2 Response

```json
{
  "data": [],
  "opc-next-page": "eyJvZmZzZXQiOjIwfQ=="
}
```

The token is opaque. The frontend and CLI must pass it back unchanged.

### 7.3 Rules

- `limit` defaults to 10 and is capped at 100.
- An invalid `page` token returns `InvalidParameter`.
- Sorting is stable.
- Filters are applied before pagination.
- `opc-next-page` is omitted on the last page.

---

## 8. ETags and concurrency

- Every mutable resource has an `etag`.
- `GET` responses include the current ETag.
- `PUT` and `DELETE` accept `if-match`.
- A missing `if-match` is accepted for local convenience.
- A mismatched `if-match` returns `412 PreconditionFailed`.
- ETags are stable across reads until the resource changes.

ETags are not cryptographic. They are local version markers.

---

## 9. Work requests

### 9.1 Creation

Long-running operations return:

```http
HTTP/1.1 202 Accepted
opc-work-request-id: ocid1.workrequest.oc1..example
location: http://localhost:4599/20160918/workRequests/ocid1.workrequest.oc1..example
```

### 9.2 Status

Allowed values:

- `ACCEPTED`
- `IN_PROGRESS`
- `SUCCEEDED`
- `FAILED`
- `CANCELING`
- `CANCELED`

### 9.3 Fields

```json
{
  "id": "ocid1.workrequest.oc1..example",
  "compartmentId": "ocid1.compartment.oc1..demo",
  "operationType": "CREATE_VCN",
  "status": "SUCCEEDED",
  "percentComplete": 100,
  "timeAccepted": "2026-11-16T18:00:00Z",
  "timeStarted": "2026-11-16T18:00:01Z",
  "timeFinished": "2026-11-16T18:00:02Z",
  "resources": []
}
```

### 9.4 Rules

- The work request is created before the resource is visible.
- Terminal states are immutable.
- `percentComplete` is monotonic.
- Errors are stored in the work request error list.
- The web console polls every two seconds until terminal.

---

## 10. Errors

### 10.1 Envelope

```json
{
  "code": "InvalidParameter",
  "message": "cidrBlock is required.",
  "status": 400,
  "opc-request-id": "20261116T180000Z-abc123"
}
```

### 10.2 Codes

| Code                      | HTTP status | When                            |
| ------------------------- | ----------- | ------------------------------- |
| `InvalidParameter`        | 400         | Missing or invalid field        |
| `NotAuthorizedOrNotFound` | 404         | Resource missing or not visible |
| `Conflict`                | 409         | Invalid state or duplicate      |
| `PreconditionFailed`      | 412         | ETag mismatch                   |
| `LimitExceeded`           | 429         | Simulated limit                 |
| `InternalError`           | 500         | Unexpected failure              |

### 10.3 Rules

- Do not leak stack traces.
- Include a stable request ID.
- Keep messages actionable.
- Use the same code for the same condition across services.
- Unknown routes return `NotAuthorizedOrNotFound`, not an HTML 404.

### 10.4 Validation messages

- Validation failures use `InvalidParameter`.
- The message names the field and the failed rule, for example: `adminPassword must contain at least one digit.`
- Do not echo a submitted password, secret, or key material in the message or logs.
- Database credential rules are defined in [DATA-MODEL.md](DATA-MODEL.md).

---

## 11. Content types

### 11.1 JSON

Requests and responses use `application/json`.

Date and time values use ISO 8601 UTC with a `Z` suffix.

OCIDs are strings.

### 11.2 Object Storage

- Object uploads use `application/octet-stream` or the provided content type.
- Object downloads return the stored content type.
- Metadata uses `opc-meta-*` headers.
- Multipart uses the OCI multipart endpoints and part numbers.
- ETags are quoted strings.

### 11.3 Errors on binary endpoints

Object Storage errors still return JSON with the OCI error envelope and an appropriate status code.

---

## 12. OCIDs

### 12.1 Shape

```text
ocid1.<resource-type>.<realm>..<unique-id>
```

Examples:

```text
ocid1.tenancy.oc1..emulator
ocid1.user.oc1..emulator
ocid1.compartment.oc1..demo
ocid1.vcn.oc1..demo
ocid1.instance.oc1..demo
```

### 12.2 Rules

- OCIDs are opaque to clients.
- The emulator accepts any OCID with the expected resource type.
- Generated OCIDs are stable for the lifetime of the resource.
- OCIDs are never translated or shortened in storage.
- The UI may truncate them for display but always copies the full value.

---

## 13. Regions and availability domains

- Region IDs use real OCI region names, such as `us-ashburn-1`.
- The region is a label, not a network location.
- Availability domains use `AD-1`, `AD-2`, and `AD-3`.
- Resources are scoped by tenancy and region.
- The web console persists the selected region in a cookie.

---

## 14. Compartments

- Every resource except tenancy-level identity resources belongs to a compartment.
- The root compartment is the tenancy OCID.
- Deleting a compartment is blocked when any service reports live resources.
- `NotAuthorizedOrNotFound` is returned when a compartment does not exist.
- The UI shows the compartment tree and blocks deletion with an explanation.

---

## 15. Object Storage paths

### 15.1 Namespace

`GET /n` returns the local namespace.

### 15.2 Buckets

```text
GET    /n/{namespace}/b
POST   /n/{namespace}/b
GET    /n/{namespace}/b/{bucket}
PUT    /n/{namespace}/b/{bucket}
DELETE /n/{namespace}/b/{bucket}
```

### 15.3 Objects

```text
GET    /n/{namespace}/b/{bucket}/o
PUT    /n/{namespace}/b/{bucket}/o/{object}
GET    /n/{namespace}/b/{bucket}/o/{object}
DELETE /n/{namespace}/b/{bucket}/o/{object}
HEAD   /n/{namespace}/b/{bucket}/o/{object}
```

### 15.4 Multipart

Multipart upload follows the OCI Object Storage API. The emulator stores parts and assembles the object on completion. Aborted uploads are cleaned up.

---

## 16. Emulator endpoints

Emulator endpoints are not part of the OCI protocol.

| Endpoint          | Purpose                    |
| ----------------- | -------------------------- |
| `/_emu/session/*` | Simulated login            |
| `/_emu/config`    | Runtime configuration      |
| `/_emu/dashboard` | Aggregated dashboard data  |
| `/_emu/requests`  | Request inspector          |
| `/_emu/search`    | Global search              |
| `/_emu/labs`      | Lab catalog and validation |
| `/_emu/reset`     | Reset state                |
| `/_emu/seed`      | Seed demo data             |
| `/_emu/export`    | Export state               |
| `/_emu/import`    | Import state               |

These endpoints return JSON and are not versioned.

---

## 17. CORS

The default topology serves the UI and API from the same origin, so CORS is not required.

The backend keeps a permissive local CORS filter for development and SDK experiments. It must not be enabled on a publicly exposed instance.

---

## 18. Request inspector contract

The inspector records:

- Request ID.
- Method.
- Path and query.
- Service and operation.
- Status.
- Duration.
- Request headers and body.
- Response headers and body.
- Error code.

It excludes:

- `/_emu/requests`.
- Static assets.
- Health checks.

Bodies larger than 64 KB are truncated. Binary bodies are represented by content type and size.

---

## 19. Protocol tests

Every service must have tests for:

- Successful create, list, get, update, and delete.
- Invalid input.
- Missing resource.
- Pagination.
- ETag mismatch where applicable.
- Work request creation and terminal state.
- `opc-request-id` presence.
- Error envelope shape.

The CLI compatibility suite is the final gate.
