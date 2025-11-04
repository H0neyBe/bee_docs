# Configuration Reference

Complete reference for all configuration options.

See [Configuration Guide](../node/configuration.md) for usage examples.

## Configuration File Format

YAML format, default location: `configs/config.yaml`

## Complete Configuration

```yaml
node:
  name: "honeypot-01"
  type: "Agent"
  address: "0.0.0.0"
  port: 8080

server:
  address: "manager.example.com:9001"
  heartbeat_interval: 30
  reconnect_delay: 5
  connection_timeout: 10

tls:
  enabled: true
  cert_file: "/etc/honeybee/certs/client.crt"
  key_file: "/etc/honeybee/certs/client.key"
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"

auth:
  totp_enabled: true
  totp_secret_dir: "/var/lib/honeybee/secrets"

log:
  level: "info"
  format: "json"
  file: "/var/log/honeybee/node.log"
```

## Configuration Sections

### Node Configuration

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | No | hostname | Node identifier |
| `type` | string | Yes | - | "Agent" or "Full" |
| `address` | string | Yes | - | IP address to report |
| `port` | uint16 | Yes | - | Port to report |

### Server Configuration

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `address` | string | Yes | - | Manager address (host:port) |
| `heartbeat_interval` | int | No | 30 | Seconds between heartbeats |
| `reconnect_delay` | int | No | 5 | Seconds before reconnect |
| `connection_timeout` | int | No | 10 | Connection timeout seconds |

### TLS Configuration

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `enabled` | bool | Yes | - | Enable TLS encryption |
| `cert_file` | string | No | - | Client certificate path |
| `key_file` | string | No | - | Client key path |
| `ca_file` | string | No | - | CA certificate path |
| `insecure_skip_verify` | bool | No | false | Skip cert verification |
| `server_name` | string | No | - | Expected server name |

### Auth Configuration

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `totp_enabled` | bool | Yes | - | Enable TOTP auth |
| `totp_secret_dir` | string | No | ~/.config/honeybee | Secret storage directory |

### Log Configuration

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `level` | string | No | info | Log level (debug/info/warn/error) |
| `format` | string | No | text | Log format (text/json) |
| `file` | string | No | - | Log file path (stdout if not set) |

## Environment Variables

Some settings can be overridden:

| Variable | Overrides | Example |
|----------|-----------|---------|
| `SERVER_ADDRESS` | `server.address` | `manager.example.com:9001` |

## Validation Rules

The node validates configuration on startup:

- `node.type` must be "Agent" or "Full"
- `server.address` must not be empty
- `server.heartbeat_interval` must be > 0
- `server.reconnect_delay` must be > 0
- If `tls.cert_file` specified, `tls.key_file` required
- `log.level` must be debug/info/warn/error

## Command Line Flags

| Flag | Description | Default |
|------|-------------|---------|
| `-config` | Path to config file | configs/config.yaml |
| `-version` | Show version and exit | - |
| `-gen-config` | Generate default config | - |

## Configuration Templates

See [Configuration Guide](../node/configuration.md) for profile templates:
- Development profile
- Production profile
- Edge deployment profile

## Next Steps

- [Configuration Guide](../node/configuration.md) - Usage examples
- [Security Guide](../node/security.md) - Security settings
- [Deployment Guide](../node/deployment.md) - Production deployment

