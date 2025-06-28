# CKA Exam Cheat Sheet for Troubleshooting

---

## Section 1: Exam Topic Overview

### Key Concepts

- **Cluster Component Failures:** API server, etcd, scheduler, controller manager issues
- **Worker Node Issues:** kubelet, kube-proxy, container runtime problems
- **Application Troubleshooting:** Pod startup, networking, resource issues
- **Log Analysis:** System and application log investigation
- **Resource Monitoring:** CPU, memory, storage utilization problems
- **Network Debugging:** Service discovery, connectivity, DNS issues

### Exam Objectives

- [ ] Evaluate cluster and node logging
- [ ] Understand how to monitor applications
- [ ] Manage container stdout & stderr logs
- [ ] Troubleshoot application failure
- [ ] Troubleshoot cluster component failure
- [ ] Troubleshoot networking

### Study Resources

- Official Kubernetes Documentation: https://kubernetes.io/docs/tasks/debug-application-cluster/
- CNCF Training Materials: Troubleshooting and Monitoring
- Practice Labs: Failure scenarios and debugging

### Exam Weight

**30%** of total exam score (HIGHEST WEIGHT!)

---

## Section 2: ASCII Drawing with Explanation

```text
                        KUBERNETES TROUBLESHOOTING FLOW
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                    PROBLEM IDENTIFICATION                       │
    │                                                                 │
    │  User Reports Issue ──> Symptom Analysis ──> Initial Diagnosis  │
    │                                                                 │
    │  ┌──────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
    │  │"App is down" │    │Check Status │    │Identify Component   │ │
    │  │"Slow response│    │Get Events   │    │Pod/Service/Node     │ │
    │  │"Cannot scale"│    │View Logs    │    │Network/Storage      │ │
    │  └──────────────┘    └─────────────┘    └─────────────────────┘ │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                    TROUBLESHOOTING LAYERS                       │
    │                                                                 │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                 APPLICATION LAYER                       │    │
    │  │                                                         │    │
    │  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │    │
    │  │ │    Pods     │  │  Services   │  │   ConfigMaps    │   │    │
    │  │ │             |  │             │  │      & Secrets  │   │    │
    │  │ │ [X] Failed  │  │ [X] No      │  │                 │   │    │
    │  │ │ [!] Pending │  │   Endpoints │  │ [X] Missing     │   │    │
    │  │ │ [~] Restart │  │ [!]  Wrong  │  │     Config      │   │    │
    │  │ │   Loop      │  │   Target    │  │                 │   │    │
    │  │ └─────────────┘  └─────────────┘  └─────────────────┘   │    │
    │  └─────────────────────────────────────────────────────────┘    │
    │                                │                                │
    │                                ▼                                │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                NETWORKING LAYER                         │    │
    │  │                                                         │    │
    │  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │    │
    │  │ │    DNS      │  │ kube-proxy  │  │ Network Policies│   │    │
    │  │ │             │  │             │  │                 │   │    │
    │  │ │ [X] Cannot  │  │ [X] Not     │  │ [X] Blocking    │   │    │
    │  │ │   Resolve   │  │   Running   │  │   Traffic       │   │    │
    │  │ │ [!]  Slow   │  │ [!]  Rules  │  │ [!]  Wrong      │   │    │
    │  │ │   Response  │  │   Outdated  │  │   Selector      │   │    │
    │  │ └─────────────┘  └─────────────┘  └─────────────────┘   │    │
    │  └─────────────────────────────────────────────────────────┘    │
    │                                │                                │
    │                                ▼                                │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                  NODE LAYER                             │    │
    │  │                                                         │    │
    │  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │    │
    │  │ │   kubelet   │  │ Container   │  │    Resources    │   │    │
    │  │ │             │  │ Runtime     │  │                 │   │    │
    │  │ │ [X] Down    │  │ [X] Failed  │  │ [X] CPU/Memory  │   │    │
    │  │ │ [!] Config  │  │   to Start  │  │   Exhausted     │   │    │
    │  │ │   Error     │  │ [!] Version │  │ [!]  Disk Full  │   │    │
    │  │ │             │  │   Mismatch  │  │                 │   │    │
    │  │ └─────────────┘  └─────────────┘  └─────────────────┘   │    │
    │  └─────────────────────────────────────────────────────────┘    │
    │                                │                                │
    │                                ▼                                │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │               CONTROL PLANE LAYER                       │    │
    │  │                                                         │    │
    │  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │    │
    │  │ │ API Server  │  │    etcd     │  │   Scheduler &   │   │    │
    │  │ │             │  │             │  │  Controllers    │   │    │
    │  │ │ [X] Not     │  │ [X] Cannot  │  │                 │   │    │
    │  │ │   Responding│  │   Connect   │  │ [X] Not Running │   │    │
    │  │ │ [!]  High   │  │ [!] Corrupt │  │ [!] Resource    │   │    │
    │  │ │   Latency   │  │   Data      │  │   Conflicts     │   │    │
    │  │ └─────────────┘  └─────────────┘  └─────────────────┘   │    │
    │  └─────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────┘

                          DEBUGGING METHODOLOGY
    
         ┌─────────────┐
         │  1. OBSERVE │ ──▶ kubectl get, describe, logs
         └─────────────┘
                │
                ▼
         ┌─────────────┐
         │  2. ANALYZE │ ──▶ Events, Resource Usage, Configs
         └─────────────┘
                │
                ▼
         ┌──────────────┐
         │3. HYPOTHESIZE│ ──▶ Form theories about root cause
         └──────────────┘
                │
                ▼
         ┌─────────────┐
         │   4. TEST   │ ──▶ Validate hypothesis with changes
         └─────────────┘
                │
                ▼
         ┌─────────────┐
         │  5. VERIFY  │ ──▶ Confirm fix resolves the issue
         └─────────────┘
```

