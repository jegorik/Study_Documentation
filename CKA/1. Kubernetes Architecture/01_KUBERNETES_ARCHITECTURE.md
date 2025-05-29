# CKA Exam Cheat Sheet for Kubernetes Architecture

---

## Section 1: Exam Topic Overview

### Key Concepts
- **Control Plane Components:** Master node components that manage the cluster
- **Worker Node Components:** Components that run on each worker node
- **Cluster Networking:** How components communicate within the cluster
- **etcd:** Distributed key-value store for cluster data
- **Container Runtime:** Software responsible for running containers

### Exam Objectives
- [ ] Understand cluster architecture and component responsibilities
- [ ] Configure and manage control plane components
- [ ] Understand worker node architecture and components
- [ ] Troubleshoot cluster communication issues
- [ ] Manage cluster networking and service discovery

### Study Resources
- Official Kubernetes Documentation: https://kubernetes.io/docs/concepts/overview/components/
- CNCF Training Materials: Kubernetes Fundamentals
- Practice Labs: Cluster setup and component management

### Exam Weight
**25%** of total exam score

---

## Section 2: ASCII Drawing with Explanation

```
                    KUBERNETES CLUSTER ARCHITECTURE
    ┌─────────────────────────────────────────────────────────────────┐
    │                        CONTROL PLANE                            │
    │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
    │  │     etcd     │  │ API Server   │  │   Controller         │   │
    │  │ (Key-Value   │  │ (REST API    │  │     Manager          │   │
    │  │  Storage)    │  │  Frontend)   │  │ (Control Loops)      │   │
    │  └──────────────┘  └──────────────┘  └──────────────────────┘   │
    │           │                │                      │             │
    │           └────────────────┼──────────────────────┘             │
    │                            │                                    │
    │  ┌──────────────────────────────────────────────────────────┐   │
    │  │                    Scheduler                             │   │
    │  │                 (Pod Assignment)                         │   │
    │  └──────────────────────────────────────────────────────────┘   │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    │ (API Calls)
                                    ▼
    ┌────────────────────────────────────────────────────────────────┐
    │                        WORKER NODES                            │
    │                                                                │
    │  ┌─────────────────┐    ┌─────────────────┐                    │
    │  │   Worker Node 1 │    │   Worker Node 2 │                    │
    │  │                 │    │                 │                    │
    │  │ ┌─────────────┐ │    │ ┌─────────────┐ │       ┌─────────┐  │
    │  │ │   kubelet   │ │    │ │   kubelet   │ │  ...  │ Node N  │  │
    │  │ │(Node Agent) │ │    │ │(Node Agent) │ │       │         │  │
    │  │ └─────────────┘ │    │ └─────────────┘ │       └─────────┘  │
    │  │ ┌─────────────┐ │    │ ┌─────────────┐ │                    │
    │  │ │ kube-proxy  │ │    │ │ kube-proxy  │ │                    │
    │  │ │ (Network)   │ │    │ │ (Network)   │ │                    │
    │  │ └─────────────┘ │    │ └─────────────┘ │                    │
    │  │ ┌─────────────┐ │    │ ┌─────────────┐ │                    │
    │  │ │ Container   │ │    │ │ Container   │ │                    │
    │  │ │ Runtime     │ │    │ │ Runtime     │ │                    │
    │  │ └─────────────┘ │    │ └─────────────┘ │                    │
    │  │                 │    │                 │                    │
    │  │    ┌─────┐      │    │    ┌─────┐      │                    │
    │  │    │ Pod │      │    │    │ Pod │      │                    │
    │  │    └─────┘      │    │    └─────┘      │                    │
    │  └─────────────────┘    └─────────────────┘                    │
    └────────────────────────────────────────────────────────────────┘
```

### Component Breakdown

#### Control Plane Components
1. **API Server (kube-apiserver):**
   - Purpose: Frontend for the Kubernetes control plane
   - Key features: REST API, authentication, authorization, admission control
   - Interactions: Receives requests from all other components and users

