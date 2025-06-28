# W01: Rolling Update Disaster Recovery

> **🎯 Exam Objectives:** Application deployment (15% weight) - Manage application deployments and rollbacks  
> **⚡ Scenario:** GameTech startup needs immediate rollback after failed production deployment during peak traffic  
> **⏱️ Time Limit:** 25 minutes  
> **💼 Difficulty:** Intermediate  

---

## 📋 Business Scenario

You're the DevOps engineer at **GameTech Studios**, a rapidly growing mobile gaming startup. During the Friday evening peak gaming hours (your highest traffic time), a new deployment of your main game server API was pushed to production. The deployment is now causing widespread player connection failures, and the Customer Success team is fielding angry complaints.

**Your Mission:** Quickly diagnose the failed deployment, implement an immediate rollback to restore service, and then investigate what went wrong to prevent future incidents.

**Business Impact:**

- 🎮 **50,000+ active players** experiencing connection failures
- 💰 **$15,000/hour** revenue loss during downtime
- 📱 **App Store rating** dropping from 4.8 to 4.2 stars
- 📞 **Customer support** overwhelmed with tickets

---

## 🎯 Lab Objectives

By completing this lab, you will master:

1. **Deployment Rollback Strategies** - Emergency rollback procedures
2. **Rolling Update Troubleshooting** - Diagnosing failed deployments  
3. **Resource Management** - Understanding resource constraints during updates
4. **Configuration Debugging** - Identifying config-related deployment failures
5. **Production Safety** - Implementing safe deployment practices

---

## 🔧 Lab Tasks

### **Task 1: Emergency Assessment (5 minutes)**

You've just received the alert. Quickly assess the current state of the production deployment.

**Your Actions:**

1. Check the current deployment status of the `gameserver-api`
2. Examine recent deployment history and identify the problematic version
3. Review current pod status and identify failure patterns
4. Check service endpoints and traffic routing

**Expected CKA Skills:**

- `kubectl get deployments`, `kubectl rollout status`
- `kubectl get pods`, `kubectl describe pods`  
- `kubectl get replicasets`
- Service connectivity troubleshooting

---

### **Task 2: Immediate Rollback (8 minutes)**

Time is critical. Execute an immediate rollback to the last working version.

**Your Actions:**

1. Perform an emergency rollback to the previous stable version
2. Monitor the rollback progress and ensure it completes successfully
3. Verify that the rollback restored pod health and service availability
4. Confirm that player connections are being accepted again

**Expected CKA Skills:**

- `kubectl rollout undo deployment`
- `kubectl rollout status --watch`
- Rolling update mechanisms
- Deployment history management

---

### **Task 3: Root Cause Investigation (8 minutes)**

Now that the service is restored, investigate what caused the failed deployment.

**Your Actions:**

1. Compare configurations between the failed version and working version
2. Examine resource utilization patterns during the failed deployment
3. Check for any resource constraint issues (CPU, memory, limits)
4. Review application logs from the failed pods
5. Identify the specific issue that caused the deployment failure

**Expected CKA Skills:**

- `kubectl describe deployment`
- `kubectl get events --sort-by=.metadata.creationTimestamp`
- `kubectl logs` with deployment contexts
- Resource requests/limits analysis
- Configuration comparison techniques

---

### **Task 4: Prevention Strategy (4 minutes)**

Implement safeguards to prevent similar incidents in the future.

**Your Actions:**

1. Configure deployment strategy parameters for safer rollouts
2. Set up readiness and liveness probes with appropriate timing
3. Implement resource quotas to prevent resource exhaustion
4. Document the incident and create a rollback playbook

**Expected CKA Skills:**

- Deployment strategy configuration (`maxUnavailable`, `maxSurge`)
- Health check configuration
- Resource management
- Production best practices

---

## 💡 Hints & Tips

<details>
<summary>🆘 <strong>Stuck on rollback?</strong> Click for guidance...</summary>

**Quick Rollback Pattern:**

```bash
# Check deployment history
kubectl rollout history deployment/gameserver-api

# Emergency rollback to previous version  
kubectl rollout undo deployment/gameserver-api

# Monitor rollback progress
kubectl rollout status deployment/gameserver-api --watch
```

**Key Concepts:**

- Kubernetes maintains deployment history in ReplicaSets
- `rollout undo` is atomic and safer than manual scaling
- Always verify service restoration after rollback

</details>

<details>
<summary>🔍 <strong>Investigation stuck?</strong> Click for troubleshooting approach...</summary>

**Debugging Strategy:**

1. **Compare ReplicaSets:** `kubectl get rs -l app=gameserver-api`
2. **Check Events:** `kubectl get events --field-selector involvedObject.name=gameserver-api`  
3. **Resource Analysis:** `kubectl top pods` and `kubectl describe nodes`
4. **Configuration Diff:** Compare working vs failed pod specifications

**Common Issues:**

- Resource limits too restrictive
- Missing environment variables
- Incorrect image tags or registry access
- Health check timing problems

</details>

<details>
<summary>⚙️ <strong>Prevention setup unclear?</strong> Click for configuration examples...</summary>

**Safe Deployment Configuration:**

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  template:
    spec:
      containers:
      - name: gameserver-api
        readinessProbe:
          initialDelaySeconds: 30
          periodSeconds: 10
        livenessProbe:
          initialDelaySeconds: 60
          periodSeconds: 30
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

</details>

---

## ✅ Success Criteria

You've successfully completed this lab when:

- [ ] **Emergency Response** - Rollback completed within 8 minutes of alert
- [ ] **Service Restoration** - All pods healthy and accepting connections  
- [ ] **Root Cause Identified** - Clear understanding of what caused the failure
- [ ] **Prevention Implemented** - Deployment safeguards configured
- [ ] **Documentation Complete** - Incident timeline and rollback procedure documented

---

## 🎓 Key Learning Outcomes

After completing this lab, you'll be expert at:

✅ **Production Incident Response** - Managing deployment emergencies under pressure  
✅ **Kubernetes Rollback Mechanisms** - Understanding deployment history and rollback strategies  
✅ **Deployment Strategy Configuration** - Setting up safe rolling update parameters  
✅ **Resource Constraint Debugging** - Identifying resource-related deployment failures  
✅ **Production Safety Practices** - Implementing deployment safeguards and monitoring  

---

## 🔄 Next Steps

Ready for more challenges? Try these related labs:

- **W02: Auto-scaling E-commerce Site** - Advanced HPA and VPA scenarios
- **T02: Network Communication Failure** - Service connectivity troubleshooting  
- **A02: RBAC Security Lockdown** - Deployment security and permissions
- **W03: Pod Placement Strategy** - Advanced scheduling and affinity rules

---

## 📚 Additional Resources

**CKA Exam Topics Covered:**

- Application deployment and management
- Rolling updates and rollbacks  
- Resource management
- Troubleshooting applications

**Documentation Links:**

- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

---

*💡 **Pro Tip:** In production, always have a rollback plan before any deployment. The faster you can recover from a failed deployment, the less impact on your users and business.*
