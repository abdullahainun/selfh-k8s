# Backup & Disaster Recovery - Velero + Restic

Complete backup and disaster recovery solution using Velero with Restic file-level backup, optimized for local-path-provisioner and homelab environments.

> **🚀 Quick Start**: See [SETUP.md](./SETUP.md) for step-by-step configuration guide.

## Features

- **✅ No Downtime Backups**: File-level backup using Restic (most applications)
- **✅ Flexible Storage**: Support external MinIO, in-cluster MinIO, or cloud storage
- **✅ Automated Scheduling**: Daily, weekly, and critical application backups
- **✅ Local-Path Optimized**: Works perfectly with local-path-provisioner
- **✅ Cross-Cluster Restore**: Backup once, restore anywhere
- **✅ Incremental Backups**: Efficient storage with deduplication
- **✅ Application Hooks**: Database consistency with pre/post-backup scripts

## Architecture

### Option 1: External Storage (Recommended for Production)
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Kubernetes     │    │     Velero      │    │  External       │
│  Cluster        │    │                 │    │  MinIO/S3       │
│                 │    │ ├─ Controller   │────│                 │
│ ├─ Apps + PVs   │────│ ├─ Node Agents  │    │ ├─ Bucket       │
│ └─ local-path   │    │ └─ Restic       │    │ └─ Retention    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Option 2: In-Cluster Storage (Good for Testing)
```
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                          │
│                                                                 │
│ ┌─────────────────┐    ┌─────────────────┐    ┌─────────────┐   │
│ │  Applications   │    │     Velero      │    │    MinIO    │   │
│ │                 │    │                 │    │             │   │
│ │ ├─ Apps + PVs   │────│ ├─ Controller   │────│ ├─ S3 API   │   │
│ │ └─ local-path   │    │ ├─ Node Agents  │    │ └─ Storage  │   │
│ └─────────────────┘    │ └─ Restic       │    └─────────────┘   │
│                        └─────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
```

## Storage Options

### 🏆 External MinIO/S3 (Recommended)
**Best for**: Production, true disaster recovery

**Pros**:
- ✅ True disaster recovery (data outside cluster)
- ✅ Better performance (dedicated resources)
- ✅ Easier scaling and maintenance
- ✅ Can survive complete cluster failure

**Cons**:
- ⚠️ Requires external infrastructure

### 🧪 In-Cluster MinIO
**Best for**: Testing, development, simple setups

**Pros**:
- ✅ Self-contained (no external dependencies) 
- ✅ Easy setup and management
- ✅ Perfect for homelab testing

**Cons**:
- ⚠️ Single point of failure
- ⚠️ Limited disaster recovery (if cluster dies, backups may be lost)

### ☁️ Cloud Storage
**Best for**: Hybrid cloud, enterprise

**Supported**: AWS S3, Google Cloud Storage, Azure Blob Storage

## Backup Strategy

### Default Schedules

| Schedule | Frequency | Retention | Purpose |
|----------|-----------|-----------|---------|
| **Daily** | 2 AM | 7 days (dev) / 7 days (prod) | Regular application backups |
| **Weekly** | 1 AM Sunday | 30 days (dev) / 90 days (prod) | Full cluster backups |
| **Critical** | Every 6h (dev) / 4h (prod) | 3 days | Mission-critical applications |

### Backup Scope

**Daily Backup includes**:
- Application namespaces
- Persistent volumes (via Restic)
- Application configurations
- Secrets and ConfigMaps

**Weekly Full Backup includes**:
- All namespaces (except system)
- Cluster resources (RBAC, CRDs, etc.)
- Complete disaster recovery snapshot

**Critical Apps Backup includes**:
- High-priority namespaces (monitoring, databases)
- More frequent protection for essential services

## No-Downtime Applications

These applications work perfectly with Velero + Restic **without downtime**:

### ✅ Stateless Applications
- Web applications, APIs, microservices
- Load balancers, reverse proxies
- Static content servers

### ✅ Crash-Safe Databases
- **PostgreSQL**: WAL ensures consistency
- **MySQL**: InnoDB crash recovery
- **MongoDB**: Journal-based recovery
- **Redis**: AOF/RDB persistence

### ✅ File-Based Applications
- **Git repositories** (Gitea, GitLab)
- **Media servers** (Jellyfin, Plex)
- **Document storage** (NextCloud)
- **Static sites**

### ✅ Monitoring & Observability
- **Prometheus**: TSDB handles interruptions well
- **Grafana**: SQLite/PostgreSQL backend crash-safe
- **Log aggregation**: ELK, Loki handle restarts

## Applications That May Need Hooks

### ⚠️ Write-Heavy Transactional Systems
- Financial applications
- Accounting software
- Real-time trading systems

**Solution**: Use pre/post-backup hooks for consistency

### ⚠️ Custom Databases
- Applications with custom file formats
- Embedded databases without crash recovery

**Solution**: Create application dumps during backup

## Deployment

### Prerequisites
- Kubernetes cluster with local-path-provisioner
- Storage backend (external MinIO recommended)
- kubectl and kustomize

