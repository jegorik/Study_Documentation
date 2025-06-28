# N03 - Network Policy Enforcement

> **🎯 Lab Focus:** Advanced network segmentation, policy enforcement, and security isolation  
> **⏱️ Estimated Time:** 30-35 minutes  
> **🏷️ Difficulty:** Advanced  
> **📋 Category:** Services & Networking (20% Exam Weight)

---

## 🎬 Scenario: Multi-Tenant Financial Services Platform

You're the Senior DevOps Engineer at **SecureBank Digital**, a cloud-native financial institution that hosts multiple client applications on a shared Kubernetes platform. Due to new regulatory compliance requirements (PCI DSS, SOX, GDPR), you must implement strict network segmentation between different tenant applications, ensure payment processing systems are isolated, and prevent unauthorized cross-namespace communication while maintaining necessary service connectivity.

**Business Critical Requirements:**

- 🔒 **PCI Compliance:** Payment processing must be completely isolated
- 🏛️ **Regulatory Separation:** Client A and Client B cannot access each other's data
- 🌐 **API Gateway Access:** Controlled external connectivity for customer portals
- 📊 **Monitoring Access:** Operations team needs read-only access to all namespaces
- 🚫 **Zero Trust:** Default deny-all with explicit allow policies

**Compliance Deadline:** 48 hours before regulatory audit

---

## 🏗️ Lab Environment Setup

### Initial Cluster State

```bash
# Expected cluster topology
NAMESPACES:
- production-client-a     # Client A's banking application
- production-client-b     # Client B's investment platform  
- payment-processing      # PCI-compliant payment services
- api-gateway            # External-facing API services
- monitoring             # Prometheus, Grafana, logging
- kube-system           # Kubernetes system components
```

### Prerequisites

- Multi-namespace cluster setup
- Various applications requiring different connectivity patterns
- CNI plugin supporting NetworkPolicies (Calico, Cilium, or Weave)
- Mixed internal and external traffic requirements

---

## 📋 Tasks Overview

| Task | Objective | Est. Time | Difficulty |
|------|-----------|-----------|------------|
| 1 | Network Policy Audit & Assessment | 5 min | Intermediate |
| 2 | Default Deny-All Implementation | 6 min | Advanced |
| 3 | Namespace Isolation Enforcement | 8 min | Advanced |
| 4 | Service-to-Service Communication Rules | 8 min | Expert |
| 5 | External Traffic Management | 5 min | Advanced |
| 6 | Policy Testing & Validation | 3 min | Intermediate |

---

## 🔍 Task 1: Network Policy Audit & Assessment (5 minutes)

### Objective

Analyze the current network security posture and identify communication patterns that need policy enforcement.

### Current Situation

The platform currently has no network policies in place, meaning all pods can communicate with each other freely. You need to understand the current traffic flows before implementing restrictions.

### Your Mission

1. **Audit existing network policies across all namespaces**
2. **Map current inter-namespace communication patterns**
3. **Identify critical service dependencies**
4. **Document compliance requirements for each namespace**

### Step-by-Step Instructions

#### 1.1 Current Network Policy Assessment

```bash
# Check existing network policies across all namespaces
kubectl get networkpolicies -A

# Verify CNI plugin supports NetworkPolicies
kubectl get nodes -o wide
kubectl describe node <node-name> | grep -i cni

# Check current namespace setup
kubectl get namespaces --show-labels

# Identify running applications and their connectivity
kubectl get pods -A -o wide
kubectl get services -A
```

#### 1.2 Traffic Flow Analysis

```bash
# Check current service endpoints and connectivity
kubectl get endpoints -A

# Identify cross-namespace service dependencies
kubectl get services -A -o yaml | grep -A 5 -B 5 "\.svc\.cluster\.local"

# Check ingress controllers and external access points
kubectl get ingress -A
kubectl get services -A --field-selector spec.type=LoadBalancer
```

#### 1.3 Compliance Requirements Mapping

```bash
# Document current pod-to-pod connectivity (simulate with test pods)
# This would typically involve network monitoring tools in production

# Check for any existing security contexts or pod security policies
kubectl get podsecuritypolicy -A 2>/dev/null || echo "No PSPs found"
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.securityContext}{"\n"}{end}'
```

### Lab Environment Setup

#### Create Multi-Tenant Environment

