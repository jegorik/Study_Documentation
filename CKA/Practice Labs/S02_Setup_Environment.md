# Lab S02 Setup Environment

> **🎯 Purpose:** Create realistic database migration scenarios and storage stress testing for DataFlow Solutions database cluster  
> **⏱️ Setup Time:** 5-8 minutes  
> **🔧 Complexity:** Intermediate

---

## 🌐 Business Context Setup

### DataFlow Solutions - Database Migration Crisis

**Company Profile:**

- **Industry:** SaaS Analytics Platform
- **Scale:** 500+ enterprise clients, 2TB+ daily data processing
- **Critical Systems:** Customer analytics dashboards, real-time reporting, API services
- **Growth Challenge:** 300% data growth in last 6 months causing VM database bottlenecks

**Migration Drivers:**

- **Performance Issues:** Query response times exceeding 500ms SLA
- **Scalability Limits:** Single VM database hitting resource constraints
- **Availability Requirements:** Need 99.9% uptime for enterprise clients
- **Compliance Needs:** SOC2 compliance requiring encrypted storage and audit trails

---

## 🎲 Chaos Simulation Scripts

### 1. Storage Pressure Simulation

Create realistic storage scenarios that mirror production challenges:

```bash
#!/bin/bash
# File: simulate_storage_pressure.sh

echo "🔥 STORAGE PRESSURE SIMULATION - DataFlow Database Migration"
echo "Simulating real-world storage challenges during database migration..."

# Function to create storage load
create_storage_load() {
    local namespace=$1
    local size=$2
    local identifier=$3
    
    kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: storage-load-$identifier
  namespace: $namespace
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: storage-pressure
        image: busybox
        command:
        - /bin/sh
        - -c
        - |
          echo "Creating storage pressure simulation..."
          # Create large files to simulate database growth
          dd if=/dev/zero of=/data/large_file_$identifier.dat bs=1M count=$size
          echo "Storage pressure created: ${size}MB file"
          sleep 300  # Keep load for 5 minutes
        volumeMounts:
        - name: temp-storage
          mountPath: /data
      volumes:
      - name: temp-storage
        emptyDir:
          sizeLimit: ${size}Gi
EOF
}

# Simulate various storage scenarios
echo "📊 Creating database growth simulation..."
create_storage_load "dataflow-db" "1024" "analytics-growth"

echo "🔄 Creating backup storage pressure..."
create_storage_load "dataflow-db" "512" "backup-surge"

echo "📈 Creating reporting data spike..."
create_storage_load "dataflow-db" "256" "reporting-spike"

# Storage performance test
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: storage-performance-test
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: perf-test
        image: busybox
        command:
        - /bin/sh
        - -c
        - |
          echo "🚀 Storage Performance Baseline Test"
          echo "Simulating DataFlow production workload patterns..."
          
          # Sequential write test (analytics data ingestion)
          echo "Testing sequential writes (analytics ingestion)..."
          time dd if=/dev/zero of=/data/seq_write.dat bs=1M count=100 oflag=direct
          
          # Random read test (dashboard queries)
          echo "Testing random reads (dashboard queries)..."
          time dd if=/data/seq_write.dat of=/dev/null bs=4k skip=\$((RANDOM % 25600)) count=1024
          
          # Mixed workload test (real-time analytics)
          echo "Testing mixed workload (real-time processing)..."
          for i in {1..10}; do
            dd if=/dev/zero of=/data/mixed_\$i.dat bs=64k count=16 &
            dd if=/data/seq_write.dat of=/dev/null bs=4k skip=\$((RANDOM % 25600)) count=64 &
          done
          wait
          
          echo "Performance test completed"
        volumeMounts:
        - name: perf-storage
          mountPath: /data
      volumes:
      - name: perf-storage
        emptyDir:
          sizeLimit: 2Gi
EOF

echo "✅ Storage pressure simulation active"
echo "⏰ Duration: 5-10 minutes"
echo "📋 Monitor with: kubectl get jobs -n dataflow-db"
```

