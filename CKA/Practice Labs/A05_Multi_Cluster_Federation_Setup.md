# A05 - Multi-Cluster Federation Setup

## 🎯 Lab Overview
**Scenario:** You're the platform architect for **GlobalTech Solutions**, managing a distributed microservices platform across 3 regions (US-East, EU-West, Asia-Pacific). The company needs to implement cross-cluster service discovery, unified resource management, and disaster recovery capabilities. Your mission is to establish cluster federation for seamless multi-region operations.

**Business Impact:** 
- **$5M+** infrastructure consolidation savings
- **99.99%** availability through multi-region redundancy
- Global service mesh enabling worldwide customer expansion

## 📋 Lab Details
- **Difficulty:** Advanced
- **Category:** Cluster Architecture (25% of CKA Exam)
- **Time Estimate:** 30-35 minutes
- **Prerequisites:** Advanced Kubernetes, networking, cluster administration

## 🚨 Tasks Overview

### Task 1: Multi-Cluster Environment Setup (8 minutes)
- Create simulated multi-cluster environment
- Configure cluster contexts and connectivity
- Establish cross-cluster networking foundation

### Task 2: Federation Control Plane Deployment (7 minutes)
- Deploy federation control plane components
- Configure cluster registration and discovery
- Validate federation API server functionality

### Task 3: Cross-Cluster Service Discovery (8 minutes)
- Implement federated service discovery
- Configure DNS for cross-cluster resolution
- Test service communication across clusters

### Task 4: Federated Resource Management (7 minutes)
- Deploy federated deployments and services
- Configure resource placement policies
- Implement cross-cluster load balancing

### Task 5: Disaster Recovery & Monitoring (5 minutes)
- Configure cluster failover mechanisms
- Implement federation monitoring
- Create operational runbooks

---

## 🛠 Task 1: Multi-Cluster Environment Setup

### Context
GlobalTech needs federated clusters for their worldwide infrastructure.

### Instructions

**Step 1: Create cluster contexts (simulate 3 clusters)**
```bash
# Create namespace-based cluster simulation
kubectl create namespace cluster-us-east
kubectl create namespace cluster-eu-west  
kubectl create namespace cluster-asia-pacific

# Label namespaces as cluster regions
kubectl label namespace cluster-us-east region=us-east cluster=primary
kubectl label namespace cluster-eu-west region=eu-west cluster=secondary
kubectl label namespace cluster-asia-pacific region=asia-pacific cluster=tertiary

# Create cluster-specific contexts
kubectl config set-context us-east --namespace=cluster-us-east
kubectl config set-context eu-west --namespace=cluster-eu-west
kubectl config set-context asia-pacific --namespace=cluster-asia-pacific
```

**Step 2: Deploy cluster infrastructure components**
```bash
# Deploy regional API gateways
for region in us-east eu-west asia-pacific; do
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: cluster-${region}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-gateway
      region: ${region}
  template:
    metadata:
      labels:
        app: api-gateway
        region: ${region}
    spec:
      containers:
      - name: gateway
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: CLUSTER_REGION
          value: ${region}
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-svc
  namespace: cluster-${region}
spec:
  selector:
    app: api-gateway
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF
done
```

**Step 3: Configure cross-cluster networking**
```bash
# Create network policies for cross-cluster communication
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-federation
  namespace: cluster-us-east
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          cluster: secondary
    - namespaceSelector:
        matchLabels:
          cluster: tertiary
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          cluster: secondary
    - namespaceSelector:
        matchLabels:
          cluster: tertiary
EOF

# Apply similar policies to other regions
for ns in cluster-eu-west cluster-asia-pacific; do
  kubectl apply -n $ns -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-federation
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from: []
  egress:
  - to: []
EOF
done
```

**Verification:**
- ✅ Three cluster namespaces created with proper labels
- ✅ Regional services deployed successfully
- ✅ Network policies enable cross-cluster communication

