# Installation Overview

This guide covers installing all HoneyBee components.

## Component Installation

### HoneyBee Core

The central orchestration manager.

**Options:**
- [Docker](./core/deployment-docker.md) - Recommended for production
- [Cargo](./core/installation.md) - For development

**Requirements:**
- Rust 1.75+ (nightly)
- Linux, Windows, or macOS

### HoneyBee Node

Individual honeypot nodes.

**Options:**
- [Pre-built Binary](./node/installation.md) - Download from releases
- [Build from Source](./node/installation.md) - Compile yourself
- [Docker](./node/deployment-docker.md) - Container deployment

**Requirements:**
- Go 1.21+ (for building)
- Static binary (no runtime dependencies)
- Linux, Windows, or macOS

### HoneyBee Potstore

Honeypot repository (automatically cloned by nodes).

**No installation required** - Nodes automatically clone from GitHub.

## Installation Order

1. **Install HoneyBee Core** first
2. **Install HoneyBee Nodes** and connect to Core
3. **Honeypots** are installed automatically via Core commands

## System Requirements

### Minimum Requirements

- **CPU**: 1 core
- **RAM**: 512 MB
- **Disk**: 1 GB free space
- **Network**: TCP connectivity between Core and Nodes

### Recommended Requirements

- **CPU**: 2+ cores
- **RAM**: 2 GB
- **Disk**: 10 GB free space (for honeypots)
- **Network**: Low latency (< 100ms)

### For Honeypots

- **Python 3.7+** (for Python-based honeypots like Cowrie)
- **PHP 7.4+** (for PHP-based honeypots like HonnyPotter)
- **Git** (for cloning Potstore)

## Platform-Specific Notes

### Linux

All components work natively on Linux. Systemd service files available.

### Windows

- Core: Requires Rust nightly
- Node: Pre-built Windows binaries available
- Honeypots: Python/PHP must be installed

### macOS

- Core: Requires Rust nightly
- Node: Pre-built macOS binaries available
- Honeypots: Python/PHP via Homebrew

## Verification

After installation, verify each component:

### Verify Core

```bash
# Check if Core is running
curl http://localhost:9002/health

# Or check logs
docker logs honeybee-core
```

### Verify Node

```bash
# Check node status
./build/honeybee-node -version

# Validate configuration
./build/honeybee-node -config configs/config.yaml -validate
```

### Verify Honeypots

Honeypots are verified automatically when installed. Check node logs for installation status.

## Next Steps

- [Quick Start Guide](./quickstart.md) - Get started quickly
- [Core Installation](./core/installation.md) - Detailed Core setup
- [Node Installation](./node/installation.md) - Detailed Node setup
- [Configuration](./core/configuration.md) - Configure components

