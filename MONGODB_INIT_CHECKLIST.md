# MongoDB Initialization Job - Implementation Checklist

**Date:** November 14, 2025  
**Goal:** Decouple database initialization from backend application using Kubernetes Job

---

## Phase 1: Prepare Init Job Files ✅ COMPLETE

### 1.1 Create init script
- [x] Create `gitops/mongodb-init/files/init-mongodb.js`
- [x] Add logic to check if data exists (skip if already initialized)
- [x] Use topic/subtopic field names (not category/subject)
- [x] Include style_modifiers field
- [x] Test script syntax is valid JavaScript

### 1.2 Copy db.json to gitops repo
- [x] Copy `backend/src/db/db.json` to `gitops/mongodb-init/files/db.json`
- [x] Verify db.json has correct structure (topic/subtopic)
- [x] Verify db.json includes style_modifiers field
- [x] Check file size (~49KB expected)

---

## Phase 2: Create Kubernetes Resources ✅ COMPLETE

### 2.1 Create ConfigMap for init files
- [x] Create `gitops/mongodb-init/templates/configmap.yaml`
- [x] Mount init-mongodb.js script
- [x] Mount db.json data file
- [x] Add proper labels and metadata

### 2.2 Create Kubernetes Job
- [x] Create `gitops/mongodb-init/templates/job.yaml`
- [x] Use mongo:7.0 image
- [x] Mount ConfigMap volumes
- [x] Configure MongoDB connection (host, auth)
- [x] Add ArgoCD sync hooks (PostSync)
- [x] Set proper restart policy (OnFailure)
- [x] Add job TTL for cleanup

### 2.3 Update Helm Chart
- [x] Add job configuration to `values.yaml`
- [x] Add enable/disable flag for job
- [x] Create Chart.yaml (version 1.0.0)
- [x] Add MongoDB secret reference
- [x] Create ArgoCD application manifest

---

## Phase 3: Update Backend ✅ COMPLETE

### 3.1 Make db.json optional in backend
- [x] Update `backend/ci/app-dockerfile/Dockerfile`
- [x] Exclude db.json from production image (49KB saved)
- [x] Keep dbcontroller.py (still needed for logic)
- [ ] Test backend builds successfully (needs rebuild)

### 3.2 Update backend initialization logic
- [x] Modify `backend/src/python/main.py`
- [x] Add `AUTO_MIGRATE_DB` environment variable (defaults to true)
- [x] Gracefully handle missing db.json when auto-migration disabled
- [x] Log appropriate messages for each scenario

### 3.3 Update backend deployment
- [x] Add environment variable: `AUTO_MIGRATE_DB=false` in K8s
- [x] Keep `AUTO_MIGRATE_DB=true` in docker-compose (local dev)
- [x] Update backend image tag to v1.0.22 in values.yaml
- [x] Changed pullPolicy to IfNotPresent (best practice for tagged images)
- [ ] Test backend connects to MongoDB successfully (after rebuild)

---

## Phase 4: Configure Sync Order ✅ COMPLETE

### 4.1 Set ArgoCD sync waves
- [x] MongoDB: sync-wave: 0 (first)
- [x] mongodb-init: sync-wave: 1 with PostSync hook
- [x] Backend: sync-wave: 2
- [x] Frontend: sync-wave: 3 (after backend)

### 4.2 Verify dependencies
- [x] Job depends on MongoDB being ready (PostSync hook)
- [x] Backend depends on job completion (via data check)
- [x] Frontend depends on backend being ready
- [ ] Verify in actual deployment (testing phase)

---

## Phase 5: Testing

### 5.1 Test with empty database (first deployment)
- [ ] Delete quiz_data collection (or use fresh EKS cluster)
- [ ] Apply/sync quiz-backend application
- [ ] Verify job runs successfully
- [ ] Check job logs: "Database Initialization Complete"
- [ ] Verify 72 documents inserted
- [ ] Test backend /api/categories returns data
- [ ] Verify job pod completes and gets cleaned up

### 5.2 Test idempotency (second deployment)
- [ ] Sync quiz-backend application again
- [ ] Verify job runs but skips initialization
- [ ] Check job logs: "Database already initialized"
- [ ] Verify document count unchanged (still 72)
- [ ] Verify backend still works
- [ ] Confirm no duplicate data

### 5.3 Test backend without auto-migration
- [ ] Verify backend doesn't try to migrate
- [ ] Check backend logs for correct startup messages
- [ ] Verify categories endpoint works
- [ ] Test question generation works

### 5.4 Test frontend connectivity
- [ ] Access frontend via ALB URL
- [ ] Verify categories dropdown populates
- [ ] Select category and verify subjects load
- [ ] Generate a question (requires OPENAI_API_KEY)

---

## Phase 6: Manual Update Testing (Optional - Future)

### 6.1 Test manual data updates
- [ ] Update db.json with new content
- [ ] Delete quiz_data collection manually
- [ ] Re-run init job
- [ ] Verify new data is loaded

### 6.2 Test MongoDB Compass connection
- [ ] Port-forward MongoDB to localhost
- [ ] Connect via MongoDB Compass
- [ ] Browse collections and documents
- [ ] Verify data structure is correct

---

## Phase 7: Documentation

### 7.1 Update deployment documentation
- [ ] Update `EKS_DEPLOYMENT_READINESS.md`
- [ ] Document init job approach
- [ ] Add troubleshooting section
- [ ] Document manual job run procedure

### 7.2 Create MongoDB management guide
- [ ] Document how to view data
- [ ] Document how to update data
- [ ] Document how to force re-initialization
- [ ] Add backup/restore procedures

### 7.3 Update README files
- [ ] Update gitops/quiz-backend/README.md
- [ ] Document the init job
- [ ] Add architecture diagram (optional)

---

## Phase 8: Cleanup

### 8.1 Clean up backend
- [ ] Remove unused db.json references in code comments
- [ ] Update backend README if needed
- [ ] Consider keeping db.json for local dev (optional)

### 8.2 Update mini-version
- [ ] Document that mini-version keeps db.json in image
- [ ] Explain difference between local and EKS approaches
- [ ] Update mini-version README

---

## Success Criteria

- [ ] ✅ Backend image doesn't include db.json
- [ ] ✅ MongoDB initializes automatically on first deployment
- [ ] ✅ Job is idempotent (safe to run multiple times)
- [ ] ✅ Backend connects and serves data successfully
- [ ] ✅ Frontend can retrieve and display categories
- [ ] ✅ Question generation works end-to-end
- [ ] ✅ Documentation is updated and clear

---

## Rollback Plan

If issues occur:
1. Keep backend as-is (with auto-migration)
2. Disable init job in values.yaml: `job.enabled: false`
3. Backend will auto-migrate from db.json (current behavior)
4. Re-enable job after fixing issues

---

## Current Status

**Last Updated:** November 14, 2025  
**Phases 1-4:** ✅ COMPLETE  
**Phase 5:** Ready for testing  
**Next Step:** Rebuild backend image with new Dockerfile, then test in EKS