---

## 🛠 Task 2: Federation Control Plane Deployment

### Instructions

**Step 1: Deploy federation control plane**
```bash
# Create federation system namespace
kubectl create namespace federation-system

# Deploy federation controller
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: federation-controller
  namespace: federation-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: federation-controller
  template:
    metadata:
      labels:
        app: federation-controller
    spec:
      containers:
      - name: controller
        image: busybox:1.35
        command: ['sh', '-c', 'while true; do echo "Federation controller running"; sleep 30; done']
      serviceAccountName: federation-controller
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: federation-controller
  namespace: federation-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: federation-controller
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: federation-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: federation-controller
subjects:
- kind: ServiceAccount
  name: federation-controller
  namespace: federation-system
EOF
```

**Step 2: Register clusters with federation**
```bash
# Create cluster registry
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-registry
  namespace: federation-system
data:
  clusters.yaml: |
    clusters:
    - name: us-east
      endpoint: https://api.us-east.globaltech.com
      region: us-east
      zone: us-east-1a
      status: ready
    - name: eu-west
      endpoint: https://api.eu-west.globaltech.com
      region: eu-west
      zone: eu-west-1a
      status: ready
    - name: asia-pacific
      endpoint: https://api.asia-pacific.globaltech.com
      region: asia-pacific
      zone: asia-pacific-1a
      status: ready
EOF

# Create cluster health monitoring
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: cluster-monitor
  namespace: federation-system
spec:
  containers:
  - name: monitor
    image: busybox:1.35
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "=== Cluster Health Check $(date) ==="
        echo "US-East: $(kubectl get pods -n cluster-us-east --no-headers | wc -l) pods"
        echo "EU-West: $(kubectl get pods -n cluster-eu-west --no-headers | wc -l) pods" 
        echo "Asia-Pacific: $(kubectl get pods -n cluster-asia-pacific --no-headers | wc -l) pods"
        sleep 60
      done
EOF
```

**Verification:**
- ✅ Federation controller deployed and running
- ✅ Cluster registry configured with all regions
- ✅ Cluster monitoring active

---

## 🛠 Task 3: Cross-Cluster Service Discovery

### Instructions

**Step 1: Deploy federated DNS**
```bash
# Create federated DNS service
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: federated-dns
  namespace: federation-system
data:
  fed-dns.conf: |
    # Federated DNS Configuration
    us-east.globaltech.local -> cluster-us-east.svc.cluster.local
    eu-west.globaltech.local -> cluster-eu-west.svc.cluster.local
    asia-pacific.globaltech.local -> cluster-asia-pacific.svc.cluster.local
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: federated-dns
  namespace: federation-system
spec:
  replicas: 2
  selector:
    matchLabels:
      app: federated-dns
  template:
    metadata:
      labels:
        app: federated-dns
    spec:
      containers:
      - name: dns
        image: busybox:1.35
        command: ['sh', '-c', 'while true; do echo "Federated DNS serving"; sleep 30; done']
        volumeMounts:
        - name: config
          mountPath: /etc/dns
      volumes:
      - name: config
        configMap:
          name: federated-dns
EOF
```

**Step 2: Test cross-cluster service discovery**
```bash
# Create test pods in each region
for region in us-east eu-west asia-pacific; do
kubectl run test-$region --image=busybox:1.35 -n cluster-$region --restart=Never -- sleep 3600
done

# Test service discovery
kubectl exec -n cluster-us-east test-us-east -- nslookup api-gateway-svc.cluster-us-east
kubectl exec -n cluster-us-east test-us-east -- nslookup api-gateway-svc.cluster-eu-west.svc.cluster.local
```

**Verification:**
- ✅ Federated DNS deployed across regions
- ✅ Cross-cluster service resolution working

---

## 🛠 Task 4: Federated Resource Management

### Instructions

