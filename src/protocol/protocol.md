# Protocol v2 Specification

HoneyBee Protocol v2 is the communication protocol between HoneyBee Core and HoneyBee Nodes.

## Overview

- **Transport**: TCP with optional TLS 1.3
- **Format**: JSON messages wrapped in versioned envelopes
- **Version**: 2
- **Bidirectional**: Both Core and Node can initiate messages

## Message Envelope

All messages are wrapped in a versioned envelope:

```json
{
  "version": 2,
  "message": {
    // Message content
  }
}
```

## Protocol Version

Current protocol version is **2**. Version is included in every message envelope.

## Message Types

### Node → Core Messages

#### NodeRegistration

Register a new node with the Core.

```json
{
  "version": 2,
  "message": {
    "NodeRegistration": {
      "node_id": 12345,
      "node_name": "my-node",
      "address": "0.0.0.0",
      "port": 8080,
      "node_type": "Full",
      "totp_code": "123456"
    }
  }
}
```

#### NodeStatusUpdate

Update node status.

```json
{
  "version": 2,
  "message": {
    "NodeStatusUpdate": {
      "node_id": 12345,
      "status": "Running"
    }
  }
}
```

#### PotStatusUpdate

Update honeypot (pot) status.

```json
{
  "version": 2,
  "message": {
    "PotStatusUpdate": {
      "node_id": 12345,
      "pot_id": "cowrie-01",
      "pot_type": "cowrie",
      "status": "Running",
      "message": "Honeypot started successfully"
    }
  }
}
```

#### PotEvent

Honeypot event (attack data).

```json
{
  "version": 2,
  "message": {
    "PotEvent": {
      "node_id": 12345,
      "pot_id": "cowrie-01",
      "event_type": "login",
      "timestamp": "2024-01-15T10:30:00Z",
      "data": {
        "username": "admin",
        "password": "password123",
        "ip": "192.168.1.100"
      }
    }
  }
}
```

### Core → Node Messages

#### RegistrationAck

Registration acknowledgment.

```json
{
  "version": 2,
  "message": {
    "RegistrationAck": {
      "accepted": true,
      "message": "Registration successful",
      "totp_key": "BASE32SECRET"  // Only on first registration
    }
  }
}
```

#### NodeCommand

Command to node.

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
            "ssh_port": "2222"
          },
          "auto_start": true
        }
      }
    }
  }
}
```

## Node Status Values

- `Connected` - Node is connected
- `Deploying` - Node is being deployed
- `Running` - Node is active
- `Stopped` - Node is stopped
- `Failed` - Node encountered an error
- `Unknown` - Status cannot be determined

## Pot Status Values

- `Installing` - Honeypot is being installed
- `Running` - Honeypot is active
- `Stopped` - Honeypot is stopped
- `Failed` - Installation or startup failed

## Node Types

- `Full` - Full-featured honeypot node
- `Agent` - Lightweight monitoring agent

## Commands

See [Command Reference](./commands.md) for all available commands.

## Events

See [Event Format](./events.md) for event structure and types.

## Security

### TLS Encryption

Optional TLS 1.3 encryption for all communication.

### TOTP Authentication

Time-based one-time password for node registration.

## Next Steps

- [Message Types](./messages.md) - Detailed message specifications
- [Command Reference](./commands.md) - All available commands
- [Event Format](./events.md) - Event structure and types

