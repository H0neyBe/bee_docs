# Building from Source

Build HoneyBee components from source.

## HoneyBee Core

### Prerequisites

- Rust 1.75+ (nightly toolchain)
- Cargo

### Build Steps

```bash
# Clone repository
git clone https://github.com/H0neyBe/honeybee_core.git
cd honeybee_core

# Install Rust nightly
rustup toolchain install nightly
rustup default nightly

# Build
cargo build --release

# Binary location
target/release/honeybee_core
```

### Build with Features

```bash
# Build with tracing support
cargo build --release --features tracing
```

## HoneyBee Node

### Prerequisites

- Go 1.21+

### Build Steps

```bash
# Clone repository
git clone https://github.com/H0neyBe/honeybee_node.git
cd honeybee_node

# Build
make build

# Or manually
go build -o build/honeybee-node ./cmd/node
```

### Cross-Platform Builds

```bash
# Linux
make build-linux

# Windows
make build-windows

# macOS
make build-darwin

# All platforms
make build-all
```

## HoneyBee Potstore

No build required - it's a repository of honeypots.

## Development Mode

### Core

```bash
# Run with debug logging
RUST_LOG=debug cargo run

# Run with tracing
cargo run --features tracing
```

### Node

```bash
# Run in development mode
make dev

# Or manually
go run ./cmd/node -config configs/config.yaml -debug
```

## Next Steps

- [Contributing](./contributing.md) - Contribution guidelines
- [Testing](./testing.md) - Testing guide

