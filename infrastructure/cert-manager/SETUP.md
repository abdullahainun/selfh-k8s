# Quick Setup Guide - Cert-Manager with CloudFlare DNS-01

Step-by-step guide to configure cert-manager for your domain with CloudFlare DNS-01 challenge.

## Prerequisites

1. **Domain managed by CloudFlare** (replace all `example.com` with your domain)
2. **CloudFlare API Token** with Zone:Read and DNS:Edit permissions
3. **Kubernetes cluster** with kubectl access

## Step 1: Customize Configuration

### 1.1 Update Domain and Email

Edit these files and replace the placeholder values:

**File: `infrastructure/cert-manager/base/cluster-issuer.yaml`**
```bash
# Replace in the file:
email: admin@example.com        → email: your-email@yourdomain.com
dnsZones: ["example.com"]      → dnsZones: ["yourdomain.com"]
```

**File: `infrastructure/cert-manager/examples/wildcard-certificate.yaml`**
```bash
# Replace in the file:
"*.example.com"     → "*.yourdomain.com"
"example.com"       → "yourdomain.com"
"grafana.example.com" → "grafana.yourdomain.com"
```

### 1.2 Quick Replace Script

```bash
# Run this in the repository root
export YOUR_DOMAIN="yourdomain.com"
export YOUR_EMAIL="admin@yourdomain.com"

# Replace domain in cluster-issuer
sed -i "s/example\.com/$YOUR_DOMAIN/g" infrastructure/cert-manager/base/cluster-issuer.yaml
sed -i "s/admin@example\.com/$YOUR_EMAIL/g" infrastructure/cert-manager/base/cluster-issuer.yaml

# Replace domain in examples
sed -i "s/example\.com/$YOUR_DOMAIN/g" infrastructure/cert-manager/examples/wildcard-certificate.yaml
```

## Step 2: Create CloudFlare API Token

1. Go to [CloudFlare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. Click **"Create Token"**
3. Use **"Custom token"** template
4. Configure permissions:
   ```
   Permissions:
   - Zone : Zone : Read
   - Zone : DNS : Edit
   
   Zone Resources:
   - Include : Specific zone : yourdomain.com
   ```
5. **Copy the generated token** (you'll need it in Step 4)

## Step 3: Deploy Cert-Manager

Choose your environment:

### Development (uses staging Let's Encrypt)
```bash
kubectl apply -k infrastructure/cert-manager/overlays/dev
```

### Production (uses production Let's Encrypt)
```bash
kubectl apply -k infrastructure/cert-manager/overlays/prod
```

## Step 4: Create CloudFlare Secret

### Option A: Direct Secret (Quick Testing)
```bash
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=YOUR_CLOUDFLARE_API_TOKEN \
  -n cert-manager
```

### Option B: Sealed Secret (GitOps Recommended)
```bash
# Create temporary secret file
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=YOUR_CLOUDFLARE_API_TOKEN \
  --dry-run=client -o yaml > /tmp/cloudflare-secret.yaml

# Seal the secret (requires kubeseal)
kubeseal -f /tmp/cloudflare-secret.yaml \
  -w infrastructure/cert-manager/base/cloudflare-sealed-secret.yaml

# Apply sealed secret
kubectl apply -f infrastructure/cert-manager/base/cloudflare-sealed-secret.yaml

# Clean up
rm /tmp/cloudflare-secret.yaml
```

## Step 5: Verify Installation

```bash
# Check cert-manager pods
kubectl get pods -n cert-manager

# Check ClusterIssuers are ready
kubectl get clusterissuer

# Expected output:
# NAME                 READY   AGE
# letsencrypt-staging  True    1m
# letsencrypt-prod     True    1m
# selfsigned           True    1m
```

## Step 6: Test Certificate Creation

### Create Test Certificate
```bash
kubectl apply -f infrastructure/cert-manager/examples/wildcard-certificate.yaml
```

### Monitor Certificate Status
```bash
# Watch certificate creation
kubectl get certificate -A -w

# Check certificate details
kubectl describe certificate wildcard-example-com

# Should show "Certificate is up to date and has not expired"
```

## Step 7: Verify DNS Challenge

During certificate creation, you can verify the DNS challenge:

```bash
# Check if TXT record was created (replace with your domain)
dig _acme-challenge.yourdomain.com TXT

# Should show a TXT record with ACME challenge
```

## Troubleshooting

### Certificate Stuck in Pending
```bash
kubectl describe certificate <certificate-name>
kubectl describe challenge <challenge-name>
kubectl logs -n cert-manager deployment/cert-manager
```

### Common Issues
1. **Wrong API Token**: Verify permissions in CloudFlare dashboard
2. **Domain Mismatch**: Ensure domain in certificate matches CloudFlare zone
3. **Rate Limits**: Use staging issuer for testing, production for final deployment

## Usage Examples

### Ingress with Automatic Certificate
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - app.yourdomain.com
    secretName: my-app-tls
  rules:
  - host: app.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

### Wildcard Certificate for Multiple Services
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-cert
  namespace: default
spec:
  secretName: wildcard-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - "*.yourdomain.com"
  - "yourdomain.com"
```

## Production Checklist

- [ ] Domain configured in CloudFlare
- [ ] API token created with minimal permissions
- [ ] Email address updated in cluster-issuer
- [ ] Domain updated in all configuration files
- [ ] Tested with staging issuer first
- [ ] Switched to production issuer
- [ ] Certificates created successfully
- [ ] Services accessible via HTTPS

Your cert-manager setup is now ready for automatic SSL certificate management! 🔐