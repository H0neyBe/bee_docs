# Using Honeypots

How to deploy and use honeypots from the HoneyBee Potstore.

## Automatic Installation

Honeypots are automatically installed when Core sends an `InstallPot` command to a node.

### Example: Install Cowrie

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
            "ssh_port": "2222",
            "telnet_port": "2223"
          },
          "auto_start": true
        }
      }
    }
  }
}
```

The node will:
1. Clone Potstore repository
2. Install Cowrie with dependencies
3. Configure HoneyBee integration
4. Start Cowrie automatically

## Manual Installation

You can also install honeypots manually for testing:

### Cowrie

```bash
# Clone Potstore
git clone https://github.com/H0neyBe/honeybee_potstore.git
cd honeybee_potstore/cowrie

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt

# Configure HoneyBee output
# Edit etc/cowrie.cfg to enable honeybee.py output plugin

# Start Cowrie
bin/cowrie start
```

### HonnyPotter

```bash
# Clone Potstore
git clone https://github.com/H0neyBe/honeybee_potstore.git
cd honeybee_potstore/HonnyPotter

# Run installation script
chmod +x install.sh
./install.sh

# Start PHP server
php -S 0.0.0.0:8080 standalone.php
```

## Managing Honeypots

### Start Honeypot

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "DeployPot": "cowrie-01"
    }
  }
}
```

### Stop Honeypot

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "StopPot": "cowrie-01"
    }
  }
}
```

### Restart Honeypot

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "RestartPot": "cowrie-01"
    }
  }
}
```

### Check Status

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "GetPotStatus": "cowrie-01"
    }
  }
}
```

## Configuration

Honeypots are configured via the `config` field in `InstallPot`:

### Cowrie Configuration

```json
{
  "config": {
    "ssh_port": "2222",
    "telnet_port": "2223",
    "hostname": "honeybee-cowrie",
    "log_path": "/var/log/cowrie"
  }
}
```

### HonnyPotter Configuration

```json
{
  "config": {
    "http_port": "8080",
    "pot_id": "honnypotter-01"
  }
}
```

## Event Monitoring

Once honeypots are running, they forward events to the node, which forwards them to Core.

### View Events

Events can be viewed via:
- Core logs
- Backend API (port 9002)
- WebSocket proxy (port 9003)

### Event Format

See [Event Format](../protocol/events.md) for event structure.

## Testing Honeypots

### Test Cowrie

```bash
# SSH to honeypot
ssh -p 2222 admin@localhost

# Or Telnet
telnet localhost 2223
```

### Test HonnyPotter

```bash
# Test login endpoint
curl -X POST http://localhost:8080/standalone.php \
  -d "log=admin&pwd=password123"
```

## Troubleshooting

### Installation Fails

1. Check Python/PHP is installed
2. Verify Git is available
3. Check disk space
4. Review node logs

### Honeypot Won't Start

1. Check dependencies are installed
2. Verify configuration
3. Check port availability
4. Review honeypot logs

### Events Not Forwarding

1. Verify event listener is running (port 9100)
2. Check honeypot configuration
3. Verify network connectivity
4. Enable debug logging

## Next Steps

- [Available Honeypots](./honeypots.md) - List of all honeypots
- [Adding New Honeypots](./adding.md) - Add your own honeypots
- [Integration Guide](./integration.md) - Technical details

