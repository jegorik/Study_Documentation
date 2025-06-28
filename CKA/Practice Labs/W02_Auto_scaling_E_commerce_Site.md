# Lab W02: Auto-scaling E-commerce Site

> **📋 Lab Category:** Workloads & Scheduling - Auto-scaling and Resource Management  
> **⏱️ Estimated Time:** 35-45 minutes  
> **🎯 Difficulty:** Advanced  
> **🔗 Exam Objectives:** Understand deployments and how to perform rolling updates and rollbacks, Use ConfigMaps and Secrets, Scale applications, Understand the primitives used to create robust, self-healing application deployments

---

## 🎬 Real-World Scenario

### Background Context

**EliteShop Commerce** is a rapidly growing e-commerce platform that experiences extreme traffic variations throughout the day and seasonal spikes during sales events. During Black Friday last year, their fixed-resource deployment crashed when traffic increased 2000% in just 30 minutes, resulting in $3.2M in lost revenue and significant customer churn.

**Your Role:** Senior Site Reliability Engineer (SRE)  
**Situation:** Preparing for the upcoming Holiday Sale season with expected 5000% traffic spikes  
**Urgency:** **CRITICAL** - Holiday sales start in 3 weeks, auto-scaling must be bulletproof

### The Challenge

You must implement a comprehensive auto-scaling solution that:

- Automatically scales pods based on CPU, memory, and custom metrics
- Handles rapid traffic spikes without service degradation
- Optimizes resource usage during low-traffic periods to reduce costs
- Implements intelligent configuration management for different traffic patterns
- Ensures zero-downtime scaling operations
- Provides comprehensive monitoring and alerting

**Business Impact:** Successfully handling holiday traffic could generate $50M+ in additional revenue, while failure could damage brand reputation and customer trust permanently.

---

## 🎯 Learning Objectives

By completing this lab, you will:

- [ ] **Primary Skill:** Master Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA) configuration for production workloads
- [ ] **Secondary Skills:** Configure custom metrics autoscaling, manage ConfigMaps for dynamic configuration, implement resource optimization strategies
- [ ] **Real-world Application:** Design enterprise-grade auto-scaling architectures that handle extreme traffic variations
- [ ] **Exam Relevance:** Core CKA workload management including scaling, ConfigMaps, resource management, and deployment strategies

---

## 🔧 Prerequisites

### Knowledge Requirements

- [ ] Basic knowledge of resource requests and limits
- [ ] Familiarity with ConfigMaps and environment variables
- [ ] Previous completion of W01 (Rolling Update Disaster Recovery) recommended

### Environment Setup

```bash
# Cluster requirements
- Kubernetes cluster v1.28+ with metrics-server enabled
- kubectl configured with appropriate permissions
- Helm 3.x for installing monitoring components
- At least 4 worker nodes with 8GB RAM each for scaling tests

# Verification commands
kubectl cluster-info
kubectl top nodes
kubectl get apiservice v1beta1.metrics.k8s.io
helm version
```

---

## 🚨 Initial Crisis State

You arrive to find an e-commerce platform with dangerous scaling limitations:

### Current Infrastructure Problems

1. **Static Resource Allocation** - Fixed 5 replicas regardless of traffic
2. **No Metrics-Based Scaling** - Manual scaling decisions only
3. **Resource Waste** - Over-provisioned during low traffic periods
4. **Traffic Spike Failures** - Insufficient capacity during peak loads
5. **Poor Configuration Management** - Hard-coded values in deployments
6. **No Cost Optimization** - Running expensive resources 24/7

### Immediate Symptoms

```bash
# Check current static deployment
kubectl get deployments -o wide
kubectl describe deployment ecommerce-app

# Verify lack of autoscaling
kubectl get hpa
kubectl get vpa 2>/dev/null || echo "VPA not configured"

# Check resource utilization
kubectl top pods
kubectl top nodes
```

---

## 📋 Tasks & Solutions

### Task 1: Deploy Metrics Server and Monitoring Infrastructure

**Scenario:** Set up the foundation for metrics-based autoscaling with comprehensive monitoring.

#### 📊 Metrics Server Installation

```bash
# 1. Install metrics-server if not present
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# 2. Verify metrics-server is working
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
kubectl top nodes
kubectl top pods -A

# 3. Install Prometheus for custom metrics (using Helm)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set grafana.adminPassword=admin123
```

#### ✅ Custom Metrics API Setup

