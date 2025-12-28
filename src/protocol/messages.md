# Message Types

Complete reference for all HoneyBee Protocol v2 message types.

## Message Envelope

All messages are wrapped in a versioned envelope:

```json
{
  "version": 2,
  "message": {
    // One of the message types below
  }
}
```

## Node → Core Messages

### NodeRegistration

Register a new node with the Core.

**Fields**:
- `node_id` (uint64) - Unique node identifier
- `node_name` (string) - Human-readable node name
- `address` (string) - Node's reported address
- `port` (uint16) - Node's reported port
- `node_type` (string) - "Full" or "Agent"
- `totp_code` (string, optional) - TOTP code for authentication

**Example**:
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

### NodeStatusUpdate

Update node status.

**Fields**:
- `node_id` (uint64) - Node identifier
- `status` (string) - Status: "Connected", "Deploying", "Running", "Stopped", "Failed", "Unknown"

**Example**:
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

### PotStatusUpdate

Update honeypot (pot) status.

**Fields**:
- `node_id` (uint64) - Node identifier
- `pot_id` (string) - Honeypot instance ID
- `pot_type` (string) - Honeypot type (cowrie, honnypotter, etc.)
- `status` (string) - Status: "Installing", "Running", "Stopped", "Failed"
- `message` (string, optional) - Status message

**Example**:
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

### PotEvent

Honeypot event (attack data).

**Fields**:
- `node_id` (uint64) - Node identifier
- `pot_id` (string) - Honeypot instance ID
- `pot_type` (string) - Honeypot type
- `event_type` (string) - Event type
- `timestamp` (string) - ISO 8601 timestamp
- `data` (object) - Event-specific data

**Example**:
```json
{
  "version": 2,
  "message": {
    "PotEvent": {
      "node_id": 12345,
      "pot_id": "cowrie-01",
      "pot_type": "cowrie",
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

### NodeEvent

General node event.

**Fields**:
- `type` (string) - Event type: "Started", "Stopped", "Error"
- `message` (string, optional) - Event message

**Example**:
```json
{
  "version": 2,
  "message": {
    "NodeEvent": {
      "type": "Started",
      "message": "Node started successfully"
    }
  }
}
```

### NodeDrop

Notify Core that node is disconnecting.

**Fields**: None

**Example**:
```json
{
  "version": 2,
  "message": {
    "NodeDrop": {}
  }
}
```

## Core → Node Messages

### RegistrationAck

Registration acknowledgment.

**Fields**:
- `accepted` (bool) - Whether registration was accepted
- `message` (string, optional) - Acknowledgment message
- `totp_key` (string, optional) - TOTP secret key (only on first registration)

**Example**:
```json
{
  "version": 2,
  "message": {
    "RegistrationAck": {
      "accepted": true,
      "message": "Registration successful",
      "totp_key": "BASE32SECRET"
    }
  }
}
```

### NodeCommand

Command to node.

**Fields**:
- `node_id` (uint64) - Target node identifier
- `command` (object) - Command object (see [Command Reference](./commands.md))

**Example**:
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
          "auto_start": true
        }
      }
    }
  }
}
```

## Message Ordering

Messages are sent in order, but Core and Node can send messages concurrently.

## Error Handling

If a message is invalid or cannot be processed:

1. Log the error
2. Send error response (if applicable)
3. Continue operation (don't disconnect)

## Next Steps

- [Protocol Specification](./protocol.md) - Protocol overview
- [Command Reference](./commands.md) - Available commands
- [Event Format](./events.md) - Event structure

