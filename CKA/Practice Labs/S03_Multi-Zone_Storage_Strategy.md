# S03 - Multi-Zone Storage Strategy

> **🎯 Lab Focus:** Advanced multi-zone storage deployment, disaster recovery, and cross-region data management  
> **⏱️ Estimated Time:** 35-40 minutes  
> **🏷️ Difficulty:** Advanced  
> **📋 Category:** Storage (10% Exam Weight)

---

## 🎬 Scenario: Global E-Commerce Platform Expansion

You're the Lead Storage Engineer at **GlobalMart**, a rapidly expanding e-commerce platform that's launching in three new geographical regions. The platform must handle millions of transactions daily with strict data residency requirements, disaster recovery capabilities, and zero-downtime deployments. You need to design and implement a sophisticated multi-zone storage strategy that ensures data availability, performance, and compliance across different geographical locations.

**Business Critical Requirements:**

- 🌍 **Multi-Region Deployment:** US-East, EU-West, APAC-South zones
- 📊 **Data Residency Compliance:** Customer data must stay in respective regions
- 🔄 **Cross-Zone Replication:** Critical data backup across zones
- ⚡ **High Performance:** Sub-100ms database response times
- 🛡️ **Disaster Recovery:** RPO < 5 minutes, RTO < 15 minutes
- 📈 **Elastic Scaling:** Handle Black Friday 10x traffic spikes

**Revenue Impact:** $2M per hour during peak shopping seasons

---

## 🏗️ Lab Environment Setup

### Initial Cluster State

```bash
# Expected multi-zone cluster topology
NODES:
- us-east-1a-master     # Primary control plane (US East)
- us-east-1b-worker-1   # US East Zone B
- us-east-1c-worker-2   # US East Zone C
- eu-west-1a-worker-3   # EU West Zone A
- eu-west-1b-worker-4   # EU West Zone B
- apac-south-1a-worker-5 # APAC South Zone A

STORAGE CLASSES:
- fast-ssd-us-east      # High-performance US storage
- standard-eu-west      # Standard EU storage
- replicated-apac       # Replicated APAC storage
- cross-zone-backup     # Multi-zone backup storage
```

### Prerequisites

- Multi-zone Kubernetes cluster (3+ zones)
- Multiple storage classes with zone affinity
- StatefulSets for database workloads
- Dynamic volume provisioning capability
- Snapshot and backup infrastructure

---

## 📋 Tasks Overview

| Task | Objective | Est. Time | Difficulty |
|------|-----------|-----------|------------|
| 1 | Multi-Zone Storage Assessment | 6 min | Intermediate |
| 2 | Zone-Aware Storage Classes | 8 min | Advanced |
| 3 | StatefulSet Multi-Zone Deployment | 10 min | Expert |
| 4 | Cross-Zone Data Replication | 8 min | Expert |
| 5 | Disaster Recovery Implementation | 5 min | Advanced |
| 6 | Performance Testing & Validation | 3 min | Intermediate |

---

## 🔍 Task 1: Multi-Zone Storage Assessment (6 minutes)

### Objective

Analyze the current storage infrastructure, identify zone distribution patterns, and assess storage performance characteristics across different zones.

### Current Situation

GlobalMart's current storage setup is single-zone focused with no disaster recovery strategy. You need to understand the existing storage patterns and plan for multi-zone expansion.

### Your Mission

1. **Audit current storage classes and their zone affinity**
2. **Assess existing PV/PVC distribution across zones**
3. **Analyze storage performance characteristics**
4. **Identify data residency and compliance requirements**

### Step-by-Step Instructions

#### 1.1 Current Storage Infrastructure Analysis

```bash
# Check existing storage classes
kubectl get storageclass -o wide

# Analyze storage class parameters and provisioners
kubectl describe storageclass

# Check current PV distribution
kubectl get pv -o wide

# Analyze PVC usage patterns
kubectl get pvc -A -o wide

# Check node distribution across zones
kubectl get nodes --show-labels | grep topology.kubernetes.io/zone
```

#### 1.2 Zone and Storage Topology Assessment

```bash
# Identify available zones
kubectl get nodes -o jsonpath='{.items[*].metadata.labels.topology\.kubernetes\.io/zone}' | tr ' ' '\n' | sort | uniq

# Check storage provisioner capabilities
kubectl get csi* -A 2>/dev/null || echo "No CSI drivers found"

# Analyze current pod-to-storage affinity
kubectl get pods -A -o yaml | grep -A 10 -B 10 "nodeAffinity\|volumeClaimTemplates"

# Check for any existing zone-specific storage
kubectl get pv -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeAffinity}{"\n"}{end}'
```

#### 1.3 Performance and Compliance Baseline

```bash
# Check current storage performance characteristics
kubectl get storageclass -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.parameters}{"\n"}{end}'

# Identify workloads requiring high performance
kubectl get pods -A -o yaml | grep -A 5 -B 5 "storageClass\|storage:"

# Document current backup and snapshot capabilities
kubectl get volumesnapshot* -A 2>/dev/null || echo "No snapshot resources found"
```

### Lab Environment Setup

#### Create Multi-Zone Node Labels (Simulation)

```bash
# Simulate multi-zone environment by labeling nodes
# In a real environment, these labels would be set by cloud provider

# Label nodes with zones (adjust node names as needed)
kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') topology.kubernetes.io/zone=us-east-1a
kubectl label node $(kubectl get nodes -o jsonpath='{.items[1].metadata.name}') topology.kubernetes.io/zone=us-east-1b
kubectl label node $(kubectl get nodes -o jsonpath='{.items[2].metadata.name}') topology.kubernetes.io/zone=us-east-1c

# Add region labels
kubectl label nodes --all topology.kubernetes.io/region=us-east-1

# Verify zone labeling
kubectl get nodes --show-labels | grep topology.kubernetes.io/zone
```

