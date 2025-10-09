# Databases

## Overview

Enterprise database architecture on AWS requires deep understanding of relational, NoSQL, in-memory, graph, time-series, and ledger databases. At the professional level, you must design data architectures that balance consistency, availability, partition tolerance (CAP theorem), performance, scalability, and cost while meeting compliance and disaster recovery requirements.

Professional architects must navigate complex trade-offs between database engines, optimize for specific access patterns, implement multi-region strategies, design for high availability and disaster recovery, and architect data migration and modernization strategies.

## Key Concepts

### Relational Databases (Amazon RDS)

**Supported Engines**
- **Amazon Aurora**: MySQL and PostgreSQL-compatible, cloud-native, 5x MySQL / 3x PostgreSQL performance
- **MySQL**: Open-source, widely adopted, version 5.7 and 8.0
- **PostgreSQL**: Advanced features, JSON support, full-text search, version 12-16
- **MariaDB**: MySQL fork, storage engines, advanced features
- **Oracle**: Enterprise features, BYOL or License Included
- **SQL Server**: Microsoft SQL Server, Standard/Enterprise/Web editions

**Deployment Options**
- **Single-AZ**: Development, non-critical workloads
- **Multi-AZ**: High availability with synchronous replication to standby (failover in 1-2 minutes)
- **Multi-AZ Cluster** (MySQL/PostgreSQL): 2 readable standbys, sub-second failover, higher throughput
- **Read Replicas**: Asynchronous replication for read scaling (up to 15 for Aurora, 5 for others)
- **Cross-Region Read Replicas**: Disaster recovery, local read performance

**Storage**
- **General Purpose SSD (gp3/gp2)**: 20 GiB to 64 TiB, 3,000-16,000 IOPS
- **Provisioned IOPS (io1)**: 100 GiB to 64 TiB, up to 80,000 IOPS, <10ms latency
- **Magnetic**: Legacy, not recommended
- **Aurora**: Shared storage, auto-scales up to 128 TiB, 6 copies across 3 AZs

**Backup and Recovery**
- **Automated Backups**: Daily snapshots + transaction logs, retention 1-35 days, point-in-time recovery
- **Manual Snapshots**: User-initiated, retained indefinitely
- **Backup Window**: 30-minute window (configurable)
- **Cross-Region Snapshot Copy**: Disaster recovery, compliance
- **Restore**: Creates new DB instance (cannot restore to existing)

**Security**
- **Encryption at Rest**: AES-256 with KMS (cannot enable after creation for non-Aurora)
- **Encryption in Transit**: SSL/TLS connections
- **IAM Database Authentication**: Token-based authentication (MySQL, PostgreSQL, Aurora)
- **Secrets Manager Integration**: Automatic password rotation
- **VPC**: Launch in private subnets
- **Security Groups**: Network access control
- **Enhanced Monitoring**: OS-level metrics with CloudWatch Logs

**Performance Optimization**
- **Parameter Groups**: Engine configuration
- **Option Groups**: Additional features (Oracle TDE, SQL Server Transparent Data Encryption)
- **Read Replicas**: Offload read traffic
- **Connection Pooling**: RDS Proxy for connection management
- **Performance Insights**: Database performance monitoring and tuning
- **Query Performance**: Slow query logs, explain plans

### Amazon Aurora

**Architecture**
- **Storage**: Shared, distributed, auto-scaling (10 GB to 128 TB)
- **6-Way Replication**: 2 copies in each of 3 AZs
- **Self-Healing**: Continuous backup to S3, automatic repair of corrupted blocks
- **Read Scaling**: Up to 15 read replicas
- **Fast Failover**: Typically <30 seconds

**Aurora Serverless**
- **v1**: Auto-scaling ACUs (Aurora Capacity Units), pause/resume, ideal for intermittent workloads
- **v2**: Instant scaling, granular scaling, always available, serverless + provisioned endpoints

**Aurora Global Database**
- **Primary Region**: Read-write
- **Secondary Regions**: Read-only (up to 5 secondary regions)
- **Replication Lag**: <1 second
- **RPO**: 1 second, RTO: <1 minute
- **Use Cases**: Disaster recovery, global read scaling, low-latency reads

**Aurora Multi-Master**
- **Write Scaling**: All nodes are read-write
- **High Availability**: No single point of failure
- **Use Cases**: Sharded applications, continuous availability

**Aurora Features**
- **Backtrack**: Rewind database to point in time (MySQL only, up to 72 hours)
- **Cloning**: Fast, copy-on-write cloning from snapshot
- **Database Activity Streams**: Real-time stream of database activity to Kinesis
- **Machine Learning**: Integration with SageMaker, Comprehend
- **RDS Proxy**: Connection pooling and management

### NoSQL Databases

**Amazon DynamoDB**
- **Key-Value and Document**: Flexible schema, millisecond latency
- **Capacity Modes**:
  - **On-Demand**: Pay per request, automatic scaling, unpredictable workloads
  - **Provisioned**: Specify RCU/WCU, lower cost for predictable traffic, Auto Scaling available
- **Consistency**:
  - **Eventually Consistent Reads**: Default, 50% cheaper than strongly consistent
  - **Strongly Consistent Reads**: Read immediately after write
  - **Transactional Reads/Writes**: ACID across multiple items
- **Indexes**:
  - **Local Secondary Index (LSI)**: Same partition key, different sort key, strongly consistent
  - **Global Secondary Index (GSI)**: Different partition and sort key, eventually consistent
