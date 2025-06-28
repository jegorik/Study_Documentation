# N05 - Load Balancer Configuration

> **Scenario:** High-traffic e-commerce platform requiring advanced load balancing  
> **Difficulty:** Advanced  
> **Estimated Time:** 25-30 minutes  
> **Exam Relevance:** Very High - Service types, ingress controllers, traffic distribution

---

## 🚨 Mission Critical Scenario

**Company:** MegaShop Global - Leading e-commerce platform  
**Situation:** Black Friday traffic surge overwhelming current load balancing infrastructure  
**Business Impact:** $150,000/minute revenue loss during peak shopping hours  
**Time Pressure:** Advanced load balancing must be operational before next traffic wave in 30 minutes

Your e-commerce platform is experiencing unprecedented traffic during Black Friday sales. The current basic load balancing setup is failing under the massive load, causing checkout failures and customer abandonment. You need to implement sophisticated load balancing strategies including:

- Layer 4 and Layer 7 load balancing
- Session affinity for shopping carts
- Health-based traffic routing
- Geographic traffic distribution
- Auto-scaling integration

The marketing team has spent millions on advertising driving traffic to the site - failure is not an option.

---

## 🎯 Learning Objectives

- **Service Types:** Master LoadBalancer, NodePort, and ClusterIP configurations
- **Ingress Controllers:** Configure advanced ingress with multiple backends
- **Traffic Distribution:** Implement weighted routing and session affinity
- **Health Checks:** Configure health-based load balancing
- **Auto-scaling Integration:** Connect load balancers with HPA/VPA

---

## 📋 Pre-Lab Setup

### Environment Requirements

- Kubernetes cluster (v1.26+)
- Ingress controller capability (NGINX/Traefik)
- kubectl configured and functional
- MetalLB or cloud load balancer support

### Initial Infrastructure State

```bash
# Create the ecommerce namespace for our application
kubectl create namespace ecommerce

# Label the namespace for organization
kubectl label namespace ecommerce app=megashop environment=production
```

---

## 🔧 Mission Tasks

### **Task 1: Multi-Tier Application Deployment** ⏱️ *6-8 minutes*

**Difficulty:** Intermediate  
**Objective:** Deploy a complete e-commerce application stack requiring different load balancing strategies

#### Your Mission

Deploy the core e-commerce components: frontend web servers, API gateways, product catalog service, and checkout service. Each tier requires different load balancing approaches.

#### Step-by-Step Instructions

1. **Deploy the frontend web server tier:**

```bash
# Create frontend deployment with multiple replicas
cat > /tmp/frontend-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-web
  namespace: ecommerce
  labels:
    app: frontend
    tier: web
spec:
  replicas: 4
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: nginx-frontend
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: frontend-config
          mountPath: /usr/share/nginx/html
      volumes:
      - name: frontend-config
        configMap:
          name: frontend-content
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-content
  namespace: ecommerce
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head><title>MegaShop - Black Friday Sale!</title></head>
    <body>
        <h1>Welcome to MegaShop Black Friday</h1>
        <p>Server: $(hostname)</p>
        <p>Served by: Frontend Web Tier</p>
    </body>
    </html>
  health: |
    OK
  ready: |
    OK
EOF

kubectl apply -f /tmp/frontend-deployment.yaml
```

2. **Deploy the API gateway tier:**

```bash
# Create API gateway with session affinity requirements
cat > /tmp/api-gateway-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: ecommerce
  labels:
    app: api-gateway
    tier: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
      tier: api
  template:
    metadata:
      labels:
        app: api-gateway
        tier: api
    spec:
      containers:
      - name: api-server
        image: nginx:alpine
        ports:
        - containerPort: 8080
        env:
        - name: API_VERSION
          value: "v2.1"
        - name: SESSION_TIMEOUT
          value: "1800"
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "400m"
        livenessProbe:
          httpGet:
            path: /api/health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        volumeMounts:
        - name: api-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: api-config
        configMap:
          name: api-gateway-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-gateway-config
  namespace: ecommerce
data:
  default.conf: |
    server {
        listen 8080;
        location /api/health {
            return 200 "API Gateway Healthy\n";
            add_header Content-Type text/plain;
        }
        location /api/ready {
            return 200 "API Gateway Ready\n";
            add_header Content-Type text/plain;
        }
        location /api/ {
            return 200 "API Gateway v2.1 - Session Enabled\n";
            add_header Content-Type text/plain;
        }
    }
EOF

kubectl apply -f /tmp/api-gateway-deployment.yaml
```

