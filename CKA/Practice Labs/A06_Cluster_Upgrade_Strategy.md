# A06 - Cluster Upgrade Strategy

> **📋 Lab Category:** Architecture  
> **⏱️ Estimated Time:** 30-35 minutes  
> **🎯 Difficulty:** Advanced  
> **🔗 Exam Objectives:** Cluster Architecture, Installation & Configuration, Maintenance

---

## 🎬 Real-World Scenario

### Background Context

You're the Lead Platform Engineer at **CloudNative Corp**, managing a critical production Kubernetes cluster running 200+ microservices for a financial trading platform. The cluster is currently on Kubernetes v1.26.0, but security vulnerabilities and new feature requirements mandate an urgent upgrade to v1.28.2. The platform processes $50M in transactions daily, requiring zero-downtime upgrade execution.

**Your Role:** Lead Platform Engineer (Cluster Operations)  
**Situation:** Critical security-driven cluster upgrade with zero-downtime requirement  
**Urgency:** Security compliance deadline in 48 hours, trading cannot be interrupted  

### The Challenge

Execute a production-grade cluster upgrade across control plane and worker nodes while maintaining service availability, data integrity, and rollback capability. You must handle upgrade complications, validate cluster health, and ensure business continuity.

---

## 🎯 Learning Objectives

By completing this lab, you will:

- [ ] **Primary Skill:** Master production Kubernetes cluster upgrade procedures
- [ ] **Secondary Skills:** Control plane upgrade, worker node management, rollback strategies
- [ ] **Real-world Application:** Zero-downtime upgrade execution and risk mitigation
- [ ] **Exam Relevance:** Cluster upgrade scenarios (critical CKA exam topic)

---

## 🏗️ Lab Environment Setup

### Initial Cluster State

```bash
# CURRENT - Kubernetes v1.26.0 cluster requiring upgrade
CLUSTER INFO:
- Control Plane: 3 nodes (HA setup) - v1.26.0
- Worker Nodes: 5 nodes - v1.26.0
- Critical workloads: Trading API, Payment Service, User Management
- Target Version: v1.28.2
- Upgrade Path: v1.26.0 → v1.27.4 → v1.28.2 (staged approach)

BUSINESS CONSTRAINTS:
- Zero downtime requirement
- Financial regulations compliance
- Rollback capability within 30 minutes
- Performance monitoring throughout upgrade
```

---

## 📋 Tasks Overview

| Task | Objective | Est. Time | Difficulty |
|------|-----------|-----------|------------|
| 1 | Pre-Upgrade Assessment & Planning | 7 min | Advanced |
| 2 | Control Plane Upgrade (1st Node) | 8 min | Expert |
| 3 | Control Plane HA Upgrade | 6 min | Advanced |
| 4 | Worker Node Rolling Upgrade | 8 min | Advanced |
| 5 | Post-Upgrade Validation & Monitoring | 6 min | Intermediate |

---

## 🔍 Task 1: Pre-Upgrade Assessment & Planning (7 minutes)

### Objective

Conduct comprehensive pre-upgrade assessment, create backup strategy, and validate upgrade readiness.

### Your Mission

Assess cluster health, backup critical components, and prepare upgrade execution plan.

### Step-by-Step Instructions

#### 1.1 Current Cluster Health Assessment

```bash
# Check current cluster version and health
kubectl version --short
kubectl get nodes -o wide
kubectl get pods -A | grep -v Running | grep -v Completed

# Verify etcd cluster health
kubectl get pods -n kube-system | grep etcd
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/peer.crt \
  --key=/etc/kubernetes/pki/etcd/peer.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  endpoint health

# Check cluster resource utilization
kubectl top nodes
kubectl top pods -A | head -20
```

#### 1.2 Critical Workload Inventory

```bash
# Identify critical workloads and their distribution
kubectl get deployments -A -o wide
kubectl get statefulsets -A -o wide
kubectl get daemonsets -A -o wide

# Check PodDisruptionBudgets
kubectl get pdb -A

# Document current service endpoints
kubectl get services -A | grep -E "(LoadBalancer|NodePort)"
kubectl get ingress -A
```

