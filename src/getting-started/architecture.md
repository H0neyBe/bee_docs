# Architecture Overview

This document provides a high-level overview of the HoneyBee architecture.

## System Components

HoneyBee consists of three main components:

1. **HoneyBee Core** - Central orchestration manager
2. **HoneyBee Node** - Distributed honeypot nodes
3. **HoneyBee Potstore** - Honeypot repository

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    HoneyBee Core                             │
│                    (Rust Application)                        │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Node Manager                              │    │
│  │  - Node Registration                                 │    │
│  │  - Status Tracking                                    │    │
│  │  - Command Distribution                               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Backend Manager                            │    │
│  │  - REST API (Port 9002)                              │    │
│  │  - WebSocket Proxy (Port 9003)                       │    │
│  │  - Event Aggregation                                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Message Protocol (v2)                      │    │
│  │  - JSON over TCP/TLS                                  │    │
│  │  - Node Port: 9001                                    │    │
│  └─────────────────────────────────────────────────────┘    │
└───────────────────────┬───────────────────────────────────────┘
                        │
                        │ Protocol v2 (TCP/TLS)
                        │
        ┌───────────────┴───────────────┐
        │                               │
   ┌────▼────┐                    ┌────▼────┐
   │  Node 1 │                    │  Node 2 │
   │  (Go)   │                    │  (Go)   │
   └────┬────┘                    └────┬────┘
        │                               │
   ┌────▼────┐                    ┌────▼────┐
   │ Cowrie  │                    │HonnyPot │
   │ Pot     │                    │ter Pot  │
   └─────────┘                    └─────────┘
```

## Component Details

### HoneyBee Core

**Purpose**: Central orchestration and management

**Responsibilities**:
- Accept node connections
- Register and authenticate nodes
- Distribute commands to nodes
- Aggregate events from nodes
- Provide API for external systems
- Track node and honeypot status

**Ports**:
- `9001` - Node connections (TCP)
- `9002` - Backend API (HTTP)
- `9003` - WebSocket proxy (WS)

**Components**:
- `NodeManager` - Manages node lifecycle
- `BackendManager` - Handles API and WebSocket
- `Message Protocol` - Protocol v2 implementation

### HoneyBee Node

**Purpose**: Deploy and manage honeypots

**Responsibilities**:
- Connect to Core manager
- Authenticate with TOTP
- Install honeypots from Potstore
- Manage honeypot lifecycle (start/stop/restart)
- Forward honeypot events to Core
- Report status to Core

**Components**:
- `NodeClient` - Connection management
- `HoneypotManager` - Honeypot lifecycle
- `Event Listener` - Receives events from honeypots (port 9100)
- `Protocol Handler` - Protocol v2 implementation

### HoneyBee Potstore

**Purpose**: Repository of pre-configured honeypots

**Structure**:
- Each honeypot in its own directory
- Standardized installation scripts
- HoneyBee integration plugins
- Configuration templates

**Honeypots**:
- **Cowrie** - SSH/Telnet honeypot (Python)
- **HonnyPotter** - WordPress login honeypot (PHP)
- More coming soon...

## Communication Flow

### Node Registration

```
Node                    Core
 │                       │
 │─── Connect ──────────>│
 │                       │
 │<── Request TOTP ──────│
 │                       │
 │─── TOTP Code ────────>│
 │                       │
 │<── Registration Ack ──│
 │                       │
```

### Honeypot Installation

```
Core                    Node                    Potstore
 │                       │                       │
 │─── InstallPot ───────>│                       │
 │                       │─── Clone ────────────>│
 │                       │<── Repository ────────│
 │                       │                       │
 │                       │─── Install ──────────>│
 │                       │<── Configure ────────│
 │                       │                       │
 │<── Status Update ──────│                       │
 │                       │                       │
```

### Event Flow

```
Honeypot              Node                    Core
 │                      │                       │
 │─── Event ───────────>│                       │
 │   (TCP:9100)         │                       │
 │                      │─── PotEvent ─────────>│
 │                      │   (Protocol v2)       │
 │                      │                       │
```

## Protocol v2

HoneyBee uses Protocol v2 for all communication:

- **Transport**: TCP with optional TLS 1.3
- **Format**: JSON messages wrapped in envelopes
- **Versioning**: Protocol version in envelope
- **Bidirectional**: Both Core and Node can initiate messages

See [Protocol Specification](./protocol/protocol.md) for details.

## Security

### Authentication

- **TOTP**: Time-based one-time passwords for node registration
- **TLS**: Optional TLS 1.3 encryption for all communication

### Authorization

- Nodes authenticate during registration
- Core validates all commands
- Honeypots communicate only with their node

## Scalability

### Horizontal Scaling

- Deploy multiple Core instances (with load balancing)
- Deploy unlimited nodes
- Each node can run multiple honeypots

### Vertical Scaling

- Core: Multi-threaded async (Tokio)
- Node: Lightweight Go binary
- Honeypots: Isolated processes

## Data Flow

1. **Commands**: Core → Node → Honeypot
2. **Events**: Honeypot → Node → Core
3. **Status**: Node → Core
4. **API**: External → Core

## Next Steps

- [Core Architecture](./core/architecture.md) - Deep dive into Core
- [Node Architecture](./node/architecture.md) - Deep dive into Node
- [Protocol Specification](./protocol/protocol.md) - Protocol details

