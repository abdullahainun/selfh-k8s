# Monitoring Stack - Prometheus, Grafana, AlertManager

Complete monitoring and observability stack for Kubernetes clusters using Prometheus Operator.

## ✅ **DEPLOYED AND WORKING!**

## Components

- **Prometheus Operator**: Manages Prometheus, AlertManager, and related resources
- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **AlertManager**: Alert routing and notifications
- **Node Exporter**: Node-level metrics (CPU, memory, disk, network)
- **Kube State Metrics**: Kubernetes object state metrics
- **ServiceMonitors**: Auto-discovery of metrics endpoints

## Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Grafana   │────│ Prometheus  │────│ Metrics     │
│  (Port 3000)│    │ (Port 9090) │    │ Sources     │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                   ┌─────────────┐
                   │AlertManager │
                   │ (Port 9093) │
                   └─────────────┘
```

## Deployment

### Prerequisites

Requires dynamic storage (local-path-provisioner) for persistent volumes.

### Deploy Monitoring Stack

**Development:**
```bash
kubectl apply -k infrastructure/monitoring/overlays/dev
```

**Production:**
```bash
kubectl apply -k infrastructure/monitoring/overlays/prod
```

### Verify Installation

```bash
# Check all pods are running
kubectl get pods -n monitoring

# Check custom resources
kubectl get prometheus,alertmanager -n monitoring

# Check services
kubectl get svc -n monitoring
```

## Access Services

### Grafana Dashboard
```bash
# Port forward
kubectl port-forward -n monitoring svc/grafana 3000:3000

# Access: http://localhost:3000
# Login: admin/admin123 (dev) or admin/changeme123! (prod)
```

### Prometheus UI
```bash
# Port forward
kubectl port-forward -n monitoring svc/prometheus 9090:9090

# Access: http://localhost:9090
```

### AlertManager UI
```bash
# Port forward  
kubectl port-forward -n monitoring svc/alertmanager 9093:9093

# Access: http://localhost:9093
```

## Storage Configuration

### Persistent Volumes

- **Prometheus**: 50Gi (dev: 20Gi) retention 30d (dev: 7d)
- **Grafana**: 10Gi (dev: 5Gi) for dashboards and configurations
- **AlertManager**: 2Gi for alert state

### Storage Classes

Uses `local-path` storage class for dynamic provisioning.

## Metrics Collection

### Automatic Discovery

ServiceMonitors automatically discover and scrape:
- **Kubernetes API Server**: Cluster-level metrics
- **Kubelet**: Node and container metrics (cAdvisor)
- **Node Exporter**: Hardware and OS metrics
- **Kube State Metrics**: Kubernetes object states

### Custom Applications

Add ServiceMonitor for your applications:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  namespace: monitoring
  labels:
    team: platform
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

## Configuration

### Environment Differences

| Feature | Development | Production |
|---------|-------------|------------|
| Prometheus Memory | 512Mi-1Gi | 2Gi-4Gi |
| Storage Retention | 7 days | 90 days |  
| Storage Size | 20Gi | 50Gi |
| Grafana Sign-up | Enabled | Disabled |
| Anonymous Access | Enabled | Disabled |
| Log Level | Debug | Info |

### Grafana Data Sources

Prometheus automatically configured as default data source at `http://prometheus:9090`.

### AlertManager Configuration

Basic webhook configuration included. Customize via `alertmanager-config` Secret:

```bash
kubectl edit secret -n monitoring alertmanager-config
```

## Troubleshooting

### Common Issues

1. **Pods stuck in PodInitializing**
   ```bash
   kubectl describe pod -n monitoring prometheus-prometheus-0
   # Check for pod security policy violations
   ```

2. **No metrics in Grafana**
   ```bash
   # Check Prometheus targets
   kubectl port-forward -n monitoring svc/prometheus 9090:9090
   # Navigate to Status > Targets
   ```

3. **ServiceMonitor not discovered**
   ```bash
   # Ensure correct labels
   kubectl get servicemonitor -n monitoring -o yaml
   # Check selector matches in Prometheus CR
   ```

### Debugging Commands

```bash
# Check Prometheus Operator logs
kubectl logs -n monitoring deployment/prometheus-operator

# Check Prometheus configuration
kubectl get prometheus -n monitoring prometheus -o yaml

# Test metrics endpoint
kubectl exec -n monitoring prometheus-prometheus-0 -- wget -qO- localhost:9090/metrics

# Check storage
kubectl get pvc -n monitoring
```

## Monitoring Stack Resources

### Resource Requirements

**Minimum (Development):**
- CPU: 2 cores total
- Memory: 2Gi total  
- Storage: 27Gi total

**Production:**
- CPU: 4+ cores total
- Memory: 6Gi+ total
- Storage: 62Gi+ total

### Pod Security

Some components require privileged access:
- **Node Exporter**: hostNetwork, hostPID for node metrics
- **Prometheus/AlertManager**: seccompProfile warnings (non-blocking)

## Security Considerations

### Network Access

All services use ClusterIP - no external exposure by default. Use port-forward or ingress for access.

### Authentication

- **Grafana**: Username/password (configurable)
- **Prometheus**: No authentication (internal access only)
- **AlertManager**: No authentication (internal access only)

### Data Persistence

- Metrics data stored in persistent volumes
- Regular backups recommended for production
- Consider data retention policies

## Scaling and High Availability

### Horizontal Scaling

```yaml
# Prometheus sharding (advanced)
spec:
  shards: 2
  
# AlertManager replicas
spec:
  replicas: 3
```

### Vertical Scaling

Increase resource limits in production overlay:

```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: "2000m"
  limits:
    memory: "8Gi"
    cpu: "4000m"
```

## Integration Examples

### With Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  rules:
  - host: grafana.homelab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
```

### With External AlertManager

Configure external webhook endpoints in AlertManager config for:
- Slack notifications
- PagerDuty integration
- Email alerts
- Custom webhooks

## Next Steps

1. **Add Custom Dashboards**: Import community dashboards to Grafana
2. **Configure Alerting Rules**: Create PrometheusRule resources
3. **External Access**: Setup ingress for web interfaces
4. **Backup Strategy**: Implement metrics and config backups
5. **Integration**: Connect with external notification systems