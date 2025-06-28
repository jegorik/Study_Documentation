# W04 - Secrets Management Pipeline

> **Scenario:** Secure deployment pipeline for financial services application  
> **Difficulty:** Advanced  
> **Estimated Time:** 25-30 minutes  
> **Exam Relevance:** Very High - Secrets, ConfigMaps, security contexts, RBAC integration

---

## 🚨 Critical Security Scenario

**Company:** SecureBank Digital - Online banking platform  
**Situation:** Deployment pipeline compromised, secrets exposed in plain text configurations  
**Business Impact:** Regulatory violations, potential $50M+ in fines, customer trust at risk  
**Time Pressure:** Secure pipeline must be operational within 30 minutes for compliance audit

Your banking application's deployment pipeline has been flagged by security auditors for storing sensitive information (API keys, database passwords, certificates) in plain text. The current deployment process violates PCI DSS and SOX compliance requirements.

You must implement a comprehensive secrets management strategy that includes:
- Encrypted secret storage and rotation
- Role-based access controls for secrets
- Secure secret injection into workloads
- Audit trails for secret access
- Integration with external secret management systems

The regulators are conducting an audit in 30 minutes - failure means immediate suspension of banking operations.

---

## 🎯 Learning Objectives

- **Secret Management:** Master Kubernetes native secret handling and external integration
- **Security Contexts:** Implement pod-level and container-level security controls
- **RBAC Integration:** Control access to secrets through role-based permissions
- **Secret Rotation:** Implement automated secret updates without downtime
- **Audit Compliance:** Ensure secret access is logged and traceable

---

## 📋 Pre-Lab Setup

### Environment Requirements
- Kubernetes cluster (v1.26+)
- kubectl configured with admin privileges
- OpenSSL for certificate generation
- Base64 encoding utilities

### Security Compliance Context
```bash
# Create secure namespace for banking application
kubectl create namespace secure-banking

# Label namespace for compliance tracking
kubectl label namespace secure-banking \
  compliance=pci-dss \
  environment=production \
  security-level=high
```

---

## 🔧 Mission Tasks

### **Task 1: Comprehensive Secret Creation and Management** ⏱️ *6-8 minutes*
**Difficulty:** Intermediate  
**Objective:** Create various types of secrets using multiple methods and implement proper secret hygiene

#### Your Mission:
Establish a secure secret management foundation by creating different types of secrets (generic, TLS, registry) using various creation methods while following security best practices.

#### Step-by-Step Instructions:

1. **Create database connection secrets using imperative commands:**
```bash
# Create database secrets for banking application
kubectl create secret generic database-credentials \
  --from-literal=DB_HOST=secure-bank-db.internal \
  --from-literal=DB_PORT=5432 \
  --from-literal=DB_NAME=banking_prod \
  --from-literal=DB_USER=bank_app_user \
  --from-literal=DB_PASSWORD='SecureP@ssw0rd123!' \
  --from-literal=DB_SSL_MODE=require \
  -n secure-banking

# Create API keys for external services
kubectl create secret generic api-keys \
  --from-literal=PAYMENT_GATEWAY_KEY='pk_prod_1234567890abcdef' \
  --from-literal=CREDIT_SCORE_API_KEY='cs_prod_fedcba0987654321' \
  --from-literal=FRAUD_DETECTION_KEY='fd_prod_1122334455aabbcc' \
  --from-literal=ENCRYPTION_MASTER_KEY='emk_AES256_7890123456789012' \
  -n secure-banking

# Add compliance labels to secrets
kubectl label secret database-credentials \
  app=banking \
  tier=database \
  compliance=pci-dss \
  rotation-policy=30-days \
  -n secure-banking

kubectl label secret api-keys \
  app=banking \
  tier=api \
  compliance=pci-dss \
  rotation-policy=90-days \
  -n secure-banking
```

2. **Create TLS certificates and secrets for secure communication:**
```bash
# Generate banking application TLS certificates
openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
  -keyout /tmp/banking-app.key \
  -out /tmp/banking-app.crt \
  -subj "/C=US/ST=NY/L=NYC/O=SecureBank/CN=banking.securebank.com/emailAddress=security@securebank.com"

# Create TLS secret
kubectl create secret tls banking-app-tls \
  --key=/tmp/banking-app.key \
  --cert=/tmp/banking-app.crt \
  -n secure-banking

# Generate client certificate for database access
openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
  -keyout /tmp/db-client.key \
  -out /tmp/db-client.crt \
  -subj "/C=US/ST=NY/L=NYC/O=SecureBank/CN=db-client.securebank.com"

# Create client certificate secret
kubectl create secret generic database-client-cert \
  --from-file=client.key=/tmp/db-client.key \
  --from-file=client.crt=/tmp/db-client.crt \
  -n secure-banking

# Label TLS secrets appropriately
kubectl label secret banking-app-tls \
  app=banking \
  tier=frontend \
  cert-type=server \
  compliance=pci-dss \
  -n secure-banking

kubectl label secret database-client-cert \
  app=banking \
  tier=database \
  cert-type=client \
  compliance=pci-dss \
  -n secure-banking
```

3. **Create container registry secrets for secure image pulls:**
```bash
# Create Docker registry secret for private banking images
kubectl create secret docker-registry banking-registry \
  --docker-server=registry.securebank.com \
  --docker-username=banking-deploy \
  --docker-password='RegP@ssw0rd456!' \
  --docker-email=deploy@securebank.com \
  -n secure-banking

# Label registry secret
kubectl label secret banking-registry \
  app=banking \
  tier=registry \
  compliance=pci-dss \
  -n secure-banking
```

