# Lab N02: Ingress Traffic Management

> **📋 Lab Category:** Networking - Advanced Traffic Routing & Load Balancing  
> **⏱️ Estimated Time:** 30-40 minutes  
> **🎯 Difficulty:** Intermediate  
> **🔗 Exam Objectives:** Understand Services and Networking, Configure and use Ingress controllers, Implement network policies

---

## 🎬 Real-World Scenario

### Background Context

**TechFlow Enterprises** is a rapidly growing SaaS company that provides multiple web applications for their 50,000+ global customers. Their Kubernetes cluster hosts 15 different microservices across development, staging, and production environments. The current setup uses basic NodePort services, creating a management nightmare with inconsistent URLs, no SSL termination, and poor load distribution.

**Your Role:** Senior DevOps Engineer  
**Situation:** The company is launching 3 new products next month and needs professional-grade traffic management  
**Urgency:** **CRITICAL** - Current infrastructure can't handle the projected 300% traffic increase during product launch

### The Challenge

You must implement a comprehensive Ingress strategy that:
- Consolidates all services under professional domain names
- Implements SSL/TLS termination with automatic certificate management
- Provides intelligent load balancing and traffic routing
- Enables path-based and host-based routing for different applications
- Supports staging/production traffic splitting
- Ensures high availability and performance monitoring

**Business Impact:** Poor traffic management during launch could result in customer churn, lost revenue ($500K+ potential), and damaged reputation.

---

## 🎯 Learning Objectives

By completing this lab, you will:

- [ ] **Primary Skill:** Design and implement enterprise-grade Ingress architecture with multiple controllers
- [ ] **Secondary Skills:** Configure SSL/TLS termination, path-based routing, and traffic policies
- [ ] **Real-world Application:** Manage complex multi-application traffic routing in production environments
- [ ] **Exam Relevance:** Master CKA networking tasks including Ingress configuration, service exposure, and traffic management

---

## 🔧 Prerequisites

### Knowledge Requirements

- [ ] Understanding of Kubernetes Services (ClusterIP, NodePort, LoadBalancer)
- [ ] Basic knowledge of HTTP/HTTPS protocols and DNS
- [ ] Familiarity with YAML configuration and kubectl commands
- [ ] Previous completion of N01 (Multi-Tier App Connectivity) recommended

### Environment Setup

```bash
# Cluster requirements
- Kubernetes cluster v1.28+ with at least 3 worker nodes
- kubectl configured with appropriate permissions
- Helm 3.x installed for package management
- openssl for certificate generation
- curl for testing endpoints

# Verification commands
kubectl cluster-info
kubectl get nodes -o wide
helm version
curl --version
```

---

## 🚨 Initial Crisis State

You arrive to find a chaotic traffic management situation:

### Current Infrastructure Problems

1. **15 different NodePort services** - Customers need to remember random port numbers
2. **No SSL termination** - All traffic is insecure HTTP
3. **Manual load balancing** - Traffic distributed poorly across pods
4. **DNS chaos** - Different subdomains pointing to random IP addresses
5. **No traffic policies** - No rate limiting, no path-based routing
6. **Staging/Prod mixing** - Risk of customers accessing test environments

### Immediate Symptoms

```bash
# Check current service chaos
kubectl get services --all-namespaces -o wide
kubectl get endpoints --all-namespaces

# Verify current NodePort madness
kubectl get services -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.type}{"\t"}{.spec.ports[*].nodePort}{"\n"}{end}'
```

---

## 📋 Tasks & Solutions

### Task 1: Deploy and Configure NGINX Ingress Controller

**Scenario:** Set up a production-ready Ingress controller with proper resource management and monitoring.

#### 🚀 Ingress Controller Installation

```bash
# 1. Add NGINX Ingress Helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# 2. Create namespace for Ingress controller
kubectl create namespace ingress-nginx

# 3. Install NGINX Ingress with production configuration
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.replicaCount=2 \
  --set controller.nodeSelector."kubernetes\.io/arch"=amd64 \
  --set controller.admissionWebhooks.enabled=true \
  --set controller.metrics.enabled=true \
  --set controller.metrics.serviceMonitor.enabled=true \
  --set controller.podSecurityPolicy.enabled=true
```

#### ✅ Production Configuration

