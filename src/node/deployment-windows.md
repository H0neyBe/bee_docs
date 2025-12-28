# Windows Service Deployment

Deploy HoneyBee Node as a Windows service.

## Prerequisites

- Windows 7 or later
- Administrator privileges
- HoneyBee Node binary (`honeybee-node.exe`)

## Installation Methods

### Method 1: NSSM (Non-Sucking Service Manager)

NSSM is the recommended way to install HoneyBee Node as a Windows service.

#### Install NSSM

1. Download NSSM from [https://nssm.cc/download](https://nssm.cc/download)
2. Extract to a directory (e.g., `C:\nssm`)
3. Add to PATH or use full path

#### Install Service

```powershell
# Install service
nssm install HoneyBeeNode "C:\Program Files\HoneyBee\honeybee-node.exe"

# Set parameters
nssm set HoneyBeeNode AppParameters "-config C:\Program Files\HoneyBee\config.yaml"

# Set working directory
nssm set HoneyBeeNode AppDirectory "C:\Program Files\HoneyBee"

# Set description
nssm set HoneyBeeNode Description "HoneyBee Node - Distributed honeypot node"

# Set startup type
nssm set HoneyBeeNode Start SERVICE_AUTO_START

# Set log output
nssm set HoneyBeeNode AppStdout "C:\Program Files\HoneyBee\logs\stdout.log"
nssm set HoneyBeeNode AppStderr "C:\Program Files\HoneyBee\logs\stderr.log"
```

#### Start Service

```powershell
# Start service
nssm start HoneyBeeNode

# Check status
nssm status HoneyBeeNode
```

### Method 2: Windows Service Wrapper

Create a service wrapper using Go or another language (advanced).

## Service Management

### Start Service

```powershell
# Using NSSM
nssm start HoneyBeeNode

# Using Windows Service Manager
Start-Service HoneyBeeNode

# Or using sc
sc start HoneyBeeNode
```

### Stop Service

```powershell
# Using NSSM
nssm stop HoneyBeeNode

# Using Windows Service Manager
Stop-Service HoneyBeeNode

# Or using sc
sc stop HoneyBeeNode
```

### Restart Service

```powershell
# Using NSSM
nssm restart HoneyBeeNode

# Using Windows Service Manager
Restart-Service HoneyBeeNode
```

### Check Status

```powershell
# Using NSSM
nssm status HoneyBeeNode

# Using Windows Service Manager
Get-Service HoneyBeeNode

# Or using sc
sc query HoneyBeeNode
```

## Configuration

Place configuration file at:

```
C:\Program Files\HoneyBee\config.yaml
```

Ensure the service has read permissions.

## Logs

Logs are written to:

```
C:\Program Files\HoneyBee\logs\
```

Configure log rotation using Windows Task Scheduler or log management tools.

## Troubleshooting

### Service Won't Start

1. Check Event Viewer for errors
2. Verify binary path is correct
3. Check configuration file exists and is valid
4. Verify user permissions

### Service Stops Unexpectedly

1. Check logs in `logs\` directory
2. Review Event Viewer
3. Verify Core manager is accessible
4. Check network connectivity

### View Logs

```powershell
# View stdout
Get-Content "C:\Program Files\HoneyBee\logs\stdout.log" -Tail 50

# View stderr
Get-Content "C:\Program Files\HoneyBee\logs\stderr.log" -Tail 50
```

## Uninstall Service

```powershell
# Stop service first
nssm stop HoneyBeeNode

# Remove service
nssm remove HoneyBeeNode confirm
```

## Next Steps

- [Deployment](./deployment.md) - Other deployment methods
- [Configuration](./configuration.md) - Configure node
- [Troubleshooting](./troubleshooting.md) - Common issues