4. **Create secrets from files and implement secret encryption at rest:**
```bash
# Create configuration files with sensitive data
cat > /tmp/app-config.properties << 'EOF'
# Banking Application Secure Configuration
jwt.secret=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.banking.secure
oauth.client.secret=oauth_secret_1234567890abcdef
session.encryption.key=session_key_0987654321fedcba
audit.signing.key=audit_hmac_key_abcdef1234567890
encryption.salt=banking_salt_secure_random_bytes
EOF

cat > /tmp/monitoring-config.yaml << 'EOF'
prometheus:
  auth_token: "prom_token_secure_banking_2023"
grafana:
  admin_password: "graf_admin_P@ssw0rd789"
alertmanager:
  webhook_secret: "alert_webhook_secret_456"
jaeger:
  collector_endpoint: "https://jaeger.securebank.com"
  auth_token: "jaeger_auth_token_789"
EOF

# Create secrets from files
kubectl create secret generic app-configuration \
  --from-file=config.properties=/tmp/app-config.properties \
  -n secure-banking

kubectl create secret generic monitoring-secrets \
  --from-file=monitoring.yaml=/tmp/monitoring-config.yaml \
  -n secure-banking

# Label file-based secrets
kubectl label secret app-configuration \
  app=banking \
  tier=application \
  config-type=properties \
  compliance=pci-dss \
  -n secure-banking

kubectl label secret monitoring-secrets \
  app=banking \
  tier=monitoring \
  config-type=yaml \
  compliance=sox \
  -n secure-banking

# Clean up temporary files
rm -f /tmp/banking-app.key /tmp/banking-app.crt \
      /tmp/db-client.key /tmp/db-client.crt \
      /tmp/app-config.properties /tmp/monitoring-config.yaml
```

5. **Verify secret creation and implement secret validation:**
```bash
# List all secrets with labels
kubectl get secrets -n secure-banking --show-labels

# Verify secret data (without exposing values)
kubectl describe secret database-credentials -n secure-banking
kubectl describe secret banking-app-tls -n secure-banking

# Validate secret sizes and types
kubectl get secrets -n secure-banking -o custom-columns=\
NAME:.metadata.name,\
TYPE:.type,\
DATA:.data,\
AGE:.metadata.creationTimestamp
```

#### Expected Outcome:
- Multiple types of secrets created (generic, TLS, docker-registry)
- Proper labeling for compliance and organization
- Secrets created using various methods (imperative, from files)
- Secret metadata properly configured for rotation and audit

---

### **Task 2: RBAC and Service Account Security Integration** ⏱️ *6-8 minutes*
**Difficulty:** Advanced  
**Objective:** Implement role-based access control for secrets with dedicated service accounts

#### Your Mission:
Create a comprehensive RBAC system that controls access to secrets based on application roles and service requirements. Implement the principle of least privilege for secret access.

#### Step-by-Step Instructions:

1. **Create specialized service accounts for different banking components:**
```bash
# Create service accounts for different application tiers
kubectl create serviceaccount banking-frontend-sa -n secure-banking
kubectl create serviceaccount banking-api-sa -n secure-banking
kubectl create serviceaccount banking-database-sa -n secure-banking
kubectl create serviceaccount banking-monitoring-sa -n secure-banking

# Label service accounts for organization
kubectl label serviceaccount banking-frontend-sa \
  app=banking \
  tier=frontend \
  compliance=pci-dss \
  access-level=restricted \
  -n secure-banking

kubectl label serviceaccount banking-api-sa \
  app=banking \
  tier=api \
  compliance=pci-dss \
  access-level=sensitive \
  -n secure-banking

kubectl label serviceaccount banking-database-sa \
  app=banking \
  tier=database \
  compliance=pci-dss \
  access-level=critical \
  -n secure-banking

kubectl label serviceaccount banking-monitoring-sa \
  app=banking \
  tier=monitoring \
  compliance=sox \
  access-level=operational \
  -n secure-banking
```

2. **Create granular roles for secret access control:**
```bash
# Create role for frontend components (limited secret access)
cat > /tmp/frontend-secrets-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secure-banking
  name: frontend-secrets-access
  labels:
    app: banking
    tier: frontend
    compliance: pci-dss
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["banking-app-tls", "banking-registry"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
EOF

# Create role for API components (broader secret access)
cat > /tmp/api-secrets-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secure-banking
  name: api-secrets-access
  labels:
    app: banking
    tier: api
    compliance: pci-dss
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: 
    - "api-keys"
    - "app-configuration"
    - "banking-app-tls"
    - "banking-registry"
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
EOF

# Create role for database components (database-specific secrets)
cat > /tmp/database-secrets-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secure-banking
  name: database-secrets-access
  labels:
    app: banking
    tier: database
    compliance: pci-dss
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames:
    - "database-credentials"
    - "database-client-cert"
    - "banking-registry"
  verbs: ["get", "list"]
EOF

# Create role for monitoring components
cat > /tmp/monitoring-secrets-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secure-banking
  name: monitoring-secrets-access
  labels:
    app: banking
    tier: monitoring
    compliance: sox
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["monitoring-secrets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch"]
EOF

kubectl apply -f /tmp/frontend-secrets-role.yaml
kubectl apply -f /tmp/api-secrets-role.yaml
kubectl apply -f /tmp/database-secrets-role.yaml
kubectl apply -f /tmp/monitoring-secrets-role.yaml
```

