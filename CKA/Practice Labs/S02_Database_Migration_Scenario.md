# Lab S02: Database Migration Scenario

> **📋 Lab Category:** Storage  
> **⏱️ Estimated Time:** 30-35 minutes  
> **🎯 Difficulty:** Intermediate  
> **🔗 Exam Objectives:** Understand storage classes, persistent volumes, Use persistent volume claims, Configure applications with persistent storage, Understand StatefulSets

---

## 🎬 Real-World Scenario

### Background Context
You're a Senior Platform Engineer at **DataFlow Solutions**, a fast-growing SaaS company that provides analytics dashboards for e-commerce businesses. The company is migrating their monolithic PostgreSQL database to a cloud-native, highly available database cluster running on Kubernetes. The migration must be completed with zero data loss and minimal downtime during business hours.

**Your Role:** Lead Database Migration Engineer responsible for Kubernetes storage infrastructure  
**Situation:** Legacy database needs migration to StatefulSet with persistent storage and backup strategy  
**Urgency:** **HIGH PRIORITY** - Major client onboarding scheduled in 3 days, system must be resilient and performant

### The Challenge
The current database setup is a single VM-based PostgreSQL instance that's becoming a bottleneck. You need to migrate to a StatefulSet-based PostgreSQL cluster with:
- **Persistent storage** that survives pod restarts and node failures
- **Backup and restore** capabilities for data protection
- **Storage classes** optimized for database workloads
- **Rolling updates** without data corruption
- **Disaster recovery** procedures tested and documented

**Critical Business Requirements:**
- 💾 **Zero Data Loss:** All customer analytics data must be preserved during migration
- ⚡ **Performance:** Database I/O must meet SLA requirements (sub-100ms query response)
- 🔄 **High Availability:** Database cluster must survive single node failures
- 📊 **Monitoring:** Storage metrics and database health monitoring required

---

## 🎯 Learning Objectives

By completing this lab, you will:
- [ ] **Primary Skill:** Master StatefulSets with persistent storage for stateful applications
- [ ] **Secondary Skills:** Configure storage classes, manage PVCs, implement backup strategies
- [ ] **Real-world Application:** Execute database migration with storage optimization and disaster recovery
- [ ] **Exam Relevance:** Core CKA storage concepts - StatefulSets, PVs, PVCs, storage classes

---

## 🔧 Prerequisites

### Knowledge Requirements
- [ ] Understanding of Kubernetes storage concepts (PV, PVC, StorageClass)
- [ ] Basic knowledge of StatefulSets vs Deployments
- [ ] Familiarity with PostgreSQL and database operations
- [ ] Experience with YAML manifests and kubectl commands

### Environment Setup
```bash
# Cluster requirements
- Kubernetes cluster with dynamic storage provisioning
- Default storage class configured
- At least 3 worker nodes for high availability
- kubectl configured with cluster-admin permissions

# Verification commands
kubectl get storageclass
kubectl get nodes
kubectl get pods -n kube-system | grep storage
```

---

## 📚 Quick Reference

### Key Commands for This Lab
```bash
kubectl get statefulsets                                      # List StatefulSets
kubectl describe statefulset <name>                          # StatefulSet details
kubectl get pvc                                              # List persistent volume claims
kubectl get pv                                               # List persistent volumes
kubectl scale statefulset <name> --replicas=<count>         # Scale StatefulSet
kubectl delete pod <statefulset-pod-name>                   # Test pod recreation
```

### Important Concepts
- **StatefulSet:** Manages deployment/scaling of stateful applications with stable identities
- **Persistent Volume (PV):** Storage resource in cluster, independent of pod lifecycle
- **Persistent Volume Claim (PVC):** Request for storage by a pod
- **Storage Class:** Defines storage provisioner and parameters for dynamic provisioning
- **Headless Service:** Service without cluster IP for direct pod-to-pod communication

---

## 🚀 Lab Tasks

### Task 1: Create Storage Infrastructure
**Objective:** Set up storage classes and prepare persistent volume infrastructure for database workloads

**Your Mission:**
Create a high-performance storage class optimized for database workloads. Configure storage parameters for IOPS, throughput, and replication. Verify that dynamic provisioning is working correctly.

**Expected Outcome:**
Storage class configured with database-optimized parameters, ready for StatefulSet volume claims.

**Hints:**
- 💡 Consider using SSD-based storage for database workloads
- 💡 Configure appropriate volume binding mode and reclaim policy
- 💡 Test storage class with a simple PVC before using with StatefulSet

---

### Task 2: Deploy PostgreSQL StatefulSet with Persistent Storage
**Objective:** Create a PostgreSQL StatefulSet with persistent volume claims and proper configuration

**Your Mission:**
Deploy a PostgreSQL StatefulSet with 3 replicas, each with its own persistent volume. Configure proper resource limits, environment variables, and health checks. Set up a headless service for internal communication.

**Expected Outcome:**
PostgreSQL StatefulSet running with 3 pods, each with persistent storage and stable network identities.

