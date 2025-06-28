# CKA Exam Day Checklist & Quick Command Reference

## 🎯 Pre-Exam Preparation

### 24 Hours Before
- [ ] Review all cheat sheets (focus on Quick Reference Cards)
- [ ] Practice 2-3 complete mock exams
- [ ] Verify understanding of exam environment and tools
- [ ] Get adequate rest and nutrition

### 2 Hours Before  
- [ ] Test your internet connection and backup
- [ ] Clear workspace of non-allowed materials
- [ ] Set up comfortable environment (lighting, chair, etc.)
- [ ] Close unnecessary applications and notifications

### 30 Minutes Before
- [ ] Review this checklist one final time
- [ ] Practice key kubectl commands
- [ ] Review time management strategy
- [ ] Take deep breaths and stay calm

---

## ⏱️ Time Management Strategy (120 minutes total)

### Question Distribution Strategy
- **Easy Questions (5-7 mins each):** 40% of exam = ~48 minutes
- **Medium Questions (8-10 mins each):** 40% of exam = ~48 minutes  
- **Hard Questions (12-15 mins each):** 20% of exam = ~24 minutes

### Time Allocation
- **First Pass (90 minutes):** Answer all questions you can solve quickly
- **Second Pass (20 minutes):** Return to challenging questions
- **Final Review (10 minutes):** Verify solutions and double-check

### Red Flags - When to Move On
- Spending more than 15 minutes on a single question
- Getting stuck on syntax or configuration details
- Unable to determine the approach within 2-3 minutes

---

## 🚀 Essential kubectl Commands - Master These!

### Cluster Information
```bash
kubectl cluster-info                        # Cluster info
kubectl get nodes -o wide                   # Node details
kubectl get componentstatuses               # Control plane health
kubectl config get-contexts                 # Available contexts
kubectl config use-context <context>        # Switch context
```bash

### Resource Management (CRUD Operations)
```bash
# Create resources
kubectl create deployment nginx --image=nginx --replicas=3
kubectl create service clusterip my-svc --tcp=80:8080
kubectl create configmap app-config --from-literal=key=value
kubectl create secret generic app-secret --from-literal=password=secret

# Get resources  
kubectl get pods,svc,deployments -o wide
kubectl get all --all-namespaces
kubectl get pods --selector=app=nginx
kubectl get events --sort-by=.metadata.creationTimestamp

# Describe resources (detailed info)
kubectl describe pod <pod-name>
kubectl describe node <node-name>
kubectl describe service <service-name>

# Edit resources
kubectl edit deployment nginx
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'

# Delete resources
kubectl delete pod <pod-name>
kubectl delete deployment nginx
kubectl delete all --selector=app=nginx
```bash

### Scaling and Updates
```bash
kubectl scale deployment nginx --replicas=5
kubectl set image deployment/nginx nginx=nginx:1.21
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=1
```bash

### Troubleshooting Commands
```bash
kubectl logs <pod-name> -f                  # Pod logs (follow)
kubectl logs <pod-name> -c <container>      # Container logs
kubectl exec -it <pod-name> -- /bin/bash    # Execute into pod
kubectl port-forward <pod-name> 8080:80     # Port forwarding
kubectl top nodes                           # Node resource usage
kubectl top pods                            # Pod resource usage
```bash

### Configuration and Secrets
```bash
kubectl create configmap app-config --from-file=config.yaml
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
kubectl get configmaps,secrets -o yaml
```text

### YAML Generation and Dry-run
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl run pod1 --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml > service.yaml
```text

---

## 📋 Common Exam Scenarios & Quick Solutions

### Scenario 1: Pod Not Starting
```bash
# Check pod status and events
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>

# Common issues to check:
# - Image pull errors
# - Resource constraints  
# - Configuration errors
# - Node capacity
```bash

### Scenario 2: Service Not Accessible
```bash
# Check service and endpoints
kubectl get svc,endpoints
kubectl describe service <service-name>

# Test connectivity
kubectl run debug --image=busybox --rm -it -- wget -qO- <service-name>:80

# Common issues:
# - Wrong selector labels
# - Port mismatches
# - Network policies blocking traffic
```bash

### Scenario 3: Node Issues
```bash
# Check node status
kubectl get nodes -o wide
kubectl describe node <node-name>

# Check node resources
kubectl top nodes
kubectl describe node <node-name> | grep -A 5 "Allocated resources"

