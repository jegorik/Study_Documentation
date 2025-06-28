# T05 - Certificate Expiry Emergency

> **📋 Lab Category:** Troubleshooting  
> **⏱️ Estimated Time:** 25-30 minutes  
> **🎯 Difficulty:** Advanced  
> **🔗 Exam Objectives:** Troubleshooting, Cluster Maintenance, Security

---

## 🎬 Real-World Scenario

### Background Context
You're the Senior DevOps Engineer at **TechCorp**, a fintech company processing $100M+ transactions daily. At 3:47 AM, monitoring alerts flood your phone - the entire Kubernetes cluster is failing authentication. API server calls are timing out, kubectl commands fail with certificate errors, and nodes are showing "NotReady" status.

**Your Role:** Senior DevOps Engineer (On-Call)  
**Situation:** Critical production outage - certificate expiry cascade failure  
**Urgency:** Every minute of downtime costs $50,000 in lost transactions  

### The Challenge
Multiple certificates have expired simultaneously, causing a complete cluster breakdown. You must diagnose and restore certificate chain integrity under extreme time pressure.

---

## 🎯 Learning Objectives

By completing this lab, you will:
- [ ] **Primary Skill:** Certificate troubleshooting and renewal procedures
- [ ] **Secondary Skills:** Cluster recovery, API server debugging, etcd access
- [ ] **Real-world Application:** Production certificate management strategies
- [ ] **Exam Relevance:** Certificate troubleshooting (frequent CKA exam topic)

---

## 🏗️ Lab Environment Setup

### Initial Cluster State
```bash
# BROKEN - Multiple certificate failures
SYMPTOMS:
- kubectl commands fail: "Unable to connect to server: x509: certificate has expired"
- Nodes show NotReady status
- API server returning 401 Unauthorized
- etcd communication failing
- kubelet unable to register with API server

AFFECTED CERTIFICATES:
- API server serving certificate (EXPIRED)
- kubelet client certificate (EXPIRED) 
- etcd peer certificate (EXPIRED)
- Front-proxy client certificate (EXPIRED)
```

---

## 📋 Tasks Overview

| Task | Objective | Est. Time | Difficulty |
|------|-----------|-----------|------------|
| 1 | Emergency Certificate Assessment | 5 min | Advanced |
| 2 | API Server Certificate Recovery | 8 min | Expert |
| 3 | Node Certificate Renewal | 7 min | Advanced |
| 4 | etcd Certificate Restoration | 3 min | Advanced |
| 5 | Cluster Health Validation | 2 min | Intermediate |

---

## 🔍 Task 1: Emergency Certificate Assessment (5 minutes)

### Objective
Rapidly diagnose which certificates have expired and understand the cascade failure impact.

### Your Mission
Identify expired certificates and assess the scope of the certificate crisis.

### Step-by-Step Instructions

#### 1.1 Direct Certificate Analysis
```bash
# Check API server certificate (bypassing kubectl)
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -A 2 "Not After"

# Check kubelet certificates
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout | grep -A 2 "Not After"

# Check etcd certificates
sudo openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout | grep -A 2 "Not After"

# Check all PKI certificates at once
sudo find /etc/kubernetes/pki -name "*.crt" -exec openssl x509 -in {} -text -noout \; -exec echo "File: {}" \; | grep -E "(File:|Not After)"
```

#### 1.2 Service Status Assessment
```bash
# Check systemd service status
sudo systemctl status kubelet
sudo systemctl status containerd

# Check container runtime connectivity
sudo crictl ps 2>/dev/null || echo "Container runtime connection failed"

# Attempt direct etcd access
sudo ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key --cacert=/etc/kubernetes/pki/etcd/ca.crt endpoint health
```

---

## 🔧 Task 2: API Server Certificate Recovery (8 minutes)

### Objective
Restore API server functionality by renewing expired certificates and restarting critical services.

### Your Mission
Renew API server certificates and restore cluster API access.

### Step-by-Step Instructions

#### 2.1 Backup Current Certificates
```bash
# Create backup directory
sudo mkdir -p /root/cert-backup-$(date +%Y%m%d-%H%M%S)
sudo cp -r /etc/kubernetes/pki /root/cert-backup-$(date +%Y%m%d-%H%M%S)/

# Document current certificate states
sudo find /etc/kubernetes/pki -name "*.crt" -exec sh -c 'echo "=== $1 ==="; openssl x509 -in "$1" -text -noout | grep -A 2 "Not After"' _ {} \;
```

#### 2.2 Renew API Server Certificates
```bash
# Renew API server certificate
sudo kubeadm certs renew apiserver

# Renew API server kubelet client certificate
sudo kubeadm certs renew apiserver-kubelet-client

# Renew front-proxy client certificate
sudo kubeadm certs renew front-proxy-client

# Verify renewals
sudo kubeadm certs check-expiration
```

#### 2.3 Restart API Server
```bash
# Move API server manifest temporarily (force restart)
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sleep 10
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

# Monitor API server startup
sudo crictl ps | grep kube-apiserver
kubectl get nodes 2>/dev/null || echo "API server still starting..."
```

---

## 🖥️ Task 3: Node Certificate Renewal (7 minutes)

