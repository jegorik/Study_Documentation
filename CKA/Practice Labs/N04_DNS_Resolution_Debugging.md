# N04 - DNS Resolution Debugging

## 🎯 Lab Overview

**Scenario:** You're the platform engineer for **CloudSync Technologies**, a global file synchronization service. At 3:47 AM, your monitoring system triggers a critical alert: microservices across multiple environments are failing to communicate, causing data sync failures for 50,000+ enterprise customers. The support team reports intermittent "service unavailable" errors, but infrastructure metrics show all pods are healthy. Your investigation reveals this is a DNS resolution crisis affecting service discovery.

**Business Impact:**

- **$2.3M/hour** revenue loss from sync service downtime
- Enterprise SLA breaches triggering penalty clauses
- Customer data integrity at risk from partial synchronization failures

**Your Mission:** Diagnose and resolve the DNS resolution issues across the cluster before the 4-hour SLA window expires.

## 📋 Lab Details

- **Difficulty:** Intermediate
- **Category:** Services & Networking (20% of CKA Exam)
- **Time Estimate:** 20-25 minutes
- **Prerequisites:** Understanding of Kubernetes DNS, Service discovery, CoreDNS

## 🚨 Tasks Overview

### Task 1: Emergency DNS Health Assessment (4 minutes)

**Situation:** Multiple pod-to-service communication failures reported across namespaces.

**Your Actions:**

- Investigate DNS resolution from within pods
- Check CoreDNS pod status and configuration
- Validate DNS service endpoints and policies

### Task 2: Service Discovery Failure Analysis (5 minutes)

**Situation:** Microservices can't discover each other using service names.

**Your Actions:**

- Test service name resolution across namespaces
- Debug DNS query patterns and responses
- Validate service DNS records and FQDN resolution

### Task 3: CoreDNS Configuration Recovery (6 minutes)

**Situation:** CoreDNS configuration appears corrupted or misconfigured.

**Your Actions:**

- Analyze CoreDNS ConfigMap and identify issues
- Restore proper DNS forwarding and caching configuration
- Verify DNS server performance and response times

### Task 4: Cross-Namespace DNS Policies (5 minutes)

**Situation:** Some namespaces can resolve services while others cannot.

**Your Actions:**

- Investigate namespace-specific DNS resolution issues
- Check NetworkPolicies affecting DNS traffic
- Validate DNS search domains and service discovery patterns

### Task 5: DNS Performance Optimization & Monitoring (5 minutes)

**Situation:** DNS resolution is working but performance is degraded.

**Your Actions:**

- Optimize CoreDNS configuration for high-throughput scenarios
- Implement DNS monitoring and alerting
- Create DNS troubleshooting runbook for future incidents

---

## 🛠 Task 1: Emergency DNS Health Assessment

### Context

The CloudSync platform consists of multiple microservices that rely heavily on DNS-based service discovery. Customer reports indicate that file synchronization is failing due to inter-service communication issues.

### Instructions

**Step 1: Create the failing infrastructure:**

```bash
# Create namespaces for CloudSync platform
kubectl create namespace cloudsync-core
kubectl create namespace cloudsync-api
kubectl create namespace cloudsync-workers

# Deploy sample microservices
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sync-api
  namespace: cloudsync-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sync-api
  template:
    metadata:
      labels:
        app: sync-api
    spec:
      containers:
      - name: api
        image: nginx:1.21
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: sync-api-service
  namespace: cloudsync-api
spec:
  selector:
    app: sync-api
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: file-processor
  namespace: cloudsync-workers
spec:
  replicas: 3
  selector:
    matchLabels:
      app: file-processor
  template:
    metadata:
      labels:
        app: file-processor
    spec:
      containers:
      - name: processor
        image: busybox:1.35
        command: ['sh', '-c', 'while true; do sleep 30; done']
---
apiVersion: v1
kind: Service
metadata:
  name: file-processor-service
  namespace: cloudsync-workers
spec:
  selector:
    app: file-processor
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
EOF
```

**Step 2: Simulate DNS resolution issues:**

