# AWS Database Solutions - SAA-C03

## Amazon RDS (Relational Database Service)

### Supported Engines
- **Amazon Aurora**: MySQL and PostgreSQL compatible
- **MySQL**: Open-source relational database
- **PostgreSQL**: Advanced open-source RDBMS
- **MariaDB**: MySQL fork
- **Oracle**: Commercial RDBMS
- **SQL Server**: Microsoft database

### Key Features

**High Availability**:
- **Multi-AZ**: Synchronous replication to standby in different AZ
- **Automatic Failover**: 1-2 minutes, no manual intervention
- **Use Case**: Production databases requiring high availability

**Read Scalability**:
- **Read Replicas**: Asynchronous replication
- **Up to 15 replicas** (Aurora), 5 replicas (other engines)
- **Cross-Region**: Disaster recovery, low latency reads
- **Promotion**: Can be promoted to standalone DB

**Backup and Recovery**:
- **Automated Backups**: Daily full backup, transaction logs every 5 minutes
- **Retention**: 1-35 days (0 to disable)
- **Manual Snapshots**: User-initiated, retained until deleted
- **Point-in-Time Recovery**: Restore to any second within retention period

### Storage
- **General Purpose SSD (gp2/gp3)**: Most workloads
- **Provisioned IOPS (io1)**: I/O intensive workloads
- **Auto Scaling**: Automatically increase storage
- **Encryption**: At rest (KMS), in transit (SSL/TLS)

### RDS Proxy
- **Connection Pooling**: Reduce database connections
- **Failover Time**: 66% reduction
- **Use Cases**: Serverless applications, Lambda functions
- **IAM Authentication**: Enforce IAM-based auth

## Amazon Aurora

### Architecture
- **Storage**: 6 copies across 3 AZs, self-healing
- **Cluster**: 1 master + up to 15 read replicas
- **Endpoints**:
  - Writer Endpoint: Primary instance
  - Reader Endpoint: Load-balanced across read replicas
  - Custom Endpoint: Subset of instances

### Key Features

**Aurora Serverless v2**:
- Auto-scaling compute capacity
- Scales in seconds (not minutes)
- Pay per second
- Use Cases: Infrequent, intermittent, unpredictable workloads

**Aurora Global Database**:
- Cross-region disaster recovery
- Up to 5 secondary regions
- <1 second replication lag
- <1 minute RTO (recovery time objective)

**Aurora Multi-Master**:
- Multiple write nodes
- Continuous availability
- All nodes can read and write

**Backtrack**:
- Rewind database to any point in time
- No backups needed for this feature
- Available for MySQL-compatible only

### Performance
- **5x MySQL Performance**: On same hardware
- **3x PostgreSQL Performance**: On same hardware
- **15 Read Replicas**: Low latency
- **Auto-scaling Storage**: 10GB increments up to 128TB

## Amazon DynamoDB

### Core Concepts
- **Tables**: Collection of items
- **Items**: Collection of attributes (rows)
- **Attributes**: Key-value pairs (columns)
- **Primary Key**: Partition key (required) + optional sort key

### Capacity Modes

**On-Demand**:
- Pay per request
- Auto-scaling, no capacity planning
- Suitable for unpredictable workloads
- More expensive per request

**Provisioned**:
- Specify RCU (Read Capacity Units) and WCU (Write Capacity Units)
- Auto-scaling available
- Reserved capacity for cost savings
- Cheaper for predictable workloads

### Advanced Features

**DynamoDB Accelerator (DAX)**:
- In-memory cache
- Microsecond latency
- 10x performance improvement
- Write-through caching
- No application code changes

**Global Tables**:
- Multi-region, multi-active replication
- <1 second replication lag
- Disaster recovery and low latency

**Streams**:
- Ordered record of item changes
- 24-hour retention
- Use Cases: Analytics, triggers, replication
- Integration with Lambda for event-driven architecture

**Point-in-Time Recovery (PITR)**:
- Restore to any second in last 35 days
- No performance impact
- Incremental backups

**Backup and Restore**:
- On-demand backups: Full backups
- AWS Backup integration
- Cross-region and cross-account

### Performance
- Single-digit millisecond latency
- Auto-scaling for capacity
- Partition key design critical for performance
- Avoid hot partitions

## Amazon ElastiCache

### Redis vs Memcached

**Redis**:
- **Data Structures**: Strings, lists, sets, sorted sets, hashes
- **Persistence**: Snapshots, AOF (Append-Only File)
- **Replication**: Multi-AZ with automatic failover
- **Backup and Restore**: Snapshots
- **Sorted Sets**: Leaderboards, real-time analytics
- **Pub/Sub**: Messaging
- **Use Cases**: Complex data types, persistence, high availability

**Memcached**:
- **Simple Key-Value**: Strings only
- **Multi-threaded**: Utilize multiple cores
- **No Persistence**: Cache only
- **No Replication**: No Multi-AZ
- **Use Cases**: Simple caching, horizontal scaling

### Redis Features

**Cluster Mode**:
- Sharding across multiple nodes
- Up to 500 nodes
- Horizontal scaling

**Redis Auth**:
- Password/token authentication
- SSL/TLS in transit encryption
- IAM authentication (Redis 6+)

