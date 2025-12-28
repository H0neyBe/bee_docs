# Testing Guide

Testing HoneyBee components.

## Testing HoneyBee Core

### Unit Tests

```bash
cd honeybee_core
cargo test
```

### Integration Tests

```bash
# Run with test configuration
cargo test --test integration
```

## Testing HoneyBee Node

### Unit Tests

```bash
cd honeybee_node
make test
```

### Integration Tests

```bash
# Test with mock Core
go test ./internal/... -v
```

## Testing Honeypots

### Test Cowrie

```bash
# Install Cowrie
cd honeybee_potstore/cowrie
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Start Cowrie
bin/cowrie start

# Test SSH connection
ssh -p 2222 admin@localhost
```

### Test HonnyPotter

```bash
# Start PHP server
cd honeybee_potstore/HonnyPotter
php -S 0.0.0.0:8080 standalone.php

# Test login
curl -X POST http://localhost:8080/standalone.php \
  -d "log=admin&pwd=password123"
```

## End-to-End Testing

1. Start HoneyBee Core
2. Start HoneyBee Node
3. Verify node registration
4. Install honeypot
5. Generate test attack
6. Verify event forwarding

## Next Steps

- [Building from Source](./building.md) - Build components
- [Contributing](./contributing.md) - Contribution guidelines

