# Database Operations and Performance - GCP Professional Cloud Database Engineer

## Overview

Comprehensive guide to database operations, performance tuning, monitoring, backup and recovery, and operational best practices for GCP databases. This guide covers operational excellence for Cloud SQL, Cloud Spanner, Bigtable, and Firestore.

**Key Topics**:
- Performance monitoring and optimization for all database services
- Backup and disaster recovery strategies with RPO/RTO considerations
- High availability configurations and failover testing
- Operations automation and maintenance windows
- Troubleshooting and incident response
- Capacity planning and scaling strategies

**Exam Focus**: The Professional Cloud Database Engineer exam heavily tests operations knowledge including performance troubleshooting, backup/recovery procedures, HA configurations, and operational decision-making under constraints.

## Cloud SQL Operations and Performance

### Cloud SQL Connection Management

**Cloud SQL Proxy**:
```bash
# Download and install Cloud SQL Proxy
curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.8.0/cloud-sql-proxy.linux.amd64
chmod +x cloud-sql-proxy

# Start proxy with automatic IAM authentication
./cloud-sql-proxy --auto-iam-authn PROJECT:REGION:INSTANCE

# Start proxy with private IP
./cloud-sql-proxy --private-ip PROJECT:REGION:INSTANCE

# Start proxy with custom port
./cloud-sql-proxy --port 5432 PROJECT:REGION:INSTANCE
```

**Private IP Configuration**:
```bash
# Create instance with private IP
gcloud sql instances create private-instance \
  --database-version=POSTGRES_15 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/vpc-network \
  --no-assign-ip

# Configure private service connection
gcloud compute addresses create google-managed-services-vpc-network \
  --global \
  --purpose=VPC_PEERING \
  --prefix-length=16 \
  --network=vpc-network

gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --ranges=google-managed-services-vpc-network \
  --network=vpc-network
```

**Connection Pooling Best Practices**:
```python
import sqlalchemy
from sqlalchemy.pool import NullPool

# Production connection pool configuration
def create_db_pool():
    return sqlalchemy.create_engine(
        sqlalchemy.engine.url.URL.create(
            drivername="postgresql+pg8000",
            username="db_user",
            password="db_password",
            host="10.0.0.3",  # Private IP
            port=5432,
            database="production_db"
        ),
        # Connection pool settings
        pool_size=5,  # Connections to keep open
        max_overflow=10,  # Additional connections under load
        pool_timeout=30,  # Seconds to wait for connection
        pool_recycle=1800,  # Recycle connections after 30 min
        pool_pre_ping=True,  # Verify connections before use
    )

# For serverless environments (Cloud Functions, Cloud Run)
def create_serverless_pool():
    return sqlalchemy.create_engine(
        connection_url,
        pool_size=1,  # Minimize connections per instance
        max_overflow=0,
        pool_timeout=30,
        pool_pre_ping=True,
        poolclass=NullPool  # No pooling for short-lived instances
    )
```

**Exam Tip**: Cloud SQL Proxy provides secure connections without managing SSL certificates and supports automatic IAM authentication. For production, always use private IP with VPC peering to avoid exposing database to internet.

### Cloud SQL Query Performance

**Query Optimization with EXPLAIN**:
```sql
-- Use EXPLAIN to analyze queries
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT c.name, COUNT(o.order_id)
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.created_at > '2024-01-01'
GROUP BY c.name;

-- Sample EXPLAIN output interpretation:
-- Seq Scan → Missing index, full table scan
-- Index Scan → Using index efficiently
-- Nested Loop → Join strategy (good for small datasets)
-- Hash Join → Join strategy (good for large datasets)
-- Sort → Expensive operation, may need index

-- Create appropriate indexes
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_customers_created ON customers(created_at);

-- Partial index for common queries
CREATE INDEX idx_active_customers ON customers(customer_id) WHERE status = 'active';

-- Covering index to avoid table lookups
CREATE INDEX idx_customer_email_covering ON customers(status) INCLUDE (customer_id, email);

-- Avoid SELECT *
SELECT customer_id, email FROM customers WHERE status = 'active';
```

**Query Insights Configuration**:
```bash
# Enable Query Insights with full configuration
gcloud sql instances patch INSTANCE \
  --insights-config-query-insights-enabled \
  --insights-config-query-string-length=4096 \
  --insights-config-record-application-tags=true \
  --insights-config-record-client-address=true \
  --insights-config-query-plans-per-minute=5

# Query Insights tracks:
# - Top queries by execution time
# - Query execution plans
# - Query frequency and patterns
# - Lock wait times
# - CPU and I/O metrics per query
```

**Analyzing Query Insights**:
```python
from google.cloud import sql_v1

def analyze_query_insights(project_id, instance_id):
    """Retrieve and analyze query insights"""
    client = sql_v1.SqlInstancesServiceClient()

    # Get query insights
    request = sql_v1.ListQueryInsightsRequest(
        project=project_id,
        instance=instance_id,
        start_time="2024-01-01T00:00:00Z",
        end_time="2024-01-02T00:00:00Z"
    )

    insights = client.list_query_insights(request=request)

    # Identify problematic queries
    slow_queries = [
        q for q in insights
        if q.avg_latency_seconds > 1.0  # Queries > 1 second
    ]

    return slow_queries
```

**Performance Tuning Flags**:
```bash
# PostgreSQL performance flags
gcloud sql instances patch pg-instance \
  --database-flags=\
shared_buffers=262144,\
effective_cache_size=786432,\
maintenance_work_mem=524288,\
work_mem=16384,\
max_connections=200,\
random_page_cost=1.1,\
effective_io_concurrency=200

# MySQL performance flags
gcloud sql instances patch mysql-instance \
  --database-flags=\
innodb_buffer_pool_size=8589934592,\
max_connections=1000,\
innodb_log_file_size=536870912,\
query_cache_type=1,\
query_cache_size=67108864
```

**Exam Tip**: Query Insights is critical for production troubleshooting. Enable it before performance issues occur. For exam scenarios asking about slow query identification, Query Insights is the first-choice answer.

### Cloud SQL Backup and Recovery Operations

**Automated Backup Configuration**:
```bash
# Configure automated backups with all options
gcloud sql instances patch INSTANCE \
  --backup-start-time=03:00 \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7 \
  --transaction-log-retention-days=7 \
  --backup-location=us  # Multi-region backup location

# Backups are:
# - Automated daily
# - Encrypted at rest
# - Stored in multi-region by default
# - Include binary logs for PITR

# Create on-demand backup before risky operations
gcloud sql backups create \
  --instance=INSTANCE \
  --description="Pre-upgrade backup $(date +%Y-%m-%d)"

# List all backups
gcloud sql backups list --instance=INSTANCE \
  --filter="status:SUCCESSFUL" \
  --limit=10
```

**Point-in-Time Recovery (PITR)**:
```bash
# Enable PITR (requires binary logging)
gcloud sql instances patch INSTANCE \
  --enable-bin-log \
  --retained-transaction-log-days=7

# PITR capabilities:
# - Restore to any point within retention window
# - Granularity down to the second
# - Creates new instance (doesn't overwrite)
# - Typical RPO: seconds to minutes

# Restore to specific timestamp
gcloud sql instances clone SOURCE_INSTANCE TARGET_INSTANCE \
  --point-in-time='2024-01-15T10:30:45.000Z'

# Restore to most recent restorable time
gcloud sql backups describe BACKUP_ID \
  --instance=INSTANCE

gcloud sql instances clone SOURCE_INSTANCE recovered-instance \
  --point-in-time='2024-01-15T23:59:59.999Z'
```

**Export Operations**:
```bash
# Full database export (SQL format)
gcloud sql export sql INSTANCE gs://bucket/backups/backup-$(date +%Y%m%d).sql \
  --database=production_db \
  --offload  # Use serverless export for large databases

# Export specific tables
gcloud sql export sql INSTANCE gs://bucket/backup.sql \
  --database=production_db \
  --table=customers,orders,transactions

# Export to CSV for data analysis
gcloud sql export csv INSTANCE gs://bucket/data.csv \
  --database=production_db \
  --query="SELECT * FROM customers WHERE created_at > '2024-01-01'"

# Import data
gcloud sql import sql INSTANCE gs://bucket/backup.sql \
  --database=production_db
```

**Backup RPO/RTO Analysis**:
```
Cloud SQL Backup Strategy:

Automated Backups:
- RPO: Up to 24 hours (daily backups)
- RTO: 15-30 minutes (restore time)
- Use case: Standard recovery

PITR (Binary Logs):
- RPO: Seconds (continuous log shipping)
- RTO: 15-30 minutes (restore + log replay)
- Use case: Critical data protection

Exports to Cloud Storage:
- RPO: Depends on export frequency
- RTO: Longer (requires import operation)
- Use case: Long-term archival, cross-region DR

Recommended Strategy:
1. Automated backups: 30 days retention
2. PITR enabled: 7 days transaction logs
3. Weekly exports to GCS: Different region
4. Monthly exports: Nearline/Coldline for compliance
```

**Exam Tip**: For questions about minimizing data loss, PITR with binary logging is the answer (RPO in seconds). For cost optimization with moderate RPO requirements, automated daily backups are sufficient.

### Cloud SQL Maintenance and Upgrades

