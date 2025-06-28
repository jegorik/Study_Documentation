# T07 - Performance Bottleneck Investigation

> **Difficulty:** Expert  
> **Category:** Troubleshooting (30% Exam Weight)  
> **Duration:** 45-50 minutes  
> **Prerequisites:** T01-T06, A01-A03, N01-N03, W01-W03, S01-S03  

## 📋 Lab Overview

### Scenario

You're the Senior Site Reliability Engineer at **TechFlow Industries**, a high-traffic SaaS platform serving 10 million users globally. The platform has been experiencing severe performance degradation over the past 48 hours:

- **User-reported issues:** 300% increase in page load times
- **Business impact:** $50,000/hour revenue loss during peak hours
- **Customer complaints:** Premium clients threatening contract cancellation
- **SLA breach:** 99.9% uptime SLA violated for first time in 2 years

The CEO has escalated this as a **P0 incident**. Multiple teams have investigated but the root cause remains elusive. As the Kubernetes expert, you're the final escalation point to identify and resolve the performance bottlenecks before the next business day.

### Crisis Situation

```text
🚨 CRITICAL ALERT DASHBOARD 🚨
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Response Time:     2.8s avg (Target: <500ms)     🔴 CRITICAL
💾 Memory Usage:      87% cluster-wide              🟠 HIGH  
🔄 CPU Utilization:   92% on worker nodes           🔴 CRITICAL
🌐 Network I/O:       Intermittent timeouts         🟠 HIGH
🗄️ Storage IOPS:     45% higher than baseline      🟠 HIGH
📈 Pod Restarts:      +400% in last 24h            🔴 CRITICAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Learning Objectives

- Master systematic performance investigation methodology
- Identify resource bottlenecks across multiple cluster components
- Analyze complex inter-service dependencies and bottlenecks
- Implement real-time performance monitoring and alerting
- Execute performance optimization under extreme time pressure
- Handle multi-layer infrastructure performance issues

### Real-World Context

This scenario mirrors incidents at companies like:

- **Netflix:** Global streaming performance degradation
- **Shopify:** Black Friday traffic surge handling
- **Slack:** Workspace performance optimization
- **GitHub:** Repository access performance issues

---

## 🛠️ Lab Environment Setup

### Cluster State (Performance Degraded)

```bash
# Cluster overview (DEGRADED STATE)
kubectl cluster-info
kubectl get nodes -o wide
kubectl top nodes
kubectl top pods --all-namespaces --sort-by=cpu

# Expected symptoms:
# - High CPU/memory usage across nodes
# - Multiple pods in CrashLoopBackOff
# - Slow kubectl response times
# - Network connectivity issues
```

### Application Architecture

```text
┌─────────────────────────────────────────────────────┐
│                 TECHFLOW PLATFORM                   │
├─────────────────────────────────────────────────────┤
│  🌐 Frontend (React SPA)                            │
│  ├── Load Balancer: 10M requests/day                │
│  └── CDN: Static assets                             │
├─────────────────────────────────────────────────────┤
│  🔗 API Gateway (Kong/Nginx)                        │
│  ├── Rate Limiting: 1000 req/min/user               │
│  ├── Authentication: JWT validation                 │
│  └── Routing: 12 microservices                      │
├─────────────────────────────────────────────────────┤
│  🏗️ Microservices Layer                             │
│  ├── user-service: User management                  │
│  ├── auth-service: Authentication                   │
│  ├── payment-service: Transactions                  │
│  ├── notification-service: Email/SMS                │
│  ├── analytics-service: Data processing             │
│  └── search-service: Elasticsearch                  │
├─────────────────────────────────────────────────────┤
│  💾 Data Layer                                      │
│  ├── PostgreSQL: Primary database                   │
│  ├── Redis: Caching & sessions                      │
│  ├── Elasticsearch: Search & analytics              │
│  └── MinIO: Object storage                          │
└─────────────────────────────────────────────────────┘
```

### Key Components Provided

- Production workloads experiencing performance issues
- Monitoring stack (Prometheus, Grafana, Jaeger)
- Multiple microservices with complex dependencies
- High-traffic load generators
- Performance testing tools

---

## 🎯 Tasks & Challenges

### Task 1: Rapid System Assessment (8 minutes)

**Objective:** Quickly identify the scope and severity of performance issues

#### Challenge Description

Multiple symptoms are manifesting simultaneously, making it difficult to identify the primary bottleneck. Previous investigations by different teams have focused on individual components without a holistic view.

#### Your Mission

1. **Cluster Health Overview**

   ```bash
   # Get immediate cluster health snapshot
   kubectl get componentstatuses
   kubectl get nodes --show-labels
   kubectl describe nodes | grep -E "(Pressure|Condition)"
   
   # Check for any node-level issues
   kubectl get events --sort-by='.lastTimestamp' | tail -20
   ```

2. **Resource Utilization Analysis**

   ```bash
   # CPU and Memory pressure points
   kubectl top nodes --sort-by=cpu
   kubectl top pods --all-namespaces --sort-by=cpu | head -20
   kubectl top pods --all-namespaces --sort-by=memory | head -20
   
   # Identify resource-constrained namespaces
   for ns in $(kubectl get ns -o name | cut -d/ -f2); do
     echo "=== Namespace: $ns ==="
     kubectl top pods -n $ns --sort-by=cpu 2>/dev/null | head -5
   done
   ```

3. **Critical Pod Status Assessment**

   ```bash
   # Find problematic pods
   kubectl get pods --all-namespaces --field-selector=status.phase!=Running
   kubectl get pods --all-namespaces -o wide | grep -E "(Error|CrashLoop|Pending)"
   
   # Check restart patterns
   kubectl get pods --all-namespaces -o custom-columns=\
   "NAMESPACE:.metadata.namespace,NAME:.metadata.name,RESTARTS:.status.containerStatuses[*].restartCount" \
   | sort -k3 -nr | head -10
   ```

4. **Service Connectivity Test**

   ```bash
   # Test inter-service communication
   kubectl run debug-pod --rm -i --tty --image=nicolaka/netshoot -- /bin/bash
   
   # Inside debug pod, test key services:
   # curl -w "@curl-format.txt" http://user-service.production.svc.cluster.local/health
   # curl -w "@curl-format.txt" http://payment-service.production.svc.cluster.local/health
   # nslookup user-service.production.svc.cluster.local
   ```

5. **Initial Metrics Collection**

   ```bash
   # Gather baseline metrics for analysis
   kubectl get hpa --all-namespaces
   kubectl get pdb --all-namespaces
   kubectl describe limitrange --all-namespaces
   kubectl describe resourcequota --all-namespaces
   ```

#### Success Criteria

- [ ] Complete cluster health assessment documented
- [ ] Top 10 resource-consuming pods identified
- [ ] Pod restart patterns analyzed
- [ ] Service connectivity status mapped
- [ ] Initial hypothesis formed about bottleneck location

#### 🔍 Investigation Notes Template

```markdown
## Initial Assessment Findings