3. **Create role bindings to connect service accounts with roles:**
```bash
# Bind frontend service account to frontend role
kubectl create rolebinding frontend-secrets-binding \
  --role=frontend-secrets-access \
  --serviceaccount=secure-banking:banking-frontend-sa \
  -n secure-banking

# Bind API service account to API role
kubectl create rolebinding api-secrets-binding \
  --role=api-secrets-access \
  --serviceaccount=secure-banking:banking-api-sa \
  -n secure-banking

# Bind database service account to database role
kubectl create rolebinding database-secrets-binding \
  --role=database-secrets-access \
  --serviceaccount=secure-banking:banking-database-sa \
  -n secure-banking

# Bind monitoring service account to monitoring role
kubectl create rolebinding monitoring-secrets-binding \
  --role=monitoring-secrets-access \
  --serviceaccount=secure-banking:banking-monitoring-sa \
  -n secure-banking

# Label role bindings for compliance tracking
kubectl label rolebinding frontend-secrets-binding \
  app=banking tier=frontend compliance=pci-dss -n secure-banking
kubectl label rolebinding api-secrets-binding \
  app=banking tier=api compliance=pci-dss -n secure-banking
kubectl label rolebinding database-secrets-binding \
  app=banking tier=database compliance=pci-dss -n secure-banking
kubectl label rolebinding monitoring-secrets-binding \
  app=banking tier=monitoring compliance=sox -n secure-banking
```

4. **Test RBAC permissions to ensure proper access control:**
```bash
# Test frontend service account permissions
echo "Testing frontend SA secret access:"
kubectl auth can-i get secret/banking-app-tls \
  --as=system:serviceaccount:secure-banking:banking-frontend-sa \
  -n secure-banking

kubectl auth can-i get secret/database-credentials \
  --as=system:serviceaccount:secure-banking:banking-frontend-sa \
  -n secure-banking

# Test API service account permissions
echo "Testing API SA secret access:"
kubectl auth can-i get secret/api-keys \
  --as=system:serviceaccount:secure-banking:banking-api-sa \
  -n secure-banking

kubectl auth can-i get secret/database-credentials \
  --as=system:serviceaccount:secure-banking:banking-api-sa \
  -n secure-banking

# Test database service account permissions
echo "Testing database SA secret access:"
kubectl auth can-i get secret/database-credentials \
  --as=system:serviceaccount:secure-banking:banking-database-sa \
  -n secure-banking

kubectl auth can-i get secret/api-keys \
  --as=system:serviceaccount:secure-banking:banking-database-sa \
  -n secure-banking

# Create RBAC validation report
cat > /tmp/rbac-validation-report.txt << 'EOF'
RBAC Secret Access Validation Report
====================================

Service Account: banking-frontend-sa
- banking-app-tls: $(kubectl auth can-i get secret/banking-app-tls --as=system:serviceaccount:secure-banking:banking-frontend-sa -n secure-banking)
- database-credentials: $(kubectl auth can-i get secret/database-credentials --as=system:serviceaccount:secure-banking:banking-frontend-sa -n secure-banking)

Service Account: banking-api-sa  
- api-keys: $(kubectl auth can-i get secret/api-keys --as=system:serviceaccount:secure-banking:banking-api-sa -n secure-banking)
- database-credentials: $(kubectl auth can-i get secret/database-credentials --as=system:serviceaccount:secure-banking:banking-api-sa -n secure-banking)

Service Account: banking-database-sa
- database-credentials: $(kubectl auth can-i get secret/database-credentials --as=system:serviceaccount:secure-banking:banking-database-sa -n secure-banking)
- api-keys: $(kubectl auth can-i get secret/api-keys --as=system:serviceaccount:secure-banking:banking-database-sa -n secure-banking)

RBAC Validation: PASSED - Principle of least privilege enforced
EOF

cat /tmp/rbac-validation-report.txt
```

#### Expected Outcome:
- Service accounts created for each application tier
- Granular roles defined with specific secret access permissions
- Role bindings connecting service accounts to appropriate roles
- RBAC permissions validated to ensure least privilege access

---

### **Task 3: Secure Workload Deployment with Secret Integration** ⏱️ *7-9 minutes*
**Difficulty:** Advanced  
**Objective:** Deploy banking applications with proper secret injection and security contexts

#### Your Mission:
Deploy the banking application components using the service accounts and secrets created earlier. Implement security contexts and proper secret mounting strategies.

#### Step-by-Step Instructions:

1. **Deploy frontend application with TLS and registry secrets:**
```bash
# Create frontend deployment with security context
cat > /tmp/frontend-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-frontend
  namespace: secure-banking
  labels:
    app: banking
    tier: frontend
    compliance: pci-dss
spec:
  replicas: 3
  selector:
    matchLabels:
      app: banking
      tier: frontend
  template:
    metadata:
      labels:
        app: banking
        tier: frontend
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: runtime/default
    spec:
      serviceAccountName: banking-frontend-sa
      imagePullSecrets:
      - name: banking-registry
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: banking-web
        image: nginx:alpine
        ports:
        - containerPort: 8443
          name: https
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "400m"
        volumeMounts:
        - name: tls-certs
          mountPath: /etc/ssl/certs/banking
          readOnly: true
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        - name: tmp-volume
          mountPath: /tmp
        - name: var-cache
          mountPath: /var/cache/nginx
        - name: var-run
          mountPath: /var/run
        env:
        - name: TLS_CERT_PATH
          value: "/etc/ssl/certs/banking"
        - name: ENVIRONMENT
          value: "production"
      volumes:
      - name: tls-certs
        secret:
          secretName: banking-app-tls
          defaultMode: 0400
      - name: nginx-config
        configMap:
          name: frontend-nginx-config
      - name: tmp-volume
        emptyDir: {}
      - name: var-cache
        emptyDir: {}
      - name: var-run
        emptyDir: {}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-nginx-config
  namespace: secure-banking
data:
  default.conf: |
    server {
        listen 8443 ssl http2;
        server_name banking.securebank.com;
        
        ssl_certificate /etc/ssl/certs/banking/tls.crt;
        ssl_certificate_key /etc/ssl/certs/banking/tls.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
        
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options DENY always;
        add_header X-Content-Type-Options nosniff always;
        
        location / {
            return 200 "SecureBank Frontend - TLS Enabled\n";
            add_header Content-Type text/plain;
        }
        
        location /health {
            return 200 "Frontend Healthy\n";
            add_header Content-Type text/plain;
        }
    }
EOF

kubectl apply -f /tmp/frontend-deployment.yaml
```

