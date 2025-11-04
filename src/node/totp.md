# TOTP Authentication Setup

Complete guide to setting up Time-based One-Time Password (TOTP) authentication for your HoneyBee node.

## Overview

TOTP provides an additional security layer beyond TLS encryption:
- **Time-based**: 6-digit codes valid for 30 seconds
- **No transmission**: Secret never sent over network
- **Standard**: RFC 6238 compliant
- **Revocable**: Easy to reset if compromised

## How TOTP Works

```
First Connection:
1. Node generates random 160-bit secret
2. Node saves secret locally
3. Node generates TOTP code from secret
4. Node sends code with registration
5. Manager validates and stores secret hash

Subsequent Connections:
1. Node loads existing secret
2. Node generates fresh TOTP code
3. Node sends code with registration
4. Manager validates against stored secret
```

## Configuration

```yaml
auth:
  totp_enabled: true
  totp_secret_dir: "/var/lib/honeybee/secrets"  # Optional
```

### Configuration Options

#### `totp_enabled` (required)

Enable or disable TOTP authentication.

⚠️ **Always `true` in production!**

```yaml
totp_enabled: true  # Production
totp_enabled: false # Testing only
```

#### `totp_secret_dir` (optional)

Directory where TOTP secret is stored.

**Default**: `~/.config/honeybee/`

**Examples**:
```yaml
totp_secret_dir: "/var/lib/honeybee/secrets"  # System-wide
totp_secret_dir: "/etc/honeybee/secrets"       # Alternative
totp_secret_dir: "~/.honeybee/secrets"         # User directory
```

## First Time Setup

### Automatic Setup

TOTP is set up automatically on first connection:

```bash
# 1. Enable TOTP in config
# config.yaml:
# auth:
#   totp_enabled: true

# 2. Start node
./honeybee-node -config configs/config.yaml

# The node will:
# - Generate random secret
# - Save to ~/.config/honeybee/.honeybee_totp_secret
# - Generate TOTP code
# - Send to manager for validation
```

Output:
```
INFO: Generated new TOTP secret (first-time registration)
INFO: TOTP code generated: 123456
INFO: Connected to server successfully
INFO: Registration accepted
```

### Manual Setup

For advanced use cases:

```bash
# 1. Create secret directory
mkdir -p ~/.config/honeybee
chmod 700 ~/.config/honeybee

# 2. Generate secret (160 bits, base32 encoded)
head -c 20 /dev/urandom | base32 > ~/.config/honeybee/.honeybee_totp_secret

# 3. Set permissions
chmod 600 ~/.config/honeybee/.honeybee_totp_secret

# 4. Start node
./honeybee-node -config configs/config.yaml
```

## Secret Storage

### File Location

The TOTP secret is stored in:
```
~/.config/honeybee/.honeybee_totp_secret    # User installation
/var/lib/honeybee/secrets/.honeybee_totp_secret  # System-wide
```

### File Format

Plain text, base32 encoded:
```
JBSWY3DPEHPK3PXP
```

### Security

**Critical**: Proper permissions are essential!

```bash
# Secret directory: accessible only by node user
chmod 700 /var/lib/honeybee/secrets

# Secret file: read/write only by node user
chmod 600 /var/lib/honeybee/secrets/.honeybee_totp_secret

# Ownership
chown honeybee:honeybee /var/lib/honeybee/secrets
chown honeybee:honeybee /var/lib/honeybee/secrets/.honeybee_totp_secret
```

## Verification

### Verify Secret Exists

```bash
# Check secret file
ls -la ~/.config/honeybee/.honeybee_totp_secret

# Should show:
# -rw------- 1 user user ... .honeybee_totp_secret
```

### Verify TOTP Generation

The node logs show TOTP usage:

```bash
# Run node
./honeybee-node -config configs/config.yaml

# Check logs for:
# "Using existing TOTP secret"
# "TOTP code generated: ******"
```

### Test Authentication

```bash
# Start node
./honeybee-node -config configs/config.yaml

# Successful: "Registration accepted"
# Failed: "Registration rejected" or timeout
```

## Resetting TOTP

### When to Reset

- First deployment to new manager
- Security compromise suspected
- Lost secret file
- Testing/development

### How to Reset

```bash
# 1. Stop node
sudo systemctl stop honeybee-node

# 2. Delete secret
rm ~/.config/honeybee/.honeybee_totp_secret
# or
sudo rm /var/lib/honeybee/secrets/.honeybee_totp_secret

# 3. Start node (will generate new secret)
sudo systemctl start honeybee-node

# 4. Verify in logs
sudo journalctl -u honeybee-node -f
# Look for: "Generated new TOTP secret"
```

## Backup & Recovery

### Backup Secret

**Critical**: Backup your TOTP secrets!

```bash
# Backup secret
sudo cp /var/lib/honeybee/secrets/.honeybee_totp_secret \
       /backup/honeybee/totp_$(hostname)_$(date +%Y%m%d).secret

# Encrypt backup (recommended)
gpg -c /backup/honeybee/totp_$(hostname)_$(date +%Y%m%d).secret

# Secure storage
chmod 600 /backup/honeybee/*.secret
```

### Restore Secret