### Node Health
- Total Nodes: X healthy, Y degraded
- Resource Pressure: CPU/Memory/Storage alerts
- Node Conditions: Ready/NotReady status

### Pod Analysis  
- Total Pods: X running, Y failed, Z pending
- Restart Leaders: [pod-name: restart-count]
- Resource Hogs: [pod-name: cpu%, memory%]

### Service Health
- Connectivity Issues: [service-name: status]
- Response Times: [service-name: latency]
- Error Rates: [service-name: error%]

### Initial Hypothesis
Primary bottleneck likely in: [Component/Layer]
Secondary issues: [List of contributing factors]
```

---

### Task 2: Deep Dive Resource Investigation (12 minutes)

**Objective:** Identify specific resource bottlenecks and their cascading effects

#### Challenge Description

Initial assessment shows multiple resource pressure points. You need to determine which bottleneck is the primary cause and which are secondary effects. Resource limits, requests, and actual usage patterns need detailed analysis.

#### Your Mission
1. **Detailed Node Resource Analysis**

   ```bash
   # Deep dive into node resource allocation
   kubectl describe nodes | grep -A 5 -B 5 "Allocated resources"
   
   # Check for node pressure conditions
   kubectl get nodes -o custom-columns=\
   "NAME:.metadata.name,READY:.status.conditions[?(@.type=='Ready')].status,\
   CPU-PRESSURE:.status.conditions[?(@.type=='PIDPressure')].status,\
   MEMORY-PRESSURE:.status.conditions[?(@.type=='MemoryPressure')].status,\
   DISK-PRESSURE:.status.conditions[?(@.type=='DiskPressure')].status"
   
   # Node capacity vs allocation
   kubectl get nodes -o=custom-columns="NAME:.metadata.name,\
   CPU-ALLOCATABLE:.status.allocatable.cpu,\
   MEMORY-ALLOCATABLE:.status.allocatable.memory"
   ```

2. **Pod Resource Requests vs Limits Analysis**

   ```bash
   # Analyze resource configuration mismatches
   kubectl get pods --all-namespaces -o=custom-columns=\
   "NAMESPACE:.metadata.namespace,\
   NAME:.metadata.name,\
   CPU-REQUEST:.spec.containers[*].resources.requests.cpu,\
   CPU-LIMIT:.spec.containers[*].resources.limits.cpu,\
   MEMORY-REQUEST:.spec.containers[*].resources.requests.memory,\
   MEMORY-LIMIT:.spec.containers[*].resources.limits.memory"
   
   # Find pods without resource specifications
   kubectl get pods --all-namespaces -o json | jq -r \
   '.items[] | select(.spec.containers[].resources.requests == null) | 
   "\(.metadata.namespace)/\(.metadata.name)"'
   ```

3. **Memory and CPU Utilization Patterns**

   ```bash
   # Historical resource usage (if Prometheus available)
   # Query Prometheus for resource trends
   curl -G 'http://prometheus.monitoring.svc.cluster.local:9090/api/v1/query' \
   --data-urlencode 'query=rate(container_cpu_usage_seconds_total[5m]) * 100' \
   | jq '.data.result[] | {pod: .metric.pod, cpu_usage: .value[1]}'
   
   # Memory usage patterns
   curl -G 'http://prometheus.monitoring.svc.cluster.local:9090/api/v1/query' \
   --data-urlencode 'query=container_memory_usage_bytes / 1024 / 1024' \
   | jq '.data.result[] | {pod: .metric.pod, memory_mb: .value[1]}'
   ```

4. **Storage I/O Analysis**

   ```bash
   # Check PV usage and performance
   kubectl get pv -o custom-columns=\
   "NAME:.metadata.name,\
   CAPACITY:.spec.capacity.storage,\
   ACCESS_MODES:.spec.accessModes[*],\
   RECLAIM_POLICY:.spec.persistentVolumeReclaimPolicy,\
   STATUS:.status.phase"
   
   # PVC usage analysis
   kubectl get pvc --all-namespaces -o custom-columns=\
   "NAMESPACE:.metadata.namespace,\
   NAME:.metadata.name,\
   REQUEST:.spec.resources.requests.storage,\
   CAPACITY:.status.capacity.storage,\
   ACCESS_MODES:.spec.accessModes[*]"
   ```

5. **Network Resource Investigation**

   ```bash
   # Check network policies that might cause bottlenecks
   kubectl get netpol --all-namespaces
   kubectl describe netpol --all-namespaces | grep -A 10 -B 2 "Spec"
   
   # Service mesh performance (if Istio/Linkerd)
   kubectl get pods -n istio-system
   kubectl top pods -n istio-system
   
   # DNS performance test
   kubectl run dns-test --image=busybox:1.28 --rm -it -- nslookup kubernetes.default
   ```

6. **Horizontal Pod Autoscaler Analysis**

   ```bash
   # HPA behavior during high load
   kubectl get hpa --all-namespaces -o wide
   kubectl describe hpa --all-namespaces | grep -A 10 -B 2 "Events"
   
   # Check if HPA is scaling appropriately
   for hpa in $(kubectl get hpa --all-namespaces -o custom-columns=":metadata.namespace,:metadata.name" --no-headers); do
     ns=$(echo $hpa | cut -d' ' -f1)
     name=$(echo $hpa | cut -d' ' -f2)
     echo "=== HPA: $ns/$name ==="
     kubectl describe hpa $name -n $ns | grep -E "(Current|Target|Conditions)"
   done
   ```

#### Success Criteria

- [ ] Node resource allocation patterns identified
- [ ] Pod resource misconfigurations documented
- [ ] Memory/CPU usage trends analyzed
- [ ] Storage I/O bottlenecks assessed
- [ ] Network resource constraints evaluated
- [ ] HPA scaling behavior understood

#### 🔍 Resource Analysis Template

```markdown
## Resource Bottleneck Analysis

