# ✅ CKA Exam Topics Checklist

> **Granular progress tracking for all CKA exam objectives**

## 📊 Overall Progress Overview

### Exam Domains Summary
| Domain | Weight | Status | Confidence | Last Review |
|--------|--------|--------|------------|-------------|
| 🔧 **Troubleshooting** | 30% | ⏳ In Progress | ⭐⭐⭐☆☆ | ___ |
| 🏗️ **Cluster Architecture** | 25% | ⏳ In Progress | ⭐⭐⭐☆☆ | ___ |
| ⚙️ **Workloads & Scheduling** | 15% | ⏳ In Progress | ⭐⭐⭐☆☆ | ___ |
| 🌐 **Services & Networking** | 20% | ⏳ In Progress | ⭐⭐⭐☆☆ | ___ |
| 💾 **Storage** | 10% | ⏳ In Progress | ⭐⭐⭐☆☆ | ___ |

**Overall Readiness:** ⏳ **In Progress** (___% Complete)

---

## 🔧 Troubleshooting (30% - Highest Priority!)

### Core Troubleshooting Skills
#### Cluster Component Issues
- [ ] **Diagnose cluster component failures**
  - [ ] Check kubelet status and logs
  - [ ] Verify API server accessibility
  - [ ] Examine etcd health and connectivity
  - [ ] Validate controller manager operation
  - [ ] Check scheduler functionality
  - **Practice:** `systemctl status kubelet`, `journalctl -u kubelet`

#### Node Troubleshooting
- [ ] **Diagnose node not ready status**
  - [ ] Check node conditions and events
  - [ ] Verify network connectivity
  - [ ] Examine resource availability (CPU/Memory/Disk)
  - [ ] Validate kubelet configuration
  - **Practice:** `kubectl describe node`, `kubectl get nodes -o wide`

- [ ] **Resolve node resource issues**
  - [ ] Identify resource pressure (memory, disk, PID)
  - [ ] Clean up unused containers and images
  - [ ] Resolve disk space issues
  - [ ] Address memory pressure
  - **Practice:** `docker system prune`, `df -h`, `free -h`

#### Pod and Workload Issues
- [ ] **Debug pod startup failures**
  - [ ] Analyze pod events and conditions
  - [ ] Check image pull issues
  - [ ] Resolve configuration problems
  - [ ] Fix resource constraint issues
  - **Practice:** `kubectl describe pod`, `kubectl logs`, `kubectl get events`

- [ ] **Troubleshoot running pod issues**
  - [ ] Debug application crashes
  - [ ] Resolve container restart loops
  - [ ] Fix liveness/readiness probe failures
  - [ ] Address performance issues
  - **Practice:** `kubectl exec -it`, `kubectl top pods`

#### Network Troubleshooting
- [ ] **Diagnose service connectivity issues**
  - [ ] Verify service endpoints
  - [ ] Test DNS resolution
  - [ ] Check network policies
  - [ ] Validate service selectors and labels
  - **Practice:** `kubectl get endpoints`, `nslookup`, `kubectl exec -- wget`

- [ ] **Resolve network communication problems**
  - [ ] Debug pod-to-pod connectivity
  - [ ] Test ingress and egress traffic
  - [ ] Validate CNI configuration
  - [ ] Check firewall and security groups
  - **Practice:** Network debugging pods, ping tests

### Application Troubleshooting
- [ ] **Debug failed deployments**
  - [ ] Check rollout status and history
  - [ ] Identify resource quota issues
  - [ ] Resolve configuration errors
  - [ ] Fix image and registry problems
  - **Practice:** `kubectl rollout status`, `kubectl describe deployment`

- [ ] **Troubleshoot service discovery issues**
  - [ ] Verify service registration
  - [ ] Test load balancing
  - [ ] Check service mesh configuration
  - [ ] Validate ingress routing
  - **Practice:** Service connectivity tests

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## 🏗️ Cluster Architecture, Installation & Configuration (25%)

### Cluster Architecture Understanding
#### Control Plane Components
- [ ] **Master the API Server**
  - [ ] Understand API server role and responsibilities
  - [ ] Know authentication and authorization flow
  - [ ] Explain admission controllers
  - [ ] Describe API versioning and groups
  - **Practice:** Examine API server configuration, test API calls

