# T03 - Node Failure Recovery

> **🎯 Lab Focus:** Advanced node failure detection, recovery procedures, and workload migration  
> **⏱️ Estimated Time:** 25-30 minutes  
> **🏷️ Difficulty:** Advanced  
> **📋 Category:** Troubleshooting (30% Exam Weight)

---

## 🎬 Scenario: Critical Infrastructure Node Failures

You're the Lead DevOps Engineer at **CloudScale Financial Services**, a trading platform that processes millions of transactions per day. During a busy trading session, multiple worker nodes in your production Kubernetes cluster have started failing due to hardware issues and network problems. Customer-facing services are experiencing downtime, and your team needs to quickly recover the affected workloads while maintaining data consistency and service availability.

**Business Impact:**

- 🔴 **Trading Platform Down:** Revenue loss of $50,000 per minute
- 🟠 **Customer Authentication Failing:** User frustration mounting
- 🟡 **Data Processing Delayed:** Risk of compliance violations
- 📈 **SLA Breach:** 99.9% uptime commitment at risk

---

## 🏗️ Lab Environment Setup

### Initial Cluster State

```bash
# Expected cluster topology
NAME               STATUS   ROLES           AGE   VERSION
cloudscale-master  Ready    control-plane   45d   v1.28.2
worker-node-01     Ready    worker          45d   v1.28.2
worker-node-02     NotReady worker          45d   v1.28.2  # ⚠️ Node failure
worker-node-03     Ready    worker          45d   v1.28.2
worker-node-04     NotReady worker          45d   v1.28.2  # ⚠️ Node failure
worker-node-05     Ready    worker          45d   v1.28.2
```

### Prerequisites

- 6-node cluster (1 master, 5 workers)
- Multiple running applications with different failure tolerance
- Various storage types (emptyDir, hostPath, PVCs)
- Resource constraints and node affinity rules

---

## 📋 Tasks Overview

| Task | Objective | Est. Time | Difficulty |
|------|-----------|-----------|------------|
| 1 | Node Failure Detection & Assessment | 4 min | Intermediate |
| 2 | Emergency Workload Evacuation | 5 min | Advanced |
| 3 | Pod Rescheduling & Recovery | 5 min | Advanced |
| 4 | Storage Recovery Procedures | 6 min | Expert |
| 5 | Node Repair & Rejoin Process | 3 min | Intermediate |
| 6 | Service Availability Validation | 2 min | Intermediate |

---

## 🔧 Task 1: Node Failure Detection & Assessment (4 minutes)

### Objective

Identify failed nodes, analyze the root cause, and assess the impact on running workloads.

### Current Situation

Multiple alerts are firing in your monitoring system indicating node failures. You need to quickly assess the cluster state and identify which applications are affected.

### Your Mission

1. **Check overall cluster health and identify failed nodes**
2. **Analyze node conditions and failure reasons**
3. **List affected pods and their current states**
4. **Identify critical services that need immediate attention**

### Step-by-Step Instructions

#### 1.1 Assess Cluster Node Status

```bash
# Check all nodes with detailed status
kubectl get nodes -o wide

# Get detailed information about node conditions
kubectl describe nodes

# Focus on NotReady nodes to understand failure reasons
kubectl describe node worker-node-02
kubectl describe node worker-node-04
```

#### 1.2 Analyze Failed Node Details

```bash
# Check node events for failure indicators
kubectl get events --field-selector involvedObject.kind=Node --sort-by='.lastTimestamp'

# Check kubelet status on failed nodes (if accessible)
# This would typically be done via SSH in a real scenario
systemctl status kubelet  # On the affected nodes

# Check system resource usage that might have caused failure
kubectl top nodes  # If metrics-server is available
```

#### 1.3 Identify Affected Workloads

```bash
# List all pods on failed nodes
kubectl get pods -A -o wide --field-selector spec.nodeName=worker-node-02
kubectl get pods -A -o wide --field-selector spec.nodeName=worker-node-04

# Check pod statuses and reasons for failures
kubectl get pods -A --field-selector status.phase!=Running

# Look for pods stuck in Terminating state
kubectl get pods -A | grep Terminating
```

#### 1.4 Critical Service Assessment

```bash
# Check status of critical services
kubectl get deployments -A
kubectl get services -A
kubectl get daemonsets -A

# Identify services with reduced availability
kubectl get endpoints -A | grep -E "(<1|0>)"
```

### Expected Findings

