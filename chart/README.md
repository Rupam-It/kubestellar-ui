# KubeStellar UI Helm Chart

This Helm chart deploys the KubeStellar UI application with Redis as a dependent chart.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installation

1. Add the Bitnami repository (required for Redis dependency):
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

2. Update dependencies:
```bash
helm dependency update
```

3. Install the chart:
```bash
helm install kubestellar-ui ./chart
```

## Accessing the Application

The application is accessible through the frontend service. By default, ingress is disabled to avoid webhook validation issues.

### Option 1: Port Forward (Recommended for testing)
```bash
kubectl port-forward service/frontend 8080:80
```
Then access the application at `http://localhost:8080`

### Option 2: NodePort
The frontend service is configured as NodePort on port 30137. Access it at:
```
http://<node-ip>:30137
```

### Option 3: Enable Ingress
To enable ingress, set the following in your values.yaml:
```yaml
ingress:
  enabled: true
  className: "nginx"  # or your ingress controller class
  host: your-domain.com
  annotations:
    kubernetes.io/ingress.class: nginx
```

**Note**: Make sure your ingress controller is properly installed and configured before enabling ingress.

## Redis Configuration

Redis is now deployed as a dependent chart using the Bitnami Redis Helm chart. The configuration can be customized in `values.yaml` under the `redis` section.

### Default Redis Configuration:
- Authentication: Disabled
- Architecture: Standalone
- Persistence: Disabled
- Resources: 100m CPU, 128Mi Memory (requests), 200m CPU, 256Mi Memory (limits)

### To enable Redis persistence:
```yaml
redis:
  master:
    persistence:
      enabled: true
      size: 8Gi
```

### To enable Redis authentication:
```yaml
redis:
  auth:
    enabled: true
    password: "your-password"
```

## Troubleshooting

### Ingress Webhook Errors
If you encounter ingress webhook validation errors:
1. Ensure ingress is disabled: `ingress.enabled: false` (default)
2. Use port-forward or NodePort to access the application
3. If you need ingress, ensure your ingress controller is properly configured

### Common Issues
- **Connection refused errors**: Usually indicates missing or misconfigured ingress controller
- **Redis connection issues**: Ensure Redis dependency is properly installed

## Uninstallation

```bash
helm uninstall kubestellar-ui
```

## Changes from Previous Version

- **Redis Sidecar Removed**: Redis is no longer deployed as a sidecar container in the backend pod
- **Redis Dependency Added**: Redis is now deployed as a separate service using the Bitnami Redis chart
- **Service Connection**: Backend connects to Redis via the service name `<release-name>-redis-master`
- **Improved Scalability**: Redis can now be scaled independently and configured with persistence
- **Ingress Made Optional**: Ingress is now disabled by default to avoid webhook validation issues 