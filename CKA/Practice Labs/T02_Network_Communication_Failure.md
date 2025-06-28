# T02: Network Communication Failure

> **🎯 Exam Objectives:** Troubleshooting (30% weight) - Diagnose network connectivity issues  
> **⚡ Scenario:** FinServ Bank's payment processing microservices losing connectivity during market open  
> **⏱️ Time Limit:** 25 minutes  
> **💼 Difficulty:** Intermediate  

---

## 📋 Business Scenario

You're the Site Reliability Engineer at **FinServ Bank**, a digital banking platform processing millions of transactions daily. At 9:30 AM EST (market open), your payment processing system starts failing. Customer transactions are timing out, and the mobile app is showing "Service Unavailable" errors.

**Your Mission:** The bank's CTO is breathing down your neck as transaction failures spike to 85%. You have 25 minutes to restore service connectivity before regulatory compliance thresholds are breached and the bank faces potential fines.

**Business Impact:**
- 💳 **$50,000/minute** in failed transaction revenue
- 📱 **500,000+ customers** unable to access accounts
- 🏛️ **Regulatory exposure** if downtime exceeds 30 minutes
- 📈 **Stock price impact** as news of outage spreads
- ☎️ **Executive escalation** with board-level visibility

---

## 🎯 Lab Objectives

By completing this lab, you will master:

1. **Network Troubleshooting Methodology** - Systematic approach to connectivity issues
2. **Service Discovery Debugging** - DNS and endpoint resolution problems
3. **Pod-to-Pod Communication** - Container networking and policy issues
4. **Service Mesh Analysis** - Traffic routing and load balancing failures
5. **Network Policy Investigation** - Security rule impacts on connectivity

---

## 🔧 Lab Tasks

### **Task 1: Rapid Assessment (5 minutes)**
The payment system is down. Quickly identify the scope and nature of the connectivity failure.

**Your Actions:**
1. Check the health status of all payment-related services and endpoints
2. Identify which specific service-to-service communications are failing
3. Determine if the issue is affecting all pods or specific instances
4. Test basic connectivity patterns between critical microservices
5. Document the failure pattern to guide your investigation

**Expected CKA Skills:**
- `kubectl get services`, `kubectl get endpoints`
- `kubectl describe service` and endpoint analysis
- Pod connectivity testing with network tools
- Service discovery troubleshooting
- Network connectivity verification

---

### **Task 2: DNS and Service Discovery (7 minutes)**
Network issues often stem from DNS problems. Investigate service discovery failures.

**Your Actions:**
1. Test DNS resolution from affected pods to critical services
2. Verify that service names resolve to correct cluster IPs
3. Check CoreDNS configuration and pod health
4. Investigate service selector matching and endpoint population
5. Validate that service ports and target ports are correctly configured

**Expected CKA Skills:**
- DNS debugging from pod contexts: `nslookup`, `dig`
- Service selector and endpoint matching analysis
- CoreDNS troubleshooting and log analysis
- Service port configuration validation
- Cluster DNS configuration verification

---

### **Task 3: Network Policy and Security (8 minutes)**
Security policies might be blocking legitimate traffic. Investigate network policy impacts.

**Your Actions:**
1. Examine existing network policies affecting the payment namespace
2. Test connectivity with and without network policy restrictions
3. Identify any recently applied policies that might block traffic
4. Verify that required ingress and egress rules are properly configured
5. Check for conflicting or overly restrictive security rules

**Expected CKA Skills:**
- `kubectl get networkpolicy` and policy analysis
- Network policy rule interpretation
- Ingress/egress rule validation
- Policy troubleshooting and testing
- Security vs connectivity balance

---

### **Task 4: Container Network Interface (5 minutes)**
If DNS and policies check out, investigate the underlying network infrastructure.

**Your Actions:**
1. Verify pod network assignments and IP allocation
2. Check for CNI plugin issues or misconfigurations
3. Test pod-to-pod connectivity at the IP level
4. Investigate any network plugin logs or errors
5. Implement a connectivity restoration plan

**Expected CKA Skills:**
- Pod network analysis and IP inspection
- CNI troubleshooting techniques
- Direct IP connectivity testing
- Network plugin log analysis
- Container networking fundamentals

