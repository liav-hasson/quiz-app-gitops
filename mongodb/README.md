# MongoDB Custom Helm Chart

Simple MongoDB 7.0 StatefulSet with pre-created PVC support.

## Why Custom Chart?

Bitnami's MongoDB chart doesn't support PVC selectors properly. StatefulSets with `volumeClaimTemplates` always create PVCs automatically, ignoring pre-created PVCs with selectors.

**This chart solves that** by:
- Using `volumes` instead of `volumeClaimTemplates` when `usePrecreatedPVC: true`
- Full control over PVC binding via labels/selectors
- Ensures specific EBS volumes are used (survives cluster teardown)

## Configuration

### Use Pre-Created PVC (Recommended for Production)

```yaml
persistence:
  usePrecreatedPVC: true
  pvcName: "datadir-mongodb-0"  # Must exist before deployment
  size: 20Gi
  storageClass: ""
  selector:  # Not used, but documented for PV binding
    matchLabels:
      type: mongodb-data
      component: mongodb
```

### Use Dynamic Provisioning (Development)

```yaml
persistence:
  usePrecreatedPVC: false
  size: 20Gi
  storageClass: "ebs-gp3"
```

## Deployment Flow

1. **Prerequisites deploy** (sync-wave: -50)
   - PV created with labels: `type: mongodb-data`, `component: mongodb`
   - PVC created with selector matching those labels
   - PVC binds to PV

2. **MongoDB deploys** (sync-wave: 10)
   - StatefulSet references pre-created PVC by name
   - No volumeClaimTemplates, so no automatic PVC creation
   - Pod mounts the existing PVC (bound to your EBS volume)

3. **Result**: MongoDB uses your specific EBS volume that survives cluster teardown

## Key Features

- ✅ Uses official `mongo:7.0` image
- ✅ Authentication via existing secret
- ✅ Liveness and readiness probes
- ✅ Headless service for StatefulSet
- ✅ Configurable resources
- ✅ No automatic PVC creation when using pre-created PVC
- ✅ Falls back to volumeClaimTemplates for dynamic provisioning

## Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.usePrecreatedPVC` | Use pre-created PVC instead of volumeClaimTemplates | `true` |
| `persistence.pvcName` | Name of pre-created PVC | `datadir-mongodb-0` |
| `persistence.size` | Storage size | `20Gi` |
| `auth.enabled` | Enable authentication | `true` |
| `auth.existingSecret` | Secret containing `mongodb-root-password` | `mongodb` |
| `resources.requests.cpu` | CPU request | `500m` |
| `resources.requests.memory` | Memory request | `1Gi` |
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `2Gi` |
