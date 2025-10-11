# Data Processing Systems - GCP Professional Data Engineer

## Overview

Design and implementation of data processing systems including storage selection, pipeline design, ETL/ELT processes, and data processing architecture for the Professional Data Engineer certification. This guide provides comprehensive coverage of GCP data processing services with production-ready code examples, optimization patterns, and exam-focused scenarios.

## Key Topics

1. **Storage Technologies** - BigQuery, Cloud Storage, Cloud SQL, Spanner, Bigtable, Firestore
2. **Data Pipeline Design** - Batch, streaming, lambda architecture, ETL/ELT, CDC patterns
3. **Data Processing Solutions** - Dataflow, Dataproc, BigQuery, Cloud Functions, Data Fusion
4. **Orchestration** - Cloud Composer (Airflow), Workflows, Scheduler
5. **Migration Strategies** - Data warehouse migration, database migration, data transfer
6. **Real-time Processing** - Pub/Sub, Dataflow streaming, BigQuery streaming inserts
7. **Optimization** - Query tuning, resource allocation, cost management, performance benchmarking

## Exam Tips for Data Processing

**Critical Exam Topics**:
- Service selection based on requirements (latency, cost, scale)
- BigQuery optimization (partitioning, clustering, slots)
- Dataflow pipeline design (windowing, triggers, watermarks)
- Streaming vs batch processing patterns
- Data migration strategies and tools
- Cost optimization across services
- Failure handling and data quality validation

**Common Exam Scenarios**:
1. Real-time analytics pipeline with late data handling
2. Cost optimization for BigQuery workloads
3. Migration from on-premises Hadoop to GCP
4. Streaming data ingestion with ordering requirements
5. Complex workflow orchestration with dependencies
6. Data lake to data warehouse ETL patterns
7. Multi-region data processing with consistency requirements
8. Schema evolution and backward compatibility

## Storage Technology Selection

### BigQuery - Comprehensive Deep Dive

#### Overview
**Use Cases**: Data warehouse, analytics, ML, BI reporting, log analytics
**Strengths**: Serverless, petabyte-scale, SQL interface, fast analytics, built-in ML
**Considerations**: Cost (storage + queries), partitioning/clustering for optimization
**Best For**: OLAP workloads, ad-hoc analysis, data science, interactive analytics

#### Partitioning Strategies

**Time-Ingestion Partitioning**:
```sql
-- Create table with ingestion-time partitioning
CREATE TABLE dataset.events_by_ingestion
(
  user_id STRING,
  event_type STRING,
  event_data JSON
)
PARTITION BY _PARTITIONDATE
OPTIONS(
  partition_expiration_days=90,
  require_partition_filter=true
);

-- Query with partition filter (reduced cost)
SELECT user_id, COUNT(*) as event_count
FROM dataset.events_by_ingestion
WHERE _PARTITIONDATE BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY user_id;
```

**Field-Based Partitioning**:
```sql
-- Partition by DATE field
CREATE TABLE dataset.events_by_date
(
  event_timestamp TIMESTAMP,
  event_date DATE,
  user_id STRING,
  event_type STRING,
  event_data JSON
)
PARTITION BY event_date
OPTIONS(
  partition_expiration_days=365,
  require_partition_filter=true
);

-- Partition by TIMESTAMP field (hourly)
CREATE TABLE dataset.events_by_hour
(
  event_timestamp TIMESTAMP,
  user_id STRING,
  event_type STRING
)
PARTITION BY TIMESTAMP_TRUNC(event_timestamp, HOUR)
OPTIONS(
  partition_expiration_days=7,
  require_partition_filter=true
);

-- Partition by INTEGER field (for range partitioning)
CREATE TABLE dataset.events_by_id
(
  event_id INT64,
  event_data JSON
)
PARTITION BY RANGE_BUCKET(event_id, GENERATE_ARRAY(0, 1000000, 10000))
OPTIONS(
  require_partition_filter=true
);
```

**Partition Expiration and Management**:
```sql
-- Set partition expiration
ALTER TABLE dataset.events_by_date
SET OPTIONS (partition_expiration_days=90);

-- Delete specific partitions
DELETE FROM dataset.events_by_date
WHERE event_date = '2023-01-01';

-- Query partition metadata
SELECT
  partition_id,
  total_rows,
  total_logical_bytes,
  last_modified_time
FROM `dataset.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'events_by_date'
ORDER BY partition_id DESC;
```

#### Clustering

**Clustering Best Practices**:
```sql
-- Create clustered table (up to 4 clustering columns)
CREATE TABLE dataset.events_clustered
(
  event_timestamp TIMESTAMP,
  event_date DATE,
  user_id STRING,
  country STRING,
  event_type STRING,
  event_data JSON
)
PARTITION BY event_date
CLUSTER BY country, event_type, user_id
OPTIONS(
  require_partition_filter=true
);

-- Query benefits from clustering (automatic pruning)
SELECT user_id, COUNT(*) as events
FROM dataset.events_clustered
WHERE event_date = '2024-01-15'
  AND country = 'US'
  AND event_type = 'purchase'
GROUP BY user_id;

-- Check clustering quality
SELECT
  table_name,
  clustering_ordinal_position,
  clustering_field_path
FROM `dataset.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'events_clustered'
  AND clustering_ordinal_position IS NOT NULL
ORDER BY clustering_ordinal_position;
```

**Clustering vs Partitioning Decision Matrix**:
- **Partitioning**: Required for time-based data lifecycle, enables partition expiration
- **Clustering**: Improves query performance on specific columns, automatic optimization
- **Both**: Use partitioning for time-based data, clustering for frequently filtered columns
- **Neither**: Small tables (<1GB) don't benefit significantly

#### Materialized Views

**Creating and Managing Materialized Views**:
```sql
-- Create materialized view for aggregated data
CREATE MATERIALIZED VIEW dataset.daily_user_metrics
PARTITION BY event_date
CLUSTER BY country, user_segment
AS
SELECT
  event_date,
  country,
  user_segment,
  COUNT(DISTINCT user_id) as unique_users,
  SUM(revenue) as total_revenue,
  AVG(session_duration) as avg_session_duration,
  COUNT(*) as total_events
FROM dataset.events_clustered
GROUP BY event_date, country, user_segment;

-- Query automatically uses materialized view
SELECT country, SUM(total_revenue) as revenue
FROM dataset.daily_user_metrics
WHERE event_date BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY country;

-- Check materialized view status
SELECT
  table_name,
  materialized_view_status,
  last_refresh_time,
  refresh_watermark
FROM `dataset.INFORMATION_SCHEMA.MATERIALIZED_VIEWS`
WHERE table_name = 'daily_user_metrics';

-- Manual refresh (automatic by default)
CALL BQ.REFRESH_MATERIALIZED_VIEW('dataset.daily_user_metrics');

-- Materialized view with enable_refresh option
CREATE MATERIALIZED VIEW dataset.hourly_metrics
OPTIONS(
  enable_refresh = true,
  refresh_interval_minutes = 30
)
AS
SELECT
  TIMESTAMP_TRUNC(event_timestamp, HOUR) as hour,
  event_type,
  COUNT(*) as event_count
FROM dataset.events_by_hour
GROUP BY hour, event_type;
```

#### BI Engine

**BI Engine Configuration**:
```sql
-- Create BI Engine reservation (requires billing project)
-- Done via console or API, not SQL

-- Query with BI Engine acceleration (automatic)
SELECT
  country,
  product_category,
  SUM(revenue) as total_revenue,
  COUNT(DISTINCT user_id) as unique_customers
FROM dataset.events_clustered
WHERE event_date >= CURRENT_DATE() - 7
GROUP BY country, product_category
ORDER BY total_revenue DESC;

-- Check BI Engine usage
SELECT
  query_info.query_hashes,
  bi_engine_statistics.bi_engine_mode,
  bi_engine_statistics.bi_engine_reasons,
  total_bytes_processed,
  total_slot_ms
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE DATE(creation_time) = CURRENT_DATE()
  AND bi_engine_statistics.bi_engine_mode IN ('FULL', 'PARTIAL')
ORDER BY creation_time DESC
LIMIT 10;
```

**BI Engine Best Practices**:
- Reserve capacity for frequently accessed datasets
- Works best with queries under 100GB
- Automatic acceleration for repeated queries
- Monitor acceleration ratio in query execution details
- Use with dashboards and reports for best ROI

#### Slots and Reservations

**Understanding Slots**:
- Default: On-demand slots (2000 slots shared across project)
- Flat-rate: Reserved slots for predictable costs
- Flex slots: Committed for 60 seconds minimum
- Autoscaling: Available with Enterprise/Enterprise Plus

**Creating Reservations**:
```sql
-- Create reservation (via gcloud)
gcloud beta bq reservations create prod-reservation \
  --location=us-central1 \
  --slots=500 \
  --ignore-idle-slots=false

-- Create assignment
gcloud beta bq reservations assignments create \
  --reservation=prod-reservation \
  --job-type=QUERY \
  --assignee-type=PROJECT \
  --assignee-id=my-project

-- Monitor slot usage
SELECT
  reservation_name,
  SUM(total_slot_ms) / (1000 * 60 * 60) as slot_hours,
  AVG(total_slot_ms / (end_time - start_time)) as avg_slots
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE DATE(creation_time) = CURRENT_DATE()
  AND reservation_name IS NOT NULL
GROUP BY reservation_name;
```

**Slot Optimization**:
```sql
-- Identify slot-intensive queries
SELECT
  user_email,
  query,
  total_slot_ms,
  total_bytes_processed,
  total_slot_ms / NULLIF(total_bytes_processed, 0) as slots_per_byte
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE DATE(creation_time) = CURRENT_DATE()
  AND job_type = 'QUERY'
  AND state = 'DONE'
ORDER BY total_slot_ms DESC
LIMIT 20;
```

#### Query Optimization

**Query Optimization Patterns**:
```sql
-- BAD: SELECT * scans all columns
SELECT * FROM dataset.large_table WHERE date = '2024-01-01';

-- GOOD: Select only needed columns
SELECT user_id, event_type, revenue
FROM dataset.large_table
WHERE date = '2024-01-01';

-- BAD: Self-join for aggregation
SELECT a.user_id, b.total_events
FROM dataset.events a
JOIN (
  SELECT user_id, COUNT(*) as total_events
  FROM dataset.events
  GROUP BY user_id
) b ON a.user_id = b.user_id;

-- GOOD: Use window functions
SELECT
  user_id,
  event_type,
  COUNT(*) OVER (PARTITION BY user_id) as total_events
FROM dataset.events;

-- Use APPROX functions for large datasets
SELECT
  country,
  APPROX_COUNT_DISTINCT(user_id) as unique_users,
  APPROX_QUANTILES(revenue, 100)[OFFSET(50)] as median_revenue,
  APPROX_TOP_COUNT(product, 10) as top_products
FROM dataset.events
GROUP BY country;

-- Optimize JOINs with partitioning
SELECT
  e.user_id,
  e.event_type,
  u.user_segment
FROM dataset.events_partitioned e
JOIN dataset.users u ON e.user_id = u.user_id
WHERE e.event_date BETWEEN '2024-01-01' AND '2024-01-31'  -- Partition filter
  AND e.country = 'US';  -- Clustering filter

-- Use ARRAY aggregation instead of GROUP BY with large cardinality
SELECT
  country,
  ARRAY_AGG(STRUCT(user_id, revenue) ORDER BY revenue DESC LIMIT 100) as top_users
FROM dataset.events
WHERE event_date = CURRENT_DATE()
GROUP BY country;

-- Pre-filter before joining
WITH active_users AS (
  SELECT DISTINCT user_id
  FROM dataset.events
  WHERE event_date = CURRENT_DATE()
    AND event_type = 'purchase'
)
SELECT
  u.user_id,
  u.user_name,
  u.user_segment
FROM dataset.users u
JOIN active_users a ON u.user_id = a.user_id;
```

**Query Execution Analysis**:
```sql
-- Analyze query performance
SELECT
  query,
  total_bytes_processed,
  total_bytes_billed,
  total_slot_ms,
  (total_slot_ms / 1000) / TIMESTAMP_DIFF(end_time, start_time, SECOND) as avg_slots,
  cache_hit,
  TIMESTAMP_DIFF(end_time, start_time, SECOND) as duration_seconds
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND job_type = 'QUERY'
  AND state = 'DONE'
ORDER BY total_slot_ms DESC
LIMIT 50;

-- Check for skewed data in joins
SELECT
  _stage_id,
  _status,
  COUNT(*) as num_workers,
  AVG(_input_rows) as avg_input_rows,
  STDDEV(_input_rows) as stddev_input_rows,
  MAX(_input_rows) / NULLIF(AVG(_input_rows), 0) as skew_ratio
FROM `region-us`.INFORMATION_SCHEMA.JOBS_TIMELINE_BY_PROJECT
WHERE job_id = 'your-job-id'
GROUP BY _stage_id, _status
HAVING skew_ratio > 2.0;
```

#### Streaming Inserts

