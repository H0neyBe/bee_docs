# API Reference

Complete API reference for HoneyBee Node internals and protocol.

## Protocol Messages

See [Message Protocol Definition](../protocol.md) for the complete protocol specification.

### Node to Manager Messages

#### NodeRegistration

Initial handshake message sent during connection establishment.

```go
type NodeRegistration struct {
    NodeID   uint64   // Unique node identifier
    NodeName string   // Human-readable node name
    Address  string   // Node address
    Port     uint16   // Node port
    NodeType NodeType // "Agent" or "Full"
    TOTPCode string   // TOTP authentication code (optional)
}
```

**JSON Example:**
```json
{
  "version": 2,
  "message": {
    "NodeRegistration": {
      "node_id": 123456789,
      "node_name": "honeypot-01",
      "address": "192.168.1.100",
      "port": 8080,
      "node_type": "Agent",
      "totp_code": "123456"
    }
  }
}
```

#### NodeStatusUpdate

Heartbeat message reporting node status.

```go
type NodeStatusUpdate struct {
    NodeID uint64     // Node identifier
    Status NodeStatus // Current status
}
```

**Status Values:**
- `Deploying` - Node is initializing
- `Running` - Node is operational
- `Stopped` - Node has stopped
- `Failed` - Node encountered error
- `Unknown` - Status unknown

#### NodeEvent

Event notification message.

```go
type NodeEvent struct {
    Type        string // Event type
    Message     string // Error message (optional)
    Description string // Alarm description (optional)
}
```

**Event Types:**
- `Started` - Node started
- `Stopped` - Node stopped
- `Alarm` - Security alert
- `Error` - Error occurred

#### NodeDrop

Graceful disconnect notification.

```json
{
  "version": 2,
  "message": "NodeDrop"
}
```

### Manager to Node Messages

#### RegistrationAck

Registration acknowledgment from manager.

```go
type RegistrationAck struct {
    NodeID   uint64  // Node identifier
    Accepted bool    // Registration accepted/rejected
    Message  *string // Optional message
    TOTPKey  string  // TOTP key for first registration (optional)
}
```

#### NodeCommand

Command from manager to node.

```go
type NodeCommand struct {
    NodeID  uint64 // Target node
    Command string // Command to execute
}
```

**Standard Commands:**
- `status` - Request status update
- `stop` - Stop the node
- `restart` - Restart the node
- Custom commands supported

## Configuration API

### Configuration Structure

```go
type Config struct {
    Node   NodeConfig
    Server ServerConfig
    TLS    TLSConfig
    Auth   AuthConfig
    Log    LogConfig
}
```

See [Configuration Reference](./config.md) for complete details.

## Client API

### NodeClient

Main client structure managing connection to manager.

```go
type NodeClient struct {
    cfg        *config.Config
    nodeID     uint64
    totpMgr    *auth.TOTPManager
    tlsConfig  *tls.Config
    conn       net.Conn
    // ... internal fields
}
```

**Methods:**
- `Run()` - Start the client event loop
- `Stop()` - Stop the client gracefully
- `connect()` - Establish connection
- `register()` - Send registration
- `sendStatusUpdate()` - Send status
- `sendEvent()` - Send event
- `handleMessage()` - Process incoming message

## Authentication API

### TOTPManager

Manages TOTP secret generation and code generation.

```go
type TOTPManager struct {
    secret    string
    secretDir string
}
```

**Methods:**
- `LoadOrGenerateSecret()` - Load existing or generate new secret
- `GenerateCode()` - Generate TOTP code
- `ValidateCode()` - Validate TOTP code
- `GetSecret()` - Get current secret
- `SetSecret()` - Set secret
- `DeleteSecret()` - Remove secret

## Logging API

### Logger Functions

```go
// Initialize logger
func Init(level, format, logFile string) error

// Logging methods
func Debug(args ...interface{})
func Info(args ...interface{})
func Warn(args ...interface{})
func Error(args ...interface{})

// Structured logging
func WithFields(fields map[string]interface{}) *logrus.Entry
```

## Constants

```go
const ProtocolVersion uint64 = 2
const HeartbeatInterval = 30 * time.Second
const ReconnectDelay = 5 * time.Second
```

## Error Types

Common error types returned by the API:

- Configuration errors
- Connection errors
- Authentication errors
- TLS errors
- Protocol errors

See [Troubleshooting Guide](../node/troubleshooting.md) for error handling.

## Future API

Planned features for future versions:

- Metrics endpoint (Prometheus)
- Health check endpoint
- Plugin system
- Hot configuration reload
- Multiple manager support

