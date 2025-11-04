# TLS Encryption Setup

Complete guide to setting up TLS 1.3 encryption for your HoneyBee node.

## Overview

TLS (Transport Layer Security) provides:
- **Encryption**: All data encrypted in transit
- **Authentication**: Verify server identity
- **Integrity**: Detect tampering

The HoneyBee node uses **TLS 1.3** with strong cipher suites for maximum security.

## TLS Configuration

```yaml
tls:
  enabled: true
  cert_file: "/etc/honeybee/certs/client.crt"  # Optional
  key_file: "/etc/honeybee/certs/client.key"   # Optional
  ca_file: "/etc/honeybee/certs/ca.crt"        # Recommended
  insecure_skip_verify: false
  server_name: "honeybee-manager"
```

## Certificate Types

### 1. CA Certificate Only (Recommended for Most Cases)

The simplest secure setup:

```yaml
tls:
  enabled: true
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"
```

**Use when**: You want to verify the server's identity.

### 2. Mutual TLS (mTLS) - Maximum Security

Both client and server authenticate each other:

```yaml
tls:
  enabled: true
  cert_file: "/etc/honeybee/certs/client.crt"
  key_file: "/etc/honeybee/certs/client.key"
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager"
```

**Use when**: Maximum security is required.

### 3. Development Only (No Certificates)

⚠️ **NOT for production!**

```yaml
tls:
  enabled: true
  insecure_skip_verify: true  # Skip verification
```

## Generating Certificates

### Option 1: Self-Signed (Development/Testing)

#### Step 1: Create Certificate Directory

```bash
mkdir -p /etc/honeybee/certs
cd /etc/honeybee/certs
```

#### Step 2: Generate CA Certificate

```bash
# Generate CA private key
openssl genrsa -out ca.key 4096

# Generate CA certificate
openssl req -new -x509 -days 365 -key ca.key -out ca.crt \
  -subj "/C=US/ST=State/L=City/O=HoneyBee/CN=HoneyBee CA"
```

#### Step 3: Generate Server Certificate

```bash
# Generate server private key
openssl genrsa -out server.key 4096

# Create server certificate signing request
openssl req -new -key server.key -out server.csr \
  -subj "/C=US/ST=State/L=City/O=HoneyBee/CN=honeybee-manager"

# Sign with CA
openssl x509 -req -days 365 -in server.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out server.crt
```

#### Step 4: Generate Client Certificate (Optional - for mTLS)

```bash
# Generate client private key
openssl genrsa -out client.key 4096

# Create client certificate signing request
openssl req -new -key client.key -out client.csr \
  -subj "/C=US/ST=State/L=City/O=HoneyBee/CN=honeybee-node"

# Sign with CA
openssl x509 -req -days 365 -in client.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out client.crt
```

#### Step 5: Set Permissions

```bash
# Private keys: read-only by owner
chmod 600 *.key

# Certificates: readable by all
chmod 644 *.crt

# Set ownership
chown honeybee:honeybee *
```

### Option 2: Let's Encrypt (Production)

For production with public domains:

```bash
# Install certbot
sudo apt install certbot

# Generate certificate
sudo certbot certonly --standalone \
  -d manager.example.com

# Certificates will be in:
# /etc/letsencrypt/live/manager.example.com/
```

Configure node:
```yaml
tls:
  enabled: true
  ca_file: "/etc/letsencrypt/live/manager.example.com/chain.pem"
  server_name: "manager.example.com"
```

### Option 3: Corporate PKI

If using corporate certificates:

1. Obtain certificates from your CA
2. Copy to `/etc/honeybee/certs/`
3. Configure paths in `config.yaml`

## Verification

### Verify Certificate Creation

```bash
# Verify CA certificate
openssl x509 -in ca.crt -text -noout

# Verify server certificate
openssl x509 -in server.crt -text -noout

# Verify certificate chain
openssl verify -CAfile ca.crt server.crt
```

### Test TLS Connection

```bash
# Test server TLS
openssl s_client -connect 127.0.0.1:9001 -CAfile ca.crt

# Test with client certificate (mTLS)
openssl s_client -connect 127.0.0.1:9001 \
  -CAfile ca.crt \
  -cert client.crt \
  -key client.key
```

### Test with Node

```bash
# Run node with TLS
./honeybee-node -config configs/config.yaml

# Check logs for:
# "TLS connection established: version=TLS 1.3"
```

## TLS Configuration Examples

### Minimal (CA Only)

```yaml
tls:
  enabled: true
  ca_file: "/etc/honeybee/certs/ca.crt"
  server_name: "honeybee-manager"
```

### Full (Mutual TLS)

```yaml
tls:
  enabled: true
  cert_file: "/etc/honeybee/certs/client.crt"
  key_file: "/etc/honeybee/certs/client.key"
  ca_file: "/etc/honeybee/certs/ca.crt"
  insecure_skip_verify: false
  server_name: "honeybee-manager.internal.example.com"
```