### Node-Level Issues
- CPU Pressure: [Node names and severity]
- Memory Pressure: [Node names and severity]
- Disk Pressure: [Node names and severity]
- Network Saturation: [Evidence and impact]

### Pod-Level Problems
- Over-allocated Pods: [Pod names and resource usage]
- Under-resourced Pods: [Pod names being throttled]
- Missing Resource Specs: [Pod names without limits/requests]

### Storage Bottlenecks
- High I/O Pods: [Pod names and I/O patterns]
- Storage Capacity Issues: [PV/PVC problems]
- Mount Performance: [Slow mount points]

### Scaling Issues
- HPA Thrashing: [HPA names with erratic scaling]
- Resource Requests vs Limits: [Mismatched configurations]
- Cluster Capacity: [Overall resource constraints]
```

---

### Task 3: Application-Level Performance Deep Dive (15 minutes)

**Objective:** Investigate application-specific bottlenecks, database performance, and service dependencies

#### Challenge Description

Resource analysis points to application-level issues. You need to investigate database connections, service dependencies, caching effectiveness, and application code performance patterns that might be causing the systemic slowdown.

#### Your Mission

1. **Database Performance Investigation**

   ```bash
   # PostgreSQL performance analysis
   kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
   SELECT query, calls, total_time, mean_time, rows
   FROM pg_stat_statements 
   ORDER BY total_time DESC 
   LIMIT 10;"
   
   # Check database connections
   kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
   SELECT count(*) as active_connections, 
          state, 
          client_addr 
   FROM pg_stat_activity 
   GROUP BY state, client_addr 
   ORDER BY active_connections DESC;"
   
   # Database locks analysis
   kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
   SELECT blocked_locks.pid AS blocked_pid,
          blocked_activity.usename AS blocked_user,
          blocking_locks.pid AS blocking_pid,
          blocking_activity.usename AS blocking_user,
          blocked_activity.query AS blocked_statement
   FROM pg_catalog.pg_locks blocked_locks
   JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
   JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
   JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
   WHERE NOT blocked_locks.granted;"
   ```

2. **Redis Cache Performance Analysis**

   ```bash
   # Redis performance metrics
   kubectl exec -n production deployment/redis -- redis-cli info stats
   kubectl exec -n production deployment/redis -- redis-cli info memory
   kubectl exec -n production deployment/redis -- redis-cli info clients
   
   # Cache hit ratio analysis
   kubectl exec -n production deployment/redis -- redis-cli eval "
   local hits = redis.call('get', 'cache_hits') or 0
   local misses = redis.call('get', 'cache_misses') or 0
   local total = hits + misses
   if total > 0 then
     return 'Hit ratio: ' .. (hits/total*100) .. '%'
   else
     return 'No cache statistics available'
   end" 0
   
   # Memory usage and eviction policies
   kubectl exec -n production deployment/redis -- redis-cli config get maxmemory-policy
   kubectl exec -n production deployment/redis -- redis-cli memory usage cache:*
   ```

3. **Microservice Dependency Analysis**

   ```bash
   # Service mesh traffic analysis (if using Istio)
   kubectl get virtualservices --all-namespaces
   kubectl get destinationrules --all-namespaces
   
   # Check service response times
   kubectl run perf-test --rm -i --tty --image=curlimages/curl -- sh
   
   # Inside the pod, test each service:
   services=("user-service" "auth-service" "payment-service" "notification-service")
   for svc in "${services[@]}"; do
     echo "Testing $svc..."
     curl -w "Time: %{time_total}s, Size: %{size_download} bytes\n" \
          -o /dev/null -s "http://$svc.production.svc.cluster.local/health"
   done
   ```

4. **Application Log Analysis**

   ```bash
   # Analyze application error patterns
   kubectl logs -n production deployment/user-service --tail=1000 | \
   grep -E "(ERROR|WARN|Exception)" | sort | uniq -c | sort -nr
   
   # Database connection errors
   kubectl logs -n production deployment/user-service --tail=1000 | \
   grep -i "connection" | tail -20
   
   # Memory and GC issues (Java applications)
   kubectl logs -n production deployment/payment-service --tail=1000 | \
   grep -E "(OutOfMemory|GC|heap)" | tail -10
   
   # Response time patterns
   kubectl logs -n production deployment/api-gateway --tail=1000 | \
   grep "response_time" | awk '{print $NF}' | sort -n | tail -20
   ```

5. **Thread and Connection Pool Analysis**

   ```bash
   # Check for thread pool exhaustion
   kubectl exec -n production deployment/user-service -- \
   curl -s http://localhost:8080/actuator/metrics/jvm.threads.live
   
   # Database connection pool status
   kubectl exec -n production deployment/user-service -- \
   curl -s http://localhost:8080/actuator/metrics/hikaricp.connections.active
   
   # HTTP client connection pools
   kubectl exec -n production deployment/user-service -- \
   curl -s http://localhost:8080/actuator/metrics/http.client.requests
   ```

6. **JVM Performance Analysis (Java Services)**

   ```bash
   # Heap usage analysis
   for deployment in user-service auth-service payment-service; do
     echo "=== $deployment JVM Metrics ==="
     kubectl exec -n production deployment/$deployment -- \
     curl -s http://localhost:8080/actuator/metrics/jvm.memory.used | jq '.measurements[0].value'
     
     kubectl exec -n production deployment/$deployment -- \
     curl -s http://localhost:8080/actuator/metrics/jvm.gc.pause | jq '.measurements'
   done
   ```

#### Success Criteria

- [ ] Database performance bottlenecks identified
- [ ] Cache effectiveness analyzed
- [ ] Service dependency latencies measured
- [ ] Application error patterns documented
- [ ] Connection pool status assessed
- [ ] JVM performance issues identified

#### 🔍 Application Performance Analysis

```markdown
## Application-Level Bottlenecks