**Streaming API Usage**:
```python
from google.cloud import bigquery

# Initialize client
client = bigquery.Client()
table_id = "project.dataset.events_streaming"

# Single row insert
rows_to_insert = [
    {"user_id": "user123", "event_type": "click", "timestamp": "2024-01-15T10:30:00"},
]
errors = client.insert_rows_json(table_id, rows_to_insert)

# Batch streaming insert with deduplication
rows_to_insert = []
for i in range(1000):
    rows_to_insert.append({
        "insertId": f"event_{i}_{int(time.time())}",  # Deduplication key
        "user_id": f"user{i}",
        "event_type": "page_view",
        "timestamp": datetime.utcnow().isoformat()
    })

errors = client.insert_rows_json(
    table_id,
    rows_to_insert,
    row_ids=[row["insertId"] for row in rows_to_insert]
)

if errors:
    print(f"Errors occurred: {errors}")
```

**Streaming Buffer Query**:
```sql
-- Query streaming buffer (immediately available, not optimized)
SELECT *
FROM dataset.events_streaming
WHERE _PARTITIONTIME IS NULL  -- Data in streaming buffer
LIMIT 100;

-- Query committed data only
SELECT *
FROM dataset.events_streaming
WHERE _PARTITIONTIME IS NOT NULL
  AND event_date = CURRENT_DATE();
```

**Streaming Best Practices**:
- Use insertId for deduplication (best-effort within 1 minute)
- Batch streaming inserts (up to 10,000 rows per request)
- Streaming buffer commits within minutes
- Streaming data immediately queryable but not clustered
- Consider Dataflow for high-volume streaming (better performance)
- Monitor streaming insert quotas (100k rows/sec per table)

### Cloud Storage
**Use Cases**: Data lake, raw data storage, backups, ML training data
**Strengths**: Unlimited scale, durability, lifecycle management, multiple classes
**Considerations**: Access patterns determine storage class
**Best For**: Unstructured data, data lake landing zone, archival

### Cloud SQL
**Use Cases**: Transactional databases, legacy application migration
**Strengths**: Managed MySQL/PostgreSQL/SQL Server, familiar interface
**Considerations**: Size limits, regional, read replicas for scaling
**Best For**: OLTP workloads, traditional relational needs

### Cloud Spanner
**Use Cases**: Global transactions, mission-critical OLTP
**Strengths**: Horizontally scalable, globally consistent, SQL interface
**Considerations**: Cost, complexity, overkill for simple use cases
**Best For**: Global applications requiring strong consistency

### Cloud Bigtable
**Use Cases**: Time-series, IoT, analytics, large-scale NoSQL
**Strengths**: Low latency, high throughput, HBase API compatible
**Considerations**: Schema design critical, not for small datasets
**Best For**: Time-series data, real-time analytics, IoT sensor data

### Firestore
**Use Cases**: Mobile/web apps, real-time sync, document storage
**Strengths**: Real-time updates, offline support, auto-scaling
**Considerations**: Document-oriented, query limitations
**Best For**: User-generated content, real-time applications

## Data Pipeline Design

### Batch Processing
**Characteristics**: Process data in defined intervals (hourly, daily)
**Tools**: Dataproc (Spark/Hadoop), Dataflow (Apache Beam), BigQuery scheduled queries
**Use Cases**: Daily reports, ETL jobs, data warehousing
**Patterns**: Extract → Transform → Load (ETL) or Extract → Load → Transform (ELT)

**Example Architecture**:
```
Cloud Storage (raw data) → Dataflow (transform) → BigQuery (warehouse)
         ↓
    Cloud Composer (orchestration)
```

### Stream Processing
**Characteristics**: Process data in real-time as it arrives
**Tools**: Dataflow (streaming), Pub/Sub (ingestion), BigQuery streaming inserts
**Use Cases**: Real-time analytics, fraud detection, monitoring
**Patterns**: Pub/Sub → Dataflow → BigQuery/Bigtable

**Example Architecture**:
```
Data Sources → Pub/Sub → Dataflow → BigQuery/Bigtable
                              ↓
                         Cloud Storage (archive)
```

### Lambda Architecture
**Batch Layer**: Historical data processing, complete views
**Speed Layer**: Real-time processing, recent data
**Serving Layer**: Merge results from both layers
**Use Cases**: Combining real-time and historical analytics

### Kappa Architecture
**Stream-only**: All data as streams, no separate batch layer
**Reprocessing**: Replay streams for corrections
**Simpler**: Single processing paradigm
**Use Cases**: Pure event-driven systems

## Data Processing Tools

### Dataflow (Apache Beam) - Comprehensive Deep Dive

#### Overview
**Strengths**: Unified batch and streaming, fully managed, auto-scaling, exactly-once processing
**Use Cases**: ETL, real-time analytics, data enrichment, event processing, ML preprocessing
**Programming**: Java, Python, Go SDKs
**Execution**: Dataflow Runner (GCP), Direct Runner (local testing), Spark/Flink runners

#### Core Concepts

**PCollections**:
- Immutable distributed datasets
- Can be bounded (batch) or unbounded (streaming)
- Elements have implicit timestamps
- Support windowing and triggering

**Transforms**:
- ParDo: Parallel processing of each element
- GroupByKey: Group elements by key
- Combine: Aggregation operations
- Flatten: Merge multiple PCollections
- Partition: Split PCollection into multiple outputs

#### Complete Dataflow Pipeline Examples

**Batch Pipeline - ETL with Transformations**:
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from apache_beam.io.gcp.bigquery import WriteToBigQuery, BigQueryDisposition
import json

class ParseCSV(beam.DoFn):
    def process(self, element):
        """Parse CSV line and return dict"""
        try:
            parts = element.split(',')
            yield {
                'user_id': parts[0],
                'event_type': parts[1],
                'timestamp': parts[2],
                'revenue': float(parts[3]) if parts[3] else 0.0,
                'country': parts[4]
            }
        except Exception as e:
            # Send to error collection
            yield beam.pvalue.TaggedOutput('errors', {
                'raw': element,
                'error': str(e)
            })

class EnrichData(beam.DoFn):
    def __init__(self, project_id, dataset_id):
        self.project_id = project_id
        self.dataset_id = dataset_id
        self.user_cache = {}

    def setup(self):
        """Setup BigQuery client for side input lookups"""
        from google.cloud import bigquery
        self.client = bigquery.Client(project=self.project_id)

    def process(self, element, user_segments):
        """Enrich event with user segment from side input"""
        user_id = element['user_id']

        # Get user segment from side input dictionary
        user_segment = user_segments.get(user_id, 'unknown')

        element['user_segment'] = user_segment
        element['enriched_at'] = beam.utils.timestamp.Timestamp.now().to_utc_datetime().isoformat()

        yield element

class CalculateMetrics(beam.DoFn):
    def process(self, element):
        """Calculate derived metrics"""
        key, events = element  # (country, [events])

        total_revenue = sum(e['revenue'] for e in events)
        unique_users = len(set(e['user_id'] for e in events))
        event_count = len(events)

        yield {
            'country': key,
            'total_revenue': total_revenue,
            'unique_users': unique_users,
            'event_count': event_count,
            'avg_revenue_per_user': total_revenue / unique_users if unique_users > 0 else 0
        }

def run_batch_pipeline():
    """Complete batch ETL pipeline"""

    pipeline_options = PipelineOptions([
        '--project=my-project',
        '--region=us-central1',
        '--runner=DataflowRunner',
        '--temp_location=gs://my-bucket/temp',
        '--staging_location=gs://my-bucket/staging',
        '--num_workers=5',
        '--max_num_workers=10',
        '--autoscaling_algorithm=THROUGHPUT_BASED',
        '--service_account_email=dataflow-sa@my-project.iam.gserviceaccount.com'
    ])

    # BigQuery table schema
    schema = {
        'fields': [
            {'name': 'country', 'type': 'STRING'},
            {'name': 'total_revenue', 'type': 'FLOAT64'},
            {'name': 'unique_users', 'type': 'INTEGER'},
            {'name': 'event_count', 'type': 'INTEGER'},
            {'name': 'avg_revenue_per_user', 'type': 'FLOAT64'}
        ]
    }

    with beam.Pipeline(options=pipeline_options) as pipeline:

        # Read user segments as side input
        user_segments = (
            pipeline
            | 'Read User Segments' >> beam.io.ReadFromBigQuery(
                query='SELECT user_id, user_segment FROM `my-project.dataset.users`',
                use_standard_sql=True
            )
            | 'Format User Segments' >> beam.Map(lambda x: (x['user_id'], x['user_segment']))
            | 'To Dict' >> beam.combiners.ToDict()
        )

        # Main pipeline
        events = (
            pipeline
            | 'Read CSV' >> beam.io.ReadFromText('gs://my-bucket/events/*.csv', skip_header_lines=1)
            | 'Parse CSV' >> beam.ParDo(ParseCSV()).with_outputs('errors', main='events')
        )

        # Process valid events
        metrics = (
            events.events
            | 'Enrich Data' >> beam.ParDo(EnrichData('my-project', 'dataset'),
                                          user_segments=beam.pvalue.AsSingleton(user_segments))
            | 'Filter High Value' >> beam.Filter(lambda x: x['revenue'] > 0)
            | 'Key By Country' >> beam.Map(lambda x: (x['country'], x))
            | 'Group By Country' >> beam.GroupByKey()
            | 'Calculate Metrics' >> beam.ParDo(CalculateMetrics())
        )

        # Write to BigQuery
        metrics | 'Write to BigQuery' >> WriteToBigQuery(
            table='my-project:dataset.country_metrics',
            schema=schema,
            write_disposition=BigQueryDisposition.WRITE_TRUNCATE,
            create_disposition=BigQueryDisposition.CREATE_IF_NEEDED
        )

        # Write errors to separate table
        events.errors | 'Write Errors' >> beam.io.WriteToText(
            'gs://my-bucket/errors/batch',
            file_name_suffix='.json'
        )

if __name__ == '__main__':
    run_batch_pipeline()
```

#### Streaming Pipeline with Windowing

**Complete Streaming Pipeline**:
```python
import apache_beam as beam
from apache_beam import window
from apache_beam.transforms.trigger import AfterWatermark, AfterProcessingTime, AccumulationMode
from apache_beam.options.pipeline_options import PipelineOptions, StandardOptions
import json
from datetime import timedelta

class ParsePubSubMessage(beam.DoFn):
    def process(self, element):
        """Parse Pub/Sub message"""
        try:
            data = json.loads(element.decode('utf-8'))

            # Extract timestamp for event-time processing
            import apache_beam.transforms.window as window
            from datetime import datetime

            event_timestamp = datetime.fromisoformat(data['timestamp'])
            timestamp = window.TimestampedValue(data, event_timestamp.timestamp())

            yield timestamp
        except Exception as e:
            yield beam.pvalue.TaggedOutput('errors', {
                'raw': element.decode('utf-8'),
                'error': str(e)
            })

class SessionActivity(beam.DoFn):
    def process(self, element, window=beam.DoFn.WindowParam):
        """Calculate session-level metrics"""
        user_id, events = element

        # Window provides session boundaries
        session_start = window.start.to_utc_datetime()
        session_end = window.end.to_utc_datetime()

        session_duration = (session_end - session_start).total_seconds()
        event_count = len(list(events))

        yield {
            'user_id': user_id,
            'session_start': session_start.isoformat(),
            'session_end': session_end.isoformat(),
            'session_duration': session_duration,
            'event_count': event_count,
            'window_start': session_start.isoformat(),
            'window_end': session_end.isoformat()
        }

class AggregateMetrics(beam.CombineFn):
    """Custom CombineFn for complex aggregations"""

    def create_accumulator(self):
        return {
            'total_events': 0,
            'total_revenue': 0.0,
            'unique_users': set()
        }

    def add_input(self, accumulator, input_element):
        accumulator['total_events'] += 1
        accumulator['total_revenue'] += input_element.get('revenue', 0.0)
        accumulator['unique_users'].add(input_element['user_id'])
        return accumulator

    def merge_accumulators(self, accumulators):
        merged = self.create_accumulator()
        for acc in accumulators:
            merged['total_events'] += acc['total_events']
            merged['total_revenue'] += acc['total_revenue']
            merged['unique_users'].update(acc['unique_users'])
        return merged

    def extract_output(self, accumulator):
        return {
            'total_events': accumulator['total_events'],
            'total_revenue': accumulator['total_revenue'],
            'unique_users': len(accumulator['unique_users'])
        }

