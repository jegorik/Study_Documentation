# T04: Resource Exhaustion Crisis

> **🎯 Exam Objectives:** Troubleshooting (30% weight) - Diagnose and resolve resource constraint issues  
> **⚡ Scenario:** CloudScale SaaS platform experiencing critical resource exhaustion during product launch  
> **⏱️ Time Limit:** 30 minutes  
> **💼 Difficulty:** Intermediate  

---

## 📋 Business Scenario

You're the DevOps Lead at **CloudScale**, a fast-growing SaaS platform for project management. Today is the official launch of your enterprise tier, and marketing has driven massive user signups. However, your Kubernetes cluster is experiencing severe performance degradation and pod failures due to resource exhaustion.

**Your Mission:** The CEO is getting live updates on system performance during the launch event. You have 30 minutes to identify and resolve resource bottlenecks before the launch turns into a public relations disaster.

**Business Impact:**
- 💰 **$250,000** in pre-paid enterprise subscriptions at risk
- 📰 **Tech press** covering the launch live with performance monitoring
- 📈 **IPO roadshow** next month depends on successful scaling demonstration
- 👥 **500+ enterprise customers** experiencing slowdowns and timeouts
- 💼 **Board meeting** in 45 minutes expecting launch success metrics

---

## 🎯 Lab Objectives

By completing this lab, you will master:

1. **Resource Monitoring and Analysis** - CPU, memory, and storage utilization patterns
2. **Resource Constraint Diagnosis** - Identifying bottlenecks and limits
3. **Pod Resource Management** - Requests, limits, and QoS classes
4. **Node Capacity Planning** - Understanding cluster resource allocation
5. **Performance Optimization** - Implementing resource efficiency improvements

---

## 🔧 Lab Tasks

### **Task 1: Emergency Resource Assessment (8 minutes)**
The platform is experiencing widespread performance issues. Quickly assess the resource situation across the cluster.

**Your Actions:**
1. Check overall cluster resource utilization (CPU, memory, storage)
2. Identify nodes experiencing resource pressure or exhaustion
3. Examine pods that are failing, pending, or being evicted
4. Analyze resource requests vs. actual usage patterns
5. Document the scope and severity of resource constraints

**Expected CKA Skills:**
- `kubectl top nodes`, `kubectl top pods`
- `kubectl describe nodes` for resource pressure analysis
- Pod resource analysis and utilization monitoring
- Node capacity and allocatable resource calculation
- Resource constraint pattern recognition

---

### **Task 2: Critical Pod Analysis (8 minutes)**
Focus on identifying which applications are consuming excessive resources and which are being starved.

**Your Actions:**
1. Identify pods with highest resource consumption relative to requests
2. Find pods that are being killed due to resource limits (OOMKilled)
3. Examine pending pods and determine why they can't be scheduled
4. Check for pods without resource requests/limits causing resource contention
5. Prioritize which resource issues are most critical for business operations

**Expected CKA Skills:**
- Pod resource consumption analysis
- OOM (Out of Memory) kill investigation
- Pod scheduling failure troubleshooting
- Resource request/limit configuration analysis
- QoS class understanding (Guaranteed, Burstable, BestEffort)

---

### **Task 3: Node Resource Management (8 minutes)**
Investigate node-level resource constraints and pressure indicators.

**Your Actions:**
1. Examine node conditions for memory pressure, disk pressure, and PID pressure
2. Calculate actual vs. allocatable resources on each node
3. Identify nodes approaching resource thresholds
4. Check for kubelet eviction policies and thresholds
5. Determine if additional nodes are needed or if resource rebalancing can help

**Expected CKA Skills:**
- Node condition monitoring and interpretation
- Resource allocatable vs. capacity understanding
- Kubelet eviction policy configuration
- Node resource pressure threshold analysis
- Cluster capacity planning calculations

---

### **Task 4: Immediate Resolution Implementation (6 minutes)**
Implement quick fixes to restore service performance and prevent further degradation.

**Your Actions:**
1. Apply appropriate resource requests and limits to resource-hungry pods
2. Scale down non-critical workloads to free up resources
3. Implement pod priorities to ensure critical services get resources first
4. Configure horizontal pod autoscaling if appropriate
5. Document emergency resource management procedures for future incidents