3. **Deploy the checkout service with specific load balancing needs:**

```bash
# Create checkout service requiring sticky sessions
cat > /tmp/checkout-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkout-service
  namespace: ecommerce
  labels:
    app: checkout
    tier: service
spec:
  replicas: 5
  selector:
    matchLabels:
      app: checkout
      tier: service
  template:
    metadata:
      labels:
        app: checkout
        tier: service
    spec:
      containers:
      - name: checkout-app
        image: nginx:alpine
        ports:
        - containerPort: 9000
        env:
        - name: CHECKOUT_VERSION
          value: "3.2"
        - name: PAYMENT_TIMEOUT
          value: "300"
        resources:
          requests:
            memory: "512Mi"
            cpu: "300m"
          limits:
            memory: "1Gi"
            cpu: "600m"
        livenessProbe:
          httpGet:
            path: /checkout/health
            port: 9000
          initialDelaySeconds: 20
          periodSeconds: 15
        readinessProbe:
          httpGet:
            path: /checkout/ready
            port: 9000
          initialDelaySeconds: 15
          periodSeconds: 10
        volumeMounts:
        - name: checkout-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: checkout-config
        configMap:
          name: checkout-service-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: checkout-service-config
  namespace: ecommerce
data:
  default.conf: |
    server {
        listen 9000;
        location /checkout/health {
            return 200 "Checkout Service Healthy\n";
            add_header Content-Type text/plain;
        }
        location /checkout/ready {
            return 200 "Checkout Service Ready\n";
            add_header Content-Type text/plain;
        }
        location /checkout/ {
            return 200 "Checkout v3.2 - Sticky Session Required\n";
            add_header Content-Type text/plain;
            add_header X-Checkout-Server $hostname;
        }
    }
EOF

kubectl apply -f /tmp/checkout-deployment.yaml
```

4. **Verify all deployments are ready:**

```bash
# Check deployment status
kubectl get deployments -n ecommerce
kubectl get pods -n ecommerce -o wide

# Wait for all pods to be ready
kubectl wait --for=condition=ready pod -l app=frontend -n ecommerce --timeout=120s
kubectl wait --for=condition=ready pod -l app=api-gateway -n ecommerce --timeout=120s
kubectl wait --for=condition=ready pod -l app=checkout -n ecommerce --timeout=120s
```

#### Expected Outcome:

- Frontend web tier deployed with 4 replicas
- API gateway tier deployed with 3 replicas
- Checkout service deployed with 5 replicas
- All pods healthy and ready for traffic

---

### **Task 2: Basic Load Balancer Service Configuration** ⏱️ *5-7 minutes*

**Difficulty:** Intermediate  
**Objective:** Configure different service types with appropriate load balancing methods

#### Your Mission

Create services for each application tier using the most appropriate service type and load balancing configuration for their specific requirements.

#### Step-by-Step Instructions

1. **Create LoadBalancer service for frontend (public access):**

```bash
# Create external-facing LoadBalancer for frontend
cat > /tmp/frontend-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: frontend-loadbalancer
  namespace: ecommerce
  labels:
    app: frontend
    service-type: external
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
spec:
  type: LoadBalancer
  selector:
    app: frontend
    tier: web
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  sessionAffinity: None
  loadBalancerSourceRanges:
  - 0.0.0.0/0
EOF

kubectl apply -f /tmp/frontend-service.yaml
```

2. **Create ClusterIP service for API gateway (internal with session affinity):**

```bash
# Create internal service with session affinity for API gateway
cat > /tmp/api-gateway-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: ecommerce
  labels:
    app: api-gateway
    service-type: internal
spec:
  type: ClusterIP
  selector:
    app: api-gateway
    tier: api
  ports:
  - name: api-port
    port: 8080
    targetPort: 8080
    protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 1800
EOF

kubectl apply -f /tmp/api-gateway-service.yaml
```

3. **Create NodePort service for checkout (with specific node access):**

```bash
# Create NodePort service for checkout with sticky sessions
cat > /tmp/checkout-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: checkout-nodeport
  namespace: ecommerce
  labels:
    app: checkout
    service-type: nodeport
  annotations:
    service.kubernetes.io/topology-aware-hints: auto
spec:
  type: NodePort
  selector:
    app: checkout
    tier: service
  ports:
  - name: checkout-port
    port: 9000
    targetPort: 9000
    nodePort: 30900
    protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 300
  externalTrafficPolicy: Local
EOF

kubectl apply -f /tmp/checkout-service.yaml
```