- **Streams**: Capture item-level changes, integrate with Lambda for triggers
- **Global Tables**: Multi-region, multi-active, <1 second replication
- **DAX (DynamoDB Accelerator)**: In-memory cache, microsecond latency, write-through cache
- **Backup**:
  - **On-Demand**: Manual, retained until deleted
  - **Point-in-Time Recovery (PITR)**: Continuous backups, restore to any second in last 35 days
- **Encryption**: At rest with KMS, in transit with TLS
- **Use Cases**: User profiles, shopping carts, gaming leaderboards, IoT, mobile backends

**Amazon DocumentDB (MongoDB-compatible)**
- **Document Database**: JSON documents, MongoDB API compatibility
- **Storage**: Auto-scaling up to 64 TB
- **Replication**: Up to 15 read replicas across 3 AZs
- **Backup**: Automated and manual snapshots
- **Use Cases**: Content management, catalogs, user profiles (MongoDB workloads)

**Amazon Keyspaces (Apache Cassandra-compatible)**
- **Wide-Column Store**: Cassandra Query Language (CQL) compatibility
- **Serverless**: On-demand and provisioned capacity
- **Multi-Region**: Cross-region replication
- **Use Cases**: IoT, time-series, high-traffic applications (Cassandra workloads)

### In-Memory Databases

**Amazon ElastiCache**
- **Redis**: Pub/sub, sorted sets, geospatial, complex data structures, persistence, replication
  - **Cluster Mode Disabled**: Single shard, up to 5 read replicas
  - **Cluster Mode Enabled**: Multiple shards (partitions), horizontal scaling, up to 500 nodes
  - **Global Datastore**: Cross-region replication, <1 second lag
  - **Backup and Restore**: RDB snapshots
  - **Auth Token**: Password protection
  - **Encryption**: At rest and in transit
- **Memcached**: Simple key-value, multi-threaded, horizontal scaling (add/remove nodes)
  - No persistence, no replication, no backup
  - Multi-threaded (better for large nodes)
- **Use Cases**: Session store, database cache, pub/sub, leaderboards, rate limiting, distributed locks

**Amazon MemoryDB for Redis**
- **Durable**: Redis with Multi-AZ durability (transaction log)
- **Primary Database**: Can replace Redis + RDS combination
- **Microsecond Latency**: In-memory performance
- **Redis Compatibility**: 6.2 and later
- **Use Cases**: Caching + primary database, gaming leaderboards, real-time analytics

### Specialized Databases

**Amazon Neptune (Graph)**
- **Property Graph and RDF**: Gremlin and SPARQL query languages
- **Use Cases**: Social networks, recommendation engines, fraud detection, knowledge graphs
- **High Availability**: Multi-AZ, up to 15 read replicas
- **Global Database**: Multi-region replication

**Amazon Timestream (Time-Series)**
- **Purpose-Built**: IoT, operational applications, analytics
- **Storage Tiers**: Memory store (recent data, fast queries), magnetic store (historical data, cost-effective)
- **Adaptive Query Processing**: Automatic optimization
- **Serverless**: Auto-scaling
- **Use Cases**: IoT telemetry, application metrics, industrial equipment monitoring

**Amazon QLDB (Quantum Ledger Database)**
- **Immutable Journal**: Cryptographically verifiable transaction log
- **Ledger**: Append-only, complete history of changes
- **PartiQL**: SQL-compatible query language
- **Use Cases**: System of record, supply chain, financial transactions, regulatory compliance

**Amazon OpenSearch Service**
- **Search and Analytics**: Full-text search, log analytics, application monitoring
- **Based on Elasticsearch and OpenSearch**
- **Indexing**: Near real-time indexing
- **Kibana/OpenSearch Dashboards**: Visualization
- **Cluster**: Master nodes, data nodes, UltraWarm (S3-backed) for cost-effective storage
- **Use Cases**: Log analytics, full-text search, clickstream analytics, application monitoring

### Database Migration

**AWS Database Migration Service (DMS)**
- **Homogeneous Migrations**: Oracle to Oracle, MySQL to MySQL
- **Heterogeneous Migrations**: Oracle to Aurora, SQL Server to MySQL
- **Continuous Replication**: CDC (Change Data Capture) for minimal downtime
- **Schema Conversion Tool (SCT)**: Convert schemas (Oracle to PostgreSQL, SQL Server to MySQL)
- **Replication Instance**: EC2 instance running DMS
- **Use Cases**: Migration to cloud, database consolidation, continuous replication for DR

**AWS DataSync**
- **File Transfer**: On-premises to S3, EFS, FSx
- **Not for databases**: Use DMS for database migration

## AWS Services Reference

### Core Services

**Amazon RDS**
- Managed relational database service
- MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- Multi-AZ for HA, Read Replicas for scaling
- Automated backups and patching

**Amazon Aurora**
- Cloud-native relational database
- MySQL and PostgreSQL compatibility
- 5x/3x performance improvement
- Global Database for multi-region

**Amazon DynamoDB**
- Serverless NoSQL key-value and document database
- Single-digit millisecond latency
- Global Tables for multi-region
- DAX for microsecond latency

**Amazon ElastiCache**
- In-memory caching service
- Redis and Memcached engines
- Sub-millisecond latency
- Cluster mode for scaling

### Supporting Services

**Amazon RDS Proxy**
- Connection pooling and management
- Reduce database connections and failover time
- IAM authentication support

**AWS Database Migration Service**
- Migrate databases to AWS
- Homogeneous and heterogeneous migrations
- Continuous replication

