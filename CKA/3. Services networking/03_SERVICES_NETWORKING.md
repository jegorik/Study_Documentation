# CKA Exam Cheat Sheet for Services & Networking

---

## Section 1: Exam Topic Overview

### Key Concepts
- **Services:** Stable network endpoints for accessing pods
- **Endpoints:** Backend pods that services route traffic to
- **Ingress:** HTTP/HTTPS routing and load balancing
- **Network Policies:** Firewall rules for pod-to-pod communication
- **DNS:** Service discovery and name resolution
- **CNI:** Container Network Interface for cluster networking

### Exam Objectives
- [ ] Create and configure different service types
- [ ] Implement ingress controllers and rules
- [ ] Configure network policies for security
- [ ] Troubleshoot service connectivity issues
- [ ] Understand cluster DNS and service discovery
- [ ] Manage network communication between pods

### Study Resources
- Official Kubernetes Documentation: https://kubernetes.io/docs/concepts/services-networking/
- CNCF Training Materials: Services and Networking
- Practice Labs: Network configuration and troubleshooting

### Exam Weight
**20%** of total exam score

---

## Section 2: ASCII Drawing with Explanation

```
                        KUBERNETES NETWORKING ARCHITECTURE
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                        EXTERNAL TRAFFIC                         │
    │                                                                 │
    │  Internet/Users  ──>  LoadBalancer  ──>  NodePort  ──>  Pods    │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
    ┌──────────────────────────────────────────────────────────────────┐
    │                         INGRESS LAYER                            │
    │                                                                  │
    │  ┌───────────────────┐    ┌─────────────────────────────────┐    │
    │  │ Ingress Controller│    │          Ingress Rules          │    │
    │  │   (nginx/traefik) │    │                                 │    │
    │  │                   │    │  /api  ──>  backend-service     │    │
    │  │  ┌─────────────┐  │    │  /web  ──>  frontend-service    │    │
    │  │  │ HTTP Router │  │    │  *.dev ──>  dev-service         │    │
    │  │  └─────────────┘  │    └─────────────────────────────────┘    │
    │  └───────────────────┘                                           │
    └──────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                       SERVICE LAYER                             │
    │                                                                 │
    │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐   │
    │ │ ClusterIP   │  │  NodePort   │  │    LoadBalancer         │   │
    │ │ (Internal)  │  │ (External)  │  │   (Cloud Provider)      │   │
    │ │             │  │             │  │                         │   │
    │ │ Virtual IP  │  │ Node:Port   │  │ External IP + Port      │   │
    │ │ 10.96.x.x   │  │ Any:30000+  │  │ Cloud LB ──> NodePort   │   │
    │ └─────────────┘  └─────────────┘  └─────────────────────────┘   │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                       ENDPOINT LAYER                            │
    │                                                                 │
    │        Service Selector ──> Label Matching ──> Pod IPs          │
    │                                                                 │
    │   app=frontend     ┌─────────┐  ┌─────────┐  ┌─────────┐        │
    │        │           │  Pod 1  │  │  Pod 2  │  │  Pod 3  │        │
    │        └──────────>│10.244.1.│  │10.244.2.│  │10.244.1.│        │
    │                    │   .10   │  │   .15   │  │   .20   │        │
    │                    └─────────┘  └─────────┘  └─────────┘        │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                      NETWORK POLICIES                           │
    │                                                                 │
    │  ┌─────────────────┐              ┌─────────────────┐           │
    │  │   Frontend      │   ALLOWED    │    Backend      │           │
    │  │   Namespace     │ ──────────>  │   Namespace     │           │
    │  │                 │              │                 │           │
    │  │  Port: 80,443   │              │   Port: 3306    │           │
    │  └─────────────────┘              └─────────────────┘           │
    │           │                                                     │
    │           │ DENIED                                              │
    │           ▼                                                     │
    │  ┌─────────────────┐                                            │
    │  │   External      │                                            │
    │  │   Networks      │                                            │
    │  └─────────────────┘                                            │
    └─────────────────────────────────────────────────────────────────┘

                          DNS RESOLUTION FLOW
    
    ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
    │      Pod        │────>│   CoreDNS       │────>│    Service      │
    │                 │     │   (kube-dns)    │     │                 │
    │ nslookup        │     │                 │     │ ClusterIP       │
    │ my-service      │<────│ DNS Resolution  │<────│ my-service      │
    │                 │     │                 │     │ 10.96.100.50    │
    └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Component Breakdown

#### Service Types
1. **ClusterIP:**
   - Purpose: Internal cluster communication only
   - Key features: Virtual IP accessible within cluster, default type
   - Interactions: Provides stable endpoint for pods, DNS resolvable

2. **NodePort:**
   - Purpose: External access through node ports (30000-32767)
   - Key features: Opens port on all nodes, includes ClusterIP functionality
   - Interactions: External traffic → Node:Port → ClusterIP → Pods

3. **LoadBalancer:**
   - Purpose: External access with cloud provider integration
   - Key features: Provisions external load balancer, includes NodePort functionality
   - Interactions: External LB → NodePort → ClusterIP → Pods

4. **ExternalName:**
   - Purpose: CNAME alias for external services
   - Key features: Returns CNAME record, no proxy/load balancing
   - Interactions: DNS resolution to external FQDN

#### Ingress Components
1. **Ingress Controller:**
   - Purpose: Implements ingress rules (nginx, traefik, istio)
   - Key features: HTTP/HTTPS routing, SSL termination, path-based routing
   - Interactions: Receives external traffic, routes based on ingress rules

2. **Ingress Rules:**
   - Purpose: Define routing rules for HTTP traffic
   - Key features: Host-based routing, path-based routing, TLS configuration
   - Interactions: Configure how ingress controller routes traffic

#### Network Policy
1. **Ingress Rules:**
   - Purpose: Control incoming traffic to pods
   - Key features: Source selection, port specification, protocol filtering
   - Interactions: Allow/deny traffic based on labels and namespaces

2. **Egress Rules:**
   - Purpose: Control outgoing traffic from pods
   - Key features: Destination selection, port specification
   - Interactions: Control pod communication to external services

---

## Section 3: Live Examples

### Scenario: Setting up Complete Service Networking for a Multi-tier Application

**Problem:** Deploy a web application with frontend, backend, and database tiers with proper networking and security

**Solution Steps:**

#### Step 1: Create Namespaces and Deployments
```bash
# Create namespaces
kubectl create namespace frontend
kubectl create namespace backend
kubectl create namespace database