def run_streaming_pipeline():
    """Complete streaming pipeline with windowing and triggers"""

    pipeline_options = PipelineOptions([
        '--project=my-project',
        '--region=us-central1',
        '--runner=DataflowRunner',
        '--streaming',
        '--enable_streaming_engine',
        '--temp_location=gs://my-bucket/temp',
        '--staging_location=gs://my-bucket/staging',
        '--num_workers=3',
        '--max_num_workers=10',
        '--autoscaling_algorithm=THROUGHPUT_BASED',
        '--worker_machine_type=n1-standard-4'
    ])

    pipeline_options.view_as(StandardOptions).streaming = True

    with beam.Pipeline(options=pipeline_options) as pipeline:

        # Read from Pub/Sub
        messages = (
            pipeline
            | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(
                subscription='projects/my-project/subscriptions/events-sub',
                with_attributes=False
            )
            | 'Parse Messages' >> beam.ParDo(ParsePubSubMessage()).with_outputs('errors', main='events')
        )

        # Fixed-time windowing with triggers
        fixed_window_metrics = (
            messages.events
            | 'Fixed 1-min Windows' >> beam.WindowInto(
                window.FixedWindows(60),  # 1-minute windows
                trigger=AfterWatermark(
                    early=AfterProcessingTime(10),  # Early firing every 10 seconds
                    late=AfterProcessingTime(30)    # Late firing every 30 seconds
                ),
                allowed_lateness=300,  # Allow 5 minutes of late data
                accumulation_mode=AccumulationMode.ACCUMULATING
            )
            | 'Aggregate Fixed Windows' >> beam.CombineGlobally(
                AggregateMetrics()
            ).without_defaults()
            | 'Format Fixed Window Output' >> beam.Map(lambda x: {
                **x,
                'window_type': 'fixed',
                'window_size': 60
            })
        )

        # Sliding window for moving averages
        sliding_window_metrics = (
            messages.events
            | 'Sliding 5-min Windows' >> beam.WindowInto(
                window.SlidingWindows(300, 60),  # 5-min windows, every 1 minute
                trigger=AfterWatermark(),
                allowed_lateness=600
            )
            | 'Key By Country' >> beam.Map(lambda x: (x['country'], x))
            | 'Aggregate Sliding Windows' >> beam.CombinePerKey(AggregateMetrics())
            | 'Format Sliding Output' >> beam.Map(lambda x: {
                'country': x[0],
                **x[1],
                'window_type': 'sliding'
            })
        )

        # Session windows for user activity
        session_metrics = (
            messages.events
            | 'Session Windows' >> beam.WindowInto(
                window.Sessions(600),  # 10-minute session gap
                trigger=AfterWatermark(late=AfterProcessingTime(60)),
                allowed_lateness=1800
            )
            | 'Key By User' >> beam.Map(lambda x: (x['user_id'], x))
            | 'Group By User Session' >> beam.GroupByKey()
            | 'Calculate Session Metrics' >> beam.ParDo(SessionActivity())
        )

        # Write to BigQuery with streaming inserts
        fixed_window_metrics | 'Write Fixed Windows' >> beam.io.WriteToBigQuery(
            table='my-project:dataset.fixed_window_metrics',
            schema='AUTO',
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND,
            create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
            method='STREAMING_INSERTS'
        )

        sliding_window_metrics | 'Write Sliding Windows' >> beam.io.WriteToBigQuery(
            table='my-project:dataset.sliding_window_metrics',
            schema='AUTO',
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND,
            method='STREAMING_INSERTS'
        )

        session_metrics | 'Write Sessions' >> beam.io.WriteToBigQuery(
            table='my-project:dataset.session_metrics',
            schema='AUTO',
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND,
            method='STREAMING_INSERTS'
        )

        # Write errors to Pub/Sub dead letter topic
        messages.errors | 'Publish Errors' >> beam.io.WriteToPubSub(
            topic='projects/my-project/topics/events-dlq',
            with_attributes=False
        )

if __name__ == '__main__':
    run_streaming_pipeline()
```

#### Windowing Strategies

**Window Types**:

1. **Fixed Windows**: Tumbling windows with fixed duration
```python
# Non-overlapping 1-hour windows
beam.WindowInto(window.FixedWindows(3600))
```

2. **Sliding Windows**: Overlapping windows
```python
# 10-minute windows, sliding every 1 minute
beam.WindowInto(window.SlidingWindows(600, 60))
```

3. **Session Windows**: Activity-based windows
```python
# Session gap of 30 minutes
beam.WindowInto(window.Sessions(1800))
```

4. **Global Window**: Default, single window for all data
```python
# Requires explicit trigger
beam.WindowInto(
    window.GlobalWindows(),
    trigger=AfterProcessingTime(60)
)
```

#### Triggers and Watermarks

**Trigger Patterns**:
```python
# Fire after watermark passes window end
AfterWatermark()

# Fire after processing time delay
AfterProcessingTime(60)

# Fire after element count
AfterCount(1000)

# Composite triggers
AfterWatermark(
    early=AfterProcessingTime(30),  # Early results
    late=AfterCount(100)             # Late data processing
)

# Repeatedly fire
trigger=beam.trigger.Repeatedly(AfterProcessingTime(10))

# Fire once after condition
trigger=beam.trigger.AfterAny(
    AfterCount(1000),
    AfterProcessingTime(60)
)
```

**Watermark Handling**:
```python
# Configure allowed lateness
beam.WindowInto(
    window.FixedWindows(60),
    trigger=AfterWatermark(late=AfterProcessingTime(30)),
    allowed_lateness=300,  # Accept data up to 5 minutes late
    accumulation_mode=AccumulationMode.DISCARDING_FIRED_PANES
)

# Accumulation modes:
# - ACCUMULATING: Include all data in each firing
# - DISCARDING_FIRED_PANES: Only new data since last firing
```

#### Side Inputs and Side Outputs

**Side Inputs for Enrichment**:
```python
class EnrichWithReferenceData(beam.DoFn):
    def process(self, element, reference_data):
        # reference_data is available to all workers
        lookup_key = element['category_id']
        category_name = reference_data.get(lookup_key, 'Unknown')

        element['category_name'] = category_name
        yield element

# Create side input
reference_data = (
    pipeline
    | 'Read Reference' >> beam.io.ReadFromBigQuery(...)
    | 'To Dict' >> beam.Map(lambda x: (x['id'], x['name']))
    | beam.combiners.ToDict()
)

# Use side input
enriched = (
    events
    | 'Enrich' >> beam.ParDo(
        EnrichWithReferenceData(),
        reference_data=beam.pvalue.AsSingleton(reference_data)
    )
)
```

**Side Outputs for Multi-path Processing**:
```python
class ClassifyEvents(beam.DoFn):
    def process(self, element):
        if element['event_type'] == 'purchase':
            yield element
        elif element['event_type'] == 'error':
            yield beam.pvalue.TaggedOutput('errors', element)
        elif element['event_type'] == 'warning':
            yield beam.pvalue.TaggedOutput('warnings', element)
        else:
            yield beam.pvalue.TaggedOutput('other', element)

result = events | beam.ParDo(ClassifyEvents()).with_outputs('errors', 'warnings', 'other', main='purchases')

# Access different outputs
result.purchases | 'Process Purchases' >> ...
result.errors | 'Handle Errors' >> ...
result.warnings | 'Log Warnings' >> ...
result.other | 'Archive Other' >> ...
```

#### Dataflow Best Practices

**Performance Optimization**:
1. **Fusion optimization**: Beam fuses compatible transforms
2. **Combiner lifting**: Move aggregations earlier in pipeline
3. **Side input caching**: Small reference data loaded once per worker
4. **Batch operations**: Group API calls (BigQuery, Bigtable)
5. **Reshuffle for hotkeys**: Distribute skewed keys
```python
# Reshuffle to break fusion and redistribute
events | beam.Reshuffle() | 'Process' >> beam.ParDo(...)
```

**Resource Management**:
```python
pipeline_options = PipelineOptions([
    '--worker_machine_type=n1-standard-4',
    '--disk_size_gb=100',
    '--num_workers=10',
    '--max_num_workers=50',
    '--autoscaling_algorithm=THROUGHPUT_BASED',
    '--enable_streaming_engine',  # Separates compute from state
    '--worker_disk_type=pd-ssd'
])
```

**Error Handling**:
```python
class RobustTransform(beam.DoFn):
    def process(self, element):
        try:
            # Processing logic
            yield process_element(element)
        except Exception as e:
            # Log error and send to DLQ
            import logging
            logging.error(f"Error processing {element}: {e}")
            yield beam.pvalue.TaggedOutput('errors', {
                'element': element,
                'error': str(e),
                'timestamp': time.time()
            })
```

**Monitoring and Metrics**:
```python
class CountingTransform(beam.DoFn):
    def __init__(self):
        self.success_counter = beam.metrics.Metrics.counter('main', 'successful_transforms')
        self.error_counter = beam.metrics.Metrics.counter('main', 'failed_transforms')
        self.processing_time = beam.metrics.Metrics.distribution('main', 'processing_time_ms')

    def process(self, element):
        start_time = time.time()
        try:
            result = transform_element(element)
            self.success_counter.inc()
            yield result
        except Exception as e:
            self.error_counter.inc()
            raise
        finally:
            duration = (time.time() - start_time) * 1000
            self.processing_time.update(int(duration))
```

#### Dataflow Templates

**Creating Flex Templates**:
```python
# template_pipeline.py
from apache_beam.options.pipeline_options import PipelineOptions
import apache_beam as beam

class MyOptions(PipelineOptions):
    @classmethod
    def _add_argparse_args(cls, parser):
        parser.add_value_provider_argument(
            '--input_subscription',
            type=str,
            help='Pub/Sub subscription'
        )
        parser.add_value_provider_argument(
            '--output_table',
            type=str,
            help='BigQuery output table'
        )

def run():
    options = MyOptions()

    with beam.Pipeline(options=options) as pipeline:
        (pipeline
         | 'Read' >> beam.io.ReadFromPubSub(subscription=options.input_subscription)
         | 'Transform' >> beam.ParDo(TransformFn())
         | 'Write' >> beam.io.WriteToBigQuery(table=options.output_table))

if __name__ == '__main__':
    run()
```

**Build and Deploy Template**:
```bash
# Build Docker image
gcloud builds submit --tag gcr.io/my-project/dataflow/my-template:latest

# Create template spec
gcloud dataflow flex-template build gs://my-bucket/templates/my-template.json \
  --image gcr.io/my-project/dataflow/my-template:latest \
  --sdk-language PYTHON \
  --metadata-file metadata.json

# Run template
gcloud dataflow flex-template run my-job \
  --template-file-gcs-location gs://my-bucket/templates/my-template.json \
  --region us-central1 \
  --parameters input_subscription=projects/my-project/subscriptions/input-sub \
  --parameters output_table=my-project:dataset.output_table
```

### Pub/Sub for Data Engineering - Deep Dive

#### Overview
**Strengths**: Fully managed messaging, global scale, at-least-once delivery, push/pull
**Use Cases**: Event ingestion, microservices communication, streaming pipelines
**Integration**: Dataflow, Cloud Functions, Cloud Run, BigQuery subscriptions
**Performance**: Millions of messages/second, sub-second latency

#### Message Ordering

**Ordered Delivery with Ordering Keys**:
```python
from google.cloud import pubsub_v1
from google.cloud.pubsub_v1 import types

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path('my-project', 'events')

# Publish with ordering key (enables FIFO per key)
future = publisher.publish(
    topic_path,
    data=b'{"user_id": "user123", "event": "login"}',
    ordering_key='user123'  # Messages with same key delivered in order
)

print(f"Published message ID: {future.result()}")

# Enable message ordering on publisher
publisher = pubsub_v1.PublisherClient(
    publisher_options=types.PublisherOptions(enable_message_ordering=True)
)
```

**Subscription with Ordering**:
```python
from google.cloud import pubsub_v1

subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path('my-project', 'events-sub')

def callback(message):
    print(f"Received: {message.data}, ordering_key: {message.ordering_key}")
    message.ack()

# Subscribe with message ordering enabled
streaming_pull_future = subscriber.subscribe(
    subscription_path,
    callback=callback,
    flow_control=pubsub_v1.types.FlowControl(max_messages=100)
)

try:
    streaming_pull_future.result()
except TimeoutError:
    streaming_pull_future.cancel()
```

**Ordering Best Practices**:
- Use ordering keys for related events (e.g., per-user events)
- Ordering reduces throughput (messages with same key processed sequentially)
- Consider partitioning by ordering key for parallelism
- Monitor ordering key distribution to avoid hotspots

#### Exactly-Once Delivery

**Idempotent Processing with Message IDs**:
```python
from google.cloud import pubsub_v1, firestore

subscriber = pubsub_v1.SubscriberClient()
db = firestore.Client()

def exactly_once_callback(message):
    message_id = message.message_id

    # Check if message already processed
    doc_ref = db.collection('processed_messages').document(message_id)

    if doc_ref.get().exists:
        print(f"Message {message_id} already processed, skipping")
        message.ack()
        return

    try:
        # Process message
        process_event(message.data)

        # Record message as processed
        doc_ref.set({'processed_at': firestore.SERVER_TIMESTAMP})

        message.ack()
    except Exception as e:
        print(f"Error processing message: {e}")
        message.nack()  # Retry later

# Subscribe
streaming_pull_future = subscriber.subscribe(
    subscription_path,
    callback=exactly_once_callback
)
```

**Exactly-Once with BigQuery Subscriptions** (Preview):
```bash
# Create BigQuery subscription (automatic exactly-once delivery)
gcloud pubsub subscriptions create events-bq-sub \
  --topic=events \
  --bigquery-table=my-project:dataset.events \
  --use-topic-schema \
  --write-metadata
