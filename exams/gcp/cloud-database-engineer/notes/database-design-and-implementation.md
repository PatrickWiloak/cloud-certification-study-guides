# Database Design and Implementation - GCP Professional Cloud Database Engineer

## Overview

Database architecture, service selection, schema design, and implementation strategies for GCP database services. This guide covers comprehensive design patterns, migration strategies, and implementation best practices for the Professional Cloud Database Engineer certification.

## Database Service Selection Framework

### Service Comparison Matrix

| Service | Type | Use Case | Scale | Consistency | Availability | Latency |
|---------|------|----------|-------|-------------|--------------|---------|
| Cloud SQL | Relational | OLTP, legacy apps | Vertical (96 vCPU) | Strong | 99.95% | Low (ms) |
| Cloud Spanner | Relational | Global OLTP | Horizontal (unlimited) | Strong (global) | 99.999% | Low (ms) |
| AlloyDB | Relational | HTAP workloads | Vertical (high perf) | Strong | 99.99% | Very Low |
| Firestore | Document | Mobile/web apps | Auto-scale | Strong/Eventual | 99.999% | Low (ms) |
| Bigtable | Wide-column | Time-series, IoT | Horizontal (PB+) | Eventual | 99.9% | Very Low (μs) |
| BigQuery | Data warehouse | Analytics, OLAP | Serverless (PB+) | Eventual | 99.99% | Medium (sec) |
| Memorystore | Cache | Session, caching | Vertical (300 GB) | Eventual | 99.9% | Ultra-low (μs) |

### Decision Tree

```
Start: What type of workload?
│
├─ OLTP (Transactional)
│  │
│  ├─ Global distribution needed?
│  │  ├─ Yes → Cloud Spanner
│  │  └─ No → Regional?
│  │     ├─ High performance PostgreSQL → AlloyDB
│  │     └─ Standard SQL → Cloud SQL
│  │
│  └─ Document-oriented?
│     ├─ Real-time sync needed → Firestore
│     └─ Flexible schema → Firestore
│
├─ OLAP (Analytics)
│  ├─ Data warehouse → BigQuery
│  └─ Real-time analytics on operational data → AlloyDB
│
├─ NoSQL
│  ├─ Time-series data → Bigtable
│  ├─ IoT/high throughput (>10K writes/sec) → Bigtable
│  ├─ Document model → Firestore
│  └─ Key-value with sub-ms latency → Memorystore
│
└─ Caching
   ├─ Session store → Memorystore
   └─ Query result cache → Memorystore
```

## Cloud SQL Deep Dive

### Supported Editions

**MySQL**:
- **MySQL 5.7**: Legacy support, EOL October 2023
- **MySQL 8.0**: Current version, improved performance, JSON support
- **Editions**: Enterprise (all features), Express (testing/dev)

**PostgreSQL**:
- **Versions**: 9.6, 10, 11, 12, 13, 14, 15, 16
- **Extensions**: PostGIS, pgaudit, pg_stat_statements, timescaledb
- **Recommended**: PostgreSQL 14+ for production

**SQL Server**:
- **Versions**: SQL Server 2017, 2019, 2022
- **Editions**:
  - Express: Free, limited to 10 GB, 1 vCPU
  - Web: Web-facing apps, up to 16 vCPU
  - Standard: Full features, up to 24 vCPU
  - Enterprise: All features, up to 96 vCPU

### Machine Types and Sizing

**Shared-core** (Development/Testing):
```
db-f1-micro:  0.6 GB RAM, 1 shared vCPU
db-g1-small:  1.7 GB RAM, 1 shared vCPU
```

**Standard** (General Purpose):
```
db-n1-standard-1:   3.75 GB RAM, 1 vCPU
db-n1-standard-2:   7.5 GB RAM, 2 vCPU
db-n1-standard-4:   15 GB RAM, 4 vCPU
db-n1-standard-8:   30 GB RAM, 8 vCPU
db-n1-standard-16:  60 GB RAM, 16 vCPU
db-n1-standard-32:  120 GB RAM, 32 vCPU
db-n1-standard-64:  240 GB RAM, 64 vCPU
db-n1-standard-96:  360 GB RAM, 96 vCPU
```

**High Memory** (Large databases, in-memory operations):
```
db-n1-highmem-2:   13 GB RAM, 2 vCPU
db-n1-highmem-4:   26 GB RAM, 4 vCPU
db-n1-highmem-8:   52 GB RAM, 8 vCPU
db-n1-highmem-16:  104 GB RAM, 16 vCPU
db-n1-highmem-32:  208 GB RAM, 32 vCPU
db-n1-highmem-64:  416 GB RAM, 64 vCPU
db-n1-highmem-96:  624 GB RAM, 96 vCPU
```

**Sizing Calculation Example**:
```
Workload: E-commerce database
- Active dataset: 50 GB
- Peak connections: 500
- Peak transactions: 1000 TPS
- Read/Write ratio: 70/30

Calculations:
1. Memory for buffer pool: 50 GB × 1.5 = 75 GB
2. Memory for connections: 500 × 10 MB = 5 GB
3. OS overhead: 10 GB
4. Total RAM needed: 90 GB

Recommendation: db-n1-highmem-16 (104 GB RAM, 16 vCPU)
Storage: 100 GB SSD (50 GB data + 50% growth + backups)
```

### High Availability Configuration

**Regional HA (Recommended)**:
```bash
gcloud sql instances create production-db \
  --database-version=POSTGRES_15 \
  --tier=db-n1-standard-4 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --enable-bin-log \
  --backup-start-time=03:00 \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7 \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=04 \
  --maintenance-window-duration=4
```

**HA Architecture**:
```
Region: us-central1
├─ Primary Zone: us-central1-a
│  └─ Primary Instance (active)
│
└─ Secondary Zone: us-central1-b
   └─ Standby Instance (synchronous replication)

Automatic Failover:
- RTO: ~60 seconds
- RPO: 0 (no data loss with synchronous replication)
- Automatic: Yes (transparent to applications)
```

**Failover Behavior**:
- Connection string remains the same (DNS-based)
- Automatic health checks every 10 seconds
- Failover triggers: zone failure, instance crash, maintenance
- Manual failover supported for testing

### Read Replicas

**Types of Read Replicas**:
1. **In-region Read Replica**: Same region as primary
2. **Cross-region Read Replica**: Different region from primary
3. **Cascading Read Replica**: Replica of a replica (up to 3 levels)
4. **External Read Replica**: Replicate to external MySQL/PostgreSQL

**Creating Read Replicas**:
```bash
# In-region read replica
gcloud sql instances create production-db-replica-1 \
  --master-instance-name=production-db \
  --tier=db-n1-standard-2 \
  --replica-type=READ \
  --availability-type=ZONAL \
  --zone=us-central1-b

# Cross-region read replica
gcloud sql instances create production-db-replica-eu \
  --master-instance-name=production-db \
  --tier=db-n1-standard-2 \
  --replica-type=READ \
  --availability-type=ZONAL \
  --region=europe-west1

# Cascading replica (replica of replica)
gcloud sql instances create production-db-replica-2 \
  --master-instance-name=production-db-replica-1 \
  --tier=db-n1-standard-1 \
  --replica-type=READ \
  --zone=us-central1-c
```

**Read Replica Architecture**:
```
Primary: us-central1-a (production-db)
  │
  ├─ Read Replica 1: us-central1-b (same region, different zone)
  │  └─ Cascading Replica: us-central1-c (reads from Replica 1)
  │
  ├─ Read Replica 2: us-east1-a (cross-region for DR)
  │
  └─ Read Replica 3: europe-west1-b (cross-region for global reads)

Replication: Asynchronous (lag typically < 1 second)
Use Cases:
- Load balancing read queries
- Geographic distribution
- Disaster recovery
- Analytics without impacting primary
```

**Promoting Read Replica to Standalone**:
```bash
# Promote replica to standalone instance
gcloud sql instances promote-replica production-db-replica-eu

# Note: This breaks replication and makes it an independent instance
```

### Automated Backups and PITR

**Backup Configuration**:
```bash
gcloud sql instances create production-db \
  --backup-start-time=03:00 \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7 \
  --enable-bin-log \
  --enable-point-in-time-recovery \
  --transaction-log-retention-days=7
```

**Backup Types**:
1. **Automated Backups**:
   - Daily backups at specified time
   - Retained for 1-365 days (default: 7)
   - Includes transaction logs for PITR
   - Stored in same region as instance

2. **On-Demand Backups**:
   ```bash
   gcloud sql backups create \
     --instance=production-db \
     --description="Pre-migration backup"
   ```

3. **Export to Cloud Storage**:
   ```bash
   gcloud sql export sql production-db gs://my-bucket/backup.sql \
     --database=mydb \
     --offload
   ```

**Point-in-Time Recovery (PITR)**:
```bash
# Clone instance to specific point in time
gcloud sql instances clone production-db production-db-pitr \
  --point-in-time='2024-01-15T10:30:00.000Z'

# Restore to new instance
gcloud sql backups restore BACKUP_ID \
  --backup-instance=production-db \
  --restore-instance=production-db-restored
```

**PITR Requirements**:
- Binary logging enabled (MySQL) or WAL archiving (PostgreSQL)
- Transaction logs retained (1-7 days)
- Available for time range: last backup to present
- RPO: As low as 5 minutes

**Backup Best Practices**:
```bash
# Production configuration
gcloud sql instances patch production-db \
  --backup-start-time=03:00 \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7 \
  --backup-location=us

# Verify backups
gcloud sql backups list --instance=production-db

# Test restore process monthly
gcloud sql instances clone production-db test-restore-$(date +%Y%m%d)
```

### Connection Management

**Connection Methods**:
1. **Private IP (Recommended)**:
   ```bash
   gcloud sql instances patch production-db \
     --network=projects/PROJECT_ID/global/networks/vpc-network \
     --no-assign-ip
   ```