### Symbol Legend

- **[X]** = Critical failure/error state
- **[!]** = Warning/degraded performance
- **[~]** = Restart/retry in progress

### Troubleshooting Layers Breakdown

#### Application Layer Issues

1. **Pod Problems:**
   - Failed containers due to image issues, resource limits, or configuration errors
   - Pending pods due to scheduling constraints or resource unavailability
   - CrashLoopBackOff due to application errors or misconfiguration

2. **Service Issues:**
   - No endpoints due to label selector mismatches
   - Wrong target ports or protocols
   - Service type misconfiguration

3. **Configuration Problems:**
   - Missing or incorrect ConfigMaps/Secrets
   - Volume mount failures
   - Environment variable issues

#### Networking Layer Issues

1. **DNS Problems:**
   - CoreDNS pod failures
   - DNS resolution timeouts
   - Incorrect DNS policies

2. **Proxy Issues:**
   - kube-proxy not running on nodes
   - Outdated iptables rules
   - Service mesh configuration problems

3. **Network Policy Issues:**
   - Overly restrictive policies blocking legitimate traffic
   - Incorrect selectors or namespaces
   - Missing ingress/egress rules

#### Node Layer Issues

1. **kubelet Problems:**
   - Service not running or misconfigured
   - Certificate or authentication issues
   - Resource management problems

2. **Container Runtime Issues:**
   - Docker/containerd/CRI-O failures
   - Image pull problems
   - Storage driver issues

3. **Resource Exhaustion:**
   - CPU, memory, or disk space limitations
   - Too many pods on a node
   - Storage capacity issues

#### Control Plane Issues

1. **API Server Problems:**
   - Service unavailable or high latency
   - Authentication/authorization failures
   - Certificate expiration

2. **etcd Issues:**
   - Data corruption or connectivity problems
   - Performance degradation
   - Backup/restore issues

3. **Controller Problems:**
   - Scheduler not placing pods
   - Controller manager not reconciling state
   - Resource conflicts or deadlocks