```bash
# Decrypt if needed
gpg -d /backup/honeybee/totp_hostname_20240101.secret.gpg > totp.secret

# Restore
sudo cp totp.secret /var/lib/honeybee/secrets/.honeybee_totp_secret

# Set permissions
sudo chmod 600 /var/lib/honeybee/secrets/.honeybee_totp_secret
sudo chown honeybee:honeybee /var/lib/honeybee/secrets/.honeybee_totp_secret

# Restart node
sudo systemctl restart honeybee-node

# Clean up
rm totp.secret
```

## Using with Authenticator Apps

While not required, you can import the secret into authenticator apps for manual verification:

```bash
# Generate QR code for authenticator app
qrencode -o totp-qr.png "otpauth://totp/HoneyBee:node-01?secret=$(cat ~/.config/honeybee/.honeybee_totp_secret)&issuer=HoneyBee"

# View QR code
display totp-qr.png

# Or manually enter secret in app
cat ~/.config/honeybee/.honeybee_totp_secret
```

This allows you to:
- Generate codes manually
- Verify code generation
- Troubleshoot authentication issues

## Common Issues

### Authentication Failed

**Problem**: "Registration rejected" or "TOTP authentication failed"

**Causes**:
1. Time synchronization issue
2. Secret mismatch between node and manager
3. Manager not configured for TOTP

**Solutions**:

```bash
# 1. Check time synchronization
timedatectl status

# 2. Sync time
sudo ntpdate -s time.nist.gov
# or
sudo systemctl restart systemd-timesyncd

# 3. Verify timezone
timedatectl set-timezone UTC

# 4. Reset secret
rm ~/.config/honeybee/.honeybee_totp_secret
./honeybee-node -config configs/config.yaml
```

### Secret Not Found

**Problem**: Node can't find TOTP secret file

**Solutions**:

```bash
# 1. Check secret directory exists
mkdir -p ~/.config/honeybee
chmod 700 ~/.config/honeybee

# 2. Check permissions
ls -la ~/.config/honeybee/

# 3. Run node (will generate new secret)
./honeybee-node -config configs/config.yaml
```

### Permission Denied

**Problem**: Can't read/write secret file

**Solutions**:

```bash
# Fix ownership
sudo chown -R honeybee:honeybee /var/lib/honeybee/secrets

# Fix permissions
sudo chmod 700 /var/lib/honeybee/secrets
sudo chmod 600 /var/lib/honeybee/secrets/.honeybee_totp_secret
```

### Time Drift

**Problem**: TOTP codes don't match due to time difference

**Solutions**:

```bash
# Install NTP
sudo apt install ntp

# Enable time synchronization
sudo systemctl enable ntp
sudo systemctl start ntp

# Force immediate sync
sudo ntpdate -s time.nist.gov

# Verify
timedatectl status
```

## Security Best Practices

### 1. Secure Storage

```bash
# Proper permissions
chmod 700 /var/lib/honeybee/secrets
chmod 600 /var/lib/honeybee/secrets/.honeybee_totp_secret
```

### 2. Regular Backups

```bash
# Automated backup script
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/honeybee"

mkdir -p "$BACKUP_DIR"
cp /var/lib/honeybee/secrets/.honeybee_totp_secret \
   "$BACKUP_DIR/totp_$DATE.secret"
gpg -c "$BACKUP_DIR/totp_$DATE.secret"
rm "$BACKUP_DIR/totp_$DATE.secret"
```

### 3. Encrypted Backups

Always encrypt backups:
```bash
gpg -c secret.backup  # Encrypt
gpg -d secret.backup.gpg  # Decrypt
```

### 4. Secure Transmission

Never transmit secrets in plain text:
- Use encrypted channels
- Use secure key management systems
- Never commit to version control

### 5. Rotation Schedule

Rotate secrets periodically:
```bash
# Every 90 days (example)
0 0 1 */3 * /usr/local/bin/rotate-totp-secret.sh
```

### 6. Audit Access

Monitor secret file access:
```bash
# Add audit rule
sudo auditctl -w /var/lib/honeybee/secrets/.honeybee_totp_secret -p rwa

# View audit logs
sudo ausearch -f /var/lib/honeybee/secrets/.honeybee_totp_secret
```

## Multi-Node Deployment

### Unique Secrets Per Node

Each node should have its own secret:

```bash
# Node 1
./honeybee-node -config node1-config.yaml

# Node 2  
./honeybee-node -config node2-config.yaml

# Each generates its own secret
```

### Centralized Secret Management

For large deployments, use secret management:

**HashiCorp Vault Example**:
```bash
# Store secret in Vault
vault kv put secret/honeybee/node1 totp="JBSWY3DPEHPK3PXP"

# Retrieve in startup script
SECRET=$(vault kv get -field=totp secret/honeybee/node1)
echo "$SECRET" > /var/lib/honeybee/secrets/.honeybee_totp_secret
```

## Monitoring

### Log TOTP Events

The node logs TOTP-related events:

```bash
# Monitor TOTP usage
sudo journalctl -u honeybee-node -f | grep -i totp

# Common log messages:
# "Generated new TOTP secret"
# "Using existing TOTP secret"
# "TOTP code generated"
# "Registration accepted"
```

### Alert on Failures

```bash
# Alert script
#!/bin/bash
tail -f /var/log/honeybee/node.log | while read line; do
  if echo "$line" | grep -q "authentication failed"; then
    echo "TOTP auth failed!" | mail -s "Alert" admin@example.com
  fi
done
```

## Next Steps

- [Complete security setup](./security.md)
- [Deploy to production](./deployment.md)
- [Troubleshooting authentication issues](./troubleshooting.md)