**Maintenance Window Configuration**:
```bash
# Configure maintenance window
gcloud sql instances patch INSTANCE \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=04 \
  --maintenance-release-channel=production

# Maintenance release channels:
# - production: Stable, tested updates (default)
# - preview: Early access to features
# - week5: Updates only during 5-week months

# Deny maintenance during critical periods
gcloud sql instances patch INSTANCE \
  --deny-maintenance-period-start-date=2024-12-15 \
  --deny-maintenance-period-end-date=2025-01-05 \
  --deny-maintenance-period-time=00:00:00

# Maintenance operations include:
# - Database version updates
# - OS patching
# - Infrastructure improvements
# - Typically 1-2 minutes downtime for HA instances
```

**Database Version Upgrades**:
```bash
# Check current version
gcloud sql instances describe INSTANCE \
  --format="value(databaseVersion)"

# Upgrade strategy (ALWAYS test first):
# 1. Create clone for testing
gcloud sql instances clone prod-instance test-upgrade-instance

# 2. Test upgrade on clone
gcloud sql instances patch test-upgrade-instance \
  --database-version=POSTGRES_15

# 3. Run application tests against clone
# 4. Verify performance and functionality
# 5. Create backup before production upgrade
gcloud sql backups create --instance=prod-instance

# 6. Upgrade production during maintenance window
gcloud sql instances patch prod-instance \
  --database-version=POSTGRES_15

# Version upgrade considerations:
# - PostgreSQL: Major version upgrades cause downtime
# - MySQL: Some upgrades are zero-downtime
# - Always check compatibility notes
# - Cannot downgrade (must restore from backup)
```

**Exam Tip**: Maintenance windows are honored for planned maintenance only, not emergency security patches. For high-availability requirements, deny maintenance during peak business periods.

## Cloud Spanner Operations and Performance

### Cloud Spanner Scaling Operations

**Manual Node Scaling**:
```bash
# View current instance configuration
gcloud spanner instances describe INSTANCE \
  --format="value(nodeCount, processingUnits)"

# Scale up nodes (recommended for high priority CPU > 65%)
gcloud spanner instances update INSTANCE \
  --nodes=5

# Scale using processing units for finer control
# 1 node = 1000 processing units
gcloud spanner instances update INSTANCE \
  --processing-units=2500  # 2.5 nodes

# Regional vs Multi-region configurations:
# Regional: 1-999 nodes per instance
# Multi-region: 3-999 nodes per instance (minimum 3)

# Scaling considerations:
# - Scale up: Immediate (no downtime)
# - Scale down: Gradual (monitors CPU to prevent issues)
# - Cost: $0.90/node-hour (regional), $9/node-hour (multi-region)
```

**Autoscaling Configuration**:
```bash
# Enable autoscaling using Cloud Functions
gcloud functions deploy spanner-autoscaler \
  --runtime=python310 \
  --trigger-topic=spanner-metrics \
  --entry-point=scale_spanner \
  --env-vars-file=config.yaml

# Autoscaling configuration (config.yaml):
# INSTANCE_ID: production-spanner
# MIN_NODES: 3
# MAX_NODES: 10
# HIGH_CPU_THRESHOLD: 65
# LOW_CPU_THRESHOLD: 35
# SCALE_UP_INCREMENT: 1
# SCALE_DOWN_INCREMENT: 1
```

**Autoscaling Implementation**:
```python
from google.cloud import spanner_admin_instance_v1
from google.cloud import monitoring_v3
import os

def scale_spanner(event, context):
    """Auto-scale Spanner based on CPU metrics"""
    instance_id = os.environ['INSTANCE_ID']
    project_id = os.environ['GCP_PROJECT']

    # Get current CPU utilization
    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{project_id}"

    query = f"""
    fetch spanner_instance
    | metric 'spanner.googleapis.com/instance/cpu/utilization'
    | filter resource.instance_id == '{instance_id}'
    | group_by 1m, [value_utilization_mean: mean(value.utilization)]
    | within 5m
    """

    results = client.query_time_series(request={
        "name": project_name,
        "query": query
    })

    avg_cpu = calculate_average(results)

    # Get current node count
    instance_admin_client = spanner_admin_instance_v1.InstanceAdminClient()
    instance_name = f"{project_name}/instances/{instance_id}"
    instance = instance_admin_client.get_instance(name=instance_name)
    current_nodes = instance.node_count

    # Scaling logic
    if avg_cpu > 65 and current_nodes < MAX_NODES:
        new_nodes = current_nodes + 1
        scale_instance(instance_name, new_nodes)
        print(f"Scaled up from {current_nodes} to {new_nodes} nodes")
    elif avg_cpu < 35 and current_nodes > MIN_NODES:
        new_nodes = current_nodes - 1
        scale_instance(instance_name, new_nodes)
        print(f"Scaled down from {current_nodes} to {new_nodes} nodes")
```

**Exam Tip**: For exam questions about Spanner performance, remember: High Priority CPU > 65% = scale up immediately. Total CPU is less critical. Multi-region instances require minimum 3 nodes.

### Cloud Spanner Query Performance

**Query Optimization with Query Plans**:
```sql
-- View query execution plan
@{FORCE_PLAN=false}
EXPLAIN ANALYZE
SELECT o.OrderId, c.CustomerName, o.OrderTotal
FROM Orders o
JOIN Customers c ON o.CustomerId = c.CustomerId
WHERE o.OrderDate > '2024-01-01'
ORDER BY o.OrderTotal DESC
LIMIT 100;

-- Query plan shows:
-- - Scan type (Index Scan vs Table Scan)
-- - Join strategy (Hash Join, Merge Join)
-- - Estimated vs actual rows
-- - Execution time breakdown

-- Force specific index for optimization
SELECT * FROM Orders@{FORCE_INDEX=OrdersByDate}
WHERE OrderDate > '2024-01-01';

-- Use HASH JOIN hint for large tables
SELECT * FROM Orders o @{JOIN_METHOD=HASH_JOIN}
JOIN OrderItems oi ON o.OrderId = oi.OrderId;

-- Batch reads for efficiency (reduces latency)
SELECT * FROM Orders
WHERE OrderId IN UNNEST(@order_ids);

-- Use STRUCT for single-RPC reads
SELECT STRUCT(
  OrderId,
  CustomerName,
  ARRAY(SELECT ItemName FROM OrderItems WHERE OrderId = Orders.OrderId)
) FROM Orders;
```

**Secondary Index Strategies**:
```sql
-- Create standard secondary index
CREATE INDEX OrdersByCustomer ON Orders(CustomerId, OrderDate DESC);

-- NULL_FILTERED index (excludes NULL values, smaller index)
CREATE NULL_FILTERED INDEX ActiveOrdersIndex
ON Orders(CustomerId, OrderDate DESC)
WHERE Status = 'ACTIVE';

-- Storing clause (avoid table lookup)
CREATE INDEX OrdersByCustomerStoring
ON Orders(CustomerId, OrderDate DESC)
STORING (OrderTotal, Status);

-- Unique index with NULL filtering
CREATE UNIQUE NULL_FILTERED INDEX UniqueActiveEmail
ON Customers(Email)
WHERE Status = 'ACTIVE';

-- Index interleaving for parent-child tables
CREATE INDEX OrderItemsByProduct ON OrderItems(ProductId)
INTERLEAVE IN Orders;
```

**Hotspot Detection and Resolution**:
```sql
-- Avoid sequential primary keys (creates hotspots)
-- Bad: Auto-incrementing ID
CREATE TABLE Orders (
  OrderId INT64 NOT NULL,
  -- All writes go to same split
) PRIMARY KEY (OrderId);

-- Good: Use UUID or hash-based key
CREATE TABLE Orders (
  OrderId STRING(36) NOT NULL,  -- UUID
  CreatedAt TIMESTAMP NOT NULL,
) PRIMARY KEY (OrderId);

-- Good: Reverse timestamp for time-series
CREATE TABLE Metrics (
  SensorId STRING(36) NOT NULL,
  Timestamp INT64 NOT NULL,  -- Reverse timestamp
) PRIMARY KEY (SensorId, Timestamp DESC);

-- Monitor for hotspots using Query Stats
SELECT
  interval_end,
  text,
  cpu_time_sec,
  rows_returned,
  rows_scanned,
  cpu_time_sec / rows_returned as cpu_per_row
FROM SPANNER_SYS.QUERY_STATS_TOP_MINUTE
WHERE interval_end >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
ORDER BY cpu_time_sec DESC
LIMIT 10;
```

**Query Stats Analysis**:
```sql
-- Top queries by execution time (last hour)
SELECT
  text as query_text,
  COUNT(*) as execution_count,
  AVG(cpu_time_sec) as avg_cpu_seconds,
  AVG(rows_returned) as avg_rows_returned,
  AVG(latency_seconds) as avg_latency_seconds
FROM SPANNER_SYS.QUERY_STATS_TOP_HOUR
WHERE interval_end >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY text
ORDER BY avg_cpu_seconds DESC
LIMIT 20;

-- Lock wait statistics
SELECT
  text as query_text,
  AVG(lock_wait_seconds) as avg_lock_wait,
  COUNT(*) as execution_count
FROM SPANNER_SYS.QUERY_STATS_TOP_HOUR
WHERE lock_wait_seconds > 0
  AND interval_end >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY text
ORDER BY avg_lock_wait DESC;

-- Find queries with high scan-to-return ratio (inefficient)
SELECT
  text as query_text,
  AVG(rows_scanned) as avg_rows_scanned,
  AVG(rows_returned) as avg_rows_returned,
  AVG(rows_scanned) / AVG(rows_returned) as scan_return_ratio
FROM SPANNER_SYS.QUERY_STATS_TOP_HOUR
WHERE rows_returned > 0
  AND interval_end >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY text
HAVING scan_return_ratio > 10
ORDER BY scan_return_ratio DESC;
```