```yaml
# Install Prometheus Adapter for custom metrics
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: adapter-config
  namespace: monitoring
data:
  config.yaml: |
    rules:
    - seriesQuery: 'http_requests_per_second{namespace!="",pod!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          pod: {resource: "pod"}
      name:
        matches: "^(.*)_per_second"
        as: "${1}_rate"
      metricsQuery: 'sum(<<.Series>>{<<.LabelMatchers>>}) by (<<.GroupBy>>)'
    - seriesQuery: 'nginx_ingress_controller_requests_rate{namespace!="",ingress!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          ingress: {resource: "ingress"}
      name:
        matches: "^(.*)_rate"
        as: "${1}"
      metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-metrics-apiserver
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-metrics-apiserver
  template:
    metadata:
      labels:
        app: custom-metrics-apiserver
    spec:
      serviceAccountName: custom-metrics-apiserver
      containers:
      - name: custom-metrics-apiserver
        image: k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.11.0
        args:
        - --secure-port=6443
        - --tls-cert-file=/var/run/serving-cert/tls.crt
        - --tls-private-key-file=/var/run/serving-cert/tls.key
        - --logtostderr=true
        - --prometheus-url=http://prometheus-kube-prometheus-prometheus.monitoring.svc:9090/
        - --metrics-relist-interval=1m
        - --v=4
        - --config=/etc/adapter/config.yaml
        ports:
        - containerPort: 6443
        volumeMounts:
        - mountPath: /var/run/serving-cert
          name: volume-serving-cert
          readOnly: true
        - mountPath: /etc/adapter/
          name: config
          readOnly: true
      volumes:
      - name: volume-serving-cert
        secret:
          secretName: cm-adapter-serving-certs
      - name: config
        configMap:
          name: adapter-config
EOF
```

### Task 2: Create E-commerce Application with Resource Specifications

**Scenario:** Deploy a realistic e-commerce application with proper resource requests and limits.

#### 🛒 E-commerce Application Deployment

