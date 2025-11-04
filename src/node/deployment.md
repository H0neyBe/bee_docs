# Production Deployment Guide

Complete guide for deploying HoneyBee nodes in production environments.

## Deployment Options

| Method | Best For | Complexity | Scalability |
|--------|----------|------------|-------------|
| **Standalone Binary** | Single server, testing | Low | Limited |
| **Systemd Service** | Linux servers | Medium | Good |
| **Docker** | Containerized environments | Medium | Excellent |
| **Kubernetes** | Cloud-native, microservices | High | Excellent |
| **Ansible** | Fleet management | Medium | Excellent |

## Pre-Deployment Checklist

Before deploying to production:

### Security
- [ ] TLS certificates generated and installed
- [ ] TOTP authentication configured
- [ ] Firewall rules configured
- [ ] Running as non-root user
- [ ] Secrets properly secured (0600/0700 permissions)

### Configuration
- [ ] Configuration file created and validated
- [ ] Server address configured correctly
- [ ] Log paths created and writable
- [ ] Certificate paths verified

### System
- [ ] Time synchronization enabled (NTP)
- [ ] Sufficient disk space for logs
- [ ] Network connectivity to manager verified
- [ ] DNS resolution working (if using hostnames)

### Monitoring
- [ ] Log rotation configured
- [ ] Monitoring/alerting set up
- [ ] Backup strategy in place
- [ ] Incident response plan ready

## Deployment Methods

### 1. Systemd Service (Recommended for Linux)

**Best for**: Production Linux servers

**Pros**:
- Native Linux service management
- Automatic start on boot
- Log integration with journald
- Easy monitoring and control

**Setup**:

```bash
# 1. Install binary
sudo cp build/honeybee-node /usr/local/bin/
sudo chmod +x /usr/local/bin/honeybee-node

# 2. Create user
sudo useradd -r -s /bin/false honeybee

# 3. Create directories
sudo mkdir -p /etc/honeybee /var/lib/honeybee/secrets /var/log/honeybee
sudo chown -R honeybee:honeybee /var/lib/honeybee /var/log/honeybee

# 4. Copy configuration
sudo cp configs/config.yaml /etc/honeybee/
sudo chown honeybee:honeybee /etc/honeybee/config.yaml
sudo chmod 600 /etc/honeybee/config.yaml

# 5. Install service file
sudo cp systemd/honeybee-node.service /etc/systemd/system/
sudo systemctl daemon-reload

# 6. Enable and start
sudo systemctl enable honeybee-node
sudo systemctl start honeybee-node

# 7. Verify
sudo systemctl status honeybee-node
```

**See**: [Systemd Deployment Guide](./deployment-systemd.md) for details.

### 2. Docker (Recommended for Containers)

**Best for**: Containerized environments, cloud deployments

**Pros**:
- Isolated environment
- Easy scaling
- Portable across platforms
- Version control

**Quick Start**:

```bash
# Build image
docker build -t honeybee-node:latest .

# Run container
docker run -d \
  --name honeybee-node \
  -v $(pwd)/configs:/app/configs:ro \
  -v $(pwd)/certs:/app/certs:ro \
  -v honeybee-logs:/app/logs \
  -v honeybee-secrets:/home/honeybee/.config/honeybee \
  --restart unless-stopped \
  honeybee-node:latest
```

**See**: [Docker Deployment Guide](./deployment-docker.md) for details.

### 3. Kubernetes (Recommended for Cloud)

**Best for**: Cloud-native deployments, microservices

**Pros**:
- Automatic scaling
- Self-healing
- Rolling updates
- Service discovery

**Quick Start**:

```bash
# Apply manifests
kubectl apply -f k8s/

# Check status
kubectl get pods -l app=honeybee-node

# View logs
kubectl logs -f -l app=honeybee-node
```

**See**: [Kubernetes Deployment Guide](./deployment-k8s.md) for details.

### 4. Standalone Binary

**Best for**: Testing, development, single deployments

**Pros**:
- Simple setup
- No dependencies
- Full control

**Setup**:

```bash
# 1. Build
make build

# 2. Configure
cp configs/config.yaml /etc/honeybee/config.yaml

# 3. Run
./build/honeybee-node -config /etc/honeybee/config.yaml
```

### 5. Ansible (Recommended for Fleet Management)

**Best for**: Managing multiple nodes, automation

**Pros**:
- Centralized management
- Consistent deployment
- Easy updates
- Configuration management

**Example Playbook**:

```yaml
---
- name: Deploy HoneyBee nodes
  hosts: honeybee_nodes
  become: yes
  
  tasks:
    - name: Install honeybee-node
      copy:
        src: build/honeybee-node
        dest: /usr/local/bin/honeybee-node
        mode: '0755'
    
    - name: Deploy configuration
      template:
        src: config.yaml.j2
        dest: /etc/honeybee/config.yaml
        mode: '0600'
        owner: honeybee
    
    - name: Deploy certificates
      copy:
        src: "certs/{{ inventory_hostname }}/"
        dest: /etc/honeybee/certs/
        mode: '0600'
        owner: honeybee
    
    - name: Start service
      systemd:
        name: honeybee-node
        state: started
        enabled: yes
```

## Configuration Management

### Environment-Specific Configs

Create separate configs for each environment:

```
configs/
├── config.dev.yaml      # Development
├── config.staging.yaml  # Staging
├── config.prod.yaml     # Production
└── config.example.yaml  # Template
```

