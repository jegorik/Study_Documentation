# CKA Exam Cheat Sheet for Storage

---

## Section 1: Exam Topic Overview

### Key Concepts

- **Persistent Volumes (PV):** Cluster-wide storage resources
- **Persistent Volume Claims (PVC):** User requests for storage
- **Storage Classes:** Dynamic provisioning templates
- **Volume Types:** Different storage backends (hostPath, NFS, cloud storage)
- **Volume Mounts:** Attaching storage to containers
- **Dynamic Provisioning:** Automatic PV creation based on PVC requests

### Exam Objectives

- [ ] Create and configure persistent volumes and claims
- [ ] Implement storage classes for dynamic provisioning
- [ ] Configure volume mounts in pods and deployments
- [ ] Understand different volume types and their use cases
- [ ] Troubleshoot storage-related issues
- [ ] Manage storage capacity and access modes

### Study Resources

- Official Kubernetes Documentation: https://kubernetes.io/docs/concepts/storage/
- CNCF Training Materials: Storage and Persistent Volumes
- Practice Labs: Storage configuration and troubleshooting

### Exam Weight

**10%** of total exam score

---

## Section 2: ASCII Drawing with Explanation

```text
                        KUBERNETES STORAGE ARCHITECTURE
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                    STORAGE LIFECYCLE                            │
    │                                                                 │
    │  ┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐  │
    │  │    Pod      │    │       PVC       │    │       PV        │  │
    │  │             │    │  (User Request) │    │ (Cluster Admin) │  │
    │  │ ┌─────────┐ │    │                 │    │                 │  │
    │  │ │Volume   │ │────│ Claim: 10Gi     │────│ Capacity: 20Gi  │  │
    │  │ │Mount    │ │    │ AccessMode: RWO │    │ AccessMode: RWO │  │
    │  │ └─────────┘ │    │ StorageClass    │    │ StorageClass    │  │
    │  └─────────────┘    └─────────────────┘    └─────────────────┘  │
    │                              │                       │          │
    │                              │                       │          │
    │                              ▼                       │          │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                 STORAGE CLASS                           │    │
    │  │                                                         │    │
    │  │  Provisioner: kubernetes.io/aws-ebs                     │    │
    │  │  Parameters: type=gp2, zone=us-west-2a                  │    │
    │  │  ReclaimPolicy: Delete                                  │    │
    │  │  VolumeBindingMode: WaitForFirstConsumer                │    │
    │  └─────────────────────────────────────────────────────────┘    │
    │                              │                                  │
    │                              ▼                                  │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │              DYNAMIC PROVISIONING                       │    │
    │  │                                                         │    │
    │  │  PVC Created ──> Storage Class ──> Cloud Provider       │    │
    │  │       │              │                    │             │    │
    │  │       │              │                    ▼             │    │
    │  │       │              │          Physical Storage        │    │
    │  │       │              │          (EBS, GCE PD, etc.)     │    │
    │  │       │              │                    │             │    │
    │  │       │              ▼                    │             │    │
    │  │       └────────> PV Created <─────────────┘             │    │
    │  └─────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                       VOLUME TYPES                              │
    │                                                                 │
    │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐   │
    │ │  hostPath   │  │   emptyDir  │  │    Persistent Volumes   │   │
    │ │ (Node Local)│  │ (Pod Scope) │  │   (Cluster Scope)       │   │
    │ │             │  │             │  │                         │   │
    │ │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ ┌─────────┐ │   │
    │ │ │  /data  │ │  │ │ Memory  │ │  │ │   NFS   │ │  Cloud  │ │   │
    │ │ │ on Node │ │  │ │or Disk  │ │  │ │ Network │ │ Storage │ │   │
    │ │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ └─────────┘ │   │
    │ └─────────────┘  └─────────────┘  └─────────────────────────┘   │
    │                                                                 │
    │      Static              Ephemeral           Persistent         │
    │    (Development)        (Temporary)         (Production)        │
    └─────────────────────────────────────────────────────────────────┘

                          ACCESS MODES & BINDING
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                       ACCESS MODES                              │
    │                                                                 │
    │  ┌─────────────────┐  ┌─────────────────┐  ┌───────────────┐    │
    │  │ ReadWriteOnce   │  │ ReadOnlyMany    │  │ReadWriteMany  │    │
    │  │     (RWO)       │  │     (ROX)       │  │    (RWX)      │    │
    │  │                 │  │                 │  │               │    │
    │  │   Single Node   │  │  Multiple Nodes │  │Multiple Nodes │    │
    │  │   Read + Write  │  │   Read Only     │  │ Read + Write  │    │
    │  │                 │  │                 │  │               │    │
    │  │  ┌───────────┐  │  │  ┌───────────┐  │  │ ┌───────────┐ │    │
    │  │  │    Pod    │  │  │  │  Pod1     │  │  │ │   Pod1    │ │    │
    │  │  │   Node1   │  │  │  │  Node1    │  │  │ │   Node1   │ │    │
    │  │  └───────────┘  │  │  └───────────┘  │  │ └───────────┘ │    │
    │  │                 │  │  ┌───────────┐  │  │ ┌───────────┐ │    │
    │  │        X        │  │  │   Pod2    │  │  │ │   Pod2    │ │    │
    │  │  ┌───────────┐  │  │  │  Node2    │  │  │ │   Node2   │ │    │
    │  │  │    Pod    │  │  │  └───────────┘  │  │ └───────────┘ │    │
    │  │  │   Node2   │  │  │                 │  │               │    │
    │  │  └───────────┘  │  │                 │  │               │    │
    │  └─────────────────┘  └─────────────────┘  └───────────────┘    │
    └─────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

#### Storage Resources

1. **Persistent Volume (PV):**
   - Purpose: Cluster-level storage resource provisioned by administrator
   - Key features: Capacity, access modes, reclaim policy, storage class
   - Interactions: Bound to PVC, managed independently of pod lifecycle

2. **Persistent Volume Claim (PVC):**
   - Purpose: User request for storage resources
   - Key features: Size request, access mode, storage class selector
   - Interactions: Binds to matching PV, mounted in pods

3. **Storage Class:**
   - Purpose: Template for dynamic volume provisioning
   - Key features: Provisioner, parameters, reclaim policy
   - Interactions: Used by PVC to create PV automatically

#### Volume Types

1. **hostPath:**
   - Purpose: Mount file/directory from host node
   - Key features: Direct access to node filesystem
   - Use cases: Development, single-node testing (not recommended for production)

2. **emptyDir:**
   - Purpose: Empty directory for pod lifetime
   - Key features: Shared between containers in pod, deleted when pod dies
   - Use cases: Temporary storage, cache, shared data between containers

3. **Network Storage (NFS, Ceph, etc.):**
   - Purpose: Shared storage across multiple nodes
   - Key features: Network-attached, persistent, multi-node access
   - Use cases: Shared application data, distributed systems

#### Access Modes

1. **ReadWriteOnce (RWO):**
   - Single node can mount volume for read-write
   - Most common for database storage
   - Supported by most storage types

2. **ReadOnlyMany (ROX):**
   - Multiple nodes can mount volume for read-only
   - Useful for configuration data, static content
   - Limited storage type support

3. **ReadWriteMany (RWX):**
   - Multiple nodes can mount volume for read-write
   - Required for shared application storage
   - Limited to specific storage types (NFS, GlusterFS)

---

## Section 3: Live Examples

### Scenario: Setting up Persistent Storage for a Database Application

**Problem:** Deploy a MySQL database with persistent storage that survives pod restarts and can be dynamically provisioned

**Solution Steps:**

#### Step 1: Create Storage Class (if not exists)

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  zone: us-central1-a
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
EOF
```