2. **Public IP with Cloud SQL Auth Proxy**:
   ```bash
   ./cloud-sql-proxy INSTANCE_CONNECTION_NAME
   ```

3. **Cloud SQL Connector Libraries**:
   ```python
   # Python example
   from google.cloud.sql.connector import Connector

   connector = Connector()

   def getconn():
       conn = connector.connect(
           "project:region:instance",
           "pg8000",
           user="postgres",
           password="password",
           db="mydb"
       )
       return conn
   ```

**Connection Pooling**:
```python
# Python with SQLAlchemy
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    "postgresql+pg8000://",
    creator=getconn,
    pool_size=5,        # Connection pool size
    max_overflow=2,      # Max connections beyond pool_size
    pool_timeout=30,     # Timeout for getting connection
    pool_recycle=1800    # Recycle connections after 30 min
)
```

### Cloud SQL Use Cases and Schema Examples

**Use Case 1: E-commerce Platform**
```sql
-- PostgreSQL schema for e-commerce
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INTEGER NOT NULL DEFAULT 0,
    category_id INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(50) DEFAULT 'pending',
    subtotal DECIMAL(10, 2) NOT NULL,
    tax DECIMAL(10, 2) NOT NULL,
    shipping DECIMAL(10, 2) NOT NULL,
    total DECIMAL(10, 2) NOT NULL,
    shipping_address_id INTEGER
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    subtotal DECIMAL(10, 2) NOT NULL
);

-- Indexes for performance
CREATE INDEX idx_customers_email ON customers(email);
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date DESC);
CREATE INDEX idx_orders_status ON orders(status) WHERE status != 'completed';
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_products_category ON products(category_id);

-- Partition large tables by date
CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
```

**Use Case 2: SaaS Multi-Tenant Application**
```sql
-- Row-level security for multi-tenancy
CREATE TABLE tenants (
    tenant_id SERIAL PRIMARY KEY,
    tenant_name VARCHAR(255) UNIQUE NOT NULL,
    subscription_tier VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tenant_users (
    user_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL REFERENCES tenants(tenant_id),
    email VARCHAR(255) NOT NULL,
    role VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(tenant_id, email)
);

CREATE TABLE tenant_data (
    data_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL REFERENCES tenants(tenant_id),
    user_id INTEGER REFERENCES tenant_users(user_id),
    data_type VARCHAR(50),
    data_value JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Enable row-level security
ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their tenant's data
CREATE POLICY tenant_isolation ON tenant_data
    USING (tenant_id = current_setting('app.current_tenant')::INTEGER);

-- Indexes
CREATE INDEX idx_tenant_data_tenant ON tenant_data(tenant_id);
CREATE INDEX idx_tenant_data_jsonb ON tenant_data USING GIN(data_value);
```

## Cloud Spanner Architecture

### Regional vs Multi-Region Configurations

**Regional Configuration**:
```
Config: regional-us-central1
Locations: us-central1
Replication: 3 replicas within region
Latency: Single-digit ms
Use Case: Single region applications, lower cost
```

**Multi-Region Configuration**:
```
Config: nam6 (North America)
Locations:
  - us-central1 (Iowa)
  - us-east4 (Northern Virginia)
  - us-west1 (Oregon)
Replication: 3 replicas per region (9 total)
Latency: Low within continent
Use Case: Continental applications

Config: nam-eur-asia1 (Global)
Locations:
  - us-central1
  - europe-west1
  - asia-northeast1
Replication: Distributed globally
Latency: Higher for writes, optimized reads
Use Case: Global applications
```

**Configuration Selection**:
```bash
# Regional instance (lowest cost, single-region)
gcloud spanner instances create prod-spanner-regional \
  --config=regional-us-central1 \
  --nodes=1 \
  --description="Regional Spanner"

# Multi-region instance (high availability, global)
gcloud spanner instances create prod-spanner-global \
  --config=nam-eur-asia1 \
  --nodes=3 \
  --description="Global Spanner"

# List available configurations
gcloud spanner instance-configs list
```

### Instances and Nodes

**Node Capacity**:
- 1 node = 1000 processing units (PU)
- 1 node ≈ 2 TB storage
- 1 node ≈ 10,000 QPS (reads)
- 1 node ≈ 2,000 QPS (writes)

**Sizing Calculation**:
```
Application Requirements:
- Storage: 5 TB
- Read QPS: 25,000
- Write QPS: 5,000
- Peak CPU: < 65%

Calculations:
1. Storage-based: 5 TB / 2 TB = 3 nodes
2. Read-based: 25,000 / 10,000 = 3 nodes
3. Write-based: 5,000 / 2,000 = 3 nodes

Recommendation: Start with 3 nodes
Monitor CPU and scale if > 65% sustained
```

**Autoscaling**:
```bash
# Set autoscaling (Preview feature)
gcloud spanner instances update prod-spanner \
  --autoscaling-min-nodes=1 \
  --autoscaling-max-nodes=5 \
  --autoscaling-high-priority-cpu-target=65 \
  --autoscaling-storage-target=95

# Manual scaling
gcloud spanner instances update prod-spanner --nodes=5
```

### Database Schemas and Interleaved Tables

**Parent-Child Interleaving**:
```sql
-- Parent table
CREATE TABLE Customers (
    CustomerId INT64 NOT NULL,
    CustomerName STRING(MAX),
    Email STRING(MAX),
    CreatedAt TIMESTAMP NOT NULL OPTIONS (
        allow_commit_timestamp=true
    ),
) PRIMARY KEY (CustomerId);

-- Child table interleaved in parent
CREATE TABLE Orders (
    CustomerId INT64 NOT NULL,
    OrderId INT64 NOT NULL,
    OrderDate TIMESTAMP NOT NULL,
    Status STRING(50),
    TotalAmount FLOAT64,
) PRIMARY KEY (CustomerId, OrderId),
  INTERLEAVE IN PARENT Customers ON DELETE CASCADE;

-- Grandchild table
CREATE TABLE OrderItems (
    CustomerId INT64 NOT NULL,
    OrderId INT64 NOT NULL,
    ItemId INT64 NOT NULL,
    ProductId INT64 NOT NULL,
    Quantity INT64,
    Price FLOAT64,
) PRIMARY KEY (CustomerId, OrderId, ItemId),
  INTERLEAVE IN PARENT Orders ON DELETE CASCADE;
```

**Benefits of Interleaving**:
- Co-located data on same splits
- Efficient parent-child queries
- Atomic operations across hierarchy
- Reduced cross-split reads

**When NOT to Interleave**:
- Child accessed independently of parent
- Child has different access patterns
- Child grows much larger than parent

### Secondary Indexes

**Index Types and Strategies**:
```sql
-- Basic secondary index
CREATE INDEX OrdersByDate ON Orders(OrderDate);

-- Composite index
CREATE INDEX OrdersByStatusDate ON Orders(Status, OrderDate);

-- Index with STORING clause (covering index)
CREATE INDEX OrdersByDate ON Orders(OrderDate)
    STORING (Status, TotalAmount);

-- NULL-filtered index
CREATE NULL_FILTERED INDEX PendingOrders ON Orders(OrderDate)
    WHERE Status = 'pending';

-- Interleaved index (follows parent-child structure)
CREATE INDEX OrdersByCustomerDate ON Orders(CustomerId, OrderDate),
    INTERLEAVE IN Customers;
```

**Index Selection Strategy**:
```sql
-- Query: Find orders by status and date
SELECT * FROM Orders
WHERE Status = 'pending'
  AND OrderDate >= '2024-01-01'
ORDER BY OrderDate;

-- Optimal index
CREATE INDEX OrdersByStatusDate ON Orders(Status, OrderDate)
    STORING (CustomerId, TotalAmount);

-- Query: Get customer with orders
SELECT c.*, o.*
FROM Customers c
JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE c.CustomerId = 12345;

-- Best: Use interleaved tables (no index needed)
-- Alternative: Create interleaved index on Orders
```

### Query Optimization

**Query Execution Plan**:
```sql
-- Use EXPLAIN to see query plan
@{USE_ADDITIONAL_PARALLELISM=TRUE}
SELECT
    c.CustomerName,
    COUNT(*) as OrderCount,
    SUM(o.TotalAmount) as TotalSpent
FROM Customers c
JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.OrderDate >= '2024-01-01'
GROUP BY c.CustomerName;

-- Optimize with query hints
@{JOIN_METHOD=HASH_JOIN}
@{USE_ADDITIONAL_PARALLELISM=TRUE}
SELECT * FROM LargeTable1 t1
JOIN LargeTable2 t2 ON t1.id = t2.id;
```

**Avoiding Hotspots**:
```sql
-- BAD: Monotonically increasing primary key
CREATE TABLE Events (
    EventId INT64 NOT NULL,  -- Sequential: causes hotspot
    EventTime TIMESTAMP,
    Data STRING(MAX),
) PRIMARY KEY (EventId);

-- GOOD: Use hash or UUID
CREATE TABLE Events (
    EventId STRING(36) NOT NULL,  -- UUID: distributed
    EventTime TIMESTAMP,
    Data STRING(MAX),
) PRIMARY KEY (EventId);

-- GOOD: Bit-reverse sequential
CREATE TABLE Events (
    EventId INT64 NOT NULL,
    EventTime TIMESTAMP,
    Data STRING(MAX),
) PRIMARY KEY (BIT_REVERSE(EventId, true));  -- Distributes writes

-- GOOD: Hash prefix
CREATE TABLE TimeSeries (
    DeviceId STRING(MAX) NOT NULL,
    Timestamp INT64 NOT NULL,
    Metric FLOAT64,
) PRIMARY KEY (FARM_FINGERPRINT(DeviceId), DeviceId, Timestamp);
```

