# Security Setup

Security is paramount for honeypot deployments. This guide walks you through securing your HoneyBee node for production use.

## Security Overview

HoneyBee node implements defense-in-depth security:

```
Layer 1: Network Security (Firewall, VPN)
Layer 2: TLS 1.3 Encryption
Layer 3: Certificate Validation
Layer 4: TOTP Authentication
Layer 5: Application Security
```

## Quick Security Checklist

Before deploying to production:

- [ ] TLS encryption enabled
- [ ] Valid certificates installed
- [ ] Certificate verification enabled (`insecure_skip_verify: false`)
- [ ] TOTP authentication enabled
- [ ] TOTP secrets secured (0600 permissions)
- [ ] Running as non-root user
- [ ] Firewall configured
- [ ] Log rotation configured
- [ ] Secrets backed up
- [ ] Monitoring enabled

## TLS Encryption

### Why TLS?

- **Confidentiality**: Encrypts all communication
- **Integrity**: Prevents tampering
- **Authentication**: Verifies server identity
- **Compliance**: Meets security standards

### Enable TLS

```yaml
tls:
  enabled: true
  ca_file: "/etc/honeybee/certs/ca.crt"
  cert_file: "/etc/honeybee/certs/client.crt"  # Optional (mutual TLS)
  key_file: "/etc/honeybee/certs/client.key"   # Optional (mutual TLS)
  insecure_skip_verify: false
  server_name: "honeybee-manager"
```

### TLS Configuration

The node uses TLS 1.3 with strong cipher suites:

- `TLS_AES_256_GCM_SHA384`
- `TLS_AES_128_GCM_SHA256`
- `TLS_CHACHA20_POLY1305_SHA256`

**See**: [TLS Setup Guide](./tls.md) for detailed instructions.

## TOTP Authentication

### Why TOTP?

- **Time-based**: Codes expire every 30 seconds
- **No shared secrets**: Secret never transmitted
- **Standards-based**: RFC 6238 compliant
- **Revocable**: Easy to reset

### Enable TOTP

```yaml
auth:
  totp_enabled: true
  totp_secret_dir: "/var/lib/honeybee/secrets"
```

### TOTP Workflow

1. **First Connection**: Node generates random secret
2. **Registration**: Secret used to generate TOTP code
3. **Validation**: Manager validates code
4. **Storage**: Secret saved for future connections

**See**: [TOTP Setup Guide](./totp.md) for detailed instructions.

## File Permissions

### Configuration Files

```bash
# Config file: readable by node user only
chmod 600 /etc/honeybee/config.yaml
chown honeybee:honeybee /etc/honeybee/config.yaml
```

### TLS Certificates

```bash
# Private keys: read-only by node user
chmod 600 /etc/honeybee/certs/*.key
chown honeybee:honeybee /etc/honeybee/certs/*.key

# Certificates: readable by all
chmod 644 /etc/honeybee/certs/*.crt
```

### TOTP Secrets

```bash
# Secret directory: accessible only by node user
chmod 700 /var/lib/honeybee/secrets
chown honeybee:honeybee /var/lib/honeybee/secrets

# Secret file: read/write by node user only
chmod 600 /var/lib/honeybee/secrets/.honeybee_totp_secret
```

### Log Files

```bash
# Log directory
chmod 755 /var/log/honeybee
chown honeybee:honeybee /var/log/honeybee

# Log files
chmod 644 /var/log/honeybee/*.log
chown honeybee:honeybee /var/log/honeybee/*.log
```

## Running as Non-Root

**Never run as root!** Create a dedicated user:

```bash
# Create honeybee user
sudo useradd -r -s /bin/false honeybee

# Create necessary directories
sudo mkdir -p /etc/honeybee /var/lib/honeybee/secrets /var/log/honeybee
sudo chown -R honeybee:honeybee /etc/honeybee /var/lib/honeybee /var/log/honeybee

# Set binary permissions
sudo chown root:root /usr/local/bin/honeybee-node
sudo chmod 755 /usr/local/bin/honeybee-node
```

## Network Security

### Firewall Configuration

```bash
# Allow outbound to manager only
sudo ufw allow out to 10.0.0.5 port 9001 proto tcp

# Deny all other outbound (optional)
sudo ufw default deny outgoing

# Allow SSH (management)
sudo ufw allow 22/tcp

# Enable firewall
sudo ufw enable
```

### VPN/Tunnel

For additional security, use a VPN or SSH tunnel:

**WireGuard Example:**
```bash
# Install WireGuard
sudo apt install wireguard

# Configure VPN
# /etc/wireguard/wg0.conf

# Connect
sudo wg-quick up wg0

# Configure node to use VPN address
# config.yaml:
# server:
#   address: "10.200.0.1:9001"  # VPN address
```

## Security Hardening

### System Hardening

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install security updates automatically
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Enable firewall
sudo ufw enable

