# Introduction to HoneyBee

Welcome to the HoneyBee documentation! HoneyBee is a distributed honeypot orchestration framework that allows you to deploy, manage, and monitor multiple honeypot nodes from a central manager.

## What is HoneyBee?

HoneyBee is a comprehensive honeypot management platform consisting of three main components:

1. **HoneyBee Core** - The central orchestration manager (written in Rust)
2. **HoneyBee Node** - Individual honeypot nodes that connect to the manager (Go implementation)
3. **HoneyBee Potstore** - Repository of pre-configured honeypots ready for deployment

## Key Features

- **🔐 Secure by Design**: TLS 1.3 encryption and TOTP authentication
- **🌐 Distributed**: Manage multiple nodes from a central location
- **📡 Real-time Communication**: JSON-based Protocol v2 over TCP/TLS
- **🔄 Resilient**: Automatic reconnection and error handling
- **📊 Observable**: Comprehensive logging and monitoring
- **🍯 Honeypot Management**: Automatic installation and lifecycle management
- **🚀 Cross-Platform**: Linux, Windows, and macOS support
- **🧪 Beta Status**: Actively developed and tested

## System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              HoneyBee Core Manager                       │
│              (Rust - TCP Server)                         │
│                                                           │
│  - Node Registry & Management                            │
│  - Message Routing (Protocol v2)                         │
│  - Status Tracking                                        │
│  - Backend API (Port 9002)                               │
│  - WebSocket Proxy (Port 9003)                           │
└──────────────┬──────────────────────────────────────────┘
               │
               │ TCP/TLS + Protocol v2
               │ Port 9001
               │
       ┌───────┴────────┬──────────────┬──────────────┐
       │                │              │              │
   ┌───▼───┐        ┌───▼───┐     ┌───▼───┐     ┌───▼───┐
   │ Node  │        │ Node  │     │ Node  │     │ Node  │
   │  (Go) │        │  (Go) │     │  (Go) │     │  (Go) │
   └───┬───┘        └───┬───┘     └───┬───┘     └───┬───┘
       │                │              │              │
       │                │              │              │
   ┌───▼───┐        ┌───▼───┐     ┌───▼───┐     ┌───▼───┐
   │Cowrie │        │HonnyP │     │Cowrie │     │Dionaea│
   │Pot    │        │otter  │     │Pot    │     │Pot    │
   └───────┘        └───────┘     └───────┘     └───────┘
```

## Component Overview

### HoneyBee Core

The central manager that orchestrates all nodes and honeypots:

- **Node Registration**: Nodes connect and register with TOTP authentication
- **Command Distribution**: Send commands to nodes (install, start, stop honeypots)
- **Status Monitoring**: Track node and honeypot status in real-time
- **Event Aggregation**: Collect events from all honeypots
- **Backend API**: RESTful API for external integrations
- **WebSocket Proxy**: Real-time updates via WebSocket

**Repository**: [honeybee_core](https://github.com/H0neyBe/honeybee_core)

### HoneyBee Node

Go-based nodes that connect to the Core manager:

- **Connection Management**: Automatic reconnection with exponential backoff
- **Honeypot Installation**: Automatically install honeypots from Potstore
- **Honeypot Lifecycle**: Start, stop, restart, and monitor honeypots
- **Event Forwarding**: Forward honeypot events to Core in real-time
- **TLS 1.3 Encryption**: Secure communication with the manager
- **TOTP Authentication**: Time-based one-time password support

**Repository**: [honeybee_node](https://github.com/H0neyBe/honeybee_node)

### HoneyBee Potstore

Repository of pre-configured honeypots:

- **Pre-configured Honeypots**: Cowrie, HonnyPotter, and more
- **Automatic Integration**: Honeypots automatically forward events to nodes
- **Easy Installation**: Nodes automatically install from Potstore
- **Standardized Format**: Consistent structure and configuration

**Repository**: [honeybee_potstore](https://github.com/H0neyBe/honeybee_potstore)

## How It Works

1. **Deploy HoneyBee Core** - Start the central manager
2. **Deploy HoneyBee Nodes** - Nodes connect to Core and register
3. **Install Honeypots** - Core sends commands to nodes to install honeypots from Potstore
4. **Monitor Attacks** - Honeypots capture attacks and forward events to Core
5. **Analyze Data** - Core aggregates all events for analysis

## Quick Links

### For New Users

- [Quick Start Guide](./getting-started/quickstart.md) - Get your first deployment running
- [Installation Overview](./getting-started/installation.md) - Install all components
- [Architecture Overview](./getting-started/architecture.md) - Understand the system

### For Node Operators

- [Node Installation](./node/installation.md) - Set up a HoneyBee node
- [Node Configuration](./node/configuration.md) - Configure your node
- [Security Setup](./node/security.md) - Set up TLS and TOTP
- [Honeypot Management](./node/honeypot-management.md) - Manage honeypots

### For System Administrators

- [Core Deployment](./core/deployment.md) - Deploy HoneyBee Core
- [Node Deployment](./node/deployment.md) - Deploy nodes in production
- [Troubleshooting](./node/troubleshooting.md) - Common issues and solutions

### For Developers

- [Protocol Specification](./protocol/protocol.md) - Message format and flow
- [Creating Custom Nodes](./development/custom-nodes.md) - Build nodes in any language
- [Adding Honeypots](./potstore/adding.md) - Add honeypots to Potstore
- [API Reference](./reference/api.md) - Core API documentation

## Status

**Current Version**: Beta (v1.0.0)  
**Protocol Version**: v2  
**Status**: Actively developed and tested

## Getting Help

- 📖 Browse this documentation
- 🐛 [Report issues](https://github.com/H0neyBe/honeybee_node/issues)
- 💬 [Join discussions](https://github.com/H0neyBe/honeybee_node/discussions)
- 📚 [Documentation Site](https://h0neybe.github.io/bee_docs/)

## License

HoneyBee is open-source software licensed under MIT. See the LICENSE files in each repository for details.
