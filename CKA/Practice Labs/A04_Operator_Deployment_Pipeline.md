# A04 - Operator Deployment Pipeline

## 🎯 Lab Overview
**Scenario:** You're the platform engineer for **FinTech Innovations**, a digital banking platform serving 2.3M customers globally. The company is implementing a new **automated trading algorithm** that requires real-time market data processing. Your team has decided to deploy a **Kubernetes Operator** to manage the complex lifecycle of trading algorithm instances, including automatic scaling, configuration updates, and failover handling. The operator must be production-ready before the NYSE market opens in 6 hours.

**Business Impact:** 
- **$50M+ daily trading volume** depends on algorithm availability
- Regulatory compliance requires zero data loss and audit trails
- Millisecond-level latency requirements for trading decisions

**Your Mission:** Design, deploy, and validate a complete Kubernetes Operator deployment pipeline that can manage trading algorithm instances with enterprise-grade reliability.

## 📋 Lab Details
- **Difficulty:** Advanced
- **Category:** Cluster Architecture & Installation (25% of CKA Exam)
- **Time Estimate:** 30-35 minutes
- **Prerequisites:** Understanding of Kubernetes Operators, CRDs, Controllers, RBAC

## 🚨 Tasks Overview

### Task 1: Custom Resource Definition (CRD) Creation (6 minutes)
**Situation:** Define the custom resource schema for trading algorithm instances.

**Your Actions:**
- Create CRD for TradingAlgorithm custom resource
- Define resource validation and status fields
- Implement proper versioning and schema evolution

### Task 2: Operator Service Account & RBAC Setup (7 minutes)
**Situation:** Configure secure access permissions for the operator.

**Your Actions:**
- Create service account with minimal required permissions
- Implement ClusterRole and RoleBinding for operator functionality
- Set up RBAC for custom resource management

### Task 3: Operator Controller Deployment (8 minutes)
**Situation:** Deploy the operator controller with proper configuration.

**Your Actions:**
- Deploy operator controller with leader election
- Configure resource watching and event handling
- Implement health checks and monitoring

### Task 4: Custom Resource Lifecycle Management (7 minutes)
**Situation:** Test the operator's ability to manage custom resource instances.

**Your Actions:**
- Create TradingAlgorithm custom resource instances
- Validate operator reconciliation and scaling behavior
- Test update and deletion scenarios

### Task 5: Production Hardening & Monitoring (7 minutes)
**Situation:** Implement production-grade monitoring and failure handling.

**Your Actions:**
- Configure operator metrics and logging
- Implement proper backup and disaster recovery
- Create operational runbooks and alerting

---

## 🛠 Task 1: Custom Resource Definition (CRD) Creation

### Context
The trading algorithm operator needs to manage complex configurations including market data sources, risk parameters, and scaling policies. The CRD must support enterprise requirements for validation and auditability.

### Instructions

**Step 1: Create the TradingAlgorithm CRD**
```bash
# Create namespace for the trading platform
kubectl create namespace trading-platform

# Define the TradingAlgorithm CRD
kubectl apply -f - <<EOF
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: tradingalgorithms.fintech.io
  annotations:
    controller-gen.kubebuilder.io/version: v0.13.0
spec:
  group: fintech.io
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required:
            - marketData
            - riskLimits
            - scalingPolicy
            properties:
              marketData:
                type: object
                required:
                - sources
                - updateFrequency
                properties:
                  sources:
                    type: array
                    items:
                      type: string
                    minItems: 1
                  updateFrequency:
                    type: string
                    pattern: '^[0-9]+(ms|s|m)$'
                  symbols:
                    type: array
                    items:
                      type: string
              riskLimits:
                type: object
                required:
                - maxPositionSize
                - dailyLossLimit
                properties:
                  maxPositionSize:
                    type: integer
                    minimum: 1000
                    maximum: 10000000
                  dailyLossLimit:
                    type: integer
                    minimum: 10000
                  stopLossPercentage:
                    type: number
                    minimum: 0.01
                    maximum: 0.50
              scalingPolicy:
                type: object
                required:
                - minReplicas
                - maxReplicas
                properties:
                  minReplicas:
                    type: integer
                    minimum: 1
                    maximum: 10
                  maxReplicas:
                    type: integer
                    minimum: 1
                    maximum: 100
                  targetCPUUtilization:
                    type: integer
                    minimum: 10
                    maximum: 90
                  scaleUpPolicy:
                    type: string
                    enum: ["aggressive", "moderate", "conservative"]
              tradingStrategy:
                type: object
                properties:
                  algorithm:
                    type: string
                    enum: ["momentum", "arbitrage", "mean-reversion", "ml-predictor"]
                  parameters:
                    type: object
                    additionalProperties:
                      type: string
          status:
            type: object
            properties:
              phase:
                type: string
                enum: ["Pending", "Active", "Scaling", "Error", "Suspended"]
              replicas:
                type: integer
              readyReplicas:
                type: integer
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    lastTransitionTime:
                      type: string
                      format: date-time
                    reason:
                      type: string
                    message:
                      type: string
              lastReconciled:
                type: string
                format: date-time
              totalTrades:
                type: integer
              dailyPnL:
                type: number
    additionalPrinterColumns:
    - name: Strategy
      type: string
      jsonPath: .spec.tradingStrategy.algorithm
    - name: Phase
      type: string
      jsonPath: .status.phase
    - name: Replicas
      type: integer
      jsonPath: .status.replicas
    - name: Ready
      type: integer
      jsonPath: .status.readyReplicas
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: tradingalgorithms
    singular: tradingalgorithm
    kind: TradingAlgorithm
    shortNames:
    - ta
    - algo
EOF
```

