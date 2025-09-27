# GCP Storage Fundamentals

## Cloud Storage Basics

### What is Cloud Storage?
Google Cloud's object storage service for storing and retrieving any amount of data from anywhere. Think of it as infinite cloud storage for files, backups, and static content.

### Core Concepts
- **Bucket**: Container for objects (globally unique name)
- **Object**: Individual files stored in Cloud Storage
- **Project**: Billing and organization boundary
- **Location**: Geographic placement of your data
- **Storage Class**: Performance and cost optimization tier

### Storage Classes Comparison

| Storage Class | Use Case | Availability SLA | Minimum Storage Duration | Access Cost |
|---------------|----------|------------------|-------------------------|-------------|
| **Standard** | Frequently accessed | 99.95% (multi-region) 99.9% (region) | None | None |
| **Nearline** | Once per month | 99.95% (multi-region) 99.9% (region) | 30 days | Low |
| **Coldline** | Once per quarter | 99.95% (multi-region) 99.9% (region) | 90 days | Higher |
| **Archive** | Once per year | 99.95% (multi-region) 99.9% (region) | 365 days | Highest |

### Location Types
| Type | Description | Examples | Use Case |
|------|-------------|----------|----------|
| **Multi-region** | Continent-wide | us, eu, asia | Global applications, high availability |
| **Dual-region** | Two specific regions | nam4 (Iowa + South Carolina) | Disaster recovery with specific regions |
| **Region** | Single geographic region | us-central1, europe-west1 | Regional applications, lower latency |

### Key Features
- **Strong Consistency**: Read-after-write consistency
- **Versioning**: Keep multiple versions of objects
- **Lifecycle Management**: Automatically transition/delete objects
- **Access Control**: IAM, ACLs, signed URLs
- **Event Notifications**: Trigger actions on object changes
- **Transfer Service**: Migrate data from other clouds or on-premises

## Persistent Disks (Block Storage)

### What are Persistent Disks?
Network-attached block storage for Compute Engine instances. Like virtual hard drives that can be attached to VMs.

### Disk Types

| Type | Performance | Use Case | Max Size |
|------|-------------|----------|----------|
| **Standard (pd-standard)** | Good | General purpose, cost-effective | 64 TB |
| **Balanced (pd-balanced)** | Better | Alternative to SSD with lower cost | 64 TB |
| **SSD (pd-ssd)** | High IOPS | Databases, high-performance | 64 TB |
| **Extreme (pd-extreme)** | Highest IOPS | Mission-critical, consistent performance | 64 TB |

### Disk Performance
- **IOPS**: Input/output operations per second
- **Throughput**: MB/s of data transfer
- **Performance scales**: With disk size (except pd-extreme)
- **Provisioned IOPS**: pd-extreme allows provisioned performance

### Regional vs Zonal Disks
| Feature | Zonal | Regional |
|---------|-------|----------|
| **Availability** | Single zone | Replicated across zones |
| **Cost** | Lower | 2x zonal cost |
| **Performance** | Higher | Slightly lower (network latency) |
| **Use Case** | Cost-sensitive | High availability required |

### Key Features
- **Snapshots**: Point-in-time backups
- **Encryption**: Automatic encryption at rest
- **Resize**: Expand disk size without downtime
- **Clone**: Create disks from other disks/snapshots
- **Multi-attach**: Read-only access from multiple VMs

## Filestore (Managed NFS)

### What is Filestore?
Fully managed Network File System (NFS) service for applications requiring shared file storage.

### Service Tiers
| Tier | Performance | Capacity | Use Case |
|------|-------------|----------|----------|
| **Basic HDD** | Good | 1-63.9 TB | General file sharing |
| **Basic SSD** | High | 2.5-63.9 TB | Performance-sensitive workloads |
| **High Scale SSD** | Very High | 10-100+ TB | High-performance computing |
| **Enterprise** | Guaranteed | 1-10 TB | Mission-critical applications |

