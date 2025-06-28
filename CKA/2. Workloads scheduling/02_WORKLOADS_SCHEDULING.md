# CKA Exam Cheat Sheet for Workloads & Scheduling

---

## Section 1: Exam Topic Overview

### Key Concepts

- **Pods:** Smallest deployable units in Kubernetes
- **ReplicaSets:** Ensures desired number of pod replicas
- **Deployments:** Declarative way to manage ReplicaSets and Pods
- **Services:** Stable network endpoint for accessing pods
- **ConfigMaps & Secrets:** Configuration and sensitive data management
- **Resource Management:** CPU, memory limits and requests

### Exam Objectives

- [ ] Create and manage pods, ReplicaSets, and deployments
- [ ] Configure services for different access patterns
- [ ] Implement resource quotas and limits
- [ ] Use ConfigMaps and Secrets for application configuration
- [ ] Understand scheduling constraints and node selection
- [ ] Manage application lifecycle and rolling updates

### Study Resources

- Official Kubernetes Documentation: https://kubernetes.io/docs/concepts/workloads/
- CNCF Training Materials: Application Lifecycle Management
- Practice Labs: Workload deployment and management

### Exam Weight

**20%** of total exam score

---

## Section 2: ASCII Drawing with Explanation

```text
                        KUBERNETES WORKLOADS HIERARCHY
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                        DEPLOYMENT                               │
    │  ┌─────────────────────────────────────────────────────────┐    │
    │  │                   ReplicaSet                            │    │
    │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐     │    │
    │  │  │  Pod 1  │  │  Pod 2  │  │  Pod 3  │  │  Pod 4  │     │    │
    │  │  │         │  │         │  │         │  │         │     │    │
    │  │  │ ┌─────┐ │  │ ┌─────┐ │  │ ┌─────┐ │  │ ┌─────┐ │     │    │
    │  │  │ │ App │ │  │ │ App │ │  │ │ App │ │  │ │ App │ │     │    │
    │  │  │ │ Ctr │ │  │ │ Ctr │ │  │ │ Ctr │ │  │ │ Ctr │ │     │    │
    │  │  │ └─────┘ │  │ └─────┘ │  │ └─────┘ │  │ └─────┘ │     │    │
    │  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘     │    │
    │  └─────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    │ (Exposed by)
                                    ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                          SERVICE                                │
    │                                                                 │
    │        ┌─────────────┐                ┌─────────────┐           │
    │        │ ClusterIP   │                │ LoadBalancer│           │
    │        │ (Internal)  │                │ (External)  │           │
    │        └─────────────┘                └─────────────┘           │
    │                │                              │                 │
    │                └──────────────┬───────────────┘                 │
    │                               │                                 │
    │                    ┌─────────────────┐                          │
    │                    │     NodePort    │                          │
    │                    │   (Node Access) │                          │
    │                    └─────────────────┘                          │
    └─────────────────────────────────────────────────────────────────┘
                                    │
                                    │ (Traffic Distribution)
                                    ▼
                        ┌───────────────────────┐
                        │     Endpoint          │
                        │   Pod Selection       │
                        │                       │
                        │  Pod1:80  Pod2:80     │
                        │  Pod3:80  Pod4:80     │
                        └───────────────────────┘

                    CONFIGURATION & SECRETS FLOW
    
    ┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
    │ ConfigMap   │────>│    Pod      │<────│    Secret       │
    │             │     │             │     │                 │
    │ app.conf    │     │ ┌─────────┐ │     │ db-password     │
    │ db-host     │────>│ │   App   │ │<────│ api-key         │
    │ log-level   │     │ │Container│ │     │ tls-cert        │
    └─────────────┘     │ └─────────┘ │     └─────────────────┘
                        │             │
                        │   Volume    │
                        │   Mounts    │
                        └─────────────┘
```

### Component Breakdown

#### Workload Resources

1. **Pod:**
   - Purpose: Smallest deployable unit, contains one or more containers
   - Key features: Shared network, storage, lifecycle
   - Interactions: Scheduled by scheduler, managed by higher-level controllers

2. **ReplicaSet:**
   - Purpose: Ensures desired number of pod replicas are running
   - Key features: Pod template, replica count, label selector
   - Interactions: Creates/deletes pods based on desired state

3. **Deployment:**
   - Purpose: Declarative management of ReplicaSets and Pods
   - Key features: Rolling updates, rollback, scaling
   - Interactions: Manages ReplicaSets which manage Pods

#### Service Types

