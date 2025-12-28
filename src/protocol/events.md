# Event Format

Honeypot events forwarded from nodes to Core.

## Event Structure

Events are sent as `PotEvent` messages:

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
        // Event-specific data
      }
    }
  }
}
```

## Event Types

### Cowrie Events

#### Login Attempt

```json
{
  "event_type": "cowrie.login.success",
  "data": {
    "username": "admin",
    "password": "password123",
    "src_ip": "192.168.1.100",
    "src_port": 54321,
    "dst_port": 2222,
    "session": "abc123"
  }
}
```

#### Command Execution

```json
{
  "event_type": "cowrie.command.input",
  "data": {
    "input": "ls -la",
    "session": "abc123",
    "src_ip": "192.168.1.100"
  }
}
```

#### File Download

```json
{
  "event_type": "cowrie.session.file_download",
  "data": {
    "url": "http://example.com/malware.sh",
    "outfile": "/tmp/malware.sh",
    "session": "abc123",
    "src_ip": "192.168.1.100"
  }
}
```

### HonnyPotter Events

#### Login Attempt

```json
{
  "event_type": "honnypotter.login.failed",
  "data": {
    "username": "admin",
    "password": "password123",
    "ip": "192.168.1.100",
    "user_agent": "Mozilla/5.0",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

#### XML-RPC Attack

```json
{
  "event_type": "honnypotter.xmlrpc.attack",
  "data": {
    "method": "wp.getUsersBlogs",
    "username": "admin",
    "password": "password123",
    "ip": "192.168.1.100"
  }
}
```

## Event Forwarding

### From Honeypot to Node

Honeypots send events to node via TCP socket:

- **Port**: 9100 (configurable via `HONEYBEE_EVENT_PORT`)
- **Format**: JSON (one event per line)
- **Protocol**: TCP

### From Node to Core

Nodes forward events to Core via Protocol v2:

- **Message Type**: `PotEvent`
- **Format**: JSON in Protocol v2 envelope
- **Protocol**: TCP/TLS with Protocol v2

## Event Flow

```
Honeypot → Node (TCP:9100) → Core (Protocol v2)
```

## Timestamps

All events include ISO 8601 timestamps:

```
2024-01-15T10:30:00Z
```

## Event Metadata

All events include:

- `node_id` - ID of the node
- `pot_id` - ID of the honeypot instance
- `pot_type` - Type of honeypot (cowrie, honnypotter, etc.)
- `event_type` - Type of event
- `timestamp` - When the event occurred
- `data` - Event-specific data

## Next Steps

- [Protocol Specification](./protocol.md) - Protocol overview
- [Command Reference](./commands.md) - Available commands
- [Message Types](./messages.md) - Message structure

