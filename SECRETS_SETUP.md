# 🔐 Secrets Setup Guide

This repository uses **SealedSecrets** to manage sensitive data securely in GitOps workflows. When deploying to a new cluster, you'll need to set up your own encrypted secrets.

## Prerequisites

1. **Kubernetes cluster** with kubectl access
2. **kubeseal CLI** installed ([Installation Guide](https://sealed-secrets.netlify.app/))
3. **SealedSecrets controller** deployed in your cluster

## 🚀 Quick Setup

### 1. Install SealedSecrets Controller

```bash
# Install the latest SealedSecrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Wait for controller to be ready
kubectl wait --for=condition=available --timeout=300s deployment/sealed-secrets-controller -n kube-system
```

### 2. Verify Installation

```bash
# Check controller is running
kubectl get pods -n kube-system | grep sealed-secrets-controller

# Test kubeseal connectivity
kubeseal --fetch-cert > /tmp/sealed-secrets-cert.pem
```

## 🔑 Configure Secrets

### Monitoring Stack Secrets

The monitoring stack requires Grafana admin credentials. Follow these steps:

#### 1. Generate Grafana Admin Credentials

```bash
# Set your desired credentials
GRAFANA_USER="admin"
GRAFANA_PASSWORD="your-secure-password-here"

# Generate encrypted username
ENCRYPTED_USER=$(echo -n "$GRAFANA_USER" | kubeseal --raw --from-file=/dev/stdin --name grafana-admin --namespace monitoring)

# Generate encrypted password  
ENCRYPTED_PASSWORD=$(echo -n "$GRAFANA_PASSWORD" | kubeseal --raw --from-file=/dev/stdin --name grafana-admin --namespace monitoring)

echo "Encrypted username: $ENCRYPTED_USER"
echo "Encrypted password: $ENCRYPTED_PASSWORD"
```

#### 2. Update SealedSecret Manifest

Edit `infrastructure/monitoring/base/grafana-sealed-secret.yaml`:

```yaml
# Grafana Admin Credentials SealedSecret
# Contains encrypted admin password for Grafana
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: grafana-admin
  namespace: monitoring
spec:
  encryptedData:
    admin-user: YOUR_ENCRYPTED_USERNAME_HERE
    admin-password: YOUR_ENCRYPTED_PASSWORD_HERE
  template:
    metadata:
      name: grafana-admin
      namespace: monitoring
    type: Opaque
```

#### 3. Apply and Verify

```bash
# Apply the updated SealedSecret from the repository
kubectl apply -k infrastructure/monitoring/base/

# Or apply just the SealedSecret
kubectl apply -f infrastructure/monitoring/base/grafana-sealed-secret.yaml

# Verify secret was created and decrypted
kubectl get secret grafana-admin -n monitoring

# (Optional) Verify decrypted values
kubectl get secret grafana-admin -n monitoring -o jsonpath='{.data.admin-user}' | base64 -d
kubectl get secret grafana-admin -n monitoring -o jsonpath='{.data.admin-password}' | base64 -d
```

## 🔍 Adding New Secrets

### For New Services

When adding new services that require secrets:

1. **Create the secret data:**
   ```bash
   # Example: Database password
   echo -n 'my-database-password' | kubeseal --raw --from-file=/dev/stdin --name my-service-secret --namespace my-namespace
   ```

2. **Create SealedSecret manifest:**
   ```yaml
   apiVersion: bitnami.com/v1alpha1
   kind: SealedSecret
   metadata:
     name: my-service-secret
     namespace: my-namespace
   spec:
     encryptedData:
       database-password: YOUR_ENCRYPTED_VALUE_HERE
     template:
       metadata:
         name: my-service-secret
         namespace: my-namespace
       type: Opaque
   ```

3. **Add to kustomization.yaml and apply:**
   ```yaml
   # Add to resources section in kustomization.yaml
   resources:
     - my-service-sealed-secret.yaml
   ```
   
   ```bash
   # Apply via kustomize
   kubectl apply -k path/to/your/manifests/
   ```

4. **Reference in your deployment:**
   ```yaml
   env:
   - name: DB_PASSWORD
     valueFrom:
       secretKeyRef:
         name: my-service-secret
         key: database-password
   ```

## 🛠️ Troubleshooting

### Common Issues

#### SealedSecret Won't Decrypt
```bash
# Check controller logs
kubectl logs -n kube-system deployment/sealed-secrets-controller

# Common error: "no key could decrypt secret"
# Solution: Generate new SealedSecret for your cluster
```

#### Controller Not Ready
```bash
# Check controller status
kubectl get pods -n kube-system | grep sealed-secrets

# Check service and endpoints
kubectl get svc sealed-secrets-controller -n kube-system
kubectl get endpoints sealed-secrets-controller -n kube-system
```

#### kubeseal Command Fails
```bash
# Test controller connectivity
kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --fetch-cert

# If fails, check if controller is accessible
kubectl port-forward -n kube-system service/sealed-secrets-controller 8080:8080
```

### Secret Management Tips

1. **Never commit plain text secrets** to Git
2. **Generate unique secrets** per environment (dev/staging/prod)
3. **Use strong passwords** and rotate regularly
4. **Document secret keys** but not values
5. **Backup your SealedSecrets controller private key** for disaster recovery

## 📚 References

- [SealedSecrets Documentation](https://sealed-secrets.netlify.app/)
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [GitOps Security Best Practices](https://www.weave.works/blog/storing-secure-sealed-secrets-using-gitops)

## 🆘 Need Help?

If you encounter issues:
1. Check the troubleshooting section above
2. Review controller logs: `kubectl logs -n kube-system deployment/sealed-secrets-controller`
3. Verify your kubeseal CLI version matches the controller version
4. Ensure your cluster has proper RBAC permissions

---

**⚠️ Security Note:** Each Kubernetes cluster has a unique SealedSecrets private key. SealedSecrets encrypted for one cluster will **not** work on another cluster. Always generate new secrets for each environment.