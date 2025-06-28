# T02 Setup Environment: Network Communication Failure

> **🎯 Scenario Setup:** FinServ Bank payment processing network failure simulation  
> **⚡ Purpose:** Create realistic network connectivity issues for troubleshooting practice  
> **⏱️ Setup Time:** 8-10 minutes  

---

## 🏛️ Scenario Context

This setup creates a **FinServ Bank** environment where:
- Payment processing microservices lose connectivity during peak trading hours
- Multiple potential root causes simulate real production network failures
- Students must systematically diagnose and restore service connectivity
- Realistic financial sector pressure and business impact

---

## 🚀 Quick Setup (Copy & Paste)

### **Step 1: Create Banking Environment Structure**

```bash
# Create dedicated namespace for payment processing
kubectl create namespace finserv-payments

# Create monitoring namespace for observability
kubectl create namespace finserv-monitoring

# Set default namespace for convenience
kubectl config set-context --current --namespace=finserv-payments

echo "✅ FinServ Bank namespaces created"
```

### **Step 2: Deploy Payment Processing Microservices**

```bash
# Deploy payment gateway service (main entry point)
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-gateway
  namespace: finserv-payments
  labels:
    app: payment-gateway
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-gateway
  template:
    metadata:
      labels:
        app: payment-gateway
        tier: frontend
    spec:
      containers:
      - name: gateway
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: SERVICE_NAME
          value: "payment-gateway"
        - name: BACKEND_URLS
          value: "http://payment-processor:8080,http://fraud-detector:8081"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
EOF

# Create service for payment gateway
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: payment-gateway-service
  namespace: finserv-payments
  labels:
    app: payment-gateway
spec:
  selector:
    app: payment-gateway
  ports:
  - port: 80
    targetPort: 80
    name: http
  type: ClusterIP
EOF

# Deploy payment processor service (core business logic)
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-processor
  namespace: finserv-payments
  labels:
    app: payment-processor
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: payment-processor
  template:
    metadata:
      labels:
        app: payment-processor
        tier: backend
    spec:
      containers:
      - name: processor
        image: nginx:1.21
        ports:
        - containerPort: 8080
        env:
        - name: SERVICE_NAME
          value: "payment-processor"
        - name: DATABASE_URL
          value: "http://payment-database:5432"
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
EOF

# Create service for payment processor
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: payment-processor
  namespace: finserv-payments
  labels:
    app: payment-processor
spec:
  selector:
    app: payment-processor
  ports:
  - port: 8080
    targetPort: 8080
    name: http
  type: ClusterIP
EOF

# Deploy fraud detection service
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fraud-detector
  namespace: finserv-payments
  labels:
    app: fraud-detector
    tier: security
spec:
  replicas: 2
  selector:
    matchLabels:
      app: fraud-detector
  template:
    metadata:
      labels:
        app: fraud-detector
        tier: security
    spec:
      containers:
      - name: detector
        image: nginx:1.21
        ports:
        - containerPort: 8081
        env:
        - name: SERVICE_NAME
          value: "fraud-detector"
        - name: ML_MODEL_URL
          value: "http://ml-service:9000"
        readinessProbe:
          httpGet:
            path: /
            port: 8081
          initialDelaySeconds: 8
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 8081
          initialDelaySeconds: 12
          periodSeconds: 10
EOF

# Create service for fraud detector
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: fraud-detector
  namespace: finserv-payments
  labels:
    app: fraud-detector
spec:
  selector:
    app: fraud-detector
  ports:
  - port: 8081
    targetPort: 8081
    name: http
  type: ClusterIP
EOF

echo "✅ Payment processing services deployed"
```

### **Step 3: Wait for Services to Stabilize**

```bash
# Wait for all deployments to be ready
kubectl rollout status deployment/payment-gateway --timeout=120s
kubectl rollout status deployment/payment-processor --timeout=120s  
kubectl rollout status deployment/fraud-detector --timeout=120s

# Verify all pods are healthy
kubectl get pods -l tier=frontend,backend,security

# Test initial connectivity
kubectl get endpoints -n finserv-payments

echo "✅ Payment system baseline established"
```

### **Step 4: Create Network Failure Scenarios (Execute during lab)**

```bash
# WARNING: These commands introduce network failures that students must troubleshoot!
# Choose ONE scenario to execute when lab starts

# SCENARIO A: DNS Configuration Problem
echo "💥 INTRODUCING DNS FAILURE..."
kubectl get configmap coredns -n kube-system -o yaml > /tmp/coredns-backup.yaml
kubectl patch configmap coredns -n kube-system --patch='
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . 8.8.8.8 8.8.4.4 {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
    # BROKEN: Missing payment namespace resolution
'

# SCENARIO B: Service Selector Mismatch  
echo "💥 INTRODUCING SERVICE SELECTOR FAILURE..."
kubectl patch service payment-processor --patch='
spec:
  selector:
    app: payment-processor-wrong
'

# SCENARIO C: Network Policy Blocking Traffic
echo "💥 INTRODUCING NETWORK POLICY FAILURE..."
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-security-lockdown
  namespace: finserv-payments
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: finserv-monitoring
    ports:
    - protocol: TCP
      port: 9090
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: TCP
      port: 53
EOF

# SCENARIO D: Endpoints Configuration Issue
echo "💥 INTRODUCING ENDPOINT FAILURE..."
kubectl patch service payment-processor --patch='
spec:
  ports:
  - port: 8080
    targetPort: 9999
    name: http
'

echo "🚨 NETWORK CRISIS INITIATED!"
echo "💳 Payment processing system is failing!"
echo "⏰ Students have 25 minutes to restore connectivity..."
```

