# Labs Guide

This document defines the lab format, validation model, and the initial lab catalog for OCI Emulator.

For the UI behavior, see [DESIGN.md](../DESIGN.md). For resource states and constraints, see [DATA-MODEL.md](DATA-MODEL.md).

---

## 1. Purpose

Labs turn the emulator into a guided learning tool. A lab defines a goal, the steps to reach it, hints, the equivalent CLI commands, and a validator that checks the real emulator state.

Labs are not tutorials with screenshots. They are executable checklists against the local API.

---

## 2. Lab lifecycle

```text
Catalog
  |
  v
Lab detail (goal, prerequisites, steps)
  |
  v
User creates resources in the console or CLI
  |
  v
Validate
  |
  +-- All checks pass  -> Completed
  |
  +-- Some checks fail -> Show expected vs found, keep state
  |
  v
Reset (optional) -> Remove lab resources
```

Completion is stored in browser local storage. The resources themselves live in the emulator storage and are validated on demand.

---

## 3. Lab definition format

Labs are JSON files under `src/main/resources/labs`. Each file describes one lab.

```json
{
  "id": "vcn-public-instance",
  "version": 1,
  "title": "Build a VCN and launch an instance",
  "description": "Create a VCN with a public subnet, then launch an instance into it.",
  "difficulty": "beginner",
  "estimatedMinutes": 20,
  "tags": ["networking", "compute"],
  "prerequisites": [
    {
      "description": "A compartment named demo exists",
      "check": {
        "type": "compartmentExists",
        "name": "demo"
      }
    }
  ],
  "steps": [
    {
      "id": "create-vcn",
      "title": "Create the VCN",
      "instructions": "Create a VCN named lab-vcn with CIDR 10.0.0.0/16.",
      "hint": "Networking > VCNs > Create VCN.",
      "cli": "bin/ocilocal network vcn create --compartment-id $COMPARTMENT_ID --cidr-block 10.0.0.0/16 --display-name lab-vcn"
    }
  ],
  "checks": [
    {
      "id": "vcn-exists",
      "description": "A VCN named lab-vcn exists with CIDR 10.0.0.0/16",
      "type": "vcn",
      "match": {
        "displayName": "lab-vcn",
        "cidrBlock": "10.0.0.0/16",
        "lifecycleState": "AVAILABLE"
      }
    }
  ],
  "reset": {
    "resources": ["instances", "subnets", "vcns"],
    "scope": "lab"
  }
}
```

### 3.1 Required fields

| Field              | Type    | Rules                                   |
| ------------------ | ------- | --------------------------------------- |
| `id`               | string  | Lowercase, hyphenated, unique           |
| `version`          | integer | Increment on breaking changes           |
| `title`            | string  | 1 to 80 characters                      |
| `description`      | string  | 1 to 200 characters                     |
| `difficulty`       | string  | `beginner`, `intermediate`, `advanced`  |
| `estimatedMinutes` | integer | Positive                                |
| `steps`            | array   | At least one step                       |
| `checks`           | array   | At least one check                      |
| `reset`            | object  | Required for labs that create resources |

### 3.2 Step fields

| Field          | Type   | Rules                                      |
| -------------- | ------ | ------------------------------------------ |
| `id`           | string | Unique in the lab                          |
| `title`        | string | Short imperative sentence                  |
| `instructions` | string | What to do, not how to click every control |
| `hint`         | string | Optional progressive hint                  |
| `cli`          | string | Optional equivalent command                |

### 3.3 Check fields

| Field         | Type   | Rules                 |
| ------------- | ------ | --------------------- |
| `id`          | string | Unique in the lab     |
| `description` | string | What is expected      |
| `type`        | string | Resource family       |
| `match`       | object | Field values to match |

Supported check types:

- `compartment`
- `vcn`
- `subnet`
- `instance`
- `volume`
- `bucket`
- `object`
- `par`
- `policy`
- `loadBalancer`
- `dnsZone`
- `autonomousDatabase`
- `mysqlDbSystem`
- `workRequest`