1. **ClusterIP:**
   - Purpose: Internal cluster communication
   - Key features: Virtual IP, only accessible within cluster
   - Interactions: Routes traffic to healthy pod endpoints

2. **NodePort:**
   - Purpose: External access through node ports
   - Key features: Opens port on all nodes, forwards to ClusterIP
   - Interactions: Accessible from outside cluster via node IP:port

3. **LoadBalancer:**
   - Purpose: External access with cloud provider load balancer
   - Key features: Provisions external load balancer, integrates with cloud
   - Interactions: Cloud provider creates and manages external LB

---

## Section 3: Live Examples

### Scenario: Deploying a Web Application with Rolling Updates

**Problem:** Deploy a new version of a web application with zero downtime and rollback capability

**Solution Steps:**

#### Step 1: Create Initial Deployment

```bash
kubectl create deployment nginx-app --image=nginx:1.20 --replicas=3
```

**Expected Output:**

```bash
deployment.apps/nginx-app created
```

#### Step 2: Expose with Service

```bash
kubectl expose deployment nginx-app --port=80 --target-port=80 --type=ClusterIP
```

**Expected Output:**

```bash
service/nginx-app exposed
```

#### Step 3: Verify Initial Deployment

```bash
kubectl get deployments,pods,services -l app=nginx-app
```

**Expected Output:**

```bash
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-app   3/3     3            3           2m

NAME                             READY   STATUS    RESTARTS   AGE
pod/nginx-app-7d6b7b5d8c-abc12   1/1     Running   0          2m
pod/nginx-app-7d6b7b5d8c-def34   1/1     Running   0          2m
pod/nginx-app-7d6b7b5d8c-ghi56   1/1     Running   0          2m

NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/nginx-app   ClusterIP   10.96.100.50   <none>        80/TCP    1m
```

#### Step 4: Perform Rolling Update

```bash
kubectl set image deployment/nginx-app nginx=nginx:1.21 --record
```

**Expected Output:**

```bash
deployment.apps/nginx-app image updated
```

#### Step 5: Monitor Rolling Update

```bash
kubectl rollout status deployment/nginx-app
```

**Expected Output:**

```bash
Waiting for deployment "nginx-app" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "nginx-app" rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx-app" successfully rolled out
```

#### Step 6: Verify Update and Rollback if Needed

```bash
# Check rollout history
kubectl rollout history deployment/nginx-app

# If rollback needed
kubectl rollout undo deployment/nginx-app --to-revision=1
```

### Additional Scenarios

- **Scenario 2:** Configuring resource limits and requests for optimal scheduling
- **Scenario 3:** Using ConfigMaps and Secrets for application configuration

---

## Section 4: Table of Useful Commands

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `kubectl create deployment` | Create deployment | `--image`, `--replicas` | `kubectl create deployment app --image=nginx` |
| `kubectl scale deployment` | Scale deployment | `--replicas=N` | `kubectl scale deployment app --replicas=5` |
| `kubectl expose deployment` | Create service | `--port`, `--type`, `--target-port` | `kubectl expose deployment app --port=80` |
| `kubectl rollout status` | Check rollout status | `deployment/name` | `kubectl rollout status deployment/app` |
| `kubectl rollout undo` | Rollback deployment | `--to-revision=N` | `kubectl rollout undo deployment/app` |

### Pod Management

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl run` | Create pod | `kubectl run nginx --image=nginx` |
| `kubectl exec` | Execute command in pod | `kubectl exec -it nginx -- /bin/bash` |
| `kubectl logs` | View pod logs | `kubectl logs nginx -f` |
| `kubectl port-forward` | Forward local port to pod | `kubectl port-forward nginx 8080:80` |

### Configuration Management

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl create configmap` | Create ConfigMap | `kubectl create configmap app-config --from-file=config.yaml` |
| `kubectl create secret` | Create Secret | `kubectl create secret generic db-secret --from-literal=password=mypass` |
| `kubectl get configmaps` | List ConfigMaps | `kubectl get configmaps -o yaml` |
| `kubectl get secrets` | List Secrets | `kubectl get secrets -o yaml` |