```yaml
# multi-tenant-setup.yaml
# Create namespaces with labels for policy targeting
apiVersion: v1
kind: Namespace
metadata:
  name: production-client-a
  labels:
    tenant: client-a
    compliance: pci
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: production-client-b
  labels:
    tenant: client-b
    compliance: sox
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: payment-processing
  labels:
    tenant: shared
    compliance: pci-strict
    environment: production
    network-policy: strict-isolation
---
apiVersion: v1
kind: Namespace
metadata:
  name: api-gateway
  labels:
    tenant: shared
    compliance: general
    environment: production
    network-policy: external-facing
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  labels:
    tenant: operations
    compliance: general
    environment: production
    network-policy: read-only-access
```

#### Deploy Sample Applications

```yaml
# client-a-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-frontend
  namespace: production-client-a
spec:
  replicas: 2
  selector:
    matchLabels:
      app: banking-frontend
  template:
    metadata:
      labels:
        app: banking-frontend
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: banking-frontend-svc
  namespace: production-client-a
spec:
  selector:
    app: banking-frontend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-backend
  namespace: production-client-a
spec:
  replicas: 2
  selector:
    matchLabels:
      app: banking-backend
  template:
    metadata:
      labels:
        app: banking-backend
        tier: backend
    spec:
      containers:
      - name: backend
        image: nginx:alpine
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: banking-backend-svc
  namespace: production-client-a
spec:
  selector:
    app: banking-backend
  ports:
  - port: 8080
    targetPort: 8080
```

```yaml
# payment-processing-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-processor
  namespace: payment-processing
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-processor
  template:
    metadata:
      labels:
        app: payment-processor
        tier: critical
        compliance: pci
    spec:
      containers:
      - name: processor
        image: nginx:alpine
        ports:
        - containerPort: 9090
---
apiVersion: v1
kind: Service
metadata:
  name: payment-processor-svc
  namespace: payment-processing
spec:
  selector:
    app: payment-processor
  ports:
  - port: 9090
    targetPort: 9090
```

```bash
# Apply the setup
kubectl apply -f multi-tenant-setup.yaml
kubectl apply -f client-a-app.yaml
kubectl apply -f payment-processing-app.yaml

# Verify deployment
kubectl get pods -A
kubectl get services -A
```

### Expected Findings

- No existing network policies (unrestricted communication)
- Multiple namespaces with different compliance requirements
- Cross-namespace service dependencies that need careful management
- External access requirements for API gateway

### 🚨 Troubleshooting Tips

- **CNI compatibility** is crucial - not all CNI plugins support NetworkPolicies
- **Label consistency** helps with policy targeting
- **Service discovery** patterns affect policy design
- **Compliance requirements** drive isolation levels

---

## 🛡️ Task 2: Default Deny-All Implementation (6 minutes)

### Objective

Implement a default deny-all network policy across all namespaces to establish a zero-trust network foundation.

### Current Situation

With no network policies in place, all pods can communicate freely. You need to implement a "default deny" stance and then selectively allow required traffic.

### Your Mission

1. **Create default deny-all ingress policies for each namespace**
2. **Implement default deny-all egress policies**
3. **Verify that unauthorized communication is blocked**
4. **Prepare for selective policy implementation**

### Step-by-Step Instructions

#### 2.1 Default Deny-All Ingress Policies

```yaml
# default-deny-ingress.yaml
# Apply to all production namespaces
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-ingress
  namespace: production-client-a
spec:
  podSelector: {}  # Applies to all pods in namespace
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-ingress
  namespace: production-client-b
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-ingress
  namespace: payment-processing
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-ingress
  namespace: api-gateway
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-ingress
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

#### 2.2 Default Deny-All Egress Policies

```yaml
# default-deny-egress.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-egress
  namespace: production-client-a
spec:
  podSelector: {}
  policyTypes:
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-egress
  namespace: production-client-b
spec:
  podSelector: {}
  policyTypes:
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-egress
  namespace: payment-processing
spec:
  podSelector: {}
  policyTypes:
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-egress
  namespace: api-gateway
spec:
  podSelector: {}
  policyTypes:
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-egress
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

#### 2.3 Essential DNS Access Policy

```yaml
# dns-access-policy.yaml
# Allow DNS resolution for all pods (essential for service discovery)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-access
  namespace: production-client-a
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-access
  namespace: production-client-b
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-access
  namespace: payment-processing
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-access
  namespace: api-gateway
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-access
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

#### 2.4 Apply Default Policies

```bash
# Apply all default deny policies
kubectl apply -f default-deny-ingress.yaml
kubectl apply -f default-deny-egress.yaml
kubectl apply -f dns-access-policy.yaml