4. **Create headless service for direct pod access (monitoring/debugging):**

```bash
# Create headless service for direct pod communication
cat > /tmp/headless-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: checkout-headless
  namespace: ecommerce
  labels:
    app: checkout
    service-type: headless
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: checkout
    tier: service
  ports:
  - name: checkout-direct
    port: 9000
    targetPort: 9000
    protocol: TCP
EOF

kubectl apply -f /tmp/headless-service.yaml
```

5. **Verify service configurations:**

```bash
# Check all services
kubectl get services -n ecommerce -o wide

# Get detailed service information
kubectl describe service frontend-loadbalancer -n ecommerce
kubectl describe service api-gateway-service -n ecommerce
kubectl describe service checkout-nodeport -n ecommerce

# Check endpoints
kubectl get endpoints -n ecommerce
```

#### Expected Outcome:

- LoadBalancer service configured for external frontend access
- ClusterIP service with session affinity for API gateway
- NodePort service with local traffic policy for checkout
- Headless service for direct pod access
- All services properly routing to healthy endpoints

---

### **Task 3: Advanced Ingress Controller Configuration** ⏱️ *7-9 minutes*

**Difficulty:** Advanced  
**Objective:** Implement sophisticated ingress routing with multiple backends and advanced features

#### Your Mission

Configure an ingress controller with advanced routing rules, SSL termination, rate limiting, and weighted traffic distribution for A/B testing scenarios.

#### Step-by-Step Instructions

1. **Install NGINX Ingress Controller (if not present):**

```bash
# Install NGINX ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Wait for ingress controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

2. **Create TLS secret for HTTPS termination:**

```bash
# Create self-signed certificate for demonstration
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /tmp/megashop.key \
  -out /tmp/megashop.crt \
  -subj "/CN=megashop.com/O=megashop"

# Create Kubernetes TLS secret
kubectl create secret tls megashop-tls \
  --key /tmp/megashop.key \
  --cert /tmp/megashop.crt \
  -n ecommerce
```

3. **Configure main ingress with path-based routing:**

```bash
# Create comprehensive ingress configuration
cat > /tmp/main-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: megashop-ingress
  namespace: ecommerce
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/upstream-hash-by: "$binary_remote_addr"
    nginx.ingress.kubernetes.io/session-cookie-name: "megashop-session"
    nginx.ingress.kubernetes.io/session-cookie-expires: "1800"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "1800"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
spec:
  tls:
  - hosts:
    - megashop.com
    - api.megashop.com
    secretName: megashop-tls
  rules:
  - host: megashop.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-loadbalancer
            port:
              number: 80
      - path: /checkout
        pathType: Prefix
        backend:
          service:
            name: checkout-nodeport
            port:
              number: 9000
  - host: api.megashop.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-gateway-service
            port:
              number: 8080
EOF

kubectl apply -f /tmp/main-ingress.yaml
```

4. **Create weighted ingress for A/B testing:**

```bash
# Deploy canary version of frontend for A/B testing
cat > /tmp/frontend-canary.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-canary
  namespace: ecommerce
  labels:
    app: frontend-canary
    tier: web
    version: canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend-canary
      tier: web
      version: canary
  template:
    metadata:
      labels:
        app: frontend-canary
        tier: web
        version: canary
    spec:
      containers:
      - name: nginx-frontend-canary
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: canary-content
          mountPath: /usr/share/nginx/html
      volumes:
      - name: canary-content
        configMap:
          name: frontend-canary-content
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-canary-content
  namespace: ecommerce
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head><title>MegaShop - NEW Black Friday Experience!</title></head>
    <body style="background-color: #f0f8ff;">
        <h1>🚀 NEW Black Friday Experience!</h1>
        <p>Server: $(hostname)</p>
        <p>Version: CANARY - Enhanced Performance</p>
    </body>
    </html>
  health: |
    OK
  ready: |
    OK
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-canary-service
  namespace: ecommerce
spec:
  selector:
    app: frontend-canary
    tier: web
    version: canary
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl apply -f /tmp/frontend-canary.yaml