**Batch Operations**:
```python
# Python client library
from google.cloud import spanner

client = spanner.Client()
instance = client.instance('prod-spanner')
database = instance.database('orders-db')

# Batch insert (efficient)
with database.batch() as batch:
    batch.insert(
        table='Orders',
        columns=['CustomerId', 'OrderId', 'OrderDate', 'TotalAmount'],
        values=[
            (1001, 5001, '2024-01-15', 99.99),
            (1002, 5002, '2024-01-15', 149.99),
            (1003, 5003, '2024-01-15', 199.99),
        ]
    )

# Read-only transaction (snapshot reads)
with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        'SELECT * FROM Orders WHERE CustomerId = @customer_id',
        params={'customer_id': 1001},
        param_types={'customer_id': spanner.param_types.INT64}
    )
```

### Transactions and Consistency

**Transaction Types**:
```python
# Read-Write Transaction (serializable)
def transfer_money(transaction):
    # Read account balances
    row = transaction.execute_sql(
        'SELECT Balance FROM Accounts WHERE AccountId = 1'
    ).one()
    balance = row[0]

    # Update balances
    transaction.update(
        table='Accounts',
        columns=['AccountId', 'Balance'],
        values=[(1, balance - 100), (2, balance + 100)]
    )

database.run_in_transaction(transfer_money)

# Read-Only Transaction (snapshot)
with database.snapshot(read_timestamp=timestamp) as snapshot:
    results = snapshot.execute_sql('SELECT * FROM Orders')

# Stale Reads (for eventual consistency, better performance)
with database.snapshot(exact_staleness=datetime.timedelta(seconds=15)) as snapshot:
    results = snapshot.execute_sql('SELECT * FROM Analytics')
```

**Cloud Spanner Best Practices**:
```sql
-- Use commit timestamps for versioning
CREATE TABLE Accounts (
    AccountId INT64 NOT NULL,
    Balance FLOAT64,
    LastModified TIMESTAMP NOT NULL OPTIONS (
        allow_commit_timestamp=true
    ),
) PRIMARY KEY (AccountId);

-- Use ARRAY for repeated fields
CREATE TABLE Users (
    UserId INT64 NOT NULL,
    Tags ARRAY<STRING(MAX)>,
) PRIMARY KEY (UserId);

-- Use STRUCT for complex types
CREATE TABLE Products (
    ProductId INT64 NOT NULL,
    Details STRUCT<
        Name STRING(MAX),
        Price FLOAT64,
        Dimensions STRUCT<Length FLOAT64, Width FLOAT64>
    >,
) PRIMARY KEY (ProductId);
```

## Firestore / Datastore

### Native Mode vs Datastore Mode

**Firestore Native Mode**:
- Real-time updates via listeners
- Client libraries for mobile/web
- Stronger consistency guarantees
- Richer querying capabilities
- Better for new applications

**Datastore Mode**:
- Server-client architecture (REST API)
- Compatible with legacy Datastore
- Eventual consistency option
- Better for server-side workloads
- Migration path from Datastore

**Choosing Between Modes**:
```
Use Firestore Native Mode:
✓ Mobile or web applications
✓ Real-time synchronization needed
✓ Offline support required
✓ New greenfield projects
✓ Need collection group queries

Use Datastore Mode:
✓ Server-to-server applications
✓ Migrating from Datastore
✓ No real-time updates needed
✓ Existing App Engine Standard apps
✓ Need entity group transactions
```

### Data Model Design

**Document Structure**:
```javascript
// Collection: users
// Document: user_12345
{
    "userId": "user_12345",
    "email": "user@example.com",
    "profile": {
        "firstName": "John",
        "lastName": "Doe",
        "age": 30,
        "avatar": "gs://bucket/avatars/user_12345.jpg"
    },
    "preferences": {
        "notifications": true,
        "theme": "dark",
        "language": "en"
    },
    "metadata": {
        "createdAt": "2024-01-15T10:30:00Z",
        "updatedAt": "2024-01-16T14:20:00Z",
        "version": 2
    }
}

// Subcollection: users/{userId}/orders
// Document: order_67890
{
    "orderId": "order_67890",
    "orderDate": "2024-01-15T10:30:00Z",
    "status": "shipped",
    "items": [
        {
            "productId": "prod_111",
            "name": "Widget",
            "quantity": 2,
            "price": 29.99
        },
        {
            "productId": "prod_222",
            "name": "Gadget",
            "quantity": 1,
            "price": 49.99
        }
    ],
    "total": 109.97,
    "shippingAddress": {
        "street": "123 Main St",
        "city": "Anytown",
        "state": "CA",
        "zip": "12345"
    }
}
```

**Denormalization Strategies**:
```javascript
// Strategy 1: Embed frequently accessed data
{
    "orderId": "order_67890",
    "customer": {
        "id": "user_12345",
        "name": "John Doe",        // Denormalized
        "email": "user@example.com"  // Denormalized
    },
    "items": [/* ... */],
    "total": 109.97
}

// Strategy 2: Use subcollections for one-to-many
// users/{userId}/orders/{orderId}
// users/{userId}/addresses/{addressId}
// users/{userId}/paymentMethods/{methodId}

// Strategy 3: Use references for many-to-many
{
    "orderId": "order_67890",
    "productRefs": [
        "products/prod_111",
        "products/prod_222"
    ]
}

// Strategy 4: Use arrays for small lists
{
    "userId": "user_12345",
    "favoriteProductIds": ["prod_111", "prod_333", "prod_555"],
    "tags": ["premium", "verified", "early-adopter"]
}
```

**Design Patterns**:
```javascript
// Pattern 1: Activity Feed (subcollection)
// users/{userId}/feed/{feedItemId}
{
    "type": "order_placed",
    "timestamp": "2024-01-15T10:30:00Z",
    "data": {
        "orderId": "order_67890",
        "total": 109.97
    }
}

// Pattern 2: Counters (sharded)
// counters/product_views/shards/{shardId}
{
    "count": 1247
}

// Pattern 3: Metadata for collections
// metadata/collections/users
{
    "totalCount": 10000,
    "lastUpdated": "2024-01-16T10:00:00Z"
}
```

### Index Strategies

**Single-Field Indexes** (Automatic):
```javascript
// Automatically indexed on each field
db.collection('products').where('category', '==', 'electronics')
db.collection('products').where('price', '>', 100)
db.collection('products').orderBy('name')
```

**Composite Indexes** (Manual):
```bash
# Create composite index via CLI
gcloud firestore indexes composite create \
  --collection-group=orders \
  --field-config=field-path=customerId,order=ASCENDING \
  --field-config=field-path=orderDate,order=DESCENDING \
  --field-config=field-path=status,order=ASCENDING

# Or via index definition file
gcloud firestore indexes composite create --field-config-file=firestore.indexes.json
```

**Index Definition File**:
```json
{
  "indexes": [
    {
      "collectionGroup": "orders",
      "queryScope": "COLLECTION",
      "fields": [
        {
          "fieldPath": "customerId",
          "order": "ASCENDING"
        },
        {
          "fieldPath": "orderDate",
          "order": "DESCENDING"
        },
        {
          "fieldPath": "status",
          "order": "ASCENDING"
        }
      ]
    },
    {
      "collectionGroup": "orders",
      "queryScope": "COLLECTION_GROUP",
      "fields": [
        {
          "fieldPath": "status",
          "order": "ASCENDING"
        },
        {
          "fieldPath": "total",
          "order": "DESCENDING"
        }
      ]
    }
  ]
}
```

**Collection Group Indexes**:
```javascript
// Query across all subcollections named "orders"
// Requires collection group index
db.collectionGroup('orders')
  .where('status', '==', 'pending')
  .orderBy('orderDate', 'desc')
  .limit(10)
```

**Index Exemptions**:
```json
{
  "fieldOverrides": [
    {
      "collectionGroup": "logs",
      "fieldPath": "message",
      "indexes": []  // Disable indexing on this field
    },
    {
      "collectionGroup": "users",
      "fieldPath": "searchTerms",
      "indexes": [
        {
          "queryScope": "COLLECTION",
          "order": "ASCENDING"
        }
      ]
    }
  ]
}
```

### Queries and Transactions

**Query Examples**:
```javascript
// Simple queries
db.collection('products')
  .where('category', '==', 'electronics')
  .where('price', '<', 500)
  .orderBy('price', 'desc')
  .limit(10)
  .get()

// Range queries (requires composite index for multiple)
db.collection('events')
  .where('date', '>=', startDate)
  .where('date', '<=', endDate)
  .where('type', '==', 'user_action')
  .orderBy('date', 'desc')

// Array membership
db.collection('users')
  .where('tags', 'array-contains', 'premium')

// Array contains any
db.collection('users')
  .where('tags', 'array-contains-any', ['premium', 'verified'])

// IN queries
db.collection('products')
  .where('productId', 'in', ['prod_111', 'prod_222', 'prod_333'])

// NOT EQUAL (requires composite index)
db.collection('orders')
  .where('status', '!=', 'cancelled')
  .orderBy('status')  // Required with !=
  .orderBy('orderDate', 'desc')
```

**Transactions**:
```javascript
// Read-Write Transaction
const orderRef = db.collection('orders').doc('order_123');
const inventoryRef = db.collection('inventory').doc('prod_456');

await db.runTransaction(async (transaction) => {
    // Read phase
    const orderDoc = await transaction.get(orderRef);
    const inventoryDoc = await transaction.get(inventoryRef);

    const quantity = orderDoc.data().quantity;
    const available = inventoryDoc.data().available;

    if (available < quantity) {
        throw new Error('Insufficient inventory');
    }

    // Write phase
    transaction.update(inventoryRef, {
        available: available - quantity
    });

    transaction.update(orderRef, {
        status: 'confirmed'
    });
});

// Batch Write (non-transactional, up to 500 operations)
const batch = db.batch();

batch.set(db.collection('products').doc('prod_1'), {name: 'Product 1'});
batch.update(db.collection('products').doc('prod_2'), {price: 29.99});
batch.delete(db.collection('products').doc('prod_3'));

await batch.commit();
```