```yaml
# Apply advanced Ingress controller configuration
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
data:
  # Performance optimizations
  worker-processes: "auto"
  max-worker-connections: "16384"
  max-worker-open-files: "65536"
  
  # Security headers
  enable-brotli: "true"
  use-gzip: "true"
  gzip-level: "6"
  
  # SSL configuration
  ssl-protocols: "TLSv1.2 TLSv1.3"
  ssl-ciphers: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384"
  ssl-prefer-server-ciphers: "false"
  
  # Rate limiting
  rate-limit: "100"
  rate-limit-rpm: "6000"
  
  # Custom error pages
  custom-http-errors: "404,503"
  
  # Load balancing
  upstream-hash-by: "$binary_remote_addr"
  
  # Monitoring and logging
  enable-access-log-for-default-backend: "true"
  log-format-escape-json: "true"
  access-log-params: "buffer=16k flush=5s"
EOF
```

#### 🔍 Verification Commands

```bash
# Verify Ingress controller is running
kubectl get pods -n ingress-nginx
kubectl get services -n ingress-nginx

# Check controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Verify admission webhook
kubectl get validatingwebhookconfigurations | grep ingress-nginx
```

### Task 2: Create Application Deployments for Testing

**Scenario:** Deploy multiple applications that represent TechFlow's microservices architecture.

#### 🏗️ Multi-Application Setup

```yaml
# Deploy TechFlow application stack
cat << 'EOF' | kubectl apply -f -
---
# Main web application
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main-app
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main-app
  template:
    metadata:
      labels:
        app: main-app
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
          name: main-app-content
---
apiVersion: v1
kind: Service
metadata:
  name: main-app-service
  namespace: default
spec:
  selector:
    app: main-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
# API service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-service
  template:
    metadata:
      labels:
        app: api-service
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=TechFlow API v1.0 - Production Ready"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: default
spec:
  selector:
    app: api-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
---
# Admin dashboard
apiVersion: apps/v1
kind: Deployment
metadata:
  name: admin-dashboard
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: admin-dashboard
  template:
    metadata:
      labels:
        app: admin-dashboard
    spec:
      containers:
      - name: dashboard
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=TechFlow Admin Dashboard - Restricted Access"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: admin-dashboard-service
  namespace: default
spec:
  selector:
    app: admin-dashboard
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
---
# Content ConfigMaps
apiVersion: v1
kind: ConfigMap
metadata:
  name: main-app-content
  namespace: default
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>TechFlow Enterprises</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; }
            .container { max-width: 800px; margin: 0 auto; text-align: center; }
            .logo { font-size: 3em; margin-bottom: 20px; }
            .subtitle { font-size: 1.2em; opacity: 0.9; }
            .features { margin-top: 40px; display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; }
            .feature { background: rgba(255,255,255,0.1); padding: 20px; border-radius: 10px; }
        </style>
    </head>
    <body>
        <div class="container">
            <div class="logo">🚀 TechFlow</div>
            <div class="subtitle">Enterprise SaaS Platform</div>
            <div class="features">
                <div class="feature">
                    <h3>🔐 Secure</h3>
                    <p>Enterprise-grade security</p>
                </div>
                <div class="feature">
                    <h3>⚡ Fast</h3>
                    <p>Lightning-fast performance</p>
                </div>
                <div class="feature">
                    <h3>📈 Scalable</h3>
                    <p>Handles millions of users</p>
                </div>
            </div>
        </div>
    </body>
    </html>
EOF
```

### Task 3: Implement Basic Ingress Configuration

**Scenario:** Create your first Ingress resource with path-based routing.

#### 🛣️ Path-Based Routing Implementation

```yaml
# Create basic Ingress with path-based routing
cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: techflow-basic-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: main-app-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-dashboard-service
            port:
              number: 80
EOF
```

#### 🧪 Basic Testing

```bash
# Get Ingress controller external IP
INGRESS_IP=$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Ingress IP: $INGRESS_IP"

# Add local DNS entry (for testing)
echo "$INGRESS_IP techflow.local" | sudo tee -a /etc/hosts

# Test path-based routing
curl -H "Host: techflow.local" http://$INGRESS_IP/
curl -H "Host: techflow.local" http://$INGRESS_IP/api
curl -H "Host: techflow.local" http://$INGRESS_IP/admin

# Check Ingress status
kubectl get ingress techflow-basic-ingress
kubectl describe ingress techflow-basic-ingress
```

### Task 4: Configure SSL/TLS with Cert-Manager

**Scenario:** Implement automatic SSL certificate management for secure HTTPS traffic.

#### 🔐 Cert-Manager Installation

```bash
# 1. Install cert-manager using Helm
helm repo add jetstack https://charts.jetstack.io
helm repo update

kubectl create namespace cert-manager

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.13.0 \
  --set installCRDs=true \
  --set global.leaderElection.namespace=cert-manager
```

#### 🏭 Certificate Issuer Configuration

