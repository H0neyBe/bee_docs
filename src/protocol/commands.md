# Command Reference

Complete reference for all HoneyBee Protocol v2 commands.

## InstallPot

Install a honeypot from Potstore or custom Git repository.

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "InstallPot": {
        "pot_id": "cowrie-01",
        "honeypot_type": "cowrie",
        "git_url": null,  // null = use Potstore
        "git_branch": null,  // null = use default branch
        "config": {
          "ssh_port": "2222",
          "telnet_port": "2223"
        },
        "auto_start": true
      }
    }
  }
}
```

**Response**: Node sends `PotStatusUpdate` with status changes.

## DeployPot

Start a honeypot instance.

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

**Response**: Node sends `PotStatusUpdate` with new status.

## StopPot

Stop a honeypot instance.

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

**Response**: Node sends `PotStatusUpdate` with new status.

## RestartPot

Restart a honeypot instance.

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

**Response**: Node sends `PotStatusUpdate` with status changes.

## GetPotStatus

Get current status of a honeypot.

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

**Response**: Node sends `PotStatusUpdate` with current status.

## GetInstalledPots

Get list of all installed honeypots on a node.

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "GetInstalledPots": {}
    }
  }
}
```

**Response**: Node sends multiple `PotStatusUpdate` messages, one for each honeypot.

## GetPotInfo

Get detailed information about a honeypot.

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "GetPotInfo": "cowrie-01"
    }
  }
}
```

**Response**: Node sends `PotStatusUpdate` with detailed information.

## Restart

Restart the node.

```json
{
  "NodeCommand": {
    "node_id": 12345,
    "command": {
      "Restart": {}
    }
  }
}
```

**Response**: Node sends `NodeStatusUpdate` with status changes.

## Command Execution Flow

1. Core sends `NodeCommand` to node
2. Node processes command
3. Node sends status updates as command progresses
4. Node sends final status update when complete

## Error Handling

If a command fails, the node sends a status update with `status: "Failed"` and an error message in the `message` field.

## Next Steps

- [Protocol Specification](./protocol.md) - Protocol overview
- [Message Types](./messages.md) - Message structure
- [Event Format](./events.md) - Event structure