#### Step 2: Create Persistent Volume Claim

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-ssd
EOF
```

#### Step 3: Verify PVC Status

```bash
kubectl get pvc mysql-pvc
```

**Expected Output:**

```bash
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pvc   Pending   -        -          -              fast-ssd       30s
```

#### Step 4: Deploy MySQL with Persistent Volume

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password123"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
EOF
```

#### Step 5: Verify PV Creation and Binding

```bash
kubectl get pv,pvc
```

**Expected Output:**

```bash
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS
persistentvolume/pvc-abc123-def456-ghi789  10Gi       RWO            Delete           Bound    default/mysql-pvc   fast-ssd

NAME                              STATUS   VOLUME                     CAPACITY   ACCESS MODES   STORAGECLASS
persistentvolumeclaim/mysql-pvc   Bound    pvc-abc123-def456-ghi789   10Gi       RWO            fast-ssd
```

#### Step 6: Test Data Persistence

```bash
# Connect to MySQL and create test data
kubectl exec -it mysql-xxx -- mysql -uroot -ppassword123 -e "CREATE DATABASE testdb; USE testdb; CREATE TABLE test (id INT, name VARCHAR(50)); INSERT INTO test VALUES (1, 'persistent data');"

# Delete and recreate pod
kubectl delete pod mysql-xxx
kubectl get pods -l app=mysql

# Verify data persistence
kubectl exec -it mysql-yyy -- mysql -uroot -ppassword123 -e "USE testdb; SELECT * FROM test;"
```