**AWS Schema Conversion Tool**
- Convert database schemas
- Assessment report for compatibility

**Amazon DocumentDB**
- MongoDB-compatible document database
- Fully managed, scalable

**Amazon Neptune**
- Graph database for highly connected datasets
- Property graph and RDF support

**Amazon Timestream**
- Time-series database for IoT and operational data
- Serverless, auto-scaling

**Amazon QLDB**
- Ledger database with immutable, verifiable transaction log
- Cryptographic verification

**Amazon OpenSearch Service**
- Search and analytics engine
- Log analytics, full-text search

## Architecture Patterns

### Pattern 1: Multi-Tier Application with RDS Multi-AZ

**Use Case**
- E-commerce application requiring high availability
- Relational data with ACID transactions
- Read-heavy workload with occasional writes

**Implementation Approach**
1. **Database**: RDS MySQL Multi-AZ in private subnets
2. **Read Replicas**: 3 read replicas in different AZs for read scaling
3. **Application**: Route writes to primary, reads to read replicas (reader endpoint)
4. **Connection Pooling**: RDS Proxy to manage connections, reduce database load
5. **Caching**: ElastiCache Redis for frequently accessed data (product catalog, user sessions)
6. **Backup**: Automated backups (7-day retention), manual snapshots before major releases
7. **Monitoring**: Performance Insights, CloudWatch alarms for CPU, connections, replication lag
8. **Security**: Encryption at rest (KMS), in transit (SSL), IAM database authentication

**Pros/Cons**
- Pros: High availability (automatic failover), read scaling, managed service, ACID compliance
- Cons: Vertical scaling limits, Multi-AZ doubles cost, read replica lag (asynchronous)

### Pattern 2: Global Application with Aurora Global Database

**Use Case**
- SaaS application serving users globally
- Low-latency reads in multiple regions
- Disaster recovery with <1 minute RTO
- Business continuity for regional failures

**Implementation Approach**
1. **Primary Region** (us-east-1): Aurora MySQL cluster (1 writer, 2 readers)
2. **Secondary Regions** (eu-west-1, ap-southeast-1): Aurora read replicas
3. **Replication**: <1 second lag between regions
4. **Routing**: Route 53 latency-based routing to nearest region
5. **Failover**: Promote secondary region to primary if regional failure
6. **Local Reads**: Applications read from local region for low latency
7. **Writes**: All writes to primary region (or use multi-master for write scaling)
8. **Monitoring**: CloudWatch metrics for replication lag, automated failover testing

**Pros/Cons**
- Pros: Low-latency global reads, fast DR (RPO 1s, RTO <1 min), managed replication
- Cons: Write bottleneck (single primary), cross-region data transfer costs, complexity

### Pattern 3: High-Throughput with DynamoDB and DAX

**Use Case**
- Gaming leaderboard with millions of users
- Single-digit millisecond read latency required
- Highly variable traffic (peaks during events)
- Global user base

**Implementation Approach**
1. **DynamoDB Table**: On-demand capacity mode (auto-scaling)
   - Partition key: UserId
   - Sort key: GameId
   - GSI: GameId (partition), Score (sort) for leaderboards
2. **DAX Cluster**: 3-node cluster for microsecond read latency
3. **Global Tables**: Multi-region active-active for low-latency worldwide
4. **Streams**: DynamoDB Streams trigger Lambda for real-time notifications
5. **Caching Strategy**: Cache read-heavy queries in DAX (top 100 leaderboard)
6. **Backup**: Point-in-time recovery enabled (35-day retention)
7. **Security**: Encryption at rest, VPC endpoints for private access, IAM for access control

**Pros/Cons**
- Pros: Unlimited scaling, millisecond latency (DAX: microsecond), global replication, serverless
- Cons: No ACID across partitions (except transactions), query limitations (no joins), cost at high scale

### Pattern 4: Analytics with Aurora + Redshift

**Use Case**
- Retail company with transactional database (orders, customers, products)
- Business intelligence and reporting requirements
- Separate OLTP and OLAP workloads

**Implementation Approach**
1. **OLTP**: Aurora PostgreSQL for transactional workload
   - Multi-AZ for HA
   - Read replicas for reporting queries (isolate from production)
2. **ETL**: AWS Glue or Lambda to extract data from Aurora
3. **OLAP**: Amazon Redshift for data warehouse
   - RA3 nodes with managed storage
   - Redshift Spectrum for querying S3 data lake
4. **Data Lake**: S3 for raw and processed data
5. **BI Tools**: QuickSight for dashboards and ad-hoc analysis
6. **Schedule**: Nightly ETL (or near real-time with Kinesis)
7. **Optimization**: Aurora read replica for ETL queries, Redshift distribution/sort keys

**Pros/Cons**
- Pros: Optimized for each workload (OLTP and OLAP), Aurora fast queries, Redshift complex analytics
- Cons: ETL complexity, data freshness lag, duplicate storage costs

### Pattern 5: Caching Strategy with ElastiCache

**Use Case**
- Social media application with high read traffic
- Database queries are expensive (complex joins)
- User sessions need to be shared across instances

**Implementation Approach**
1. **Database**: RDS PostgreSQL Multi-AZ
2. **ElastiCache Redis Cluster**: Cluster mode enabled for horizontal scaling
   - 3 shards, 2 replicas per shard (total 9 nodes)
   - Multi-AZ with automatic failover
3. **Caching Pattern**:
   - **Lazy Loading**: Check cache, if miss read from DB and populate cache
   - **Write-Through**: Write to DB and cache simultaneously
   - **TTL**: Expire keys after 1 hour to prevent stale data