### Database Performance
- Slow Queries: [Query patterns and execution times]
- Connection Issues: [Pool exhaustion, connection leaks]
- Lock Contention: [Blocking queries and deadlocks]
- Index Performance: [Missing or inefficient indexes]

### Cache Performance
- Hit Ratio: [Current percentage vs target]
- Memory Usage: [Current vs allocated]
- Eviction Rate: [Keys being evicted prematurely]
- Cache Warming: [Cold cache issues]

### Service Dependencies
- Response Times: [Service-to-service latency]
- Error Rates: [Inter-service communication failures]
- Circuit Breakers: [Services in open/half-open state]
- Retry Storms: [Cascading retry failures]

### Application Health
- Memory Leaks: [Heap usage trends]
- GC Pressure: [Garbage collection frequency/duration]
- Thread Exhaustion: [Thread pool utilization]
- Connection Pools: [Pool exhaustion patterns]
```

---

### Task 4: Network and Infrastructure Bottlenecks (10 minutes)

**Objective:** Investigate network latency, DNS issues, and infrastructure-level performance problems

#### Challenge Description

Application metrics suggest network-related bottlenecks. You need to investigate DNS resolution times, network policies affecting performance, ingress controller bottlenecks, and inter-node communication issues.

#### Your Mission

1. **DNS Performance Investigation**

   ```bash
   # DNS resolution time testing
   kubectl run dns-perf-test --rm -i --tty --image=tutum/dnsutils -- sh
   
   # Inside the container, test DNS performance:
   # dig @kube-dns.kube-system.svc.cluster.local user-service.production.svc.cluster.local
   # time nslookup user-service.production.svc.cluster.local
   # dig +trace user-service.production.svc.cluster.local
   
   # Check CoreDNS performance
   kubectl logs -n kube-system deployment/coredns --tail=100 | grep -E "(error|timeout|slow)"
   kubectl top pods -n kube-system -l k8s-app=kube-dns
   ```

2. **Network Policy Impact Assessment**

   ```bash
   # Check network policies affecting performance
   kubectl get netpol --all-namespaces -o yaml | grep -A 10 -B 10 "spec:"
   
   # Test network connectivity between services
   kubectl run netshoot --rm -i --tty --image nicolaka/netshoot -- sh
   
   # Network latency testing between services
   # traceroute user-service.production.svc.cluster.local
   # iperf3 -c payment-service.production.svc.cluster.local
   # curl -w "@curl-format.txt" http://auth-service.production.svc.cluster.local/health
   ```

3. **Ingress Controller Performance**

   ```bash
   # Nginx Ingress Controller metrics
   kubectl get pods -n ingress-nginx -o wide
   kubectl top pods -n ingress-nginx
   kubectl logs -n ingress-nginx deployment/nginx-ingress-controller --tail=200 | \
   grep -E "(error|timeout|upstream)"
   
   # Check ingress backend response times
   kubectl get ingress --all-namespaces -o wide
   kubectl describe ingress --all-namespaces | grep -A 5 -B 5 "backend"
   
   # Load balancer connection stats
   kubectl exec -n ingress-nginx deployment/nginx-ingress-controller -- \
   curl -s http://localhost:10254/nginx_status
   ```

4. **Inter-Node Network Performance**

   ```bash
   # Node-to-node network testing
   nodes=($(kubectl get nodes -o jsonpath='{.items[*].metadata.name}'))
   
   # Test network between nodes (run from privileged pod)
   kubectl run network-test --rm -i --tty --privileged --image nicolaka/netshoot -- sh
   
   # Bandwidth testing between nodes
   # iperf3 -s &  # On one node
   # iperf3 -c <node-ip>  # From another node
   
   # Check for network interface issues
   kubectl get nodes -o wide
   for node in "${nodes[@]}"; do
     echo "=== Node: $node ==="
     kubectl debug node/$node -it --image=nicolaka/netshoot -- \
     sh -c "ip addr show; ethtool eth0 | grep Speed"
   done
   ```

5. **CNI Plugin Performance**

   ```bash
   # Check CNI plugin status and performance
   kubectl get pods -n kube-system -l k8s-app=calico-node  # or flannel, weave
   kubectl logs -n kube-system daemonset/calico-node --tail=100 | grep -E "(error|warn)"
   
   # Network interface statistics
   kubectl run network-debug --rm -i --tty --privileged --image nicolaka/netshoot -- sh
   # cat /proc/net/dev
   # ss -tuln | grep :80
   # netstat -i
   ```

6. **Service Mesh Performance (if applicable)**

   ```bash
   # Istio/Envoy performance metrics
   kubectl get pods -n istio-system
   kubectl top pods -n istio-system
   
   # Envoy proxy stats
   kubectl exec -n production deployment/user-service -c istio-proxy -- \
   curl -s http://localhost:15000/stats | grep -E "(upstream|downstream|circuit_breaker)"
   
   # Service mesh overhead analysis
   kubectl exec -n production deployment/user-service -c istio-proxy -- \
   curl -s http://localhost:15000/stats/prometheus | grep envoy_cluster_upstream_rq_time
   ```

#### Success Criteria

- [ ] DNS resolution performance measured
- [ ] Network policy impact assessed
- [ ] Ingress controller bottlenecks identified
- [ ] Inter-node communication validated
- [ ] CNI plugin health verified
- [ ] Service mesh overhead quantified

#### 🔍 Network Performance Analysis

```markdown
## Network Infrastructure Bottlenecks

