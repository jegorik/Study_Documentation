# T08 - Performance Degradation Hunt

> **Scenario:** Global SaaS platform experiencing cascading performance issues  
> **Difficulty:** Advanced  
> **Estimated Time:** 30-35 minutes  
> **Exam Relevance:** High - Multi-component troubleshooting, performance analysis

---

## 🚨 Crisis Scenario

**Company:** CloudScale Analytics - Global SaaS platform serving 2M+ users  
**Situation:** Performance degradation affecting multiple services, customer complaints escalating  
**Business Impact:** $75,000/hour revenue loss, SLA violations triggering penalties  
**Time Pressure:** Performance must be restored within 35 minutes before triggering automatic failover

Your mission-critical Kubernetes cluster is experiencing mysterious performance degradation. Multiple microservices are running slow, some pods are failing health checks, and customer-facing APIs are timing out. The monitoring dashboard shows various warning signs, but the root cause isn't immediately obvious.

You need to systematically hunt down the performance bottlenecks and restore the cluster to optimal performance before customer churn becomes critical.

---

## 🎯 Learning Objectives

- **Performance Analysis:** Identify resource bottlenecks in multi-tier applications
- **Systematic Debugging:** Use methodical approach to isolate performance issues
- **Resource Management:** Understand CPU, memory, and I/O constraints
- **Monitoring Integration:** Leverage metrics and logs for performance investigation
- **Production Recovery:** Apply performance fixes in live environment

---

## 📋 Pre-Lab Setup

### Environment Requirements
- Kubernetes cluster (v1.26+)
- kubectl configured and functional
- Metrics server installed
- Basic monitoring tools (optional)

### Initial Cluster State
```bash
# The cluster is running a simulated e-commerce platform
# with multiple microservices experiencing performance issues
```

---

## 🔧 Mission Tasks

### **Task 1: Initial Performance Assessment** ⏱️ *6-8 minutes*
**Difficulty:** Intermediate  
**Objective:** Establish baseline performance metrics and identify obvious bottlenecks

#### Your Mission:
The platform's API gateway is reporting 5-second response times (normally <200ms). Start your investigation by gathering cluster-wide performance data.

#### Step-by-Step Instructions:

1. **Check overall cluster health:**
```bash
# Examine cluster nodes
kubectl get nodes -o wide
kubectl describe nodes

# Check node resource usage
kubectl top nodes

# Look for any obvious node issues
kubectl get events --sort-by='.lastTimestamp' | tail -20
```

2. **Analyze pod performance across namespaces:**
```bash
# Get resource usage for all pods
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# Check for pods with high resource consumption
kubectl get pods --all-namespaces -o wide
```

3. **Identify problematic workloads:**
```bash
# Look for pods in non-running states
kubectl get pods --all-namespaces --field-selector=status.phase!=Running

# Check for recent pod restarts
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[0].restartCount}{"\n"}{end}' | sort -k2 -nr
```

#### Expected Outcome:
- Baseline performance metrics collected
- High-resource consuming pods identified
- Any obviously failing components discovered

---

### **Task 2: Deep Dive Resource Analysis** ⏱️ *8-10 minutes*
**Difficulty:** Advanced  
**Objective:** Analyze resource constraints and bottlenecks affecting specific services

#### Your Mission:
The initial assessment revealed some concerning patterns. Now dive deeper into the specific services showing performance issues.

#### Step-by-Step Instructions:

1. **Examine the ecommerce namespace (primary business application):**
```bash
# Focus on the main application namespace
kubectl get pods -n ecommerce -o wide
kubectl top pods -n ecommerce

# Check resource requests and limits
kubectl describe pods -n ecommerce | grep -A5 -B5 -i "requests\|limits"

# Look for resource pressure indicators
kubectl get events -n ecommerce --sort-by='.lastTimestamp'
```

2. **Analyze database performance bottlenecks:**
```bash
# Check database pods specifically
kubectl logs -n ecommerce deployment/postgres-db --tail=50
kubectl describe pod -n ecommerce -l app=postgres-db

# Look for database connection issues
kubectl get pods -n ecommerce -l tier=database -o yaml | grep -A10 -B10 resources
```

