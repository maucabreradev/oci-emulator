# Troubleshooting

Common startup, environment, CLI, and UI problems.

For configuration details, see [CONFIGURATION.md](CONFIGURATION.md). For CLI profile errors, see [CLI.md](CLI.md) section 7. For security limits, see [SECURITY.md](../SECURITY.md).

---

## Port already in use

If `4599` is busy:

```bash
lsof -i :4599
```

Stop the process or set `FLOCI_OCI_PORT` to another port. Remember to update `bin/ocilocal` if you change the port.

---

## Quarkus fails to start with the wrong Java version

Check:

```bash
java -version
mvn -version
```

Both must report Java 25. The repository pins the version in `.java-version`; a version manager reads it automatically. Otherwise set `JAVA_HOME` to the Java 25 installation.

---

## Quinoa does not start

Confirm Node and pnpm:

```bash
node --version
pnpm --version
```

Then reinstall frontend dependencies:

```bash
cd console
pnpm install
```

---

## The UI shows a proxy or connection error in production

The Node SSR server is not running or is not reachable on `4600`.

```bash
make run
```

If you intentionally do not want SSR:

```bash
make run-lite
```

---

## Login redirects in a loop

- Use the same host consistently. `localhost` and `127.0.0.1` have different cookie stores.
- Allow cookies for `localhost`.
- Clear the `emu_session` cookie and try again.
- Confirm the emulator clock is correct. Session cookies have a TTL.

---

## CLI cannot find the config

Use the wrapper:

```bash
bin/ocilocal iam region list
```

Or set:

```bash
export OCI_CLI_CONFIG_FILE="$PWD/.oci/config"
export OCI_CLI_PROFILE=OCIEMU
export OCI_CLI_ENDPOINT=http://localhost:4599
```

---

## CLI authentication errors

Regenerate the local key and config:

```bash
make setup
```

The server does not verify the signature. Authentication errors usually mean the profile or endpoint is wrong.

---

## Object upload fails with a size error

Increase the request size limit:

```bash
FLOCI_OCI_MAX_REQUEST_SIZE=256 make dev
```

---

## Reset does not work

Stop the emulator before deleting `data/`. A running server may keep file handles or rewrite state.

---

## WSL2 networking

Use `http://localhost:4599` from Windows and from WSL2. If your WSL2 configuration isolates networking, use the WSL2 IP address and bind the server to `0.0.0.0` only for local development.

---

## Docker

Docker is not required. Do not enable services that depend on Docker unless you have a working Docker environment.

---

## Data directory permissions

If the emulator cannot write to `./data`:

```bash
ls -ld data
```

The current user must own the directory. Fix it with:

```bash
chmod u+rwx data
```

Do not run the emulator as root.

---

## Session cookie not stored

The console needs cookies for `localhost`. Check that the browser does not block third-party or local cookies and that you are not mixing `localhost` with `127.0.0.1`.
