# Infrastructure

Platform-level components and services that provide the foundation for self-hosted applications.

## Components

### 🌐 ingress-nginx
NGINX Ingress Controller for routing external traffic to services.
- *Coming soon: Complete ingress-nginx setup*

### 🔒 cert-manager  
Automatic SSL/TLS certificate management with Let's Encrypt.
- *Coming soon: cert-manager with ACME configuration*

### 💾 storage
Persistent volume configurations and storage classes.
- *Coming soon: Local storage, NFS, and cloud storage configs*

### 📛 namespace
Kubernetes namespace definitions for organizing applications.
- `namespace/` - Basic namespace setup

## Usage

Deploy infrastructure components before applications:

```bash
# 1. Create namespaces
kubectl apply -k infrastructure/namespace

# 2. Setup storage
kubectl apply -k infrastructure/storage

# 3. Install ingress controller
kubectl apply -k infrastructure/ingress-nginx

# 4. Setup certificate management
kubectl apply -k infrastructure/cert-manager
```

## Dependencies

Infrastructure components should be deployed in this order:
1. **namespace** - Create logical separation
2. **storage** - Persistent volume support
3. **ingress-nginx** - External access
4. **cert-manager** - SSL certificates

## Remote Usage

```yaml
# Use infrastructure as remote base
resources:
  - https://github.com/username/selfh-k8s//infrastructure/namespace?ref=main
  - https://github.com/username/selfh-k8s//infrastructure/ingress-nginx?ref=main
```