**Exam Tip**: For Spanner query optimization questions, look for: 1) Secondary indexes to avoid full table scans, 2) STORING clause to avoid table lookups, 3) Proper primary key design to avoid hotspots.

### Cloud Spanner Backup and Recovery

**Backup Operations**:
```bash
# Create on-demand backup
gcloud spanner backups create backup-20240115 \
  --instance=production-instance \
  --database=production-db \
  --retention-period=90d \
  --version-time='2024-01-15T10:00:00Z'

# Create backup with encryption
gcloud spanner backups create encrypted-backup \
  --instance=production-instance \
  --database=production-db \
  --retention-period=30d \
  --kms-key=projects/PROJECT/locations/REGION/keyRings/RING/cryptoKeys/KEY

# List backups
gcloud spanner backups list \
  --instance=production-instance \
  --filter="state:READY" \
  --format="table(name,database,createTime,expireTime,sizeBytes)"

# Update backup retention
gcloud spanner backups update backup-20240115 \
  --instance=production-instance \
  --expiration-date=2024-07-15T00:00:00Z

# Delete backup
gcloud spanner backups delete backup-20240115 \
  --instance=production-instance
```

**Restore Operations**:
```bash
# Restore to new database
gcloud spanner databases restore \
  --source-backup=backup-20240115 \
  --source-instance=production-instance \
  --destination-database=restored-db \
  --destination-instance=production-instance

# Restore to different instance
gcloud spanner databases restore \
  --source-backup=backup-20240115 \
  --source-instance=us-central1-instance \
  --destination-database=dr-database \
  --destination-instance=europe-west1-instance

# Restore with encryption
gcloud spanner databases restore \
  --source-backup=encrypted-backup \
  --source-instance=production-instance \
  --destination-database=restored-db \
  --destination-instance=production-instance \
  --kms-key=projects/PROJECT/locations/REGION/keyRings/RING/cryptoKeys/KEY
```

**Automated Backup Strategy**:
```python
from google.cloud import spanner_admin_database_v1
from datetime import datetime, timedelta

def create_scheduled_backup(instance_id, database_id, project_id):
    """Create backup with naming convention and retention"""
    client = spanner_admin_database_v1.DatabaseAdminClient()

    # Backup naming: db-YYYYMMDD-HHMM
    backup_id = f"{database_id}-{datetime.now().strftime('%Y%m%d-%H%M')}"

    # Create backup
    instance_path = client.instance_path(project_id, instance_id)
    database_path = client.database_path(project_id, instance_id, database_id)
    backup_path = f"{instance_path}/backups/{backup_id}"

    expire_time = datetime.now() + timedelta(days=30)

    backup = spanner_admin_database_v1.Backup(
        database=database_path,
        expire_time=expire_time,
    )

    operation = client.create_backup(
        parent=instance_path,
        backup_id=backup_id,
        backup=backup
    )

    print(f"Creating backup {backup_id}...")
    backup = operation.result(timeout=3600)
    print(f"Backup created: {backup.name}")

    # Cleanup old backups (retain last 30)
    delete_old_backups(client, instance_path, 30)

def delete_old_backups(client, instance_path, retain_count):
    """Delete backups beyond retention count"""
    backups = client.list_backups(parent=instance_path)
    backup_list = sorted(backups, key=lambda b: b.create_time, reverse=True)

    for backup in backup_list[retain_count:]:
        client.delete_backup(name=backup.name)
        print(f"Deleted old backup: {backup.name}")
```

**Spanner RPO/RTO Targets**:
```
Backup Strategy for Cloud Spanner:

On-Demand Backups:
- RPO: Time of last backup (configurable)
- RTO: 15-30 minutes for small DBs, hours for TB+
- Cost: $0.10/GB/month
- Use case: Regular protection, compliance

Import/Export Operations:
- RPO: Time of last export
- RTO: Hours (depends on data size)
- Cost: Cloud Storage costs only
- Use case: Cross-region DR, archival

Multi-Region Spanner:
- RPO: Near-zero (synchronous replication)
- RTO: Zero (automatic failover)
- Cost: 9x regional cost
- Use case: Mission-critical applications

Recommended Strategy:
1. Daily automated backups: 30-day retention
2. Weekly backups: 90-day retention
3. Pre-change backups: Before schema changes
4. Multi-region for critical workloads
```

**Exam Tip**: Spanner backups are consistent snapshots and can be restored to any instance in any region. For cross-region DR, backups are more cost-effective than multi-region instances unless RPO/RTO requirements are very strict.

## Bigtable Operations and Performance

### Bigtable Cluster Operations

**Cluster Scaling**:
```bash
# View current cluster configuration
gcloud bigtable clusters describe CLUSTER \
  --instance=INSTANCE \
  --format="json"

# Scale cluster nodes
gcloud bigtable clusters update CLUSTER \
  --instance=INSTANCE \
  --num-nodes=10

# Scaling guidelines:
# - Storage per node: 8TB HDD or 2.5TB SSD
# - Throughput per node:
#   - HDD: 10-20MB/s reads, 10MB/s writes
#   - SSD: 200-220MB/s reads, 180MB/s writes
# - Scale up: Immediate (no downtime)
# - Scale down: Gradual (monitors load)
```

**Autoscaling Configuration**:
```bash
# Enable autoscaling for cluster
gcloud bigtable clusters update CLUSTER \
  --instance=INSTANCE \
  --autoscaling-min-nodes=3 \
  --autoscaling-max-nodes=30 \
  --autoscaling-cpu-target=70

# Autoscaling considerations:
# - Monitors CPU utilization
# - Scales based on target percentage
# - Minimum 3 nodes for production
# - Scale-up: Fast (within minutes)
# - Scale-down: Conservative (prevents thrashing)
```

**Replication Configuration**:
```bash
# Add cluster for replication
gcloud bigtable clusters create CLUSTER-EU \
  --instance=INSTANCE \
  --zone=europe-west1-b \
  --num-nodes=3

# Configure replication for app profile
gcloud bigtable app-profiles create multi-cluster-profile \
  --instance=INSTANCE \
  --route-any

# Single-cluster routing (for analytics workloads)
gcloud bigtable app-profiles create analytics-profile \
  --instance=INSTANCE \
  --route-to=CLUSTER-US \
  --transactional-writes

# Replication benefits:
# - Automatic failover
# - Reduced latency (geo-distribution)
# - Increased throughput
# - Eventually consistent
```

**Exam Tip**: Bigtable replication is eventually consistent (typically < 1 second). For strongly consistent reads, use single-cluster routing. Multi-cluster routing provides automatic failover and load balancing.

### Bigtable Performance Optimization

**Key Design Best Practices**:
```python
from google.cloud import bigtable
from google.cloud.bigtable import row_filters
import hashlib

# Bad: Sequential keys (creates hotspots)
def bad_row_key(user_id, timestamp):
    return f"{timestamp}#{user_id}"  # All writes go to same tablet

# Good: Hash-based key distribution
def good_row_key(user_id, timestamp):
    # Hash prefix distributes writes
    hash_prefix = hashlib.md5(user_id.encode()).hexdigest()[:4]
    return f"{hash_prefix}#{user_id}#{timestamp}"

# Good: Reverse domain for web URLs
def url_row_key(url):
    # com.example.www instead of www.example.com
    parts = url.split('.')
    return '.'.join(reversed(parts))

# Good: Salting for time-series data
def timeseries_row_key(sensor_id, timestamp):
    # Distribute across multiple tablets
    salt = hash(sensor_id) % 100  # 100 buckets
    return f"{salt:02d}#{sensor_id}#{timestamp}"

# Query pattern example
def read_timeseries_data(table, sensor_id, start_time, end_time):
    """Read time-series with salted keys"""
    rows = []
    for salt in range(100):
        prefix = f"{salt:02d}#{sensor_id}#"
        row_range = row_range.StartingRowKey(prefix)
        # Read from each salt bucket
        partial_rows = table.read_rows(row_range=row_range)
        rows.extend(partial_rows)
    return rows
```

**Column Family and Garbage Collection**:
```python
from google.cloud import bigtable

def configure_column_families():
    """Configure column families with appropriate GC policies"""
    client = bigtable.Client(project=PROJECT_ID, admin=True)
    instance = client.instance(INSTANCE_ID)
    table = instance.table(TABLE_ID)

    # Create column families with different GC policies

    # 1. Max versions (keep last N versions)
    cf_versions = table.column_family(
        'profile',
        max_versions_gc_rule=bigtable.column_family.MaxVersionsGCRule(3)
    )

    # 2. Max age (keep data for N days)
    cf_metrics = table.column_family(
        'metrics',
        max_age_gc_rule=bigtable.column_family.MaxAgeGCRule(
            datetime.timedelta(days=7)
        )
    )

    # 3. Union (keep if ANY condition matches)
    union_rule = bigtable.column_family.GCRuleUnion([
        bigtable.column_family.MaxVersionsGCRule(2),
        bigtable.column_family.MaxAgeGCRule(datetime.timedelta(days=30))
    ])
    cf_events = table.column_family('events', gc_rule=union_rule)

    # 4. Intersection (keep only if ALL conditions match)
    intersection_rule = bigtable.column_family.GCRuleIntersection([
        bigtable.column_family.MaxVersionsGCRule(5),
        bigtable.column_family.MaxAgeGCRule(datetime.timedelta(days=90))
    ])
    cf_archive = table.column_family('archive', gc_rule=intersection_rule)

    # Create table with column families
    if not table.exists():
        table.create(column_families={
            'profile': cf_versions,
            'metrics': cf_metrics,
            'events': cf_events,
            'archive': cf_archive
        })
```

