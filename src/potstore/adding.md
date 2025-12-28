# Adding New Honeypots

Guide for adding new honeypots to the HoneyBee Potstore.

## Requirements

To add a honeypot to the Potstore, it must:

1. ✅ Be installable (Python: `pip install`, PHP: standalone or composer)
2. ✅ Support Python 3.7+ or PHP 7.4+
3. ✅ Send events to TCP socket `localhost:9100`
4. ✅ Use JSON format for events (one per line)
5. ✅ Include installation instructions
6. ✅ Include HoneyBee integration code

## Directory Structure

Create a directory for your honeypot:

```
honeybee_potstore/
└── your-honeypot/
    ├── README.md              # Honeypot documentation
    ├── install.sh             # Installation script (Linux/macOS)
    ├── install.ps1            # Installation script (Windows)
    ├── requirements.txt       # Python dependencies (if Python)
    ├── honeybee-integration/  # HoneyBee integration code
    └── ...
```

## HoneyBee Integration

### Python Honeypots

Create an output plugin or integration module:

```python
# honeybee-integration/honeybee.py
import json
import socket
import os

def send_event(event_data):
    """Send event to HoneyBee Node"""
    port = int(os.getenv('HONEYBEE_EVENT_PORT', '9100'))
    pot_id = os.getenv('HONEYBEE_POT_ID', 'honeypot-01')
    
    event = {
        "pot_id": pot_id,
        "event_type": event_data.get("event_type"),
        "timestamp": event_data.get("timestamp"),
        "data": event_data
    }
    
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect(('localhost', port))
        sock.sendall((json.dumps(event) + '\n').encode())
        sock.close()
    except Exception as e:
        print(f"Failed to send event: {e}")
```

### PHP Honeypots

Create a forwarder script:

```php
<?php
// honeybee-forwarder.php
function sendToHoneyBee($eventData) {
    $port = (int)($_ENV['HONEYBEE_EVENT_PORT'] ?? 9100);
    $potId = $_ENV['HONEYBEE_POT_ID'] ?? 'honeypot-01';
    
    $event = [
        'pot_id' => $potId,
        'event_type' => $eventData['event_type'],
        'timestamp' => date('c'),
        'data' => $eventData
    ];
    
    $socket = @fsockopen('localhost', $port, $errno, $errstr, 1);
    if ($socket) {
        fwrite($socket, json_encode($event) . "\n");
        fclose($socket);
    }
}
?>
```

## Installation Scripts

### install.sh (Linux/macOS)

```bash
#!/bin/bash
set -e

echo "Installing Your Honeypot..."

# Create directories
mkdir -p logs

# Install dependencies (example for Python)
if [ -f "requirements.txt" ]; then
    pip install -r requirements.txt
fi

# Set permissions
chmod +x your-honeypot.py

echo "Installation complete!"
```

### install.ps1 (Windows)

```powershell
Write-Host "Installing Your Honeypot..."

# Create directories
New-Item -ItemType Directory -Force -Path logs

# Install dependencies
if (Test-Path "requirements.txt") {
    pip install -r requirements.txt
}

Write-Host "Installation complete!"
```

## Update potstore.json

Add your honeypot to `potstore.json`:

```json
{
  "id": "your-honeypot",
  "name": "Your Honeypot",
  "version": "1.0.0",
  "type": "your-honeypot",
  "description": "Description of your honeypot",
  "protocols": ["protocol1", "protocol2"],
  "default_ports": {
    "protocol1": 2222
  },
  "requirements": {
    "python": ">=3.7",
    "os": ["linux", "windows", "macos"]
  },
  "directory": "your-honeypot",
  "setup": {
    "install": "./install.sh",
    "output_plugin": "honeybee-integration/honeybee.py"
  },
  "status": "stable",
  "features": [
    "Feature 1",
    "Feature 2"
  ],
  "honeybee_integration": {
    "event_port": 9100,
    "event_format": "json",
    "output_plugin": "honeybee.py",
    "auto_configure": true
  }
}
```

## Event Format

Events must follow this structure:

```json
{
  "pot_id": "your-honeypot-01",
  "event_type": "your.event.type",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    // Event-specific data
  }
}
```

## Testing

1. Install your honeypot manually
2. Configure HoneyBee integration
3. Start honeypot
4. Generate test events
5. Verify events are sent to `localhost:9100`
6. Check events are received by node

## Submission

1. Fork the Potstore repository
2. Add your honeypot directory
3. Update `potstore.json`
4. Create a pull request
5. Include documentation and examples

## Next Steps

- [Integration Guide](./integration.md) - Technical integration details
- [Available Honeypots](./honeypots.md) - See existing examples