### DNS Performance
- Resolution Time: [Average DNS lookup time]
- CoreDNS Health: [CPU/memory usage, error rates]
- Cache Hit Ratio: [DNS cache effectiveness]
- External DNS: [Upstream resolver performance]

### Network Policies
- Policy Overhead: [Rule processing time]
- Blocked Connections: [Denied traffic patterns]
- Policy Conflicts: [Overlapping or conflicting rules]

### Ingress Performance
- Backend Response Time: [Upstream service latency]
- Connection Pooling: [Keep-alive effectiveness]
- SSL Termination: [Certificate processing overhead]
- Rate Limiting: [Throttling impact]

### Infrastructure Network
- Node-to-Node Latency: [Inter-node communication time]
- Bandwidth Utilization: [Network saturation levels]
- Packet Loss: [Network reliability issues]
- CNI Overhead: [Container networking performance]
```

---

### Task 5: Real-Time Performance Optimization (12 minutes)

**Objective:** Implement immediate fixes and optimizations to restore performance

#### Challenge Description

With bottlenecks identified, you need to implement rapid fixes while the system is under load. This requires careful prioritization of changes and real-time monitoring of their impact.

#### Your Mission

1. **Critical Resource Adjustments**

   ```bash
   # Immediate CPU/Memory limit adjustments for critical services
   kubectl patch deployment user-service -n production -p='
   {
     "spec": {
       "template": {
         "spec": {
           "containers": [{
             "name": "user-service",
             "resources": {
               "requests": {"cpu": "500m", "memory": "1Gi"},
               "limits": {"cpu": "2000m", "memory": "4Gi"}
             }
           }]
         }
       }
     }
   }'
   
   # Scale up critical services immediately
   kubectl scale deployment user-service -n production --replicas=10
   kubectl scale deployment payment-service -n production --replicas=8
   kubectl scale deployment auth-service -n production --replicas=6
   ```

2. **Database Performance Optimization**

   ```bash
   # Optimize PostgreSQL configuration
   kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
   -- Kill long-running queries
   SELECT pg_terminate_backend(pid) 
   FROM pg_stat_activity 
   WHERE state = 'active' 
     AND query_start < now() - interval '5 minutes'
     AND query NOT LIKE '%pg_stat_activity%';
   
   -- Analyze slow queries
   SELECT query, calls, total_time, mean_time 
   FROM pg_stat_statements 
   WHERE mean_time > 1000 
   ORDER BY total_time DESC;
   "
   
   # Add database connection pooling
   kubectl apply -f - << EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: pgbouncer
     namespace: production
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: pgbouncer
     template:
       metadata:
         labels:
           app: pgbouncer
       spec:
         containers:
         - name: pgbouncer
           image: pgbouncer/pgbouncer:latest
           env:
           - name: DATABASES_HOST
             value: "postgresql.production.svc.cluster.local"
           - name: DATABASES_PORT
             value: "5432"
           - name: POOL_MODE
             value: "transaction"
           - name: MAX_CLIENT_CONN
             value: "1000"
           - name: DEFAULT_POOL_SIZE
             value: "25"
           ports:
           - containerPort: 5432
   EOF
   ```

3. **Cache Optimization**

   ```bash
   # Scale Redis for better cache performance
   kubectl scale deployment redis -n production --replicas=3
   
   # Optimize Redis configuration
   kubectl exec -n production deployment/redis -- redis-cli config set maxmemory-policy allkeys-lru
   kubectl exec -n production deployment/redis -- redis-cli config set maxmemory 4gb
   
   # Add Redis Cluster for horizontal scaling
   kubectl apply -f - << EOF
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: redis-cluster-config
     namespace: production
   data:
     redis.conf: |
       cluster-enabled yes
       cluster-config-file nodes.conf
       cluster-node-timeout 5000
       appendonly yes
       maxmemory 2gb
       maxmemory-policy allkeys-lru
   EOF
   ```

4. **Network Performance Improvements**

   ```bash
   # Optimize CoreDNS for better performance
   kubectl patch configmap coredns -n kube-system -p='
   {
     "data": {
       "Corefile": ".:53 {\n    errors\n    health {\n       lameduck 5s\n    }\n    ready\n    kubernetes cluster.local in-addr.arpa ip6.arpa {\n       pods insecure\n       fallthrough in-addr.arpa ip6.arpa\n       ttl 30\n    }\n    prometheus :9153\n    forward . 8.8.8.8 8.8.4.4 {\n       max_concurrent 1000\n    }\n    cache 300\n    loop\n    reload\n    loadbalance\n}"
     }
   }'
   
   # Scale CoreDNS for better DNS performance
   kubectl scale deployment coredns -n kube-system --replicas=5
   
   # Optimize nginx ingress for high throughput
   kubectl patch configmap nginx-configuration -n ingress-nginx -p='
   {
     "data": {
       "worker-processes": "auto",
       "worker-connections": "65536",
       "keepalive-requests": "10000",
       "upstream-keepalive-connections": "320",
       "upstream-keepalive-requests": "10000"
     }
   }'
   ```

5. **Application-Level Optimizations**

   ```bash
   # Update HPA for more aggressive scaling
   kubectl patch hpa user-service-hpa -n production -p='
   {
     "spec": {
       "minReplicas": 5,
       "maxReplicas": 50,
       "targetCPUUtilizationPercentage": 60,
       "scaleUpPeriod": "30s",
       "scaleDownPeriod": "60s"
     }
   }'
   
   # Add pod disruption budgets to prevent cascading failures
   kubectl apply -f - << EOF
   apiVersion: policy/v1
   kind: PodDisruptionBudget
   metadata:
     name: user-service-pdb
     namespace: production
   spec:
     minAvailable: 3
     selector:
       matchLabels:
         app: user-service
   EOF
   
   # Configure readiness/liveness probes for faster failure detection
   kubectl patch deployment payment-service -n production -p='
   {
     "spec": {
       "template": {
         "spec": {
           "containers": [{
             "name": "payment-service",
             "readinessProbe": {
               "httpGet": {"path": "/health", "port": 8080},
               "initialDelaySeconds": 10,
               "periodSeconds": 5,
               "timeoutSeconds": 3
             },
             "livenessProbe": {
               "httpGet": {"path": "/health", "port": 8080},
               "initialDelaySeconds": 30,
               "periodSeconds": 10,
               "timeoutSeconds": 5
             }
           }]
         }
       }
     }
   }'
   ```

6. **Real-Time Impact Monitoring**

   ```bash
   # Monitor the impact of changes in real-time
   watch -n 5 'kubectl top nodes; echo "---"; kubectl top pods -n production --sort-by=cpu | head -10'
   
   # Check service response times after optimizations
   kubectl run performance-monitor --rm -i --tty --image=curlimages/curl -- sh
   # while true; do 
   #   curl -w "Time: %{time_total}s\n" -o /dev/null -s http://user-service.production.svc.cluster.local/health
   #   sleep 2
   # done
   
   # Monitor database performance improvements
   kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
   SELECT count(*) as active_connections, state 
   FROM pg_stat_activity 
   GROUP BY state;"
   ```

#### Success Criteria

- [ ] Critical services scaled and optimized
- [ ] Database performance improved
- [ ] Cache hit ratio increased
- [ ] Network latency reduced
- [ ] Application response times improved
- [ ] Real-time monitoring showing positive trends

#### 🔍 Optimization Impact Tracking

```markdown
## Performance Optimization Results

