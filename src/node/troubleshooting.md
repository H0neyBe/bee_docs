# Troubleshooting Guide

Common issues and solutions for HoneyBee node deployment and operation.

## Quick Diagnostic

```bash
# Check if node is running
ps aux | grep honeybee-node

# Check service status (systemd)
sudo systemctl status honeybee-node

# View recent logs
sudo journalctl -u honeybee-node -n 100 --no-pager

# Test network connectivity
nc -zv manager.example.com 9001

# Verify certificates
openssl verify -CAfile /etc/honeybee/certs/ca.crt \
  /etc/honeybee/certs/server.crt
```

## Connection Issues

### Connection Refused

**Symptoms**:
- `Connection refused`
- `Connection timeout`
- Node can't reach manager

**Causes**:
1. Manager not running
2. Firewall blocking connection
3. Incorrect address/port
4. Network connectivity issue

**Solutions**:

```bash
# 1. Check manager is running
ssh manager-host
ps aux | grep honeybee-core

# 2. Test network connectivity
ping manager.example.com
nc -zv manager.example.com 9001

# 3. Check firewall on node
sudo ufw status
sudo ufw allow out to manager.example.com port 9001

# 4. Check firewall on manager
# (on manager host)
sudo ufw status
sudo ufw allow 9001/tcp

# 5. Verify configuration
cat /etc/honeybee/config.yaml | grep address
```

### Connection Drops Frequently

**Symptoms**:
- Frequent reconnections
- "Connection closed by server"
- Unstable connection

**Causes**:
1. Network instability
2. Manager restarts
3. Keepalive not working
4. Firewall timeout

**Solutions**:

```bash
# 1. Check network quality
ping -c 100 manager.example.com
mtr manager.example.com

# 2. Increase reconnect delay
# config.yaml:
# server:
#   reconnect_delay: 10

# 3. Check manager logs
# (on manager)
sudo journalctl -u honeybee-core -f

# 4. Enable TCP keepalive
# (Usually enabled by default)
```

## TLS Issues

### TLS Handshake Failed

**Symptoms**:
- `TLS handshake failed`
- `Certificate verify failed`
- Connection fails after TCP succeeds

**Solutions**:

```bash
# 1. Test TLS connection
openssl s_client -connect manager.example.com:9001 \
  -CAfile /etc/honeybee/certs/ca.crt

# 2. Verify certificate chain
openssl verify -CAfile /etc/honeybee/certs/ca.crt \
  /etc/honeybee/certs/server.crt

# 3. Check certificate expiry
openssl x509 -in /etc/honeybee/certs/ca.crt -noout -dates

# 4. Verify server name matches certificate
openssl x509 -in /etc/honeybee/certs/server.crt -noout -subject

# 5. Check config matches
cat /etc/honeybee/config.yaml | grep server_name
```

### Certificate Signed by Unknown Authority

**Symptoms**:
- `x509: certificate signed by unknown authority`
- TLS verification fails

**Solutions**:

```bash
# 1. Ensure CA certificate is specified
# config.yaml:
# tls:
#   ca_file: "/etc/honeybee/certs/ca.crt"

# 2. Verify CA certificate file exists
ls -la /etc/honeybee/certs/ca.crt

# 3. Check CA certificate is correct
openssl x509 -in /etc/honeybee/certs/ca.crt -text -noout

# 4. Regenerate if needed
./generate-certs.sh
```

### Certificate Name Mismatch

**Symptoms**:
- `certificate is valid for X, not Y`
- Server name doesn't match certificate

**Solutions**:

```bash
# 1. Check certificate CN/SAN
openssl x509 -in /etc/honeybee/certs/server.crt \
  -noout -subject -ext subjectAltName

# 2. Update config to match
# config.yaml:
# tls:
#   server_name: "honeybee-manager"  # Must match certificate

# 3. Or regenerate certificate with correct CN
openssl req -new -key server.key -out server.csr \
  -subj "/CN=correct-hostname"
```

## Authentication Issues

### TOTP Authentication Failed

**Symptoms**:
- `Registration rejected`
- `TOTP authentication failed`
- Node can connect but can't authenticate

**Causes**:
1. Time synchronization issue
2. Secret mismatch
3. Manager not configured for TOTP

**Solutions**:

```bash
# 1. Check time synchronization
timedatectl status

# 2. Sync time
sudo ntpdate -s time.nist.gov
# or
sudo systemctl restart systemd-timesyncd

# 3. Verify timezone is UTC
timedatectl set-timezone UTC

# 4. Reset TOTP secret
sudo systemctl stop honeybee-node
sudo rm /var/lib/honeybee/secrets/.honeybee_totp_secret
sudo systemctl start honeybee-node

# 5. Check logs for new secret generation
sudo journalctl -u honeybee-node -f | grep TOTP
```

### Time Drift

**Symptoms**:
- Intermittent authentication failures
- Works sometimes, fails other times

**Solutions**:

```bash
# 1. Install NTP
sudo apt install ntp

# 2. Enable time sync
sudo systemctl enable ntp
sudo systemctl start ntp

# 3. Verify time is synchronized
timedatectl status | grep synchronized

# 4. Force sync
sudo ntpdate -s time.nist.gov

# 5. Check time difference
# (both node and manager should be within 30 seconds)
```

## Configuration Issues

### Invalid Configuration

**Symptoms**:
- `Invalid configuration`
- Node won't start
- Configuration parse errors

**Solutions**:

```bash
# 1. Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('/etc/honeybee/config.yaml'))"

# 2. Check for required fields
cat /etc/honeybee/config.yaml

# 3. Use example config
cp configs/config.example.yaml /etc/honeybee/config.yaml

# 4. Test configuration
./honeybee-node -config /etc/honeybee/config.yaml
```

### File Not Found

**Symptoms**:
- `No such file or directory`
- Can't find certificates, config, etc.

**Solutions**:

```bash
# 1. Check file exists
ls -la /etc/honeybee/config.yaml

# 2. Check permissions
ls -la /etc/honeybee/certs/

# 3. Verify paths in config
cat /etc/honeybee/config.yaml | grep _file

# 4. Create missing directories
sudo mkdir -p /etc/honeybee/certs
sudo mkdir -p /var/lib/honeybee/secrets
sudo mkdir -p /var/log/honeybee
```

### Permission Denied

**Symptoms**:
- `Permission denied`
- Can't read/write files

**Solutions**:

```bash
# 1. Check file ownership
ls -la /etc/honeybee/config.yaml

# 2. Fix ownership
sudo chown -R honeybee:honeybee /etc/honeybee
sudo chown -R honeybee:honeybee /var/lib/honeybee
sudo chown -R honeybee:honeybee /var/log/honeybee

# 3. Fix permissions
sudo chmod 600 /etc/honeybee/config.yaml
sudo chmod 600 /etc/honeybee/certs/*.key
sudo chmod 700 /var/lib/honeybee/secrets

# 4. Verify service user
ps aux | grep honeybee-node | grep -v grep
```

## Performance Issues

### High CPU Usage

**Symptoms**:
- Node using excessive CPU
- System slowdown

**Solutions**:

```bash
# 1. Check CPU usage
top -p $(pgrep honeybee-node)

# 2. Check for rapid reconnections
sudo journalctl -u honeybee-node | grep "Connecting to server"

# 3. Increase reconnect delay
# config.yaml:
# server:
#   reconnect_delay: 10

# 4. Check for errors causing restart loop
sudo journalctl -u honeybee-node -n 100
```

### High Memory Usage

**Symptoms**:
- Node using excessive memory
- Out of memory errors

**Solutions**:

```bash
# 1. Check memory usage
ps aux | grep honeybee-node

# 2. Check for memory leaks
# (restart and monitor)
sudo systemctl restart honeybee-node
watch -n 1 'ps aux | grep honeybee-node'

# 3. Check log file size
du -h /var/log/honeybee/

# 4. Enable log rotation
# See log rotation section
```

### Disk Space Issues

**Symptoms**:
- `No space left on device`
- Can't write logs

**Solutions**:

```bash
# 1. Check disk space
df -h

# 2. Check log file sizes
du -sh /var/log/honeybee/*

# 3. Clean old logs
sudo find /var/log/honeybee -name "*.log.*" -mtime +30 -delete

# 4. Set up log rotation
sudo vi /etc/logrotate.d/honeybee-node
```

## Logging Issues

### No Logs Appearing

**Symptoms**:
- No log output
- Log file empty
- Can't find logs

**Solutions**:

```bash
# 1. Check log configuration
cat /etc/honeybee/config.yaml | grep -A 3 "^log:"

# 2. Check if logging to file or stdout
# If no file specified, check stdout:
sudo journalctl -u honeybee-node -f

# 3. Check log file permissions
ls -la /var/log/honeybee/

# 4. Create log directory
sudo mkdir -p /var/log/honeybee
sudo chown honeybee:honeybee /var/log/honeybee

# 5. Verify log level
# config.yaml:
# log:
#   level: "debug"  # Temporarily for troubleshooting
```

### Log Rotation Not Working

**Symptoms**:
- Log files growing indefinitely
- Disk space running out

**Solutions**:

```bash
# 1. Check logrotate config
cat /etc/logrotate.d/honeybee-node

# 2. Test logrotate
sudo logrotate -d /etc/logrotate.d/honeybee-node

# 3. Force rotation
sudo logrotate -f /etc/logrotate.d/honeybee-node

# 4. Check logrotate service
sudo systemctl status logrotate.timer
```

## Systemd Issues

### Service Won't Start

**Symptoms**:
- `Failed to start honeybee-node.service`
- Service immediately stops

**Solutions**:

```bash
# 1. Check service status
sudo systemctl status honeybee-node

# 2. View detailed errors
sudo journalctl -u honeybee-node -n 50 --no-pager

# 3. Check service file
cat /etc/systemd/system/honeybee-node.service

# 4. Verify binary exists
ls -la /usr/local/bin/honeybee-node

# 5. Test binary manually
sudo -u honeybee /usr/local/bin/honeybee-node \
  -config /etc/honeybee/config.yaml

# 6. Reload systemd
sudo systemctl daemon-reload
```

### Service Won't Stop

**Symptoms**:
- Service hangs on stop
- Has to be killed forcefully

**Solutions**:

```bash
# 1. Check process
ps aux | grep honeybee-node

# 2. Send SIGTERM manually
sudo kill -TERM $(pgrep honeybee-node)

# 3. Wait for graceful shutdown
# Node should send NodeDrop and exit

# 4. If still hung, force kill
sudo kill -9 $(pgrep honeybee-node)

# 5. Check for resource locks
lsof -p $(pgrep honeybee-node)
```

## Debug Mode

Enable debug logging for troubleshooting:

```yaml
# config.yaml
log:
  level: "debug"
  format: "text"  # Easier to read
  file: "/var/log/honeybee/debug.log"
```

```bash
# Restart with debug logging
sudo systemctl restart honeybee-node

# Watch debug logs
sudo tail -f /var/log/honeybee/debug.log
```

## Getting Help

### Collect Diagnostic Information

```bash
#!/bin/bash
# collect-diagnostics.sh

echo "=== System Info ==="
uname -a
cat /etc/os-release

echo -e "\n=== Node Status ==="
sudo systemctl status honeybee-node

echo -e "\n=== Recent Logs ==="
sudo journalctl -u honeybee-node -n 100 --no-pager

echo -e "\n=== Configuration ==="
sudo cat /etc/honeybee/config.yaml

echo -e "\n=== Network ==="
nc -zv manager.example.com 9001

echo -e "\n=== Certificates ==="
openssl x509 -in /etc/honeybee/certs/ca.crt -noout -dates

echo -e "\n=== Disk Space ==="
df -h

echo -e "\n=== Permissions ==="
ls -la /etc/honeybee/
ls -la /var/lib/honeybee/
ls -la /var/log/honeybee/
```

### Report Issues

Include:
1. HoneyBee version
2. Operating system
3. Configuration (redact secrets!)
4. Error messages
5. Logs (last 100 lines)
6. Steps to reproduce

## Additional Resources

- [Installation Guide](./installation.md)
- [Configuration Guide](./configuration.md)
- [Security Guide](./security.md)
- [TLS Setup](./tls.md)
- [TOTP Setup](./totp.md)
- [Deployment Guide](./deployment.md)

## Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| `connection refused` | Manager not running or firewall | Check manager, test connectivity |
| `TLS handshake failed` | Certificate issue | Verify certificates |
| `x509: certificate signed by unknown authority` | Missing CA cert | Add ca_file to config |
| `TOTP authentication failed` | Time drift | Sync time with NTP |
| `permission denied` | Wrong file permissions | Fix ownership/permissions |
| `config file not found` | Wrong path | Check -config flag |
| `invalid configuration` | Config syntax error | Validate YAML |

---

Still having issues? Open an issue on [GitHub](https://github.com/yourusername/honeybee/issues).