```

#### Dead Letter Topics

**Configure Dead Letter Topic**:
```python
from google.cloud import pubsub_v1

publisher = pubsub_v1.PublisherClient()
subscriber = pubsub_v1.SubscriberClient()

# Create dead letter topic
dlq_topic_path = publisher.topic_path('my-project', 'events-dlq')
publisher.create_topic(request={"name": dlq_topic_path})

# Create subscription with dead letter policy
subscription_path = subscriber.subscription_path('my-project', 'events-sub')
topic_path = publisher.topic_path('my-project', 'events')

subscription = subscriber.create_subscription(
    request={
        "name": subscription_path,
        "topic": topic_path,
        "dead_letter_policy": {
            "dead_letter_topic": dlq_topic_path,
            "max_delivery_attempts": 5
        },
        "ack_deadline_seconds": 60,
        "message_retention_duration": {"seconds": 604800},  # 7 days
        "retry_policy": {
            "minimum_backoff": {"seconds": 10},
            "maximum_backoff": {"seconds": 600}
        }
    }
)
```

**Monitor Dead Letter Queue**:
```python
# Subscribe to DLQ for monitoring
def dlq_callback(message):
    import logging
    logging.error(f"Dead letter message: {message.data}")
    logging.error(f"Delivery attempts: {message.delivery_attempt}")

    # Alert or investigate
    send_alert(f"Message failed after {message.delivery_attempt} attempts")

    message.ack()

dlq_subscription = subscriber.subscription_path('my-project', 'events-dlq-sub')
dlq_future = subscriber.subscribe(dlq_subscription, callback=dlq_callback)
```

#### Pub/Sub to Dataflow Integration

**Stream Processing Pattern**:
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

def run():
    options = PipelineOptions([
        '--streaming',
        '--project=my-project',
        '--runner=DataflowRunner',
        '--region=us-central1'
    ])

    with beam.Pipeline(options=options) as pipeline:
        (pipeline
         | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(
             subscription='projects/my-project/subscriptions/events-sub')
         | 'Decode' >> beam.Map(lambda x: x.decode('utf-8'))
         | 'Parse JSON' >> beam.Map(json.loads)
         | 'Transform' >> beam.ParDo(TransformEvents())
         | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
             'my-project:dataset.events',
             method='STREAMING_INSERTS'))
```

#### Pub/Sub Best Practices

**Publisher Batching**:
```python
from google.cloud import pubsub_v1
from google.cloud.pubsub_v1 import types

# Configure batching settings
batch_settings = types.BatchSettings(
    max_bytes=1024 * 1024,  # 1 MB
    max_latency=0.05,  # 50 ms
    max_messages=100
)

publisher = pubsub_v1.PublisherClient(batch_settings=batch_settings)

# Publish multiple messages (automatically batched)
for i in range(1000):
    future = publisher.publish(topic_path, data=f"Message {i}".encode())
    # Don't block on result unless needed
```

**Flow Control for Subscribers**:
```python
flow_control = pubsub_v1.types.FlowControl(
    max_messages=1000,  # Max outstanding messages
    max_bytes=1024 * 1024 * 100,  # 100 MB
    max_lease_duration=600  # 10 minutes
)

streaming_pull_future = subscriber.subscribe(
    subscription_path,
    callback=callback,
    flow_control=flow_control
)
```

**Schema Validation**:
```python
from google.cloud import pubsub_v1
from google.cloud.pubsub_v1 import types

# Create schema
schema_client = pubsub_v1.SchemaServiceClient()
schema_path = schema_client.schema_path('my-project', 'event-schema')

schema = {
    "name": schema_path,
    "type": types.Schema.Type.AVRO,
    "definition": '{"type": "record", "name": "Event", "fields": [{"name": "user_id", "type": "string"}, {"name": "timestamp", "type": "long"}]}'
}

created_schema = schema_client.create_schema(
    request={"parent": f"projects/my-project", "schema": schema, "schema_id": "event-schema"}
)

# Create topic with schema
topic = publisher.create_topic(
    request={
        "name": topic_path,
        "schema_settings": {"schema": schema_path, "encoding": types.Encoding.JSON}
    }
)
```

### Dataproc - Comprehensive Deep Dive

#### Overview
**Strengths**: Managed Spark/Hadoop, familiar tools, job isolation, fast cluster creation
**Use Cases**: Spark/Hadoop migrations, existing Spark jobs, ML with Spark MLlib, data lake processing
**Optimization**: Ephemeral clusters, autoscaling, preemptible workers, initialization actions
**Integration**: Cloud Storage (HCFS), BigQuery connector, Hive Metastore

#### Cluster Configuration

**Create Optimized Cluster**:
```bash
# Create production cluster with autoscaling
gcloud dataproc clusters create prod-cluster \
  --region=us-central1 \
  --zone=us-central1-a \
  --master-machine-type=n1-standard-4 \
  --master-boot-disk-size=100GB \
  --num-workers=2 \
  --worker-machine-type=n1-standard-4 \
  --worker-boot-disk-size=100GB \
  --num-secondary-workers=4 \
  --secondary-worker-type=preemptible \
  --secondary-worker-boot-disk-size=100GB \
  --image-version=2.1-debian11 \
  --scopes=cloud-platform \
  --project=my-project \
  --enable-component-gateway \
  --optional-components=JUPYTER,ZEPPELIN \
  --metadata='PIP_PACKAGES=pandas numpy scikit-learn' \
  --initialization-actions=gs://my-bucket/init-script.sh \
  --autoscaling-policy=my-autoscaling-policy \
  --properties=spark:spark.executor.memory=3g,spark:spark.executor.cores=2

# Create autoscaling policy
gcloud dataproc autoscaling-policies create my-autoscaling-policy \
  --region=us-central1 \
  --worker-min-instances=2 \
  --worker-max-instances=20 \
  --secondary-worker-min-instances=0 \
  --secondary-worker-max-instances=50 \
  --scale-up-factor=0.5 \
  --scale-down-factor=1.0 \
  --scale-up-min-worker-fraction=0.0 \
  --scale-down-min-worker-fraction=0.0
```

**Ephemeral Cluster Pattern**:
```bash
#!/bin/bash
# Create cluster, run job, delete cluster

CLUSTER_NAME="ephemeral-cluster-$(date +%s)"
JOB_FILE="gs://my-bucket/jobs/word-count.py"

# Create cluster
gcloud dataproc clusters create $CLUSTER_NAME \
  --region=us-central1 \
  --num-workers=5 \
  --worker-machine-type=n1-standard-4 \
  --max-idle=10m

# Submit job
gcloud dataproc jobs submit pyspark $JOB_FILE \
  --cluster=$CLUSTER_NAME \
  --region=us-central1 \
  -- gs://my-bucket/input gs://my-bucket/output

# Cluster auto-deletes after 10 min idle
```

#### Initialization Actions

**Custom Initialization Script**:
```bash
#!/bin/bash
# init-script.sh - Run on all nodes during cluster creation

set -e

# Install Python packages
pip3 install --upgrade \
  pandas==1.5.3 \
  numpy==1.24.2 \
  scikit-learn==1.2.2 \
  google-cloud-storage==2.10.0

# Install custom libraries
gsutil cp gs://my-bucket/libs/custom-lib.jar /usr/lib/spark/jars/

# Configure Spark
cat >> /etc/spark/conf/spark-defaults.conf <<EOF
spark.executor.memory=4g
spark.executor.cores=2
spark.dynamicAllocation.enabled=true
spark.shuffle.service.enabled=true
EOF

# Setup monitoring agent
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
bash add-google-cloud-ops-agent-repo.sh --also-install

# Log completion
echo "Initialization complete" >> /var/log/init-script.log
```

#### PySpark Job Examples

**ETL Pipeline with BigQuery Connector**:
```python
# pyspark_etl.py
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, to_date, count, sum as spark_sum, avg

def run_etl():
    """Complete ETL pipeline with BigQuery integration"""

    spark = SparkSession.builder \
        .appName("ETL Pipeline") \
        .config("spark.jars.packages", "com.google.cloud.spark:spark-bigquery-with-dependencies_2.12:0.32.0") \
        .getOrCreate()

    # Read from BigQuery
    events_df = spark.read \
        .format("bigquery") \
        .option("table", "my-project.dataset.events") \
        .option("filter", "event_date >= '2024-01-01'") \
        .load()

    # Read from Cloud Storage (Parquet)
    users_df = spark.read \
        .parquet("gs://my-bucket/users/*.parquet")

    # Transform: Join and aggregate
    result_df = events_df \
        .join(users_df, events_df.user_id == users_df.user_id, "left") \
        .groupBy(
            to_date(col("event_timestamp")).alias("date"),
            col("country"),
            col("user_segment")
        ) \
        .agg(
            count("*").alias("event_count"),
            spark_sum("revenue").alias("total_revenue"),
            avg("session_duration").alias("avg_session_duration")
        ) \
        .filter(col("event_count") > 10)

    # Write to BigQuery
    result_df.write \
        .format("bigquery") \
        .option("table", "my-project.dataset.daily_metrics") \
        .option("writeMethod", "direct") \
        .option("temporaryGcsBucket", "my-bucket-temp") \
        .mode("append") \
        .save()

    # Also write to Cloud Storage (Partitioned Parquet)
    result_df.write \
        .partitionBy("date", "country") \
        .mode("overwrite") \
        .parquet("gs://my-bucket/output/daily-metrics")

    spark.stop()

if __name__ == "__main__":
    run_etl()
```

**Spark Streaming Job**:
```python
# spark_streaming.py
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window, count
from pyspark.sql.types import StructType, StringType, TimestampType, DoubleType

def run_streaming():
    """Spark Structured Streaming with Pub/Sub"""

    spark = SparkSession.builder \
        .appName("Streaming Pipeline") \
        .getOrCreate()

    # Define schema
    schema = StructType() \
        .add("user_id", StringType()) \
        .add("event_type", StringType()) \
        .add("timestamp", TimestampType()) \
        .add("revenue", DoubleType()) \
        .add("country", StringType())

    # Read from Pub/Sub
    events_df = spark.readStream \
        .format("pubsub") \
        .option("subscription", "projects/my-project/subscriptions/events-sub") \
        .load()

    # Parse JSON data
    parsed_df = events_df \
        .select(from_json(col("data").cast("string"), schema).alias("event")) \
        .select("event.*")

    # Windowed aggregation
    windowed_counts = parsed_df \
        .withWatermark("timestamp", "10 minutes") \
        .groupBy(
            window(col("timestamp"), "5 minutes", "1 minute"),
            col("country")
        ) \
        .agg(
            count("*").alias("event_count")
        )

    # Write to BigQuery (streaming)
    query = windowed_counts.writeStream \
        .format("bigquery") \
        .option("table", "my-project:dataset.streaming_metrics") \
        .option("checkpointLocation", "gs://my-bucket/checkpoints/streaming") \
        .outputMode("append") \
        .start()

    query.awaitTermination()

if __name__ == "__main__":
    run_streaming()
```

#### Spark Optimization

**Performance Tuning**:
```python
# Configure Spark for optimal performance
spark = SparkSession.builder \
    .appName("Optimized Job") \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
    .config("spark.sql.adaptive.skewJoin.enabled", "true") \
    .config("spark.dynamicAllocation.enabled", "true") \
    .config("spark.dynamicAllocation.minExecutors", "2") \
    .config("spark.dynamicAllocation.maxExecutors", "20") \
    .config("spark.executor.memory", "4g") \
    .config("spark.executor.cores", "2") \
    .config("spark.sql.shuffle.partitions", "200") \
    .config("spark.sql.files.maxPartitionBytes", "134217728") \
    .config("spark.serializer", "org.apache.spark.serializer.KryoSerializer") \
    .config("spark.speculation", "true") \
    .getOrCreate()

# Cache frequently accessed data
users_df.cache()

# Repartition for parallelism
events_df = events_df.repartition(200, "event_date")

# Broadcast small dimension tables
from pyspark.sql.functions import broadcast
result_df = large_df.join(broadcast(small_df), "id")

# Persist intermediate results
intermediate_df.persist(StorageLevel.MEMORY_AND_DISK)
```

**Avoid Data Skew**:
```python
# Detect skew
from pyspark.sql.functions import col, count

skew_check = df.groupBy("partition_key").agg(count("*").alias("count"))
skew_check.orderBy(col("count").desc()).show()

# Solution 1: Salt keys
from pyspark.sql.functions import rand, concat, lit

salted_df = df.withColumn("salted_key", concat(col("key"), lit("_"), (rand() * 10).cast("int")))

# Solution 2: Adaptive query execution
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionFactor", "5")
```

#### Job Submission Patterns

**Submit PySpark Job**:
```bash
# Submit job to existing cluster
gcloud dataproc jobs submit pyspark \
  gs://my-bucket/jobs/etl_pipeline.py \
  --cluster=prod-cluster \
  --region=us-central1 \
  --jars=gs://spark-lib/bigquery/spark-bigquery-latest.jar \
  --properties=spark.executor.memory=4g,spark.executor.cores=2 \
  -- \
  --input gs://my-bucket/input \
  --output gs://my-bucket/output \
  --date 2024-01-15

# Submit with dependencies
gcloud dataproc jobs submit pyspark \
  gs://my-bucket/jobs/ml_pipeline.py \
  --cluster=ml-cluster \
  --region=us-central1 \
  --py-files=gs://my-bucket/libs/utils.zip \
  --files=gs://my-bucket/config/config.yaml \
  --properties=spark.python.version=3
```