**Hints:**
- 💡 Use volumeClaimTemplates in StatefulSet for automatic PVC creation
- 💡 Configure PostgreSQL for replication (primary/replica setup)
- 💡 Set appropriate CPU and memory limits for database workloads

---

### Task 3: Implement Data Migration and Backup Strategy
**Objective:** Migrate existing data to the new StatefulSet and implement backup procedures

**Your Mission:**
Create a data migration job that populates the new PostgreSQL cluster with sample data. Implement a backup strategy using Kubernetes jobs and persistent volumes. Test the backup and restore procedures.

**Expected Outcome:**
Data successfully migrated to StatefulSet, automated backup system operational, restore procedure verified.

**Hints:**
- 💡 Use Kubernetes Jobs for one-time migration tasks
- 💡 Create separate PVC for backup storage
- 💡 Test backup integrity by restoring to a test environment

---

### Task 4: Test High Availability and Storage Resilience
**Objective:** Verify that the database cluster survives pod failures, node failures, and storage scenarios

**Your Mission:**
Test the resilience of your StatefulSet by simulating various failure scenarios: pod deletion, node drain, and storage detachment. Verify that data persists across all scenarios and that the cluster recovers properly.

**Expected Outcome:**
Database cluster demonstrates high availability with persistent storage surviving all failure scenarios.

**Hints:**
- 💡 Delete pods and verify automatic recreation with same storage
- 💡 Drain a node and verify pod rescheduling maintains data
- 💡 Monitor PVC binding and storage attachment during failures

---

### Task 5: Configure Storage Monitoring and Alerting
**Objective:** Implement monitoring for storage metrics, database performance, and capacity planning

**Your Mission:**
Set up monitoring for storage utilization, IOPS, latency, and database-specific metrics. Create alerts for storage capacity and performance thresholds. Document storage scaling procedures.

**Expected Outcome:**
Comprehensive storage and database monitoring in place with actionable alerts and scaling procedures.

**Hints:**
- 💡 Monitor PVC usage, storage class performance metrics
- 💡 Set up PostgreSQL-specific monitoring (connections, query performance)
- 💡 Create runbooks for storage expansion and performance tuning

---

### Task 6: Implement Rolling Updates and Disaster Recovery
**Objective:** Test StatefulSet rolling updates and implement disaster recovery procedures

**Your Mission:**
Perform a rolling update of the PostgreSQL StatefulSet (image version upgrade) while maintaining data integrity. Create and test a disaster recovery plan including backup restoration and cluster rebuilding procedures.

**Expected Outcome:**
Successful rolling update with zero data loss, tested disaster recovery procedures documented and verified.

**Hints:**
- 💡 Use partition-based rolling updates for controlled deployment
- 💡 Test backup restoration to a completely new cluster
- 💡 Document RTO/RPO requirements and validate against procedures

---

### Task 7: Storage Performance Optimization and Scaling
**Objective:** Optimize storage performance and demonstrate horizontal and vertical scaling capabilities

**Your Mission:**
Benchmark storage performance under load, optimize configuration for your workload, and demonstrate both scaling up (more storage per pod) and scaling out (more replicas) scenarios.

**Expected Outcome:**
Storage performance optimized for database workloads, scaling procedures tested and documented for production use.

**Hints:**
- 💡 Use tools like pgbench for database load testing
- 💡 Test PVC expansion (if supported by storage class)
- 💡 Document performance baselines and scaling triggers

---

## ⏰ Time Management

**Exam Pace Training:**
- [ ] Task 1: 4 minutes (Storage infrastructure)
- [ ] Task 2: 8 minutes (StatefulSet deployment)
- [ ] Task 3: 6 minutes (Migration and backup)
- [ ] Task 4: 5 minutes (HA testing)
- [ ] Task 5: 4 minutes (Monitoring setup)
- [ ] Task 6: 5 minutes (Rolling updates)
- [ ] Task 7: 3 minutes (Performance optimization)
- [ ] **Total Target:** 35 minutes

*⚠️ If taking longer than target time, focus on core StatefulSet and storage concepts, then review solution*

---

## 🔧 Step-by-Step Solution

<details>
<summary><strong>📖 Click to reveal detailed solution (try solving first!)</strong></summary>

### Step 1: Create Storage Infrastructure
**Command/Action:**
```bash
# Create namespace for database workloads
kubectl create namespace dataflow-db

# Create optimized storage class for database workloads
kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd-db
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: kubernetes.io/aws-ebs  # Adjust for your cloud provider
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  fsType: ext4
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
allowVolumeExpansion: true
EOF

# Test storage class with a sample PVC
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storage-test
  namespace: dataflow-db
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd-db
  resources:
    requests:
      storage: 1Gi
EOF

# Verify storage class and test PVC
kubectl get storageclass
kubectl get pvc -n dataflow-db
kubectl describe pvc storage-test -n dataflow-db
```

**Explanation:**
We create a storage class optimized for database workloads with high IOPS and throughput. The `WaitForFirstConsumer` binding mode ensures volumes are created in the same zone as the pod. `Retain` reclaim policy prevents accidental data loss.

