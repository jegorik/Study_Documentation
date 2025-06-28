# S01 Setup Environment: Dynamic Storage Provisioning

> **🎯 Scenario Setup:** DataFlow Analytics storage infrastructure for data processing pipeline  
> **⚡ Purpose:** Create realistic dynamic storage provisioning environment for hands-on practice  
> **⏱️ Setup Time:** 3-5 minutes  

---

## 📊 Scenario Context

This setup creates a **DataFlow Analytics** environment where:
- Data processing workloads need on-demand storage provisioning
- Multiple storage classes simulate different performance tiers
- Storage requirements vary from small (10GB) to large (50GB) volumes
- Students practice real-world storage lifecycle management

---

## 🚀 Quick Setup (Copy & Paste)

### **Step 1: Create Namespace and Base Environment**

```bash
# Create dedicated namespace for the data analytics platform
kubectl create namespace dataflow-analytics

# Set default namespace for convenience
kubectl config set-context --current --namespace=dataflow-analytics

echo "✅ DataFlow Analytics namespace created"
```

### **Step 2: Prepare Storage Infrastructure**

```bash
# Note: This setup works on most Kubernetes clusters
# For specific cloud providers, you may need to adjust the provisioner

# Create a sample storage class (students will create their own)
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
parameters:
  type: local
  replication-type: async
EOF

# Create some local persistent volumes for testing (simulates cloud storage)
# These will be used if dynamic provisioning isn't available
for i in {1..3}; do
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-${i}
  labels:
    storage-tier: standard
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: manual
  local:
    path: /tmp/data${i}
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')
EOF
done

echo "✅ Base storage infrastructure prepared"
```

### **Step 3: Create Sample Data Processing Applications**

```bash
# Create a sample data ingestion app template (students will modify this)
cat <<EOF > /tmp/data-ingestion-template.yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-ingestion
  namespace: dataflow-analytics
  labels:
    app: data-ingestion
    component: pipeline
spec:
  containers:
  - name: ingestion
    image: busybox:1.35
    command: ['sh', '-c']
    args:
    - |
      echo "🔄 DataFlow Analytics - Data Ingestion Started"
      echo "📊 Processing customer behavior data..."
      mkdir -p /data/input /data/output
      echo "Sample customer data batch $(date)" > /data/input/customers.csv
      echo "Processing..." > /data/processing.log
      while true; do
        echo "$(date): Processing 1000 records/sec" >> /data/processing.log
        sleep 30
      done
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    emptyDir: {}  # Students will replace this with PVC
  restartPolicy: Never
EOF

# Create a sample data processing job template
cat <<EOF > /tmp/data-processor-template.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: analytics-processor
  namespace: dataflow-analytics
  labels:
    app: analytics-processor
    component: pipeline
spec:
  template:
    metadata:
      labels:
        app: analytics-processor
    spec:
      containers:
      - name: processor
        image: busybox:1.35
        command: ['sh', '-c']
        args:
        - |
          echo "🔬 DataFlow Analytics - Big Data Processing Started"
          echo "📈 Analyzing customer patterns..."
          mkdir -p /analytics/raw /analytics/processed
          echo "Raw data processing batch $(date)" > /analytics/raw/dataset.json
          echo "Analysis results $(date)" > /analytics/processed/insights.json
          echo "✅ Analytics processing completed"
        volumeMounts:
        - name: analytics-volume
          mountPath: /analytics
      volumes:
      - name: analytics-volume
        emptyDir: {}  # Students will replace this with PVC
      restartPolicy: Never
EOF

echo "✅ Application templates created in /tmp/"
echo "📋 Students will modify these to use dynamic storage"
```

### **Step 4: Create Monitoring and Validation Tools**

```bash
# Create a storage monitoring script
cat <<EOF > /tmp/storage-monitor.sh
#!/bin/bash
echo "📊 DataFlow Analytics - Storage Status Dashboard"
echo "================================================"
echo
echo "🏗️  STORAGE CLASSES:"
kubectl get storageclass -o wide
echo
echo "💾 PERSISTENT VOLUMES:"
kubectl get pv -o wide
echo  
echo "📋 PERSISTENT VOLUME CLAIMS:"
kubectl get pvc -n dataflow-analytics -o wide
echo
echo "🔗 VOLUME BINDINGS:"
kubectl get pvc -n dataflow-analytics -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\t"}{.spec.volumeName}{"\n"}{end}' | column -t
echo
echo "📊 STORAGE USAGE:"
kubectl top pods -n dataflow-analytics 2>/dev/null || echo "Metrics not available"
EOF

chmod +x /tmp/storage-monitor.sh

echo "✅ Storage monitoring tools ready"
```