**Bulk Read and Write Operations**:
```python
from google.cloud import bigtable
from google.cloud.bigtable import row_filters

def bulk_write_data(table, data_items):
    """Efficiently write large batches of data"""
    rows = []

    for item in data_items:
        row_key = f"{item['id']}#{item['timestamp']}".encode()
        row = table.direct_row(row_key)

        # Set multiple cells
        row.set_cell('cf1', 'data', item['data'])
        row.set_cell('cf1', 'status', item['status'])
        row.set_cell('cf1', 'metadata', item['metadata'])

        rows.append(row)

        # Batch in groups of 100,000 for optimal performance
        if len(rows) >= 100000:
            table.mutate_rows(rows)
            rows = []

    # Write remaining rows
    if rows:
        table.mutate_rows(rows)

def bulk_read_with_filter(table, start_key, end_key):
    """Efficiently read with row filters"""
    # Create row range
    row_range = row_range.RowRange(
        start_key=start_key.encode(),
        end_key=end_key.encode()
    )

    # Apply filters for efficiency
    filter = row_filters.RowFilterChain([
        # Only return latest version
        row_filters.CellsPerColumnLimitFilter(1),
        # Only return specific column
        row_filters.ColumnQualifierRegexFilter(b'data'),
        # Apply value filter
        row_filters.ValueRegexFilter(b'active')
    ])

    # Read with filter
    partial_rows = table.read_rows(
        row_range=row_range,
        filter_=filter
    )

    return list(partial_rows)
```

**Key Visualizer Usage**:
```bash
# Key Visualizer is enabled by default
# Access through Cloud Console -> Bigtable -> [Instance] -> Key Visualizer

# Key Visualizer shows:
# 1. Heat maps of access patterns
# 2. Hotspot identification (dark regions)
# 3. Row key distribution
# 4. Temporal access patterns

# Interpreting Key Visualizer:
# - Bright spots = Hotspots (need better key distribution)
# - Horizontal lines = Sequential scans
# - Diagonal lines = Time-based sequential writes
# - Even distribution = Good key design

# Common patterns identified:
# 1. Sequential key hotspots → Add hash prefix
# 2. Large rows → Split into multiple rows
# 3. Empty tablets → Review key design
# 4. Uneven access → Consider table splits
```

**Performance Monitoring**:
```python
from google.cloud import monitoring_v3
from datetime import datetime, timedelta

def monitor_bigtable_metrics(project_id, instance_id):
    """Monitor key Bigtable performance metrics"""
    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{project_id}"

    # Define metrics to monitor
    metrics = [
        'bigtable.googleapis.com/cluster/cpu_load',
        'bigtable.googleapis.com/cluster/storage_utilization',
        'bigtable.googleapis.com/server/request_count',
        'bigtable.googleapis.com/server/latencies',
    ]

    end_time = datetime.utcnow()
    start_time = end_time - timedelta(hours=1)

    interval = monitoring_v3.TimeInterval({
        "start_time": {"seconds": int(start_time.timestamp())},
        "end_time": {"seconds": int(end_time.timestamp())},
    })

    for metric_type in metrics:
        results = client.list_time_series(
            request={
                "name": project_name,
                "filter": f'metric.type="{metric_type}" AND resource.instance_id="{instance_id}"',
                "interval": interval,
                "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL,
            }
        )

        for result in results:
            print(f"Metric: {metric_type}")
            for point in result.points:
                print(f"  Value: {point.value.double_value}, Time: {point.interval.end_time}")
```

**Exam Tip**: For Bigtable performance questions, focus on: 1) Key design to avoid hotspots (use hash prefixes), 2) Appropriate node count (based on storage and throughput), 3) GC policies to manage storage costs.

### Bigtable Backup and Recovery

**Backup Operations**:
```bash
# Create backup
gcloud bigtable backups create backup-20240115 \
  --instance=INSTANCE \
  --cluster=CLUSTER \
  --table=TABLE \
  --retention-period=30d \
  --async

# List backups
gcloud bigtable backups list \
  --instance=INSTANCE \
  --cluster=CLUSTER \
  --format="table(name,sourceTable,expireTime,sizeBytes,state)"

# Describe backup
gcloud bigtable backups describe backup-20240115 \
  --instance=INSTANCE \
  --cluster=CLUSTER

# Update backup expiration
gcloud bigtable backups update backup-20240115 \
  --instance=INSTANCE \
  --cluster=CLUSTER \
  --retention-period=60d

# Delete backup
gcloud bigtable backups delete backup-20240115 \
  --instance=INSTANCE \
  --cluster=CLUSTER
```

**Restore Operations**:
```bash
# Restore to new table
gcloud bigtable backups restore backup-20240115 \
  --instance=INSTANCE \
  --cluster=CLUSTER \
  --destination=RESTORED_TABLE \
  --async

# Restore to different instance
gcloud bigtable backups restore backup-20240115 \
  --source-instance=SOURCE_INSTANCE \
  --source-cluster=SOURCE_CLUSTER \
  --destination-instance=DEST_INSTANCE \
  --destination=RESTORED_TABLE
```

**Automated Backup Script**:
```python
from google.cloud import bigtable_admin_v2
from datetime import datetime, timedelta

def create_automated_backup(project_id, instance_id, cluster_id, table_id):
    """Create automated backup with retention management"""
    client = bigtable_admin_v2.BigtableTableAdminClient()

    # Create backup
    cluster_path = client.cluster_path(project_id, instance_id, cluster_id)
    table_path = client.table_path(project_id, instance_id, table_id)

    backup_id = f"{table_id}-{datetime.now().strftime('%Y%m%d-%H%M')}"
    expire_time = datetime.now() + timedelta(days=30)

    backup = bigtable_admin_v2.Backup(
        source_table=table_path,
        expire_time=expire_time
    )

    operation = client.create_backup(
        parent=cluster_path,
        backup_id=backup_id,
        backup=backup
    )

    print(f"Creating backup: {backup_id}")
    result = operation.result(timeout=3600)
    print(f"Backup created: {result.name}")

    # Cleanup old backups
    cleanup_old_backups(client, cluster_path, retain_count=30)

def cleanup_old_backups(client, cluster_path, retain_count):
    """Delete backups beyond retention count"""
    backups = client.list_backups(parent=cluster_path)
    backup_list = sorted(
        backups,
        key=lambda b: b.start_time,
        reverse=True
    )

    for backup in backup_list[retain_count:]:
        client.delete_backup(name=backup.name)
        print(f"Deleted old backup: {backup.name}")
```

**Bigtable RPO/RTO Strategy**:
```
Bigtable Backup and Recovery Options:

Backups:
- RPO: Time of last backup
- RTO: 30 minutes to hours (depends on table size)
- Cost: $0.15/GB/month
- Use case: Disaster recovery, compliance

Replication (Multi-cluster):
- RPO: < 1 second (eventually consistent)
- RTO: Seconds (automatic failover)
- Cost: 2x storage + compute
- Use case: HA, geo-distribution

Export to Cloud Storage:
- RPO: Time of last export
- RTO: Hours (import operation)
- Cost: Cloud Storage costs
- Use case: Long-term archival, analytics

Recommended Strategy:
1. Enable replication for critical tables
2. Daily backups: 30-day retention
3. Weekly exports to GCS for compliance
4. Test restore procedures quarterly
```

**Exam Tip**: Bigtable backups are taken at the cluster level and can be restored to any instance. For lowest RPO/RTO, use replication across clusters. Backups are for disaster recovery, not real-time failover.

## Firestore Operations and Performance

### Firestore Index Management

**Composite Index Creation**:
```javascript
// Query requiring composite index
db.collection('orders')
  .where('customerId', '==', 'user123')
  .where('status', '==', 'pending')
  .orderBy('orderDate', 'desc')
  .limit(10);

// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "orders",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "customerId", "order": "ASCENDING" },
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "orderDate", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "products",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "category", "order": "ASCENDING" },
        { "fieldPath": "price", "order": "ASCENDING" },
        { "fieldPath": "rating", "order": "DESCENDING" }
      ]
    }
  ]
}
```

**Deploy Indexes**:
```bash
# Deploy indexes from configuration file
firebase deploy --only firestore:indexes

# Create single index via gcloud
gcloud firestore indexes composite create \
  --collection-group=orders \
  --query-scope=COLLECTION \
  --field-config field-path=customerId,order=ASCENDING \
  --field-config field-path=status,order=ASCENDING \
  --field-config field-path=orderDate,order=DESCENDING

# List indexes
gcloud firestore indexes composite list \
  --format="table(name,state,queryScope)"

# Describe specific index
gcloud firestore indexes composite describe INDEX_ID

# Delete index
gcloud firestore indexes composite delete INDEX_ID
```

**Index Exemptions**:
```javascript
// firestore.indexes.json - Exemption for large arrays
{
  "fieldOverrides": [
    {
      "collectionGroup": "products",
      "fieldPath": "tags",
      "indexes": []  // Disable automatic indexing for 'tags' field
    },
    {
      "collectionGroup": "documents",
      "fieldPath": "content",
      "indexes": []  // Large text fields should be exempted
    }
  ]
}
```

**Exam Tip**: Firestore automatically creates single-field indexes. Composite indexes are required for queries with multiple WHERE clauses or WHERE + ORDER BY. Array-contains queries with range filters need composite indexes.