```bash
# Backup current CoreDNS config
kubectl get configmap coredns -n kube-system -o yaml > coredns-backup.yaml

# Introduce DNS configuration issues
kubectl patch configmap coredns -n kube-system --type merge -p='
{
  "data": {
    "Corefile": ".:53 {\n    errors\n    health {\n      lameduck 5s\n    }\n    ready\n    kubernetes cluster.local in-addr.arpa ip6.arpa {\n      pods insecure\n      fallthrough in-addr.arpa ip6.arpa\n      ttl 30\n    }\n    prometheus :9153\n    forward . 8.8.8.8 {\n      max_concurrent 1000\n    }\n    cache 30\n    loop\n    reload\n    loadbalance\n}"
  }
}'

# Restart CoreDNS to apply broken config
kubectl rollout restart deployment/coredns -n kube-system
```

**Step 3: Test DNS resolution from pods:**

```bash
# Get a test pod for DNS debugging
kubectl run dns-debug --image=busybox:1.35 --rm -it --restart=Never -- sh

# Inside the pod, test basic DNS resolution
nslookup kubernetes.default.svc.cluster.local
nslookup sync-api-service.cloudsync-api.svc.cluster.local
nslookup file-processor-service.cloudsync-workers.svc.cluster.local

# Test shorter service names
nslookup sync-api-service.cloudsync-api
nslookup file-processor-service.cloudsync-workers

# Check DNS configuration in pod
cat /etc/resolv.conf
exit
```

**Step 4: Check CoreDNS pod status:**

```bash
# Check CoreDNS pod health
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl describe pods -n kube-system -l k8s-app=kube-dns

# Check CoreDNS logs for errors
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50

# Verify CoreDNS service
kubectl get svc -n kube-system kube-dns
kubectl describe svc -n kube-system kube-dns
```

**Verification:**

- ✅ DNS queries from pods should fail or timeout
- ✅ CoreDNS logs should show configuration errors
- ✅ Service discovery between namespaces should be broken

---

## 🛠 Task 2: Service Discovery Failure Analysis

### Context

You need to understand the scope and nature of the DNS failures affecting the CloudSync platform.

### Instructions

**Step 1: Test comprehensive service discovery:**

```bash
# Create test pods in each namespace for debugging
kubectl run test-pod-api --image=busybox:1.35 -n cloudsync-api --restart=Never -- sleep 3600
kubectl run test-pod-workers --image=busybox:1.35 -n cloudsync-workers --restart=Never -- sleep 3600
kubectl run test-pod-core --image=busybox:1.35 -n cloudsync-core --restart=Never -- sleep 3600

# Test from API namespace
kubectl exec -n cloudsync-api test-pod-api -- nslookup sync-api-service
kubectl exec -n cloudsync-api test-pod-api -- nslookup file-processor-service.cloudsync-workers
kubectl exec -n cloudsync-api test-pod-api -- nslookup kubernetes.default

# Test from Workers namespace
kubectl exec -n cloudsync-workers test-pod-workers -- nslookup file-processor-service
kubectl exec -n cloudsync-workers test-pod-workers -- nslookup sync-api-service.cloudsync-api
kubectl exec -n cloudsync-workers test-pod-workers -- nslookup kubernetes.default
```

**Step 2: Analyze DNS query patterns:**

```bash
# Check what DNS servers pods are using
kubectl exec -n cloudsync-api test-pod-api -- cat /etc/resolv.conf
kubectl exec -n cloudsync-workers test-pod-workers -- cat /etc/resolv.conf

# Test direct DNS server queries
kubectl exec -n cloudsync-api test-pod-api -- nslookup kubernetes.default.svc.cluster.local 10.96.0.10

# Check if DNS service is accessible
kubectl exec -n cloudsync-api test-pod-api -- telnet 10.96.0.10 53
```

**Step 3: Validate service endpoints:**

```bash
# Check if services have valid endpoints
kubectl get endpoints -n cloudsync-api sync-api-service
kubectl get endpoints -n cloudsync-workers file-processor-service
kubectl describe endpoints -n cloudsync-api sync-api-service

# Verify service DNS records should exist
kubectl get svc --all-namespaces
kubectl get svc -n cloudsync-api sync-api-service -o yaml
```

**Step 4: Test FQDN patterns:**

```bash
# Test various FQDN patterns that should work
kubectl exec -n cloudsync-api test-pod-api -- nslookup sync-api-service.cloudsync-api.svc.cluster.local
kubectl exec -n cloudsync-api test-pod-api -- nslookup sync-api-service.cloudsync-api.svc
kubectl exec -n cloudsync-api test-pod-api -- nslookup sync-api-service.cloudsync-api

# Test cross-namespace FQDN resolution
kubectl exec -n cloudsync-workers test-pod-workers -- nslookup sync-api-service.cloudsync-api.svc.cluster.local
```

