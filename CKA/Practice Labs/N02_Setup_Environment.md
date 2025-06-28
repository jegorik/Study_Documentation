# N02 Setup Environment: Ingress Traffic Management

> **🎯 Purpose:** Create a chaotic traffic management environment to simulate real-world Ingress challenges  
> **⚠️ Warning:** This setup creates suboptimal networking for learning purposes  
> **🔧 Cleanup:** Run cleanup commands after completing the lab

---

## 🚀 Quick Setup (5 minutes)

```bash
# Run this script to create the chaotic networking environment
curl -sSL https://raw.githubusercontent.com/cka-labs/setups/main/N02-setup.sh | bash
```

**OR** follow the manual setup below:

---

## 📋 Manual Environment Setup

### Step 1: Create Multiple NodePort Services (The Problem)

```yaml
# Deploy multiple applications with inefficient NodePort services
cat << 'EOF' | kubectl apply -f -
---
# Web application with random NodePort
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-frontend
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp-frontend
  template:
    metadata:
      labels:
        app: webapp-frontend
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: content
          mountPath: /usr/share/nginx/html
      volumes:
      - name: content
        configMap:
          name: frontend-content
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-frontend-service
  namespace: default
spec:
  selector:
    app: webapp-frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
  type: NodePort
---
# API service with different NodePort
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-api
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp-api
  template:
    metadata:
      labels:
        app: webapp-api
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=Legacy API Service - Port 30002"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-api-service
  namespace: default
spec:
  selector:
    app: webapp-api
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30002
  type: NodePort
---
# Admin service with yet another NodePort
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-admin
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp-admin
  template:
    metadata:
      labels:
        app: webapp-admin
    spec:
      containers:
      - name: admin
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=Admin Dashboard - Port 30003"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-admin-service
  namespace: default
spec:
  selector:
    app: webapp-admin
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30003
  type: NodePort
---
# Documentation service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-docs
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp-docs
  template:
    metadata:
      labels:
        app: webapp-docs
    spec:
      containers:
      - name: docs
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=Documentation Portal - Port 30004"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-docs-service
  namespace: default
spec:
  selector:
    app: webapp-docs
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30004
  type: NodePort
---
# Content for frontend
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-content
  namespace: default
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>TechFlow Legacy System</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; background: #f0f0f0; }
            .container { max-width: 800px; margin: 0 auto; background: white; padding: 20px; border-radius: 5px; }
            .warning { background: #ffebee; border: 1px solid #f44336; padding: 15px; margin: 20px 0; border-radius: 4px; }
            .service-list { list-style: none; padding: 0; }
            .service-list li { background: #e3f2fd; margin: 10px 0; padding: 15px; border-radius: 4px; }
            .port { color: #f44336; font-weight: bold; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>🏭 TechFlow Legacy System</h1>
            <div class="warning">
                <strong>⚠️ Infrastructure Issues:</strong>
                <ul>
                    <li>Multiple random ports for different services</li>
                    <li>No SSL/HTTPS support</li>
                    <li>Manual load balancing required</li>
                    <li>Poor user experience with port numbers</li>
                </ul>
            </div>
            
            <h2>Available Services:</h2>
            <ul class="service-list">
                <li>Frontend: <span class="port">http://node-ip:30001</span></li>
                <li>API: <span class="port">http://node-ip:30002</span></li>
                <li>Admin: <span class="port">http://node-ip:30003</span></li>
                <li>Docs: <span class="port">http://node-ip:30004</span></li>
            </ul>
            
            <p><strong>Problem:</strong> Customers need to remember different ports and there's no unified access point!</p>
        </div>
    </body>
    </html>
EOF
```

### Step 2: Create Additional Chaos with LoadBalancer Services

```yaml
# Add some LoadBalancer services that may not work properly
cat << 'EOF' | kubectl apply -f -
---
# Monitoring service that should be internal but exposed
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-service
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: monitoring-service
  template:
    metadata:
      labels:
        app: monitoring-service
    spec:
      containers:
      - name: monitor
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=Internal Monitoring - Should not be public!"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: monitoring-service-lb
  namespace: default
spec:
  selector:
    app: monitoring-service
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
---
# Database admin tool - also incorrectly exposed
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-admin
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db-admin
  template:
    metadata:
      labels:
        app: db-admin
    spec:
      containers:
      - name: admin
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=Database Admin Tool - SECURITY RISK!"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: db-admin-lb
  namespace: default
spec:
  selector:
    app: db-admin
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
EOF
```

