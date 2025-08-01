# Quick Setup Guide - Velero Backup & Disaster Recovery

Step-by-step guide to deploy Velero backup solution with your choice of storage backend.

## Prerequisites

- ✅ Kubernetes cluster with `local-path-provisioner`
- ✅ `kubectl` and `kustomize` installed
- ✅ Storage backend (external MinIO recommended)

## Option 1: External MinIO/S3 Setup (Recommended)

### Step 1: Prepare Storage Backend

**For External MinIO:**
1. Ensure MinIO is running and accessible
2. Create a bucket for backups (e.g., `k8s-backups`)
3. Create access credentials with read/write permissions

**For AWS S3:**
1. Create S3 bucket
2. Create IAM user with appropriate S3 permissions
3. Generate access key and secret

**For Google Cloud Storage:**
1. Create GCS bucket
2. Create service account with Storage Admin role
3. Generate access key for S3 compatibility

### Step 2: Configure Credentials

```bash
# Copy template and edit with your credentials
cp infrastructure/backup/storage/external/cloud-credentials-template.yaml cloud-credentials.yaml

# Edit the file with your actual credentials
nano cloud-credentials.yaml
```

**Example for External MinIO:**
```yaml
[default]
aws_access_key_id = your-minio-access-key
aws_secret_access_key = your-minio-secret-key
```

### Step 3: Update Storage Configuration

Edit `infrastructure/backup/storage/external/backup-storage-location.yaml`:

```yaml
spec:
  objectStorage:
    bucket: your-bucket-name     # Change this
    prefix: your-cluster-name    # Optional: organize backups
  config:
    s3Url: http://your-minio-endpoint:9000  # Change this
```

### Step 4: Create Credentials Secret

```bash
# Create secret from credentials file
kubectl create secret generic cloud-credentials \
  --from-file=cloud=cloud-credentials.yaml -n velero

# Clean up credentials file
rm cloud-credentials.yaml
```

### Step 5: Deploy Velero

```bash
# For production with external storage
kubectl apply -k infrastructure/backup/overlays/prod

# For development with external storage
kubectl apply -k infrastructure/backup/overlays/dev
```

## Option 2: In-Cluster MinIO Setup (Simple)

### Step 1: Customize MinIO Configuration

Edit `infrastructure/backup/storage/minio/minio-deployment.yaml`:

```yaml
# Update MinIO credentials (line ~23)
stringData:
  MINIO_ROOT_USER: your-username
  MINIO_ROOT_PASSWORD: your-strong-password

# Update storage size if needed (line ~40)
resources:
  requests:
    storage: 200Gi  # Adjust based on needs
```

### Step 2: Update Kustomization

Edit overlays to use in-cluster MinIO:

**For Development:**
```bash
# Edit infrastructure/backup/overlays/dev/kustomization.yaml
# Uncomment this line:
  - ../../storage/minio          # Option 1: In-cluster MinIO
# Comment this line:
  # - ../../storage/external     # Option 2: External MinIO/S3
```

### Step 3: Deploy

```bash
# Deploy with in-cluster MinIO
kubectl apply -k infrastructure/backup/overlays/dev
```

### Step 4: Access MinIO Console (Optional)

```bash
# Port-forward to access MinIO console
kubectl port-forward -n velero svc/minio 9001:9001

# Access: http://localhost:9001
# Login with credentials from step 1
```

## Verification

### Step 1: Check Pod Status

```bash
# Verify all pods are running
kubectl get pods -n velero

# Expected output:
# NAME                     READY   STATUS    ROLE
# velero-xxx               1/1     Running   # Main controller
# node-agent-xxx           1/1     Running   # On each node
# minio-xxx                1/1     Running   # If using in-cluster
```

### Step 2: Check Backup Storage Location

```bash
# Verify storage location is available
kubectl get backupstoragelocation -n velero

# Should show "Available" status
```

### Step 3: Test Manual Backup

```bash
# Create test backup
kubectl create -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-config
  namespace: default
data:
  message: "Hello Velero!"
---
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: test-backup
  namespace: velero
spec:
  includedNamespaces:
  - default
  includedResources:
  - configmaps
  ttl: "1h"
EOF

# Check backup status
kubectl get backup test-backup -n velero

# Should show "Completed" after a few minutes
```

### Step 4: Test Restore

```bash
# Delete test ConfigMap
kubectl delete configmap test-config

# Restore from backup
kubectl create -f - <<EOF
apiVersion: velero.io/v1
kind: Restore
metadata:
  name: test-restore
  namespace: velero
spec:
  backupName: test-backup
  includedResources:
  - configmaps
EOF

# Verify ConfigMap is restored
kubectl get configmap test-config
```