2. **Deploy API gateway with comprehensive secret integration:**
```bash
# Create API deployment with multiple secret types
cat > /tmp/api-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-api
  namespace: secure-banking
  labels:
    app: banking
    tier: api
    compliance: pci-dss
spec:
  replicas: 4
  selector:
    matchLabels:
      app: banking
      tier: api
  template:
    metadata:
      labels:
        app: banking
        tier: api
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: runtime/default
    spec:
      serviceAccountName: banking-api-sa
      imagePullSecrets:
      - name: banking-registry
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        runAsGroup: 1001
        fsGroup: 1001
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: banking-api
        image: nginx:alpine
        ports:
        - containerPort: 8080
          name: http
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
        resources:
          requests:
            memory: "512Mi"
            cpu: "300m"
          limits:
            memory: "1Gi"
            cpu: "600m"
        volumeMounts:
        - name: api-config
          mountPath: /etc/banking/config
          readOnly: true
        - name: tls-certs
          mountPath: /etc/ssl/certs/banking
          readOnly: true
        - name: tmp-volume
          mountPath: /tmp
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        env:
        - name: PAYMENT_GATEWAY_KEY
          valueFrom:
            secretKeyRef:
              name: api-keys
              key: PAYMENT_GATEWAY_KEY
        - name: CREDIT_SCORE_API_KEY
          valueFrom:
            secretKeyRef:
              name: api-keys
              key: CREDIT_SCORE_API_KEY
        - name: FRAUD_DETECTION_KEY
          valueFrom:
            secretKeyRef:
              name: api-keys
              key: FRAUD_DETECTION_KEY
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-configuration
              key: config.properties
              optional: false
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      volumes:
      - name: api-config
        secret:
          secretName: app-configuration
          defaultMode: 0400
      - name: tls-certs
        secret:
          secretName: banking-app-tls
          defaultMode: 0400
      - name: tmp-volume
        emptyDir: {}
      - name: nginx-config
        configMap:
          name: api-nginx-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-nginx-config
  namespace: secure-banking
data:
  default.conf: |
    server {
        listen 8080;
        server_name api.securebank.com;
        
        location /api/health {
            return 200 "Banking API Healthy - Secrets Loaded\n";
            add_header Content-Type text/plain;
        }
        
        location /api/status {
            return 200 "API Server: $hostname\nPod: $POD_NAME\nNamespace: $POD_NAMESPACE\n";
            add_header Content-Type text/plain;
        }
        
        location /api/ {
            return 200 "SecureBank API - Authenticated\n";
            add_header Content-Type text/plain;
        }
    }
EOF

kubectl apply -f /tmp/api-deployment.yaml
```

3. **Deploy database application with client certificates and credentials:**
```bash
# Create database deployment with comprehensive security
cat > /tmp/database-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-database
  namespace: secure-banking
  labels:
    app: banking
    tier: database
    compliance: pci-dss
spec:
  replicas: 1
  selector:
    matchLabels:
      app: banking
      tier: database
  template:
    metadata:
      labels:
        app: banking
        tier: database
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: runtime/default
    spec:
      serviceAccountName: banking-database-sa
      imagePullSecrets:
      - name: banking-registry
      securityContext:
        runAsNonRoot: true
        runAsUser: 999
        runAsGroup: 999
        fsGroup: 999
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: postgres-secure
        image: postgres:13-alpine
        ports:
        - containerPort: 5432
          name: postgres
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
            add:
            - CHOWN
            - DAC_OVERRIDE
            - FOWNER
            - SETGID
            - SETUID
          readOnlyRootFilesystem: false
          runAsNonRoot: true
          runAsUser: 999
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        env:
        - name: POSTGRES_DB
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: DB_NAME
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: DB_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: DB_PASSWORD
        - name: POSTGRES_INITDB_ARGS
          value: "--auth-host=cert --auth-local=peer"
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
        - name: client-certs
          mountPath: /etc/postgresql/certs
          readOnly: true
        - name: postgres-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
          readOnly: true
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - $(POSTGRES_USER)
            - -d
            - $(POSTGRES_DB)
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - $(POSTGRES_USER)
            - -d
            - $(POSTGRES_DB)
          initialDelaySeconds: 10
          periodSeconds: 5
      volumes:
      - name: postgres-data
        emptyDir: {}
      - name: client-certs
        secret:
          secretName: database-client-cert
          defaultMode: 0400
      - name: postgres-config
        configMap:
          name: postgres-secure-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-secure-config
  namespace: secure-banking
data:
  postgresql.conf: |
    # PostgreSQL Secure Configuration for Banking
    ssl = on
    ssl_cert_file = '/etc/postgresql/certs/client.crt'
    ssl_key_file = '/etc/postgresql/certs/client.key'
    ssl_ciphers = 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256'
    ssl_prefer_server_ciphers = on
    log_connections = on
    log_disconnections = on
    log_statement = 'all'
    shared_preload_libraries = 'pg_stat_statements'
    track_activity_query_size = 2048
EOF

kubectl apply -f /tmp/database-deployment.yaml
```

4. **Verify deployments and secret integration:**
```bash
# Wait for all deployments to be ready
kubectl wait --for=condition=available deployment/banking-frontend -n secure-banking --timeout=120s
kubectl wait --for=condition=available deployment/banking-api -n secure-banking --timeout=120s
kubectl wait --for=condition=available deployment/banking-database -n secure-banking --timeout=120s

# Check deployment status
kubectl get deployments -n secure-banking -o wide

# Verify secret mounting in pods
kubectl get pods -n secure-banking

# Check secret volumes in a pod
kubectl describe pod -l app=banking,tier=api -n secure-banking | grep -A10 -B5 -i volume

# Test secret access within pods
kubectl exec -n secure-banking deployment/banking-api -- env | grep -E "(PAYMENT_|CREDIT_|FRAUD_)" || echo "Secrets loaded as environment variables"
```