#### Create Base Namespaces

```yaml
# multi-zone-namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce-us-east
  labels:
    region: us-east
    compliance: gdpr-us
    data-residency: us
---
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce-eu-west
  labels:
    region: eu-west
    compliance: gdpr-eu
    data-residency: eu
---
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce-apac
  labels:
    region: apac-south
    compliance: gdpr-apac
    data-residency: apac
---
apiVersion: v1
kind: Namespace
metadata:
  name: cross-zone-backup
  labels:
    purpose: disaster-recovery
    access-level: restricted
```

```bash
# Apply namespace setup
kubectl apply -f multi-zone-namespaces.yaml
```

### Expected Findings

- Current storage is likely single-zone focused
- No zone-aware storage classes exist
- Limited disaster recovery capabilities
- Need for zone-specific performance optimization

### 🚨 Troubleshooting Tips

- **Zone labels** are crucial for proper scheduling and storage affinity
- **Storage provisioners** must support zone-aware provisioning
- **Node distribution** affects storage accessibility and performance
- **Cloud provider integration** is essential for dynamic provisioning

---

## 📦 Task 2: Zone-Aware Storage Classes (8 minutes)

### Objective

Create comprehensive zone-aware storage classes that optimize performance, ensure data residency, and provide different service levels across geographical zones.

### Current Situation

Existing storage classes don't consider zone placement or regional requirements. You need to create specialized storage classes for different zones and use cases.

### Your Mission

1. **Create high-performance storage classes for each zone**
2. **Implement region-specific storage classes for compliance**
3. **Design cross-zone replication storage classes**
4. **Set up backup and archival storage classes**

### Step-by-Step Instructions

#### 2.1 Zone-Specific High-Performance Storage Classes

```yaml
# high-performance-storage-classes.yaml
# US East Zone - Ultra-fast SSD storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ultra-fast-us-east
  labels:
    region: us-east
    performance: ultra-high
provisioner: kubernetes.io/aws-ebs  # Use appropriate provisioner
parameters:
  type: gp3
  iops: "16000"
  throughput: "1000"
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - us-east-1a
    - us-east-1b
    - us-east-1c
---
# EU West Zone - GDPR compliant storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gdpr-compliant-eu-west
  labels:
    region: eu-west
    compliance: gdpr
    data-residency: eu
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "12000"
  throughput: "750"
  fsType: ext4
  encrypted: "true"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - eu-west-1a
    - eu-west-1b
---
# APAC South Zone - Cost-optimized storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cost-optimized-apac
  labels:
    region: apac-south
    performance: standard
    cost: optimized
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - apac-south-1a
    - apac-south-1b
```

#### 2.2 Cross-Zone Replication Storage Classes

```yaml
# cross-zone-replication-storage.yaml
# Multi-zone replicated storage for critical data
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: multi-zone-replicated
  labels:
    replication: cross-zone
    criticality: high
    backup: enabled
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "8000"
  throughput: "500"
  fsType: ext4
  replication-type: synchronous
allowVolumeExpansion: true
volumeBindingMode: Immediate  # For replicated storage
# No topology constraints - can span zones
---
# Regional backup storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: regional-backup
  labels:
    purpose: backup
    retention: long-term
    cost: low
provisioner: kubernetes.io/aws-ebs
parameters:
  type: sc1  # Cold HDD for cost-effective backup
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain  # Keep backups even if PVC is deleted
```

#### 2.3 Database-Optimized Storage Classes

```yaml
# database-optimized-storage.yaml
# Primary database storage - ultra-high performance
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: database-primary
  labels:
    workload: database
    tier: primary
    performance: maximum
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io2  # Provisioned IOPS for consistent performance
  iops: "20000"
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - us-east-1a  # Primary zone
---
# Database replica storage - high performance
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: database-replica
  labels:
    workload: database
    tier: replica
    performance: high
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "12000"
  throughput: "750"
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
# Can be placed in any zone for read replicas
```

#### 2.4 Snapshot and Archive Storage Classes

```yaml
# snapshot-archive-storage.yaml
# Snapshot storage class for point-in-time recovery
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: snapshot-storage
  labels:
    purpose: snapshots
    retention: medium-term
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
allowVolumeExpansion: false
volumeBindingMode: Immediate
reclaimPolicy: Retain
---
# Archive storage for long-term retention
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: archive-storage
  labels:
    purpose: archive
    retention: long-term
    cost: minimal
provisioner: kubernetes.io/aws-ebs
parameters:
  type: sc1  # Cold storage
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

#### 2.5 Apply Storage Classes

```bash
# Apply all storage class configurations
kubectl apply -f high-performance-storage-classes.yaml
kubectl apply -f cross-zone-replication-storage.yaml
kubectl apply -f database-optimized-storage.yaml
kubectl apply -f snapshot-archive-storage.yaml

# Verify storage classes are created
kubectl get storageclass

# Check storage class details
kubectl describe storageclass ultra-fast-us-east
kubectl describe storageclass gdpr-compliant-eu-west
kubectl describe storageclass multi-zone-replicated

# Set default storage class for each region (optional)
kubectl patch storageclass ultra-fast-us-east -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Expected Outcomes

- Zone-aware storage classes for optimal performance
- Compliance-ready storage classes for GDPR requirements
- Cross-zone replication capabilities for disaster recovery
- Specialized storage classes for different workload types

---

## 🗄️ Task 3: StatefulSet Multi-Zone Deployment (10 minutes)

### Objective

Deploy critical database and application workloads across multiple zones using StatefulSets with sophisticated storage strategies and anti-affinity rules.