### Firestore Query Performance

**Efficient Query Patterns**:
```javascript
const admin = require('firebase-admin');
const db = admin.firestore();

// Good: Use composite index
async function efficientQuery() {
  const snapshot = await db.collection('orders')
    .where('customerId', '==', 'user123')
    .where('status', 'in', ['pending', 'processing'])
    .orderBy('createdAt', 'desc')
    .limit(20)
    .get();

  return snapshot.docs.map(doc => doc.data());
}

// Bad: Fetching entire collection
async function inefficientQuery() {
  const snapshot = await db.collection('orders').get();
  return snapshot.docs
    .filter(doc => doc.data().customerId === 'user123')
    .map(doc => doc.data());
}

// Good: Paginated queries
async function paginatedQuery(lastDoc = null, pageSize = 20) {
  let query = db.collection('orders')
    .where('status', '==', 'completed')
    .orderBy('createdAt', 'desc')
    .limit(pageSize);

  if (lastDoc) {
    query = query.startAfter(lastDoc);
  }

  const snapshot = await query.get();
  return {
    docs: snapshot.docs.map(doc => doc.data()),
    lastDoc: snapshot.docs[snapshot.docs.length - 1]
  };
}

// Good: Denormalization for read performance
async function denormalizedRead(orderId) {
  // Store customer name directly in order (avoid join)
  const orderDoc = await db.collection('orders').doc(orderId).get();
  return orderDoc.data();  // Includes customerName, no join needed
}

// Good: Batch reads
async function batchRead(orderIds) {
  const batch = orderIds.map(id =>
    db.collection('orders').doc(id).get()
  );
  const snapshots = await Promise.all(batch);
  return snapshots.map(snap => snap.data());
}
```

**Query Limitations and Workarounds**:
```javascript
// Limitation: No OR queries (use multiple queries)
async function orQuery(customerId) {
  const [pendingOrders, processingOrders] = await Promise.all([
    db.collection('orders')
      .where('customerId', '==', customerId)
      .where('status', '==', 'pending')
      .get(),
    db.collection('orders')
      .where('customerId', '==', customerId)
      .where('status', '==', 'processing')
      .get()
  ]);

  return [...pendingOrders.docs, ...processingOrders.docs];
}

// Limitation: Array-contains with other filters
// Workaround: Use map with boolean values
// Instead of: tags: ['electronics', 'sale']
// Use: tags: { electronics: true, sale: true }
async function categoryQuery() {
  return await db.collection('products')
    .where('tags.electronics', '==', true)
    .where('price', '>', 100)
    .get();
}

// Limitation: != operator (use > and < queries)
async function notEqualQuery(customerId) {
  const [lessThan, greaterThan] = await Promise.all([
    db.collection('orders')
      .where('customerId', '<', customerId)
      .get(),
    db.collection('orders')
      .where('customerId', '>', customerId)
      .get()
  ]);

  return [...lessThan.docs, ...greaterThan.docs];
}
```

### Firestore Batch Operations

**Batch Writes**:
```javascript
// Batch writes (up to 500 operations)
async function batchWrite(items) {
  const batch = db.batch();

  items.forEach(item => {
    const docRef = db.collection('items').doc(item.id);
    batch.set(docRef, item);
  });

  // Atomic: Either all succeed or all fail
  await batch.commit();
}

// Batched updates with error handling
async function updateBatch(updates) {
  const batchSize = 500;
  const batches = [];

  for (let i = 0; i < updates.length; i += batchSize) {
    const chunk = updates.slice(i, i + batchSize);
    const batch = db.batch();

    chunk.forEach(update => {
      const docRef = db.collection('orders').doc(update.id);
      batch.update(docRef, { status: update.status });
    });

    batches.push(batch.commit());
  }

  // Execute all batches in parallel
  await Promise.all(batches);
}

// Transactions for read-modify-write
async function transferInventory(fromId, toId, amount) {
  return await db.runTransaction(async (transaction) => {
    const fromRef = db.collection('inventory').doc(fromId);
    const toRef = db.collection('inventory').doc(toId);

    const fromDoc = await transaction.get(fromRef);
    const toDoc = await transaction.get(toRef);

    const fromQuantity = fromDoc.data().quantity;
    const toQuantity = toDoc.data().quantity;

    if (fromQuantity < amount) {
      throw new Error('Insufficient inventory');
    }

    transaction.update(fromRef, { quantity: fromQuantity - amount });
    transaction.update(toRef, { quantity: toQuantity + amount });
  });
}
```

**Exam Tip**: Firestore batches are atomic (all or nothing) and limited to 500 operations. Transactions provide read-modify-write atomicity but have retry logic and may execute multiple times.

### Firestore Export and Import

**Export Operations**:
```bash
# Export entire database
gcloud firestore export gs://bucket/firestore-backup-$(date +%Y%m%d)

# Export specific collections
gcloud firestore export gs://bucket/firestore-backup \
  --collection-ids=users,orders,products

# Export with namespace (multi-tenant)
gcloud firestore export gs://bucket/firestore-backup \
  --collection-ids=users \
  --namespace-ids=tenant1,tenant2

# Scheduled exports via Cloud Scheduler
gcloud scheduler jobs create http firestore-daily-backup \
  --schedule="0 2 * * *" \
  --uri="https://firestore.googleapis.com/v1/projects/PROJECT/databases/(default):exportDocuments" \
  --message-body='{"outputUriPrefix":"gs://bucket/scheduled-backups/$(date +%Y%m%d)"}' \
  --oauth-service-account-email=SERVICE_ACCOUNT@PROJECT.iam.gserviceaccount.com
```

**Import Operations**:
```bash
# Import from export
gcloud firestore import gs://bucket/firestore-backup/2024-01-15

# Import specific collections
gcloud firestore import gs://bucket/firestore-backup/2024-01-15 \
  --collection-ids=users,orders

# Import considerations:
# - Documents are overwritten if they exist
# - Does not delete existing documents
# - Indexes must exist before import
# - Can take hours for large datasets
```

**Firestore RPO/RTO Strategy**:
```
Firestore Backup and Recovery:

Managed Exports:
- RPO: Time of last export (daily recommended)
- RTO: 2-4 hours (import operation)
- Cost: Cloud Storage costs only
- Use case: Disaster recovery, compliance

Multi-region Firestore:
- RPO: Near-zero (synchronous replication)
- RTO: Zero (automatic failover)
- Cost: Higher than single-region
- Use case: High availability requirements

Recommended Strategy:
1. Daily automated exports to GCS
2. Multi-region for production databases
3. Weekly exports to different region
4. Retain exports for 90 days minimum
```

**Exam Tip**: Firestore exports are the only backup mechanism. Always enable automated daily exports for production databases. Multi-region provides automatic failover but doesn't replace backups.

## Unified Monitoring and Alerting

### Cloud Monitoring Dashboards

**Comprehensive Database Dashboard**:
```python
from google.cloud import monitoring_dashboard_v1

def create_database_monitoring_dashboard(project_id):
    """Create unified database monitoring dashboard"""
    client = monitoring_dashboard_v1.DashboardsServiceClient()
    project_name = f"projects/{project_id}"

    dashboard = monitoring_dashboard_v1.Dashboard(
        display_name="Database Operations Dashboard",
        grid_layout=monitoring_dashboard_v1.GridLayout(
            widgets=[
                # Cloud SQL CPU
                monitoring_dashboard_v1.Widget(
                    title="Cloud SQL CPU Utilization",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="cloudsql.googleapis.com/database/cpu/utilization"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={"seconds": 60},
                                            per_series_aligner="ALIGN_MEAN",
                                        ),
                                    ),
                                ),
                            ),
                        ],
                        y_axis=monitoring_dashboard_v1.XyChart.Axis(
                            label="CPU %",
                            scale="LINEAR",
                        ),
                    ),
                ),
                # Spanner CPU
                monitoring_dashboard_v1.Widget(
                    title="Spanner High Priority CPU",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="spanner.googleapis.com/instance/cpu/utilization_by_priority" metric.label.priority="high"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={"seconds": 60},
                                            per_series_aligner="ALIGN_MEAN",
                                        ),
                                    ),
                                ),
                            ),
                        ],
                    ),
                ),
                # Bigtable CPU
                monitoring_dashboard_v1.Widget(
                    title="Bigtable CPU Load",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="bigtable.googleapis.com/cluster/cpu_load"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={"seconds": 60},
                                            per_series_aligner="ALIGN_MEAN",
                                        ),
                                    ),
                                ),
                            ),
                        ],
                    ),
                ),
                # Firestore Operations
                monitoring_dashboard_v1.Widget(
                    title="Firestore Operations/sec",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="firestore.googleapis.com/document/read_count"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={"seconds": 60},
                                            per_series_aligner="ALIGN_RATE",
                                        ),
                                    ),
                                ),
                            ),
                        ],
                    ),
                ),
            ],
        ),
    )

    response = client.create_dashboard(
        parent=project_name,
        dashboard=dashboard
    )

    print(f"Dashboard created: {response.name}")
    return response
```

### Alert Policies

