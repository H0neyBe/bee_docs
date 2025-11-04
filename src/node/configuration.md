# Configuration Guide

Complete reference for configuring your HoneyBee node.

## Configuration File

The node uses YAML format for configuration. The default location is `configs/config.yaml`, but you can specify a different path with the `-config` flag.

## Full Configuration Example

```yaml
# Node identity and settings
node:
  name: "honeypot-01"
  type: "Agent"
  address: "0.0.0.0"
  port: 8080

# Manager connection settings
server:
  address: "manager.example.com:9001"
  heartbeat_interval: 30
  reconnect_delay: 5
  connection_timeout: 10

# TLS encryption settings
tls:
  enabled: true
  cert_file: "/etc/honeybee/certs/client.crt"
  key_file: "/etc/honeybee/certs/client.key"
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"

# Authentication settings
auth:
  totp_enabled: true
  totp_secret_dir: "/var/lib/honeybee/secrets"

# Logging settings
log:
  level: "info"
  format: "json"
  file: "/var/log/honeybee/node.log"
```

## Configuration Sections

### Node Settings

```yaml
node:
  name: "honeypot-01"    # Node identifier
  type: "Agent"          # Node type: "Agent" or "Full"
  address: "0.0.0.0"     # Address to report to manager
  port: 8080             # Port to report to manager
```

#### `name` (string, optional)

Unique identifier for this node. If not specified, defaults to the system hostname.

**Examples:**
```yaml
name: "honeypot-01"
name: "edge-sensor-us-east-1"
name: "production-full-node"
```

**Best practices:**
- Use descriptive names
- Include location or purpose
- Keep it unique across your deployment

#### `type` (string, required)

The type of node determines its capabilities:

| Type | Description | Use Case |
|------|-------------|----------|
| `Agent` | Lightweight monitoring | Edge deployment, resource-constrained |
| `Full` | Full honeypot features | Production honeypots, interactive deception |

**Example:**
```yaml
type: "Agent"  # or "Full"
```

#### `address` (string, required)

IP address that this node reports to the manager. This is the address where the node can be reached.

**Examples:**
```yaml
address: "0.0.0.0"        # Listen on all interfaces
address: "192.168.1.100"  # Specific internal IP
address: "10.0.0.50"      # Private network IP
```

#### `port` (integer, required)

Port number to report to the manager.

**Example:**
```yaml
port: 8080
```

### Server Settings

```yaml
server:
  address: "manager.example.com:9001"
  heartbeat_interval: 30
  reconnect_delay: 5
  connection_timeout: 10
```

#### `address` (string, required)

Manager server address in `host:port` format.

**Examples:**
```yaml
address: "127.0.0.1:9001"              # Local testing
address: "manager.example.com:9001"   # Production domain
address: "10.0.0.5:9001"               # Internal IP
```

#### `heartbeat_interval` (integer, optional, default: 30)

Seconds between status update heartbeats.

**Recommended values:**
- `30` - Default, good for most cases
- `10-15` - High-availability scenarios
- `60` - Low-bandwidth environments

**Example:**
```yaml
heartbeat_interval: 30  # seconds
```

#### `reconnect_delay` (integer, optional, default: 5)

Seconds to wait before attempting reconnection after connection loss.

**Example:**
```yaml
reconnect_delay: 5  # seconds
```

#### `connection_timeout` (integer, optional, default: 10)

Timeout in seconds for initial connection attempts.

**Example:**
```yaml
connection_timeout: 10  # seconds
```

### TLS Settings

```yaml
tls:
  enabled: true
  cert_file: "/path/to/client.crt"
  key_file: "/path/to/client.key"
  ca_file: "/path/to/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"
```

#### `enabled` (boolean, required)

Enable or disable TLS encryption.

**⚠️ Security Warning:** Always `true` in production!

**Example:**
```yaml
enabled: true  # Always use true in production
```

#### `cert_file` (string, optional)

Path to client certificate for mutual TLS authentication.

**Example:**
```yaml
cert_file: "/etc/honeybee/certs/client.crt"
```

#### `key_file` (string, optional)

Path to client private key for mutual TLS.

**Example:**
```yaml
key_file: "/etc/honeybee/certs/client.key"
```

**Note:** Both `cert_file` and `key_file` must be specified together for mutual TLS.

#### `ca_file` (string, optional)

Path to CA certificate for server verification.

**Example:**
```yaml
ca_file: "/etc/honeybee/certs/ca.crt"
```

#### `insecure_skip_verify` (boolean, optional, default: false)