---

## 🔍 Verification Commands

### **Environment Health Check**
```bash
# Overall system status
kubectl get all -n finserv-payments

# Service and endpoint status
kubectl get svc,endpoints -n finserv-payments

# Pod connectivity status
kubectl get pods -o wide -n finserv-payments

# Recent events that might indicate problems
kubectl get events --sort-by=.metadata.creationTimestamp -n finserv-payments
```

### **Connectivity Testing Tools**
```bash
# Create debug pod for network testing
kubectl run network-debug --image=busybox:1.35 --rm -it -n finserv-payments -- sh

# Inside debug pod, test various connectivity scenarios:
# nslookup payment-processor
# nslookup payment-processor.finserv-payments.svc.cluster.local
# wget -qO- payment-processor:8080
# wget -qO- payment-gateway-service
# nc -zv payment-processor 8080

# DNS debugging from outside pod
kubectl exec -it network-debug -n finserv-payments -- nslookup payment-processor
kubectl exec -it network-debug -n finserv-payments -- nslookup kubernetes.default
```

### **Real-time Monitoring**
```bash
# Monitor service endpoint changes
kubectl get endpoints -w -n finserv-payments

# Watch pod status changes
kubectl get pods -w -n finserv-payments

# Monitor network policy applications
kubectl get networkpolicy -w -n finserv-payments

# Check CoreDNS health
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

---

## 🎯 Expected Failure Symptoms

Students should observe these symptoms depending on the scenario:

### **DNS Failure Symptoms:**
- Services resolve to `NXDOMAIN` or incorrect IPs
- Connectivity works with direct IP but fails with service names
- CoreDNS logs show resolution errors
- Cross-namespace DNS resolution problems

### **Service Selector Symptoms:**
- Service endpoints show no available backends
- `kubectl get endpoints` shows empty endpoint list
- Pods are healthy but not receiving traffic
- Service traffic load balancing fails

### **Network Policy Symptoms:**
- Connection timeouts between services
- Pods can't reach external DNS
- Communication works from some pods but not others
- Policy logs (if available) show blocked connections

### **Port Configuration Symptoms:**
- Connections refused on expected ports
- Service responds but backend pods don't receive traffic
- Port mismatch between service and container definitions
- Health checks failing unexpectedly

---

## 🧹 Cleanup Commands

```bash
# Restore DNS configuration (if modified)
kubectl apply -f /tmp/coredns-backup.yaml 2>/dev/null || true
kubectl rollout restart deployment/coredns -n kube-system

# Reset service configurations
kubectl patch service payment-processor -n finserv-payments --patch='
spec:
  selector:
    app: payment-processor
  ports:
  - port: 8080
    targetPort: 8080
    name: http
'

# Remove network policies
kubectl delete networkpolicy payment-security-lockdown -n finserv-payments 2>/dev/null || true

# Clean up namespaces
kubectl delete namespace finserv-payments finserv-monitoring

# Clean up temporary files
rm -f /tmp/coredns-backup.yaml

# Reset context
kubectl config set-context --current --namespace=default

echo "✅ FinServ Bank environment cleaned up successfully"
```

---

## 🔧 Instructor Notes

### **Timing Setup:**
1. **Pre-lab (8-10 min):** Execute Steps 1-3 to establish baseline environment
2. **Lab Start:** Choose and execute ONE failure scenario from Step 4
3. **During Lab:** Monitor student progress and provide guided hints if needed

### **Scenario Selection Guide:**
- **DNS Failure (A):** Best for testing systematic DNS troubleshooting
- **Service Selector (B):** Good for service/endpoint relationship understanding  
- **Network Policy (C):** Advanced scenario for security-aware students
- **Port Configuration (D):** Basic but tricky configuration debugging

### **Common Student Approaches:**
- **Overwhelmed Response:** May try complex solutions before checking basics
- **Tool Dependency:** Over-reliance on monitoring tools vs systematic debugging
- **Scope Confusion:** Difficulty determining if problem is local or cluster-wide
- **DNS Assumptions:** Not considering DNS as potential root cause

### **Success Indicators:**
- Systematic troubleshooting approach (service → endpoints → pods → network)
- Effective use of kubectl debugging commands
- Proper DNS testing from pod contexts
- Understanding of service discovery mechanics
- Ability to verify fix effectiveness

### **Extension Scenarios:**
- **Multi-failure:** Combine 2+ failure modes for advanced challenge
- **Cross-namespace:** Add services in different namespaces with communication issues
- **External Dependencies:** Include external service connectivity problems
- **Performance:** Add latency/bandwidth issues alongside connectivity problems

### **Troubleshooting Hints by Scenario:**
- **DNS:** Focus on CoreDNS logs and configuration
- **Selector:** Compare service selectors with pod labels
- **Network Policy:** Trace ingress/egress rules systematically
- **Ports:** Verify service port → targetPort → containerPort chain

---

*🏛️ **Reality Check:** Financial services networks are complex and regulated. Any connectivity failure during trading hours has immediate business impact and regulatory scrutiny, making systematic troubleshooting essential.*