**Critical Alert Policies**:
```python
from google.cloud import monitoring_v3

def create_database_alerts(project_id):
    """Create critical database alert policies"""
    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    # Cloud SQL High CPU Alert
    cloudsql_cpu_policy = monitoring_v3.AlertPolicy(
        display_name="Cloud SQL High CPU (>80%)",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="CPU > 80% for 5 minutes",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="cloudsql.googleapis.com/database/cpu/utilization"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.8,
                    duration={"seconds": 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner="ALIGN_MEAN",
                        ),
                    ],
                )
            )
        ],
        alert_strategy=monitoring_v3.AlertPolicy.AlertStrategy(
            auto_close={"seconds": 1800}
        ),
        combiner=monitoring_v3.AlertPolicy.ConditionCombinerType.AND,
    )

    # Spanner High Priority CPU Alert
    spanner_cpu_policy = monitoring_v3.AlertPolicy(
        display_name="Spanner High Priority CPU (>65%)",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="High Priority CPU > 65%",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="spanner.googleapis.com/instance/cpu/utilization_by_priority" metric.label.priority="high"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.65,
                    duration={"seconds": 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner="ALIGN_MEAN",
                        ),
                    ],
                )
            )
        ],
        combiner=monitoring_v3.AlertPolicy.ConditionCombinerType.AND,
    )

    # Cloud SQL Replication Lag Alert
    replication_lag_policy = monitoring_v3.AlertPolicy(
        display_name="Cloud SQL High Replication Lag",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Replication lag > 60 seconds",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="cloudsql.googleapis.com/database/replication/replica_lag"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=60,
                    duration={"seconds": 180},
                ),
            )
        ],
        combiner=monitoring_v3.AlertPolicy.ConditionCombinerType.AND,
    )

    # Bigtable Storage Utilization
    bigtable_storage_policy = monitoring_v3.AlertPolicy(
        display_name="Bigtable High Storage Utilization (>80%)",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Storage > 80% of capacity",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="bigtable.googleapis.com/cluster/storage_utilization"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.8,
                    duration={"seconds": 600},
                ),
            )
        ],
        combiner=monitoring_v3.AlertPolicy.ConditionCombinerType.AND,
    )

    # Create alert policies
    policies = [
        cloudsql_cpu_policy,
        spanner_cpu_policy,
        replication_lag_policy,
        bigtable_storage_policy,
    ]

    for policy in policies:
        created = client.create_alert_policy(
            name=project_name,
            alert_policy=policy
        )
        print(f"Created alert policy: {created.display_name}")
```

**Exam Tip**: For monitoring questions, remember: Cloud SQL Insights for query performance, Spanner Query Stats for SQL analysis, Bigtable Key Visualizer for hotspot detection. Always monitor database-specific metrics (CPU for Spanner, connections for Cloud SQL).

## High Availability and Disaster Recovery

### Cloud SQL HA Configuration

**Regional HA (99.95% SLA)**:
```bash
# Create instance with regional HA
gcloud sql instances create prod-instance \
  --database-version=POSTGRES_15 \
  --tier=db-n1-standard-4 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --backup-start-time=03:00 \
  --enable-bin-log \
  --retained-transaction-log-days=7

# Regional HA configuration:
# - Primary instance in one zone
# - Standby instance in different zone (same region)
# - Synchronous replication to standby
# - Automatic failover (< 2 minutes)
# - No data loss during failover

# Test failover
gcloud sql instances failover prod-instance

# Failover behavior:
# - Promotes standby to primary
# - Creates new standby
# - Redirects connections automatically
# - Application may need to retry connections
```

**Read Replicas for DR**:
```bash
# Create cross-region read replica
gcloud sql instances create prod-replica \
  --master-instance-name=prod-instance \
  --region=europe-west1 \
  --tier=db-n1-standard-4 \
  --replica-type=READ_POOL

# Promote replica to standalone (DR scenario)
gcloud sql instances promote-replica prod-replica

# DR Architecture:
# Primary (us-central1) - REGIONAL HA
#   ├─ Standby (us-central1) - Automatic failover
#   ├─ Read Replica (us-east1) - Same continent DR
#   └─ Read Replica (europe-west1) - Cross-continent DR

# Typical RTO/RPO:
# - Regional HA failover: RTO < 2 min, RPO = 0
# - Read replica promotion: RTO = 5-10 min, RPO = seconds to minutes
```

**HA Testing and Validation**:
```python
from google.cloud import sql_v1
import time

def test_ha_failover(project_id, instance_id):
    """Test HA failover and measure RTO"""
    client = sql_v1.SqlInstancesServiceClient()
    instance_path = client.instance_path(project_id, instance_id)

    print("Initiating failover...")
    start_time = time.time()

    # Trigger failover
    operation = client.failover(instance=instance_path)

    # Wait for completion
    operation.result(timeout=600)

    # Measure RTO
    end_time = time.time()
    rto_seconds = end_time - start_time

    print(f"Failover completed in {rto_seconds:.2f} seconds")

    # Verify instance is healthy
    instance = client.get(name=instance_path)
    print(f"Instance state: {instance.state}")
    print(f"Availability type: {instance.settings.availability_type}")

    return rto_seconds
```

**Exam Tip**: For Cloud SQL, REGIONAL availability type provides automatic failover with zero data loss. Read replicas provide DR capability but require manual promotion. Always test failover procedures before production deployment.

### Cloud Spanner HA Configuration

**Multi-Region for 99.999% SLA**:
```bash
# Create multi-region instance
gcloud spanner instances create global-instance \
  --config=nam3 \
  --processing-units=3000 \
  --description="Multi-region production instance"

# Multi-region configurations:
# North America:
# - nam3: Iowa, South Carolina, Oklahoma (3 regions)
# - nam6: Iowa, SC, Oregon, LA (4 regions)
# - nam10: Iowa, SC, OR, LA, Montreal, Toronto (6 regions)
# - nam12: Iowa, SC, OR, LA, Toronto, Virginia (6 regions)

# Europe:
# - eur3: Belgium, Finland, Netherlands (3 regions)
# - eur5: London, Belgium, Finland, Netherlands, Zurich (5 regions)

# Asia:
# - asia1: Tokyo, Osaka, Seoul (3 regions)

# Regional for 99.99% SLA (lower cost)
gcloud spanner instances create regional-instance \
  --config=regional-us-central1 \
  --processing-units=1000
```

**Spanner Leader Placement**:
```sql
-- Set default leader for optimal write latency
ALTER DATABASE production
SET OPTIONS (
  default_leader = 'us-central1'
);

-- Check leader placement
SELECT
  table_name,
  parent_table_name,
  spanner_state,
  count(1) as num_splits
FROM INFORMATION_SCHEMA.TABLES
WHERE table_schema = ''
GROUP BY table_name, parent_table_name, spanner_state;

-- Multi-region write patterns:
-- - Writes go to leader region (configured)
-- - Reads are served from nearest replica
-- - Synchronous replication (zero data loss)
-- - Automatic failover (transparent to applications)
```

**Exam Tip**: Spanner multi-region provides 99.999% SLA with automatic failover and zero data loss. Regional instances are 10x cheaper but have lower SLA (99.99%). Choose based on availability requirements.

### Bigtable HA with Replication

**Multi-Cluster Replication**:
```bash
# Create instance with two clusters
gcloud bigtable instances create ha-instance \
  --display-name="HA Production Instance" \
  --cluster=us-central-cluster \
  --cluster-zone=us-central1-b \
  --cluster-num-nodes=3 \
  --cluster-storage-type=SSD

# Add second cluster for replication
gcloud bigtable clusters create eu-cluster \
  --instance=ha-instance \
  --zone=europe-west1-b \
  --num-nodes=3 \
  --storage-type=SSD

# Configure app profile for multi-cluster routing
gcloud bigtable app-profiles create multi-cluster-profile \
  --instance=ha-instance \
  --route-any \
  --description="Multi-cluster routing with failover"

# Replication characteristics:
# - Eventually consistent (typically < 1 second)
# - Automatic conflict resolution (last-write-wins)
# - Transparent failover
# - Increased read throughput
# - Geographic distribution
```

**Single-Cluster Routing**:
```bash
# Create app profile for single-cluster routing
gcloud bigtable app-profiles create analytics-profile \
  --instance=ha-instance \
  --route-to=us-central-cluster \
  --description="Analytics workload - strongly consistent"

# Use cases:
# - Multi-cluster routing: Latency-sensitive reads, automatic failover
# - Single-cluster routing: Strong consistency, batch analytics
```

**Exam Tip**: Bigtable replication provides HA and geo-distribution. Multi-cluster routing offers automatic failover but eventual consistency. Single-cluster routing provides strong consistency but manual failover.

### Firestore HA Configuration

**Multi-Region Setup**:
```bash
# Create multi-region database
gcloud firestore databases create \
  --database=production-db \
  --location=nam5 \
  --type=firestore-native

# Multi-region locations:
# - nam5: US multi-region (99.999% SLA)
# - eur3: Europe multi-region (99.999% SLA)

# Regional locations:
# - us-central1, europe-west1, asia-northeast1
# - 99.99% SLA
# - Lower cost
```

**Exam Tip**: Firestore multi-region provides automatic failover and 99.999% SLA. Regional databases are cheaper but have lower SLA. Choose based on availability and budget requirements.

## Operational Scenarios and Solutions

### Scenario 1: Cloud SQL Performance Degradation

**Problem**: Application experiencing slow database queries, increased latency.

**Troubleshooting Steps**:
```bash
# 1. Check CPU and memory utilization
gcloud sql instances describe INSTANCE \
  --format="value(settings.tier,settings.dataDiskSizeGb)"

# 2. Review Query Insights for slow queries
# Navigate to Cloud Console → Cloud SQL → Query Insights

# 3. Check connection count
gcloud monitoring read \
  'metric.type="cloudsql.googleapis.com/database/network/connections"' \
  --format="table(point.interval.end_time,point.value.int64_value)"

# 4. Analyze replication lag (if using replicas)
gcloud monitoring read \
  'metric.type="cloudsql.googleapis.com/database/replication/replica_lag"' \
  --filter='resource.database_id="PROJECT:INSTANCE"'
```

