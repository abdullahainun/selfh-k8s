<div align="center">
  <h3 align="center">Self-Hosted K8s Homelab</h3>
  <p align="center">
    Production-ready Kubernetes homelab with GitOPS, databases, monitoring, and security
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
- [Infrastructure](#infrastructure)
- [Applications](#applications)
- [Security Setup](#security-setup)
- [Contributing](#contributing)
- [License](#license)

## About

A complete Kubernetes homelab template with production-ready infrastructure and applications. Built with GitOps principles using FluxCD, this template enables you to fork, customize, and deploy self-hosted services with enterprise-grade security and observability.

## Features

✅ **Complete Infrastructure Stack**
- 🔐 SSL certificates with cert-manager + Cloudflare
- 🗄️ Database layer (PostgreSQL, MySQL, Redis)
- 📊 Monitoring stack (Prometheus, Grafana, AlertManager)
- 🛡️ Security policies with OPA Gatekeeper
- 🔒 Sealed secrets for credential management

✅ **GitOps-First Architecture**
- 🚀 FluxCD for declarative deployments
- 📦 Automated Helm chart management via HelmRepository/HelmRelease
- 🔄 Multi-environment overlays (dev/prod)
- 🔧 Mixed deployment patterns: GitOps + native Kubernetes manifests

✅ **Production Applications**
- 🛠️ Development & collaboration tools
- 📝 Productivity & automation platforms
- 🔐 Security & identity management
- 🚀 CI/CD & DevOps tooling
- 🎯 Ready for media and monitoring apps

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

4. **Deploy applications** (examples)
   ```bash
   # Browse available applications
   ls apps/*/
   
   # Deploy specific categories
   kubectl apply -k apps/development/<app-name>/overlays/prod/
   kubectl apply -k apps/productivity/<app-name>/overlays/prod/
   kubectl apply -k apps/security/<app-name>/overlays/prod/
   
   # Deploy infrastructure services
   kubectl apply -k infrastructure/<service-name>/overlays/prod/
   ```

> **📋 Note:** Many applications use FluxCD's HelmRepository and HelmRelease for automated Helm chart deployment. Ensure FluxCD is running before deploying Helm-based services.

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

## Infrastructure

### Core Components

| Component | Purpose | Status |
|-----------|---------|--------|
| **cert-manager** | SSL certificates via Cloudflare | ✅ Ready |
| **sealed-secrets** | Encrypted secrets management | ✅ Ready |
| **metallb** | Load balancer for bare metal | ✅ Ready |
| **ingress-nginx** | HTTP/HTTPS traffic routing | ✅ Ready |
| **storage** | Local path provisioner | ✅ Ready |

### Data & Observability

| Component | Purpose | Status |
|-----------|---------|--------|
| **postgresql** | Primary database | ✅ Ready |
| **mysql** | Secondary database | ✅ Ready |
| **redis** | Cache and sessions | ✅ Ready |
| **monitoring** | Prometheus + Grafana stack | ✅ Ready |
| **gatekeeper** | Policy enforcement | ✅ Ready |

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

## Applications

> **🚀 Advanced Option:** For automated GitOps deployment using FluxCD, see the [Advanced GitOps Guide](docs/advanced-gitops.md). This provides full automation where changes to Git automatically deploy to your cluster.

### Application Categories

#### 🛠️ Development & Collaboration
Code management, database design, documentation, and team collaboration tools

#### 📝 Productivity & Automation  
Workflow automation, business process management, communication APIs, and productivity suites

#### 🔐 Security & Identity
Password management, authentication services, and security monitoring tools

#### 🚀 CI/CD & DevOps
Continuous integration, deployment pipelines, and infrastructure automation

#### 🎬 Media & Content
Media servers, content management, and streaming services *(ready for deployment)*

#### 📊 Monitoring & Observability
Application monitoring, log aggregation, and alerting systems *(infrastructure included)*

### Adding New Apps

**Manual Deployment:**
1. Create app structure: `apps/<category>/<app>/base/`
2. Add Kubernetes manifests
3. Create dev/prod overlays
4. Configure ingress and secrets
5. Deploy with `kubectl apply -k`

**GitOps Deployment:**
1. Follow steps 1-4 above
2. Add Kustomization CRD to `cluster/apps/<category>.yaml`
3. Commit to Git - FluxCD will automatically deploy

**Contributing:**
5. Submit pull request with your changes

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