```yaml
# Create comprehensive e-commerce application stack
cat << 'EOF' | kubectl apply -f -
---
# Namespace for e-commerce application
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
  labels:
    environment: production
    team: sre
---
# ConfigMap for application configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-config
  namespace: ecommerce
data:
  # Application settings
  MAX_CONNECTIONS: "1000"
  CACHE_TTL: "300"
  LOG_LEVEL: "info"
  ENABLE_METRICS: "true"
  
  # Database settings
  DB_POOL_SIZE: "20"
  DB_TIMEOUT: "30"
  
  # Performance tuning based on load
  WORKER_PROCESSES: "auto"
  WORKER_CONNECTIONS: "1024"
  
  # Feature flags for traffic scaling
  ENABLE_CACHE: "true"
  ENABLE_CDN: "true"
  ENABLE_COMPRESSION: "true"
---
# Secret for sensitive configuration
apiVersion: v1
kind: Secret
metadata:
  name: ecommerce-secrets
  namespace: ecommerce
type: Opaque
data:
  DB_PASSWORD: cHJvZC1wYXNzd29yZC0xMjM=  # prod-password-123
  API_KEY: c2stbGl2ZS1hcGkta2V5LWFiYzEyMw==  # sk-live-api-key-abc123
  JWT_SECRET: c3VwZXItc2VjcmV0LWp3dC1rZXk=  # super-secret-jwt-key
---
# Frontend deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-frontend
  namespace: ecommerce
  labels:
    app: ecommerce-frontend
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ecommerce-frontend
  template:
    metadata:
      labels:
        app: ecommerce-frontend
        tier: frontend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: frontend
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: http
        - containerPort: 8080
          name: metrics
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        env:
        - name: BACKEND_URL
          value: "http://ecommerce-backend:8080"
        envFrom:
        - configMapRef:
            name: ecommerce-config
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
        - name: app-content
          mountPath: /usr/share/nginx/html
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-frontend-config
      - name: app-content
        configMap:
          name: frontend-content
---
# Backend API deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-backend
  namespace: ecommerce
  labels:
    app: ecommerce-backend
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ecommerce-backend
  template:
    metadata:
      labels:
        app: ecommerce-backend
        tier: backend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9090"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo:0.2.3
        args:
        - "-text=EliteShop Backend API v2.1 - Pod: $(HOSTNAME)"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 9090
          name: metrics
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
        env:
        - name: HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: DATABASE_URL
          value: "postgresql://app:$(DB_PASSWORD)@ecommerce-database:5432/ecommerce"
        envFrom:
        - configMapRef:
            name: ecommerce-config
        - secretRef:
            name: ecommerce-secrets
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
---
# Database deployment (stateful for demo)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecommerce-database
  namespace: ecommerce
  labels:
    app: ecommerce-database
    tier: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecommerce-database
  template:
    metadata:
      labels:
        app: ecommerce-database
        tier: database
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        env:
        - name: POSTGRES_DB
          value: ecommerce
        - name: POSTGRES_USER
          value: app
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ecommerce-secrets
              key: DB_PASSWORD
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-data
        emptyDir: {}
---
# Supporting ConfigMaps
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-frontend-config
  namespace: ecommerce
data:
  default.conf: |
    upstream backend {
        server ecommerce-backend:8080;
    }
    
    server {
        listen 80;
        server_name _;
        
        location /health {
            return 200 'healthy\n';
            add_header Content-Type text/plain;
        }
        
        location /ready {
            return 200 'ready\n';
            add_header Content-Type text/plain;
        }
        
        location /api/ {
            proxy_pass http://backend/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        
        location /metrics {
            stub_status on;
            access_log off;
        }
        
        location / {
            root /usr/share/nginx/html;
            index index.html;
            try_files $uri $uri/ /index.html;
        }
    }
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-content
  namespace: ecommerce
data:
  index.html: |
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>EliteShop Commerce - Auto-Scaling Demo</title>
        <style>
            * { margin: 0; padding: 0; box-sizing: border-box; }
            body { 
                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                color: white;
                min-height: 100vh;
                display: flex;
                align-items: center;
                justify-content: center;
            }
            .container {
                max-width: 1200px;
                width: 90%;
                text-align: center;
                backdrop-filter: blur(10px);
                background: rgba(255,255,255,0.1);
                padding: 60px 40px;
                border-radius: 20px;
                box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            }
            .logo { font-size: 4em; margin-bottom: 20px; }
            .title { font-size: 3em; margin-bottom: 20px; font-weight: 300; }
            .subtitle { font-size: 1.3em; opacity: 0.9; margin-bottom: 40px; }
            .features {
                display: grid;
                grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
                gap: 30px;
                margin-top: 50px;
            }
            .feature {
                background: rgba(255,255,255,0.15);
                padding: 30px;
                border-radius: 15px;
                border: 1px solid rgba(255,255,255,0.2);
                transition: transform 0.3s ease;
            }
            .feature:hover { transform: translateY(-5px); }
            .feature-icon { font-size: 3em; margin-bottom: 15px; }
            .feature h3 { font-size: 1.5em; margin-bottom: 15px; }
            .feature p { opacity: 0.9; line-height: 1.6; }
            .stats {
                display: flex;
                justify-content: space-around;
                margin-top: 40px;
                flex-wrap: wrap;
                gap: 20px;
            }
            .stat {
                background: rgba(255,255,255,0.2);
                padding: 20px;
                border-radius: 10px;
                min-width: 150px;
            }
            .stat-number { font-size: 2em; font-weight: bold; color: #FFD700; }
            .stat-label { font-size: 0.9em; opacity: 0.8; }
            .load-test {
                margin-top: 40px;
                padding: 20px;
                background: rgba(255,255,255,0.1);
                border-radius: 10px;
                border: 2px dashed rgba(255,255,255,0.3);
            }
            .btn {
                background: linear-gradient(45deg, #FF6B6B, #4ECDC4);
                color: white;
                border: none;
                padding: 15px 30px;
                border-radius: 25px;
                font-size: 1.1em;
                cursor: pointer;
                margin: 10px;
                transition: transform 0.3s ease;
            }
            .btn:hover { transform: scale(1.05); }
        </style>
    </head>
    <body>
        <div class="container">
            <div class="logo">🛒</div>
            <h1 class="title">EliteShop Commerce</h1>
            <p class="subtitle">Enterprise Auto-Scaling E-commerce Platform</p>
            
            <div class="features">
                <div class="feature">
                    <div class="feature-icon">📈</div>
                    <h3>Auto-Scaling</h3>
                    <p>Automatically scales from 3 to 50+ pods based on CPU, memory, and custom metrics to handle traffic spikes seamlessly.</p>
                </div>
                <div class="feature">
                    <div class="feature-icon">⚡</div>
                    <h3>High Performance</h3>
                    <p>Optimized for handling 100,000+ concurrent users with intelligent load balancing and resource management.</p>
                </div>
                <div class="feature">
                    <div class="feature-icon">💰</div>
                    <h3>Cost Efficient</h3>
                    <p>Smart scaling reduces costs by 60% during low-traffic periods while maintaining performance during peaks.</p>
                </div>
                <div class="feature">
                    <div class="feature-icon">🔄</div>
                    <h3>Zero Downtime</h3>
                    <p>Rolling updates and graceful scaling ensure 99.99% uptime even during major traffic events.</p>
                </div>
            </div>
            
            <div class="stats">
                <div class="stat">
                    <div class="stat-number" id="current-pods">3</div>
                    <div class="stat-label">Active Pods</div>
                </div>
                <div class="stat">
                    <div class="stat-number" id="cpu-usage">15%</div>
                    <div class="stat-label">CPU Usage</div>
                </div>
                <div class="stat">
                    <div class="stat-number" id="requests-sec">250</div>
                    <div class="stat-label">Requests/sec</div>
                </div>
                <div class="stat">
                    <div class="stat-number">99.9%</div>
                    <div class="stat-label">Uptime</div>
                </div>
            </div>
            
            <div class="load-test">
                <h3>🧪 Load Testing Dashboard</h3>
                <p>Simulate traffic spikes to test auto-scaling behavior</p>
                <button class="btn" onclick="simulateLoad()">🚀 Simulate Black Friday Load</button>
                <button class="btn" onclick="simulateNormal()">📱 Normal Traffic</button>
            </div>
        </div>
        
        <script>
            function simulateLoad() {
                document.getElementById('current-pods').textContent = Math.floor(Math.random() * 30) + 10;
                document.getElementById('cpu-usage').textContent = Math.floor(Math.random() * 60) + 70 + '%';
                document.getElementById('requests-sec').textContent = Math.floor(Math.random() * 8000) + 2000;
            }
            
            function simulateNormal() {
                document.getElementById('current-pods').textContent = Math.floor(Math.random() * 3) + 3;
                document.getElementById('cpu-usage').textContent = Math.floor(Math.random() * 30) + 10 + '%';
                document.getElementById('requests-sec').textContent = Math.floor(Math.random() * 200) + 100;
            }
            
            // Auto-update stats
            setInterval(() => {
                const baseRequests = parseInt(document.getElementById('requests-sec').textContent);
                const variation = Math.floor(Math.random() * 100) - 50;
                document.getElementById('requests-sec').textContent = Math.max(50, baseRequests + variation);
            }, 3000);
        </script>
    </body>
    </html>
EOF
```