### Current Situation

Applications need to be distributed across zones for high availability while maintaining data consistency and performance. You need to implement StatefulSets that can handle zone failures gracefully.

### Your Mission

1. **Deploy primary database cluster across zones**
2. **Implement read replicas with zone distribution**
3. **Create application tiers with storage affinity**
4. **Configure anti-affinity and zone spreading**

### Step-by-Step Instructions

#### 3.1 Primary Database Cluster Deployment

```yaml
# primary-database-cluster.yaml
# PostgreSQL primary cluster with zone awareness
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-primary
  namespace: ecommerce-us-east
  labels:
    app: postgres
    tier: primary
    region: us-east
spec:
  serviceName: postgres-primary-svc
  replicas: 3
  selector:
    matchLabels:
      app: postgres
      tier: primary
  template:
    metadata:
      labels:
        app: postgres
        tier: primary
        region: us-east
    spec:
      affinity:
        # Ensure pods are spread across zones
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - postgres
              - key: tier
                operator: In
                values:
                - primary
            topologyKey: topology.kubernetes.io/zone
        # Prefer placement in specific zones
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                - us-east-1a
                - us-east-1b
                - us-east-1c
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: ecommerce
        - name: POSTGRES_USER
          value: admin
        - name: POSTGRES_PASSWORD
          value: securepassword123
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 1000m
            memory: 2Gi
          limits:
            cpu: 2000m
            memory: 4Gi
        livenessProbe:
          tcpSocket:
            port: 5432
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U admin -d ecommerce
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: database-primary
      resources:
        requests:
          storage: 100Gi
---
# Service for primary database
apiVersion: v1
kind: Service
metadata:
  name: postgres-primary-svc
  namespace: ecommerce-us-east
spec:
  selector:
    app: postgres
    tier: primary
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None  # Headless service for StatefulSet
```

#### 3.2 Read Replica Deployment Across Zones

```yaml
# database-read-replicas.yaml
# Read replicas distributed across zones
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-read-replica
  namespace: ecommerce-us-east
  labels:
    app: postgres
    tier: replica
    region: us-east
spec:
  serviceName: postgres-replica-svc
  replicas: 6  # 2 replicas per zone
  selector:
    matchLabels:
      app: postgres
      tier: replica
  template:
    metadata:
      labels:
        app: postgres
        tier: replica
        region: us-east
    spec:
      affinity:
        # Distribute replicas across zones
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - postgres
                - key: tier
                  operator: In
                  values:
                  - replica
              topologyKey: topology.kubernetes.io/zone
        # Ensure even distribution across all zones
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                - us-east-1a
                - us-east-1b
                - us-east-1c
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: ecommerce
        - name: POSTGRES_USER
          value: admin
        - name: POSTGRES_PASSWORD
          value: securepassword123
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        - name: POSTGRES_MASTER_SERVICE
          value: postgres-primary-svc.ecommerce-us-east.svc.cluster.local
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - name: postgres-replica-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
          limits:
            cpu: 1000m
            memory: 2Gi
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U admin -d ecommerce
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-replica-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: database-replica
      resources:
        requests:
          storage: 50Gi
---
# Service for read replicas
apiVersion: v1
kind: Service
metadata:
  name: postgres-replica-svc
  namespace: ecommerce-us-east
spec:
  selector:
    app: postgres
    tier: replica
  ports:
  - port: 5432
    targetPort: 5432
```

#### 3.3 Application Tier with Zone-Aware Storage

```yaml
# application-tier-deployment.yaml
# E-commerce application with zone-aware persistent storage
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: ecommerce-app
  namespace: ecommerce-us-east
  labels:
    app: ecommerce
    tier: application
spec:
  serviceName: ecommerce-app-svc
  replicas: 9  # 3 replicas per zone
  selector:
    matchLabels:
      app: ecommerce
      tier: application
  template:
    metadata:
      labels:
        app: ecommerce
        tier: application
    spec:
      affinity:
        # Distribute across zones for high availability
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - ecommerce
              topologyKey: topology.kubernetes.io/zone
        # Node affinity to ensure zone distribution
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                - us-east-1a
                - us-east-1b
                - us-east-1c
      containers:
      - name: ecommerce-app
        image: nginx:alpine  # Placeholder for actual application
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: app-storage
          mountPath: /app/data
        - name: app-logs
          mountPath: /app/logs
        - name: cache-storage
          mountPath: /app/cache
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
        env:
        - name: DATABASE_URL
          value: postgres://admin:securepassword123@postgres-primary-svc:5432/ecommerce
        - name: READ_REPLICA_URL
          value: postgres://admin:securepassword123@postgres-replica-svc:5432/ecommerce
        - name: ZONE
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['topology.kubernetes.io/zone']
  volumeClaimTemplates:
  - metadata:
      name: app-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ultra-fast-us-east
      resources:
        requests:
          storage: 20Gi
  - metadata:
      name: app-logs
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ultra-fast-us-east
      resources:
        requests:
          storage: 10Gi
  - metadata:
      name: cache-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ultra-fast-us-east
      resources:
        requests:
          storage: 5Gi
---
# Service for application tier
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-app-svc
  namespace: ecommerce-us-east
spec:
  selector:
    app: ecommerce
    tier: application
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

#### 3.4 Cross-Zone Cache Cluster

```yaml
# cross-zone-cache-cluster.yaml
# Redis cluster spread across zones for session management
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: ecommerce-us-east
  labels:
    app: redis
    tier: cache
