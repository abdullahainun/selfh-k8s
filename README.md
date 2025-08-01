<div align="center">
  <h3 align="center">Self-Hosted K8s Homelab</h3>
  <p align="center">
    Self-hosted Kubernetes template for homelab, baremetal, or cloud deployments with essential infrastructure and applications
    <br />
    <a href="docs/"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/abdullahainun/selfh-k8s/issues">Report Bug</a>
    ·
    <a href="https://github.com/abdullahainun/selfh-k8s/issues">Request Feature</a>
  </p>
</div>

> **🚧 Development Status:** This project is actively under development. New applications and features are being added regularly. While the infrastructure is production-ready, expect frequent updates and improvements. Check the [Issues](https://github.com/abdullahainun/selfh-k8s/issues) for current roadmap and known issues.

## Table of Contents

- [About](#about)
- [Features](#features)
- [Quick Start](#quick-start)
- [Security Setup](#security-setup)
- [Contributing](#contributing)
- [License](#license)

## About

A comprehensive Kubernetes template for deploying self-hosted services across homelab, baremetal, or cloud environments. Built with GitOps principles using FluxCD, this template provides essential infrastructure components and popular applications that you can fork, customize, and deploy to bootstrap your fresh cluster.

## Features

✅ **Essential Infrastructure Components**
- 🔐 Automated SSL certificates with cert-manager + Cloudflare
- 🗄️ Database services (PostgreSQL, MySQL, Redis)
- 📊 Observability stack (Prometheus, Grafana, AlertManager, Loki)
- 🛡️ Security policies with OPA Gatekeeper
- 🔒 Encrypted secrets management with SealedSecrets

✅ **GitOps-First Architecture**
- 🚀 FluxCD for declarative deployments
- 📦 Automated Helm chart management via HelmRepository/HelmRelease
- 🔄 Multi-environment overlays (dev/prod)
- 🔧 Mixed deployment patterns: GitOps + native Kubernetes manifests

✅ **Self-Hosted Applications**
- 🛠️ Development & collaboration tools
- 📝 Productivity & automation platforms  
- 🔐 Security & identity management
- 🚀 CI/CD & DevOps tooling
- 🎯 Extensible for additional services

### Built With

- ![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white) **Core Platform**
- ![FluxCD](https://img.shields.io/badge/flux-5468FF?style=for-the-badge&logo=flux&logoColor=white) **GitOps Controller** *(Critical for Helm-based deployments)*
- ![Helm](https://img.shields.io/badge/helm-0F1689?style=for-the-badge&logo=helm&logoColor=white) **Package Management**

## Quick Start

### Prerequisites

- **Kubernetes cluster** (1.24+) with storage class
- **kubectl** configured and connected
- **FluxCD** installed and configured for GitOps workflows
- **Domain name** for SSL certificates (Cloudflare DNS)
- **Basic understanding** of Kubernetes, Kustomize, and GitOps concepts

### Installation

1. **Fork and clone**
   ```bash
   git clone https://github.com/abdullahainun/selfh-k8s.git
   cd selfh-k8s
   ```

2. **Deploy core infrastructure** (in order)
   ```bash
   # Storage and networking
   kubectl apply -k infrastructure/storage/overlays/prod/
   kubectl apply -k infrastructure/metallb/overlays/prod/
   kubectl apply -k infrastructure/ingress-nginx/overlays/prod/
   
   # Security and certificates
   kubectl apply -k infrastructure/sealed-secrets/overlays/prod/
   kubectl apply -k infrastructure/cert-manager/overlays/prod/  # Configure secrets first
   
   # GitOps controller (required for Helm-based applications)
   kubectl apply -k infrastructure/flux-system/overlays/prod/
   ```

3. **Deploy databases** (after configuring sealed secrets)
   ```bash
   kubectl apply -k infrastructure/postgresql/overlays/prod/
   kubectl apply -k infrastructure/mysql/overlays/prod/
   kubectl apply -k infrastructure/redis/overlays/prod/
   ```

4. **Deploy applications** 
   ```bash
   # Browse available categories and applications
   ls apps/
   
   # Deploy applications from any category
   kubectl apply -k apps/<category>/<app-name>/overlays/prod/
   kubectl apply -k infrastructure/<service-name>/overlays/prod/
   ```
   
   > 📁 **Available Applications**: See [`apps/`](apps/) directory for current catalog

> **📋 Note:** Many applications use FluxCD's HelmRepository and HelmRelease for automated Helm chart deployment. Ensure FluxCD is running before deploying Helm-based services.

> **🚀 Advanced Option:** For automated GitOps deployment using FluxCD, see the [Advanced GitOps Guide](docs/advanced-gitops.md). This provides full automation where changes to Git automatically deploy to your cluster.

> **⚠️ Important:** Configure secrets and SSL certificates before deploying applications. See [SECRETS_SETUP.md](SECRETS_SETUP.md) for detailed setup.

## Security Setup

This repository uses **SealedSecrets** for secure credential management. When deploying to a new cluster:

1. **Install SealedSecrets controller**
2. **Generate your own encrypted secrets** 
3. **Update secret manifests** with your encrypted values

📋 **Detailed Guide:** [SECRETS_SETUP.md](SECRETS_SETUP.md)

### Required Secrets

| Service | Secret Name | Keys | Description |
|---------|-------------|------|-------------|
| Grafana | `grafana-admin` | `admin-user`, `admin-password` | Grafana admin credentials |
| PostgreSQL | `postgresql-credentials` | `postgres-password` | Database admin password |
| MySQL | `mysql-credentials` | `mysql-root-password` | Database root password |
| Vaultwarden | `vaultwarden-config` | `database-url`, `admin-token` | Vaultwarden database and admin access |

> **🔐 Security Note:** Each cluster generates unique encryption keys. SealedSecrets from one cluster will not work on another.

### Directory Structure

```
selfh-k8s/
├── apps/                   # Application services
│   ├── development/        # Code management & design tools
│   ├── productivity/       # Automation & collaboration platforms
│   ├── security/          # Identity & security management
│   ├── media/             # Media servers & content management
│   └── monitoring/        # Application monitoring tools
├── infrastructure/         # Platform components
│   ├── cert-manager/      # SSL certificate automation
│   ├── postgresql/        # Primary database services
│   ├── monitoring/        # Infrastructure observability
│   ├── jenkins/           # CI/CD & automation platform
│   ├── flux-system/       # GitOps deployment controller
│   └── ...               # Storage, networking, security
└── docs/                  # Setup guides & documentation
```


## Contributing

Contributions make the open source community amazing! Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/add-awesomeapp`)
3. Add your service to `apps/productivity/awesomeapp/`
4. Commit your Changes (`git commit -m 'feat: add awesomeapp service'`)
5. Push to the Branch (`git push origin feature/add-awesomeapp`)
6. Open a Pull Request

## License

Distributed under the MIT License. See `LICENSE` for more information.

## Contact

Ainun Abdullah - [@abdullahainun](https://github.com/abdullahainun)

Project Link: [https://github.com/abdullahainun/selfh-k8s](https://github.com/abdullahainun/selfh-k8s)

<p align="right">(<a href="#self-hosted-k8s-services">back to top</a>)</p>