### Task 3: Configure Horizontal Pod Autoscaler (HPA)

**Scenario:** Implement intelligent horizontal scaling based on multiple metrics including CPU, memory, and custom metrics.

#### 🔄 Basic HPA Configuration

```yaml
# Create HPA for frontend with CPU and memory metrics
cat << 'EOF' | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-frontend-hpa
  namespace: ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-frontend
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
      selectPolicy: Min
---
# HPA for backend with more aggressive scaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-backend-hpa
  namespace: ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-backend
  minReplicas: 2
  maxReplicas: 30
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 30
      policies:
      - type: Percent
        value: 200
        periodSeconds: 15
      - type: Pods
        value: 8
        periodSeconds: 15
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 180
      policies:
      - type: Percent
        value: 25
        periodSeconds: 30
      selectPolicy: Min
EOF
```

#### 🎯 Advanced Custom Metrics HPA

```yaml
# Custom metrics HPA based on request rate and queue length
cat << 'EOF' | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-custom-metrics-hpa
  namespace: ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-frontend
  minReplicas: 3
  maxReplicas: 100
  metrics:
  # CPU threshold for basic scaling
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  # Custom metric: HTTP requests per second
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  # External metric: Queue depth from external system
  - type: External
    external:
      metric:
        name: queue_messages_ready
        selector:
          matchLabels:
            queue: "orders"
      target:
        type: AverageValue
        averageValue: "30"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 30
      policies:
      - type: Percent
        value: 300
        periodSeconds: 15
      - type: Pods
        value: 10
        periodSeconds: 15
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
      selectPolicy: Min
EOF
```

### Task 4: Configure Vertical Pod Autoscaler (VPA)

**Scenario:** Implement vertical scaling to optimize resource allocation and reduce waste.

#### 📏 VPA Installation and Configuration

```bash
# 1. Install VPA (if not present)
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/
./hack/vpa-install.sh
cd ../../../
```

#### ⬆️ VPA Configuration for Applications

```yaml
# VPA for database with recommendation mode
cat << 'EOF' | kubectl apply -f -
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: ecommerce-database-vpa
  namespace: ecommerce
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-database
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: postgres
      minAllowed:
        cpu: 250m
        memory: 512Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
      controlledValues: RequestsAndLimits
---
# VPA for backend with initial recommendations only
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: ecommerce-backend-vpa
  namespace: ecommerce
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-backend
  updatePolicy:
    updateMode: "Initial"
  resourcePolicy:
    containerPolicies:
    - containerName: backend
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 1Gi
      controlledResources: ["cpu", "memory"]
---
# VPA for frontend with recommendation mode for analysis
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: ecommerce-frontend-vpa
  namespace: ecommerce
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-frontend
  updatePolicy:
    updateMode: "Off"  # Recommendation only for analysis
  resourcePolicy:
    containerPolicies:
    - containerName: frontend
      minAllowed:
        cpu: 50m
        memory: 64Mi
      maxAllowed:
        cpu: 1
        memory: 512Mi
EOF
```

