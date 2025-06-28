# T01 Lab Setup - Production Pod Crash Loop Test Environment

This file contains the necessary resources to set up the crash loop scenario for Lab T01.

## 🚀 Quick Setup

### Option 1: Memory Limit Crash Loop (Recommended)
```yaml
# t01-crash-loop-setup.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
  namespace: default
  labels:
    app: product-catalog
    scenario: black-friday
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      containers:
      - name: product-catalog
        image: nginx:1.21
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"    # Too low - will cause OOMKilled
            cpu: "100m"
        command: ["/bin/bash"]
        args: 
        - -c
        - |
          echo "Starting product catalog service..."
          echo "Connecting to database..."
          # Simulate memory-intensive startup
          dd if=/dev/zero of=/tmp/bigfile bs=1M count=100
          echo "Service started successfully"
          nginx -g 'daemon off;'
---
apiVersion: v1
kind: Service
metadata:
  name: product-catalog-service
  namespace: default
spec:
  selector:
    app: product-catalog
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

### Option 2: Application Error Crash Loop
```yaml
# t01-app-error-setup.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      containers:
      - name: product-catalog
        image: busybox:1.35
        command: ["/bin/sh"]
        args:
        - -c
        - |
          echo "Product Catalog Service v2.1.3"
          echo "Initializing Black Friday configuration..."
          echo "Connecting to database at db.prod.internal:5432..."
          sleep 5
          echo "ERROR: failed to connect to database: connection timeout after 30s"
          echo "PANIC: database connection failed - cannot start service"
          exit 1
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

## 🔧 Setup Instructions

### Deploy the Crash Loop Scenario:
```bash
# Create the problematic deployment
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
  namespace: default
  labels:
    app: product-catalog
    scenario: black-friday
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      containers:
      - name: product-catalog
        image: nginx:1.21
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"
        command: ["/bin/bash"]
        args: 
        - -c
        - |
          echo "Starting product catalog service..."
          echo "Connecting to database..."
          dd if=/dev/zero of=/tmp/bigfile bs=1M count=100
          echo "Service started successfully"
          nginx -g 'daemon off;'
EOF

# Wait a few minutes for the crash loop to develop
sleep 120

# Verify the crash loop is happening
kubectl get pods
```

### Verify the Problem:
```bash
# You should see something like:
# NAME                               READY   STATUS             RESTARTS   AGE
# product-catalog-7d4b8f9c8d-abc12   0/1     CrashLoopBackOff   3          5m
# product-catalog-7d4b8f9c8d-def34   0/1     CrashLoopBackOff   2          5m
# product-catalog-7d4b8f9c8d-ghi56   0/1     CrashLoopBackOff   4          5m
```

## 🎯 Lab Exercise

Now follow the T01 lab steps to:
1. Assess the situation
2. Analyze pod details and logs
3. Investigate cluster events
4. Implement the fix

## 🔧 Solution (Spoiler Alert!)

<details>
<summary>Click to reveal the fix</summary>

```bash
# The issue is memory limit too low (64Mi) for the operation
# Fix by increasing memory limit:

kubectl patch deployment product-catalog -p='{"spec":{"template":{"spec":{"containers":[{"name":"product-catalog","resources":{"limits":{"memory":"512Mi"},"requests":{"memory":"256Mi"}}}]}}}}'

# Or edit directly:
kubectl edit deployment product-catalog
# Change memory limits from 64Mi to 512Mi

# Verify the fix:
kubectl get pods -w
```

</details>

## 🧹 Cleanup

```bash
# Remove the test deployment when done
kubectl delete deployment product-catalog
kubectl delete service product-catalog-service
```

## 📝 Lab Validation Notes

This setup creates a realistic crash loop scenario that:
- ✅ Simulates real-world memory pressure issues
- ✅ Provides clear troubleshooting steps
- ✅ Has a definitive fix that can be verified
- ✅ Teaches important resource management concepts
- ✅ Can be completed in ~15 minutes

Perfect for testing the T01 lab effectiveness!