### Objective
Restore kubelet certificates and bring nodes back to Ready status.

### Your Mission
Renew kubelet certificates and restore node connectivity.

### Step-by-Step Instructions

#### 3.1 Kubelet Certificate Renewal
```bash
# Renew kubelet serving certificate
sudo kubeadm certs renew apiserver-kubelet-client

# Check kubelet configuration
sudo cat /var/lib/kubelet/config.yaml | grep -E "(clientCAFile|tlsCertFile|tlsPrivateKeyFile)"

# Restart kubelet service
sudo systemctl restart kubelet
sudo systemctl status kubelet
```

#### 3.2 Bootstrap Token Recovery (if needed)
```bash
# Create new bootstrap token
sudo kubeadm token create --print-join-command

# Check if kubelet auto-renewal is configured
sudo cat /var/lib/kubelet/config.yaml | grep rotateCertificates
```

#### 3.3 Verify Node Recovery
```bash
# Wait for node to become Ready
kubectl get nodes -w

# Check kubelet logs for certificate issues
sudo journalctl -u kubelet -f --since "5 minutes ago"
```

---

## 💾 Task 4: etcd Certificate Restoration (3 minutes)

### Objective
Restore etcd cluster communication by renewing peer and server certificates.

### Your Mission
Renew etcd certificates and verify cluster health.

### Step-by-Step Instructions

#### 4.1 Renew etcd Certificates
```bash
# Renew etcd server certificate
sudo kubeadm certs renew etcd-server

# Renew etcd peer certificate
sudo kubeadm certs renew etcd-peer

# Renew etcd healthcheck client certificate
sudo kubeadm certs renew etcd-healthcheck-client
```

#### 4.2 Restart etcd and Verify
```bash
# Restart etcd (via manifest manipulation)
sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/
sleep 5
sudo mv /tmp/etcd.yaml /etc/kubernetes/manifests/

# Verify etcd health
sleep 10
sudo ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key --cacert=/etc/kubernetes/pki/etcd/ca.crt endpoint health
```

---

## ✅ Task 5: Cluster Health Validation (2 minutes)

### Objective
Confirm complete cluster recovery and implement monitoring for future certificate expirations.

### Your Mission
Validate all cluster components are healthy and set up certificate monitoring.

### Step-by-Step Instructions

#### 5.1 Comprehensive Health Check
```bash
# Check all nodes are Ready
kubectl get nodes

# Verify system pods are running
kubectl get pods -n kube-system

# Test basic cluster functionality
kubectl create deployment test-nginx --image=nginx --replicas=2
kubectl get pods -l app=test-nginx
kubectl delete deployment test-nginx
```

#### 5.2 Certificate Expiration Monitoring Setup
```bash
# Check all certificate expiration dates
sudo kubeadm certs check-expiration

# Create monitoring script for future use
cat << 'EOF' | sudo tee /usr/local/bin/check-k8s-certs.sh
#!/bin/bash
echo "=== Kubernetes Certificate Expiration Check ==="
kubeadm certs check-expiration
echo "=== Next check recommended: $(date -d '+30 days')==="
EOF

sudo chmod +x /usr/local/bin/check-k8s-certs.sh

# Test the monitoring script
sudo /usr/local/bin/check-k8s-certs.sh
```

---

## 🎯 Expected Outcomes

Upon completion, you should have:
- ✅ All expired certificates renewed
- ✅ API server responding to kubectl commands
- ✅ All nodes in Ready status
- ✅ etcd cluster healthy and accessible
- ✅ Certificate monitoring script deployed
- ✅ Understanding of certificate interdependencies

---

## 🚨 Emergency Troubleshooting Guide

### Common Issues & Quick Fixes

**Issue:** kubectl still failing after certificate renewal
```bash
# Solution: Update kubeconfig
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

**Issue:** etcd still showing unhealthy
```bash
# Solution: Restart all control plane components
sudo mv /etc/kubernetes/manifests/*.yaml /tmp/
sleep 10
sudo mv /tmp/*.yaml /etc/kubernetes/manifests/
```

**Issue:** Nodes still NotReady after kubelet restart
```bash
# Solution: Check for CNI issues
kubectl get pods -n kube-system | grep -E "(calico|flannel|weave)"
```

---

## 📚 Knowledge Check

### Critical Understanding Questions
1. **Certificate Dependencies:** Which certificates must be valid for kubectl to work?
2. **Recovery Order:** Why is API server certificate renewal the first priority?
3. **Automation:** How would you prevent this emergency in production?

### Advanced Scenarios
- What if the CA certificate itself had expired?
- How would you handle this in a multi-master cluster?
- What's the impact of certificate rotation on running workloads?

---

## 🔄 Production Prevention Strategy

### Implement Certificate Lifecycle Management
```bash
# Add to crontab for automated renewal (30 days before expiry)
0 2 1 * * /usr/local/bin/check-k8s-certs.sh | mail -s "K8s Certificate Status" admin@company.com

# Consider cert-manager for automated certificate management
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
```

---

*⚠️ "Certificate expiry is the #1 cause of Kubernetes production outages. Master this scenario to prevent costly downtime!" - CKA Success Tip*