### Task 5: Implement Dynamic Configuration Management

**Scenario:** Create intelligent ConfigMap management that adjusts application behavior based on traffic patterns.

#### 🔧 Traffic-Aware Configuration

```yaml
# Create ConfigMaps for different traffic scenarios
cat << 'EOF' | kubectl apply -f -
---
# Low traffic configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-config-low-traffic
  namespace: ecommerce
  labels:
    traffic-profile: low
data:
  MAX_CONNECTIONS: "500"
  CACHE_TTL: "600"
  LOG_LEVEL: "warn"
  DB_POOL_SIZE: "10"
  WORKER_PROCESSES: "2"
  WORKER_CONNECTIONS: "512"
  ENABLE_CACHE: "true"
  ENABLE_CDN: "false"
  ENABLE_COMPRESSION: "false"
---
# Medium traffic configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-config-medium-traffic
  namespace: ecommerce
  labels:
    traffic-profile: medium
data:
  MAX_CONNECTIONS: "2000"
  CACHE_TTL: "300"
  LOG_LEVEL: "info"
  DB_POOL_SIZE: "20"
  WORKER_PROCESSES: "4"
  WORKER_CONNECTIONS: "1024"
  ENABLE_CACHE: "true"
  ENABLE_CDN: "true"
  ENABLE_COMPRESSION: "true"
---
# High traffic configuration (Black Friday mode)
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-config-high-traffic
  namespace: ecommerce
  labels:
    traffic-profile: high
data:
  MAX_CONNECTIONS: "10000"
  CACHE_TTL: "60"
  LOG_LEVEL: "error"
  DB_POOL_SIZE: "50"
  WORKER_PROCESSES: "auto"
  WORKER_CONNECTIONS: "4096"
  ENABLE_CACHE: "true"
  ENABLE_CDN: "true"
  ENABLE_COMPRESSION: "true"
  # Additional high-traffic optimizations
  ENABLE_REQUEST_THROTTLING: "true"
  ENABLE_CIRCUIT_BREAKER: "true"
  ENABLE_BULK_OPERATIONS: "true"
---
# ConfigMap switching automation job
apiVersion: batch/v1
kind: CronJob
metadata:
  name: traffic-config-switcher
  namespace: ecommerce
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: config-switcher
          containers:
          - name: switcher
            image: bitnami/kubectl:latest
            command:
            - /bin/bash
            - -c
            - |
              #!/bin/bash
              
              # Get current replica count to determine traffic level
              FRONTEND_REPLICAS=$(kubectl get deployment ecommerce-frontend -n ecommerce -o jsonpath='{.spec.replicas}')
              BACKEND_REPLICAS=$(kubectl get deployment ecommerce-backend -n ecommerce -o jsonpath='{.spec.replicas}')
              
              # Determine traffic profile based on replica count
              if [ $FRONTEND_REPLICAS -le 5 ] && [ $BACKEND_REPLICAS -le 3 ]; then
                TRAFFIC_PROFILE="low"
              elif [ $FRONTEND_REPLICAS -le 15 ] && [ $BACKEND_REPLICAS -le 10 ]; then
                TRAFFIC_PROFILE="medium"
              else
                TRAFFIC_PROFILE="high"
              fi
              
              echo "Current replicas: Frontend=$FRONTEND_REPLICAS, Backend=$BACKEND_REPLICAS"
              echo "Selected traffic profile: $TRAFFIC_PROFILE"
              
              # Update deployment to use appropriate ConfigMap
              kubectl patch deployment ecommerce-frontend -n ecommerce -p "{
                \"spec\": {
                  \"template\": {
                    \"spec\": {
                      \"containers\": [{
                        \"name\": \"frontend\",
                        \"envFrom\": [{
                          \"configMapRef\": {
                            \"name\": \"ecommerce-config-${TRAFFIC_PROFILE}-traffic\"
                          }
                        }]
                      }]
                    }
                  }
                }
              }"
              
              kubectl patch deployment ecommerce-backend -n ecommerce -p "{
                \"spec\": {
                  \"template\": {
                    \"spec\": {
                      \"containers\": [{
                        \"name\": \"backend\",
                        \"envFrom\": [{
                          \"configMapRef\": {
                            \"name\": \"ecommerce-config-${TRAFFIC_PROFILE}-traffic\"
                          }
                        }]
                      }]
                    }
                  }
                }
              }"
              
              echo "Configuration updated to $TRAFFIC_PROFILE traffic profile"
          restartPolicy: OnFailure
---
# ServiceAccount for config switcher
apiVersion: v1
kind: ServiceAccount
metadata:
  name: config-switcher
  namespace: ecommerce
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: ecommerce
  name: config-switcher-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "patch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: config-switcher-binding
  namespace: ecommerce
subjects:
- kind: ServiceAccount
  name: config-switcher
  namespace: ecommerce
roleRef:
  kind: Role
  name: config-switcher-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

### Task 6: Load Testing and Auto-scaling Validation

**Scenario:** Perform comprehensive load testing to validate auto-scaling behavior under realistic traffic patterns.

#### 🚀 Load Testing Setup

```bash
# 1. Install and configure load testing tools
kubectl create namespace loadtest