4. **Session Store**: Store user sessions in Redis (fast access, shared across instances)
5. **Pub/Sub**: Use Redis Pub/Sub for real-time features (notifications, chat)
6. **Eviction Policy**: allkeys-lru (evict least recently used keys when memory full)
7. **Monitoring**: Cache hit ratio (target >80%), evictions, replication lag

**Pros/Cons**
- Pros: Sub-millisecond latency, reduce database load (80-90% offloaded), session sharing
- Cons: Cache invalidation complexity, potential stale data, additional cost

### Pattern 6: Event-Driven with DynamoDB Streams

**Use Case**
- E-commerce order processing with multiple downstream systems
- Real-time inventory updates
- Order notifications and analytics

**Implementation Approach**
1. **DynamoDB Table**: Orders table
   - Partition key: OrderId
   - Attributes: CustomerId, Items, Status, Timestamp
2. **DynamoDB Streams**: Capture item-level changes (INSERT, MODIFY, REMOVE)
3. **Lambda Functions**: Triggered by Streams
   - **Inventory Lambda**: Update inventory table when order placed
   - **Notification Lambda**: Send email/SMS to customer
   - **Analytics Lambda**: Stream to Kinesis Firehose → S3 → Athena
4. **Error Handling**: Lambda DLQ for failed processing, retry logic
5. **Idempotency**: Use OrderId to prevent duplicate processing
6. **Monitoring**: Lambda errors, DynamoDB stream age, throttled requests

**Pros/Cons**
- Pros: Real-time processing, decoupled architecture, serverless, exactly-once delivery
- Cons: Lambda concurrency limits, complexity in error handling, Streams 24-hour retention

## Best Practices

### Enterprise-Level Recommendations

**Database Selection**
- **OLTP (Transactional)**: RDS, Aurora for relational, DynamoDB for key-value/document
- **OLAP (Analytics)**: Redshift for data warehouse, Athena for ad-hoc queries on S3
- **Caching**: ElastiCache Redis/Memcached, DAX for DynamoDB
- **Graph**: Neptune for highly connected data
- **Time-Series**: Timestream for IoT, operational metrics
- **Search**: OpenSearch for full-text search, log analytics

**High Availability**
- Multi-AZ for production databases (RDS, Aurora, ElastiCache)
- Automated backups with 7-35 day retention
- Test failover procedures quarterly
- Read replicas in different AZs
- Global Tables (DynamoDB) or Global Database (Aurora) for multi-region

**Disaster Recovery**
- Define RPO and RTO requirements
- Cross-region snapshots (RDS, Aurora)
- Global Tables (DynamoDB) for active-active DR (RPO near-zero, RTO <1 min)
- Aurora Global Database (RPO 1s, RTO <1 min)
- Test DR procedures regularly (monthly or quarterly)

**Performance Optimization**
- Right-size instances based on CloudWatch metrics
- Use read replicas to offload read traffic
- Caching with ElastiCache or DAX
- Connection pooling with RDS Proxy
- Performance Insights for query tuning
- Indexes for frequently queried columns (cost: storage, write performance)

### Security Considerations

**Encryption**
- At rest: Enable at creation (cannot add later for RDS except Aurora)
- In transit: Enforce SSL/TLS connections
- KMS customer-managed keys for sensitive data
- Encrypt snapshots (automatic if source encrypted)

**Network Security**
- Private subnets for databases (no internet gateway)
- Security groups allow only application tier
- VPC endpoints for AWS service access (DynamoDB, S3)
- No public access (except for specific use cases with careful IP whitelisting)

**Access Control**
- IAM database authentication (RDS MySQL, PostgreSQL, Aurora)
- IAM policies for DynamoDB access (fine-grained with conditions)
- Secrets Manager for password management and rotation
- Principle of least privilege

**Auditing**
- CloudTrail for API calls (who created/modified database)
- Database audit logs (RDS option groups for Oracle, SQL Server)
- Performance Insights for query visibility
- VPC Flow Logs for network traffic

### Cost Optimization

**Right-Sizing**
- Use CloudWatch metrics to identify underutilized databases
- Downsize instances during non-peak hours (dev/test)
- Aurora Serverless for intermittent workloads

**Reserved Instances**
- 1 or 3-year commitment for RDS/ElastiCache (up to 60% savings)
- Reserved Capacity for DynamoDB (predictable workloads)

**Storage Optimization**
- RDS: gp3 instead of gp2 (20% cheaper, same performance)
- DynamoDB: On-demand vs. Provisioned (predictable traffic → provisioned)
- Aurora: Shared storage (pay for actual usage, not allocated)
- Redshift: RA3 nodes with managed storage (pay for compute separately)

**Data Lifecycle**
- Archive old data to S3 (cheaper than database storage)
- DynamoDB TTL for automatic expiration
- ElastiCache eviction policies
- RDS snapshot lifecycle (delete old snapshots)

### Performance Tuning

**Query Optimization**
- Indexes on frequently queried columns
- EXPLAIN plans for slow queries
- Avoid SELECT *, fetch only needed columns
- Pagination for large result sets
- Batch operations (DynamoDB BatchGetItem, BatchWriteItem)

**Connection Management**
- Connection pooling in application
- RDS Proxy for managing connections (Lambda, serverless)
- Limit max connections per client
- Monitor connections in CloudWatch