### Additional Scenarios

- **Scenario 2:** Creating static PV for existing storage and manual binding
- **Scenario 3:** Configuring multi-container pod with shared volume storage

---

## Section 4: Table of Useful Commands

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `kubectl get pv` | List persistent volumes | `-o wide`, `--sort-by=.spec.capacity.storage` | `kubectl get pv -o wide` |
| `kubectl get pvc` | List persistent volume claims | `-o wide`, `--all-namespaces` | `kubectl get pvc -A` |
| `kubectl describe pv` | Show PV details | `[pv-name]` | `kubectl describe pv pvc-123` |
| `kubectl describe pvc` | Show PVC details | `[pvc-name]` | `kubectl describe pvc mysql-pvc` |
| `kubectl get storageclass` | List storage classes | `-o yaml` | `kubectl get sc -o yaml` |

### Storage Class Management

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl create -f` | Create storage class | `kubectl create -f storageclass.yaml` |
| `kubectl patch storageclass` | Update storage class | `kubectl patch sc fast-ssd -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'` |
| `kubectl get sc` | List storage classes | `kubectl get storageclass` |

### Volume Operations

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl apply -f` | Create PV/PVC from YAML | `kubectl apply -f pvc.yaml` |
| `kubectl delete pvc` | Delete PVC | `kubectl delete pvc mysql-pvc` |
| `kubectl patch pv` | Update PV reclaim policy | `kubectl patch pv pvc-123 -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'` |

### Troubleshooting Storage

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get events` | Check storage events | `kubectl get events --field-selector involvedObject.kind=PersistentVolume` |
| `kubectl logs` | Check provisioner logs | `kubectl logs -n kube-system -l app=gce-pd-driver` |
| `kubectl describe node` | Check node storage | `kubectl describe node worker-1 \| grep -A 10 "Allocated resources"` |

---

## Section 5: Best Practices

### Storage Design

1. **Storage Class Strategy:**
   - Create environment-specific storage classes (dev, staging, prod)
   - Use appropriate storage types for workload requirements
   - Set reasonable default storage class
   - Configure proper reclaim policies
   - Example production storage class:

   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: prod-ssd
     annotations:
       storageclass.kubernetes.io/is-default-class: "true"
   provisioner: kubernetes.io/gce-pd
   parameters:
     type: pd-ssd
     replication-type: regional-pd
   reclaimPolicy: Retain
   volumeBindingMode: WaitForFirstConsumer
   ```

2. **PVC Sizing Strategy:**
   - Plan storage capacity based on application growth
   - Use monitoring to track storage utilization
   - Implement storage quotas per namespace
   - Consider expansion capabilities of storage backend

3. **Access Mode Selection:**
   - Use ReadWriteOnce (RWO) for databases and single-instance apps
   - Use ReadOnlyMany (ROX) for configuration and static content
   - Use ReadWriteMany (RWX) only when absolutely necessary
   - Understand storage backend limitations

### Performance Optimization

1. **Storage Performance:**
   - Choose appropriate storage type for IOPS requirements
   - Use SSD for high-performance workloads
   - Consider local storage for latency-sensitive applications
   - Monitor storage performance metrics

