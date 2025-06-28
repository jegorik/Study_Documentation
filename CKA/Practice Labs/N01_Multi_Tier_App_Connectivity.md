# Lab N01: Multi-Tier App Connectivity

> **📋 Lab Category:** Services & Networking  
> **⏱️ Estimated Time:** 15 minutes  
> **🎯 Difficulty:** Basic  
> **🔗 Exam Objectives:** Understand connectivity between Pods, Use ClusterIP, NodePort, LoadBalancer service types and endpoints, Know how to use Ingress controllers and Ingress resources

---

## 🎬 Real-World Scenario

### Background Context

You're a DevOps Engineer at **ModernCommerce**, a digital marketplace that's launching their new microservices-based e-commerce platform. The development team has containerized their application into three tiers: frontend (React), backend API (Node.js), and database (PostgreSQL). They need these services to communicate securely within the Kubernetes cluster while exposing the frontend to external users.

**Your Role:** Platform Engineer responsible for application networking and service discovery  
**Situation:** Development team needs guidance on proper service configuration for their three-tier architecture  
**Urgency:** **MEDIUM PRIORITY** - Demo for investors scheduled in 2 days, application must be accessible and functional

### The Challenge

The development team has deployed their containers but they're struggling with service-to-service communication. The frontend can't reach the backend API, and the backend can't connect to the database. Additionally, they need the frontend accessible from outside the cluster while keeping the backend and database internal-only.

**Critical Business Requirements:**

- 💰 **Investor Demo:** Application must work end-to-end for investor showcase
- 🔒 **Security:** Database should not be accessible from outside the cluster
- 🌐 **External Access:** Frontend needs to be reachable by external users
- 📊 **Service Discovery:** Backend API must be discoverable by frontend

---

## 🎯 Learning Objectives

By completing this lab, you will:

- [ ] **Primary Skill:** Master Kubernetes service types (ClusterIP, NodePort, LoadBalancer) and their use cases
- [ ] **Secondary Skills:** Configure pod-to-pod communication, implement service discovery, expose applications externally
- [ ] **Real-world Application:** Design networking for multi-tier applications with security considerations
- [ ] **Exam Relevance:** Core CKA networking skills - services, endpoints, ingress, connectivity

---

## 🔧 Prerequisites

### Knowledge Requirements

- [ ] Basic understanding of Kubernetes pods and deployments
- [ ] Familiarity with networking concepts (IP addressing, ports, DNS)
- [ ] Understanding of multi-tier application architecture

### Environment Setup

```bash
# Cluster requirements
- Working Kubernetes cluster (from A01 or managed service)
- kubectl configured and working
- Cluster with networking (CNI) properly configured

# Verification commands
kubectl cluster-info
kubectl get nodes
kubectl get pods -n kube-system | grep -E "(coredns|calico|flannel|weave)"
```

---

## 📚 Quick Reference

### Key Commands for This Lab

```bash
kubectl expose deployment <name> --type=<type> --port=<port>  # Create service
kubectl get services -o wide                                  # List services with details
kubectl describe service <service-name>                       # Service details and endpoints
kubectl get endpoints                                          # View service endpoints
kubectl port-forward service/<name> <local-port>:<service-port>  # Test connectivity
```

### Important Concepts

- **ClusterIP:** Internal-only service, default type for inter-cluster communication
- **NodePort:** Exposes service on each node's IP at a static port (30000-32767)
- **LoadBalancer:** Cloud provider external load balancer (if supported)
- **Service Discovery:** Automatic DNS resolution for services within cluster

---

## 🚀 Lab Tasks

### Task 1: Deploy the Three-Tier Application

**Objective:** Create deployments for frontend, backend, and database components

**Your Mission:**
Deploy the three application tiers using the provided YAML manifests. Verify that all pods are running but note that they cannot communicate yet since no services are configured.

**Expected Outcome:**
Three deployments running with healthy pods: frontend (React app), backend (API server), and database (PostgreSQL).

**Hints:**

- 💡 Start by deploying all three tiers without services to see the connectivity problem
- 💡 Check pod logs to see connection failures between tiers
- 💡 Note the labels on each deployment - you'll need them for service selectors

