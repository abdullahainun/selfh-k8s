# Architecture Overview

## System Architecture

This homelab Kubernetes setup follows a layered architecture approach:

### Layer 1: Platform Services
- **cert-manager**: Automatic TLS certificate provisioning
- **ingress-nginx**: HTTP/HTTPS traffic routing and SSL termination
- **metallb**: Load balancer for bare metal environments
- **monitoring**: Prometheus, Grafana, and AlertManager stack

### Layer 2: Storage
- **Local Persistent Volumes**: For single-node applications
- **NFS/Longhorn**: For shared storage requirements
- **MinIO**: S3-compatible object storage

### Layer 3: Applications
- **Productivity**: Gitea, Vaultwarden, Vikunja, Wiki.js
- **Media**: Immich, Calibre, FileBrowser
- **AI Tools**: Open WebUI, LiteLLM
- **Utilities**: Excalidraw, Linkding, Memos

## Network Architecture

```
Internet → Cloudflare → Router → MetalLB → Ingress-nginx → Services
```

## Storage Strategy

- **Platform services**: Local storage with backups
- **Stateful applications**: Persistent Volumes with backup strategies
- **Media files**: Large volume storage with redundancy

## Security Model

- Network policies for service isolation
- RBAC for service accounts
- Non-root containers
- Resource quotas and limits
- Regular security updates
