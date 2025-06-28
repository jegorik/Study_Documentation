# W03: Pod Placement Strategy

> **🎯 Exam Objectives:** Workloads & Scheduling (15% weight) - Understand how resource limits can affect pod scheduling  
> **⚡ Scenario:** E-commerce platform needs intelligent pod placement for Black Friday traffic surge  
> **⏱️ Time Limit:** 20 minutes  
> **💼 Difficulty:** Intermediate  

---

## 📋 Business Scenario

You're the Platform Engineer at **ShopFlow**, a major e-commerce platform preparing for Black Friday - your biggest traffic day of the year. Your infrastructure team needs to strategically place workloads across a heterogeneous cluster to ensure optimal performance, resource utilization, and fault tolerance.

The challenge: Your cluster has different node types (CPU-optimized, memory-optimized, GPU nodes) and you need to ensure that the right workloads land on the right nodes while maintaining high availability across availability zones.

**Your Mission:** Implement sophisticated pod placement strategies using node selectors, affinity rules, taints, and tolerations to create a resilient, performance-optimized deployment architecture.

**Business Impact:**
- 🛒 **10x traffic spike** expected (normal: 10k concurrent users → Black Friday: 100k+)
- 💰 **$2M+ revenue** at stake during peak 6-hour window
- 🚀 **Sub-200ms response times** required for checkout API
- 🌍 **99.99% availability** SLA commitment to customers
- 📊 **Cost optimization** - right workloads on right (cheaper) nodes

---

## 🎯 Lab Objectives

By completing this lab, you will master:

1. **Node Selection** - Strategic placement using nodeSelector and node affinity
2. **Pod Affinity/Anti-Affinity** - Co-location and separation strategies  
3. **Taints and Tolerations** - Dedicated node pools and workload isolation
4. **Resource-Based Scheduling** - CPU/memory-aware placement decisions
5. **Multi-Zone Deployment** - High availability across failure domains

---

## 🔧 Lab Tasks

### **Task 1: Infrastructure Assessment (3 minutes)**
Before implementing placement strategies, understand your cluster topology and node capabilities.

**Your Actions:**
1. Examine available nodes and their labels/taints
2. Identify node types: CPU-optimized, memory-optimized, GPU nodes
3. Check current resource utilization across nodes
4. Verify availability zone distribution

**Expected CKA Skills:**
- `kubectl get nodes --show-labels`
- `kubectl describe nodes`
- `kubectl top nodes`
- Understanding node selectors and affinity

**Success Criteria:**
- [ ] Document 3 different node types in cluster
- [ ] Identify nodes in different availability zones
- [ ] Understand current resource allocation patterns

---

### **Task 2: Frontend Web Server Placement (4 minutes)**
Deploy frontend web servers that need to be distributed across zones for high availability but avoid GPU nodes to save costs.

**Scenario Details:**
- **Application:** `shopflow-frontend` 
- **Requirements:** 6 replicas across 3 availability zones (2 per zone)
- **Constraints:** Avoid expensive GPU nodes, prefer CPU-optimized nodes
- **Resources:** 200m CPU, 256Mi memory per pod

**Your Actions:**
1. Create a deployment with zone anti-affinity rules
2. Add node affinity to prefer CPU-optimized nodes
3. Ensure pods avoid GPU nodes using taints/tolerations
4. Verify even distribution across availability zones

**Expected CKA Skills:**
- Pod affinity and anti-affinity rules
- Node affinity (preferred vs required)
- Understanding taints and tolerations
- Zone-aware scheduling

**Success Criteria:**
- [ ] 6 frontend pods running
- [ ] Exactly 2 pods per availability zone
- [ ] No pods scheduled on GPU nodes
- [ ] Pods prefer CPU-optimized node types

---

### **Task 3: Database Workload Co-location (4 minutes)**
Deploy database pods that need to be co-located with their caching layer for optimal performance, but separated from each other for fault tolerance.

**Scenario Details:**
- **Applications:** `shopflow-database` and `shopflow-redis-cache`
- **Requirements:** Each DB pod must be co-located with exactly one Redis pod
- **Constraints:** DB pods must be on different nodes from each other
- **Resources:** DB (1000m CPU, 2Gi memory), Redis (500m CPU, 512Mi memory)