3. **Investigate API gateway performance:**
```bash
# API gateway is the entry point - check its health
kubectl logs -n ecommerce deployment/api-gateway --tail=100
kubectl describe deployment -n ecommerce api-gateway

# Check if API gateway is being throttled
kubectl get hpa -n ecommerce
kubectl describe hpa -n ecommerce api-gateway-hpa 2>/dev/null || echo "No HPA found"
```

4. **Check for network-related performance issues:**
```bash
# Look at service configurations
kubectl get svc -n ecommerce -o wide
kubectl describe svc -n ecommerce api-gateway-service

# Check endpoints health
kubectl get endpoints -n ecommerce
```

#### Expected Outcome:
- Specific resource bottlenecks identified
- Database performance issues uncovered
- API gateway constraints analyzed
- Network connectivity issues ruled out

---

### **Task 3: Node-Level Performance Investigation** ⏱️ *6-8 minutes*
**Difficulty:** Advanced  
**Objective:** Identify node-level constraints that might be causing pod performance issues

#### Your Mission:
Your pod-level analysis revealed resource pressure, but you need to understand if this is due to node-level constraints, scheduling issues, or resource allocation problems.

#### Step-by-Step Instructions:

1. **Deep-dive node resource analysis:**
```bash
# Get detailed node resource information
kubectl describe nodes | grep -A10 -B5 "Allocated resources"
kubectl describe nodes | grep -A15 "Non-terminated pods"

# Check for node pressure conditions
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="MemoryPressure")].status}{"\t"}{.status.conditions[?(@.type=="DiskPressure")].status}{"\t"}{.status.conditions[?(@.type=="PIDPressure")].status}{"\n"}{end}'
```

2. **Analyze pod scheduling patterns:**
```bash
# Check if pods are unevenly distributed
kubectl get pods --all-namespaces -o wide | awk '{print $8}' | sort | uniq -c

# Look for scheduling failures
kubectl get events --all-namespaces | grep -i "failed.*schedule\|insufficient"

# Check for tainted nodes
kubectl describe nodes | grep -i taint
```

3. **Investigate disk and I/O performance:**
```bash
# Check for disk pressure
kubectl describe nodes | grep -A5 -B5 -i "disk\|storage"

# Look for pods with high I/O requirements
kubectl get pods --all-namespaces -o yaml | grep -A5 -B5 -i "volume"
```

4. **Check for resource quota constraints:**
```bash
# Look for namespace resource quotas
kubectl get resourcequota --all-namespaces
kubectl describe resourcequota -n ecommerce 2>/dev/null || echo "No resource quota in ecommerce namespace"

# Check limit ranges
kubectl get limitrange --all-namespaces
kubectl describe limitrange -n ecommerce 2>/dev/null || echo "No limit range in ecommerce namespace"
```

#### Expected Outcome:
- Node-level resource constraints identified
- Pod scheduling issues uncovered
- I/O bottlenecks discovered
- Resource quota conflicts found

---

### **Task 4: Application-Level Performance Debugging** ⏱️ *6-8 minutes*
**Difficulty:** Advanced  
**Objective:** Dive into application logs and metrics to identify code-level performance issues

#### Your Mission:
You've identified infrastructure constraints, but the performance issues might also stem from application-level problems like inefficient queries, connection pool exhaustion, or memory leaks.

#### Step-by-Step Instructions:

1. **Analyze application logs for performance patterns:**
```bash
# Look for slow query patterns in the database
kubectl logs -n ecommerce deployment/postgres-db --tail=200 | grep -i "slow\|long\|timeout"

# Check API response times in gateway logs
kubectl logs -n ecommerce deployment/api-gateway --tail=200 | grep -E "[0-9]+ms|[0-9]+s"

# Look for connection pool issues
kubectl logs -n ecommerce deployment/user-service --tail=100 | grep -i "connection\|pool\|timeout"
```

2. **Investigate memory usage patterns:**
```bash
# Check for memory leaks or high allocation
kubectl exec -n ecommerce deployment/user-service -- ps aux
kubectl exec -n ecommerce deployment/api-gateway -- free -h

# Look for OOMKilled events
kubectl get events --all-namespaces | grep -i "oomkilled\|out.*memory"
```