- Two worker nodes (worker-node-02, worker-node-04) are NotReady
- Several pods are in Pending or Unknown state
- Some critical services have reduced replica counts
- DaemonSet pods on failed nodes need attention

### 🚨 Troubleshooting Tips

- **Node conditions** provide detailed failure reasons (DiskPressure, MemoryPressure, NetworkUnavailable)
- **Pod phases** indicate current state (Pending, Running, Succeeded, Failed, Unknown)
- **Events** show the sequence of failures and system responses

---

## 🚑 Task 2: Emergency Workload Evacuation (5 minutes)

### Objective

Safely evacuate workloads from failed nodes and prevent new pods from being scheduled on them.

### Current Situation

You have identified the failed nodes and affected workloads. Now you need to safely move critical applications to healthy nodes while preventing data loss.

### Your Mission

1. **Cordon failed nodes to prevent new scheduling**
2. **Drain nodes safely with proper handling of PodDisruptionBudgets**
3. **Force delete stuck pods if necessary**
4. **Monitor workload migration progress**

### Step-by-Step Instructions

#### 2.1 Cordon Failed Nodes

```bash
# Prevent new pods from being scheduled on failed nodes
kubectl cordon worker-node-02
kubectl cordon worker-node-04

# Verify nodes are cordoned
kubectl get nodes | grep SchedulingDisabled
```

#### 2.2 Drain Nodes Safely

```bash
# Start with graceful drain (respects PodDisruptionBudgets)
kubectl drain worker-node-02 --ignore-daemonsets --delete-emptydir-data --timeout=300s

kubectl drain worker-node-04 --ignore-daemonsets --delete-emptydir-data --timeout=300s

# Monitor drain progress
kubectl get pods -A -o wide | grep -E "(worker-node-02|worker-node-04)"
```

#### 2.3 Handle Stuck Pods

```bash
# If pods are stuck in Terminating state, force delete them
# First, try graceful deletion with shorter grace period
kubectl delete pod <stuck-pod-name> -n <namespace> --grace-period=0

# If still stuck, force delete (use with caution)
kubectl delete pod <stuck-pod-name> -n <namespace> --grace-period=0 --force

# For multiple stuck pods
kubectl get pods -A --field-selector status.phase=Unknown -o json | \
  jq '.items[] | select(.metadata.deletionTimestamp != null) | "\(.metadata.namespace) \(.metadata.name)"' -r | \
  xargs -n2 bash -c 'kubectl delete pod $1 -n $0 --grace-period=0 --force'
```

#### 2.4 Monitor Migration Progress

```bash
# Watch pods being rescheduled
kubectl get pods -A -o wide --watch

# Check that critical deployments maintain desired replica count
kubectl get deployments -A -o wide

# Monitor resource usage on remaining nodes
kubectl top nodes
```

### Lab Environment - Simulated Failures

#### Create Test Environment

```yaml
# trading-platform.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trading-engine
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: trading-engine
  template:
    metadata:
      labels:
        app: trading-engine
    spec:
      containers:
      - name: trading-engine
        image: nginx:alpine
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
      nodeSelector:
        node-type: worker
---
apiVersion: v1
kind: Service
metadata:
  name: trading-engine-svc
  namespace: production
spec:
  selector:
    app: trading-engine
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
# Add PodDisruptionBudget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: trading-engine-pdb
  namespace: production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: trading-engine
```

```bash
# Apply the configuration
kubectl create namespace production
kubectl apply -f trading-platform.yaml
```

### Expected Outcomes

- Failed nodes are cordoned and no new pods scheduled
- Existing pods are gracefully moved to healthy nodes
- Critical services maintain minimum availability through PDBs
- Resource usage is redistributed across remaining nodes

### 🚨 Troubleshooting Tips

- **PodDisruptionBudgets** may prevent draining if not enough replicas
- **Local storage** (emptyDir, hostPath) will be lost during pod migration
- **DaemonSets** require `--ignore-daemonsets` flag during drain
- **StatefulSets** need special consideration for data persistence

---

## 🔄 Task 3: Pod Rescheduling & Recovery (5 minutes)

### Objective

Ensure critical pods are properly rescheduled and recover services to full capacity.

### Current Situation

Pods have been evacuated from failed nodes, but some services may be running with reduced capacity. You need to verify proper rescheduling and scale up if necessary.

### Your Mission

1. **Verify all critical pods are running**
2. **Scale up deployments if needed**
3. **Check resource constraints on remaining nodes**
4. **Implement temporary scheduling constraints if needed**

### Step-by-Step Instructions

#### 3.1 Verify Pod Rescheduling