**Submit Spark Job (Scala/Java)**:
```bash
gcloud dataproc jobs submit spark \
  --cluster=prod-cluster \
  --region=us-central1 \
  --jar=gs://my-bucket/jobs/etl-job-1.0.jar \
  --class=com.example.ETLJob \
  --jars=gs://my-bucket/libs/dependencies.jar \
  --properties=spark.executor.memory=8g \
  -- \
  --input-path gs://my-bucket/input \
  --output-path gs://my-bucket/output
```

**Workflow Template for Job Orchestration**:
```bash
# Create workflow template
gcloud dataproc workflow-templates create daily-etl \
  --region=us-central1

# Add managed cluster
gcloud dataproc workflow-templates set-managed-cluster daily-etl \
  --region=us-central1 \
  --cluster-name=ephemeral-cluster \
  --num-workers=5 \
  --worker-machine-type=n1-standard-4 \
  --autoscaling-policy=my-policy

# Add jobs in sequence
gcloud dataproc workflow-templates add-job pyspark \
  gs://my-bucket/jobs/extract.py \
  --step-id=extract \
  --workflow-template=daily-etl \
  --region=us-central1

gcloud dataproc workflow-templates add-job pyspark \
  gs://my-bucket/jobs/transform.py \
  --step-id=transform \
  --workflow-template=daily-etl \
  --region=us-central1 \
  --start-after=extract

gcloud dataproc workflow-templates add-job pyspark \
  gs://my-bucket/jobs/load.py \
  --step-id=load \
  --workflow-template=daily-etl \
  --region=us-central1 \
  --start-after=transform

# Instantiate workflow
gcloud dataproc workflow-templates instantiate daily-etl \
  --region=us-central1
```

#### Dataproc Best Practices

**Cost Optimization**:
1. **Use preemptible workers**: 50-80% cost savings for secondary workers
2. **Ephemeral clusters**: Create/delete clusters per job
3. **Enhanced flexibility mode**: Mix preemptible and standard workers
4. **Autoscaling**: Scale based on YARN metrics
5. **Scheduled deletion**: Set max-idle or max-age

**Performance Optimization**:
1. **Use Cloud Storage**: Store data in GCS, not HDFS
2. **BigQuery connector**: Direct read/write without staging
3. **Component gateway**: Access UIs without SSH
4. **Image version**: Use latest for performance improvements
5. **Local SSDs**: For shuffle-heavy workloads

**Monitoring**:
```bash
# View job logs
gcloud dataproc jobs describe JOB_ID --region=us-central1

# Monitor cluster
gcloud dataproc clusters describe CLUSTER_NAME --region=us-central1

# View YARN metrics
# Access via component gateway: https://CLUSTER_NAME-m:8088
```

### BigQuery
**Strengths**: Serverless, SQL interface, ML capabilities, fast analytics
**Use Cases**: Data warehouse, analytics, BI, ML
**Optimization**: Partitioning, clustering, materialized views

**Best Practices**:
- Partition by date for time-series data
- Cluster on common filter columns
- Use SELECT specific columns, avoid SELECT *
- Use approximate functions for large datasets
- Monitor slot usage and optimize queries
- Use scheduled queries for recurring tasks
- Implement incremental loads

### Cloud Composer (Apache Airflow) - Comprehensive Deep Dive

#### Overview
**Strengths**: Workflow orchestration, managed Airflow, Python-based DAGs, built-in operators
**Use Cases**: Complex workflows, dependencies, scheduling, data pipeline orchestration
**Integration**: All GCP services, custom operators, third-party integrations
**Architecture**: Runs on GKE, includes web server, scheduler, workers, database

#### DAG Creation and Structure

**Complete Production DAG Example**:
```python
# dags/daily_etl_pipeline.py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.providers.google.cloud.operators.bigquery import (
    BigQueryCreateEmptyDatasetOperator,
    BigQueryInsertJobOperator,
    BigQueryCheckOperator
)
from airflow.providers.google.cloud.operators.dataflow import DataflowCreatePythonJobOperator
from airflow.providers.google.cloud.operators.dataproc import (
    DataprocCreateClusterOperator,
    DataprocSubmitJobOperator,
    DataprocDeleteClusterOperator
)
from airflow.providers.google.cloud.operators.gcs import GCSDeleteObjectsOperator
from airflow.providers.google.cloud.sensors.gcs import GCSObjectExistenceSensor
from airflow.providers.google.cloud.transfers.gcs_to_bigquery import GCSToBigQueryOperator
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.dummy import DummyOperator
from airflow.utils.task_group import TaskGroup
from airflow.models import Variable
import logging

# Default arguments
default_args = {
    'owner': 'data-engineering',
    'depends_on_past': False,
    'email': ['data-team@company.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(minutes=30),
    'execution_timeout': timedelta(hours=2),
}

# DAG definition
dag = DAG(
    'daily_etl_pipeline',
    default_args=default_args,
    description='Daily ETL pipeline with comprehensive error handling',
    schedule_interval='0 2 * * *',  # Run at 2 AM daily
    start_date=datetime(2024, 1, 1),
    catchup=False,
    max_active_runs=1,
    tags=['production', 'etl', 'daily'],
    doc_md="""
    # Daily ETL Pipeline

    This DAG processes daily event data through multiple stages:
    1. Validate input data availability
    2. Extract data from Cloud Storage
    3. Transform with Dataflow
    4. Load to BigQuery
    5. Run data quality checks
    6. Generate summary reports
    """
)

# Configuration variables
PROJECT_ID = Variable.get('gcp_project_id')
DATASET_ID = 'analytics'
BUCKET_NAME = Variable.get('data_bucket')
DATAFLOW_TEMPLATE = f'gs://{BUCKET_NAME}/templates/transform-template.json'

# Task 1: Check if source data exists
check_source_data = GCSObjectExistenceSensor(
    task_id='check_source_data',
    bucket=BUCKET_NAME,
    object='raw/events/{{ ds }}/*.json',
    timeout=600,
    poke_interval=60,
    mode='reschedule',  # Free up worker slot while waiting
    dag=dag
)

# Task 2: Data quality validation
def validate_data_quality(**context):
    """Validate source data before processing"""
    from google.cloud import storage
    import json

    execution_date = context['ds']
    bucket = storage.Client().bucket(BUCKET_NAME)
    blobs = bucket.list_blobs(prefix=f'raw/events/{execution_date}/')

    total_records = 0
    invalid_records = 0

    for blob in blobs:
        content = blob.download_as_text()
        for line in content.splitlines():
            total_records += 1
            try:
                record = json.loads(line)
                # Validate required fields
                if not all(k in record for k in ['user_id', 'timestamp', 'event_type']):
                    invalid_records += 1
            except json.JSONDecodeError:
                invalid_records += 1

    error_rate = invalid_records / total_records if total_records > 0 else 0

    logging.info(f"Total records: {total_records}, Invalid: {invalid_records}, Error rate: {error_rate:.2%}")

    # Push to XCom for downstream tasks
    context['task_instance'].xcom_push(key='total_records', value=total_records)
    context['task_instance'].xcom_push(key='error_rate', value=error_rate)

    # Fail if error rate too high
    if error_rate > 0.05:  # 5% threshold
        raise ValueError(f"Error rate {error_rate:.2%} exceeds threshold")

    return total_records

validate_data = PythonOperator(
    task_id='validate_data_quality',
    python_callable=validate_data_quality,
    provide_context=True,
    dag=dag
)

# Task 3: Branch based on data volume
def choose_processing_method(**context):
    """Choose processing method based on data volume"""
    total_records = context['task_instance'].xcom_pull(
        task_ids='validate_data_quality',
        key='total_records'
    )

    # Use Dataflow for large datasets, BigQuery for small
    if total_records > 1000000:
        return 'process_with_dataflow'
    else:
        return 'process_with_bigquery'

branch_processing = BranchPythonOperator(
    task_id='branch_processing_method',
    python_callable=choose_processing_method,
    provide_context=True,
    dag=dag
)

# Task 4a: Process with Dataflow
process_dataflow = DataflowCreatePythonJobOperator(
    task_id='process_with_dataflow',
    job_name='daily-etl-{{ ds_nodash }}',
    py_file=f'gs://{BUCKET_NAME}/pipelines/transform_pipeline.py',
    options={
        'input': f'gs://{BUCKET_NAME}/raw/events/{{{{ ds }}}}/*.json',
        'output': f'{PROJECT_ID}:{DATASET_ID}.events',
        'temp_location': f'gs://{BUCKET_NAME}/temp',
        'staging_location': f'gs://{BUCKET_NAME}/staging',
        'region': 'us-central1',
        'num_workers': 5,
        'max_num_workers': 20,
        'autoscaling_algorithm': 'THROUGHPUT_BASED',
    },
    dag=dag
)

# Task 4b: Process with BigQuery
process_bigquery = BigQueryInsertJobOperator(
    task_id='process_with_bigquery',
    configuration={
        'query': {
            'query': f"""
                CREATE OR REPLACE TABLE `{PROJECT_ID}.{DATASET_ID}.events_{{{{ ds_nodash }}}}` AS
                SELECT
                    JSON_VALUE(data, '$.user_id') as user_id,
                    TIMESTAMP(JSON_VALUE(data, '$.timestamp')) as event_timestamp,
                    JSON_VALUE(data, '$.event_type') as event_type,
                    SAFE_CAST(JSON_VALUE(data, '$.revenue') AS FLOAT64) as revenue,
                    JSON_VALUE(data, '$.country') as country
                FROM `{PROJECT_ID}.{DATASET_ID}.raw_events`
                WHERE DATE(timestamp) = '{{{{ ds }}}}'
                    AND JSON_VALUE(data, '$.user_id') IS NOT NULL
            """,
            'useLegacySql': False,
        }
    },
    dag=dag
)

# Task 5: Join processing paths
join_processing = DummyOperator(
    task_id='join_processing',
    trigger_rule='one_success',  # Succeed if either branch succeeds
    dag=dag
)

# Task Group: Data Quality Checks
with TaskGroup('data_quality_checks', dag=dag) as quality_checks:

    check_row_count = BigQueryCheckOperator(
        task_id='check_row_count',
        sql=f"""
            SELECT COUNT(*) > 0
            FROM `{PROJECT_ID}.{DATASET_ID}.events`
            WHERE DATE(event_timestamp) = '{{{{ ds }}}}'
        """,
        use_legacy_sql=False,
    )

    check_null_values = BigQueryCheckOperator(
        task_id='check_null_values',
        sql=f"""
            SELECT COUNT(*) = 0
            FROM `{PROJECT_ID}.{DATASET_ID}.events`
            WHERE DATE(event_timestamp) = '{{{{ ds }}}}'
                AND (user_id IS NULL OR event_type IS NULL)
        """,
        use_legacy_sql=False,
    )

    check_revenue_range = BigQueryCheckOperator(
        task_id='check_revenue_range',
        sql=f"""
            SELECT COUNT(*) = 0
            FROM `{PROJECT_ID}.{DATASET_ID}.events`
            WHERE DATE(event_timestamp) = '{{{{ ds }}}}'
                AND (revenue < 0 OR revenue > 10000)
        """,
        use_legacy_sql=False,
    )

    check_row_count >> check_null_values >> check_revenue_range

# Task 6: Create aggregated tables
create_daily_summary = BigQueryInsertJobOperator(
    task_id='create_daily_summary',
    configuration={
        'query': {
            'query': f"""
                CREATE OR REPLACE TABLE `{PROJECT_ID}.{DATASET_ID}.daily_summary` AS
                SELECT
                    DATE(event_timestamp) as event_date,
                    country,
                    event_type,
                    COUNT(*) as event_count,
                    COUNT(DISTINCT user_id) as unique_users,
                    SUM(revenue) as total_revenue,
                    AVG(revenue) as avg_revenue
                FROM `{PROJECT_ID}.{DATASET_ID}.events`
                WHERE DATE(event_timestamp) = '{{{{ ds }}}}'
                GROUP BY event_date, country, event_type
            """,
            'useLegacySql': False,
        }
    },
    dag=dag
)

# Task 7: Send completion notification
def send_completion_notification(**context):
    """Send notification with pipeline metrics"""
    from google.cloud import monitoring_v3
    import time

    execution_date = context['ds']
    total_records = context['task_instance'].xcom_pull(
        task_ids='validate_data_quality',
        key='total_records'
    )

    logging.info(f"Pipeline completed for {execution_date}")
    logging.info(f"Processed {total_records} records")

    # Write custom metric to Cloud Monitoring
    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{PROJECT_ID}"

    series = monitoring_v3.TimeSeries()
    series.metric.type = 'custom.googleapis.com/airflow/records_processed'
    series.resource.type = 'global'

    now = time.time()
    seconds = int(now)
    nanos = int((now - seconds) * 10 ** 9)
    interval = monitoring_v3.TimeInterval(
        {"end_time": {"seconds": seconds, "nanos": nanos}}
    )
    point = monitoring_v3.Point(
        {"interval": interval, "value": {"int64_value": total_records}}
    )
    series.points = [point]

    client.create_time_series(name=project_name, time_series=[series])

notify_completion = PythonOperator(
    task_id='notify_completion',
    python_callable=send_completion_notification,
    provide_context=True,
    dag=dag
)

# Task 8: Cleanup temporary files
cleanup_temp_files = GCSDeleteObjectsOperator(
    task_id='cleanup_temp_files',
    bucket_name=BUCKET_NAME,
    prefix='temp/{{ ds }}/',
    dag=dag
)

# Define task dependencies
check_source_data >> validate_data >> branch_processing
branch_processing >> [process_dataflow, process_bigquery]
[process_dataflow, process_bigquery] >> join_processing
join_processing >> quality_checks >> create_daily_summary
create_daily_summary >> notify_completion >> cleanup_temp_files
```

