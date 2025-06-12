# Hardware Requirements

## Minimum Requirements

### Single Node (Testing)
- **CPU**: 4 cores
- **RAM**: 8GB
- **Storage**: 100GB SSD
- **Network**: 1Gbps

### Production Homelab (Recommended)
- **Nodes**: 3-4 nodes
- **CPU**: 4 cores per node
- **RAM**: 16GB per node
- **Storage**: 256GB SSD per node
- **Network**: 1Gbps with dedicated VLAN

## Tested Hardware

### Mini PCs (Recommended)
- **Lenovo ThinkCentre M720q**: Intel i3, 24GB RAM, 512GB SSD
- **Intel NUC**: Various models with sufficient specs
- **HP EliteDesk Mini**: Compact and reliable

## Resource Planning

### Platform Services
| Service       | CPU Request | Memory Request | Storage |
| ------------- | ----------- | -------------- | ------- |
| cert-manager  | 100m        | 128Mi          | 1Gi     |
| ingress-nginx | 100m        | 90Mi           | -       |
| metallb       | 100m        | 100Mi          | -       |
| prometheus    | 200m        | 400Mi          | 10Gi    |
| grafana       | 100m        | 128Mi          | 2Gi     |

### Applications
| Service     | CPU Request | Memory Request | Storage |
| ----------- | ----------- | -------------- | ------- |
| Gitea       | -           | -              | -       |
| Vaultwarden | -           | -              | -       |

## Network Requirements

### IP Address Planning
- **MetalLB Pool**: Reserve 10-20 IP addresses
- **Ingress**: At least 1 public IP
- **Services**: Plan for future growth

### DNS Requirements
- Wildcard DNS or individual A records
- SSL certificate management
- Dynamic DNS (optional)

## Storage Considerations

### Storage Classes
- **Local Storage**: Fast, single-node applications
- **NFS**: Shared storage for multi-node access
- **Object Storage**: Media and backup storage

### Backup Strategy
- Regular etcd backups
- Application data backups
- Configuration backups
