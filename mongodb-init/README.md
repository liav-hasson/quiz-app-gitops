# MongoDB Initialization

This Helm chart deploys a Kubernetes Job that initializes MongoDB with quiz application data.

## Purpose

- Populates MongoDB with quiz categories, subjects, keywords, and style modifiers
- Runs automatically after MongoDB deployment (ArgoCD PostSync hook)
- Idempotent: Checks if data exists and skips initialization if already populated
- Completely decouples database initialization from backend application

## Data Structure

The job loads data from `files/db.json` and transforms it into MongoDB documents:

```javascript
{
  topic: "Containers",              // Main category
  subtopic: "Docker Commands",      // Subcategory
  keywords: ["docker run", "docker ps", ...],
  style_modifiers: ["command syntax", "practical examples", ...],
  created_at: ISODate("..."),
  updated_at: ISODate("...")
}
```

## Configuration

Key values in `values.yaml`:

```yaml
job:
  enabled: true                    # Enable/disable the job
  usePostSyncHook: true           # Run after MongoDB is ready
  backoffLimit: 3                 # Retry attempts on failure
  ttlSecondsAfterFinished: 86400  # Cleanup after 24 hours

mongodb:
  host: "mongodb.mongodb.svc.cluster.local"
  port: 27017
  database: "quizdb"
  auth:
    enabled: true
    existingSecret: "mongodb"     # MongoDB credentials secret
```

## How It Works

1. **ArgoCD deploys MongoDB** (sync-wave: 0)
2. **MongoDB becomes healthy** (healthcheck passes)
3. **Job runs** (PostSync hook, sync-wave: 1)
   - Connects to MongoDB
   - Checks if `quiz_data` collection has documents
   - If empty: loads data from db.json, transforms, and inserts
   - If populated: exits successfully without changes
4. **Backend deploys** (sync-wave: 2) and connects to initialized database

## Manual Execution

To manually run the job (e.g., after data updates):

```bash
# Delete the existing job
kubectl delete job -n mongodb -l app.kubernetes.io/instance=mongodb-init

# ArgoCD will automatically recreate it on next sync
argocd app sync mongodb-init
```

## Force Re-initialization

To completely reset the database:

```bash
# 1. Delete all data
kubectl exec -n mongodb statefulset/mongodb -- mongosh -u root -p <password> quizdb --eval "db.quiz_data.deleteMany({})"

# 2. Re-run the job
kubectl delete job -n mongodb -l app.kubernetes.io/instance=mongodb-init
argocd app sync mongodb-init
```

## Updating Data

1. Edit `files/db.json` with new content
2. Commit and push changes to git
3. Sync ArgoCD application: `argocd app sync mongodb-init`
4. Job will detect existing data and skip (unless you force re-init)

## Verification

Check job status:
```bash
# View job
kubectl get jobs -n mongodb

# View job logs
kubectl logs -n mongodb job/mongodb-init-<timestamp>

# Verify data
kubectl exec -n mongodb statefulset/mongodb -- mongosh -u root -p <password> quizdb --eval "db.quiz_data.countDocuments()"
```

Expected output: 72 documents (13 categories, 72 subjects)

## Files

- `files/init-mongodb.js` - MongoDB initialization script
- `files/db.json` - Quiz data (categories, subjects, keywords, style modifiers)
- `templates/configmap.yaml` - ConfigMaps for script and data
- `templates/job.yaml` - Kubernetes Job definition
- `templates/serviceaccount.yaml` - Service account for the job


## Troubleshooting

**Job fails to connect to MongoDB:**
- Check MongoDB is running: `kubectl get pods -n mongodb`
- Verify secret exists: `kubectl get secret -n mongodb mongodb`
- Check logs: `kubectl logs -n mongodb job/mongodb-init-<timestamp>`

**Data not appearing:**
- Check job completed successfully: `kubectl get jobs -n mongodb`
- View job logs for errors
- Verify MongoDB connection from backend

**Job runs but skips initialization:**
- Data already exists in database
- To force re-init, delete collection first (see "Force Re-initialization")