**Caching**
- Cache frequently accessed data (80/20 rule)
- Cache invalidation strategy (TTL, write-through)
- Cache hit ratio >80%
- ElastiCache for database queries, DAX for DynamoDB

**Read Replicas**
- Offload read traffic from primary
- Analytical queries on read replica
- Monitor replication lag (should be <1 second)
- Use reader endpoint for automatic load balancing (Aurora)

## Common Scenarios

### Scenario 1: Migration from Oracle to Aurora PostgreSQL

**Context**: Enterprise migrating from Oracle RAC (500 GB) to AWS, reduce licensing costs

**Migration Strategy**
1. **Assessment**: AWS Schema Conversion Tool (SCT) to analyze Oracle schema
   - Identify incompatibilities (PL/SQL → PL/pgSQL, Oracle-specific features)
   - Estimate effort and complexity
2. **Schema Conversion**: SCT to convert schema, stored procedures, functions
   - Manual conversion for complex PL/SQL
   - Testing in non-production Aurora cluster
3. **Data Migration**: AWS DMS for data transfer
   - Full load: Initial data copy
   - CDC (Change Data Capture): Continuous replication for cutover
4. **Cutover**: Stop writes to Oracle, final sync, point application to Aurora
5. **Post-Migration**: Performance tuning, query optimization, monitoring

**Key Considerations**
- Oracle licensing costs: $47,500/processor/year (Enterprise Edition)
- Aurora cost: ~$500/month (db.r6g.2xlarge, 500 GB storage)
- Savings: $40K+/year
- Downtime: <1 hour with DMS CDC
- Testing: Load testing, compatibility testing before cutover

### Scenario 2: IoT Application with 1M Devices

**Context**: IoT platform ingesting telemetry from 1M devices, 1 event/second/device

**Architecture**
1. **Ingestion**: AWS IoT Core (MQTT) → Kinesis Data Streams
2. **Processing**: Lambda for real-time processing and aggregation
3. **Storage**:
   - **Hot Data** (last 7 days): DynamoDB (partition key: DeviceId, sort key: Timestamp)
   - **Warm Data** (8-90 days): Timestream (automatic tiering to magnetic store)
   - **Cold Data** (>90 days): S3 with lifecycle to Glacier
4. **Querying**:
   - Real-time: DynamoDB for device status, last reading
   - Historical: Timestream for analytics, Athena for S3 queries
5. **Alerting**: DynamoDB Streams → Lambda → SNS for threshold violations
6. **Dashboard**: QuickSight connected to Timestream

**Key Considerations**
- Throughput: 1M events/second = 86B events/day
- DynamoDB: On-demand capacity (auto-scaling), TTL for automatic deletion after 7 days
- Timestream: Memory store for recent data (fast queries), magnetic store for cost-effective historical
- Cost: DynamoDB $0.25/write million, Timestream $0.036/GB-hr memory, $0.03/GB-month magnetic
- Partitioning: DynamoDB partition key (DeviceId) ensures even distribution

### Scenario 3: Multi-Tenant SaaS with Data Isolation

**Context**: B2B SaaS with 10,000 customers, requiring strict data isolation

**Database Strategy Options**

**Option 1: Database per Tenant (Highest Isolation)**
- **Approach**: Separate RDS instance per customer
- **Pros**: Complete isolation, customization per tenant, backup/restore per tenant
- **Cons**: High cost (10,000 x $100/month = $1M/month), operational complexity
- **Use Case**: Enterprise customers with compliance requirements (HIPAA, PCI)

**Option 2: Schema per Tenant (Medium Isolation)**
- **Approach**: Single RDS instance, separate schema per tenant
- **Pros**: Lower cost, easier backups, resource sharing
- **Cons**: Limited scalability (schema limit), schema management complexity
- **Use Case**: Medium-sized customers, moderate isolation requirements

**Option 3: Shared Schema with Tenant ID (Lowest Isolation)**
- **Approach**: Single schema, TenantId column in all tables
- **Pros**: Lowest cost, simplest architecture, easy scaling
- **Cons**: Risk of cross-tenant data leakage (application bugs), complex queries
- **Use Case**: Small customers, cost-sensitive

**Recommended Approach (Hybrid)**
1. **Tier 1** (Enterprise, 10 customers): Dedicated Aurora cluster per tenant
2. **Tier 2** (Business, 100 customers): Aurora cluster with 10 customers per cluster (schema per tenant)
3. **Tier 3** (Standard, 9,890 customers): Shared Aurora cluster with TenantId in tables

**Implementation**
- **DynamoDB**: Partition key includes TenantId for automatic sharding
- **Application**: Tenant context in JWT, validated at API Gateway
- **Security**: IAM policies with ABAC (tenant tag), RLS (Row-Level Security) in PostgreSQL
- **Monitoring**: CloudWatch metrics per tenant (custom dimensions)

### Scenario 4: Financial Services with ACID and Audit Trail

**Context**: Trading platform requiring ACID transactions, complete audit trail, regulatory compliance

**Architecture**
1. **Database**: Aurora PostgreSQL (ACID compliance)
   - Multi-AZ for high availability
   - Read replicas for reporting (separate from production)
2. **Transactions**: Use database transactions for trade execution (BEGIN, COMMIT, ROLLBACK)
3. **Audit Trail**: Multiple layers
   - **Database Audit Logs**: RDS option group for pgAudit extension
   - **Application Logs**: CloudWatch Logs with all trade activity
   - **QLDB**: Immutable ledger for compliance (cryptographically verifiable history)
   - **Database Activity Streams**: Real-time stream of database activity to Kinesis
