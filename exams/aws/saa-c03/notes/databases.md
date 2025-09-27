# AWS Database Fundamentals

## Database Service Overview

### Relational vs NoSQL
| Aspect | Relational (SQL) | NoSQL |
|--------|------------------|-------|
| **Structure** | Fixed schema, tables, rows | Flexible schema, documents, key-value |
| **ACID** | Full ACID compliance | Eventual consistency (typically) |
| **Scaling** | Vertical (scale up) | Horizontal (scale out) |
| **Use Cases** | Complex relationships, transactions | High scale, flexible data, simple queries |
| **Examples** | MySQL, PostgreSQL, Oracle | MongoDB, DynamoDB, Cassandra |

### AWS Database Decision Tree
```
Need ACID transactions? → RDS/Aurora
Need NoSQL flexibility? → DynamoDB
Need data warehouse? → Redshift
Need in-memory cache? → ElastiCache
Need graph database? → Neptune
Need time-series data? → Timestream
```

## Amazon RDS (Relational Database Service)

### What is RDS?
Managed relational database service that makes it easy to set up, operate, and scale databases in the cloud.

### Supported Database Engines
| Engine | Latest Version | Use Case |
|--------|----------------|----------|
| **MySQL** | 8.0 | Web applications, e-commerce |
| **PostgreSQL** | 14 | Complex queries, JSON data |
| **MariaDB** | 10.6 | MySQL alternative, better performance |
| **Oracle** | 19c | Enterprise applications |
| **SQL Server** | 2019 | Microsoft stack applications |
| **Amazon Aurora** | MySQL 8.0 / PostgreSQL 13 | Cloud-native, high performance |

### RDS Instance Classes
| Class | Type | Use Case |
|-------|------|----------|
| **db.t3/t4g** | Burstable | Development, light workloads |
| **db.m5/m6** | General Purpose | Balanced performance |
| **db.r5/r6** | Memory Optimized | Memory-intensive applications |
| **db.x1/x2** | High Memory | In-memory databases |

### RDS Storage Types
| Type | IOPS | Throughput | Use Case |
|------|------|------------|----------|
| **General Purpose (gp2)** | 3-16,000 | Up to 250 MB/s | Most workloads |
| **General Purpose (gp3)** | 3,000-16,000 | 125-1,000 MB/s | Baseline performance |
| **Provisioned IOPS (io1)** | Up to 64,000 | Up to 1,000 MB/s | I/O intensive |
| **Magnetic** | Not specified | Not specified | Legacy (not recommended) |

### Multi-AZ Deployments
**High availability configuration**

#### How Multi-AZ Works
1. **Primary database**: Handles all read/write operations
2. **Standby replica**: Synchronously replicated in different AZ
3. **Automatic failover**: If primary fails, standby becomes primary
4. **Single endpoint**: Application uses same connection string

#### Multi-AZ Benefits
- **High availability**: Automatic failover
- **Data durability**: Synchronous replication
- **Maintenance**: Zero-downtime updates
- **Backup**: Backups from standby (no I/O impact)

### Read Replicas
**Scaling read operations**

#### Read Replica Features
- **Asynchronous replication**: Eventually consistent
- **Cross-region**: Can be in different regions
- **Multiple replicas**: Up to 15 per source
- **Promote to standalone**: Can become independent database
- **Read-only**: Cannot write to read replicas

#### Read Replica Use Cases
- **Read scaling**: Distribute read traffic
- **Reporting**: Analytics without impacting production
- **Cross-region**: Serve users closer to their location
- **Disaster recovery**: Promote replica in different region

### RDS Backup and Recovery
| Feature | Automated Backups | Manual Snapshots |
|---------|-------------------|------------------|
| **Retention** | 1-35 days | Until deleted |
| **Frequency** | Daily + transaction logs | On-demand |
| **Point-in-time recovery** | Yes | No |
| **Cross-region copy** | Automatic option | Manual |
| **Encryption** | Same as source | Same as source |

## Amazon Aurora

### What is Aurora?
MySQL and PostgreSQL-compatible relational database built for the cloud, combining performance and availability.

### Aurora Architecture
- **Cluster volume**: Shared storage across AZs
- **Writer instance**: Handles write operations
- **Reader instances**: Handle read operations (up to 15)
- **Storage auto-scaling**: Grows from 10GB to 128TB automatically

### Aurora vs RDS Comparison
| Feature | Aurora | RDS |
|---------|--------|-----|
| **Performance** | 5x MySQL, 3x PostgreSQL | Standard engine performance |
| **Storage** | Auto-scaling, distributed | Fixed size, single AZ |
| **Failover** | < 30 seconds | 1-2 minutes |
| **Read replicas** | Up to 15 | Up to 5 |
| **Backtrack** | Yes (MySQL) | No |
| **Global database** | Yes | No |

### Aurora Serverless
**On-demand, auto-scaling configuration**

