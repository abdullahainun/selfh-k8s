# Storage - Dynamic Persistent Volume Provisioning

This directory contains Kubernetes manifests for local-path-provisioner, enabling dynamic persistent volume provisioning for homelab environments.

## Features

- **Dynamic PV Provisioning**: Automatically creates persistent volumes when PVCs are requested
- **Environment-Specific Paths**: Different storage paths for dev/prod environments  
- **Default Storage Class**: Configured as the default storage class for easy PVC creation
- **Resource Management**: Production overlay includes resource limits and requests

## Components

- **local-path-provisioner**: Rancher's local-path-provisioner v0.0.28
- **Storage Class**: `local-path` with `WaitForFirstConsumer` binding mode
- **RBAC**: Service account, roles, and bindings for provisioner operation

## Directory Structure

```
storage/
├── base/
│   └── kustomization.yaml          # Base local-path-provisioner setup
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml      # Dev environment config
│   └── prod/
│       └── kustomization.yaml     # Production config with resource limits
└── test/                          # Test resources for validation
```

## Deployment

### Prerequisites

**Important**: The local-path-provisioner requires privileged pod security context to create directories on nodes. You need to label the namespace after deployment:

```bash
kubectl label namespace local-path-storage \
  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/audit=privileged \
  pod-security.kubernetes.io/warn=privileged
```

### Deploy Storage Infrastructure

**Development:**
```bash
kubectl apply -k infrastructure/storage/overlays/dev
kubectl label namespace local-path-storage pod-security.kubernetes.io/enforce=privileged pod-security.kubernetes.io/audit=privileged pod-security.kubernetes.io/warn=privileged
```

**Production:**
```bash
kubectl apply -k infrastructure/storage/overlays/prod
kubectl label namespace local-path-storage pod-security.kubernetes.io/enforce=privileged pod-security.kubernetes.io/audit=privileged pod-security.kubernetes.io/warn=privileged
```

### Verify Installation

```bash
# Check provisioner pod
kubectl get pods -n local-path-storage

# Verify storage class
kubectl get storageclass local-path

# Test dynamic provisioning
kubectl apply -k infrastructure/storage/test
kubectl get pvc test-dynamic-pvc
kubectl delete -k infrastructure/storage/test
```

## Configuration

### Storage Paths

- **Dev Environment**: `/opt/local-path-provisioner/dev`
- **Prod Environment**: `/opt/local-path-provisioner/prod`

### Resource Limits (Production)

```yaml
resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 200m
    memory: 128Mi
```

## Usage in Applications

Applications can now use dynamic persistent volumes without manual PV creation:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-data
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path  # or omit for default
  resources:
    requests:
      storage: 5Gi
```

The provisioner will automatically:
1. Create a directory on the scheduled node
2. Create a PersistentVolume pointing to that directory
3. Bind the PVC to the PV

## Node Selection

Volumes are created on the node where the pod is scheduled. The provisioner supports:
- **Single-node clusters**: Works perfectly
- **Multi-node clusters**: Volume follows pod scheduling
- **Node affinity**: Respects pod's node selection constraints

## Backup Considerations

Since this uses local storage on nodes:
- **Plan backups**: Data is node-local, not replicated
- **Node failure**: Data loss if node fails permanently
- **Cluster migration**: Manual data migration required

For production with high availability requirements, consider:
- Longhorn for replicated block storage
- NFS for shared storage
- Rook-Ceph for distributed storage

## Troubleshooting

### Common Issues

1. **PVC stuck in Pending**
   ```bash
   kubectl describe pvc <pvc-name>
   # Look for ProvisioningFailed events
   ```

2. **Pod Security Policy violations**
   ```bash
   # Ensure namespace has privileged pod security labels
   kubectl get namespace local-path-storage -o yaml | grep pod-security
   ```

3. **Permission denied errors**
   ```bash
   # Check provisioner logs
   kubectl logs -n local-path-storage deployment/local-path-provisioner
   ```

### Debugging Commands

```bash
# Check provisioner status
kubectl get deployment -n local-path-storage

# View provisioner logs
kubectl logs -n local-path-storage -l app=local-path-provisioner

# List all PVs created by provisioner
kubectl get pv -l provisioned-by=rancher.io/local-path

# Check node storage usage
kubectl exec -n local-path-storage deployment/local-path-provisioner -- df -h /opt/local-path-provisioner/
```

## Migration from Manual PVs

If you have existing applications using manual PVs, you can migrate:

1. **Create new PVC** with dynamic provisioning
2. **Copy data** from old PV to new PV
3. **Update application** to use new PVC
4. **Clean up** old manual PVs

Example data migration:
```bash
# Create temporary pod to copy data
kubectl run data-migration --image=busybox --rm -it -- sh
# Mount both old and new volumes, then copy data
```