### Option 1: External Storage (Recommended)
```bash
# 1. Configure your external MinIO/S3 credentials
cp infrastructure/backup/storage/external/cloud-credentials-template.yaml cloud-credentials.yaml
# Edit cloud-credentials.yaml with your actual credentials

# 2. Create secret
kubectl create secret generic cloud-credentials \
  --from-file=cloud=cloud-credentials.yaml -n velero

# 3. Update storage configuration
# Edit infrastructure/backup/storage/external/backup-storage-location.yaml
# Update s3Url and bucket name

# 4. Deploy Velero
kubectl apply -k infrastructure/backup/overlays/prod
```

### Option 2: In-Cluster MinIO
```bash
# Deploy with in-cluster MinIO (uncomment minio in overlay)
kubectl apply -k infrastructure/backup/overlays/dev
```

## Usage Examples

### Manual Backup
```bash
# Backup specific namespace
kubectl create -f - <<EOF
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: myapp-backup
  namespace: velero
spec:
  includedNamespaces:
  - myapp
  defaultVolumesToFsBackup: true
EOF
```

### Restore from Backup
```bash
# List available backups
velero backup get

# Restore specific backup
velero restore create --from-backup daily-backup-20240130020000

# Restore to different namespace
velero restore create --from-backup daily-backup-20240130020000 \
  --namespace-mappings production:staging
```

### Monitor Backup Status
```bash
# Check backup status
kubectl get backup -n velero

# Check backup details
kubectl describe backup daily-backup-20240130020000 -n velero

# Check restore status
kubectl get restore -n velero
```

## Backup Best Practices

### 1. **Test Restores Regularly**
```bash
# Monthly restore test to separate namespace
velero restore create test-restore-$(date +%Y%m%d) \
  --from-backup weekly-full-backup-latest \
  --namespace-mappings production:restore-test
```

### 2. **Monitor Backup Health**
```bash
# Check failed backups
kubectl get backup -n velero | grep -v Completed

# Monitor backup size trends
velero backup describe daily-backup-latest
```

### 3. **Application-Specific Configurations**

**PostgreSQL with consistency**:
```yaml
metadata:
  annotations:
    pre.hook.backup.velero.io/container: postgres
    pre.hook.backup.velero.io/command: '["/bin/bash", "-c", "pg_dump mydb > /tmp/backup.sql"]'
    post.hook.backup.velero.io/container: postgres  
    post.hook.backup.velero.io/command: '["/bin/bash", "-c", "rm /tmp/backup.sql"]'
```

**Exclude cache volumes**:
```yaml
metadata:
  annotations:
    backup.velero.io/backup-volumes-excludes: cache,tmp,logs
```

### 4. **Resource Planning**

**Storage Requirements**:
- Estimate: ~30-50% of original data size (due to compression/deduplication)
- Plan for 3-6 months of retention
- Monitor growth trends

**Performance Impact**:
- Backup typically uses 10-20% CPU during operation
- Network bandwidth depends on change rate
- Schedule during low-traffic periods

## Troubleshooting

### Common Issues

1. **Backup Stuck in Progress**
```bash
# Check Velero logs
kubectl logs -n velero deployment/velero

# Check node agent logs
kubectl logs -n velero daemonset/node-agent
```

2. **Storage Connection Issues**
```bash
# Verify credentials secret
kubectl get secret cloud-credentials -n velero -o yaml

# Test storage connectivity
kubectl run -it --rm debug --image=amazon/aws-cli --restart=Never -- \
  aws s3 ls s3://your-bucket --endpoint-url=http://your-minio-endpoint
```

3. **Restore Failures**
```bash
# Check restore details
kubectl describe restore <restore-name> -n velero

# Common issues: namespace conflicts, resource dependencies
```

### Debug Commands
```bash
# Install Velero CLI
curl -fsSL -o velero-v1.14.1-linux-amd64.tar.gz \
  https://github.com/vmware-tanzu/velero/releases/download/v1.14.1/velero-v1.14.1-linux-amd64.tar.gz
tar -xvf velero-v1.14.1-linux-amd64.tar.gz
sudo mv velero-v1.14.1-linux-amd64/velero /usr/local/bin/

# Debug backup
velero backup describe <backup-name> --details

# Debug restore
velero restore describe <restore-name> --details

# Check backup logs
velero backup logs <backup-name>
```

## Security Considerations

### 1. **Storage Security**
- Use strong credentials for MinIO/S3
- Enable encryption at rest in storage backend
- Rotate access keys regularly
- Restrict network access to storage endpoints

### 2. **Kubernetes Security**
- Velero requires broad cluster permissions (necessary for backup/restore)
- Use sealed-secrets for credential management
- Monitor Velero namespace for unauthorized access

### 3. **Data Encryption**
- Restic provides client-side encryption
- Configure encryption repository password
- Store encryption keys securely

## Monitoring & Alerting

### Key Metrics to Monitor
- Backup success/failure rates
- Backup duration trends
- Storage usage growth
- Restore testing results

### Integration with Prometheus
```yaml
# ServiceMonitor for Velero metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: velero
  namespace: velero
spec:
  selector:
    matchLabels:
      app: velero
  endpoints:
  - port: http-metrics
```

### Alerting Rules
```yaml
# Alert on backup failures
- alert: VeleroBackupFailure
  expr: velero_backup_failure_total > 0
  labels:
    severity: critical
  annotations:
    summary: "Velero backup failed"
```

This backup solution provides enterprise-grade disaster recovery for your Kubernetes homelab! 💾🛡️