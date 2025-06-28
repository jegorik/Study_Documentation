# T06 - Storage Mount Issues

> **📋 Lab Category:** Troubleshooting  
> **⏱️ Estimated Time:** 20-25 minutes  
> **🎯 Difficulty:** Intermediate  
> **🔗 Exam Objectives:** Troubleshooting, Storage, Pod Lifecycle

---

## 🎬 Real-World Scenario

### Background Context

You're the Platform Engineer at **DataFlow Inc**, a data analytics company processing terabytes of customer data daily. The morning deployment of a critical data pipeline has failed - pods are stuck in `Pending` status with storage mount failures. The data science team is blocked from their quarterly reporting deadline, and management is demanding immediate resolution.

**Your Role:** Platform Engineer (Storage Specialist)  
**Situation:** Critical data pipeline failure due to storage mounting issues  
**Urgency:** $200K quarterly report deadline in 4 hours  

### The Challenge

Multiple pods cannot mount persistent volumes due to various storage configuration issues. You must diagnose and resolve storage mounting problems across different scenarios to restore the data pipeline.

---

## 🎯 Learning Objectives

By completing this lab, you will:

- [ ] **Primary Skill:** Diagnose and resolve storage mounting failures
- [ ] **Secondary Skills:** PV/PVC troubleshooting, storage class issues, node storage problems
- [ ] **Real-world Application:** Production storage troubleshooting workflows
- [ ] **Exam Relevance:** Storage troubleshooting (common CKA exam scenario)

---

## 🏗️ Lab Environment Setup

### Initial Cluster State

```bash
# BROKEN - Multiple storage mount failures
SYMPTOMS:
- Pods stuck in Pending status: "pod has unbound immediate PersistentVolumeClaims"
- Mount failures: "unable to attach or mount volumes"
- Node disk space issues affecting local storage
- StorageClass provisioning failures
- PV/PVC binding problems

AFFECTED WORKLOADS:
- data-processor pods (PVC binding failed)
- database pods (mount permission errors)
- backup-service (node storage exhaustion)
- analytics-app (StorageClass misconfiguration)
```

---

## 📋 Tasks Overview

| Task | Objective | Est. Time | Difficulty |
|------|-----------|-----------|------------|
| 1 | Storage Mount Failure Assessment | 4 min | Intermediate |
| 2 | PV/PVC Binding Resolution | 6 min | Intermediate |
| 3 | Node Storage Issues Recovery | 5 min | Advanced |
| 4 | StorageClass Configuration Fix | 3 min | Intermediate |
| 5 | Storage Health Validation | 2 min | Basic |

---

## 🔍 Task 1: Storage Mount Failure Assessment (4 minutes)

### Objective

Rapidly identify the root causes of storage mounting failures across different workloads.

### Your Mission

Diagnose storage mount issues and categorize failure types.

### Step-by-Step Instructions

#### 1.1 Pod Storage Status Analysis

```bash
# Check pods with storage issues
kubectl get pods -A | grep -E "(Pending|ContainerCreating)"

# Examine specific pod storage events
kubectl describe pod data-processor-0 -n data-pipeline
kubectl describe pod postgres-primary-0 -n database
kubectl describe pod backup-service-0 -n backup

# Check PVC status across namespaces
kubectl get pvc -A
kubectl get pv
```

#### 1.2 Storage Events Investigation

```bash
# Check cluster-wide storage events
kubectl get events -A --field-selector type=Warning | grep -i storage

# Examine specific storage-related events
kubectl get events -A --field-selector reason=FailedMount
kubectl get events -A --field-selector reason=FailedAttachVolume

# Check for provisioning failures
kubectl get events -A --field-selector reason=ProvisioningFailed
```

#### 1.3 Node Storage Capacity Assessment

```bash
# Check node storage capacity
kubectl top nodes
kubectl describe nodes | grep -A 5 -B 5 "Allocated resources"

# Check for node storage pressure
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="DiskPressure")].status}{"\n"}{end}'

# Examine node conditions
kubectl describe nodes | grep -A 10 "Conditions"
```

### Lab Environment Setup (Create Broken Scenarios)

#### Create Failing Storage Scenarios

```yaml
# broken-storage-scenarios.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: data-pipeline
---
apiVersion: v1
kind: Namespace
metadata:
  name: database
---
apiVersion: v1
kind: Namespace
metadata:
  name: backup
---
# Broken PVC - No matching PV
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-processor-pvc
  namespace: data-pipeline
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Gi  # Intentionally large to cause issues
  storageClassName: non-existent-storage-class
---
# Pod trying to use broken PVC
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: data-processor
  namespace: data-pipeline
spec:
  serviceName: data-processor
  replicas: 1
  selector:
    matchLabels:
      app: data-processor
  template:
    metadata:
      labels:
        app: data-processor
    spec:
      containers:
      - name: processor
        image: nginx:alpine
        volumeMounts:
        - name: data-storage
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: non-existent-storage-class
      resources:
        requests:
          storage: 500Gi
```