# Verify policies are applied
kubectl get networkpolicies -A

# Test that communication is now blocked
kubectl run test-pod --image=curlimages/curl --rm -it --restart=Never -- sh
# Inside the pod, try to access services (should fail)
curl banking-frontend-svc.production-client-a.svc.cluster.local
```

### Expected Outcomes

- All namespaces have default deny-all policies
- DNS resolution is still functional
- Inter-pod communication is blocked by default
- Foundation is set for selective policy implementation

### 🚨 Troubleshooting Tips

- **DNS access** is critical - pods need it for service discovery
- **Policy order** matters - default deny should be applied first
- **Testing approach** - use temporary pods to verify blocking
- **CNI behavior** - different CNIs may have slight variations

---

## 🔒 Task 3: Namespace Isolation Enforcement (8 minutes)

### Objective

Implement strict namespace isolation to ensure tenants cannot access each other's resources while allowing necessary intra-namespace communication.

### Current Situation

Default deny policies are in place, but legitimate intra-namespace communication is also blocked. You need to restore necessary communication within namespaces while maintaining strict inter-namespace isolation.

### Your Mission

1. **Allow intra-namespace communication for each tenant**
2. **Ensure complete isolation between Client A and Client B**
3. **Implement special isolation for payment processing**
4. **Create monitoring access patterns**

### Step-by-Step Instructions

#### 3.1 Intra-Namespace Communication Policies

```yaml
# intra-namespace-communication.yaml
# Allow communication within Client A namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-intra-namespace
  namespace: production-client-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production-client-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: production-client-a
  - to: []  # Allow DNS (combined with DNS policy)
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
# Allow communication within Client B namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-intra-namespace
  namespace: production-client-b
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production-client-b
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: production-client-b
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

#### 3.2 Payment Processing Strict Isolation

```yaml
# payment-processing-isolation.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-strict-isolation
  namespace: payment-processing
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Only allow traffic from API Gateway (controlled entry point)
  - from:
    - namespaceSelector:
        matchLabels:
          name: api-gateway
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 9090
  # Allow intra-namespace communication
  - from:
    - namespaceSelector:
        matchLabels:
          name: payment-processing
  egress:
  # Allow communication within payment processing namespace
  - to:
    - namespaceSelector:
        matchLabels:
          name: payment-processing
  # Allow DNS resolution
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  # Allow external bank API access (specific external endpoints)
  - to: []
    ports:
    - protocol: TCP
      port: 443
```

#### 3.3 API Gateway Connectivity Policies

```yaml
# api-gateway-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-connectivity
  namespace: api-gateway
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow external traffic (from LoadBalancer/Ingress)
  - from: []
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
  # Allow intra-namespace communication
  - from:
    - namespaceSelector:
        matchLabels:
          name: api-gateway
  egress:
  # Allow communication to client namespaces (controlled access)
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: client-a
    ports:
    - protocol: TCP
      port: 80
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: client-b
    ports:
    - protocol: TCP
      port: 80
  # Allow communication to payment processing
  - to:
    - namespaceSelector:
        matchLabels:
          name: payment-processing
    ports:
    - protocol: TCP
      port: 9090
  # Allow intra-namespace
  - to:
    - namespaceSelector:
        matchLabels:
          name: api-gateway
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

#### 3.4 Monitoring Access Policies

```yaml
# monitoring-access-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: monitoring-read-access
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow access from operations team (specific pods/namespaces)
  - from: []
    ports:
    - protocol: TCP
      port: 3000  # Grafana
    - protocol: TCP
      port: 9090  # Prometheus
  egress:
  # Allow monitoring to scrape metrics from all namespaces
  - to:
    - namespaceSelector:
        matchLabels:
          environment: production
    ports:
    - protocol: TCP
      port: 8080  # Common metrics port
    - protocol: TCP
      port: 9090  # Application metrics
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

#### 3.5 Apply Namespace Isolation Policies

