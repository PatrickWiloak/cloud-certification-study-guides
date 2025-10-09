# Database Design and Implementation - GCP Professional Cloud Database Engineer

## Overview

Database architecture, service selection, schema design, and implementation strategies for GCP database services.

## Database Service Selection

### Cloud SQL
**Use Cases**: Traditional RDBMS workloads, application migration, transactional systems
**Engines**: MySQL 5.7/8.0, PostgreSQL 9.6-15, SQL Server 2017/2019/2022
**Limits**: Up to 96 vCPUs, 624 GB RAM, 64 TB storage

**When to Use**:
- Legacy application migration
- Traditional relational database needs
- ACID transactions required
- Existing SQL expertise

### Cloud Spanner
**Use Cases**: Global applications, mission-critical OLTP, horizontal scalability
**Features**: Global consistency, automatic sharding, 99.999% availability
**Limits**: Unlimited scale (horizontal)

**When to Use**:
- Global transaction requirements
- Horizontal scalability needed
- Strong consistency across regions
- 99.999% availability SLA required

### AlloyDB for PostgreSQL
**Use Cases**: High-performance PostgreSQL workloads, analytics + transactional
**Features**: 4x faster than standard PostgreSQL, columnar engine
**Compatibility**: 100% PostgreSQL compatible

**When to Use**:
- PostgreSQL migration with performance needs
- Hybrid transactional/analytical workloads
- Need PostgreSQL ecosystem
- Real-time analytics on operational data

### Firestore
**Use Cases**: Mobile/web apps, real-time sync, document storage
**Features**: NoSQL document database, offline support, real-time updates
**Limits**: 1 MB document size, 20,000 writes/sec per database

**When to Use**:
- Mobile/web application backends
- Real-time collaboration
- Offline-first applications
- Flexible schema requirements

### Cloud Bigtable
**Use Cases**: Time-series, IoT, analytics, large-scale NoSQL
**Features**: Wide-column store, HBase API, low latency, high throughput
**Limits**: Minimum 1 node (300 GB storage), scales to petabytes

**When to Use**:
- Time-series data (IoT, financial)
- High-throughput analytics
- Large-scale NoSQL (> 1 TB)
- HBase migration

### BigQuery
**Use Cases**: Data warehouse, analytics, BI, ML
**Features**: Serverless, petabyte-scale, SQL interface, ML integration
**Limits**: Virtually unlimited

**When to Use**:
- Data warehousing
- Business intelligence
- Ad-hoc analytics
- Not for OLTP

## Database Design Patterns

### Cloud SQL Design
**Schema Design**:
```sql
-- Normalized design for OLTP
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2)
);

CREATE INDEX idx_customer_orders ON orders(customer_id, order_date);
```

**Read Replica Architecture**:
```
Primary Instance (us-central1)
  ├─ Read Replica 1 (us-central1-a)
  ├─ Read Replica 2 (us-central1-b)
  └─ Cross-region Read Replica (europe-west1)
```

### Cloud Spanner Design
**Schema Best Practices**:
```sql
-- Parent-child relationship for co-location
CREATE TABLE Customers (
    CustomerId INT64 NOT NULL,
    Email STRING(MAX),
    Name STRING(MAX),
) PRIMARY KEY (CustomerId);

CREATE TABLE Orders (
    CustomerId INT64 NOT NULL,
    OrderId INT64 NOT NULL,
    OrderDate TIMESTAMP,
    TotalAmount FLOAT64,
) PRIMARY KEY (CustomerId, OrderId),
  INTERLEAVE IN PARENT Customers ON DELETE CASCADE;

-- Secondary index
CREATE INDEX OrdersByDate ON Orders(OrderDate) STORING (TotalAmount);
```

**Avoid Hotspots**:
```sql
-- Bad: Sequential primary key causes hotspots
PRIMARY KEY (timestamp, user_id)

-- Good: Hash or UUID to distribute writes
PRIMARY KEY (FARM_FINGERPRINT(user_id), timestamp)
```

### Firestore Design
**Document Structure**:
```javascript
// Collection: users
{
  "userId": "user123",
  "email": "user@example.com",
  "profile": {
    "name": "John Doe",
    "age": 30
  },
  "preferences": {
    "notifications": true,
    "theme": "dark"
  }
}

// Subcollection: users/{userId}/orders
{
  "orderId": "order456",
  "items": [...],
  "total": 99.99,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Denormalization Strategy**:
```javascript
// Denormalize frequently accessed data
{
  "orderId": "order456",
  "customer": {
    "id": "user123",
    "name": "John Doe",  // Denormalized
    "email": "user@example.com"  // Denormalized
  },
  "items": [...]
}
```

### Bigtable Schema Design
**Row Key Design** (Critical):
```
// Bad: Sequential timestamps cause hotspots
row_key = timestamp + sensor_id

// Good: Reverse domain + timestamp
row_key = sensor_id + reverse_timestamp