### 2. Database Migration Scenarios

Simulate various migration challenges:

```bash
#!/bin/bash
# File: simulate_migration_scenarios.sh

echo "🎯 DATABASE MIGRATION SCENARIOS - DataFlow Solutions"

# Scenario 1: Legacy data corruption simulation
create_data_corruption_scenario() {
    kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: legacy-data-corruption
  namespace: dataflow-db
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: corruption-sim
        image: postgres:14
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        command:
        - /bin/bash
        - -c
        - |
          echo "🔧 Simulating legacy database corruption scenario..."
          
          # Create corrupted data table
          psql -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local \
               -U dataflow_user -d dataflow << 'EOSQL'
          
          -- Simulate corrupted analytics data from legacy system
          CREATE TABLE IF NOT EXISTS legacy_corrupted_data (
            id SERIAL PRIMARY KEY,
            customer_id INTEGER,
            event_timestamp TIMESTAMP,
            corrupted_json_data TEXT,  -- Invalid JSON from legacy system
            migration_status VARCHAR(20) DEFAULT 'needs_cleanup'
          );
          
          -- Insert problematic data that needs cleaning
          INSERT INTO legacy_corrupted_data (customer_id, event_timestamp, corrupted_json_data) VALUES 
          (1, '2024-01-15 10:30:00', '{"invalid": json, "missing_quotes": value}'),
          (2, '2024-01-15 11:45:00', '{"truncated": "data...'),
          (3, '2024-01-15 12:00:00', 'not_json_at_all'),
          (1, '2024-01-15 13:15:00', '{"nested": {"broken": }'),
          (4, '2024-01-15 14:30:00', '{"unicode_issue": "€£¥"}');
          
          \echo 'Legacy corruption scenario created - requires data cleaning during migration'
          EOSQL
EOF
}

# Scenario 2: High load during migration
create_high_load_scenario() {
    kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: migration-load-simulation
  namespace: dataflow-db
spec:
  parallelism: 3
  completions: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: load-generator
        image: postgres:14
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        command:
        - /bin/bash
        - -c
        - |
          echo "📈 Simulating production load during migration..."
          
          # Simulate continuous analytics queries during migration
          for i in {1..100}; do
            psql -h postgres-cluster-0.postgres-headless.dataflow-db.svc.cluster.local \
                 -U dataflow_user -d dataflow -c "
            SELECT 
              c.name,
              COUNT(ae.id) as event_count,
              MAX(ae.timestamp) as last_event
            FROM customers c 
            LEFT JOIN analytics_events ae ON c.id = ae.customer_id 
            GROUP BY c.id, c.name;" &> /dev/null
            
            sleep 0.5
            
            if [ \$((i % 20)) -eq 0 ]; then
              echo "Completed \$i/100 queries..."
            fi
          done
          
          echo "Load simulation completed"
EOF
}

# Scenario 3: Network interruption during migration
create_network_interruption() {
    kubectl apply -f - <<EOF
apiVersion: v1
kind: NetworkPolicy
metadata:
  name: migration-network-chaos
  namespace: dataflow-db
spec:
  podSelector:
    matchLabels:
      app: postgres-cluster
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: dataflow-db
    - podSelector: {}
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: dataflow-db
    - podSelector: {}
  - to: []  # Allow DNS
    ports:
    - protocol: UDP
      port: 53
EOF

    echo "⚠️  Network policy applied - simulating intermittent connectivity"
    echo "🕐 Will auto-remove in 2 minutes..."
    
    # Schedule cleanup
    sleep 120
    kubectl delete networkpolicy migration-network-chaos -n dataflow-db --ignore-not-found=true
    echo "✅ Network chaos cleaned up"
}

# Execute scenarios
echo "🎬 Starting DataFlow migration scenarios..."

create_data_corruption_scenario
echo "✅ Data corruption scenario active"

create_high_load_scenario  
echo "✅ High load scenario active"

# Start network chaos in background
create_network_interruption &

echo ""
echo "📊 ACTIVE MIGRATION SCENARIOS:"
echo "  1. Legacy data corruption (requires cleanup)"
echo "  2. Production load simulation (3 parallel jobs)"
echo "  3. Network interruption (2-minute duration)"
echo ""
echo "🔍 Monitor scenarios:"
echo "  kubectl get jobs -n dataflow-db"
echo "  kubectl get networkpolicies -n dataflow-db"
echo "  kubectl get pods -n dataflow-db -o wide"
```

