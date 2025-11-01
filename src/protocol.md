# Message Protocol Definition

This document describes the bidirectional messaging contract used by Honeybee Core nodes and the node manager.

## Protocol Version

Current protocol version: **2** ([`PROTOCOL_VERSION`](../../honeybee_core/bee_message/src/common.rs)). Every payload must embed this version inside the [`MessageEnvelope`](../../honeybee_core/bee_message/src/common.rs) wrapper. Nodes should refuse to operate if the version they receive from the manager does not match their compiled version.

## Message Envelope

All protocol payloads share a common generic wrapper defined in [`bee_message/src/common.rs`](../../honeybee_core/bee_message/src/common.rs):

```rust
pub struct MessageEnvelope<T> {
    pub version: u64,
    pub message: T,
}
```

**JSON Example**

```json
{
    "version": 2,
    "message": {
        "NodeStatusUpdate": {
            "node_id": 123456789,
            "status": "Running"
        }
    }
}
```

The `message` field is populated with either [`NodeToManagerMessage`](../../honeybee_core/bee_message/src/node_to_manager.rs) or [`ManagerToNodeMessage`](../../honeybee_core/bee_message/src/manager_to_node.rs) depending on direction.

## Directional Message Families

### Node → Manager (`NodeToManagerMessage`)

Nodes emit the following variants defined in [`bee_message/src/node_to_manager.rs`](../../honeybee_core/bee_message/src/node_to_manager.rs):

#### NodeRegistration

Purpose: register the connection and provide basic metadata.

Structure:

```rust
pub struct NodeRegistration {
    pub node_id: u64,
    pub node_name: String,
    pub address: String,
    pub port: u16,
    pub node_type: NodeType,
}
```

JSON Example:

```json
{
    "version": 2,
    "message": {
        "NodeRegistration": {
            "node_id": 123456789,
            "node_name": "production-honeypot-01",
            "address": "192.168.1.100",
            "port": 8080,
            "node_type": "Full"
        }
    }
}
```

Usage:
1. Send immediately after establishing the TCP connection.
2. Wait for a [`RegistrationAck`](../../honeybee_core/bee_message/src/manager_to_node.rs) before sending any additional message types.
3. Reconnect and send a new registration if the socket drops.

**Implementation Reference**: See [`NodeClient::register`](../../node_example/rust/src/main.rs) for example implementation.

#### NodeStatusUpdate

Purpose: publish the live operating state of the node.

Structure:

```rust
pub struct NodeStatusUpdate {
    pub node_id: u64,
    pub status: NodeStatus,
}
```

JSON Example:

```json
{
    "version": 2,
    "message": {
        "NodeStatusUpdate": {
            "node_id": 123456789,
            "status": "Running"
        }
    }
}
```

Usage:
- Emit immediately after registration.
- Continue sending on an interval (30 seconds recommended, see [`HEARTBEAT_INTERVAL`](../../node_example/rust/src/main.rs)).
- Send whenever the status meaningfully changes.

**Implementation Reference**: See [`NodeClient::send_status_update`](../../node_example/rust/src/main.rs) and [`Node::handle_message`](../../honeybee_core/src/node_manager/node.rs) for implementation examples.

#### NodeEvent

Purpose: report noteworthy events.

Structure:

```rust
pub enum NodeEvent {
    Started,
    Stopped,
    Alarm,
    Error { message: String },
}
```

JSON Examples:

```json
{
    "version": 2,
    "message": {
        "NodeEvent": "Started"
    }
}
```

```json
{
    "version": 2,
    "message": {
        "NodeEvent": {
            "Error": {
                "message": "Failed to bind to port 8080"
            }
        }
    }
}
```

Usage:
- Pair with status updates to give operators more context.
- When reporting an error, also send [`NodeStatusUpdate`](../../honeybee_core/bee_message/src/node_to_manager.rs) with `Failed`.

**Implementation Reference**: See [`NodeClient::send_event`](../../node_example/rust/src/main.rs) for example implementation.

#### NodeDrop

Purpose: request a graceful disconnect.

JSON Example:

```json
{
    "version": 2,
    "message": "NodeDrop"
}
```

Usage:
1. Send before intentionally closing the socket.
2. Allow the manager up to one second to clean up before exiting.

