# Data and Analytics Services - Cloud Engineer

## BigQuery

### Overview
- Serverless data warehouse
- SQL queries on petabyte-scale data
- Columnar storage
- No infrastructure management

### Key Features
- Fast SQL queries (seconds on TB/PB data)
- Real-time analysis
- Built-in machine learning (BQML)
- Automatic scaling
- Pay per query or flat-rate pricing

### Datasets and Tables
**Dataset**: Container for tables
- Regional or multi-regional
- Access controls at dataset level

**Table Types**:
- Native tables: Standard BigQuery tables
- External tables: Query data in Cloud Storage, Bigtable, etc.
- Views: Saved queries
- Materialized views: Precomputed results

### Query Optimization
1. Use partitioned tables
2. Use clustered tables
3. Select only needed columns
4. Denormalize when appropriate
5. Use approximate aggregation functions
6. Cache results

### Pricing
**On-Demand**: $5 per TB scanned
**Flat-Rate**: Reserved slots for predictable costs
**Storage**: $0.02/GB active, $0.01/GB long-term

## Cloud Pub/Sub

### Overview
- Messaging service
- Asynchronous messaging between applications
- Real-time event streaming
- At-least-once delivery

### Core Concepts
**Topic**: Named resource for messages
**Subscription**: Named resource for consuming messages
**Message**: Data + attributes
**Publisher**: Sends messages to topic
**Subscriber**: Receives messages from subscription

### Subscription Types
**Pull**: Subscriber requests messages
**Push**: Pub/Sub sends messages to HTTPS endpoint

### Message Retention
- Default: 7 days
- Configurable: 10 minutes to 7 days
- Acknowledged messages removed immediately

### Use Cases
- Event ingestion
- Stream analytics
- Asynchronous workflows
- Real-time data integration

## Cloud Dataflow

### Overview
- Serverless stream and batch processing
- Based on Apache Beam
- Auto-scaling
- Unified programming model

### Features
- Real-time and batch processing
- Exactly-once processing
- Windowing and triggers
- State and timers
- Integration with BigQuery, Pub/Sub, etc.

### Use Cases
- ETL pipelines
- Real-time analytics
- Data enrichment
- ML feature engineering

## Cloud Dataproc

### Overview
- Managed Hadoop and Spark
- Quick cluster creation (90 seconds)
- Integrated with GCP services
- Pay per second

### Use Cases
- Migrate existing Spark/Hadoop workloads
- Batch processing
- Machine learning with Spark MLlib
- Data lake analytics

### Clusters
**Standard**: 1 master, N workers
**High Availability**: 3 masters, N workers
**Single Node**: 1 master, 0 workers (dev/test)

### Ephemeral vs Persistent
**Ephemeral**: Create for job, delete after
**Persistent**: Long-running cluster

## Cloud Composer

### Overview
- Managed Apache Airflow
- Workflow orchestration
- Schedule and monitor pipelines
- Python-based DAGs

### Use Cases
- Data pipeline orchestration
- Multi-cloud workflows
- ETL automation
- ML pipeline orchestration

## Exam Tips

### Service Selection
- **BigQuery**: Data warehouse, SQL analytics
- **Pub/Sub**: Asynchronous messaging, event ingestion
- **Dataflow**: Stream and batch processing (Apache Beam)
- **Dataproc**: Hadoop/Spark workloads
- **Composer**: Workflow orchestration (Airflow)

### Key Points
- BigQuery: Serverless, no infrastructure, pay per query
- Pub/Sub: At-least-once delivery, 7-day retention
- Partitioned tables: Reduce query costs in BigQuery
- Dataflow: Auto-scaling, unified batch/stream
- Dataproc: Managed Hadoop/Spark clusters

### Commands
```bash
# Create BigQuery dataset
bq mk --dataset PROJECT_ID:DATASET_NAME

# Query BigQuery
bq query --use_legacy_sql=false 'SELECT * FROM dataset.table LIMIT 10'

# Create Pub/Sub topic
gcloud pubsub topics create TOPIC_NAME

# Publish message
gcloud pubsub topics publish TOPIC_NAME --message="Hello"

# Create subscription
gcloud pubsub subscriptions create SUB_NAME --topic=TOPIC_NAME
```