### Key Characteristics
- **POSIX-compliant**: Standard file system semantics
- **Regional**: Available across zones in a region
- **Scalable**: Grow and shrink capacity as needed
- **Concurrent Access**: Multiple Compute Engine instances
- **Backup**: Automated backups available

### Use Cases
- **Content Management**: Shared content across web servers
- **Data Analytics**: Shared datasets for processing
- **Media Workflows**: Video/audio processing pipelines
- **Enterprise Applications**: Legacy applications requiring NFS

## Cloud SQL (Managed Relational Databases)

### Supported Database Engines
- **MySQL**: Popular open-source database
- **PostgreSQL**: Advanced open-source database
- **SQL Server**: Microsoft's relational database

### Instance Types
| Type | vCPUs | Memory | Use Case |
|------|-------|--------|----------|
| **Shared-core** | Shared | 0.6-3.75 GB | Development, testing |
| **Standard** | 1-96 | 3.75-624 GB | Production workloads |
| **High-memory** | 2-96 | 13-624 GB | Memory-intensive applications |

### High Availability & Disaster Recovery
- **Regional Persistent Disks**: Automatic failover within region
- **Cross-Region Replicas**: Read replicas in different regions
- **Point-in-Time Recovery**: Restore to specific timestamp
- **Automated Backups**: Daily backups with transaction logs

### Key Features
- **Automatic Updates**: OS and database patches
- **Scaling**: Vertical scaling with minimal downtime
- **Security**: Encryption at rest and in transit
- **Monitoring**: Integration with Cloud Monitoring
- **Private IP**: VPC-native connectivity

## Firestore (NoSQL Document Database)

### What is Firestore?
Fully managed NoSQL document database designed for mobile and web applications.

### Firestore vs Datastore
| Feature | Firestore (Native) | Datastore |
|---------|-------------------|-----------|
| **Real-time**: Yes | No |
| **Mobile SDKs**: Yes | No |
| **ACID Transactions**: Yes | Limited |
| **Queries**: More flexible | Basic |
| **Status**: Current | Legacy |

### Data Model
- **Documents**: JSON-like records
- **Collections**: Groups of documents
- **Subcollections**: Collections within documents
- **Fields**: Key-value pairs in documents

### Key Features
- **Real-time Listeners**: Live synchronization
- **Offline Support**: Local caching and sync
- **ACID Transactions**: Multi-document transactions
- **Security Rules**: Client-side security
- **Auto-scaling**: Handles traffic spikes automatically

## BigQuery (Data Warehouse)

### What is BigQuery?
Fully managed, serverless data warehouse for analytics using standard SQL.

### Key Characteristics
- **Serverless**: No infrastructure to manage
- **Petabyte Scale**: Handle massive datasets
- **Standard SQL**: ANSI SQL 2011 compliance
- **Streaming Inserts**: Real-time data ingestion
- **Machine Learning**: Built-in ML capabilities

### Storage & Compute Separation
- **Storage**: Pay for data stored
- **Compute**: Pay for queries processed
- **Slots**: Units of computational capacity
- **On-demand**: Pay per query
- **Flat-rate**: Reserved slots for predictable pricing

### Data Organization
- **Datasets**: Top-level container for tables
- **Tables**: Structured data with schema
- **Partitioning**: Divide tables by date/time
- **Clustering**: Organize data within partitions

## Bigtable (NoSQL Wide-Column)

### What is Bigtable?
High-performance NoSQL database service for large analytical and operational workloads.

### Key Characteristics
- **Low Latency**: Single-digit millisecond latency
- **High Throughput**: Millions of operations per second
- **Scalable**: Petabyte+ scale
- **Wide-Column**: Non-relational, wide-column format
- **HBase Compatible**: Standard HBase API

### Use Cases
- **Time Series**: IoT data, financial data
- **Real-time Analytics**: Personalization, fraud detection
- **Machine Learning**: Feature storage, model serving
- **Gaming**: Leaderboards, player data