```bash
# Apply broken scenarios
kubectl apply -f broken-storage-scenarios.yaml

# Verify problems are created
kubectl get pods -n data-pipeline
kubectl get pvc -n data-pipeline
```

---

## 📦 Task 2: PV/PVC Binding Resolution (6 minutes)

### Objective

Resolve PersistentVolume and PersistentVolumeClaim binding failures.

### Your Mission

Fix PV/PVC binding issues and restore storage connectivity.

### Step-by-Step Instructions

#### 2.1 Create Missing StorageClass

```yaml
# fixed-storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-local-storage
provisioner: kubernetes.io/host-path
parameters:
  type: DirectoryOrCreate
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```

```bash
# Apply working storage class
kubectl apply -f fixed-storage-class.yaml
kubectl get storageclass
```

#### 2.2 Create Appropriately Sized PV

```yaml
# working-persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-processor-pv
spec:
  capacity:
    storage: 50Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-local-storage
  hostPath:
    path: /tmp/data-processor
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: database-pv
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-local-storage
  hostPath:
    path: /tmp/database
```

```bash
# Apply PVs
kubectl apply -f working-persistent-volume.yaml
kubectl get pv
```

#### 2.3 Fix PVC Configuration

```bash
# Delete broken PVC
kubectl delete pvc data-processor-pvc -n data-pipeline

# Create working PVC
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-processor-pvc-fixed
  namespace: data-pipeline
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: fast-local-storage
EOF

# Verify PVC binding
kubectl get pvc -n data-pipeline
kubectl describe pvc data-processor-pvc-fixed -n data-pipeline
```

#### 2.4 Update StatefulSet with Fixed Storage

```bash
# Delete broken StatefulSet
kubectl delete statefulset data-processor -n data-pipeline

# Create working StatefulSet
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: data-processor-fixed
  namespace: data-pipeline
spec:
  serviceName: data-processor-fixed
  replicas: 1
  selector:
    matchLabels:
      app: data-processor-fixed
  template:
    metadata:
      labels:
        app: data-processor-fixed
    spec:
      containers:
      - name: processor
        image: nginx:alpine
        volumeMounts:
        - name: data-storage
          mountPath: /data
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo 'Processing data...' > /data/status.log; sleep 30; done"]
  volumeClaimTemplates:
  - metadata:
      name: data-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-local-storage
      resources:
        requests:
          storage: 20Gi
EOF

# Monitor pod creation
kubectl get pods -n data-pipeline -w
```

---

## 🖥️ Task 3: Node Storage Issues Recovery (5 minutes)

### Objective

Resolve node-level storage problems affecting pod scheduling and mounting.

### Your Mission

Fix node storage capacity and permission issues.

### Step-by-Step Instructions

#### 3.1 Node Storage Cleanup

```bash
# Check current node storage usage
kubectl describe nodes | grep -A 5 "Allocated resources"

# Clean up node storage (simulate)
kubectl get pods -A | grep Evicted | awk '{print $1, $2}' | xargs -n 2 kubectl delete pod -n

# Check for failed pods consuming storage
kubectl get pods -A --field-selector=status.phase=Failed
kubectl delete pods -A --field-selector=status.phase=Failed

# Verify storage pressure relief
kubectl describe nodes | grep -A 2 -B 2 "DiskPressure"
```

#### 3.2 Fix Storage Permissions

```bash
# Create directories with proper permissions (simulate node access)
sudo mkdir -p /tmp/data-processor /tmp/database /tmp/backup-storage
sudo chmod 755 /tmp/data-processor /tmp/database /tmp/backup-storage

# Set proper ownership (simulate container user access)
sudo chown 1000:1000 /tmp/data-processor /tmp/database /tmp/backup-storage

# Verify permissions
ls -la /tmp/ | grep -E "(data-processor|database|backup-storage)"
```

#### 3.3 Create Database Pod with Fixed Storage

```yaml
# database-with-storage.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: database
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: fast-local-storage
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-primary
  namespace: database
spec:
  serviceName: postgres-primary
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: dataflow
        - name: POSTGRES_USER
          value: admin
        - name: POSTGRES_PASSWORD
          value: securepass123
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 1000m
            memory: 2Gi
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-local-storage
      resources:
        requests:
          storage: 20Gi
```

```bash
# Apply database configuration
kubectl apply -f database-with-storage.yaml

# Monitor database pod startup
kubectl get pods -n database -w
```

---

## ⚙️ Task 4: StorageClass Configuration Fix (3 minutes)

### Objective

Resolve StorageClass configuration issues and validate dynamic provisioning.

### Your Mission

Fix StorageClass parameters and test dynamic volume provisioning.

### Step-by-Step Instructions

#### 4.1 Create Backup Service with Dynamic Storage

```yaml
# backup-service-storage.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
  namespace: backup
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-local-storage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backup-service
  namespace: backup
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backup-service
  template:
    metadata:
      labels:
        app: backup-service
    spec:
      containers:
      - name: backup
        image: alpine:latest
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo 'Backup job running...' >> /backup/backup.log; sleep 60; done"]
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
```