The validator reads the storage state. It does not inspect request logs.

---

## 4. Validation model

### 4.1 Rule

A check passes when at least one resource of the requested type matches every field in `match`.

- String comparisons are case-sensitive except for names, which are compared exactly as stored.
- `lifecycleState` is compared exactly.
- Arrays match when the expected value is contained.
- Nested fields use dot notation in the `match` keys.
- Unknown fields fail the check and log a warning in development.

### 4.2 Result shape

```json
{
  "labId": "vcn-public-instance",
  "validatedAt": "2026-11-16T18:00:00Z",
  "completed": false,
  "checks": [
    {
      "id": "vcn-exists",
      "passed": true,
      "expected": "VCN lab-vcn with CIDR 10.0.0.0/16",
      "found": "ocid1.vcn.oc1..lab"
    },
    {
      "id": "instance-running",
      "passed": false,
      "expected": "Instance lab-instance in RUNNING state",
      "found": "Instance is PROVISIONING"
    }
  ]
}
```

### 4.3 Feedback rules

- A failed check explains what was expected and what was found.
- A check never fails because a resource is in a transient state if the state is expected to change. The user is told to wait and retry.
- Validation is read-only.
- Validation does not create resources.

---

## 5. Reset model

Reset removes only the resources created by the lab, identified by a lab tag or a naming convention.

```json
{
  "freeformTags": {
    "oci-emulator-lab": "vcn-public-instance"
  }
}
```

Rules:

- Every resource created by a lab carries the lab tag.
- Reset deletes tagged resources in dependency order.
- Reset requires confirmation in the UI.
- Reset never removes resources without the lab tag.
- If a resource cannot be deleted because of a dependency, reset reports it and stops.

---

## 6. Initial catalog

### 6.1 Lab 1: Build a VCN with a public subnet and launch an instance

| Field      | Value                 |
| ---------- | --------------------- |
| ID         | `vcn-public-instance` |
| Difficulty | Beginner              |
| Duration   | 20 minutes            |
| Tags       | networking, compute   |

**Goal**: create a VCN, a public subnet with an internet gateway, and an instance.

**Prerequisites**:

- A compartment named `demo`.
- Seed data loaded or an empty emulator.

**Steps**:

1. Create a VCN `lab-vcn` with CIDR `10.0.0.0/16`.
2. Create an internet gateway `lab-igw` in `lab-vcn`.
3. Create a route table rule to `0.0.0.0/0` through `lab-igw`.
4. Create a public subnet `lab-public-subnet` with CIDR `10.0.1.0/24`.
5. Launch an instance `lab-instance` in `lab-public-subnet`.
6. Wait until the instance is `RUNNING`.

**Checks**:

- VCN exists and is `AVAILABLE`.
- Internet gateway exists and is enabled.
- Route table contains the default route.
- Subnet exists with the expected CIDR and is public.
- Instance exists and is `RUNNING`.
- Instance has a VNIC in the subnet.

**CLI examples**:

```bash
bin/ocilocal network vcn create \
  --compartment-id "$COMPARTMENT_ID" \
  --cidr-block 10.0.0.0/16 \
  --display-name lab-vcn

bin/ocilocal network internet-gateway create \
  --compartment-id "$COMPARTMENT_ID" \
  --vcn-id "$VCN_ID" \
  --is-enabled true \
  --display-name lab-igw

bin/ocilocal network subnet create \
  --compartment-id "$COMPARTMENT_ID" \
  --vcn-id "$VCN_ID" \
  --cidr-block 10.0.1.0/24 \
  --display-name lab-public-subnet

bin/ocilocal compute instance launch \
  --compartment-id "$COMPARTMENT_ID" \
  --availability-domain AD-1 \
  --shape VM.Standard.E4.Flex \
  --subnet-id "$SUBNET_ID" \
  --image-id "$IMAGE_ID" \
  --display-name lab-instance \
  --wait-for-state RUNNING
```

