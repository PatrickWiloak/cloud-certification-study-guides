# AWS Database Services

## Overview

AWS offers a comprehensive portfolio of purpose-built database services to support diverse data models including relational, key-value, document, in-memory, graph, time series, wide column, and ledger databases.

## Amazon RDS (Relational Database Service)

### Overview
Amazon RDS is a managed relational database service that makes it easy to set up, operate, and scale databases in the cloud. AWS manages the infrastructure, operating system, and database software.

### Supported Database Engines
- **Amazon Aurora**: AWS-designed, MySQL and PostgreSQL compatible
- **MySQL**: Open-source relational database
- **PostgreSQL**: Advanced open-source relational database
- **MariaDB**: MySQL fork with additional features
- **Oracle Database**: Enterprise database with license-included or BYOL
- **Microsoft SQL Server**: Microsoft's relational database

### Key Features

#### Automated Administration
- **Provisioning**: Launch database in minutes
- **Patching**: Automatic software patching
- **Backups**: Automated daily backups with point-in-time recovery
- **Snapshots**: Manual snapshots for long-term retention
- **Monitoring**: Performance metrics and CloudWatch integration

#### High Availability
- **Multi-AZ Deployment**:
  - Synchronous replication to standby instance
  - Automatic failover in case of failure
  - Different Availability Zone for standby
  - Zero downtime maintenance with Multi-AZ
  - Protection against AZ failure

- **Read Replicas**:
  - Asynchronous replication
  - Up to 15 read replicas (Aurora), 5 (others)
  - Scale read-heavy workloads
  - Cross-region read replicas available
  - Can be promoted to standalone database

#### Scalability
- **Vertical Scaling**: Change instance size
- **Storage Scaling**: Automatic storage scaling
- **Read Scaling**: Add read replicas
- **Push-button Scaling**: No downtime required (most cases)

### Backup and Recovery

#### Automated Backups
- **Retention**: 0-35 days (default 7 days)
- **Point-in-Time Recovery**: Restore to any second within retention period
- **Storage**: S3 storage for backups
- **Transaction Logs**: Backed up every 5 minutes
- **No Performance Impact**: Backups from standby (Multi-AZ)

#### Database Snapshots
- **Manual**: User-initiated snapshots
- **Retention**: Keep indefinitely
- **Copy**: Copy to other regions
- **Share**: Share with other AWS accounts
- **Restore**: Create new database from snapshot

### Security Features
- **Encryption at Rest**: KMS encryption for data and backups
- **Encryption in Transit**: SSL/TLS connections
- **Network Isolation**: VPC for network isolation
- **IAM Integration**: Database authentication with IAM
- **Security Groups**: Control network access
- **Audit Logging**: Track database activity

### Use Cases
- **Web Applications**: Scalable backend databases
- **E-commerce**: Transaction processing
- **Mobile Applications**: Backend data storage
- **Enterprise Applications**: ERP, CRM systems
- **SaaS Applications**: Multi-tenant databases

### RDS vs Self-Managed

| Feature | Amazon RDS | Self-Managed EC2 |
|---------|-----------|------------------|
| Setup | Minutes | Hours/Days |
| Patching | Automatic | Manual |
| Backups | Automatic | Manual setup |
| High Availability | Multi-AZ | Manual setup |
| Scaling | Push-button | Manual |
| Monitoring | Built-in | Manual setup |
| Cost | Service fee | Instance cost only |
| Control | Limited | Full control |

## Amazon Aurora

### Overview
Amazon Aurora is a MySQL and PostgreSQL-compatible relational database built for the cloud. It combines the performance and availability of high-end commercial databases with the simplicity and cost-effectiveness of open-source databases.

### Key Features
- **Performance**: Up to 5x throughput of MySQL, 3x of PostgreSQL
- **Scalability**: Up to 128 TB per database instance
- **Durability**: 6 copies of data across 3 Availability Zones
- **Self-healing**: Continuous backup to S3
- **Fast Recovery**: Automatic crash recovery in less than 60 seconds