spec:
  serviceName: redis-cluster-svc
  replicas: 6  # 2 masters + 4 replicas across 3 zones
  selector:
    matchLabels:
      app: redis
      tier: cache
  template:
    metadata:
      labels:
        app: redis
        tier: cache
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis
            topologyKey: topology.kubernetes.io/zone
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-storage
          mountPath: /data
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 1Gi
        command:
        - redis-server
        - --appendonly
        - "yes"
        - --save
        - "900 1"
        - --save
        - "300 10"
  volumeClaimTemplates:
  - metadata:
      name: redis-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ultra-fast-us-east
      resources:
        requests:
          storage: 10Gi
---
# Service for Redis cluster
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-svc
  namespace: ecommerce-us-east
spec:
  selector:
    app: redis
    tier: cache
  ports:
  - port: 6379
    targetPort: 6379
  clusterIP: None
```

#### 3.5 Deploy Multi-Zone StatefulSets

```bash
# Apply all StatefulSet configurations
kubectl apply -f primary-database-cluster.yaml
kubectl apply -f database-read-replicas.yaml
kubectl apply -f application-tier-deployment.yaml
kubectl apply -f cross-zone-cache-cluster.yaml

# Monitor deployment progress
kubectl get statefulsets -n ecommerce-us-east
kubectl get pods -n ecommerce-us-east -o wide

# Check zone distribution
kubectl get pods -n ecommerce-us-east -o wide | awk '{print $1, $7}' | grep -E "(postgres|ecommerce|redis)"

# Verify storage allocation
kubectl get pvc -n ecommerce-us-east

# Check that pods are spread across zones
kubectl get pods -n ecommerce-us-east -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\n"}{end}' | while read pod node; do
  zone=$(kubectl get node $node -o jsonpath='{.metadata.labels.topology\.kubernetes\.io/zone}')
  echo "$pod -> $node ($zone)"
done
```

### Expected Outcomes

- Database clusters distributed across all zones
- Application tiers with proper anti-affinity rules
- Cache clusters with cross-zone replication
- Storage volumes allocated in appropriate zones

---

## 🔄 Task 4: Cross-Zone Data Replication (8 minutes)

### Objective

Implement sophisticated cross-zone data replication strategies for disaster recovery, data consistency, and compliance requirements.

### Current Situation

While applications are distributed across zones, you need to ensure data replication and backup strategies that can handle zone failures and maintain business continuity.

### Your Mission

1. **Set up cross-zone database replication**
2. **Implement volume snapshot strategies**
3. **Create cross-zone backup procedures**
4. **Configure data synchronization between regions**

### Step-by-Step Instructions

#### 4.1 Volume Snapshot Configuration

```yaml
# volume-snapshot-class.yaml
# Snapshot class for cross-zone backups
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: cross-zone-snapshot-class
  labels:
    purpose: disaster-recovery
driver: ebs.csi.aws.com  # Use appropriate CSI driver
deletionPolicy: Retain
parameters:
  csi.storage.k8s.io/snapshotter-secret-name: ebs-snapshot-secret
  csi.storage.k8s.io/snapshotter-secret-namespace: kube-system
---
# Automated snapshot policy
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: automated-backup-snapshots
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: ebs.csi.aws.com
deletionPolicy: Delete
parameters:
  csi.storage.k8s.io/snapshotter-secret-name: ebs-snapshot-secret
  csi.storage.k8s.io/snapshotter-secret-namespace: kube-system
```

#### 4.2 Cross-Zone Backup StatefulSet

```yaml
# cross-zone-backup-system.yaml
# Backup service that replicates data across zones
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: backup-service
  namespace: cross-zone-backup
  labels:
    app: backup-service
    purpose: disaster-recovery
spec:
  serviceName: backup-service-svc
  replicas: 3  # One in each zone
  selector:
    matchLabels:
      app: backup-service
  template:
    metadata:
      labels:
        app: backup-service
        purpose: disaster-recovery
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - backup-service
            topologyKey: topology.kubernetes.io/zone
      containers:
      - name: backup-agent
        image: postgres:13  # Contains pg_dump and other tools
        command:
        - /bin/bash
        - -c
        - |
          while true; do
            echo "Starting backup process..."
            # Database backup
            pg_dump -h postgres-primary-svc.ecommerce-us-east.svc.cluster.local \
                   -U admin -d ecommerce > /backup/db_backup_$(date +%Y%m%d_%H%M%S).sql
            
            # Compress backup
            gzip /backup/db_backup_*.sql
            
            # Clean old backups (keep last 7 days)
            find /backup -name "*.gz" -mtime +7 -delete
            
            echo "Backup completed. Sleeping for 1 hour..."
            sleep 3600
          done
        env:
        - name: PGPASSWORD
          value: securepassword123
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 2Gi
  volumeClaimTemplates:
  - metadata:
      name: backup-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: regional-backup
      resources:
        requests:
          storage: 500Gi
```

#### 4.3 Automated Snapshot CronJob

```yaml
# automated-snapshot-cronjob.yaml
# Automated snapshot creation across zones
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-snapshot-job
  namespace: ecommerce-us-east
  labels:
    purpose: automated-backup
spec:
  schedule: "0 */6 * * *"  # Every 6 hours
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: snapshot-creator
            image: bitnami/kubectl:latest
            command:
            - /bin/bash
            - -c
            - |
              echo "Creating snapshots for all database PVCs..."
              
              # Get all postgres PVCs
              for pvc in $(kubectl get pvc -l app=postgres -o jsonpath='{.items[*].metadata.name}'); do
                snapshot_name="${pvc}-snapshot-$(date +%Y%m%d-%H%M%S)"
                
                cat <<EOF | kubectl apply -f -
              apiVersion: snapshot.storage.k8s.io/v1
              kind: VolumeSnapshot
              metadata:
                name: ${snapshot_name}
                namespace: ecommerce-us-east
                labels:
                  pvc-source: ${pvc}
                  automated: "true"
              spec:
                volumeSnapshotClassName: cross-zone-snapshot-class
                source:
                  persistentVolumeClaimName: ${pvc}
              EOF
                
                echo "Created snapshot: ${snapshot_name}"
              done
              
              echo "Cleaning up old snapshots (keep last 48 hours)..."
              # Delete snapshots older than 48 hours
              kubectl get volumesnapshot -l automated=true -o json | \
              jq -r ".items[] | select(.metadata.creationTimestamp < \"$(date -d '48 hours ago' -Iseconds)\") | .metadata.name" | \
              xargs -r kubectl delete volumesnapshot
              
              echo "Snapshot job completed successfully"
          serviceAccountName: snapshot-creator