```bash
# Check that all pods are properly scheduled
kubectl get pods -A -o wide | grep -v Running

# Verify critical deployments have desired replica count
kubectl get deployments -A
kubectl get replicasets -A

# Check for any failed or pending pods
kubectl describe pods -A | grep -E "(Failed|Pending)"
```

#### 3.2 Scale Up Critical Services

```bash
# If any critical services are under-replicated, scale them up
kubectl scale deployment trading-engine -n production --replicas=4

# For other critical applications
kubectl scale deployment user-authentication -n production --replicas=3
kubectl scale deployment payment-processor -n production --replicas=2

# Monitor scaling progress
kubectl rollout status deployment/trading-engine -n production
```

#### 3.3 Resource Capacity Assessment

```bash
# Check resource availability on remaining nodes
kubectl describe nodes | grep -A 5 "Allocated resources"

# Look for resource pressure
kubectl top nodes
kubectl top pods -A --sort-by=cpu
kubectl top pods -A --sort-by=memory

# Check for resource quota constraints
kubectl get resourcequota -A
```

#### 3.4 Temporary Scheduling Adjustments

```yaml
# If needed, create anti-affinity rules to distribute critical pods
# critical-app-affinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trading-engine
  namespace: production
spec:
  replicas: 4
  selector:
    matchLabels:
      app: trading-engine
  template:
    metadata:
      labels:
        app: trading-engine
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - trading-engine
              topologyKey: kubernetes.io/hostname
      containers:
      - name: trading-engine
        image: nginx:alpine
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

```bash
# Apply anti-affinity rules
kubectl apply -f critical-app-affinity.yaml

# Monitor pod distribution
kubectl get pods -n production -o wide
```

### Expected Outcomes

- All critical pods are running on healthy nodes
- Services are back to full capacity
- Resource usage is balanced across remaining nodes
- Anti-affinity rules ensure better distribution

---

## 💾 Task 4: Storage Recovery Procedures (6 minutes)

### Objective

Handle storage-related issues caused by node failures and ensure data consistency.

### Current Situation

Some applications use persistent storage that was attached to the failed nodes. You need to assess storage impact and recover data access.

### Your Mission

1. **Identify affected PersistentVolumes and claims**
2. **Handle hostPath and local storage issues**
3. **Recover network-attached storage**
4. **Verify data integrity and availability**

### Step-by-Step Instructions

#### 4.1 Assess Storage Impact

```bash
# Check PersistentVolume status
kubectl get pv -o wide

# Check PersistentVolumeClaims
kubectl get pvc -A

# Look for volumes stuck in specific states
kubectl get pv | grep -E "(Failed|Pending|Released)"

# Check which pods had storage attachments on failed nodes
kubectl get pods -A -o yaml | grep -B 5 -A 5 "worker-node-0[24]"
```

#### 4.2 Handle Local Storage Issues

```bash
# Identify pods that used hostPath or local storage on failed nodes
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\t"}{.spec.volumes[*].hostPath.path}{"\n"}{end}' | grep -E "worker-node-0[24]"

# For hostPath volumes, data may be lost - check backup procedures
# Mark affected PVs for cleanup if they can't be recovered
kubectl patch pv <pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}'
```

#### 4.3 Recover Network Storage

```bash
# For network-attached storage (NFS, iSCSI, EBS), check connectivity
kubectl describe pv | grep -A 10 -B 5 "NFS\|iSCSI\|AWSElasticBlockStore"

# Check if PVs can be reattached to pods on healthy nodes
kubectl get pods -A -o yaml | grep -A 10 -B 10 persistentVolumeClaim

# Force detach volumes from failed nodes (cloud providers)
# This is typically done through cloud provider APIs
# For AWS EBS:
# aws ec2 detach-volume --volume-id vol-xxxxx --force
```

#### 4.4 Storage Recovery Examples

#### Database Storage Recovery

```yaml
# database-recovery.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-cluster
  namespace: production
spec:
  serviceName: postgres
  replicas: 3
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
          value: tradingdb
        - name: POSTGRES_USER
          value: dbuser
        - name: POSTGRES_PASSWORD
          value: dbpass123
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      # Add node affinity to avoid failed nodes
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: NotIn
                values:
                - worker-node-02
                - worker-node-04
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 50Gi
```

#### Shared Storage Recovery

```yaml
# shared-storage.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-data-recovery
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /exports/shared-data
    server: nfs-server.production.svc.cluster.local
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-data-claim
  namespace: production
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  volumeName: shared-data-recovery
```

#### 4.5 Verify Data Recovery

```bash
# Apply storage configurations
kubectl apply -f database-recovery.yaml
kubectl apply -f shared-storage.yaml