Skip TLS certificate verification.

**⚠️ Security Warning:** Only set to `true` for local testing!

**Example:**
```yaml
insecure_skip_verify: false  # Always false in production
```

#### `server_name` (string, optional)

Expected server name in TLS certificate (for SNI and verification).

**Example:**
```yaml
server_name: "honeybee-manager"
```

### Authentication Settings

```yaml
auth:
  totp_enabled: true
  totp_secret_dir: "/var/lib/honeybee/secrets"
```

#### `totp_enabled` (boolean, required)

Enable TOTP authentication.

**⚠️ Security Warning:** Always `true` in production!

**Example:**
```yaml
totp_enabled: true  # Always true in production
```

#### `totp_secret_dir` (string, optional)

Directory to store TOTP secret. Defaults to `~/.config/honeybee/`.

**Examples:**
```yaml
totp_secret_dir: "/var/lib/honeybee/secrets"  # System-wide
totp_secret_dir: "/etc/honeybee/secrets"       # Alternative
# Default: ~/.config/honeybee/
```

**Permissions:** Must be readable/writable only by the node user (0700).

### Logging Settings

```yaml
log:
  level: "info"
  format: "json"
  file: "/var/log/honeybee/node.log"
```

#### `level` (string, optional, default: "info")

Log level controls verbosity.

| Level | Use Case |
|-------|----------|
| `debug` | Development, troubleshooting |
| `info` | Normal production operation |
| `warn` | Only warnings and errors |
| `error` | Only errors |

**Example:**
```yaml
level: "info"  # Recommended for production
```

#### `format` (string, optional, default: "text")

Log output format.

| Format | Description | Use Case |
|--------|-------------|----------|
| `text` | Human-readable | Console, development |
| `json` | Structured JSON | Log aggregation, parsing |

**Example:**
```yaml
format: "json"  # Recommended for production
```

#### `file` (string, optional)

Path to log file. If not specified, logs go to stdout.

**Examples:**
```yaml
file: "/var/log/honeybee/node.log"  # System log directory
file: "./logs/node.log"             # Relative path
# Omit for stdout only
```

## Configuration Validation

The node validates configuration on startup:

```bash
# Test configuration
./honeybee-node -config configs/config.yaml

# If valid, you'll see:
INFO: Configuration loaded successfully

# If invalid, you'll see specific error:
ERROR: Invalid configuration: <details>
```

## Environment Variables

Some settings can be overridden with environment variables:

```bash
# Override server address
export SERVER_ADDRESS="manager.example.com:9001"

# Run node
./honeybee-node -config configs/config.yaml
```

## Configuration Profiles

### Development Profile

```yaml
node:
  name: "dev-node"
  type: "Agent"

server:
  address: "127.0.0.1:9001"

tls:
  enabled: false  # ⚠️ Testing only!

auth:
  totp_enabled: false  # ⚠️ Testing only!

log:
  level: "debug"
  format: "text"
```

### Production Profile

```yaml
node:
  name: "prod-honeypot-01"
  type: "Full"
  address: "10.0.1.100"
  port: 8080

server:
  address: "manager.internal.example.com:9001"
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

### Edge Deployment Profile

```yaml
node:
  name: "edge-sensor-01"
  type: "Agent"
  address: "0.0.0.0"
  port: 8080

server:
  address: "vpn.manager.example.com:9001"
  heartbeat_interval: 60  # Longer interval for bandwidth
  reconnect_delay: 10
  connection_timeout: 30

tls:
  enabled: true
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"

auth:
  totp_enabled: true

log:
  level: "warn"  # Minimal logging
  format: "json"
  file: "/var/log/honeybee/node.log"
```

## Security Best Practices

1. **Always enable TLS in production**
   ```yaml
   tls:
     enabled: true
     insecure_skip_verify: false
   ```

2. **Always enable TOTP in production**
   ```yaml
   auth:
     totp_enabled: true
   ```

3. **Use appropriate log levels**
   ```yaml
   log:
     level: "info"  # Not "debug" in production
   ```

4. **Secure file permissions**
   ```bash
   chmod 600 /etc/honeybee/config.yaml
   chmod 600 /etc/honeybee/certs/*.key
   chmod 700 /var/lib/honeybee/secrets
   ```

5. **Use strong server names**
   ```yaml
   tls:
     server_name: "honeybee-manager.internal.example.com"
   ```

## Next Steps

- [Set up TLS encryption](./tls.md)
- [Configure TOTP authentication](./totp.md)
- [Deploy to production](./deployment.md)

