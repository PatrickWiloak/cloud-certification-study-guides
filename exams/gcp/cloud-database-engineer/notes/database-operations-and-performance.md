# Database Operations and Performance - GCP Professional Cloud Database Engineer

## Overview

Database operations, performance tuning, monitoring, backup and recovery, and operational best practices for GCP databases.

## Performance Optimization

### Cloud SQL Performance
**Query Optimization**:
```sql
-- Use EXPLAIN to analyze queries
EXPLAIN ANALYZE
SELECT c.name, COUNT(o.order_id)
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.created_at > '2024-01-01'
GROUP BY c.name;

-- Create appropriate indexes
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_customers_created ON customers(created_at);

-- Avoid SELECT *
SELECT customer_id, email FROM customers WHERE status = 'active';
```

**Connection Pooling**:
```python
import sqlalchemy

# Create connection pool
pool = sqlalchemy.create_engine(
    'postgresql+pg8000://user:pass@/db',
    pool_size=10,
    max_overflow=5,
    pool_pre_ping=True,
    pool_recycle=3600
)

# Use connections from pool
with pool.connect() as conn:
    result = conn.execute("SELECT * FROM users")
```

**Performance Insights**:
```bash
# Enable Query Insights
gcloud sql instances patch INSTANCE \
  --insights-config-query-insights-enabled \
  --insights-config-query-string-length=1024 \
  --insights-config-record-application-tags \
  --insights-config-record-client-address

# View slow queries in Cloud Console
```

### Cloud Spanner Performance
**Query Optimization**:
```sql
-- Use FORCE_INDEX for query plans
SELECT * FROM Orders@{FORCE_INDEX=OrdersByDate}
WHERE OrderDate > '2024-01-01';

-- Batch reads for efficiency
SELECT * FROM Orders WHERE OrderId IN UNNEST(@order_ids);

-- Use secondary indexes wisely
CREATE NULL_FILTERED INDEX ActiveOrdersIndex ON Orders(CustomerId)
WHERE Status = 'ACTIVE';
```

**Monitoring CPU Utilization**:
```bash
# Check CPU utilization
gcloud spanner operations list \
  --instance=INSTANCE \
  --database=DATABASE \
  --filter="metadata.progress.startTime>2024-01-01"

# Add nodes if high CPU (>65% sustained)
gcloud spanner instances update INSTANCE \
  --nodes=5
```

### Firestore Performance
**Index Optimization**:
```javascript
// Query that needs composite index
db.collection('orders')
  .where('customerId', '==', 'user123')
  .where('status', '==', 'pending')
  .orderBy('orderDate', 'desc')
  .limit(10);

// Avoid array-contains + range query (needs composite index)
db.collection('products')
  .where('tags', 'array-contains', 'electronics')
  .where('price', '>', 100);
```

**Batch Operations**:
```javascript
// Batch writes (up to 500 operations)
const batch = db.batch();

for (let i = 0; i < 100; i++) {
  const docRef = db.collection('items').doc();
  batch.set(docRef, {name: `Item ${i}`});
}

await batch.commit();
```

### Bigtable Performance
**Schema Optimization**:
```python
# Good row key design for time-series
row_key = f"{sensor_id}#{reverse_timestamp}"

# Use column qualifiers efficiently
column_family = 'metrics'
column_qualifier = f"temp_{timestamp_ms}"
```

**Batch Operations**:
```python
from google.cloud import bigtable

# Batch mutations
table = instance.table('metrics')
rows = []

for i in range(1000):
    row_key = f"sensor_{i}#{timestamp}".encode()
    row = table.direct_row(row_key)
    row.set_cell('cf1', 'temperature', str(value))
    rows.append(row)

# Batch write
table.mutate_rows(rows)
```

## Backup and Recovery

### Cloud SQL Backups
**Automated Backups**:
```bash
# Configure automated backups
gcloud sql instances patch INSTANCE \
  --backup-start-time=03:00 \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7

# Create on-demand backup
gcloud sql backups create \
  --instance=INSTANCE \
  --description="Pre-upgrade backup"

# List backups
gcloud sql backups list --instance=INSTANCE

# Restore from backup
gcloud sql backups restore BACKUP_ID \
  --backup-instance=SOURCE_INSTANCE \
  --backup-id=BACKUP_ID \
  --instance=TARGET_INSTANCE
```

**Point-in-Time Recovery**:
```bash
# Enable PITR (binary logging)
gcloud sql instances patch INSTANCE \
  --enable-bin-log \
  --retained-transaction-log-days=7

# Restore to specific time
gcloud sql instances clone SOURCE_INSTANCE TARGET_INSTANCE \
  --point-in-time='2024-01-15T10:30:00.000Z'
```

**Export for Long-term Storage**:
```bash
# Export database
gcloud sql export sql INSTANCE gs://bucket/backup.sql \
  --database=DATABASE_NAME

# Export specific tables
gcloud sql export sql INSTANCE gs://bucket/tables.sql \
  --database=DATABASE_NAME \
  --table=customers,orders
```

### Cloud Spanner Backups
```bash
# Create backup
gcloud spanner backups create BACKUP_NAME \
  --instance=INSTANCE \
  --database=DATABASE \
  --retention-period=30d \
  --version-time='2024-01-15T10:00:00Z'

# Restore from backup
gcloud spanner databases restore \
  --source-backup=BACKUP_NAME \
  --source-instance=INSTANCE \
  --destination-database=NEW_DATABASE \
  --destination-instance=INSTANCE
```

