# Examples

Practical examples for common HoneyBee node use cases.

## Basic Examples

### Minimal Configuration (Testing Only)

```yaml
node:
  name: "test-node"
  type: "Agent"
  address: "0.0.0.0"
  port: 8080

server:
  address: "127.0.0.1:9001"

tls:
  enabled: false  # ⚠️ Testing only!

auth:
  totp_enabled: false  # ⚠️ Testing only!

log:
  level: "debug"
  format: "text"
```

### Production Configuration

```yaml
node:
  name: "prod-honeypot-01"
  type: "Full"
  address: "10.0.1.100"
  port: 8080

server:
  address: "manager.internal:9001"
  heartbeat_interval: 30

tls:
  enabled: true
  cert_file: "/etc/honeybee/certs/client.crt"
  key_file: "/etc/honeybee/certs/client.key"
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"

auth:
  totp_enabled: true
  totp_secret_dir: "/var/lib/honeybee/secrets"

log:
  level: "info"
  format: "json"
  file: "/var/log/honeybee/node.log"
```

## Deployment Examples

### Systemd Service

See [Systemd Deployment Guide](./deployment-systemd.md)

### Docker

See [Docker Deployment Guide](./deployment-docker.md)

### Kubernetes

See [Kubernetes Deployment Guide](./deployment-k8s.md)

## Security Examples

### TLS Setup

See [TLS Setup Guide](./tls.md)

### TOTP Setup

See [TOTP Setup Guide](./totp.md)

## More Examples

For comprehensive examples including:
- Multi-node deployments
- Ansible playbooks
- Certificate generation scripts
- Monitoring configurations
- Log aggregation

See the complete examples in [honeybee_node/docs/EXAMPLES.md](../../honeybee_node/docs/EXAMPLES.md).