**Expected Output:**
```
NAME          PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION
fast-ssd-db   kubernetes.io/aws-ebs   Retain          WaitForFirstConsumer   true

NAME           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
storage-test   Pending                                      fast-ssd-db    30s
```

---

### Step 2: Deploy PostgreSQL StatefulSet
**Command/Action:**
```bash
# Create headless service for StatefulSet
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
  namespace: dataflow-db
  labels:
    app: postgres-cluster
spec:
  clusterIP: None
  selector:
    app: postgres-cluster
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP
    name: postgres
EOF

# Create ConfigMap for PostgreSQL configuration
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  namespace: dataflow-db
data:
  postgresql.conf: |
    # PostgreSQL configuration optimized for Kubernetes
    max_connections = 200
    shared_buffers = 256MB
    effective_cache_size = 1GB
    work_mem = 4MB
    maintenance_work_mem = 64MB
    wal_buffers = 16MB
    checkpoint_completion_target = 0.9
    random_page_cost = 1.1
    effective_io_concurrency = 200
    
    # Replication settings
    wal_level = replica
    max_wal_senders = 3
    hot_standby = on
    hot_standby_feedback = on
    
    # Logging
    log_destination = 'stderr'
    logging_collector = off
    log_min_messages = warning
    log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
EOF

# Deploy PostgreSQL StatefulSet
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-cluster
  namespace: dataflow-db
  labels:
    app: postgres-cluster
spec:
  serviceName: postgres-headless
  replicas: 3
  selector:
    matchLabels:
      app: postgres-cluster
  template:
    metadata:
      labels:
        app: postgres-cluster
    spec:
      containers:
      - name: postgres
        image: postgres:14
        env:
        - name: POSTGRES_DB
          value: dataflow
        - name: POSTGRES_USER
          value: dataflow_user
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - containerPort: 5432
          name: postgres
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        - name: postgres-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
        livenessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - pg_isready -U dataflow_user -d dataflow -h 127.0.0.1 -p 5432
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - pg_isready -U dataflow_user -d dataflow -h 127.0.0.1 -p 5432
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
      volumes:
      - name: postgres-config
        configMap:
          name: postgres-config
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd-db
      resources:
        requests:
          storage: 20Gi
EOF

# Create secret for PostgreSQL password
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: dataflow-db
type: Opaque
data:
  password: ZGF0YWZsb3dfcGFzcw==  # base64 encoded "dataflow_pass"
EOF

# Verify StatefulSet deployment
kubectl get statefulset -n dataflow-db
kubectl get pods -n dataflow-db
kubectl get pvc -n dataflow-db
```

**Explanation:**
The StatefulSet creates PostgreSQL pods with stable identities (postgres-cluster-0, postgres-cluster-1, etc.). Each pod gets its own PVC created from the volumeClaimTemplate. The headless service provides stable DNS names for each pod.

**Expected Output:**
```
NAME               READY   AGE
postgres-cluster   3/3     2m

NAME                 READY   STATUS    RESTARTS   AGE
postgres-cluster-0   1/1     Running   0          2m
postgres-cluster-1   1/1     Running   0          90s
postgres-cluster-2   1/1     Running   0          60s

NAME                                      STATUS   VOLUME                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-storage-postgres-cluster-0      Bound    pvc-abc123                 20Gi       RWO            fast-ssd-db    2m
postgres-storage-postgres-cluster-1      Bound    pvc-def456                 20Gi       RWO            fast-ssd-db    90s
postgres-storage-postgres-cluster-2      Bound    pvc-ghi789                 20Gi       RWO            fast-ssd-db    60s
```

---

