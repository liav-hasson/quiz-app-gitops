# Prerequisites

This directory contains one-time Kubernetes resources that must exist before other applications can be deployed.

## Resources

- **argocd-targetgroupbinding.yaml** - Connects ArgoCD to ALB for HTTPS access
- **mongodb-ebs-volume.yaml** - PersistentVolume and PVC for existing EBS volume `vol-020cd08dcd3d4f91a`

## Deployment

These resources are automatically deployed by ArgoCD via the `prerequisites` Application with **sync wave -2** (before all other applications).

### Order of Deployment

```
-2: Prerequisites (this)
    ├── ArgoCD TargetGroupBinding
    └── MongoDB PV/PVC
    
-1: Infrastructure (External Secrets, ALB Controller)

 0: Platform Services (MongoDB, Jenkins)
 
 1: Applications (quiz-backend)
 
 2: Frontend (quiz-frontend)
```

## Manual Deployment (if needed)

If ArgoCD is not yet set up or you need to apply manually:

```bash
kubectl apply -f argocd-targetgroupbinding.yaml
kubectl apply -f mongodb-ebs-volume.yaml
```

## Notes

- **ArgoCD TGB**: Must be updated with the correct ALB Target Group ARN from Terraform outputs
- **MongoDB PV**: Must match the availability zone of your EKS nodes
- **Data Persistence**: MongoDB PV has `Retain` policy - data is never deleted
