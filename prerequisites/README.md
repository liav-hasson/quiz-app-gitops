# Prerequisites

Core Kubernetes resources that must exist before other applications deploy.

## Resources

### Storage
- **storageclass-ebs-gp3.yaml** - Default StorageClass for dynamic EBS provisioning
- **mongodb-ebs-volume.yaml** - PersistentVolume pointing to existing EBS volume (`vol-020cd08dcd3d4f91a`)
- **mongodb-pvc.yaml** - Pre-created PVC that binds to the MongoDB PV

### Networking
- **argocd-targetgroupbinding.yaml** - Connects ArgoCD to AWS ALB for HTTPS access
- **grafana-targetgroupbinding.yaml** - Connects Grafana to AWS ALB for HTTPS access

## Deployment Order

ArgoCD deploys these with **sync-wave: -2** (before all other apps):

```
-2: Prerequisites (PV, PVC, TargetGroupBindings)
 0: MongoDB (adopts pre-created PVC)
 1: Applications (quiz-backend, quiz-frontend)
```

## MongoDB Persistence Strategy

The MongoDB setup uses **pre-created PV/PVC** to bind an existing EBS volume:

1. **PV** (`mongodb-ebs-pv`) - Points to EBS volume with `storageClassName: ""`
2. **PVC** (`datadir-mongodb-0`) - Pre-created with `volumeName: mongodb-ebs-pv`
3. **StatefulSet** - Adopts existing PVC instead of creating new one

This ensures data survives cluster recreation.

## Fresh Deployment

When creating a fresh cluster:

```bash
# 1. Terraform creates EBS volume and outputs volume ID
# 2. Update mongodb-ebs-volume.yaml with the volume ID
# 3. ArgoCD automatically applies prerequisites first
# 4. MongoDB StatefulSet adopts the pre-created PVC
```

## Important Notes

- **MongoDB PV** uses `Retain` policy - volume is never deleted automatically
- **PVC must exist before StatefulSet** - sync-wave ensures correct order
- **Availability Zone** - PV nodeAffinity must match EKS node zone (`eu-north-1a`)
