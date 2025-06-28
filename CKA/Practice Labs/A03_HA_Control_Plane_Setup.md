# A03: HA Control Plane Setup

> **📋 Lab Category:** Architecture - Cluster Infrastructure & High Availability  
> **⏱️ Estimated Time:** 30-40 minutes  
> **🎯 Difficulty:** Advanced  
> **🔗 Exam Objectives:** Install and configure a multi-node cluster with HA control plane, Manage cluster upgrade, Understand etcd backup and restore

---

## 🎬 Real-World Scenario

### Background Context
**CloudNative Corp** is a fast-growing SaaS platform serving 50,000+ enterprise customers with mission-critical applications. Their current single-node control plane has become a dangerous single point of failure. Last month, a control plane node failure caused a 4-hour outage that cost $500K in SLA penalties and damaged customer trust.

**Your Role:** Principal Infrastructure Engineer  
**Situation:** The Board of Directors has mandated implementing High Availability infrastructure after the outage incident  
**Urgency:** **CRITICAL** - Must deploy HA control plane before Q4 launch of new enterprise features

### The Challenge
You must design and implement a production-ready HA control plane that:
- Eliminates single points of failure
- Provides automatic failover capabilities
- Maintains cluster availability during node maintenance
- Supports rolling updates without downtime
- Implements proper etcd cluster management with backup/restore procedures

**Business Impact:**
- 💰 **$50K/hour** cost of downtime for enterprise SaaS platform
- 📈 **99.99% SLA** commitment to enterprise customers
- 🚀 **Q4 Product Launch** depends on reliable infrastructure
- 👥 **50,000+ users** rely on platform availability

---

## 🎯 Learning Objectives

By completing this lab, you will master:

1. **HA Control Plane Architecture** - Design multi-master cluster topology
2. **Load Balancer Configuration** - Set up control plane endpoint load balancing
3. **etcd Cluster Management** - Configure external etcd cluster for HA
4. **Failover Testing** - Validate automatic failover capabilities
5. **Backup/Restore Procedures** - Implement etcd backup and recovery strategies

---

## 🔧 Lab Tasks

### **Task 1: Infrastructure Planning & Load Balancer Setup (8 minutes)**
Design and implement the load balancer infrastructure required for HA control plane.

**Scenario Details:**
- **Infrastructure:** 3 control plane nodes, 1 load balancer node
- **Requirements:** HAProxy load balancer for API server traffic
- **Network:** Control plane endpoint must be highly available
- **Ports:** 6443 (API server), 2379-2380 (etcd), 10250-10252 (kubelet/controller)

**Your Actions:**
1. Set up HAProxy load balancer configuration
2. Configure health checks for API server endpoints
3. Verify load balancer connectivity to all control plane nodes
4. Test load balancer failover behavior

**Expected CKA Skills:**
- Load balancer configuration for Kubernetes
- Understanding control plane communication ports
- Network planning for HA clusters
- Health check implementation

**Success Criteria:**
- [ ] HAProxy configured and running
- [ ] Health checks functional for all control plane nodes
- [ ] Load balancer endpoint accessible
- [ ] Round-robin traffic distribution verified

---

### **Task 2: First Control Plane Node Bootstrap (7 minutes)**
Initialize the first control plane node with HA configuration.

**Scenario Details:**
- **Node:** control-plane-1 (primary bootstrap node)
- **Configuration:** External etcd, load balancer endpoint
- **Requirements:** Proper certificate configuration for HA
- **Network:** Pod and service CIDR configuration

**Your Actions:**
1. Initialize cluster with HA-specific kubeadm configuration
2. Configure API server for load balancer endpoint
3. Set up etcd cluster configuration
4. Verify initial control plane functionality

**Expected CKA Skills:**
- kubeadm init with HA configuration
- Certificate management for multi-master
- etcd cluster initialization
- API server configuration

**Success Criteria:**
- [ ] First control plane node successfully initialized
- [ ] etcd cluster started and healthy
- [ ] API server accessible via load balancer
- [ ] Admin kubeconfig configured

---

### **Task 3: Additional Control Plane Nodes (8 minutes)**
Join additional control plane nodes to create true HA cluster.

**Scenario Details:**
- **Nodes:** control-plane-2, control-plane-3
- **Process:** Join existing cluster as control plane nodes
- **Requirements:** Certificate distribution and etcd joining
- **Validation:** All control plane nodes operational

**Your Actions:**
1. Prepare additional control plane nodes
2. Execute control plane join process
3. Verify etcd cluster member status
4. Validate all API servers are operational

**Expected CKA Skills:**
- kubeadm join for control plane nodes
- Certificate sharing between control plane nodes
- etcd cluster member management
- Multi-master validation techniques