# Check storage binding
kubectl get pvc -A
kubectl get pv

# Verify pods can access data
kubectl exec -it postgres-cluster-0 -n production -- psql -U dbuser -d tradingdb -c "\l"

# Check application functionality
kubectl logs -f deployment/trading-engine -n production
```

### Expected Outcomes

- Network-attached storage is properly reattached
- Local storage issues are identified and handled
- Database connectivity is restored
- Data integrity is verified

### 🚨 Troubleshooting Tips

- **Local storage** (hostPath, emptyDir) is lost when nodes fail
- **Network storage** usually survives but may need manual detachment/reattachment
- **StatefulSets** require careful handling to maintain data consistency
- **Backup verification** is crucial for data recovery validation

---

## 🔧 Task 5: Node Repair & Rejoin Process (3 minutes)

### Objective

Simulate node repair and safely bring recovered nodes back into the cluster.

### Current Situation

Infrastructure team has resolved the hardware issues on the failed nodes. You need to bring them back online and integrate them into the cluster safely.

### Your Mission

1. **Verify node health before rejoining**
2. **Uncordon recovered nodes**
3. **Gradually rebalance workloads**
4. **Monitor cluster stability**

### Step-by-Step Instructions

#### 5.1 Pre-Rejoin Health Checks

```bash
# Simulate node recovery (in real scenario, this involves hardware fixes)
# Check node status
kubectl get nodes

# For this lab, simulate nodes coming back online
# In reality, you'd verify hardware, network, and kubelet status
# Check kubelet status
systemctl status kubelet  # On recovered nodes

# Verify node can communicate with cluster
kubectl get nodes worker-node-02 -o yaml | grep -A 5 conditions
```

#### 5.2 Uncordon Recovered Nodes

```bash
# Bring nodes back into scheduling
kubectl uncordon worker-node-02
kubectl uncordon worker-node-04

# Verify nodes are ready for scheduling
kubectl get nodes
kubectl describe nodes worker-node-02 | grep -A 5 "Conditions"
```

#### 5.3 Gradual Workload Rebalancing

```bash
# Don't force immediate rebalancing - let natural scheduling occur
# Monitor new pod placement
kubectl get pods -A -o wide --watch

# If needed, trigger rebalancing by rolling restart
kubectl rollout restart deployment/trading-engine -n production

# Check resource distribution
kubectl top nodes
```

#### 5.4 Cluster Health Validation

```bash
# Verify all nodes are healthy
kubectl get nodes
kubectl get pods -A | grep -v Running

# Check system pods
kubectl get pods -n kube-system

# Verify DaemonSets are running on all nodes
kubectl get daemonsets -A -o wide
```

### Expected Outcomes

- Recovered nodes rejoin cluster successfully
- New pods are scheduled on recovered nodes
- Cluster capacity is restored
- System stability is maintained

---

## ✅ Task 6: Service Availability Validation (2 minutes)

### Objective

Confirm all services are fully operational and performance has returned to normal.

### Current Situation

Nodes are recovered and workloads are rebalanced. You need to validate that all critical services are operating normally and SLA requirements are met.

### Your Mission

1. **Test service endpoints and connectivity**
2. **Verify application functionality**
3. **Check performance metrics**
4. **Document incident and lessons learned**

### Step-by-Step Instructions

#### 6.1 Service Endpoint Testing

```bash
# Test critical service endpoints
kubectl get services -A
kubectl get endpoints -A | grep -v "<none>"

# Test internal connectivity
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -- sh
# Inside the test pod:
curl trading-engine-svc.production.svc.cluster.local
curl user-auth-svc.production.svc.cluster.local
curl payment-svc.production.svc.cluster.local
```

#### 6.2 Application Health Checks

```bash
# Check deployment health
kubectl get deployments -A
kubectl rollout status deployment/trading-engine -n production

# Verify readiness and liveness probes
kubectl describe pods -n production | grep -A 10 "Readiness\|Liveness"

# Check application logs for errors
kubectl logs -f deployment/trading-engine -n production --tail=50
```

#### 6.3 Performance Validation

```bash
# Monitor resource usage
kubectl top nodes
kubectl top pods -A --sort-by=cpu

# Check for any resource warnings
kubectl get events --sort-by='.lastTimestamp' | grep -i warning