- [ ] **Comprehend etcd functionality**
  - [ ] Understand etcd's role as cluster data store
  - [ ] Know backup and restore procedures
  - [ ] Explain clustering and high availability
  - [ ] Describe data encryption at rest
  - **Practice:** `etcdctl` commands, backup/restore scenarios

- [ ] **Grasp Controller Manager operations**
  - [ ] Understand various controllers (deployment, replica set, etc.)
  - [ ] Know controller reconciliation loops
  - [ ] Explain leader election
  - [ ] Describe controller failure handling
  - **Practice:** Observe controller behavior, examine logs

- [ ] **Master Scheduler functionality**
  - [ ] Understand pod scheduling decisions
  - [ ] Know node selection criteria
  - [ ] Explain scheduling policies and priorities
  - [ ] Describe custom schedulers
  - **Practice:** Scheduling constraints, node affinity

#### Worker Node Components
- [ ] **Master kubelet operations**
  - [ ] Understand pod lifecycle management
  - [ ] Know container runtime interaction
  - [ ] Explain volume management
  - [ ] Describe node status reporting
  - **Practice:** Kubelet configuration, static pods

- [ ] **Comprehend kube-proxy functionality**
  - [ ] Understand service load balancing
  - [ ] Know iptables/IPVS modes
  - [ ] Explain traffic routing
  - [ ] Describe service discovery
  - **Practice:** Service connectivity, proxy modes

- [ ] **Grasp Container Runtime**
  - [ ] Understand CRI interface
  - [ ] Know Docker/containerd/CRI-O differences
  - [ ] Explain image management
  - [ ] Describe security contexts
  - **Practice:** Container runtime commands, image operations

### Cluster Installation and Configuration
#### Cluster Setup Methods
- [ ] **kubeadm installation**
  - [ ] Understand kubeadm init process
  - [ ] Know worker node joining procedures
  - [ ] Explain certificate management
  - [ ] Describe cluster upgrades
  - **Practice:** Full cluster setup with kubeadm

- [ ] **Manual cluster setup**
  - [ ] Understand manual component installation
  - [ ] Know certificate generation and distribution
  - [ ] Explain systemd service configuration
  - [ ] Describe network setup
  - **Practice:** "Kubernetes the Hard Way" tutorial

#### High Availability Clusters
- [ ] **Multi-master setup**
  - [ ] Understand load balancer configuration
  - [ ] Know etcd clustering
  - [ ] Explain API server high availability
  - [ ] Describe failover scenarios
  - **Practice:** HA cluster deployment

- [ ] **Backup and Restore**
  - [ ] Master etcd backup procedures
  - [ ] Know cluster state restoration
  - [ ] Understand disaster recovery
  - [ ] Explain backup automation
  - **Practice:** `etcdctl snapshot save/restore`

### Network Configuration
- [ ] **CNI (Container Network Interface)**
  - [ ] Understand CNI plugin architecture
  - [ ] Know popular CNI solutions (Calico, Flannel, Weave)
  - [ ] Explain network policy implementation
  - [ ] Describe multi-network configurations
  - **Practice:** CNI plugin installation and configuration

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## ⚙️ Workloads & Scheduling (15%)

### Pod Management
#### Basic Pod Operations
- [ ] **Create and manage pods**
  - [ ] Use imperative commands (`kubectl run`)
  - [ ] Create pods from YAML manifests
  - [ ] Understand pod lifecycle phases
  - [ ] Manage pod deletion and cleanup
  - **Practice:** `kubectl run nginx --image=nginx`, YAML creation

- [ ] **Multi-container pods**
  - [ ] Design sidecar patterns
  - [ ] Implement init containers
  - [ ] Configure shared volumes between containers
  - [ ] Understand container communication
  - **Practice:** Multi-container pod scenarios

#### Pod Configuration
- [ ] **Resource management**
  - [ ] Set CPU and memory requests/limits
  - [ ] Understand resource quotas
  - [ ] Configure quality of service classes
  - [ ] Monitor resource usage
  - **Practice:** Resource constraint scenarios

