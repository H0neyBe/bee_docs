# HoneyBee Core Architecture

Technical architecture of HoneyBee Core.

## Overview

HoneyBee Core is built with Rust using the Tokio async runtime for high-performance concurrent operations.

## Components

### Node Manager

Manages all connected nodes:

- **Node Registration**: Handles node connections and registration
- **Status Tracking**: Tracks node and honeypot status
- **Command Distribution**: Sends commands to nodes
- **Event Aggregation**: Collects events from nodes

**Location**: `src/node_manager/manager.rs`

### Backend Manager

Provides external interfaces:

- **REST API**: HTTP API on port 9002
- **WebSocket Proxy**: WebSocket server on port 9003
- **Event Streaming**: Streams events to connected clients

**Location**: `src/backend_manager/manager.rs`

### Message Protocol

Implements Protocol v2:

- **Message Parsing**: Parses JSON Protocol v2 messages
- **Message Routing**: Routes messages to appropriate handlers
- **Version Validation**: Validates protocol version

**Location**: `bee_message/`

## Architecture Diagram

```
┌─────────────────────────────────────────┐
│         HoneyBee Core                    │
│                                          │
│  ┌──────────────────────────────────┐  │
│  │      Node Manager                 │  │
│  │  - Node Registry                  │  │
│  │  - Status Tracking                │  │
│  │  - Command Queue                  │  │
│  └──────────────────────────────────┘  │
│                                          │
│  ┌──────────────────────────────────┐  │
│  │      Backend Manager               │  │
│  │  - REST API (9002)                 │  │
│  │  - WebSocket (9003)                │  │
│  └──────────────────────────────────┘  │
│                                          │
│  ┌──────────────────────────────────┐  │
│  │      Message Protocol             │  │
│  │  - Protocol v2                    │  │
│  │  - JSON Parsing                   │  │
│  └──────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

## Data Flow

### Node Registration

```
Node → Core: NodeRegistration
Core → Node: RegistrationAck
Node → Core: NodeStatusUpdate
```

### Command Distribution

```
Core → Node: NodeCommand
Node → Core: PotStatusUpdate (progress)
Node → Core: PotStatusUpdate (complete)
```

### Event Aggregation

```
Honeypot → Node: Event (TCP:9100)
Node → Core: PotEvent (Protocol v2)
Core → Backend: Event (API/WebSocket)
```

## Concurrency

Core uses Tokio for async operations:

- **Non-blocking I/O**: All network operations are async
- **Concurrent Connections**: Handles multiple nodes simultaneously
- **Message Channels**: Uses channels for inter-component communication

## State Management

- **Node State**: In-memory node registry
- **Honeypot State**: Tracked per node
- **Event Buffer**: Events buffered before forwarding

## Next Steps

- [Overview](./overview.md) - Core overview
- [Configuration](./configuration.md) - Configuration details
- [Deployment](./deployment.md) - Deployment guide