# Create canary ingress for A/B testing (10% traffic)
cat > /tmp/canary-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: megashop-canary-ingress
  namespace: ecommerce
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
    nginx.ingress.kubernetes.io/canary-by-header: "canary"
    nginx.ingress.kubernetes.io/canary-by-cookie: "canary-user"
spec:
  tls:
  - hosts:
    - megashop.com
    secretName: megashop-tls
  rules:
  - host: megashop.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-canary-service
            port:
              number: 80
EOF

kubectl apply -f /tmp/canary-ingress.yaml
```

5. **Verify ingress configurations:**

```bash
# Check ingress resources
kubectl get ingress -n ecommerce

# Get detailed ingress information
kubectl describe ingress megashop-ingress -n ecommerce
kubectl describe ingress megashop-canary-ingress -n ecommerce

# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=20
```

#### Expected Outcome:

- NGINX ingress controller deployed and operational
- TLS certificates configured for HTTPS termination
- Path-based routing configured for different services
- Canary deployment configured for A/B testing
- Rate limiting and security features enabled

---

### **Task 4: Advanced Load Balancing with Health Checks** ⏱️ *5-7 minutes*

**Difficulty:** Advanced  
**Objective:** Implement sophisticated health checking and traffic distribution policies

#### Your Mission

Configure advanced health checking mechanisms and implement custom load balancing algorithms to ensure optimal traffic distribution and automatic failover.

#### Step-by-Step Instructions

1. **Configure advanced health checks with custom probe endpoints:**

```bash
# Create health check monitoring deployment
cat > /tmp/health-monitor.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: health-monitor
  namespace: ecommerce
  labels:
    app: health-monitor
    component: monitoring
spec:
  replicas: 2
  selector:
    matchLabels:
      app: health-monitor
  template:
    metadata:
      labels:
        app: health-monitor
        component: monitoring
    spec:
      containers:
      - name: health-checker
        image: nginx:alpine
        ports:
        - containerPort: 8090
        livenessProbe:
          httpGet:
            path: /monitor/health
            port: 8090
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /monitor/ready
            port: 8090
          initialDelaySeconds: 5
          periodSeconds: 3
          timeoutSeconds: 2
          successThreshold: 2
          failureThreshold: 2
        startupProbe:
          httpGet:
            path: /monitor/startup
            port: 8090
          initialDelaySeconds: 5
          periodSeconds: 2
          timeoutSeconds: 1
          failureThreshold: 10
        volumeMounts:
        - name: health-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: health-config
        configMap:
          name: health-monitor-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: health-monitor-config
  namespace: ecommerce
data:
  default.conf: |
    server {
        listen 8090;
        location /monitor/health {
            access_log off;
            return 200 "Health Monitor OK\n";
            add_header Content-Type text/plain;
        }
        location /monitor/ready {
            access_log off;
            return 200 "Monitor Ready\n";
            add_header Content-Type text/plain;
        }
        location /monitor/startup {
            access_log off;
            return 200 "Monitor Started\n";
            add_header Content-Type text/plain;
        }
        location /metrics {
            return 200 "# Health metrics\nhealth_status 1\nready_status 1\n";
            add_header Content-Type text/plain;
        }
    }
---
apiVersion: v1
kind: Service
metadata:
  name: health-monitor-service
  namespace: ecommerce
spec:
  selector:
    app: health-monitor
  ports:
  - port: 8090
    targetPort: 8090
  type: ClusterIP
EOF

kubectl apply -f /tmp/health-monitor.yaml
```

2. **Implement custom load balancing with EndpointSlices:**

```bash
# Create custom endpoint slice for advanced load balancing
cat > /tmp/custom-endpoints.yaml << 'EOF'
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: checkout-weighted-endpoints
  namespace: ecommerce
  labels:
    kubernetes.io/service-name: checkout-weighted-service
addressType: IPv4
ports:
- name: checkout
  port: 9000
  protocol: TCP
endpoints:
- addresses:
  - "10.244.1.10"  # Example IP - in real scenario, these would be actual pod IPs
  conditions:
    ready: true
    serving: true
    terminating: false
  zone: "zone-a"
  hints:
    forZones:
    - name: "zone-a"
- addresses:
  - "10.244.1.11"
  conditions:
    ready: true
    serving: true
    terminating: false
  zone: "zone-b"
  hints:
    forZones:
    - name: "zone-b"