### 3. Monitoring and Alerting Setup

Create realistic monitoring scenarios:

```bash
#!/bin/bash
# File: setup_monitoring_chaos.sh

echo "📊 MONITORING SETUP - DataFlow Database Migration"

# Create monitoring stress scenarios
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: monitoring-scenarios
  namespace: dataflow-db
data:
  storage-alerts.sh: |
    #!/bin/bash
    echo "🚨 DataFlow Storage Alert Simulation"
    
    # Simulate storage threshold alerts
    echo "ALERT: Storage utilization >85% on postgres-cluster-1"
    echo "ALERT: Backup storage >90% full"
    echo "ALERT: Database growth rate exceeded baseline by 200%"
    
    # Query real storage metrics
    kubectl get pvc -n dataflow-db -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,CAPACITY:.status.capacity.storage"
    
    # Simulate performance degradation
    echo "WARNING: Query response time >200ms detected"
    echo "WARNING: Connection pool >80% utilized"
    
  performance-simulation.sql: |
    -- DataFlow production query patterns
    -- Simulate dashboard loading queries
    SELECT 
      c.name as customer_name,
      COUNT(ae.id) as total_events,
      COUNT(CASE WHEN ae.event_type = 'page_view' THEN 1 END) as page_views,
      COUNT(CASE WHEN ae.event_type = 'button_click' THEN 1 END) as interactions,
      AVG(CASE WHEN ae.event_data->>'duration' IS NOT NULL 
               THEN (ae.event_data->>'duration')::numeric 
               END) as avg_session_duration
    FROM customers c
    LEFT JOIN analytics_events ae ON c.id = ae.customer_id
    WHERE ae.timestamp >= NOW() - INTERVAL '24 hours'
    GROUP BY c.id, c.name
    ORDER BY total_events DESC;
    
    -- Simulate real-time analytics queries
    SELECT 
      DATE_TRUNC('hour', timestamp) as hour,
      event_type,
      COUNT(*) as event_count
    FROM analytics_events 
    WHERE timestamp >= NOW() - INTERVAL '7 days'
    GROUP BY DATE_TRUNC('hour', timestamp), event_type
    ORDER BY hour DESC, event_count DESC;
    
    -- Simulate heavy reporting query
    WITH customer_metrics AS (
      SELECT 
        customer_id,
        DATE(timestamp) as event_date,
        COUNT(*) as daily_events,
        COUNT(DISTINCT event_type) as unique_event_types
      FROM analytics_events
      GROUP BY customer_id, DATE(timestamp)
    )
    SELECT 
      c.name,
      AVG(cm.daily_events) as avg_daily_events,
      MAX(cm.daily_events) as peak_daily_events,
      AVG(cm.unique_event_types) as avg_event_types
    FROM customers c
    JOIN customer_metrics cm ON c.id = cm.customer_id
    GROUP BY c.id, c.name;
    
  backup-verification.sh: |
    #!/bin/bash
    echo "🔍 DataFlow Backup Verification"
    
    # Check backup storage usage
    kubectl exec -n dataflow-db postgres-cluster-0 -- df -h /backup 2>/dev/null || echo "Backup mount not available"
    
    # List recent backups
    echo "Recent backups:"
    kubectl exec -n dataflow-db postgres-cluster-0 -- ls -la /backup/*.sql 2>/dev/null || echo "No backup files found"
    
    # Simulate backup integrity check
    echo "Backup integrity: OK (simulated)"
    echo "Last backup: $(date -d '1 hour ago')"
    echo "Next scheduled backup: $(date -d '1 day')"
EOF

# Create monitoring job
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dataflow-monitoring-simulation
  namespace: dataflow-db
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: monitoring-sim
            image: bitnami/kubectl:latest
            volumeMounts:
            - name: monitoring-scripts
              mountPath: /scripts
            command:
            - /bin/bash
            - -c
            - |
              echo "🔍 DataFlow Monitoring Check - $(date)"
              
              # Run storage alerts
              /scripts/storage-alerts.sh
              
              echo ""
              echo "📊 Current Cluster Status:"
              kubectl get pods -n dataflow-db
              echo ""
              kubectl get pvc -n dataflow-db
              
              # Simulate random alerts based on time
              if [ \$((RANDOM % 10)) -lt 3 ]; then
                echo "🚨 ALERT: Simulated threshold breach detected"
              fi
              
              echo "✅ Monitoring check completed"
          volumes:
          - name: monitoring-scripts
            configMap:
              name: monitoring-scenarios
              defaultMode: 0755
EOF

echo "✅ Monitoring simulation setup complete"
echo "📊 Monitoring will run every 5 minutes"
echo "🔍 View logs: kubectl logs -n dataflow-db -l job-name=dataflow-monitoring-simulation"
```