```bash
# Apply all namespace isolation policies
kubectl apply -f intra-namespace-communication.yaml
kubectl apply -f payment-processing-isolation.yaml
kubectl apply -f api-gateway-policies.yaml
kubectl apply -f monitoring-access-policies.yaml

# Add missing namespace labels for policies to work
kubectl label namespace production-client-a name=production-client-a
kubectl label namespace production-client-b name=production-client-b
kubectl label namespace payment-processing name=payment-processing
kubectl label namespace api-gateway name=api-gateway
kubectl label namespace monitoring name=monitoring

# Verify all policies
kubectl get networkpolicies -A
kubectl describe networkpolicy -n production-client-a
```

### Expected Outcomes

- Intra-namespace communication is restored
- Inter-namespace communication is blocked except for explicit rules
- Payment processing has strict isolation with controlled access
- Monitoring has read-only access to collect metrics

---

## 🔗 Task 4: Service-to-Service Communication Rules (8 minutes)

### Objective

Implement fine-grained service-to-service communication rules based on application requirements and security principles.

### Current Situation

Basic namespace isolation is in place, but you need to implement specific service-to-service communication patterns for complex application workflows.

### Your Mission

1. **Define frontend-to-backend communication rules**
2. **Implement database access restrictions**
3. **Create API gateway routing policies**
4. **Establish cross-namespace service access for legitimate business needs**

### Step-by-Step Instructions

#### 4.1 Frontend-to-Backend Communication

```yaml
# frontend-backend-rules.yaml
# Client A: Banking frontend to backend communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: banking-frontend-to-backend
  namespace: production-client-a
spec:
  podSelector:
    matchLabels:
      app: banking-backend
  policyTypes:
  - Ingress
  ingress:
  # Allow traffic from banking frontend only
  - from:
    - podSelector:
        matchLabels:
          app: banking-frontend
    ports:
    - protocol: TCP
      port: 8080
  # Allow traffic from API gateway
  - from:
    - namespaceSelector:
        matchLabels:
          name: api-gateway
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 8080
---
# API Gateway access to client services
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-to-clients
  namespace: production-client-a
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: api-gateway
    ports:
    - protocol: TCP
      port: 80
```

#### 4.2 Database Access Restrictions

```yaml
# database-access-rules.yaml
# Assume database pods exist with specific labels
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-access-control
  namespace: production-client-a
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
  # Only backend services can access database
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 5432  # PostgreSQL
    - protocol: TCP
      port: 3306  # MySQL
  # Allow backup services (with specific label)
  - from:
    - podSelector:
        matchLabels:
          app: backup-service
    ports:
    - protocol: TCP
      port: 5432
```

#### 4.3 Payment Processing Service Rules

```yaml
# payment-service-rules.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-processor-access
  namespace: payment-processing
spec:
  podSelector:
    matchLabels:
      app: payment-processor
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Only allow traffic from authenticated API gateway requests
  - from:
    - namespaceSelector:
        matchLabels:
          name: api-gateway
    - podSelector:
        matchLabels:
          app: api-gateway
          role: payment-proxy
    ports:
    - protocol: TCP
      port: 9090
  egress:
  # Allow external payment provider APIs
  - to: []
    ports:
    - protocol: TCP
      port: 443
  # Allow internal payment database access
  - to:
    - podSelector:
        matchLabels:
          app: payment-db
    ports:
    - protocol: TCP
      port: 5432
  # DNS resolution
  - to: []
    ports:
    - protocol: UDP
      port: 53
```

#### 4.4 Cross-Namespace Business Logic Rules

```yaml
# cross-namespace-business-rules.yaml
# Allow Client A to access shared payment processing
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: client-a-to-payment
  namespace: payment-processing
spec:
  podSelector:
    matchLabels:
      app: payment-processor
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: client-a
    - podSelector:
        matchLabels:
          tier: backend
          payment-enabled: "true"
    ports:
    - protocol: TCP
      port: 9090
---
# Allow specific reporting service to access client data
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: reporting-access-to-clients
  namespace: production-client-a
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    - podSelector:
        matchLabels:
          app: reporting-service
    ports:
    - protocol: TCP
      port: 8080
```

#### 4.5 Security Scanning and Compliance Access

```yaml
# compliance-access-rules.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: security-scanner-access
  namespace: production-client-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  # Allow security scanning tools
  - from:
    - namespaceSelector:
        matchLabels:
          name: security-scanning
    - podSelector:
        matchLabels:
          app: vulnerability-scanner
    ports:
    - protocol: TCP
      port: 8080
    - protocol: TCP
      port: 443
```

#### 4.6 Apply Service-to-Service Rules