---

## 💡 Hints & Tips

<details>
<summary>🆘 <strong>Service connectivity failing?</strong> Click for debugging strategy...</summary>

**Service Troubleshooting Pattern:**
```bash
# Check service and endpoints
kubectl get svc payment-service -o wide
kubectl get endpoints payment-service

# Test from a debug pod
kubectl run debug-pod --image=busybox:1.35 --rm -it -- sh
# Inside pod:
nslookup payment-service
wget -qO- payment-service:8080/health

# Check service selector matching
kubectl get pods -l app=payment-service
kubectl describe svc payment-service
```

**Key Concepts:**
- Services route traffic through endpoints
- Selector labels must match pod labels exactly
- Service ports must map to container ports correctly
- DNS names follow pattern: `service.namespace.svc.cluster.local`
</details>

<details>
<summary>🔍 <strong>DNS resolution problems?</strong> Click for DNS debugging...</summary>

**DNS Debugging Strategy:**
```bash
# Check CoreDNS status
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Test DNS from affected pod
kubectl exec -it payment-pod -- nslookup payment-service
kubectl exec -it payment-pod -- nslookup payment-service.default.svc.cluster.local

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Verify DNS configuration
kubectl get configmap coredns -n kube-system -o yaml
```

**Common DNS Issues:**
- CoreDNS pods not running or unhealthy
- Incorrect service names or namespaces
- DNS policy configuration problems
- Network policy blocking DNS traffic (port 53)
</details>

<details>
<summary>⚙️ <strong>Network policies blocking traffic?</strong> Click for policy analysis...</summary>

**Network Policy Investigation:**
```bash
# List all network policies
kubectl get netpol --all-namespaces

# Check policies affecting payment namespace
kubectl get netpol -n payment-namespace
kubectl describe netpol <policy-name> -n payment-namespace

# Test connectivity with temporary policy bypass
# (Save original policy first!)
kubectl get netpol blocking-policy -o yaml > policy-backup.yaml
kubectl delete netpol blocking-policy

# Test connectivity, then restore policy
kubectl apply -f policy-backup.yaml
```

**Policy Analysis Tips:**
- Policies are additive - multiple policies can affect same pods
- Empty `podSelector: {}` affects all pods in namespace
- Default deny policies block all traffic unless explicitly allowed
- Both ingress AND egress rules may need configuration
</details>

---

## ✅ Success Criteria

You've successfully completed this lab when:

- [ ] **Root Cause Identified** - Clear understanding of what caused the connectivity failure
- [ ] **Service Restoration** - Payment processing services accepting connections again
- [ ] **Connectivity Verified** - End-to-end transaction flow working correctly
- [ ] **Prevention Implemented** - Monitoring or configuration changes to prevent recurrence
- [ ] **Documentation Complete** - Incident timeline and resolution steps recorded

---

## 🎓 Key Learning Outcomes

After completing this lab, you'll be expert at:

✅ **Systematic Network Troubleshooting** - Methodical approach to connectivity issues  
✅ **Service Discovery Mastery** - DNS and endpoint debugging skills  
✅ **Network Policy Analysis** - Understanding security impacts on connectivity  
✅ **Container Networking** - Pod-to-pod communication and CNI fundamentals  
✅ **Production Incident Response** - Managing network emergencies under pressure  

---

## 🔄 Next Steps

Ready for more network challenges? Try these related labs:

- **N02: Ingress Traffic Management** - External connectivity and load balancing
- **N03: Network Policy Enforcement** - Advanced security and isolation scenarios
- **T04: Resource Exhaustion Crisis** - Resource-related network performance issues
- **N04: DNS Resolution Debugging** - Deep dive into cluster DNS troubleshooting

---

## 📚 Additional Resources

**CKA Exam Topics Covered:**
- Network troubleshooting and connectivity
- Service discovery and DNS resolution
- Network policies and security
- Container networking fundamentals

**Documentation Links:**
- [Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
- [DNS Troubleshooting](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

---

*💡 **Pro Tip:** Network issues in production often cascade quickly. Having a systematic troubleshooting approach helps you identify root causes faster and avoid treating symptoms instead of the underlying problem.*