```bash
# Apply backup service
kubectl apply -f backup-service-storage.yaml

# Verify backup service deployment
kubectl get pods -n backup
kubectl get pvc -n backup
```

#### 4.2 Test Storage Class Functionality

```bash
# Create test PVC to validate storage class
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-dynamic-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-local-storage
EOF

# Check dynamic provisioning
kubectl get pvc test-dynamic-pvc
kubectl describe pvc test-dynamic-pvc

# Cleanup test PVC
kubectl delete pvc test-dynamic-pvc
```

---

## ✅ Task 5: Storage Health Validation (2 minutes)

### Objective

Confirm all storage issues are resolved and implement monitoring.

### Your Mission

Validate storage health across all workloads and set up monitoring.

### Step-by-Step Instructions

#### 5.1 Comprehensive Storage Health Check

```bash
# Check all PVCs are bound
kubectl get pvc -A

# Verify all pods are running
kubectl get pods -A | grep -E "(data-pipeline|database|backup)"

# Test storage accessibility
kubectl exec -n data-pipeline data-processor-fixed-0 -- df -h /data
kubectl exec -n database postgres-primary-0 -- df -h /var/lib/postgresql/data
kubectl exec -n backup backup-service-$(kubectl get pods -n backup -o name | cut -d/ -f2) -- df -h /backup
```

#### 5.2 Storage Performance Validation

```bash
# Test write operations
kubectl exec -n data-pipeline data-processor-fixed-0 -- sh -c "echo 'Storage test' > /data/test.txt && cat /data/test.txt"
kubectl exec -n database postgres-primary-0 -- sh -c "echo 'DB storage test' > /var/lib/postgresql/data/pgdata/test.txt"
kubectl exec -n backup backup-service-$(kubectl get pods -n backup -o name | cut -d/ -f2) -- sh -c "echo 'Backup test' > /backup/test.txt"

# Verify storage metrics
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"
```

#### 5.3 Create Storage Monitoring Script

```bash
# Create storage monitoring script
cat << 'EOF' | tee /tmp/check-storage-health.sh
#!/bin/bash
echo "=== Kubernetes Storage Health Check ==="
echo "Date: $(date)"
echo ""
echo "=== PVC Status ==="
kubectl get pvc -A
echo ""
echo "=== PV Status ==="
kubectl get pv
echo ""
echo "=== Node Storage Capacity ==="
kubectl top nodes
echo ""
echo "=== Storage Events (Last 10) ==="
kubectl get events -A --field-selector type=Warning | grep -i storage | tail -10
EOF

chmod +x /tmp/check-storage-health.sh

# Test monitoring script
/tmp/check-storage-health.sh
```

---

## 🎯 Expected Outcomes

Upon completion, you should have:

- ✅ All PVCs successfully bound to PVs
- ✅ Pods running with mounted storage volumes
- ✅ Node storage capacity within healthy limits
- ✅ StorageClass functioning for dynamic provisioning
- ✅ Storage monitoring script deployed
- ✅ Understanding of storage troubleshooting workflow

---

## 🚨 Common Storage Issues & Quick Fixes

### Issue: Pod stuck in Pending due to PVC

```bash
# Check PVC status and events
kubectl describe pvc <pvc-name> -n <namespace>
kubectl get events -n <namespace> --field-selector involvedObject.name=<pvc-name>
```

### Issue: Mount permission denied

```bash
# Fix storage permissions on node
sudo chown -R 1000:1000 /path/to/volume
sudo chmod -R 755 /path/to/volume
```

### Issue: StorageClass not found

```bash
# List available storage classes
kubectl get storageclass
# Create missing storage class or fix PVC storageClassName
```

### Issue: Node storage pressure

```bash
# Clean up unused resources
kubectl get pods -A --field-selector=status.phase=Failed -o name | xargs kubectl delete
docker system prune -f  # On nodes
```

---

## 📚 Knowledge Check

### Critical Understanding Questions

1. **PV vs PVC:** What's the difference and how do they bind?
2. **Storage Classes:** How does dynamic provisioning work?
3. **Mount Issues:** What are common causes of mount failures?

### Advanced Scenarios

- How would you handle storage expansion for running pods?
- What's the impact of changing storage class parameters?
- How do you troubleshoot CSI driver issues?

---

## 🔄 Production Best Practices

### Storage Monitoring Setup

```bash
# Set up regular storage health checks
echo "0 */6 * * * /usr/local/bin/check-storage-health.sh | mail -s 'K8s Storage Health' admin@company.com" | crontab -

# Monitor storage usage trends
kubectl top nodes --no-headers | awk '{print $1, $5}' > /tmp/storage-usage-$(date +%Y%m%d).log
```

---

*💡 "Storage issues are the #2 cause of pod failures in Kubernetes. Master these troubleshooting skills for production reliability!" - CKA Success Tip*
