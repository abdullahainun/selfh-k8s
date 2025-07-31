# 🚀 Advanced GitOps with FluxCD

This guide shows how to implement full GitOps workflow using FluxCD for automated deployments. This is an **advanced setup** that complements the default manual deployment approach.

## 📋 Overview

**Default Approach vs GitOps:**

| Aspect | Manual (Default) | GitOps (Advanced) |
|--------|------------------|-------------------|
| **Deployment** | `kubectl apply -k` | Automatic from Git |
| **Updates** | Manual re-apply | Git push → auto-deploy |
| **Rollback** | Manual commands | Git revert → auto-rollback |
| **Complexity** | Simple | Advanced |
| **Best for** | Learning, experimentation | Production, automation |

## 🎯 When to Use GitOps

**✅ Choose GitOps if:**
- You want full automation
- Multiple people manage the cluster
- You need audit trails via Git
- You prefer declarative over imperative
- You're comfortable with FluxCD concepts

**⚠️ Stick with Manual if:**
- You're learning Kubernetes basics
- You frequently experiment with configurations
- You prefer direct control over deployments
- You want simpler troubleshooting

## 🚀 Implementation Steps

### Prerequisites

- Existing selfh-k8s repository setup
- FluxCD CLI installed ([Installation Guide](https://fluxcd.io/flux/installation/))
- GitHub token with repo permissions
- Basic understanding of Kubernetes and Git

### Step 1: Bootstrap FluxCD

**1.1 Install FluxCD CLI:**
```bash
# macOS
brew install fluxcd/tap/flux

# Linux
curl -s https://fluxcd.io/install.sh | sudo bash

# Verify installation
flux --version
```

**1.2 Fork and Clone Repository:**
```bash
# Fork repository on GitHub first, then:
git clone https://github.com/yourusername/selfh-k8s.git
cd selfh-k8s
```

**1.3 Bootstrap FluxCD:**
```bash
# Set GitHub credentials
export GITHUB_TOKEN=<your-github-token>
export GITHUB_USER=<your-github-username>

# Bootstrap FluxCD with this repository
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=selfh-k8s \
  --branch=main \
  --path=./cluster/base \
  --personal
```

This command will:
- Install FluxCD in your cluster
- Create `cluster/` directory in your repository
- Set up automatic synchronization between Git and cluster

### Step 2: Create GitOps Structure

**2.1 Create Cluster Directory Structure:**
```bash
mkdir -p cluster/{base,infrastructure,apps}
```

**2.2 Create Base Kustomization:**
```yaml
# cluster/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../infrastructure/
  - ../apps/
```

**2.3 Create Infrastructure Kustomizations:**
```yaml
# cluster/infrastructure/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - core.yaml
  - databases.yaml
  - monitoring.yaml
```

### Step 3: Define Deployment Order

**3.1 Core Infrastructure:**
```yaml
# cluster/infrastructure/core.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-core
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/storage/overlays/prod"
  prune: true
  wait: true
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-networking
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/metallb/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-core
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-ingress
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/ingress-nginx/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-networking
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-security
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/sealed-secrets/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-ingress
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-certificates
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/cert-manager/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-security
```

**3.2 Database Layer:**
```yaml
# cluster/infrastructure/databases.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-postgresql
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/postgresql/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-certificates
  healthChecks:
    - apiVersion: apps/v1
      kind: StatefulSet
      name: postgresql
      namespace: postgresql
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-mysql
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/mysql/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-certificates
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-redis
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/redis/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-certificates
```

**3.3 Monitoring Stack:**
```yaml
# cluster/infrastructure/monitoring.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-monitoring
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/monitoring/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-postgresql
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: grafana
      namespace: monitoring
    - apiVersion: apps/v1
      kind: StatefulSet
      name: prometheus-server
      namespace: monitoring
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure-jenkins
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./infrastructure/jenkins/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-monitoring
```

### Step 4: Configure Application Deployments

**4.1 Create Application Kustomizations:**
```yaml
# cluster/apps/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - development.yaml
  - productivity.yaml
  - security.yaml
```

**4.2 Development Applications:**
```yaml
# cluster/apps/development.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps-development-gitea
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./apps/development/gitea/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-postgresql
    - name: infrastructure-monitoring
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps-development-chartdb
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./apps/development/chartdb/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-monitoring
```

**4.3 Productivity Applications:**
```yaml
# cluster/apps/productivity.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps-productivity-n8n
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./apps/productivity/n8n/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-postgresql
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps-productivity-activepieces
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: "./apps/productivity/activepieces/overlays/prod"
  prune: true
  dependsOn:
    - name: infrastructure-monitoring
```

### Step 5: Deploy and Verify

**5.1 Commit and Push:**
```bash
git add cluster/
git commit -m "feat: add GitOps cluster configuration"
git push origin main
```

**5.2 Verify Deployment:**
```bash
# Check FluxCD status
flux get all

# Check Kustomizations
flux get kustomizations

# Check specific deployments
kubectl get all -A

# Monitor deployment progress
flux logs --follow --tail=10
```

## 🔍 Monitoring and Management

### Check Deployment Status

```bash
# Overall FluxCD health
flux check

# Specific kustomization status
flux get kustomization infrastructure-core

# Application status
flux get helmreleases -A

# Troubleshoot issues
flux logs --level=error --tail=50
```

### Manual Reconciliation

```bash
# Force sync specific kustomization
flux reconcile kustomization infrastructure-core

# Force sync all
flux reconcile source git flux-system
```

### Suspend/Resume

```bash
# Suspend automatic deployment
flux suspend kustomization apps-development-gitea

# Resume automatic deployment
flux resume kustomization apps-development-gitea
```

## 🔧 Troubleshooting

### Common Issues

**1. Kustomization Fails to Apply:**
```bash
# Check kustomization status
flux describe kustomization infrastructure-core

# Check source
flux describe source git flux-system

# Manual validation
kubectl apply --dry-run=client -k infrastructure/storage/overlays/prod/
```

**2. Dependency Issues:**
```bash
# Check dependency tree
flux tree kustomization infrastructure-monitoring

# Verify dependent resources exist
kubectl get kustomization -n flux-system
```

**3. HelmRelease Issues:**
```bash
# Check Helm release status
flux get helmrelease jenkins -n jenkins

# Check Helm repository
flux get source helm -A

# Manual Helm operations
helm list -A
```

### Recovery Procedures

**1. Reset Kustomization:**
```bash
# Delete and recreate
flux delete kustomization infrastructure-core
git push  # Will recreate automatically
```

**2. Force Reconciliation:**
```bash
# Update source
flux reconcile source git flux-system

# Update specific kustomization
flux reconcile kustomization infrastructure-core --with-source
```

## 🎯 Best Practices

### 1. Deployment Strategy

- **Start small**: Begin with core infrastructure
- **Use dependencies**: Ensure proper ordering
- **Health checks**: Add health checks for critical components
- **Gradual rollout**: Test in dev environment first

### 2. Repository Management

- **Branch protection**: Protect main branch
- **Pull requests**: Review changes before deployment
- **Semantic commits**: Use conventional commit messages
- **Secrets management**: Never commit plain secrets

### 3. Monitoring

- **FluxCD alerts**: Set up notifications for failures
- **Resource monitoring**: Monitor cluster resources
- **Log aggregation**: Centralize FluxCD logs
- **Regular health checks**: Automated flux check

### 4. Security

- **RBAC**: Limit FluxCD permissions
- **Network policies**: Restrict FluxCD network access
- **Image scanning**: Scan container images
- **Secrets rotation**: Regular secret rotation

## 🔄 Migration from Manual

### Gradual Migration Strategy

**Phase 1: Infrastructure Only**
```bash
# Start with core infrastructure
mkdir cluster/infrastructure
# Move only storage, networking, security
```

**Phase 2: Add Databases**
```bash
# Add database layer
# Keep applications manual for now
```

**Phase 3: Critical Applications**
```bash
# Add monitoring and essential apps
# Test thoroughly before proceeding
```

**Phase 4: All Applications**
```bash
# Migrate remaining applications
# Remove manual deployment documentation
```

### Rollback Plan

**If GitOps causes issues:**
```bash
# Suspend all FluxCD operations
flux suspend kustomization --all

# Return to manual deployment
kubectl apply -k infrastructure/storage/overlays/prod/
# ... continue with manual commands
```

## 📚 Additional Resources

- [FluxCD Documentation](https://fluxcd.io/flux/)
- [GitOps Toolkit](https://toolkit.fluxcd.io/)
- [Kustomize Documentation](https://kustomize.io/)
- [Kubernetes GitOps Best Practices](https://www.weave.works/technologies/gitops/)

## 🆘 Getting Help

**If you encounter issues:**

1. **Check FluxCD status**: `flux check`
2. **Review logs**: `flux logs --tail=100`
3. **Validate manifests**: `flux diff kustomization <name>`
4. **Community support**: [FluxCD Slack](https://cloud-native.slack.com/)
5. **Repository issues**: [GitHub Issues](https://github.com/abdullahainun/selfh-k8s/issues)

---

**⚠️ Important Note:** This is an advanced setup. Ensure you understand the implications before implementing in production. The default manual approach is perfectly valid for homelab use cases.