**Implementation Reference**: See [`NodeClient::send_node_drop`](../../node_example/rust/src/main.rs) for example implementation.

### Manager → Node (`ManagerToNodeMessage`)

The manager uses these variants defined in [`bee_message/src/manager_to_node.rs`](../../honeybee_core/bee_message/src/manager_to_node.rs):

#### NodeCommand

Purpose: instruct a node to perform an action.

Structure:

```rust
pub struct NodeCommand {
    pub node_id: u64,
    pub command: String,
}
```

Helper constructors are available: [`NodeCommand::status`](../../honeybee_core/bee_message/src/manager_to_node.rs), [`NodeCommand::stop`](../../honeybee_core/bee_message/src/manager_to_node.rs), [`NodeCommand::restart`](../../honeybee_core/bee_message/src/manager_to_node.rs), and [`NodeCommand::custom`](../../honeybee_core/bee_message/src/manager_to_node.rs).

JSON Example:

```json
{
    "version": 2,
    "message": {
        "NodeCommand": {
            "node_id": 123456789,
            "command": "status"
        }
    }
}
```

Usage:
- Nodes should acknowledge by sending the requested status/event combination.
- Unknown commands should trigger a [`NodeEvent::Error`](../../honeybee_core/bee_message/src/node_to_manager.rs) describing the issue.

**Implementation Reference**: See [`NodeClient::handle_message`](../../node_example/rust/src/main.rs) for command handling examples.

#### RegistrationAck

Purpose: confirm whether a registration succeeded.

Structure:

```rust
pub struct RegistrationAck {
    pub node_id: u64,
    pub accepted: bool,
    pub message: Option<String>,
}
```

JSON Example:

```json
{
    "version": 2,
    "message": {
        "RegistrationAck": {
            "node_id": 123456789,
            "accepted": true,
            "message": "Node production-honeypot-01 successfully registered"
        }
    }
}
```

Usage:
- Sent exactly once per connection immediately after parsing the [`NodeRegistration`](../../honeybee_core/bee_message/src/node_to_manager.rs).
- On rejection, close the socket after delivering the message string.

**Implementation Reference**: See [`NodeManager::listen`](../../honeybee_core/src/node_manager/manager.rs) for acknowledgment sending logic.

## Node Types

[`NodeType`](../../honeybee_core/bee_message/src/common.rs) classifies how much capability a node exposes:

- `Full`: A full-featured honeypot that can run interactive deception workloads.
- `Agent`: A lightweight probe used for telemetry or edge collection.

## Node Status Values

[`NodeStatus`](../../honeybee_core/bee_message/src/common.rs) communicates the high-level lifecycle state:

| Status    | Description                                   | Typical Trigger                        |
|-----------|-----------------------------------------------|----------------------------------------|
| Deploying | Node is bootstrapping but not yet ready       | During initialization routines         |
| Running   | Node is healthy and answering requests        | Normal operations and heartbeat ticks  |
| Stopped   | Node intentionally halted                     | After a stop command or graceful exit  |
| Failed    | Node cannot continue without intervention     | Irrecoverable error or panic           |
| Unknown   | Manager cannot decide the current state       | Default while awaiting first heartbeat |

## Communication Patterns

### Initial Connection Handshake

```
Node                           Manager
    |                               |
    |--- TCP Connect -------------> |
    |                               |
    |--- NodeRegistration --------> |
    |                               |
    |<-- RegistrationAck --------- |
    |                               |
    |--- NodeStatusUpdate(Running)->|
    |                               |
    |<-- NodeCommand(*) -----------|
```

**Implementation Reference**: See [`NodeClient::run`](../../node_example/rust/src/main.rs) and [`NodeManager::listen`](../../honeybee_core/src/node_manager/manager.rs) for the complete handshake flow.

### Heartbeats

```
Node                           Manager
    |                               |
    |--- NodeStatusUpdate --------> |
    |                               |
    |    wait ~30s                  |
    |                               |
    |--- NodeStatusUpdate --------> |
```

**Implementation Reference**: See heartbeat logic in [Rust client](../../node_example/rust/src/main.rs), [Go client](../../node_example/go/client/client.go), and [Python client](../../node_example/python/main.py).

### Command Flow

