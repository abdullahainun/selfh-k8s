# Jenkins

Jenkins CI/CD server deployed using the official Jenkins Helm chart.

## Overview

This Jenkins deployment includes:
- Persistent storage for Jenkins home directory
- Kubernetes integration for dynamic agents
- Pipeline support via workflow-aggregator plugin
- Basic security configuration

## Deployment

### Prerequisites

- Kubernetes cluster with Flux CD installed
- `local-path` storage class available

### Deploy Jenkins

1. **Deploy to development environment:**
   ```bash
   kubectl apply -k infrastructure/jenkins/overlays/dev
   ```

2. **Deploy to production environment:**
   ```bash
   kubectl apply -k infrastructure/jenkins/overlays/prod
   ```

3. **Verify deployment:**
   ```bash
   kubectl get pods -n jenkins
   kubectl get svc -n jenkins
   ```

## Accessing Jenkins

### Port Forward (for testing)

```bash
kubectl port-forward svc/jenkins 8080:8080 -n jenkins
```

Then access Jenkins at: http://localhost:8080

### No Password Required

**Important**: This Jenkins deployment is configured with **unsecured mode** for simplicity:
- No initial admin password is required
- Direct access to Jenkins without authentication
- Suitable for development/testing environments only

### Service Details

- **Service Name**: `jenkins`
- **Namespace**: `jenkins`
- **Internal URL**: `jenkins.jenkins.svc.cluster.local:8080`
- **Port**: `8080`

## Configuration

### Installed Plugins

The deployment includes minimal plugins to avoid version conflicts:
- `kubernetes` - Kubernetes integration for dynamic agents
- `workflow-aggregator` - Pipeline support

### Storage

- **Persistent Volume**: 20Gi
- **Storage Class**: `local-path`
- **Mount Path**: `/var/jenkins_home`

### Resources

- **CPU Request**: 500m
- **Memory Request**: 1Gi
- **CPU Limit**: 2
- **Memory Limit**: 4Gi

## Post-Installation Setup

### 1. Access Jenkins UI

1. Access Jenkins UI via port-forward or ingress
2. **No password required** - Jenkins will load directly
3. Jenkins is ready to use immediately

### 2. Enable Security (Recommended for Production)

For production environments, enable proper security:

1. Go to **Manage Jenkins** > **Configure Global Security**
2. Under **Security Realm**, choose appropriate authentication:
   - **Jenkins' own user database** (for simple setup)
   - **LDAP** (for enterprise integration)
   - **SAML** (for SSO)
3. Under **Authorization**, choose:
   - **Logged-in users can do anything** (basic)
   - **Matrix-based security** (advanced)
4. **Enable** "Prevent Cross Site Request Forgery exploits"
5. Save configuration
6. Create admin user when prompted

### 3. Recommended Additional Plugins

Install these plugins via Jenkins UI (Manage Jenkins > Manage Plugins):

- **Git**: `git`
- **GitHub**: `github`
- **Blue Ocean**: `blueocean`
- **Docker Pipeline**: `docker-workflow`
- **Configuration as Code**: `configuration-as-code`

### 4. Configure Kubernetes Cloud

1. Go to Manage Jenkins > Configure System
2. Scroll to "Cloud" section
3. Kubernetes cloud should be auto-configured
4. Test connection to ensure it works

## Kubernetes Agent Configuration

Jenkins is configured to use Kubernetes pods as build agents:

- **Namespace**: `jenkins`
- **Service Account**: `jenkins`
- **Pod Template**: Default Kubernetes agent template

## Troubleshooting

### Pod Not Starting

Check pod logs:
```bash
kubectl logs jenkins-0 -c init -n jenkins
kubectl describe pod jenkins-0 -n jenkins
```

### Plugin Installation Issues

If you encounter plugin dependency conflicts:

1. Remove problematic plugins from `helm-release.yaml`
2. Let Jenkins auto-install dependencies
3. Install additional plugins via UI after initial setup

### Storage Issues

Check PVC status:
```bash
kubectl get pvc -n jenkins
kubectl describe pvc jenkins -n jenkins
```

### Service Account Permissions

Verify service account has proper permissions:
```bash
kubectl get sa jenkins -n jenkins
kubectl describe sa jenkins -n jenkins
```

## Customization

### Adding Plugins

Edit `infrastructure/jenkins/base/helm-release.yaml`:

```yaml
installPlugins:
  - kubernetes
  - workflow-aggregator
  - git  # Add new plugins here
```

### Resource Limits

Modify resources in `infrastructure/jenkins/base/helm-release.yaml`:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "2"
    memory: "4Gi"
```

### Environment-Specific Configuration

Use overlays for environment-specific settings:
- `overlays/dev/` - Development configuration
- `overlays/prod/` - Production configuration

## Security Notes

⚠️ **Important Security Warning**: This deployment uses **unsecured mode** by default:

- **No authentication required** - anyone can access Jenkins
- **No authorization controls** - all users have admin access
- **Suitable for development/testing only**

### Current Security Configuration

```yaml
securityRealm: |-
  <securityRealm class="hudson.security.SecurityRealm$None"/>
authorizationStrategy: |-
  <authorizationStrategy class="hudson.security.AuthorizationStrategy$Unsecured"/>
```

### For Production Use

**Must enable proper security**:
1. Configure authentication (user database, LDAP, SAML)
2. Set up authorization strategy (role-based access)
3. Enable CSRF protection
4. Use HTTPS/TLS
5. Regular security updates

## Maintenance

### Backup

Jenkins home directory (`/var/jenkins_home`) contains all configuration and build data. Ensure regular backups of the persistent volume.

### Updates

Update Jenkins version by modifying the chart version in `helm-release.yaml`:

```yaml
spec:
  chart:
    spec:
      version: "5.7.12"  # Update this version
```

## Support

For issues specific to this deployment, check:
1. Pod logs: `kubectl logs jenkins-0 -n jenkins`
2. Helm release status: `kubectl get helmrelease jenkins -n jenkins`
3. Service status: `kubectl get svc jenkins -n jenkins`