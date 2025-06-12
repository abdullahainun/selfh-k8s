# Getting Started Guide

## Prerequisites

### Kubernetes Cluster Requirements
- Kubernetes version 1.25 or higher
- At least 3 worker nodes (recommended)
- 8GB RAM minimum per node
- 100GB storage per node

### Required Tools
- `kubectl` configured for your cluster
- `kustomize` (optional, included in kubectl 1.14+)
- `helm` (for Helm-based deployments)

## Step-by-Step Setup

### 1. Clone Repository
```bash
git clone https://github.com/yourusername/homelab-k8s-services.git
cd homelab-k8s-services
```

### 2. Customize Configuration
```bash
# Copy and modify configuration templates
cp platform/metallb/overlays/homelab/ipaddresspool.yaml.template \
   platform/metallb/overlays/homelab/ipaddresspool.yaml

# Edit IP ranges to match your network
vim platform/metallb/overlays/homelab/ipaddresspool.yaml
```

### 3. Deploy Platform Services
```bash
# Deploy in order
kubectl apply -k platform/cert-manager/overlays/homelab
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=cert-manager -n cert-manager --timeout=300s

kubectl apply -k platform/ingress-nginx/overlays/homelab
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx --timeout=300s

kubectl apply -k platform/metallb/overlays/homelab
```

### 4. Deploy Applications
```bash
# Deploy productivity apps
kubectl apply -k apps/productivity/gitea/overlays/homelab
kubectl apply -k apps/productivity/vaultwarden/overlays/homelab
```

### 5. Verify Deployment
```bash
# Check all services
kubectl get pods --all-namespaces
kubectl get ingress --all-namespaces
```

## Configuration Guide

### Environment Customization
Each service can be customized through Kustomize overlays in the `overlays/homelab` directory.

### Secret Management
1. Copy `.template` files to actual secret files
2. Fill in your actual values
3. Apply secrets before applications

## Troubleshooting

See [Troubleshooting Guide](troubleshooting.md) for common issues and solutions.
