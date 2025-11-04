# Systemd Service Deployment

Detailed guide for deploying HoneyBee node as a systemd service on Linux.

## Overview

Systemd is the standard init system for modern Linux distributions. Running HoneyBee node as a systemd service provides:

- Automatic start on boot
- Service management (start/stop/restart)
- Log integration with journald
- Resource limits and security hardening
- Service dependencies
- Automatic restart on failure

## Installation Steps

### 1. Build the Binary

```bash
cd honeybee_node
make build
```

### 2. Install Binary

```bash
sudo cp build/honeybee-node /usr/local/bin/
sudo chmod +x /usr/local/bin/honeybee-node
```

### 3. Create Service User

```bash
sudo useradd -r -s /bin/false -d /var/lib/honeybee honeybee
```

### 4. Create Directory Structure

```bash
sudo mkdir -p /etc/honeybee
sudo mkdir -p /etc/honeybee/certs
sudo mkdir -p /var/lib/honeybee/secrets
sudo mkdir -p /var/log/honeybee

sudo chown -R honeybee:honeybee /etc/honeybee
sudo chown -R honeybee:honeybee /var/lib/honeybee
sudo chown -R honeybee:honeybee /var/log/honeybee
```

### 5. Install Configuration

```bash
sudo cp configs/config.yaml /etc/honeybee/
sudo chown honeybee:honeybee /etc/honeybee/config.yaml
sudo chmod 600 /etc/honeybee/config.yaml
```

Edit `/etc/honeybee/config.yaml` for your environment.

### 6. Install Certificates

```bash
sudo cp certs/*.crt /etc/honeybee/certs/
sudo cp certs/*.key /etc/honeybee/certs/
sudo chown honeybee:honeybee /etc/honeybee/certs/*
sudo chmod 600 /etc/honeybee/certs/*.key
sudo chmod 644 /etc/honeybee/certs/*.crt
```

### 7. Create Service File

Create `/etc/systemd/system/honeybee-node.service`:

```ini
[Unit]
Description=HoneyBee Node - Honeypot Agent
Documentation=https://github.com/yourusername/honeybee
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=honeybee
Group=honeybee

ExecStart=/usr/local/bin/honeybee-node -config /etc/honeybee/config.yaml
Restart=on-failure
RestartSec=10s

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

# Resource limits
LimitNOFILE=65536
TasksMax=4096

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=honeybee-node

[Install]
WantedBy=multi-user.target
```

### 8. Enable and Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable honeybee-node
sudo systemctl start honeybee-node
```

### 9. Verify Service

```bash
sudo systemctl status honeybee-node
```

## Service Management

### Basic Commands

```bash
# Start service
sudo systemctl start honeybee-node

# Stop service
sudo systemctl stop honeybee-node

# Restart service
sudo systemctl restart honeybee-node

# Reload configuration (if supported)
sudo systemctl reload honeybee-node

# Enable auto-start on boot
sudo systemctl enable honeybee-node

# Disable auto-start
sudo systemctl disable honeybee-node

# Check status
sudo systemctl status honeybee-node

# Check if enabled
sudo systemctl is-enabled honeybee-node

# Check if active
sudo systemctl is-active honeybee-node
```

### View Logs

```bash
# Follow logs in real-time
sudo journalctl -u honeybee-node -f

# View last 100 lines
sudo journalctl -u honeybee-node -n 100

# View logs since last boot
sudo journalctl -u honeybee-node -b

# View logs for specific time period
sudo journalctl -u honeybee-node --since "1 hour ago"
sudo journalctl -u honeybee-node --since "2024-01-01" --until "2024-01-02"
```

## Advanced Configuration

### Service Override

Create custom overrides without modifying main service file:

```bash
sudo systemctl edit honeybee-node
```

This creates `/etc/systemd/system/honeybee-node.service.d/override.conf`.

Example override:
```ini
[Service]
Environment="DEBUG=1"
RestartSec=5s
```

### Resource Limits

Add to service file:

```ini
[Service]
# CPU limit (50%)
CPUQuota=50%

# Memory limit
MemoryLimit=512M
MemoryMax=1G

# I/O priority (best-effort, level 4)
IOSchedulingClass=best-effort
IOSchedulingPriority=4

# Process limits
TasksMax=512
LimitNOFILE=65536
```

### Dependencies

Ensure service starts after other services:

```ini
[Unit]
After=network-online.target ntp.service
Requires=network-online.target
```

## Troubleshooting

### Service Won't Start

```bash
# Check service status
sudo systemctl status honeybee-node

# View full logs
sudo journalctl -u honeybee-node -n 100 --no-pager

# Test configuration
sudo -u honeybee /usr/local/bin/honeybee-node \
  -config /etc/honeybee/config.yaml

# Check file permissions
ls -la /usr/local/bin/honeybee-node
ls -la /etc/honeybee/
```

### Check for Errors

```bash
# Check for failed units
systemctl --failed

# Analyze service
systemd-analyze verify honeybee-node.service

# Check for dependency issues
systemctl list-dependencies honeybee-node
```

### Permission Issues

```bash
# Verify ownership
sudo chown -R honeybee:honeybee /etc/honeybee
sudo chown -R honeybee:honeybee /var/lib/honeybee
sudo chown -R honeybee:honeybee /var/log/honeybee

# Verify permissions
sudo chmod 700 /var/lib/honeybee/secrets
sudo chmod 600 /etc/honeybee/config.yaml
sudo chmod 600 /etc/honeybee/certs/*.key
```

## Monitoring

### Health Check Script

Create `/usr/local/bin/honeybee-health-check.sh`:

```bash
#!/bin/bash

# Check if service is active
if ! systemctl is-active --quiet honeybee-node; then
    echo "CRITICAL: honeybee-node is not running"
    exit 2
fi

# Check recent errors
ERROR_COUNT=$(journalctl -u honeybee-node --since "5 minutes ago" | grep -c ERROR)
if [ "$ERROR_COUNT" -gt 10 ]; then
    echo "WARNING: $ERROR_COUNT errors in last 5 minutes"
    exit 1
fi

echo "OK: honeybee-node is healthy"
exit 0
```

### Timer for Health Checks

Create `/etc/systemd/system/honeybee-health-check.timer`:

```ini
[Unit]
Description=HoneyBee Node Health Check Timer

[Timer]
OnCalendar=*:0/5
Persistent=true

[Install]
WantedBy=timers.target
```

## Log Rotation

Create `/etc/logrotate.d/honeybee-node`:

```
/var/log/honeybee/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0644 honeybee honeybee
    postrotate
        systemctl reload honeybee-node > /dev/null 2>&1 || true
    endscript
}
```

## Uninstallation

```bash
# Stop and disable service
sudo systemctl stop honeybee-node
sudo systemctl disable honeybee-node

# Remove service file
sudo rm /etc/systemd/system/honeybee-node.service
sudo systemctl daemon-reload

# Remove files
sudo rm /usr/local/bin/honeybee-node
sudo rm -rf /etc/honeybee
sudo rm -rf /var/lib/honeybee
sudo rm -rf /var/log/honeybee

# Remove user
sudo userdel honeybee
```

## Next Steps

- [Configuration Guide](./configuration.md)
- [Security Setup](./security.md)
- [Troubleshooting](./troubleshooting.md)