- [ ] **Security contexts**
  - [ ] Configure user and group IDs
  - [ ] Set file system permissions
  - [ ] Enable/disable privilege escalation
  - [ ] Configure capabilities
  - **Practice:** Security context configurations

### Workload Controllers
#### Deployments
- [ ] **Create and manage deployments**
  - [ ] Use imperative commands
  - [ ] Create from YAML manifests
  - [ ] Scale deployments up/down
  - [ ] Update deployment configurations
  - **Practice:** `kubectl create deployment`, scaling scenarios

- [ ] **Rolling updates and rollbacks**
  - [ ] Perform rolling updates
  - [ ] Monitor rollout status
  - [ ] Rollback to previous versions
  - [ ] Configure rollout strategies
  - **Practice:** `kubectl rollout`, update scenarios

#### ReplicaSets and DaemonSets
- [ ] **ReplicaSet management**
  - [ ] Understand relationship with deployments
  - [ ] Create standalone ReplicaSets
  - [ ] Scale replica counts
  - [ ] Handle replica failures
  - **Practice:** ReplicaSet scenarios, failure handling

- [ ] **DaemonSet operations**
  - [ ] Deploy DaemonSets for node-level services
  - [ ] Update DaemonSet configurations
  - [ ] Handle node scheduling constraints
  - [ ] Monitor DaemonSet status
  - **Practice:** DaemonSet deployment, node affinity

#### StatefulSets and Jobs
- [ ] **StatefulSet management**
  - [ ] Deploy stateful applications
  - [ ] Manage persistent volumes
  - [ ] Handle ordered pod deployment
  - [ ] Configure headless services
  - **Practice:** Database deployments, scaling scenarios

- [ ] **Job and CronJob operations**
  - [ ] Create one-time jobs
  - [ ] Schedule recurring jobs
  - [ ] Handle job failures and retries
  - [ ] Monitor job completion
  - **Practice:** Batch processing scenarios

### Scheduling
#### Node Selection
- [ ] **Node selectors and affinity**
  - [ ] Use node selectors for basic node selection
  - [ ] Configure node affinity rules
  - [ ] Implement anti-affinity constraints
  - [ ] Use topology spread constraints
  - **Practice:** Node placement scenarios

- [ ] **Taints and tolerations**
  - [ ] Apply taints to nodes
  - [ ] Configure pod tolerations
  - [ ] Understand taint effects (NoSchedule, PreferNoSchedule, NoExecute)
  - [ ] Implement node isolation
  - **Practice:** Node isolation scenarios

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## 🌐 Services & Networking (20%)

### Service Types and Configuration
#### Core Service Types
- [ ] **ClusterIP services**
  - [ ] Create internal cluster services
  - [ ] Configure service selectors
  - [ ] Test internal connectivity
  - [ ] Understand virtual IP allocation
  - **Practice:** Internal service communication

- [ ] **NodePort services**
  - [ ] Expose services on node ports
  - [ ] Configure port ranges
  - [ ] Test external connectivity
  - [ ] Understand port allocation
  - **Practice:** External access scenarios

- [ ] **LoadBalancer services**
  - [ ] Deploy cloud load balancer services
  - [ ] Configure external IP allocation
  - [ ] Test load balancing behavior
  - [ ] Understand cloud provider integration
  - **Practice:** Cloud load balancer scenarios

#### Advanced Service Features
- [ ] **Headless services**
  - [ ] Create services without cluster IP
  - [ ] Enable direct pod addressing
  - [ ] Configure StatefulSet services
  - [ ] Understand DNS record creation
  - **Practice:** StatefulSet service scenarios

- [ ] **Service discovery**
  - [ ] Use DNS for service discovery
  - [ ] Configure environment variables
  - [ ] Test service resolution
  - [ ] Understand service endpoints
  - **Practice:** Service discovery scenarios

### Ingress and Network Policies
#### Ingress Controllers
- [ ] **Ingress configuration**
  - [ ] Deploy ingress controllers
  - [ ] Create ingress resources
  - [ ] Configure path-based routing
  - [ ] Set up host-based routing
  - **Practice:** Ingress deployment scenarios