### Step 3: Implement Data Migration and Backup Strategy
**Command/Action:**
```bash
# Create sample data migration job
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: data-migrator
        image: postgres:14
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        command:
        - /bin/bash
        - -c
        - |
          # Wait for PostgreSQL to be ready
          until pg_isready -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local -p 5432 -U dataflow_user; do
            echo "Waiting for PostgreSQL..."
            sleep 5
          done
          
          # Create sample tables and data
          psql -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local -U dataflow_user -d dataflow << 'EOSQL'
          CREATE TABLE IF NOT EXISTS customers (
            id SERIAL PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            email VARCHAR(150) UNIQUE NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
          );
          
          CREATE TABLE IF NOT EXISTS analytics_events (
            id SERIAL PRIMARY KEY,
            customer_id INTEGER REFERENCES customers(id),
            event_type VARCHAR(50) NOT NULL,
            event_data JSONB,
            timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
          );
          
          -- Insert sample data
          INSERT INTO customers (name, email) VALUES 
          ('TechCorp Solutions', 'admin@techcorp.com'),
          ('DataMining Inc', 'contact@datamining.com'),
          ('Analytics Pro', 'info@analyticspro.com'),
          ('CloudFirst Ltd', 'team@cloudfirst.com'),
          ('InnovateSoft', 'hello@innovatesoft.com');
          
          INSERT INTO analytics_events (customer_id, event_type, event_data) VALUES 
          (1, 'page_view', '{"page": "/dashboard", "duration": 45}'),
          (1, 'button_click', '{"button": "export_data", "section": "reports"}'),
          (2, 'login', '{"method": "sso", "ip": "192.168.1.100"}'),
          (3, 'report_generated', '{"type": "monthly", "records": 15000}'),
          (4, 'api_call', '{"endpoint": "/api/v1/metrics", "response_time": 120}');
          
          \echo 'Data migration completed successfully!'
          EOSQL
EOF

# Create backup storage PVC
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-storage
  namespace: dataflow-db
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd-db
  resources:
    requests:
      storage: 50Gi
EOF

# Create backup CronJob
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgres-backup
  namespace: dataflow-db
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: postgres-backup
            image: postgres:14
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
            command:
            - /bin/bash
            - -c
            - |
              BACKUP_FILE="/backup/postgres-backup-$(date +%Y%m%d-%H%M%S).sql"
              echo "Starting backup to $BACKUP_FILE"
              
              pg_dump -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local \
                      -U dataflow_user \
                      -d dataflow \
                      --verbose \
                      --format=custom \
                      --compress=9 \
                      --file="$BACKUP_FILE"
              
              if [ $? -eq 0 ]; then
                echo "Backup completed successfully: $BACKUP_FILE"
                # Keep only last 7 backups
                ls -t /backup/postgres-backup-*.sql | tail -n +8 | xargs -r rm
              else
                echo "Backup failed!"
                exit 1
              fi
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-storage
EOF

# Wait for migration job to complete
kubectl wait --for=condition=complete job/data-migration -n dataflow-db --timeout=300s

# Verify data migration
kubectl exec -n dataflow-db postgres-cluster-0 -- psql -U dataflow_user -d dataflow -c "SELECT COUNT(*) FROM customers;"
kubectl exec -n dataflow-db postgres-cluster-0 -- psql -U dataflow_user -d dataflow -c "SELECT COUNT(*) FROM analytics_events;"

# Test backup manually (without waiting for cron)
kubectl create job --from=cronjob/postgres-backup postgres-backup-manual -n dataflow-db
```

**Explanation:**
The migration job populates the database with sample data representing a real analytics platform. The backup CronJob runs daily and creates compressed PostgreSQL dumps, maintaining only the last 7 backups for space efficiency.

**Expected Output:**
```
job.batch/data-migration condition met

 count 
-------
     5

 count 
-------
     5

job.batch/postgres-backup-manual created
```

---

### Step 4: Test High Availability and Storage Resilience
**Command/Action:**
```bash
# Test 1: Pod deletion and recreation
echo "=== Test 1: Pod Deletion Resilience ==="
kubectl delete pod postgres-cluster-1 -n dataflow-db

# Wait for pod to be recreated
kubectl wait --for=condition=ready pod/postgres-cluster-1 -n dataflow-db --timeout=180s

# Verify data persists after pod recreation
kubectl exec -n dataflow-db postgres-cluster-1 -- psql -U dataflow_user -d dataflow -c "SELECT name FROM customers LIMIT 3;"

# Test 2: Simulate node drain (if multiple nodes available)
echo "=== Test 2: Node Drain Simulation ==="
NODE_NAME=$(kubectl get pods postgres-cluster-2 -n dataflow-db -o jsonpath='{.spec.nodeName}')
echo "Pod postgres-cluster-2 is on node: $NODE_NAME"

# Cordon the node to prevent new scheduling
kubectl cordon $NODE_NAME

# Delete the pod to force rescheduling
kubectl delete pod postgres-cluster-2 -n dataflow-db

# Wait for pod to be rescheduled and ready
kubectl wait --for=condition=ready pod/postgres-cluster-2 -n dataflow-db --timeout=300s

# Verify pod moved to different node
NEW_NODE=$(kubectl get pods postgres-cluster-2 -n dataflow-db -o jsonpath='{.spec.nodeName}')
echo "Pod postgres-cluster-2 moved to node: $NEW_NODE"

# Uncordon the node
kubectl uncordon $NODE_NAME

# Verify data integrity after rescheduling
kubectl exec -n dataflow-db postgres-cluster-2 -- psql -U dataflow_user -d dataflow -c "SELECT COUNT(*) FROM analytics_events;"

# Test 3: PVC attachment verification
echo "=== Test 3: Storage Attachment Verification ==="
kubectl get pvc -n dataflow-db
kubectl describe pvc postgres-storage-postgres-cluster-0 -n dataflow-db | grep -A5 "Events"

# Test 4: StatefulSet scaling
echo "=== Test 4: StatefulSet Scaling ==="
kubectl scale statefulset postgres-cluster --replicas=4 -n dataflow-db
kubectl wait --for=condition=ready pod/postgres-cluster-3 -n dataflow-db --timeout=300s

# Verify new replica gets its own storage
kubectl get pvc -n dataflow-db | grep postgres-cluster-3

# Scale back down
kubectl scale statefulset postgres-cluster --replicas=3 -n dataflow-db
kubectl wait --for=delete pod/postgres-cluster-3 -n dataflow-db --timeout=180s

# Verify PVC persists even after pod deletion
kubectl get pvc -n dataflow-db | grep postgres-cluster-3
```

