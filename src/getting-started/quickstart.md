# Quick Start Guide

This guide will help you get HoneyBee up and running quickly. We'll deploy HoneyBee Core and connect a HoneyBee Node.

## Prerequisites

- **HoneyBee Core**: Rust 1.75+ (nightly)
- **HoneyBee Node**: Go 1.21+
- **For Honeypots**: Python 3.7+ or PHP 7.4+, Git

## Step 1: Deploy HoneyBee Core

### Using Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/H0neyBe/honeybee_core.git
cd honeybee_core

# Build and run with Docker Compose
docker-compose up -d

# Check status
docker-compose ps
```

The Core will be available on:
- **Node Port**: 9001 (for node connections)
- **Backend API**: 9002 (for API access)
- **WebSocket Proxy**: 9003 (for WebSocket connections)

### Using Cargo (Development)

```bash
# Clone the repository
git clone https://github.com/H0neyBe/honeybee_core.git
cd honeybee_core

# Build
cargo build --release

# Run
cargo run --release
```

## Step 2: Deploy HoneyBee Node

### Using Pre-built Binary

```bash
# Download from releases (when available)
# Or build from source (see below)
```

### Building from Source

```bash
# Clone the repository
git clone https://github.com/H0neyBe/honeybee_node.git
cd honeybee_node

# Build
make build

# Generate configuration
./build/honeybee-node -gen-config
```

### Configure the Node

Edit `configs/config.yaml`:

```yaml
node:
  name: "my-first-node"
  type: "Full"

server:
  address: "localhost:9001"  # Core manager address

tls:
  enabled: false  # Set to true in production

auth:
  totp_enabled: false  # Set to true in production

honeypot:
  enabled: true
  base_dir: "~/.honeybee/honeypots"
```

### Run the Node

```bash
./build/honeybee-node -config configs/config.yaml
```

You should see:
```
Starting HoneyBee Node...
Connected to server successfully
Registration accepted
Node started successfully, running...
```

## Step 3: Verify Connection

Check the Core logs to see the node registration:

```
Node registered: my-first-node (ID: 12345)
Node status: Running
```

## Step 4: Install a Honeypot

Once the node is connected, you can install honeypots. This is typically done via the Core's API or CLI.

### Example: Install Cowrie

The Core will send an `InstallPot` command to the node:

```json
{
  "version": 2,
  "message": {
    "NodeCommand": {
      "node_id": 12345,
      "command": {
        "InstallPot": {
          "pot_id": "cowrie-01",
          "honeypot_type": "cowrie",
          "config": {
            "ssh_port": "2222",
            "telnet_port": "2223"
          },
          "auto_start": true
        }
      }
    }
  }
}
```

The node will:
1. Clone the Potstore repository
2. Install Cowrie with dependencies
3. Configure HoneyBee integration
4. Start the honeypot automatically

## Step 5: Monitor Events

Once a honeypot is running, it will forward events to the Core. You can monitor these events via:

- **Core Logs**: Check Core console output
- **Backend API**: Query events via REST API (port 9002)
- **WebSocket**: Real-time updates via WebSocket (port 9003)

## Next Steps

- [Core Configuration](./core/configuration.md) - Configure HoneyBee Core
- [Node Configuration](./node/configuration.md) - Configure your node
- [Security Setup](./node/security.md) - Enable TLS and TOTP
- [Honeypot Management](./node/honeypot-management.md) - Manage honeypots
- [Protocol Documentation](./protocol/protocol.md) - Understand the protocol

## Troubleshooting

### Node Won't Connect

1. Verify Core is running: `curl http://localhost:9002/health`
2. Check Core address in node config
3. Check firewall rules
4. Enable debug logging: `./build/honeybee-node -config configs/config.yaml -debug`

### Honeypot Installation Fails

1. Ensure Python/PHP is installed
2. Check disk space
3. Verify Git is available
4. Review node logs for errors

See [Troubleshooting Guide](./node/troubleshooting.md) for more help.