# Node operations
kubectl cordon <node-name>              # Mark unschedulable
kubectl drain <node-name> --ignore-daemonsets --force
kubectl uncordon <node-name>            # Mark schedulable
```bash

### Scenario 4: Persistent Volume Issues
```bash
# Check PV and PVC status
kubectl get pv,pvc
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>

# Common issues:
# - Storage class problems
# - Access mode mismatches
# - Size constraints
```bash

### Scenario 5: Security and RBAC
```bash
# Check permissions
kubectl auth can-i create pods --as=system:serviceaccount:default:mysa
kubectl get roles,rolebindings,clusterroles,clusterrolebindings

# Create service account and bind role
kubectl create serviceaccount mysa
kubectl create role pod-reader --verb=get,list --resource=pods
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:mysa
```bash

---

## 🔧 Critical File Locations & Configurations

### Important Directories
```bash
/etc/kubernetes/                    # Kubernetes configuration
/etc/kubernetes/manifests/          # Static pod manifests
/etc/kubernetes/pki/               # Certificates
/var/lib/kubelet/                  # Kubelet data
/var/lib/etcd/                     # etcd data
/var/log/containers/               # Container logs
```text

### Key Configuration Files
```bash
~/.kube/config                     # User kubeconfig
/etc/kubernetes/admin.conf         # Admin kubeconfig  
/var/lib/kubelet/config.yaml       # Kubelet configuration
/etc/kubernetes/manifests/kube-apiserver.yaml    # API server config
```bash

### Service Files (systemd)
```bash
/etc/systemd/system/kubelet.service
/etc/systemd/system/docker.service
/etc/systemd/system/containerd.service
```text

---

## 🎯 Exam Day Best Practices

### Before Starting Each Question
1. **Read Carefully:** Understand the context, namespace, and cluster
2. **Switch Context:** Verify you're working in the correct cluster
3. **Plan Approach:** Decide on imperative vs declarative method
4. **Note Requirements:** Pay attention to specific names, labels, ports

### During Implementation
1. **Use Shortcuts:** Prefer imperative commands when possible
2. **Verify Frequently:** Check your work at each step
3. **Use --dry-run:** Test commands before applying
4. **Save Time:** Use aliases and tab completion

### After Completing Each Question
1. **Test Solution:** Verify the requirement is met
2. **Check Status:** Ensure resources are running/ready
3. **Clean Up:** Remove temporary resources if needed
4. **Move On:** Don't over-optimize working solutions

### Common Mistakes to Avoid
- Not reading the question context (cluster, namespace)
- Forgetting to switch contexts when specified
- Not verifying the solution works
- Spending too much time on syntax perfection
- Not using available documentation

---

## 🔍 Quick Reference - Resource Specifications

### Pod with Resource Limits
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 512Mi
```yaml

### Service Definition
```yaml
apiVersion: v1
kind: Service  
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```text

### Network Policy Example
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```yaml

### Persistent Volume Claim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```text

---

## 📚 Last-Minute Documentation References

### kubectl Cheat Sheet
- Location: `https://kubernetes.io/docs/reference/kubectl/cheatsheet/`
- Bookmark: Essential for command syntax

### API Reference  
- Location: `https://kubernetes.io/docs/reference/kubernetes-api/`
- Use: Resource specifications and field references

### Tasks and Tutorials
- Location: `https://kubernetes.io/docs/tasks/`
- Use: Step-by-step guides for common tasks

---

## ✅ Final Confidence Check

### I am ready if I can:
- [ ] Create pods, deployments, and services using both imperative and declarative methods
- [ ] Configure resource limits, probes, and environment variables
- [ ] Troubleshoot pod scheduling and startup issues
- [ ] Create and manage persistent volumes and claims
- [ ] Configure network policies and test connectivity
- [ ] Manage cluster nodes (cordon, drain, uncordon)
- [ ] Create service accounts and configure RBAC
- [ ] Backup and restore etcd data
- [ ] Use kubectl efficiently with shortcuts and aliases
- [ ] Navigate Kubernetes documentation quickly

### Remember During the Exam:
- **Stay Calm:** Take deep breaths if you feel overwhelmed
- **Manage Time:** Don't get stuck on any single question
- **Verify Solutions:** Always test that your implementation works
- **Use Documentation:** The Kubernetes docs are your friend
- **Trust Your Preparation:** You've studied hard and you're ready!

---

**Good luck on your CKA exam! 🎉**

*Remember: The goal is not perfection, but demonstrating competency. You've got this!*