### Real-Time Updates

**Snapshot Listeners**:
```javascript
// Listen to document changes
const unsubscribe = db.collection('orders').doc('order_123')
  .onSnapshot((doc) => {
    console.log('Current data:', doc.data());
  }, (error) => {
    console.error('Error:', error);
  });

// Listen to query changes
db.collection('orders')
  .where('status', '==', 'pending')
  .onSnapshot((snapshot) => {
    snapshot.docChanges().forEach((change) => {
      if (change.type === 'added') {
        console.log('New order:', change.doc.data());
      }
      if (change.type === 'modified') {
        console.log('Updated order:', change.doc.data());
      }
      if (change.type === 'removed') {
        console.log('Removed order:', change.doc.data());
      }
    });
  });

// Detach listener
unsubscribe();
```

**Offline Persistence**:
```javascript
// Enable offline persistence (mobile/web)
firebase.firestore().enablePersistence()
  .then(() => {
    console.log('Offline persistence enabled');
  })
  .catch((err) => {
    if (err.code == 'failed-precondition') {
      // Multiple tabs open
    } else if (err.code == 'unimplemented') {
      // Browser doesn't support
    }
  });

// Check if data is from cache
db.collection('products').doc('prod_123')
  .get({ source: 'cache' })
  .then((doc) => {
    if (doc.metadata.fromCache) {
      console.log('Data from cache');
    }
  });
```

## Cloud Bigtable Design

### Instance Types and Configuration

**Instance Types**:
```bash
# Production instance (replication, HA)
gcloud bigtable instances create prod-bigtable \
  --display-name="Production Bigtable" \
  --cluster-config=id=cluster-us-central1,zone=us-central1-a,nodes=3 \
  --cluster-config=id=cluster-us-east1,zone=us-east1-b,nodes=3

# Development instance (no replication, 1-2 nodes)
gcloud bigtable instances create dev-bigtable \
  --display-name="Development Bigtable" \
  --cluster-config=id=cluster-dev,zone=us-central1-a,nodes=1 \
  --instance-type=DEVELOPMENT
```

**Cluster Configuration**:
```
Minimum Requirements:
- Production: 1 node per cluster (recommend 3+)
- Development: 1 node total
- Node capacity: ~10 GB/s throughput per node
- Storage: No limit (pay for usage)

Scaling:
- Horizontal: Add nodes for more throughput
- Nodes can be added/removed dynamically
- Rebalancing is automatic
- ~10 minutes to add/remove nodes
```

**Replication**:
```bash
# Add replication cluster
gcloud bigtable clusters create cluster-europe \
  --instance=prod-bigtable \
  --zone=europe-west1-b \
  --nodes=3

# Set replication for app profiles
gcloud bigtable app-profiles create multi-cluster \
  --instance=prod-bigtable \
  --route-any

# Single cluster routing (lower latency)
gcloud bigtable app-profiles create us-central1-profile \
  --instance=prod-bigtable \
  --route-to=cluster-us-central1
```

### Table Design Principles

**Key Concepts**:
- Row Key: Primary key, sorted lexicographically
- Column Family: Group of columns, defined at table creation
- Column Qualifier: Individual columns within family
- Cell: Intersection of row and column, timestamped
- Versions: Multiple timestamped values per cell

**Table Creation**:
```python
from google.cloud import bigtable
from google.cloud.bigtable import column_family

client = bigtable.Client(project='my-project', admin=True)
instance = client.instance('prod-bigtable')
table = instance.table('metrics')

# Create table with column families
cf_metadata = column_family.MaxVersionsGCRule(1)  # Keep latest only
cf_metrics = column_family.MaxVersionsGCRule(100)  # Keep 100 versions
cf_events = column_family.MaxAgeGCRule(datetime.timedelta(days=30))  # 30 day TTL

table.create()
table.column_family('metadata', gc_rule=cf_metadata)
table.column_family('metrics', gc_rule=cf_metrics)
table.column_family('events', gc_rule=cf_events)
```

### Row Key Patterns

**Pattern 1: Time-Series Data (Reverse Timestamp)**:
```python
# BAD: Sequential timestamp causes hotspot
row_key = f"{timestamp}#{device_id}"
# All writes go to same server (end of keyspace)

# GOOD: Reverse timestamp distributes writes
import sys
max_timestamp = 2**63 - 1
reverse_timestamp = max_timestamp - int(timestamp * 1000)
row_key = f"{device_id}#{reverse_timestamp}"

# Example:
# device_001#9223370036854775807  (most recent)
# device_001#9223370036854775000
# device_001#9223370036854774000  (oldest)
```

**Pattern 2: Field Promotion (Move Key Data to Row Key)**:
```python
# BAD: Query requires scanning all rows
row_key = f"{user_id}"
# Query for region='us-west' requires full scan

# GOOD: Promote region to row key
row_key = f"{region}#{user_id}"
# Now can efficiently query all users in 'us-west'

# Examples:
# us-west#user_001
# us-west#user_002
# us-east#user_003
```

**Pattern 3: Hash Prefix (Distribute Hot Keys)**:
```python
# BAD: Popular devices cause hotspots
row_key = f"{device_id}#{timestamp}"

# GOOD: Hash prefix distributes load
import hashlib
hash_prefix = hashlib.md5(device_id.encode()).hexdigest()[:4]
row_key = f"{hash_prefix}#{device_id}#{timestamp}"

# Examples:
# a3f2#device_popular#1234567890
# b7e1#device_popular#1234567891
# c4d9#device_normal#1234567892
```

**Pattern 4: Reverse Domain (Natural Grouping)**:
```python
# For website analytics
# BAD: Forward domain
row_key = f"{domain}#{page}"  # example.com#/page1

# GOOD: Reverse domain groups related sites
row_key = f"{reverse_domain}#{page}"  # com.example#/page1

# Examples:
# com.example#/products
# com.example#/about
# com.example.blog#/post1
```

### Column Family Design

**Best Practices**:
```python
# Organize by access patterns
column_families = {
    # Frequently accessed, hot data
    'summary': {
        'max_versions': 1,
        'max_age': None
    },
    # Historical data, multiple versions
    'history': {
        'max_versions': 100,
        'max_age': None
    },
    # Temporary data with TTL
    'temp': {
        'max_versions': 1,
        'max_age_days': 7
    },
    # Large infrequently accessed data
    'raw_data': {
        'max_versions': 1,
        'max_age': None
    }
}

# Create column families with different GC rules
from google.cloud.bigtable import column_family

# Keep only latest version
cf_summary = column_family.MaxVersionsGCRule(1)

# Keep 100 versions for audit trail
cf_history = column_family.MaxVersionsGCRule(100)

# Keep data for 30 days
cf_temp = column_family.MaxAgeGCRule(datetime.timedelta(days=30))

# Combine rules: Keep 10 versions OR data newer than 7 days
cf_combined = column_family.GCRuleUnion([
    column_family.MaxVersionsGCRule(10),
    column_family.MaxAgeGCRule(datetime.timedelta(days=7))
])
```

### Compaction and Performance

**Compaction**:
- Background process that merges SSTables
- Removes deleted data and expired cells
- Automatic but can be monitored
- Impacts read performance if behind

**Write Performance**:
```python
# Batch writes for efficiency
from google.cloud import bigtable
from google.cloud.bigtable import row

client = bigtable.Client(project='my-project', admin=True)
instance = client.instance('prod-bigtable')
table = instance.table('metrics')

# Batch mutations
rows = []
for i in range(1000):
    row_key = f"device_{i}#timestamp"
    r = table.direct_row(row_key)
    r.set_cell('metrics', 'temperature', str(temp_value))
    r.set_cell('metrics', 'humidity', str(humidity_value))
    rows.append(r)

# Write in batches of 100
response = table.mutate_rows(rows)
```

**Read Performance**:
```python
# Use row key prefixes for efficient scans
row_set = row_set.RowSet()
row_set.add_row_range(
    row_range.RowRange(
        start_key=b"device_001#",
        end_key=b"device_001~"
    )
)

# Read with filters
from google.cloud.bigtable import row_filters

# Only read specific columns
col_filter = row_filters.ColumnQualifierRegexFilter(b'temperature')

# Limit cells per column
limit_filter = row_filters.CellsColumnLimitFilter(1)

# Chain filters
chain = row_filters.RowFilterChain([col_filter, limit_filter])

rows = table.read_rows(row_set=row_set, filter_=chain)
```

### Bigtable Use Cases and Schema Examples

**Use Case 1: IoT Time-Series Data**
```python
# Schema design for sensor data
# Row Key: {sensor_id}#{reverse_timestamp}
# Column Families: metadata, metrics

# Table: iot_sensors
column_families = {
    'metadata': MaxVersionsGCRule(1),  # Latest device info
    'metrics': MaxVersionsGCRule(100),  # Keep history
}

# Example row
row_key = "sensor_001#9223370036854775807"
columns = {
    'metadata:location': 'warehouse_a',
    'metadata:type': 'temperature_sensor',
    'metrics:temperature': '72.5',
    'metrics:humidity': '45.2',
    'metrics:battery': '87'
}

# Query: Get latest readings for sensor
def get_latest_readings(sensor_id, limit=10):
    row_set = RowSet()
    row_set.add_row_range(
        RowRange(
            start_key=f"{sensor_id}#".encode(),
            end_key=f"{sensor_id}~".encode()
        )
    )

    limit_filter = row_filters.CellsRowLimitFilter(limit)
    return table.read_rows(row_set=row_set, filter_=limit_filter)
```