```
Node                           Manager
    |                               |
    |<-- NodeCommand(stop) -------- |
    |                               |
    |--- NodeStatusUpdate(Stopped)->|
    |--- NodeEvent(Stopped) ------> |
    |--- NodeDrop ----------------> |
```

**Implementation Reference**: See [`NodeClient::handle_message`](../../node_example/rust/src/main.rs) for command handling.

### Error Reporting

```
Node                           Manager
    |                               |
    |--- NodeEvent(Error) --------> |
    |--- NodeStatusUpdate(Failed) ->|
    |    attempt recovery           |
    |--- NodeStatusUpdate(Running)->|
         OR
    |--- NodeDrop ----------------> |
```

## Wire Format

- Encoding: UTF-8 JSON.
- Each TCP write must contain exactly one serialized envelope.
- No extra framing bytes; the consumer should parse until a full JSON value is read.
- Recommended read buffer: at least 4 KiB (see buffer allocation in [`NodeManager::listen`](../../honeybee_core/src/node_manager/manager.rs)).
- Recommended maximum payload size: 64 KiB.

## Protocol Compliance Checklist

Node responsibilities:

1. Send [`NodeRegistration`](../../honeybee_core/bee_message/src/node_to_manager.rs) first and wait for [`RegistrationAck`](../../honeybee_core/bee_message/src/manager_to_node.rs).
2. Follow with an immediate [`NodeStatusUpdate`](../../honeybee_core/bee_message/src/node_to_manager.rs).
3. Maintain heartbeat cadence and report transitions promptly.
4. Respond to [`NodeCommand`](../../honeybee_core/bee_message/src/manager_to_node.rs) variants without crashing on unknown commands.
5. Notify the manager with [`NodeDrop`](../../honeybee_core/bee_message/src/node_to_manager.rs) before voluntarily disconnecting.
6. Include the exact [`PROTOCOL_VERSION`](../../honeybee_core/bee_message/src/common.rs) in every envelope.

Manager responsibilities:

1. Parse registrations and reply with [`RegistrationAck`](../../honeybee_core/bee_message/src/manager_to_node.rs).
2. Track node metadata and lifecycle transitions (see [`Node`](../../honeybee_core/src/node_manager/node.rs)).
3. Issue [`NodeCommand`](../../honeybee_core/bee_message/src/manager_to_node.rs) messages as needed.
4. Handle abrupt disconnects and clean up internal state.
5. Validate protocol versions and log mismatches.

## Error Handling Guidelines

- Malformed JSON: log the payload and continue reading (see error handling in [`NodeManager::listen`](../../honeybee_core/src/node_manager/manager.rs)).
- Version mismatch: log a warning; optionally reject the connection.
- Unexpected message sequence: warn and consider closing the socket after notifying the peer.
- Repeated deserialization failures: disconnect to protect resources.

## Security Notes

- Deploy behind authenticated tunnels (VPN, mTLS, etc.).
- Enforce rate limits and payload size checks to prevent abuse.
- Treat [`NodeCommand`](../../honeybee_core/bee_message/src/manager_to_node.rs) payloads as untrusted input on the node side.

## Implementation References

- Message definitions:
  - Common types: [`bee_message/src/common.rs`](../../honeybee_core/bee_message/src/common.rs)
  - Node to Manager: [`bee_message/src/node_to_manager.rs`](../../honeybee_core/bee_message/src/node_to_manager.rs)
  - Manager to Node: [`bee_message/src/manager_to_node.rs`](../../honeybee_core/bee_message/src/manager_to_node.rs)
  - Bidirectional: [`bee_message/src/core.rs`](../../honeybee_core/bee_message/src/core.rs)
- Manager implementation: [`honeybee_core/src/node_manager/manager.rs`](../../honeybee_core/src/node_manager/manager.rs)
- Node handler implementation: [`honeybee_core/src/node_manager/node.rs`](../../honeybee_core/src/node_manager/node.rs)
- Example clients:
  - Rust: [`node_example/rust/src/main.rs`](../../node_example/rust/src/main.rs)
  - Python: [`node_example/python/main.py`](../../node_example/python/main.py)
  - Go: [`node_example/go/main.go`](../../node_example/go/main.go)

## Version History

| Version | Notes                                                    |
|---------|----------------------------------------------------------|
| 2       | Introduced directional enums and ACK, generic envelopes  |
| 1       | Initial message envelope design                          |