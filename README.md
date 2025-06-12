# Homelab K8s Services

> **Production-Ready Homelab Services for Kubernetes**
>
> Battle-tested Kubernetes manifests and Helm charts for self-hosted applications, running reliably in production homelab environments.

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Kustomize](https://img.shields.io/badge/Kustomize-EF7B4D?style=flat-square&logo=kubernetes&logoColor=white)](https://kustomize.io/)
[![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat-square&logo=helm&logoColor=white)](https://helm.sh/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)

## 🎯 What This Repository Offers

This repository contains **production-ready Kubernetes manifests** for popular self-hosted applications, designed and tested in real homelab environments. All services are configured with:

- ✅ **Security best practices** (RBAC, Network Policies, non-root containers)
- ✅ **Resource management** (limits, requests, health checks)
- ✅ **High availability** configurations where applicable
- ✅ **Persistent storage** strategies
- ✅ **Ingress and TLS** automation
- ✅ **Monitoring integration** ready

## 🏗️ Repository Structure

```
├── platform/          # Core infrastructure services
├── apps/              # Self-hosted applications
├── helm-charts/       # Helm packages for complex deployments
├── manifests/         # All-in-one compiled manifests
├── scripts/           # Automation and setup scripts
├── docs/              # Comprehensive documentation
└── examples/          # GitOps and quick-start examples
```

## 🚀 Quick Start

### Prerequisites
- Kubernetes cluster (v1.25+)
- kubectl configured
- Ingress controller capability
- Storage provisioner

### Deploy Platform Services
```bash
# Clone repository
git clone https://github.com/yourusername/homelab-k8s-services.git
cd homelab-k8s-services

# Deploy core platform
kubectl apply -k platform/cert-manager/overlays/homelab
kubectl apply -k platform/ingress-nginx/overlays/homelab
kubectl apply -k platform/metallb/overlays/homelab
```

### Deploy Applications
```bash
# Deploy productivity apps
kubectl apply -k apps/productivity/gitea/overlays/homelab
kubectl apply -k apps/productivity/vaultwarden/overlays/homelab
```

## 📦 Available Services

### Platform Services
| Service           | Description                             | Status        |
| ----------------- | --------------------------------------- | ------------- |
| **cert-manager**  | Automatic TLS certificate management    | ✅ Ready       |
| **ingress-nginx** | Ingress controller with SSL termination | ✅ Ready       |
| **metallb**       | Load balancer for bare metal            | ✅ Ready       |
| **monitoring**    | Prometheus + Grafana stack              | 🚧 In Progress |

### Applications

#### 🔧 Productivity
| Service         | Description                           | Status        |
| --------------- | ------------------------------------- | ------------- |
| **Gitea**       | Self-hosted Git service               | ✅ Ready       |
| **Vaultwarden** | Bitwarden-compatible password manager | ✅ Ready       |
| **Vikunja**     | Task and project management           | 🚧 In Progress |
| **Wiki.js**     | Modern wiki software                  | 🚧 In Progress |

#### 🎨 Media & Content
| Service        | Description                  | Status        |
| -------------- | ---------------------------- | ------------- |
| **Immich**     | Self-hosted photo management | 🚧 In Progress |
| **Calibre**    | E-book server and manager    | 🚧 In Progress |
| **Excalidraw** | Virtual whiteboard           | 🚧 In Progress |

#### 🤖 AI Tools
| Service        | Description                           | Status  |
| -------------- | ------------------------------------- | ------- |
| **Open WebUI** | ChatGPT-like interface for local LLMs | ✅ Ready |
| **LiteLLM**    | LLM proxy server                      | ✅ Ready |

## 🏠 My Homelab Setup

This repository powers my personal homelab:
- **Hardware**: 3x Mini PC M720q (Intel i3, 24GB RAM, 512GB SSD)
- **Kubernetes**: kubeadm cluster with 1 control plane + 2 workers
- **Storage**: Local persistent volumes + NFS
- **Network**: MetalLB + Ingress-nginx + Cloudflare DNS
- **Monitoring**: Prometheus + Grafana + AlertManager

**Uptime**: Running these services reliably for 12+ months with minimal downtime.

## 📚 Documentation

- [🏗️ Architecture Overview](docs/architecture.md)
- [🚀 Getting Started Guide](docs/getting-started.md)
- [⚙️ Hardware Requirements](docs/hardware-requirements.md)
- [🔧 Troubleshooting](docs/troubleshooting.md)
- [🤝 Contributing](docs/contributing.md)

## 🛡️ Security

All services are configured with security best practices:
- Non-root containers where possible
- Resource limits and requests
- Network policies for traffic isolation
- RBAC with least privilege principle
- Regular security updates

## 🤝 Contributing

Contributions are welcome! Please check our [Contributing Guide](docs/contributing.md) for:
- How to add new services
- Code standards and best practices
- Testing requirements
- Documentation guidelines

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⭐ Star History

If this repository helps you build your homelab, please consider giving it a star! ⭐

---

**Built with ❤️ for the homelab community**