**Your Actions:**
1. Deploy Redis cache pods with specific labels
2. Deploy database pods with affinity to Redis pods
3. Configure anti-affinity between database pods
4. Ensure proper resource allocation

**Expected CKA Skills:**
- Pod-to-pod affinity rules
- Anti-affinity for fault tolerance
- Resource requests and limits
- Label-based scheduling

**Success Criteria:**
- [ ] 3 database pods on separate nodes
- [ ] Each DB pod co-located with a Redis pod
- [ ] No two DB pods on the same node
- [ ] All pods have proper resource allocation

---

### **Task 4: ML Workload on GPU Nodes (3 minutes)**
Deploy machine learning recommendation engine pods that require GPU resources and can tolerate the specialized node taints.

**Scenario Details:**
- **Application:** `shopflow-ml-recommendations`
- **Requirements:** Must run on GPU nodes only
- **Constraints:** GPU nodes are tainted to prevent regular workloads
- **Resources:** 2000m CPU, 4Gi memory, 1 GPU per pod

**Your Actions:**
1. Create deployment that targets GPU nodes specifically
2. Add tolerations for GPU node taints
3. Configure GPU resource requirements
4. Verify pods land only on GPU-enabled nodes

**Expected CKA Skills:**
- Node selectors for specific hardware
- Tolerations for tainted nodes
- GPU resource specifications
- Hardware-specific scheduling

**Success Criteria:**
- [ ] 2 ML pods running
- [ ] All pods scheduled on GPU nodes only
- [ ] Pods tolerate GPU node taints
- [ ] GPU resources properly allocated

---

### **Task 5: Priority and Preemption Setup (3 minutes)**
Configure priority classes to ensure critical workloads can preempt less important ones during resource contention.

**Scenario Details:**
- **High Priority:** Payment processing (`shopflow-payments`)
- **Medium Priority:** Frontend and database workloads
- **Low Priority:** ML recommendations (can be delayed)
- **Emergency:** Implement preemption policies

**Your Actions:**
1. Create priority classes for different workload tiers
2. Assign priority classes to existing deployments
3. Test preemption by creating resource pressure
4. Verify high-priority workloads displace low-priority ones

**Expected CKA Skills:**
- Priority class creation and assignment
- Understanding preemption policies
- Resource contention management
- Workload prioritization strategies

**Success Criteria:**
- [ ] 3 priority classes created (high, medium, low)
- [ ] All deployments assigned appropriate priorities
- [ ] Payment workloads can preempt ML workloads
- [ ] Cluster maintains critical service availability

---

### **Task 6: Validation and Troubleshooting (3 minutes)**
Verify your placement strategy works correctly and troubleshoot any scheduling issues.

**Your Actions:**
1. Verify all workloads are placed according to strategy
2. Test node failure scenarios and pod redistribution  
3. Check resource utilization across node types
4. Validate high availability requirements are met

**Expected CKA Skills:**
- Deployment validation and testing
- Node failure simulation
- Resource monitoring and optimization
- Troubleshooting scheduling issues

**Success Criteria:**
- [ ] All pods scheduled according to placement rules
- [ ] Workloads survive single node failures
- [ ] Resource utilization optimized across node types
- [ ] High availability maintained across zones

---

## 🏗️ Environment Setup

```bash
# Quick cluster setup for this lab
# Note: In real exam, cluster will be pre-configured

# Label nodes for different types
kubectl label nodes node-1 node-type=cpu-optimized
kubectl label nodes node-2 node-type=cpu-optimized  
kubectl label nodes node-3 node-type=memory-optimized
kubectl label nodes node-4 node-type=memory-optimized
kubectl label nodes node-5 node-type=gpu-enabled
kubectl label nodes node-6 node-type=gpu-enabled

# Label nodes for availability zones
kubectl label nodes node-1 node-2 topology.kubernetes.io/zone=us-west-2a
kubectl label nodes node-3 node-4 topology.kubernetes.io/zone=us-west-2b
kubectl label nodes node-5 node-6 topology.kubernetes.io/zone=us-west-2c

# Taint GPU nodes to prevent regular workloads
kubectl taint nodes node-5 node-6 hardware=gpu:NoSchedule
```

---

## 📚 Key Commands Reference

