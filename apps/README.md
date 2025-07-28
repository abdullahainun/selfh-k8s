# Self-Hosted Applications

This directory contains Kubernetes manifests for popular self-hosted applications, organized by category.

## Categories

### 📋 Productivity
Applications for task management, collaboration, and productivity.
- `excalidraw/` - Collaborative drawing tool

### 🎬 Media
Media servers, photo management, and entertainment applications.
- *Coming soon: jellyfin, photoprism, immich*

### 📊 Monitoring
Monitoring, metrics, and observability tools.
- *Coming soon: prometheus, grafana, uptime-kuma*

### 💻 Development
Development tools, CI/CD, and code hosting platforms.
- *Coming soon: gitea, code-server, jenkins*

## Usage

### Option 1: Copy and Customize
```bash
cp -r apps/productivity/excalidraw my-excalidraw
cd my-excalidraw
# Edit manifests as needed
kubectl apply -k overlays/dev
```

### Option 2: Remote Reference
```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - https://github.com/username/selfh-k8s//apps/productivity/excalidraw/base?ref=main

patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 2
    target:
      kind: Deployment
      name: excalidraw
```

### Option 3: Use Environment Overlays
```bash
# Development
kubectl apply -k apps/productivity/excalidraw/overlays/dev

# Production  
kubectl apply -k apps/productivity/excalidraw/overlays/prod
```

## Structure

Each application follows this structure:
```
app-name/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml      # (if needed)
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

## Contributing

To add a new self-hosted application:
1. Choose the appropriate category
2. Follow the existing structure
3. Include both base manifests and overlays
4. Update this README