### Step 3: Create Staging Environment Without Proper Separation

```bash
# Create staging namespace but without proper isolation
kubectl create namespace staging

# Deploy staging versions that might conflict
kubectl create deployment staging-app --image=nginx:1.20 --namespace=staging
kubectl expose deployment staging-app --port=80 --type=NodePort --namespace=staging

# Add some test data to staging
kubectl create configmap staging-config --from-literal=environment=staging --from-literal=debug=true --namespace=staging
```

### Step 4: Simulate DNS and Certificate Issues

```bash
# Create some fake certificates and DNS entries that will cause problems
kubectl create secret tls fake-tls-cert \
  --cert=<(openssl req -x509 -nodes -days 1 -newkey rsa:2048 -keyout /dev/null -out /dev/stdout -subj "/CN=wrong-domain.com") \
  --key=<(openssl genrsa 2048) \
  --namespace=default

# Create a misconfigured Ingress (if Ingress controller exists)
cat << 'EOF' | kubectl apply -f - 2>/dev/null || echo "No Ingress controller - this will be installed in the lab"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: broken-ingress
  namespace: default
spec:
  tls:
  - hosts:
    - wrong-domain.com
    secretName: fake-tls-cert
  rules:
  - host: wrong-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nonexistent-service
            port:
              number: 80
EOF
```

### Step 5: Create Resource Constraints and Performance Issues

```yaml
# Deploy resource-hungry applications that will compete
cat << 'EOF' | kubectl apply -f -
---
# CPU intensive app without proper resource limits
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-hog
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cpu-hog
  template:
    metadata:
      labels:
        app: cpu-hog
    spec:
      containers:
      - name: hog
        image: busybox:1.35
        command: ["sh", "-c", "while true; do echo 'consuming CPU'; done"]
        # No resource limits = potential resource starvation
---
# Memory intensive app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: memory-leak
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: memory-leak
  template:
    metadata:
      labels:
        app: memory-leak
    spec:
      containers:
      - name: leak
        image: busybox:1.35
        command: ["sh", "-c", "data=''; while true; do data=$data$(head -c 1024 /dev/zero | tr '\\0' 'x'); sleep 1; done"]
        # No memory limits = potential OOM issues
EOF
```

### Step 6: Verify Chaotic State

```bash
# Create verification script
cat << 'EOF' > check-chaos.sh
#!/bin/bash
echo "=== TechFlow Infrastructure Chaos Assessment ==="
echo ""

# Check NodePort services
echo "1. NodePort Services (should be multiple with random ports):"
kubectl get services -o custom-columns=NAME:.metadata.name,TYPE:.spec.type,PORTS:.spec.ports[*].nodePort | grep NodePort

echo ""
echo "2. LoadBalancer Services (exposing internal tools):"
kubectl get services -o custom-columns=NAME:.metadata.name,TYPE:.spec.type,EXTERNAL-IP:.status.loadBalancer.ingress[*].ip | grep LoadBalancer

echo ""
echo "3. Resource Usage (should show CPU/memory stress):"
kubectl top pods 2>/dev/null || echo "Metrics server not available"

echo ""
echo "4. Service Access Points (customer confusion):"
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
echo "Customers must access:"
echo "  - Main app: http://$NODE_IP:30001"
echo "  - API: http://$NODE_IP:30002"  
echo "  - Admin: http://$NODE_IP:30003"
echo "  - Docs: http://$NODE_IP:30004"

echo ""
echo "5. Security Issues:"
echo "  - No HTTPS anywhere"
echo "  - Internal monitoring exposed publicly"
echo "  - Database admin tools accessible"
echo "  - No rate limiting or access controls"

echo ""
echo "6. Staging/Production Issues:"
kubectl get deployments --all-namespaces | grep -E "staging|default"

echo ""
echo "7. Certificate Problems:"
kubectl get secrets -o custom-columns=NAME:.metadata.name,TYPE:.type | grep tls

if kubectl get ingress broken-ingress 2>/dev/null; then
    echo ""
    echo "8. Broken Ingress Configuration:"
    kubectl describe ingress broken-ingress | grep -A 5 -B 5 "Events:\|Warning"
fi

echo ""
echo "=== Problems Summary ==="
echo "❌ Multiple random ports (poor UX)"
echo "❌ No SSL/TLS security"
echo "❌ Internal services exposed publicly"
echo "❌ No traffic management or rate limiting"
echo "❌ Resource contention issues"
echo "❌ No proper staging/production separation"
echo "❌ Manual load balancing required"
echo ""
echo "🎯 Ready to implement proper Ingress traffic management!"
EOF

chmod +x check-chaos.sh
./check-chaos.sh
```

