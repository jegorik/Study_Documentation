# A01 Lab Setup - Bare Metal Cluster Bootstrap Test Environment

This file contains setup instructions and scripts for creating the infrastructure needed for Lab A01.

## 🚀 Environment Options

### Option 1: Local VMs (Recommended for Learning)

```bash
# Using Vagrant for local VMs
cat > Vagrantfile << 'EOF'
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  
  # Control plane node
  config.vm.define "control-plane" do |cp|
    cp.vm.hostname = "control-plane"
    cp.vm.network "private_network", ip: "192.168.56.10"
    cp.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end
  
  # Worker nodes
  (1..2).each do |i|
    config.vm.define "worker-#{i}" do |worker|
      worker.vm.hostname = "worker-#{i}"
      worker.vm.network "private_network", ip: "192.168.56.#{10+i}"
      worker.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
      end
    end
  end
end
EOF

# Start the VMs
vagrant up

# SSH access
vagrant ssh control-plane
vagrant ssh worker-1
vagrant ssh worker-2
```

### Option 2: Cloud VMs (AWS Example)

```bash
# Create security group
aws ec2 create-security-group \
    --group-name k8s-cluster-sg \
    --description "Kubernetes cluster security group"

# Add required rules
aws ec2 authorize-security-group-ingress \
    --group-name k8s-cluster-sg \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-name k8s-cluster-sg \
    --protocol tcp \
    --port 6443 \
    --source-group k8s-cluster-sg

aws ec2 authorize-security-group-ingress \
    --group-name k8s-cluster-sg \
    --protocol tcp \
    --port 2379-2380 \
    --source-group k8s-cluster-sg

aws ec2 authorize-security-group-ingress \
    --group-name k8s-cluster-sg \
    --protocol tcp \
    --port 10250-10252 \
    --source-group k8s-cluster-sg

# Launch instances
aws ec2 run-instances \
    --image-id ami-0e472ba40eb589f49 \
    --count 3 \
    --instance-type t3.medium \
    --key-name your-key-pair \
    --security-groups k8s-cluster-sg \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=k8s-cluster}]'
```

### Option 3: Container-based Testing (kind)

```bash
# Alternative: Use kind for quick testing (not true bare metal but good for command practice)
cat > kind-config.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind-config.yaml --name bootstrap-lab
```

## 🔧 Pre-Lab Setup Script

### Automated Infrastructure Preparation

```bash
#!/bin/bash
# save as: setup-node.sh
# Run this on each node before starting the lab

set -e

echo "=== Kubernetes Node Setup Script ==="
echo "This script prepares Ubuntu 22.04 for Kubernetes cluster"

# Update system
echo "Updating system packages..."
sudo apt update && sudo apt upgrade -y

# Disable swap
echo "Disabling swap..."
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load kernel modules
echo "Loading required kernel modules..."
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl
echo "Configuring sysctl parameters..."
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

# Install containerd
echo "Installing containerd..."
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y containerd.io

# Configure containerd
echo "Configuring containerd..."
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install Kubernetes tools
echo "Installing kubeadm, kubelet, kubectl..."
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

echo "=== Node setup complete! ==="
echo "Services status:"
sudo systemctl status containerd --no-pager -l
sudo systemctl status kubelet --no-pager -l
echo "Ready for kubeadm cluster initialization!"
```

## 🎯 Lab Exercise Environment

### Quick Verification Checklist

```bash
# Run on each node before starting lab
echo "=== Pre-Lab Verification ==="

# Check swap is disabled
echo "Swap status:"
free -h | grep -i swap

# Check required services
echo "Service status:"
sudo systemctl is-active containerd
sudo systemctl is-active kubelet

# Check kernel modules
echo "Kernel modules:"
lsmod | grep br_netfilter
lsmod | grep overlay

# Check network connectivity
echo "Network test (replace IPs with your actual nodes):"
ping -c 2 192.168.56.10  # control plane
ping -c 2 192.168.56.11  # worker 1
ping -c 2 192.168.56.12  # worker 2

echo "=== Ready to start A01 lab! ==="
```

### Lab Environment Variables

```bash
# Set these on each node for consistency
export CONTROL_PLANE_IP="192.168.56.10"
export WORKER1_IP="192.168.56.11"
export WORKER2_IP="192.168.56.12"
export POD_CIDR="192.168.0.0/16"
export K8S_VERSION="1.30.0"
```

## 🧪 Alternative Practice Scenarios

### Scenario A: Fresh Installation

- Start with clean Ubuntu 22.04 systems
- Follow complete setup process
- Practice troubleshooting common issues

### Scenario B: Cluster Expansion

- Start with single-node cluster
- Practice adding worker nodes
- Verify scheduling across all nodes

### Scenario C: Version-specific Setup

- Practice with specific Kubernetes versions
- Test compatibility between components
- Handle version upgrade scenarios

## 🔧 Common Setup Issues & Fixes

### Issue 1: Container Runtime Not Ready

```bash
# Symptoms: kubelet fails to start
# Fix: Check containerd configuration
sudo systemctl status containerd
sudo journalctl -u containerd

# Ensure SystemdCgroup is enabled
grep SystemdCgroup /etc/containerd/config.toml
sudo systemctl restart containerd
```

### Issue 2: Network Connectivity Problems

```bash
# Symptoms: kubeadm join fails with connection timeout
# Fix: Check firewall and network configuration
sudo ufw status
sudo netstat -tlnp | grep :6443

# Ensure proper routing
ip route show
```

### Issue 3: Token Expiration

```bash
# Symptoms: "token not found" during join
# Fix: Generate new token on control plane
sudo kubeadm token create --print-join-command
```

## 🧹 Cleanup Instructions

### Complete Environment Cleanup

```bash
# Reset Kubernetes cluster
sudo kubeadm reset -f

# Clean up containerd
sudo systemctl stop containerd
sudo rm -rf /var/lib/containerd/*
sudo systemctl start containerd

# Remove packages (optional)
sudo apt remove -y kubeadm kubectl kubelet
sudo apt autoremove -y

# Vagrant cleanup (if using Vagrant)
vagrant destroy -f
```

### Partial Reset (Keep Infrastructure)

```bash
# Just reset cluster configuration
sudo kubeadm reset -f
sudo rm -rf $HOME/.kube/config
sudo systemctl restart kubelet
```

## 📝 Lab Validation Notes

This setup environment provides:

- ✅ Multiple infrastructure options (VMs, cloud, containers)
- ✅ Automated setup scripts for quick preparation
- ✅ Comprehensive verification procedures
- ✅ Troubleshooting guidance for common issues
- ✅ Clean environment reset capabilities

Perfect for practicing A01 cluster bootstrap scenarios and validating kubeadm expertise!