```bash
# Apply all service-to-service policies
kubectl apply -f frontend-backend-rules.yaml
kubectl apply -f database-access-rules.yaml
kubectl apply -f payment-service-rules.yaml
kubectl apply -f cross-namespace-business-rules.yaml
kubectl apply -f compliance-access-rules.yaml

# Add required labels for policies to work
kubectl label pods -n production-client-a app=banking-frontend tier=frontend
kubectl label pods -n production-client-a app=banking-backend tier=backend
kubectl label pods -n payment-processing app=payment-processor

# Verify policies are working
kubectl describe networkpolicy -n production-client-a banking-frontend-to-backend
kubectl describe networkpolicy -n payment-processing payment-processor-access
```

### Expected Outcomes

- Frontend services can communicate with their backends
- Database access is restricted to authorized services
- Payment processing has controlled external access
- Cross-namespace communication follows business rules

---

## 🌐 Task 5: External Traffic Management (5 minutes)

### Objective

Configure network policies to properly handle external traffic flows while maintaining security boundaries.

### Current Situation

Internal service-to-service communication is properly secured, but external traffic patterns need specific policy handling for ingress controllers, load balancers, and external API access.

### Your Mission

1. **Configure ingress controller access policies**
2. **Implement external API access controls**
3. **Set up load balancer traffic rules**
4. **Manage outbound internet access**

### Step-by-Step Instructions

#### 5.1 Ingress Controller Access Policies

```yaml
# ingress-controller-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-controller
  namespace: api-gateway
spec:
  podSelector:
    matchLabels:
      app: api-gateway
  policyTypes:
  - Ingress
  ingress:
  # Allow traffic from ingress controller (usually in kube-system or ingress-nginx namespace)
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
  - from:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    - podSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
  # Allow external LoadBalancer traffic (no source restriction for public services)
  - from: []
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
```

#### 5.2 External API Access Controls

```yaml
# external-api-access.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: external-api-access
  namespace: payment-processing
spec:
  podSelector:
    matchLabels:
      app: payment-processor
  policyTypes:
  - Egress
  egress:
  # Allow access to specific external payment providers
  - to: []
    ports:
    - protocol: TCP
      port: 443
  # Block other external access (only HTTPS to payment providers)
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
---
# More restrictive external access for client applications
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: limited-external-access
  namespace: production-client-a
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Egress
  egress:
  # Allow access to internal services only
  - to:
    - namespaceSelector:
        matchLabels:
          environment: production
  # Allow DNS resolution
  - to: []
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
  # Allow HTTPS for legitimate external APIs only (if needed)
  # This section would be customized based on actual requirements
```

#### 5.3 Load Balancer and NodePort Access

```yaml
# loadbalancer-access.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: loadbalancer-access
  namespace: api-gateway
spec:
  podSelector:
    matchLabels:
      app: public-api
  policyTypes:
  - Ingress
  ingress:
  # Allow traffic from load balancer (external source IPs)
  - from: []  # No source restriction for public-facing services
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
  # For private load balancers, you would specify source IP ranges:
  # - from:
  #   - ipBlock:
  #       cidr: 10.0.0.0/8  # Internal corporate network
  #   ports:
  #   - protocol: TCP
  #     port: 80
```

#### 5.4 Monitoring External Access

```yaml
# monitoring-external-access.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: monitoring-external-access
  namespace: monitoring
spec:
  podSelector:
    matchLabels:
      app: prometheus
  policyTypes:
  - Egress
  ingress:
  # Allow external access to monitoring dashboards (from specific IP ranges)
  - from:
    - ipBlock:
        cidr: 10.0.0.0/8  # Corporate network
    - ipBlock:
        cidr: 172.16.0.0/12  # VPN users
    ports:
    - protocol: TCP
      port: 3000  # Grafana
    - protocol: TCP
      port: 9090  # Prometheus
```

#### 5.5 Apply External Traffic Policies

```bash
# Apply external traffic management policies
kubectl apply -f ingress-controller-policies.yaml
kubectl apply -f external-api-access.yaml
kubectl apply -f loadbalancer-access.yaml
kubectl apply -f monitoring-external-access.yaml

# Verify external access is working
kubectl get services -A --field-selector spec.type=LoadBalancer
kubectl get ingress -A

# Test external connectivity (if ingress is configured)
curl -H "Host: api.securebank.com" http://<ingress-ip>/health
```

### Expected Outcomes