#### 1.3 Backup Critical Components

```bash
# Backup etcd data
sudo mkdir -p /opt/etcd-backup/$(date +%Y%m%d-%H%M%S)
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/peer.crt \
  --key=/etc/kubernetes/pki/etcd/peer.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  snapshot save /opt/etcd-backup/$(date +%Y%m%d-%H%M%S)/etcd-snapshot.db

# Backup cluster configuration
sudo cp -r /etc/kubernetes /opt/cluster-backup-$(date +%Y%m%d-%H%M%S)

# Export current cluster state
kubectl get all -A -o yaml > /tmp/cluster-state-backup-$(date +%Y%m%d-%H%M%S).yaml
kubectl get pv,pvc -A -o yaml > /tmp/storage-backup-$(date +%Y%m%d-%H%M%S).yaml
```

#### 1.4 Upgrade Planning and Validation

```bash
# Check available kubeadm versions
sudo apt update
apt-cache policy kubeadm | head -20

# Validate upgrade path
sudo kubeadm upgrade plan

# Check for deprecated APIs
kubectl api-resources --verbs=list --output=name | xargs -n 1 kubectl get --show-kind --ignore-not-found -A

# Document current addon versions
kubectl get pods -n kube-system -o yaml | grep image: | sort | uniq
```

### Lab Environment Setup

#### Create Critical Workloads for Testing

```yaml
# critical-workloads.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: trading-platform
---
apiVersion: v1
kind: Namespace
metadata:
  name: payment-service
---
# Trading API Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trading-api
  namespace: trading-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: trading-api
  template:
    metadata:
      labels:
        app: trading-api
    spec:
      containers:
      - name: api
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
---
# PodDisruptionBudget for trading API
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: trading-api-pdb
  namespace: trading-platform
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: trading-api
---
# Payment Service StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: payment-service
  namespace: payment-service
spec:
  serviceName: payment-service
  replicas: 2
  selector:
    matchLabels:
      app: payment-service
  template:
    metadata:
      labels:
        app: payment-service
    spec:
      containers:
      - name: payment
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
```

```bash
# Deploy critical workloads
kubectl apply -f critical-workloads.yaml

# Verify deployment
kubectl get pods -n trading-platform
kubectl get pods -n payment-service
kubectl get pdb -A
```

---

## 🔧 Task 2: Control Plane Upgrade (1st Node) (8 minutes)

### Objective

Upgrade the first control plane node while maintaining cluster availability through HA configuration.

### Your Mission

Execute control plane upgrade on primary node with health monitoring and rollback readiness.

### Step-by-Step Instructions

#### 2.1 Prepare Control Plane for Upgrade

```bash
# Identify control plane nodes
kubectl get nodes -l node-role.kubernetes.io/control-plane

# Drain first control plane node (exclude this step in single-node clusters)
CONTROL_PLANE_1=$(kubectl get nodes -l node-role.kubernetes.io/control-plane -o jsonpath='{.items[0].metadata.name}')
echo "Upgrading control plane node: $CONTROL_PLANE_1"

# For multi-node control plane, drain the node
kubectl drain $CONTROL_PLANE_1 --ignore-daemonsets --delete-emptydir-data --force

# Check that other control plane nodes are handling load
kubectl get pods -n kube-system | grep -E "(api|etcd|scheduler|controller)"
```

#### 2.2 Upgrade kubeadm and kubelet on Control Plane

```bash
# Update package repository
sudo apt update

# Upgrade kubeadm to target version (staged approach: v1.27.4 first)
TARGET_VERSION="1.27.4"
sudo apt-mark unhold kubeadm
sudo apt install -y kubeadm=${TARGET_VERSION}-00
sudo apt-mark hold kubeadm

# Verify kubeadm version
kubeadm version

# Plan the upgrade
sudo kubeadm upgrade plan v${TARGET_VERSION}
```

#### 2.3 Execute Control Plane Upgrade

