# HoneyBee Documentation

Welcome to the HoneyBee documentation! HoneyBee is a distributed honeypot orchestration framework that allows you to deploy and manage multiple honeypot nodes from a central manager.

## What is HoneyBee?

HoneyBee consists of two main components:

1. **HoneyBee Core** - The central orchestration manager (written in Rust)
2. **HoneyBee Nodes** - Individual honeypot instances that connect to the manager

## Key Features

- **🔐 Secure by Design**: TLS 1.3 encryption and TOTP authentication
- **🌐 Distributed**: Manage multiple nodes from a central location
- **📡 Real-time Communication**: JSON-based protocol over TCP
- **🔄 Resilient**: Automatic reconnection and error handling
- **📊 Observable**: Comprehensive logging and monitoring
- **🚀 Production-Ready**: Battle-tested architecture

## Architecture Overview

```
┌─────────────────────────────────────────────┐
│         HoneyBee Core Manager               │
│         (Rust - TCP Server)                 │
│                                             │
│  - Node Registry                            │
│  - Message Routing                          │
│  - Status Tracking                          │
└──────────────┬──────────────────────────────┘
               │
               │ TCP/TLS + Protocol v2
               │
       ┌───────┴────────┬──────────────┐
       │                │              │
   ┌───▼───┐        ┌───▼───┐     ┌───▼───┐
   │ Node  │        │ Node  │     │ Node  │
   │  (Go) │        │(Python)│    │ (Rust)│
   └───────┘        └───────┘     └───────┘
```

## Quick Links

### For Node Operators

- [Quick Start Guide](./node/installation.md) - Get your first node running
- [Configuration Guide](./node/configuration.md) - Configure your node
- [Security Setup](./node/security.md) - Set up TLS and TOTP

### For Developers

- [Protocol Specification](./protocol.md) - Message format and flow
- [Creating Custom Nodes](./creating_nodes.md) - Build nodes in any language
- [Architecture](./node/architecture.md) - Technical deep dive

### For System Administrators

- [Deployment Guide](./node/deployment.md) - Production deployment
- [Troubleshooting](./node/troubleshooting.md) - Common issues and solutions

## Getting Help

- 📖 Browse this documentation
- 🐛 [Report issues](https://github.com/yourusername/honeybee)
- 💬 [Community discussions](https://github.com/yourusername/honeybee/discussions)

## License

HoneyBee is open-source software. See the LICENSE file for details.