**Solution Decision Tree**:
```
Performance Issue
├─ High CPU (>80%)
│  ├─ Check slow queries → Optimize with indexes
│  ├─ Add read replicas → Offload read traffic
│  └─ Scale up tier → More CPU/memory
├─ High Connections
│  ├─ Implement connection pooling
│  ├─ Increase max_connections flag
│  └─ Use Cloud SQL Proxy
├─ Slow Queries
│  ├─ Enable Query Insights
│  ├─ Run EXPLAIN ANALYZE
│  ├─ Add appropriate indexes
│  └─ Optimize query structure
└─ High Replication Lag
   ├─ Check network between regions
   ├─ Review write volume
   └─ Consider read replica placement
```

**Resolution Script**:
```python
def resolve_cloudsql_performance(project_id, instance_id):
    """Automated performance issue resolution"""
    from google.cloud import sql_v1, monitoring_v3

    sql_client = sql_v1.SqlInstancesServiceClient()
    monitoring_client = monitoring_v3.MetricServiceClient()

    instance_path = sql_client.instance_path(project_id, instance_id)
    instance = sql_client.get(name=instance_path)

    # Check CPU utilization
    cpu_query = f'''
    fetch cloudsql_database
    | metric cloudsql.googleapis.com/database/cpu/utilization
    | filter resource.database_id == "{project_id}:{instance_id}"
    | within 1h
    '''

    results = monitoring_client.query_time_series(
        name=f"projects/{project_id}",
        query=cpu_query
    )

    avg_cpu = calculate_average_cpu(results)

    # Decision logic
    if avg_cpu > 0.8:
        print("HIGH CPU: Consider scaling up tier or optimizing queries")
        print(f"Current tier: {instance.settings.tier}")
        print("Recommendation: Increase to next tier or add read replicas")

    # Check slow queries via Query Insights
    # Recommend indexes for top slow queries

    return recommendations
```

**Exam Tip**: For Cloud SQL performance issues, always check: 1) Query Insights for slow queries, 2) CPU/Memory metrics, 3) Connection count. Resolution depends on root cause: optimize queries, scale resources, or add read replicas.

### Scenario 2: Spanner High CPU Utilization

**Problem**: Spanner high-priority CPU > 65%, queries experiencing timeouts.

**Troubleshooting**:
```sql
-- 1. Identify expensive queries
SELECT
  text as query_text,
  COUNT(*) as execution_count,
  AVG(cpu_time_sec) as avg_cpu_seconds,
  AVG(latency_seconds) as avg_latency,
  AVG(rows_scanned) / NULLIF(AVG(rows_returned), 0) as scan_ratio
FROM SPANNER_SYS.QUERY_STATS_TOP_HOUR
WHERE interval_end >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY text
ORDER BY avg_cpu_seconds DESC
LIMIT 10;

-- 2. Check for hotspots
SELECT
  interval_end,
  text,
  cpu_time_sec,
  lock_wait_seconds
FROM SPANNER_SYS.QUERY_STATS_TOP_MINUTE
WHERE lock_wait_seconds > 0.1
ORDER BY interval_end DESC
LIMIT 20;

-- 3. Review schema for optimization
SELECT
  t.table_name,
  t.parent_table_name,
  t.on_delete_action,
  COUNT(i.index_name) as index_count
FROM INFORMATION_SCHEMA.TABLES t
LEFT JOIN INFORMATION_SCHEMA.INDEXES i ON t.table_name = i.table_name
WHERE t.table_schema = ''
GROUP BY t.table_name, t.parent_table_name, t.on_delete_action;
```

**Solution**:
```bash
# Immediate action: Scale up nodes
gcloud spanner instances update INSTANCE \
  --nodes=10

# Long-term solutions:
# 1. Add secondary indexes for frequent queries
# 2. Use STORING clause to avoid table lookups
# 3. Optimize primary key design to avoid hotspots
# 4. Use query hints (FORCE_INDEX, JOIN_METHOD)
# 5. Implement caching layer (Memorystore)
```

**Exam Tip**: Spanner CPU > 65% requires immediate scaling. For long-term optimization, focus on secondary indexes with STORING clause and proper primary key design to avoid hotspots.

### Scenario 3: Bigtable Hotspot Detection

**Problem**: Bigtable showing hotspots in Key Visualizer, uneven load distribution.

**Troubleshooting**:
```bash
# 1. Review Key Visualizer in Cloud Console
# Look for bright regions indicating hotspots

# 2. Check node CPU utilization
gcloud monitoring read \
  'metric.type="bigtable.googleapis.com/cluster/cpu_load"' \
  --filter='resource.instance="INSTANCE" AND resource.cluster="CLUSTER"' \
  --format="table(point.interval.end_time,point.value.double_value)"

# 3. Review row key design
# Identify if keys are sequential or monotonically increasing
```

**Solution - Key Redesign**:
```python
# Before (Hotspot):
def old_row_key(user_id, timestamp):
    return f"{timestamp}#{user_id}"  # Sequential timestamps create hotspots

# After (Distributed):
def new_row_key(user_id, timestamp):
    # Add hash prefix for distribution
    import hashlib
    hash_prefix = hashlib.md5(user_id.encode()).hexdigest()[:4]
    # Reverse timestamp for range scans
    reverse_ts = 9999999999 - timestamp
    return f"{hash_prefix}#{user_id}#{reverse_ts}"

# Migration strategy:
# 1. Write to both old and new key formats
# 2. Backfill historical data
# 3. Update read paths to check both formats
# 4. Remove old key format after validation
```

**Exam Tip**: Bigtable hotspots are caused by poor key design. Use hash prefixes, reverse timestamps, or salting to distribute load. Key Visualizer is the primary tool for hotspot detection.

### Scenario 4: Backup Failure Recovery

**Problem**: Automated backup failed, need to ensure data protection.

**Troubleshooting and Resolution**:
```bash
# Cloud SQL - Check backup status
gcloud sql operations list \
  --instance=INSTANCE \
  --filter="operationType:BACKUP_VOLUME AND status:ERROR" \
  --format="table(insertTime,operationType,status,error.code,error.message)"

# Common causes:
# 1. Insufficient storage quota
# 2. Network connectivity issues
# 3. Locked tables (MySQL)
# 4. Backup location permissions

# Resolution:
# 1. Create immediate on-demand backup
gcloud sql backups create \
  --instance=INSTANCE \
  --description="Manual backup after failure"

# 2. Verify backup settings
gcloud sql instances describe INSTANCE \
  --format="json(settings.backupConfiguration)"

# 3. If backups continue failing, export to GCS
gcloud sql export sql INSTANCE gs://bucket/emergency-backup.sql \
  --database=DATABASE \
  --offload

# 4. Monitor backup success
gcloud monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="Backup Failure Alert" \
  --condition-threshold \
  --filter='metric.type="cloudsql.googleapis.com/database/backup/status"'
```

**Exam Tip**: For backup failures, always create immediate manual backup, then investigate root cause. Implement monitoring alerts for backup failures to catch issues early.

### Scenario 5: Cross-Region Disaster Recovery Drill

**Problem**: Need to test DR procedures for Cloud SQL.

**DR Test Procedure**:
```bash
# 1. Document current state
gcloud sql instances describe prod-instance \
  --format="json" > prod-state.json

# 2. Promote read replica in DR region
gcloud sql instances promote-replica dr-replica \
  --async

# 3. Wait for promotion to complete
gcloud sql operations wait OPERATION_ID

# 4. Update application configuration to use DR instance
# Update connection strings to point to dr-replica

# 5. Perform validation tests
# - Verify data integrity
# - Test read/write operations
# - Measure RTO

# 6. Document findings
# - Actual RTO vs target
# - Issues encountered
# - Application changes needed

# 7. Failback procedure
# - Create new read replica from original primary
# - Wait for replication to catch up
# - Switch application back
# - Delete promoted DR instance
```

**Exam Tip**: DR tests should be performed quarterly. Always document actual RTO/RPO vs targets. Read replica promotion is one-way; requires creating new replica for ongoing DR protection.

### Scenario 6: Firestore Query Performance Issues

**Problem**: Firestore queries slow, hitting index limits.

**Troubleshooting**:
```javascript
// Check for missing indexes
// Error: "The query requires an index"

// Solution: Create composite index
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "orders",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "customerId", "order": "ASCENDING" },
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}

// Deploy indexes
// firebase deploy --only firestore:indexes

// Optimize query patterns
async function optimizedQuery(customerId) {
  // Bad: Fetching all then filtering
  const allOrders = await db.collection('orders').get();
  return allOrders.docs.filter(d => d.data().customerId === customerId);

  // Good: Server-side filtering
  return await db.collection('orders')
    .where('customerId', '==', customerId)
    .where('status', '==', 'active')
    .orderBy('createdAt', 'desc')
    .limit(50)
    .get();
}
```

**Exam Tip**: Firestore automatically creates single-field indexes but composite indexes must be created manually. Index creation can take hours for large collections.

### Scenario 7: Connection Pool Exhaustion

**Problem**: Cloud SQL running out of connections during traffic spikes.

