# Lab A02: RBAC Security Lockdown

> **📋 Lab Category:** Architecture - Cluster Security & Access Control  
> **⏱️ Estimated Time:** 25-35 minutes  
> **🎯 Difficulty:** Intermediate  
> **🔗 Exam Objectives:** Manage role-based access control (RBAC), Configure security contexts, Understand service accounts

---

## 🎬 Real-World Scenario

### Background Context
**SecureTech Financial** is a fintech company that just experienced a security audit revealing critical RBAC vulnerabilities. The audit found that developers have excessive cluster permissions, service accounts are poorly configured, and there's no proper separation of duties between teams. The company's Kubernetes cluster hosts critical financial applications processing millions of transactions daily.

**Your Role:** Senior Platform Security Engineer  
**Situation:** The security team has mandated immediate RBAC lockdown to meet compliance requirements  
**Urgency:** **HIGH** - Must implement proper access controls within 24 hours to pass regulatory compliance audit

### The Challenge
You must implement a comprehensive RBAC security model that:
- Removes excessive permissions from developers
- Creates proper role separation between Dev, QA, and Ops teams
- Secures service accounts with minimal required permissions
- Implements namespace-based access control
- Ensures Pod Security Standards compliance

**Business Impact:** Failing this audit could result in $2M+ in regulatory fines and loss of banking partnerships.

---

## 🎯 Learning Objectives

By completing this lab, you will:
- [ ] **Primary Skill:** Design and implement comprehensive RBAC policies for multi-team environments
- [ ] **Secondary Skills:** Configure service accounts, roles, role bindings, and security contexts
- [ ] **Real-world Application:** Secure production Kubernetes clusters meeting enterprise compliance requirements
- [ ] **Exam Relevance:** Master CKA RBAC tasks including role creation, binding management, and security context configuration

---

## 🔧 Prerequisites

### Knowledge Requirements
- [ ] Understanding of Kubernetes authentication and authorization concepts
- [ ] Basic knowledge of RBAC components (Roles, ClusterRoles, RoleBindings, ClusterRoleBindings)
- [ ] Familiarity with service accounts and security contexts
- [ ] Previous completion of A01 (Bare Metal Cluster Bootstrap) recommended

### Environment Setup
```bash
# Cluster requirements
- Kubernetes cluster v1.28+ with RBAC enabled
- kubectl configured with cluster-admin access
- Multiple namespaces for team separation
- curl or openssl tools for certificate management

# Verification commands
kubectl cluster-info
kubectl auth can-i '*' '*' --all-namespaces
kubectl get clusterroles | head -10
kubectl get ns
```

---

## 🚨 Initial Crisis State

You arrive to find a cluster with dangerously permissive access controls:

### Current Security Issues
1. **Developers have cluster-admin access** - Can delete entire namespaces
2. **Service accounts use default permissions** - Overly broad access
3. **No namespace isolation** - Teams can access each other's resources
4. **Pod security contexts missing** - Containers running as root
5. **Excessive ClusterRoleBindings** - Too many users with cluster-wide access

### Immediate Symptoms
```bash
# Check current overly permissive setup
kubectl get clusterrolebindings -o wide
kubectl get rolebindings --all-namespaces
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.securityContext}{"\n"}{end}'
```

---

## 📋 Tasks & Solutions

### Task 1: Audit Current RBAC Configuration
**Scenario:** First, understand what security vulnerabilities exist in the current setup.

#### 🔍 Investigation Steps
```bash
# 1. Check who has cluster-admin access
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | "\(.metadata.name): \(.subjects)"'

# 2. List all service accounts and their permissions
kubectl get sa --all-namespaces
kubectl auth can-i --list --as=system:serviceaccount:default:default

# 3. Check for overly permissive roles
kubectl get clusterroles -o json | jq -r '.items[] | select(.rules[]?.verbs[]? == "*") | .metadata.name'

# 4. Identify pods running with excessive privileges
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.securityContext.runAsUser // "ROOT"}{"\n"}{end}'
```

#### ✅ Solution Details
```bash
# Create audit script for comprehensive RBAC assessment
cat << 'EOF' > rbac-audit.sh
#!/bin/bash
echo "=== RBAC Security Audit ==="
echo ""

echo "1. Cluster Admin Bindings:"
kubectl get clusterrolebindings -o custom-columns=NAME:.metadata.name,SUBJECTS:.subjects[*].name | grep -E "cluster-admin|admin"

echo ""
echo "2. Overly Permissive ClusterRoles:"
for role in $(kubectl get clusterroles -o name); do
  if kubectl get $role -o jsonpath='{.rules[*].verbs}' | grep -q '\*'; then
    echo "  $(basename $role) - HAS WILDCARD PERMISSIONS"
  fi
done

echo ""
echo "3. Service Account Permissions Check:"
for ns in $(kubectl get ns -o name | cut -d/ -f2); do
  echo "  Namespace: $ns"
  kubectl auth can-i --list --as=system:serviceaccount:$ns:default 2>/dev/null | head -3
done

echo ""
echo "4. Pods Running as Root:"
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.securityContext.runAsUser == null or .spec.securityContext.runAsUser == 0) | "\(.metadata.namespace)/\(.metadata.name)"'
EOF

chmod +x rbac-audit.sh
./rbac-audit.sh
```