```bash
# Apply the upgrade to first control plane node
sudo kubeadm upgrade apply v${TARGET_VERSION} --yes

# Monitor upgrade progress
kubectl get nodes
kubectl get pods -n kube-system

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt install -y kubelet=${TARGET_VERSION}-00 kubectl=${TARGET_VERSION}-00
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Verify kubelet is running
sudo systemctl status kubelet
```

#### 2.4 Validate First Control Plane Upgrade

```bash
# Uncordon the control plane node
kubectl uncordon $CONTROL_PLANE_1

# Verify node is ready and upgraded
kubectl get nodes -o wide

# Check cluster component health
kubectl get pods -n kube-system
kubectl get componentstatuses 2>/dev/null || echo "componentstatuses deprecated in newer versions"

# Test basic cluster functionality
kubectl get pods -A | head -10
kubectl create deployment test-upgrade --image=nginx --replicas=1
kubectl get pods -l app=test-upgrade
kubectl delete deployment test-upgrade
```

---

## 🏗️ Task 3: Control Plane HA Upgrade (6 minutes)

### Objective

Complete the remaining control plane nodes upgrade while maintaining high availability.

### Your Mission

Upgrade remaining control plane nodes in rolling fashion with continuous availability validation.

### Step-by-Step Instructions

#### 3.1 Upgrade Additional Control Plane Nodes

```bash
# Get list of remaining control plane nodes
CONTROL_PLANE_NODES=$(kubectl get nodes -l node-role.kubernetes.io/control-plane -o jsonpath='{.items[*].metadata.name}')
echo "All control plane nodes: $CONTROL_PLANE_NODES"

# For each remaining control plane node (if HA setup exists)
for node in $(kubectl get nodes -l node-role.kubernetes.io/control-plane -o jsonpath='{.items[1:].metadata.name}'); do
  echo "Processing control plane node: $node"
  
  # Note: In real environment, you would SSH to each node and run these commands
  # For simulation, we'll show the process
  
  echo "Would run on $node:"
  echo "  1. sudo apt update"
  echo "  2. sudo apt-mark unhold kubeadm"
  echo "  3. sudo apt install -y kubeadm=${TARGET_VERSION}-00"
  echo "  4. sudo kubeadm upgrade node"
  echo "  5. sudo apt install -y kubelet=${TARGET_VERSION}-00 kubectl=${TARGET_VERSION}-00"
  echo "  6. sudo systemctl daemon-reload && sudo systemctl restart kubelet"
done
```

#### 3.2 Validate Control Plane Cluster Health

```bash
# Check all control plane nodes are upgraded
kubectl get nodes -l node-role.kubernetes.io/control-plane -o wide

# Verify etcd cluster health
kubectl get pods -n kube-system | grep etcd
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/peer.crt \
  --key=/etc/kubernetes/pki/etcd/peer.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  endpoint health

# Check API server availability
kubectl cluster-info
kubectl get --raw /healthz
```

#### 3.3 Critical Workload Health Check

```bash
# Verify critical workloads are still running
kubectl get pods -n trading-platform
kubectl get pods -n payment-service

# Check service connectivity
kubectl get services -A
kubectl get endpoints -A | grep -E "(trading|payment)"

# Test API functionality
kubectl get pods --all-namespaces | wc -l
kubectl get nodes | grep Ready | wc -l
```

---

## 🖥️ Task 4: Worker Node Rolling Upgrade (8 minutes)

### Objective

Perform rolling upgrade of worker nodes while maintaining workload availability through proper draining and scheduling.

### Your Mission

Upgrade worker nodes in controlled manner with workload redistribution and health validation.

### Step-by-Step Instructions

#### 4.1 Prepare Worker Node Upgrade Strategy

```bash
# List all worker nodes
kubectl get nodes -l '!node-role.kubernetes.io/control-plane'

# Check pod distribution across worker nodes
kubectl get pods -A -o wide | grep -v "kube-system" | awk '{print $8}' | sort | uniq -c

# Verify PodDisruptionBudgets are in place
kubectl get pdb -A
```

