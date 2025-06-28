# Lab A01: Bare Metal Cluster Bootstrap

> **📋 Lab Category:** Cluster Architecture, Installation & Configuration  
> **⏱️ Estimated Time:** 20 minutes  
> **🎯 Difficulty:** Intermediate  
> **🔗 Exam Objectives:** Prepare underlying infrastructure, Create and manage Kubernetes clusters using kubeadm, Manage the lifecycle of Kubernetes clusters

---

## 🎬 Real-World Scenario

### Background Context
You're a Senior Platform Engineer at **CloudNative Dynamics**, a fast-growing fintech startup that's outgrown their managed Kubernetes service costs. The company is spending $45,000/month on managed clusters and wants to move to self-managed infrastructure to reduce costs by 70% while gaining more control.

**Your Role:** Lead Platform Engineer responsible for infrastructure architecture  
**Situation:** Management approved the migration to bare metal Kubernetes clusters after a successful cost-benefit analysis  
**Urgency:** **HIGH PRIORITY** - Need production-ready cluster in 2 weeks for new product launch, development teams are blocked

### The Challenge
You've been tasked with setting up the company's first self-managed Kubernetes cluster on bare metal servers. This will become the template for all future production clusters. The infrastructure team has already provisioned three Ubuntu 22.04 servers, and you need to bootstrap a working Kubernetes cluster using kubeadm.

**Critical Business Requirements:**
- 💰 **Cost Savings:** Target 70% reduction in infrastructure costs
- 🚀 **Performance:** Better control over compute resources and networking  
- 🔒 **Security:** Full control over cluster configuration and compliance
- ⚡ **Timeline:** Functional cluster needed within 2 days for dev team testing

---

## 🎯 Learning Objectives

By completing this lab, you will:
- [ ] **Primary Skill:** Master kubeadm cluster initialization and worker node joining process
- [ ] **Secondary Skills:** Infrastructure preparation, container runtime setup, networking configuration
- [ ] **Real-world Application:** Build production-ready Kubernetes clusters from scratch
- [ ] **Exam Relevance:** Core CKA cluster management skills - kubeadm, networking, node management

---

## 🔧 Prerequisites

### Knowledge Requirements
- [ ] Basic understanding of Linux system administration
- [ ] Familiarity with container concepts and Docker/containerd
- [ ] Understanding of Kubernetes architecture (control plane, worker nodes)
- [ ] Basic networking concepts (IP addressing, routing)

### Environment Setup
```bash
# Infrastructure requirements
- 3 Ubuntu 22.04 servers (can be VMs)
  - 1 control plane node: 2 CPU, 4GB RAM, 20GB disk
  - 2 worker nodes: 2 CPU, 4GB RAM, 20GB disk
- Network connectivity between all nodes
- SSH access to all nodes with sudo privileges

# Verification commands
ssh user@control-plane-node "sudo ls -la"
ssh user@worker-node-1 "sudo ls -la"  
ssh user@worker-node-2 "sudo ls -la"
```

---

## 📚 Quick Reference

### Key Commands for This Lab
```bash
kubeadm init --pod-network-cidr=192.168.0.0/16  # Initialize control plane
kubeadm join <endpoint> --token <token>         # Join worker nodes
kubectl apply -f <cni-manifest>                 # Install CNI plugin
kubectl get nodes                               # Verify cluster status
```

### Important Concepts
- **kubeadm:** Tool for bootstrapping Kubernetes clusters following best practices
- **Container Runtime:** containerd/Docker - runs containers on each node
- **CNI Plugin:** Container Network Interface - enables pod networking
- **Control Plane:** API server, etcd, scheduler, controller manager components

---

## 🚀 Lab Tasks

### Task 1: Infrastructure Preparation and Container Runtime Setup
**Objective:** Prepare all three nodes with required dependencies and container runtime

**Your Mission:**
Set up the foundation for your Kubernetes cluster by installing and configuring the container runtime (containerd), disabling swap, configuring kernel modules, and installing kubeadm, kubelet, and kubectl on all three nodes.

**Expected Outcome:**
All three nodes have containerd running, swap disabled, required kernel modules loaded, and Kubernetes tools installed and ready for cluster initialization.

**Hints:**
- 💡 Swap must be completely disabled on all nodes for kubelet to work properly
- 💡 Certain kernel modules like br_netfilter are required for networking
- 💡 Use the official Kubernetes apt repository for the most current packages

---

### Task 2: Control Plane Initialization
**Objective:** Initialize the Kubernetes control plane using kubeadm

**Your Mission:**
Use kubeadm to initialize the control plane node, configure kubectl for the admin user, and prepare the join command for worker nodes. Choose appropriate pod network CIDR that won't conflict with your infrastructure.

**Expected Outcome:**
Control plane is running with all components healthy, kubectl is configured for cluster access, and you have the join command ready for worker nodes.