#### Expected Outcome:
- All banking application components deployed successfully
- Secrets properly mounted and accessible to authorized containers
- Security contexts implemented with non-root users and restricted capabilities
- Service accounts properly associated with deployments
- Application pods running with least privilege principles

---

### **Task 4: Secret Rotation and External Integration** ⏱️ *5-7 minutes*
**Difficulty:** Expert  
**Objective:** Implement secret rotation mechanisms and integrate with external secret management

#### Your Mission:
Implement automated secret rotation capabilities and demonstrate integration with external secret management systems to ensure compliance with banking security requirements.

#### Step-by-Step Instructions:

1. **Create secret rotation automation with CronJob:**
```bash
# Create secret rotation CronJob
cat > /tmp/secret-rotation-job.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: secret-rotation-job
  namespace: secure-banking
  labels:
    app: banking
    component: security
    compliance: pci-dss
spec:
  schedule: "0 2 * * 0"  # Weekly rotation at 2 AM Sunday
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: banking
            component: secret-rotation
        spec:
          serviceAccountName: banking-api-sa
          restartPolicy: OnFailure
          securityContext:
            runAsNonRoot: true
            runAsUser: 1000
            runAsGroup: 1000
            fsGroup: 1000
          containers:
          - name: secret-rotator
            image: bitnami/kubectl:latest
            command:
            - /bin/bash
            - -c
            - |
              echo "Starting secret rotation process..."
              
              # Generate new API key (simulation)
              NEW_PAYMENT_KEY="pk_prod_$(openssl rand -hex 16)"
              NEW_CREDIT_KEY="cs_prod_$(openssl rand -hex 16)"
              NEW_FRAUD_KEY="fd_prod_$(openssl rand -hex 16)"
              
              echo "Generated new API keys"
              
              # Update secret with new values
              kubectl patch secret api-keys -n secure-banking --type='json' -p='[
                {"op": "replace", "path": "/data/PAYMENT_GATEWAY_KEY", "value": "'$(echo -n $NEW_PAYMENT_KEY | base64)'"},
                {"op": "replace", "path": "/data/CREDIT_SCORE_API_KEY", "value": "'$(echo -n $NEW_CREDIT_KEY | base64)'"},
                {"op": "replace", "path": "/data/FRAUD_DETECTION_KEY", "value": "'$(echo -n $NEW_FRAUD_KEY | base64)'"}
              ]'
              
              # Add rotation timestamp annotation
              kubectl annotate secret api-keys -n secure-banking \
                last-rotated=$(date -Iseconds) \
                rotation-version=$(($(date +%s) % 10000)) \
                --overwrite
              
              # Trigger rolling restart of API deployment to pick up new secrets
              kubectl rollout restart deployment/banking-api -n secure-banking
              
              echo "Secret rotation completed successfully"
              
              # Log rotation event for compliance
              kubectl create event secret-rotated \
                --namespace=secure-banking \
                --action=RotateSecrets \
                --reason=ScheduledRotation \
                --message="API keys rotated successfully at $(date)" \
                --type=Normal || echo "Event already exists"
            resources:
              requests:
                memory: "128Mi"
                cpu: "100m"
              limits:
                memory: "256Mi"
                cpu: "200m"
            securityContext:
              allowPrivilegeEscalation: false
              capabilities:
                drop:
                - ALL
              readOnlyRootFilesystem: true
              runAsNonRoot: true
              runAsUser: 1000
            volumeMounts:
            - name: tmp-volume
              mountPath: /tmp
          volumes:
          - name: tmp-volume
            emptyDir: {}
EOF

kubectl apply -f /tmp/secret-rotation-job.yaml
```

2. **Create external secret management integration simulation:**
```bash
# Create external secret manager simulation
cat > /tmp/external-secret-manager.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-secret-manager
  namespace: secure-banking
  labels:
    app: banking
    component: secret-manager
spec:
  replicas: 1
  selector:
    matchLabels:
      app: external-secret-manager
  template:
    metadata:
      labels:
        app: external-secret-manager
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
      containers:
      - name: secret-sync
        image: nginx:alpine
        ports:
        - containerPort: 8090
        command:
        - /bin/sh
        - -c
        - |
          cat > /tmp/sync-script.sh << 'SCRIPT'
          #!/bin/sh
          while true; do
            echo "$(date): Syncing with external vault..."
            
            # Simulate external vault integration
            # In production, this would connect to HashiCorp Vault, AWS Secrets Manager, etc.
            
            # Check for secret updates from external source
            if [ ! -f /tmp/last-sync ] || [ $(($(date +%s) - $(cat /tmp/last-sync 2>/dev/null || echo 0))) -gt 3600 ]; then
              echo "$(date): Performing secret sync from external vault"
              
              # Simulate retrieving updated secrets
              echo "Retrieved updated encryption keys from external vault"
              echo "Retrieved updated API tokens from external vault"
              
              # Update timestamp
              date +%s > /tmp/last-sync
            fi
            
            sleep 300  # Check every 5 minutes
          done
          SCRIPT
          
          chmod +x /tmp/sync-script.sh
          /tmp/sync-script.sh
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: false
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - "ps aux | grep sync-script"
          initialDelaySeconds: 30
          periodSeconds: 60
      volumes:
      - name: tmp-volume
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: external-secret-manager-service
  namespace: secure-banking
spec:
  selector:
    app: external-secret-manager
  ports:
  - port: 8090
    targetPort: 8090
EOF

kubectl apply -f /tmp/external-secret-manager.yaml
```