```yaml
# Create ClusterIssuer for Let's Encrypt (staging and production)
cat << 'EOF' | kubectl apply -f -
---
# Staging issuer for testing
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: devops@techflow.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
---
# Production issuer
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: devops@techflow.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
---
# Self-signed issuer for development
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
EOF
```

#### 🛡️ HTTPS Ingress Configuration

```yaml
# Update Ingress with SSL/TLS configuration
cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: techflow-https-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "selfsigned-issuer"
    nginx.ingress.kubernetes.io/hsts: "true"
    nginx.ingress.kubernetes.io/hsts-max-age: "31536000"
    nginx.ingress.kubernetes.io/hsts-include-subdomains: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - techflow.local
    secretName: techflow-tls-secret
  rules:
  - host: techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: main-app-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-dashboard-service
            port:
              number: 80
EOF
```

### Task 5: Advanced Traffic Management and Load Balancing

**Scenario:** Implement sophisticated traffic policies including rate limiting, session affinity, and custom headers.

#### ⚙️ Advanced Ingress Annotations

```yaml
# Create production-grade Ingress with advanced features
cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: techflow-production-ingress
  namespace: default
  annotations:
    # SSL Configuration
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "selfsigned-issuer"
    
    # Security Headers
    nginx.ingress.kubernetes.io/configuration-snippet: |
      add_header X-Frame-Options "SAMEORIGIN" always;
      add_header X-Content-Type-Options "nosniff" always;
      add_header X-XSS-Protection "1; mode=block" always;
      add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Rate Limiting
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-burst: "200"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    
    # Load Balancing
    nginx.ingress.kubernetes.io/upstream-hash-by: "$binary_remote_addr"
    nginx.ingress.kubernetes.io/load-balance: "ip_hash"
    
    # Timeouts
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    
    # Buffer sizes
    nginx.ingress.kubernetes.io/proxy-buffer-size: "16k"
    nginx.ingress.kubernetes.io/proxy-buffers-number: "8"
    
    # Custom responses
    nginx.ingress.kubernetes.io/custom-http-errors: "404,503"
    nginx.ingress.kubernetes.io/default-backend: "default/error-page-service"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - techflow.local
    - api.techflow.local
    - admin.techflow.local
    secretName: techflow-wildcard-tls
  rules:
  # Main application
  - host: techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: main-app-service
            port:
              number: 80
  # API subdomain with different rate limits
  - host: api.techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
  # Admin with stricter access controls
  - host: admin.techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-dashboard-service
            port:
              number: 80
---
# Error page service for custom error handling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: error-page
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: error-page
  template:
    metadata:
      labels:
        app: error-page
    spec:
      containers:
      - name: error-page
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: error-content
          mountPath: /usr/share/nginx/html
      volumes:
      - name: error-content
        configMap:
          name: error-page-content
---
apiVersion: v1
kind: Service
metadata:
  name: error-page-service
  namespace: default
spec:
  selector:
    app: error-page
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: error-page-content
  namespace: default
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>TechFlow - Service Temporarily Unavailable</title>
        <style>
            body { font-family: Arial, sans-serif; text-align: center; margin-top: 100px; background: #f5f5f5; }
            .error-container { max-width: 600px; margin: 0 auto; background: white; padding: 40px; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
            .error-code { font-size: 4em; color: #e74c3c; margin-bottom: 20px; }
            h1 { color: #2c3e50; }
            p { color: #7f8c8d; font-size: 1.1em; line-height: 1.6; }
        </style>
    </head>
    <body>
        <div class="error-container">
            <div class="error-code">🚧</div>
            <h1>Service Temporarily Unavailable</h1>
            <p>We're working hard to restore service. Please try again in a few moments.</p>
            <p>If the problem persists, please contact our support team.</p>
        </div>
    </body>
    </html>
EOF
```

### Task 6: Implement Traffic Splitting and Canary Deployments

**Scenario:** Set up blue-green and canary deployment capabilities using Ingress traffic splitting.

#### 🚥 Canary Deployment Setup

