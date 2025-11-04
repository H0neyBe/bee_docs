# Architecture

Technical architecture of the HoneyBee node implementation.

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    HoneyBee Node                        │
├─────────────────────────────────────────────────────────┤
│  cmd/node/main.go                                       │
│  ├── Configuration Loading                              │
│  ├── Logger Initialization                              │
│  ├── Signal Handling                                    │
│  └── Client Lifecycle Management                        │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│              Internal Packages                          │
├─────────────────────────────────────────────────────────┤
│  internal/                                              │
│  ├── client/      (Core client logic)                   │
│  ├── protocol/    (Message definitions)                 │
│  ├── auth/        (TLS & TOTP)                          │
│  ├── config/      (Configuration management)            │
│  └── logger/      (Structured logging)                  │
└─────────────────────────────────────────────────────────┘
```

## Component Details

### Client Package

The core client manages:
- TCP/TLS connection to manager
- Message serialization/deserialization
- Heartbeat scheduling
- Command processing
- Automatic reconnection

### Protocol Package

Defines all Protocol v2 message types:
- NodeRegistration
- NodeStatusUpdate
- NodeEvent
- NodeCommand
- RegistrationAck

### Auth Package

Handles authentication:
- **TLS**: Certificate management, TLS 1.3 configuration
- **TOTP**: Secret generation, code generation, storage

### Config Package

Configuration management:
- YAML parsing
- Validation
- Default generation
- Type-safe access

### Logger Package

Structured logging:
- Multiple formats (text/JSON)
- Configurable levels
- Field-based logging
- File and stdout output

## Concurrency Model

```
Main Goroutine: Connection management & lifecycle
  ├── Reader Goroutine: Asynchronous message reading
  ├── Heartbeat Ticker: Periodic status updates
  └── Signal Handler: Graceful shutdown
```

## Security Layers

1. **Network Layer**: Firewall, VPN
2. **Transport Layer**: TLS 1.3
3. **Authentication Layer**: TOTP
4. **Application Layer**: Input validation

## State Machine

```
[Start] → [Connecting] → [Registering] → [Running] → [Stopped]
            ↑               ↓                ↓
            └───────────────┴────────────────┘
                    (Reconnect on error)
```

For more details, see:
- [Design decisions documentation](../../honeybee_node/docs/ARCHITECTURE.md)
- [Protocol specification](../protocol.md)
- [Security guide](./security.md)