// Good: Hashed prefix
row_key = hash(sensor_id)[:4] + sensor_id + timestamp
```

**Column Family Structure**:
```python
# Define column families
column_families = {
    'metadata': {'max_versions': 1},
    'metrics': {'max_versions': 100},  # Keep history
    'events': {'max_age_days': 30}     # Auto-expire
}
```

## Implementation

### Cloud SQL Setup
```bash
# Create instance with HA
gcloud sql instances create production-db \
  --database-version=POSTGRES_14 \
  --tier=db-n1-standard-4 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --backup-start-time=03:00 \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=04 \
  --enable-bin-log \
  --retained-backups-count=30

# Create read replica
gcloud sql instances create production-db-replica \
  --master-instance-name=production-db \
  --tier=db-n1-standard-2 \
  --zone=us-central1-b

# Configure private IP
gcloud sql instances patch production-db \
  --network=projects/PROJECT/global/networks/vpc \
  --no-assign-ip
```

### Cloud Spanner Setup
```bash
# Create instance
gcloud spanner instances create production-spanner \
  --config=regional-us-central1 \
  --nodes=3 \
  --description="Production Spanner Instance"

# Create database
gcloud spanner databases create orders-db \
  --instance=production-spanner

# Apply DDL
gcloud spanner databases ddl update orders-db \
  --instance=production-spanner \
  --ddl-file=schema.sql
```

### Firestore Setup
```bash
# Create database (default or named)
gcloud firestore databases create \
  --location=us-east1 \
  --type=firestore-native

# Create composite index
gcloud firestore indexes composite create \
  --collection-group=orders \
  --field-config=field-path=customerId,order=ASCENDING \
  --field-config=field-path=orderDate,order=DESCENDING
```

### Bigtable Setup
```bash
# Create instance
gcloud bigtable instances create production-bigtable \
  --cluster=cluster-1 \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=3 \
  --display-name="Production Bigtable"

# Add replication
gcloud bigtable clusters create cluster-2 \
  --instance=production-bigtable \
  --zone=us-east1-b \
  --num-nodes=3
```

## Migration Strategies

### Database Migration Service
```bash
# Create migration job for MySQL
gcloud database-migration migration-jobs create mysql-migration \
  --region=us-central1 \
  --type=CONTINUOUS \
  --source=SOURCE_CONNECTION_PROFILE \
  --destination=DESTINATION_CONNECTION_PROFILE \
  --display-name="MySQL to Cloud SQL Migration"

# Promote migration (cutover)
gcloud database-migration migration-jobs promote mysql-migration \
  --region=us-central1
```

### Manual Migration Steps
1. **Assessment**: Inventory, compatibility check
2. **Schema Migration**: DDL conversion, testing
3. **Data Migration**: Initial load, CDC setup
4. **Application Migration**: Connection string updates, testing
5. **Cutover**: Final sync, switch, validation

## Best Practices

### Cloud SQL
1. Use private IP for security
2. Enable automated backups
3. Configure maintenance windows
4. Use read replicas for read scaling
5. Enable SSL/TLS for connections
6. Monitor slow queries
7. Right-size instance tier
8. Use connection pooling

### Cloud Spanner
1. Design schema to avoid hotspots
2. Use interleaved tables for parent-child
3. Monitor split patterns
4. Use appropriate primary key design
5. Leverage secondary indexes wisely
6. Batch writes when possible
7. Monitor CPU utilization
8. Plan for growth (node scaling)

### Firestore
1. Design for queries (create indexes)
2. Avoid deep nesting (max 100 levels)
3. Use subcollections for scalability
4. Implement security rules
5. Denormalize for read performance
6. Monitor read/write costs
7. Use offline persistence
8. Batch writes (500 operations max)

### Bigtable
1. Design row keys carefully (avoid hotspots)
2. Use column families appropriately
3. Set garbage collection policies
4. Pre-split tables for large datasets
5. Monitor node CPU utilization
6. Use replication for HA
7. Batch writes for efficiency
8. Implement proper error handling

## Common Scenarios

**Scenario**: Global e-commerce platform
**Solution**: Cloud Spanner for orders, Cloud SQL read replicas for regional data, Memorystore for caching

**Scenario**: IoT sensor data platform
**Solution**: Bigtable for time-series data, Dataflow for ingestion, BigQuery for analytics

**Scenario**: Mobile app with offline support
**Solution**: Firestore for real-time sync, Cloud Functions for backend logic, Cloud Storage for media

**Scenario**: Legacy Oracle migration
**Solution**: AlloyDB for PostgreSQL if re-platforming, Cloud SQL if lift-and-shift, Database Migration Service

## Study Tips

1. Understand service selection criteria
2. Practice schema design for each service
3. Know migration strategies
4. Implement HA configurations
5. Performance tuning techniques
6. Cost optimization approaches
7. Hands-on with each database service

## Additional Resources

- [Cloud SQL Documentation](https://cloud.google.com/sql/docs)
- [Cloud Spanner Documentation](https://cloud.google.com/spanner/docs)
- [Firestore Documentation](https://cloud.google.com/firestore/docs)
- [Bigtable Documentation](https://cloud.google.com/bigtable/docs)
- [Database Migration Service](https://cloud.google.com/database-migration)