#### When to Use Aurora Serverless
- **Infrequent workloads**: Applications used occasionally
- **Unpredictable workloads**: Variable traffic patterns
- **Development/testing**: Cost-effective for non-production
- **New applications**: Uncertain capacity requirements

#### Aurora Serverless Features
- **Auto-scaling**: Scales compute up/down automatically
- **Pay per use**: Charge only for resources consumed
- **Pause/resume**: Automatically pause when not in use
- **No connection management**: Simplified connection handling

### Aurora Global Database
**Cross-region disaster recovery and read scaling**

#### Global Database Features
- **Cross-region replication**: < 1 second lag
- **Up to 5 secondary regions**: Global read scaling
- **Recovery Point Objective (RPO)**: < 1 second
- **Recovery Time Objective (RTO)**: < 1 minute
- **Read replicas**: Up to 16 per secondary region

## Amazon DynamoDB

### What is DynamoDB?
Fully managed NoSQL database service providing fast and predictable performance with seamless scalability.

### Key Characteristics
- **Serverless**: No servers to manage
- **Single-digit millisecond latency**: High performance
- **Automatic scaling**: Handle traffic spikes
- **Global tables**: Multi-region replication
- **ACID transactions**: Full transaction support

### Data Model
```
Table: Users
├── Partition Key: UserID (Hash key)
├── Sort Key: Timestamp (Range key) [Optional]
└── Attributes: Name, Email, Status, etc.

Item Example:
{
  "UserID": "123",
  "Name": "John Doe", 
  "Email": "john@example.com",
  "LoginCount": 42,
  "LastLogin": "2023-01-15T10:30:00Z"
}
```

### Primary Keys
| Type | Components | Use Case |
|------|------------|----------|
| **Partition Key only** | Hash key | Simple key-value access |
| **Composite Key** | Hash key + Range key | Complex queries, sorting |

### DynamoDB Indexes
#### Global Secondary Index (GSI)
- **Different partition/sort key**: Query on non-key attributes
- **Separate throughput**: Independent capacity settings
- **Eventually consistent**: May have slight delay
- **Limit**: 20 per table

#### Local Secondary Index (LSI)
- **Same partition key**: Different sort key
- **Shared throughput**: Uses table's capacity
- **Strongly consistent**: Immediate consistency
- **Limit**: 10 per table, must be created at table creation

### Capacity Modes
| Mode | Description | Use Case |
|------|-------------|----------|
| **On-Demand** | Pay per request | Unpredictable traffic |
| **Provisioned** | Pre-allocated capacity | Predictable traffic |

### DynamoDB Streams
**Capture data modification events**

#### Stream Records
- **INSERT**: New item added
- **MODIFY**: Item updated
- **REMOVE**: Item deleted
- **NEW_AND_OLD_IMAGES**: Before and after values

#### Stream Use Cases
- **Real-time analytics**: Process changes as they happen
- **Replication**: Keep other systems in sync
- **Audit trails**: Track all changes
- **Triggers**: Invoke Lambda functions

## Amazon ElastiCache

### What is ElastiCache?
Managed in-memory caching service supporting Redis and Memcached engines.

### Redis vs Memcached
| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Data Types** | Strings, lists, sets, hashes | Strings only |
| **Persistence** | Disk snapshots | Memory only |
| **Replication** | Master-slave | None |
| **Clustering** | Yes | Yes |
| **Multi-AZ** | Yes | No |
| **Backup/Restore** | Yes | No |

### ElastiCache Use Cases
- **Database caching**: Reduce database load
- **Session storage**: Share sessions across servers
- **Real-time analytics**: Fast data processing
- **Gaming leaderboards**: High-performance rankings
- **Chat/messaging**: Real-time communication

### Redis Cluster Modes
| Mode | Description | Use Case |
|------|-------------|----------|
| **Disabled** | Single primary node | Simple caching |
| **Enabled** | Multiple shards | High availability, scaling |

### Caching Strategies
#### Lazy Loading (Cache-Aside)
```
1. Check cache for data
2. If cache miss, get from database
3. Write data to cache
4. Return data to application
```

#### Write-Through
```
1. Write data to cache
2. Write data to database
3. Return success to application
```

#### TTL (Time To Live)
```
Set expiration time for cached data
Automatically removes stale data
Balances freshness vs performance
```

## Amazon Redshift

### What is Redshift?
Fully managed data warehouse service designed for analytical workloads and business intelligence.

### Key Features
- **Columnar storage**: Optimized for analytics
- **Massively parallel processing**: Distribute queries across nodes
- **Compression**: Reduce storage costs
- **Result caching**: Cache query results
- **Federated queries**: Query external data sources

### Node Types
| Type | Use Case | Storage | Compute |
|------|----------|---------|---------|
| **RA3** | General purpose | Managed storage | Flexible compute |
| **DC2** | Compute intensive | Local SSD | High performance |
| **DS2** | Storage intensive | Local HDD | Balanced |