### Resource Management

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl top pods` | Pod resource usage | `kubectl top pods --sort-by=cpu` |
| `kubectl describe pod` | Pod resource limits | `kubectl describe pod nginx` |
| `kubectl apply -f` | Apply resource quotas | `kubectl apply -f resource-quota.yaml` |

---

## Section 5: Best Practices

### Deployment Best Practices

1. **Resource Management:**
   - Always set resource requests and limits
   - Use appropriate CPU and memory values
   - Monitor resource utilization regularly
   - Example:

   ```yaml
   resources:
     requests:
       cpu: 100m
       memory: 128Mi
     limits:
       cpu: 500m
       memory: 512Mi
   ```

2. **Health Checks:**
   - Implement readiness and liveness probes
   - Use appropriate probe settings
   - Configure proper timeouts and intervals
   - Example:

   ```yaml
   livenessProbe:
     httpGet:
       path: /health
       port: 8080
     initialDelaySeconds: 30
     periodSeconds: 10
   ```

3. **Scaling Strategy:**
   - Use Horizontal Pod Autoscaler (HPA)
   - Configure appropriate scaling metrics
   - Set min/max replica limits
   - Consider pod disruption budgets

### Configuration Management

1. **ConfigMaps:**
   - Use for non-sensitive configuration data
   - Version configuration changes
   - Separate environment-specific configs
   - Mount as volumes or environment variables

2. **Secrets:**
   - Use for sensitive data (passwords, keys, certificates)
   - Enable encryption at rest
   - Limit secret access with RBAC
   - Regular rotation of sensitive data

3. **Environment Separation:**
   - Use namespaces for environment isolation
   - Consistent naming conventions
   - Environment-specific resource quotas

### Service and Networking

1. **Service Design:**
   - Use ClusterIP for internal communication
   - NodePort only when necessary
   - LoadBalancer for external services
   - Consider Ingress for HTTP/HTTPS traffic

2. **Network Policies:**
   - Implement least privilege access
   - Control pod-to-pod communication
   - Secure external traffic flows

### Security Considerations

1. **Pod Security:**
   - Run containers as non-root user
   - Use security contexts appropriately
   - Enable Pod Security Standards
   - Regular image vulnerability scanning

2. **RBAC Implementation:**
   - Principle of least privilege
   - Role-based access for service accounts
   - Regular access reviews

### Performance Optimization

1. **Resource Efficiency:**
   - Right-size resource requests/limits
   - Use node affinity for optimal placement
   - Implement cluster autoscaling
   - Monitor and adjust based on usage patterns

2. **Scheduling Optimization:**
   - Use node selectors and affinity rules
   - Implement pod anti-affinity for high availability
   - Consider topology spread constraints

### Troubleshooting Tips

1. **Pod Issues:**
   - Check events: `kubectl describe pod`
   - Review logs: `kubectl logs pod-name`
   - Verify resource constraints
   - Check node resources and scheduling

2. **Service Issues:**
   - Verify endpoints: `kubectl get endpoints`
   - Check service selector labels
   - Test connectivity from within cluster
   - Validate network policies

3. **Deployment Issues:**
   - Check rollout status and history
   - Verify image availability
   - Review resource quotas
   - Examine replica set events

### Exam-Specific Tips

- **Time Management:** Practice creating resources with YAML files
- **Command Shortcuts:** Use imperative commands for quick creation
- **Verification:** Always verify deployments and services work correctly
- **Documentation:** Know kubectl cheat sheet and examples

---

## Quick Reference Card

### Essential Commands for Workloads & Scheduling

```bash
kubectl create deployment app --image=nginx --replicas=3
kubectl expose deployment app --port=80 --type=ClusterIP
kubectl scale deployment app --replicas=5
kubectl set image deployment/app container=nginx:1.21
kubectl rollout status deployment/app
```

### Quick Pod Creation and Management

```bash
kubectl run nginx --image=nginx --restart=Never              # Create pod
kubectl run nginx --image=nginx --dry-run=client -o yaml     # Generate YAML
kubectl exec -it nginx -- /bin/bash                          # Execute command
kubectl logs nginx -f                                        # View logs
kubectl delete pod nginx                                     # Delete pod
```

### ConfigMap and Secret Quick Commands

```bash
kubectl create configmap app-config --from-literal=key=value
kubectl create secret generic app-secret --from-literal=password=secret
kubectl get configmaps,secrets
```

### Key Files and Paths

- Pod specs: `/etc/kubernetes/manifests/` (static pods)
- Container logs: `/var/log/containers/`
- kubelet config: `/var/lib/kubelet/config.yaml`

---

## Related Topics

- [Kubernetes Architecture]
- [Services and Networking]
- [Storage and Volumes]
- [Security and RBAC]

---

*Last Updated: May 29, 2025*
*CKA Exam Version: v1.30*