---

## Section 3: Live Examples

### Scenario 1: Debugging a Pod Stuck in Pending State

**Problem:** A pod has been in "Pending" state for several minutes and is not being scheduled

**Solution Steps:**

#### Step 1: Check Pod Status and Events

```bash
kubectl get pods
kubectl describe pod my-app-pod
```

**Expected Output:**

```bash
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  2m    default-scheduler  0/3 nodes are available: 1 node(s) had taint that the pod didn't tolerate, 2 Insufficient cpu.
```

#### Step 2: Analyze Node Resources

```bash
kubectl get nodes -o wide
kubectl describe nodes
kubectl top nodes
```

#### Step 3: Check Resource Requirements

```bash
kubectl get pod my-app-pod -o yaml | grep -A 10 resources
```

#### Step 4: Identify and Fix the Issue

```bash
# Option 1: Reduce resource requests
kubectl edit pod my-app-pod

# Option 2: Check node taints and tolerations
kubectl describe nodes | grep -A 5 Taints

# Option 3: Scale cluster or free up resources
kubectl delete pod resource-heavy-pod
```

### Scenario 2: Service Connectivity Troubleshooting

**Problem:** Application cannot connect to a backend service

**Solution Steps:**

#### Step 1: Verify Service and Endpoints

```bash
kubectl get services
kubectl get endpoints backend-service
kubectl describe service backend-service
```

#### Step 2: Test DNS Resolution

```bash
kubectl run debug --image=busybox --rm -it --restart=Never -- nslookup backend-service.default.svc.cluster.local
```

#### Step 3: Test Direct Connectivity

```bash
# Get service ClusterIP
kubectl get service backend-service -o jsonpath='{.spec.clusterIP}'

# Test connectivity from debug pod
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -qO- http://10.96.100.50:80
```

#### Step 4: Check Network Policies

```bash
kubectl get networkpolicies
kubectl describe networkpolicy backend-netpol
```

### Scenario 3: Node Troubleshooting

**Problem:** A worker node shows "NotReady" status

**Solution Steps:**

#### Step 1: Check Node Status and Conditions

```bash
kubectl get nodes
kubectl describe node worker-node-1
```

#### Step 2: Check kubelet Status on the Node

```bash
# SSH to the node or use kubectl node-shell
systemctl status kubelet
journalctl -u kubelet -f
```

#### Step 3: Check Container Runtime

```bash
systemctl status docker
# or
systemctl status containerd

# Check container runtime connectivity
crictl ps
```

#### Step 4: Check Node Resources

```bash
kubectl top node worker-node-1
df -h  # Check disk space
free -h  # Check memory
```

---

## Section 4: Table of Useful Commands

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `kubectl get events` | Show cluster events | `--sort-by=.metadata.creationTimestamp`, `--field-selector` | `kubectl get events --sort-by=.metadata.creationTimestamp` |
| `kubectl logs` | Show pod/container logs | `-f`, `-c container`, `--previous` | `kubectl logs nginx-pod -f` |
| `kubectl describe` | Detailed resource information | `pod/node/service` | `kubectl describe pod nginx-pod` |
| `kubectl exec` | Execute commands in container | `-it`, `-c container` | `kubectl exec -it nginx-pod -- /bin/bash` |
| `kubectl top` | Resource usage statistics | `nodes`, `pods` | `kubectl top nodes` |

### Debugging Commands

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get pods --all-namespaces` | List all pods | `kubectl get pods -A -o wide` |
| `kubectl get events --field-selector` | Filter events | `kubectl get events --field-selector involvedObject.name=nginx-pod` |
| `kubectl logs --previous` | Previous container logs | `kubectl logs nginx-pod --previous` |
| `kubectl port-forward` | Port forwarding for testing | `kubectl port-forward nginx-pod 8080:80` |

### Node Debugging

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get nodes -o wide` | Node status with details | `kubectl get nodes -o wide` |
| `kubectl describe node` | Node detailed information | `kubectl describe node worker-1` |
| `kubectl cordon/uncordon` | Node scheduling control | `kubectl cordon worker-1` |
| `kubectl drain` | Safely evict pods from node | `kubectl drain worker-1 --ignore-daemonsets` |