### Development (Insecure)

```yaml
tls:
  enabled: true
  insecure_skip_verify: true  # ⚠️ Testing only!
```

## Common Issues

### Certificate Verification Failed

**Problem**: `TLS handshake failed: certificate verify failed`

**Solutions**:
1. Check CA certificate is correct
2. Verify server name matches certificate CN
3. Ensure certificate is not expired
4. Check system time is synchronized

```bash
# Check certificate expiry
openssl x509 -in /etc/honeybee/certs/ca.crt -noout -dates

# Sync time
sudo ntpdate -s time.nist.gov
```

### Unknown Authority

**Problem**: `x509: certificate signed by unknown authority`

**Solutions**:
1. Provide CA certificate in configuration
2. Or add CA to system trust store

```bash
# Add to system trust (Ubuntu/Debian)
sudo cp ca.crt /usr/local/share/ca-certificates/honeybee-ca.crt
sudo update-ca-certificates
```

### Wrong Host Name

**Problem**: `TLS handshake failed: ... certificate is valid for X, not Y`

**Solutions**:
1. Update `server_name` in config to match certificate
2. Or regenerate certificate with correct CN/SAN

### Connection Refused

**Problem**: Connection refused or timeout

**Solutions**:
1. Ensure manager has TLS enabled
2. Check firewall rules
3. Verify manager is listening on correct port

## Certificate Management

### Certificate Rotation

Plan certificate rotation before expiry:

```bash
# Check expiry
openssl x509 -in server.crt -noout -dates

# Generate new certificates (follow generation steps)

# Update configuration with new paths

# Restart node
sudo systemctl restart honeybee-node
```

### Revocation

If a certificate is compromised:

1. Generate new certificates
2. Update all nodes
3. Restart services

```bash
# Generate new CA and certificates
./generate-certs.sh

# Deploy to all nodes
ansible-playbook deploy-certs.yml

# Restart nodes
ansible honeybee-nodes -a "systemctl restart honeybee-node"
```

## Security Best Practices

1. **Never commit private keys to version control**
   ```bash
   # Add to .gitignore
   echo "*.key" >> .gitignore
   echo "*.pem" >> .gitignore
   ```

2. **Use strong key sizes**
   - Minimum: 2048 bits
   - Recommended: 4096 bits

3. **Regular rotation**
   - Certificates: Every 90-365 days
   - CA: Every 1-3 years

4. **Secure storage**
   ```bash
   chmod 600 /etc/honeybee/certs/*.key
   chmod 700 /etc/honeybee/certs
   ```

5. **Use appropriate CN/SAN**
   - Match actual hostname/IP
   - Include all aliases

## Automation

### Certificate Generation Script

```bash
#!/bin/bash
# generate-certs.sh

CERT_DIR="/etc/honeybee/certs"
DAYS=365

mkdir -p "$CERT_DIR"
cd "$CERT_DIR"

# CA
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days $DAYS -key ca.key -out ca.crt \
  -subj "/CN=HoneyBee CA"

# Server
openssl genrsa -out server.key 4096
openssl req -new -key server.key -out server.csr \
  -subj "/CN=honeybee-manager"
openssl x509 -req -days $DAYS -in server.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

# Client
openssl genrsa -out client.key 4096
openssl req -new -key client.key -out client.csr \
  -subj "/CN=honeybee-node"
openssl x509 -req -days $DAYS -in client.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt

# Permissions
chmod 600 *.key
chmod 644 *.crt

echo "Certificates generated in $CERT_DIR"
```

### Ansible Playbook

```yaml
---
- name: Deploy TLS certificates
  hosts: honeybee_nodes
  become: yes
  tasks:
    - name: Create certs directory
      file:
        path: /etc/honeybee/certs
        state: directory
        owner: honeybee
        group: honeybee
        mode: '0700'
    
    - name: Copy CA certificate
      copy:
        src: certs/ca.crt
        dest: /etc/honeybee/certs/ca.crt
        owner: honeybee
        group: honeybee
        mode: '0644'
    
    - name: Copy client certificate
      copy:
        src: "certs/{{ inventory_hostname }}.crt"
        dest: /etc/honeybee/certs/client.crt
        owner: honeybee
        group: honeybee
        mode: '0644'
    
    - name: Copy client key
      copy:
        src: "certs/{{ inventory_hostname }}.key"
        dest: /etc/honeybee/certs/client.key
        owner: honeybee
        group: honeybee
        mode: '0600'
    
    - name: Restart honeybee-node
      systemd:
        name: honeybee-node
        state: restarted
```

## Next Steps

- [Configure TOTP authentication](./totp.md)
- [Deploy to production](./deployment.md)
- [Troubleshooting TLS issues](./troubleshooting.md)