---

## 🔍 Verification Commands

### **Environment Health Check**
```bash
# Check namespace and initial setup
kubectl get all -n dataflow-analytics

# Verify storage classes
kubectl get storageclass

# Check available persistent volumes
kubectl get pv

# Run storage monitoring dashboard
/tmp/storage-monitor.sh
```

### **During Lab Monitoring**
```bash
# Watch PVC creation and binding in real-time
kubectl get pvc -w -n dataflow-analytics

# Monitor PV automatic creation
kubectl get pv -w

# Check storage class usage
kubectl describe storageclass dataflow-ssd 2>/dev/null || echo "StorageClass not created yet"

# Monitor application storage usage
kubectl exec -n dataflow-analytics <pod-name> -- df -h /data 2>/dev/null || echo "No pods with mounted storage yet"
```

### **Storage Testing Commands**
```bash
# Test data persistence (run after students deploy applications)
kubectl exec -n dataflow-analytics data-ingestion -- sh -c 'echo "test persistence" > /data/test.txt'
kubectl delete pod data-ingestion -n dataflow-analytics
# Re-deploy pod and check if data persists
kubectl exec -n dataflow-analytics data-ingestion -- cat /data/test.txt

# Check volume expansion capabilities
kubectl get storageclass dataflow-ssd -o jsonpath='{.allowVolumeExpansion}'
```

---

## 🎯 Expected Student Actions

### **Task 1: StorageClass Creation**
Students should create:
```bash
kubectl create storageclass dataflow-ssd --provisioner=<appropriate-provisioner> --parameters=<storage-params>
```

### **Task 2: PVC Creation**
Students should create PVCs like:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: analytics-data-small
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: dataflow-ssd
```

### **Task 3: Application Integration**
Students should modify the templates to use their PVCs:
```yaml
volumes:
- name: data-volume
  persistentVolumeClaim:
    claimName: analytics-data-small
```

---

## 🧹 Cleanup Commands

```bash
# Clean up all lab resources
kubectl delete namespace dataflow-analytics

# Clean up any orphaned PVs (if using local storage)
kubectl delete pv local-pv-1 local-pv-2 local-pv-3 2>/dev/null || true

# Clean up storage classes (keep default ones)
kubectl delete storageclass dataflow-ssd standard-storage 2>/dev/null || true

# Clean up temporary files
rm -f /tmp/data-ingestion-template.yaml
rm -f /tmp/data-processor-template.yaml  
rm -f /tmp/storage-monitor.sh

# Reset context
kubectl config set-context --current --namespace=default

echo "✅ Environment cleaned up successfully"
```

---

## 🔧 Instructor Notes

### **Timing Setup:**
1. **Pre-lab (3-5 min):** Execute all setup steps
2. **Lab Start:** Provide students with template files and storage-monitor.sh
3. **During Lab:** Use monitoring commands to verify student progress

### **Common Student Challenges:**
- **Provisioner Selection:** May not know appropriate provisioner for their cluster
- **Access Modes:** Confusion between ReadWriteOnce vs ReadWriteMany
- **Binding Issues:** PVCs stuck in Pending due to StorageClass problems
- **Volume Mounting:** Incorrect volume mount configuration in pods

### **Adaptation for Different Environments:**

**For Cloud Providers:**
```bash
# AWS EKS
provisioner: ebs.csi.aws.com

# Google GKE  
provisioner: pd.csi.storage.gke.io

# Azure AKS
provisioner: disk.csi.azure.com
```

**For Local/Development:**
```bash
# Local path provisioner
provisioner: kubernetes.io/no-provisioner
# or rancher.io/local-path for k3s/kind
```

### **Success Indicators:**
- StorageClass created with appropriate parameters
- PVCs automatically trigger PV creation and binding
- Applications successfully mount and use dynamic volumes
- Data persists across pod restarts
- Storage monitoring shows healthy usage

### **Extension Scenarios:**
- Add volume expansion demonstrations
- Include multiple storage tiers (SSD, HDD, etc.)
- Introduce storage quotas and limits
- Add backup/snapshot functionality
- Include multi-AZ storage scenarios

---

*📊 **Reality Check:** This scenario mirrors actual data analytics platforms where dynamic storage provisioning eliminates the operational overhead of manual volume management while ensuring data persistence and scalability.*