### Node Information
```bash
# View node labels and taints
kubectl get nodes --show-labels
kubectl describe node <node-name>
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

# Check resource utilization
kubectl top nodes
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

### Pod Placement
```bash
# Check pod placement
kubectl get pods -o wide
kubectl get pods --field-selector spec.nodeName=<node-name>

# Describe scheduling decisions
kubectl describe pod <pod-name>
kubectl get events --sort-by='.metadata.creationTimestamp'
```

### Priority Classes
```bash
# List priority classes
kubectl get priorityclasses
kubectl describe priorityclass <priority-class-name>

# Check pod priorities
kubectl get pods -o custom-columns=NAME:.metadata.name,PRIORITY:.spec.priority
```

---

## 🔍 Troubleshooting Guide

### Common Issues

**Pod Stuck in Pending State**
```bash
# Check scheduling constraints
kubectl describe pod <pod-name>
kubectl get events --field-selector involvedObject.name=<pod-name>

# Common causes:
# - Node selector doesn't match any nodes
# - Insufficient resources on target nodes  
# - Taint/toleration mismatch
# - Affinity rules too restrictive
```

**Pods Not Distributed as Expected**
```bash
# Verify affinity/anti-affinity rules
kubectl get pod <pod-name> -o yaml | grep -A 10 affinity

# Check if nodes have required labels
kubectl get nodes --selector=<label-key>=<label-value>

# Validate zone distribution
kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,ZONE:.metadata.labels.topology\.kubernetes\.io/zone
```

**Resource Allocation Issues**
```bash
# Check node resource availability
kubectl describe nodes | grep -A 5 "Allocatable\|Allocated"

# Verify resource requests/limits
kubectl get pods -o custom-columns=NAME:.metadata.name,CPU-REQ:.spec.containers[0].resources.requests.cpu,MEM-REQ:.spec.containers[0].resources.requests.memory
```

---

## 🎯 Success Validation

### Final Verification Checklist

**Placement Strategy Validation:**
- [ ] Frontend pods distributed across 3 zones (2 per zone)
- [ ] Database pods on separate nodes, co-located with Redis
- [ ] ML workloads only on GPU nodes
- [ ] No regular workloads on GPU nodes
- [ ] Priority classes properly assigned

**High Availability Test:**
- [ ] Workloads survive single node failure
- [ ] Pods redistribute correctly after node recovery
- [ ] Critical services maintain availability

**Resource Optimization:**
- [ ] CPU-intensive workloads on CPU-optimized nodes
- [ ] Memory-intensive workloads on memory-optimized nodes
- [ ] GPU workloads only consume GPU resources when needed

**Troubleshooting Skills:**
- [ ] Can diagnose scheduling failures quickly
- [ ] Understand affinity/anti-affinity impact
- [ ] Can modify placement rules without disruption

---

## 🚀 Real-World Applications

### Production Use Cases

**E-commerce Platforms:**
- Separate payment processing from general web traffic
- Co-locate cache with application servers
- Distribute frontend across availability zones

**Machine Learning Pipelines:**
- Isolate GPU workloads on specialized nodes
- Use preemption for batch vs real-time workloads
- Optimize resource allocation by workload type

**Multi-Tenant Environments:**
- Isolate customer workloads using taints/tolerations
- Implement fair resource sharing with priorities
- Maintain security boundaries with node affinity

### Key Takeaways

1. **Strategic Placement** = Better performance + lower costs
2. **Affinity Rules** = Powerful but can over-constrain scheduling
3. **Resource Awareness** = Critical for efficient cluster utilization
4. **High Availability** = Always plan for node/zone failures
5. **Priorities** = Essential for managing resource contention

---

## 📈 Next Steps

**After This Lab:**
- Practice W04: Secrets Management Pipeline
- Review advanced scheduling concepts
- Study production placement strategies
- Explore custom schedulers and scheduling profiles

**Related CKA Topics:**
- Resource quotas and limits
- Horizontal Pod Autoscaler placement
- DaemonSet scheduling behavior
- Static pod placement patterns

---

*💡 **Pro Tip:** In the real exam, always check node labels and available resources before implementing placement strategies. A quick `kubectl get nodes --show-labels` can save you from scheduling headaches!*