**Explanation:**
These tests verify that StatefulSets maintain data persistence across pod failures, node failures, and scaling operations. PVCs remain bound to their respective pods and storage persists independent of pod lifecycle.

**Expected Output:**
```
=== Test 1: Pod Deletion Resilience ===
pod "postgres-cluster-1" deleted
pod/postgres-cluster-1 condition met

       name        
-------------------
 TechCorp Solutions
 DataMining Inc
 Analytics Pro

=== Test 4: StatefulSet Scaling ===
statefulset.apps/postgres-cluster scaled
pod/postgres-cluster-3 condition met

postgres-storage-postgres-cluster-3   Bound    pvc-xyz789   20Gi   RWO   fast-ssd-db   30s

postgres-storage-postgres-cluster-3   Bound    pvc-xyz789   20Gi   RWO   fast-ssd-db   2m
```

---

### Step 5: Configure Storage Monitoring
**Command/Action:**
```bash
# Create ServiceMonitor for Prometheus (if using Prometheus Operator)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: postgres-metrics
  namespace: dataflow-db
  labels:
    app: postgres-cluster
spec:
  selector:
    app: postgres-cluster
  ports:
  - port: 9187
    targetPort: 9187
    name: metrics
EOF

# Deploy PostgreSQL Exporter as sidecar (updated StatefulSet)
kubectl patch statefulset postgres-cluster -n dataflow-db --type='merge' -p='
{
  "spec": {
    "template": {
      "spec": {
        "containers": [
          {
            "name": "postgres-exporter",
            "image": "prometheuscommunity/postgres-exporter:latest",
            "ports": [
              {
                "containerPort": 9187,
                "name": "metrics"
              }
            ],
            "env": [
              {
                "name": "DATA_SOURCE_NAME",
                "value": "postgresql://dataflow_user:dataflow_pass@localhost:5432/dataflow?sslmode=disable"
              }
            ],
            "resources": {
              "requests": {
                "memory": "64Mi",
                "cpu": "50m"
              },
              "limits": {
                "memory": "128Mi",
                "cpu": "100m"
              }
            }
          }
        ]
      }
    }
  }
}'

# Create ConfigMap with monitoring queries
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-monitoring
  namespace: dataflow-db
data:
  monitoring-queries.sql: |
    -- Storage monitoring queries
    SELECT 
      schemaname,
      tablename,
      pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
    FROM pg_tables 
    WHERE schemaname NOT IN ('information_schema', 'pg_catalog')
    ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
    
    -- Connection monitoring
    SELECT state, count(*) 
    FROM pg_stat_activity 
    GROUP BY state;
    
    -- Database size monitoring
    SELECT 
      datname,
      pg_size_pretty(pg_database_size(datname)) as size
    FROM pg_database;
  
  storage-check.sh: |
    #!/bin/bash
    # Storage utilization check script
    NAMESPACE="dataflow-db"
    
    echo "=== PostgreSQL Storage Monitoring ==="
    echo "Timestamp: $(date)"
    echo
    
    echo "PVC Usage:"
    kubectl get pvc -n $NAMESPACE -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,CAPACITY:.status.capacity.storage,STORAGECLASS:.spec.storageClassName"
    echo
    
    echo "Pod Storage Usage:"
    for pod in $(kubectl get pods -n $NAMESPACE -l app=postgres-cluster -o jsonpath='{.items[*].metadata.name}'); do
      echo "Pod: $pod"
      kubectl exec -n $NAMESPACE $pod -- df -h /var/lib/postgresql/data
      echo
    done
    
    echo "Database Sizes:"
    kubectl exec -n $NAMESPACE postgres-cluster-0 -- psql -U dataflow_user -d dataflow -c "
    SELECT 
      datname,
      pg_size_pretty(pg_database_size(datname)) as size
    FROM pg_database 
    WHERE datname NOT IN ('template0', 'template1');"
EOF

# Create monitoring job
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  name: storage-monitoring
  namespace: dataflow-db
spec:
  schedule: "*/15 * * * *"  # Every 15 minutes
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: storage-monitor
            image: bitnami/kubectl:latest
            volumeMounts:
            - name: monitoring-scripts
              mountPath: /scripts
            command:
            - /bin/bash
            - /scripts/storage-check.sh
          volumes:
          - name: monitoring-scripts
            configMap:
              name: postgres-monitoring
              defaultMode: 0755
EOF

# Run monitoring check manually
kubectl create job --from=cronjob/storage-monitoring storage-monitoring-manual -n dataflow-db
sleep 10
kubectl logs -n dataflow-db job/storage-monitoring-manual
```

**Explanation:**
This sets up comprehensive monitoring for PostgreSQL storage including PVC utilization, database sizes, and connection metrics. The PostgreSQL exporter provides Prometheus metrics, while the monitoring job provides regular health checks.