**Caching Strategies**:
- **Lazy Loading**: Load data on cache miss
- **Write-Through**: Write to cache and DB simultaneously
- **TTL (Time to Live)**: Expire old data

## Amazon Redshift

### Architecture
- **Data Warehouse**: Columnar storage, OLAP
- **Cluster**: Leader node + compute nodes
- **Node Types**:
  - RA3: Managed storage, independent scaling
  - DC2: Dense compute, SSD storage
  - DS2: Dense storage (previous generation)

### Key Features

**Spectrum**:
- Query S3 data directly
- No need to load data
- Petabyte-scale
- Serverless query execution

**Concurrency Scaling**:
- Handle burst of queries
- Pay per second
- Automatic scaling

**Redshift Serverless**:
- No cluster management
- Auto-scaling compute
- Pay for usage

**Enhanced VPC Routing**:
- Force traffic through VPC
- Use VPC features (NAT, IGW, endpoints)

### Performance
- **Columnar Storage**: 10x compression
- **Massively Parallel Processing (MPP)**
- **Result Caching**: Fast repeated queries
- **AQUA (Advanced Query Accelerator)**: 10x faster queries

## Database Comparison

### Relational vs NoSQL

**Use RDS/Aurora When**:
- ACID transactions required
- Complex joins
- Structured data with schema
- Existing SQL applications

**Use DynamoDB When**:
- Key-value access patterns
- Flexible schema
- Massive scale (millions of requests/second)
- Single-digit millisecond latency required

### OLTP vs OLAP

**OLTP (Online Transaction Processing)**: RDS, Aurora, DynamoDB
- High volume of short transactions
- Real-time applications
- Read and write heavy

**OLAP (Online Analytical Processing)**: Redshift
- Complex queries on large datasets
- Historical data analysis
- Read-heavy, periodic writes

## Other Database Services

### Amazon DocumentDB
- MongoDB-compatible
- JSON document database
- Fully managed, scalable
- Use Case: Content management, catalogs, user profiles

### Amazon Neptune
- Graph database
- SPARQL and Gremlin query languages
- Use Cases: Social networks, fraud detection, knowledge graphs

### Amazon Timestream
- Time-series database
- 1000x faster, 1/10th cost vs relational databases
- Use Cases: IoT, DevOps, analytics

### Amazon Quantum Ledger Database (QLDB)
- Immutable, cryptographically verifiable ledger
- Centralized, fully managed
- Use Cases: Financial transactions, supply chain, regulatory

### Amazon Keyspaces
- Cassandra-compatible
- Wide-column database
- Serverless, auto-scaling
- Use Cases: High-scale IoT, time-series data

## Database Selection Decision Tree

```
Need ACID transactions + Joins?
├─ Yes → RDS or Aurora
│   └─ Need high performance + scalability?
│       ├─ Yes → Aurora
│       └─ No → RDS (MySQL, PostgreSQL, etc.)
│
└─ No → NoSQL or Specialized
    ├─ Simple key-value, massive scale → DynamoDB
    ├─ In-memory caching → ElastiCache (Redis or Memcached)
    ├─ Data warehouse, analytics → Redshift
    ├─ Graph data → Neptune
    ├─ Time-series → Timestream
    ├─ Immutable ledger → QLDB
    └─ Document store (MongoDB) → DocumentDB
```

## Cost Optimization

### RDS/Aurora
1. Use Reserved Instances for predictable workloads (up to 69% savings)
2. Delete unused snapshots
3. Right-size instances based on CloudWatch metrics
4. Use read replicas instead of over-provisioning primary
5. Aurora Serverless v2 for variable workloads

### DynamoDB
1. Use On-Demand for unpredictable workloads
2. Use Provisioned with Auto Scaling for predictable
3. Reserved capacity for consistent workloads
4. Implement DAX instead of over-provisioning throughput
5. Use TTL to automatically delete expired items

### ElastiCache
1. Use Reserved Nodes for steady workloads
2. Right-size cluster based on metrics
3. Use Memcached for simple caching (cheaper)

### Redshift
1. Use Redshift Serverless for variable workloads
2. Reserved Instances for production clusters
3. Pause/resume for dev/test environments
4. Use S3 with Spectrum instead of loading all data

## Exam Tips

### Common Scenarios
- **Multi-AZ failover**: RDS Multi-AZ
- **Read-heavy workload**: Read Replicas or DynamoDB with DAX
- **Global database**: Aurora Global Database or DynamoDB Global Tables
- **In-memory cache**: ElastiCache (Redis for complex, Memcached for simple)
- **Data warehouse**: Redshift
- **Millisecond latency at scale**: DynamoDB
- **GraphQL/MongoDB**: AppSync + DynamoDB or DocumentDB
- **Ledger/audit trail**: QLDB
- **IoT time-series**: Timestream

### Key Points
- Multi-AZ is for high availability, Read Replicas for read scaling
- Aurora is always the most performant RDS option
- DynamoDB: Think partition key design, avoid hot keys
- ElastiCache: Redis for persistence, Memcached for simplicity
- Redshift: OLAP workloads, not OLTP
- Always enable encryption and automated backups in production