**Use Case 2: Financial Market Data**
```python
# Schema design for stock ticks
# Row Key: {exchange}#{symbol}#{reverse_timestamp}

# Table: market_data
column_families = {
    'trade': MaxVersionsGCRule(1),
    'quote': MaxVersionsGCRule(1),
    'daily': MaxAgeGCRule(timedelta(days=365))
}

# Example row
row_key = "NASDAQ#AAPL#9223370036854775807"
columns = {
    'trade:price': '150.25',
    'trade:volume': '1000',
    'quote:bid': '150.20',
    'quote:ask': '150.30',
    'daily:open': '149.50',
    'daily:high': '151.00',
    'daily:low': '149.00'
}

# Query: Get all trades for a symbol in time range
def get_trades(exchange, symbol, start_time, end_time):
    max_timestamp = 2**63 - 1
    start_reverse = max_timestamp - int(end_time * 1000)
    end_reverse = max_timestamp - int(start_time * 1000)

    row_set = RowSet()
    row_set.add_row_range(
        RowRange(
            start_key=f"{exchange}#{symbol}#{start_reverse}".encode(),
            end_key=f"{exchange}#{symbol}#{end_reverse}".encode()
        )
    )

    return table.read_rows(row_set=row_set)
```

## Memorystore for Redis

### Redis Standard vs Enterprise

**Redis Standard**:
```bash
# Basic tier (no replication)
gcloud redis instances create basic-cache \
  --size=1 \
  --region=us-central1 \
  --tier=BASIC

# Standard tier (HA with replication)
gcloud redis instances create standard-cache \
  --size=5 \
  --region=us-central1 \
  --tier=STANDARD
```

**Configuration Comparison**:
```
Basic Tier:
- Size: 1-300 GB
- Availability: No SLA
- Replication: None
- Use Case: Development, caching

Standard Tier:
- Size: 1-300 GB
- Availability: 99.9% SLA
- Replication: Async replication to standby
- Automatic failover: Yes
- Use Case: Production caching

Enterprise Tiers (Memorystore for Redis Cluster):
- Size: 5 GB - 10 TB
- Availability: 99.99% SLA (regional) or 99.99% (zonal)
- Features: Clustering, RDB/AOF persistence, active-active replication
- Use Case: High availability, large datasets
```

### High Availability and Replication

**Standard Tier HA**:
```bash
gcloud redis instances create prod-cache \
  --size=10 \
  --region=us-central1 \
  --tier=STANDARD \
  --redis-version=redis_6_x \
  --enable-auth \
  --reserved-ip-range=10.0.0.0/29 \
  --network=projects/PROJECT/global/networks/vpc
```

**Architecture**:
```
Standard Tier HA:
├─ Primary Node: us-central1-a
│  └─ Serves read/write traffic
│
└─ Replica Node: us-central1-b
   └─ Async replication from primary
   └─ Automatic failover (RTO: < 2 minutes)
```

**Redis Cluster (Enterprise)**:
```bash
gcloud redis clusters create enterprise-cache \
  --shard-count=3 \
  --replica-count=1 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/vpc
```

### Persistence Options

**RDB (Redis Database Snapshot)**:
```bash
# Configure RDB snapshots
gcloud redis instances update prod-cache \
  --persistence-mode=RDB \
  --rdb-snapshot-period=12h \
  --rdb-snapshot-start-time=2024-01-01T03:00:00Z
```

**AOF (Append-Only File)**:
```bash
# Configure AOF persistence
gcloud redis instances update prod-cache \
  --persistence-mode=AOF \
  --aof-append-fsync=everysec
```

**Persistence Comparison**:
```
RDB:
- Periodic snapshots (every 1, 6, 12, or 24 hours)
- Smaller file size
- Faster restarts
- May lose data between snapshots
- Use Case: Caching with acceptable data loss

AOF:
- Logs every write operation
- Better durability
- Larger file size
- Slower restarts
- Options: always, everysec (recommended), no
- Use Case: Session store, critical cache data
```

### Migration Strategies

**Migration from Self-Hosted Redis**:
```bash
# Step 1: Create Memorystore instance
gcloud redis instances create migration-target \
  --size=10 \
  --region=us-central1 \
  --tier=STANDARD \
  --redis-version=redis_6_x

# Step 2: Export data from source Redis
redis-cli --rdb /tmp/dump.rdb

# Step 3: Import to Memorystore
gcloud redis instances import migration-target \
  gs://my-bucket/dump.rdb \
  --region=us-central1

# Step 4: Update application connection strings
# Old: redis://source-host:6379
# New: redis://MEMORYSTORE_IP:6379
```

**Live Migration with Replication**:
```python
# Use Redis replication for zero-downtime migration
# 1. Configure source Redis as master
# 2. Set up Memorystore instance
# 3. Configure replication from source to Memorystore
# 4. Monitor replication lag
# 5. Switch application to Memorystore
# 6. Break replication

# Python example for monitoring
import redis

source = redis.Redis(host='source-host', port=6379)
target = redis.Redis(host='memorystore-ip', port=6379)

# Check replication lag
info = target.info('replication')
print(f"Replication offset: {info['master_repl_offset']}")
```

### Memorystore Use Cases

**Use Case 1: Session Store**
```python
# Flask session configuration
from flask import Flask, session
from flask_session import Session
import redis

app = Flask(__name__)
app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = redis.from_url('redis://MEMORYSTORE_IP:6379')
Session(app)

@app.route('/login')
def login():
    session['user_id'] = 12345
    session['username'] = 'john_doe'
    return 'Logged in'
```

**Use Case 2: Query Result Cache**
```python
# Cache expensive database queries
import redis
import json
from functools import wraps

redis_client = redis.Redis(host='MEMORYSTORE_IP', port=6379)

def cache_query(ttl=300):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Create cache key
            cache_key = f"{func.__name__}:{json.dumps(args)}:{json.dumps(kwargs)}"

            # Try to get from cache
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)

            # Execute query
            result = func(*args, **kwargs)

            # Store in cache
            redis_client.setex(cache_key, ttl, json.dumps(result))
            return result
        return wrapper
    return decorator

@cache_query(ttl=600)
def get_product(product_id):
    # Expensive database query
    return db.query(f"SELECT * FROM products WHERE id = {product_id}")
```

## Database Selection Criteria

### Detailed Comparison by Use Case

**OLTP Workloads**:
```
Requirements → Service Selection

Single Region, < 64 TB, Traditional SQL:
→ Cloud SQL
  - Cost: Moderate (instance-based)
  - Scale: Vertical (up to 96 vCPU, 624 GB RAM)
  - Use: E-commerce, CRM, ERP

Single Region, High Performance PostgreSQL:
→ AlloyDB
  - Cost: Higher (4x performance)
  - Scale: Vertical (optimized)
  - Use: Analytics + OLTP hybrid

Multiple Regions, Global Transactions:
→ Cloud Spanner
  - Cost: High (node-based pricing)
  - Scale: Horizontal (unlimited)
  - Use: Global gaming, financial trading

Document Model, Real-time Sync:
→ Firestore
  - Cost: Pay per operation
  - Scale: Automatic
  - Use: Mobile apps, collaboration tools
```

**Analytics Workloads**:
```
Data Warehouse, Ad-hoc Queries:
→ BigQuery
  - Cost: Pay per query (storage + compute)
  - Scale: Petabyte+
  - Use: Business intelligence, data lakes

Real-time Analytics on Operational Data:
→ AlloyDB
  - Cost: Higher (columnar engine)
  - Scale: Vertical
  - Use: Real-time dashboards, fraud detection

Time-Series, High Throughput:
→ Bigtable
  - Cost: Node-based + storage
  - Scale: Petabyte+
  - Use: IoT, financial ticks, monitoring
```

**Cost Analysis Example**:
```
Scenario: E-commerce database
- Storage: 100 GB
- Transactions: 1,000 TPS
- Availability: 99.95%

Cloud SQL (db-n1-standard-4, Regional HA):
- Instance: $365/month
- Storage: $17/month (100 GB SSD)
- Backup: $2/month (20 GB)
- Total: ~$384/month

Cloud Spanner (1 node, Regional):
- Node: $677/month
- Storage: $30/month (100 GB)
- Total: ~$707/month

Decision: Cloud SQL (sufficient scale, 45% cheaper)

Scenario: Global application
- Storage: 500 GB
- Transactions: 5,000 TPS
- Availability: 99.999%
- Regions: 3 (US, EU, Asia)

Cloud SQL with Cross-Region Replicas:
- Primary + 2 cross-region replicas: ~$1,200/month
- Complex failover management
- No global transactions

Cloud Spanner (3 nodes, Multi-region nam-eur-asia1):
- Nodes: $2,430/month
- Storage: $150/month
- Total: ~$2,580/month
- Automatic global transactions
- 99.999% SLA

Decision: Cloud Spanner (global consistency required)
```

### Performance Characteristics

**Latency Comparison**:
```
Service          | Read Latency | Write Latency | Throughput
-----------------|--------------|---------------|-------------
Cloud SQL        | 1-5 ms       | 2-10 ms       | 10K QPS
Cloud Spanner    | 1-3 ms       | 3-5 ms        | Unlimited
AlloyDB          | 0.5-2 ms     | 1-3 ms        | 20K+ QPS
Firestore        | 5-10 ms      | 10-20 ms      | 10K writes/s
Bigtable         | 0.1-1 ms     | 0.1-1 ms      | 100K+ QPS
Memorystore      | 0.1-0.5 ms   | 0.1-0.5 ms    | 100K+ QPS
BigQuery         | 1-5 sec      | N/A           | Petabyte scan
```

## Design Scenarios

### Scenario 1: Global E-Commerce Platform

**Requirements**:
- Multi-region presence (US, EU, Asia)
- Strong consistency for inventory
- Fast product catalog access
- Real-time order tracking
- Scalable to millions of users