---
apiVersion: v1
kind: Service
metadata:
  name: checkout-weighted-service
  namespace: ecommerce
  annotations:
    service.kubernetes.io/topology-aware-hints: "auto"
    service.kubernetes.io/topology-mode: "auto"
spec:
  selector:
    app: checkout
    tier: service
  ports:
  - port: 9000
    targetPort: 9000
  type: ClusterIP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 300
  internalTrafficPolicy: Local
EOF

kubectl apply -f /tmp/custom-endpoints.yaml
```

3. **Configure service mesh-style load balancing with annotations:**

```bash
# Create service with advanced load balancing annotations
cat > /tmp/advanced-lb-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-advanced
  namespace: ecommerce
  annotations:
    # Advanced load balancing configuration
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: "2"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "10"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "30"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-path: "/api/health"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
    # Custom load balancing algorithm
    traefik.ingress.kubernetes.io/load-balancer-method: "wrr"  # Weighted Round Robin
    nginx.ingress.kubernetes.io/upstream-hash-by: "$request_uri"
    nginx.ingress.kubernetes.io/upstream-hash-by-subset: "true"
spec:
  type: LoadBalancer
  selector:
    app: api-gateway
    tier: api
  ports:
  - name: api-http
    port: 8080
    targetPort: 8080
    protocol: TCP
  sessionAffinity: None
  loadBalancerSourceRanges:
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16
  externalTrafficPolicy: Cluster
EOF

kubectl apply -f /tmp/advanced-lb-service.yaml
```

4. **Verify health check and load balancing configurations:**

```bash
# Test health check endpoints
kubectl get pods -n ecommerce -l app=health-monitor

# Check endpoint slice configuration
kubectl get endpointslices -n ecommerce -o wide

# Verify advanced service configurations
kubectl describe service api-gateway-advanced -n ecommerce

# Test load balancing by checking endpoint distribution
kubectl get endpoints -n ecommerce -o yaml | grep -A10 -B5 addresses
```

#### Expected Outcome:

- Advanced health monitoring system deployed
- Custom endpoint slices configured for weighted routing
- Service mesh-style load balancing annotations applied
- Health check verification and failover mechanisms active

---

### **Task 5: Auto-scaling Integration and Performance Validation** ⏱️ *6-8 minutes*

**Difficulty:** Expert  
**Objective:** Integrate load balancers with auto-scaling and validate performance under load

#### Your Mission

Connect your load balancing infrastructure with Horizontal Pod Autoscaler and Vertical Pod Autoscaler to handle dynamic traffic patterns. Validate the entire system under simulated load conditions.

#### Step-by-Step Instructions

1. **Configure Horizontal Pod Autoscaler for all tiers:**

```bash
# Create HPA for frontend tier
kubectl autoscale deployment frontend-web -n ecommerce \
  --cpu-percent=70 \
  --min=4 \
  --max=20

# Create HPA for API gateway
kubectl autoscale deployment api-gateway -n ecommerce \
  --cpu-percent=60 \
  --min=3 \
  --max=15

# Create HPA for checkout service
kubectl autoscale deployment checkout-service -n ecommerce \
  --cpu-percent=80 \
  --min=5 \
  --max=25

# Create advanced HPA with custom metrics
cat > /tmp/advanced-hpa.yaml << 'EOF'
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: checkout-advanced-hpa
  namespace: ecommerce
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: checkout-service
  minReplicas: 5
  maxReplicas: 30
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
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 30
      - type: Pods
        value: 5
        periodSeconds: 30
      selectPolicy: Max
EOF

kubectl apply -f /tmp/advanced-hpa.yaml
```

2. **Create load testing deployment to validate scaling:**

```bash
# Deploy load testing tool
cat > /tmp/load-tester.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-tester
  namespace: ecommerce
  labels:
    app: load-tester
spec:
  replicas: 3
  selector:
    matchLabels:
      app: load-tester
  template:
    metadata:
      labels:
        app: load-tester
    spec:
      containers:
      - name: load-generator
        image: busybox:latest
        command: ['sh', '-c']
        args:
        - |
          while true; do
            # Simulate frontend load
            wget -q -O- http://frontend-loadbalancer.ecommerce.svc.cluster.local/ || true
            
            # Simulate API load
            wget -q -O- http://api-gateway-service.ecommerce.svc.cluster.local:8080/api/health || true
            
            # Simulate checkout load
            wget -q -O- http://checkout-nodeport.ecommerce.svc.cluster.local:9000/checkout/ready || true
            
            sleep 0.1
          done
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: load-tester-service
  namespace: ecommerce