### Step 5: Cleanup Test Resources

```bash
# Clean up test resources
kubectl delete backup test-backup -n velero
kubectl delete restore test-restore -n velero
kubectl delete configmap test-config
```

## Install Velero CLI (Optional but Recommended)

```bash
# Download and install Velero CLI
curl -fsSL -o velero-v1.14.1-linux-amd64.tar.gz \
  https://github.com/vmware-tanzu/velero/releases/download/v1.14.1/velero-v1.14.1-linux-amd64.tar.gz

tar -xzf velero-v1.14.1-linux-amd64.tar.gz
sudo mv velero-v1.14.1-linux-amd64/velero /usr/local/bin/

# Verify installation
velero version
```

## Configure Scheduled Backups

The deployment includes default schedules:

### View Scheduled Backups
```bash
# List backup schedules
kubectl get schedule -n velero

# Check specific schedule
kubectl describe schedule daily-backup -n velero
```

### Customize Backup Schedules

Edit `infrastructure/backup/base/backup-schedules.yaml` to:
- Add/remove namespaces from backup scope
- Adjust backup frequency
- Modify retention policies
- Configure backup hooks for specific applications

### Example: Add New Namespace to Daily Backup

```bash
# Edit the daily backup schedule
kubectl edit schedule daily-backup -n velero

# Add your namespace to includedNamespaces list
spec:
  template:
    spec:
      includedNamespaces:
      - "default"
      - "monitoring"
      - "your-new-namespace"  # Add this
```

## Common Customizations

### 1. Backup Specific Applications Only

Create custom backup for specific apps:

```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: database-backup
  namespace: velero
spec:
  schedule: "0 */3 * * *"  # Every 3 hours
  template:
    spec:
      includedNamespaces:
      - database
      - redis
      defaultVolumesToFsBackup: true
      ttl: "72h"
```

### 2. Exclude Cache/Temp Volumes

Add annotations to pods:

```yaml
metadata:
  annotations:
    backup.velero.io/backup-volumes-excludes: cache,tmp,logs
```

### 3. Add Pre/Post Backup Hooks

For database consistency:

```yaml
metadata:
  annotations:
    pre.hook.backup.velero.io/container: postgres
    pre.hook.backup.velero.io/command: '["/bin/bash", "-c", "pg_dump mydb > /tmp/backup.sql"]'
    post.hook.backup.velero.io/container: postgres
    post.hook.backup.velero.io/command: '["/bin/bash", "-c", "rm /tmp/backup.sql"]'
```

## Monitoring Backups

### Dashboard Metrics

If you have monitoring stack deployed:

```bash
# Port-forward to Velero metrics
kubectl port-forward -n velero svc/velero 8086:8086

# Access metrics: http://localhost:8086/metrics
```

### Email Notifications (Optional)

Configure alerts for backup failures:

```yaml
# Example: Slack notification on backup failure
apiVersion: v1
kind: ConfigMap
metadata:
  name: backup-alert
  namespace: velero
data:
  webhook.sh: |
    #!/bin/bash
    curl -X POST -H 'Content-type: application/json' \
    --data '{"text":"Backup failed: $BACKUP_NAME"}' \
    YOUR_SLACK_WEBHOOK_URL
```

## Next Steps

1. **Set up regular restore testing** (monthly recommended)
2. **Configure monitoring and alerting** for backup health
3. **Document restore procedures** for your applications
4. **Test disaster recovery scenarios** end-to-end
5. **Set up off-site backup replication** for critical data

## Troubleshooting

### Backup Stuck in Progress
```bash
# Check Velero controller logs
kubectl logs -n velero deployment/velero -f

# Check node agent logs (if using Restic)
kubectl logs -n velero daemonset/node-agent -f
```

### Storage Connection Issues
```bash
# Test MinIO connectivity
kubectl run -it --rm debug --image=minio/mc --restart=Never -- \
  mc config host add minio http://your-endpoint:9000 access-key secret-key

# List buckets
kubectl run -it --rm debug --image=minio/mc --restart=Never -- \
  mc ls minio
```

### Restore Not Working
```bash
# Check restore details
kubectl describe restore <restore-name> -n velero

# Common issues:
# - Namespace already exists
# - Resource conflicts
# - Missing dependencies
```

Your Velero backup system is now ready for production use! 🎯

**Pro tip**: Start with daily backups, test restores monthly, and gradually add more sophisticated backup hooks as needed.