**Step 2: Create validation webhook (optional but production-ready)**
```bash
# Create a validating webhook configuration
kubectl apply -f - <<EOF
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionWebhook
metadata:
  name: trading-algorithm-validator
webhooks:
- name: validate.tradingalgorithm.fintech.io
  clientConfig:
    service:
      name: trading-operator-webhook
      namespace: trading-platform
      path: "/validate"
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["fintech.io"]
    apiVersions: ["v1"]
    resources: ["tradingalgorithms"]
  admissionReviewVersions: ["v1", "v1beta1"]
  sideEffects: None
  failurePolicy: Fail
EOF
```

**Step 3: Verify CRD installation**
```bash
# Check CRD is properly installed
kubectl get crd tradingalgorithms.fintech.io
kubectl describe crd tradingalgorithms.fintech.io

# Test CRD with a sample resource
kubectl apply -f - <<EOF
apiVersion: fintech.io/v1
kind: TradingAlgorithm
metadata:
  name: sample-momentum-algo
  namespace: trading-platform
spec:
  marketData:
    sources: ["NYSE", "NASDAQ", "CME"]
    updateFrequency: "100ms"
    symbols: ["AAPL", "GOOGL", "MSFT", "TSLA"]
  riskLimits:
    maxPositionSize: 1000000
    dailyLossLimit: 50000
    stopLossPercentage: 0.05
  scalingPolicy:
    minReplicas: 2
    maxReplicas: 20
    targetCPUUtilization: 70
    scaleUpPolicy: "moderate"
  tradingStrategy:
    algorithm: "momentum"
    parameters:
      lookbackPeriod: "15m"
      momentumThreshold: "0.02"
      riskAdjustment: "conservative"
EOF

# Verify the resource was created
kubectl get tradingalgorithms -n trading-platform
kubectl describe tradingalgorithm sample-momentum-algo -n trading-platform
```

**Verification:**
- ✅ CRD should be successfully installed with proper validation schema
- ✅ Sample TradingAlgorithm resource should be created without errors
- ✅ Custom columns should display relevant information

---

## 🛠 Task 2: Operator Service Account & RBAC Setup

### Context
The operator needs appropriate permissions to manage TradingAlgorithm resources, deployments, services, and other Kubernetes objects while following the principle of least privilege.

### Instructions

**Step 1: Create service account for the operator**
```bash
# Create service account
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: trading-operator
  namespace: trading-platform
  annotations:
    description: "Service account for FinTech trading algorithm operator"
automountServiceAccountToken: true
EOF
```

**Step 2: Create ClusterRole with required permissions**
```bash
# Create ClusterRole for operator permissions
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: trading-operator-manager
rules:
# Permission to manage TradingAlgorithm custom resources
- apiGroups: ["fintech.io"]
  resources: ["tradingalgorithms"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["fintech.io"]
  resources: ["tradingalgorithms/status"]
  verbs: ["get", "update", "patch"]
- apiGroups: ["fintech.io"]
  resources: ["tradingalgorithms/finalizers"]
  verbs: ["update"]

# Permission to manage Deployments
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments/status"]
  verbs: ["get"]

# Permission to manage Services
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

# Permission to manage ConfigMaps and Secrets
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

# Permission to manage HorizontalPodAutoscalers
- apiGroups: ["autoscaling"]
  resources: ["horizontalpodautoscalers"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

# Permission to read Pods and Nodes for monitoring
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]

# Permission for leader election
- apiGroups: ["coordination.k8s.io"]
  resources: ["leases"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

# Permission to create events
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "patch"]
EOF
```

