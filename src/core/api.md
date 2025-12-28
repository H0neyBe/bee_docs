# HoneyBee Core API

Backend API and CLI reference for HoneyBee Core.

## REST API

Base URL: `http://localhost:9002`

### Health Check

**GET** `/health`

Check if Core is running.

**Response**:
```json
{
  "status": "ok"
}
```

### Get Nodes

**GET** `/api/v1/nodes`

Get list of all registered nodes.

**Response**:
```json
{
  "nodes": [
    {
      "node_id": 12345,
      "node_name": "my-node",
      "status": "Running",
      "node_type": "Full"
    }
  ]
}
```

### Get Node

**GET** `/api/v1/nodes/{node_id}`

Get details of a specific node.

**Response**:
```json
{
  "node_id": 12345,
  "node_name": "my-node",
  "status": "Running",
  "node_type": "Full",
  "honeypots": [
    {
      "pot_id": "cowrie-01",
      "pot_type": "cowrie",
      "status": "Running"
    }
  ]
}
```

### Get Honeypots

**GET** `/api/v1/honeypots`

Get list of all honeypots.

**Response**:
```json
{
  "honeypots": [
    {
      "node_id": 12345,
      "pot_id": "cowrie-01",
      "pot_type": "cowrie",
      "status": "Running"
    }
  ]
}
```

## WebSocket Proxy

**URL**: `ws://localhost:9003`

Real-time updates via WebSocket. Connects to Core's backend and forwards Protocol v2 messages.

**Message Format**: JSON Protocol v2 messages

## CLI (Future)

Command-line interface for managing Core (planned).

## Authentication

API authentication is planned for future versions.

## Rate Limiting

Rate limiting is planned for future versions.

## Next Steps

- [API Reference](../reference/api.md) - Complete API reference
- [Protocol Specification](../protocol/protocol.md) - Protocol details