### Before Optimization
- Average Response Time: [Baseline measurement]
- CPU Utilization: [Pre-optimization levels]
- Memory Usage: [Pre-optimization levels]
- Error Rate: [Pre-optimization percentage]

### Immediate Changes Applied
- Resource Scaling: [Services scaled and new limits]
- Database Optimizations: [Connection pooling, query optimization]
- Cache Improvements: [Redis scaling and configuration]
- Network Optimizations: [DNS, ingress, CNI improvements]

### After Optimization
- Average Response Time: [Post-optimization measurement]
- CPU Utilization: [Post-optimization levels]
- Memory Usage: [Post-optimization levels]
- Error Rate: [Post-optimization percentage]

### Performance Gains
- Response Time Improvement: [Percentage improvement]
- Resource Efficiency: [Better resource utilization]
- Stability Metrics: [Reduced restart rates, fewer errors]
```

---

### Task 6: Long-term Performance Strategy & Prevention (8 minutes)

**Objective:** Implement monitoring, alerting, and preventive measures to avoid future performance crises

#### Challenge Description

With immediate performance restored, you need to establish long-term monitoring and alerting systems to prevent future incidents. This includes capacity planning, automated scaling policies, and comprehensive observability.

#### Your Mission

1. **Advanced Monitoring Setup**

   ```yaml
   # Deploy comprehensive monitoring stack
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: prometheus-rules
     namespace: monitoring
   data:
     performance-rules.yml: |
       groups:
       - name: performance.rules
         rules:
         - alert: HighCPUUsage
           expr: rate(container_cpu_usage_seconds_total[5m]) * 100 > 80
           for: 2m
           annotations:
             summary: "High CPU usage detected"
             description: "Pod {{ $labels.pod }} CPU usage above 80%"
         
         - alert: HighMemoryUsage
           expr: container_memory_usage_bytes / container_spec_memory_limit_bytes * 100 > 85
           for: 2m
           annotations:
             summary: "High memory usage detected"
             description: "Pod {{ $labels.pod }} memory usage above 85%"
         
         - alert: PodRestartRate
           expr: rate(kube_pod_container_status_restarts_total[15m]) > 0.1
           for: 5m
           annotations:
             summary: "High pod restart rate"
             description: "Pod {{ $labels.pod }} restarting frequently"
         
         - alert: DatabaseConnectionPoolExhaustion
           expr: hikaricp_connections_active / hikaricp_connections_max > 0.9
           for: 1m
           annotations:
             summary: "Database connection pool near exhaustion"
         
         - alert: SlowDatabaseQueries
           expr: pg_stat_statements_mean_time_ms > 1000
           for: 2m
           annotations:
             summary: "Slow database queries detected"
   ```

2. **Automated Scaling Policies**

   ```yaml
   # Enhanced HPA with custom metrics
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: user-service-advanced-hpa
     namespace: production
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: user-service
     minReplicas: 3
     maxReplicas: 100
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
     - type: Resource
       resource:
         name: memory
         target:
           type: Utilization
           averageUtilization: 80
     - type: Pods
       pods:
         metric:
           name: response_time_p95
         target:
           type: AverageValue
           averageValue: "500m"
     behavior:
       scaleDown:
         stabilizationWindowSeconds: 300
         policies:
         - type: Percent
           value: 10
           periodSeconds: 60
       scaleUp:
         stabilizationWindowSeconds: 60
         policies:
         - type: Percent
           value: 50
           periodSeconds: 30
         - type: Pods
           value: 5
           periodSeconds: 30
   ---
   # Vertical Pod Autoscaler for right-sizing
   apiVersion: autoscaling.k8s.io/v1
   kind: VerticalPodAutoscaler
   metadata:
     name: user-service-vpa
     namespace: production
   spec:
     targetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: user-service
     updatePolicy:
       updateMode: "Auto"
     resourcePolicy:
       containerPolicies:
       - containerName: user-service
         minAllowed:
           cpu: 100m
           memory: 256Mi
         maxAllowed:
           cpu: 4000m
           memory: 8Gi
   ```

3. **Capacity Planning Dashboard**

   ```yaml
   # Grafana dashboard configuration
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: capacity-planning-dashboard
     namespace: monitoring
   data:
     dashboard.json: |
       {
         "dashboard": {
           "title": "Capacity Planning & Performance Trends",
           "panels": [
             {
               "title": "Resource Utilization Trends",
               "targets": [
                 {
                   "expr": "avg(rate(container_cpu_usage_seconds_total[5m])) by (namespace)",
                   "legendFormat": "{{namespace}} CPU"
                 },
                 {
                   "expr": "avg(container_memory_usage_bytes) by (namespace) / 1024 / 1024 / 1024",
                   "legendFormat": "{{namespace}} Memory (GB)"
                 }
               ]
             },
             {
               "title": "Database Performance Metrics",
               "targets": [
                 {
                   "expr": "pg_stat_database_tup_returned + pg_stat_database_tup_fetched",
                   "legendFormat": "Database Reads/sec"
                 },
                 {
                   "expr": "pg_stat_database_tup_inserted + pg_stat_database_tup_updated + pg_stat_database_tup_deleted",
                   "legendFormat": "Database Writes/sec"
                 }
               ]
             }
           ]
         }
       }
   ```

4. **Performance Testing Automation**

   ```yaml
   # Automated load testing job
   apiVersion: batch/v1
   kind: CronJob
   metadata:
     name: performance-baseline-test
     namespace: testing
   spec:
     schedule: "0 2 * * *"  # Daily at 2 AM
     jobTemplate:
       spec:
         template:
           spec:
             containers:
             - name: k6-loadtest
               image: grafana/k6:latest
               command:
               - k6
               - run
               - --vus=100
               - --duration=10m
               - /scripts/performance-test.js
               volumeMounts:
               - name: test-scripts
                 mountPath: /scripts
               env:
               - name: TARGET_HOST
                 value: "https://api.techflow.com"
               - name: PROMETHEUS_URL
                 value: "http://prometheus.monitoring.svc.cluster.local:9090"
             volumes:
             - name: test-scripts
               configMap:
                 name: k6-test-scripts
             restartPolicy: OnFailure
   ```

5. **Incident Response Automation**

   ```yaml
   # Automated incident response
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: incident-response-playbook
     namespace: monitoring
   data:
     auto-scale-script.sh: |
       #!/bin/bash
       # Automated response to high CPU utilization
       if [ "$ALERT_NAME" = "HighCPUUsage" ]; then
         NAMESPACE=$(echo $ALERT_LABELS | jq -r '.namespace')
         DEPLOYMENT=$(echo $ALERT_LABELS | jq -r '.deployment')
         CURRENT_REPLICAS=$(kubectl get deployment $DEPLOYMENT -n $NAMESPACE -o jsonpath='{.spec.replicas}')
         NEW_REPLICAS=$((CURRENT_REPLICAS + 2))
         
         if [ $NEW_REPLICAS -le 20 ]; then
           kubectl scale deployment $DEPLOYMENT -n $NAMESPACE --replicas=$NEW_REPLICAS
           echo "Scaled $DEPLOYMENT to $NEW_REPLICAS replicas due to high CPU"
         fi
       fi
     
     database-optimization.sh: |
       #!/bin/bash
       # Automated database optimization
       if [ "$ALERT_NAME" = "SlowDatabaseQueries" ]; then
         kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
         VACUUM ANALYZE;
         REINDEX DATABASE techflow;
         "
         echo "Database maintenance completed"
       fi
   ```

6. **Documentation and Runbooks**

   ```markdown
   # Performance Incident Response Runbook
   
   ## Quick Response Checklist
   - [ ] Check cluster resource utilization
   - [ ] Verify database connection pools
   - [ ] Monitor cache hit ratios
   - [ ] Assess network latency
   - [ ] Review recent deployments
   
   ## Escalation Thresholds
   - Response time > 2s: Scale services immediately
   - CPU > 90%: Add node capacity
   - Memory > 95%: Implement memory optimization
   - Error rate > 5%: Enable circuit breakers
   
   ## Common Root Causes
   1. Insufficient resource allocation
   2. Database query performance
   3. Cache misses or expiration
   4. Network bottlenecks
   5. Memory leaks in applications
   ```

#### Success Criteria

- [ ] Comprehensive monitoring and alerting deployed
- [ ] Automated scaling policies implemented
- [ ] Capacity planning dashboard created
- [ ] Performance testing automation established
- [ ] Incident response procedures documented
- [ ] Long-term performance strategy defined

#### 🔍 Prevention Strategy Implementation

```markdown
## Long-term Performance Strategy