**Expected CKA Skills:**
- Resource request/limit configuration and application
- Workload scaling and priority management
- Pod priority class configuration
- HPA (Horizontal Pod Autoscaler) setup and tuning
- Emergency response documentation and procedures

---

## 💡 Hints & Tips

<details>
<summary>🆘 <strong>Can't see resource usage?</strong> Click for monitoring guidance...</summary>

**Resource Monitoring Commands:**
```bash
# Check cluster-wide resource usage
kubectl top nodes
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# Detailed node resource information
kubectl describe nodes | grep -A 15 "Allocated resources"

# Check for resource pressure
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="MemoryPressure")].status}{"\t"}{.status.conditions[?(@.type=="DiskPressure")].status}{"\n"}{end}'
```

**Key Concepts:**
- `kubectl top` requires metrics-server to be running
- Node conditions indicate resource pressure states
- Allocatable resources = Total capacity - System reserved
- Resource requests affect scheduling decisions
</details>

<details>
<summary>🔍 <strong>Pods failing or pending?</strong> Click for scheduling analysis...</summary>

**Pod Scheduling Troubleshooting:**
```bash
# Check pending pods and reasons
kubectl get pods --all-namespaces --field-selector=status.phase=Pending

# Examine scheduling failures
kubectl describe pod <pending-pod-name>

# Check events for resource-related issues
kubectl get events --sort-by=.metadata.creationTimestamp | grep -i "insufficient\|failed\|oom"

# Find OOMKilled pods
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].lastState.terminated.reason}{"\n"}{end}' | grep OOMKilled
```

**Common Resource Issues:**
- Insufficient CPU/memory for pod requests
- Node resource exhaustion preventing scheduling
- Missing resource limits causing resource contention
- OOM kills from containers exceeding memory limits
</details>

<details>
<summary>⚙️ <strong>Need to configure resources?</strong> Click for resource management patterns...</summary>

**Resource Configuration Examples:**
```yaml
# Pod with proper resource configuration
apiVersion: v1
kind: Pod
metadata:
  name: optimized-app
spec:
  containers:
  - name: app
    image: myapp:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

**Emergency Resource Management:**
```bash
# Quick resource limit application
kubectl patch deployment webapp --patch='
spec:
  template:
    spec:
      containers:
      - name: webapp
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"'

# Scale down non-critical workloads
kubectl scale deployment background-jobs --replicas=1

# Check resource usage after changes
kubectl top pods -l app=webapp
```
</details>

---

## ✅ Success Criteria

You've successfully completed this lab when:

- [ ] **Resource Bottlenecks Identified** - Clear understanding of cluster resource constraints
- [ ] **Critical Applications Stabilized** - Essential services have adequate resources
- [ ] **Performance Restored** - Response times and success rates improved
- [ ] **Resource Monitoring Implemented** - Ongoing visibility into resource usage
- [ ] **Prevention Measures Applied** - Resource limits and monitoring to prevent recurrence

---

## 🎓 Key Learning Outcomes

After completing this lab, you'll be expert at:

✅ **Resource Monitoring Mastery** - Comprehensive cluster resource analysis skills  
✅ **Constraint Diagnosis** - Identifying and analyzing resource bottlenecks  
✅ **Pod Resource Management** - Optimal request/limit configuration strategies  
✅ **Node Capacity Planning** - Understanding cluster scaling and capacity needs  
✅ **Performance Optimization** - Implementing resource efficiency improvements under pressure  

---

## 🔄 Next Steps

Ready for more resource management challenges? Try these related labs:

- **W02: Auto-scaling E-commerce Site** - Advanced HPA and VPA implementation
- **N04: DNS Resolution Debugging** - Resource impacts on DNS performance
- **T08: Performance Degradation Hunt** - Advanced performance troubleshooting
- **A03: HA Control Plane Setup** - Resource planning for highly available clusters

---

## 📚 Additional Resources

**CKA Exam Topics Covered:**
- Resource monitoring and troubleshooting
- Pod resource management and QoS
- Node capacity planning and management
- Performance optimization techniques

**Documentation Links:**
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Node Conditions](https://kubernetes.io/docs/concepts/architecture/nodes/#condition)

---

*💡 **Pro Tip:** Resource exhaustion is one of the most common production issues in Kubernetes. Having a systematic approach to resource monitoring and optimization is essential for maintaining reliable services at scale.*
