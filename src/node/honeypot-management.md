# Honeypot Management

How HoneyBee Node manages honeypots (pots).

## Overview

HoneyBee Node automatically installs, configures, and manages honeypots from the HoneyBee Potstore. Honeypots are installed on-demand via commands from HoneyBee Core.

## Installation Flow

1. **Core sends InstallPot command** - Specifies honeypot type, ID, and configuration
2. **Node clones Potstore** - Downloads honeypot from GitHub
3. **Node sets up environment** - Creates virtual environments, installs dependencies
4. **Node configures integration** - Sets up event forwarding to Core
5. **Node reports status** - Sends installation progress to Core

## Supported Honeypots

From the [HoneyBee Potstore](../potstore/honeypots.md):

- **Cowrie** - SSH and Telnet honeypot (Python/Twisted)
- **HonnyPotter** - WordPress login honeypot (PHP)
- More coming soon...

## Installation Process

### Automatic Installation

When Core sends an `InstallPot` command:

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

1. Clone Potstore repository (if not already cloned)
2. Copy honeypot to `~/.honeybee/honeypots/cowrie-01/`
3. Create Python virtual environment
4. Install dependencies (`pip install -r requirements.txt`)
5. Configure HoneyBee integration
6. Start honeypot (if `auto_start: true`)

### Manual Installation

Honeypots can also be installed manually, but automatic installation is recommended.

## Honeypot Lifecycle

### States

- **Installing** - Honeypot is being installed
- **Running** - Honeypot is active
- **Stopped** - Honeypot is stopped
- **Failed** - Installation or startup failed

### Commands

#### Start Honeypot

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

#### Stop Honeypot

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

#### Restart Honeypot

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

#### Get Status

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

## Event Forwarding

All honeypot events are automatically forwarded to Core:

- **TCP Socket** - Events sent to `localhost:9100` (configurable)
- **JSON Format** - Structured event data
- **Real-time** - Events forwarded immediately

### Event Format

```json
{
  "node_id": 12345,
  "pot_id": "cowrie-01",
  "pot_type": "cowrie",
  "event_type": "login",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "username": "admin",
    "password": "password123",
    "ip": "192.168.1.100",
    "session": "abc123"
  }
}
```

## Configuration

Honeypots are configured via the `InstallPot` command's `config` field:

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

## Directory Structure

Honeypots are installed in:

```
~/.honeybee/honeypots/
├── cowrie-01/          # First Cowrie instance
│   ├── cowrie-env/     # Python virtual environment
│   ├── src/            # Cowrie source
│   ├── etc/            # Configuration
│   └── logs/           # Logs
├── cowrie-02/          # Second Cowrie instance
└── honnypotter-01/     # HonnyPotter instance
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

See [Troubleshooting](./troubleshooting.md) for more help.