**Expected Output:**
```
=== PostgreSQL Storage Monitoring ===
Timestamp: Wed May 29 10:30:00 UTC 2025

PVC Usage:
NAME                                      STATUS   CAPACITY   STORAGECLASS
postgres-storage-postgres-cluster-0      Bound    20Gi       fast-ssd-db
postgres-storage-postgres-cluster-1      Bound    20Gi       fast-ssd-db
postgres-storage-postgres-cluster-2      Bound    20Gi       fast-ssd-db

Database Sizes:
 datname  |  size   
----------+---------
 dataflow | 128 MB
```

---

### Step 6: Rolling Updates and Disaster Recovery
**Command/Action:**
```bash
# Test rolling update with PostgreSQL version upgrade
echo "=== Rolling Update Test ==="
kubectl patch statefulset postgres-cluster -n dataflow-db -p='
{
  "spec": {
    "updateStrategy": {
      "type": "RollingUpdate",
      "rollingUpdate": {
        "partition": 0
      }
    },
    "template": {
      "spec": {
        "containers": [
          {
            "name": "postgres",
            "image": "postgres:15"
          }
        ]
      }
    }
  }
}'

# Monitor rolling update progress
kubectl rollout status statefulset/postgres-cluster -n dataflow-db

# Verify data integrity after update
kubectl exec -n dataflow-db postgres-cluster-0 -- psql -U dataflow_user -d dataflow -c "SELECT version();"
kubectl exec -n dataflow-db postgres-cluster-0 -- psql -U dataflow_user -d dataflow -c "SELECT COUNT(*) FROM customers;"

# Create disaster recovery backup
echo "=== Disaster Recovery Backup ==="
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: disaster-recovery-backup
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: dr-backup
        image: postgres:15
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
        command:
        - /bin/bash
        - -c
        - |
          BACKUP_FILE="/backup/disaster-recovery-$(date +%Y%m%d-%H%M%S).sql"
          echo "Creating disaster recovery backup: $BACKUP_FILE"
          
          # Full cluster backup with all databases
          pg_dumpall -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local \
                     -U dataflow_user \
                     --verbose \
                     --file="$BACKUP_FILE"
          
          if [ $? -eq 0 ]; then
            echo "Disaster recovery backup completed successfully"
            ls -la /backup/disaster-recovery-*.sql
          else
            echo "Disaster recovery backup failed!"
            exit 1
          fi
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-storage
EOF

kubectl wait --for=condition=complete job/disaster-recovery-backup -n dataflow-db --timeout=300s

# Simulate disaster recovery test (create test namespace)
echo "=== Disaster Recovery Test ==="
kubectl create namespace dataflow-dr-test

# Create test PostgreSQL instance for restore validation
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-dr-test
  namespace: dataflow-dr-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres-dr-test
  template:
    metadata:
      labels:
        app: postgres-dr-test
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: dataflow
        - name: POSTGRES_USER
          value: dataflow_user
        - name: POSTGRES_PASSWORD
          value: dataflow_pass
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        emptyDir: {}
EOF

kubectl wait --for=condition=available deployment/postgres-dr-test -n dataflow-dr-test --timeout=180s

# Test restore process
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: disaster-recovery-restore-test
  namespace: dataflow-dr-test
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: dr-restore
        image: postgres:15
        env:
        - name: PGPASSWORD
          value: dataflow_pass
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
        command:
        - /bin/bash
        - -c
        - |
          # Wait for PostgreSQL to be ready
          until pg_isready -h postgres-dr-test.dataflow-dr-test.svc.cluster.local -p 5432 -U dataflow_user; do
            echo "Waiting for test PostgreSQL..."
            sleep 5
          done
          
          # Find latest disaster recovery backup
          BACKUP_FILE=$(ls -t /backup/disaster-recovery-*.sql | head -1)
          echo "Restoring from: $BACKUP_FILE"
          
          # Restore backup
          psql -h postgres-dr-test.dataflow-dr-test.svc.cluster.local \
               -U dataflow_user \
               -d dataflow \
               -f "$BACKUP_FILE"
          
          if [ $? -eq 0 ]; then
            echo "Disaster recovery restore completed successfully"
            # Verify restored data
            psql -h postgres-dr-test.dataflow-dr-test.svc.cluster.local \
                 -U dataflow_user \
                 -d dataflow \
                 -c "SELECT 'Customers: ' || COUNT(*) FROM customers; SELECT 'Events: ' || COUNT(*) FROM analytics_events;"
          else
            echo "Disaster recovery restore failed!"
            exit 1
          fi
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-storage
          readOnly: true
EOF

kubectl wait --for=condition=complete job/disaster-recovery-restore-test -n dataflow-dr-test --timeout=300s
kubectl logs -n dataflow-dr-test job/disaster-recovery-restore-test
```

**Explanation:**
Rolling updates are performed in a controlled manner, updating pods one at a time while maintaining service availability. The disaster recovery test validates that backups can successfully restore data to a new environment.