# Deploy frontend
kubectl create deployment web-app --image=nginx:1.21 --replicas=3 -n frontend

# Deploy backend API
kubectl create deployment api-app --image=node:16-alpine --replicas=2 -n backend

# Deploy database
kubectl create deployment db-app --image=mysql:8.0 --replicas=1 -n database
```

#### Step 2: Create Services for Each Tier
```bash
# Frontend service (LoadBalancer for external access)
kubectl expose deployment web-app --port=80 --type=LoadBalancer -n frontend

# Backend service (ClusterIP for internal access)
kubectl expose deployment api-app --port=3000 --type=ClusterIP -n backend

# Database service (ClusterIP for internal access)
kubectl expose deployment db-app --port=3306 --type=ClusterIP -n database
```

#### Step 3: Verify Service Creation and Endpoints
```bash
kubectl get services --all-namespaces
```
**Expected Output:**
```
NAMESPACE   NAME      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
frontend    web-app   LoadBalancer   10.96.100.10   <pending>     80:30080/TCP   2m
backend     api-app   ClusterIP      10.96.100.20   <none>        3000/TCP       2m
database    db-app    ClusterIP      10.96.100.30   <none>        3306/TCP       2m
```

#### Step 4: Configure Ingress for HTTP Routing
```bash
# Create ingress resource
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: frontend
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-app
            port:
              number: 3000
