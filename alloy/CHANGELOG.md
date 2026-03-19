# Changelog

## 1.2.0 - 2026-03-19

### Added
- mTLS client certificate support (`tls_ca_cert`, `tls_client_cert`, `tls_client_key`)
- Basic auth support (`basic_auth_user`, `basic_auth_password`)
- SSL volume mapping for certificate access from `/ssl/`
- TLS server name override (`tls_server_name`) for TLS verification against a specific hostname
- DNS overrides (`extra_hosts`) for resolving hostnames when DNS is unavailable (e.g., `.local` mDNS conflicts)
- Certificate file validation on startup with clear error messages

## 1.0.0 - 2026-02-21

### Added
- Initial release
- Grafana Alloy v1.13.1
- Systemd journal log shipping to Loki
- Journal field relabeling (unit, hostname, syslog_identifier, transport, container_name, level)
- Debug UI on port 12345
- Configurable Loki URL, log level, and additional config
- Watchdog health check via Alloy's `/-/ready` endpoint
- Support for amd64 and aarch64 architectures
