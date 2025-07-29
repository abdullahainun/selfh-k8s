<div align="center">
  <h3 align="center">Self-Hosted K8s Services</h3>
  <p align="center">
    A simple Kubernetes homelab running self-hosted applications with GitOps deployment
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

- [About The Project](#about-the-project)
- [Built With](#built-with)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## About The Project

A Kubernetes homelab for running self-hosted applications using GitOps principles. This repository contains manifests for various services organized by category, making it easy to deploy and manage containerized applications.

**Key Features:**
- 🚀 GitOps deployment with Kustomize
- 📦 Pre-configured service templates
- 🏗️ Organized by application categories
- 🔧 Development and production overlays

### Built With

- ![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
- ![Kustomize](https://img.shields.io/badge/kustomize-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

## Getting Started

### Prerequisites

- Kubernetes cluster (1.20+)
- kubectl configured
- Kustomize (optional, included in kubectl)

### Installation

1. Clone the repository
   ```sh
   git clone https://github.com/abdullahainun/selfh-k8s.git
   cd selfh-k8s
   ```

2. Deploy a service
   ```sh
   kubectl apply -k apps/development/chartdb/overlays/prod
   ```

## Usage

Browse available services in the `apps/` directory organized by category:

- **Development** - Dev tools and utilities (ChartDB, OpenGist)
- **Productivity** - Collaboration tools (Excalidraw)
- **Media** - Media management (coming soon)
- **Monitoring** - System monitoring (coming soon)

### Directory Structure

```
selfh-k8s/
├── apps/                   # Application services
│   ├── development/        # Dev tools and utilities
│   ├── productivity/       # Collaboration tools
│   ├── media/             # Media management
│   └── monitoring/        # System monitoring
├── infrastructure/         # Platform components
│   ├── ingress-nginx/     # Traffic routing
│   ├── cert-manager/      # TLS certificates
│   └── storage/           # Storage classes
└── examples/              # Usage examples
    └── remote-base-example/ # Example using remote base
```

### Adding New Services

1. Create directory: `apps/<category>/<service>/base/`
2. Add manifests: `deployment.yaml`, `service.yaml`, `kustomization.yaml`
3. Create overlays for dev/prod environments
4. Submit PR for review

## Roadmap

- [ ] Add media services
- [ ] Implement monitoring stack
- [ ] Add automated testing
- [ ] Create documentation site

See [open issues](https://github.com/abdullahainun/selfh-k8s/issues) for proposed features and known issues.

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