**Success Criteria:**
- [ ] 3 control plane nodes successfully joined
- [ ] etcd cluster has 3 healthy members
- [ ] All API servers responding through load balancer
- [ ] kubectl commands work from any control plane node

---

### **Task 4: Worker Node Integration (4 minutes)**
Join worker nodes to the HA cluster and verify cluster functionality.

**Scenario Details:**
- **Nodes:** worker-1, worker-2
- **Connection:** Must connect via load balancer endpoint
- **Requirements:** Proper kubelet configuration for HA
- **Validation:** Full cluster operational status

**Your Actions:**
1. Join worker nodes using load balancer endpoint
2. Verify worker node connectivity to all control plane nodes
3. Deploy test workloads to validate cluster functionality
4. Confirm pod scheduling across all nodes

**Expected CKA Skills:**
- Worker node joining to HA cluster
- Kubelet configuration for multiple API servers
- Cluster functionality validation
- Workload deployment and scheduling

**Success Criteria:**
- [ ] Worker nodes successfully joined cluster
- [ ] Nodes show Ready status
- [ ] Test pods can be scheduled and run
- [ ] Service networking functional

---

### **Task 5: Failover Testing & Validation (8 minutes)**
Test the HA cluster's resilience by simulating control plane node failures.

**Scenario Details:**
- **Test Scenarios:** Single node failure, API server restart, etcd member failure
- **Requirements:** Cluster must remain operational during tests
- **Validation:** API availability, etcd consensus, workload continuity
- **Recovery:** Verify automatic recovery procedures

**Your Actions:**
1. Test single control plane node failure
2. Verify API server failover through load balancer
3. Test etcd cluster resilience
4. Validate workload continuity during failures

**Expected CKA Skills:**
- HA cluster testing methodologies
- Understanding failover mechanisms
- etcd cluster behavior during failures
- Troubleshooting HA cluster issues

**Success Criteria:**
- [ ] Cluster survives single control plane node failure
- [ ] API server remains accessible during node failure
- [ ] etcd maintains quorum with node failures
- [ ] Workloads continue running during tests

---

### **Task 6: Backup & Recovery Procedures (5 minutes)**
Implement etcd backup and restoration procedures for disaster recovery.

**Scenario Details:**
- **Backup:** Create comprehensive etcd cluster backup
- **Testing:** Simulate disaster recovery scenario
- **Requirements:** Backup automation and validation
- **Documentation:** Operational procedures for production use

**Your Actions:**
1. Create etcd cluster backup
2. Test backup restoration process
3. Verify cluster state after restoration
4. Document backup/recovery procedures

**Expected CKA Skills:**
- etcd backup using etcdctl
- Cluster restoration procedures
- Backup validation and testing
- Disaster recovery planning

**Success Criteria:**
- [ ] Successful etcd backup created
- [ ] Backup restoration process tested
- [ ] Cluster fully operational after restore
- [ ] Backup procedures documented

---

## 🏗️ Environment Setup

```bash
# HA Control Plane Lab Environment Setup
# Note: In real exam, infrastructure will be pre-configured

# Node Configuration (5 nodes total)
# Load Balancer: 192.168.56.10 (haproxy)
# Control Plane 1: 192.168.56.11 (control-plane-1)
# Control Plane 2: 192.168.56.12 (control-plane-2)  
# Control Plane 3: 192.168.56.13 (control-plane-3)
# Worker 1: 192.168.56.21 (worker-1)
# Worker 2: 192.168.56.22 (worker-2)

# HAProxy Configuration Template
cat > /etc/haproxy/haproxy.cfg << EOF
global
    log         127.0.0.1 local0
    chroot      /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user        haproxy
    group       haproxy
    daemon

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option                  http-server-close
    option                  forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

frontend k8s-api
    bind *:6443
    mode tcp
    default_backend k8s-api-backend

backend k8s-api-backend
    mode tcp
    balance roundrobin
    option tcp-check
    server control-plane-1 192.168.56.11:6443 check fall 3 rise 2
    server control-plane-2 192.168.56.12:6443 check fall 3 rise 2  
    server control-plane-3 192.168.56.13:6443 check fall 3 rise 2
EOF

# Kubeadm HA Configuration Template
cat > kubeadm-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.30.0
controlPlaneEndpoint: "192.168.56.10:6443"
networking:
  podSubnet: "192.168.0.0/16"
  serviceSubnet: "10.96.0.0/12"
etcd:
  external:
    endpoints:
    - https://192.168.56.11:2379
    - https://192.168.56.12:2379
    - https://192.168.56.13:2379
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "192.168.56.11"
  bindPort: 6443
EOF
```

---

## 📚 Key Commands Reference