**Step 3: Create ClusterRoleBinding**
```bash
# Bind ClusterRole to service account
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: trading-operator-manager-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: trading-operator-manager
subjects:
- kind: ServiceAccount
  name: trading-operator
  namespace: trading-platform
EOF
```

**Step 4: Create additional Role for namespace-specific permissions**
```bash
# Create Role for namespace-specific operations
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: trading-platform
  name: trading-operator-leader-election
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["coordination.k8s.io"]
  resources: ["leases"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "patch"]
EOF

# Create RoleBinding
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: trading-operator-leader-election
  namespace: trading-platform
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: trading-operator-leader-election
subjects:
- kind: ServiceAccount
  name: trading-operator
  namespace: trading-platform
EOF
```

**Step 5: Verify RBAC configuration**
```bash
# Verify service account
kubectl get serviceaccount trading-operator -n trading-platform

# Check ClusterRole and ClusterRoleBinding
kubectl describe clusterrole trading-operator-manager
kubectl describe clusterrolebinding trading-operator-manager-binding

# Test permissions using auth can-i
kubectl auth can-i get tradingalgorithms --as=system:serviceaccount:trading-platform:trading-operator -n trading-platform
kubectl auth can-i create deployments --as=system:serviceaccount:trading-platform:trading-operator -n trading-platform
kubectl auth can-i update tradingalgorithms/status --as=system:serviceaccount:trading-platform:trading-operator -n trading-platform
```

**Verification:**
- ✅ Service account should be created with proper annotations
- ✅ ClusterRole should have appropriate permissions for operator functionality
- ✅ RBAC auth can-i tests should return "yes" for required permissions

---

## 🛠 Task 3: Operator Controller Deployment

### Context
Deploy the actual operator controller that will watch for TradingAlgorithm resources and manage their lifecycle, including creation of deployments, services, and scaling policies.

### Instructions

**Step 1: Create operator configuration**
```bash
# Create ConfigMap for operator configuration
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: trading-operator-config
  namespace: trading-platform
data:
  config.yaml: |
    operator:
      name: "trading-operator"
      namespace: "trading-platform"
      reconcileInterval: "30s"
      leaderElection:
        enabled: true
        id: "trading-operator-leader"
      metrics:
        bindAddress: ":8080"
        enabled: true
      health:
        bindAddress: ":8081"
        enabled: true
    trading:
      defaultImage: "nginx:1.21"  # Placeholder for trading algorithm image
      defaultResources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "512Mi"
      defaultLabels:
        app.kubernetes.io/managed-by: "trading-operator"
        fintech.io/component: "trading-algorithm"
EOF
```

**Step 2: Deploy the operator controller**
```bash
# Deploy operator controller
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trading-operator-controller
  namespace: trading-platform
  labels:
    app: trading-operator
    component: controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: trading-operator
      component: controller
  template:
    metadata:
      labels:
        app: trading-operator
        component: controller
    spec:
      serviceAccountName: trading-operator
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        fsGroup: 65532
      containers:
      - name: manager
        image: busybox:1.35
        # Note: In production, this would be your actual operator image
        command: ['sh', '-c']
        args:
        - |
          echo "Trading Operator Controller Starting..."
          echo "Namespace: trading-platform"
          echo "ServiceAccount: trading-operator"
          echo "Watching for TradingAlgorithm resources..."
          
          # Simulate operator behavior
          while true; do
            echo "$(date): Reconciling TradingAlgorithm resources..."
            
            # Check for TradingAlgorithm resources
            if [ -f /var/run/secrets/kubernetes.io/serviceaccount/token ]; then
              echo "  - ServiceAccount token found, operator is properly configured"
            fi
            
            echo "  - Leader election: Enabled"
            echo "  - Metrics endpoint: :8080/metrics"
            echo "  - Health endpoint: :8081/healthz"
            
            sleep 60
          done
        ports:
        - containerPort: 8080
          name: metrics
          protocol: TCP
        - containerPort: 8081
          name: health
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /healthz
            port: health
          initialDelaySeconds: 15
          periodSeconds: 20
          timeoutSeconds: 10
        readinessProbe:
          httpGet:
            path: /readyz
            port: health
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 5
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        volumeMounts:
        - name: config
          mountPath: /etc/config
          readOnly: true
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
      volumes:
      - name: config
        configMap:
          name: trading-operator-config
      terminationGracePeriodSeconds: 10
EOF
```