### Monitoring & Alerting
- Proactive Alerts: [CPU, memory, response time thresholds]
- Business Metrics: [Revenue impact, user experience]
- Capacity Planning: [Growth projections and scaling plans]
- Trend Analysis: [Historical performance patterns]

### Automation
- Auto-scaling: [HPA and VPA configurations]
- Load Testing: [Continuous performance validation]
- Incident Response: [Automated remediation workflows]
- Capacity Management: [Predictive scaling based on patterns]

### Process Improvements
- Performance Reviews: [Regular architecture assessments]
- Code Reviews: [Performance impact analysis]
- Deployment Gates: [Performance regression detection]
- Team Training: [Performance engineering skills]

### Technology Investments
- Advanced Monitoring: [APM, distributed tracing]
- Caching Strategy: [Multi-layer caching architecture]
- Database Optimization: [Read replicas, partitioning]
- Infrastructure: [Node auto-scaling, better instance types]
```

---

## 🔍 Troubleshooting Guide

### Issue 1: Multiple Cascading Failures

**Symptoms:**

- Multiple pods failing simultaneously
- Resource constraints across nodes
- Service-to-service communication timeouts

**Investigation Steps:**

```bash
# Check for resource pressure
kubectl describe nodes | grep -E "(Pressure|Condition)"
kubectl get events --sort-by='.lastTimestamp' | grep -E "(Failed|Error)"

