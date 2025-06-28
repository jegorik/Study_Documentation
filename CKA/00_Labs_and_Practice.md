# 💻 CKA Labs and Practice Scenarios

> **Hands-on practice scenarios designed to simulate real CKA exam tasks**

## 📋 Table of Contents

- [How to Use This Guide](#-how-to-use-this-guide)
- [Practice Environment Setup](#️-practice-environment-setup)
- [Lab Categories](#-lab-categories)
- [Beginner Labs](#beginner-labs)
- [Intermediate Labs](#intermediate-labs)
- [Advanced Labs](#advanced-labs)
- [Exam Simulation](#-exam-simulation)
- [Troubleshooting Scenarios](#-troubleshooting-scenarios)

---

## 🎯 How to Use This Guide

### Study Progression

1. **Complete theory** for each topic first
2. **Practice basic commands** from command reference
3. **Work through labs** in order of difficulty
4. **Time yourself** to simulate exam pressure
5. **Review solutions** and understand alternatives

### Lab Format

Each lab includes:

- **Objective:** What you need to accomplish
- **Context:** Scenario background
- **Tasks:** Step-by-step requirements
- **Time Limit:** Suggested completion time
- **Solution:** Complete walkthrough
- **Verification:** How to confirm success

---

## 🛠️ Practice Environment Setup

### Recommended Environments

1. **Local Development:**
   - **minikube** - Single node cluster
   - **kind** - Kubernetes in Docker
   - **Docker Desktop** - Built-in Kubernetes

2. **Cloud Platforms:**
   - **Google GKE** - Free tier available
   - **AWS EKS** - Managed Kubernetes
   - **Azure AKS** - Azure Kubernetes Service

3. **Practice Platforms:**
   - **Killer.sh** - CKA exam simulator
   - **KodeKloud** - Interactive labs
   - **Katacoda** - Browser-based scenarios

### Quick Setup (minikube)

```bash
# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start --driver=docker

# Verify installation
kubectl cluster-info
kubectl get nodes
```

---

## 📚 Lab Categories

### By Exam Domain

| Domain | Labs | Weight | Priority |
|--------|------|--------|----------|
| **Architecture** | 1-3 | 25% | High |
| **Workloads** | 4-8 | 20% | High |
| **Services** | 9-12 | 20% | Medium |
| **Storage** | 13-15 | 10% | Medium |
| **Troubleshooting** | 16-20 | 30% | **Highest** |

### By Difficulty

- **🟢 Beginner:** Basic resource creation and management
- **🟡 Intermediate:** Multi-step scenarios with dependencies
- **🔴 Advanced:** Complex troubleshooting and optimization
- **⚫ Exam Simulation:** Timed, exam-like conditions

---

## 🟢 Beginner Labs

### Lab 1: Pod Management Basics

**Domain:** Workloads & Scheduling  
**Time Limit:** 15 minutes  
**Objective:** Create and manage basic pods

#### Tasks:

1. Create a pod named `nginx-pod` using the `nginx:1.20` image
2. Verify the pod is running
3. Get the pod's IP address
4. Access the pod and check nginx is serving content
5. View the pod's logs
6. Delete the pod

#### Solution:

```bash
# Step 1: Create pod
kubectl run nginx-pod --image=nginx:1.20

# Step 2: Verify pod status
kubectl get pods nginx-pod

# Step 3: Get pod IP
kubectl get pod nginx-pod -o wide

# Step 4: Access pod (from another pod)
kubectl run test --image=busybox --rm -it -- wget -qO- http://POD_IP

# Step 5: View logs
kubectl logs nginx-pod

# Step 6: Delete pod
kubectl delete pod nginx-pod
```

#### Verification:

```bash
# Check pod was deleted
kubectl get pods nginx-pod
# Should return: Error from server (NotFound)
```

---

### Lab 2: Deployment and Scaling

**Domain:** Workloads & Scheduling  
**Time Limit:** 20 minutes  
**Objective:** Create deployments and manage scaling

#### Tasks

1. Create a deployment named `web-app` with 3 replicas using `nginx:1.21`
2. Verify all pods are running
3. Scale the deployment to 5 replicas
4. Update the image to `nginx:1.22`
5. Check rollout status
6. Scale down to 2 replicas

#### Solution

```bash
# Step 1: Create deployment
kubectl create deployment web-app --image=nginx:1.21 --replicas=3

# Step 2: Verify pods
kubectl get deployments web-app
kubectl get pods -l app=web-app

# Step 3: Scale up
kubectl scale deployment web-app --replicas=5

# Step 4: Update image
kubectl set image deployment/web-app nginx=nginx:1.22

# Step 5: Check rollout
kubectl rollout status deployment/web-app

# Step 6: Scale down
kubectl scale deployment web-app --replicas=2
```

---

### Lab 3: Service Creation

**Domain:** Services & Networking  
**Time Limit:** 15 minutes  
**Objective:** Expose deployments with services

#### Tasks

1. Use the deployment from Lab 2 (or create a new one)
2. Expose the deployment as a ClusterIP service on port 80
3. Test the service works from within the cluster
4. Change service type to NodePort
5. Verify external access works

#### Solution

```bash
# Step 1: Ensure deployment exists
kubectl get deployment web-app || kubectl create deployment web-app --image=nginx:1.21 --replicas=2

# Step 2: Create ClusterIP service
kubectl expose deployment web-app --port=80 --type=ClusterIP

# Step 3: Test service
kubectl run test-pod --image=busybox --rm -it -- wget -qO- http://web-app

# Step 4: Change to NodePort
kubectl patch service web-app -p '{"spec":{"type":"NodePort"}}'

# Step 5: Get NodePort and test
kubectl get service web-app
# Access via http://NODE_IP:NODE_PORT
```

---

## 🟡 Intermediate Labs

### Lab 4: ConfigMaps and Secrets

**Domain:** Workloads & Scheduling  
**Time Limit:** 25 minutes  
**Objective:** Configure applications using ConfigMaps and Secrets

#### Scenario

You need to deploy a web application that requires database configuration and sensitive credentials.

#### Tasks

1. Create a ConfigMap named `app-config` with:
   - `database_url=postgres://db.example.com:5432/myapp`
   - `log_level=INFO`
   - `feature_flags={"new_ui": true, "analytics": false}`

2. Create a Secret named `app-secrets` with:
   - `db_username=admin`
   - `db_password=secretpass123`

3. Create a deployment that uses both the ConfigMap and Secret
4. Verify the environment variables are set correctly

#### Solution

```bash
# Step 1: Create ConfigMap
kubectl create configmap app-config \
  --from-literal=database_url="postgres://db.example.com:5432/myapp" \
  --from-literal=log_level="INFO" \
  --from-literal=feature_flags='{"new_ui": true, "analytics": false}'

# Step 2: Create Secret
kubectl create secret generic app-secrets \
  --from-literal=db_username=admin \
  --from-literal=db_password=secretpass123

# Step 3: Create deployment with config
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-config
spec:
  replicas: 1
  selector:
    matchLabels:
      app: configured-app
  template:
    metadata:
      labels:
        app: configured-app
    spec:
      containers:
      - name: app
        image: nginx
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log_level
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_password
EOF

# Step 4: Verify environment variables
kubectl exec deployment/app-with-config -- env | grep -E "(DATABASE_URL|LOG_LEVEL|DB_)"
```

---

### Lab 5: Persistent Storage

**Domain:** Storage  
**Time Limit:** 30 minutes  
**Objective:** Work with persistent volumes and claims

#### Tasks

1. Create a PersistentVolume with 1Gi capacity
2. Create a PersistentVolumeClaim that binds to the PV
3. Create a pod that mounts the PVC
4. Write data to the mounted volume
5. Delete and recreate the pod, verify data persists

#### Solution

```bash
# Step 1: Create PersistentVolume
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /tmp/pv-data
EOF

# Step 2: Create PersistentVolumeClaim
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
EOF

# Step 3: Create pod with PVC
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
EOF

# Step 4: Write data
kubectl exec storage-pod -- sh -c 'echo "Hello from CKA lab" > /data/test.txt'

# Step 5: Delete and recreate pod, check data
kubectl delete pod storage-pod
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod-2
spec:
  containers:
  - name: app
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
EOF

kubectl exec storage-pod-2 -- cat /data/test.txt
```

---

## 🔴 Advanced Labs

### Lab 6: Multi-Container Pod with Init Container

**Domain:** Workloads & Scheduling  
**Time Limit:** 35 minutes  
**Objective:** Create complex pod configurations

#### Scenario

Deploy a web application that requires database initialization before starting.

#### Tasks

1. Create a pod with an init container that simulates database setup
2. Main container should start only after init container completes
3. Use shared volume between init and main containers
4. Verify the sequence works correctly

#### Solution

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-app
spec:
  initContainers:
  - name: db-setup
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo "Initializing database..."
      sleep 10
      echo "Database ready" > /shared/db-status
      echo "Init container completed"
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  containers:
  - name: web-app
    image: nginx
    command: ['sh', '-c']
    args:
    - |
      echo "Waiting for database..."
      while [ ! -f /shared/db-status ]; do
        sleep 1
      done
      echo "Database is ready, starting web app"
      nginx -g 'daemon off;'
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  - name: sidecar-logger
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "$(date): App is running"
        sleep 30
      done
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  volumes:
  - name: shared-data
    emptyDir: {}
EOF

# Monitor pod startup
kubectl get pods multi-container-app -w

# Check logs from different containers
kubectl logs multi-container-app -c db-setup
kubectl logs multi-container-app -c web-app
kubectl logs multi-container-app -c sidecar-logger
```

---

## ⚫ Exam Simulation

### Exam Scenario 1: Complete Application Deployment

**Time Limit:** 45 minutes  
**Points:** 25/100  

#### Context

You are tasked with deploying a complete application stack for a development team.

#### Tasks

1. **Namespace Setup (5 points)**
   - Create namespace `dev-team`
   - Set as default for your context

2. **Database Layer (8 points)**
   - Create a StatefulSet named `postgres-db` with 1 replica
   - Use image `postgres:13`
   - Set environment variable `POSTGRES_DB=appdb`
   - Create Secret for database credentials
   - Use persistent storage (2Gi)

3. **Application Layer (7 points)**
   - Create Deployment `web-app` with 3 replicas
   - Use image `nginx:1.21`
   - Configure resource limits: CPU 200m, Memory 256Mi
   - Configure resource requests: CPU 100m, Memory 128Mi

4. **Service Layer (5 points)**
   - Expose database as ClusterIP service on port 5432
   - Expose web-app as NodePort service on port 80

#### Time Management

- Task 1: 5 minutes
- Task 2: 20 minutes
- Task 3: 15 minutes
- Task 4: 5 minutes

#### Solution

```bash
# Task 1: Namespace (5 min)
kubectl create namespace dev-team
kubectl config set-context --current --namespace=dev-team

# Task 2: Database (20 min)
# Create secret
kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_USER=admin \
  --from-literal=POSTGRES_PASSWORD=secretpass

# Create PVC for database
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
EOF

# Create StatefulSet
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-db
spec:
  serviceName: postgres-db
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: appdb
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_PASSWORD
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
EOF

# Task 3: Application (15 min)
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
EOF

# Task 4: Services (5 min)
kubectl expose statefulset postgres-db --port=5432 --type=ClusterIP
kubectl expose deployment web-app --port=80 --type=NodePort
```

#### Verification

```bash
# Check all resources
kubectl get all
kubectl get pvc
kubectl get secrets

# Verify resource limits
kubectl describe deployment web-app

# Test connectivity
kubectl run test --image=busybox --rm -it -- nc -zv postgres-db 5432
```

---

## 🔧 Troubleshooting Scenarios

### Scenario 1: Pod Not Starting

**Time Limit:** 15 minutes  
**Objective:** Debug and fix a failing pod

#### Problem Setup

```bash
# Deploy broken pod
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: broken-pod
spec:
  containers:
  - name: app
    image: nginx:nonexistent-tag
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "2Gi"
        cpu: "2000m"
      limits:
        memory: "1Gi"
        cpu: "1000m"
EOF
```

#### Your Task

Identify and fix all issues preventing the pod from starting.

#### Debugging Steps

```bash
# Check pod status
kubectl get pods broken-pod

# Get detailed information
kubectl describe pod broken-pod

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check node resources
kubectl top nodes
```

#### Issues to Find

1. Invalid image tag
2. Resource requests > limits
3. Insufficient node resources

---

### Scenario 2: Service Not Accessible

**Time Limit:** 20 minutes  
**Objective:** Fix networking issues

#### Problem Setup

```bash
# Create deployment and broken service
kubectl create deployment test-app --image=nginx --port=80
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: test-service
spec:
  selector:
    app: wrong-label
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
EOF
```

#### Your Task

Fix the service so it properly routes traffic to the pods.

#### Solution

```bash
# Check service endpoints
kubectl get endpoints test-service
# Should show no endpoints

# Check deployment labels
kubectl get deployment test-app --show-labels

# Fix service selector and targetPort
kubectl patch service test-service -p '{"spec":{"selector":{"app":"test-app"},"ports":[{"port":80,"targetPort":80}]}}'

# Verify fix
kubectl get endpoints test-service
kubectl run test --image=busybox --rm -it -- wget -qO- http://test-service
```

---

## 📊 Progress Tracking

### Lab Completion Checklist

#### Beginner Labs

- [ ] Lab 1: Pod Management Basics
- [ ] Lab 2: Deployment and Scaling  
- [ ] Lab 3: Service Creation

#### Intermediate Labs

- [ ] Lab 4: ConfigMaps and Secrets
- [ ] Lab 5: Persistent Storage
- [ ] Lab 6: Multi-Container Pods

#### Advanced Labs

- [ ] Lab 7: StatefulSets and Headless Services
- [ ] Lab 8: DaemonSets and Node Scheduling
- [ ] Lab 9: Jobs and CronJobs

#### Exam Simulations

- [ ] Scenario 1: Complete Application Deployment
- [ ] Scenario 2: Cluster Upgrade
- [ ] Scenario 3: Backup and Restore

#### Troubleshooting

- [ ] Pod Not Starting
- [ ] Service Not Accessible
- [ ] Node Issues
- [ ] Network Policies
- [ ] Storage Problems

### Study Schedule Recommendation

- **Week 1-2:** Beginner labs (1-2 labs per day)
- **Week 3-4:** Intermediate labs (1 lab per day)
- **Week 5-6:** Advanced labs (1 lab every 2 days)
- **Week 7:** Exam simulations (1 per day)
- **Week 8:** Troubleshooting focus

---

## 💡 Tips for Success

### During Practice

- **Time yourself** - Exam pressure is real
- **Practice typing YAML** - Speed matters
- **Use kubectl shortcuts** - Learn the imperative commands
- **Verify your work** - Always test what you create
- **Clean up** - Practice good resource management

### Common Pitfalls

- **YAML indentation** - Use spaces, not tabs
- **Label selectors** - Must match exactly
- **Resource names** - Follow Kubernetes naming conventions
- **Namespace context** - Always verify which namespace you're in
- **Resource limits** - Understand requests vs limits

### Exam Day Preparation

- Practice in **Ubuntu environment** (exam OS)
- Get comfortable with **vim/nano** editors
- Memorize **essential kubectl commands**
- Practice **time management** strategies
- Review **Kubernetes documentation** structure

---

*Last Updated: May 29, 2025*  
*CKA Exam Version: v1.30*  
*Labs Version: 1.0*