# Deploy load generator
cat << 'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-generator
  namespace: loadtest
spec:
  replicas: 5
  selector:
    matchLabels:
      app: load-generator
  template:
    metadata:
      labels:
        app: load-generator
    spec:
      containers:
      - name: load-gen
        image: williamyeh/wrk
        command: ["sleep", "3600"]
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
EOF
```

#### 🧪 Comprehensive Load Testing Script

```bash
# Create comprehensive load testing script
cat << 'EOF' > load-test-suite.sh
#!/bin/bash
set -e

echo "=== EliteShop Auto-scaling Load Test Suite ==="
echo ""

# Configuration
NAMESPACE="ecommerce"
SERVICE_URL="http://ecommerce-frontend.${NAMESPACE}.svc.cluster.local"
LOADTEST_NAMESPACE="loadtest"

# Function to get current metrics
get_metrics() {
    echo "Current System State:"
    echo "Frontend Replicas: $(kubectl get deployment ecommerce-frontend -n $NAMESPACE -o jsonpath='{.status.replicas}')"
    echo "Backend Replicas: $(kubectl get deployment ecommerce-backend -n $NAMESPACE -o jsonpath='{.status.replicas}')"
    echo "CPU Usage:"
    kubectl top pods -n $NAMESPACE | grep ecommerce
    echo "HPA Status:"
    kubectl get hpa -n $NAMESPACE
    echo ""
}

# Function to wait for scaling
wait_for_scaling() {
    local target_replicas=$1
    local deployment=$2
    local timeout=300
    local count=0
    
    echo "Waiting for $deployment to scale to $target_replicas replicas..."
    while [ $count -lt $timeout ]; do
        current=$(kubectl get deployment $deployment -n $NAMESPACE -o jsonpath='{.status.readyReplicas}')
        if [ "$current" -ge "$target_replicas" ]; then
            echo "✅ $deployment scaled to $current replicas"
            return 0
        fi
        sleep 5
        ((count+=5))
        echo "Current: $current, Target: $target_replicas (${count}s)"
    done
    echo "❌ Timeout waiting for scaling"
    return 1
}

# Test 1: Baseline measurement
echo "=== Test 1: Baseline Measurement ==="
get_metrics
sleep 30

# Test 2: Gradual load increase
echo "=== Test 2: Gradual Load Increase ==="
echo "Starting gradual load increase..."

kubectl exec -n $LOADTEST_NAMESPACE deployment/load-generator -- \
    wrk -t4 -c50 -d120s --latency $SERVICE_URL &

sleep 60
get_metrics
wait_for_scaling 5 ecommerce-frontend

# Test 3: Traffic spike simulation (Black Friday)
echo "=== Test 3: Black Friday Traffic Spike ==="
echo "Simulating Black Friday traffic spike..."

for i in {1..10}; do
    kubectl exec -n $LOADTEST_NAMESPACE deployment/load-generator -- \
        wrk -t8 -c200 -d180s --latency $SERVICE_URL &
done

sleep 120
get_metrics
wait_for_scaling 15 ecommerce-frontend
wait_for_scaling 8 ecommerce-backend

# Test 4: Sustained high load
echo "=== Test 4: Sustained High Load ==="
echo "Testing sustained high load handling..."

for i in {1..15}; do
    kubectl exec -n $LOADTEST_NAMESPACE deployment/load-generator -- \
        wrk -t12 -c500 -d300s --latency $SERVICE_URL &
done

sleep 180
get_metrics