2. **etcd:**
   - Purpose: Consistent and highly-available key-value store
   - Key features: Stores all cluster data, configuration, and state
   - Interactions: Only the API server communicates directly with etcd

3. **Controller Manager (kube-controller-manager):**
   - Purpose: Runs control loops that regulate cluster state
   - Key features: Node controller, ReplicaSet controller, Service controller
   - Interactions: Watches API server for changes and takes corrective actions

4. **Scheduler (kube-scheduler):**
   - Purpose: Assigns pods to worker nodes
   - Key features: Resource requirements, constraints, affinity rules
   - Interactions: Watches for unscheduled pods and selects appropriate nodes

#### Worker Node Components
1. **kubelet:**
   - Purpose: Node agent that manages pod lifecycle
   - Key features: Pod creation, health checks, resource monitoring
   - Interactions: Communicates with API server and container runtime

2. **kube-proxy:**
   - Purpose: Network proxy that manages network rules
   - Key features: Service load balancing, IP translation
   - Interactions: Implements Kubernetes Service concepts on each node

3. **Container Runtime:**
   - Purpose: Software responsible for running containers
   - Key features: Docker, containerd, CRI-O support
   - Interactions: Receives instructions from kubelet to manage containers

---

## Section 3: Live Examples

### Scenario: Diagnosing a Failed Pod Scheduling Issue

**Problem:** A pod is stuck in "Pending" state and not being scheduled to any worker node

**Solution Steps:**

#### Step 1: Check Pod Status
```bash
kubectl get pods -o wide
```
**Expected Output:**
```
NAME        READY   STATUS    RESTARTS   AGE   IP       NODE
my-app-pod  0/1     Pending   0          5m    <none>   <none>
```

#### Step 2: Investigate Pod Events
```bash
kubectl describe pod my-app-pod
```
**Expected Output:**
```
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  5m    default-scheduler  0/3 nodes are available: 3 Insufficient cpu.
```

#### Step 3: Check Node Resources
```bash
kubectl describe nodes
```
**Expected Output:**
```
Name:               worker-node-1
Capacity:
  cpu:                2
  memory:             4Gi
Allocatable:
  cpu:                1800m
  memory:             3.5Gi
Allocated resources:
  cpu:                1750m (97%)
  memory:             3.2Gi (91%)
```

#### Step 4: Check Pod Resource Requirements
```bash
kubectl get pod my-app-pod -o yaml | grep -A 5 resources
```
**Expected Output:**
```
resources:
  requests:
    cpu: 500m
    memory: 512Mi
```

#### Step 5: Solution Implementation
```bash
# Option 1: Reduce pod resource requests
kubectl edit pod my-app-pod
# Change cpu request from 500m to 200m

# Option 2: Add more nodes or increase node capacity
kubectl get nodes -o wide
```

### Additional Scenarios
- **Scenario 2:** API Server connectivity issues - Check certificates and network policies
- **Scenario 3:** etcd backup and restore - Verify cluster state persistence

---

## Section 4: Table of Useful Commands

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `kubectl cluster-info` | Display cluster information | `--dump` | `kubectl cluster-info` |
| `kubectl get nodes` | List cluster nodes | `-o wide`, `--show-labels` | `kubectl get nodes -o wide` |
| `kubectl describe node` | Show node details | `[node-name]` | `kubectl describe node worker-1` |
| `kubectl get componentstatuses` | Check component health | `-o yaml` | `kubectl get componentstatuses` |
| `kubectl top nodes` | Show node resource usage | `--sort-by=cpu` | `kubectl top nodes` |

### Control Plane Specific Commands

#### API Server
| Command | Description | Example |
|---------|-------------|---------|
| `kubectl api-resources` | List available API resources | `kubectl api-resources --verbs=list` |
| `kubectl api-versions` | List API versions | `kubectl api-versions` |
| `kubectl proxy` | Create proxy to API server | `kubectl proxy --port=8080` |