**Hints:**
- 💡 Save the kubeadm join command output - you'll need it for worker nodes
- 💡 Configure kubectl immediately after initialization to verify the control plane
- 💡 The pod network CIDR should not overlap with your node network

---

### Task 3: Container Network Interface (CNI) Installation
**Objective:** Install and configure pod networking using a CNI plugin

**Your Mission:**
Install a CNI plugin (Calico recommended) to enable pod-to-pod communication across the cluster. Verify that the control plane node becomes Ready after networking is configured.

**Expected Outcome:**
Pod networking is functional, control plane node shows Ready status, and kube-system pods are all running successfully.

**Hints:**
- 💡 The control plane node will remain NotReady until CNI is installed
- 💡 Calico is a popular choice that works well with kubeadm
- 💡 Watch the kube-system pods to ensure they all start successfully

---

### Task 4: Worker Node Joining and Cluster Verification
**Objective:** Join both worker nodes to the cluster and verify full cluster functionality

**Your Mission:**
Use the kubeadm join command on both worker nodes to add them to the cluster. Verify that all nodes are Ready, all system pods are running, and the cluster can schedule workloads across all nodes.

**Expected Outcome:**
Three-node cluster with all nodes in Ready state, system pods distributed across nodes, and ability to schedule test workloads successfully.

**Hints:**
- 💡 Run the exact join command from the kubeadm init output on each worker
- 💡 Worker nodes may take a few minutes to show Ready status
- 💡 Test with a simple deployment to verify scheduling works across nodes

---

## ⏰ Time Management

**Exam Pace Training:**
- [ ] Task 1: 6 minutes (Infrastructure prep)
- [ ] Task 2: 5 minutes (Control plane init)
- [ ] Task 3: 4 minutes (CNI installation)
- [ ] Task 4: 5 minutes (Worker join & verify)
- [ ] **Total Target:** 20 minutes

*⚠️ If taking longer than target time, move to solution section and understand the efficient approach*

---

## 🔧 Step-by-Step Solution

<details>
<summary><strong>📖 Click to reveal detailed solution (try solving first!)</strong></summary>

### Step 1: Infrastructure Preparation (Run on ALL nodes)
**Command/Action:**
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Disable swap permanently
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

# Install containerd
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y containerd.io

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install kubeadm, kubelet, kubectl
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

**Explanation:**
This comprehensive setup prepares each node with all required components. Disabling swap is critical as kubelet won't start with swap enabled. The kernel modules and sysctl parameters enable proper networking functionality.

**Expected Output:**
```
containerd.service is active and running
kubelet.service is active and running
All packages installed successfully
```

---

### Step 2: Control Plane Initialization (Control plane node only)
**Command/Action:**
```bash
# Initialize the control plane
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --control-plane-endpoint=$(hostname -I | awk '{print $1}')

# Configure kubectl for regular user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify control plane is running
kubectl cluster-info
kubectl get nodes
kubectl get pods -n kube-system
```

**Explanation:**
kubeadm init sets up the control plane components (API server, etcd, scheduler, controller manager). The pod network CIDR defines the IP range for pod networking. Copying the admin config enables kubectl access.

**Expected Output:**
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.10:6443 --token abcd12.1234567890abcdef \
    --discovery-token-ca-cert-hash sha256:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
```

---

### Step 3: CNI Installation (Control plane node)
**Command/Action:**
```bash
# Install Calico CNI
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml

# Wait for calico pods to be ready
kubectl wait --for=condition=ready pod -l k8s-app=calico-node -n kube-system --timeout=300s

# Verify control plane node becomes Ready
kubectl get nodes
kubectl get pods -n kube-system
```

**Explanation:**
Calico provides pod-to-pod networking across the cluster. Without a CNI plugin, nodes remain NotReady and pods cannot communicate. Calico is production-ready and works well with kubeadm.

**Expected Output:**
```
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created

NAME             STATUS   ROLES           AGE   VERSION
control-plane    Ready    control-plane   5m    v1.30.0
```

---

### Step 4: Worker Node Joining (Worker nodes)
**Command/Action:**
```bash
# On each worker node, run the join command from kubeadm init output
sudo kubeadm join 10.0.0.10:6443 --token abcd12.1234567890abcdef \
    --discovery-token-ca-cert-hash sha256:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef

# From control plane, verify all nodes joined
kubectl get nodes
kubectl get pods -n kube-system -o wide

# Test cluster functionality with a sample deployment
kubectl create deployment nginx --image=nginx:1.21
kubectl scale deployment nginx --replicas=3
kubectl get pods -o wide
kubectl expose deployment nginx --port=80 --type=ClusterIP
kubectl get services
```

**Explanation:**
The join command connects worker nodes to the control plane. Each worker gets the necessary certificates and joins the cluster. Testing with a deployment verifies that scheduling works across all nodes.

**Expected Output:**
```
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