---

### Task 2: Configure Internal Service Communication

**Objective:** Create ClusterIP services for backend and database to enable internal communication

**Your Mission:**
Create ClusterIP services for the backend API and database. Configure the backend to connect to the database, and verify the connection is working. Use service DNS names for discovery.

**Expected Outcome:**
Backend can successfully connect to database using service DNS name, database is only accessible within the cluster.

**Hints:**

- 💡 ClusterIP is the default service type for internal communication
- 💡 Service DNS follows the pattern: `<service-name>.<namespace>.svc.cluster.local`
- 💡 Check backend logs to confirm database connection success

---

### Task 3: Expose Frontend Externally with NodePort

**Objective:** Create a NodePort service to make the frontend accessible from outside the cluster

**Your Mission:**
Create a NodePort service for the frontend application. Configure the frontend to communicate with the backend API using the internal service. Test external access to the frontend.

**Expected Outcome:**
Frontend is accessible from outside the cluster via NodePort, and can successfully communicate with the backend API internally.

**Hints:**

- 💡 NodePort automatically allocates a port in the 30000-32767 range
- 💡 You can specify a specific NodePort if needed
- 💡 Test access using `curl http://<node-ip>:<nodeport>`

---

### Task 4: Verify End-to-End Connectivity and Service Discovery

**Objective:** Test complete application flow and verify service discovery is working properly

**Your Mission:**
Verify that the complete application flow works: external user → frontend → backend → database. Test service discovery by checking that services can be resolved by DNS names. Document the service architecture.

**Expected Outcome:**
Complete three-tier application working end-to-end with proper service discovery and security boundaries.

**Hints:**

- 💡 Use `kubectl exec` to test DNS resolution from within pods
- 💡 Check service endpoints to ensure they point to the correct pods
- 💡 Verify that database is not accessible from outside the cluster

---

## ⏰ Time Management

**Exam Pace Training:**

- [ ] Task 1: 3 minutes (Deploy applications)
- [ ] Task 2: 5 minutes (Internal services)
- [ ] Task 3: 4 minutes (External access)
- [ ] Task 4: 3 minutes (Verification)
- [ ] **Total Target:** 15 minutes

*⚠️ If taking longer than target time, move to solution section and understand the efficient approach*

---

## 🔧 Step-by-Step Solution

<details>
<summary><strong>📖 Click to reveal detailed solution (try solving first!)</strong></summary>

### Step 1: Deploy Three-Tier Application

**Command/Action:**

```bash
# Create namespace for the application
kubectl create namespace ecommerce

# Deploy PostgreSQL database
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  namespace: ecommerce
  labels:
    app: database
    tier: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
        tier: database
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: ecommerce
        - name: POSTGRES_USER
          value: app
        - name: POSTGRES_PASSWORD
          value: secretpassword
        ports:
        - containerPort: 5432
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
EOF

# Deploy Backend API
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: ecommerce
  labels:
    app: backend
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        tier: backend
    spec:
      containers:
      - name: api
        image: nginx:1.21  # Simulating backend API
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_URL
          value: "postgresql://app:secretpassword@database.ecommerce.svc.cluster.local:5432/ecommerce"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
EOF

# Deploy Frontend
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ecommerce
  labels:
    app: frontend
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: web
        image: nginx:1.21  # Simulating React frontend
        ports:
        - containerPort: 80
        env:
        - name: BACKEND_URL
          value: "http://backend.ecommerce.svc.cluster.local:8080"
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
EOF

# Verify deployments
kubectl get deployments -n ecommerce
kubectl get pods -n ecommerce
```

**Explanation:**
We deploy three separate deployments representing each tier of our application. Each has specific labels that we'll use for service selectors. The environment variables show how applications would reference each other.

**Expected Output:**

```bash
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
database   1/1     1            1           2m
backend    2/2     2            2           2m
frontend   3/3     3            3           2m
```

---

### Step 2: Create Internal Services (ClusterIP)

**Command/Action:**