3. **Analyze CPU usage patterns:**
```bash
# Check for CPU-intensive processes
kubectl exec -n ecommerce deployment/product-service -- top -bn1
kubectl exec -n ecommerce deployment/order-service -- top -bn1 | head -10

# Look for thread contention or high context switching
kubectl logs -n ecommerce deployment/product-service --tail=50 | grep -i "thread\|lock\|wait"
```

4. **Check external dependencies:**
```bash
# Look for external service timeouts
kubectl logs -n ecommerce deployment/payment-service --tail=100 | grep -i "external\|timeout\|503\|502"

# Check DNS resolution times
kubectl exec -n ecommerce deployment/api-gateway -- nslookup kubernetes.default.svc.cluster.local
kubectl exec -n ecommerce deployment/api-gateway -- dig +time=1 +tries=1 google.com
```

#### Expected Outcome:
- Application-level bottlenecks identified
- Memory leak or CPU spike patterns found
- External dependency issues discovered
- Database query performance problems uncovered

---

### **Task 5: Performance Optimization and Recovery** ⏱️ *8-10 minutes*
**Difficulty:** Expert  
**Objective:** Apply systematic fixes to restore cluster performance while maintaining availability

#### Your Mission:
Based on your investigation, implement targeted fixes to address the performance bottlenecks. Prioritize changes that will have the most immediate impact on customer experience.

#### Step-by-Step Instructions:

1. **Address immediate resource constraints:**
```bash
# Scale up resource-constrained deployments
kubectl scale deployment -n ecommerce api-gateway --replicas=5
kubectl scale deployment -n ecommerce user-service --replicas=3

# Update resource limits for critical services
kubectl patch deployment -n ecommerce postgres-db -p='{"spec":{"template":{"spec":{"containers":[{"name":"postgres","resources":{"limits":{"memory":"2Gi","cpu":"1000m"},"requests":{"memory":"1Gi","cpu":"500m"}}}]}}}}'
```

2. **Optimize database performance:**
```bash
# Restart database to clear connection issues
kubectl rollout restart deployment -n ecommerce postgres-db

# Configure connection pooling
kubectl patch deployment -n ecommerce postgres-db -p='{"spec":{"template":{"spec":{"containers":[{"name":"postgres","env":[{"name":"POSTGRES_MAX_CONNECTIONS","value":"200"}]}]}}}}'

# Wait for rollout to complete
kubectl rollout status deployment -n ecommerce postgres-db --timeout=120s
```

3. **Implement CPU and memory optimizations:**
```bash
# Apply resource requests to ensure proper scheduling
kubectl patch deployment -n ecommerce product-service -p='{"spec":{"template":{"spec":{"containers":[{"name":"product-service","resources":{"requests":{"memory":"256Mi","cpu":"200m"},"limits":{"memory":"512Mi","cpu":"500m"}}}]}}}}'

kubectl patch deployment -n ecommerce order-service -p='{"spec":{"template":{"spec":{"containers":[{"name":"order-service","resources":{"requests":{"memory":"256Mi","cpu":"200m"},"limits":{"memory":"512Mi","cpu":"500m"}}}]}}}}'
```

4. **Configure horizontal pod autoscaling:**
```bash
# Set up HPA for critical services
kubectl autoscale deployment -n ecommerce api-gateway --cpu-percent=70 --min=3 --max=10
kubectl autoscale deployment -n ecommerce user-service --cpu-percent=70 --min=2 --max=6
kubectl autoscale deployment -n ecommerce product-service --cpu-percent=70 --min=2 --max=5
```

5. **Verify performance recovery:**
```bash
# Check that all pods are running with new configurations
kubectl get pods -n ecommerce -o wide
kubectl top pods -n ecommerce

# Verify HPA is functioning
kubectl get hpa -n ecommerce

# Test API response time (simulate)
kubectl exec -n ecommerce deployment/api-gateway -- curl -w "Response time: %{time_total}s\n" -s -o /dev/null http://localhost:8080/health

# Check recent events for successful scaling
kubectl get events -n ecommerce --sort-by='.lastTimestamp' | tail -10
```

