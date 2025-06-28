# Lab T01: Production Pod Crash Loop

> **📋 Lab Category:** Troubleshooting  
> **⏱️ Estimated Time:** 15 minutes  
> **🎯 Difficulty:** Basic  
> **🔗 Exam Objectives:** Troubleshoot clusters and nodes, Monitor cluster and application resource usage, Manage and evaluate container output streams

---

## 🎬 Real-World Scenario

### Background Context

You're a DevOps Engineer at **TechMart**, a rapidly growing e-commerce platform. It's Black Friday morning at 6:00 AM EST, and your company is expecting the highest traffic volume of the year. The marketing team has invested $2M in advertising campaigns driving customers to the website for exclusive deals.

**Your Role:** Senior DevOps Engineer on the Platform Reliability team  
**Situation:** The main product catalog service has started experiencing intermittent failures, and pods are entering crash loops  
**Urgency:** **CRITICAL** - Revenue loss of $50,000 per minute during peak hours. CEO and CTO are monitoring the situation directly.

### The Challenge

Your monitoring alerts are firing rapidly. The `product-catalog` deployment pods are crashing and restarting continuously. Customers can't browse products, add items to cart, or complete purchases. The incident was escalated to you as the primary troubleshooter.

**Critical Business Impact:**

- 🚨 **Revenue Loss:** $50,000/minute during peak traffic
- 😡 **Customer Experience:** Website showing 500 errors
- 📈 **Traffic Peak:** Expecting 10x normal load within 2 hours
- 📞 **Executive Pressure:** CEO demanding immediate resolution

---

## 🎯 Learning Objectives

By completing this lab, you will:

- [ ] **Primary Skill:** Master systematic pod troubleshooting using kubectl logs, describe, and events
- [ ] **Secondary Skills:** Analyze resource constraints, identify configuration issues, implement quick fixes
- [ ] **Real-world Application:** Handle production incidents under time pressure with methodical approach
- [ ] **Exam Relevance:** Core CKA troubleshooting skills - logs analysis, event investigation, resource debugging

---

## 🔧 Prerequisites

### Knowledge Requirements

- [ ] Basic understanding of Kubernetes pods and deployments
- [ ] Familiarity with kubectl command-line interface
- [ ] Understanding of container lifecycle and restart policies

### Environment Setup

```bash
# Cluster requirements
- Kubernetes cluster (kind/minikube/cloud cluster)
- kubectl configured and working
- Cluster with at least 1 worker node

# Verification commands
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
```

---

## 📚 Quick Reference

### Key Commands for This Lab

```bash
kubectl get pods -o wide              # List pods with node placement
kubectl describe pod <pod-name>       # Detailed pod information
kubectl logs <pod-name>              # Current pod logs
kubectl logs <pod-name> --previous   # Previous container logs
kubectl get events --sort-by='.lastTimestamp'  # Recent cluster events
```

### Important Concepts

- **Pod Lifecycle:** Pending → Running → Succeeded/Failed → Terminating
- **Restart Policy:** Always, OnFailure, Never - determines container restart behavior
- **Resource Limits:** CPU/Memory constraints that can cause pod failures
- **Exit Codes:** Container exit codes provide failure reasons

---

## 🚀 Lab Tasks

### Task 1: Initial Incident Assessment

**Objective:** Quickly identify the scope and nature of the pod crash loop issue

**Your Mission:**
You've been called in urgently. First, you need to assess the current state of the product-catalog service. Identify which pods are failing, their current status, and get an overview of the incident scope.

**Expected Outcome:**
Clear understanding of how many pods are affected, their current states, and initial indicators of what might be causing the crash loop.

**Hints:**

- 💡 Start with a broad view of all pods to identify the problematic ones
- 💡 Look for patterns in pod names, ages, and restart counts
- 💡 Note which worker nodes the failing pods are scheduled on

---

### Task 2: Deep Dive into Pod Failure Analysis

**Objective:** Investigate the root cause of the crash loop using kubectl logs and describe

**Your Mission:**
Now that you've identified the failing pods, dive deep into the logs and pod details. Examine both current and previous container logs to understand why the containers are crashing. Use `kubectl describe` to get comprehensive pod information including events and resource usage.

**Expected Outcome:**
Clear identification of the root cause - whether it's a configuration issue, resource constraint, dependency failure, or application bug.

**Hints:**

- 💡 Check both current and previous container logs - the previous logs often contain the actual error
- 💡 Pay attention to exit codes and termination reasons in pod descriptions
- 💡 Look for resource-related issues like OOMKilled (Out of Memory)

---

### Task 3: Event Timeline Investigation

**Objective:** Use cluster events to understand the sequence of failures and any related issues

**Your Mission:**
Examine the cluster events to build a timeline of what happened. Look for patterns, correlations with other cluster activities, and any warnings that preceded the crash loops.