```bash
# Create ClusterIP service for database (internal only)
kubectl expose deployment database \
  --namespace=ecommerce \
  --type=ClusterIP \
  --port=5432 \
  --target-port=5432 \
  --name=database

# Create ClusterIP service for backend API (internal, will be accessed by frontend)
kubectl expose deployment backend \
  --namespace=ecommerce \
  --type=ClusterIP \
  --port=8080 \
  --target-port=80 \
  --name=backend

# Verify services and endpoints
kubectl get services -n ecommerce
kubectl get endpoints -n ecommerce
kubectl describe service database -n ecommerce
```

**Explanation:**
ClusterIP services provide internal DNS names and load balancing for pod-to-pod communication. The database service makes PostgreSQL accessible at `database.ecommerce.svc.cluster.local:5432`, while the backend is available at `backend.ecommerce.svc.cluster.local:8080`.

**Expected Output:**

```bash
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
database   ClusterIP   10.96.100.123    <none>        5432/TCP   1m
backend    ClusterIP   10.96.200.234    <none>        8080/TCP   1m

NAME       ENDPOINTS                               AGE
database   192.168.1.10:5432                      1m
backend    192.168.1.11:80,192.168.1.12:80        1m
```

---

### Step 3: Expose Frontend with NodePort

**Command/Action:**

```bash
# Create NodePort service for frontend (external access)
kubectl expose deployment frontend \
  --namespace=ecommerce \
  --type=NodePort \
  --port=80 \
  --target-port=80 \
  --name=frontend

# Get the NodePort assigned
kubectl get service frontend -n ecommerce

# Test external access (replace with your node IP)
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
NODE_PORT=$(kubectl get service frontend -n ecommerce -o jsonpath='{.spec.ports[0].nodePort}')
echo "Frontend accessible at: http://$NODE_IP:$NODE_PORT"

# Test connectivity
curl -I http://$NODE_IP:$NODE_PORT
```

**Explanation:**
NodePort service exposes the frontend on every node's IP at a high port (30000-32767). This allows external users to access the application while keeping internal services secure.

**Expected Output:**

```bash
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
frontend   NodePort   10.96.300.345   <none>        80:31234/TCP   1m

Frontend accessible at: http://192.168.56.10:31234
HTTP/1.1 200 OK
```

---

### Step 4: Verify End-to-End Connectivity

**Command/Action:**

```bash
# Test service discovery from frontend pod
FRONTEND_POD=$(kubectl get pods -n ecommerce -l app=frontend -o jsonpath='{.items[0].metadata.name}')

# Test DNS resolution
kubectl exec -n ecommerce $FRONTEND_POD -- nslookup backend.ecommerce.svc.cluster.local
kubectl exec -n ecommerce $FRONTEND_POD -- nslookup database.ecommerce.svc.cluster.local

# Test backend connectivity from frontend
kubectl exec -n ecommerce $FRONTEND_POD -- wget -qO- http://backend.ecommerce.svc.cluster.local:8080

# Test that database is NOT accessible externally (should fail)
curl http://$NODE_IP:5432 || echo "Good! Database is not accessible externally"

# Check service endpoints are healthy
kubectl get endpoints -n ecommerce
kubectl describe service backend -n ecommerce

# Summary of service architecture
echo "=== Service Architecture Summary ==="
kubectl get services -n ecommerce -o wide
echo "Frontend (External): http://$NODE_IP:$NODE_PORT"
echo "Backend (Internal): backend.ecommerce.svc.cluster.local:8080"
echo "Database (Internal): database.ecommerce.svc.cluster.local:5432"
```

**Explanation:**
We verify that DNS resolution works for service discovery, internal communication is functional, and external access is properly controlled. The database should only be accessible internally.

**Expected Output:**

```bash
Name:	backend.ecommerce.svc.cluster.local
Address: 10.96.200.234

HTTP/1.1 200 OK
curl: (7) Failed to connect to 192.168.56.10 port 5432: Connection refused
Good! Database is not accessible externally

=== Service Architecture Summary ===
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
database   ClusterIP   10.96.100.123   <none>        5432/TCP       5m
backend    ClusterIP   10.96.200.234   <none>        8080/TCP       5m
frontend   NodePort    10.96.300.345   <none>        80:31234/TCP   3m
```

**Success Indicators:**