#### Expected Outcome:
- Critical resource constraints resolved
- Database performance optimized
- Auto-scaling configured for future load
- System performance restored to acceptable levels
- Recovery verified through metrics and testing

---

## 🧪 Troubleshooting Guide

### Common Issues and Solutions

#### **Issue: "No metrics available"**
```bash
# Install or verify metrics server
kubectl get pods -n kube-system | grep metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

#### **Issue: "Cannot exec into pods"**
```bash
# Check pod status and debug
kubectl get pods -n ecommerce -o wide
kubectl describe pod -n ecommerce <pod-name>
kubectl logs -n ecommerce <pod-name>
```

#### **Issue: "Resource patches failing"**
```bash
# Verify deployment exists and check syntax
kubectl get deployment -n ecommerce
kubectl explain deployment.spec.template.spec.containers.resources
```

#### **Issue: "HPA not scaling"**
```bash
# Check metrics server and resource requests
kubectl top pods -n ecommerce
kubectl describe hpa -n ecommerce
kubectl get events -n ecommerce | grep -i hpa
```

---

## 🎓 Knowledge Check

### Key Concepts Reinforced

1. **Performance Troubleshooting Methodology:**
   - Systematic approach: Cluster → Node → Pod → Application
   - Resource analysis across multiple layers
   - Impact assessment and prioritization

2. **Resource Management:**
   - CPU and memory pressure identification
   - Resource requests vs limits optimization
   - Horizontal Pod Autoscaling configuration

3. **Database Performance:**
   - Connection pool optimization
   - Resource allocation for stateful workloads
   - Database-specific troubleshooting

4. **Application Performance:**
   - Log analysis for performance patterns
   - Memory leak detection
   - External dependency impact assessment

### Exam Tips

- **Time Management:** Performance issues can be complex - allocate enough time for systematic investigation
- **Systematic Approach:** Don't jump to solutions; gather data first
- **Resource Commands:** Master `kubectl top`, `kubectl describe`, and resource patch operations
- **Monitoring Integration:** Use metrics and logs together for complete picture
- **Production Safety:** Always verify changes and monitor impact

---

## 🏆 Success Criteria

### Task Completion Checklist
- [ ] **Task 1:** Cluster performance baseline established and bottlenecks identified
- [ ] **Task 2:** Service-specific resource constraints analyzed
- [ ] **Task 3:** Node-level performance issues investigated
- [ ] **Task 4:** Application-level performance problems identified
- [ ] **Task 5:** Performance optimizations applied and recovery verified

### Performance Indicators
- [ ] All critical pods running with appropriate resources
- [ ] HPA configured and functional for key services
- [ ] Database performance optimized
- [ ] API response times improved
- [ ] System stability restored

### Time Benchmarks
- **Basic proficiency:** Complete in 40 minutes
- **Intermediate:** Complete in 35 minutes  
- **Advanced:** Complete in 30 minutes
- **Expert:** Complete in 25 minutes

---

## 🚀 Extensions & Real-World Applications

### Production Scenarios
- **Black Friday Traffic:** Handling unexpected load spikes
- **Memory Leak Hunt:** Long-running application degradation
- **Database Bottlenecks:** Query optimization and resource scaling
- **Microservices Cascade:** Troubleshooting interconnected service failures

### Advanced Techniques
- Custom metrics for application-specific performance
- Node resource optimization and tainting strategies
- Network performance analysis and optimization
- Storage I/O performance tuning

### Integration Points
- APM tools integration (Prometheus, Grafana)
- Log aggregation systems (ELK stack)
- Distributed tracing for microservices
- Capacity planning and forecasting

---

*💡 **Pro Tip:** Performance troubleshooting is often about finding the "weakest link" in your system. Use systematic elimination to isolate the bottleneck, then apply targeted fixes rather than broad resource increases.*

---

**🎯 Mastery Goal:** After completing this lab, you should be able to systematically diagnose and resolve complex performance issues in production Kubernetes environments, applying both infrastructure-level and application-level optimizations while maintaining system availability.