### Task 2: Create Team-Based Namespace Structure
**Scenario:** Implement proper namespace isolation for Dev, QA, and Operations teams.

#### 🔧 Implementation Steps
```bash
# 1. Create team namespaces with proper labels
kubectl create namespace dev-team
kubectl create namespace qa-team  
kubectl create namespace ops-team
kubectl create namespace shared-services

# 2. Label namespaces for RBAC policies
kubectl label namespace dev-team team=development tier=non-prod
kubectl label namespace qa-team team=qa tier=non-prod
kubectl label namespace ops-team team=operations tier=prod
kubectl label namespace shared-services team=shared tier=both

# 3. Verify namespace setup
kubectl get namespaces --show-labels
```

#### ✅ Solution Details
```yaml
# Apply comprehensive namespace configuration
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
  labels:
    team: development
    tier: non-prod
    security.level: standard
---
apiVersion: v1
kind: Namespace
metadata:
  name: qa-team
  labels:
    team: qa
    tier: non-prod
    security.level: standard
---
apiVersion: v1
kind: Namespace
metadata:
  name: ops-team
  labels:
    team: operations
    tier: prod
    security.level: high
---
apiVersion: v1
kind: Namespace
metadata:
  name: shared-services
  labels:
    team: shared
    tier: both
    security.level: standard
---
# Network policies for namespace isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace
  namespace: dev-team
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: development
    - namespaceSelector:
        matchLabels:
          team: shared
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          team: development
    - namespaceSelector:
        matchLabels:
          team: shared
EOF
```

### Task 3: Design Role-Based Access Control Matrix
**Scenario:** Create specific roles for each team with minimum required permissions.

#### 🎯 Role Design Strategy
```bash
# Developer Role: Limited to their namespace, read-only cluster info
# QA Role: Read access to dev namespace, full access to qa namespace
# Operations Role: Broad cluster access for maintenance and monitoring
# Service Account Roles: Minimal permissions for application functionality
```

#### ✅ Solution Implementation
```yaml
# Create team-specific roles and service accounts
cat << 'EOF' | kubectl apply -f -
---
# Developer Role - Namespace-scoped permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets", "persistentvolumeclaims"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "list", "create"]
---
# QA Role - Read dev, full qa access
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: qa-team
  name: qa-engineer
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: qa-readonly
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list"]
---
# Operations ClusterRole - Cluster-wide maintenance access
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ops-engineer
rules:
- apiGroups: [""]
  resources: ["nodes", "persistentvolumes", "namespaces"]
  verbs: ["get", "list", "patch", "update"]
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "daemonsets", "statefulsets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
---
# Service Account for applications
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: dev-team
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: app-minimal-access
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
  resourceNames: ["app-config-secret"]
EOF
```

### Task 4: Create and Bind Team Users
**Scenario:** Set up user certificates and bind them to appropriate roles.

#### 🔐 User Certificate Creation
```bash
# 1. Create private keys for team users
openssl genrsa -out dev-user.key 2048
openssl genrsa -out qa-user.key 2048
openssl genrsa -out ops-user.key 2048

# 2. Create certificate signing requests
openssl req -new -key dev-user.key -out dev-user.csr -subj "/CN=dev-user/O=development"
openssl req -new -key qa-user.key -out qa-user.csr -subj "/CN=qa-user/O=qa"
openssl req -new -key ops-user.key -out ops-user.csr -subj "/CN=ops-user/O=operations"

# 3. Create Kubernetes CSR objects and approve them
for user in dev-user qa-user ops-user; do
  cat << EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ${user}
spec:
  request: $(cat ${user}.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
  kubectl certificate approve ${user}
done
```