### Firestore Backups
```bash
# Export data
gcloud firestore export gs://bucket/firestore-backup \
  --collection-ids=users,orders

# Import data
gcloud firestore import gs://bucket/firestore-backup
```

### Bigtable Backups
```bash
# Create backup
gcloud bigtable backups create BACKUP_NAME \
  --instance=INSTANCE \
  --cluster=CLUSTER \
  --table=TABLE \
  --retention-period=30d

# Restore backup
gcloud bigtable backups restore BACKUP_NAME \
  --destination=NEW_TABLE \
  --destination-instance=INSTANCE
```

## High Availability

### Cloud SQL HA
**Regional HA Configuration**:
```bash
# Create with HA
gcloud sql instances create ha-instance \
  --availability-type=REGIONAL \
  --region=us-central1

# Failover testing
gcloud sql instances failover ha-instance
```

**Cross-Region Disaster Recovery**:
```
Primary (us-central1)
  ├─ Standby (us-central1) - automatic failover
  └─ Read Replica (europe-west1) - manual promotion
```

### Cloud Spanner HA
**Multi-Region Configuration**:
```bash
# Create multi-region instance (99.999% SLA)
gcloud spanner instances create multi-region-instance \
  --config=nam3 \  # North America multi-region
  --nodes=3

# Configurations:
# - nam3: Iowa, South Carolina, Oklahoma
# - nam6: Iowa, South Carolina, Oregon, Los Angeles
# - eur3: Belgium, Finland, Netherlands
```

### Firestore Multi-Region
```bash
# Create multi-region database
gcloud firestore databases create \
  --location=nam5 \
  --type=firestore-native

# Multi-region locations:
# - nam5: US multi-region
# - eur3: Europe multi-region
```

## Monitoring and Alerting

### Cloud SQL Monitoring
```python
from google.cloud import monitoring_v3

def create_database_alert(project_id):
    client = monitoring_v3.AlertPolicyServiceClient()

    alert_policy = monitoring_v3.AlertPolicy(
        display_name="High Database CPU",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="CPU > 80%",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="cloudsql.googleapis.com/database/cpu/utilization"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.8,
                    duration={"seconds": 300}
                )
            )
        ],
        notification_channels=[CHANNEL_ID]
    )

    policy = client.create_alert_policy(
        name=f"projects/{project_id}",
        alert_policy=alert_policy
    )
```

### Key Metrics to Monitor
**Cloud SQL**:
- CPU utilization
- Memory utilization
- Disk I/O
- Connection count
- Replication lag
- Query execution time

**Cloud Spanner**:
- CPU utilization (high priority)
- Storage utilization
- Read/write latency
- Transaction commit latency

**Firestore**:
- Read/write operations per second
- Document count
- Storage size
- Connection errors

**Bigtable**:
- CPU utilization per node
- Disk utilization
- Row read/write counts
- Replication latency

## Maintenance Operations

### Cloud SQL Maintenance
```bash
# Configure maintenance window
gcloud sql instances patch INSTANCE \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=04 \
  --maintenance-release-channel=production

# Deny maintenance period
gcloud sql instances patch INSTANCE \
  --deny-maintenance-period-start-date=2024-12-15 \
  --deny-maintenance-period-end-date=2024-01-05 \
  --deny-maintenance-period-time=00:00:00
```

### Database Upgrades
```bash
# Cloud SQL version upgrade
gcloud sql instances patch INSTANCE \
  --database-version=POSTGRES_15

# Test upgrade on clone first
gcloud sql instances clone SOURCE TARGET
gcloud sql instances patch TARGET --database-version=POSTGRES_15
# Test application
# Then upgrade production
```

## Best Practices

### Performance
1. Monitor key metrics continuously
2. Optimize queries regularly
3. Use appropriate indexes
4. Implement connection pooling
5. Right-size database resources
6. Use caching for hot data
7. Batch operations when possible
8. Regular performance testing

### Backup and Recovery
1. Automate backups
2. Test restore procedures regularly
3. Store backups in different region
4. Implement PITR for critical databases
5. Document recovery procedures
6. Set appropriate retention periods
7. Monitor backup success
8. Encrypt backups

### Operations
1. Schedule maintenance windows
2. Monitor database health
3. Implement alerting
4. Document procedures
5. Regular capacity planning
6. Automation where possible
7. Security patching
8. Disaster recovery testing

## Common Scenarios

**Scenario**: Database performance degradation
**Solution**: Check slow query logs, analyze query plans, optimize indexes, review connection pool settings, scale resources if needed

**Scenario**: Disaster recovery test
**Solution**: Restore from backup to test environment, verify data integrity, test application connectivity, document findings

**Scenario**: High availability requirement (99.99%)
**Solution**: Cloud SQL REGIONAL availability type, automated backups, read replicas, failover testing

## Study Tips

1. Practice backup and restore procedures
2. Understand performance tuning for each service
3. Configure monitoring and alerting
4. Implement HA configurations
5. Work with maintenance operations
6. Troubleshoot common issues
7. Know service limitations and quotas

## Additional Resources

- [Cloud SQL Best Practices](https://cloud.google.com/sql/docs/postgres/best-practices)
- [Cloud Spanner Performance](https://cloud.google.com/spanner/docs/tune-query-performance)
- [Firestore Best Practices](https://firebase.google.com/docs/firestore/best-practices)
- [Bigtable Performance](https://cloud.google.com/bigtable/docs/performance)