### HAProxy Management
```bash
# Start/stop HAProxy
sudo systemctl start haproxy
sudo systemctl enable haproxy
sudo systemctl status haproxy

# Test configuration
sudo haproxy -f /etc/haproxy/haproxy.cfg -c

# Monitor connections
sudo netstat -tlnp | grep :6443
```

### HA Cluster Bootstrap
```bash
# Initialize first control plane node
sudo kubeadm init --config=kubeadm-config.yaml --upload-certs

# Join additional control plane nodes
sudo kubeadm join 192.168.56.10:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <cert-key>

# Join worker nodes
sudo kubeadm join 192.168.56.10:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

### HA Cluster Validation
```bash
# Check cluster status
kubectl get nodes -o wide
kubectl get pods -n kube-system

# Verify etcd cluster
kubectl get pods -n kube-system | grep etcd
sudo etcdctl member list --endpoints=https://192.168.56.11:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Test API server endpoints
curl -k https://192.168.56.11:6443/version
curl -k https://192.168.56.12:6443/version
curl -k https://192.168.56.13:6443/version
```

### Backup and Recovery
```bash
# Create etcd backup
sudo etcdctl snapshot save backup.db \
  --endpoints=https://192.168.56.11:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
sudo etcdctl snapshot status backup.db --write-out=table

# Restore from backup (emergency procedure)
sudo etcdctl snapshot restore backup.db \
  --name control-plane-1 \
  --initial-cluster control-plane-1=https://192.168.56.11:2380 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-advertise-peer-urls https://192.168.56.11:2380
```

---

## 🔍 Troubleshooting Guide

### Common Issues

**Control Plane Join Failures**
```bash
# Check certificate-key validity
sudo kubeadm token list
sudo kubeadm init phase upload-certs --upload-certs

# Verify load balancer connectivity
telnet 192.168.56.10 6443
curl -k https://192.168.56.10:6443/version

# Common causes:
# - Expired certificate keys
# - Load balancer misconfiguration
# - Firewall blocking required ports
# - Incorrect kubeadm configuration
```

**etcd Cluster Issues**
```bash
# Check etcd member status
sudo etcdctl member list --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify etcd cluster health
sudo etcdctl endpoint health --cluster \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Check etcd logs
sudo journalctl -u etcd -f
```

**Load Balancer Problems**
```bash
# Test HAProxy configuration
sudo haproxy -f /etc/haproxy/haproxy.cfg -c

# Check backend server status
echo "show stat" | socat stdio tcp4-connect:127.0.0.1:9999

# Verify port binding
sudo netstat -tlnp | grep :6443
```

---

## 🎯 Success Validation

### Final Verification Checklist

**HA Infrastructure:**
- [ ] 3 control plane nodes running and healthy
- [ ] Load balancer distributing traffic correctly
- [ ] etcd cluster with 3 healthy members
- [ ] All API servers accessible and responsive

**Failover Testing:**
- [ ] Cluster survives single control plane node failure
- [ ] API remains accessible during node failures
- [ ] etcd maintains quorum with member failures
- [ ] Workloads continue operating during failures

**Backup/Recovery:**
- [ ] etcd backup created successfully
- [ ] Backup restoration process tested
- [ ] Cluster fully operational after restore
- [ ] Recovery procedures documented

**Production Readiness:**
- [ ] All certificates properly configured
- [ ] Network policies allowing required traffic
- [ ] Monitoring and alerting configured
- [ ] Operational procedures documented

---

## 🚀 Real-World Applications

### Production Use Cases

**Enterprise Environments:**
- Multi-region control plane deployment
- Integration with enterprise load balancers (F5, NetScaler)
- Certificate management with enterprise PKI
- Automated backup and disaster recovery

**Cloud Native Platforms:**
- Managed Kubernetes service architecture
- Auto-scaling control plane components
- Integration with cloud load balancers
- Cross-availability zone deployment

**Financial Services:**
- Regulatory compliance requirements
- Zero-downtime maintenance procedures
- Audit trail and compliance reporting
- Business continuity planning

### Key Takeaways

1. **Eliminate Single Points of Failure** = Business continuity
2. **Load Balancer Configuration** = Critical for HA success
3. **etcd Cluster Management** = Foundation of cluster state
4. **Testing Procedures** = Validate HA before production
5. **Backup/Recovery** = Essential for disaster preparedness

---

## 📈 Next Steps

**After This Lab:**
- Practice A04: Operator Deployment Pipeline
- Study advanced etcd operations
- Explore cluster monitoring and alerting
- Review production HA best practices

**Related CKA Topics:**
- Cluster upgrade procedures
- Certificate management and rotation
- Troubleshooting cluster components
- Performance optimization

---

*💡 **Pro Tip:** In production HA setups, always test failover scenarios regularly. A monthly "chaos engineering" exercise can help identify weaknesses before they cause real outages!*