**Expected Outcome:**
Complete timeline of events leading to the incident and any related cluster issues that might be contributing factors.

**Hints:**

- 💡 Sort events by timestamp to see the sequence of what happened
- 💡 Look for events related to scheduling, resource allocation, or image pulling
- 💡 Check if there are events from other components that might be related

---

### Task 4: Implement Emergency Fix and Verification

**Objective:** Apply a quick fix to restore service and verify the resolution

**Your Mission:**
Based on your analysis, implement an immediate fix to get the service back online. This might involve adjusting resource limits, fixing configuration, or restarting deployments. Then verify that the fix is working and the service is stable.

**Expected Outcome:**
Product catalog service is running stably with healthy pods, ready to handle Black Friday traffic.

**Hints:**

- 💡 Sometimes a simple deployment restart can resolve transient issues
- 💡 If it's a resource issue, you might need to adjust limits or requests
- 💡 Monitor the pods for a few minutes to ensure they remain stable

---

## ⏰ Time Management

**Exam Pace Training:**

- [ ] Task 1: 3 minutes (Quick assessment)
- [ ] Task 2: 6 minutes (Deep analysis)
- [ ] Task 3: 3 minutes (Event correlation)
- [ ] Task 4: 3 minutes (Fix and verify)
- [ ] **Total Target:** 15 minutes

*⚠️ If taking longer than target time, move to solution section and understand the efficient approach*

---

## 🔧 Step-by-Step Solution

<details>
<summary><strong>📖 Click to reveal detailed solution (try solving first!)</strong></summary>

### Step 1: Initial Pod Assessment

**Command/Action:**

```bash
# Get an overview of all pods to identify the failing ones
kubectl get pods -A

# Focus on the problematic namespace/deployment
kubectl get pods -n default -o wide

# Look specifically for pods with high restart counts
kubectl get pods --field-selector=status.phase=Running --show-labels
kubectl get pods --field-selector=status.phase!=Running
```

**Explanation:**
We start with a broad assessment to quickly identify which pods are in trouble. The `-A` flag shows all namespaces, while `-o wide` provides additional details like node placement and IP addresses. High restart counts indicate crash loops.

**Expected Output:**

```bash
NAME                               READY   STATUS             RESTARTS   AGE
product-catalog-7d4b8f9c8d-abc12   0/1     CrashLoopBackOff   5          8m
product-catalog-7d4b8f9c8d-def34   0/1     CrashLoopBackOff   4          8m
product-catalog-7d4b8f9c8d-ghi56   0/1     CrashLoopBackOff   6          8m
```

---

### Step 2: Analyze Pod Details and Logs

**Command/Action:**

```bash
# Get detailed information about one of the failing pods
kubectl describe pod product-catalog-7d4b8f9c8d-abc12

# Check current container logs
kubectl logs product-catalog-7d4b8f9c8d-abc12

# CRITICAL: Check previous container logs (often contains the actual error)
kubectl logs product-catalog-7d4b8f9c8d-abc12 --previous
```

**Explanation:**
The `describe` command provides comprehensive pod information including events, resource usage, and failure reasons. Current logs might be empty if the container crashed immediately, so checking previous logs is crucial for finding the actual error.

**Expected Output:**

```bash
# From kubectl describe - look for:
Events:
  Warning  Failed     2m    kubelet  Error: failed to create containerd task: OCI runtime create failed

# From kubectl logs --previous:
Error: failed to connect to database: connection timeout after 30s
panic: database connection failed
```

---

### Step 3: Investigate Cluster Events

**Command/Action:**

```bash
# Get recent events sorted by time
kubectl get events --sort-by='.lastTimestamp' --all-namespaces

# Filter events for our specific pods
kubectl get events --field-selector involvedObject.name=product-catalog-7d4b8f9c8d-abc12

# Look at events from the last 30 minutes
kubectl get events --sort-by='.lastTimestamp' | head -20
```

**Explanation:**
Events provide cluster-wide context and can reveal issues like resource constraints, scheduling problems, or infrastructure issues that might not be obvious from pod logs alone.

**Expected Output:**

```bash
LAST SEEN   TYPE      REASON     OBJECT                               MESSAGE
2m          Warning   Failed     pod/product-catalog-7d4b8f9c8d-abc12 Error: OOMKilled
3m          Warning   BackOff    pod/product-catalog-7d4b8f9c8d-abc12 Back-off restarting failed container
```

---

### Step 4: Implement Fix Based on Root Cause

**For Memory Issues (OOMKilled):**

```bash
# Edit the deployment to increase memory limits
kubectl edit deployment product-catalog

# Or patch directly
kubectl patch deployment product-catalog -p='{"spec":{"template":{"spec":{"containers":[{"name":"product-catalog","resources":{"limits":{"memory":"512Mi"},"requests":{"memory":"256Mi"}}}]}}}}'
```