**Expected Output:**
```
=== Rolling Update Test ===
statefulset.apps/postgres-cluster patched
deployment "postgres-cluster" successfully rolled out

                                                 version                                                  
----------------------------------------------------------------------------------------------------------
 PostgreSQL 15.2 on x86_64-pc-linux-gnu, compiled by gcc (Debian 11.2.0-2) 11.2.0, 64-bit

=== Disaster Recovery Test ===
Disaster recovery restore completed successfully
 ?column?  
-----------
 Customers: 5
 Events: 5
```

---

### Step 7: Performance Optimization and Scaling
**Command/Action:**
```bash
# Install pgbench for performance testing
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: pgbench-setup
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: pgbench-init
        image: postgres:15
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        command:
        - /bin/bash
        - -c
        - |
          # Initialize pgbench tables
          pgbench -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local \
                  -U dataflow_user \
                  -d dataflow \
                  -i \
                  -s 10
          
          echo "pgbench tables initialized with scale factor 10"
EOF

kubectl wait --for=condition=complete job/pgbench-setup -n dataflow-db --timeout=300s

# Run performance benchmark
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: performance-benchmark
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: pgbench-test
        image: postgres:15
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        command:
        - /bin/bash
        - -c
        - |
          echo "=== PostgreSQL Performance Benchmark ==="
          echo "Starting pgbench performance test..."
          
          # Run benchmark with 10 clients, 100 transactions each
          pgbench -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local \
                  -U dataflow_user \
                  -d dataflow \
                  -c 10 \
                  -j 2 \
                  -t 100 \
                  -r
          
          echo "Performance benchmark completed"
EOF

kubectl wait --for=condition=complete job/performance-benchmark -n dataflow-db --timeout=300s
kubectl logs -n dataflow-db job/performance-benchmark

# Test PVC expansion (if supported by storage class)
echo "=== Testing PVC Expansion ==="
kubectl patch pvc postgres-storage-postgres-cluster-0 -n dataflow-db -p='{"spec":{"resources":{"requests":{"storage":"30Gi"}}}}'

# Monitor expansion
kubectl get pvc postgres-storage-postgres-cluster-0 -n dataflow-db -w &
WATCH_PID=$!
sleep 30
kill $WATCH_PID

# Verify expanded storage
kubectl exec -n dataflow-db postgres-cluster-0 -- df -h /var/lib/postgresql/data

# Test horizontal scaling under load
echo "=== Testing Horizontal Scaling ==="
kubectl scale statefulset postgres-cluster --replicas=4 -n dataflow-db
kubectl wait --for=condition=ready pod/postgres-cluster-3 -n dataflow-db --timeout=300s

# Run concurrent benchmark on multiple replicas
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: load-test-distributed
  namespace: dataflow-db
spec:
  parallelism: 3
  completions: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: load-generator
        image: postgres:15
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: POD_INDEX
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
        command:
        - /bin/bash
        - -c
        - |
          TARGET_HOST="postgres-cluster-\${POD_INDEX:-0}.postgres-headless.dataflow-db.svc.cluster.local"
          echo "Running load test against: \$TARGET_HOST"
          
          pgbench -h \$TARGET_HOST \
                  -U dataflow_user \
                  -d dataflow \
                  -c 5 \
                  -j 1 \
                  -t 50 \
                  -r \
                  --report-per-command
EOF

kubectl wait --for=condition=complete job/load-test-distributed -n dataflow-db --timeout=600s

# Generate performance report
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: performance-report
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: reporter
        image: postgres:15
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        command:
        - /bin/bash
        - -c
        - |
          echo "=== PostgreSQL Cluster Performance Report ==="
          echo "Generated: $(date)"
          echo
          
          for i in {0..3}; do
            if kubectl get pod postgres-cluster-\$i -n dataflow-db &>/dev/null; then
              echo "=== postgres-cluster-\$i Performance Metrics ==="
              
              # Database stats
              psql -h postgres-cluster-\$i.postgres-headless.dataflow-db.svc.cluster.local \
                   -U dataflow_user \
                   -d dataflow \
                   -c "SELECT 
                        datname,
                        pg_size_pretty(pg_database_size(datname)) as size,
                        (SELECT count(*) FROM pg_stat_activity WHERE datname = d.datname) as connections
                       FROM pg_database d 
                       WHERE datname = 'dataflow';"
              
              # Table sizes
              psql -h postgres-cluster-\$i.postgres-headless.dataflow-db.svc.cluster.local \
                   -U dataflow_user \
                   -d dataflow \
                   -c "SELECT 
                        schemaname,
                        tablename,
                        pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
                       FROM pg_tables 
                       WHERE schemaname = 'public'
                       ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;"
              echo
            fi
          done
EOF

kubectl wait --for=condition=complete job/performance-report -n dataflow-db --timeout=180s
kubectl logs -n dataflow-db job/performance-report
```

**Explanation:**
Performance testing includes pgbench benchmarks to measure database throughput and latency. PVC expansion demonstrates storage scalability, while horizontal scaling shows how StatefulSets handle increased replicas with persistent storage.

