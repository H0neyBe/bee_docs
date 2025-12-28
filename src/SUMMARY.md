# Summary

[Introduction](./intro.md)

# Getting Started

- [Quick Start Guide](./getting-started/quickstart.md)
- [Installation Overview](./getting-started/installation.md)
- [Architecture Overview](./getting-started/architecture.md)

# HoneyBee Core

The central orchestration manager written in Rust.

- [Overview](./core/overview.md)
- [Installation](./core/installation.md)
- [Configuration](./core/configuration.md)
- [Deployment](./core/deployment.md)
  - [Docker](./core/deployment-docker.md)
  - [Systemd](./core/deployment-systemd.md)
- [Architecture](./core/architecture.md)
- [API & CLI](./core/api.md)

# HoneyBee Node

The Go implementation of HoneyBee nodes.

- [Overview](./node/overview.md)
- [Installation](./node/installation.md)
- [Configuration](./node/configuration.md)
- [Node Types](./node/node-types.md)
- [Security](./node/security.md)
  - [TLS Setup](./node/tls.md)
  - [TOTP Setup](./node/totp.md)
- [Deployment](./node/deployment.md)
  - [Systemd Service](./node/deployment-systemd.md)
  - [Docker](./node/deployment-docker.md)
  - [Windows Service](./node/deployment-windows.md)
- [Honeypot Management](./node/honeypot-management.md)
- [Architecture](./node/architecture.md)
- [Examples](./node/examples.md)
- [Troubleshooting](./node/troubleshooting.md)

# HoneyBee Potstore

The honeypot repository and registry.

- [Overview](./potstore/overview.md)
- [Available Honeypots](./potstore/honeypots.md)
- [Using Honeypots](./potstore/using.md)
- [Adding New Honeypots](./potstore/adding.md)
- [Honeypot Integration](./potstore/integration.md)

# Protocol

- [Protocol v2 Specification](./protocol/protocol.md)
- [Message Types](./protocol/messages.md)
- [Command Reference](./protocol/commands.md)
- [Event Format](./protocol/events.md)

# Reference

- [Configuration Reference](./reference/config.md)
- [API Reference](./reference/api.md)
- [Error Codes](./reference/errors.md)

# Development

- [Building from Source](./development/building.md)
- [Contributing](./development/contributing.md)
- [Creating Custom Nodes](./development/custom-nodes.md)
- [Testing](./development/testing.md)