spec:
  selector:
    app: load-tester
  ports:
  - port: 80
EOF

kubectl apply -f /tmp/load-tester.yaml
```

3. **Monitor auto-scaling behavior:**

```bash
# Watch HPA scaling in real-time
kubectl get hpa -n ecommerce --watch &
HPA_WATCH_PID=$!

# Monitor pod scaling
kubectl get pods -n ecommerce -l app=checkout --watch &
POD_WATCH_PID=$!

# Wait and observe scaling behavior
echo "Monitoring auto-scaling for 60 seconds..."
sleep 60

# Stop monitoring
kill $HPA_WATCH_PID $POD_WATCH_PID 2>/dev/null || true
```

4. **Validate load balancer performance and distribution:**

```bash
# Create performance validation script
cat > /tmp/validate-lb.sh << 'EOF'
#!/bin/bash
echo "=== Load Balancer Performance Validation ==="

# Test frontend load balancing
echo "Testing Frontend Load Distribution:"
for i in {1..10}; do
  kubectl exec -n ecommerce deployment/load-tester -- wget -q -O- http://frontend-loadbalancer.ecommerce.svc.cluster.local/ | grep "Server:" || echo "Request $i failed"
done

# Test API gateway session affinity
echo -e "\nTesting API Gateway Session Affinity:"
for i in {1..5}; do
  kubectl exec -n ecommerce deployment/load-tester -- wget -q -O- --header="Cookie: megashop-session=test123" http://api-gateway-service.ecommerce.svc.cluster.local:8080/api/ || echo "Session test $i failed"
done

# Test checkout service health
echo -e "\nTesting Checkout Service Health:"
kubectl exec -n ecommerce deployment/load-tester -- wget -q -O- http://checkout-nodeport.ecommerce.svc.cluster.local:9000/checkout/health || echo "Checkout health check failed"

# Check ingress accessibility
echo -e "\nTesting Ingress Configuration:"
kubectl get ingress -n ecommerce -o jsonpath='{.items[*].status.loadBalancer.ingress[*].ip}' 2>/dev/null || echo "Ingress IP not available (normal in some environments)"

echo -e "\n=== Validation Complete ==="
EOF

chmod +x /tmp/validate-lb.sh
/tmp/validate-lb.sh
```

5. **Generate comprehensive load balancing report:**

```bash
# Create final validation report
cat > /tmp/lb-configuration-report.md << 'EOF'
# Load Balancer Configuration Report

## Deployment Status
$(kubectl get deployments -n ecommerce -o wide)

## Service Configuration
$(kubectl get services -n ecommerce -o wide)

## Ingress Configuration  
$(kubectl get ingress -n ecommerce)

## Auto-scaling Status
$(kubectl get hpa -n ecommerce)

## Endpoint Distribution
$(kubectl get endpoints -n ecommerce)

## Performance Metrics
- Frontend Tier: LoadBalancer service with 4-20 replicas
- API Gateway: ClusterIP with session affinity, 3-15 replicas  
- Checkout Service: NodePort with sticky sessions, 5-25 replicas
- Health Monitoring: Active with custom probes
- Auto-scaling: Configured with advanced policies

## Load Balancing Features Implemented
✅ Multiple service types (LoadBalancer, ClusterIP, NodePort, Headless)
✅ Session affinity and sticky sessions
✅ Health-based routing with custom probes
✅ Path-based ingress routing
✅ A/B testing with canary deployments
✅ TLS termination and rate limiting
✅ Auto-scaling integration
✅ Weighted traffic distribution
✅ Geographic traffic routing hints

## Black Friday Readiness
- System can handle traffic surge with auto-scaling
- Session persistence maintains shopping cart state
- Health checks ensure traffic only reaches healthy pods
- A/B testing allows performance optimization
- Load distribution prevents single points of failure

Status: READY FOR BLACK FRIDAY TRAFFIC 🚀
EOF

