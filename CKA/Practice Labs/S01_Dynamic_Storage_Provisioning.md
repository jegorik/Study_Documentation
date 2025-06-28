# S01: Dynamic Storage Provisioning

> **🎯 Exam Objectives:** Storage management (10% weight) - Configure and manage persistent volumes  
> **⚡ Scenario:** DataFlow Analytics needs scalable storage infrastructure for their growing data pipeline  
> **⏱️ Time Limit:** 20 minutes  
> **💼 Difficulty:** Basic  

---

## 📋 Business Scenario

You're the Infrastructure Engineer at **DataFlow Analytics**, a fast-growing startup that processes customer behavior data for e-commerce clients. Your data engineering team has been manually creating storage volumes for each new data processing job, but this approach is becoming unmanageable as you scale to handle more clients.

**Your Mission:** Implement dynamic storage provisioning to automatically create storage volumes on-demand for data processing workloads, eliminating manual volume management and enabling rapid scaling.

**Business Requirements:**

- 📊 **Data Processing Jobs** need 10GB-50GB storage per job
- 🔄 **Auto-scaling** storage based on demand without manual intervention
- 💰 **Cost Efficiency** - only provision storage when needed
- 🛡️ **Data Persistence** - ensure data survives pod restarts
- ⚡ **Fast Provisioning** - storage available within 30 seconds of request

---

## 🎯 Lab Objectives

By completing this lab, you will master:

1. **StorageClass Configuration** - Dynamic provisioning setup
2. **PVC/PV Relationship** - Understanding persistent volume claims
3. **Volume Lifecycle Management** - Creation, binding, and cleanup
4. **Storage Parameters** - Configuring provisioning parameters
5. **Application Integration** - Mounting dynamic volumes in workloads

---

## 🔧 Lab Tasks

### **Task 1: StorageClass Setup (5 minutes)**

Configure dynamic storage provisioning for the data analytics platform.

**Your Actions:**

1. Create a StorageClass named `dataflow-ssd` for high-performance data processing
2. Configure automatic volume provisioning with SSD performance characteristics
3. Set appropriate parameters for data analytics workloads (ext4 filesystem, immediate binding)
4. Verify the StorageClass is available and set as default

**Expected CKA Skills:**

- `kubectl create storageclass`
- StorageClass configuration parameters
- Default storage class management
- Provisioner selection and configuration

---

### **Task 2: Dynamic Volume Claims (6 minutes)**

Create persistent volume claims that will trigger automatic volume provisioning.

**Your Actions:**

1. Create a PVC named `analytics-data-small` requesting 10GB of storage
2. Create a PVC named `analytics-data-large` requesting 50GB of storage  
3. Configure appropriate access modes for multi-pod data sharing
4. Verify that persistent volumes are automatically created and bound
5. Examine the automatically created PVs and their properties

**Expected CKA Skills:**

- `kubectl create pvc` and PVC YAML configuration
- Access modes understanding (ReadWriteOnce, ReadWriteMany, ReadOnlyMany)
- PV/PVC binding mechanics
- Volume status monitoring

---

### **Task 3: Application Integration (6 minutes)**

Deploy data processing applications that utilize the dynamically provisioned storage.

**Your Actions:**

1. Create a data ingestion pod that mounts the small storage volume
2. Deploy a data processing job that uses the large storage volume
3. Configure proper volume mount paths for data analytics workflows
4. Test data persistence by writing files and restarting pods
5. Verify data remains available after pod recreation

**Expected CKA Skills:**

- Pod volume mounting and configuration
- Volume mount path management
- Data persistence verification
- Pod-storage integration patterns

---

### **Task 4: Storage Management (3 minutes)**

Implement storage lifecycle management and cleanup procedures.

**Your Actions:**

1. Demonstrate storage expansion capabilities (if supported)
2. Configure volume retention policies for production data
3. Clean up test data and implement storage reclaim procedures
4. Document storage usage patterns and cost optimization recommendations

