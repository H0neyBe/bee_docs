# Error Codes

Reference for error codes and messages.

## Node Errors

### Connection Errors

- `CONNECTION_FAILED` - Failed to connect to Core
- `TLS_HANDSHAKE_FAILED` - TLS handshake failed
- `REGISTRATION_FAILED` - Node registration failed
- `AUTHENTICATION_FAILED` - TOTP authentication failed

### Configuration Errors

- `CONFIG_READ_FAILED` - Failed to read configuration file
- `CONFIG_PARSE_FAILED` - Failed to parse configuration
- `CONFIG_INVALID` - Configuration validation failed
- `EMPTY_NODE_NAME` - Node name cannot be empty
- `INVALID_NODE_TYPE` - Invalid node type (must be "Full" or "Agent")
- `EMPTY_SERVER_ADDR` - Server address is required

### Honeypot Errors

- `HONEYPOT_NOT_FOUND` - Honeypot not found
- `HONEYPOT_ALREADY_EXISTS` - Honeypot already exists
- `HONEYPOT_INSTALL_FAILED` - Honeypot installation failed
- `HONEYPOT_START_FAILED` - Failed to start honeypot
- `HONEYPOT_STOP_FAILED` - Failed to stop honeypot

## Core Errors

### Node Management Errors

- `NODE_NOT_FOUND` - Node not found
- `NODE_ALREADY_REGISTERED` - Node already registered
- `INVALID_NODE_ID` - Invalid node ID

### Command Errors

- `COMMAND_FAILED` - Command execution failed
- `INVALID_COMMAND` - Invalid command type
- `COMMAND_TIMEOUT` - Command execution timeout

## Protocol Errors

- `PROTOCOL_VERSION_MISMATCH` - Protocol version mismatch
- `INVALID_MESSAGE_FORMAT` - Invalid message format
- `MESSAGE_PARSE_FAILED` - Failed to parse message

## Error Response Format

Errors are returned in status updates:

```json
{
  "status": "Failed",
  "message": "Error description"
}
```

## Troubleshooting

See [Troubleshooting Guide](../node/troubleshooting.md) for common errors and solutions.