**Verification:**

- ✅ Service name resolution should fail across namespaces
- ✅ FQDN resolution should also fail
- ✅ Direct DNS server queries should show the issue

---

## 🛠 Task 3: CoreDNS Configuration Recovery

### Context

CoreDNS configuration has been corrupted and needs to be fixed to restore service discovery.

### Instructions

**Step 1: Analyze current CoreDNS configuration:**

```bash
# Check current CoreDNS ConfigMap
kubectl get configmap coredns -n kube-system -o yaml

# Look for configuration issues in the Corefile
kubectl get configmap coredns -n kube-system -o jsonpath='{.data.Corefile}'

# Compare with backup
cat coredns-backup.yaml
```

**Step 2: Identify and fix CoreDNS issues:**

```bash
# The current config has issues - fix them
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
EOF
```

**Step 3: Restart CoreDNS and verify functionality:**

```bash
# Restart CoreDNS deployment
kubectl rollout restart deployment/coredns -n kube-system

# Wait for rollout to complete
kubectl rollout status deployment/coredns -n kube-system

# Check CoreDNS pods are healthy
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Verify no errors in logs
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=20
```

**Step 4: Test DNS resolution recovery:**

```bash
# Test service resolution from test pods
kubectl exec -n cloudsync-api test-pod-api -- nslookup sync-api-service
kubectl exec -n cloudsync-api test-pod-api -- nslookup file-processor-service.cloudsync-workers
kubectl exec -n cloudsync-workers test-pod-workers -- nslookup sync-api-service.cloudsync-api

# Test FQDN resolution
kubectl exec -n cloudsync-api test-pod-api -- nslookup kubernetes.default.svc.cluster.local

# Test external DNS resolution
kubectl exec -n cloudsync-api test-pod-api -- nslookup google.com
```

**Verification:**

- ✅ CoreDNS pods should be running without errors
- ✅ Service name resolution should work within and across namespaces
- ✅ External DNS resolution should also work

---

## 🛠 Task 4: Cross-Namespace DNS Policies

### Context

Ensure DNS resolution works correctly across all namespaces and implement proper network policies if needed.

### Instructions

**Step 1: Test cross-namespace service discovery:**

```bash
# Test service discovery patterns across namespaces
kubectl exec -n cloudsync-core test-pod-core -- nslookup sync-api-service.cloudsync-api
kubectl exec -n cloudsync-core test-pod-core -- nslookup file-processor-service.cloudsync-workers

# Test service connectivity (not just DNS)
kubectl exec -n cloudsync-core test-pod-core -- wget -qO- --timeout=5 http://sync-api-service.cloudsync-api
kubectl exec -n cloudsync-api test-pod-api -- wget -qO- --timeout=5 http://file-processor-service.cloudsync-workers:8080
```

**Step 2: Check for NetworkPolicies affecting DNS:**

```bash
# Check for existing NetworkPolicies
kubectl get networkpolicies --all-namespaces

# Check if there are any policies that might block DNS traffic
kubectl describe networkpolicies --all-namespaces
```

**Step 3: Create a DNS-specific NetworkPolicy for testing:**

```bash
# Create a NetworkPolicy that ensures DNS traffic is allowed
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: cloudsync-api
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
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 8080
EOF
```

**Step 4: Validate DNS search domains:**

```bash
# Check DNS search domains in pods
kubectl exec -n cloudsync-api test-pod-api -- cat /etc/resolv.conf
kubectl exec -n cloudsync-workers test-pod-workers -- cat /etc/resolv.conf

# Test short name resolution within namespace
kubectl exec -n cloudsync-api test-pod-api -- nslookup sync-api-service

# Test if search domains work correctly
kubectl exec -n cloudsync-workers test-pod-workers -- nslookup file-processor-service
```

**Verification:**

- ✅ DNS resolution should work across all namespaces
- ✅ NetworkPolicies should not interfere with DNS traffic
- ✅ Search domains should allow short service names within namespaces

---

## 🛠 Task 5: DNS Performance Optimization & Monitoring

### Context

Optimize CoreDNS for the high-throughput CloudSync platform and implement monitoring.

### Instructions

**Step 1: Optimize CoreDNS configuration for performance:**

```bash
# Apply optimized CoreDNS configuration
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
           prefer_udp
        }
        cache 60 {
           success 9984 30
           denial 9984 5
        }
        loop
        reload
        loadbalance round_robin
    }
EOF

# Restart to apply optimizations
kubectl rollout restart deployment/coredns -n kube-system
kubectl rollout status deployment/coredns -n kube-system
```

