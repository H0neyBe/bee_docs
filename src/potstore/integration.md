# HoneyBee Integration

Technical guide for integrating honeypots with HoneyBee.

## Overview

Honeypots integrate with HoneyBee by sending events to the HoneyBee Node, which forwards them to the Core manager.

## Event Flow

```
Honeypot → Node (TCP:9100) → Core (Protocol v2)
```

## Integration Methods

### Method 1: Output Plugin (Python)

For Python honeypots, create an output plugin:

```python
# honeybee.py
import json
import socket
import os
from datetime import datetime

class HoneyBeeOutput:
    def __init__(self):
        self.port = int(os.getenv('HONEYBEE_EVENT_PORT', '9100'))
        self.pot_id = os.getenv('HONEYBEE_POT_ID', 'honeypot-01')
        self.pot_type = os.getenv('HONEYBEE_HONEYPOT_TYPE', 'honeypot')
    
    def send_event(self, event_type, data):
        event = {
            "pot_id": self.pot_id,
            "pot_type": self.pot_type,
            "event_type": event_type,
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "data": data
        }
        
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect(('localhost', self.port))
            sock.sendall((json.dumps(event) + '\n').encode())
            sock.close()
        except Exception as e:
            print(f"Failed to send event to HoneyBee: {e}")
    
    def start(self):
        # Called when honeypot starts
        self.send_event("honeypot.started", {})
    
    def stop(self):
        # Called when honeypot stops
        self.send_event("honeypot.stopped", {})
```

### Method 2: Forwarder Script (PHP)

For PHP honeypots, create a forwarder:

```php
<?php
// honeybee-forwarder.php
class HoneyBeeForwarder {
    private $port;
    private $potId;
    private $potType;
    
    public function __construct() {
        $this->port = (int)($_ENV['HONEYBEE_EVENT_PORT'] ?? 9100);
        $this->potId = $_ENV['HONEYBEE_POT_ID'] ?? 'honeypot-01';
        $this->potType = $_ENV['HONEYBEE_HONEYPOT_TYPE'] ?? 'honeypot';
    }
    
    public function sendEvent($eventType, $data) {
        $event = [
            'pot_id' => $this->potId,
            'pot_type' => $this->potType,
            'event_type' => $eventType,
            'timestamp' => date('c'),
            'data' => $data
        ];
        
        $socket = @fsockopen('localhost', $this->port, $errno, $errstr, 1);
        if ($socket) {
            fwrite($socket, json_encode($event) . "\n");
            fclose($socket);
            return true;
        }
        return false;
    }
}
?>
```

## Environment Variables

HoneyBee Node sets these environment variables for honeypots:

- `HONEYBEE_EVENT_PORT` - Port to send events to (default: 9100)
- `HONEYBEE_POT_ID` - ID of the honeypot instance
- `HONEYBEE_HONEYPOT_TYPE` - Type of honeypot

## Event Format

Events must be JSON, one per line:

```json
{"pot_id":"cowrie-01","pot_type":"cowrie","event_type":"login","timestamp":"2024-01-15T10:30:00Z","data":{"username":"admin"}}
{"pot_id":"cowrie-01","pot_type":"cowrie","event_type":"command","timestamp":"2024-01-15T10:30:01Z","data":{"input":"ls"}}
```

## Connection Handling

- **Protocol**: TCP
- **Address**: localhost
- **Port**: 9100 (configurable)
- **Format**: JSON lines (one event per line)
- **Error Handling**: Fail silently, don't block honeypot

## Best Practices

1. **Non-blocking**: Don't block honeypot if connection fails
2. **Error Handling**: Log errors but continue operation
3. **Batching**: Send events immediately (real-time)
4. **Format**: Always use JSON, one event per line
5. **Timestamps**: Use ISO 8601 format with Z timezone

## Testing Integration

### Test Event Sending

```python
import socket
import json

event = {
    "pot_id": "test-01",
    "pot_type": "test",
    "event_type": "test.event",
    "timestamp": "2024-01-15T10:30:00Z",
    "data": {"test": "data"}
}

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('localhost', 9100))
sock.sendall((json.dumps(event) + '\n').encode())
sock.close()
```

### Verify Events Received

Check node logs to verify events are received and forwarded to Core.

## Next Steps

- [Adding New Honeypots](./adding.md) - Add your honeypot
- [Available Honeypots](./honeypots.md) - See examples

