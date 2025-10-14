# Data and Analytics Services - Cloud Engineer

**[📖 Data Analytics Products](https://cloud.google.com/products/big-data)** - Google Cloud data and analytics services

## Cloud Storage

### Overview
- Object storage service for unstructured data
- Highly durable (99.999999999% durability)
- Scalable to exabytes
- Worldwide accessibility with low latency
- Integrated with all GCP services

### Storage Classes
| Class | Use Case | Min Storage | Retrieval Cost | Monthly Cost/GB |
|-------|----------|-------------|----------------|-----------------|
| Standard | Frequently accessed data | None | None | $0.020 |
| Nearline | < 1x/month access | 30 days | $0.01/GB | $0.010 |
| Coldline | < 1x/quarter access | 90 days | $0.02/GB | $0.004 |
| Archive | < 1x/year access | 365 days | $0.05/GB | $0.0012 |

**[📖 Storage Classes Guide](https://cloud.google.com/storage/docs/storage-classes)** - Choose the right storage class
**[📖 gsutil Tool](https://cloud.google.com/storage/docs/gsutil)** - Command-line tool for Cloud Storage

### Bucket Creation and Management
```bash
# Create bucket
gsutil mb -c STANDARD -l us-central1 gs://my-bucket

# Create multi-regional bucket
gsutil mb -c STANDARD -l us gs://my-multi-region-bucket

# Set storage class
gsutil defstorageclass set NEARLINE gs://my-bucket

# Enable versioning
gsutil versioning set on gs://my-bucket

# Check versioning status
gsutil versioning get gs://my-bucket

# Upload file
gsutil cp file.txt gs://my-bucket/

# Upload folder recursively
gsutil -m cp -r ./folder gs://my-bucket/

# List bucket contents
gsutil ls gs://my-bucket

# List with details (size, timestamp)
gsutil ls -l gs://my-bucket

# Download file
gsutil cp gs://my-bucket/file.txt ./

# Sync directories
gsutil rsync -r ./local-dir gs://my-bucket/remote-dir

# Delete file
gsutil rm gs://my-bucket/file.txt

# Delete bucket (must be empty)
gsutil rb gs://my-bucket
```

### Lifecycle Policies
```bash
# Create lifecycle policy file (lifecycle.json)
cat > lifecycle.json <<EOF
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 365}
      }
    ]
  }
}
EOF

# Apply lifecycle policy
gsutil lifecycle set lifecycle.json gs://my-bucket

# View lifecycle policy
gsutil lifecycle get gs://my-bucket
```

### Retention Policies and Locks
```bash
# Set retention policy (90 days)
gsutil retention set 90d gs://my-bucket

# View retention policy
gsutil retention get gs://my-bucket

# Lock retention policy (IRREVERSIBLE)
gsutil retention lock gs://my-bucket

# Remove retention policy (only if not locked)
gsutil retention clear gs://my-bucket
```

### Object Versioning
```bash
# Enable versioning
gsutil versioning set on gs://my-bucket

# List all versions
gsutil ls -a gs://my-bucket/file.txt

# Copy specific version
gsutil cp gs://my-bucket/file.txt#1234567890 ./file-old.txt

# Delete specific version
gsutil rm gs://my-bucket/file.txt#1234567890

# Delete all versions
gsutil rm -a gs://my-bucket/file.txt
```

### Signed URLs
```bash
# Create signed URL (valid for 1 hour)
gsutil signurl -d 1h key.json gs://my-bucket/file.txt

# Create signed URL with custom HTTP method
gsutil signurl -m PUT -d 1h key.json gs://my-bucket/file.txt

# Using gcloud
gcloud storage sign-url gs://my-bucket/file.txt --duration=1h
```

### IAM vs ACLs
**IAM (Recommended)**:
- Apply permissions at bucket or project level
- Role-based access control
- Integrates with organization policies
- Uniform bucket-level access

**ACLs (Legacy)**:
- Object-level permissions
- Finer-grained control
- Can conflict with IAM

```bash
# Enable uniform bucket-level access (IAM only)
gsutil uniformbucketlevelaccess set on gs://my-bucket

# Grant IAM role
gsutil iam ch user:user@example.com:objectViewer gs://my-bucket

# View IAM policy
gsutil iam get gs://my-bucket

# Set ACL (if uniform access disabled)
gsutil acl set private gs://my-bucket/file.txt

# Grant ACL permission
gsutil acl ch -u user@example.com:READ gs://my-bucket/file.txt
```

## Cloud SQL

### Overview
- Fully managed relational database service
- Supports MySQL, PostgreSQL, SQL Server
- Automated backups, replication, failover
- High availability with 99.95% SLA
- Scale up to 96 CPU cores, 624 GB RAM

**[📖 Cloud SQL Overview](https://cloud.google.com/sql/docs/introduction)** - Managed relational databases
**[📖 Backup and Recovery](https://cloud.google.com/sql/docs/mysql/backup-recovery/backups)** - Backup and restore operations

### Instance Creation
```bash
# Create MySQL instance
gcloud sql instances create my-instance \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-1 \
  --region=us-central1 \
  --root-password=mypassword \
  --backup \
  --backup-start-time=03:00

# Create PostgreSQL instance
gcloud sql instances create pg-instance \
  --database-version=POSTGRES_15 \
  --tier=db-custom-2-8192 \
  --region=us-central1 \
  --root-password=mypassword

# List instances
gcloud sql instances list

# Describe instance
gcloud sql instances describe my-instance

# Delete instance
gcloud sql instances delete my-instance
```

### Machine Types and Sizing
```bash
# Shared-core (dev/test)
--tier=db-f1-micro      # 0.6 GB RAM
--tier=db-g1-small      # 1.7 GB RAM

# Standard
--tier=db-n1-standard-1  # 1 CPU, 3.75 GB
--tier=db-n1-standard-2  # 2 CPU, 7.5 GB
--tier=db-n1-standard-4  # 4 CPU, 15 GB

# High memory
--tier=db-n1-highmem-2   # 2 CPU, 13 GB
--tier=db-n1-highmem-4   # 4 CPU, 26 GB

# Custom
--tier=db-custom-2-8192  # 2 CPU, 8 GB (8192 MB)
```

### High Availability Configuration
```bash
# Create HA instance
gcloud sql instances create ha-instance \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --root-password=mypassword

# Enable HA on existing instance
gcloud sql instances patch my-instance \
  --availability-type=REGIONAL

# Disable HA (fallback to zonal)
gcloud sql instances patch my-instance \
  --availability-type=ZONAL
```

### Read Replicas
```bash
# Create read replica
gcloud sql instances create my-replica \
  --master-instance-name=my-instance \
  --tier=db-n1-standard-1 \
  --region=us-east1

# Create read replica in same region
gcloud sql instances create my-local-replica \
  --master-instance-name=my-instance \
  --tier=db-n1-standard-1

# Promote replica to standalone
gcloud sql instances promote-replica my-replica

# Delete replica
gcloud sql instances delete my-replica
```

### Backups and Point-in-Time Recovery
```bash
# Enable automated backups
gcloud sql instances patch my-instance \
  --backup-start-time=03:00 \
  --backup-location=us

# Create on-demand backup
gcloud sql backups create \
  --instance=my-instance \
  --description="Manual backup before upgrade"

# List backups
gcloud sql backups list --instance=my-instance

# Restore from backup
gcloud sql backups restore BACKUP_ID \
  --backup-instance=my-instance \
  --backup-id=1234567890

# Clone instance (creates new instance from backup)
gcloud sql instances clone my-instance cloned-instance

# Enable binary logging (required for PITR)
gcloud sql instances patch my-instance \
  --enable-bin-log

# Point-in-time recovery (restore to specific timestamp)
gcloud sql instances restore-backup my-instance \
  --backup-id=BACKUP_ID \
  --backup-instance=SOURCE_INSTANCE
```

### Database and User Management
```bash
# Create database
gcloud sql databases create mydb --instance=my-instance

# List databases
gcloud sql databases list --instance=my-instance

# Delete database
gcloud sql databases delete mydb --instance=my-instance

# Create user
gcloud sql users create myuser \
  --instance=my-instance \
  --password=mypassword

# List users
gcloud sql users list --instance=my-instance

# Change user password
gcloud sql users set-password myuser \
  --instance=my-instance \
  --password=newpassword

# Delete user
gcloud sql users delete myuser --instance=my-instance
```

### Connection Methods
```bash
# Connect via Cloud SQL Proxy
./cloud_sql_proxy -instances=PROJECT:REGION:INSTANCE=tcp:3306

# Connect from local machine
gcloud sql connect my-instance --user=root

# Get connection name
gcloud sql instances describe my-instance --format="value(connectionName)"

# Authorize external IP
gcloud sql instances patch my-instance \
  --authorized-networks=203.0.113.5/32
```

### Maintenance Windows
```bash
# Set maintenance window (Sunday 3 AM)
gcloud sql instances patch my-instance \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=3

# Enable maintenance notifications
gcloud sql instances patch my-instance \
  --maintenance-release-channel=production

# Deny maintenance period (no updates during black Friday)
gcloud sql instances patch my-instance \
  --deny-maintenance-period-start-date=2025-11-20 \
  --deny-maintenance-period-end-date=2025-11-30
```

## Cloud Spanner

### Overview
- Globally distributed, horizontally scalable relational database
- Strong consistency across regions
- SQL queries with ACID transactions
- Automatic sharding and replication
- 99.999% availability SLA for multi-region

**[📖 Cloud Spanner Overview](https://cloud.google.com/spanner/docs/overview)** - Global relational database
**[📖 Spanner Schema Design](https://cloud.google.com/spanner/docs/schema-design)** - Schema and data model best practices

### Instance Configuration
```bash
# Create regional instance
gcloud spanner instances create my-instance \
  --config=regional-us-central1 \
  --nodes=1 \
  --description="Regional Spanner instance"

# Create multi-region instance
gcloud spanner instances create global-instance \
  --config=nam3 \
  --processing-units=1000 \
  --description="Multi-region instance"

# List available configurations
gcloud spanner instance-configs list

# Update node count (scaling)
gcloud spanner instances update my-instance --nodes=3

# Update processing units (finer-grained scaling)
gcloud spanner instances update my-instance --processing-units=500

# Delete instance
gcloud spanner instances delete my-instance
```

### Configuration Types
- **Regional**: Single region, 99.99% SLA (e.g., us-central1)
- **Multi-Regional**: Span multiple regions, 99.999% SLA (e.g., nam3, eur3)
- **Custom**: Configure your own read-write regions

### Database and Schema Management
```bash
# Create database
gcloud spanner databases create mydb \
  --instance=my-instance

# Create database with DDL
gcloud spanner databases create mydb \
  --instance=my-instance \
  --ddl="CREATE TABLE Users (
    UserId INT64 NOT NULL,
    Name STRING(100),
    Email STRING(100)
  ) PRIMARY KEY (UserId)"

# Execute DDL
gcloud spanner databases ddl update mydb \
  --instance=my-instance \
  --ddl="CREATE TABLE Orders (
    OrderId INT64 NOT NULL,
    UserId INT64 NOT NULL,
    Amount FLOAT64,
    CreatedAt TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
  ) PRIMARY KEY (OrderId)"

# List databases
gcloud spanner databases list --instance=my-instance

# Delete database
gcloud spanner databases delete mydb --instance=my-instance
```

### Interleaved Tables
Interleaving physically co-locates child rows with parent rows for better performance.

```sql
-- Parent table
CREATE TABLE Users (
  UserId INT64 NOT NULL,
  Name STRING(100)
) PRIMARY KEY (UserId);

-- Interleaved child table
CREATE TABLE Orders (
  UserId INT64 NOT NULL,
  OrderId INT64 NOT NULL,
  Amount FLOAT64
) PRIMARY KEY (UserId, OrderId),
  INTERLEAVE IN PARENT Users ON DELETE CASCADE;
```

```bash
# Create interleaved tables
gcloud spanner databases ddl update mydb \
  --instance=my-instance \
  --ddl="CREATE TABLE Users (UserId INT64, Name STRING(100)) PRIMARY KEY (UserId);
        CREATE TABLE Orders (UserId INT64, OrderId INT64, Amount FLOAT64)
        PRIMARY KEY (UserId, OrderId), INTERLEAVE IN PARENT Users ON DELETE CASCADE;"
```

### Secondary Indexes
```bash
# Create index
gcloud spanner databases ddl update mydb \
  --instance=my-instance \
  --ddl="CREATE INDEX UsersByEmail ON Users(Email)"

# Create unique index
gcloud spanner databases ddl update mydb \
  --instance=my-instance \
  --ddl="CREATE UNIQUE INDEX UsersByEmail ON Users(Email)"

# Create storing index (include extra columns)
gcloud spanner databases ddl update mydb \
  --instance=my-instance \
  --ddl="CREATE INDEX UsersByEmail ON Users(Email) STORING (Name)"

# Drop index
gcloud spanner databases ddl update mydb \
  --instance=my-instance \
  --ddl="DROP INDEX UsersByEmail"
```

### Queries and Transactions
```bash
# Execute query
gcloud spanner databases execute-sql mydb \
  --instance=my-instance \
  --sql="SELECT * FROM Users WHERE UserId = 1"

# Execute DML
gcloud spanner databases execute-sql mydb \
  --instance=my-instance \
  --sql="INSERT INTO Users (UserId, Name, Email) VALUES (1, 'John', 'john@example.com')"
```

## Bigtable

### Overview
- NoSQL wide-column database
- Petabyte-scale, sub-10ms latency
- Ideal for time-series, IoT, analytics workloads
- HBase compatible
- Linear scalability

**[📖 Bigtable Overview](https://cloud.google.com/bigtable/docs/overview)** - NoSQL database service
**[📖 Schema Design for Time Series](https://cloud.google.com/bigtable/docs/schema-design-time-series)** - Time-series data patterns

### Instance and Cluster Creation
```bash
# Create instance with cluster
gcloud bigtable instances create my-instance \
  --display-name="My Bigtable Instance" \
  --cluster=my-cluster \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=3 \
  --instance-type=PRODUCTION

# Create development instance (single node)
gcloud bigtable instances create dev-instance \
  --display-name="Dev Instance" \
  --cluster=dev-cluster \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=1 \
  --instance-type=DEVELOPMENT

# List instances
gcloud bigtable instances list

# Update instance (scaling)
gcloud bigtable clusters update my-cluster \
  --instance=my-instance \
  --num-nodes=5

# Enable autoscaling
gcloud bigtable clusters update my-cluster \
  --instance=my-instance \
  --autoscaling-min-nodes=3 \
  --autoscaling-max-nodes=10 \
  --autoscaling-cpu-target=70

# Delete instance
gcloud bigtable instances delete my-instance
```

### Table Creation and Management
```bash
# Create table
cbt -project=my-project -instance=my-instance createtable my-table

# List tables
cbt -project=my-project -instance=my-instance ls

# Create column family
cbt -project=my-project -instance=my-instance createfamily my-table cf1

# Set garbage collection policy (keep last 7 days)
cbt -project=my-project -instance=my-instance setgcpolicy my-table cf1 maxage=7d

# Set GC policy (keep last 3 versions)
cbt -project=my-project -instance=my-instance setgcpolicy my-table cf1 maxversions=3

# Write data
cbt -project=my-project -instance=my-instance set my-table r1 cf1:c1=value1

# Read data
cbt -project=my-project -instance=my-instance read my-table

# Read specific row
cbt -project=my-project -instance=my-instance lookup my-table r1

# Delete table
cbt -project=my-project -instance=my-instance deletetable my-table
```

### Row Key Design Best Practices
**Critical for performance**:
1. **Avoid monotonically increasing keys** (timestamps, sequential IDs)
   - Bad: `timestamp#user_id`
   - Good: `reverse_domain#timestamp` or `user_id#timestamp`

2. **Distribute writes evenly**
   - Use hash prefix: `md5(user_id)[:4]#user_id#timestamp`
   - Reverse timestamps: `Long.MAX_VALUE - timestamp`

3. **Design for access patterns**
   - If querying by user: `user_id#timestamp`
   - If querying by device: `device_id#timestamp`

4. **Keep row keys short** (4-100 bytes optimal)

### Column Families and Columns
```bash
# Multiple column families
cbt createfamily my-table profile
cbt createfamily my-table metrics
cbt createfamily my-table logs

# Set different GC policies
cbt setgcpolicy my-table profile maxversions=1
cbt setgcpolicy my-table metrics maxage=30d
cbt setgcpolicy my-table logs maxage=7d or maxversions=3
```

### Replication
```bash
# Add replica cluster
gcloud bigtable clusters create my-cluster-replica \
  --instance=my-instance \
  --zone=us-east1-b \
  --num-nodes=3

# List clusters
gcloud bigtable clusters list --instances=my-instance

# Create app profile for single-cluster routing
gcloud bigtable app-profiles create single-cluster-profile \
  --instance=my-instance \
  --route-to=my-cluster

# Create app profile for multi-cluster routing (replication)
gcloud bigtable app-profiles create multi-cluster-profile \
  --instance=my-instance \
  --route-any

# Update app profile
gcloud bigtable app-profiles update multi-cluster-profile \
  --instance=my-instance \
  --route-to=my-cluster-replica
```

## Firestore / Datastore

### Overview
- NoSQL document database
- Strong consistency within region
- Automatic scaling
- ACID transactions
- Real-time updates (Firestore Native mode)

**Modes**:
- **Datastore mode**: Server-side applications, no real-time updates
- **Native mode**: Mobile/web apps, real-time sync

### Database Creation
```bash
# Create Firestore database (Native mode)
gcloud firestore databases create \
  --location=us-central \
  --type=firestore-native

# Create Datastore database
gcloud firestore databases create \
  --location=us-central \
  --type=datastore-mode

# Note: Only one database per project (default), multiple databases in preview
```

### Collections and Documents
```bash
# Datastore: Create entity (via gcloud)
gcloud datastore entities create --kind=User \
  --properties='name:string=John,email:string=john@example.com,age:integer=30'

# Query entities
gcloud datastore queries fetch --kind=User

# Delete entity
gcloud datastore entities delete --kind=User --keys=KEY_ID
```

### Indexes
**Single-field indexes**: Automatic
**Composite indexes**: Must be defined

```yaml
# index.yaml
indexes:
- kind: Task
  properties:
  - name: priority
    direction: desc
  - name: created
    direction: asc

- kind: Order
  properties:
  - name: status
  - name: amount
    direction: desc
```

```bash
# Deploy indexes
gcloud datastore indexes create index.yaml

# List indexes
gcloud datastore indexes list

# Cleanup unused indexes
gcloud datastore indexes cleanup index.yaml
```

### Queries and Transactions
```bash
# Export data
gcloud datastore export gs://my-bucket/datastore-export \
  --kinds=User,Order \
  --namespaces=default

# Import data
gcloud datastore import gs://my-bucket/datastore-export/export.overall_export_metadata

# Operations list
gcloud datastore operations list
```

## BigQuery

### Overview
- Serverless data warehouse
- SQL queries on petabyte-scale data
- Columnar storage
- No infrastructure management
- Sub-second query response

**[📖 BigQuery Overview](https://cloud.google.com/bigquery/docs/introduction)** - Serverless data warehouse
**[📖 Query Best Practices](https://cloud.google.com/bigquery/docs/best-practices-performance-compute)** - Optimize query performance
**[📖 Cost Optimization](https://cloud.google.com/bigquery/docs/best-practices-costs)** - Control BigQuery costs

### Dataset and Table Creation
```bash
# Create dataset
bq mk --dataset --location=US --default_table_expiration=3600 my_dataset

# Create dataset with description
bq mk --dataset \
  --description="Sales data warehouse" \
  --location=us-central1 \
  my_project:sales_data

# List datasets
bq ls

# List datasets in project
bq ls --project_id=my_project

# Show dataset details
bq show my_dataset

# Create table from schema
bq mk --table my_dataset.my_table schema.json

# Create table with inline schema
bq mk --table my_dataset.users \
  user_id:INTEGER,name:STRING,email:STRING,created:TIMESTAMP

# Create external table (data in Cloud Storage)
bq mk --external_table_definition=schema.json@CSV=gs://my-bucket/data.csv \
  my_dataset.external_table

# Load data from CSV
bq load --source_format=CSV \
  my_dataset.my_table \
  gs://my-bucket/data.csv \
  schema.json

# Load data with autodetect schema
bq load --autodetect \
  --source_format=CSV \
  my_dataset.my_table \
  gs://my-bucket/data.csv

# Delete table
bq rm -t my_dataset.my_table

# Delete dataset
bq rm -r -d my_dataset
```

### Partitioning
**Benefits**: Reduce query costs, improve performance

```bash
# Create time-partitioned table (ingestion time)
bq mk --table \
  --time_partitioning_type=DAY \
  my_dataset.partitioned_table \
  schema.json

# Create time-partitioned table (on specific column)
bq mk --table \
  --time_partitioning_type=DAY \
  --time_partitioning_field=transaction_date \
  my_dataset.transactions \
  schema.json

# Create integer-range partitioned table
bq mk --table \
  --range_partitioning=customer_id,0,1000000,100 \
  my_dataset.customers \
  schema.json

# Partition expiration (90 days)
bq mk --table \
  --time_partitioning_type=DAY \
  --time_partitioning_expiration=7776000 \
  my_dataset.logs \
  schema.json

# Query partitioned table (cost optimization)
bq query --use_legacy_sql=false '
SELECT *
FROM `my_dataset.transactions`
WHERE transaction_date = "2025-01-15"
'
```

### Clustering
**Benefits**: Further optimize queries within partitions

```bash
# Create partitioned + clustered table
bq mk --table \
  --time_partitioning_type=DAY \
  --time_partitioning_field=date \
  --clustering_fields=customer_id,product_id \
  my_dataset.sales \
  schema.json

# Best practices:
# - Cluster on columns frequently used in WHERE/JOIN
# - Up to 4 clustering columns
# - Order matters (most selective first)
```

### Materialized Views
```bash
# Create materialized view
bq query --use_legacy_sql=false '
CREATE MATERIALIZED VIEW my_dataset.daily_sales AS
SELECT
  DATE(order_timestamp) as order_date,
  product_id,
  SUM(amount) as total_sales
FROM my_dataset.orders
GROUP BY order_date, product_id
'

# Refresh materialized view
bq query --use_legacy_sql=false '
REFRESH MATERIALIZED VIEW my_dataset.daily_sales
'

# Query will automatically use materialized view when beneficial
```

### Slots and Reservations
```bash
# List reservations
bq ls --reservations --location=us-central1 --project_id=my_project

# Create reservation (flat-rate pricing)
bq mk --reservation \
  --location=us-central1 \
  --slots=500 \
  production_reservation

# Create assignment (link reservation to project/folder/org)
bq mk --reservation_assignment \
  --reservation_id=production_reservation \
  --job_type=QUERY \
  --assignee_type=PROJECT \
  --assignee_id=my_project

# Update reservation slots
bq update --reservation \
  --location=us-central1 \
  --slots=1000 \
  production_reservation
```

### Query Optimization
```bash
# Dry run to estimate costs
bq query --dry_run --use_legacy_sql=false '
SELECT * FROM my_dataset.large_table
'

# Query with destination table
bq query --use_legacy_sql=false \
  --destination_table=my_dataset.results \
  --replace \
  'SELECT * FROM my_dataset.source_table WHERE date > "2025-01-01"'

# Query with table expiration
bq query --use_legacy_sql=false \
  --destination_table=my_dataset.temp_results \
  --time_partitioning_expiration=86400 \
  'SELECT * FROM my_dataset.source'

# Query with caching disabled
bq query --use_legacy_sql=false --use_cache=false 'SELECT ...'

# Extract query results to Cloud Storage
bq extract --destination_format=CSV \
  my_dataset.results \
  gs://my-bucket/results-*.csv
```

### Advanced Queries
```bash
# Query with parameters
bq query --use_legacy_sql=false \
  --parameter=min_amount:FLOAT64:1000 \
  'SELECT * FROM my_dataset.orders WHERE amount > @min_amount'

# Export to Cloud Storage
bq extract \
  --destination_format=NEWLINE_DELIMITED_JSON \
  --compression=GZIP \
  my_dataset.my_table \
  gs://my-bucket/export-*.json.gz

# Copy table
bq cp my_dataset.source_table my_dataset.destination_table

# Copy table from another project
bq cp source_project:dataset.table my_project:dataset.table
```

## Troubleshooting Scenarios

### Scenario 1: Cloud Storage High Egress Costs
**Problem**: Unexpected egress charges from Cloud Storage bucket.

**Step-by-step resolution**:
1. **Identify traffic patterns**:
   ```bash
   # Enable request logs
   gsutil logging set on -b gs://logging-bucket gs://my-bucket

   # Analyze logs in BigQuery
   bq query --use_legacy_sql=false '
   SELECT client_ip, COUNT(*) as requests, SUM(bytes_sent) as total_bytes
   FROM `my_project.storage_logs.usage_*`
   GROUP BY client_ip
   ORDER BY total_bytes DESC
   LIMIT 10
   '
   ```

2. **Check bucket location vs. compute location**:
   ```bash
   gsutil ls -L -b gs://my-bucket | grep Location
   gcloud compute instances list --format="table(name,zone)"
   ```

3. **Solutions**:
   - Move bucket to same region as compute resources
   - Enable CDN for public content
   - Use signed URLs with restrictions
   - Implement request rate limiting

4. **Prevention**:
   ```bash
   # Create bucket in same region as GKE cluster
   gsutil mb -c STANDARD -l us-central1 gs://app-data-bucket
   ```

### Scenario 2: Cloud SQL High CPU and Slow Queries
**Problem**: Cloud SQL instance showing 95% CPU, application timeouts.

**Step-by-step resolution**:
1. **Check current metrics**:
   ```bash
   gcloud sql instances describe my-instance \
     --format="value(settings.tier,settings.dataDiskSizeGb)"
   ```

2. **Enable query insights**:
   ```bash
   gcloud sql instances patch my-instance \
     --insights-config-query-insights-enabled \
     --insights-config-query-string-length=1024 \
     --insights-config-record-application-tags
   ```

3. **Analyze slow queries** (via Cloud Console):
   - Navigation: SQL > Instance > Query Insights
   - Sort by execution time and frequency

4. **Check for missing indexes**:
   ```sql
   -- MySQL: Check queries without indexes
   SELECT * FROM mysql.slow_log
   WHERE rows_examined > 1000
   ORDER BY query_time DESC LIMIT 10;
   ```

5. **Immediate mitigation**:
   ```bash
   # Scale up instance
   gcloud sql instances patch my-instance \
     --tier=db-n1-standard-4

   # Add read replica for read-heavy workloads
   gcloud sql instances create my-replica \
     --master-instance-name=my-instance \
     --tier=db-n1-standard-2
   ```

6. **Long-term fixes**:
   - Add appropriate indexes
   - Implement query caching
   - Use connection pooling (PgBouncer/ProxySQL)

### Scenario 3: Spanner High Latency Across Regions
**Problem**: Application in us-central1 experiencing high latency reading from Spanner.

**Step-by-step resolution**:
1. **Check instance configuration**:
   ```bash
   gcloud spanner instances describe my-instance \
     --format="value(config)"
   ```

2. **Identify if instance is multi-region**:
   ```bash
   gcloud spanner instance-configs describe nam3
   # Shows: Read-write: us-east1, us-central1, us-west1
   ```

3. **Check read latency by location**:
   - Use Cloud Monitoring metrics
   - Query: `spanner.googleapis.com/api/request_latencies`

4. **Solutions**:
   ```bash
   # If regional instance, migrate to multi-region
   # 1. Create new multi-region instance
   gcloud spanner instances create global-instance \
     --config=nam3 \
     --processing-units=1000

   # 2. Create database in new instance
   gcloud spanner databases create mydb \
     --instance=global-instance

   # 3. Export/import data using Dataflow
   ```

5. **Optimize queries for distribution**:
   - Use stale reads for non-critical data: `SELECT ... FROM ... FOR SYSTEM_TIME AS OF ...`
   - Batch mutations to reduce round trips

### Scenario 4: Bigtable Hotspotting
**Problem**: Bigtable showing uneven node utilization, some nodes at 90% CPU.

**Step-by-step resolution**:
1. **Check Key Visualizer** (Cloud Console):
   - Navigation: Bigtable > Instance > Key Visualizer
   - Look for dark vertical bands (hotspots)

2. **Identify problematic row keys**:
   ```bash
   # Check row key distribution
   cbt -project=my-project -instance=my-instance \
     read my-table count=100
   ```

3. **Analyze row key pattern**:
   - Sequential keys: `user_001`, `user_002` (BAD)
   - Timestamp-prefixed: `2025-01-15#data` (BAD)
   - Hashed keys: `a3f2#user_001` (GOOD)

4. **Solutions**:
   - **Field promotion**: Move high-cardinality field first
     - Bad: `timestamp#user_id`
     - Good: `user_id#timestamp`

   - **Salting**: Add random prefix
     - Bad: `timestamp#user_id`
     - Good: `{random(0-99)}#timestamp#user_id`

   - **Reverse timestamp**:
     ```python
     # Python example
     reverse_ts = (2**63 - 1) - timestamp_ms
     row_key = f"user_{user_id}#{reverse_ts}"
     ```

5. **Scale nodes temporarily**:
   ```bash
   gcloud bigtable clusters update my-cluster \
     --instance=my-instance \
     --num-nodes=10
   ```

6. **Monitor improvement**:
   - Wait 15-20 minutes for rebalancing
   - Check CPU distribution in monitoring

### Scenario 5: BigQuery Query Scanning Too Much Data
**Problem**: Query costs exceeding budget, scanning TBs of data unnecessarily.

**Step-by-step resolution**:
1. **Estimate query cost before running**:
   ```bash
   bq query --dry_run --use_legacy_sql=false '
   SELECT * FROM `my-project.my_dataset.large_table`
   WHERE date >= "2025-01-01"
   '
   # Output shows bytes that will be processed
   ```

2. **Common issues**:
   - Selecting all columns: `SELECT *`
   - Not filtering partitioned columns
   - Querying entire table instead of specific partitions

3. **Optimize query**:
   ```bash
   # Bad query (scans entire table)
   bq query --dry_run --use_legacy_sql=false '
   SELECT * FROM `my_dataset.transactions`
   WHERE status = "completed"
   '

   # Good query (uses partition)
   bq query --dry_run --use_legacy_sql=false '
   SELECT user_id, amount, status
   FROM `my_dataset.transactions`
   WHERE DATE(transaction_date) = "2025-01-15"
     AND status = "completed"
   '
   ```

4. **Implement partitioning**:
   ```bash
   # Create partitioned table from existing table
   bq query --use_legacy_sql=false \
     --destination_table=my_dataset.transactions_partitioned \
     --time_partitioning_field=transaction_date \
     --clustering_fields=user_id,status \
   'SELECT * FROM my_dataset.transactions'
   ```

5. **Add clustering**:
   ```bash
   bq update --clustering_fields=customer_id,product_id \
     my_dataset.sales
   ```

6. **Set up cost controls**:
   ```bash
   # Set custom quota to limit query bytes
   bq mk --transfer_config \
     --target_dataset=my_dataset \
     --display_name="Cost Control" \
     --params='{"query":"SELECT 1","custom_user_name":"cost-control"}'

   # Set maximum bytes billed in query
   bq query --maximum_bytes_billed=1000000000 \
     --use_legacy_sql=false 'SELECT ...'
   ```

### Scenario 6: Firestore Read/Write Limits Exceeded
**Problem**: Application getting "RESOURCE_EXHAUSTED" errors from Firestore.

**Step-by-step resolution**:
1. **Check quota limits**:
   - 10,000 writes/second to a document
   - 500 writes/second to collection
   - 1 million concurrent connections

2. **Identify hot documents**:
   ```bash
   # Check Firestore metrics in Cloud Monitoring
   gcloud monitoring time-series list \
     --filter='metric.type="firestore.googleapis.com/document/write_count"'
   ```

3. **Common causes**:
   - Counter document updated too frequently
   - Fan-out writes to same collection
   - Unbounded array growth

4. **Solutions for high-write counters**:
   - **Distributed counter pattern**:
     ```
     /counters/page_views/shards/{shard_id}
     # Sum 10 shards instead of single counter
     ```

   - **Batch writes**:
     ```bash
     # Instead of individual writes, batch them
     # SDK example: batch.commit() instead of doc.set()
     ```

5. **Query optimization**:
   ```bash
   # Create composite index
   gcloud firestore indexes composite create \
     --collection-group=orders \
     --field-config field-path=status,order=ascending \
     --field-config field-path=created,order=descending
   ```

6. **Scale considerations**:
   - Shard high-write documents
   - Use subcollections for unbounded data
   - Implement pagination for large queries

### Scenario 7: Cloud SQL Backup Restore Taking Too Long
**Problem**: Restoring 500 GB Cloud SQL backup estimated 6+ hours.

**Step-by-step resolution**:
1. **Check backup details**:
   ```bash
   gcloud sql backups list --instance=my-instance
   gcloud sql backups describe BACKUP_ID --instance=my-instance
   ```

2. **Understanding restore times**:
   - Automated backups: Slower (consistent snapshot)
   - Binary logs: Required for point-in-time recovery
   - Larger instances restore faster

3. **Faster alternatives**:

   **Option 1: Clone instead of restore**:
   ```bash
   # Creates new instance from backup (faster)
   gcloud sql instances clone my-instance cloned-instance \
     --point-in-time=2025-01-15T10:30:00.000Z
   ```

   **Option 2: Use read replica**:
   ```bash
   # If replica exists, promote it
   gcloud sql instances promote-replica my-replica
   ```

   **Option 3: Export/Import specific databases**:
   ```bash
   # Export single database
   gcloud sql export sql my-instance gs://my-bucket/db-export.sql \
     --database=mydb

   # Import to new instance (faster for partial restore)
   gcloud sql import sql new-instance gs://my-bucket/db-export.sql \
     --database=mydb
   ```

4. **Optimize future backups**:
   ```bash
   # Enable binary logging for faster PITR
   gcloud sql instances patch my-instance \
     --enable-bin-log \
     --backup-start-time=02:00

   # Set transaction log retention
   gcloud sql instances patch my-instance \
     --retained-transaction-log-days=7
   ```

5. **Create scheduled export for critical data**:
   ```bash
   # Create Cloud Scheduler job for exports
   gcloud scheduler jobs create http export-job \
     --schedule="0 2 * * *" \
     --uri="https://sqladmin.googleapis.com/sql/v1beta4/projects/my-project/instances/my-instance/export" \
     --message-body='{
       "exportContext": {
         "fileType": "SQL",
         "uri": "gs://my-bucket/scheduled-backup.sql",
         "databases": ["mydb"]
       }
     }'
   ```

### Scenario 8: Datastore/Firestore Query Performance Degradation
**Problem**: Queries that took 100ms now taking 5+ seconds.

**Step-by-step resolution**:
1. **Check if indexes are ready**:
   ```bash
   gcloud datastore indexes list
   # Look for status: CREATING vs READY
   ```

2. **Identify missing indexes**:
   - Error message: "no matching index found"
   - Solution: Create composite index

3. **Create required indexes**:
   ```yaml
   # index.yaml
   indexes:
   - kind: Product
     properties:
     - name: category
     - name: price
       direction: desc
     - name: created
       direction: desc
   ```

   ```bash
   gcloud datastore indexes create index.yaml
   ```

4. **Check for exploding indexes**:
   - Multiple list properties = index explosion
   - Each combination needs an index

   **Bad schema**:
   ```python
   {
     "tags": ["electronics", "sale", "featured"],  # list
     "categories": ["phones", "android"],  # list
   }
   # Creates 3 × 2 = 6 index entries
   ```

   **Good schema**:
   ```python
   {
     "tags": "electronics,sale,featured",  # single string
     "categories": ["phones", "android"],
   }
   ```

5. **Optimize query patterns**:
   ```bash
   # Instead of IN query (multiple reads)
   # Bad: WHERE category IN ['a', 'b', 'c']

   # Use separate queries or restructure data
   # Good: WHERE category = 'a'
   ```

6. **Monitor query performance**:
   ```bash
   # Enable Firestore tracing
   gcloud services enable cloudtrace.googleapis.com

   # View traces in Cloud Console
   # Navigation: Trace > Trace List
   ```

## Database Selection Guide

### Comparison Table

| Database | Type | Use Case | Scaling | Consistency | Latency |
|----------|------|----------|---------|-------------|---------|
| **Cloud SQL** | Relational | Traditional apps, OLTP | Vertical (96 cores) | Strong | 5-10ms |
| **Cloud Spanner** | Relational | Global apps, >2TB | Horizontal (unlimited) | Strong | 5-10ms (regional), 100ms (global) |
| **Bigtable** | NoSQL Wide-column | Time-series, IoT, analytics | Horizontal (PB scale) | Eventual | <10ms (single cluster) |
| **Firestore** | NoSQL Document | Mobile/web apps | Automatic | Strong (regional) | 10-20ms |
| **Datastore** | NoSQL Document | Server apps | Automatic | Strong | 10-20ms |
| **BigQuery** | Data warehouse | Analytics, BI | Automatic | Strong | Seconds (analytics) |

### Decision Tree

**Need SQL and ACID transactions?**
- Yes → Need multi-region strong consistency?
  - Yes → **Cloud Spanner**
  - No → Database size > 10TB?
    - Yes → **Cloud Spanner**
    - No → **Cloud SQL**

**NoSQL with low latency?**
- Document model (mobile/web) → **Firestore**
- Document model (server) → **Datastore**
- Wide-column (time-series, IoT) → **Bigtable**
- Key-value (simple) → **Firestore** or **Memorystore**

**Analytics workloads?**
- Interactive SQL analytics → **BigQuery**
- Stream processing → **Dataflow** + **BigQuery**
- Hadoop/Spark jobs → **Dataproc**

### Cost Optimization Tips

1. **Cloud SQL**:
   - Use shared-core instances for dev/test
   - Stop instances during non-business hours
   - Use committed use discounts (CUD)
   - Delete old automated backups
   ```bash
   # Stop instance (no charge except storage)
   gcloud sql instances patch my-instance --activation-policy=NEVER

   # Start instance
   gcloud sql instances patch my-instance --activation-policy=ALWAYS
   ```

2. **Cloud Spanner**:
   - Start with 100 processing units (minimum)
   - Use regional config for single-region apps
   - Scale nodes during peak hours only
   ```bash
   # Scale down during off-peak
   gcloud spanner instances update my-instance --processing-units=100
   ```

3. **Bigtable**:
   - Development instances: 1 node, no replication, no SLA
   - Use autoscaling for variable workloads
   - Delete old data with garbage collection policies
   ```bash
   cbt setgcpolicy my-table cf1 maxage=30d
   ```

4. **BigQuery**:
   - Use partitioned tables (reduce data scanned)
   - Select specific columns, not `SELECT *`
   - Use flat-rate pricing for predictable costs
   - Enable table expiration for temp tables
   ```bash
   bq mk --table --expiration=3600 my_dataset.temp_table
   ```

5. **Cloud Storage**:
   - Use lifecycle policies to transition storage classes
   - Enable Nearline/Coldline for infrequent access
   - Use gsutil -m for parallel uploads (faster)

## Data Migration Patterns

### MySQL to Cloud SQL
```bash
# 1. Export from source MySQL
mysqldump -h source-host -u root -p \
  --databases mydb \
  --single-transaction \
  --set-gtid-purged=OFF \
  > mydb-export.sql

# 2. Upload to Cloud Storage
gsutil cp mydb-export.sql gs://my-bucket/

# 3. Import to Cloud SQL
gcloud sql import sql my-instance \
  gs://my-bucket/mydb-export.sql \
  --database=mydb

# 4. For large databases (>10GB), use parallel export/import
```

### On-premises to BigQuery
```bash
# 1. Export to CSV/JSON
# 2. Upload to Cloud Storage
gsutil -m cp -r ./data gs://my-bucket/bq-data/

# 3. Load to BigQuery
bq load --source_format=CSV \
  --autodetect \
  --max_bad_records=100 \
  my_dataset.my_table \
  gs://my-bucket/bq-data/*.csv

# 4. Use Dataflow for complex transformations
```

### AWS DynamoDB to Bigtable
```bash
# 1. Export DynamoDB to S3
aws dynamodb export-table-to-point-in-time \
  --table-arn arn:aws:dynamodb:region:account:table/MyTable \
  --s3-bucket my-bucket \
  --export-format DYNAMODB_JSON

# 2. Transfer to Cloud Storage
gsutil -m cp -r s3://aws-bucket gs://gcp-bucket

# 3. Use Dataflow template to import to Bigtable
gcloud dataflow jobs run import-job \
  --gcs-location=gs://dataflow-templates/latest/GCS_SequenceFile_to_Cloud_Bigtable \
  --region=us-central1 \
  --parameters=...
```

## Exam Tips

### Key Concepts to Remember

1. **Storage Classes**:
   - Standard: Hot data, no minimum storage
   - Nearline: 30-day minimum, $0.01/GB retrieval
   - Coldline: 90-day minimum, $0.02/GB retrieval
   - Archive: 365-day minimum, $0.05/GB retrieval

2. **Cloud SQL**:
   - Maximum size: 64 TB
   - HA: Synchronous replication to standby (same region)
   - Read replicas: Asynchronous, cross-region supported
   - Automatic backups: Retained for 7 days (configurable)

3. **Spanner**:
   - Minimum: 1 node (2 TB, 10,000 QPS) or 100 processing units
   - Multi-region: 99.999% SLA (5 nines)
   - Regional: 99.99% SLA (4 nines)
   - Best for: >2TB relational data with global scale

4. **Bigtable**:
   - Minimum: 3 nodes for production (1 for dev)
   - Row key design critical for performance
   - Best for: >1TB, >100,000 QPS, time-series data

5. **BigQuery**:
   - Pay per query: $5/TB scanned
   - Partitioning: Reduce costs by 90%+
   - Clustering: Further optimization within partitions
   - Flat-rate: 500 slots minimum

### Common Exam Scenarios

**Scenario: Application needs global SQL database with 99.999% availability**
- Answer: **Cloud Spanner** with multi-region configuration

**Scenario: Store 100TB of time-series IoT data with <10ms latency**
- Answer: **Bigtable** with row key design: `device_id#reverse_timestamp`

**Scenario: Analyze petabytes of logs with SQL**
- Answer: **BigQuery** with partitioned tables

**Scenario: Migrate 500GB MySQL database to GCP**
- Answer: **Cloud SQL** (under 10TB limit)

**Scenario: Document database for mobile app with offline sync**
- Answer: **Firestore Native mode**

**Scenario: Query data in Cloud Storage without loading to database**
- Answer: **BigQuery external tables**

### Common Pitfalls

1. Using Cloud SQL for >10TB (use Spanner)
2. Not partitioning BigQuery tables (high costs)
3. Monotonic row keys in Bigtable (hotspotting)
4. Not enabling binary logs for Cloud SQL PITR
5. Using ACLs instead of IAM for Cloud Storage
6. Not setting lifecycle policies for temp data
7. Forgetting to enable HA for production Cloud SQL

### Performance Optimization

**Cloud Storage**:
- Use parallel uploads: `gsutil -m cp`
- Composite uploads for large files
- Enable CDN for public content

**Cloud SQL**:
- Enable Query Insights
- Add read replicas for read-heavy workloads
- Use connection pooling
- Optimize indexes

**Bigtable**:
- Design row keys for even distribution
- Use column families for different access patterns
- Enable autoscaling
- Use SSD storage (not HDD)

**BigQuery**:
- Partition tables by date
- Cluster on filter/join columns
- Avoid `SELECT *`
- Use materialized views for repeated queries
- Cache results (enabled by default)