```yaml
# Deploy canary version of the main application
cat << 'EOF' | kubectl apply -f -
---
# Canary version deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main-app-canary
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main-app-canary
      version: canary
  template:
    metadata:
      labels:
        app: main-app-canary
        version: canary
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
          name: main-app-canary-content
---
apiVersion: v1
kind: Service
metadata:
  name: main-app-canary-service
  namespace: default
spec:
  selector:
    app: main-app-canary
    version: canary
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
# Canary content - new feature version
apiVersion: v1
kind: ConfigMap
metadata:
  name: main-app-canary-content
  namespace: default
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>TechFlow Enterprises - New Features!</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); color: white; }
            .container { max-width: 800px; margin: 0 auto; text-align: center; }
            .logo { font-size: 3em; margin-bottom: 20px; }
            .subtitle { font-size: 1.2em; opacity: 0.9; }
            .new-badge { background: #ffeb3b; color: #333; padding: 5px 15px; border-radius: 20px; font-weight: bold; }
            .features { margin-top: 40px; display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; }
            .feature { background: rgba(255,255,255,0.1); padding: 20px; border-radius: 10px; }
        </style>
    </head>
    <body>
        <div class="container">
            <div class="logo">🚀 TechFlow <span class="new-badge">NEW!</span></div>
            <div class="subtitle">Enterprise SaaS Platform - Beta Features</div>
            <div class="features">
                <div class="feature">
                    <h3>🤖 AI-Powered</h3>
                    <p>New AI assistant features</p>
                </div>
                <div class="feature">
                    <h3>📊 Analytics</h3>
                    <p>Advanced reporting dashboard</p>
                </div>
                <div class="feature">
                    <h3>🌐 Global</h3>
                    <p>Multi-region deployment</p>
                </div>
            </div>
        </div>
    </body>
    </html>
---
# Canary Ingress - 10% traffic to new version
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: techflow-canary-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "selfsigned-issuer"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - techflow.local
    secretName: techflow-tls-secret
  rules:
  - host: techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: main-app-canary-service
            port:
              number: 80
EOF
```

#### 🎯 A/B Testing Configuration

```yaml
# Header-based canary routing for A/B testing
cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: techflow-header-canary
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "X-Canary-Version"
    nginx.ingress.kubernetes.io/canary-by-header-value: "beta"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - techflow.local
    secretName: techflow-tls-secret
  rules:
  - host: techflow.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: main-app-canary-service
            port:
              number: 80
EOF
```

### Task 7: Monitoring and Observability

**Scenario:** Implement comprehensive monitoring for your Ingress infrastructure.

#### 📊 Monitoring Setup

```bash
# Enable Ingress controller metrics
kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","args":["--enable-metrics=true","--metrics-per-host=true"]}]}}}}'

# Check metrics endpoint
INGRESS_POD=$(kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n ingress-nginx $INGRESS_POD 10254:10254 &

# Test metrics (in another terminal)
curl http://localhost:10254/metrics | grep nginx_
```

#### 🔍 Ingress Testing and Validation

```bash
# Comprehensive testing script
cat << 'EOF' > test-ingress.sh
#!/bin/bash
set -e

echo "=== TechFlow Ingress Testing Suite ==="
echo ""

# Get Ingress IP
INGRESS_IP=$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
if [ -z "$INGRESS_IP" ]; then
    INGRESS_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
    INGRESS_PORT=$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[0].nodePort}')
    INGRESS_URL="http://$INGRESS_IP:$INGRESS_PORT"
else
    INGRESS_URL="https://$INGRESS_IP"
fi

echo "Testing Ingress at: $INGRESS_URL"
echo ""

# Test 1: Basic connectivity
echo "1. Testing basic connectivity..."
if curl -k -H "Host: techflow.local" -s "$INGRESS_URL" | grep -q "TechFlow"; then
    echo "✅ Main application accessible"
else
    echo "❌ Main application failed"
fi

# Test 2: API path routing
echo "2. Testing API path routing..."
if curl -k -H "Host: techflow.local" -s "$INGRESS_URL/api" | grep -q "API"; then
    echo "✅ API path routing working"
else
    echo "❌ API path routing failed"
fi

# Test 3: Admin path routing
echo "3. Testing admin path routing..."
if curl -k -H "Host: techflow.local" -s "$INGRESS_URL/admin" | grep -q "Admin"; then
    echo "✅ Admin path routing working"
else
    echo "❌ Admin path routing failed"
fi

# Test 4: SSL/TLS certificate
echo "4. Testing SSL certificate..."
if curl -k -I -H "Host: techflow.local" "$INGRESS_URL" | grep -q "HTTP/2 200\|HTTP/1.1 200"; then
    echo "✅ HTTPS working"
else
    echo "❌ HTTPS failed"
fi

# Test 5: Canary deployment (weight-based)
echo "5. Testing canary deployment..."
canary_hits=0
for i in {1..20}; do
    if curl -k -H "Host: techflow.local" -s "$INGRESS_URL" | grep -q "NEW!"; then
        ((canary_hits++))
    fi
done
echo "Canary hits: $canary_hits/20 (expected ~2 for 10% traffic)"

# Test 6: Header-based canary
echo "6. Testing header-based canary..."
if curl -k -H "Host: techflow.local" -H "X-Canary-Version: beta" -s "$INGRESS_URL" | grep -q "NEW!"; then
    echo "✅ Header-based canary working"
else
    echo "❌ Header-based canary failed"
fi

# Test 7: Rate limiting (if enabled)
echo "7. Testing rate limiting..."
rate_limited=false
for i in {1..150}; do
    response=$(curl -k -H "Host: techflow.local" -s -o /dev/null -w "%{http_code}" "$INGRESS_URL")
    if [ "$response" = "429" ]; then
        rate_limited=true
        break
    fi
done

if [ "$rate_limited" = true ]; then
    echo "✅ Rate limiting working (got 429 Too Many Requests)"
else
    echo "⚠️  Rate limiting not triggered (check configuration)"
fi

echo ""
echo "=== Test Summary ==="
kubectl get ingress
echo ""
kubectl get services | grep -E "main-app|api-service|admin-dashboard"
echo ""
echo "=== Certificate Status ==="
kubectl get certificate
kubectl describe certificate techflow-tls-secret | grep -A 5 "Status:"

EOF

chmod +x test-ingress.sh
./test-ingress.sh
```