**Step 1: Deploy federated applications**
```bash
# Create federated deployment
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: global-web-app
  namespace: cluster-us-east
  annotations:
    federation.globaltech.com/replicate: "true"
    federation.globaltech.com/regions: "us-east,eu-west,asia-pacific"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: global-web-app
  template:
    metadata:
      labels:
        app: global-web-app
        federation: "true"
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: REGION
          value: "us-east"
EOF

# Replicate to other regions
for region in eu-west asia-pacific; do
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: global-web-app
  namespace: cluster-${region}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: global-web-app
  template:
    metadata:
      labels:
        app: global-web-app
        federation: "true"
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: REGION
          value: "${region}"
EOF
done
```

**Step 2: Configure global load balancer**
```bash
# Create federated service
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: global-web-svc
  namespace: federation-system
  annotations:
    federation.globaltech.com/load-balance: "round-robin"
spec:
  type: LoadBalancer
  selector:
    app: global-web-app
    federation: "true"
  ports:
  - port: 80
    targetPort: 80
EOF
```

**Verification:**
- ✅ Applications deployed across all regions
- ✅ Global load balancer configured

---

## 🛠 Task 5: Disaster Recovery & Monitoring

### Instructions

**Step 1: Configure cluster failover**
```bash
# Create failover policy
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: failover-policy
  namespace: federation-system
data:
  policy.yaml: |
    failover:
      primary: us-east
      secondary: eu-west
      tertiary: asia-pacific
      healthcheck_interval: 30s
      failover_threshold: 3
EOF

# Deploy failover controller
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: failover-controller
  namespace: federation-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: failover-controller
  template:
    metadata:
      labels:
        app: failover-controller
    spec:
      containers:
      - name: controller
        image: busybox:1.35
        command: ['sh', '-c']
        args:
        - |
          while true; do
            echo "Monitoring cluster health for failover..."
            echo "Primary: us-east - Status: Healthy"
            echo "Secondary: eu-west - Status: Healthy"  
            echo "Tertiary: asia-pacific - Status: Healthy"
            sleep 30
          done
EOF
```

**Step 2: Create operational runbook**
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: federation-runbook
  namespace: federation-system
data:
  runbook.md: |
    # Federation Operations Runbook
    
    ## Health Checks
    1. Check cluster status: kubectl get pods -n federation-system
    2. Verify regional deployments: kubectl get deployments --all-namespaces
    3. Test cross-cluster connectivity
    
    ## Disaster Recovery
    1. Detect failed cluster
    2. Trigger traffic redirection
    3. Scale remaining clusters
    4. Monitor performance impact
    
    ## Emergency Contacts
    - Platform Team: platform@globaltech.com
    - On-call Engineer: +1-800-PLATFORM
EOF
```

**Verification:**
- ✅ Failover policies configured
- ✅ Monitoring and runbooks deployed

---

## 📚 Success Criteria

### Task Completion Checklist
- [ ] **Task 1:** Multi-cluster environment established with proper networking
- [ ] **Task 2:** Federation control plane deployed and cluster registry active
- [ ] **Task 3:** Cross-cluster service discovery functional
- [ ] **Task 4:** Federated deployments with global load balancing
- [ ] **Task 5:** Disaster recovery and monitoring operational

### Production Readiness
- [ ] Multi-region application deployment successful
- [ ] Cross-cluster service communication verified
- [ ] Failover mechanisms tested and documented
- [ ] Monitoring covers all federation components

**🏆 Lab Complete!** You've successfully implemented cluster federation for GlobalTech's worldwide infrastructure.

---

## 🔄 Cleanup

```bash
# Remove federation components
kubectl delete namespace federation-system

# Remove regional clusters
kubectl delete namespace cluster-us-east cluster-eu-west cluster-asia-pacific

# Remove contexts
kubectl config unset contexts.us-east
kubectl config unset contexts.eu-west  
kubectl config unset contexts.asia-pacific

echo "Multi-cluster federation lab cleanup completed!"
```