### Service and Network Debugging

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get endpoints` | Service endpoint information | `kubectl get endpoints my-service` |
| `kubectl run debug` | Create debugging pod | `kubectl run debug --image=busybox --rm -it` |
| `nslookup` | DNS resolution testing | `nslookup my-service.default.svc.cluster.local` |
| `curl` | HTTP connectivity testing | `curl -v http://my-service:80` |

### Log Analysis

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl logs -l app=nginx` | Logs from labeled pods | `kubectl logs -l app=nginx --tail=100` |
| `kubectl logs --all-containers` | All container logs in pod | `kubectl logs nginx-pod --all-containers=true` |
| `journalctl -u kubelet` | System service logs | `journalctl -u kubelet --since "1 hour ago"` |

---

## Section 5: Best Practices

### Systematic Troubleshooting Approach

1. **Start with High-Level Overview:**
   - Check cluster-wide status: `kubectl get nodes`
   - Review recent events: `kubectl get events --sort-by=.metadata.creationTimestamp`
   - Identify affected namespaces and resources
   - Gather initial symptoms and timeline

2. **Follow the Data Path:**
   - User → Load Balancer → Ingress → Service → Pod → Container
   - Test each layer systematically
   - Verify configuration at each step
   - Check logs and metrics at each component

3. **Use the Debugging Hierarchy:**

   ```bash
   # 1. Check if resources exist
   kubectl get pods,services,deployments
   
   # 2. Check resource status and events
   kubectl describe pod problematic-pod
   
   # 3. Check logs for errors
   kubectl logs problematic-pod -f
   
   # 4. Test connectivity and DNS
   kubectl run debug --image=busybox --rm -it
   
   # 5. Check resource usage
   kubectl top pods,nodes
   ```

### Application Troubleshooting

1. **Pod Lifecycle Issues:**
   - **ImagePullBackOff:** Verify image exists and registry access
   - **CrashLoopBackOff:** Check application logs and configuration
   - **Pending:** Verify resource requests, node capacity, and scheduling constraints
   - **Error:** Check resource limits, security contexts, and volume mounts

2. **Configuration Debugging:**
   - Verify ConfigMaps and Secrets exist and are mounted correctly
   - Check environment variables and their values
   - Validate volume mounts and permissions
   - Test configuration changes in isolation

3. **Resource Management:**
   - Monitor CPU and memory usage patterns
   - Set appropriate resource requests and limits
   - Implement health checks (readiness/liveness probes)
   - Use debugging containers for investigation

### Network Troubleshooting

1. **Service Discovery Issues:**
   - Verify service exists and has valid endpoints
   - Check label selectors match pod labels
   - Test DNS resolution from within cluster
   - Validate service port configuration

2. **Connectivity Problems:**
   - Test pod-to-pod communication
   - Verify network policies allow traffic
   - Check kube-proxy status and configuration
   - Validate ingress controller and rules

3. **DNS Debugging:**

   ```bash
   # Check CoreDNS pods
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   
   # Test DNS resolution
   kubectl run debug --image=busybox --rm -it -- nslookup kubernetes.default
   
   # Check DNS configuration
   kubectl get configmap coredns -n kube-system -o yaml
   ```

### Node and Cluster Troubleshooting

1. **Node Issues:**
   - Check kubelet status and logs: `systemctl status kubelet`
   - Verify container runtime: `systemctl status docker`
   - Monitor node resources: `kubectl top nodes`
   - Check disk space and inode usage: `df -h && df -i`

2. **Control Plane Issues:**
   - Verify API server accessibility: `kubectl cluster-info`
   - Check control plane pod status: `kubectl get pods -n kube-system`
   - Monitor etcd health and performance
   - Review controller manager and scheduler logs

3. **Certificate and Security Issues:**
   - Check certificate expiration: `openssl x509 -in cert.pem -text -noout`
   - Verify RBAC permissions: `kubectl auth can-i create pods`
   - Review authentication and authorization logs
   - Check service account and token configuration

### Performance Troubleshooting

1. **Resource Monitoring:**
   - Use `kubectl top` for real-time resource usage
   - Implement proper monitoring with Prometheus/Grafana
   - Set up alerting for resource thresholds
   - Regular capacity planning and scaling

2. **Application Performance:**
   - Monitor application metrics and logs
   - Implement distributed tracing
   - Use profiling tools for performance analysis
   - Optimize resource requests and limits

### Log Management and Analysis

1. **Centralized Logging:**
   - Implement cluster-wide logging solution (ELK, Fluentd)
   - Structure application logs for better parsing
   - Include correlation IDs for request tracing
   - Regular log rotation and retention policies

2. **Log Analysis Techniques:**
   - Use grep and awk for pattern matching
   - Implement log aggregation and search
   - Create alerts for error patterns
   - Regular log review and analysis

### Common Issues and Solutions

1. **Pod Issues:**

   ```bash
   # Pod stuck in Pending
   kubectl describe pod <pod-name>  # Check events for scheduling issues
   
   # Pod in CrashLoopBackOff
   kubectl logs <pod-name> --previous  # Check previous container logs
   
   # Pod not receiving traffic
   kubectl get endpoints <service-name>  # Verify service endpoints
   ```

2. **Service Issues:**

   ```bash
   # Service has no endpoints
   kubectl get pods -l <service-selector>  # Check if pods match selector
   
   # Cannot connect to service
   kubectl run debug --image=busybox --rm -it -- wget -qO- <service-name>:<port>
   ```

3. **Node Issues:**

   ```bash
   # Node NotReady
   kubectl describe node <node-name>  # Check node conditions
   systemctl status kubelet  # Check kubelet status on node
   ```

### Exam-Specific Tips

- **Time Management:** Practice common troubleshooting scenarios for speed
- **Command Efficiency:** Master kubectl shortcuts and debugging commands
- **Systematic Approach:** Always follow logical troubleshooting steps
- **Documentation:** Know where to find troubleshooting guides quickly
- **Verification:** Always test that your fix resolves the issue

---

## Quick Reference Card

### Essential Troubleshooting Commands

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod-name>
kubectl logs <pod-name> -f --previous
kubectl exec -it <pod-name> -- /bin/bash
kubectl top nodes,pods
```

### Quick Debugging Workflow

```bash
# 1. Check overall status
kubectl get nodes,pods -o wide

# 2. Get recent events
kubectl get events --sort-by=.metadata.creationTimestamp | tail -20

# 3. Describe problematic resource
kubectl describe pod <failing-pod>

# 4. Check logs
kubectl logs <failing-pod> --tail=50

# 5. Test connectivity
kubectl run debug --image=busybox --rm -it -- sh
```

### Network Debugging

```bash
# DNS test
nslookup <service-name>.<namespace>.svc.cluster.local

# Connectivity test
wget -qO- http://<service-name>:<port>

# Check endpoints
kubectl get endpoints <service-name>
```

### Common Issues Quick Fixes

- **Pending Pod:** Check node resources and scheduling constraints
- **CrashLoopBackOff:** Check previous logs and resource limits
- **Service No Endpoints:** Verify label selectors and pod status
- **DNS Issues:** Check CoreDNS pods and configuration
- **Node NotReady:** Check kubelet and container runtime status

---

## Related Topics

- [Kubernetes Architecture]
- [Workloads & Scheduling]
- [Services & Networking]
- [Storage]

---

*Last Updated: May 29, 2025*
*CKA Exam Version: v1.30*