### Aurora Replicas
- **Number**: Up to 15 Aurora Replicas
- **Failover**: Automatic failover to replica (priority-based)
- **Read Scaling**: Distribute read traffic across replicas
- **Lag**: Sub-10ms replica lag
- **Endpoint**: Reader endpoint for load-balanced reads

### Aurora Serverless
- **Auto-Scaling**: Automatically scales capacity up/down
- **Pay-per-Use**: Pay only for database capacity used
- **Use Cases**: Infrequent, intermittent, or unpredictable workloads
- **Versions**: v1 (simpler), v2 (instant scaling, more features)
- **Cost Effective**: No need to provision for peak capacity

### Aurora Global Database
- **Cross-Region**: Span multiple AWS Regions
- **Replication**: Sub-second replication lag
- **Disaster Recovery**: Fast recovery from region-wide outages
- **Low Latency**: Low-latency global reads
- **Promotion**: Promote secondary region in < 1 minute

### Use Cases
- **Enterprise Applications**: Mission-critical applications
- **SaaS Applications**: Multi-tenant applications
- **Web/Mobile Games**: Scalable game backends
- **Financial Services**: High-availability transactional systems

## Amazon DynamoDB

### Overview
Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability.

### Key Characteristics
- **Fully Managed**: No servers to manage
- **Performance**: Single-digit millisecond latency at any scale
- **Scalability**: Automatic scaling of throughput capacity
- **Availability**: Multi-AZ, multi-region replication available
- **Durability**: Built-in redundancy and backup

### Data Model

#### Tables
- **Schema-less**: No rigid schema definition
- **Primary Key**: Required for each item (partition key or partition + sort key)
- **Attributes**: Flexible attributes per item
- **Item Size**: Maximum 400 KB per item

#### Primary Keys

**Partition Key (Hash Key)**
- **Simple**: Single attribute primary key
- **Distribution**: Determines physical partition
- **Uniqueness**: Must be unique for each item

**Composite Key (Partition + Sort Key)**
- **Two Attributes**: Partition key + sort key
- **Uniqueness**: Combination must be unique
- **Queries**: Enable range queries on sort key
- **Use Case**: One-to-many relationships

### Capacity Modes

#### Provisioned Capacity
- **Read/Write Units**: Specify RCU and WCU
- **Predictable**: Consistent, predictable workloads
- **Auto Scaling**: Automatically adjust capacity
- **Reserved Capacity**: Purchase reserved capacity for savings
- **Cost**: Pay for provisioned capacity

#### On-Demand Capacity
- **Auto-Scaling**: Automatically scales up/down
- **Unpredictable**: Variable, unpredictable workloads
- **No Planning**: No capacity planning required
- **Cost**: Pay per request
- **Use Case**: New tables, unpredictable workloads

### Key Features

#### DynamoDB Streams
- **Change Log**: Time-ordered sequence of changes
- **Retention**: 24-hour retention
- **Use Cases**: Triggers, replication, analytics
- **Integration**: Lambda triggers, data pipelines

#### Global Tables
- **Multi-Region**: Fully replicated across regions
- **Active-Active**: Read and write to any region
- **Replication**: Automatic, sub-second replication
- **Use Cases**: Global applications, disaster recovery

#### Point-in-Time Recovery (PITR)
- **Backup**: Continuous backups for 35 days
- **Recovery**: Restore to any second within retention
- **No Performance Impact**: Backups don't affect performance

#### DAX (DynamoDB Accelerator)
- **In-Memory Cache**: Microsecond response times
- **Compatible**: Fully compatible with DynamoDB API
- **Use Cases**: Read-heavy workloads, gaming leaderboards
- **Cluster**: Highly available multi-node cluster

### Secondary Indexes

#### Global Secondary Index (GSI)
- **Alternative Key**: Different partition and/or sort key
- **Projection**: Specify attributes to include
- **Capacity**: Separate RCU/WCU from table
- **Creation**: Can be added after table creation
- **Queries**: Query on alternate attributes

#### Local Secondary Index (LSI)
- **Same Partition Key**: Same partition key, different sort key
- **Creation**: Must be created with table
- **Capacity**: Shares table's RCU/WCU
- **Limit**: Maximum 5 LSIs per table
- **Projection**: All, keys-only, or include specific attributes