#### 4.2 Rolling Worker Node Upgrade

```bash
# Upgrade worker nodes one by one
WORKER_NODES=$(kubectl get nodes -l '!node-role.kubernetes.io/control-plane' -o jsonpath='{.items[*].metadata.name}')

for worker in $WORKER_NODES; do
  echo "=== Upgrading worker node: $worker ==="
  
  # Drain the worker node
  kubectl drain $worker --ignore-daemonsets --delete-emptydir-data --force
  
  # Wait for pods to be rescheduled
  sleep 30
  
  # Check that critical workloads are still available
  kubectl get pods -n trading-platform | grep Running | wc -l
  kubectl get pods -n payment-service | grep Running | wc -l
  
  # Simulate worker node upgrade (in real scenario, SSH to node and run these)
  echo "On $worker, would run:"
  echo "  sudo apt update"
  echo "  sudo apt-mark unhold kubeadm kubelet kubectl"
  echo "  sudo apt install -y kubeadm=${TARGET_VERSION}-00 kubelet=${TARGET_VERSION}-00 kubectl=${TARGET_VERSION}-00"
  echo "  sudo apt-mark hold kubeadm kubelet kubectl"
  echo "  sudo kubeadm upgrade node"
  echo "  sudo systemctl daemon-reload && sudo systemctl restart kubelet"
  
  # Simulate upgrade completion and uncordon
  sleep 10
  kubectl uncordon $worker
  
  # Wait for node to be ready
  kubectl wait --for=condition=Ready node/$worker --timeout=300s
  
  # Verify node is upgraded and ready
  kubectl get node $worker -o wide
  
  echo "Worker node $worker upgrade completed"
  echo "=================================="
done
```

#### 4.3 Verify Worker Node Upgrades

```bash
# Check all nodes are on target version
kubectl get nodes -o wide

# Verify all pods are running correctly
kubectl get pods -A | grep -v Running | grep -v Completed

# Check workload distribution is healthy
kubectl get pods -o wide -A | grep -E "(trading|payment)" | awk '{print $8}' | sort | uniq -c

# Validate critical services
kubectl get deployments -n trading-platform
kubectl get statefulsets -n payment-service
```

---

## ✅ Task 5: Post-Upgrade Validation & Monitoring (6 minutes)

### Objective

Comprehensive post-upgrade validation, performance testing, and monitoring setup for the upgraded cluster.

### Your Mission

Validate cluster functionality, test critical workflows, and establish ongoing monitoring.

### Step-by-Step Instructions

#### 5.1 Comprehensive Cluster Validation

```bash
# Verify cluster version upgrade
kubectl version --short
kubectl get nodes -o wide

# Check all system pods are healthy
kubectl get pods -n kube-system
kubectl get pods -A | grep -v Running | grep -v Completed

# Validate core cluster functions
kubectl cluster-info
kubectl get --raw /healthz
kubectl get --raw /readyz
```

#### 5.2 Workload Functionality Testing

```bash
# Test deployment operations
kubectl create deployment upgrade-test --image=nginx:alpine --replicas=3
kubectl scale deployment upgrade-test --replicas=5
kubectl get pods -l app=upgrade-test

# Test service connectivity
kubectl expose deployment upgrade-test --port=80 --type=ClusterIP
kubectl get endpoints upgrade-test

# Test pod-to-pod networking
kubectl run test-pod --image=busybox --rm -it --restart=Never -- nslookup kubernetes.default

# Cleanup test resources
kubectl delete deployment upgrade-test
kubectl delete service upgrade-test
```

#### 5.3 Critical Business Workload Validation

```bash
# Test trading platform functionality
kubectl get pods -n trading-platform -o wide
kubectl logs -n trading-platform deployment/trading-api --tail=5

# Test payment service functionality
kubectl get pods -n payment-service -o wide
kubectl exec -n payment-service payment-service-0 -- redis-cli ping

# Verify persistent storage is intact
kubectl get pv,pvc -A
kubectl describe pvc -A | grep -A 3 -B 3 "Bound"
```

#### 5.4 Performance and Security Validation

