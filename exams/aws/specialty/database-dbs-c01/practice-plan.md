# AWS Certified Database - Specialty (DBS-C01) - Practice Plan

## ⚠️ IMPORTANT NOTICE

**This certification was RETIRED on April 29, 2024 and is no longer available for new candidates.**

If you're looking to pursue a database certification, please consider:
- **AWS Certified Data Engineer - Associate** (Launched March 2024) - Recommended replacement
  - **[📖 Data Engineer Certification](https://aws.amazon.com/certification/certified-data-engineer-associate/)** - Official certification page
- **AWS Certified Solutions Architect - Professional** (Covers advanced database architectures)
  - **[📖 Solutions Architect Professional](https://aws.amazon.com/certification/certified-solutions-architect-professional/)** - Advanced certification
- **AWS Certified Solutions Architect - Associate** (Covers database fundamentals)
  - **[📖 Solutions Architect Associate](https://aws.amazon.com/certification/certified-solutions-architect-associate/)** - Foundation certification

---

## Historical Practice Plan (For Reference Only)

This practice plan is preserved for those who earned the DBS-C01 certification and want to reference their study materials, or for understanding AWS database concepts.

### Week 1-2: Relational Databases - RDS and Aurora
- RDS instance deployment (MySQL, PostgreSQL, Oracle, SQL Server, MariaDB)
  - **[📖 RDS Getting Started](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.html)** - Launch your first RDS instance
- Multi-AZ deployments for high availability
  - **[📖 Multi-AZ Deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)** - High availability configuration
- Read replicas for scaling reads
  - **[📖 Read Replicas Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)** - Creating and managing read replicas
- Aurora clusters and global databases
  - **[📖 Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)** - Multi-region setup
- Aurora Serverless configuration
  - **[📖 Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)** - Serverless configuration
- Backup and restore procedures
  - **[📖 RDS Backup and Restore](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_CommonTasks.BackupRestore.html)** - Backup strategies
- Automated backups vs manual snapshots

### Week 3-4: NoSQL Databases - DynamoDB
- Table design and partition key selection
  - **[📖 DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)** - Design patterns and optimization
  - **[📖 Partition Key Design](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)** - Choosing effective partition keys
- DynamoDB Streams for change capture
  - **[📖 DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)** - Change data capture
- Global tables for multi-region replication
  - **[📖 Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)** - Multi-region active-active
- DynamoDB Accelerator (DAX) for caching
  - **[📖 DAX Documentation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)** - In-memory caching
- On-demand vs provisioned capacity
  - **[📖 Capacity Modes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html)** - Billing and scaling
- Secondary indexes (GSI and LSI)
  - **[📖 Secondary Indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)** - GSI and LSI overview
- DynamoDB Transactions
  - **[📖 Transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transactions.html)** - ACID transactions
- Point-in-time recovery
  - **[📖 Point-in-Time Recovery](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html)** - Backup and restore

### Week 5: Specialized Databases
- **Amazon Neptune** - Graph database for social networks, knowledge graphs
  - **[📖 Neptune User Guide](https://docs.aws.amazon.com/neptune/latest/userguide/)** - Graph database guide
- **Amazon QLDB** - Ledger database for immutable audit trails
  - **[📖 QLDB Developer Guide](https://docs.aws.amazon.com/qldb/latest/developerguide/)** - Ledger database guide
- **Amazon Timestream** - Time series data for IoT and monitoring
  - **[📖 Timestream Developer Guide](https://docs.aws.amazon.com/timestream/latest/developerguide/)** - Time series database
- **Amazon DocumentDB** - MongoDB-compatible document database
  - **[📖 DocumentDB Developer Guide](https://docs.aws.amazon.com/documentdb/latest/developerguide/)** - MongoDB compatibility
- **Amazon Keyspaces** - Apache Cassandra-compatible database
  - **[📖 Keyspaces Developer Guide](https://docs.aws.amazon.com/keyspaces/latest/devguide/)** - Cassandra compatibility
- **Amazon MemoryDB for Redis** - In-memory database
  - **[📖 MemoryDB Developer Guide](https://docs.aws.amazon.com/memorydb/latest/devguide/)** - Redis-compatible in-memory database

### Week 6-7: Data Warehousing and Caching
- Amazon Redshift cluster design and node types
  - **[📖 Redshift Clusters](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-clusters.html)** - Cluster management
- Redshift distribution styles and sort keys
  - **[📖 Table Design Best Practices](https://docs.aws.amazon.com/redshift/latest/dg/c_designing-tables-best-practices.html)** - Distribution and sort keys
- Redshift Spectrum for querying S3 data
  - **[📖 Redshift Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html)** - Query external data
- Redshift Serverless
  - **[📖 Redshift Serverless](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-serverless.html)** - Serverless data warehouse
- ElastiCache for Redis and Memcached
  - **[📖 ElastiCache User Guide](https://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/)** - Caching strategies
- Caching strategies and patterns
  - **[📖 Caching Strategies](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Strategies.html)** - Cache design patterns
- In-memory database use cases

### Week 8-9: Database Migration
- AWS Database Migration Service (DMS) architecture
  - **[📖 DMS User Guide](https://docs.aws.amazon.com/dms/latest/userguide/)** - Migration overview
  - **[📖 DMS Best Practices](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html)** - Planning and execution
- Homogeneous vs heterogeneous migrations
  - **[📖 Migration Types](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Introduction.html)** - Understanding migration scenarios
- Schema Conversion Tool (SCT)
  - **[📖 SCT User Guide](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/)** - Schema conversion
- Change Data Capture (CDC) for ongoing replication
  - **[📖 CDC with DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Task.CDC.html)** - Continuous replication
- Migration validation and testing
  - **[📖 Data Validation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Validating.html)** - Validating migrated data
- Snowball Edge for large database migrations
  - **[📖 AWS Snowball](https://docs.aws.amazon.com/snowball/)** - Large-scale data transfer
- Database migration strategies (lift-and-shift, re-platform, refactor)

### Week 10: Database Security
- Encryption at rest with KMS
  - **[📖 RDS Encryption](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html)** - Encrypting database instances
- Encryption in transit with SSL/TLS
  - **[📖 Using SSL/TLS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html)** - Secure connections
- IAM database authentication
  - **[📖 IAM DB Authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)** - Token-based authentication
- VPC security for databases
  - **[📖 VPC and RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html)** - Network isolation
- Database activity streams
  - **[📖 Database Activity Streams](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/DBActivityStreams.html)** - Real-time monitoring
- Secrets Manager for credential rotation
  - **[📖 Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)** - Rotating credentials
- Network isolation with VPC endpoints
  - **[📖 VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)** - Private connectivity
- Compliance and auditing with CloudTrail
  - **[📖 Logging with CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)** - API auditing

### Week 11: Monitoring and Performance
- CloudWatch metrics for databases
  - **[📖 CloudWatch Monitoring](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html)** - Database metrics
- Enhanced monitoring for RDS
  - **[📖 Enhanced Monitoring](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Monitoring.OS.html)** - OS-level metrics
- Performance Insights for query analysis
  - **[📖 Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html)** - Query performance tuning
- Slow query logs
  - **[📖 Database Logs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.html)** - Accessing log files
- Database parameter groups optimization
  - **[📖 Parameter Groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html)** - Database tuning
- Query optimization techniques
  - **[📖 Query Optimization](https://docs.aws.amazon.com/redshift/latest/dg/c-optimizing-query-performance.html)** - Performance best practices
- Identifying and resolving bottlenecks
- Auto Scaling for read replicas and DynamoDB
  - **[📖 DynamoDB Auto Scaling](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/AutoScaling.html)** - Capacity management

### Week 12: Review and Practice Exams
- Complete 2-3 full practice exams
- Hands-on labs for each database service
- Review incorrect answers and weak areas
- Database selection decision trees
- Cost optimization strategies
- Disaster recovery scenarios

## Key Concepts by Domain

### 1. Workload-Specific Database Design (26%)

**Database Selection Criteria**
- **Relational (RDS/Aurora)**: ACID transactions, complex queries, established schemas
- **Key-Value (DynamoDB)**: High throughput, simple queries, flexible schema
- **Document (DocumentDB)**: Semi-structured data, JSON documents
- **Graph (Neptune)**: Highly connected data, relationship queries
- **Time Series (Timestream)**: IoT data, monitoring metrics, logs
- **Ledger (QLDB)**: Immutable history, cryptographic verification
- **In-Memory (ElastiCache/MemoryDB)**: Microsecond latency, caching

**Design Patterns**
- Normalization vs denormalization
- Partition key design for DynamoDB
- Read replica placement
- Caching layer architecture
- Multi-region database design

### 2. Deployment and Migration (20%)

**Deployment Best Practices**
- Infrastructure as code (CloudFormation)
- Automated deployment pipelines
- Blue/green deployments for databases
- Canary deployments

**Migration Strategies**
- DMS for minimal downtime migrations
- Snapshot and restore for one-time migrations
- SCT for schema conversion (Oracle/SQL Server to Aurora)
- Validation and cutover planning

### 3. Management and Operations (18%)

**Operational Excellence**
- Automated backups and retention policies
- Maintenance window scheduling
- Parameter group management
- Version upgrades (minor and major)
- Scaling strategies (vertical and horizontal)

**High Availability**
- Multi-AZ deployments
- Aurora fault tolerance
- DynamoDB global tables
- Automated failover procedures

### 4. Monitoring and Troubleshooting (18%)

**Monitoring**
- CloudWatch metrics and alarms
- Performance Insights
- Enhanced monitoring
- Database activity streams
- Custom CloudWatch dashboards

**Troubleshooting**
- Slow query identification
- Connection pooling issues
- Replication lag
- Storage and IOPS constraints
- Lock contention

### 5. Database Security (18%)

**Security Layers**
- Network: VPC, security groups, NACLs
- Access: IAM policies, database users
- Encryption: KMS for at-rest, SSL/TLS for in-transit
- Auditing: CloudTrail, database logs
- Secrets: Secrets Manager, Parameter Store

## Hands-On Lab Exercises

### Lab 1: RDS Multi-AZ with Read Replicas
1. Create RDS MySQL instance with Multi-AZ
2. Enable automated backups
3. Create read replica in different region
4. Test failover scenario
5. Restore from snapshot

### Lab 2: DynamoDB Global Tables
1. Design partition key strategy
2. Create DynamoDB table with streams
3. Configure global table in 3 regions
4. Test cross-region replication
5. Implement DAX caching layer

### Lab 3: Aurora Serverless
1. Deploy Aurora Serverless v2 cluster
2. Configure auto-scaling
3. Test scaling under load
4. Implement Data API for serverless apps
5. Monitor with Performance Insights

### Lab 4: Database Migration with DMS
1. Set up source and target databases
2. Configure DMS replication instance
3. Create migration task with CDC
4. Validate data consistency
5. Perform cutover

### Lab 5: Database Security Hardening
1. Enable encryption at rest
2. Configure IAM database authentication
3. Set up Secrets Manager rotation
4. Enable database activity streams
5. Configure VPC endpoints

## Exam Preparation Tips

### Focus Areas
- Database selection for specific use cases
- RDS vs Aurora capabilities and differences
- DynamoDB partition key design
- Migration strategies and DMS
- Security best practices (encryption, IAM, VPC)
- High availability and disaster recovery
- Performance optimization and troubleshooting

### Common Scenarios
- Choosing between relational and NoSQL databases
- Designing highly available database architectures
- Migrating on-premises databases to AWS
- Optimizing slow database queries
- Implementing multi-region database solutions
- Database security and compliance requirements

## Recommended Migration Path

### For Database Professionals

**Primary Path: AWS Certified Data Engineer - Associate**
- Comprehensive coverage of AWS data services
- Includes database design, implementation, and operations
- Aligned with modern data engineering roles
- Associate-level certification

**Alternative Path: Solutions Architect Track**
1. AWS Certified Solutions Architect - Associate
   - Database architecture fundamentals
   - Multi-tier application design
   - Cost optimization

2. AWS Certified Solutions Architect - Professional
   - Advanced database architectures
   - Enterprise-scale migrations
   - Multi-region strategies

**Complementary Certifications**
- AWS Certified SysOps Administrator - Associate (Operations focus)
- AWS Certified Security - Specialty (Database security)
- AWS Certified Machine Learning - Specialty (Databases for ML)

## Learning Resources

### Official AWS Resources
- [AWS Database Blog](https://aws.amazon.com/blogs/database/)
- [AWS Database Documentation](https://docs.aws.amazon.com/)
- [AWS Skill Builder](https://skillbuilder.aws/) - Free database courses
- [AWS re:Invent Database Sessions](https://www.youtube.com/user/AmazonWebServices)

### Hands-On Practice
- AWS Free Tier for experimentation
- [AWS Workshops - Database Tracks](https://workshops.aws/)
- [AWS Samples - Database GitHub repos](https://github.com/aws-samples)

### Study Guides and Practice Tests
- AWS Official Practice Exam (if available before retirement)
- Tutorials Dojo practice tests (historical)
- Whizlabs practice exams (historical)
- A Cloud Guru / Linux Academy courses (historical)

### Database-Specific Resources
- **RDS/Aurora**: [RDS User Guide](https://docs.aws.amazon.com/rds/)
- **DynamoDB**: [DynamoDB Developer Guide](https://docs.aws.amazon.com/dynamodb/)
- **Redshift**: [Redshift Database Developer Guide](https://docs.aws.amazon.com/redshift/)
- **DMS**: [DMS User Guide](https://docs.aws.amazon.com/dms/)

## Database Service Comparison Chart

| Database | Type | Use Cases | Key Features |
|----------|------|-----------|--------------|
| **RDS MySQL/PostgreSQL** | Relational | Traditional apps, ACID transactions | Multi-AZ, read replicas, automated backups |
| **Aurora** | Relational | High performance, cloud-native apps | 5x faster than MySQL, auto-scaling storage |
| **DynamoDB** | NoSQL Key-Value | High throughput, serverless apps | Single-digit ms latency, global tables |
| **DocumentDB** | NoSQL Document | MongoDB workloads | MongoDB compatible, fully managed |
| **Neptune** | Graph | Social networks, fraud detection | Gremlin/SPARQL support, high availability |
| **QLDB** | Ledger | Audit trails, immutable records | Cryptographically verifiable |
| **Timestream** | Time Series | IoT, monitoring, analytics | Built-in time series analytics |
| **Redshift** | Data Warehouse | Analytics, BI, reporting | Columnar storage, Spectrum for S3 |
| **ElastiCache** | In-Memory | Caching, session store | Redis/Memcached, microsecond latency |

## Notes

- If you hold the DBS-C01 certification, it remains valid for 3 years from the date earned
- The database knowledge from DBS-C01 remains highly relevant
- AWS database services continue to be critical for cloud applications
- Consider pursuing current certifications to maintain and expand your expertise

---

**Last Updated**: October 2024
**Status**: RETIRED - Reference Only