NAME             STATUS   ROLES           AGE   VERSION
control-plane    Ready    control-plane   10m   v1.30.0
worker-node-1    Ready    <none>          2m    v1.30.0
worker-node-2    Ready    <none>          1m    v1.30.0

nginx-deployment-xyz   1/1     Running   0          30s   192.168.1.10   worker-node-1
nginx-deployment-abc   1/1     Running   0          30s   192.168.1.11   worker-node-2
nginx-deployment-def   1/1     Running   0          30s   192.168.1.12   control-plane
```

**Success Indicators:**
- [ ] All nodes show Ready status
- [ ] All kube-system pods are running
- [ ] Test deployment pods are distributed across nodes
- [ ] kubectl commands work from control plane

</details>

---

## 🎓 Knowledge Check

### Understanding Questions
1. **kubeadm vs Manual Setup:** Why is kubeadm preferred over manual component installation for production clusters?
2. **CNI Requirement:** Why do nodes remain NotReady until a CNI plugin is installed?
3. **Token Security:** How long are kubeadm join tokens valid, and how would you regenerate them?
4. **Container Runtime:** What's the difference between containerd and Docker for Kubernetes?

### Hands-On Challenges
- [ ] **Variation 1:** Bootstrap a cluster with different CNI (Flannel or Weave)
- [ ] **Variation 2:** Add a new worker node to existing cluster
- [ ] **Integration:** Configure cluster with specific Kubernetes version

---

## 🔍 Common Pitfalls & Troubleshooting

### Frequent Mistakes
1. **Swap Not Disabled**
   - **Symptom:** kubelet fails to start with swap enabled error
   - **Cause:** Kubernetes requires swap to be completely disabled
   - **Fix:** `sudo swapoff -a` and edit /etc/fstab to remove swap entries

2. **Incorrect Pod Network CIDR**
   - **Symptom:** Pod networking doesn't work or conflicts with node network
   - **Cause:** Pod CIDR overlaps with existing network infrastructure
   - **Fix:** Choose non-overlapping CIDR like 192.168.0.0/16 or 10.244.0.0/16

3. **Missing Container Runtime Configuration**
   - **Symptom:** Pods fail to start with runtime errors
   - **Cause:** containerd not properly configured for systemd cgroups
   - **Fix:** Enable SystemdCgroup in containerd config and restart service

### Debug Commands
```bash
# Essential debugging commands for cluster setup
sudo systemctl status kubelet                    # Check kubelet service
sudo journalctl -xeu kubelet                    # View kubelet logs
kubectl get componentstatuses                    # Check control plane health
kubectl describe node <node-name>               # Detailed node information
sudo kubeadm token list                         # View available tokens
```

---

## 🌟 Real-World Applications

### Enterprise Scenarios
- **Cost Optimization:** Moving from managed services to self-managed clusters
- **Compliance Requirements:** Full control over cluster configuration for regulatory needs
- **Performance Tuning:** Custom kernel parameters and networking configurations
- **Multi-Environment Setup:** Consistent cluster deployment across dev/staging/prod

### Best Practices Learned
- 🏆 **Infrastructure Preparation:** Always prepare all nodes before cluster initialization
- 🏆 **Version Consistency:** Use same Kubernetes version across all cluster components
- 🏆 **Network Planning:** Choose pod CIDR that doesn't conflict with infrastructure
- 🏆 **Security First:** Secure join tokens and configure RBAC early

---

## 📚 Additional Resources

### Related CKA Topics
- Prepare underlying infrastructure for installing a Kubernetes cluster
- Create and manage Kubernetes clusters using kubeadm
- Manage the lifecycle of Kubernetes clusters
- Understand extension interfaces (CNI, CSI, CRI)

### Extended Learning
- [ ] Explore different CNI plugins (Flannel, Weave, Cilium)
- [ ] Learn about kubeadm configuration files for advanced setups
- [ ] Study high availability control plane configurations

---

## 📝 Lab Completion

### Self-Assessment
- [ ] I can complete this lab within 20 minutes
- [ ] I understand each step of the cluster bootstrap process
- [ ] I can troubleshoot common kubeadm issues independently
- [ ] I can adapt this process for different infrastructure setups
- [ ] I'm confident with cluster lifecycle management

### Notes & Reflections
*Record insights about cluster architecture, networking decisions, or automation opportunities you identified during this lab.*

---

### 🏁 Lab Status
- [ ] **Started:** ___________
- [ ] **Completed:** ___________  
- [ ] **Reviewed:** ___________
- [ ] **Mastered:** ___________

**Time Taken:** _______ | **Target Time:** 20 minutes  
**Difficulty Rating:** ___/5 | **Confidence Level:** ___/5

---

*💡 **Next Lab Recommendation:** N01 - Multi-Tier App Connectivity (Networking foundations) or T02 - Network Communication Failure (Advanced troubleshooting)*