#### ✅ User Binding Solution
```yaml
# Create RoleBindings for team users
cat << 'EOF' | kubectl apply -f -
---
# Bind developer to dev namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: dev-team
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
---
# Bind QA user to both qa and dev namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: qa-user-binding
  namespace: qa-team
subjects:
- kind: User
  name: qa-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: qa-engineer
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: qa-readonly-binding
  namespace: dev-team
subjects:
- kind: User
  name: qa-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: qa-readonly
  apiGroup: rbac.authorization.k8s.io
---
# Bind operations user cluster-wide
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ops-user-binding
subjects:
- kind: User
  name: ops-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: ops-engineer
  apiGroup: rbac.authorization.k8s.io
---
# Bind service account to minimal role
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-service-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: dev-team
roleRef:
  kind: Role
  name: app-minimal-access
  apiGroup: rbac.authorization.k8s.io
EOF
```

### Task 5: Implement Pod Security Standards
**Scenario:** Configure security contexts and pod security policies to prevent privilege escalation.

#### 🛡️ Security Context Implementation
```yaml
# Deploy test applications with proper security contexts
cat << 'EOF' | kubectl apply -f -
---
# Secure application deployment in dev namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-web-app
  namespace: dev-team
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secure-web-app
  template:
    metadata:
      labels:
        app: secure-web-app
    spec:
      serviceAccountName: app-service-account
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: web-app
        image: nginx:1.21
        ports:
        - containerPort: 8080
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        - name: var-cache
          mountPath: /var/cache/nginx
        - name: var-run
          mountPath: /var/run
      volumes:
      - name: tmp-volume
        emptyDir: {}
      - name: var-cache
        emptyDir: {}
      - name: var-run
        emptyDir: {}
---
# ConfigMap for application configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev-team
data:
  nginx.conf: |
    user nginx;
    worker_processes auto;
    error_log /dev/stderr;
    pid /var/run/nginx.pid;
    
    events {
        worker_connections 1024;
    }
    
    http {
        server {
            listen 8080;
            location / {
                return 200 'Secure App Running\n';
                add_header Content-Type text/plain;
            }
        }
    }
---
# Secret for application credentials
apiVersion: v1
kind: Secret
metadata:
  name: app-config-secret
  namespace: dev-team
type: Opaque
data:
  api-key: c2VjdXJlLWFwaS1rZXktMTIzNDU=  # base64 encoded "secure-api-key-12345"
EOF
```

### Task 6: Test and Validate RBAC Implementation
**Scenario:** Verify that the RBAC configuration works as intended and prevents unauthorized access.

#### 🧪 Comprehensive Testing Strategy
```bash
# 1. Test developer permissions
echo "=== Testing Developer Access ==="
kubectl auth can-i create pods --as=dev-user --namespace=dev-team
kubectl auth can-i delete namespaces --as=dev-user
kubectl auth can-i get nodes --as=dev-user

# 2. Test QA permissions  
echo "=== Testing QA Access ==="
kubectl auth can-i get pods --as=qa-user --namespace=dev-team
kubectl auth can-i create deployments --as=qa-user --namespace=qa-team
kubectl auth can-i delete pods --as=qa-user --namespace=ops-team

# 3. Test Operations permissions
echo "=== Testing Ops Access ==="
kubectl auth can-i get nodes --as=ops-user
kubectl auth can-i patch nodes --as=ops-user
kubectl auth can-i create namespaces --as=ops-user

# 4. Test service account permissions
echo "=== Testing Service Account Access ==="
kubectl auth can-i get configmaps --as=system:serviceaccount:dev-team:app-service-account --namespace=dev-team
kubectl auth can-i create pods --as=system:serviceaccount:dev-team:app-service-account --namespace=dev-team
```

#### ✅ Validation Script
```bash
# Create comprehensive RBAC validation script
cat << 'EOF' > validate-rbac.sh
#!/bin/bash
echo "=== RBAC Validation Test Suite ==="
echo ""

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Test function
test_permission() {
    local user=$1
    local verb=$2
    local resource=$3
    local namespace=$4
    local expected=$5
    
    if [ -n "$namespace" ]; then
        result=$(kubectl auth can-i $verb $resource --as=$user --namespace=$namespace 2>/dev/null)
    else
        result=$(kubectl auth can-i $verb $resource --as=$user 2>/dev/null)
    fi
    
    if [ "$result" = "$expected" ]; then
        echo -e "${GREEN}✓${NC} $user can $verb $resource in $namespace: $result"
    else
        echo -e "${RED}✗${NC} $user can $verb $resource in $namespace: Expected $expected, got $result"
    fi
}

echo "1. Developer Permissions Test:"
test_permission "dev-user" "create" "pods" "dev-team" "yes"
test_permission "dev-user" "create" "pods" "qa-team" "no"
test_permission "dev-user" "delete" "namespaces" "" "no"
test_permission "dev-user" "get" "nodes" "" "no"

echo ""
echo "2. QA Permissions Test:"
test_permission "qa-user" "get" "pods" "dev-team" "yes"
test_permission "qa-user" "create" "deployments" "qa-team" "yes"
test_permission "qa-user" "delete" "pods" "ops-team" "no"

echo ""
echo "3. Operations Permissions Test:"
test_permission "ops-user" "get" "nodes" "" "yes"
test_permission "ops-user" "patch" "nodes" "" "yes"
test_permission "ops-user" "create" "pods" "ops-team" "yes"

echo ""
echo "4. Service Account Test:"
test_permission "system:serviceaccount:dev-team:app-service-account" "get" "configmaps" "dev-team" "yes"
test_permission "system:serviceaccount:dev-team:app-service-account" "create" "pods" "dev-team" "no"

echo ""
echo "5. Security Context Validation:"
pod_security=$(kubectl get pod -n dev-team -l app=secure-web-app -o jsonpath='{.items[0].spec.securityContext.runAsNonRoot}')
if [ "$pod_security" = "true" ]; then
    echo -e "${GREEN}✓${NC} Pods running with non-root security context"
else
    echo -e "${RED}✗${NC} Pods NOT running with proper security context"
fi

echo ""
echo "=== RBAC Validation Complete ==="
EOF

chmod +x validate-rbac.sh
./validate-rbac.sh
```

