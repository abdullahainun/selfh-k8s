# Cert-Manager - Automatic SSL/TLS Certificate Management

Automated certificate management using cert-manager with Let's Encrypt DNS-01 challenge via CloudFlare. Template ready for any domain - just customize and deploy!

> **🚀 Quick Start**: See [SETUP.md](./SETUP.md) for step-by-step configuration guide with your domain.

## Features

- **Let's Encrypt Integration**: Automatic SSL certificate provisioning and renewal
- **DNS-01 Challenge**: Works with private clusters (no public IP required)  
- **CloudFlare DNS**: Automated DNS challenge via CloudFlare API
- **Multiple Issuers**: Staging and production Let's Encrypt issuers
- **Wildcard Support**: Single certificate for all subdomains
- **GitOps Ready**: Kustomize-based configuration

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   cert-manager  │────│  Let's Encrypt  │────│   CloudFlare    │
│   Controller    │    │   ACME Server   │    │   DNS API       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                        │                        │
         │                        │                        │
    ┌─────────┐              ┌─────────┐              ┌─────────┐
    │ Create  │              │ DNS-01  │              │ TXT     │
    │ Certificate │          │ Challenge│             │ Record  │
    └─────────┘              └─────────┘              └─────────┘
```

## Prerequisites

### 1. CloudFlare Setup

1. **Domain**: Your domain managed by CloudFlare (replace `example.com` in config files)
2. **API Token**: Create token with permissions:
   - `Zone:Zone:Read` for your domain
   - `Zone:DNS:Edit` for your domain

### 2. Create CloudFlare API Token

1. Go to [CloudFlare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. Click "Create Token"
3. Use "Custom token" template:
   ```
   Permissions:
   - Zone : Zone : Read
   - Zone : DNS : Edit
   
   Zone Resources:
   - Include : Specific zone : yourdomain.com
   ```
4. Copy the generated token

## Deployment

### Step 1: Deploy cert-manager

```bash
# Development (uses staging by default)
kubectl apply -k infrastructure/cert-manager/overlays/dev

# Production 
kubectl apply -k infrastructure/cert-manager/overlays/prod
```

### Step 2: Create CloudFlare Secret

**Option A: Direct kubectl (for testing)**
```bash
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=YOUR_CLOUDFLARE_API_TOKEN \
  -n cert-manager
```

**Option B: Sealed Secret (recommended for GitOps)**
```bash
# Create temporary secret
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=YOUR_CLOUDFLARE_API_TOKEN \
  --dry-run=client -o yaml > cloudflare-secret.yaml

# Seal it
kubeseal -f cloudflare-secret.yaml -w cloudflare-sealed-secret.yaml \
  --controller-namespace sealed-secrets

# Apply sealed secret
kubectl apply -f cloudflare-sealed-secret.yaml

# Clean up temporary file
rm cloudflare-secret.yaml
```

### Step 3: Verify Installation

```bash
# Check cert-manager pods
kubectl get pods -n cert-manager

# Check ClusterIssuers
kubectl get clusterissuer

# Check if CloudFlare secret exists
kubectl get secret -n cert-manager cloudflare-api-token
```

## Available ClusterIssuers

### 1. Let's Encrypt Staging
- **Name**: `letsencrypt-staging`
- **Use**: Testing and development
- **Rate Limits**: Generous (for testing)
- **Trust**: Not trusted by browsers (shows warning)

### 2. Let's Encrypt Production  
- **Name**: `letsencrypt-prod`
- **Use**: Production certificates
- **Rate Limits**: 50 certificates/week per domain
- **Trust**: Trusted by all browsers

### 3. Self-Signed
- **Name**: `selfsigned`
- **Use**: Development without external dependencies
- **Trust**: Not trusted (requires manual trust)

## Certificate Examples

### Wildcard Certificate (Recommended)

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-ainun-cloud
  namespace: default
spec:
  secretName: wildcard-ainun-cloud-tls
  issuerRef:
    name: letsencrypt-staging  # Use prod for production
    kind: ClusterIssuer
  dnsNames:
  - "*.ainun.cloud"
  - "ainun.cloud"
```

### Individual Service Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: grafana-cert
  namespace: monitoring
spec:
  secretName: grafana-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - "grafana.ainun.cloud"