**For Configuration Issues:**

```bash
# If it's a config issue, might need to restart deployment
kubectl rollout restart deployment product-catalog

# Or scale down and up to force recreation
kubectl scale deployment product-catalog --replicas=0
kubectl scale deployment product-catalog --replicas=3
```

**Verification Commands:**

```bash
# Watch pods come back online
kubectl get pods -w

# Verify they're staying healthy
kubectl get pods
kubectl describe pod <new-pod-name>

# Check logs to ensure no more errors
kubectl logs <new-pod-name>
```

**Success Indicators:**

- [ ] All pods show Status: Running and Ready: 1/1
- [ ] Restart count stops increasing
- [ ] No error messages in current logs
- [ ] Application responds to health checks

</details>

---

## 🎓 Knowledge Check

### Understanding Questions

1. **Log Analysis Strategy:** Why is it important to check `--previous` logs when troubleshooting crash loops?
2. **Resource Management:** What's the difference between a pod being OOMKilled vs. failing due to insufficient CPU?
3. **Event Correlation:** How do cluster events help you understand the broader context of pod failures?
4. **Production Debugging:** In a time-critical incident, what's the most efficient order for troubleshooting steps?

### Hands-On Challenges

- [ ] **Variation 1:** Simulate a memory limit issue and practice the fix
- [ ] **Variation 2:** Create a pod with wrong image tag and troubleshoot
- [ ] **Integration:** Practice this workflow with different types of application failures

---

## 🔍 Common Pitfalls & Troubleshooting

### Frequent Mistakes

1. **Only Checking Current Logs**
   - **Symptom:** Empty or minimal log output from failing pods
   - **Cause:** Container crashed before writing logs, or logs were from previous restart
   - **Fix:** Always check `kubectl logs <pod> --previous` for crash loop issues

2. **Ignoring Resource Constraints**
   - **Symptom:** Pods keep restarting without clear application errors
   - **Cause:** Memory or CPU limits too low for application requirements
   - **Fix:** Check pod description for OOMKilled events and adjust resource limits

3. **Not Correlating Events**
   - **Symptom:** Missing the bigger picture of what caused the issue
   - **Cause:** Only looking at pod-specific information without cluster context
   - **Fix:** Always check cluster events for scheduling, resource, or infrastructure issues

### Debug Commands

```bash
# Essential debugging commands for crash loops
kubectl describe pod <pod-name>                    # Comprehensive pod info
kubectl logs <pod-name> --previous                 # Previous container logs
kubectl get events --sort-by='.lastTimestamp'      # Recent cluster events
kubectl top pods                                   # Current resource usage
kubectl get pods -o yaml <pod-name>               # Full pod specification
```

---

## 🌟 Real-World Applications

### Enterprise Scenarios

- **Black Friday/Cyber Monday:** High-traffic events where quick resolution is critical for revenue
- **Production Incidents:** Any crash loop during business hours affects customer experience and revenue
- **Deployment Rollouts:** New releases sometimes introduce resource or configuration issues

### Best Practices Learned

- 🏆 **Systematic Approach:** Always follow the same troubleshooting sequence for consistency
- 🏆 **Previous Logs First:** In crash loops, previous container logs usually contain the real error
- 🏆 **Resource Awareness:** Many production issues stem from inadequate resource allocation
- 🏆 **Time Management:** In critical incidents, quick assessment → deep dive → fix → verify

---

## 📚 Additional Resources

### Related CKA Topics

- Monitor cluster and application resource usage
- Manage and evaluate container output streams
- Troubleshoot clusters and nodes

### Extended Learning

- [ ] Practice with different types of application failures (startup, runtime, shutdown)
- [ ] Learn about liveness and readiness probes for better failure detection
- [ ] Explore monitoring tools like Prometheus for proactive issue detection

---

## 📝 Lab Completion

### Self-Assessment

- [ ] I can complete this lab within 15 minutes
- [ ] I understand the systematic approach to crash loop debugging
- [ ] I can explain why each troubleshooting step is necessary
- [ ] I can handle variations of this scenario (different failure types)
- [ ] I'm confident in similar exam scenarios

### Notes & Reflections

*Use this space for personal insights about troubleshooting patterns, command shortcuts you discovered, or connections to other Kubernetes concepts.*

---

### 🏁 Lab Status

- [ ] **Started:** ___________
- [ ] **Completed:** ___________  
- [ ] **Reviewed:** ___________
- [ ] **Mastered:** ___________

**Time Taken:** _______ | **Target Time:** 15 minutes  
**Difficulty Rating:** ___/5 | **Confidence Level:** ___/5

---

*💡 **Next Lab Recommendation:** A01 - Bare Metal Cluster Bootstrap (Architecture foundations) or T02 - Network Communication Failure (More advanced troubleshooting)*