#### Advanced Airflow Patterns

**XComs for Data Passing**:
```python
# Push data to XCom
def push_data(**context):
    data = {'key': 'value', 'count': 100}
    context['task_instance'].xcom_push(key='my_data', value=data)
    return data  # Also pushed to XCom with default key 'return_value'

# Pull data from XCom
def pull_data(**context):
    data = context['task_instance'].xcom_pull(
        task_ids='push_data_task',
        key='my_data'
    )
    print(f"Retrieved data: {data}")

# XCom with TaskFlow API (Airflow 2.0+)
from airflow.decorators import task

@task
def extract():
    return {'data': [1, 2, 3, 4, 5]}

@task
def transform(data_dict):
    return [x * 2 for x in data_dict['data']]

@task
def load(data):
    print(f"Loading: {data}")

# Build DAG
data = extract()
transformed = transform(data)
load(transformed)
```

**Sensors for External Dependencies**:
```python
from airflow.sensors.external_task import ExternalTaskSensor
from airflow.providers.google.cloud.sensors.bigquery import BigQueryTableExistenceSensor

# Wait for another DAG to complete
wait_for_upstream = ExternalTaskSensor(
    task_id='wait_for_upstream_dag',
    external_dag_id='upstream_pipeline',
    external_task_id='final_task',
    execution_delta=timedelta(hours=1),  # Offset if schedules differ
    timeout=3600,
    mode='reschedule',
    dag=dag
)

# Wait for BigQuery table
wait_for_table = BigQueryTableExistenceSensor(
    task_id='wait_for_dimension_table',
    project_id=PROJECT_ID,
    dataset_id='dimensions',
    table_id='products',
    poke_interval=300,
    timeout=7200,
    mode='reschedule',
    dag=dag
)

# Custom sensor
from airflow.sensors.base import BaseSensorOperator

class CustomDataSensor(BaseSensorOperator):
    def __init__(self, condition_callable, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.condition_callable = condition_callable

    def poke(self, context):
        """Check if condition is met"""
        return self.condition_callable(context)

def check_data_freshness(**context):
    from google.cloud import bigquery
    client = bigquery.Client()

    query = """
        SELECT MAX(updated_at) as last_update
        FROM `project.dataset.table`
    """
    result = client.query(query).result()
    last_update = list(result)[0].last_update

    # Check if updated in last hour
    from datetime import datetime, timedelta
    return datetime.utcnow() - last_update < timedelta(hours=1)

wait_for_fresh_data = CustomDataSensor(
    task_id='wait_for_fresh_data',
    condition_callable=check_data_freshness,
    poke_interval=300,
    timeout=3600,
    dag=dag
)
```

**Dynamic Task Generation**:
```python
from airflow.operators.python import PythonOperator

def process_partition(partition_id, **context):
    """Process a single partition"""
    logging.info(f"Processing partition: {partition_id}")
    # Processing logic here

# Generate tasks dynamically
partitions = ['2024-01-01', '2024-01-02', '2024-01-03']

for partition in partitions:
    task = PythonOperator(
        task_id=f'process_partition_{partition.replace("-", "_")}',
        python_callable=process_partition,
        op_kwargs={'partition_id': partition},
        dag=dag
    )

    # Set dependencies
    start_task >> task >> end_task
```

**SubDAGs and TaskGroups**:
```python
from airflow.utils.task_group import TaskGroup

# TaskGroups (preferred over SubDAGs)
with TaskGroup('preprocessing', dag=dag) as preprocessing:
    validate = PythonOperator(
        task_id='validate',
        python_callable=validate_data
    )

    clean = PythonOperator(
        task_id='clean',
        python_callable=clean_data
    )

    deduplicate = PythonOperator(
        task_id='deduplicate',
        python_callable=deduplicate_data
    )

    validate >> clean >> deduplicate

with TaskGroup('postprocessing', dag=dag) as postprocessing:
    aggregate = PythonOperator(
        task_id='aggregate',
        python_callable=aggregate_data
    )

    export = PythonOperator(
        task_id='export',
        python_callable=export_data
    )

    aggregate >> export

# Use task groups in DAG
start >> preprocessing >> process >> postprocessing >> end
```

#### Composer Configuration and Optimization

**Environment Configuration**:
```bash
# Create Composer environment
gcloud composer environments create prod-composer \
    --location=us-central1 \
    --machine-type=n1-standard-4 \
    --node-count=3 \
    --disk-size=50 \
    --python-version=3 \
    --airflow-version=2.5.1 \
    --env-variables=\
PROJECT_ID=my-project,\
ENVIRONMENT=production \
    --scheduler-cpu=2 \
    --scheduler-memory=7.5 \
    --scheduler-storage=5 \
    --scheduler-count=2 \
    --web-server-cpu=2 \
    --web-server-memory=7.5 \
    --web-server-storage=5 \
    --worker-cpu=2 \
    --worker-memory=7.5 \
    --worker-storage=5 \
    --min-workers=2 \
    --max-workers=12 \
    --maintenance-window-start=2024-01-01T00:00:00Z \
    --maintenance-window-end=2024-01-01T04:00:00Z \
    --maintenance-window-recurrence='FREQ=WEEKLY;BYDAY=SU'

# Update environment
gcloud composer environments update prod-composer \
    --location=us-central1 \
    --update-env-variables=NEW_VAR=value \
    --update-airflow-configs=core-max_active_runs_per_dag=3

# Set Airflow configuration
gcloud composer environments update prod-composer \
    --location=us-central1 \
    --update-airflow-configs=\
core-parallelism=32,\
core-dag_concurrency=16,\
core-max_active_runs_per_dag=3,\
scheduler-min_file_process_interval=30,\
webserver-worker_refresh_interval=1800
```

**Performance Best Practices**:
```python
# 1. Use pools to limit concurrency
from airflow.models import Pool

# Define in Airflow UI or programmatically
# Pool: bigquery_pool, Slots: 5

task_with_pool = BigQueryInsertJobOperator(
    task_id='query_task',
    pool='bigquery_pool',  # Limit concurrent BigQuery tasks
    dag=dag
)

# 2. Use task concurrency limits
task = PythonOperator(
    task_id='limited_task',
    python_callable=my_function,
    max_active_tis_per_dag=2,  # Max 2 instances running
    dag=dag
)

# 3. Use reschedule mode for sensors
sensor = GCSObjectExistenceSensor(
    task_id='wait_for_file',
    bucket='my-bucket',
    object='data.csv',
    mode='reschedule',  # Free up worker slot
    poke_interval=300,
    dag=dag
)

# 4. Optimize XCom usage
# Avoid storing large data in XCom
# Instead, store references (GCS paths, table names)
def push_reference(**context):
    # BAD: Storing large data
    # large_data = fetch_large_dataset()
    # context['ti'].xcom_push(key='data', value=large_data)

    # GOOD: Store reference
    gcs_path = 'gs://bucket/data.parquet'
    context['ti'].xcom_push(key='data_path', value=gcs_path)

# 5. Use trigger rules appropriately
task = PythonOperator(
    task_id='always_run',
    python_callable=cleanup,
    trigger_rule='all_done',  # Run even if upstream fails
    dag=dag
)
```

#### Monitoring and Debugging

**Custom Metrics and Logging**:
```python
from airflow.decorators import task
import logging

@task
def monitored_task(**context):
    """Task with comprehensive monitoring"""
    import time
    from google.cloud import monitoring_v3

    start_time = time.time()

    try:
        # Processing logic
        result = process_data()

        # Log metrics
        duration = time.time() - start_time
        logging.info(f"Task completed in {duration:.2f} seconds")

        # Write to Cloud Monitoring
        client = monitoring_v3.MetricServiceClient()
        project_name = f"projects/{PROJECT_ID}"

        series = monitoring_v3.TimeSeries()
        series.metric.type = 'custom.googleapis.com/airflow/task_duration'
        series.metric.labels['task_id'] = context['task_instance'].task_id
        series.resource.type = 'global'

        point = monitoring_v3.Point({
            "interval": monitoring_v3.TimeInterval({
                "end_time": {"seconds": int(time.time())}
            }),
            "value": {"double_value": duration}
        })
        series.points = [point]

        client.create_time_series(name=project_name, time_series=[series])

        return result

    except Exception as e:
        # Log error
        logging.error(f"Task failed: {str(e)}", exc_info=True)

        # Send alert
        send_alert_to_pagerduty(context, str(e))

        raise
```

**Error Handling and Retries**:
```python
from airflow.exceptions import AirflowSkipException, AirflowFailException

def robust_task(**context):
    """Task with comprehensive error handling"""
    try:
        result = risky_operation()

        # Validate result
        if not validate_result(result):
            # Skip downstream tasks
            raise AirflowSkipException("Invalid result, skipping downstream tasks")

        return result

    except TemporaryError as e:
        # Retry-able error
        logging.warning(f"Temporary error: {e}, will retry")
        raise  # Trigger retry

    except PermanentError as e:
        # Non-retry-able error
        logging.error(f"Permanent error: {e}, failing task")
        raise AirflowFailException(f"Unrecoverable error: {e}")

task = PythonOperator(
    task_id='robust_task',
    python_callable=robust_task,
    retries=3,
    retry_delay=timedelta(minutes=5),
    retry_exponential_backoff=True,
    max_retry_delay=timedelta(hours=1),
    dag=dag
)
```

#### Composer Best Practices

**DAG Best Practices**:
1. **Idempotency**: Tasks should produce same result when re-run
2. **Incremental processing**: Process only new data, not entire dataset
3. **Avoid top-level code**: Don't execute expensive operations at DAG parse time
4. **Use Variables and Connections**: Externalize configuration
5. **Set timeouts**: Prevent tasks from running indefinitely
6. **Use appropriate trigger rules**: Handle complex dependencies
7. **Test DAGs**: Unit test task logic, integration test full DAG

**Deployment**:
```bash
# Deploy DAGs to Composer
COMPOSER_BUCKET=$(gcloud composer environments describe prod-composer \
    --location=us-central1 \
    --format='value(config.dagGcsPrefix)')

# Upload DAG file
gsutil cp dags/my_dag.py ${COMPOSER_BUCKET}/dags/

# Upload plugins
gsutil -m rsync -r plugins/ ${COMPOSER_BUCKET}/plugins/

# Upload dependencies
gsutil cp requirements.txt ${COMPOSER_BUCKET}/requirements.txt
```

## Data Migration Strategies

### Data Warehouse Migration
**Assessment**: Inventory current data warehouse, understand workloads
**Design**: Schema design, partitioning strategy, access patterns
**Migration**: Use Transfer Service, Storage Transfer Service, partner tools
**Validation**: Data quality checks, reconciliation
**Optimization**: Query optimization, cost management

**Steps**:
1. Discovery and assessment
2. Proof of concept
3. Schema and data model design
4. Pilot migration
5. Full migration in phases
6. Optimization and tuning

### Database Migration
**Homogeneous**: Same database engine (MySQL → Cloud SQL)
**Heterogeneous**: Different engines (Oracle → Cloud Spanner)
**Tools**: Database Migration Service, native replication, third-party tools
**Approach**: Minimal downtime vs. maintenance window

**Migration Patterns**:
- **One-time**: Full export/import, suitable for small databases
- **Continuous replication**: CDC (Change Data Capture), minimal downtime
- **Hybrid operation**: Gradual cutover, validate before switching

## Best Practices

### Data Pipeline Best Practices
1. **Idempotency**: Design pipelines to handle reruns safely
2. **Error Handling**: Implement retry logic, dead letter queues
3. **Monitoring**: Track data quality, latency, throughput
4. **Data Quality**: Validation at ingestion and transformation
5. **Schema Evolution**: Handle schema changes gracefully
6. **Partitioning**: Optimize for query patterns
7. **Documentation**: Document data lineage and transformations
8. **Testing**: Unit test transformations, integration test pipelines