2. **Volume Placement:**
   - Use topology-aware scheduling
   - Consider node affinity for storage locality
   - Implement pod disruption budgets for stateful apps
   - Plan for multi-zone deployments

### Security Considerations

1. **Data Protection:**
   - Enable encryption at rest and in transit
   - Implement proper backup strategies
   - Use security contexts to control volume access
   - Regular security scanning of storage systems

2. **Access Control:**
   - Use RBAC to control PVC creation
   - Implement namespace isolation
   - Restrict storage class usage
   - Audit storage access patterns

### Backup and Recovery

1. **Backup Strategies:**
   - Implement automated backup schedules
   - Test backup and restore procedures
   - Use volume snapshots when available
   - Document recovery procedures
   - Example backup using volume snapshots:

   ```yaml
   apiVersion: snapshot.storage.k8s.io/v1
   kind: VolumeSnapshot
   metadata:
     name: mysql-snapshot
   spec:
     volumeSnapshotClassName: csi-snapshotter
     source:
       persistentVolumeClaimName: mysql-pvc
   ```

2. **Disaster Recovery:**
   - Plan for multi-region deployments
   - Implement cross-region backup replication
   - Document disaster recovery procedures
   - Regular disaster recovery testing

### Troubleshooting Best Practices

1. **Common Storage Issues:**
   - PVC stuck in Pending state
   - Check storage class availability and parameters
   - Verify node capacity and resources
   - Check provisioner pod status and logs
   - Validate access modes compatibility

2. **Debugging Steps:**

   ```bash
   # Check PVC status and events
   kubectl describe pvc mysql-pvc
   
   # Check storage class configuration
   kubectl describe sc fast-ssd
   
   # Check provisioner logs
   kubectl logs -n kube-system -l app=csi-driver
   
   # Check node capacity
   kubectl describe nodes | grep -A 5 "Allocated resources"
   ```

3. **Performance Issues:**
   - Monitor storage latency and throughput
   - Check for resource contention
   - Validate storage backend health
   - Consider storage optimization

### Cost Management

1. **Storage Optimization:**
   - Right-size storage requests
   - Implement storage lifecycle policies
   - Use appropriate storage tiers
   - Monitor storage utilization and costs

2. **Resource Management:**
   - Set storage quotas and limits
   - Clean up unused PVCs
   - Implement automated cleanup policies
   - Regular cost analysis and optimization

### Exam-Specific Tips

- **Time Management:** Practice creating PV/PVC scenarios quickly
- **Command Shortcuts:** Master kubectl commands for storage resources
- **Verification:** Always verify storage binding and mounting
- **Documentation:** Know where to find storage examples and troubleshooting guides

---

## Quick Reference Card

### Essential Commands for Storage

```bash
kubectl get pv,pvc -o wide                    # List storage resources
kubectl describe pvc mysql-pvc                # PVC details and events
kubectl get storageclass                      # Available storage classes
kubectl apply -f pvc.yaml                     # Create PVC from YAML
kubectl delete pvc mysql-pvc                  # Delete PVC
```

### Quick PVC Creation

```bash
# Create PVC with specific storage class
kubectl create -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-ssd
EOF
```

### Volume Mount in Pod

```yaml
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: app-storage
      mountPath: /data
  volumes:
  - name: app-storage
    persistentVolumeClaim:
      claimName: app-storage
```

### Troubleshooting Commands

```bash
kubectl get events --field-selector involvedObject.kind=PersistentVolume
kubectl describe storageclass fast-ssd
kubectl logs -n kube-system -l app=provisioner
```

### Key Storage Facts

- **RWO:** Single node read-write (most common)
- **ROX:** Multiple nodes read-only
- **RWX:** Multiple nodes read-write (limited support)
- **Reclaim Policies:** Retain, Delete, Recycle

---

## Related Topics

- [Kubernetes Architecture]
- [Workloads & Scheduling]
- [Security and RBAC]
- [Troubleshooting and Monitoring]

---

*Last Updated: May 29, 2025*
*CKA Exam Version: v1.30*