```bash
# Check resource utilization post-upgrade
kubectl top nodes
kubectl top pods -A | head -20

# Validate security policies are intact
kubectl get networkpolicies -A
kubectl get podsecuritypolicies 2>/dev/null || echo "PSPs deprecated, checking Pod Security Standards"

# Check addon compatibility
kubectl get pods -n kube-system | grep -E "(dns|proxy|cni)"

# Verify ingress and load balancer functionality
kubectl get ingress -A
kubectl get services -A | grep LoadBalancer
```

#### 5.5 Monitoring and Alerting Setup

```bash
# Create upgrade monitoring script
cat << 'EOF' | tee /tmp/post-upgrade-monitor.sh
#!/bin/bash
echo "=== Post-Upgrade Cluster Monitoring ==="
echo "Date: $(date)"
echo ""
echo "=== Cluster Version ==="
kubectl version --short
echo ""
echo "=== Node Status ==="
kubectl get nodes -o wide
echo ""
echo "=== Critical Workload Health ==="
kubectl get pods -n trading-platform
kubectl get pods -n payment-service
echo ""
echo "=== Resource Utilization ==="
kubectl top nodes 2>/dev/null || echo "Metrics server not available"
echo ""
echo "=== System Pod Health ==="
kubectl get pods -n kube-system | grep -v Running | grep -v Completed || echo "All system pods healthy"
echo ""
echo "=== API Server Health ==="
kubectl get --raw /healthz
echo ""
echo "=== Monitoring completed at $(date) ==="
EOF

chmod +x /tmp/post-upgrade-monitor.sh

# Run initial monitoring check
/tmp/post-upgrade-monitor.sh

# Set up periodic monitoring (would add to cron in production)
echo "Consider adding to crontab: */15 * * * * /tmp/post-upgrade-monitor.sh >> /var/log/k8s-health.log"
```

---

## 🎯 Expected Outcomes

Upon completion, you should have:

- ✅ Cluster successfully upgraded from v1.26.0 to v1.27.4
- ✅ All control plane nodes running target version
- ✅ All worker nodes running target version
- ✅ Critical workloads maintaining availability throughout upgrade
- ✅ Complete backup and rollback capability established
- ✅ Post-upgrade monitoring and validation procedures implemented

---

## 🚨 Emergency Rollback Procedures

### If Upgrade Fails on Control Plane

```bash
# Restore etcd from backup
sudo systemctl stop kubelet
sudo ETCDCTL_API=3 etcdctl snapshot restore /opt/etcd-backup/[timestamp]/etcd-snapshot.db

# Restore kubernetes configuration
sudo cp -r /opt/cluster-backup-[timestamp]/* /etc/kubernetes/

# Restart services
sudo systemctl start kubelet
```

### If Worker Node Upgrade Fails

```bash
# Isolate failed node
kubectl cordon [failed-node]
kubectl drain [failed-node] --ignore-daemonsets --force

# Rebuild node or restore from backup
# Rejoin cluster with previous version
```

---

## 📚 Knowledge Check

### Critical Understanding Questions

1. **Upgrade Strategy:** Why is a staged upgrade approach recommended?
2. **High Availability:** How does HA control plane ensure zero downtime?
3. **Rollback Planning:** What components need backup before upgrade?

### Advanced Scenarios

- How would you handle custom resource definitions during upgrade?
- What's the strategy for upgrading clusters with custom CNI plugins?
- How do you validate addon compatibility with new Kubernetes versions?

---

## 🔄 Production Best Practices

### Upgrade Automation Pipeline

```bash
# Consider implementing automated upgrade pipeline
echo "Best practices for production:"
echo "1. Test upgrades in staging environment first"
echo "2. Implement gradual rollout strategies"
echo "3. Use monitoring and alerting throughout process"
echo "4. Maintain detailed upgrade runbooks"
echo "5. Practice rollback procedures regularly"
```

---

*⚡ "Kubernetes cluster upgrades require careful planning and execution. Master these procedures for reliable production operations!" - CKA Success Tip*
