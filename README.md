# 🏠 Homelab K8s Services

A **GitOps-powered** Kubernetes homelab with **Cloudflare Tunnel** integration, featuring **automated preview environments** and **community-driven service catalog**.

## 🚀 Quick Start

### Deploy Your Service
1. **Fork** this repository
2. **Add your service** to `apps/<category>/<service>/`
3. **Create a Pull Request**
4. **Comment** `/preview` to deploy preview environment
5. **Merge** to deploy to production

### Available Commands
```bash
# Preview & Planning (Everyone)
/help                    # Show available commands
/plan                    # Show deployment plan
/status                  # Check preview environments

# Deployment (Maintainers & Collaborators)
/preview                 # Deploy all changed services
/preview ai/open-webui   # Deploy specific service
/cleanup                 # Manual cleanup
```

## 📁 Directory Structure
```
homelab-k8s/
├── apps/                    # Application services
│   ├── ai/                  # AI/ML services
│   │   └── open-webui/      # Example service
│   ├── dev/                 # Development tools
│   └── media/               # Media services
├── platform/               # Platform components
│   ├── ingress-nginx/       # Ingress controller
│   ├── cert-manager/        # TLS certificates
│   └── metallb/             # Load balancer
└── clusters/homelab/        # Cluster configuration
    └── flux-system/         # GitOps configuration
```

## 🛠️ Adding a New Service

### 1. Create Service Directory
```bash
mkdir -p apps/<category>/<service-name>/base
cd apps/<category>/<service-name>/base
```

### 2. Add Kubernetes Manifests
Create your service files:
- `deployment.yaml` - Your application
- `service.yaml` - Service exposure
- `kustomization.yaml` - Kustomize config

### 3. Example Service Structure
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx:latest
        ports:
        - containerPort: 80
---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
```

## 🌐 Preview Environments

When you create a PR:
- **Auto-generated domains**: `pr-123-service.abdullahainun.site`
- **Zero-trust protection**: Cloudflare security
- **Auto-cleanup**: Removed when PR closes
- **Resource limits**: Fair usage policies

## 🤝 Contributing

### For Everyone
- 📋 Use `/plan` to explore deployments (read-only)
- 📊 Use `/status` to check environments
- 🤝 Contribute improvements via PRs

### For Collaborators
- 🚀 Deploy preview environments with `/preview`
- 🧹 Cleanup resources with `/cleanup`
- ✅ Full access to all commands

### Want Deploy Access?
- Become a regular contributor
- Request collaborator access from [@abdullahainun](https://github.com/abdullahainun)
- Join the homelab community!

## 🔧 Tech Stack

- **🚢 Kubernetes**: Container orchestration
- **🔄 Flux CD**: GitOps continuous deployment
- **🌐 Ingress NGINX**: Traffic routing
- **☁️ Cloudflare Tunnel**: Secure external access & DNS
- **🤖 GitHub Actions**: CI/CD automation

## 📊 Cluster Info

```bash
# Cluster nodes
homelab-k8s-cp-1       # Control plane
homelab-k8s-worker-1   # Worker node
homelab-k8s-worker-2   # Worker node
```

## 📞 Support

- 🐛 **Issues**: [GitHub Issues](https://github.com/abdullahainun/homelab-k8s/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/abdullahainun/homelab-k8s/discussions)
- 📧 **Contact**: [@abdullahainun](https://github.com/abdullahainun)

---

⭐ **Star this repo** if you find it useful! | 🤝 **Contributions welcome** | 🏠 **Built with ❤️ for the homelab community**
