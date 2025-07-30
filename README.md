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

## Table of Contents

- [About](#about)
- [Features](#features)
- [Quick Start](#quick-start)
- [Infrastructure](#infrastructure)
- [Applications](#applications)
- [Contributing](#contributing)
- [License](#license)

## About

A complete Kubernetes homelab template with production-ready infrastructure and applications. Fork, customize, and deploy your own self-hosted services with enterprise-grade security and observability.

## Features

✅ **Complete Infrastructure Stack**
- 🔐 SSL certificates with cert-manager + Cloudflare
- 🗄️ Database layer (PostgreSQL, MySQL, Redis)
- 📊 Monitoring stack (Prometheus, Grafana, AlertManager)
- 🛡️ Security policies with OPA Gatekeeper
- 🔒 Sealed secrets for credential management

✅ **GitOps Ready**
- 🚀 FluxCD for automated deployments
- 📦 Helm integration for complex applications
- 🔄 Multi-environment overlays (dev/prod)

✅ **Production Applications**
- 🛠️ Development tools (ChartDB, OpenGist)
- 📝 Productivity apps (Excalidraw)
- 🎯 Ready for media and monitoring apps

### Built With

- ![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
- ![FluxCD](https://img.shields.io/badge/flux-5468FF?style=for-the-badge&logo=flux&logoColor=white)
- ![Helm](https://img.shields.io/badge/helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)

## Quick Start

### Prerequisites

- **Kubernetes cluster** (1.24+) with storage class
- **kubectl** configured and connected
- **Domain name** for SSL certificates (Cloudflare DNS)
- **Basic understanding** of Kubernetes and Kustomize

### Installation

1. **Fork and clone**
   ```bash
   git clone https://github.com/yourusername/selfh-k8s.git
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
   
   # GitOps (optional)
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
   kubectl apply -k apps/development/chartdb/overlays/prod/
   kubectl apply -k apps/productivity/excalidraw/overlays/prod/
   ```

> **⚠️ Important:** Configure secrets and SSL certificates before deploying applications. See [docs/](docs/) for detailed setup.

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
│   ├── development/        # Dev tools (ChartDB, OpenGist)
│   ├── productivity/       # Collaboration (Excalidraw)
│   ├── media/             # Media services
│   └── monitoring/        # Monitoring apps
├── infrastructure/         # Platform components
│   ├── cert-manager/      # SSL certificates
│   ├── postgresql/        # Database cluster
│   ├── monitoring/        # Observability stack
│   ├── flux-system/       # GitOps controller
│   └── ...               # Other infrastructure
└── docs/                  # Documentation
```

## Applications

### Available Apps

- **ChartDB** - Database design tool
- **OpenGist** - Code snippet sharing
- **Excalidraw** - Collaborative whiteboarding

### Adding New Apps

1. Create app structure: `apps/<category>/<app>/base/`
2. Add Kubernetes manifests
3. Create dev/prod overlays
4. Configure ingress and secrets
5. Submit pull request

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