---
# ServiceAccount for snapshot operations
apiVersion: v1
kind: ServiceAccount
metadata:
  name: snapshot-creator
  namespace: ecommerce-us-east
---
# ClusterRole for snapshot operations
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: snapshot-creator-role
rules:
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list"]
- apiGroups: ["snapshot.storage.k8s.io"]
  resources: ["volumesnapshots"]
  verbs: ["get", "list", "create", "delete"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: snapshot-creator-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: snapshot-creator-role
subjects:
- kind: ServiceAccount
  name: snapshot-creator
  namespace: ecommerce-us-east
```

#### 4.4 Cross-Region Data Sync Service

```yaml
# cross-region-sync-service.yaml
# Service for synchronizing data between regions
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cross-region-sync
  namespace: cross-zone-backup
  labels:
    app: cross-region-sync
    purpose: disaster-recovery
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cross-region-sync
  template:
    metadata:
      labels:
        app: cross-region-sync
    spec:
      containers:
      - name: sync-agent
        image: amazon/aws-cli:latest
        command:
        - /bin/bash
        - -c
        - |
          while true; do
            echo "Starting cross-region sync..."
            
            # Sync backups to secondary region
            aws s3 sync /backup s3://globalmart-backup-eu-west/ \
              --storage-class STANDARD_IA \
              --delete \
              --region eu-west-1
            
            # Sync to APAC region
            aws s3 sync /backup s3://globalmart-backup-apac/ \
              --storage-class STANDARD_IA \
              --delete \
              --region ap-south-1
            
            echo "Cross-region sync completed. Sleeping for 2 hours..."
            sleep 7200
          done
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: access-key-id
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: secret-access-key
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
          readOnly: true
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 1Gi
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: cross-region-backup-pvc
---
# PVC for cross-region sync storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cross-region-backup-pvc
  namespace: cross-zone-backup
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: regional-backup
  resources:
    requests:
      storage: 1Ti
```

#### 4.5 Deploy Replication Infrastructure

```bash
# Apply snapshot configuration
kubectl apply -f volume-snapshot-class.yaml
kubectl apply -f cross-zone-backup-system.yaml
kubectl apply -f automated-snapshot-cronjob.yaml
kubectl apply -f cross-region-sync-service.yaml

# Create AWS credentials secret (with actual credentials)
kubectl create secret generic aws-credentials \
  --from-literal=access-key-id=YOUR_ACCESS_KEY \
  --from-literal=secret-access-key=YOUR_SECRET_KEY \
  -n cross-zone-backup

# Verify backup services are running
kubectl get statefulsets -n cross-zone-backup
kubectl get cronjobs -n ecommerce-us-east

# Test manual snapshot creation
kubectl create -f - <<EOF
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: test-snapshot-$(date +%s)
  namespace: ecommerce-us-east
spec:
  volumeSnapshotClassName: cross-zone-snapshot-class
  source:
    persistentVolumeClaimName: postgres-storage-postgres-primary-0
EOF

# Check snapshot status
kubectl get volumesnapshot -n ecommerce-us-east
```

### Expected Outcomes

- Automated snapshot creation every 6 hours
- Cross-zone backup services running in each zone
- Cross-region data synchronization active
- Disaster recovery capabilities established

---

## 🛡️ Task 5: Disaster Recovery Implementation (5 minutes)

### Objective

Implement comprehensive disaster recovery procedures that can quickly restore services in case of zone failures or data corruption.

### Current Situation

Backup and replication systems are in place. Now you need to create recovery procedures and test disaster recovery scenarios.

### Your Mission

1. **Create disaster recovery playbooks**
2. **Implement automated failover mechanisms**
3. **Test zone failure scenarios**
4. **Validate recovery time objectives (RTO)**

### Step-by-Step Instructions

#### 5.1 Disaster Recovery ConfigMap

```yaml
# disaster-recovery-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: disaster-recovery-config
  namespace: cross-zone-backup
  labels:
    purpose: disaster-recovery
data:
  recovery-procedures.md: |
    # Disaster Recovery Procedures
    
    ## Zone Failure Recovery
    
    ### 1. Database Recovery
    ```bash
    # Identify affected zone
    FAILED_ZONE="us-east-1a"
    
    # Scale up replicas in healthy zones
    kubectl scale statefulset postgres-read-replica --replicas=8 -n ecommerce-us-east
    
    # Promote read replica to primary if needed
    kubectl patch statefulset postgres-primary -p '{"spec":{"template":{"spec":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"topology.kubernetes.io/zone","operator":"NotIn","values":["'$FAILED_ZONE'"]}]}]}}}}}}'
    ```
    
    ### 2. Application Recovery
    ```bash
    # Reschedule application pods away from failed zone
    kubectl patch statefulset ecommerce-app -p '{"spec":{"template":{"spec":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"topology.kubernetes.io/zone","operator":"NotIn","values":["'$FAILED_ZONE'"]}]}]}}}}}}'
    
    # Scale up to maintain capacity
    kubectl scale statefulset ecommerce-app --replicas=12 -n ecommerce-us-east
    ```
    
    ### 3. Storage Recovery
    ```bash
    # List available snapshots
    kubectl get volumesnapshot -n ecommerce-us-east
    
    # Restore from latest snapshot
    SNAPSHOT_NAME="postgres-storage-postgres-primary-0-snapshot-latest"
    kubectl create -f - <<EOF
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: postgres-recovery-pvc
      namespace: ecommerce-us-east
    spec:
      accessModes:
      - ReadWriteOnce
      storageClassName: database-primary
      resources:
        requests:
          storage: 100Gi
      dataSource:
        name: $SNAPSHOT_NAME
        kind: VolumeSnapshot
        apiGroup: snapshot.storage.k8s.io
    EOF
    ```
    
  rto-targets.yaml: |
    # Recovery Time Objectives
    
    Database Recovery: 5 minutes
    Application Recovery: 3 minutes  
    Storage Recovery: 10 minutes
    Full Service Restoration: 15 minutes
    
    # Recovery Point Objectives
    
    Database RPO: 5 minutes (snapshot frequency)
    Application Data RPO: 1 hour (backup frequency)
    Configuration RPO: Real-time (GitOps)
```

#### 5.2 Automated Failover Script

```yaml
# automated-failover-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: disaster-recovery-test
  namespace: cross-zone-backup
  labels:
    purpose: dr-testing
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: dr-test
        image: bitnami/kubectl:latest
        command:
        - /bin/bash
        - -c
        - |
          echo "=== Disaster Recovery Test ==="
          
          # Simulate zone failure by cordoning nodes
          FAILED_ZONE="us-east-1a"
          echo "Simulating failure in zone: $FAILED_ZONE"
          
          # Cordon all nodes in the failed zone
          kubectl get nodes -l topology.kubernetes.io/zone=$FAILED_ZONE -o jsonpath='{.items[*].metadata.name}' | \
          xargs -n1 kubectl cordon
          
          echo "Nodes in $FAILED_ZONE cordoned"
          
          # Force reschedule pods from failed zone
          kubectl get pods -n ecommerce-us-east -o wide | grep $FAILED_ZONE | awk '{print $1}' | \
          xargs -n1 kubectl delete pod -n ecommerce-us-east --grace-period=0 --force
          
          echo "Pods from failed zone deleted"
          
          # Wait for pods to reschedule
          echo "Waiting for pods to reschedule..."
          kubectl wait --for=condition=Ready pod -l app=postgres -n ecommerce-us-east --timeout=300s
          kubectl wait --for=condition=Ready pod -l app=ecommerce -n ecommerce-us-east --timeout=300s
          
          # Verify all pods are running in healthy zones
          echo "=== Recovery Verification ==="
          kubectl get pods -n ecommerce-us-east -o wide | grep -v $FAILED_ZONE
          
          # Test database connectivity
          echo "Testing database connectivity..."
          kubectl run db-test --image=postgres:13 --rm -it --restart=Never -n ecommerce-us-east -- \
            psql -h postgres-primary-svc -U admin -d ecommerce -c "SELECT version();"
          
          # Uncordon nodes to restore normal operation
          echo "Restoring normal operation..."
          kubectl get nodes -l topology.kubernetes.io/zone=$FAILED_ZONE -o jsonpath='{.items[*].metadata.name}' | \
          xargs -n1 kubectl uncordon
          
          echo "=== Disaster Recovery Test Completed ==="
      serviceAccountName: snapshot-creator
```

#### 5.3 Health Check and Monitoring

```yaml
# health-check-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: health-monitor
  namespace: cross-zone-backup
  labels:
    app: health-monitor
spec:
  replicas: 1
  selector:
    matchLabels:
      app: health-monitor
  template:
    metadata:
      labels:
        app: health-monitor
    spec:
      containers:
      - name: health-check
        image: curlimages/curl:latest
        command:
        - /bin/sh
        - -c
        - |
          while true; do
            echo "=== Health Check $(date) ==="
            
            # Check database connectivity
            if kubectl run db-health-check --image=postgres:13 --rm --restart=Never -n ecommerce-us-east -- \
               psql -h postgres-primary-svc -U admin -d ecommerce -c "SELECT 1;" > /dev/null 2>&1; then
              echo "✅ Database: HEALTHY"
            else
              echo "❌ Database: UNHEALTHY"
            fi
            
            # Check application availability
            if kubectl get pods -n ecommerce-us-east -l app=ecommerce | grep -q Running; then
              echo "✅ Application: HEALTHY"
            else
              echo "❌ Application: UNHEALTHY"
            fi
            
            # Check zone distribution
            echo "Pod distribution across zones:"
            kubectl get pods -n ecommerce-us-east -o wide | awk '{print $7}' | sort | uniq -c
            
            echo "Sleeping for 5 minutes..."
            sleep 300
          done
        env:
        - name: PGPASSWORD
          value: securepassword123
        resources:
          requests:
            cpu: 50m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
```

#### 5.4 Execute Disaster Recovery Tests

```bash
# Apply disaster recovery configuration
kubectl apply -f disaster-recovery-config.yaml
kubectl apply -f automated-failover-job.yaml
kubectl apply -f health-check-service.yaml

# Run disaster recovery test
kubectl create job dr-test-manual --from=job/disaster-recovery-test -n cross-zone-backup

# Monitor the test
kubectl logs job/dr-test-manual -n cross-zone-backup -f

# Check health monitoring
kubectl logs deployment/health-monitor -n cross-zone-backup -f

# Verify service availability during test
while true; do
  echo "Testing service availability..."
  kubectl run availability-test --image=curlimages/curl --rm --restart=Never -n ecommerce-us-east -- \
    curl -s ecommerce-app-svc:8080 || echo "Service unavailable"
  sleep 10
done
```

### Expected Outcomes

- Disaster recovery procedures documented and tested
- Automated failover mechanisms functional
- RTO/RPO targets met during testing
- Service availability maintained during zone failures

---

## ✅ Task 6: Performance Testing & Validation (3 minutes)

### Objective

Validate that the multi-zone storage strategy meets performance requirements and business objectives.

### Current Situation

All infrastructure is deployed and disaster recovery is tested. You need to validate performance characteristics and ensure the solution meets business requirements.

### Your Mission

1. **Test storage performance across zones**
2. **Validate database response times**
3. **Measure failover times**
4. **Generate compliance and performance reports**

### Step-by-Step Instructions

#### 6.1 Storage Performance Testing

```bash
# Test storage performance in each zone
for zone in us-east-1a us-east-1b us-east-1c; do
  echo "Testing storage performance in zone: $zone"
  
  kubectl run storage-perf-test-$zone \
    --image=ubuntu:latest \
    --rm -it --restart=Never \
    --overrides='{"spec":{"nodeSelector":{"topology.kubernetes.io/zone":"'$zone'"}}}' -- \
    bash -c '
      # Install fio for storage testing
      apt-get update && apt-get install -y fio
      
      # Test sequential write performance
      fio --name=seqwrite --rw=write --bs=1M --size=1G --direct=1
      
      # Test random read/write performance  
      fio --name=randwrite --rw=randwrite --bs=4k --size=500M --direct=1 --runtime=30s
      
      # Test database-like workload
      fio --name=dbsim --rw=randrw --bs=8k --size=500M --direct=1 --runtime=30s --rwmixread=70
    '
done
```

#### 6.2 Database Performance Validation

```bash
# Test database performance
kubectl run db-performance-test --image=postgres:13 --rm -it --restart=Never -n ecommerce-us-east -- \
bash -c '
  # Install pgbench
  apt-get update && apt-get install -y postgresql-contrib
  
  # Initialize pgbench
  pgbench -h postgres-primary-svc -U admin -d ecommerce -i -s 10
  
  # Run performance test
  echo "Testing database performance..."
  pgbench -h postgres-primary-svc -U admin -d ecommerce -c 10 -j 2 -t 1000
  
  # Test read replica performance
  echo "Testing read replica performance..."
  pgbench -h postgres-replica-svc -U admin -d ecommerce -c 5 -j 1 -t 500 -S
'

# Monitor database metrics during test
kubectl top pods -n ecommerce-us-east -l app=postgres
```

#### 6.3 Failover Time Measurement

```bash
# Measure failover performance
echo "Measuring failover times..."

# Start monitoring service availability
(while true; do
  if kubectl run availability-check --image=curlimages/curl --rm --restart=Never -n ecommerce-us-east -- \
     curl -s postgres-primary-svc:5432 > /dev/null 2>&1; then
    echo "$(date): Service AVAILABLE"
  else
    echo "$(date): Service UNAVAILABLE"
  fi
  sleep 1
done) &

MONITOR_PID=$!

# Simulate pod failure
FAILED_POD=$(kubectl get pods -n ecommerce-us-east -l app=postgres,tier=primary -o jsonpath='{.items[0].metadata.name}')
echo "Failing pod: $FAILED_POD"

START_TIME=$(date +%s)
kubectl delete pod $FAILED_POD -n ecommerce-us-east --force --grace-period=0

# Wait for recovery
kubectl wait --for=condition=Ready pod -l app=postgres,tier=primary -n ecommerce-us-east --timeout=300s
END_TIME=$(date +%s)

# Calculate failover time
FAILOVER_TIME=$((END_TIME - START_TIME))
echo "Failover completed in: ${FAILOVER_TIME} seconds"

# Stop monitoring
kill $MONITOR_PID
```

#### 6.4 Generate Performance Report

```bash
# Generate comprehensive performance report
cat > performance_report.md << EOF
# Multi-Zone Storage Performance Report

## Executive Summary
- **Deployment Date:** $(date)
- **Zones Covered:** us-east-1a, us-east-1b, us-east-1c
- **Total Storage Allocated:** $(kubectl get pvc -A -o jsonpath='{.items[*].spec.resources.requests.storage}' | tr ' ' '\n' | grep -E '[0-9]+Gi' | sed 's/Gi//' | awk '{sum+=$1} END {print sum "Gi"}')
- **Applications Deployed:** $(kubectl get statefulsets -A --no-headers | wc -l) StatefulSets

## Performance Metrics

### Storage Performance
- **Primary Database Storage:** Ultra-high IOPS (20,000+)
- **Replica Storage:** High IOPS (12,000+)
- **Application Storage:** Fast SSD performance
- **Backup Storage:** Cost-optimized cold storage

### Database Performance
- **Primary Database Response:** <5ms average
- **Read Replica Response:** <10ms average
- **Concurrent Connections:** 100+ supported
- **Throughput:** 1000+ TPS

### Disaster Recovery Metrics
- **Snapshot Frequency:** Every 6 hours
- **Cross-Zone Backup:** Every hour
- **RTO Achieved:** ${FAILOVER_TIME} seconds (Target: <900s)
- **RPO Achieved:** <5 minutes (Target: <5 minutes)

## Zone Distribution
$(kubectl get pods -n ecommerce-us-east -o wide | awk '{print $7}' | sort | uniq -c)

## Compliance Status
- ✅ Data Residency: All data stored in designated regions
- ✅ Encryption: All storage classes use encryption at rest
- ✅ Backup Retention: 7-day retention policy implemented
- ✅ Cross-Zone Replication: Active across all zones

## Recommendations
1. Monitor storage utilization and plan for expansion
2. Implement automated alerting for failover events
3. Conduct monthly disaster recovery drills
4. Review and update backup retention policies quarterly

## Cost Optimization
- Primary storage: Premium tier for performance
- Replica storage: Standard tier for cost balance
- Backup storage: Cold tier for long-term retention
- Archive storage: Minimal cost for compliance

Report Generated: $(date)
EOF

echo "Performance report generated: performance_report.md"
cat performance_report.md
```

### Expected Outcomes

- Storage performance meets business requirements
- Database response times under target thresholds
- Failover times within acceptable limits
- Comprehensive performance documentation

---

## 🎯 Success Criteria

### ✅ Must Complete (Pass)

- [ ] Analyzed and documented current multi-zone storage landscape
- [ ] Created zone-aware storage classes for different performance tiers
- [ ] Deployed StatefulSets with proper zone distribution and anti-affinity
- [ ] Implemented cross-zone data replication and backup strategies
- [ ] Configured disaster recovery procedures and tested failover scenarios
- [ ] Validated performance meets business requirements (<100ms response times)

### 🌟 Excellence Indicators (Distinction)

- [ ] Zero data loss during zone failure simulations
- [ ] Sub-15 minute recovery time objectives achieved
- [ ] Cost-optimized storage tier selection implemented
- [ ] Comprehensive compliance and audit documentation
- [ ] Automated monitoring and alerting for storage health
- [ ] Performance optimization for different workload types

### ⚡ Speed Benchmarks

- **Target Time:** 35 minutes
- **Expert Time:** 30 minutes
- **Storage Configuration:** < 15 minutes
- **StatefulSet Deployment:** < 15 minutes
- **Testing & Validation:** < 10 minutes

---

## 🔍 Post-Lab Review

### Key Learning Outcomes

1. **Multi-Zone Architecture:** Designing storage solutions that span multiple availability zones
2. **Storage Class Strategy:** Creating specialized storage classes for different performance and compliance needs
3. **StatefulSet Management:** Deploying stateful applications with zone awareness and anti-affinity
4. **Disaster Recovery:** Implementing comprehensive backup, replication, and recovery procedures
5. **Performance Optimization:** Balancing performance, cost, and availability across zones

### Real-World Applications

- **E-Commerce Platforms:** Handling peak traffic with zone-resilient storage
- **Financial Services:** Meeting strict compliance and disaster recovery requirements
- **SaaS Applications:** Providing consistent performance across global regions
- **Media Streaming:** Managing large-scale content delivery with zone distribution

### Advanced Storage Patterns

#### 1. Multi-Region Storage Federation

```yaml
# Example: Cross-region storage federation
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: global-replicated-storage
parameters:
  replication-regions: "us-east-1,eu-west-1,ap-south-1"
  consistency-level: "eventual"
  conflict-resolution: "last-write-wins"
```

#### 2. Intelligent Tiering

```yaml
# Example: Automated storage tiering
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: intelligent-tiering
parameters:
  initial-tier: "fast-ssd"
  tier-down-after: "30d"
  archive-after: "90d"
  access-pattern-monitoring: "enabled"
```

### Common Pitfalls & Solutions

| Problem | Symptom | Solution |
|---------|---------|----------|
| Cross-zone network latency | Slow replication performance | Use dedicated replication networks and optimize batch sizes |
| Storage cost explosion | High monthly storage bills | Implement tiering and lifecycle policies |
| Zone affinity conflicts | Pods can't schedule | Review anti-affinity rules and zone capacity |
| Backup window impact | Performance degradation during backups | Use snapshot-based backups and off-peak scheduling |

### Exam Tips

- **Practice zone constraints** and topology spread constraints
- **Understand storage class parameters** for different cloud providers
- **Master StatefulSet** zone distribution patterns
- **Know snapshot and backup** restoration procedures
- **Remember cost implications** of different storage tiers

---

## 📚 Additional Resources

### Kubernetes Documentation

- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Volume Snapshots](https://kubernetes.io/docs/concepts/storage/volume-snapshots/)
- [Pod Topology Spread Constraints](https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/)

### Cloud Provider Guides

- [AWS EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [Azure Disk CSI Driver](https://docs.microsoft.com/en-us/azure/aks/azure-disk-csi)
- [Google Persistent Disk CSI](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/gce-pd-csi-driver)

### Best Practices

- [Storage Best Practices](https://kubernetes.io/docs/concepts/storage/storage-best-practices/)
- [StatefulSet Best Practices](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#best-practices)
- [Disaster Recovery Planning](https://kubernetes.io/docs/concepts/cluster-administration/disaster-recovery/)

---

## 🚀 Next Steps

### Immediate Actions

1. **Practice with different cloud providers** to understand CSI driver differences
2. **Experiment with various storage types** (block, file, object) for different use cases
3. **Implement monitoring and alerting** for storage performance and capacity
4. **Create runbooks** for common storage operations and disaster scenarios

### Advanced Scenarios to Explore

- **Multi-Cloud Storage:** Deploying across different cloud providers
- **Edge Computing Storage:** Managing storage in edge locations
- **Container Image Storage:** Optimizing container registry storage
- **Backup Automation:** Advanced backup and restore automation

### Production Preparation

- Implement **automated capacity planning** based on usage patterns
- Create **cost optimization dashboards** for storage spending
- Establish **performance baselines** and SLA monitoring
- Train team on **disaster recovery procedures** and regular drills

---

*🎯 "Data is the new oil, but storage is the refinery. A well-designed multi-zone storage strategy turns raw data into reliable, available, and performant business value."*

**Lab Complete!** 🎉 You've successfully implemented an enterprise-grade multi-zone storage strategy that can handle global scale, compliance requirements, and disaster recovery scenarios!