### Use Cases
- **Mobile Applications**: User profiles, session data
- **Gaming**: Player data, game state, leaderboards
- **IoT**: Sensor data, device metadata
- **E-commerce**: Shopping carts, product catalogs
- **Ad Tech**: Real-time bidding, click stream data

## Amazon ElastiCache

### Overview
Amazon ElastiCache is a fully managed in-memory caching service supporting Redis and Memcached engines.

### Supported Engines

#### Redis
- **Data Structures**: Strings, lists, sets, sorted sets, hashes
- **Persistence**: Optional data persistence
- **Replication**: Master-replica replication
- **High Availability**: Automatic failover with Redis Cluster
- **Pub/Sub**: Publish/subscribe messaging
- **Transactions**: Atomic operations
- **Geospatial**: Geospatial data support

#### Memcached
- **Simple**: Simple key-value store
- **Multi-threaded**: Better for multi-threaded applications
- **Horizontal Scaling**: Add/remove nodes easily
- **No Persistence**: Data lost on restart
- **Use Case**: Simple caching needs

### Key Features

#### Performance
- **Sub-millisecond Latency**: In-memory performance
- **Throughput**: Millions of operations per second
- **Scaling**: Horizontal and vertical scaling

#### High Availability (Redis)
- **Multi-AZ**: Automatic failover across AZs
- **Replication**: Up to 5 read replicas
- **Cluster Mode**: Partition data across shards
- **Backup/Restore**: Automated and manual snapshots

#### Security
- **VPC**: Network isolation
- **Encryption**: At-rest and in-transit encryption
- **IAM**: IAM authentication for Redis
- **Redis AUTH**: Password protection

### Use Cases
- **Session Store**: Web session management
- **Database Caching**: Reduce database load
- **Leaderboards**: Gaming leaderboards (sorted sets)
- **Pub/Sub**: Real-time messaging
- **Rate Limiting**: API rate limiting
- **Real-time Analytics**: Stream processing

## Amazon Redshift

### Overview
Amazon Redshift is a fast, fully managed data warehouse that makes it simple and cost-effective to analyze data using standard SQL and existing business intelligence tools.

### Key Features
- **Columnar Storage**: Efficient data compression and query performance
- **Massively Parallel Processing (MPP)**: Distributes queries across nodes
- **Performance**: Up to 10x faster than traditional databases
- **Scalability**: Petabyte-scale data warehousing
- **SQL Compatible**: Standard SQL queries

### Architecture

#### Cluster
- **Leader Node**: Manages client connections, query planning
- **Compute Nodes**: Execute queries, store data
- **Slices**: Parallel processing units per compute node

#### Node Types
- **RA3**: Managed storage, scales compute and storage independently
- **DC2**: Dense compute, SSD storage
- **DS2**: Dense storage, HDD storage (previous generation)

### Key Capabilities

#### Redshift Spectrum
- **Query S3**: Query data directly in S3
- **No Loading**: No need to load data into Redshift
- **Exabyte Scale**: Query exabytes of data
- **Format Support**: Parquet, ORC, JSON, CSV

#### Concurrency Scaling
- **Automatic**: Adds capacity for concurrent queries
- **Consistent**: Consistent performance during load spikes
- **Cost**: Pay per-second for additional capacity

#### Automated Backups
- **Snapshots**: Automated and manual snapshots
- **Retention**: 1-35 days (default 1 day)
- **Cross-Region**: Copy snapshots to another region
- **Point-in-Time**: Restore to any snapshot

### Use Cases
- **Business Intelligence**: Analytics and reporting
- **Data Warehousing**: Central data repository
- **Log Analysis**: Analyze application and system logs
- **Big Data**: Large-scale data analysis
- **Historical Data**: Long-term data storage and analysis

## Additional AWS Database Services

### Amazon DocumentDB
- **MongoDB Compatible**: Document database compatible with MongoDB
- **Fully Managed**: Automated patching, backups, scaling
- **Use Cases**: Content management, catalogs, user profiles
- **Storage**: Automatically scales up to 64 TB

