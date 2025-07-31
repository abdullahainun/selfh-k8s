# 🚀 GitOps Cluster Configuration

This directory contains **optional** FluxCD configuration for automated GitOps deployment. 

## 📋 Overview

The files in this directory enable **full GitOps automation** where FluxCD automatically deploys applications from this Git repository. This is an alternative to the default manual `kubectl apply` approach.

## 🎯 Usage

**For Manual Deployment (Default):**
- Ignore this directory completely
- Follow the main README.md installation steps
- Use `kubectl apply -k` commands as documented

**For GitOps Deployment (Advanced):**
- Follow the [Advanced GitOps Guide](../docs/advanced-gitops.md)
- Use `flux bootstrap` to set up automated deployment
- FluxCD will automatically apply changes from Git

## 📁 Directory Structure

```
cluster/
├── base/              # Base Kustomization that references everything
├── infrastructure/    # Infrastructure deployment order and dependencies  
├── apps/             # Application deployment configurations
└── README.md         # This file
```

## ⚠️ Important Notes

1. **This is optional** - The repository works perfectly without GitOps
2. **Advanced users only** - Requires FluxCD knowledge
3. **Test thoroughly** - GitOps changes affect entire cluster
4. **Backup plan** - Know how to return to manual deployment

## 📚 Documentation

- **Getting Started**: [Advanced GitOps Guide](../docs/advanced-gitops.md)
- **Main README**: [Manual Deployment](../README.md) 
- **Secrets Setup**: [Secrets Guide](../SECRETS_SETUP.md)

---

**💡 Recommendation**: Start with manual deployment to understand the applications, then optionally migrate to GitOps for automation.