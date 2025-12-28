# Docker Deployment

Deploy HoneyBee Core using Docker.

## Quick Start

```bash
# Clone repository
git clone https://github.com/H0neyBe/honeybee_core.git
cd honeybee_core

# Start with docker-compose
docker-compose up -d

# Check status
docker-compose ps
docker logs honeybee-core
```

## Docker Compose

The `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  honeybee-core:
    image: ${DOCKERHUB_USERNAME:-h0neybe}/honeybee-core:latest
    container_name: honeybee-core
    ports:
      - "9001:9001"  # Node port
      - "9002:9002"  # Backend port
      - "9003:9003"  # WebSocket port
    volumes:
      - ./bee_config.toml:/app/bee_config.toml:ro
      - honeybee-logs:/app/logs
    restart: unless-stopped
    environment:
      - RUST_LOG=${RUST_LOG:-info}
    healthcheck:
      test: ["CMD", "nc", "-z", "localhost", "9001"]
      interval: 30s
      timeout: 3s
      start_period: 10s
      retries: 3

volumes:
  honeybee-logs:
```

## Building Image

```bash
# Build from source
docker build -t honeybee-core:latest .

# Or use pre-built image
docker pull h0neybe/honeybee-core:latest
```

## Running Container

```bash
# Run with default config
docker run -d \
  --name honeybee-core \
  -p 9001:9001 \
  -p 9002:9002 \
  -p 9003:9003 \
  honeybee-core:latest

# Run with custom config
docker run -d \
  --name honeybee-core \
  -p 9001:9001 \
  -p 9002:9002 \
  -p 9003:9003 \
  -v $(pwd)/bee_config.toml:/app/bee_config.toml:ro \
  -v honeybee-logs:/app/logs \
  honeybee-core:latest
```

## Environment Variables

```bash
docker run -d \
  --name honeybee-core \
  -e RUST_LOG=debug \
  honeybee-core:latest
```

## Volumes

- **Config**: Mount `bee_config.toml` as read-only
- **Logs**: Use named volume for logs persistence

## Health Checks

The container includes a health check that verifies port 9001 is listening.

## Networking

### Bridge Network (Default)

Containers can communicate via bridge network.

### Host Network

For better performance:

```yaml
network_mode: "host"
```

## Resource Limits

```yaml
deploy:
  resources:
    limits:
      cpus: '2'
      memory: 2G
    reservations:
      cpus: '1'
      memory: 1G
```

## Troubleshooting

### View Logs

```bash
docker logs honeybee-core
docker logs -f honeybee-core  # Follow logs
```

### Execute Commands

```bash
docker exec -it honeybee-core sh
```

### Check Status

```bash
docker ps | grep honeybee-core
docker inspect honeybee-core
```

## Next Steps

- [Configuration](./configuration.md) - Configure Core
- [Deployment](./deployment.md) - Other deployment methods