---

## 🎯 Environment Validation

Before starting the lab, verify these chaotic conditions exist:

### Expected Problems ✅

- [ ] Multiple NodePort services with random port numbers
- [ ] Internal services exposed via LoadBalancer
- [ ] No HTTPS/SSL termination anywhere
- [ ] No unified domain or URL structure
- [ ] Resource contention between applications
- [ ] Staging and production environments mixed
- [ ] No rate limiting or security policies
- [ ] Poor customer experience with port-based access

### Quick Validation Commands

```bash
# Check service types
kubectl get services --all-namespaces

# Test NodePort access
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
curl -s http://$NODE_IP:30001 | grep -o "TechFlow"
curl -s http://$NODE_IP:30002 | grep -o "API"

# Verify no Ingress controller
kubectl get pods --all-namespaces | grep -i ingress || echo "No Ingress controller found (expected)"

# Check for SSL certificates
kubectl get secrets --all-namespaces | grep tls
```

---

## 🧹 Cleanup Commands

**Run these after completing the lab to remove the test environment:**

```bash
# Remove all test deployments
kubectl delete deployment webapp-frontend webapp-api webapp-admin webapp-docs monitoring-service db-admin cpu-hog memory-leak

# Remove problematic services
kubectl delete service webapp-frontend-service webapp-api-service webapp-admin-service webapp-docs-service monitoring-service-lb db-admin-lb

# Remove staging environment
kubectl delete namespace staging

# Remove broken configurations
kubectl delete ingress broken-ingress 2>/dev/null || true
kubectl delete secret fake-tls-cert

# Remove ConfigMaps
kubectl delete configmap frontend-content

# Clean up any remaining test resources
kubectl delete all -l app=webapp-frontend
kubectl delete all -l app=webapp-api
kubectl delete all -l app=webapp-admin

echo "✅ N02 environment cleanup complete"
```

---

## 🔧 Troubleshooting

### Common Setup Issues

**Issue:** "NodePort services not accessible"

```bash
# Check node IPs and firewall rules
kubectl get nodes -o wide
kubectl get services -o wide

# Test local connectivity
kubectl port-forward service/webapp-frontend-service 8080:80 &
curl http://localhost:8080
```

**Issue:** "LoadBalancer external IP pending"

```bash
# This is expected in many test environments
kubectl get services -o wide
# External IPs may show <pending> - this is part of the chaos simulation
```

**Issue:** "High CPU usage from test workloads"

```bash
# This is intentional to simulate resource pressure
kubectl top pods
kubectl delete deployment cpu-hog memory-leak  # if it becomes problematic
```

### Environment Requirements

```bash
# Verify cluster resources
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl get pods --all-namespaces | grep -E "Error|CrashLoopBackOff"

# Check available storage
kubectl get storageclass
kubectl get pv
```

---

## 📝 Notes for Lab Facilitators

### Setup Time: ~5 minutes

### Prerequisites

- Kubernetes cluster with at least 3 worker nodes
- kubectl access with cluster-admin permissions
- No existing Ingress controller (will be installed during lab)
- Sufficient cluster resources for multiple deployments

### Learning Objectives Alignment

This setup creates the exact traffic management chaos that N02 lab is designed to solve, providing realistic context for learning enterprise Ingress management.

### Resource Usage

- **CPU:** Moderate (test workloads will create some pressure)
- **Memory:** Low to moderate
- **Storage:** Minimal (only for container images)
- **Network:** Multiple exposed services for realistic testing

**🎯 Ready to start N02 - Ingress Traffic Management!**