# Look for common failure patterns
kubectl get pods --all-namespaces --field-selector=status.phase!=Running
kubectl describe pod <failing-pod> | grep -A 10 -B 10 "Events:"
```

**Common Root Causes:**

- Node resource exhaustion
- Network partitions
- DNS resolution failures
- Shared dependency failures (database, cache)

**Resolution Strategy:**

1. Identify the primary failure (usually resource or network)
2. Scale critical services immediately
3. Fix the root cause
4. Allow cascading services to recover

### Issue 2: Intermittent Performance Degradation

**Symptoms:**

- Performance issues come and go
- Difficult to reproduce consistently
- Metrics show periodic spikes

**Investigation Steps:**

```bash
# Look for periodic patterns
kubectl top pods --all-namespaces --sort-by=cpu | head -20
kubectl get hpa --all-namespaces -w  # Watch scaling behavior

# Check for batch jobs or cron jobs
kubectl get jobs --all-namespaces
kubectl get cronjobs --all-namespaces -o wide
```

**Common Root Causes:**

- Batch processing jobs consuming resources
- Memory leaks causing periodic restarts
- Cache invalidation patterns
- Auto-scaling thrashing

**Resolution Strategy:**

1. Identify the periodic trigger
2. Adjust resource allocation or scheduling
3. Implement proper resource isolation
4. Optimize the triggering process

### Issue 3: Database Connection Pool Exhaustion

**Symptoms:**

- "Connection pool exhausted" errors
- Long database query queues
- Application timeouts

**Investigation Steps:**

```bash
# Check database connections
kubectl exec -n production deployment/postgresql -- psql -U admin -d techflow -c "
SELECT count(*), state FROM pg_stat_activity GROUP BY state;"

# Check application connection pool settings
kubectl exec -n production deployment/user-service -- \
curl -s http://localhost:8080/actuator/metrics/hikaricp.connections
```

**Resolution Strategy:**

1. Implement connection pooling (PgBouncer)
2. Optimize query performance
3. Scale database read replicas
4. Implement connection limits per service

---

## 📊 Success Metrics

### Performance Recovery Benchmarks

- [ ] **Response Time Restoration**
  - Target: <500ms average
  - Achieved: \_\_\_ms average
  - Improvement: \_\_\_% reduction

- [ ] **Resource Utilization Optimization**
  - CPU usage: <70% cluster-wide
  - Memory usage: <80% cluster-wide
  - Storage I/O: Within baseline levels

- [ ] **Service Availability**
  - Uptime: >99.9%
  - Error rate: <0.1%
  - Zero pod crash loops

### Business Impact Recovery

- [ ] **Revenue Protection**
  - Revenue loss stopped
  - Customer complaints resolved
  - SLA compliance restored

- [ ] **Operational Excellence**
  - Mean Time to Detection (MTTD): <5 minutes
  - Mean Time to Resolution (MTTR): <60 minutes
  - Incident documentation complete

### Long-term Sustainability

- [ ] **Monitoring & Alerting**
  - Proactive alerts implemented
  - Capacity planning dashboard active
  - Performance baselines established

- [ ] **Process Improvements**
  - Incident response playbook updated
  - Performance testing automated
  - Team training completed

---

## 🎓 Key Learning Outcomes

Upon completing this expert-level lab, you will have mastered:

### Advanced Troubleshooting Skills

- **Systematic performance investigation methodology**
- **Multi-layer bottleneck identification techniques**
- **Real-time optimization under pressure**
- **Root cause analysis in complex distributed systems**

### Production Engineering Excellence

- **Performance monitoring and alerting design**
- **Capacity planning and resource optimization**
- **Incident response automation**
- **Long-term performance strategy development**

### CKA Expert-Level Competencies

- **Complex troubleshooting scenarios** (Critical exam skill)
- **Resource management optimization** (Production-ready skills)
- **Performance analysis across all cluster components**
- **Real-world incident response procedures**

---

## 🌐 Real-World Applications

### Industry Scenarios This Prepares You For

1. **E-commerce Peak Traffic Events**
   - Black Friday/Cyber Monday traffic surges
   - Flash sales causing system overload
   - Global expansion performance challenges

2. **Financial Services High Availability**
   - Trading platform performance requirements
   - Payment processing optimization
   - Regulatory compliance under load

3. **Media & Entertainment Scaling**
   - Viral content traffic spikes
   - Live streaming performance
   - Content delivery optimization

4. **Enterprise SaaS Operations**
   - Multi-tenant performance isolation
   - Customer-specific SLA requirements
   - Cost optimization under scale

### Career Skills Developed

- **Site Reliability Engineering (SRE)**
- **DevOps Performance Engineering**
- **Platform Engineering Leadership**
- **Production Incident Management**

---

## 📚 Additional Resources

### Advanced Topics for Further Study

- [Kubernetes Performance Tuning Guide](https://kubernetes.io/docs/concepts/cluster-administration/system-logs/)
- [Prometheus Performance Monitoring](https://prometheus.io/docs/practices/instrumentation/)
- [Distributed Systems Performance](http://www.brendangregg.com/systems-performance-2nd-edition-book.html)

### Professional Development

- **SRE Certification Paths**
- **Performance Engineering Specialization**
- **Kubernetes Expert Certifications (CKS, CKAD)**

---

## 🚀 Next Steps

### Immediate Actions

1. [ ] Document lessons learned from this incident
2. [ ] Implement monitoring recommendations
3. [ ] Update incident response procedures
4. [ ] Schedule performance review sessions

### Advanced Challenges

1. [ ] Create chaos engineering scenarios
2. [ ] Implement predictive scaling algorithms
3. [ ] Design multi-cluster performance strategies
4. [ ] Build automated performance regression detection

### Certification Readiness

- [ ] Master all troubleshooting methodologies
- [ ] Practice time-constrained debugging
- [ ] Understand performance implications of all Kubernetes resources
- [ ] Develop confidence in high-pressure problem solving

---

*💡 "Expert troubleshooters don't just solve problems—they prevent them. Master systematic investigation, implement proactive monitoring, and build resilient systems that perform under any load."*