**Solution Architecture**:
```
Database Layer:
├─ Cloud Spanner (Global)
│  └─ Orders, Payments, Inventory (strong consistency)
│  └─ Config: nam-eur-asia1 (3 regions)
│  └─ 5 nodes per region (15 total)
│
├─ Firestore (Regional instances)
│  └─ Product catalog (denormalized for reads)
│  └─ User profiles and preferences
│  └─ Shopping cart (real-time updates)
│
├─ Memorystore Redis
│  └─ Session management
│  └─ Product recommendations cache
│  └─ Flash sale inventory cache
│
└─ BigQuery
   └─ Analytics and reporting
   └─ Customer behavior analysis
```

**Schema Design**:
```sql
-- Cloud Spanner: Orders (strong consistency)
CREATE TABLE Orders (
    OrderId STRING(36) NOT NULL,  -- UUID
    CustomerId INT64 NOT NULL,
    OrderDate TIMESTAMP NOT NULL,
    Status STRING(50),
    TotalAmount FLOAT64,
    Region STRING(10),
) PRIMARY KEY (OrderId);

CREATE INDEX OrdersByCustomer ON Orders(CustomerId, OrderDate DESC);

CREATE TABLE OrderItems (
    OrderId STRING(36) NOT NULL,
    ItemId INT64 NOT NULL,
    ProductId INT64 NOT NULL,
    Quantity INT64,
    Price FLOAT64,
) PRIMARY KEY (OrderId, ItemId),
  INTERLEAVE IN PARENT Orders ON DELETE CASCADE;

-- Inventory with optimistic locking
CREATE TABLE Inventory (
    ProductId INT64 NOT NULL,
    WarehouseId INT64 NOT NULL,
    Quantity INT64 NOT NULL,
    Version INT64 NOT NULL,  -- For optimistic locking
    LastUpdated TIMESTAMP NOT NULL OPTIONS (
        allow_commit_timestamp=true
    ),
) PRIMARY KEY (ProductId, WarehouseId);
```

```javascript
// Firestore: Product catalog (fast reads)
// Collection: products
{
    "productId": "prod_12345",
    "name": "Premium Widget",
    "description": "High-quality widget",
    "price": 99.99,
    "category": "electronics",
    "images": [
        "gs://bucket/images/prod_12345_1.jpg",
        "gs://bucket/images/prod_12345_2.jpg"
    ],
    "attributes": {
        "color": "blue",
        "size": "medium",
        "weight": "1.5kg"
    },
    "availability": {
        "inStock": true,
        "regions": ["us", "eu", "asia"]
    },
    "ratings": {
        "average": 4.5,
        "count": 1250
    }
}
```

### Scenario 2: IoT Sensor Platform

**Requirements**:
- 100,000 sensors reporting every 10 seconds
- 1 million metrics/second sustained
- Historical data retention: 2 years
- Real-time alerting on anomalies
- Time-series analysis

**Solution Architecture**:
```
Ingestion:
└─ Pub/Sub → Dataflow → Bigtable
   └─ Buffer for spikes
   └─ Aggregation and enrichment
   └─ High-throughput writes

Storage:
├─ Bigtable (Hot data: 90 days)
│  └─ Instance: 10 nodes (US, EU clusters)
│  └─ Table: sensor_metrics
│  └─ Row Key: {sensor_id}#{reverse_timestamp}
│
└─ BigQuery (Cold data: 2 years)
   └─ Daily export from Bigtable
   └─ Partitioned by date
   └─ Used for historical analysis

Real-time:
└─ Memorystore
   └─ Latest sensor values (TTL: 5 minutes)
   └─ Alert thresholds and configurations
```

**Bigtable Schema**:
```python
# Table: sensor_metrics
# Row Key: {sensor_id}#{reverse_timestamp}

column_families = {
    'metadata': {
        'max_versions': 1,  # Latest sensor info
        'columns': ['location', 'type', 'status']
    },
    'metrics': {
        'max_versions': 8640,  # 1 day of 10-sec intervals
        'columns': ['temperature', 'humidity', 'pressure', 'battery']
    },
    'aggregates': {
        'max_versions': 1,  # Pre-computed aggregates
        'columns': ['hourly_avg', 'daily_avg', 'max', 'min']
    }
}

# Example row key calculation
def create_row_key(sensor_id, timestamp):
    max_timestamp = 2**63 - 1
    reverse_ts = max_timestamp - int(timestamp * 1000000)  # microseconds
    return f"{sensor_id}#{reverse_ts:019d}"

# Row key example:
# sensor_001#9223370036854775807
# sensor_001#9223370036844775807  (10 seconds later)

# Query: Get sensor data for time range
def get_sensor_data(sensor_id, start_time, end_time):
    max_timestamp = 2**63 - 1
    start_key = f"{sensor_id}#{max_timestamp - int(end_time * 1000000):019d}"
    end_key = f"{sensor_id}#{max_timestamp - int(start_time * 1000000):019d}"

    row_set = RowSet()
    row_set.add_row_range(RowRange(start_key=start_key, end_key=end_key))
    return table.read_rows(row_set=row_set)
```

### Scenario 3: SaaS Multi-Tenant Application

**Requirements**:
- 10,000 tenants
- Row-level security
- Per-tenant data isolation
- Scalable from 10 to 1M+ users per tenant
- Complex queries across tenant data

**Solution Architecture**:
```
Strategy: Hybrid approach

Small Tenants (< 1000 users):
└─ Cloud SQL (Shared schema)
   └─ Row-level security policies
   └─ Single database, partitioned by tenant_id

Medium Tenants (1000-100K users):
└─ Cloud SQL (Dedicated database per tenant)
   └─ Separate databases on shared instance
   └─ Connection pooling per tenant

Large Tenants (> 100K users):
└─ Cloud Spanner (Dedicated instance)
   └─ Horizontal scalability
   └─ Global distribution if needed
```

**Cloud SQL Schema (Shared)**:
```sql
-- Enable row-level security
CREATE TABLE tenants (
    tenant_id SERIAL PRIMARY KEY,
    tenant_name VARCHAR(255) UNIQUE NOT NULL,
    plan VARCHAR(50),  -- free, basic, premium, enterprise
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL REFERENCES tenants(tenant_id),
    email VARCHAR(255) NOT NULL,
    role VARCHAR(50),
    UNIQUE(tenant_id, email)
);

CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL REFERENCES tenants(tenant_id),
    owner_user_id INTEGER REFERENCES users(user_id),
    title VARCHAR(255),
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Enable RLS
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only access their tenant's data
CREATE POLICY tenant_isolation ON documents
    FOR ALL
    USING (tenant_id = current_setting('app.current_tenant_id')::INTEGER);

-- Policy: Admins can see all data
CREATE POLICY admin_access ON documents
    FOR ALL
    USING (
        current_setting('app.is_admin')::BOOLEAN = true
    );

-- Indexes for performance
CREATE INDEX idx_documents_tenant ON documents(tenant_id, created_at DESC);
CREATE INDEX idx_users_tenant_email ON users(tenant_id, email);
```

**Application Code**:
```python
# Set tenant context for RLS
def set_tenant_context(connection, tenant_id, is_admin=False):
    connection.execute(
        f"SET app.current_tenant_id = {tenant_id};"
        f"SET app.is_admin = {is_admin};"
    )

# Query will automatically filter by tenant
def get_tenant_documents(connection, tenant_id):
    set_tenant_context(connection, tenant_id)
    return connection.execute("SELECT * FROM documents ORDER BY created_at DESC")
```

### Scenario 4: Mobile Gaming Platform

**Requirements**:
- Global player base
- Real-time leaderboards
- Player profiles and progress
- In-game events and notifications
- Offline play support

**Solution Architecture**:
```
Player Data:
├─ Firestore (Global)
│  └─ Player profiles and game state
│  └─ Real-time updates
│  └─ Offline persistence
│  └─ Collection groups for global queries
│
├─ Cloud Spanner (Global)
│  └─ Transactions (in-app purchases)
│  └─ Player inventory
│  └─ Strong consistency required
│
└─ Memorystore
   └─ Real-time leaderboards (Redis Sorted Sets)
   └─ Active sessions
   └─ Rate limiting
```

**Firestore Schema**:
```javascript
// Collection: players
{
    "playerId": "player_12345",
    "username": "ProGamer",
    "email": "player@example.com",
    "profile": {
        "level": 45,
        "xp": 125000,
        "coins": 10000,
        "gems": 500
    },
    "stats": {
        "gamesPlayed": 1250,
        "wins": 750,
        "losses": 500,
        "winRate": 0.60
    },
    "lastSeen": "2024-01-16T10:30:00Z",
    "region": "us-west"
}

// Subcollection: players/{playerId}/inventory
{
    "itemId": "item_sword_legendary",
    "quantity": 1,
    "acquiredAt": "2024-01-15T10:00:00Z",
    "attributes": {
        "damage": 150,
        "rarity": "legendary"
    }
}

// Subcollection: players/{playerId}/achievements
{
    "achievementId": "first_win",
    "unlockedAt": "2024-01-01T10:00:00Z",
    "progress": 1.0
}

// Collection: leaderboards (aggregated daily)
{
    "leaderboardId": "daily_2024_01_16",
    "region": "us-west",
    "topPlayers": [
        {"playerId": "player_123", "username": "ProGamer", "score": 10000},
        {"playerId": "player_456", "username": "ElitePlayer", "score": 9500}
    ],
    "updatedAt": "2024-01-16T23:59:59Z"
}
```