# Verify HPA is functioning
kubectl get hpa -A
```

#### 6.4 Incident Documentation

```bash
# Generate incident report
echo "Node Failure Recovery Incident Report" > incident_report.txt
echo "=================================" >> incident_report.txt
echo "Date: $(date)" >> incident_report.txt
echo "Failed Nodes: worker-node-02, worker-node-04" >> incident_report.txt
echo "Duration: Approximately 30 minutes" >> incident_report.txt
echo "Impact: Reduced capacity, some service degradation" >> incident_report.txt
echo "Resolution: Node repair and workload migration" >> incident_report.txt
echo "" >> incident_report.txt
kubectl get events --sort-by='.lastTimestamp' >> incident_report.txt
```

### Expected Outcomes

- All services respond correctly
- Performance metrics are within normal ranges
- No error logs or warnings
- Complete incident documentation

---

## 🎯 Success Criteria

### ✅ Must Complete (Pass)

- [ ] Identified failed nodes and affected workloads
- [ ] Successfully evacuated pods from failed nodes
- [ ] Restored critical services to full capacity
- [ ] Handled storage recovery appropriately
- [ ] Brought recovered nodes back online safely
- [ ] Validated complete service restoration

### 🌟 Excellence Indicators (Distinction)

- [ ] Minimal service downtime during recovery
- [ ] Proper use of PodDisruptionBudgets
- [ ] Efficient resource rebalancing
- [ ] Complete data integrity maintenance
- [ ] Comprehensive incident documentation
- [ ] Implemented improvements for future failures

### ⚡ Speed Benchmarks

- **Target Time:** 25 minutes
- **Expert Time:** 20 minutes
- **Detection Phase:** < 5 minutes
- **Recovery Phase:** < 15 minutes
- **Validation Phase:** < 5 minutes

---

## 🔍 Post-Lab Review

### Key Learning Outcomes

1. **Node Failure Detection:** Systematic approach to identifying and assessing node failures
2. **Workload Migration:** Safe evacuation and rescheduling procedures
3. **Storage Recovery:** Handling different storage types during node failures
4. **Cluster Resilience:** Building and maintaining resilient cluster architectures

### Real-World Applications

- **Production Outages:** Quick response to hardware failures
- **Maintenance Windows:** Planned node maintenance procedures
- **Disaster Recovery:** Node-level disaster recovery procedures
- **Capacity Planning:** Understanding impact of node failures on capacity

### Common Pitfalls & Solutions

| Problem | Symptom | Solution |
|---------|---------|----------|
| Stuck pods in Terminating | Pods won't delete gracefully | Force delete with `--grace-period=0 --force` |
| PDB prevents draining | Drain command hangs | Adjust PDB settings or scale up replicas |
| Local storage data loss | Applications fail to start | Implement proper backup/restore procedures |
| Resource pressure | New pods stay in Pending | Monitor and plan for reduced capacity scenarios |

### Exam Tips

- **Practice kubectl drain commands** with various flags
- **Understand PodDisruptionBudget** behavior during maintenance
- **Know storage recovery procedures** for different volume types
- **Master node troubleshooting** using describe and events

---

## 📚 Additional Resources

### Kubernetes Documentation

- [Node Management](https://kubernetes.io/docs/concepts/architecture/nodes/)
- [Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
- [Pod Disruption Budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)

### Troubleshooting Guides

- [Cluster Troubleshooting](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)
- [Node Troubleshooting](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/#looking-at-logs)

### Best Practices

- [Production Readiness](https://kubernetes.io/docs/concepts/cluster-administration/production-environment/)
- [Cluster Management](https://kubernetes.io/docs/concepts/cluster-administration/)

---

## 🚀 Next Steps

### Immediate Actions

1. **Practice variations** with different failure scenarios
2. **Automate recovery procedures** using scripts
3. **Test with real applications** in your environment
4. **Document your recovery playbooks**

### Advanced Scenarios to Explore

- **T05 - Certificate Expiry Emergency:** Handle certificate-related failures
- **T07 - Multi-Component Cluster Failure:** Complex disaster scenarios
- **A07 - Disaster Recovery Simulation:** Full cluster recovery procedures

### Production Preparation

- Implement **monitoring and alerting** for node health
- Create **automated recovery scripts** for common scenarios
- Establish **incident response procedures** for your team
- Practice **regular disaster recovery drills**

---

*🎯 "In production, it's not if nodes will fail, but when. Being prepared with tested procedures is the difference between a minor incident and a major outage."*

**Lab Complete!** 🎉 You've successfully demonstrated advanced node failure recovery skills essential for CKA certification and production operations.