- [ ] **TLS termination**
  - [ ] Configure SSL certificates
  - [ ] Set up TLS termination
  - [ ] Handle certificate management
  - [ ] Test HTTPS connectivity
  - **Practice:** TLS configuration scenarios

#### Network Policies
- [ ] **Traffic segmentation**
  - [ ] Create ingress network policies
  - [ ] Configure egress network policies
  - [ ] Implement namespace isolation
  - [ ] Test policy enforcement
  - **Practice:** Network policy scenarios

- [ ] **Pod-to-pod communication**
  - [ ] Control inter-pod traffic
  - [ ] Configure label-based rules
  - [ ] Implement deny-all policies
  - [ ] Test network connectivity
  - **Practice:** Micro-segmentation scenarios

### DNS and Service Mesh
- [ ] **CoreDNS configuration**
  - [ ] Understand cluster DNS
  - [ ] Configure DNS policies
  - [ ] Troubleshoot DNS issues
  - [ ] Custom DNS configurations
  - **Practice:** DNS troubleshooting scenarios

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## 💾 Storage (10%)

### Persistent Volumes and Claims
#### Storage Architecture
- [ ] **Persistent Volume (PV) management**
  - [ ] Create static persistent volumes
  - [ ] Configure access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
  - [ ] Set reclaim policies (Retain, Delete, Recycle)
  - [ ] Understand volume types and capabilities
  - **Practice:** PV creation and configuration

- [ ] **Persistent Volume Claim (PVC) operations**
  - [ ] Create PVC requests
  - [ ] Bind PVCs to PVs
  - [ ] Configure storage class references
  - [ ] Handle claim expansion
  - **Practice:** PVC binding scenarios

#### Dynamic Provisioning
- [ ] **Storage Classes**
  - [ ] Create storage class definitions
  - [ ] Configure provisioner parameters
  - [ ] Set default storage classes
  - [ ] Understand volume binding modes
  - **Practice:** Dynamic provisioning scenarios

- [ ] **CSI (Container Storage Interface)**
  - [ ] Understand CSI driver architecture
  - [ ] Deploy CSI drivers
  - [ ] Configure volume snapshots
  - [ ] Handle volume expansion
  - **Practice:** CSI driver deployment

### Volume Types and Usage
#### Volume Types
- [ ] **EmptyDir volumes**
  - [ ] Create temporary storage
  - [ ] Configure memory-backed volumes
  - [ ] Understand volume lifecycle
  - [ ] Share data between containers
  - **Practice:** EmptyDir scenarios

- [ ] **HostPath volumes**
  - [ ] Mount host directories
  - [ ] Configure directory permissions
  - [ ] Understand security implications
  - [ ] Handle node-specific data
  - **Practice:** HostPath mounting scenarios

- [ ] **ConfigMap and Secret volumes**
  - [ ] Mount configuration data
  - [ ] Configure file permissions
  - [ ] Handle updates and reloads
  - [ ] Use subPath for specific files
  - **Practice:** Configuration volume scenarios

#### Storage Best Practices
- [ ] **Data persistence patterns**
  - [ ] Design stateful applications
  - [ ] Handle data backup and restore
  - [ ] Implement data migration strategies
  - [ ] Configure volume monitoring
  - **Practice:** Data persistence scenarios

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## 📚 Configuration and Secrets Management

### ConfigMaps
- [ ] **Create and manage ConfigMaps**
  - [ ] Create from literals, files, and directories
  - [ ] Mount as volumes or environment variables
  - [ ] Update configurations without restarts
  - [ ] Handle configuration versioning
  - **Practice:** `kubectl create configmap`, volume mounting

### Secrets
- [ ] **Secret types and usage**
  - [ ] Generic secrets for passwords/keys
  - [ ] TLS secrets for certificates
  - [ ] Docker registry secrets
  - [ ] Service account tokens
  - **Practice:** Secret creation and mounting