- [ ] All services have healthy endpoints
- [ ] Frontend accessible via NodePort from external network
- [ ] Backend accessible internally via service DNS
- [ ] Database accessible internally but not externally
- [ ] Service discovery working via DNS resolution

</details>

---

## 🎓 Knowledge Check

### Understanding Questions

1. **Service Types:** When would you use ClusterIP vs NodePort vs LoadBalancer service types?
2. **DNS Resolution:** How does Kubernetes service discovery work, and what's the DNS naming pattern?
3. **Security Boundaries:** Why is it important to use ClusterIP for internal services like databases?
4. **Load Balancing:** How do services distribute traffic across multiple pod replicas?

### Hands-On Challenges

- [ ] **Variation 1:** Convert the NodePort to LoadBalancer (if on cloud provider)
- [ ] **Variation 2:** Add Ingress controller for path-based routing
- [ ] **Integration:** Configure network policies to restrict database access

---

## 🔍 Common Pitfalls & Troubleshooting

### Frequent Mistakes

1. **Wrong Service Selector**
   - **Symptom:** Service has no endpoints or wrong endpoints
   - **Cause:** Service selector doesn't match pod labels
   - **Fix:** Check labels with `kubectl get pods --show-labels` and update service selector

2. **Incorrect Port Configuration**
   - **Symptom:** Connection refused when accessing service
   - **Cause:** Service port doesn't match container port
   - **Fix:** Verify container port with `kubectl describe pod` and match in service

3. **DNS Resolution Issues**
   - **Symptom:** Service not reachable by DNS name
   - **Cause:** Incorrect namespace or DNS name format
   - **Fix:** Use full DNS name: `<service>.<namespace>.svc.cluster.local`

### Debug Commands

```bash
# Essential debugging commands for service connectivity
kubectl get services -o wide                         # List all services
kubectl describe service <service-name>              # Detailed service info
kubectl get endpoints                                 # Service endpoints
kubectl exec <pod> -- nslookup <service-name>       # Test DNS resolution
kubectl port-forward service/<name> <port>          # Local testing
```

---

## 🌟 Real-World Applications

### Enterprise Scenarios

- **Microservices Architecture:** Service-to-service communication patterns
- **Security Compliance:** Internal-only services with external-facing frontends
- **Multi-Environment Deployments:** Consistent networking across dev/staging/prod
- **Traffic Management:** Load balancing and service discovery at scale

### Best Practices Learned

- 🏆 **Security First:** Use ClusterIP for internal services, NodePort/LoadBalancer only when needed
- 🏆 **Service Discovery:** Leverage DNS names for loose coupling between services
- 🏆 **Port Management:** Use consistent port naming and documentation
- 🏆 **Testing Strategy:** Always verify connectivity and security boundaries

---

## 📚 Additional Resources

### Related CKA Topics

- Understand connectivity between Pods
- Use ClusterIP, NodePort, LoadBalancer service types and endpoints
- Know how to use Ingress controllers and Ingress resources
- Understand and use CoreDNS

### Extended Learning

- [ ] Explore Ingress controllers for advanced routing
- [ ] Study Network Policies for micro-segmentation
- [ ] Learn about service mesh solutions (Istio, Linkerd)

---

## 📝 Lab Completion

### Self-Assessment

- [ ] I can complete this lab within 15 minutes
- [ ] I understand the differences between service types
- [ ] I can configure service discovery and internal communication
- [ ] I can troubleshoot service connectivity issues independently
- [ ] I'm confident with multi-tier application networking

### Notes & Reflections

*Record insights about service architecture decisions, security considerations, or networking patterns you discovered.*

---

### 🏁 Lab Status

- [ ] **Started:** ___________
- [ ] **Completed:** ___________  
- [ ] **Reviewed:** ___________
- [ ] **Mastered:** ___________

**Time Taken:** _______ | **Target Time:** 15 minutes  
**Difficulty Rating:** ___/5 | **Confidence Level:** ___/5

---

*💡 **Next Lab Recommendation:** W01 - Rolling Update Disaster Recovery (Workloads) or S01 - Dynamic Storage Provisioning (Storage basics)*