EOF
```

#### Step 5: Implement Network Policies
```bash
# Create network policy for backend (only allow frontend access)
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-netpol
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: api-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 3000
EOF
```

#### Step 6: Test Connectivity and DNS Resolution
```bash
# Test service DNS resolution
kubectl run test-pod --image=busybox --rm -it --restart=Never -- nslookup api-app.backend.svc.cluster.local

# Test cross-namespace communication
kubectl exec -it -n frontend web-app-xxx -- curl http://api-app.backend.svc.cluster.local:3000
```

### Additional Scenarios
- **Scenario 2:** Troubleshooting service connectivity issues with endpoint verification
- **Scenario 3:** Configuring SSL termination and certificate management with ingress

---

## Section 4: Table of Useful Commands

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `kubectl expose` | Create service for deployment | `--port`, `--type`, `--target-port` | `kubectl expose deployment app --port=80` |
| `kubectl get services` | List services | `-o wide`, `--all-namespaces` | `kubectl get svc -o wide` |
| `kubectl get endpoints` | List service endpoints | `-o yaml` | `kubectl get endpoints my-service` |
| `kubectl describe service` | Show service details | `[service-name]` | `kubectl describe svc my-service` |
| `kubectl get ingress` | List ingress resources | `-o yaml` | `kubectl get ingress -o yaml` |

### Service Management
| Command | Description | Example |
|---------|-------------|---------|
| `kubectl create service` | Create service directly | `kubectl create service clusterip my-svc --tcp=80:8080` |
| `kubectl patch service` | Update service configuration | `kubectl patch svc my-svc -p '{"spec":{"type":"NodePort"}}'` |
| `kubectl port-forward service` | Forward local port to service | `kubectl port-forward svc/my-svc 8080:80` |

### Network Policy Management
| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get networkpolicies` | List network policies | `kubectl get netpol -A` |
| `kubectl describe networkpolicy` | Show policy details | `kubectl describe netpol my-policy` |
| `kubectl apply -f` | Apply network policy | `kubectl apply -f networkpolicy.yaml` |

### DNS and Connectivity Testing
| Command | Description | Example |
|---------|-------------|---------|
| `kubectl run debug` | Create debug pod | `kubectl run debug --image=busybox --rm -it` |
| `nslookup` | DNS resolution test | `nslookup my-service.default.svc.cluster.local` |
| `curl` | HTTP connectivity test | `curl http://my-service:80` |
| `wget` | Download test | `wget -qO- http://my-service:80` |

### Ingress Management
| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get ingress` | List ingress resources | `kubectl get ing -A` |
| `kubectl describe ingress` | Show ingress details | `kubectl describe ing my-ingress` |
| `kubectl create ingress` | Create ingress resource | `kubectl create ingress my-ing --rule="host/path=svc:port"` |

---

## Section 5: Best Practices

### Service Design
1. **Service Type Selection:**
   - Use ClusterIP for internal communication (default and most secure)
   - Use NodePort only when LoadBalancer is not available
   - Use LoadBalancer for production external services
   - Use ExternalName for external service integration
   - Example service manifest:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-app-service
   spec:
     selector:
       app: my-app
     ports:
     - port: 80
       targetPort: 8080
       protocol: TCP
     type: ClusterIP
   ```

2. **Service Naming:**
   - Use descriptive, consistent naming conventions
   - Include environment in name when necessary
   - Follow DNS naming standards (lowercase, hyphens)
   - Consider service discovery implications

3. **Port Management:**
   - Use standard ports when possible (80, 443, 3306)
   - Document custom port usage
   - Avoid port conflicts within namespaces
   - Consider security implications of exposed ports

### Ingress Configuration
1. **Ingress Controller Selection:**
   - Choose based on requirements (nginx, traefik, istio)
   - Consider cloud provider offerings (ALB, GCE)
   - Plan for high availability and scaling
   - Implement health checks and monitoring

2. **SSL/TLS Configuration:**
   - Always use HTTPS in production
   - Implement proper certificate management
   - Use cert-manager for automated certificate lifecycle
   - Configure proper cipher suites and protocols

