# Systemd Deployment

Deploy HoneyBee Core as a systemd service on Linux.

## Prerequisites

- Linux system with systemd
- Rust binary built: `target/release/honeybee_core`
- Configuration file: `bee_config.toml`

## Installation Steps

### 1. Create User

```bash
sudo useradd -r -s /bin/false honeybee
```

### 2. Create Directories

```bash
sudo mkdir -p /usr/local/bin
sudo mkdir -p /etc/honeybee
sudo mkdir -p /var/log/honeybee
```

### 3. Install Binary

```bash
sudo cp target/release/honeybee_core /usr/local/bin/honeybee_core
sudo chmod +x /usr/local/bin/honeybee_core
sudo chown honeybee:honeybee /usr/local/bin/honeybee_core
```

### 4. Install Configuration

```bash
sudo cp bee_config.toml /etc/honeybee/bee_config.toml
sudo chmod 644 /etc/honeybee/bee_config.toml
sudo chown honeybee:honeybee /etc/honeybee/bee_config.toml
```

### 5. Create Service File

```bash
sudo tee /etc/systemd/system/honeybee-core.service > /dev/null <<EOF
[Unit]
Description=HoneyBee Core Manager
After=network.target

[Service]
Type=simple
User=honeybee
Group=honeybee
WorkingDirectory=/etc/honeybee
ExecStart=/usr/local/bin/honeybee_core
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

# Security settings
NoNewPrivileges=true
PrivateTmp=true

# Resource limits
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### 6. Set Permissions

```bash
sudo chown -R honeybee:honeybee /etc/honeybee /var/log/honeybee
sudo chmod 755 /etc/honeybee
sudo chmod 755 /var/log/honeybee
```

### 7. Enable and Start

```bash
sudo systemctl daemon-reload
sudo systemctl enable honeybee-core
sudo systemctl start honeybee-core
```

## Service Management

### Start Service

```bash
sudo systemctl start honeybee-core
```

### Stop Service

```bash
sudo systemctl stop honeybee-core
```

### Restart Service

```bash
sudo systemctl restart honeybee-core
```

### Check Status

```bash
sudo systemctl status honeybee-core
```

### View Logs

```bash
# Journal logs
sudo journalctl -u honeybee-core -f

# Or view last 100 lines
sudo journalctl -u honeybee-core -n 100
```

## Configuration

Edit configuration:

```bash
sudo nano /etc/honeybee/bee_config.toml
sudo systemctl restart honeybee-core
```

## Log Rotation

Create logrotate config:

```bash
sudo tee /etc/logrotate.d/honeybee-core > /dev/null <<EOF
/var/log/honeybee/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 honeybee honeybee
    sharedscripts
    postrotate
        systemctl reload honeybee-core > /dev/null 2>&1 || true
    endscript
}
EOF
```

## Troubleshooting

### Service Won't Start

```bash
# Check status
sudo systemctl status honeybee-core

# Check logs
sudo journalctl -u honeybee-core -n 50

# Check configuration
sudo /usr/local/bin/honeybee_core --help
```

### Permission Issues

```bash
# Fix ownership
sudo chown -R honeybee:honeybee /etc/honeybee /var/log/honeybee

# Fix permissions
sudo chmod 755 /etc/honeybee
sudo chmod 755 /var/log/honeybee
```

### Port Already in Use

```bash
# Find process using port
sudo lsof -i :9001

# Kill process (if needed)
sudo kill <PID>
```

## Next Steps

- [Configuration](./configuration.md) - Configure Core
- [Deployment](./deployment.md) - Other deployment methods