4. **Encryption**: KMS customer-managed keys, TLS 1.3 in transit
5. **Backup**: Automated backups (35-day retention), manual snapshots before major events
6. **Compliance**: SOC 2, FINRA, SEC regulations
7. **Monitoring**: Performance Insights, CloudWatch alarms for errors

**Key Considerations**
- ACID: Aurora PostgreSQL provides full ACID guarantees
- Audit: Immutable audit trail with QLDB (cannot be altered)
- Retention: 7-year retention for regulatory compliance (SEC Rule 17a-4)
- Encryption: FIPS 140-2 validated encryption
- DR: Cross-region automated backups (RPO 5 min, RTO 1 hour)

### Scenario 5: Read-Heavy Application with ElastiCache

**Context**: News website with 10M daily users, read-heavy (99% reads), occasional writes

**Architecture**
1. **Database**: Aurora MySQL (1 writer, 2 readers)
2. **Caching**: ElastiCache Redis Cluster (cluster mode enabled)
   - 6 shards, 2 replicas per shard (18 nodes total)
   - Multi-AZ with automatic failover
3. **Caching Strategy**:
   - **Article Content**: Cache for 1 hour (write-through on publish)
   - **User Sessions**: Cache indefinitely (logout deletes)
   - **Trending Articles**: Cache for 5 minutes (lazy loading)
4. **Cache Invalidation**: Pub/Sub to notify application of content updates
5. **Fallback**: If cache miss, read from Aurora read replica
6. **Monitoring**: CloudWatch metrics for cache hit ratio (target 95%+), evictions

**Performance Results**
- **Before Caching**: 50K database queries/second, 80% CPU on Aurora
- **After Caching**: 95% cache hit ratio → 2.5K database queries/second, 10% CPU
- **Cost Savings**: Reduce Aurora instance size (xlarge → large), 50% cost reduction

## AWS CLI Examples