cat /tmp/lb-configuration-report.md
```

#### Expected Outcome:

- HPA configured for all application tiers
- Load testing deployment generating realistic traffic
- Auto-scaling behavior validated under load
- Performance metrics collected and analyzed
- Comprehensive load balancing system ready for production traffic

---

## 🧪 Troubleshooting Guide

### Common Issues and Solutions

#### **Issue: "Ingress controller not ready"**

```bash
# Check ingress controller status
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Restart ingress controller if needed
kubectl rollout restart deployment/ingress-nginx-controller -n ingress-nginx
```

#### **Issue: "LoadBalancer service pending"**

```bash
# Check cloud provider configuration
kubectl describe service frontend-loadbalancer -n ecommerce

# For local testing, use port-forward instead
kubectl port-forward service/frontend-loadbalancer 8080:80 -n ecommerce
```

#### **Issue: "Session affinity not working"**

```bash
# Verify service configuration
kubectl get service api-gateway-service -n ecommerce -o yaml | grep -A5 sessionAffinity

# Test session persistence
kubectl exec -n ecommerce deployment/load-tester -- wget -v --header="Cookie: test=session123" http://api-gateway-service.ecommerce.svc.cluster.local:8080/api/
```

#### **Issue: "HPA not scaling"**

```bash
# Check metrics server
kubectl top nodes
kubectl top pods -n ecommerce

# Verify HPA configuration
kubectl describe hpa -n ecommerce
kubectl get events -n ecommerce | grep -i hpa
```

---

## 🎓 Knowledge Check

### Key Concepts Reinforced

1. **Service Types and Load Balancing:**
   - LoadBalancer vs ClusterIP vs NodePort vs Headless
   - Session affinity and traffic policies
   - External traffic handling strategies

2. **Ingress Configuration:**
   - Path-based and host-based routing
   - TLS termination and security features
   - Canary deployments and A/B testing

3. **Health Checks and Reliability:**
   - Liveness, readiness, and startup probes
   - Custom health check endpoints
   - Failover and traffic distribution

4. **Auto-scaling Integration:**
   - HPA configuration with custom metrics
   - Scaling policies and behavior tuning
   - Load testing and validation

### Exam Tips

- **Service Types:** Understand when to use each service type and their limitations
- **Session Affinity:** Know how to configure sticky sessions for stateful applications
- **Ingress Features:** Master path-based routing, TLS, and advanced annotations
- **Health Checks:** Configure appropriate probe settings for different application types
- **Auto-scaling:** Integrate load balancers with HPA for dynamic capacity management

---

## 🏆 Success Criteria

### Task Completion Checklist

- [ ] **Task 1:** Multi-tier application deployed with appropriate configurations
- [ ] **Task 2:** Different service types configured with correct load balancing
- [ ] **Task 3:** Advanced ingress routing with TLS and canary deployment
- [ ] **Task 4:** Health checks and custom load balancing policies implemented
- [ ] **Task 5:** Auto-scaling integration validated under load

### Load Balancing Validation

- [ ] All service types functioning correctly
- [ ] Session affinity working for stateful services
- [ ] Ingress routing traffic appropriately
- [ ] Health checks preventing traffic to unhealthy pods
- [ ] Auto-scaling responding to load changes

### Time Benchmarks

- **Basic proficiency:** Complete in 35 minutes
- **Intermediate:** Complete in 30 minutes  
- **Advanced:** Complete in 25 minutes
- **Expert:** Complete in 20 minutes

---

## 🚀 Extensions & Real-World Applications

### Production Scenarios

- **Multi-Region Load Balancing:** Geographic traffic distribution
- **Blue-Green Deployments:** Zero-downtime deployment strategies
- **Circuit Breaker Patterns:** Fault tolerance and graceful degradation
- **Rate Limiting:** DDoS protection and API throttling

### Advanced Techniques

- **Service Mesh Integration:** Istio/Linkerd load balancing
- **Custom Load Balancers:** MetalLB configuration
- **CDN Integration:** CloudFlare/CloudFront integration
- **Performance Optimization:** Connection pooling and keep-alive tuning

### Integration Points

- **Monitoring:** Prometheus metrics and Grafana dashboards
- **Logging:** Centralized load balancer log analysis
- **Security:** WAF integration and SSL/TLS management
- **Automation:** GitOps-based traffic management

---

*💡 **Pro Tip:** In production load balancing, always implement comprehensive health checks and gradual traffic shifting. Monitor key metrics like response time, error rate, and connection count to ensure optimal performance.*

---

**🎯 Mastery Goal:** After completing this lab, you should be able to design and implement sophisticated load balancing solutions for production Kubernetes environments, handling complex traffic patterns with high availability and optimal performance.