# Disable unnecessary services
sudo systemctl disable <service>
```

### Systemd Hardening

Add to `/etc/systemd/system/honeybee-node.service`:

```ini
[Service]
# Security hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/log/honeybee /var/lib/honeybee
ProtectKernelTunables=true
ProtectControlGroups=true
RestrictRealtime=true
RestrictNamespaces=true
```

### AppArmor Profile

Create `/etc/apparmor.d/usr.local.bin.honeybee-node`:

```
#include <tunables/global>

/usr/local/bin/honeybee-node {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  
  /usr/local/bin/honeybee-node r,
  /etc/honeybee/** r,
  /var/lib/honeybee/** rw,
  /var/log/honeybee/** rw,
  
  # Network
  network inet stream,
  network inet6 stream,
  
  # Deny everything else
  /** deny,
}
```

## Monitoring & Auditing

### Enable Audit Logging

```yaml
log:
  level: "info"
  format: "json"  # Easier to parse
  file: "/var/log/honeybee/node.log"
```

### Monitor Security Events

```bash
# Watch for failed connections
sudo journalctl -u honeybee-node -f | grep "failed"

# Monitor authentication
sudo tail -f /var/log/honeybee/node.log | grep "authentication"

# Check for errors
sudo tail -f /var/log/honeybee/node.log | grep "ERROR"
```

### Set Up Alerts

Use tools like `fail2ban` or custom scripts:

```bash
# Example: Alert on repeated failures
#!/bin/bash
tail -f /var/log/honeybee/node.log | while read line; do
  if echo "$line" | grep -q "authentication failed"; then
    # Send alert
    echo "Security alert: Authentication failure" | mail -s "HoneyBee Alert" admin@example.com
  fi
done
```

## Backup & Recovery

### Backup TOTP Secrets

```bash
# Backup secret
sudo cp /var/lib/honeybee/secrets/.honeybee_totp_secret \
       /backup/honeybee/totp_secret_$(date +%Y%m%d).backup

# Encrypt backup
gpg -c /backup/honeybee/totp_secret_$(date +%Y%m%d).backup
```

### Backup Certificates

```bash
# Backup certificates
sudo tar czf /backup/honeybee/certs_$(date +%Y%m%d).tar.gz \
       /etc/honeybee/certs/

# Encrypt
gpg -c /backup/honeybee/certs_$(date +%Y%m%d).tar.gz
```

### Recovery

```bash
# Restore TOTP secret
sudo cp /backup/honeybee/totp_secret.backup \
       /var/lib/honeybee/secrets/.honeybee_totp_secret
sudo chmod 600 /var/lib/honeybee/secrets/.honeybee_totp_secret
sudo chown honeybee:honeybee /var/lib/honeybee/secrets/.honeybee_totp_secret

# Restore certificates
sudo tar xzf /backup/honeybee/certs.tar.gz -C /
```

## Security Incident Response

### If Compromised

1. **Immediate Actions**:
   ```bash
   # Stop the node
   sudo systemctl stop honeybee-node
   
   # Isolate from network
   sudo ufw deny out to manager.example.com port 9001
   ```

2. **Investigate**:
   ```bash
   # Check logs
   sudo journalctl -u honeybee-node --since "1 hour ago"
   
   # Check file changes
   sudo find /etc/honeybee -type f -mtime -1
   ```

3. **Recovery**:
   ```bash
   # Reset TOTP
   sudo rm /var/lib/honeybee/secrets/.honeybee_totp_secret
   
   # Regenerate certificates if needed
   # (See TLS setup guide)
   
   # Restart with clean state
   sudo systemctl start honeybee-node
   ```

## Security Best Practices Summary

### Do's ✅

- ✅ Enable TLS encryption
- ✅ Enable TOTP authentication
- ✅ Use valid certificates
- ✅ Run as non-root user
- ✅ Set proper file permissions
- ✅ Configure firewall
- ✅ Monitor logs
- ✅ Regular backups
- ✅ Keep software updated
- ✅ Use strong passwords/secrets

### Don'ts ❌

- ❌ Disable TLS in production
- ❌ Disable TOTP in production
- ❌ Skip certificate verification
- ❌ Run as root
- ❌ Use weak permissions
- ❌ Expose to public internet
- ❌ Ignore security updates
- ❌ Store secrets in version control
- ❌ Use default credentials
- ❌ Disable logging

## Security Resources

- [TLS Setup Guide](./tls.md)
- [TOTP Setup Guide](./totp.md)
- [Deployment Guide](./deployment.md)
- [Troubleshooting](./troubleshooting.md)

## Compliance

### GDPR Considerations

- Minimize data collection
- Implement data retention policies
- Provide audit trails
- Ensure secure data storage

### PCI-DSS Considerations

- Use strong encryption (TLS 1.3)
- Implement access controls
- Monitor and log access
- Regular security assessments

## Next Steps

1. [Set up TLS certificates](./tls.md)
2. [Configure TOTP authentication](./totp.md)
3. [Deploy securely](./deployment.md)

