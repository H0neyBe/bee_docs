# Node Types

HoneyBee supports two types of nodes: **Full** and **Agent**.

## Full Node

A **Full** node provides complete honeypot management capabilities.

### Capabilities

- ✅ **Honeypot Installation** - Can install honeypots from Potstore or Git repositories
- ✅ **Honeypot Management** - Can start, stop, restart, and monitor honeypots
- ✅ **Event Forwarding** - Forwards all honeypot events to the Core manager
- ✅ **Status Reporting** - Reports honeypot status and health metrics

### Configuration

```yaml
node:
  type: "Full"

honeypot:
  enabled: true  # Required for Full nodes
```

### Use Cases

- Production deployments
- Nodes that need to run honeypots
- Complete honeypot management

## Agent Node

An **Agent** node is a lightweight monitoring probe.

### Capabilities

- ✅ **Status Reporting** - Reports node health and status
- ✅ **Event Reception** - Can receive events from external sources
- ❌ **No Honeypot Management** - Cannot install or manage honeypots
- ❌ **No Event Forwarding** - Does not forward honeypot events

### Configuration

```yaml
node:
  type: "Agent"

honeypot:
  enabled: false  # Typically disabled for Agent nodes
```

### Use Cases

- Lightweight monitoring nodes
- Network probes
- Status-only reporting
- Nodes without honeypot requirements

## Important Notes

- **Node type is informational** - Sent to Core during registration
- **Honeypot management is controlled by `honeypot.enabled`** - Actual capability depends on this setting
- **Core uses node type** - Core may use node type to determine which commands to send
- **Default is "Full"** - New nodes default to "Full" for maximum capabilities

## Choosing a Node Type

### Use Full Node When:

- You need to deploy honeypots
- You want complete management capabilities
- You're deploying in production

### Use Agent Node When:

- You only need status monitoring
- You want minimal resource usage
- You're deploying lightweight probes