```bash
# RDS - Create Multi-AZ MySQL database
aws rds create-db-instance \
  --db-instance-identifier production-mysql \
  --db-instance-class db.r6g.2xlarge \
  --engine mysql \
  --engine-version 8.0.35 \
  --master-username admin \
  --master-user-password 'SecurePassword123!' \
  --allocated-storage 500 \
  --storage-type gp3 \
  --iops 12000 \
  --storage-throughput 500 \
  --multi-az \
  --db-subnet-group-name production-subnet-group \
  --vpc-security-group-ids sg-12345678 \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:05:00" \
  --storage-encrypted \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012 \
  --enable-iam-database-authentication \
  --enable-cloudwatch-logs-exports '["error","general","slowquery"]' \
  --enable-performance-insights \
  --tags Key=Environment,Value=Production Key=Application,Value=ECommerce

# RDS - Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier production-mysql-replica-1 \
  --source-db-instance-identifier production-mysql \
  --db-instance-class db.r6g.xlarge \
  --availability-zone us-east-1b \
  --publicly-accessible false

# RDS - Create cross-region read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier production-mysql-dr-replica \
  --source-db-instance-identifier arn:aws:rds:us-east-1:123456789012:db:production-mysql \
  --db-instance-class db.r6g.2xlarge \
  --region eu-west-1

# RDS - Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier production-mysql \
  --db-snapshot-identifier production-mysql-snapshot-2025-01-10 \
  --tags Key=Purpose,Value=BeforeMigration

# RDS - Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier production-mysql-restored \
  --db-snapshot-identifier production-mysql-snapshot-2025-01-10 \
  --db-instance-class db.r6g.2xlarge \
  --multi-az

# Aurora - Create cluster
aws rds create-db-cluster \
  --db-cluster-identifier production-aurora-cluster \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.05.2 \
  --master-username admin \
  --master-user-password 'SecurePassword123!' \
  --db-subnet-group-name production-subnet-group \
  --vpc-security-group-ids sg-12345678 \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --storage-encrypted \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012 \
  --enable-cloudwatch-logs-exports '["audit","error","general","slowquery"]' \
  --enable-iam-database-authentication

# Aurora - Add instance to cluster
aws rds create-db-instance \
  --db-instance-identifier production-aurora-instance-1 \
  --db-cluster-identifier production-aurora-cluster \
  --db-instance-class db.r6g.2xlarge \
  --engine aurora-mysql \
  --publicly-accessible false

# Aurora - Create Global Database
aws rds create-global-cluster \
  --global-cluster-identifier production-global-cluster \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.05.2 \
  --source-db-cluster-identifier arn:aws:rds:us-east-1:123456789012:cluster:production-aurora-cluster

# Aurora - Add secondary region
aws rds create-db-cluster \
  --db-cluster-identifier production-aurora-cluster-eu \
  --engine aurora-mysql \
  --global-cluster-identifier production-global-cluster \
  --region eu-west-1

# DynamoDB - Create table with on-demand capacity
aws dynamodb create-table \
  --table-name Orders \
  --attribute-definitions \
    AttributeName=OrderId,AttributeType=S \
    AttributeName=CustomerId,AttributeType=S \
    AttributeName=OrderDate,AttributeType=S \
  --key-schema \
    AttributeName=OrderId,KeyType=HASH \
  --global-secondary-indexes '[
    {
      "IndexName": "CustomerIndex",
      "KeySchema": [
        {"AttributeName": "CustomerId", "KeyType": "HASH"},
        {"AttributeName": "OrderDate", "KeyType": "RANGE"}
      ],
      "Projection": {"ProjectionType": "ALL"}
    }
  ]' \
  --billing-mode PAY_PER_REQUEST \
  --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES \
  --sse-specification Enabled=true,SSEType=KMS,KMSMasterKeyId=arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012 \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --tags Key=Environment,Value=Production

# DynamoDB - Enable Global Tables
aws dynamodb update-table \
  --table-name Orders \
  --replica-updates '[
    {"Create": {"RegionName": "eu-west-1"}},
    {"Create": {"RegionName": "ap-southeast-1"}}
  ]'

# DynamoDB - Put item
aws dynamodb put-item \
  --table-name Orders \
  --item '{"OrderId": {"S": "order-12345"}, "CustomerId": {"S": "customer-789"}, "OrderDate": {"S": "2025-01-10"}, "TotalAmount": {"N": "99.99"}}'

# DynamoDB - Query with GSI
aws dynamodb query \
  --table-name Orders \
  --index-name CustomerIndex \
  --key-condition-expression "CustomerId = :cid" \
  --expression-attribute-values '{":cid": {"S": "customer-789"}}' \
  --limit 10

# DAX - Create cluster
aws dax create-cluster \
  --cluster-name orders-dax-cluster \
  --node-type dax.r5.large \
  --replication-factor 3 \
  --iam-role-arn arn:aws:iam::123456789012:role/DAXServiceRole \
  --subnet-group-name production-subnet-group \
  --security-group-ids sg-12345678 \
  --sse-specification Enabled=true

# ElastiCache - Create Redis cluster (cluster mode enabled)
aws elasticache create-replication-group \
  --replication-group-id production-redis \
  --replication-group-description "Production Redis cluster" \
  --engine redis \
  --engine-version 7.0 \
  --cache-node-type cache.r6g.xlarge \
  --num-node-groups 3 \
  --replicas-per-node-group 2 \
  --cache-subnet-group-name production-subnet-group \
  --security-group-ids sg-12345678 \
  --at-rest-encryption-enabled \
  --transit-encryption-enabled \
  --auth-token "SecurePassword123!" \
  --automatic-failover-enabled \
  --multi-az-enabled \
  --snapshot-retention-limit 7 \
  --snapshot-window "03:00-04:00" \
  --tags Key=Environment,Value=Production

# ElastiCache - Create Memcached cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id production-memcached \
  --cache-node-type cache.r6g.xlarge \
  --engine memcached \
  --engine-version 1.6.17 \
  --num-cache-nodes 3 \
  --cache-subnet-group-name production-subnet-group \
  --security-group-ids sg-12345678 \
  --az-mode cross-az

# RDS Proxy - Create proxy
aws rds create-db-proxy \
  --db-proxy-name production-mysql-proxy \
  --engine-family MYSQL \
  --auth '[{"AuthScheme":"SECRETS","SecretArn":"arn:aws:secretsmanager:us-east-1:123456789012:secret:db-password","IAMAuth":"REQUIRED"}]' \
  --role-arn arn:aws:iam::123456789012:role/RDSProxyRole \
  --vpc-subnet-ids subnet-11111111 subnet-22222222 \
  --require-tls

# RDS Proxy - Register target
aws rds register-db-proxy-targets \
  --db-proxy-name production-mysql-proxy \
  --db-instance-identifiers production-mysql

# DMS - Create replication instance
aws dms create-replication-instance \
  --replication-instance-identifier dms-migration-instance \
  --replication-instance-class dms.c5.2xlarge \
  --allocated-storage 200 \
  --vpc-security-group-ids sg-12345678 \
  --replication-subnet-group-identifier dms-subnet-group \
  --multi-az \
  --engine-version 3.5.1 \
  --publicly-accessible false

# DMS - Create source endpoint (Oracle)
aws dms create-endpoint \
  --endpoint-identifier oracle-source \
  --endpoint-type source \
  --engine-name oracle \
  --username admin \
  --password 'SourcePassword123!' \
  --server-name oracle.example.com \
  --port 1521 \
  --database-name PROD

# DMS - Create target endpoint (Aurora)
aws dms create-endpoint \
  --endpoint-identifier aurora-target \
  --endpoint-type target \
  --engine-name aurora-postgresql \
  --username admin \
  --password 'TargetPassword123!' \
  --server-name production-aurora-cluster.cluster-abc123.us-east-1.rds.amazonaws.com \
  --port 5432 \
  --database-name production

# DMS - Create replication task
aws dms create-replication-task \
  --replication-task-identifier oracle-to-aurora-migration \
  --source-endpoint-arn arn:aws:dms:us-east-1:123456789012:endpoint:oracle-source \
  --target-endpoint-arn arn:aws:dms:us-east-1:123456789012:endpoint:aurora-target \
  --replication-instance-arn arn:aws:dms:us-east-1:123456789012:rep:dms-migration-instance \
  --migration-type full-load-and-cdc \
  --table-mappings file://table-mappings.json

# DMS - Start replication task
aws dms start-replication-task \
  --replication-task-arn arn:aws:dms:us-east-1:123456789012:task:oracle-to-aurora-migration \
  --start-replication-task-type start-replication
```

## Study Tips

### SAP-C02 Exam Focus Areas

**High-Priority Database Topics**
- Database selection based on workload (OLTP, OLAP, NoSQL, graph, time-series)
- RDS Multi-AZ vs. Read Replicas (HA vs. scaling)
- Aurora Global Database for multi-region
- DynamoDB capacity modes, Global Tables, DAX
- ElastiCache Redis vs. Memcached
- Database migration strategies (DMS, heterogeneous migrations)
- Backup and disaster recovery (RPO/RTO)
- Caching strategies (lazy loading, write-through, TTL)