### Security Context and RBAC
- [ ] **Pod Security Standards**
  - [ ] Configure security contexts
  - [ ] Implement pod security policies
  - [ ] Handle privileged containers
  - [ ] Set up network policies
  - **Practice:** Security constraint scenarios

- [ ] **RBAC Configuration**
  - [ ] Create roles and cluster roles
  - [ ] Configure role bindings
  - [ ] Test permissions with `kubectl auth can-i`
  - [ ] Implement least privilege access
  - **Practice:** RBAC scenarios

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## 🎯 Exam-Specific Skills

### Time Management
- [ ] **Quick resource creation**
  - [ ] Master imperative commands
  - [ ] Use YAML generation shortcuts
  - [ ] Practice common patterns
  - [ ] Build muscle memory for commands
  - **Target:** Create basic resources in <2 minutes

### Documentation Navigation
- [ ] **Kubernetes.io efficiency**
  - [ ] Know key documentation sections
  - [ ] Use search effectively
  - [ ] Find examples quickly
  - [ ] Copy-paste YAML templates
  - **Target:** Find relevant docs in <30 seconds

### Verification and Testing
- [ ] **Solution validation**
  - [ ] Test all created resources
  - [ ] Verify connectivity and functionality
  - [ ] Check resource status and events
  - [ ] Confirm requirements are met
  - **Target:** Verify solutions in <1 minute

**Confidence Level:** ⭐⭐⭐☆☆ | **Status:** ⏳ In Progress | **Last Review:** ___

---

## 📊 Study Progress Tracking

### Weekly Goals
#### Week 1-2: Foundation
- [ ] Complete Architecture understanding
- [ ] Master basic kubectl commands
- [ ] Practice pod management scenarios

#### Week 3-4: Core Skills
- [ ] Complete Workloads & Scheduling
- [ ] Master Services & Networking
- [ ] Practice configuration management

#### Week 5-6: Advanced Topics
- [ ] Complete Storage understanding
- [ ] Master Troubleshooting skills
- [ ] Practice complex scenarios

#### Week 7: Exam Preparation
- [ ] Complete all mock exams
- [ ] Practice under time pressure
- [ ] Review weak areas

#### Week 8: Final Preparation
- [ ] Light review only
- [ ] Confidence building
- [ ] Exam day preparation

### Confidence Rating Scale
- ⭐☆☆☆☆ **1/5 - Just Started:** Basic awareness, no hands-on experience
- ⭐⭐☆☆☆ **2/5 - Learning:** Understanding concepts, beginning practice
- ⭐⭐⭐☆☆ **3/5 - Developing:** Can complete tasks with reference
- ⭐⭐⭐⭐☆ **4/5 - Proficient:** Can complete tasks independently
- ⭐⭐⭐⭐⭐ **5/5 - Expert:** Can teach others, handle complex scenarios

### Exam Readiness Indicators
✅ **Ready for exam when:**
- [ ] All topics rated ⭐⭐⭐⭐☆ or higher
- [ ] Mock exam scores consistently >80%
- [ ] Can complete practice scenarios within time limits
- [ ] Troubleshooting feels routine and confident
- [ ] kubectl commands are muscle memory

---

## 📅 Study Session Log

### Week ___: ________________

| Date | Topics Studied | Time Spent | Practice Labs | Confidence Change | Notes |
|------|---------------|------------|---------------|-------------------|-------|
| ___ | ___ | ___ min | ___ | ⭐⭐⭐☆☆ → ⭐⭐⭐⭐☆ | ___ |
| ___ | ___ | ___ min | ___ | ⭐⭐⭐☆☆ → ⭐⭐⭐⭐☆ | ___ |
| ___ | ___ | ___ min | ___ | ⭐⭐⭐☆☆ → ⭐⭐⭐⭐☆ | ___ |

**Weekly Reflection:**
- **Strengths this week:** ___
- **Areas needing more focus:** ___
- **Plan for next week:** ___

---

*Use this checklist to track your detailed progress through every CKA exam objective. Update confidence ratings weekly and focus extra time on areas below ⭐⭐⭐☆☆.*

---

*Last Updated: May 29, 2025*  
*CKA Exam Version: v1.30*  
*Checklist Version: 1.0*