**Step 3: Create health check endpoints (simulated)**
```bash
# Create a simple health check service
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: trading-operator-metrics
  namespace: trading-platform
  labels:
    app: trading-operator
    component: metrics
spec:
  selector:
    app: trading-operator
    component: controller
  ports:
  - name: metrics
    port: 8080
    targetPort: metrics
    protocol: TCP
  - name: health
    port: 8081
    targetPort: health
    protocol: TCP
  type: ClusterIP
EOF
```

**Step 4: Implement leader election ConfigMap**
```bash
# Create leader election lease
kubectl apply -f - <<EOF
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata:
  name: trading-operator-leader
  namespace: trading-platform
spec:
  holderIdentity: ""
  leaseDurationSeconds: 60
  renewTime: null
  acquireTime: null
EOF
```

**Step 5: Verify operator deployment**
```bash
# Check operator deployment status
kubectl get deployment trading-operator-controller -n trading-platform
kubectl rollout status deployment/trading-operator-controller -n trading-platform

# Check operator pods
kubectl get pods -n trading-platform -l app=trading-operator
kubectl describe pod -n trading-platform -l app=trading-operator

# Check operator logs
kubectl logs -n trading-platform -l app=trading-operator --tail=20

# Verify service account token mounting
kubectl exec -n trading-platform deployment/trading-operator-controller -- ls -la /var/run/secrets/kubernetes.io/serviceaccount/
```

**Verification:**
- ✅ Operator deployment should be running successfully
- ✅ Pods should be ready with health checks passing
- ✅ Service account token should be mounted correctly
- ✅ Leader election lease should be created

---

## 🛠 Task 4: Custom Resource Lifecycle Management

### Context
Test the operator's functionality by creating, updating, and managing TradingAlgorithm custom resources to ensure the complete lifecycle is working properly.

### Instructions

**Step 1: Create production trading algorithm instances**
```bash
# Create momentum trading algorithm
kubectl apply -f - <<EOF
apiVersion: fintech.io/v1
kind: TradingAlgorithm
metadata:
  name: momentum-trader-prod
  namespace: trading-platform
  labels:
    environment: production
    strategy: momentum
    risk-level: medium
spec:
  marketData:
    sources: ["NYSE", "NASDAQ"]
    updateFrequency: "50ms"
    symbols: ["AAPL", "GOOGL", "MSFT", "AMZN", "TSLA"]
  riskLimits:
    maxPositionSize: 5000000
    dailyLossLimit: 100000
    stopLossPercentage: 0.03
  scalingPolicy:
    minReplicas: 3
    maxReplicas: 15
    targetCPUUtilization: 60
    scaleUpPolicy: "moderate"
  tradingStrategy:
    algorithm: "momentum"
    parameters:
      lookbackPeriod: "10m"
      momentumThreshold: "0.015"
      riskAdjustment: "moderate"
EOF

# Create arbitrage trading algorithm
kubectl apply -f - <<EOF
apiVersion: fintech.io/v1
kind: TradingAlgorithm
metadata:
  name: arbitrage-trader-prod
  namespace: trading-platform
  labels:
    environment: production
    strategy: arbitrage
    risk-level: low
spec:
  marketData:
    sources: ["NYSE", "NASDAQ", "CME"]
    updateFrequency: "25ms"
    symbols: ["SPY", "QQQ", "IVV", "VTI"]
  riskLimits:
    maxPositionSize: 10000000
    dailyLossLimit: 50000
    stopLossPercentage: 0.01
  scalingPolicy:
    minReplicas: 5
    maxReplicas: 25
    targetCPUUtilization: 75
    scaleUpPolicy: "aggressive"
  tradingStrategy:
    algorithm: "arbitrage"
    parameters:
      spreadThreshold: "0.005"
      executionSpeed: "ultra-fast"
      hedgeRatio: "1.0"
EOF
```