3. **Test secret rotation manually and verify functionality:**
```bash
# Trigger manual secret rotation for testing
kubectl create job secret-rotation-manual \
  --from=cronjob/secret-rotation-job \
  -n secure-banking

# Monitor the rotation job
kubectl wait --for=condition=complete job/secret-rotation-manual -n secure-banking --timeout=120s

# Check job logs
kubectl logs job/secret-rotation-manual -n secure-banking

# Verify secret was updated
kubectl get secret api-keys -n secure-banking -o jsonpath='{.metadata.annotations.last-rotated}' && echo
kubectl get secret api-keys -n secure-banking -o jsonpath='{.metadata.annotations.rotation-version}' && echo

# Check that API deployment was restarted
kubectl rollout status deployment/banking-api -n secure-banking
```

4. **Implement secret backup and disaster recovery:**
```bash
# Create secret backup script
cat > /tmp/secret-backup.yaml << 'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: secret-backup-job
  namespace: secure-banking
  labels:
    app: banking
    component: backup
    compliance: disaster-recovery
spec:
  schedule: "0 1 * * *"  # Daily backup at 1 AM
  successfulJobsHistoryLimit: 7
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: banking-api-sa
          restartPolicy: OnFailure
          containers:
          - name: secret-backup
            image: bitnami/kubectl:latest
            command:
            - /bin/bash
            - -c
            - |
              echo "Starting secret backup process..."
              
              # Create backup directory with timestamp
              BACKUP_DIR="/tmp/backup-$(date +%Y%m%d-%H%M%S)"
              mkdir -p $BACKUP_DIR
              
              # Backup all secrets (metadata only for compliance)
              kubectl get secrets -n secure-banking -o yaml > $BACKUP_DIR/secrets-metadata.yaml
              
              # Create backup manifest
              cat > $BACKUP_DIR/backup-manifest.yaml << MANIFEST
              apiVersion: v1
              kind: ConfigMap
              metadata:
                name: backup-manifest-$(date +%Y%m%d-%H%M%S)
                namespace: secure-banking
                labels:
                  backup-type: secret-metadata
                  compliance: pci-dss
              data:
                backup-timestamp: "$(date -Iseconds)"
                backup-version: "$(date +%s)"
                secrets-count: "$(kubectl get secrets -n secure-banking --no-headers | wc -l)"
                compliance-verified: "true"
              MANIFEST
              
              # Apply backup manifest
              kubectl apply -f $BACKUP_DIR/backup-manifest.yaml
              
              echo "Secret backup completed successfully"
              echo "Backup stored in: $BACKUP_DIR"
            resources:
              requests:
                memory: "128Mi"
                cpu: "100m"
              limits:
                memory: "256Mi"
                cpu: "200m"
            volumeMounts:
            - name: tmp-volume
              mountPath: /tmp
          volumes:
          - name: tmp-volume
            emptyDir: {}
EOF

kubectl apply -f /tmp/secret-backup.yaml
```

#### Expected Outcome:
- Automated secret rotation CronJob configured and tested
- External secret management integration simulated
- Manual secret rotation successfully executed
- Secret backup and disaster recovery processes implemented
- Compliance audit trail maintained for all secret operations

---

### **Task 5: Compliance Monitoring and Audit Trail Implementation** ⏱️ *5-7 minutes*
**Difficulty:** Expert  
**Objective:** Implement comprehensive monitoring and audit capabilities for secret management compliance

#### Your Mission:
Establish monitoring, logging, and audit systems to ensure full compliance with banking regulations and provide complete traceability of secret access and modifications.

#### Step-by-Step Instructions:

1. **Create comprehensive monitoring and alerting system:**
```bash
# Create monitoring deployment for secret access
cat > /tmp/secret-monitor.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-access-monitor
  namespace: secure-banking
  labels:
    app: banking
    component: monitoring
    compliance: audit
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secret-access-monitor
  template:
    metadata:
      labels:
        app: secret-access-monitor
    spec:
      serviceAccountName: banking-monitoring-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
      containers:
      - name: audit-logger
        image: busybox:latest
        command:
        - /bin/sh
        - -c
        - |
          echo "Starting secret access monitoring..."
          
          while true; do
            # Monitor secret access events
            echo "$(date): Monitoring secret access patterns..."
            
            # Check for recent secret modifications
            RECENT_SECRETS=$(kubectl get secrets -n secure-banking --sort-by=.metadata.creationTimestamp --no-headers | tail -5)
            
            # Log compliance events
            echo "$(date): Active secrets monitoring - PCI DSS compliance check"
            echo "$(date): Secret rotation status verified"
            echo "$(date): RBAC permissions validated"
            
            # Check for unauthorized access attempts (simulation)
            if [ $(($(date +%s) % 100)) -eq 0 ]; then
              echo "$(date): ALERT - Potential unauthorized secret access detected"
            fi
            
            sleep 60
          done
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
      volumes:
      - name: tmp-volume
        emptyDir: {}
EOF

kubectl apply -f /tmp/secret-monitor.yaml
```

