# OCI CLI Guide

How to point the official OCI CLI at the local emulator.

For the wire protocol and error envelope, see [PROTOCOL.md](PROTOCOL.md). For service coverage, see [COMPATIBILITY.md](COMPATIBILITY.md). For setup, see [README.md](../README.md).

---

## 1. What `make setup` creates

```text
.venv/                 Python 3.12 virtual environment
.oci/config            OCI CLI profile for the emulator
.oci/key.pem           Local RSA private key
.oci/key_public.pem    Local RSA public key
bin/ocilocal           Wrapper that points the CLI at the emulator
```

The `.oci` directory is ignored by Git. It contains local development credentials, not Oracle Cloud credentials.

---

## 2. Profile

`.oci/config` contains:

```ini
[OCIEMU]
user=ocid1.user.oc1..emulator
fingerprint=<generated-md5>
tenancy=ocid1.tenancy.oc1..emulator
region=us-ashburn-1
key_file=./.oci/key.pem
```

The tenancy, user, and fingerprint are local identifiers. They are not valid Oracle Cloud credentials. The same user OCID is used by the web console session.

---

## 3. Wrapper

`bin/ocilocal` sets the profile, config file, and endpoint:

```sh
#!/bin/sh
export OCI_CLI_CONFIG_FILE="$PWD/.oci/config"
export OCI_CLI_PROFILE=OCIEMU
export OCI_CLI_ENDPOINT="${OCI_CLI_ENDPOINT:-http://localhost:4599}"
exec .venv/bin/oci "$@"
```

Use it exactly like the official CLI:

```bash
bin/ocilocal <service> <command> [options]
```

---

## 4. Direct CLI usage

You can also use the installed CLI directly:

```bash
OCI_CLI_CONFIG_FILE="$PWD/.oci/config" \
OCI_CLI_PROFILE=OCIEMU \
OCI_CLI_ENDPOINT=http://localhost:4599 \
.venv/bin/oci iam region list
```

Or add the profile to your global `~/.oci/config` and use:

```bash
oci --profile OCIEMU --endpoint http://localhost:4599 iam region list
```

---

## 5. Common commands

### Identity

```bash
bin/ocilocal iam region list
bin/ocilocal iam compartment list --compartment-id ocid1.tenancy.oc1..emulator
bin/ocilocal iam user list --compartment-id ocid1.tenancy.oc1..emulator
bin/ocilocal iam group list --compartment-id ocid1.tenancy.oc1..emulator
bin/ocilocal iam policy list --compartment-id ocid1.tenancy.oc1..emulator
```

### Object Storage

```bash
bin/ocilocal os ns get
bin/ocilocal os bucket create \
  --compartment-id ocid1.compartment.oc1..demo \
  --name demo-bucket

bin/ocilocal os object put \
  --bucket-name demo-bucket \
  --file README.md \
  --name README.md

bin/ocilocal os object list --bucket-name demo-bucket
bin/ocilocal os object get --bucket-name demo-bucket --name README.md --file downloaded.md
```

### Networking

```bash
bin/ocilocal network vcn create \
  --compartment-id ocid1.compartment.oc1..demo \
  --cidr-block 10.0.0.0/16 \
  --display-name demo-vcn

bin/ocilocal network vcn list --compartment-id ocid1.compartment.oc1..demo
bin/ocilocal network subnet create \
  --compartment-id ocid1.compartment.oc1..demo \
  --vcn-id <vcn-ocid> \
  --cidr-block 10.0.1.0/24 \
  --display-name public-subnet
```

### Compute

```bash
bin/ocilocal compute shape list --compartment-id ocid1.compartment.oc1..demo
bin/ocilocal compute image list --compartment-id ocid1.compartment.oc1..demo
bin/ocilocal compute instance launch \
  --compartment-id ocid1.compartment.oc1..demo \
  --availability-domain AD-1 \
  --shape VM.Standard.E4.Flex \
  --subnet-id <subnet-ocid> \
  --image-id <image-ocid> \
  --display-name demo-instance

bin/ocilocal compute instance list --compartment-id ocid1.compartment.oc1..demo
```

### Block Volume

```bash
bin/ocilocal bv volume create \
  --compartment-id ocid1.compartment.oc1..demo \
  --availability-domain AD-1 \
  --display-name demo-volume \
  --size-in-gbs 50

bin/ocilocal bv volume list --compartment-id ocid1.compartment.oc1..demo
```

### Load Balancer

```bash
bin/ocilocal lb load-balancer list --compartment-id ocid1.compartment.oc1..demo
```

Creating a load balancer with listeners and backends is easier from the web console wizard. The backend supports the same operations through the API.

### DNS

```bash
bin/ocilocal dns zone create \
  --compartment-id ocid1.compartment.oc1..demo \
  --name example.local \
  --zone-type PRIMARY

bin/ocilocal dns zone list --compartment-id ocid1.compartment.oc1..demo
```

### Autonomous Database

```bash
bin/ocilocal db autonomous-database create \
  --compartment-id ocid1.compartment.oc1..demo \
  --db-name DEMOADB \
  --display-name demo-adb \
  --cpu-core-count 1 \
  --data-storage-size-in-tbs 1 \
  --admin-password 'Emulator#12345'

bin/ocilocal db autonomous-database list --compartment-id ocid1.compartment.oc1..demo
```

Password rules are defined in [DATA-MODEL.md](DATA-MODEL.md) section 12.

### MySQL

```bash
bin/ocilocal mysql db-system list --compartment-id ocid1.compartment.oc1..demo
```

Creating a MySQL DB system requires a shape, subnet, and storage size. Use the web console wizard or pass the required parameters explicitly.

### Work requests

```bash
bin/ocilocal work-requests work-request list --compartment-id ocid1.compartment.oc1..demo
bin/ocilocal work-requests work-request get --work-request-id <work-request-ocid>
```

---

## 6. Wait for state

Asynchronous resources support the CLI wait pattern:

```bash
bin/ocilocal compute instance launch ... --wait-for-state RUNNING
bin/ocilocal db autonomous-database create ... --wait-for-state AVAILABLE
```

The emulator updates lifecycle states in the background so the wait command can observe transitions.

---

## 7. Troubleshooting

| Error                     | Cause                                  | Fix                                                                  |
| ------------------------- | -------------------------------------- | -------------------------------------------------------------------- |
| `ConfigFileNotFound`      | Wrong config path                      | Run commands through `bin/ocilocal` or set `OCI_CLI_CONFIG_FILE`     |
| `ProfileNotFound`         | Wrong profile name                     | Set `OCI_CLI_PROFILE=OCIEMU`                                         |
| `InvalidPrivateKey`       | Key file is missing or unreadable      | Run `make setup` again                                               |
| `Could not connect`       | Emulator is not running                | Start `make dev` or `make run`                                       |
| `NotAuthorizedOrNotFound` | Resource or compartment does not exist | Check the OCID and compartment                                       |
| `InvalidParameter`        | Missing or invalid argument            | Read the command help with `bin/ocilocal <service> <command> --help` |

The emulator parses the OCI signature but does not verify it. Signature errors usually mean the CLI is not using the local profile or endpoint.

For more startup and environment issues, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md).