**Step 2: Create mock deployments that operator would manage**
```bash
# Simulate what the operator would create for momentum trader
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: momentum-trader-prod-deployment
  namespace: trading-platform
  labels:
    app.kubernetes.io/managed-by: trading-operator
    fintech.io/component: trading-algorithm
    fintech.io/instance: momentum-trader-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: momentum-trader-prod
  template:
    metadata:
      labels:
        app: momentum-trader-prod
        fintech.io/strategy: momentum
    spec:
      containers:
      - name: algorithm
        image: nginx:1.21  # Placeholder for trading algorithm
        ports:
        - containerPort: 80
        env:
        - name: STRATEGY
          value: "momentum"
        - name: SYMBOLS
          value: "AAPL,GOOGL,MSFT,AMZN,TSLA"
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 1000m
            memory: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: momentum-trader-prod-service
  namespace: trading-platform
  labels:
    app.kubernetes.io/managed-by: trading-operator
    fintech.io/component: trading-algorithm
spec:
  selector:
    app: momentum-trader-prod
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF

# Simulate HPA that operator would create
kubectl apply -f - <<EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: momentum-trader-prod-hpa
  namespace: trading-platform
  labels:
    app.kubernetes.io/managed-by: trading-operator
    fintech.io/component: trading-algorithm
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: momentum-trader-prod-deployment
  minReplicas: 3
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
EOF
```

**Step 3: Test resource updates and scaling**
```bash
# Update trading algorithm configuration
kubectl patch tradingalgorithm momentum-trader-prod -n trading-platform --type='merge' -p='
{
  "spec": {
    "scalingPolicy": {
      "maxReplicas": 20,
      "targetCPUUtilization": 50
    },
    "riskLimits": {
      "maxPositionSize": 7500000
    }
  }
}'

# Update status to simulate operator reconciliation
kubectl patch tradingalgorithm momentum-trader-prod -n trading-platform --type='merge' --subresource=status -p='
{
  "status": {
    "phase": "Active",
    "replicas": 3,
    "readyReplicas": 3,
    "conditions": [
      {
        "type": "Ready",
        "status": "True",
        "lastTransitionTime": "'$(date -Iseconds)'",
        "reason": "DeploymentReady",
        "message": "All trading algorithm pods are ready"
      }
    ],
    "lastReconciled": "'$(date -Iseconds)'",
    "totalTrades": 1542,
    "dailyPnL": 15420.50
  }
}'
```

**Step 4: Test deletion and cleanup**
```bash
# Create a test algorithm for deletion
kubectl apply -f - <<EOF
apiVersion: fintech.io/v1
kind: TradingAlgorithm
metadata:
  name: test-deletion-algo
  namespace: trading-platform
spec:
  marketData:
    sources: ["NYSE"]
    updateFrequency: "1s"
    symbols: ["TEST"]
  riskLimits:
    maxPositionSize: 1000
    dailyLossLimit: 100
  scalingPolicy:
    minReplicas: 1
    maxReplicas: 2
    targetCPUUtilization: 80
    scaleUpPolicy: "conservative"
  tradingStrategy:
    algorithm: "momentum"
    parameters:
      test: "true"
EOF

# Add finalizer to simulate operator cleanup
kubectl patch tradingalgorithm test-deletion-algo -n trading-platform --type='merge' -p='
{
  "metadata": {
    "finalizers": ["fintech.io/trading-algorithm-cleanup"]
  }
}'

# Delete the resource
kubectl delete tradingalgorithm test-deletion-algo -n trading-platform

# Check that resource is stuck in terminating (due to finalizer)
kubectl get tradingalgorithm test-deletion-algo -n trading-platform

# Remove finalizer to complete deletion
kubectl patch tradingalgorithm test-deletion-algo -n trading-platform --type='merge' -p='
{
  "metadata": {
    "finalizers": []
  }
}'
```

**Step 5: Verify complete lifecycle management**
```bash
# List all trading algorithms
kubectl get tradingalgorithms -n trading-platform -o wide

# Check status of algorithms
kubectl describe tradingalgorithm momentum-trader-prod -n trading-platform
kubectl describe tradingalgorithm arbitrage-trader-prod -n trading-platform

# Check managed resources
kubectl get deployments -n trading-platform -l app.kubernetes.io/managed-by=trading-operator
kubectl get services -n trading-platform -l app.kubernetes.io/managed-by=trading-operator
kubectl get hpa -n trading-platform -l app.kubernetes.io/managed-by=trading-operator

# Verify operator is processing events
kubectl logs -n trading-platform -l app=trading-operator --tail=30
```

**Verification:**
- ✅ TradingAlgorithm resources should be created and updated successfully
- ✅ Status fields should be populated with realistic values
- ✅ Managed deployments, services, and HPAs should exist
- ✅ Resource deletion should work with proper finalizer handling

---

## 🛠 Task 5: Production Hardening & Monitoring