# Expected: Should scale to near maximum replicas
wait_for_scaling 25 ecommerce-frontend
wait_for_scaling 15 ecommerce-backend

# Test 5: Traffic drop and scale-down
echo "=== Test 5: Traffic Drop and Scale-down ==="
echo "Stopping load generators to test scale-down behavior..."

# Kill all load generators
kubectl scale deployment load-generator --replicas=0 -n $LOADTEST_NAMESPACE
kubectl scale deployment load-generator --replicas=5 -n $LOADTEST_NAMESPACE

echo "Waiting 10 minutes for scale-down (HPA stabilization)..."
sleep 600

get_metrics

# Test 6: VPA recommendations analysis
echo "=== Test 6: VPA Recommendations Analysis ==="
echo "VPA Recommendations:"
kubectl describe vpa -n $NAMESPACE

# Test 7: Configuration switching validation
echo "=== Test 7: Configuration Switching Validation ==="
echo "Current ConfigMap usage:"
kubectl get deployment ecommerce-frontend -n $NAMESPACE -o yaml | grep -A 5 envFrom

# Final metrics
echo "=== Final System State ==="
get_metrics

echo "=== Load Test Summary ==="
echo "✅ Gradual scaling tested"
echo "✅ Traffic spike handling tested"
echo "✅ Sustained load capacity tested"
echo "✅ Scale-down behavior tested"
echo "✅ VPA recommendations generated"
echo "✅ Configuration switching validated"

echo ""
echo "Performance Metrics Summary:"
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/namespaces/$NAMESPACE/pods" | jq -r '.items[] | "\(.metadata.name): CPU=\(.containers[0].usage.cpu), Memory=\(.containers[0].usage.memory)"'

EOF

chmod +x load-test-suite.sh
./load-test-suite.sh
```

### Task 7: Monitoring and Alerting Setup

**Scenario:** Implement comprehensive monitoring and alerting for auto-scaling events and performance metrics.

#### 📊 Grafana Dashboard Configuration

```yaml
# Create Grafana dashboard for auto-scaling monitoring
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: autoscaling-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  autoscaling-dashboard.json: |
    {
      "dashboard": {
        "id": null,
        "title": "EliteShop Auto-scaling Dashboard",
        "tags": ["kubernetes", "autoscaling", "ecommerce"],
        "timezone": "browser",
        "panels": [
          {
            "title": "Pod Replicas Over Time",
            "type": "graph",
            "targets": [
              {
                "expr": "kube_deployment_status_replicas{namespace=\"ecommerce\"}",
                "legendFormat": "{{deployment}} replicas"
              }
            ]
          },
          {
            "title": "CPU Utilization",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(container_cpu_usage_seconds_total{namespace=\"ecommerce\"}[5m]) * 100",
                "legendFormat": "{{pod}} CPU %"
              }
            ]
          },
          {
            "title": "Memory Utilization", 
            "type": "graph",
            "targets": [
              {
                "expr": "container_memory_usage_bytes{namespace=\"ecommerce\"} / container_spec_memory_limit_bytes * 100",
                "legendFormat": "{{pod}} Memory %"
              }
            ]
          },
          {
            "title": "HPA Scaling Events",
            "type": "table",
            "targets": [
              {
                "expr": "increase(kube_hpa_status_desired_replicas{namespace=\"ecommerce\"}[5m])",
                "legendFormat": "{{hpa}} scaling events"
              }
            ]
          }
        ]
      }
    }
---
# Prometheus rules for auto-scaling alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: autoscaling-alerts
  namespace: monitoring
spec:
  groups:
  - name: autoscaling.rules
    rules:
    - alert: HighCPUUsage
      expr: rate(container_cpu_usage_seconds_total{namespace="ecommerce"}[5m]) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage detected"
        description: "Pod {{ $labels.pod }} has CPU usage above 80% for more than 2 minutes"
    
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{namespace="ecommerce"} / container_spec_memory_limit_bytes * 100 > 85
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage detected"
        description: "Pod {{ $labels.pod }} has memory usage above 85% for more than 2 minutes"
    
    - alert: HPAScalingFailure
      expr: kube_hpa_status_condition{namespace="ecommerce", condition="ScalingActive", status="false"} == 1
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "HPA scaling failure"
        description: "HPA {{ $labels.hpa }} in namespace {{ $labels.namespace }} failed to scale"
    
    - alert: MaxReplicasReached
      expr: kube_hpa_status_current_replicas{namespace="ecommerce"} >= kube_hpa_spec_max_replicas{namespace="ecommerce"}
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "HPA reached maximum replicas"
        description: "HPA {{ $labels.hpa }} has reached its maximum replica count of {{ $value }}"
    
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total{namespace="ecommerce"}[5m]) > 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Pod crash looping"
        description: "Pod {{ $labels.pod }} is crash looping in namespace {{ $labels.namespace }}"
