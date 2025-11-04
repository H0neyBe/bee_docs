# Kubernetes Deployment

Guide for deploying HoneyBee node on Kubernetes.

## Quick Start

```bash
# Apply manifests
kubectl apply -f k8s/

# Check pods
kubectl get pods -l app=honeybee-node

# View logs
kubectl logs -f -l app=honeybee-node
```

## Complete Guide

See the [main deployment guide](./deployment.md) for Kubernetes deployment instructions, including:

- Deployment manifests
- ConfigMap configuration
- Secret management
- Service configuration
- StatefulSet for persistent storage
- Horizontal Pod Autoscaling

## Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: honeybee-node
spec:
  replicas: 3
  selector:
    matchLabels:
      app: honeybee-node
  template:
    metadata:
      labels:
        app: honeybee-node
    spec:
      containers:
      - name: honeybee-node
        image: honeybee-node:1.0.0
        volumeMounts:
        - name: config
          mountPath: /app/configs
        - name: tls
          mountPath: /app/certs
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: honeybee-node-config
      - name: tls
        secret:
          secretName: honeybee-tls
```

For complete Kubernetes deployment instructions, see [Deployment Guide](./deployment.md).