**Expected Output:**
```
=== PostgreSQL Performance Benchmark ===
pgbench (15.2)
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 100
number of transactions actually processed: 1000/1000
latency average = 45.123 ms
initial connection time = 12.345 ms
tps = 221.654321 (without initial connection time)

=== PostgreSQL Cluster Performance Report ===
 datname  |  size   | connections 
----------+---------+-------------
 dataflow | 145 MB  |           1

 schemaname |    tablename     |  size   
------------+------------------+---------
 public     | pgbench_accounts | 128 MB
 public     | pgbench_history  | 32 kB
```

**Success Indicators:**
- [ ] All StatefulSet pods running with persistent storage
- [ ] Data migration completed successfully
- [ ] Backup and restore procedures tested and working
- [ ] High availability validated through failure scenarios
- [ ] Storage monitoring and alerting configured
- [ ] Rolling updates completed without data loss
- [ ] Performance benchmarks completed with acceptable results
- [ ] Horizontal and vertical scaling demonstrated

</details>

---

## 🎓 Knowledge Check

### Understanding Questions
1. **StatefulSets vs Deployments:** When should you use StatefulSets instead of Deployments for database workloads?
2. **Persistent Storage:** How do volumeClaimTemplates differ from regular volumes in terms of data persistence?
3. **Storage Classes:** What parameters should you consider when designing storage classes for database workloads?
4. **Backup Strategies:** How does Kubernetes-native backup differ from traditional database backup approaches?

### Hands-On Challenges
- [ ] **Variation 1:** Implement database replication with primary/replica configuration
- [ ] **Variation 2:** Set up cross-region backup storage with object storage
- [ ] **Integration:** Configure network policies to secure database cluster access

---

## 🔍 Common Pitfalls & Troubleshooting

### Frequent Mistakes
1. **PVC Binding Issues**
   - **Symptom:** Pods stuck in Pending state, PVC showing "Pending" status
   - **Cause:** Storage class not available or insufficient resources
   - **Fix:** Check storage class exists and has adequate capacity: `kubectl get storageclass`

2. **StatefulSet Pod Ordering Problems**
   - **Symptom:** Pods not starting in order (0, 1, 2...)
   - **Cause:** Previous pod not ready or storage issues
   - **Fix:** Check pod readiness probes and PVC status: `kubectl describe pod <name>`

3. **Data Loss During Updates**
   - **Symptom:** Database data missing after StatefulSet update
   - **Cause:** Incorrect volume mounts or PVC deletion
   - **Fix:** Verify PVC retention policy and volume mount paths

### Debug Commands
```bash
# Essential debugging commands for StatefulSets and storage
kubectl get statefulsets -o wide                    # StatefulSet status
kubectl describe statefulset <name>                 # Detailed StatefulSet info
kubectl get pvc -o wide                            # PVC status and binding
kubectl describe pvc <pvc-name>                     # PVC details and events
kubectl get events --sort-by=.metadata.creationTimestamp  # Recent events
kubectl exec <pod> -- df -h                        # Check storage usage
```

---

## 🌟 Real-World Applications

### Enterprise Scenarios
- **Database Migration:** Moving legacy databases to Kubernetes with zero downtime
- **Multi-Cloud Storage:** Using storage classes for different performance tiers
- **Disaster Recovery:** Implementing automated backup and restore procedures
- **Compliance:** Meeting data retention and backup requirements for regulated industries

### Best Practices Learned
- 🏆 **Storage Design:** Choose appropriate storage classes based on workload requirements
- 🏆 **Backup Strategy:** Implement automated, tested backup and restore procedures
- 🏆 **Monitoring:** Monitor both Kubernetes storage metrics and application-specific metrics
- 🏆 **Scaling Strategy:** Plan for both vertical (storage expansion) and horizontal scaling

---

## 📚 Additional Resources

### Related CKA Topics
- Understand storage classes, persistent volumes
- Use persistent volume claims
- Configure applications with persistent storage
- Understand StatefulSets vs Deployments

### Extended Learning
- [ ] Explore Container Storage Interface (CSI) drivers
- [ ] Study multi-zone storage replication strategies
- [ ] Learn about storage performance optimization
- [ ] Practice with different database technologies (MySQL, MongoDB, etc.)

---

## 📝 Lab Completion

### Self-Assessment
- [ ] I can complete this lab within 35 minutes
- [ ] I understand StatefulSets and their use cases for stateful applications
- [ ] I can configure persistent storage with appropriate storage classes
- [ ] I can implement backup and restore strategies for databases
- [ ] I'm confident troubleshooting storage and StatefulSet issues

### Notes & Reflections
*Record insights about StatefulSet design decisions, storage performance considerations, or backup strategies you discovered.*

---

### 🏁 Lab Status
- [ ] **Started:** ___________
- [ ] **Completed:** ___________  
- [ ] **Reviewed:** ___________
- [ ] **Mastered:** ___________

**Time Taken:** _______ | **Target Time:** 35 minutes  
**Difficulty Rating:** ___/5 | **Confidence Level:** ___/5

---

*💡 **Next Lab Recommendation:** T01 - Pod Crash Loop Analysis (Troubleshooting) or C02 - Advanced Cluster Backup (Cluster Maintenance)*
