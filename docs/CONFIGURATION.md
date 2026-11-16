# Configuration

Runtime configuration for OCI Emulator: backend defaults, environment variables, storage, the Node SSR server, service toggles, and UI settings.

For the runtime topology, see [ARCHITECTURE.md](../ARCHITECTURE.md). For data layout and persistence rules, see [DATA-MODEL.md](DATA-MODEL.md). For user-facing setup, see [README.md](../README.md).

---

## 1. Backend defaults

The defaults live in `src/main/resources/application.yml`:

```yaml
floci-oci:
  host: 127.0.0.1
  port: 4599
  storage:
    mode: persistent
    dir: ./data
  emulator:
    ui:
      proxy:
        enabled: true
        target: http://127.0.0.1:4600
    inspector:
      enabled: true
      capacity: 500
    session:
      ttl: 8h
```

The server binds to loopback by default. Do not change `host` to a public interface unless you understand that the emulator has no authentication or TLS.

---

## 2. Environment variables

| Variable                                | Default                 | Purpose                                   |
| --------------------------------------- | ----------------------- | ----------------------------------------- |
| `FLOCI_OCI_HOST`                        | `127.0.0.1`             | Bind address                              |
| `FLOCI_OCI_PORT`                        | `4599`                  | Public API and UI port                    |
| `FLOCI_OCI_STORAGE_MODE`                | `persistent`            | Storage mode: `persistent` or `in-memory` |
| `FLOCI_OCI_STORAGE_DIR`                 | `./data`                | Persistent data directory                 |
| `FLOCI_OCI_MAX_REQUEST_SIZE`            | `64`                    | Maximum request size in MB                |
| `FLOCI_OCI_EMULATOR_UI_PROXY_ENABLED`   | `true` in prod          | Enable the Node SSR proxy                 |
| `FLOCI_OCI_EMULATOR_UI_PROXY_TARGET`    | `http://127.0.0.1:4600` | SSR server target                         |
| `FLOCI_OCI_EMULATOR_INSPECTOR_ENABLED`  | `true`                  | Enable the request inspector              |
| `FLOCI_OCI_EMULATOR_INSPECTOR_CAPACITY` | `500`                   | Ring buffer size                          |
| `FLOCI_OCI_EMULATOR_SESSION_TTL`        | `8h`                    | Simulated session lifetime                |

Quarkus maps `floci-oci.port` to `FLOCI_OCI_PORT` and so on. Use the environment form in scripts and CI.

Examples:

```bash
FLOCI_OCI_PORT=4699 make dev
FLOCI_OCI_STORAGE_MODE=in-memory make dev
FLOCI_OCI_STORAGE_DIR=/tmp/oci-emulator-data make run
```

If you change the port, update `bin/ocilocal` or set `OCI_CLI_ENDPOINT` to the new value.

---

## 3. Node SSR configuration

The production UI server reads:

| Variable     | Default                 | Purpose                        |
| ------------ | ----------------------- | ------------------------------ |
| `PORT`       | `4600`                  | SSR server port                |
| `API_ORIGIN` | `http://127.0.0.1:4599` | Backend origin used during SSR |

`make run` starts Quarkus and this server. `make run-lite` skips it and serves the prerendered and client-rendered build directly from Quarkus.

---

## 4. Service toggles

Services can be enabled or disabled:

```yaml
floci-oci:
  services:
    core:
      enabled: true
    loadbalancer:
      enabled: true
    apigateway:
      enabled: true
    dns:
      enabled: true
    database:
      enabled: true
    mysql:
      enabled: true
    queue:
      enabled: false
    streaming:
      enabled: false
    kms:
      enabled: false
    vault:
      enabled: false
    organizations:
      enabled: false
    governance-rules:
      enabled: false
    functions:
      enabled: false
    oke:
      enabled: false
    postgresql:
      enabled: false
```

Restart the backend after changing service configuration. Disabled services are not registered and return an OCI-shaped error. Enabling a service that needs Docker requires a working Docker environment, which is not part of the default setup.

---

## 5. UI settings

UI preferences are stored in browser cookies and local storage:

- `emu.locale`: `en` or `es`.
- `emu.region`: selected region.
- `emu.compartment`: selected compartment.
- `emu_session`: simulated session payload.

Clearing site data resets the UI preferences and signs you out. The session cookie is not a security token. See [ARCHITECTURE.md](../ARCHITECTURE.md) section 7.

---

## 6. Storage and data directory

All emulator state lives in `./data` by default. It is managed by the floci storage backend. Do not edit it by hand while the server is running.

- `persistent` mode keeps state across restarts.
- `in-memory` mode starts empty on every run and is intended for tests.

Object Storage keeps real object bytes in the storage backend. Metadata and control plane resources are stored as JSON. See [DATA-MODEL.md](DATA-MODEL.md) for the resource model and [ARCHITECTURE.md](../ARCHITECTURE.md) section 9 for the storage design.

---

## 7. Ports and profiles

| Profile | Public | UI server          | Notes                                 |
| ------- | ------ | ------------------ | ------------------------------------- |
| `dev`   | `4599` | Quinoa on `4200`   | Angular dev server with HMR           |
| `prod`  | `4599` | Node SSR on `4600` | Quarkus proxies the UI                |
| `lite`  | `4599` | None               | Prerendered and client-rendered build |

Only the public port is meant to be opened in a browser. Keep `4600` internal.
