# Configuration Reference

Complete configuration reference for all HoneyBee components.

## HoneyBee Core Configuration

File: `bee_config.toml`

```toml
[server]
host = "127.0.0.1"        # Bind address
node_port = 9001         # Port for node connections
backend_port = 9002      # Port for backend API
debug = false            # Enable debug mode

[logging]
level = "debug"          # Log level: trace, debug, info, warn, error
folder = "logs"          # Log directory
force_color = true       # Force colored output

[proxy]
enabled = true           # Enable WebSocket proxy
host = "0.0.0.0"        # Proxy bind address
port = 9003             # Proxy port
```

## HoneyBee Node Configuration

File: `configs/config.yaml`

```yaml
node:
  name: "honeybee-node-01"  # Unique node identifier
  type: "Full"              # "Full" or "Agent"
  address: "0.0.0.0"        # Address to report
  port: 8080                # Port to report

server:
  address: "127.0.0.1:9001"  # Core manager address
  heartbeat_interval: 30     # Heartbeat interval (seconds)
  reconnect_delay: 5         # Reconnect delay (seconds)
  connection_timeout: 10      # Connection timeout (seconds)

tls:
  enabled: true               # Enable TLS encryption
  insecure_skip_verify: false # Skip certificate verification
  server_name: "honeybee-manager"  # Server name for TLS
  cert_file: ""              # Client certificate (optional)
  key_file: ""               # Client key (optional)
  ca_file: ""                # CA certificate (optional)

auth:
  totp_enabled: true         # Enable TOTP authentication
  totp_secret_dir: ""        # TOTP secret directory (optional)

log:
  level: "info"              # Log level: debug, info, warn, error
  format: "text"             # Log format: text or json
  file: ""                   # Log file path (optional)

honeypot:
  enabled: true              # Enable honeypot management
  base_dir: "~/.honeybee/honeypots"  # Honeypot installation directory
  default_ssh_port: 2222     # Default SSH port
  default_telnet_port: 2223  # Default Telnet port
```

## Environment Variables

### HoneyBee Node

- `HONEYBEE_EVENT_PORT` - Port for honeypot events (default: 9100)
- `HONEYBEE_POT_ID` - Honeypot instance ID
- `HONEYBEE_HONEYPOT_TYPE` - Honeypot type

### HoneyBee Core

- `RUST_LOG` - Rust logging level (default: info)

## Configuration Validation

Both Core and Node validate configuration on startup. Invalid configuration will cause the component to exit with an error.

## Next Steps

- [Core Configuration](../core/configuration.md) - Core configuration details
- [Node Configuration](../node/configuration.md) - Node configuration details