---

## 🎯 Verification Commands

```bash
# Final comprehensive security check
echo "=== Final Security Audit ==="

# 1. Verify no users have cluster-admin except necessary system accounts
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .subjects[]? | select(.kind=="User") | .name'

# 2. Check that service accounts have minimal permissions
kubectl describe rolebinding app-service-binding -n dev-team

# 3. Verify pod security contexts are enforced
kubectl get pods -n dev-team -o jsonpath='{range .items[*]}{.metadata.name}{": runAsNonRoot="}{.spec.securityContext.runAsNonRoot}{", runAsUser="}{.spec.securityContext.runAsUser}{"\n"}{end}'

# 4. Test application functionality with new security model
kubectl get pods -n dev-team -l app=secure-web-app
kubectl logs -n dev-team deployment/secure-web-app

# 5. Verify network policies are working
kubectl describe networkpolicy deny-cross-namespace -n dev-team
```

---

## 💡 Key Learning Points

### RBAC Best Practices Mastered
1. **Principle of Least Privilege:** Each user and service account has only the minimum permissions needed
2. **Namespace Isolation:** Teams are properly separated with network policies enforcing boundaries
3. **Role Separation:** Clear distinction between Dev, QA, and Ops responsibilities
4. **Security Context Enforcement:** All applications run with non-root users and restricted capabilities
5. **Service Account Security:** Applications use dedicated service accounts with minimal permissions

### Common RBAC Pitfalls Avoided
- **Avoided:** Giving developers cluster-admin access for convenience
- **Avoided:** Using the default service account for applications
- **Avoided:** Overly broad ClusterRoleBindings that grant unnecessary cluster-wide access
- **Avoided:** Running containers as root without security contexts
- **Avoided:** Cross-namespace access without proper network policies

### Enterprise Security Considerations
- **Compliance:** RBAC model meets SOC2 and PCI DSS requirements
- **Auditing:** All access is properly logged and attributable to specific users
- **Scalability:** Role model can easily extend to additional teams and namespaces
- **Maintenance:** Regular RBAC audits ensure permissions remain appropriate

---

## 🚀 Bonus Challenges

### Advanced Security Hardening
1. **Pod Security Standards:** Implement Pod Security Standards admission controller
2. **OPA Gatekeeper:** Add policy-as-code validation for additional security rules
3. **Admission Controllers:** Configure custom admission controllers for advanced validation
4. **Secret Management:** Integrate with external secret management systems
5. **Certificate Rotation:** Implement automatic certificate rotation for user access

### Real-World Extensions
1. **Multi-Cluster RBAC:** Extend this model across multiple Kubernetes clusters
2. **Integration Testing:** Create automated tests to validate RBAC changes
3. **Monitoring Integration:** Set up alerts for unauthorized access attempts
4. **Documentation:** Create runbooks for common RBAC troubleshooting scenarios

---

## 📚 Additional Resources

- [Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [CKA Exam RBAC Topics](https://github.com/cncf/curriculum/blob/master/CKA_Curriculum_v1.28.pdf)
- [RBAC Security Best Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)

**Next Lab:** N02 - Ingress Traffic Management (Complex load balancing and SSL termination scenarios)

---

*🎯 **Success Criteria:** You've successfully implemented enterprise-grade RBAC security that protects the cluster while enabling team productivity. The audit findings have been resolved, and SecureTech Financial now has a compliant, secure Kubernetes environment ready for production financial workloads.*