**Expected CKA Skills:**

- Volume expansion and resize operations
- Reclaim policies (Retain, Delete, Recycle)
- Storage monitoring and capacity planning
- Production storage management practices

---

## 💡 Hints & Tips

<details>
<summary>🆘 <strong>StorageClass configuration unclear?</strong> Click for guidance...</summary>

**Basic StorageClass Pattern:**

```bash
# Check available provisioners first
kubectl get storageclasses

# Create custom StorageClass
kubectl create storageclass dataflow-ssd \
  --provisioner=kubernetes.io/no-provisioner \
  --parameters replication-type=async \
  --allow-volume-expansion=true
```

**Key Concepts:**

- Provisioner determines storage backend (cloud provider, local, etc.)
- Parameters control storage characteristics (performance, replication, etc.)
- Volume expansion allows growing volumes after creation
- Binding mode controls when volumes are created (Immediate vs WaitForFirstConsumer)

</details>

<details>
<summary>🔍 <strong>PVC binding issues?</strong> Click for troubleshooting approach...</summary>

**PVC Debugging Strategy:**

```bash
# Check PVC status and events
kubectl describe pvc analytics-data-small

# Check for available PVs
kubectl get pv

# Verify StorageClass exists and is correct
kubectl get storageclass

# Check provisioner logs if using external provisioner
kubectl logs -n kube-system -l app=storage-provisioner
```

**Common Issues:**

- StorageClass not found or typo in name
- Insufficient cluster storage capacity
- Provisioner not running or misconfigured
- Node selector conflicts preventing binding

</details>

<details>
<summary>⚙️ <strong>Volume mounting problems?</strong> Click for configuration examples...</summary>

**Pod Volume Mount Configuration:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-processor
spec:
  containers:
  - name: processor
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: analytics-data-large
```

**Data Persistence Testing:**

```bash
# Write test data
kubectl exec data-processor -- sh -c 'echo "test data" > /data/test.txt'

# Delete pod and recreate
kubectl delete pod data-processor
kubectl apply -f pod.yaml

# Verify data persisted
kubectl exec data-processor -- cat /data/test.txt
```

</details>

---

## ✅ Success Criteria

You've successfully completed this lab when:

- [ ] **StorageClass Created** - Dynamic provisioning configured and functional
- [ ] **Automatic PV Creation** - PVCs trigger automatic volume provisioning
- [ ] **Application Integration** - Pods successfully mount and use dynamic volumes
- [ ] **Data Persistence** - Data survives pod restarts and recreation
- [ ] **Storage Management** - Lifecycle policies and cleanup procedures implemented

---

## 🎓 Key Learning Outcomes

After completing this lab, you'll be expert at:

✅ **Dynamic Storage Provisioning** - Configuring automatic volume creation  
✅ **StorageClass Management** - Setting up storage classes for different workload needs  
✅ **PV/PVC Lifecycle** - Understanding volume claiming and binding processes  
✅ **Application Storage Integration** - Mounting persistent volumes in workloads  
✅ **Storage Operations** - Managing volume expansion, retention, and cleanup  

---

## 🔄 Next Steps

Ready for more storage challenges? Try these related labs:

- **S02: Database Migration Scenario** - Advanced volume types and backup strategies
- **S03: Multi-Zone Storage Strategy** - CSI drivers and availability zone management
- **T06: Storage Mount Issues** - Troubleshooting persistent volume problems
- **W02: Auto-scaling E-commerce Site** - Storage considerations for scaling applications

---

## 📚 Additional Resources

**CKA Exam Topics Covered:**

- Persistent volumes and persistent volume claims
- Storage classes and dynamic provisioning
- Volume lifecycle management
- Application storage integration

**Documentation Links:**

- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)

---

*💡 **Pro Tip:** Dynamic storage provisioning is essential for production Kubernetes clusters. Set up appropriate storage classes early to avoid manual volume management overhead as your applications scale.*
