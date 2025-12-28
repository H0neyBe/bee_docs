# HoneyBee Core Installation

This guide covers installing HoneyBee Core from source or using Docker.

## Prerequisites

- **Rust**: 1.75+ (nightly toolchain)
- **OS**: Linux, Windows, or macOS
- **Network**: Ports 9001, 9002, 9003 available

## Installation Methods

### Method 1: Docker (Recommended)

The easiest way to run HoneyBee Core:

```bash
# Clone repository
git clone https://github.com/H0neyBe/honeybee_core.git
cd honeybee_core

# Build and run
docker-compose up -d

# Check status
docker-compose ps
docker logs honeybee-core
```

### Method 2: Cargo (Development)

For development or custom builds:

```bash
# Clone repository
git clone https://github.com/H0neyBe/honeybee_core.git
cd honeybee_core

# Install Rust nightly (if not already installed)
rustup toolchain install nightly
rustup default nightly

# Build
cargo build --release

# Run
cargo run --release
```

## Configuration

Core uses `bee_config.toml` for configuration:

```toml
[server]
host = "127.0.0.1"        # Bind address
node_port = 9001          # Port for node connections
backend_port = 9002       # Port for backend API
debug = false             # Enable debug mode

[logging]
level = "debug"           # Log level: trace, debug, info, warn, error
folder = "logs"          # Log directory
force_color = true       # Force colored output

[proxy]
enabled = true           # Enable WebSocket proxy
host = "0.0.0.0"        # Proxy bind address
port = 9003             # Proxy port
```

## Verification

### Check if Core is Running

```bash
# Check process
ps aux | grep honeybee_core

# Or check Docker
docker ps | grep honeybee-core

# Check logs
docker logs honeybee-core
# Or
tail -f logs/debug.log
```

### Test Node Port

```bash
# Test node connection port
telnet localhost 9001

# Or with netcat
nc -zv localhost 9001
```

### Test Backend API

```bash
# Health check
curl http://localhost:9002/health

# Or check API endpoint
curl http://localhost:9002/api/v1/nodes
```

## Running with Tracing

For performance analysis:

```bash
cargo run --release --features tracing
```

Then connect with Tracy profiler.

## Next Steps

- [Configuration](./configuration.md) - Detailed configuration options
- [Deployment](./deployment.md) - Production deployment
- [Architecture](./architecture.md) - Technical architecture

