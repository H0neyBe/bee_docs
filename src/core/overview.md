# HoneyBee Core Overview

HoneyBee Core is the central orchestration manager for the HoneyBee platform. Written in Rust, it manages all nodes, distributes commands, and aggregates events from honeypots.

## What is HoneyBee Core?

HoneyBee Core is a distributed honeypot management system that allows you to:

- **Orchestrate Nodes**: Manage multiple honeypot nodes from a central location
- **Deploy Honeypots**: Install and manage honeypots across all nodes
- **Monitor Status**: Track node and honeypot health in real-time
- **Aggregate Events**: Collect attack data from all honeypots
- **Provide API**: RESTful API and WebSocket for external integrations

## Key Features

- **🚀 High Performance**: Built with Rust and Tokio async runtime
- **🔐 Secure**: TLS 1.3 support and TOTP authentication
- **📡 Real-time**: WebSocket proxy for live updates
- **🌐 Distributed**: Manage nodes across networks
- **📊 Observable**: Comprehensive logging and monitoring
- **🔌 Extensible**: Backend API for custom integrations

## Architecture

Core consists of three main components:

1. **Node Manager** - Handles node registration and communication
2. **Backend Manager** - Provides REST API and WebSocket proxy
3. **Message Protocol** - Protocol v2 implementation

## Ports

- **9001** - Node connections (TCP/TLS)
- **9002** - Backend API (HTTP)
- **9003** - WebSocket proxy (WS)

## Components

### Node Manager

Manages all connected nodes:

- Node registration with TOTP
- Status tracking
- Command distribution
- Event aggregation

### Backend Manager

Provides external interfaces:

- REST API for programmatic access
- WebSocket proxy for real-time updates
- Event streaming

### Message Protocol

Implements Protocol v2:

- JSON message format
- Versioned envelopes
- Bidirectional communication

## Quick Start

```bash
# Using Docker
docker-compose up -d

# Or using Cargo
cargo run --release
```

## Configuration

Core configuration is in `bee_config.toml`:

```toml
[server]
host = "127.0.0.1"
node_port = 9001
backend_port = 9002
debug = false

[logging]
level = "debug"
folder = "logs"
force_color = true

[proxy]
enabled = true
host = "0.0.0.0"
port = 9003
```

## Next Steps

- [Installation](./installation.md) - Install HoneyBee Core
- [Configuration](./configuration.md) - Configure Core
- [Deployment](./deployment.md) - Deploy in production
- [Architecture](./architecture.md) - Technical details