### Performance Optimization
1. **Partitioning**: Time-based partitioning for temporal data
2. **Clustering**: Organize data by commonly filtered columns
3. **Denormalization**: Trade storage for query performance
4. **Caching**: Use materialized views for expensive queries
5. **Parallel Processing**: Leverage distributed processing
6. **Resource Allocation**: Right-size compute resources
7. **Data Skew**: Avoid hotspots in distributed processing
8. **Compression**: Reduce storage and I/O

### Data Fusion - No-Code Data Integration

#### Overview
**Strengths**: Visual pipeline builder, no-code/low-code, pre-built connectors, CDAP-based
**Use Cases**: Data migration, replication, ETL without code, citizen data engineers
**Integration**: 150+ connectors (databases, SaaS, files, APIs)
**Edition**: Basic (batch only), Enterprise (batch + streaming), Developer (testing)

**When to Use Data Fusion vs Dataflow**:
- **Data Fusion**: No-code requirement, business users, simple transformations, pre-built connectors
- **Dataflow**: Complex transformations, custom logic, advanced windowing, programmatic control

#### Pipeline Creation Example

**Batch Pipeline Architecture**:
```
Source (Cloud SQL) → Wrangler (clean) → Join (reference data) → Group By → Sink (BigQuery)
```

**Data Fusion Pipeline Configuration** (JSON export):
```json
{
  "name": "Customer360Pipeline",
  "description": "Aggregate customer data from multiple sources",
  "artifact": {
    "name": "cdap-data-pipeline",
    "version": "6.7.0",
    "scope": "SYSTEM"
  },
  "config": {
    "stages": [
      {
        "name": "CloudSQL_Source",
        "plugin": {
          "name": "CloudSQL",
          "type": "batchsource",
          "properties": {
            "referenceName": "customers",
            "connectionString": "jdbc:mysql://10.0.0.3:3306/prod_db",
            "importQuery": "SELECT * FROM customers WHERE updated_at >= '${logicalStartTime}'",
            "numSplits": "4"
          }
        }
      },
      {
        "name": "Wrangler",
        "plugin": {
          "name": "Wrangler",
          "type": "transform",
          "properties": {
            "directives": "parse-as-csv :body ',' true\ndrop :body\nset-type :age integer\nuppercase :country\nfill-null-or-empty :email 'unknown@example.com'"
          }
        }
      },
      {
        "name": "Joiner",
        "plugin": {
          "name": "Joiner",
          "type": "batchjoiner",
          "properties": {
            "joinKeys": "customer_id",
            "requiredInputs": "CloudSQL_Source,GCS_Reference",
            "selectedFields": "CloudSQL_Source.customer_id,CloudSQL_Source.name,GCS_Reference.segment"
          }
        }
      },
      {
        "name": "GroupByAggregate",
        "plugin": {
          "name": "GroupByAggregate",
          "type": "batchaggregator",
          "properties": {
            "groupByFields": "country,segment",
            "aggregates": "customer_count:Count(customer_id),avg_age:Avg(age)"
          }
        }
      },
      {
        "name": "BigQuery_Sink",
        "plugin": {
          "name": "BigQueryTable",
          "type": "batchsink",
          "properties": {
            "dataset": "analytics",
            "table": "customer_segments",
            "truncateTable": "false",
            "operation": "INSERT"
          }
        }
      }
    ],
    "connections": [
      {"from": "CloudSQL_Source", "to": "Wrangler"},
      {"from": "Wrangler", "to": "Joiner"},
      {"from": "Joiner", "to": "GroupByAggregate"},
      {"from": "GroupByAggregate", "to": "BigQuery_Sink"}
    ]
  }
}
```

**Data Wrangling Directives**:
```
# Parse CSV
parse-as-csv :column ',' true

# Handle nulls
fill-null-or-empty :email 'unknown@example.com'

# Type conversion
set-type :age integer
set-type :revenue double

# String operations
uppercase :country
lowercase :email
trim :name

# Date operations
parse-as-date :date_str 'yyyy-MM-dd'

# Filtering
filter-rows-on condition-true :age > 18

# Renaming
rename :old_name :new_name

# Dropping columns
drop :unused_column
```

#### Scheduling and Triggers

**Schedule Pipeline**:
- Cron expression: `0 2 * * *` (daily at 2 AM)
- Max concurrent runs: 1
- Timeout: 4 hours

**Trigger Configuration**:
```json
{
  "schedule": {
    "name": "Daily Customer ETL",
    "description": "Run customer aggregation daily",
    "cronExpression": "0 2 * * *",
    "properties": {
      "logicalStartTime": "${logical start time}",
      "maxConcurrentRuns": "1"
    }
  }
}
```

#### Data Fusion Best Practices
1. **Use Wrangler**: Interactively develop transformations before deploying
2. **Incremental loading**: Use logical start time for incremental queries
3. **Connection reuse**: Create reusable connections in Connection Manager
4. **Monitoring**: Set up alerts for pipeline failures
5. **Version control**: Export pipelines as JSON for version control
6. **Resource allocation**: Size compute profile based on data volume

## Complete Real-World Data Engineering Scenarios

### Scenario 1: Real-Time Fraud Detection Pipeline

**Requirements**:
- Process credit card transactions in real-time
- Detect fraudulent transactions within 100ms
- Store results for analysis
- Handle 100,000 transactions/second
- Maintain 99.9% availability

**Architecture**:
```
Transaction API → Pub/Sub → Dataflow (streaming) → {
    Bigtable (real-time lookups),
    BigQuery (analytics),
    Pub/Sub (alerts)
}
```

**Implementation**:
```python
# Dataflow fraud detection pipeline
import apache_beam as beam
from apache_beam import window
from apache_beam.options.pipeline_options import PipelineOptions
import json

class FraudDetection(beam.DoFn):
    def __init__(self, project_id):
        self.project_id = project_id

    def setup(self):
        from google.cloud import bigtable
        self.bt_client = bigtable.Client(project=self.project_id)
        self.table = self.bt_client.instance('fraud-detection').table('user-profiles')

    def process(self, element):
        """Detect fraud using real-time user profile from Bigtable"""
        transaction = element

        # Get user profile from Bigtable
        row_key = f"user#{transaction['user_id']}"
        row = self.table.read_row(row_key.encode())

        if row:
            # Extract user features
            avg_amount = float(row.cells['stats']['avg_amount'][0].value.decode())
            transaction_count = int(row.cells['stats']['count'][0].value.decode())

            # Fraud detection logic
            amount = transaction['amount']
            is_fraud = (
                amount > avg_amount * 5 or  # 5x average
                transaction['country'] != row.cells['profile']['country'][0].value.decode() or
                amount > 10000
            )

            fraud_score = calculate_fraud_score(transaction, row)

            result = {
                **transaction,
                'is_fraud': is_fraud,
                'fraud_score': fraud_score,
                'avg_amount': avg_amount,
                'transaction_count': transaction_count
            }

            if is_fraud:
                yield beam.pvalue.TaggedOutput('alerts', result)

            yield result
        else:
            # New user - flag for review
            yield {**transaction, 'is_fraud': False, 'fraud_score': 0.5, 'new_user': True}

def run_fraud_pipeline():
    options = PipelineOptions([
        '--project=my-project',
        '--runner=DataflowRunner',
        '--streaming',
        '--region=us-central1',
        '--num_workers=20',
        '--max_num_workers=100',
        '--autoscaling_algorithm=THROUGHPUT_BASED'
    ])

    with beam.Pipeline(options=options) as pipeline:
        transactions = (
            pipeline
            | 'Read Pub/Sub' >> beam.io.ReadFromPubSub(subscription='projects/my-project/subscriptions/transactions')
            | 'Parse JSON' >> beam.Map(json.loads)
        )

        # Detect fraud
        results = (
            transactions
            | 'Detect Fraud' >> beam.ParDo(FraudDetection('my-project')).with_outputs('alerts', main='transactions')
        )

        # Write all transactions to BigQuery
        results.transactions | 'To BigQuery' >> beam.io.WriteToBigQuery(
            'my-project:fraud.transactions',
            schema='AUTO',
            method='STREAMING_INSERTS'
        )

        # Update user profiles in Bigtable
        results.transactions | 'Update Profiles' >> beam.ParDo(UpdateUserProfile())

        # Send alerts to Pub/Sub
        results.alerts | 'Publish Alerts' >> beam.io.WriteToPubSub(topic='projects/my-project/topics/fraud-alerts')
```

**Performance**: 100k TPS, <100ms latency, auto-scales to 100 workers

### Scenario 2: Data Lake to Warehouse ETL with CDC

**Requirements**:
- Ingest data from on-premises Oracle database (CDC)
- Store raw data in data lake (Cloud Storage)
- Transform and load to BigQuery
- Maintain data lineage
- SLA: 15-minute freshness

**Architecture**:
```
Oracle (CDC) → Datastream → Cloud Storage (Avro) → Cloud Composer {
    Dataflow (transform) → BigQuery (partitioned/clustered),
    Data Catalog (lineage)
}
```

**Cloud Composer DAG**:
```python
# CDC to BigQuery DAG
from airflow import DAG
from airflow.providers.google.cloud.operators.dataflow import DataflowCreatePythonJobOperator
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator
from airflow.providers.google.cloud.sensors.gcs import GCSObjectsWithPrefixExistenceSensor
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-engineering',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'cdc_to_bigquery',
    default_args=default_args,
    schedule_interval='*/15 * * * *',  # Every 15 minutes
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Wait for new CDC files
wait_for_cdc = GCSObjectsWithPrefixExistenceSensor(
    task_id='wait_for_cdc_files',
    bucket='cdc-bucket',
    prefix='oracle/cdc/{{ execution_date.strftime("%Y-%m-%d/%H-%M") }}',
    timeout=900,  # 15 minutes
    poke_interval=60,
    dag=dag
)

# Process CDC with Dataflow
process_cdc = DataflowCreatePythonJobOperator(
    task_id='process_cdc',
    py_file='gs://my-bucket/pipelines/cdc_processor.py',
    job_name='cdc-processor-{{ ts_nodash }}',
    options={
        'input': 'gs://cdc-bucket/oracle/cdc/{{ execution_date.strftime("%Y-%m-%d/%H-%M") }}/*.avro',
        'output_table': 'my-project:dwh.customers',
        'temp_location': 'gs://my-bucket/temp',
    },
    dag=dag
)

# Merge CDC changes into target table
merge_cdc = BigQueryInsertJobOperator(
    task_id='merge_cdc_changes',
    configuration={
        'query': {
            'query': """
                MERGE `my-project.dwh.customers` T
                USING `my-project.staging.customers_cdc` S
                ON T.customer_id = S.customer_id
                WHEN MATCHED AND S.operation = 'DELETE' THEN DELETE
                WHEN MATCHED AND S.operation = 'UPDATE' THEN UPDATE SET
                    name = S.name,
                    email = S.email,
                    updated_at = S.updated_at
                WHEN NOT MATCHED AND S.operation = 'INSERT' THEN INSERT
                    (customer_id, name, email, created_at)
                    VALUES (S.customer_id, S.name, S.email, S.created_at)
            """,
            'useLegacySql': False,
        }
    },
    dag=dag
)

wait_for_cdc >> process_cdc >> merge_cdc
```

### Scenario 3: IoT Time-Series Analytics

**Requirements**:
- Ingest 1M sensor readings/second
- Store raw data for compliance (7 years)
- Real-time dashboards (5-minute lag)
- Historical analysis capabilities
- Multi-region support

**Architecture**:
```
IoT Devices → Pub/Sub (multi-region) → Dataflow {
    → Bigtable (hot data, 90 days),
    → Cloud Storage (cold data, 7 years),
    → BigQuery (analytics, aggregated)
}
```

**Dataflow Pipeline**:
```python
import apache_beam as beam
from apache_beam import window

def run_iot_pipeline():
    options = PipelineOptions([
        '--streaming',
        '--project=my-project',
        '--region=us-central1',
        '--num_workers=50'
    ])

    with beam.Pipeline(options=options) as pipeline:
        readings = (
            pipeline
            | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(
                topic='projects/my-project/topics/iot-readings')
            | 'Parse' >> beam.Map(json.loads)
            | 'Add Timestamp' >> beam.Map(
                lambda x: beam.window.TimestampedValue(
                    x,
                    x['timestamp']
                ))
        )

        # Write raw data to Cloud Storage (cold storage)
        readings | 'To Avro' >> beam.io.WriteToAvro(
            'gs://iot-archive/readings',
            schema=avro_schema,
            file_name_suffix='.avro',
            num_shards=100
        )

        # Write to Bigtable (hot storage)
        readings | 'To Bigtable' >> beam.io.WriteToBigtable(
            project_id='my-project',
            instance_id='iot-data',
            table_id='readings'
        )

        # Aggregate for BigQuery (analytics)
        aggregated = (
            readings
            | 'Window 5min' >> beam.WindowInto(window.FixedWindows(300))
            | 'Key by Sensor' >> beam.Map(lambda x: (x['sensor_id'], x))
            | 'Group' >> beam.GroupByKey()
            | 'Aggregate' >> beam.Map(lambda x: {
                'sensor_id': x[0],
                'window_start': x[1][0]['timestamp'],
                'avg_value': sum(r['value'] for r in x[1]) / len(x[1]),
                'min_value': min(r['value'] for r in x[1]),
                'max_value': max(r['value'] for r in x[1]),
                'count': len(x[1])
            })
        )

        aggregated | 'To BigQuery' >> beam.io.WriteToBigQuery(
            'my-project:iot.sensor_metrics',
            schema='AUTO',
            method='STREAMING_INSERTS'
        )
```