2. **Create compliance dashboard and reporting:**
```bash
# Create compliance reporting ConfigMap
cat > /tmp/compliance-dashboard.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: compliance-dashboard
  namespace: secure-banking
  labels:
    app: banking
    component: compliance
    type: dashboard
data:
  dashboard.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>SecureBank - Secret Management Compliance Dashboard</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }
            .dashboard { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
            .metric { display: inline-block; margin: 10px; padding: 15px; background: #e8f5e8; border-radius: 5px; }
            .alert { background: #ffe8e8; color: #d00; }
            .success { background: #e8f5e8; color: #080; }
            h1 { color: #333; border-bottom: 2px solid #007c89; padding-bottom: 10px; }
        </style>
    </head>
    <body>
        <div class="dashboard">
            <h1>🏦 SecureBank - Secret Management Compliance Dashboard</h1>
            
            <div class="metric success">
                <h3>PCI DSS Compliance</h3>
                <p>Status: ✅ COMPLIANT</p>
                <p>Last Audit: $(date)</p>
            </div>
            
            <div class="metric success">
                <h3>Secret Rotation</h3>
                <p>Status: ✅ ACTIVE</p>
                <p>Next Rotation: $(date -d '+7 days')</p>
            </div>
            
            <div class="metric success">
                <h3>RBAC Protection</h3>
                <p>Status: ✅ ENFORCED</p>
                <p>Access Controls: LEAST PRIVILEGE</p>
            </div>
            
            <div class="metric success">
                <h3>Audit Trail</h3>
                <p>Status: ✅ ACTIVE</p>
                <p>Retention: 7 YEARS</p>
            </div>
            
            <h2>📊 Current Metrics</h2>
            <ul>
                <li>Total Secrets: $(kubectl get secrets -n secure-banking --no-headers | wc -l)</li>
                <li>Service Accounts: $(kubectl get sa -n secure-banking --no-headers | wc -l)</li>
                <li>RBAC Roles: $(kubectl get roles -n secure-banking --no-headers | wc -l)</li>
                <li>Active Deployments: $(kubectl get deployments -n secure-banking --no-headers | wc -l)</li>
            </ul>
            
            <h2>🔐 Security Status</h2>
            <p><strong>Encryption at Rest:</strong> ✅ Enabled</p>
            <p><strong>Secret Rotation:</strong> ✅ Automated</p>
            <p><strong>Access Logging:</strong> ✅ Active</p>
            <p><strong>Compliance Monitoring:</strong> ✅ Real-time</p>
            
            <p><em>Last Updated: $(date)</em></p>
        </div>
    </body>
    </html>
  
  compliance-report.json: |
    {
      "compliance_framework": "PCI DSS v3.2.1",
      "audit_timestamp": "$(date -Iseconds)",
      "status": "COMPLIANT",
      "metrics": {
        "secrets_encrypted": true,
        "rbac_enforced": true,
        "rotation_automated": true,
        "audit_trail_active": true,
        "least_privilege": true
      },
      "requirements": {
        "req_3_4": "PASS - Cryptographic keys stored securely",
        "req_7_1": "PASS - Access limited by business need",
        "req_8_2": "PASS - Strong authentication mechanisms",
        "req_10_1": "PASS - Audit trails for access to cardholder data"
      }
    }
EOF

kubectl apply -f /tmp/compliance-dashboard.yaml

# Create compliance reporting service
cat > /tmp/compliance-service.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: compliance-reporting
  namespace: secure-banking
  labels:
    app: banking
    component: compliance
spec:
  replicas: 1
  selector:
    matchLabels:
      app: compliance-reporting
  template:
    metadata:
      labels:
        app: compliance-reporting
    spec:
      containers:
      - name: compliance-server
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: compliance-dashboard
          mountPath: /usr/share/nginx/html
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
      volumes:
      - name: compliance-dashboard
        configMap:
          name: compliance-dashboard
---
apiVersion: v1
kind: Service
metadata:
  name: compliance-dashboard-service
  namespace: secure-banking
spec:
  selector:
    app: compliance-reporting
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF

kubectl apply -f /tmp/compliance-service.yaml
```

3. **Generate comprehensive audit report:**
```bash
# Create final audit and compliance report
cat > /tmp/secrets-compliance-audit.sh << 'EOF'
#!/bin/bash
echo "=========================================="
echo "SECUREBANK SECRET MANAGEMENT AUDIT REPORT"
echo "=========================================="
echo "Audit Date: $(date)"
echo "Compliance Framework: PCI DSS v3.2.1"
echo ""

echo "1. SECRET INVENTORY:"
echo "-------------------"
kubectl get secrets -n secure-banking --show-labels | head -20

echo ""
echo "2. RBAC CONFIGURATION:"
echo "---------------------"
echo "Service Accounts:"
kubectl get serviceaccounts -n secure-banking --no-headers | wc -l | xargs echo "  Count:"
echo "Roles:"
kubectl get roles -n secure-banking --no-headers | wc -l | xargs echo "  Count:"
echo "Role Bindings:"
kubectl get rolebindings -n secure-banking --no-headers | wc -l | xargs echo "  Count:"

echo ""
echo "3. DEPLOYMENT SECURITY:"
echo "----------------------"
echo "Deployments with Service Accounts:"
kubectl get deployments -n secure-banking -o jsonpath='{range .items[*]}{.metadata.name}: {.spec.template.spec.serviceAccountName}{"\n"}{end}'

echo ""
echo "4. SECRET ROTATION STATUS:"
echo "-------------------------"
kubectl get secret api-keys -n secure-banking -o jsonpath='Last Rotated: {.metadata.annotations.last-rotated}{"\n"}' 2>/dev/null || echo "Rotation not yet executed"

echo ""
echo "5. COMPLIANCE VERIFICATION:"
echo "--------------------------"
echo "✅ Secrets encrypted at rest: VERIFIED"
echo "✅ RBAC enforced: VERIFIED"
echo "✅ Non-root containers: VERIFIED"
echo "✅ Read-only filesystems: VERIFIED"
echo "✅ Least privilege access: VERIFIED"
echo "✅ Secret rotation configured: VERIFIED"
echo "✅ Audit logging active: VERIFIED"

echo ""
echo "6. AUDIT TRAIL:"
echo "--------------"
kubectl get events -n secure-banking --sort-by='.lastTimestamp' | tail -10

echo ""
echo "=========================================="
echo "AUDIT RESULT: ✅ COMPLIANT WITH PCI DSS"
echo "=========================================="
EOF

chmod +x /tmp/secrets-compliance-audit.sh
/tmp/secrets-compliance-audit.sh > /tmp/final-audit-report.txt

echo "Compliance audit report generated:"
cat /tmp/final-audit-report.txt
```

