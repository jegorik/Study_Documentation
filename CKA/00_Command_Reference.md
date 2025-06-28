# 🔧 CKA Command Reference

> **Quick kubectl commands for CKA exam preparation and daily Kubernetes administration**

## 📋 Table of Contents

- [Essential Shortcuts](#-essential-shortcuts)
- [Pod Management](#-pod-management)
- [Deployment & ReplicaSet](#-deployment--replicaset)
- [Services & Networking](#-services--networking)
- [Configuration Management](#️-configuration-management)
- [Storage & Volumes](#-storage--volumes)
- [Troubleshooting](#-troubleshooting)
- [Resource Management](#-resource-management)
- [Security & RBAC](#-security--rbac)
- [Exam-Specific Tips](#-exam-specific-tips)

---

## 🚀 Essential Shortcuts

### Quick Resource Creation

```bash
# Generate YAML without creating resource
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Create and save YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Apply from file
kubectl apply -f deployment.yaml

# Imperative commands (faster for exam)
kubectl create deployment nginx --image=nginx --replicas=3
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

### Context & Cluster Management

```bash
# List contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Current context
kubectl config current-context

# Set namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

### Common Options

| Option | Description | Example |
|--------|-------------|---------|
| `-o yaml` | Output in YAML format | `kubectl get pod nginx -o yaml` |
| `-o wide` | Show additional columns | `kubectl get pods -o wide` |
| `--dry-run=client` | Don't create, just validate | `kubectl apply -f pod.yaml --dry-run=client` |
| `--force` | Force delete/recreate | `kubectl delete pod nginx --force` |
| `-w` | Watch for changes | `kubectl get pods -w` |

---

## 🔄 Pod Management

### Pod Creation & Management

```bash
# Create pod (imperative)
kubectl run nginx --image=nginx

# Create pod with specific restart policy
kubectl run nginx --image=nginx --restart=Never

# Create pod with command
kubectl run nginx --image=nginx --command -- sleep 3600

# Create pod with labels
kubectl run nginx --image=nginx --labels="app=nginx,env=prod"

# Create pod with resource limits
kubectl run nginx --image=nginx --requests='cpu=100m,memory=128Mi' --limits='cpu=500m,memory=512Mi'
```

### Pod Operations

```bash
# List pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl get pods -l app=nginx

# Describe pod
kubectl describe pod nginx

# Get pod logs
kubectl logs nginx
kubectl logs nginx -f                    # Follow logs
kubectl logs nginx --previous           # Previous container logs
kubectl logs nginx -c container-name    # Specific container logs

# Execute commands in pod
kubectl exec nginx -- ls /
kubectl exec -it nginx -- /bin/bash

# Port forwarding
kubectl port-forward nginx 8080:80

# Copy files
kubectl cp nginx:/path/to/file ./local-file
kubectl cp ./local-file nginx:/path/to/file
```

### Pod Deletion

```bash
# Delete pod
kubectl delete pod nginx

# Force delete pod
kubectl delete pod nginx --force --grace-period=0

# Delete pods by label
kubectl delete pods -l app=nginx
```

---

## 📦 Deployment & ReplicaSet

### Deployment Management

```bash
# Create deployment
kubectl create deployment nginx --image=nginx --replicas=3

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Update deployment image
kubectl set image deployment/nginx nginx=nginx:1.21

# Annotate deployment (for rollout history)
kubectl annotate deployment nginx kubernetes.io/change-cause="Updated to nginx 1.21"
```

### Rolling Updates & Rollbacks

```bash
# Check rollout status
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx

# Rollback to previous version
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Pause/Resume rollout
kubectl rollout pause deployment/nginx
kubectl rollout resume deployment/nginx
```

### ReplicaSet Management

```bash
# List replicasets
kubectl get rs
kubectl get replicasets

# Describe replicaset
kubectl describe rs nginx-deployment-abc123

# Scale replicaset (not recommended, use deployment)
kubectl scale rs nginx-deployment-abc123 --replicas=4
```

---

## 🌐 Services & Networking

### Service Creation

```bash
# Expose deployment as ClusterIP (default)
kubectl expose deployment nginx --port=80

# Expose as NodePort
kubectl expose deployment nginx --port=80 --type=NodePort

# Expose as LoadBalancer
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Expose with specific target port
kubectl expose deployment nginx --port=80 --target-port=8080

# Create service imperatively
kubectl create service clusterip nginx --tcp=80:80
kubectl create service nodeport nginx --tcp=80:80
```

### Service Management

```bash
# List services
kubectl get services
kubectl get svc

# Describe service
kubectl describe service nginx

# Get service endpoints
kubectl get endpoints nginx

# Test service connectivity
kubectl run test-pod --image=busybox --rm -it -- wget -qO- http://nginx-service
```

### Network Troubleshooting

```bash
# Check DNS resolution
kubectl run test-pod --image=busybox --rm -it -- nslookup kubernetes.default

# Test pod-to-pod connectivity
kubectl exec -it pod1 -- ping pod2-ip

# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy <policy-name>
```

---

## ⚙️ Configuration Management

### ConfigMaps

```bash
# Create configmap from literal values
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Create configmap from file
kubectl create configmap app-config --from-file=config.properties

# Create configmap from directory
kubectl create configmap app-config --from-file=./config-dir/

# Create configmap from env file
kubectl create configmap app-config --from-env-file=config.env

# List configmaps
kubectl get configmaps
kubectl get cm

# View configmap data
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

### Secrets

```bash
# Create secret from literal values
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=secret

# Create secret from file
kubectl create secret generic app-secret --from-file=credentials.txt

# Create TLS secret
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key

# Create docker registry secret
kubectl create secret docker-registry regcred --docker-username=user --docker-password=pass --docker-email=email@example.com

# List secrets
kubectl get secrets

# View secret (base64 encoded)
kubectl get secret app-secret -o yaml

# Decode secret
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 -d
```

---

## 💾 Storage & Volumes

### Persistent Volumes

```bash
# List persistent volumes
kubectl get pv
kubectl get persistentvolumes

# Describe persistent volume
kubectl describe pv pv-name

# List persistent volume claims
kubectl get pvc
kubectl get persistentvolumeclaims

# Describe persistent volume claim
kubectl describe pvc pvc-name
```

### Storage Classes

```bash
# List storage classes
kubectl get storageclass
kubectl get sc

# Describe storage class
kubectl describe storageclass standard

# Set default storage class
kubectl patch storageclass standard -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## 🔍 Troubleshooting

### Cluster Diagnostics

```bash
# Check cluster info
kubectl cluster-info
kubectl cluster-info dump

# Check node status
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node-name>

# Check component status (deprecated in newer versions)
kubectl get componentstatuses
kubectl get cs
```

### Pod Troubleshooting

```bash
# Get pod events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector involvedObject.name=pod-name

# Check pod resource usage
kubectl top pods
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Check node resource usage
kubectl top nodes

# Debug pod
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl exec -it <pod-name> -- /bin/sh
```

### Advanced Troubleshooting

```bash
# Check API server logs (if access to control plane)
journalctl -u kubelet
journalctl -u docker

# Check kubelet logs
systemctl status kubelet

# Validate YAML syntax
kubectl apply --dry-run=client -f pod.yaml

# Explain resource structure
kubectl explain pod.spec.containers
kubectl explain deployment.spec
```

---

## 📊 Resource Management

### Resource Quotas

```bash
# List resource quotas
kubectl get resourcequota
kubectl get quota

# Describe resource quota
kubectl describe resourcequota <quota-name>

# Create resource quota
kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2
```

### Limit Ranges

```bash
# List limit ranges
kubectl get limitrange
kubectl get limits

# Describe limit range
kubectl describe limitrange <limit-name>
```

### Resource Monitoring

```bash
# Node resource usage
kubectl top nodes

# Pod resource usage
kubectl top pods
kubectl top pods -n kube-system
kubectl top pods --containers
```

---

## 🔐 Security & RBAC

### Service Accounts

```bash
# List service accounts
kubectl get serviceaccounts
kubectl get sa

# Create service account
kubectl create serviceaccount my-sa

# Describe service account
kubectl describe serviceaccount my-sa

# Get service account token
kubectl get secret $(kubectl get sa my-sa -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d
```

### RBAC

```bash
# List roles and cluster roles
kubectl get roles
kubectl get clusterroles

# List role bindings
kubectl get rolebindings
kubectl get clusterrolebindings

# Create role
kubectl create role pod-reader --verb=get,list,watch --resource=pods

# Create cluster role
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# Create role binding
kubectl create rolebinding pod-reader-binding --role=pod-reader --user=jane

# Create cluster role binding
kubectl create clusterrolebinding pod-reader-binding --clusterrole=pod-reader --user=jane

# Check permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as=system:serviceaccount:default:my-sa
```

---

## 🎯 Exam-Specific Tips

### Time-Saving Commands

```bash
# Quick alias setup (add to ~/.bashrc)
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'

# Quick namespace switching
kubectl config set-context --current --namespace=<namespace>

# Fast pod creation with shell access
kubectl run debug --image=busybox --rm -it -- sh

# Quick service test
kubectl run test --image=busybox --rm -it -- wget -qO- http://service-name:port
```

### Essential One-Liners

```bash
# Get all resources in namespace
kubectl get all

# Delete all pods with label
kubectl delete pods -l app=myapp

# Scale all deployments to 0
kubectl get deployments -o name | xargs -I {} kubectl scale {} --replicas=0

# Restart deployment (force update)
kubectl rollout restart deployment/nginx

# Get pod IP addresses
kubectl get pods -o wide --no-headers | awk '{print $1,$(NF-1)}'

# Find pods using most CPU
kubectl top pods --sort-by=cpu --no-headers | head -5

# Check which pods are not running
kubectl get pods --field-selector=status.phase!=Running
```

### YAML Generation Templates

```bash
# Pod
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Deployment
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Service
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml

# Job
kubectl create job my-job --image=busybox --dry-run=client -o yaml

# CronJob
kubectl create cronjob my-cron --image=busybox --schedule="0/5 * * * *" --dry-run=client -o yaml
```

---

## 📚 Quick Reference Card

### Most Used Commands (Exam Focus)

```bash
# The Big 5 for CKA
kubectl get pods -o wide
kubectl describe pod <name>
kubectl logs <pod> -f
kubectl exec -it <pod> -- /bin/bash
kubectl apply -f <file>

# Emergency Troubleshooting
kubectl get events --sort-by='.lastTimestamp'
kubectl top nodes
kubectl get nodes
kubectl cluster-info

# Fast Resource Creation
kubectl run <name> --image=<image>
kubectl create deployment <name> --image=<image>
kubectl expose deployment <name> --port=<port>
kubectl scale deployment <name> --replicas=<num>
```

### Key File Locations (Ubuntu Exam Environment)

```bash
# Kubelet config
/var/lib/kubelet/config.yaml

# Static pod manifests
/etc/kubernetes/manifests/

# Container logs
/var/log/containers/

# Kubernetes logs
/var/log/pods/

# etcd data
/var/lib/etcd/
```

---

💡 **Pro Tip:** Practice these commands daily and create your own shortcuts. During the exam, efficiency with kubectl is crucial for time management!

---

*Last Updated: May 29, 2025*  
*CKA Exam Version: v1.30*  
*Command Reference Version: 1.0*