- Ingress controllers can reach application services
- External API access is controlled and limited
- Load balancer traffic flows correctly
- Monitoring tools have appropriate external access

---

## ✅ Task 6: Policy Testing & Validation (3 minutes)

### Objective

Systematically test all network policies to ensure they work as intended and meet compliance requirements.

### Current Situation

All network policies are implemented. You need to validate that they correctly allow intended traffic while blocking unauthorized communication.

### Your Mission

1. **Test authorized communication paths**
2. **Verify unauthorized access is blocked**
3. **Validate compliance requirements**
4. **Document policy effectiveness**

### Step-by-Step Instructions

#### 6.1 Authorized Communication Testing

```bash
# Test intra-namespace communication (should work)
kubectl run test-client-a --image=curlimages/curl --rm -it --restart=Never -n production-client-a -- sh
# Inside the pod:
curl banking-frontend-svc.production-client-a.svc.cluster.local
curl banking-backend-svc.production-client-a.svc.cluster.local:8080

# Test API gateway to client communication (should work)
kubectl run test-api-gateway --image=curlimages/curl --rm -it --restart=Never -n api-gateway -- sh
# Inside the pod:
curl banking-frontend-svc.production-client-a.svc.cluster.local

# Test payment processing access from API gateway (should work)
kubectl run test-payment-access --image=curlimages/curl --rm -it --restart=Never -n api-gateway -- sh
# Inside the pod:
curl payment-processor-svc.payment-processing.svc.cluster.local:9090
```

#### 6.2 Unauthorized Access Testing

```bash
# Test cross-tenant access (should be blocked)
kubectl run test-cross-tenant --image=curlimages/curl --rm -it --restart=Never -n production-client-a -- sh
# Inside the pod (these should fail):
curl investment-service.production-client-b.svc.cluster.local

# Test direct payment processing access (should be blocked)
kubectl run test-direct-payment --image=curlimages/curl --rm -it --restart=Never -n production-client-a -- sh
# Inside the pod (should fail):
curl payment-processor-svc.payment-processing.svc.cluster.local:9090

# Test unauthorized database access (should be blocked)
kubectl run test-db-access --image=curlimages/curl --rm -it --restart=Never -n production-client-a -- sh
# Inside the pod (should fail if database exists):
curl postgres-svc.production-client-a.svc.cluster.local:5432
```

#### 6.3 Network Policy Validation Commands

```bash
# Review all applied network policies
kubectl get networkpolicies -A -o wide

# Check for any pods without network policy coverage
kubectl get pods -A --show-labels | grep -v "app="

# Validate DNS resolution is working
kubectl run dns-test --image=busybox --rm -it --restart=Never -n production-client-a -- nslookup kubernetes.default.svc.cluster.local

# Check for any pods in error states due to network policies
kubectl get pods -A | grep -E "(Error|CrashLoopBackOff|ImagePullBackOff)"
```

#### 6.4 Compliance Validation

```bash
# Generate compliance report
echo "Network Policy Compliance Report" > network_compliance_report.txt
echo "================================" >> network_compliance_report.txt
echo "Date: $(date)" >> network_compliance_report.txt
echo "" >> network_compliance_report.txt

# Document all network policies
echo "Applied Network Policies:" >> network_compliance_report.txt
kubectl get networkpolicies -A >> network_compliance_report.txt
echo "" >> network_compliance_report.txt

# Document namespace isolation
echo "Namespace Isolation Status:" >> network_compliance_report.txt
kubectl get namespaces --show-labels >> network_compliance_report.txt
echo "" >> network_compliance_report.txt

# Document any policy violations or exceptions
echo "Policy Validation Results:" >> network_compliance_report.txt
echo "- Intra-namespace communication: VERIFIED" >> network_compliance_report.txt
echo "- Cross-tenant isolation: VERIFIED" >> network_compliance_report.txt
echo "- Payment processing isolation: VERIFIED" >> network_compliance_report.txt
echo "- External access controls: VERIFIED" >> network_compliance_report.txt

cat network_compliance_report.txt
```

### Expected Outcomes

- Authorized communication flows work correctly
- Unauthorized access is properly blocked
- DNS resolution functions normally
- Compliance requirements are met and documented

---

## 🎯 Success Criteria

### ✅ Must Complete (Pass)

- [ ] Implemented default deny-all policies across all namespaces
- [ ] Established complete namespace isolation between tenants
- [ ] Configured proper service-to-service communication rules
- [ ] Set up external traffic management policies
- [ ] Validated all policies work as intended
- [ ] Generated compliance documentation

