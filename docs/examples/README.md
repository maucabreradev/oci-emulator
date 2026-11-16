# API Examples

This directory contains request and response examples for the supported OCI endpoints. They are used for:

- Backend protocol tests.
- Frontend mocks.
- Documentation.
- Comparing emulator responses with the official OCI CLI.

## Files

| File                            | Operation                |
| ------------------------------- | ------------------------ |
| `vcn-create.request.json`       | Create a VCN             |
| `vcn-create.response.json`      | VCN create response      |
| `instance-launch.request.json`  | Launch an instance       |
| `instance-launch.response.json` | Instance launch response |
| `bucket-create.request.json`    | Create a bucket          |
| `bucket-create.response.json`   | Bucket create response   |

## Conventions

- All values are local, fictional, or generated. Do not use real OCI credentials or resource identifiers.
- OCIDs follow the shape `ocid1.<type>.<realm>..<unique>`.
- Timestamps are ISO 8601 UTC.
- Responses omit fields that the emulator does not implement.
- Headers are documented separately; the JSON files contain bodies only.

## Example with curl

Create a VCN:

```bash
curl -sS -X POST http://localhost:4599/20160918/vcns \
  -H 'Content-Type: application/json' \
  -H 'x-emu-region: us-ashburn-1' \
  -d @docs/examples/vcn-create.request.json
```

List VCNs:

```bash
curl -sS 'http://localhost:4599/20160918/vcns?compartmentId=ocid1.compartment.oc1..demo'
```

Launch an instance:

```bash
curl -sS -X POST http://localhost:4599/20160918/instances \
  -H 'Content-Type: application/json' \
  -d @docs/examples/instance-launch.request.json
```

Create a bucket:

```bash
curl -sS -X POST http://localhost:4599/n/ociemu/b \
  -H 'Content-Type: application/json' \
  -d @docs/examples/bucket-create.request.json
```

The same operations with the official CLI:

```bash
bin/ocilocal network vcn create \
  --compartment-id ocid1.compartment.oc1..demo \
  --cidr-block 10.0.0.0/16 \
  --display-name demo-vcn

bin/ocilocal compute instance launch \
  --compartment-id ocid1.compartment.oc1..demo \
  --availability-domain AD-1 \
  --shape VM.Standard.E4.Flex \
  --subnet-id ocid1.subnet.oc1..demo \
  --image-id ocid1.image.oc1..demo \
  --display-name demo-instance

bin/ocilocal os bucket create \
  --compartment-id ocid1.compartment.oc1..demo \
  --name demo-bucket
```

## Common response headers

```text
opc-request-id: 20261116T180000Z-example
location: http://localhost:4599/20160918/vcns/ocid1.vcn.oc1..demo
etag: "2026-11-16T18:00:00Z"
```

Asynchronous operations also return:

```text
opc-work-request-id: ocid1.workrequest.oc1..demo
```

## Updating examples

When a request or response shape changes:

1. Update the JSON file.
2. Update the matching protocol test.
3. Update the corresponding row in [../COMPATIBILITY.md](../COMPATIBILITY.md).
4. Update [../DATA-MODEL.md](../DATA-MODEL.md) if a field was added or removed.
5. Verify the JSON is valid:

```bash
for file in docs/examples/*.json; do
  jq empty "$file"
done
```