### Context
Implement production-grade monitoring, logging, and operational procedures for the trading algorithm operator.

### Instructions

**Step 1: Set up operator metrics and monitoring**
```bash
# Create ServiceMonitor for Prometheus scraping
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: trading-operator-metrics
  namespace: trading-platform
  labels:
    app: trading-operator
spec:
  selector:
    matchLabels:
      app: trading-operator
      component: metrics
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
EOF

# Create monitoring dashboard config
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: trading-operator-dashboard
  namespace: trading-platform
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "Trading Algorithm Operator",
        "panels": [
          {
            "title": "Trading Algorithms",
            "type": "stat",
            "targets": [
              {
                "expr": "kube_customresource_info{customresource_group=\"fintech.io\",customresource_kind=\"TradingAlgorithm\"}"
              }
            ]
          },
          {
            "title": "Operator Reconcile Rate",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(controller_runtime_reconcile_total[5m])"
              }
            ]
          },
          {
            "title": "Trading Algorithm Status",
            "type": "table",
            "targets": [
              {
                "expr": "kube_customresource_info{customresource_group=\"fintech.io\"}"
              }
            ]
          }
        ]
      }
    }
EOF
```

**Step 2: Implement logging and audit trail**
```bash
# Create logging configuration
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: trading-operator-logging
  namespace: trading-platform
data:
  logging.yaml: |
    logging:
      level: info
      format: json
      outputs:
        - stdout
        - file
      file:
        path: /var/log/operator/trading-operator.log
        maxSize: 100MB
        maxBackups: 5
        compress: true
      audit:
        enabled: true
        webhook:
          url: "https://audit.fintech.io/webhook"
          headers:
            Authorization: "Bearer ${AUDIT_TOKEN}"
EOF

# Create audit policy for trading operations
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: trading-audit-policy
  namespace: trading-platform
data:
  audit-policy.yaml: |
    apiVersion: audit.k8s.io/v1
    kind: Policy
    rules:
    - level: Metadata
      namespaces: ["trading-platform"]
      resources:
      - group: "fintech.io"
        resources: ["tradingalgorithms"]
      omitStages:
      - RequestReceived
    - level: Request
      namespaces: ["trading-platform"]
      resources:
      - group: ""
        resources: ["secrets", "configmaps"]
      - group: "apps"
        resources: ["deployments"]
EOF
```

**Step 3: Create backup and disaster recovery procedures**
```bash
# Create backup script for trading algorithms
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: trading-backup-script
  namespace: trading-platform
data:
  backup.sh: |
    #!/bin/bash
    set -e
    
    BACKUP_DIR="/backups/$(date +%Y%m%d-%H%M%S)"
    mkdir -p "$BACKUP_DIR"
    
    echo "Starting backup of trading algorithms..."
    
    # Backup all TradingAlgorithm resources
    kubectl get tradingalgorithms -n trading-platform -o yaml > "$BACKUP_DIR/trading-algorithms.yaml"
    
    # Backup operator configuration
    kubectl get configmap trading-operator-config -n trading-platform -o yaml > "$BACKUP_DIR/operator-config.yaml"
    
    # Backup RBAC configuration
    kubectl get clusterrole trading-operator-manager -o yaml > "$BACKUP_DIR/cluster-role.yaml"
    kubectl get clusterrolebinding trading-operator-manager-binding -o yaml > "$BACKUP_DIR/cluster-role-binding.yaml"
    
    # Backup CRD
    kubectl get crd tradingalgorithms.fintech.io -o yaml > "$BACKUP_DIR/crd.yaml"
    
    echo "Backup completed: $BACKUP_DIR"
    
    # Upload to S3 (in production)
    # aws s3 cp "$BACKUP_DIR" s3://fintech-backups/trading-operator/ --recursive
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: trading-operator-backup
  namespace: trading-platform
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: bitnami/kubectl:latest
            command: ["/bin/bash", "/scripts/backup.sh"]
            volumeMounts:
            - name: backup-script
              mountPath: /scripts
            - name: backup-storage
              mountPath: /backups
          volumes:
          - name: backup-script
            configMap:
              name: trading-backup-script
              defaultMode: 0755
          - name: backup-storage
            emptyDir: {}
          restartPolicy: OnFailure
EOF
```