**Solution**:
```python
import sqlalchemy
from sqlalchemy.pool import QueuePool

# Problem: Each Cloud Run instance creates connections
# With 100 instances * 5 connections = 500 connections
# Exceeds Cloud SQL connection limit

# Solution 1: Reduce pool size for serverless
engine = sqlalchemy.create_engine(
    connection_url,
    pool_size=1,  # Minimal pool for serverless
    max_overflow=0,
    pool_timeout=30,
    pool_pre_ping=True
)

# Solution 2: Increase Cloud SQL max_connections
# gcloud sql instances patch INSTANCE \
#   --database-flags=max_connections=500

# Solution 3: Use Cloud SQL Proxy with connection limiting
# ./cloud-sql-proxy --max-connections=400 PROJECT:REGION:INSTANCE

# Solution 4: Implement connection retry logic
def execute_with_retry(query, max_retries=3):
    for attempt in range(max_retries):
        try:
            with engine.connect() as conn:
                return conn.execute(query)
        except sqlalchemy.exc.OperationalError as e:
            if "too many connections" in str(e):
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            raise
```

**Exam Tip**: For serverless workloads (Cloud Functions, Cloud Run), use minimal connection pools (1-2 connections). For traditional servers, pool size should be based on concurrent query needs, not application threads.

### Scenario 8: Multi-Region Data Compliance

**Problem**: Need to store EU customer data in Europe for GDPR compliance.

**Solution Architecture**:
```bash
# Option 1: Separate Spanner instances per region
gcloud spanner instances create eu-instance \
  --config=eur3 \
  --processing-units=1000

gcloud spanner instances create us-instance \
  --config=nam3 \
  --processing-units=1000

# Application routes based on customer location
# if customer.region == 'EU':
#     use eu-instance
# else:
#     use us-instance

# Option 2: Multi-region Spanner with data residency
# Use regional tables for sensitive data
CREATE TABLE EUCustomers (
  CustomerId STRING(36) NOT NULL,
  Data STRING(MAX)
) PRIMARY KEY (CustomerId)

# Option 3: Separate Firestore databases
gcloud firestore databases create eu-customers \
  --location=eur3 \
  --type=firestore-native

gcloud firestore databases create us-customers \
  --location=nam5 \
  --type=firestore-native

# Option 4: Bigtable regional instances
gcloud bigtable instances create eu-instance \
  --display-name="EU Customer Data" \
  --cluster=eu-cluster \
  --cluster-zone=europe-west1-b \
  --cluster-num-nodes=3
```

**Exam Tip**: For data residency requirements, use regional instances or separate instances per geography. Multi-region instances replicate data across regions which may violate data residency requirements.

## Performance Tuning Best Practices

### Query Optimization Summary

**Cloud SQL**:
1. Use EXPLAIN ANALYZE to identify slow operations
2. Create indexes for WHERE, JOIN, and ORDER BY columns
3. Use covering indexes to avoid table lookups
4. Implement query result caching (Redis/Memorystore)
5. Use read replicas for read-heavy workloads
6. Enable Query Insights for continuous monitoring

**Cloud Spanner**:
1. Monitor Query Stats for expensive queries
2. Use secondary indexes with STORING clause
3. Avoid sequential primary keys (causes hotspots)
4. Use query hints (FORCE_INDEX, JOIN_METHOD)
5. Batch reads with IN UNNEST for multiple IDs
6. Keep High Priority CPU < 65%

**Bigtable**:
1. Design row keys for even distribution (hash prefix)
2. Use Key Visualizer to detect hotspots
3. Configure appropriate GC policies for column families
4. Batch mutations (up to 100,000 rows)
5. Use row filters to reduce data transfer
6. Scale nodes based on throughput requirements

**Firestore**:
1. Create composite indexes for complex queries
2. Use pagination (limit + startAfter)
3. Denormalize data to avoid joins
4. Batch writes (up to 500 operations)
5. Disable indexing for large text fields
6. Use document listeners for real-time data

## Exam Tips and Patterns

### Decision-Making Framework

**When to use each database**:
```
Cloud SQL:
- Need: Relational data, ACID transactions
- Scale: Up to few TB, vertical scaling
- Use case: Traditional applications, migrations from MySQL/PostgreSQL

Cloud Spanner:
- Need: Global scale, strong consistency, SQL
- Scale: Petabytes, horizontal scaling
- Use case: Financial systems, global applications

Bigtable:
- Need: High throughput, time-series data
- Scale: Petabytes, massive read/write
- Use case: IoT, analytics, time-series

Firestore:
- Need: Document database, mobile/web apps
- Scale: Automatic, serverless
- Use case: Mobile apps, real-time sync
```

### Common Exam Question Patterns

**Performance Questions**:
- Slow queries → Enable Query Insights/Query Stats
- High CPU → Scale up (immediate), optimize queries (long-term)
- Hotspots → Review key design, add distribution (hash/salt)
- Connection issues → Connection pooling, Cloud SQL Proxy

**Backup/Recovery Questions**:
- Minimize data loss → PITR for Cloud SQL, multi-region for Spanner
- Point-in-time → Cloud SQL PITR (binary logs), Spanner version_time
- Cross-region DR → Read replicas, backups to different region
- Fastest recovery → HA configuration with automatic failover

**High Availability Questions**:
- 99.99% SLA → Cloud SQL REGIONAL, Spanner regional, Firestore regional
- 99.999% SLA → Spanner multi-region, Firestore multi-region
- Zero downtime → HA configuration, rolling maintenance
- Automatic failover → REGIONAL Cloud SQL, multi-region Spanner, Bigtable replication

**Cost Optimization Questions**:
- Reduce storage → GC policies (Bigtable), index optimization (Firestore)
- Reduce compute → Right-size instances, autoscaling, serverless exports
- Reduce backup costs → Adjust retention periods, use GCS lifecycle
- Reduce network → Private IP, same-region resources, caching

**Security Questions**:
- Encryption at rest → All databases encrypted by default, use CMEK for control
- Encryption in transit → SSL/TLS required, Cloud SQL Proxy enforces
- Access control → IAM roles, database users, VPC Service Controls
- Audit logging → Cloud Audit Logs, Query Insights

### Key Metrics and Thresholds

```
Cloud SQL:
- CPU > 80% → Scale up or optimize
- Connections > 80% of max → Increase max_connections or add pooling
- Replication lag > 60s → Network issue or overloaded replica
- Disk > 85% → Increase disk size

Cloud Spanner:
- High Priority CPU > 65% → Scale up immediately
- Storage > 75% → Review data retention, add nodes
- Lock wait > 1s → Review transaction patterns, add indexes
- Latency > 100ms → Check network, review query complexity

Bigtable:
- CPU > 70% → Add nodes or optimize key design
- Storage > 70% per node → Add nodes
- Latency > 10ms → Scale nodes, review key design
- Hotspots visible → Redesign row keys

Firestore:
- Read/Write > quota → Upgrade plan, optimize queries
- Index count > 200 → Review index strategy
- Document size > 500KB → Split into multiple documents
```

### Operations Best Practices Checklist

**Before Production**:
- [ ] Enable automated backups with appropriate retention
- [ ] Configure PITR for critical databases (Cloud SQL)
- [ ] Set up monitoring and alerting for key metrics
- [ ] Implement HA configuration (REGIONAL, multi-region)
- [ ] Test failover and recovery procedures
- [ ] Configure maintenance windows for low-traffic periods
- [ ] Enable Query Insights/Query Stats for performance monitoring
- [ ] Implement connection pooling with proper sizing
- [ ] Set up private IP connectivity (no public internet)
- [ ] Configure IAM roles and database authentication
- [ ] Document runbooks for common scenarios
- [ ] Perform load testing with production-like data
- [ ] Create read replicas for DR (Cloud SQL)
- [ ] Configure alerts for backup failures

**Regular Operations**:
- [ ] Review slow query reports weekly
- [ ] Test backup restoration monthly
- [ ] Perform DR drills quarterly
- [ ] Review and optimize index usage
- [ ] Monitor storage growth trends
- [ ] Review security and access logs
- [ ] Update maintenance windows as needed
- [ ] Capacity planning based on growth
- [ ] Clean up old backups beyond retention
- [ ] Test database upgrades on clones first
- [ ] Review cost optimization opportunities
- [ ] Update documentation for any changes

## Summary

This comprehensive guide covers all operational aspects of GCP database services for the Professional Cloud Database Engineer exam:

**Cloud SQL Operations**: Connection management with Cloud SQL Proxy, Query Insights for performance monitoring, automated backups with PITR, regional HA configuration with automatic failover, version upgrades and maintenance windows.

**Cloud Spanner Operations**: Node scaling and autoscaling strategies, Query Stats for performance analysis, backup and restore procedures, multi-region configuration for 99.999% SLA, secondary index optimization with STORING clause.

**Bigtable Operations**: Cluster replication and scaling, Key Visualizer for hotspot detection, row key design patterns, GC policies for cost optimization, performance monitoring and tuning.

**Firestore Operations**: Composite index management, query optimization patterns, batch operations and transactions, export/import procedures, multi-region configuration.

**Unified Monitoring**: Cloud Monitoring dashboards for all databases, alert policies for critical metrics, performance monitoring tools specific to each service.

**High Availability**: Regional and multi-region configurations, failover testing procedures, DR planning with RPO/RTO targets, read replica strategies.

**Operational Scenarios**: Real-world troubleshooting scenarios with step-by-step solutions, decision trees for common issues, scripts for automated remediation.

The exam focuses heavily on operational decision-making, performance troubleshooting, backup/recovery procedures, and understanding the trade-offs between different configuration options. Practice implementing these configurations hands-on and understand when to apply each solution based on requirements.