**Step 2: Scale CoreDNS for high availability:**

```bash
# Scale CoreDNS deployment for better performance
kubectl scale deployment coredns --replicas=3 -n kube-system

# Verify scaling
kubectl get deployment coredns -n kube-system
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

**Step 3: Create DNS monitoring tools:**

```bash
# Create a DNS monitoring script
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: dns-monitor
  namespace: kube-system
data:
  monitor.sh: |
    #!/bin/bash
    while true; do
      echo "=== DNS Health Check $(date) ==="
      
      # Test internal service resolution
      time nslookup kubernetes.default.svc.cluster.local
      
      # Test external resolution
      time nslookup google.com
      
      # Check CoreDNS metrics
      wget -qO- http://localhost:9153/metrics | grep coredns_dns_requests_total
      
      echo "=== End Check ==="
      sleep 60
    done
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dns-monitor
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dns-monitor
  template:
    metadata:
      labels:
        app: dns-monitor
    spec:
      containers:
      - name: monitor
        image: busybox:1.35
        command: ['sh', '/scripts/monitor.sh']
        volumeMounts:
        - name: scripts
          mountPath: /scripts
      volumes:
      - name: scripts
        configMap:
          name: dns-monitor
          defaultMode: 0755
EOF
```

**Step 4: Performance testing and validation:**

```bash
# Run DNS performance tests
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: dns-perf-test
  namespace: default
spec:
  containers:
  - name: test
    image: busybox:1.35
    command: ['sh', '-c']
    args:
    - |
      echo "Starting DNS performance test..."
      for i in {1..100}; do
        time nslookup kubernetes.default.svc.cluster.local >/dev/null 2>&1
      done
      echo "DNS performance test completed"
      sleep 3600
  restartPolicy: Never
EOF

# Monitor the test
kubectl logs dns-perf-test -f

# Check CoreDNS metrics
kubectl port-forward -n kube-system svc/kube-dns 9153:9153 &
curl http://localhost:9153/metrics | grep coredns_dns_requests_total
```

**Step 5: Create DNS troubleshooting runbook:**

```bash
# Create troubleshooting documentation
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: dns-troubleshooting-runbook
  namespace: kube-system
data:
  runbook.md: |
    # DNS Troubleshooting Runbook for CloudSync Platform
    
    ## Quick Health Checks
    1. **Check CoreDNS pod status**: kubectl get pods -n kube-system -l k8s-app=kube-dns
    2. **Test DNS from pod**: kubectl run dns-test --image=busybox --rm -it -- nslookup kubernetes.default
    3. **Check CoreDNS logs**: kubectl logs -n kube-system -l k8s-app=kube-dns
    
    ## Common Issues
    
    ### Issue: Service names not resolving
    - Check service exists: kubectl get svc -n <namespace>
    - Verify endpoints: kubectl get endpoints -n <namespace>
    - Test FQDN: nslookup service.namespace.svc.cluster.local
    
    ### Issue: Cross-namespace resolution fails
    - Check search domains: cat /etc/resolv.conf
    - Test FQDN resolution
    - Verify NetworkPolicies don't block DNS
    
    ### Issue: External DNS fails
    - Check CoreDNS forward configuration
    - Verify upstream DNS servers
    - Test: nslookup google.com
    
    ## Performance Issues
    - Scale CoreDNS: kubectl scale deployment coredns --replicas=3 -n kube-system
    - Check cache hit ratio in metrics
    - Adjust cache TTL settings
    
    ## Emergency Recovery
    1. Restore CoreDNS config: kubectl apply -f coredns-backup.yaml
    2. Restart CoreDNS: kubectl rollout restart deployment/coredns -n kube-system
    3. Verify resolution: Run test pods and check DNS
EOF

# Display the runbook
kubectl get configmap dns-troubleshooting-runbook -n kube-system -o jsonpath='{.data.runbook\.md}'
```

**Verification:**

- ✅ CoreDNS should be running with 3 replicas
- ✅ DNS resolution should be fast and reliable
- ✅ Monitoring tools should be collecting DNS metrics

---

## 🧪 Troubleshooting Guide

### Common Issues and Solutions

**Issue 1: DNS queries timeout or fail:**

```bash
# Check CoreDNS pod status
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Look for configuration errors
kubectl logs -n kube-system -l k8s-app=kube-dns

