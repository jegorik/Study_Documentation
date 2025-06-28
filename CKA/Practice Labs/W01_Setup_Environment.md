# W01 Setup Environment: Rolling Update Disaster Recovery

> **🎯 Scenario Setup:** GameTech Studios production deployment crisis simulation  
> **⚡ Purpose:** Create a realistic failed deployment scenario for emergency rollback practice  
> **⏱️ Setup Time:** 5-8 minutes  

---

## 🎮 Scenario Context

This setup creates a **GameTech Studios** production environment where:

- A working game server API is running successfully
- A new deployment will be pushed that introduces critical failures  
- Students must emergency rollback during "peak gaming hours"
- Realistic production constraints and business pressure

---

## 🚀 Quick Setup (Copy & Paste)

### **Step 1: Create Namespace and Initial Resources**

```bash
# Create dedicated namespace
kubectl create namespace gametech-prod

# Set default namespace for convenience
kubectl config set-context --current --namespace=gametech-prod

# Create a working stable deployment (v1.2.5)
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gameserver-api
  namespace: gametech-prod
  labels:
    app: gameserver-api
    version: v1.2.5
spec:
  replicas: 6
  selector:
    matchLabels:
      app: gameserver-api
  template:
    metadata:
      labels:
        app: gameserver-api
        version: v1.2.5
    spec:
      containers:
      - name: gameserver-api
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: VERSION
          value: "v1.2.5-stable"
        - name: ENVIRONMENT
          value: "production"
        - name: MAX_CONNECTIONS
          value: "1000"
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
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
EOF

# Create service for the game server
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: gameserver-api-service
  namespace: gametech-prod
  labels:
    app: gameserver-api
spec:
  selector:
    app: gameserver-api
  ports:
  - port: 80
    targetPort: 80
    name: http
  type: ClusterIP
EOF

# Create a NodePort service for external access simulation
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: gameserver-api-nodeport
  namespace: gametech-prod
  labels:
    app: gameserver-api
spec:
  selector:
    app: gameserver-api
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    name: http
  type: NodePort
EOF
```

### **Step 2: Wait for Stable State**

```bash
# Wait for deployment to be ready
kubectl rollout status deployment/gameserver-api --timeout=120s

# Verify all pods are healthy
kubectl get pods -l app=gameserver-api

# Test service connectivity
kubectl get endpoints gameserver-api-service

echo "✅ Stable production environment ready!"
echo "📊 Current status:"
kubectl get deployments,pods,services -l app=gameserver-api
```

### **Step 3: Create the Failed Deployment (Execute this when lab starts)**

```bash
# WARNING: This simulates the problematic deployment that students will need to rollback!
# Only run this AFTER confirming the stable environment is ready

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gameserver-api
  namespace: gametech-prod
  labels:
    app: gameserver-api
    version: v1.3.0
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
      maxSurge: 2
  selector:
    matchLabels:
      app: gameserver-api
  template:
    metadata:
      labels:
        app: gameserver-api
        version: v1.3.0
    spec:
      containers:
      - name: gameserver-api
        image: nginx:1.23
        ports:
        - containerPort: 80
        env:
        - name: VERSION
          value: "v1.3.0-beta"
        - name: ENVIRONMENT
          value: "production"
        - name: MAX_CONNECTIONS
          value: "2000"
        # PROBLEM 1: Aggressive readiness probe causing failures
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 2
          periodSeconds: 3
          failureThreshold: 1
        # PROBLEM 2: Overly restrictive liveness probe
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 1
        # PROBLEM 3: Insufficient resource limits causing OOM kills
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "100Mi"
            cpu: "100m"
EOF

echo "💥 DEPLOYMENT CRISIS INITIATED!"
echo "🚨 Failed deployment v1.3.0 has been pushed to production!"
echo "⏰ Students have 25 minutes to resolve the crisis..."
```

---

## 🔍 Verification Commands

### **Environment Health Check**

```bash
# Quick status overview
kubectl get deployments,rs,pods,services -n gametech-prod

# Check deployment history
kubectl rollout history deployment/gameserver-api -n gametech-prod

# Monitor real-time events
kubectl get events --sort-by=.metadata.creationTimestamp -n gametech-prod

# Pod health details
kubectl get pods -l app=gameserver-api -o wide -n gametech-prod

# Service endpoints
kubectl get endpoints -n gametech-prod
```

### **Simulated Production Monitoring**

```bash
# Simulate monitoring alerts
echo "🚨 ALERT: GameServer API Response Time: 15.2s (SLA: <2s)"
echo "🚨 ALERT: Failed Health Checks: 847 in last 5 minutes"  
echo "🚨 ALERT: Player Connection Success Rate: 23% (SLA: >95%)"
echo "💰 ALERT: Revenue Impact: $12,500 lost in last 15 minutes"

# Show "production" logs
kubectl logs -l app=gameserver-api --tail=20 -n gametech-prod
```

---

## 🎯 Expected Failure Symptoms

When the failed deployment is applied, students should observe:

1. **Pod Status Issues:**
   - Pods stuck in `CrashLoopBackOff` or `Pending` state
   - Readiness probe failures in pod events
   - Resource constraint warnings

2. **Service Degradation:**
   - Endpoints dropping out of service rotation
   - Connection timeouts and errors
   - Uneven traffic distribution

3. **Resource Problems:**
   - Memory limit exceeded (OOM kills)
   - CPU throttling warnings
   - Node resource pressure

---

## 🧹 Cleanup Commands

```bash
# Clean up after lab completion
kubectl delete namespace gametech-prod

# Verify cleanup
kubectl get all -n gametech-prod 2>/dev/null || echo "✅ Environment cleaned up successfully"

# Reset context if needed
kubectl config set-context --current --namespace=default
```

---

## 🔧 Instructor Notes

### **Timing Setup:**

1. **Pre-lab (5 min):** Execute Steps 1-2 to establish stable environment
2. **Lab Start:** Execute Step 3 to trigger the crisis scenario  
3. **During Lab:** Monitor student progress and provide hints if needed

### **Common Student Challenges:**

- **Panic Response:** Students may try complex solutions instead of simple rollback
- **Wrong Commands:** May attempt manual scaling instead of `rollout undo`
- **Incomplete Verification:** Not confirming service restoration after rollback
- **Investigation Skills:** Difficulty correlating pod failures to resource constraints

### **Success Indicators:**

- Fast rollback execution (< 8 minutes from crisis start)
- Proper use of `kubectl rollout` commands
- Systematic troubleshooting approach for root cause analysis
- Understanding of deployment strategy parameters

### **Extension Scenarios:**

- Add network policies for additional complexity
- Introduce multiple microservices with dependency failures
- Include database connection issues for deeper troubleshooting
- Add monitoring/alerting components for more realistic production environment

---

*🎮 **Reality Check:** This scenario mirrors actual GameTech production incidents where rolling deployments fail during peak traffic, requiring immediate rollback and post-incident analysis to prevent recurrence.*