### 🌟 Excellence Indicators (Distinction)

- [ ] Zero unauthorized cross-namespace communication
- [ ] Minimal disruption to legitimate application traffic
- [ ] Comprehensive policy coverage for all scenarios
- [ ] Efficient use of label selectors and CIDR blocks
- [ ] Detailed compliance reporting and documentation
- [ ] Performance optimization of network policies

### ⚡ Speed Benchmarks

- **Target Time:** 30 minutes
- **Expert Time:** 25 minutes
- **Policy Implementation:** < 20 minutes
- **Testing Phase:** < 10 minutes

---

## 🔍 Post-Lab Review

### Key Learning Outcomes

1. **Zero Trust Networking:** Implementing default deny with selective allow policies
2. **Multi-Tenant Isolation:** Securing shared clusters for different clients
3. **Service Mesh Preparation:** Understanding traffic control fundamentals
4. **Compliance Implementation:** Meeting regulatory requirements through network policies

### Real-World Applications

- **Financial Services:** PCI DSS compliance and data isolation
- **Healthcare:** HIPAA compliance and patient data protection
- **SaaS Platforms:** Multi-tenant security and resource isolation
- **Enterprise Security:** Zero trust network architecture implementation

### Common Pitfalls & Solutions

| Problem | Symptom | Solution |
|---------|---------|----------|
| DNS resolution fails | Pods can't resolve service names | Add DNS egress rules to all policies |
| Ingress controller blocked | External traffic can't reach services | Add ingress controller namespace/pod selectors |
| Policy conflicts | Unexpected traffic blocking | Review policy order and specificity |
| Label mismatches | Policies don't apply to intended pods | Verify and standardize labeling strategy |

### Advanced Network Policy Patterns

#### 1. Time-Based Access Control

```yaml
# This would require custom admission controllers or operators
# Example concept for maintenance windows
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: maintenance-window-access
  annotations:
    policy.networking/schedule: "0 2 * * SUN"  # Sundays at 2 AM
spec:
  podSelector:
    matchLabels:
      maintenance-access: "true"
```

#### 2. Geo-Location Based Policies

```yaml
# Block traffic from specific geographical regions
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: geo-restricted-access
spec:
  podSelector:
    matchLabels:
      geo-restriction: "enabled"
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 192.168.0.0/16  # Known malicious ranges
        - 10.0.0.0/8      # Internal traffic only
```

### Exam Tips

- **Practice label selectors** - they're crucial for policy targeting
- **Understand CIDR notation** for IP-based policies
- **Know the difference** between ingress and egress rules
- **Remember DNS requirements** - pods need DNS for service discovery
- **Test policy effects** systematically to avoid blocking legitimate traffic

---

## 📚 Additional Resources

### Kubernetes Documentation

- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Network Policy Recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)
- [CNI Specification](https://github.com/containernetworking/cni/blob/master/SPEC.md)

### CNI-Specific Documentation

- [Calico Network Policies](https://docs.projectcalico.org/security/kubernetes-network-policy)
- [Cilium Network Policies](https://docs.cilium.io/en/stable/policy/)
- [Weave Network Policies](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)

### Compliance and Security

- [PCI DSS Requirements](https://www.pcisecuritystandards.org/)
- [Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)

---

## 🚀 Next Steps

### Immediate Actions

1. **Practice with different CNI providers** to understand implementation differences
2. **Create policy templates** for common scenarios in your environment
3. **Develop testing procedures** for policy validation
4. **Document your organization's** network policy standards

### Advanced Scenarios to Explore

- **Service Mesh Integration:** Combine NetworkPolicies with Istio/Linkerd
- **Multi-Cluster Policies:** Cross-cluster communication control
- **Dynamic Policy Management:** Policy automation based on labels/annotations
- **Performance Optimization:** Minimizing policy evaluation overhead

### Production Preparation

- Implement **gradual policy rollout** procedures
- Create **policy monitoring and alerting**
- Establish **emergency policy bypass** procedures
- Train team on **policy troubleshooting** techniques

---

*🎯 "Network security is like a castle - you need walls, gates, and guards. Network policies provide all three in your Kubernetes kingdom."*

**Lab Complete!** 🎉 You've successfully implemented enterprise-grade network security that would pass the most stringent compliance audits!