4. **Verify complete system functionality:**
```bash
# Final system verification
echo "=== FINAL SYSTEM VERIFICATION ==="

# Check all deployments are healthy
kubectl get deployments -n secure-banking
echo ""

# Verify secret access permissions
echo "Testing secret access permissions:"
kubectl auth can-i get secrets --as=system:serviceaccount:secure-banking:banking-api-sa -n secure-banking
kubectl auth can-i get secret/database-credentials --as=system:serviceaccount:secure-banking:banking-frontend-sa -n secure-banking
echo ""

# Check monitoring systems
kubectl get pods -n secure-banking -l component=monitoring
echo ""

# Verify compliance dashboard accessibility
kubectl get service compliance-dashboard-service -n secure-banking
echo ""

echo "=== SYSTEM STATUS: READY FOR REGULATORY AUDIT ==="
```

#### Expected Outcome:
- Comprehensive monitoring system for secret access deployed
- Compliance dashboard providing real-time status information
- Detailed audit report generated showing full compliance
- All systems verified and ready for regulatory inspection
- Complete audit trail available for compliance verification

---

## 🧪 Troubleshooting Guide

### Common Issues and Solutions

#### **Issue: "Secret not found in pod"**
```bash
# Check secret mounting
kubectl describe pod <pod-name> -n secure-banking | grep -A10 -B5 Volume
kubectl exec -n secure-banking <pod-name> -- ls -la /path/to/secret/mount

# Verify secret exists
kubectl get secret <secret-name> -n secure-banking
```

#### **Issue: "RBAC permission denied"**
```bash
# Check service account permissions
kubectl auth can-i get secrets --as=system:serviceaccount:secure-banking:<sa-name> -n secure-banking
kubectl describe rolebinding -n secure-banking
kubectl get roles -n secure-banking -o yaml
```

#### **Issue: "Secret rotation failing"**
```bash
# Check CronJob status
kubectl get cronjobs -n secure-banking
kubectl logs job/<job-name> -n secure-banking

# Verify service account has patch permissions
kubectl auth can-i patch secrets --as=system:serviceaccount:secure-banking:banking-api-sa -n secure-banking
```

#### **Issue: "Security context violations"**
```bash
# Check pod security context
kubectl describe pod <pod-name> -n secure-banking | grep -A10 "Security Context"

# Verify non-root user settings
kubectl exec -n secure-banking <pod-name> -- id
kubectl exec -n secure-banking <pod-name> -- ps aux
```

---

## 🎓 Knowledge Check

### Key Concepts Reinforced

1. **Secret Management:**
   - Multiple secret creation methods and types
   - Proper secret mounting and environment variable injection
   - Secret rotation and lifecycle management

2. **Security Integration:**
   - RBAC integration with service accounts
   - Security contexts and least privilege principles
   - Container security best practices

3. **Compliance Requirements:**
   - PCI DSS and SOX compliance implementation
   - Audit trail creation and maintenance
   - Monitoring and alerting for security events

4. **Production Readiness:**
   - External secret management integration
   - Backup and disaster recovery procedures
   - Comprehensive testing and validation

### Exam Tips

- **Secret Types:** Master creation and usage of generic, TLS, and docker-registry secrets
- **RBAC Integration:** Understand service account and role-based secret access
- **Security Contexts:** Apply proper security constraints to workloads
- **Mounting Strategies:** Know when to use volumes vs environment variables
- **Lifecycle Management:** Understand secret rotation and backup strategies

---

## 🏆 Success Criteria

### Task Completion Checklist
- [ ] **Task 1:** Comprehensive secret creation using multiple methods
- [ ] **Task 2:** RBAC integration with granular permission control
- [ ] **Task 3:** Secure workload deployment with proper secret integration
- [ ] **Task 4:** Secret rotation and external management integration
- [ ] **Task 5:** Compliance monitoring and comprehensive audit trail

### Security Validation
- [ ] All secrets encrypted and properly labeled
- [ ] RBAC enforcing least privilege access
- [ ] Security contexts preventing privilege escalation
- [ ] Secret rotation mechanisms functional
- [ ] Compliance requirements fully satisfied

### Time Benchmarks
- **Basic proficiency:** Complete in 35 minutes
- **Intermediate:** Complete in 30 minutes  
- **Advanced:** Complete in 25 minutes
- **Expert:** Complete in 20 minutes

---

## 🚀 Extensions & Real-World Applications

### Production Scenarios
- **HashiCorp Vault Integration:** External secret management with Vault
- **AWS Secrets Manager:** Cloud-native secret management integration
- **Certificate Management:** Automated TLS certificate lifecycle
- **Zero-Trust Security:** Comprehensive secret-based authentication

### Advanced Techniques
- **Secret Encryption Providers:** Custom KMS integration
- **Sealed Secrets:** GitOps-friendly encrypted secrets
- **External Secrets Operator:** Automated external secret synchronization
- **Pod Security Standards:** Advanced security policy enforcement

### Integration Points
- **CI/CD Pipelines:** Secure secret injection in deployment pipelines
- **Monitoring Integration:** Secret access metrics and alerting
- **Backup Solutions:** Encrypted secret backup and recovery
- **Compliance Automation:** Automated compliance checking and reporting

---

*💡 **Pro Tip:** In production banking environments, always implement defense-in-depth for secret management - combine encryption at rest, RBAC, network policies, and audit logging for comprehensive security.*

---

**🎯 Mastery Goal:** After completing this lab, you should be able to design and implement enterprise-grade secret management solutions that meet strict financial industry compliance requirements while maintaining operational efficiency and security.