**Redis Leaderboard**:
```python
import redis

redis_client = redis.Redis(host='MEMORYSTORE_IP', port=6379)

# Update player score (sorted set)
def update_leaderboard(player_id, score, leaderboard_name='daily'):
    redis_client.zadd(f"leaderboard:{leaderboard_name}", {player_id: score})

# Get top players
def get_top_players(leaderboard_name='daily', limit=100):
    return redis_client.zrevrange(
        f"leaderboard:{leaderboard_name}",
        0,
        limit - 1,
        withscores=True
    )

# Get player rank
def get_player_rank(player_id, leaderboard_name='daily'):
    rank = redis_client.zrevrank(f"leaderboard:{leaderboard_name}", player_id)
    return rank + 1 if rank is not None else None

# Reset daily leaderboard
def reset_leaderboard(leaderboard_name='daily'):
    redis_client.delete(f"leaderboard:{leaderboard_name}")
```

## Migration Patterns

### Database Migration Service (DMS)

**Supported Migrations**:
```
Source Databases:
- MySQL 5.5, 5.6, 5.7, 8.0
- PostgreSQL 9.4+
- Oracle
- SQL Server

Target Databases:
- Cloud SQL (MySQL, PostgreSQL, SQL Server)
- AlloyDB for PostgreSQL

Migration Types:
- One-time: Bulk data transfer
- Continuous: CDC (Change Data Capture)
```

**MySQL to Cloud SQL Migration**:
```bash
# Step 1: Create connection profiles
gcloud database-migration connection-profiles create mysql source-mysql \
  --region=us-central1 \
  --username=root \
  --password=SOURCE_PASSWORD \
  --host=SOURCE_IP \
  --port=3306

gcloud database-migration connection-profiles create cloudsql target-cloudsql \
  --region=us-central1 \
  --cloudsql-instance=projects/PROJECT/instances/INSTANCE

# Step 2: Create migration job
gcloud database-migration migration-jobs create mysql-migration \
  --region=us-central1 \
  --type=CONTINUOUS \
  --source=source-mysql \
  --destination=target-cloudsql \
  --display-name="MySQL to Cloud SQL Migration"

# Step 3: Start migration
gcloud database-migration migration-jobs start mysql-migration \
  --region=us-central1

# Step 4: Monitor migration
gcloud database-migration migration-jobs describe mysql-migration \
  --region=us-central1

# Step 5: Promote (cutover)
gcloud database-migration migration-jobs promote mysql-migration \
  --region=us-central1
```

**PostgreSQL to AlloyDB Migration**:
```bash
# Similar process with AlloyDB as target
gcloud database-migration connection-profiles create alloydb target-alloydb \
  --region=us-central1 \
  --alloydb-cluster=projects/PROJECT/locations/REGION/clusters/CLUSTER \
  --alloydb-primary=projects/PROJECT/locations/REGION/clusters/CLUSTER/instances/PRIMARY

gcloud database-migration migration-jobs create pg-to-alloydb \
  --region=us-central1 \
  --type=CONTINUOUS \
  --source=source-postgres \
  --destination=target-alloydb
```

### On-Premises to GCP Migration

**Phase 1: Assessment**
```bash
# Assess database size and schema
mysql -e "SELECT
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
GROUP BY table_schema;"

# Check for incompatible features
mysql -e "SELECT ROUTINE_NAME, ROUTINE_TYPE
FROM information_schema.ROUTINES
WHERE ROUTINE_SCHEMA = 'mydb';"

# Analyze query patterns
mysql -e "SELECT * FROM mysql.slow_log ORDER BY query_time DESC LIMIT 10;"
```

**Phase 2: Schema Migration**
```bash
# Export schema only
mysqldump --no-data --routines --triggers mydb > schema.sql

# Review and modify for Cloud SQL compatibility
# Remove DEFINER clauses
sed -i 's/DEFINER=[^ ]* / /g' schema.sql

# Import to Cloud SQL
mysql -h CLOUD_SQL_IP -u root -p mydb < schema.sql
```

**Phase 3: Data Migration**
```bash
# Option 1: DMS (Recommended for large databases)
# See DMS section above

# Option 2: mysqldump for smaller databases (< 100 GB)
mysqldump --single-transaction mydb | gzip > mydb.sql.gz
gsutil cp mydb.sql.gz gs://my-bucket/
gcloud sql import sql INSTANCE_NAME gs://my-bucket/mydb.sql.gz --database=mydb

# Option 3: Export to CSV and import
mysql -e "SELECT * INTO OUTFILE '/tmp/users.csv'
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '\"'
LINES TERMINATED BY '\n'
FROM mydb.users;"

gsutil cp /tmp/users.csv gs://my-bucket/
gcloud sql import csv INSTANCE_NAME gs://my-bucket/users.csv \
  --database=mydb \
  --table=users
```

**Phase 4: Cutover**
```bash
# Pre-cutover checklist
# 1. Verify data consistency
# 2. Test application with Cloud SQL
# 3. Set up monitoring
# 4. Create rollback plan

# Cutover steps
# 1. Enable maintenance mode on application
# 2. Stop writes to source database
# 3. Perform final incremental sync
# 4. Update application connection strings
# 5. Start application with Cloud SQL
# 6. Monitor for errors
# 7. Keep source database for rollback (1-7 days)
```

### Database Modernization Patterns

**Pattern 1: Lift and Shift (Cloud SQL)**
```
Source: On-premises MySQL/PostgreSQL
Target: Cloud SQL
Effort: Low
Timeline: Days to weeks
Use Case: Quick migration, minimal changes

Steps:
1. Create Cloud SQL instance (matching version)
2. Use DMS for data migration
3. Update connection strings
4. Optimize post-migration
```

**Pattern 2: Re-Platform (AlloyDB)**
```
Source: On-premises PostgreSQL or Oracle
Target: AlloyDB for PostgreSQL
Effort: Medium
Timeline: Weeks to months
Use Case: Performance improvement, hybrid OLTP/OLAP

Steps:
1. Schema conversion (if from Oracle)
2. Test compatibility with PostgreSQL
3. Migrate using DMS or pg_dump
4. Optimize for columnar engine
5. Tune for analytics workloads
```

**Pattern 3: Re-Architect (Cloud Spanner)**
```
Source: Sharded MySQL/PostgreSQL
Target: Cloud Spanner
Effort: High
Timeline: Months
Use Case: Global scale, eliminate sharding complexity

Steps:
1. Schema redesign (avoid hotspots)
2. Application refactoring
3. Data migration with custom ETL
4. Gradual traffic migration
5. Remove sharding logic
```

**Pattern 4: Hybrid (SQL + NoSQL)**
```
Source: Monolithic RDBMS
Target: Cloud SQL + Firestore/Bigtable
Effort: High
Timeline: Months
Use Case: Optimize for different access patterns

Example:
- Transactional data → Cloud SQL
- Product catalog → Firestore (fast reads)
- Time-series logs → Bigtable
- Analytics → BigQuery

Steps:
1. Identify data access patterns
2. Design target architecture
3. Build data pipeline
4. Implement dual writes
5. Migrate read traffic gradually
6. Remove old schema
```

## Exam Tips for Professional Cloud Database Engineer

### Service Selection Questions

**Key Decision Factors**:
```
1. Consistency Requirements:
   - Strong consistency → Cloud SQL, Cloud Spanner, AlloyDB
   - Eventual consistency OK → Firestore, Bigtable, BigQuery

2. Scale Requirements:
   - Vertical scale (< 96 vCPU) → Cloud SQL, AlloyDB
   - Horizontal scale (unlimited) → Cloud Spanner, Firestore, Bigtable

3. Geographic Distribution:
   - Single region → Cloud SQL, AlloyDB
   - Multi-region → Cloud Spanner
   - Global → Cloud Spanner, Firestore

4. Data Model:
   - Relational → Cloud SQL, Cloud Spanner, AlloyDB
   - Document → Firestore
   - Wide-column → Bigtable
   - Key-value → Memorystore

5. Latency Requirements:
   - Sub-millisecond → Memorystore, Bigtable
   - Single-digit milliseconds → Cloud SQL, Cloud Spanner, AlloyDB
   - Seconds acceptable → BigQuery

6. Cost Considerations:
   - Budget-conscious → Cloud SQL
   - Pay for performance → AlloyDB, Cloud Spanner
   - Pay per operation → Firestore, BigQuery
```

### Schema Design Questions

**Common Exam Scenarios**:

**Scenario: Prevent Hotspots in Cloud Spanner**
```
Question: Design primary key to distribute writes evenly

Bad Approaches:
❌ Timestamp-based primary key
❌ Sequential IDs
❌ Date-based sharding

Good Approaches:
✅ UUID or hash-based keys
✅ Hash prefix + business key
✅ Bit-reverse sequential IDs
✅ Multiple fields avoiding monotonic increase

Example:
PRIMARY KEY (UUID())
PRIMARY KEY (FARM_FINGERPRINT(user_id), user_id, timestamp)
```

**Scenario: Firestore Query Optimization**
```
Question: Design indexes for complex queries

Rules:
- Single-field indexes: Automatic
- Composite indexes: Manual creation required
- Equality filters before range filters
- OrderBy fields must be indexed

Example Query:
db.collection('orders')
  .where('status', '==', 'pending')
  .where('total', '>', 100)
  .orderBy('total', 'desc')
  .orderBy('created_at', 'desc')

Required Index:
- status (ASCENDING)
- total (DESCENDING)
- created_at (DESCENDING)
```

**Scenario: Bigtable Row Key Design**
```
Question: Design row key for time-series IoT data

Requirements:
- 100,000 sensors
- Query by sensor and time range
- Avoid hotspots
- Efficient scans

Solution:
row_key = f"{sensor_id}#{reverse_timestamp}"

Why:
✓ Sensor ID first (group related data)
✓ Reverse timestamp (recent data first)
✓ Distributes writes across sensors
✓ Efficient range scans per sensor
```

### Migration Questions

**Exam Focus Areas**:

**DMS Capabilities**:
```
Supported:
✅ MySQL → Cloud SQL (MySQL)
✅ PostgreSQL → Cloud SQL (PostgreSQL)
✅ PostgreSQL → AlloyDB
✅ Oracle → PostgreSQL/AlloyDB (preview)
✅ SQL Server → Cloud SQL (SQL Server)

Features:
✅ Continuous migration (CDC)
✅ Automatic schema migration
✅ Minimal downtime
✅ VPC peering support

NOT Supported:
❌ MongoDB → Firestore (use custom migration)
❌ Cassandra → Bigtable (use HBase tools)
❌ DynamoDB → Bigtable (use custom ETL)
```

**Downtime Strategies**:
```
Zero-Downtime Migration:
1. Set up target database
2. Initial bulk load
3. Enable CDC (continuous replication)
4. Monitor replication lag
5. Switch read traffic (test)
6. Switch write traffic (cutover)
7. Decommission source

Acceptable-Downtime Migration:
1. Set up target database
2. Maintenance window announcement
3. Stop application writes
4. Final data sync
5. Update connection strings
6. Restart application
7. Monitor and validate
```

### Performance Tuning Questions

**Cloud SQL Optimization**:
```
Common Issues and Solutions:

Slow Queries:
- Use EXPLAIN ANALYZE
- Add appropriate indexes
- Optimize query structure
- Consider materialized views

Connection Issues:
- Implement connection pooling
- Right-size max_connections
- Use Cloud SQL Proxy
- Consider read replicas

High CPU:
- Identify slow queries (pg_stat_statements)
- Add indexes
- Optimize queries
- Scale up instance tier

Storage I/O:
- Upgrade to SSD (if HDD)
- Increase storage size
- Partition large tables
- Archive old data
```

**Cloud Spanner Optimization**:
```
Hotspot Detection:
- Monitor CPU by split
- Check for sequential keys
- Review query patterns
- Use Key Visualizer tool

Optimization Techniques:
- Redesign primary keys
- Use interleaved tables
- Add covering indexes
- Batch operations
- Use stale reads for analytics

Query Optimization:
- Use query hints
- Avoid full table scans
- Leverage secondary indexes
- Use query parameters
- Monitor query stats
```

**Bigtable Optimization**:
```
Performance Issues:

High Latency:
- Check node CPU utilization
- Add nodes if > 70% CPU
- Optimize row key design
- Use appropriate filters
- Enable compression

Uneven Load:
- Redesign row keys
- Use hash prefixes
- Pre-split tables
- Avoid sequential writes
- Monitor Key Visualizer

Storage Issues:
- Review GC policies
- Adjust TTL settings
- Monitor compaction
- Check replication lag
```

### Cost Optimization Questions

**Exam Scenarios**:

**Scenario: Reduce Cloud SQL Costs**
```
Strategies:
1. Right-size instances (CPU/RAM monitoring)
2. Use committed use discounts
3. Schedule non-prod instance stop/start
4. Optimize storage (delete old backups)
5. Use HDD for non-performance critical
6. Reduce over-provisioned read replicas
7. Export cold data to Cloud Storage
8. Use Cloud SQL Insights to identify waste
```

**Scenario: Optimize Firestore Costs**
```
Cost Factors:
- Document reads/writes
- Storage
- Network egress

Optimization:
1. Cache frequently accessed data (Memorystore)
2. Use collection group queries (fewer reads)
3. Implement pagination (limit results)
4. Denormalize to reduce joins
5. Use batch operations
6. Exempt large fields from indexing
7. Set TTL on temporary data
8. Use Firestore emulator for dev/test
```

**Scenario: Bigtable Cost Optimization**:
```
Cost Components:
- Node hours
- Storage (SSD vs HDD)
- Network egress

Optimization:
1. Use development instances for non-prod
2. Adjust node count based on traffic
3. Use HDD for cold data (70% cheaper storage)
4. Implement GC policies (reduce storage)
5. Use replication only where needed
6. Monitor and remove unused tables
7. Compress data before writing
8. Co-locate compute and Bigtable (reduce egress)
```

## Summary and Quick Reference

### Service Selection Cheat Sheet

```
Use Cloud SQL when:
✓ Traditional RDBMS workload
✓ Single region deployment
✓ < 64 TB database
✓ Existing SQL expertise
✓ Lift-and-shift migration

Use Cloud Spanner when:
✓ Global transactions required
✓ > 64 TB database
✓ 99.999% availability needed
✓ Horizontal scalability
✓ Strong consistency across regions

Use AlloyDB when:
✓ PostgreSQL migration
✓ High performance needed (4x Cloud SQL)
✓ Hybrid OLTP/OLAP workload
✓ Real-time analytics
✓ Single region sufficient

Use Firestore when:
✓ Mobile/web application
✓ Real-time synchronization
✓ Offline support needed
✓ Document data model
✓ Flexible schema

Use Bigtable when:
✓ Time-series data
✓ IoT workload
✓ > 1 TB NoSQL data
✓ High throughput (> 10K writes/sec)
✓ HBase compatibility

Use Memorystore when:
✓ Caching layer
✓ Session storage
✓ Sub-millisecond latency
✓ Redis compatibility
✓ Temporary data

Use BigQuery when:
✓ Data warehouse
✓ Analytics/BI
✓ Ad-hoc queries
✓ Petabyte scale
✓ NOT for OLTP
```

### Key Design Principles

**Cloud SQL**:
- Use private IP for security
- Enable automated backups and PITR
- Configure HA for production
- Use read replicas for read scaling
- Implement connection pooling
- Monitor slow queries
- Partition large tables

**Cloud Spanner**:
- Avoid monotonically increasing keys
- Use interleaved tables for parent-child
- Design indexes carefully
- Monitor CPU per split
- Use batch operations
- Leverage stale reads when appropriate
- Plan for node scaling

**Firestore**:
- Denormalize for read performance
- Use subcollections for scalability
- Create composite indexes for complex queries
- Implement security rules
- Limit document nesting
- Batch writes when possible
- Use offline persistence for mobile

**Bigtable**:
- Row key design is critical
- Avoid sequential keys
- Use column families appropriately
- Set garbage collection policies
- Pre-split tables for large datasets
- Monitor node CPU utilization
- Use replication for HA

**Memorystore**:
- Use Standard tier for production
- Enable persistence for critical data
- Implement connection pooling
- Set appropriate TTLs
- Monitor memory usage
- Use pipelining for bulk operations

### Common Exam Pitfalls

**Avoid These Mistakes**:
```
1. Using BigQuery for OLTP
   → BigQuery is for analytics only

2. Sequential primary keys in Cloud Spanner
   → Causes hotspots, use UUID/hash

3. Deep nesting in Firestore (> 5 levels)
   → Use subcollections instead

4. Timestamp-based row keys in Bigtable
   → Use reverse timestamp or hash prefix

5. Public IP for Cloud SQL in production
   → Use private IP with VPC

6. No backup strategy
   → Always enable automated backups

7. Under-provisioning Cloud Spanner nodes
   → Monitor CPU, keep < 65%

8. Over-indexing Firestore
   → Index only what you query

9. Single node Bigtable in production
   → Use 3+ nodes for HA

10. No monitoring/alerting
    → Set up Cloud Monitoring proactively
```

### Pre-Exam Checklist

**Must Know for Exam**:
```
☑ Service selection criteria (all databases)
☑ HA configuration for each service
☑ Backup and recovery strategies
☑ Schema design best practices
☑ Migration approaches (DMS vs manual)
☑ Performance optimization techniques
☑ Cost optimization strategies
☑ Security best practices (VPC, IAM, encryption)
☑ Monitoring and troubleshooting
☑ Scaling strategies (vertical vs horizontal)
☑ Replication configurations
☑ Row-level security in Cloud SQL
☑ Interleaved tables in Cloud Spanner
☑ Row key design in Bigtable
☑ Index strategies in Firestore
☑ Connection management patterns
```

**Hands-On Practice Recommended**:
```
1. Create Cloud SQL instance with HA and read replicas
2. Design and create Cloud Spanner schema with interleaved tables
3. Build Firestore data model with composite indexes
4. Design Bigtable schema for time-series data
5. Set up Memorystore for caching
6. Perform database migration using DMS
7. Implement connection pooling
8. Configure backups and test restore
9. Monitor database performance
10. Optimize slow queries
```

## Additional Resources

**Official Documentation**:
- [Cloud SQL Documentation](https://cloud.google.com/sql/docs)
- [Cloud Spanner Documentation](https://cloud.google.com/spanner/docs)
- [AlloyDB Documentation](https://cloud.google.com/alloydb/docs)
- [Firestore Documentation](https://cloud.google.com/firestore/docs)
- [Bigtable Documentation](https://cloud.google.com/bigtable/docs)
- [Memorystore Documentation](https://cloud.google.com/memorystore/docs)
- [Database Migration Service](https://cloud.google.com/database-migration)

**Best Practices Guides**:
- [Cloud SQL Best Practices](https://cloud.google.com/sql/docs/mysql/best-practices)
- [Cloud Spanner Schema Design](https://cloud.google.com/spanner/docs/schema-design)
- [Bigtable Schema Design Guide](https://cloud.google.com/bigtable/docs/schema-design)
- [Firestore Data Model](https://firebase.google.com/docs/firestore/data-model)

**Tools and Utilities**:
- [Cloud SQL Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy)
- [Spanner Key Visualizer](https://cloud.google.com/spanner/docs/key-visualizer)
- [Bigtable Key Visualizer](https://cloud.google.com/bigtable/docs/keyvis-overview)
- [Database Migration Service Console](https://console.cloud.google.com/dbmigration)

**Training and Certification**:
- [Professional Cloud Database Engineer Exam Guide](https://cloud.google.com/certification/cloud-database-engineer)
- [Google Cloud Skills Boost - Database Learning Paths](https://www.cloudskillsboost.google/catalog)
- [Coursera - Google Cloud Database Engineering](https://www.coursera.org/learn/gcp-databases)