---

## 🎯 Scenario Activation

### Quick Setup (2-3 minutes)

```bash
# Make scripts executable and run basic setup
chmod +x *.sh

# Start storage pressure simulation
./simulate_storage_pressure.sh

# Monitor initial impact
kubectl get pods -n dataflow-db -w
```

### Full Scenario Activation (5-8 minutes)

```bash
# Full chaos simulation
./simulate_storage_pressure.sh
./simulate_migration_scenarios.sh  
./setup_monitoring_chaos.sh

# Monitor all scenarios
kubectl get jobs,pods,networkpolicies -n dataflow-db
```

---

## 🔧 Environment Cleanup

### Scenario Cleanup

```bash
#!/bin/bash
# File: cleanup_environment.sh

echo "🧹 Cleaning up DataFlow migration scenarios..."

# Stop all simulation jobs
kubectl delete jobs -n dataflow-db -l scenario=simulation --ignore-not-found=true

# Remove network policies
kubectl delete networkpolicy migration-network-chaos -n dataflow-db --ignore-not-found=true

# Clean up monitoring
kubectl delete cronjob dataflow-monitoring-simulation -n dataflow-db --ignore-not-found=true

# Remove test data (optional)
# kubectl exec -n dataflow-db postgres-cluster-0 -- psql -U dataflow_user -d dataflow -c "DROP TABLE IF EXISTS legacy_corrupted_data;"

echo "✅ Environment cleanup completed"
```

---

## 📊 Expected Environment State

After setup, you should have:

- [ ] Storage pressure simulation jobs running
- [ ] Database with legacy corruption scenarios
- [ ] Production load simulation active
- [ ] Monitoring and alerting scenarios
- [ ] Network chaos (intermittent)

**Success Indicators:**

```bash
# Should show multiple jobs and scenarios
kubectl get jobs -n dataflow-db
kubectl get pods -n dataflow-db | grep -E "(Running|Completed)"
kubectl top pods -n dataflow-db  # Resource usage visible
```

This realistic environment setup provides authentic database migration challenges that mirror real-world scenarios at DataFlow Solutions, giving you hands-on experience with production-grade storage and StatefulSet management issues.

---

*🎯 **Environment Ready!** Your DataFlow Solutions database migration lab environment is now active with realistic challenges and monitoring scenarios.*