### Redshift Architecture
```
Cluster
├── Leader Node (Query planning, result aggregation)
└── Compute Nodes (Data storage, query execution)
    ├── Node Slice 1
    ├── Node Slice 2
    └── Node Slice N
```

### Data Distribution Styles
| Style | Description | Use Case |
|-------|-------------|----------|
| **AUTO** | Redshift chooses | General recommendation |
| **EVEN** | Round-robin distribution | No clear distribution key |
| **KEY** | Distribute by column value | Join optimization |
| **ALL** | Copy to all nodes | Small dimension tables |

### Redshift Spectrum
**Query data in S3 without loading**

#### Spectrum Benefits
- **No data movement**: Query data where it lives
- **Cost effective**: Pay only for queries run
- **Scalable**: Virtually unlimited S3 storage
- **Format support**: Parquet, ORC, JSON, CSV

## Other Database Services

### Amazon Neptune
**Managed graph database**

#### Graph Use Cases
- **Social networks**: Friend relationships, recommendations
- **Fraud detection**: Transaction pattern analysis
- **Knowledge graphs**: Connected information
- **Network security**: Threat analysis

#### Supported APIs
- **Apache TinkerPop Gremlin**: Property graph
- **W3C SPARQL**: RDF graphs

### Amazon DocumentDB
**MongoDB-compatible document database**

#### Key Features
- **MongoDB compatibility**: Existing applications work
- **Fully managed**: AWS handles operations
- **Scalable**: Separate compute and storage
- **Backup and restore**: Point-in-time recovery

### Amazon Timestream
**Time-series database**

#### Time-Series Use Cases
- **IoT applications**: Sensor data collection
- **DevOps**: Application metrics
- **Analytics**: Operational intelligence
- **Monitoring**: System performance tracking

### Amazon QLDB
**Immutable ledger database**

#### Ledger Use Cases
- **Financial systems**: Transaction history
- **Supply chain**: Product tracking
- **Regulatory compliance**: Audit trails
- **Voting systems**: Transparent records

## Database Migration

### AWS Database Migration Service (DMS)
**Migrate databases to AWS**

#### Migration Types
| Type | Description | Downtime |
|------|-------------|----------|
| **One-time** | Single migration event | Yes |
| **Continuous** | Ongoing replication | Minimal |
| **Test migration** | Validate migration process | N/A |

#### Schema Conversion Tool (SCT)
- **Heterogeneous migrations**: Different database engines
- **Schema analysis**: Identify conversion complexity
- **Code conversion**: Stored procedures, functions
- **Assessment reports**: Migration feasibility

### Migration Strategies
#### Homogeneous (Same database engine)
```
Oracle on-premises → Oracle on RDS
MySQL on-premises → MySQL on RDS
```

#### Heterogeneous (Different database engines)
```
Oracle on-premises → PostgreSQL on RDS
SQL Server on-premises → Aurora MySQL
```

## Database Best Practices

### Performance Optimization
- **Choose appropriate instance types**: Match workload requirements
- **Use read replicas**: Scale read operations
- **Implement caching**: Reduce database load
- **Monitor performance**: CloudWatch metrics and logs
- **Optimize queries**: Use indexes, avoid full table scans

### Security Best Practices
- **Encryption at rest**: Enable for sensitive data
- **Encryption in transit**: Use SSL/TLS connections
- **VPC**: Deploy in private subnets
- **IAM**: Use database authentication where possible
- **Security groups**: Restrict network access

### Backup and Recovery
- **Automated backups**: Enable for production databases
- **Cross-region backups**: Disaster recovery planning
- **Test recovery**: Regular recovery testing
- **Point-in-time recovery**: Enable transaction log backups
- **Snapshot management**: Automate snapshot lifecycle

### Cost Optimization
- **Right-sizing**: Choose appropriate instance types
- **Reserved instances**: Commit to long-term usage
- **Storage optimization**: Use appropriate storage types
- **Monitoring**: Track usage and costs
- **Cleanup**: Remove unused snapshots and backups

## Common Database Pitfalls

### RDS Issues
- Not enabling automated backups
- Using Single-AZ for production
- Insufficient monitoring and alerting
- Wrong instance type for workload
- Not using parameter groups for optimization

### DynamoDB Issues
- Poor partition key design causing hot partitions
- Over-provisioning capacity (unnecessary costs)
- Not using indexes efficiently
- Lack of error handling for throttling
- Not implementing appropriate backup strategies

### Performance Problems
- Missing database indexes
- Inefficient query patterns
- Not using connection pooling
- Inadequate instance sizing
- Poor network configuration

### Security Vulnerabilities
- Databases in public subnets
- Overly permissive security groups
- Not using encryption
- Weak authentication methods
- Missing audit logging