3. **Routing Strategy:**
   - Use host-based routing for different applications
   - Use path-based routing for microservices
   - Implement proper redirect and rewrite rules
   - Consider canary deployment strategies

### Network Security
1. **Network Policy Implementation:**
   - Start with deny-all default policy
   - Implement least privilege access
   - Use namespace-based isolation
   - Regular policy review and updates
   - Example deny-all policy:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: default-deny-all
   spec:
     podSelector: {}
     policyTypes:
     - Ingress
     - Egress
   ```

2. **Security Best Practices:**
   - Use service meshes for advanced security (istio, linkerd)
   - Implement mTLS for service-to-service communication
   - Regular security scanning and updates
   - Monitor network traffic and anomalies

### DNS and Service Discovery
1. **DNS Configuration:**
   - Understand cluster DNS hierarchy
   - Use fully qualified service names when needed
   - Configure custom DNS policies when required
   - Monitor DNS resolution performance

2. **Service Discovery:**
   - Use services for stable endpoints
   - Implement proper health checks
   - Consider service mesh for advanced discovery
   - Use headless services when needed

### Performance Optimization
1. **Service Performance:**
   - Monitor service response times and errors
   - Implement proper load balancing algorithms
   - Use session affinity when required
   - Optimize endpoint updates and propagation

2. **Network Performance:**
   - Choose appropriate CNI plugin for workload
   - Monitor network latency and throughput
   - Implement proper resource limits
   - Consider network topology and locality

### Troubleshooting Strategies
1. **Service Connectivity Issues:**
   - Verify service and endpoint existence
   - Check pod labels and service selectors
   - Test DNS resolution
   - Validate network policies
   - Example debugging:
   ```bash
   # Check service details
   kubectl describe service my-service
   
   # Check endpoints
   kubectl get endpoints my-service
   
   # Test DNS
   kubectl run debug --image=busybox --rm -it -- nslookup my-service
   
   # Test connectivity
   kubectl run debug --image=busybox --rm -it -- wget -qO- my-service:80
   ```

2. **Ingress Issues:**
   - Verify ingress controller is running
   - Check ingress resource configuration
   - Validate backend service connectivity
   - Review ingress controller logs

3. **Network Policy Issues:**
   - Check policy selector syntax
   - Verify namespace labels
   - Test connectivity step by step
   - Use network policy tools for validation

### Exam-Specific Tips
- **Time Management:** Practice creating services with both imperative and declarative methods
- **Command Shortcuts:** Master kubectl expose command with various options
- **Verification:** Always test service connectivity after creation
- **Documentation:** Know how to quickly find service and ingress examples

---

## Quick Reference Card

### Essential Commands for Services & Networking
```bash
kubectl expose deployment app --port=80 --type=ClusterIP
kubectl get services,endpoints -o wide
kubectl describe service my-service
kubectl get ingress -A
kubectl apply -f networkpolicy.yaml
```

### Service Creation Shortcuts
```bash
# Create ClusterIP service
kubectl expose deployment app --port=80

# Create NodePort service  
kubectl expose deployment app --port=80 --type=NodePort

# Create LoadBalancer service
kubectl expose deployment app --port=80 --type=LoadBalancer

# Create service with specific target port
kubectl expose deployment app --port=80 --target-port=8080
```

### Network Troubleshooting Commands
```bash
kubectl run debug --image=busybox --rm -it -- sh
nslookup my-service.default.svc.cluster.local
wget -qO- http://my-service:80
ping my-service
```

### Key DNS Names
- Service: `my-service.namespace.svc.cluster.local`
- Pod: `pod-ip.namespace.pod.cluster.local`
- Default namespace: `my-service` (short name)

---

## Related Topics
- [Kubernetes Architecture]
- [Workloads & Scheduling]
- [Security and RBAC]
- [Storage and Volumes]

---

*Last Updated: May 29, 2025*
*CKA Exam Version: v1.30*