**Step 4: Create operational runbooks**
```bash
# Create operational runbook
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: trading-operator-runbook
  namespace: trading-platform
data:
  runbook.md: |
    # Trading Algorithm Operator Runbook
    
    ## Emergency Procedures
    
    ### Operator Pod Crash
    1. Check operator logs: kubectl logs -n trading-platform -l app=trading-operator
    2. Check pod events: kubectl describe pod -n trading-platform -l app=trading-operator
    3. Restart deployment: kubectl rollout restart deployment/trading-operator-controller -n trading-platform
    
    ### Trading Algorithm Not Scaling
    1. Check HPA status: kubectl get hpa -n trading-platform
    2. Verify metrics server: kubectl top pods -n trading-platform
    3. Check resource requests/limits in deployment
    
    ### Custom Resource Stuck in Terminating
    1. Check finalizers: kubectl get tradingalgorithm <name> -o yaml | grep finalizers
    2. Remove finalizers: kubectl patch tradingalgorithm <name> --type='merge' -p='{"metadata":{"finalizers":[]}}'
    
    ## Monitoring & Alerts
    
    ### Key Metrics to Monitor
    - controller_runtime_reconcile_errors_total
    - controller_runtime_reconcile_time_seconds
    - kube_customresource_info{customresource_group="fintech.io"}
    - trading_algorithm_total_trades
    - trading_algorithm_daily_pnl
    
    ### Alert Thresholds
    - Reconcile errors > 5/min
    - Reconcile time > 30s
    - Trading algorithm pods not ready > 5min
    - Daily PnL deviation > 10%
    
    ## Troubleshooting
    
    ### Common Issues
    1. **RBAC Permissions**: Verify service account has required permissions
    2. **CRD Schema**: Check for validation errors in CRD
    3. **Resource Conflicts**: Look for conflicting deployments/services
    4. **Network Policies**: Ensure operator can communicate with API server
    
    ### Debug Commands
    \`\`\`bash
    # Check operator status
    kubectl get deployment trading-operator-controller -n trading-platform
    
    # View recent events
    kubectl get events -n trading-platform --sort-by='.lastTimestamp'
    
    # Test operator permissions
    kubectl auth can-i '*' '*' --as=system:serviceaccount:trading-platform:trading-operator
    
    # Check custom resource status
    kubectl get tradingalgorithms -n trading-platform -o wide
    \`\`\`
EOF
```

**Step 5: Set up alerting and health checks**
```bash
# Create alerting rules
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: trading-operator-alerts
  namespace: trading-platform
spec:
  groups:
  - name: trading-operator
    rules:
    - alert: TradingOperatorDown
      expr: up{job="trading-operator-metrics"} == 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Trading Operator is down"
        description: "Trading Algorithm Operator has been down for more than 5 minutes"
    
    - alert: TradingAlgorithmReconcileErrors
      expr: rate(controller_runtime_reconcile_errors_total[5m]) > 0.1
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "High reconcile error rate"
        description: "Trading operator is experiencing high reconcile errors"
    
    - alert: TradingAlgorithmNotReady
      expr: kube_customresource_info{customresource_group="fintech.io"} unless on(name, namespace) kube_deployment_status_replicas_ready
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "Trading Algorithm not ready"
        description: "Trading algorithm {{ $labels.name }} has been not ready for more than 10 minutes"
EOF

# Create health check endpoint test
kubectl run operator-health-check --image=curlimages/curl --rm -it --restart=Never -- \
  curl -f http://trading-operator-metrics.trading-platform:8081/healthz

# Verify metrics endpoint
kubectl run operator-metrics-check --image=curlimages/curl --rm -it --restart=Never -- \
  curl http://trading-operator-metrics.trading-platform:8080/metrics
```

**Verification:**
- ✅ Monitoring and metrics should be properly configured
- ✅ Backup procedures should be automated with CronJob
- ✅ Operational runbook should provide clear troubleshooting steps
- ✅ Health checks and alerting should be functional

---

## 🧪 Troubleshooting Guide

### Common Issues and Solutions

**Issue 1: CRD validation errors**
```bash
# Check CRD schema validation
kubectl get crd tradingalgorithms.fintech.io -o yaml | grep -A 20 openAPIV3Schema

# Test resource creation with invalid data
kubectl apply -f - <<EOF
apiVersion: fintech.io/v1
kind: TradingAlgorithm
metadata:
  name: invalid-test
spec:
  riskLimits:
    maxPositionSize: 999  # Should fail - below minimum 1000
EOF
```

**Issue 2: RBAC permission denied**
```bash
# Test specific permissions
kubectl auth can-i get tradingalgorithms --as=system:serviceaccount:trading-platform:trading-operator -n trading-platform
kubectl auth can-i create deployments --as=system:serviceaccount:trading-platform:trading-operator -n trading-platform

# Check role bindings
kubectl describe clusterrolebinding trading-operator-manager-binding
```

