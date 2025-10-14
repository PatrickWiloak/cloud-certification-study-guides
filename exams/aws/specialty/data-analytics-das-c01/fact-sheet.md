# AWS Data Analytics Specialty (DAS-C01) - Fact Sheet

## Quick Reference

**Exam Code:** DAS-C01
**Duration:** 180 minutes
**Questions:** 65 questions
**Passing Score:** 750/1000
**Cost:** $300 USD
**Validity:** 3 years
**Difficulty:** ⭐⭐⭐⭐⭐

## Exam Domains

| Domain | Weight | Key Focus |
|--------|--------|-----------|
| Collection | 18% | Kinesis, IoT, DMS, data ingestion |
| Storage & Data Management | 22% | S3, Lake Formation, data lakes |
| Processing | 24% | Lambda, Glue, EMR, batch/stream processing |
| Analysis & Visualization | 18% | Athena, Redshift, QuickSight |
| Security | 18% | Encryption, IAM, data governance |

## Data Collection Services

### Amazon Kinesis

**Kinesis Data Streams** - Real-time data streaming
- Shard-based architecture: 1 MB/sec or 1,000 records/sec write, 2 MB/sec read per shard
- Retention: 24 hours (default) to 365 days
- Consumers: Kinesis Client Library (KCL), Lambda, Kinesis Data Analytics
- **[📖 Kinesis Data Streams Concepts](https://docs.aws.amazon.com/streams/latest/dev/key-concepts.html)** - Shards, records, producers, consumers
- **[📖 Shard Management](https://docs.aws.amazon.com/streams/latest/dev/kinesis-using-sdk-java-resharding.html)** - Splitting and merging shards
- **[📖 Kinesis Producers](https://docs.aws.amazon.com/streams/latest/dev/building-producers.html)** - SDK, KPL, Kinesis Agent

**Kinesis Data Firehose** - Load streaming data to destinations
- Near real-time (60 sec minimum buffer)
- Destinations: S3, Redshift, OpenSearch, HTTP endpoints, Datadog, Splunk
- Transformation: Lambda functions for data transformation
- Compression: GZIP, ZIP, Snappy for S3
- No shard management (fully managed)
- **[📖 Firehose Concepts](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html)** - Delivery streams, buffering
- **[📖 Data Transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)** - Lambda integration
- **[📖 Delivery to S3](https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html#create-destination-s3)** - Partitioning, compression

**Kinesis Data Analytics** - Real-time analytics on streaming data
- SQL or Apache Flink for processing
- Sources: Kinesis Data Streams, Firehose
- Sinks: Kinesis Data Streams, Firehose, Lambda
- **[📖 Kinesis Analytics SQL](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/streaming-sql-concepts.html)** - Streaming SQL concepts
- **[📖 Apache Flink Applications](https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html)** - Java/Scala applications

### AWS Glue

**Glue Data Catalog** - Centralized metadata repository
- Stores table definitions, schema, location
- Integrated with Athena, Redshift Spectrum, EMR
- Crawlers: Automatic schema discovery from S3, JDBC, DynamoDB
- **[📖 Glue Data Catalog](https://docs.aws.amazon.com/glue/latest/dg/components-overview.html#data-catalog-intro)** - Databases, tables, connections
- **[📖 Crawlers](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html)** - Automatic schema detection
- **[📖 Partitions](https://docs.aws.amazon.com/glue/latest/dg/tables-described.html#tables-partition)** - Partition pruning for performance

**Glue ETL** - Serverless ETL service
- PySpark or Scala for transformations
- Dynamic frames: Extension of Spark DataFrames
- Job bookmarks: Track processed data to avoid reprocessing
- Development endpoints for testing
- **[📖 Glue ETL Concepts](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-intro.html)** - Jobs, triggers, workflows
- **[📖 DynamicFrames](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-api-crawler-pyspark-extensions-dynamic-frame.html)** - Schema flexibility
- **[📖 Job Bookmarks](https://docs.aws.amazon.com/glue/latest/dg/monitor-continuations.html)** - Incremental processing
- **[📖 Glue Studio](https://docs.aws.amazon.com/glue/latest/ug/what-is-glue-studio.html)** - Visual ETL designer

**Glue DataBrew** - Visual data preparation
- 250+ built-in transformations
- Profile data quality
- No code data cleaning
- **[📖 DataBrew Concepts](https://docs.aws.amazon.com/databrew/latest/dg/what-is.html)** - Datasets, recipes, jobs

### Other Collection Services

**AWS Database Migration Service (DMS)**
- Continuous replication for data pipelines
- CDC (Change Data Capture) for real-time updates
- Supports 20+ source and target databases
- **[📖 DMS for Analytics](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Introduction.html)** - Streaming data to S3, Kinesis
- **[📖 DMS CDC](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Task.CDC.html)** - Change data capture
- **[📖 DMS Targets](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.html)** - S3, Kinesis, Redshift

**AWS IoT Core**
- Device connectivity and message broker
- Rules engine to route messages to Kinesis, Lambda, S3
- **[📖 IoT Core Rules](https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html)** - SQL-based message routing
- **[📖 IoT Core Analytics](https://docs.aws.amazon.com/iot/latest/developerguide/iot-analytics.html)** - IoT data processing
- **[📖 Device Shadows](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html)** - Device state

**Amazon MSK** - Managed Apache Kafka
- Fully managed Kafka clusters
- MSK Connect for Kafka Connect connectors
- MSK Serverless for automatic scaling
- **[📖 MSK Documentation](https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html)** - Brokers, topics, partitions
- **[📖 MSK Connect](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect.html)** - Kafka connectors
- **[📖 MSK Serverless](https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html)** - Auto-scaling Kafka

## Storage & Data Management

### Amazon S3

**S3 for Data Lakes**
- Standard storage class for hot data
- Intelligent-Tiering for cost optimization
- Glacier for archival
- Versioning for data protection
- **[📖 S3 for Data Lakes](https://docs.aws.amazon.com/whitepapers/latest/building-data-lakes/building-data-lake-aws.html)** - Data lake architecture
- **[📖 S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)** - Performance and cost trade-offs
- **[📖 S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)** - Transition rules
- **[📖 S3 Inventory](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-inventory.html)** - Audit and report

**S3 Partitioning**
- Organize data by date, region, type for query performance
- Hive-style partitions: `s3://bucket/year=2023/month=10/day=12/`
- Partition projection in Athena for dynamic partitions
- **[📖 S3 Key Design](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html)** - Request performance
- **[📖 Partition Projection](https://docs.aws.amazon.com/athena/latest/ug/partition-projection.html)** - Query optimization

**S3 Replication**
- Cross-Region Replication (CRR) for DR
- Same-Region Replication (SRR) for compliance
- **[📖 S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)** - Configuration and use cases
- **[📖 Replication Time Control](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-time-control.html)** - Predictable replication

### AWS Lake Formation

**Centralized Data Lake Management**
- Fine-grained access control (column/row/cell-level)
- Blueprints for data ingestion
- Data catalog integration
- Governed tables for ACID transactions
- **[📖 Lake Formation Concepts](https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html)** - Data lakes, permissions
- **[📖 Security](https://docs.aws.amazon.com/lake-formation/latest/dg/lake-formation-permissions.html)** - Tag-based access control (TBAC)
- **[📖 Governed Tables](https://docs.aws.amazon.com/lake-formation/latest/dg/governed-tables.html)** - ACID transactions on S3
- **[📖 Blueprints](https://docs.aws.amazon.com/lake-formation/latest/dg/workflows-about.html)** - Automated data ingestion
- **[📖 Data Filters](https://docs.aws.amazon.com/lake-formation/latest/dg/data-filters-about.html)** - Row and cell-level security

## Processing Services

### AWS Lambda

**Event-Driven Processing**
- Trigger from S3, Kinesis, DynamoDB Streams, SQS
- 15-minute max execution time
- 10 GB memory max
- **[📖 Lambda Event Sources](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html)** - Stream processing
- **[📖 Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)** - Shared libraries
- **[📖 Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)** - Reserved and provisioned

### Amazon EMR

**Managed Hadoop/Spark Clusters**
- Frameworks: Hadoop, Spark, Hive, Presto, HBase, Flink
- Cluster types: Transient (terminate after job) or long-running
- Instance types: Master, Core (HDFS), Task (compute only)
- **[📖 EMR Overview](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-what-is-emr.html)** - Architecture
- **[📖 EMR on EKS](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/emr-eks.html)** - Run on Kubernetes
- **[📖 EMR Serverless](https://docs.aws.amazon.com/emr/latest/EMR-Serverless-UserGuide/emr-serverless.html)** - No cluster management
- **[📖 Spot Instances](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-instance-purchasing-options.html)** - Cost optimization

**EMR File Systems**
- EMRFS: Direct S3 access with consistent view
- HDFS: Local storage on cluster
- **[📖 EMRFS](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-fs.html)** - S3 integration
- **[📖 HDFS Configuration](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hdfs-config.html)** - HDFS on EMR

### AWS Batch

**Managed Batch Processing**
- Docker containers for batch jobs
- Job queues and compute environments
- Job dependencies and scheduling
- **[📖 Batch Concepts](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html)** - Jobs, queues, compute environments
- **[📖 Job Definitions](https://docs.aws.amazon.com/batch/latest/userguide/job_definitions.html)** - Container properties
- **[📖 Array Jobs](https://docs.aws.amazon.com/batch/latest/userguide/array_jobs.html)** - Parallel processing

### AWS Step Functions

**Workflow Orchestration**
- Coordinate Lambda, Batch, EMR, Glue, SageMaker
- Standard (up to 1 year) or Express (up to 5 minutes) workflows
- Error handling with retry and catch
- **[📖 Step Functions for Data Pipelines](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-standard-vs-express.html)** - Workflow types
- **[📖 Service Integrations](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-service-integrations.html)** - AWS service integration
- **[📖 Error Handling](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html)** - Retry and catch patterns

## Analysis & Visualization

### Amazon Athena

**Serverless SQL Queries on S3**
- Presto-based SQL engine
- Pay per query ($ 5 per TB scanned)
- File formats: Parquet, ORC, JSON, CSV, Avro
- Partitioning for cost/performance optimization
- Federated queries to RDS, DynamoDB, Redshift
- **[📖 Athena Concepts](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)** - Databases, tables, queries
- **[📖 Performance Tuning](https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html)** - Partitioning, compression, columnar formats
- **[📖 Workgroups](https://docs.aws.amazon.com/athena/latest/ug/workgroups.html)** - Query isolation and cost control
- **[📖 Federated Query](https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source.html)** - Query across data sources

**Optimization**
- Use Parquet/ORC for 10x+ performance vs CSV
- Compress data (GZIP, Snappy)
- Partition by frequently filtered columns
- Use columnar formats for selective column reads
- **[📖 Columnar Formats](https://docs.aws.amazon.com/athena/latest/ug/columnar-storage.html)** - Parquet vs ORC
- **[📖 Compression Support](https://docs.aws.amazon.com/athena/latest/ug/compression-support.html)** - Supported formats
- **[📖 Query Optimization](https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html)** - Best practices

### Amazon Redshift

**Petabyte-Scale Data Warehouse**
- Columnar storage, MPP architecture
- Node types: RA3 (managed storage), DC2 (compute-dense), DS2 (storage-dense)
- Distribution styles: KEY, EVEN, ALL, AUTO
- Sort keys: Compound, interleaved
- **[📖 Redshift Architecture](https://docs.aws.amazon.com/redshift/latest/dg/c_high_level_system_architecture.html)** - Leader and compute nodes
- **[📖 Distribution Styles](https://docs.aws.amazon.com/redshift/latest/dg/c_choosing_dist_sort.html)** - KEY, EVEN, ALL
- **[📖 Sort Keys](https://docs.aws.amazon.com/redshift/latest/dg/t_Sorting_data.html)** - Compound vs interleaved
- **[📖 Compression Encoding](https://docs.aws.amazon.com/redshift/latest/dg/c_Compression_encodings.html)** - Column compression

**Redshift Spectrum** - Query S3 directly from Redshift
- External tables in Glue Data Catalog
- Query S3 data without loading
- Scalable compute separate from Redshift cluster
- **[📖 Redshift Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html)** - External tables
- **[📖 Spectrum Best Practices](https://docs.aws.amazon.com/redshift/latest/dg/c-spectrum-external-performance.html)** - Performance optimization

**Concurrency Scaling**
- Automatic scaling for burst query traffic
- Up to 10 clusters (default)
- First hour free per day
- **[📖 Concurrency Scaling](https://docs.aws.amazon.com/redshift/latest/dg/concurrency-scaling.html)** - Automatic capacity
- **[📖 WLM Configuration](https://docs.aws.amazon.com/redshift/latest/dg/cm-c-defining-query-queues.html)** - Workload management

**Materialized Views**
- Pre-computed query results
- Auto-refresh or manual refresh
- Improves query performance
- **[📖 Materialized Views](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-overview.html)** - Creation and management
- **[📖 Auto Refresh](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-auto-mv.html)** - Automatic refresh

### Amazon QuickSight

**Business Intelligence & Visualization**
- SPICE: In-memory calculation engine
- Data sources: S3, Athena, Redshift, RDS, SaaS
- ML Insights: Anomaly detection, forecasting, auto-narratives
- Embedding: Dashboards in applications
- **[📖 QuickSight Concepts](https://docs.aws.amazon.com/quicksight/latest/user/welcome.html)** - Datasets, analyses, dashboards
- **[📖 SPICE](https://docs.aws.amazon.com/quicksight/latest/user/spice.html)** - In-memory engine
- **[📖 ML Insights](https://docs.aws.amazon.com/quicksight/latest/user/ml-insights.html)** - Anomaly detection
- **[📖 Embedding Dashboards](https://docs.aws.amazon.com/quicksight/latest/user/embedded-analytics.html)** - Embed in applications
- **[📖 Q for QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/quicksight-q.html)** - Natural language queries

### Amazon OpenSearch Service

**Search and Analytics Engine**
- Formerly Elasticsearch Service
- Log analytics, full-text search, application monitoring
- Kibana dashboards
- **[📖 OpenSearch Documentation](https://docs.aws.amazon.com/opensearch-service/)** - Domains, indices, shards
- **[📖 Index Management](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ism.html)** - Index lifecycle
- **[📖 Performance Tuning](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/performance-tuning.html)** - Optimization

## Security for Analytics

### Encryption

**At Rest**
- S3: SSE-S3, SSE-KMS, SSE-C
- Redshift: KMS encryption for cluster and snapshots
- Kinesis: KMS encryption for data streams
- Glue Data Catalog: KMS encryption
- **[📖 S3 Encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/serv-side-encryption.html)** - Server-side encryption options
- **[📖 KMS Key Policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)** - Grant access to services

**In Transit**
- TLS/SSL for all data transfers
- VPC endpoints for private connectivity
- **[📖 VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)** - S3, DynamoDB, Kinesis

### Access Control

**IAM Policies**
- Service-level access control
- Bucket policies for S3
- **[📖 S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)** - Examples

**Lake Formation Permissions**
- Fine-grained access control (column/row/cell)
- Tag-based access control (TBAC)
- Data filters for row-level security
- **[📖 Lake Formation Security](https://docs.aws.amazon.com/lake-formation/latest/dg/security-data-access.html)** - Permissions model

**Redshift Security**
- VPC for network isolation
- IAM roles for COPY/UNLOAD to S3
- Database users and groups
- **[📖 Redshift Security](https://docs.aws.amazon.com/redshift/latest/mgmt/iam-redshift-user-mgmt.html)** - IAM and database users

## Common Analytics Architectures

### Lambda Architecture
```
Batch Layer: S3 → Glue/EMR → Redshift (historical, accurate)
Speed Layer: Kinesis → Lambda/Kinesis Analytics → DynamoDB (real-time, approximate)
Serving Layer: Athena/Redshift/QuickSight (query both layers)
```
**[📖 Lambda Architecture](https://docs.aws.amazon.com/whitepapers/latest/build-modern-data-streaming-analytics-architectures/lambda-architecture.html)** - Batch and speed layers

### Modern Data Lake
```
Ingestion: Kinesis, DMS, IoT → S3 (raw zone)
Processing: Glue ETL → S3 (processed zone)
Catalog: Glue Data Catalog
Access Control: Lake Formation
Query: Athena, Redshift Spectrum
Visualization: QuickSight
```

### Real-Time Analytics Pipeline
```
IoT devices → IoT Core → Kinesis Data Streams
  → Kinesis Data Analytics (SQL/Flink)
  → Kinesis Data Firehose → S3/OpenSearch
  → QuickSight (real-time dashboards)
```

## Exam Tips

**Service Selection Matrix:**
- **Real-time streaming:** Kinesis Data Streams
- **Near real-time load:** Kinesis Data Firehose
- **Batch ETL:** Glue, EMR
- **Ad-hoc SQL on S3:** Athena
- **Data warehouse:** Redshift
- **BI dashboards:** QuickSight
- **Search/logs:** OpenSearch
- **Graph data:** Neptune

**Performance Keywords:**
- "Cost-effective" → Athena (pay per query), S3 Intelligent-Tiering, Spot for EMR
- "Real-time" → Kinesis Data Streams, Lambda
- "Petabyte-scale" → Redshift, EMR
- "Serverless" → Athena, Glue, Lambda, Firehose
- "ML insights" → QuickSight ML Insights, SageMaker

---

**Pro Tip:** This exam is all about choosing the RIGHT service for the use case. Understand data velocity (batch vs streaming), volume (GB vs PB), variety (structured vs unstructured), and cost constraints. Know when to use Glue vs EMR, Athena vs Redshift, Kinesis Streams vs Firehose!
