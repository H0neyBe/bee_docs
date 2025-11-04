# Installation & Quick Start

Get your HoneyBee node running in 5 minutes!

## Prerequisites

Choose one of the following:

### Option 1: Pre-built Binary (Easiest)
- Download from releases page

### Option 2: Build from Source
- Go 1.21 or later
- Git
- Make (optional but recommended)

### Option 3: Docker
- Docker 20.10+
- Docker Compose (optional)

## Quick Start (Development)

### Step 1: Get the Binary

#### Build from Source

```bash
# Clone the repository
git clone https://github.com/yourusername/honeybee.git
cd honeybee/honeybee_node

# Download dependencies
go mod download

# Build
make build
# or: go build -o build/honeybee-node ./cmd/node
```

#### Download Pre-built Binary

```bash
# Download latest release
wget https://github.com/yourusername/honeybee/releases/latest/download/honeybee-node-linux-amd64

# Make executable
chmod +x honeybee-node-linux-amd64
mv honeybee-node-linux-amd64 honeybee-node
```

### Step 2: Generate Configuration

```bash
# Create configs directory
mkdir -p configs

# Generate default configuration
./build/honeybee-node -gen-config
```

This creates `configs/config.yaml` with default settings.

### Step 3: Configure for Testing

For local testing, edit `configs/config.yaml`:

```yaml
node:
  name: "test-node-01"
  type: "Agent"
  address: "0.0.0.0"
  port: 8080

server:
  address: "127.0.0.1:9001"  # Your manager address
  heartbeat_interval: 30
  reconnect_delay: 5

tls:
  enabled: false  # ⚠️ ONLY for local testing!
  # Never disable in production

auth:
  totp_enabled: false  # ⚠️ ONLY for local testing!
  # Never disable in production

log:
  level: "debug"
  format: "text"
```

> **⚠️ Warning**: This configuration is for testing only. For production, see [Security Setup](./security.md).

### Step 4: Start the Manager

First, ensure the HoneyBee Core manager is running:

```bash
# In a separate terminal
cd ../honeybee_core
cargo run
```

The manager should output:
```
Node Manager listening on: 127.0.0.1:9001
```

### Step 5: Run the Node

```bash
./build/honeybee-node -config configs/config.yaml
```

You should see:
```
 _   _                       ____             
| | | | ___  _ __   ___ _   | __ )  ___  ___ 
| |_| |/ _ \| '_ \ / _ \ | | |  _ \ / _ \/ _ \
|  _  | (_) | | | |  __/ |_| | |_) |  __/  __/
|_| |_|\___/|_| |_|\___|\__, |____/ \___|\___|
                        |___/  Node v1.0.0

INFO[...] HoneyBee Node v1.0.0 starting...
INFO[...] Connected to server successfully
INFO[...] Registration accepted
INFO[...] Heartbeat sent
```

🎉 **Success!** Your node is connected.

## Installation Options

### 1. System-Wide Installation

```bash
# Build the binary
make build

# Install to /usr/local/bin
sudo cp build/honeybee-node /usr/local/bin/
sudo chmod +x /usr/local/bin/honeybee-node

# Verify installation
honeybee-node -version
```

### 2. User Installation

```bash
# Build the binary
make build

# Install to user bin directory
mkdir -p ~/bin
cp build/honeybee-node ~/bin/
chmod +x ~/bin/honeybee-node

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/bin:$PATH"

# Verify installation
honeybee-node -version
```

### 3. Docker Installation

```bash
# Build Docker image
cd honeybee_node
docker build -t honeybee-node:latest .

# Run container
docker run -d \
  --name honeybee-node \
  -v $(pwd)/configs/config.yaml:/app/configs/config.yaml \
  honeybee-node:latest
```

### 4. Kubernetes Installation

```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/

# Check pod status
kubectl get pods -l app=honeybee-node

# View logs
kubectl logs -f -l app=honeybee-node
```

## Verifying Installation

### Check Version

```bash
honeybee-node -version
# Output: HoneyBee Node v1.0.0
```

### Test Connection

```bash
# Start node in foreground
honeybee-node -config configs/config.yaml

# Check logs for:
# ✅ "Connected to server successfully"
# ✅ "Registration accepted"
# ✅ "Heartbeat sent"
```

### Verify Manager Side

On the manager, check logs for:
```
INFO: Node <id> (<name>) registered from <address>
```

## Configuration Files

After installation, you'll have:

```
~/.config/honeybee/
├── .honeybee_totp_secret    # TOTP secret (generated on first run)
└── config.yaml              # Optional user config

/etc/honeybee/               # System-wide (production)
├── config.yaml              # Main configuration
├── certs/                   # TLS certificates
│   ├── ca.crt
│   ├── client.crt
│   └── client.key
└── secrets/                 # TOTP secrets

/var/log/honeybee/           # Logs
└── node.log
```

## Post-Installation

### Set Up Security

⚠️ **Before production deployment**, you must:

1. ✅ Enable TLS encryption - [TLS Setup Guide](./tls.md)
2. ✅ Enable TOTP authentication - [TOTP Setup Guide](./totp.md)
3. ✅ Configure firewall rules
4. ✅ Run as non-root user
5. ✅ Set up log rotation

See the complete [Security Setup Guide](./security.md).

### Configure for Your Environment

Edit your configuration file to match your setup:

```yaml
node:
  name: "prod-honeypot-01"      # Unique name
  type: "Agent"                  # or "Full"

server:
  address: "manager.example.com:9001"  # Your manager

tls:
  enabled: true                  # ✅ Required
  ca_file: "/etc/honeybee/certs/ca.crt"
  cert_file: "/etc/honeybee/certs/client.crt"
  key_file: "/etc/honeybee/certs/client.key"

auth:
  totp_enabled: true            # ✅ Required
```

See [Configuration Guide](./configuration.md) for all options.

### Set Up as a Service

For production, run as a systemd service:

```bash
# Copy service file
sudo cp systemd/honeybee-node.service /etc/systemd/system/

# Enable and start
sudo systemctl enable honeybee-node
sudo systemctl start honeybee-node

# Check status
sudo systemctl status honeybee-node
```

See [Systemd Deployment](./deployment-systemd.md) for details.

## Next Steps

1. ✅ Node installed and running
2. 📖 [Configure your node](./configuration.md)
3. 🔐 [Set up security](./security.md)
4. 🚀 [Deploy to production](./deployment.md)

## Troubleshooting Installation

### Build Fails

```bash
# Update Go
go version  # Should be 1.21+

# Clean and rebuild
make clean
make deps
make build
```

### Connection Issues

```bash
# Verify manager is running
nc -zv 127.0.0.1 9001

# Check firewall
sudo ufw status

# Test network
ping manager.example.com
```

### Permission Issues

```bash
# Fix binary permissions
chmod +x honeybee-node

# Fix config permissions
chmod 644 configs/config.yaml

# Fix TOTP secret directory
chmod 700 ~/.config/honeybee
```

For more help, see [Troubleshooting Guide](./troubleshooting.md).

