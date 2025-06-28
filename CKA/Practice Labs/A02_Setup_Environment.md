# A02 Setup Environment: RBAC Security Lockdown

> **🎯 Purpose:** Create a vulnerable RBAC environment that simulates real-world security issues  
> **⚠️ Warning:** This setup intentionally creates security vulnerabilities for learning purposes  
> **🔧 Cleanup:** Run cleanup commands after completing the lab

---

## 🚀 Quick Setup (5 minutes)

```bash
# Run this script to create the vulnerable environment
curl -sSL https://raw.githubusercontent.com/cka-labs/setups/main/A02-setup.sh | bash
```

**OR** follow the manual setup below:

---

## 📋 Manual Environment Setup

### Step 1: Create Overly Permissive Users
```bash
# Create users with excessive permissions (simulating current state)
kubectl create clusterrolebinding dangerous-dev-binding \
  --clusterrole=cluster-admin \
  --user=dev-user

kubectl create clusterrolebinding dangerous-qa-binding \
  --clusterrole=cluster-admin \
  --user=qa-user

# Create service account with default permissions
kubectl create serviceaccount default-app-sa
kubectl create clusterrolebinding default-app-binding \
  --clusterrole=edit \
  --serviceaccount=default:default-app-sa
```

### Step 2: Deploy Insecure Applications
```yaml
# Create applications running with poor security practices
cat << 'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: insecure-app
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: insecure-app
  template:
    metadata:
      labels:
        app: insecure-app
    spec:
      # No service account specified - uses default
      # No security context - runs as root
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        # Container runs as root by default
        # No security context restrictions
---
apiVersion: apps/v1
kind: Deployment  
metadata:
  name: privileged-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: privileged-app
  template:
    metadata:
      labels:
        app: privileged-app
    spec:
      containers:
      - name: privileged
        image: busybox:1.35
        command: ["sleep", "3600"]
        securityContext:
          privileged: true  # Highly dangerous!
          runAsUser: 0      # Running as root
        volumeMounts:
        - name: host-root
          mountPath: /host
      volumes:
      - name: host-root
        hostPath:
          path: /
          type: Directory
EOF
```

### Step 3: Create Secrets and ConfigMaps with Excessive Access
```bash
# Create sensitive data that shouldn't be widely accessible
kubectl create secret generic financial-secrets \
  --from-literal=db-password=super-secret-123 \
  --from-literal=api-key=sk-prod-1234567890 \
  --from-literal=jwt-secret=ultra-secret-jwt-key

kubectl create configmap sensitive-config \
  --from-literal=database-url=postgresql://prod-db:5432/financial \
  --from-literal=redis-url=redis://prod-cache:6379 \
  --from-literal=debug-mode=true

# Make secrets accessible to default service account (vulnerability)
kubectl create rolebinding default-secret-access \
  --clusterrole=edit \
  --serviceaccount=default:default
```

### Step 4: Create Multiple Namespaces Without Proper Isolation
```bash
# Create namespaces that will need proper RBAC
kubectl create namespace development
kubectl create namespace qa-testing
kubectl create namespace production
kubectl create namespace shared-tools

# Add some workloads to different namespaces
kubectl create deployment test-app --image=nginx:1.21 --namespace=development
kubectl create deployment staging-app --image=nginx:1.21 --namespace=qa-testing
kubectl create deployment critical-app --image=nginx:1.21 --namespace=production
```

### Step 5: Verify Vulnerable State
```bash
# Confirm the environment has security issues
echo "=== Checking Security Vulnerabilities ==="

echo "1. Users with cluster-admin access:"
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | "\(.metadata.name): \(.subjects)"'

echo ""
echo "2. Pods running as root:"
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.securityContext.runAsUser // "ROOT"}{"\n"}{end}'

echo ""
echo "3. Privileged containers:"
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[]?.securityContext.privileged == true) | "\(.metadata.namespace)/\(.metadata.name)"'

echo ""
echo "4. Default service account permissions:"
kubectl auth can-i '*' '*' --as=system:serviceaccount:default:default

echo ""
echo "5. Secrets accessible to default SA:"
kubectl auth can-i get secrets --as=system:serviceaccount:default:default
```

---

## 🎯 Environment Validation

Before starting the lab, verify these conditions exist:

### Expected Vulnerabilities ✅
- [ ] Multiple users have cluster-admin access
- [ ] Applications running as root (UID 0)
- [ ] Privileged containers with host access
- [ ] Default service accounts with excessive permissions
- [ ] Secrets accessible to unauthorized accounts
- [ ] No namespace isolation or network policies
- [ ] No pod security standards enforcement

### Quick Validation Script
```bash
cat << 'EOF' > check-vulnerable-state.sh
#!/bin/bash
echo "=== Vulnerability Assessment ==="

# Check 1: Excessive cluster-admin bindings
admin_count=$(kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .subjects[]? | select(.kind=="User") | .name' | wc -l)
echo "User accounts with cluster-admin: $admin_count (should be >2 for this lab)"

# Check 2: Root containers
root_pods=$(kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.securityContext.runAsUser == null or .spec.securityContext.runAsUser == 0) | "\(.metadata.namespace)/\(.metadata.name)"' | wc -l)
echo "Pods running as root: $root_pods (should be >0)"

# Check 3: Privileged containers
priv_pods=$(kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[]?.securityContext.privileged == true) | "\(.metadata.namespace)/\(.metadata.name)"' | wc -l)
echo "Privileged containers: $priv_pods (should be >0)"

# Check 4: Default SA permissions
default_perms=$(kubectl auth can-i get secrets --as=system:serviceaccount:default:default)
echo "Default SA can access secrets: $default_perms (should be 'yes')"

if [ $admin_count -gt 2 ] && [ $root_pods -gt 0 ] && [ $priv_pods -gt 0 ] && [ "$default_perms" = "yes" ]; then
    echo ""
    echo "✅ Environment is properly configured with security vulnerabilities"
    echo "🚀 Ready to start A02 lab!"
else
    echo ""
    echo "❌ Environment setup incomplete. Please run setup commands again."
fi
EOF

chmod +x check-vulnerable-state.sh
./check-vulnerable-state.sh
```

---

## 🧹 Cleanup Commands

**Run these after completing the lab to remove the test environment:**

```bash
# Remove vulnerable RBAC bindings
kubectl delete clusterrolebinding dangerous-dev-binding dangerous-qa-binding default-app-binding

# Remove insecure applications
kubectl delete deployment insecure-app privileged-app --namespace=default
kubectl delete deployment test-app --namespace=development
kubectl delete deployment staging-app --namespace=qa-testing  
kubectl delete deployment critical-app --namespace=production

# Remove test secrets and configmaps
kubectl delete secret financial-secrets
kubectl delete configmap sensitive-config
kubectl delete rolebinding default-secret-access

# Remove test namespaces (optional - keep if you want to reuse)
kubectl delete namespace development qa-testing production shared-tools

# Remove any test service accounts
kubectl delete serviceaccount default-app-sa

# Clean up any CSRs from the lab
kubectl get csr | grep -E "dev-user|qa-user|ops-user" | awk '{print $1}' | xargs kubectl delete csr

echo "✅ A02 environment cleanup complete"
```

---

## 🔧 Troubleshooting

### Common Setup Issues

**Issue:** "cluster-admin clusterrole not found"
```bash
# Solution: Verify RBAC is enabled
kubectl auth can-i '*' '*' --all-namespaces
kubectl get clusterroles | grep cluster-admin
```

**Issue:** "jq command not found"
```bash
# Install jq for JSON processing
# Ubuntu/Debian:
sudo apt-get install jq

# CentOS/RHEL:
sudo yum install jq

# macOS:
brew install jq
```

**Issue:** "Cannot create clusterrolebinding"
```bash
# Ensure you have cluster-admin access
kubectl auth can-i create clusterrolebindings
```

### Verification Commands
```bash
# Check cluster RBAC status
kubectl get clusterroles | wc -l
kubectl get clusterrolebindings | wc -l

# Verify test deployments
kubectl get deployments --all-namespaces

# Check if secrets were created
kubectl get secrets
kubectl get configmaps
```

---

## 📝 Notes for Lab Facilitators

### Setup Time: ~5 minutes
### Prerequisites:
- Kubernetes cluster with RBAC enabled
- kubectl with cluster-admin permissions
- jq installed for JSON processing
- openssl for certificate operations

### Learning Objectives Alignment:
This setup creates the exact security vulnerabilities that the A02 lab is designed to fix, providing realistic context for learning enterprise RBAC management.

**🎯 Ready to start A02 - RBAC Security Lockdown!**