```

### Using Certificate in Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - grafana.ainun.cloud
    secretName: grafana-tls
  rules:
  - host: grafana.ainun.cloud
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

## Testing Certificate Issuance

### 1. Create Test Certificate

```bash
kubectl apply -f infrastructure/cert-manager/examples/wildcard-certificate.yaml
```

### 2. Monitor Certificate Status

```bash
# Watch certificate creation
kubectl get certificate -A -w

# Check certificate details
kubectl describe certificate wildcard-ainun-cloud

# Check certificate secret
kubectl get secret wildcard-ainun-cloud-tls -o yaml
```

### 3. Check cert-manager Logs

```bash
kubectl logs -n cert-manager deployment/cert-manager -f
```

## DNS Challenge Process

1. **Certificate Request**: User/Ingress requests certificate
2. **Challenge Creation**: cert-manager creates ACME challenge
3. **DNS Record**: cert-manager creates TXT record via CloudFlare API
4. **Verification**: Let's Encrypt verifies TXT record exists
5. **Certificate Issuance**: Let's Encrypt issues certificate
6. **Secret Creation**: cert-manager stores certificate in Kubernetes Secret
7. **Cleanup**: TXT record is removed

## Troubleshooting

### Common Issues

1. **Certificate Stuck in Pending**
   ```bash
   kubectl describe certificate <cert-name>
   kubectl describe challenge <challenge-name>
   ```

2. **CloudFlare Authentication Failed**
   ```bash
   # Check API token secret
   kubectl get secret -n cert-manager cloudflare-api-token -o yaml
   
   # Verify API token permissions in CloudFlare dashboard
   ```

3. **DNS Challenge Failed**
   ```bash
   # Check if TXT record was created
   dig _acme-challenge.grafana.ainun.cloud TXT
   
   # Check cert-manager controller logs
   kubectl logs -n cert-manager deployment/cert-manager
   ```

4. **Rate Limit Exceeded**
   ```bash
   # Check Let's Encrypt rate limits
   # Switch to staging issuer for testing
   # Wait for rate limit reset (weekly)
   ```

### Debug Commands

```bash
# List all certificates across namespaces
kubectl get certificate -A

# Check ClusterIssuer status
kubectl describe clusterissuer letsencrypt-prod

# View certificate events
kubectl get events --field-selector involvedObject.kind=Certificate

# Check ACME account status
kubectl get secret -n cert-manager letsencrypt-prod -o yaml

# Test CloudFlare API connectivity
kubectl run --rm -i --tty debug --image=curlimages/curl --restart=Never -- \
  curl -X GET "https://api.cloudflare.com/client/v4/zones" \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

## Security Considerations

### API Token Security
- **Minimal Permissions**: Only Zone:Read and DNS:Edit for specific zone
- **Secret Management**: Use sealed-secrets for GitOps
- **Token Rotation**: Regularly rotate CloudFlare API tokens
- **Namespace Isolation**: Store secrets in cert-manager namespace

### Certificate Management
- **Private Key Security**: Kubernetes manages private keys securely
- **Certificate Renewal**: Automatic renewal 30 days before expiry
- **Backup Strategy**: Include certificate secrets in cluster backups

## Production Recommendations

### 1. Certificate Strategy
- **Wildcard Certificates**: Use `*.ainun.cloud` for simplicity
- **Per-Service Certificates**: Use individual certs for sensitive services
- **Certificate Namespaces**: Deploy certificates in same namespace as service

### 2. Monitoring
- **Certificate Expiry**: Monitor certificate expiration dates
- **Renewal Status**: Alert on failed certificate renewals
- **Challenge Success**: Monitor DNS challenge success rates

### 3. DNS Management
- **Zone Delegation**: Consider delegating subzones for better organization
- **TTL Configuration**: Use appropriate TTL values for DNS records
- **Backup DNS**: Have backup DNS provider configuration

## Integration Examples

### With Monitoring Stack

```yaml
# Grafana with automatic HTTPS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - grafana.ainun.cloud
    secretName: grafana-tls
  rules:
  - host: grafana.ainun.cloud
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

### Development Environment

```yaml
# Development services with staging certificates
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: dev-services
  namespace: dev
spec:
  secretName: dev-services-tls
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
  dnsNames:
  - "*.dev.ainun.cloud"
```

This setup provides enterprise-grade SSL certificate management for your private Kubernetes cluster! 🔐