# Docker Deployment

Guide for deploying HoneyBee node using Docker containers.

## Quick Start

```bash
# Build image
docker build -t honeybee-node:latest .

# Run container
docker run -d --name honeybee-node honeybee-node:latest
```

## Complete Guide

See the [main deployment guide](./deployment.md) for Docker deployment instructions, including:

- Docker image building
- Docker Compose configuration
- Volume management
- Network configuration
- Security hardening
- Monitoring

## Docker Compose Example

```yaml
version: '3.8'

services:
  honeybee-node:
    image: honeybee-node:latest
    container_name: honeybee-node
    restart: unless-stopped
    volumes:
      - ./configs/config.yaml:/app/configs/config.yaml:ro
      - ./certs:/app/certs:ro
      - honeybee-logs:/app/logs
      - honeybee-secrets:/home/honeybee/.config/honeybee
    environment:
      - TZ=UTC
    networks:
      - honeybee-net

volumes:
  honeybee-logs:
  honeybee-secrets:

networks:
  honeybee-net:
    driver: bridge
```

For complete Docker deployment instructions, see [Deployment Guide](./deployment.md).