### Amazon Neptune
- **Graph Database**: Fully managed graph database
- **Support**: Property graph and RDF models
- **Query**: Gremlin and SPARQL query languages
- **Use Cases**: Social networks, recommendation engines, fraud detection
- **Performance**: Optimized for graph queries

### Amazon Keyspaces (for Apache Cassandra)
- **Cassandra Compatible**: Managed Cassandra-compatible database
- **Serverless**: On-demand pricing option
- **Use Cases**: Time-series data, IoT applications
- **Scalability**: Scales automatically

### Amazon QLDB (Quantum Ledger Database)
- **Ledger Database**: Immutable, cryptographically verifiable transaction log
- **Journal**: Central, trusted authority
- **Use Cases**: Financial transactions, supply chain, regulatory compliance
- **Transparency**: Complete change history

### Amazon Timestream
- **Time Series**: Purpose-built time-series database
- **Performance**: 1000x faster, 1/10th cost of relational databases
- **Use Cases**: IoT applications, operational metrics, real-time analytics
- **Retention**: Automated tiering to cost-optimize storage

## Database Selection Guide

### Choosing the Right Database

#### Relational (RDS/Aurora)
- **Use When**: ACID compliance, complex queries, joins required
- **Examples**: E-commerce, ERP, CRM, financial applications

#### NoSQL (DynamoDB)
- **Use When**: High throughput, flexible schema, key-value access
- **Examples**: Gaming, ad tech, IoT, mobile apps

#### In-Memory (ElastiCache)
- **Use When**: Sub-millisecond latency, caching, session management
- **Examples**: Session stores, leaderboards, real-time analytics

#### Data Warehouse (Redshift)
- **Use When**: Complex analytics, business intelligence, historical data
- **Examples**: BI dashboards, data analytics, reporting

#### Document (DocumentDB)
- **Use When**: Flexible schema, JSON documents, content management
- **Examples**: Content management, catalogs, user profiles

#### Graph (Neptune)
- **Use When**: Connected data, relationship queries
- **Examples**: Social networks, recommendation engines, knowledge graphs

#### Time Series (Timestream)
- **Use When**: Time-stamped data, IoT, metrics
- **Examples**: IoT applications, DevOps monitoring, analytics

## Database Migration

### AWS Database Migration Service (DMS)
- **Purpose**: Migrate databases to AWS
- **Source**: On-premises, EC2, RDS, other clouds
- **Target**: RDS, Aurora, Redshift, DynamoDB, S3
- **Minimal Downtime**: Continuous data replication
- **Schema Conversion**: AWS Schema Conversion Tool (SCT)

### Migration Types
- **Homogeneous**: Same database engine (Oracle to RDS Oracle)
- **Heterogeneous**: Different engines (Oracle to Aurora)
- **Continuous Replication**: Keep source and target in sync

## Best Practices

### General Database Best Practices
1. **Choose Right Database**: Match database to use case
2. **Enable Backups**: Automated backups for all production databases
3. **Multi-AZ**: Use Multi-AZ for production workloads
4. **Monitoring**: Enable CloudWatch monitoring and alarms
5. **Security**: Enable encryption, use VPC, implement least privilege
6. **Scaling**: Plan for growth, use read replicas, caching
7. **Cost Optimization**: Right-size instances, use reserved capacity

### Performance Optimization
1. **Indexing**: Create appropriate indexes
2. **Caching**: Use ElastiCache for frequently accessed data
3. **Read Replicas**: Distribute read traffic
4. **Connection Pooling**: Reuse database connections
5. **Query Optimization**: Optimize slow queries

## Key Takeaways

1. **RDS**: Managed relational databases, automated administration, Multi-AZ for HA
2. **Aurora**: High-performance MySQL/PostgreSQL compatible, up to 15 replicas
3. **DynamoDB**: Fully managed NoSQL, millisecond latency, automatic scaling
4. **ElastiCache**: In-memory caching (Redis/Memcached), sub-millisecond latency
5. **Redshift**: Data warehouse, columnar storage, petabyte-scale analytics
6. **Purpose-Built**: AWS offers purpose-built databases for specific use cases
7. **Migration**: DMS enables easy migration from various sources to AWS
8. **Selection**: Choose database based on data model, access patterns, and scale requirements