# Verify CoreDNS service
kubectl get svc -n kube-system kube-dns
```

**Issue 2: Service discovery works sporadically:**

```bash
# Check CoreDNS configuration
kubectl get configmap coredns -n kube-system -o yaml

# Verify search domains in pods
kubectl exec <pod-name> -- cat /etc/resolv.conf

# Test DNS caching behavior
kubectl exec <pod-name> -- nslookup <service> && kubectl exec <pod-name> -- nslookup <service>
```

**Issue 3: Cross-namespace resolution fails:**

```bash
# Test FQDN resolution
kubectl exec <pod> -- nslookup service.namespace.svc.cluster.local

# Check NetworkPolicies
kubectl get networkpolicies --all-namespaces

# Verify service endpoints exist
kubectl get endpoints -n <namespace> <service>
```

### Performance Optimization Checklist

- ✅ CoreDNS cache TTL optimized (60 seconds)
- ✅ Multiple CoreDNS replicas for load distribution
- ✅ Proper cache success/denial ratios configured
- ✅ Round-robin load balancing enabled
- ✅ DNS monitoring and alerting in place

---

## 📚 Knowledge Check

### Key Concepts Tested

1. **DNS Resolution Patterns**
   - Short names vs FQDN resolution
   - Search domain behavior
   - Cross-namespace service discovery

2. **CoreDNS Configuration**
   - Corefile syntax and plugins
   - Forward configuration and upstream servers
   - Cache configuration and performance tuning

3. **Kubernetes DNS Architecture**
   - DNS service discovery mechanism
   - Pod DNS configuration (/etc/resolv.conf)
   - Service DNS record creation

4. **Troubleshooting Methodology**
   - Systematic DNS issue diagnosis
   - Pod-level vs cluster-level testing
   - Configuration validation and recovery

### CKA Exam Relevance

- **Service Discovery:** Understanding how pods find services
- **DNS Configuration:** Managing CoreDNS in production clusters
- **Network Troubleshooting:** Diagnosing connectivity issues
- **Performance Optimization:** Scaling and tuning DNS infrastructure

### Time Benchmarks

- **Beginner:** 30-35 minutes
- **Intermediate:** 20-25 minutes  
- **Advanced:** 15-20 minutes

---

## 🎯 Success Criteria

### Task Completion Checklist

- [ ] **Task 1:** DNS health assessment completed with issues identified
- [ ] **Task 2:** Service discovery failure patterns documented and tested
- [ ] **Task 3:** CoreDNS configuration fixed and DNS resolution restored
- [ ] **Task 4:** Cross-namespace DNS policies validated and optimized
- [ ] **Task 5:** DNS monitoring implemented with performance optimization

### Production Readiness Validation

- [ ] All pods can resolve service names within their namespace
- [ ] Cross-namespace service discovery works via FQDN
- [ ] External DNS resolution functions properly
- [ ] CoreDNS performance metrics are within acceptable ranges
- [ ] DNS troubleshooting runbook is available for operations team

### Business Impact Resolution

- [ ] **Service Discovery Restored:** Microservices can communicate reliably
- [ ] **Data Sync Recovery:** File synchronization service is operational
- [ ] **SLA Compliance:** 4-hour resolution window met
- [ ] **Customer Impact Minimized:** Enterprise customers can resume normal operations

**🏆 Lab Complete!** You've successfully diagnosed and resolved a complex DNS resolution crisis, restoring service discovery for a critical enterprise platform.

---

## 🔄 Cleanup

```bash
# Remove test resources
kubectl delete pod dns-debug --ignore-not-found
kubectl delete pod test-pod-api -n cloudsync-api --ignore-not-found
kubectl delete pod test-pod-workers -n cloudsync-workers --ignore-not-found
kubectl delete pod test-pod-core -n cloudsync-core --ignore-not-found
kubectl delete pod dns-perf-test --ignore-not-found

# Remove monitoring tools
kubectl delete deployment dns-monitor -n kube-system
kubectl delete configmap dns-monitor -n kube-system
kubectl delete configmap dns-troubleshooting-runbook -n kube-system

# Remove test namespaces and resources
kubectl delete namespace cloudsync-core cloudsync-api cloudsync-workers

# Restore original CoreDNS configuration if needed
# kubectl apply -f coredns-backup.yaml
# kubectl rollout restart deployment/coredns -n kube-system

# Clean up backup files
rm -f coredns-backup.yaml

echo "DNS Resolution Debugging lab cleanup completed!"
```