#### etcd Management
| Command | Description | Example |
|---------|-------------|---------|
| `ETCDCTL_API=3 etcdctl get` | Query etcd directly | `etcdctl get --prefix /registry/pods` |
| `ETCDCTL_API=3 etcdctl snapshot` | Create etcd backup | `etcdctl snapshot save backup.db` |
| `ETCDCTL_API=3 etcdctl snapshot restore` | Restore etcd backup | `etcdctl snapshot restore backup.db` |

#### Node Management
| Command | Description | Example |
|---------|-------------|---------|
| `kubectl cordon` | Mark node unschedulable | `kubectl cordon worker-1` |
| `kubectl uncordon` | Mark node schedulable | `kubectl uncordon worker-1` |
| `kubectl drain` | Drain node for maintenance | `kubectl drain worker-1 --ignore-daemonsets` |

---

## Section 5: Best Practices

### Management Best Practices
1. **High Availability Control Plane:**
   - Run multiple API server instances
   - Use external load balancer for API server access
   - Implement etcd clustering with odd number of members (3, 5, 7)

2. **Node Management:**
   - Label nodes appropriately for workload placement
   - Monitor node resource utilization
   - Implement node auto-scaling for dynamic workloads

3. **Resource Planning:**
   - Reserve system resources on nodes
   - Plan for control plane resource requirements
   - Monitor cluster growth and capacity

### Security Considerations
1. **API Server Security:**
   - Enable TLS encryption for all communications
   - Implement proper authentication (certificates, tokens)
   - Configure authorization (RBAC)
   - Enable audit logging

2. **etcd Security:**
   - Enable TLS for etcd cluster communication
   - Implement etcd authentication
   - Regular backup procedures
   - Secure backup storage

3. **Network Security:**
   - Use network policies to control traffic
   - Secure inter-component communication
   - Regular certificate rotation

### Performance Optimization
1. **API Server Optimization:**
   - Configure appropriate request limits
   - Enable API priority and fairness
   - Monitor API server metrics
   - Use watch caching effectively

2. **etcd Optimization:**
   - Use SSD storage for etcd
   - Monitor etcd performance metrics
   - Regular defragmentation
   - Proper backup strategies

3. **Scheduler Optimization:**
   - Configure scheduler profiles for different workloads
   - Use node affinity and anti-affinity
   - Implement resource quotas

### Troubleshooting Tips
1. **Control Plane Issues:**
   - Check component logs: `journalctl -u kubelet`
   - Verify certificates: `openssl x509 -in cert.pem -text -noout`
   - Test API server connectivity: `kubectl cluster-info`

2. **Node Issues:**
   - Check kubelet status: `systemctl status kubelet`
   - Verify container runtime: `crictl ps`
   - Check node conditions: `kubectl describe node`

3. **Network Issues:**
   - Test pod-to-pod communication
   - Verify kube-proxy configuration
   - Check CNI plugin status

### Exam-Specific Tips
- **Time Management:** Practice component troubleshooting scenarios
- **Command Shortcuts:** Alias frequently used commands
- **Verification:** Always check cluster state after changes
- **Documentation:** Know where to find kubeadm and component documentation

---

## Quick Reference Card

### Essential Commands for Kubernetes Architecture
```bash
kubectl cluster-info                   # Cluster information
kubectl get nodes -o wide              # Node status and details
kubectl get componentstatuses          # Control plane health
kubectl describe node <node-name>      # Detailed node information
kubectl top nodes                      # Node resource usage
```

### Key Files and Paths
- Kubelet config: `/var/lib/kubelet/config.yaml`
- Kubeconfig: `~/.kube/config` or `/etc/kubernetes/admin.conf`
- Static pod manifests: `/etc/kubernetes/manifests/`
- Certificate directory: `/etc/kubernetes/pki/`
- etcd data: `/var/lib/etcd/`

### Critical Control Plane Pods
- `kube-apiserver`
- `kube-controller-manager` 
- `kube-scheduler`
- `etcd`

---

## Related Topics
- [Cluster Installation and Configuration]
- [Networking and Service Discovery]
- [Security and RBAC]
- [Troubleshooting and Monitoring]

---

*Last Updated: May 29, 2025*
*CKA Exam Version: v1.30*
