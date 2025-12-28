# HoneyBee Core Configuration

Complete configuration reference for HoneyBee Core.

## Configuration File

Core uses `bee_config.toml` for configuration. The file is loaded from the current directory or can be specified via command line.

## Configuration Sections

### [server]

Server configuration:

```toml
[server]
host = "127.0.0.1"        # Bind address for node connections
node_port = 9001          # Port for node connections (TCP)
backend_port = 9002       # Port for backend API (HTTP)
debug = false             # Enable debug mode
```

**Options**:
- `host` - IP address or hostname to bind to (default: "127.0.0.1")
- `node_port` - Port for node connections (default: 9001)
- `backend_port` - Port for backend API (default: 9002)
- `debug` - Enable debug mode for additional logging (default: false)

### [logging]

Logging configuration:

```toml
[logging]
level = "debug"           # Log level
folder = "logs"           # Log directory
force_color = true        # Force colored output
```

**Options**:
- `level` - Log level: `trace`, `debug`, `info`, `warn`, `error` (default: "debug")
- `folder` - Directory for log files (default: "logs")
- `force_color` - Force colored output even when not in TTY (default: true)

### [proxy]

WebSocket proxy configuration:

```toml
[proxy]
enabled = true           # Enable WebSocket proxy
host = "0.0.0.0"        # Bind address
port = 9003             # Proxy port
```

**Options**:
- `enabled` - Enable WebSocket proxy (default: true)
- `host` - Bind address for proxy (default: "0.0.0.0")
- `port` - Port for WebSocket connections (default: 9003)

## Example Configurations

### Development

```toml
[server]
host = "127.0.0.1"
node_port = 9001
backend_port = 9002
debug = true

[logging]
level = "debug"
folder = "logs"
force_color = true

[proxy]
enabled = true
host = "127.0.0.1"
port = 9003
```

### Production

```toml
[server]
host = "0.0.0.0"
node_port = 9001
backend_port = 9002
debug = false

[logging]
level = "info"
folder = "/var/log/honeybee"
force_color = false

[proxy]
enabled = true
host = "0.0.0.0"
port = 9003
```

## Environment Variables

Core can also be configured via environment variables (future feature).

## Configuration Validation

Core validates configuration on startup. Invalid configuration will cause Core to exit with an error message.

## Next Steps

- [Deployment](./deployment.md) - Deploy with configuration
- [Architecture](./architecture.md) - Understand how configuration is used

