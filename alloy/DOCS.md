# Grafana Alloy for Home Assistant

Ship Home Assistant OS logs to a remote [Loki](https://grafana.com/oss/loki/) instance using [Grafana Alloy](https://grafana.com/docs/alloy/latest/).

This add-on replaces the deprecated Promtail add-on, which is incompatible with modern HAOS versions (11+) due to systemd 252+ compact journal format changes.

## Configuration

### Required

- **loki_url**: The full URL to your Loki push endpoint (e.g., `http://192.168.1.45:3100/loki/api/v1/push`)

### Optional

- **log_level**: Alloy log verbosity (`debug`, `info`, `warn`, `error`). Default: `info`
- **additional_config**: Extra Alloy config blocks to append (advanced users)

### TLS / mTLS

For Loki endpoints secured with TLS or mutual TLS (client certificates). Certificate files must be placed in the Home Assistant `/ssl/` directory (accessible via Samba, SSH, or the File Editor add-on). Option values are **filenames** relative to `/ssl/`.

- **tls_ca_cert**: CA certificate filename (e.g., `ca.crt`). Use when Loki's certificate is signed by a private CA.
- **tls_client_cert**: Client certificate filename (e.g., `client.crt`). Required for mTLS.
- **tls_client_key**: Client key filename (e.g., `client.key`). Required for mTLS.
- **tls_server_name**: Override the server name for TLS verification and SNI. Use when connecting by IP address but the server certificate contains a hostname (e.g., `loki-ingest.example.com`).

### Basic Auth

For Loki endpoints protected with HTTP basic authentication.

- **basic_auth_user**: Username
- **basic_auth_password**: Password

### Example: mTLS

Place your certificates in `/ssl/` on your Home Assistant instance:

```
/ssl/ca.crt          # CA that signed the Loki server certificate
/ssl/client.crt      # Client certificate
/ssl/client.key      # Client private key
```

Add-on configuration:

```yaml
loki_url: "https://loki-ingest.example.com/loki/api/v1/push"
tls_ca_cert: "ca.crt"
tls_client_cert: "client.crt"
tls_client_key: "client.key"
```

### Example: mTLS with IP address

When DNS cannot resolve the Loki hostname (e.g., `.local` domain conflicts with mDNS), connect by IP and use `tls_server_name` to set the expected hostname for TLS verification and SNI routing:

```yaml
loki_url: "https://192.168.1.45/loki/api/v1/push"
tls_ca_cert: "ca.crt"
tls_client_cert: "client.crt"
tls_client_key: "client.key"
tls_server_name: "loki-ingest.example.com"
```

### Example: Basic Auth

```yaml
loki_url: "https://loki.example.com/loki/api/v1/push"
basic_auth_user: "admin"
basic_auth_password: "secret"
```

### Example: mTLS + Basic Auth

Both can be combined if your endpoint requires it:

```yaml
loki_url: "https://loki-ingest.example.com/loki/api/v1/push"
tls_ca_cert: "ca.crt"
tls_client_cert: "client.crt"
tls_client_key: "client.key"
basic_auth_user: "admin"
basic_auth_password: "secret"
```

## Labels

All journal entries are shipped to Loki with these labels:

| Label | Source |
|-------|--------|
| `job` | `systemd-journal` (static) |
| `unit` | systemd unit name |
| `hostname` | machine hostname |
| `syslog_identifier` | process identifier |
| `transport` | journal transport type |
| `container_name` | Docker container name (for add-ons) |
| `level` | log priority (debug, info, warning, error, etc.) |

## Debug UI

The Alloy debug UI is available at `http://<haos-ip>:12345` when the add-on is running. Use it to inspect component health, view the pipeline DAG, and troubleshoot issues.

## Advanced: Additional Config

The `additional_config` option lets you append raw Alloy config blocks. For example, to also scrape a file:

```
local.file_match "extra" { path_targets = [{__path__ = "/config/home-assistant.log"}] }
loki.source.file "extra" { targets = local.file_match.extra.targets forward_to = [loki.write.loki.receiver] }
```

Note: This is injected as-is into the config file. Syntax errors will prevent Alloy from starting.

## Troubleshooting

- **No logs in Loki**: Check that `loki_url` is reachable from HAOS. Try `ping <loki-host>` from the SSH add-on.
- **Add-on crashes on start**: Check the add-on log for Alloy config errors. Set `log_level: debug` for verbose output.
- **"timestamp too old" in Loki**: Normal on first start. Alloy reads the full journal history; Loki rejects entries outside its retention window. Resolves in 1-2 minutes.
- **TLS certificate errors**: Verify the filenames in the add-on config match the files in `/ssl/`. Check the add-on log for specific certificate errors (expired, wrong CA, etc.).
- **"file not found" on startup**: The add-on validates that certificate files exist before starting. Ensure the files are in `/ssl/` on your Home Assistant instance.

## Support

Report issues at: https://github.com/ecohash-co/ha-addon-alloy/issues
