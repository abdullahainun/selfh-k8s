# Contributing Guide

Thank you for your interest in contributing to homelab-k8s-services!

## How to Contribute

### Reporting Issues
- Use GitHub Issues for bug reports and feature requests
- Provide detailed information about your environment
- Include steps to reproduce issues

### Adding New Services
1. Fork the repository
2. Create a feature branch
3. Add service following the established structure
4. Test thoroughly
5. Submit a pull request

## Service Standards

### Directory Structure
```
apps/category/service-name/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── configmap.yaml
├── overlays/
│   └── homelab/
│       ├── kustomization.yaml
│       └── patches/
├── secret.yaml.template
└── README.md
```

### Required Files

#### README.md
- Service description
- Installation instructions
- Configuration options
- Troubleshooting tips

#### kustomization.yaml
- Proper resource references
- Common labels
- Namespace specification

#### Manifests
- Security best practices
- Resource limits/requests
- Health checks
- Non-root containers

### Code Standards

#### Kubernetes Manifests
- Use latest stable API versions
- Include resource limits and requests
- Implement proper health checks
- Follow security best practices

#### Documentation
- Clear and concise
- Include examples
- Provide troubleshooting guidance

### Testing Requirements

#### Before Submitting
1. Test deployment on clean cluster
2. Verify all resources are created
3. Test basic functionality
4. Check resource usage
5. Validate security settings

#### Automated Tests
- Manifests must pass `kubectl apply --dry-run`
- Kustomize builds must succeed
- All resources must have proper labels

## Review Process

### Pull Request Requirements
- Clear description of changes
- Reference related issues
- Include testing evidence
- Update documentation

### Review Criteria
- Code quality and standards
- Security best practices
- Documentation completeness
- Testing coverage

## Development Setup

### Prerequisites
- Kubernetes cluster for testing
- kubectl configured
- kustomize installed

### Local Testing
```bash
# Validate manifests
kubectl apply --dry-run=client -k apps/category/service/overlays/homelab

# Test kustomize build
kustomize build apps/category/service/overlays/homelab

# Deploy for testing
kubectl apply -k apps/category/service/overlays/homelab
```

## Recognition

Contributors will be recognized in:
- README.md contributors section
- Release notes
- Project documentation

Thank you for helping make this project better!