### Performance Considerations
- **Node Count**: More nodes = higher performance
- **Key Design**: Avoid hotspotting
- **Column Families**: Group related columns
- **Garbage Collection**: Clean up old data

## Storage Decision Framework

### Choosing Storage Type
```
Object storage for files/backups? → Cloud Storage
Block storage for VMs? → Persistent Disks
Shared file system? → Filestore
Relational database? → Cloud SQL
Document database for mobile/web? → Firestore
Data warehouse/analytics? → BigQuery
High-performance NoSQL? → Bigtable
```

### Cloud Storage Class Selection
```
Frequent access? → Standard
Monthly access? → Nearline
Quarterly access? → Coldline
Yearly access or archive? → Archive
Unknown pattern? → Start with Standard, use lifecycle
```

### Persistent Disk Type Selection
```
General purpose? → Balanced (pd-balanced)
High IOPS database? → SSD (pd-ssd)
Consistent high performance? → Extreme (pd-extreme)
Cost-sensitive backup? → Standard (pd-standard)
```

## Practical Examples

### Web Application Storage
- **Static Assets**: Cloud Storage + Cloud CDN
- **Application Data**: Persistent Disks for VMs
- **User Data**: Firestore for real-time features
- **Analytics**: Stream to BigQuery
- **File Uploads**: Cloud Storage with signed URLs

### Data Analytics Pipeline
- **Raw Data**: Cloud Storage data lake
- **Processing**: Dataflow with Persistent Disks
- **Data Warehouse**: BigQuery for analytics
- **Results**: Cloud Storage or Firestore
- **Monitoring**: Cloud Monitoring for pipeline health

### Gaming Backend
- **Player Data**: Firestore for real-time sync
- **Leaderboards**: Bigtable for high performance
- **Game Assets**: Cloud Storage + CDN
- **Analytics**: BigQuery for player behavior
- **Backups**: Cloud Storage Archive class

## Best Practices

### Cloud Storage
- Choose appropriate storage class for access patterns
- Use lifecycle policies for cost optimization
- Implement proper access controls (IAM)
- Enable versioning for important data
- Use regional/multi-regional based on availability needs
- Monitor storage costs and usage

### Persistent Disks
- Choose appropriate disk type for workload
- Use regional disks for high availability
- Regular snapshots for backup/disaster recovery
- Monitor disk performance metrics
- Size disks appropriately for performance needs
- Use encryption for sensitive data

### Database Services
- Choose right database type for use case
- Implement proper indexing strategies
- Monitor performance and optimize queries
- Use read replicas for read-heavy workloads
- Regular backups and test recovery procedures
- Implement proper security measures

## Cost Optimization Strategies

### Cloud Storage
- Use appropriate storage classes
- Implement lifecycle policies
- Delete unnecessary object versions
- Use regional storage for regional access
- Monitor and alert on unexpected usage
- Compress data when possible

### Persistent Disks
- Right-size disks for actual usage
- Use snapshots efficiently (incremental)
- Consider zonal vs regional based on needs
- Monitor unused/unattached disks
- Use appropriate disk types for workload

### Database Optimization
- **Cloud SQL**: Right-size instances, use read replicas
- **BigQuery**: Partition tables, avoid SELECT *
- **Firestore**: Design efficient data structure
- **Bigtable**: Optimize node count for workload

## Common Pitfalls

### Cloud Storage Issues
- Not understanding storage class minimum durations
- Incorrect lifecycle policy configurations
- Bucket naming conflicts (global namespace)
- Not optimizing for access patterns
- Unexpected egress costs

### Persistent Disk Issues
- Over-provisioning disk size for performance
- Not taking regular snapshots
- Incorrect disk type for workload
- Not monitoring disk utilization
- Forgetting about snapshot costs

### Database Common Issues
- **Cloud SQL**: Not using connection pooling
- **BigQuery**: Querying without LIMIT or WHERE clauses
- **Firestore**: Poor data structure design
- **Bigtable**: Hotspotting due to poor key design