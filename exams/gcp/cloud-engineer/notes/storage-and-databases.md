# Storage and Databases - GCP Associate Cloud Engineer

## Overview

This document covers Google Cloud storage and database services including Cloud Storage, Cloud SQL, Cloud Firestore, Cloud Bigtable, and BigQuery. Understanding storage options and database selection is essential for the Associate Cloud Engineer certification.

## Key Topics

### 1. Cloud Storage
- Object storage for unstructured data
- Storage classes for different access patterns
- Lifecycle management and versioning
- Access control and security

### 2. Cloud SQL
- Fully managed relational databases (MySQL, PostgreSQL, SQL Server)
- High availability and automatic backups
- Read replicas for scaling read operations
- Point-in-time recovery

### 3. Cloud Firestore
- NoSQL document database
- Real-time synchronization
- Mobile and web client libraries
- Automatic scaling and multi-region replication

### 4. Cloud Bigtable
- Wide-column NoSQL database
- High-throughput, low-latency workloads
- Scales to billions of rows
- Integration with big data tools

### 5. BigQuery
- Serverless data warehouse
- SQL interface for analytics
- Petabyte-scale data analysis
- Machine learning capabilities

## GCP Services Reference

### Cloud Storage
- **Storage Classes**: Standard, Nearline, Coldline, Archive
- **Buckets**: Containers for storing objects
- **Objects**: Individual files stored in buckets
- **Lifecycle Management**: Automatic object management based on rules
- **Versioning**: Keep multiple versions of objects
- **Object Lifecycle**: Automate transitions between storage classes
- **Signed URLs**: Time-limited access to objects
- **CORS**: Cross-origin resource sharing configuration

### Cloud SQL
- **Instances**: Managed database instances
- **Databases**: Individual databases within an instance
- **Backups**: Automated and on-demand backups
- **Read Replicas**: Scale read operations across multiple replicas
- **Failover Replicas**: High availability configuration
- **Point-in-Time Recovery**: Restore to specific timestamp
- **Maintenance Windows**: Scheduled maintenance periods

### Cloud Firestore
- **Documents**: Individual records with fields
- **Collections**: Groups of documents
- **Indexes**: Query optimization structures
- **Security Rules**: Access control at document level
- **Real-time Listeners**: Subscribe to data changes
- **Offline Persistence**: Client-side caching

### Cloud Bigtable
- **Instances**: Container for Bigtable clusters
- **Clusters**: Sets of nodes in a specific zone
- **Tables**: Store data in rows and column families
- **Column Families**: Groups of related columns
- **Replication**: Multi-cluster replication for availability
- **Backups**: Full cluster backups for disaster recovery

### BigQuery
- **Datasets**: Containers for tables and views
- **Tables**: Structured data storage
- **Views**: Virtual tables based on queries
- **Partitioned Tables**: Divide tables by date or range
- **Clustered Tables**: Organize data by column values
- **Streaming Inserts**: Real-time data ingestion
- **BigQuery ML**: Machine learning in SQL

## Best Practices

### Cloud Storage Best Practices
1. **Storage Class Selection**: Choose based on access frequency and requirements
2. **Lifecycle Policies**: Automate transitions to lower-cost storage classes
3. **Regional vs Multi-Regional**: Balance cost, performance, and availability
4. **Object Versioning**: Enable for critical data protection
5. **Access Control**: Use IAM and ACLs appropriately
6. **Encryption**: Use default encryption or customer-managed keys
7. **Retention Policies**: Prevent accidental deletion of critical data
8. **Transfer Optimization**: Use gsutil for large transfers with parallelism

### Cloud SQL Best Practices
1. **High Availability**: Enable for production workloads
2. **Automated Backups**: Configure daily backups with appropriate retention
3. **Read Replicas**: Use for scaling read-heavy workloads
4. **Connection Pooling**: Implement to manage database connections efficiently
5. **Private IP**: Use for secure, internal connectivity
6. **Maintenance Windows**: Schedule during low-traffic periods
7. **Monitoring**: Set up alerts for CPU, memory, and storage
8. **Query Optimization**: Use slow query logs and explain plans

### Cloud Firestore Best Practices
1. **Data Model Design**: Design for queries, avoid deep nesting
2. **Security Rules**: Implement granular access control
3. **Index Management**: Create composite indexes for complex queries
4. **Batch Operations**: Use batches and transactions for multiple writes
5. **Offline Persistence**: Enable for mobile and web apps
6. **Data Validation**: Implement validation in security rules
7. **Limit Subcollections**: Use collections at root level when possible
8. **Cost Optimization**: Monitor read/write operations and storage

### Cloud Bigtable Best Practices
1. **Schema Design**: Use tall and narrow tables, avoid hotspotting
2. **Row Key Design**: Critical for performance, avoid sequential keys
3. **Replication**: Use for disaster recovery and load balancing
4. **Node Scaling**: Scale based on throughput requirements
5. **Column Families**: Organize related data, limit number of families
6. **Garbage Collection**: Configure appropriate retention policies
7. **Monitoring**: Track CPU utilization and latency metrics
8. **Bulk Loading**: Use Dataflow or other tools for initial data loads