**Scenario-Based Questions**
- Choose database based on requirements (consistency, latency, scalability, cost)
- Design multi-region database architecture
- Optimize read-heavy or write-heavy workloads
- Implement caching to reduce database load
- Migrate from on-premises or commercial databases
- Design for high availability and disaster recovery
- Cost optimization strategies

**Common Decision Points**
- **RDS vs. Aurora**: Standard features vs. cloud-native, cost vs. performance
- **Aurora vs. DynamoDB**: Relational vs. NoSQL, ACID vs. eventual consistency
- **ElastiCache Redis vs. Memcached**: Complex data types vs. simple cache
- **ElastiCache vs. DAX**: General purpose vs. DynamoDB-specific
- **Multi-AZ vs. Read Replicas**: HA vs. read scaling
- **On-Demand vs. Provisioned** (DynamoDB): Unpredictable vs. predictable traffic

### Key Differences from SAA-C03

**SAA-C03 Database Knowledge**
- Basic RDS (engines, Multi-AZ, Read Replicas)
- DynamoDB fundamentals
- ElastiCache basics
- Aurora overview

**Additional SAP-C02 Requirements**
- Aurora Global Database for DR and global reads
- Aurora Serverless for variable workloads
- DynamoDB advanced features (Global Tables, Streams, PITR, DAX)
- ElastiCache Global Datastore for multi-region
- Database migration strategies with DMS and SCT
- RDS Proxy for connection management
- Specialized databases (Neptune, Timestream, QLDB, DocumentDB, Keyspaces)
- Multi-tenant database strategies
- Database security (IAM auth, encryption, audit logs)
- Performance optimization (Performance Insights, caching, indexes)

### Complex Scenarios to Master

**Global Database Architectures**
- Aurora Global Database (RPO 1s, RTO <1min)
- DynamoDB Global Tables (active-active)
- Multi-region read replicas
- Route 53 routing to nearest region
- Failover procedures and testing

**Migration Strategies**
- Homogeneous (Oracle to Aurora, MySQL to RDS MySQL)
- Heterogeneous (SQL Server to Aurora PostgreSQL)
- Schema Conversion Tool for compatibility
- DMS full load + CDC for minimal downtime
- Cutover planning and rollback

**Performance Optimization**
- Caching layers (ElastiCache, DAX)
- Read replicas for read scaling
- Connection pooling with RDS Proxy
- Query optimization with Performance Insights
- Index design and maintenance

**High Availability and DR**
- Multi-AZ deployments
- Cross-region automated backups
- Point-in-time recovery
- RPO/RTO requirements and design
- Failover testing procedures

**Multi-Tenant Data Isolation**
- Database per tenant (highest isolation)
- Schema per tenant (medium isolation)
- Shared schema with TenantId (lowest isolation)
- Hybrid approach based on customer tier
- Security and query complexity

### Practice Lab Recommendations

1. **RDS Multi-AZ and Read Replicas**
   - Create RDS MySQL Multi-AZ
   - Add read replicas in multiple AZs
   - Test failover (reboot with failover)
   - Measure failover time and replication lag
   - Configure RDS Proxy

2. **Aurora Global Database**
   - Create Aurora cluster in primary region
   - Add Aurora cluster in secondary region (Global Database)
   - Test replication lag
   - Simulate regional failure and promote secondary
   - Measure RPO and RTO

3. **DynamoDB with DAX**
   - Create DynamoDB table with GSI
   - Create DAX cluster
   - Test performance with and without DAX
   - Measure cache hit ratio
   - Enable Global Tables for multi-region

4. **ElastiCache Redis Caching**
   - Create Redis cluster (cluster mode enabled)
   - Implement caching in application (lazy loading)
   - Measure cache hit ratio
   - Test failover (delete primary node)
   - Compare performance with and without caching

5. **Database Migration with DMS**
   - Set up source database (MySQL or PostgreSQL)
   - Create target Aurora cluster
   - Create DMS replication instance
   - Create source and target endpoints
   - Create and start replication task (full load + CDC)
   - Monitor migration progress

6. **Performance Tuning**
   - Enable Performance Insights on RDS
   - Identify slow queries
   - Add indexes to improve performance
   - Test before and after metrics
   - Use EXPLAIN to understand query plans

## Additional Resources

### AWS Whitepapers
- AWS Database Services Overview
- Best Practices for Amazon RDS
- Amazon Aurora Best Practices
- Amazon DynamoDB Best Practices
- Database Caching Strategies Using Redis
- Migrating Databases to AWS
- Disaster Recovery for Databases on AWS

### Documentation Links
- Amazon RDS: https://docs.aws.amazon.com/rds/
- Amazon Aurora: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/
- Amazon DynamoDB: https://docs.aws.amazon.com/dynamodb/
- Amazon ElastiCache: https://docs.aws.amazon.com/elasticache/
- AWS DMS: https://docs.aws.amazon.com/dms/
- Amazon Neptune: https://docs.aws.amazon.com/neptune/
- Amazon Timestream: https://docs.aws.amazon.com/timestream/
- Amazon QLDB: https://docs.aws.amazon.com/qldb/

### Video Resources
- AWS re:Invent Database sessions (search "DAT" track)
- Deep Dive on Amazon Aurora
- Amazon DynamoDB Deep Dive
- Database Migration Best Practices
- Amazon ElastiCache Best Practices