### Using Templates

Use Go templates or envsubst for dynamic configuration:

```yaml
# config.template.yaml
server:
  address: "${MANAGER_ADDRESS}"
  
tls:
  ca_file: "${CERT_PATH}/ca.crt"
```

```bash
# Substitute variables
envsubst < config.template.yaml > config.yaml
```

### Secrets Management

#### HashiCorp Vault

```bash
# Store secrets
vault kv put secret/honeybee/node1 \
  totp="JBSWY3DPEHPK3PXP" \
  tls_key="@client.key"

# Retrieve in startup script
vault kv get -field=totp secret/honeybee/node1 > /var/lib/honeybee/secrets/.honeybee_totp_secret
```

#### Kubernetes Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: honeybee-secrets
type: Opaque
data:
  totp-secret: SkJTV1kzRFBFSFBLM1BYUA==  # base64 encoded
```

## Monitoring & Observability

### Logging

**Structured Logging** (recommended):
```yaml
log:
  level: "info"
  format: "json"
  file: "/var/log/honeybee/node.log"
```

**Log Aggregation**:
- Elasticsearch + Kibana (ELK)
- Grafana Loki
- Splunk
- CloudWatch (AWS)

### Metrics

Future feature - Prometheus metrics endpoint:
```yaml
metrics:
  enabled: true
  port: 9090
```

### Health Checks

**Systemd**:
```bash
sudo systemctl status honeybee-node
```

**Docker**:
```bash
docker ps | grep honeybee-node
```

**Kubernetes**:
```bash
kubectl get pods -l app=honeybee-node
```

## High Availability

### Multiple Nodes

Deploy multiple nodes for redundancy:

```yaml
# Node 1
node:
  name: "honeypot-01"
  address: "10.0.1.10"

# Node 2
node:
  name: "honeypot-02"
  address: "10.0.1.11"
```

### Load Balancing

If using multiple managers (future feature):

```yaml
server:
  addresses:
    - "manager1.example.com:9001"
    - "manager2.example.com:9001"
  failover: true
```

### Monitoring & Alerting

Set up alerts for:
- Connection failures
- Authentication failures
- High error rates
- Disk space issues
- Certificate expiration

## Scaling

### Horizontal Scaling

Deploy multiple node instances:

**Docker Swarm**:
```bash
docker service create \
  --name honeybee-nodes \
  --replicas 5 \
  honeybee-node:latest
```

**Kubernetes**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: honeybee-node
spec:
  replicas: 5
```

### Resource Allocation

**Minimum**:
- CPU: 0.1 cores
- Memory: 20 MB
- Disk: 100 MB

**Recommended**:
- CPU: 0.25 cores
- Memory: 50 MB
- Disk: 500 MB

## Updates & Maintenance

### Rolling Updates

**Zero-downtime updates**:

1. Deploy new version
2. Test on staging
3. Roll out gradually
4. Monitor for issues
5. Rollback if needed

**Kubernetes**:
```bash
kubectl set image deployment/honeybee-node \
  honeybee-node=honeybee-node:v1.1.0

kubectl rollout status deployment/honeybee-node
```

### Blue-Green Deployment

1. Deploy new version alongside old
2. Test new version
3. Switch traffic to new version
4. Keep old version for quick rollback

### Certificate Rotation

```bash
# 1. Generate new certificates
./generate-certs.sh

# 2. Deploy to nodes
ansible-playbook deploy-certs.yml

# 3. Rolling restart
kubectl rollout restart deployment/honeybee-node
```

## Disaster Recovery

### Backup Strategy

**What to backup**:
- Configuration files
- TLS certificates
- TOTP secrets
- Log archives

**Backup script**:
```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/honeybee/$DATE"

mkdir -p "$BACKUP_DIR"

# Backup config
cp /etc/honeybee/config.yaml "$BACKUP_DIR/"

# Backup certs
tar czf "$BACKUP_DIR/certs.tar.gz" /etc/honeybee/certs/

# Backup secrets
cp /var/lib/honeybee/secrets/.honeybee_totp_secret "$BACKUP_DIR/"

# Encrypt
gpg -c "$BACKUP_DIR"/*
```

### Recovery Procedure

1. Install honeybee-node
2. Restore configuration
3. Restore certificates
4. Restore TOTP secrets
5. Start service
6. Verify connection

## Troubleshooting Deployment

### Service Won't Start

```bash
# Check service status
sudo systemctl status honeybee-node

# Check logs
sudo journalctl -u honeybee-node -n 50

# Verify configuration
./honeybee-node -config /etc/honeybee/config.yaml

# Check permissions
ls -la /etc/honeybee/config.yaml
```

### Connection Issues

```bash
# Test network connectivity
nc -zv manager.example.com 9001

# Test DNS
nslookup manager.example.com

# Check firewall
sudo ufw status
```

### Certificate Issues

```bash
# Verify certificates
openssl verify -CAfile /etc/honeybee/certs/ca.crt \
  /etc/honeybee/certs/client.crt

# Check permissions
ls -la /etc/honeybee/certs/
```

## Next Steps

- [Systemd deployment details](./deployment-systemd.md)
- [Docker deployment details](./deployment-docker.md)
- [Kubernetes deployment details](./deployment-k8s.md)
- [Troubleshooting guide](./troubleshooting.md)