---

## 🎯 Verification Commands

```bash
# Final comprehensive verification
echo "=== Final Ingress Infrastructure Audit ==="

# 1. Check all Ingress resources
kubectl get ingress --all-namespaces -o wide

# 2. Verify Ingress controller health
kubectl get pods -n ingress-nginx
kubectl get services -n ingress-nginx

# 3. Check SSL certificates
kubectl get certificates
kubectl get secrets | grep tls

# 4. Verify traffic distribution
kubectl top pods -l app=main-app
kubectl top pods -l app=main-app-canary

# 5. Check Ingress controller logs for errors
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller | tail -20

# 6. Validate DNS and routing
nslookup techflow.local
dig techflow.local

# 7. Performance metrics
kubectl top nodes
kubectl get hpa 2>/dev/null || echo "No HPA configured"
```

---

## 💡 Key Learning Points

### Ingress Architecture Mastered

1. **Controller Management:** NGINX Ingress controller with production configuration
2. **SSL/TLS Automation:** Cert-manager integration for automatic certificate provisioning
3. **Traffic Routing:** Path-based and host-based routing with intelligent load balancing
4. **Advanced Features:** Rate limiting, security headers, custom error pages
5. **Canary Deployments:** Weight-based and header-based traffic splitting
6. **Monitoring:** Metrics exposure and health checking

### Production Best Practices Applied

- **Security:** HTTPS enforcement, security headers, rate limiting
- **Performance:** Connection pooling, buffer optimization, compression
- **Reliability:** Health checks, graceful failover, custom error handling
- **Scalability:** Multiple replicas, load balancing algorithms
- **Observability:** Comprehensive logging and metrics collection

### Common Ingress Pitfalls Avoided

- **Avoided:** Using insecure HTTP in production
- **Avoided:** Single point of failure with one Ingress controller
- **Avoided:** Missing rate limiting and security headers
- **Avoided:** Poor SSL certificate management
- **Avoided:** Inadequate traffic splitting for deployments

---

## 🚀 Bonus Challenges

### Advanced Traffic Management

1. **Multi-Cluster Ingress:** Implement cross-cluster traffic routing
2. **Service Mesh Integration:** Integrate with Istio for advanced traffic policies
3. **WAF Integration:** Add Web Application Firewall capabilities
4. **Geo-routing:** Implement geographic-based traffic routing
5. **Circuit Breaker:** Add circuit breaker patterns for resilience

### Enterprise Extensions

1. **OAuth Integration:** Add authentication at the Ingress layer
2. **API Gateway:** Transform Ingress into a full API gateway
3. **Cost Optimization:** Implement traffic-based auto-scaling
4. **Compliance:** Add request/response logging for audit requirements

---

## 📚 Additional Resources

- [NGINX Ingress Controller Documentation](https://kubernetes.github.io/ingress-nginx/)
- [Cert-Manager Documentation](https://cert-manager.io/docs/)
- [Kubernetes Ingress Concepts](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [CKA Exam Networking Topics](https://github.com/cncf/curriculum/blob/master/CKA_Curriculum_v1.28.pdf)

**Next Lab:** W02 - Auto-scaling E-commerce Site (HPA, VPA, and ConfigMaps for dynamic scaling)

---

*🎯 **Success Criteria:** You've successfully implemented enterprise-grade Ingress traffic management that can handle TechFlow's product launch. The infrastructure now supports secure HTTPS traffic, intelligent routing, canary deployments, and comprehensive monitoring. The company is ready for their 300% traffic increase with professional-grade traffic management.*
