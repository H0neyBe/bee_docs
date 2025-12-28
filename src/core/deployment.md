# HoneyBee Core Deployment

Production deployment guide for HoneyBee Core.

## Deployment Methods

### Docker (Recommended)

Easiest and most portable:

```bash
# Build image
docker build -t honeybee-core:latest .

# Run container
docker run -d \
  --name honeybee-core \
  -p 9001:9001 \
  -p 9002:9002 \
  -p 9003:9003 \
  -v $(pwd)/bee_config.toml:/app/bee_config.toml:ro \
  -v honeybee-logs:/app/logs \
  honeybee-core:latest
```

### Systemd Service

For Linux systems:

```bash
# Copy binary
sudo cp target/release/honeybee_core /usr/local/bin/
sudo chmod +x /usr/local/bin/honeybee_core

# Create systemd service
sudo tee /etc/systemd/system/honeybee-core.service > /dev/null <<EOF
[Unit]
Description=HoneyBee Core
After=network.target

[Service]
Type=simple
User=honeybee
ExecStart=/usr/local/bin/honeybee_core
WorkingDirectory=/etc/honeybee
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# Create user and directories
sudo useradd -r -s /bin/false honeybee
sudo mkdir -p /etc/honeybee /var/log/honeybee
sudo chown honeybee:honeybee /etc/honeybee /var/log/honeybee

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable honeybee-core
sudo systemctl start honeybee-core
```

## Security Considerations

### Firewall

Configure firewall to allow only necessary ports:

```bash
# Allow node connections
sudo ufw allow 9001/tcp

# Allow backend API (restrict to internal network)
sudo ufw allow from 10.0.0.0/8 to any port 9002

# Allow WebSocket (restrict as needed)
sudo ufw allow 9003/tcp
```

### TLS (Future)

TLS support for node connections is planned. Currently, nodes can use TLS when connecting.

### Non-root User

Always run Core as a non-root user:

```bash
sudo useradd -r -s /bin/false honeybee
sudo chown -R honeybee:honeybee /usr/local/bin/honeybee_core /etc/honeybee
```

## Monitoring

### Health Checks

```bash
# Check API health
curl http://localhost:9002/health

# Check process
systemctl status honeybee-core
```

### Logs

```bash
# View logs
tail -f /var/log/honeybee/debug.log

# Or with systemd
journalctl -u honeybee-core -f
```

## Scaling

### Horizontal Scaling

Deploy multiple Core instances behind a load balancer:

- Use sticky sessions for node connections
- Share state via database (future feature)
- Load balance API requests

### Vertical Scaling

Core is multi-threaded and will use available CPU cores automatically.

## Backup

### Configuration

Backup `bee_config.toml`:

```bash
cp bee_config.toml bee_config.toml.backup
```

### Logs

Rotate logs regularly:

```bash
# Use logrotate
sudo logrotate -f /etc/logrotate.d/honeybee-core
```

## Troubleshooting

### Core Won't Start

1. Check configuration file syntax
2. Verify ports are available
3. Check logs for errors
4. Ensure user has permissions

### Nodes Can't Connect

1. Check firewall rules
2. Verify Core is listening: `netstat -tuln | grep 9001`
3. Check Core logs for connection attempts
4. Verify network connectivity

## Next Steps

- [Docker Deployment](./deployment-docker.md) - Detailed Docker setup
- [Systemd Deployment](./deployment-systemd.md) - Detailed systemd setup
- [Architecture](./architecture.md) - Understand Core internals