EOF
```

---

## 🎯 Verification Commands

```bash
# Final comprehensive auto-scaling verification
echo "=== Auto-scaling Infrastructure Audit ==="

# 1. Check HPA status and behavior
kubectl get hpa -n ecommerce -o wide
kubectl describe hpa ecommerce-frontend-hpa -n ecommerce
kubectl describe hpa ecommerce-backend-hpa -n ecommerce

# 2. Verify VPA recommendations
kubectl get vpa -n ecommerce
kubectl describe vpa ecommerce-database-vpa -n ecommerce

# 3. Check current resource utilization
kubectl top pods -n ecommerce
kubectl top nodes

# 4. Verify metrics server functionality
kubectl get apiservice v1beta1.metrics.k8s.io
kubectl top pods -n ecommerce --sort-by=cpu

# 5. Test configuration management
kubectl get configmaps -n ecommerce -l traffic-profile
kubectl describe deployment ecommerce-frontend -n ecommerce | grep -A 10 "Environment"

# 6. Check scaling events in HPA history
kubectl get events -n ecommerce --field-selector reason=SuccessfulRescale
kubectl describe hpa -n ecommerce | grep -A 10 "Events"

# 7. Validate services are accessible
kubectl get services -n ecommerce
kubectl port-forward -n ecommerce service/ecommerce-frontend 8080:80 &
curl http://localhost:8080

# 8. Verify monitoring setup
kubectl get servicemonitors -n monitoring
kubectl get prometheusrules -n monitoring
```

---

## 💡 Key Learning Points

### Auto-scaling Architecture Mastered

1. **Horizontal Pod Autoscaler (HPA):** Multi-metric scaling with CPU, memory, and custom metrics
2. **Vertical Pod Autoscaler (VPA):** Resource optimization and right-sizing
3. **Configuration Management:** Dynamic ConfigMap switching based on traffic patterns
4. **Load Testing:** Comprehensive validation of scaling behavior under realistic loads
5. **Monitoring Integration:** Grafana dashboards and Prometheus alerting for scaling events
6. **Performance Optimization:** Cost reduction during low traffic, performance during peaks

### Production Auto-scaling Best Practices

- **Scaling Policies:** Conservative scale-down, aggressive scale-up for traffic spikes
- **Resource Management:** Proper requests/limits for predictable scaling behavior
- **Metrics Strategy:** Multi-dimensional scaling based on business-relevant metrics
- **Configuration Automation:** Traffic-aware configuration management
- **Monitoring:** Comprehensive observability for scaling decisions and performance

### Common Auto-scaling Pitfalls Avoided

- **Avoided:** Aggressive scale-down causing service disruption
- **Avoided:** Missing resource requests/limits preventing HPA function
- **Avoided:** Single-metric scaling missing important performance indicators
- **Avoided:** Static configuration not adapting to traffic patterns
- **Avoided:** Lack of monitoring for scaling events and failures

---

## 🚀 Bonus Challenges

### Advanced Auto-scaling Features

1. **Predictive Scaling:** Implement machine learning-based traffic prediction
2. **Multi-Cluster Scaling:** Scale across multiple Kubernetes clusters
3. **Custom Metrics:** Integrate business metrics (orders/second, revenue/minute)
4. **Chaos Engineering:** Test auto-scaling under failure conditions
5. **Cost Optimization:** Advanced algorithms for cost-efficient scaling

### Enterprise Extensions

1. **SLA-Based Scaling:** Scale based on service level objectives
2. **Geographic Scaling:** Region-aware scaling for global applications
3. **Database Auto-scaling:** Extend auto-scaling to database layers
4. **Security Integration:** Scale security controls with application load

---

## 📚 Additional Resources

- [Kubernetes HPA Documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
- [Custom Metrics API](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/custom-metrics-api.md)
- [CKA Exam Scaling Topics](https://github.com/cncf/curriculum/blob/master/CKA_Curriculum_v1.28.pdf)

**Next Lab:** S02 - Database Migration Scenario (Advanced storage and volume management for data persistence)

---

*🎯 **Success Criteria:** You've successfully implemented enterprise-grade auto-scaling that can handle EliteShop's holiday traffic surge. The system now automatically scales from 3 to 50+ pods based on demand, optimizes resource usage during low traffic, and provides comprehensive monitoring. EliteShop is ready to handle 5000% traffic spikes without losing a single sale.*