**Bigtable Schema Design**:
```
Row key: sensor_id#timestamp (reverse timestamp for recent-first scan)
Column family: readings
Columns: temperature, pressure, humidity, battery_level

Example row key: sensor_12345#9999999999999999999 (max_timestamp - actual_timestamp)

Query pattern:
- Recent readings for sensor: scan prefix "sensor_12345#"
- Time range query: scan with start/end keys
```

**Cost Optimization**:
- Bigtable: $0.65/GB/month, $0.065/hour per node (SSD)
- Cloud Storage Nearline: $0.01/GB/month for archive
- BigQuery: $0.02/GB storage, $5/TB queries
- Total: ~$15K/month for 1M events/sec (with aggregation)

### Scenario 4: Multi-Cloud Data Synchronization

**Requirements**:
- Sync data between AWS S3 and GCS
- Transform during transfer
- Bi-directional sync
- Maintain consistency
- Cost-efficient transfer

**Architecture**:
```
AWS S3 → Transfer Service → GCS → Dataflow (transform) → BigQuery
GCS → AWS S3 (reverse sync)
```

**Implementation**:
```bash
# Create transfer job from AWS S3 to GCS
gcloud transfer jobs create s3://aws-bucket gs://gcs-bucket \
  --source-creds-file=aws-credentials.json \
  --schedule-starts='2024-01-01T00:00:00Z' \
  --schedule-repeats-every='1d' \
  --overwrite-when='different' \
  --delete-from='destination-if-unique'

# Transform with Dataflow after transfer
# Triggered by Cloud Function on GCS write
```

**Cloud Function Trigger**:
```python
from google.cloud import dataflow_v1beta3
import os

def trigger_dataflow(event, context):
    """Trigger Dataflow when file lands in GCS"""
    file_name = event['name']

    if file_name.endswith('.csv'):
        client = dataflow_v1beta3.FlexTemplatesServiceClient()

        request = dataflow_v1beta3.LaunchFlexTemplateRequest(
            project_id=os.environ['PROJECT_ID'],
            launch_template=dataflow_v1beta3.LaunchFlexTemplateParameter(
                job_name=f'transform-{file_name.replace("/", "-")}',
                container_spec_gcs_path='gs://my-bucket/templates/transform.json',
                parameters={
                    'input': f'gs://{event["bucket"]}/{file_name}',
                    'output': f'{os.environ["PROJECT_ID"]}:dataset.table'
                }
            )
        )

        response = client.launch_flex_template(request=request)
        print(f'Launched job: {response.job.id}')
```

### Scenario 5: Machine Learning Feature Store Pipeline

**Requirements**:
- Extract features from multiple sources
- Transform and aggregate features
- Store in low-latency feature store
- Support online and offline serving
- Version feature definitions

**Architecture**:
```
Sources (BigQuery, Bigtable, Spanner) → Dataflow → Vertex AI Feature Store {
    Online serving (Bigtable),
    Offline serving (BigQuery)
}
```

**Feature Engineering Pipeline**:
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

class ExtractUserFeatures(beam.DoFn):
    def process(self, element):
        """Extract and engineer user features"""
        user_id = element['user_id']

        features = {
            'user_id': user_id,
            'total_purchases': element['purchase_count'],
            'avg_purchase_amount': element['total_revenue'] / element['purchase_count'] if element['purchase_count'] > 0 else 0,
            'days_since_last_purchase': (datetime.now() - element['last_purchase_date']).days,
            'favorite_category': element['top_category'],
            'is_premium': element['total_revenue'] > 1000,
            'purchase_frequency': element['purchase_count'] / element['days_active'] if element['days_active'] > 0 else 0,
        }

        yield features

def run_feature_pipeline():
    options = PipelineOptions([
        '--project=my-project',
        '--runner=DataflowRunner',
        '--region=us-central1'
    ])

    with beam.Pipeline(options=options) as pipeline:
        # Extract features from BigQuery
        user_features = (
            pipeline
            | 'Read from BigQuery' >> beam.io.ReadFromBigQuery(
                query="""
                    SELECT
                        user_id,
                        COUNT(DISTINCT order_id) as purchase_count,
                        SUM(amount) as total_revenue,
                        MAX(order_date) as last_purchase_date,
                        MODE(category) as top_category,
                        DATE_DIFF(CURRENT_DATE(), MIN(registration_date), DAY) as days_active
                    FROM `my-project.ecommerce.orders`
                    GROUP BY user_id
                """,
                use_standard_sql=True
            )
            | 'Engineer Features' >> beam.ParDo(ExtractUserFeatures())
        )

        # Write to Vertex AI Feature Store (online)
        user_features | 'To Feature Store' >> beam.ParDo(WriteToFeatureStore())

        # Write to BigQuery (offline)
        user_features | 'To BigQuery' >> beam.io.WriteToBigQuery(
            'my-project:ml.user_features',
            schema='AUTO',
            write_disposition=beam.io.BigQueryDisposition.WRITE_TRUNCATE
        )
```

## Service Selection Matrix

| Requirement | BigQuery | Dataflow | Dataproc | Data Fusion | Pub/Sub |
|------------|----------|----------|----------|-------------|---------|
| **SQL queries** | Best | Limited | Via Spark SQL | Via transforms | No |
| **Streaming** | Inserts only | Excellent | Good | Enterprise only | Excellent |
| **Batch** | Excellent | Excellent | Excellent | Excellent | No |
| **No-code** | Partial | No | No | Yes | No |
| **Custom code** | UDFs | Full control | Full control | Limited | No |
| **Latency** | Seconds | Sub-second | Seconds | Minutes | Milliseconds |
| **Scale** | Petabytes | Unlimited | TBs | TBs | Unlimited |
| **Cost (relative)** | Low | Medium | Low-Medium | Medium | Low |
| **Ops overhead** | None | Low | Medium | Low | None |

## Performance Benchmarks

### BigQuery vs Dataproc (Spark)

**Query**: Aggregate 1TB of data (COUNT, SUM, AVG by dimensions)

| Metric | BigQuery | Dataproc (50 workers) |
|--------|----------|----------------------|
| **Query time** | 23 seconds | 4.2 minutes |
| **Setup time** | 0 (serverless) | 2-3 minutes |
| **Total time** | 23 seconds | 6-7 minutes |
| **Cost** | $5 ($5/TB) | $12 (compute time) |
| **Optimization** | Partition + cluster | Caching + tuning |

**Takeaway**: BigQuery faster for ad-hoc analytics, Dataproc better for iterative ML workloads

### Dataflow vs Cloud Functions

**Workload**: Process 100M JSON messages, enrich, write to BigQuery

| Metric | Dataflow (streaming) | Cloud Functions (Pub/Sub trigger) |
|--------|---------------------|-----------------------------------|
| **Throughput** | 100k msg/sec | 10k msg/sec (concurrent limit) |
| **Latency** | 2-5 seconds | 100ms-1sec per message |
| **Cost (24hr)** | $150 (workers) | $240 (invocations) |
| **Use case** | High throughput | Low volume, simple logic |

**Takeaway**: Dataflow for high-volume streaming, Cloud Functions for event-driven microservices

## Exam Preparation Tips

### Common Exam Scenarios

**1. Service Selection Questions**:
- Question pattern: "Which service for real-time analytics with 10ms latency?"
- Answer approach: Consider latency, throughput, complexity, cost
- Bigtable for <10ms, BigQuery for SQL analytics, Dataflow for transformations

**2. Cost Optimization**:
- Dataproc: Preemptible workers, autoscaling, ephemeral clusters
- BigQuery: Partitioning, clustering, flat-rate pricing for predictable workloads
- Dataflow: Right-size workers, use Streaming Engine, batch vs streaming

**3. Streaming Window Questions**:
- Fixed vs sliding vs session windows
- Early/late data handling with watermarks
- Allowed lateness configuration

**4. Data Migration**:
- Online (Datastream) vs offline (Transfer Service)
- Homogeneous vs heterogeneous migrations
- Validation strategies

**5. Orchestration**:
- Composer for complex workflows
- Workflows for simple state machines
- Scheduler for simple cron jobs

### Key Formulas

**Partition Pruning Savings**:
```
Cost reduction = (1 - partitions_scanned/total_partitions) * 100%
Example: Query 1 day of 365-day partitioned table = 99.7% cost reduction
```

**Clustering Benefit**:
```
Bytes scanned reduction = Depends on data distribution
Typical: 30-70% reduction for well-clustered data
```

**Dataflow Cost**:
```
Hourly cost = num_workers * worker_cost * hours
worker_cost = (vCPU_cost + memory_cost + disk_cost)
```

### Must-Know Concepts

1. **BigQuery**:
   - Partition types (time-ingestion, field-based)
   - Clustering (up to 4 columns)
   - Slot allocation (on-demand vs flat-rate)
   - Streaming buffer behavior

2. **Dataflow**:
   - Window types (fixed, sliding, session, global)
   - Triggers (watermark, processing time, count)
   - Side inputs vs side outputs
   - ParDo vs Combine vs GroupByKey

3. **Pub/Sub**:
   - Ordering keys for FIFO
   - Dead letter topics for error handling
   - Message retention (7 days)
   - Exactly-once delivery patterns

4. **Dataproc**:
   - Ephemeral vs persistent clusters
   - Preemptible workers for cost savings
   - BigQuery connector for direct access
   - Workflow templates for orchestration

5. **Composer/Airflow**:
   - DAG structure (operators, sensors, dependencies)
   - XCom for data passing
   - Trigger rules (all_success, one_failed, all_done)
   - Pools for resource management

### Exam Question Patterns

**Pattern 1: Minimize Cost**
"Company wants to reduce BigQuery costs for queries on 5-year dataset"
- Answer: Partition by date, cluster by common filters, use materialized views

**Pattern 2: Minimize Latency**
"Real-time dashboard needs <1 second data freshness"
- Answer: Pub/Sub → Dataflow → Bigtable (for low latency) + BigQuery (for analytics)

**Pattern 3: Handle Late Data**
"Streaming pipeline receives events up to 1 hour late"
- Answer: Use watermarks with allowed_lateness, AfterWatermark trigger with late firing

**Pattern 4: Data Quality**
"Ensure no duplicate records in BigQuery from streaming inserts"
- Answer: Use insertId in streaming API, or use Dataflow with deduplication

**Pattern 5: Failure Handling**
"Some messages fail processing repeatedly"
- Answer: Configure dead letter topic with max_delivery_attempts

## Common Scenarios

**Scenario**: Real-time clickstream analytics
**Solution**: Web servers → Pub/Sub → Dataflow → BigQuery, with Cloud Functions for alerting

**Scenario**: Daily batch ETL from on-premises
**Solution**: Data Transfer Service → Cloud Storage → Dataflow → BigQuery, orchestrated by Cloud Composer

**Scenario**: IoT sensor data processing
**Solution**: IoT devices → Pub/Sub → Dataflow → Bigtable (hot data) + Cloud Storage (cold data)

**Scenario**: Data warehouse migration from Teradata
**Solution**: Assessment → BigQuery schema design → Transfer Service for data → Query migration → Validation

## Study Tips

1. **Understand service selection**: Know when to use each storage and processing service
2. **Pipeline patterns**: Batch vs. streaming, lambda vs. kappa
3. **Hands-on practice**: Build pipelines with Dataflow and Composer
4. **BigQuery optimization**: Partitioning, clustering, query optimization
5. **Migration strategies**: Database and data warehouse migration approaches
6. **Data quality**: Validation and monitoring strategies
7. **Cost optimization**: Resource sizing, committed use, query optimization

## Key Commands

```bash
# Dataflow
gcloud dataflow jobs run JOB_NAME --gcs-location=gs://template-bucket/template --region=us-central1

# Dataproc
gcloud dataproc clusters create CLUSTER --region=us-central1 --num-workers=2 --worker-machine-type=n1-standard-4 --enable-component-gateway
gcloud dataproc jobs submit spark --cluster=CLUSTER --region=us-central1 --jar=gs://bucket/job.jar

# BigQuery
bq mk --dataset PROJECT:DATASET
bq load --source_format=CSV DATASET.TABLE gs://bucket/data.csv schema.json
bq query --use_legacy_sql=false 'SELECT * FROM DATASET.TABLE LIMIT 10'

# Data Transfer
gcloud transfer jobs create gs://source-bucket gs://dest-bucket --name=my-transfer
```

## Additional Resources

- [Data Engineering on GCP](https://cloud.google.com/architecture/data-engineering)
- [Dataflow Documentation](https://cloud.google.com/dataflow/docs)
- [BigQuery Best Practices](https://cloud.google.com/bigquery/docs/best-practices)
- [Data Migration Guides](https://cloud.google.com/architecture/migration-to-gcp-getting-started)