### BigQuery Best Practices
1. **Partitioning**: Use date or range partitioning for large tables
2. **Clustering**: Organize data within partitions for better performance
3. **Query Optimization**: Avoid SELECT *, use appropriate filters
4. **Slot Management**: Monitor and optimize query slot usage
5. **Cost Control**: Set project-level and user-level quotas
6. **Table Expiration**: Set expiration for temporary tables
7. **Materialized Views**: Use for frequently accessed aggregations
8. **Streaming Inserts**: Use for real-time data, understand cost implications

## Common Scenarios

### Scenario 1: Static Website Hosting
**Requirement**: Host static website files with global access
**Solution**:
- Use Cloud Storage bucket with public access
- Enable website configuration
- Use Cloud CDN for global content delivery
- Implement lifecycle policies for old versions

### Scenario 2: Transactional Application Database
**Requirement**: Relational database for e-commerce application
**Solution**:
- Use Cloud SQL with high availability enabled
- Configure automated backups and point-in-time recovery
- Implement read replicas for reporting queries
- Use private IP for secure connectivity from GKE or Compute Engine

### Scenario 3: Mobile App Backend
**Requirement**: Real-time data synchronization for mobile application
**Solution**:
- Use Cloud Firestore for document storage
- Implement security rules for user-level access control
- Use real-time listeners for live updates
- Enable offline persistence for mobile clients

### Scenario 4: IoT Time-Series Data
**Requirement**: Store and query billions of sensor readings
**Solution**:
- Use Cloud Bigtable for high-throughput writes
- Design row keys with sensor ID and timestamp
- Implement time-based retention policies
- Query using Dataflow or other big data tools

### Scenario 5: Data Warehouse and Analytics
**Requirement**: Analyze petabytes of log data
**Solution**:
- Use BigQuery for data warehouse
- Partition tables by date for cost optimization
- Use streaming inserts for real-time data ingestion
- Create scheduled queries for regular reports

## Study Tips

### Hands-On Practice
1. **Cloud Storage**: Create buckets, upload objects, configure lifecycle policies
2. **Cloud SQL**: Create instances, databases, configure backups and replicas
3. **Firestore**: Create collections and documents, write security rules
4. **Bigtable**: Create instances, design tables, load sample data
5. **BigQuery**: Create datasets and tables, run queries, use partitioning

### Key Concepts to Master
1. **Storage class selection**: When to use each Cloud Storage class
2. **Database selection**: Choosing between SQL, Firestore, Bigtable, BigQuery
3. **Backup and recovery**: Understanding options for each service
4. **Scaling strategies**: Read replicas, sharding, partitioning
5. **Cost optimization**: Storage classes, lifecycle management, query optimization

### Common Exam Topics
1. Choosing appropriate storage service for use case
2. Configuring Cloud Storage lifecycle policies
3. Setting up Cloud SQL high availability and backups
4. Understanding Firestore security rules and data model
5. Designing Bigtable schema for performance
6. Optimizing BigQuery queries and costs
7. Implementing data migration strategies

### gsutil and gcloud Command Examples

```bash
# Cloud Storage (gsutil)
gsutil mb gs://my-bucket-name
gsutil cp local-file.txt gs://my-bucket-name/
gsutil ls gs://my-bucket-name/
gsutil rm gs://my-bucket-name/file.txt
gsutil rsync -r local-dir gs://my-bucket-name/remote-dir

# Set lifecycle policy
gsutil lifecycle set lifecycle.json gs://my-bucket-name/

# Enable versioning
gsutil versioning set on gs://my-bucket-name/

# Cloud SQL
gcloud sql instances create my-instance --database-version=MYSQL_8_0 --tier=db-n1-standard-1 --region=us-central1
gcloud sql databases create my-database --instance=my-instance
gcloud sql backups create --instance=my-instance
gcloud sql instances describe my-instance

# Create read replica
gcloud sql instances create my-replica --master-instance-name=my-instance --tier=db-n1-standard-1

# Cloud Firestore
gcloud firestore databases create --location=us-east1
gcloud firestore indexes composite create --collection-group=myCollection --field-config=field1,order=ASCENDING

# Cloud Bigtable
gcloud bigtable instances create my-instance --cluster=my-cluster --cluster-zone=us-central1-a --display-name="My Instance"
gcloud bigtable clusters create my-cluster-2 --instance=my-instance --zone=us-east1-b

# BigQuery (bq)
bq mk my_dataset
bq mk --table my_dataset.my_table schema.json
bq query --use_legacy_sql=false 'SELECT * FROM my_dataset.my_table LIMIT 10'
bq load --source_format=CSV my_dataset.my_table gs://my-bucket/data.csv

# Create partitioned table
bq mk --table --time_partitioning_field=timestamp my_dataset.partitioned_table schema.json
```

## Additional Resources

- [Cloud Storage Documentation](https://cloud.google.com/storage/docs)
- [Cloud SQL Documentation](https://cloud.google.com/sql/docs)
- [Cloud Firestore Documentation](https://cloud.google.com/firestore/docs)
- [Cloud Bigtable Documentation](https://cloud.google.com/bigtable/docs)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Storage Options Guide](https://cloud.google.com/products/storage)
