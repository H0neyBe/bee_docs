# Creating Custom Nodes

Build HoneyBee nodes in any language.

## Overview

HoneyBee nodes can be implemented in any language that supports:
- TCP sockets
- JSON parsing
- TLS (optional)

## Protocol Requirements

Your node must implement:

1. **Connect to Core** - TCP connection to Core (port 9001)
2. **Register** - Send `NodeRegistration` message
3. **Handle Commands** - Process `NodeCommand` messages
4. **Send Status** - Send `NodeStatusUpdate` periodically
5. **Send Events** - Send `PotEvent` messages (if managing honeypots)

## Implementation Steps

### 1. Connect to Core

```python
import socket
import ssl

# Connect to Core
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Optional: Wrap with TLS
if use_tls:
    context = ssl.create_default_context()
    sock = context.wrap_socket(sock, server_hostname="honeybee-manager")

sock.connect(("localhost", 9001))
```

### 2. Register Node

```python
import json

registration = {
    "version": 2,
    "message": {
        "NodeRegistration": {
            "node_id": 12345,
            "node_name": "my-python-node",
            "address": "0.0.0.0",
            "port": 8080,
            "node_type": "Full",
            "totp_code": "123456"  # If TOTP enabled
        }
    }
}

message = json.dumps(registration) + "\n"
sock.sendall(message.encode())
```

### 3. Handle Messages

```python
while True:
    data = sock.recv(4096)
    if not data:
        break
    
    message = json.loads(data.decode())
    
    if "RegistrationAck" in message["message"]:
        ack = message["message"]["RegistrationAck"]
        if ack["accepted"]:
            print("Registration successful")
    
    elif "NodeCommand" in message["message"]:
        command = message["message"]["NodeCommand"]
        handle_command(command)
```

### 4. Send Heartbeat

```python
import time

while True:
    status_update = {
        "version": 2,
        "message": {
            "NodeStatusUpdate": {
                "node_id": 12345,
                "status": "Running"
            }
        }
    }
    
    message = json.dumps(status_update) + "\n"
    sock.sendall(message.encode())
    
    time.sleep(30)  # Every 30 seconds
```

## Example Implementations

### Python

See example Python node (if available in repository).

### Rust

See example Rust node (if available in repository).

## Protocol Details

See [Protocol Specification](../protocol/protocol.md) for complete protocol details.

## Next Steps

- [Protocol Specification](../protocol/protocol.md) - Protocol details
- [Message Types](../protocol/messages.md) - Message reference
- [Command Reference](../protocol/commands.md) - Available commands