### 6.2 Lab 2: Bucket, object, and pre-authenticated request

| Field      | Value          |
| ---------- | -------------- |
| ID         | `bucket-par`   |
| Difficulty | Beginner       |
| Duration   | 15 minutes     |
| Tags       | object-storage |

**Goal**: create a bucket, upload an object, and generate a read PAR.

**Steps**:

1. Create a bucket `lab-bucket`.
2. Upload a small text file as `hello.txt`.
3. Confirm the object is listed with the correct size.
4. Create a read PAR for `hello.txt` that expires in one hour.
5. Download the object through the PAR URL.
6. Revoke the PAR.

**Checks**:

- Bucket exists and is `ACTIVE`.
- Object exists with a non-zero size.
- PAR exists and targets the object.
- PAR access type is `ObjectRead`.
- PAR state is `ACTIVE` before revocation.

**CLI examples**:

```bash
bin/ocilocal os bucket create \
  --compartment-id "$COMPARTMENT_ID" \
  --name lab-bucket

bin/ocilocal os object put \
  --bucket-name lab-bucket \
  --file hello.txt \
  --name hello.txt

bin/ocilocal os preauth-request create \
  --bucket-name lab-bucket \
  --name lab-par \
  --access-type ObjectRead \
  --time-expires "<future-date>" \
  --object-name hello.txt
```

Replace `<future-date>` with an ISO 8601 timestamp in the future, for example one hour from now.

### 6.3 Lab 3: Compartment and IAM policy

| Field      | Value                |
| ---------- | -------------------- |
| ID         | `compartment-policy` |
| Difficulty | Intermediate         |
| Duration   | 15 minutes           |
| Tags       | identity             |

**Goal**: create a compartment, a group, and a policy that references them.

**Steps**:

1. Create a compartment `lab-team`.
2. Create a group `lab-operators`.
3. Create a policy `lab-policy` in `lab-team` with a statement that allows the group to manage instances.
4. Read the policy and confirm the statement is stored.
5. Attempt to delete the compartment and observe the blocking message.

**Checks**:

- Compartment exists and is `ACTIVE`.
- Group exists.
- Policy exists with at least one statement.
- Policy statement references the group name.
- Compartment deletion is blocked by the existing resources.

**Note**: policies are not evaluated. The lab teaches the model, not authorization.

### 6.4 Lab 4: Load balancer and DNS zone

| Field      | Value              |
| ---------- | ------------------ |
| ID         | `lb-dns`           |
| Difficulty | Advanced           |
| Duration   | 25 minutes         |
| Tags       | load-balancer, dns |

**Goal**: create a load balancer with one listener and a DNS zone with a record.

**Steps**:

1. Create a load balancer in two public subnets.
2. Add a listener on port 80 with a backend set.
3. Add one backend with a private IP.
4. Wait until the load balancer is `ACTIVE`.
5. Create a DNS zone `lab.example.local`.
6. Add an `A` record that points to the load balancer private IP.
7. Confirm the record is listed.

**Checks**:

- Load balancer exists and is `ACTIVE`.
- Listener exists on port 80.
- Backend set has at least one backend.
- DNS zone exists and is `ACTIVE`.
- Record exists with type `A`.

**Note**: there is no traffic forwarding and no DNS resolution.

---

## 7. Adding a lab

1. Copy the structure from section 3.
2. Use an existing check type or add one to the validator.
3. Tag every resource the lab creates.
4. Define reset order.
5. Add the lab to the catalog.
6. Add a Playwright test that completes the lab against a fresh emulator.
7. Update this document.

---

## 8. Safety rules

- Labs must not require internet access.
- Labs must not require real credentials.
- Labs must not delete resources they did not create.
- Labs must explain any destructive step before it happens.
- Labs must state when a feature is simulated and not real.
- Lab validation must never be the only test for a backend operation.