**Issue 3: Operator reconciliation issues**
```bash
# Check operator logs for errors
kubectl logs -n trading-platform -l app=trading-operator --tail=50

# Check leader election status
kubectl get lease trading-operator-leader -n trading-platform

# Verify custom resource status
kubectl get tradingalgorithms -n trading-platform -o yaml
```

### Performance Optimization
- Monitor reconcile time and adjust worker count
- Implement caching for frequently accessed resources
- Use resource version for efficient watches
- Optimize CRD with proper indexing

---

## 📚 Knowledge Check

### Key Concepts Tested
1. **Custom Resource Definitions (CRDs)**
   - Schema validation and versioning
   - OpenAPI v3 schema design
   - Additional printer columns and short names

2. **Operator Pattern**
   - Controller reconciliation loops
   - Resource watching and event handling
   - Leader election for high availability

3. **RBAC Security**
   - Service account configuration
   - ClusterRole and RoleBinding design
   - Principle of least privilege

4. **Production Operations**
   - Monitoring and alerting
   - Backup and disaster recovery
   - Operational runbooks and troubleshooting

### CKA Exam Relevance
- **Cluster Architecture:** Understanding operator deployment patterns
- **Security:** RBAC configuration for operators
- **Troubleshooting:** Debugging operator and custom resource issues
- **Resource Management:** CRD lifecycle and validation

### Time Benchmarks
- **Beginner:** 40-45 minutes
- **Intermediate:** 30-35 minutes  
- **Advanced:** 25-30 minutes

---

## 🎯 Success Criteria

### Task Completion Checklist
- [ ] **Task 1:** TradingAlgorithm CRD created with proper validation schema
- [ ] **Task 2:** RBAC configured with minimal required permissions
- [ ] **Task 3:** Operator controller deployed with leader election and health checks
- [ ] **Task 4:** Custom resource lifecycle management tested and validated
- [ ] **Task 5:** Production monitoring, backup, and operational procedures implemented

### Production Readiness Validation
- [ ] CRD supports all required trading algorithm configurations
- [ ] Operator has appropriate permissions for resource management
- [ ] Controller implements proper reconciliation and error handling
- [ ] Custom resources can be created, updated, and deleted safely
- [ ] Monitoring and alerting provide operational visibility

### Business Impact Achievement
- [ ] **Trading Platform Ready:** Operator can manage algorithm instances at scale
- [ ] **Regulatory Compliance:** Audit trails and backup procedures implemented
- [ ] **Operational Excellence:** Monitoring and runbooks enable 24/7 operations
- [ ] **Risk Management:** Proper validation prevents misconfigured trading algorithms

**🏆 Lab Complete!** You've successfully implemented a production-ready Kubernetes Operator deployment pipeline for managing trading algorithm instances with enterprise-grade reliability and compliance.

---

## 🔄 Cleanup

```bash
# Remove custom resources
kubectl delete tradingalgorithms --all -n trading-platform

# Remove operator deployment
kubectl delete deployment trading-operator-controller -n trading-platform

# Remove RBAC
kubectl delete clusterrolebinding trading-operator-manager-binding
kubectl delete clusterrole trading-operator-manager
kubectl delete rolebinding trading-operator-leader-election -n trading-platform
kubectl delete role trading-operator-leader-election -n trading-platform
kubectl delete serviceaccount trading-operator -n trading-platform

# Remove CRD
kubectl delete crd tradingalgorithms.fintech.io

# Remove monitoring and backup resources
kubectl delete servicemonitor trading-operator-metrics -n trading-platform
kubectl delete prometheusrule trading-operator-alerts -n trading-platform
kubectl delete cronjob trading-operator-backup -n trading-platform

# Remove configmaps
kubectl delete configmap trading-operator-config -n trading-platform
kubectl delete configmap trading-operator-dashboard -n trading-platform
kubectl delete configmap trading-operator-logging -n trading-platform
kubectl delete configmap trading-backup-script -n trading-platform
kubectl delete configmap trading-operator-runbook -n trading-platform

# Remove services
kubectl delete service trading-operator-metrics -n trading-platform

# Remove webhook configuration
kubectl delete validatingadmissionwebhook trading-algorithm-validator

# Remove namespace
kubectl delete namespace trading-platform

echo "Operator Deployment Pipeline lab cleanup completed!"
```
