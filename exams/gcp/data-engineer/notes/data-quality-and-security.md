# Data Quality and Security - GCP Professional Data Engineer

## Overview

Comprehensive guide to data quality, governance, security, and compliance for data engineering on GCP. This document covers validation frameworks, access control patterns, encryption strategies, DLP implementations, and regulatory compliance for the Professional Data Engineer certification exam.

**Exam Weight**: Data quality and security questions appear throughout the exam, representing approximately 20-25% of total questions. Expect scenario-based questions on security architecture, compliance implementations, and data governance patterns.

## Key Topics

1. **Data Quality Frameworks** - Validation, profiling, cleansing, monitoring, anomaly detection
2. **Data Catalog & Metadata** - Cataloging, tagging, lineage tracking, policy tags
3. **Data Security** - Encryption (CMEK/CSEK), access control, DLP, audit logging
4. **IAM for Data Services** - Row-level security, column-level security, authorized views
5. **VPC Service Controls** - Data exfiltration prevention, service perimeters
6. **Data Governance** - Dataplex, lifecycle management, retention policies
7. **Compliance Frameworks** - GDPR, HIPAA, SOC2, PCI-DSS, regulatory requirements
8. **Audit & Monitoring** - Cloud Audit Logs, compliance reporting, data residency

## Data Quality Frameworks

### Data Quality Dimensions

**The Six Dimensions of Data Quality**:
1. **Accuracy**: Data correctly represents real-world entities
2. **Completeness**: All required data is present
3. **Consistency**: Data is uniform across datasets
4. **Timeliness**: Data is available when needed
5. **Validity**: Data conforms to defined formats and rules
6. **Uniqueness**: No unwanted duplicates exist

### Data Validation

**At Ingestion (Schema Enforcement)**:
- Schema validation with Pub/Sub schema registry
- Data type checking and coercion
- Range and constraint validation
- Null value handling strategies
- Duplicate detection and deduplication
- Format validation (dates, emails, phone numbers)
- Referential integrity checks

**During Processing (Business Rules)**:
- Business rule validation in Dataflow
- Cross-field validation
- Referential integrity across datasets
- Data completeness checks
- Consistency validation
- Statistical anomaly detection
- Temporal consistency checks

**Post-Processing (Quality Metrics)**:
- Data quality score calculation
- Comparison with historical baselines
- Drift detection
- SLA compliance verification

**Tools and Services**:
- **Dataplex Data Quality**: Managed data quality service
- **Custom Dataflow Validation**: Programmable validation logic
- **BigQuery Data Quality Checks**: SQL-based validation
- **Great Expectations**: Open-source data validation framework
- **dbt tests**: SQL-based data testing
- **Apache Beam ValidateRunner**: Pipeline validation

### Schema Validation Implementation

**Pub/Sub Schema Enforcement**:
```python
from google.cloud import pubsub_v1
from google.pubsub_v1.types import Encoding

# Create publisher with schema
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path(project_id, topic_id)

# Define Avro schema
avro_schema = {
    "type": "record",
    "name": "CustomerEvent",
    "fields": [
        {"name": "customer_id", "type": "string"},
        {"name": "email", "type": "string"},
        {"name": "purchase_amount", "type": "double"},
        {"name": "timestamp", "type": "long"}
    ]
}

# Create schema
schema_client = pubsub_v1.SchemaServiceClient()
schema_path = schema_client.schema_path(project_id, schema_id)
schema = pubsub_v1.types.Schema(
    name=schema_path,
    type_=pubsub_v1.types.Schema.Type.AVRO,
    definition=json.dumps(avro_schema)
)
created_schema = schema_client.create_schema(
    request={"parent": f"projects/{project_id}", "schema": schema, "schema_id": schema_id}
)

# Publish with schema validation
data = {"customer_id": "C123", "email": "user@example.com",
        "purchase_amount": 99.99, "timestamp": 1234567890}
encoded_data = json.dumps(data).encode("utf-8")
future = publisher.publish(topic_path, encoded_data)
```

**BigQuery Schema Validation**:
```python
from google.cloud import bigquery

def validate_and_load_with_schema(project_id, dataset_id, table_id, source_uri):
    """Load data with strict schema validation"""
    client = bigquery.Client(project=project_id)

    # Define strict schema
    schema = [
        bigquery.SchemaField("customer_id", "STRING", mode="REQUIRED"),
        bigquery.SchemaField("email", "STRING", mode="REQUIRED"),
        bigquery.SchemaField("age", "INTEGER", mode="NULLABLE"),
        bigquery.SchemaField("purchase_amount", "NUMERIC", mode="REQUIRED"),
        bigquery.SchemaField("created_at", "TIMESTAMP", mode="REQUIRED"),
    ]

    table_ref = client.dataset(dataset_id).table(table_id)

    job_config = bigquery.LoadJobConfig(
        schema=schema,
        source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,
        # Strict validation - reject rows that don't match schema
        schema_update_options=[],
        max_bad_records=0,  # Don't tolerate any bad records
        write_disposition=bigquery.WriteDisposition.WRITE_APPEND,
    )

    load_job = client.load_table_from_uri(
        source_uri, table_ref, job_config=job_config
    )

    try:
        load_job.result()  # Wait for job to complete
        print(f"Loaded {load_job.output_rows} rows")
    except Exception as e:
        print(f"Validation failed: {e}")
        # Log validation errors for review
        for error in load_job.errors:
            print(f"Error: {error['message']}")
```

### Dataflow Data Validation Pipeline

**Comprehensive Validation Example**:
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from apache_beam.metrics import Metrics
import re
from datetime import datetime

class ValidationMetrics:
    """Track validation metrics"""
    def __init__(self):
        self.valid_records = Metrics.counter('validation', 'valid_records')
        self.invalid_email = Metrics.counter('validation', 'invalid_email')
        self.invalid_age = Metrics.counter('validation', 'invalid_age')
        self.missing_required = Metrics.counter('validation', 'missing_required')
        self.duplicate_records = Metrics.counter('validation', 'duplicate_records')

class ValidateRecord(beam.DoFn):
    """Comprehensive record validation"""

    def __init__(self):
        self.metrics = ValidationMetrics()
        self.email_pattern = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')

    def process(self, element):
        errors = []

        # Required field validation
        required_fields = ['customer_id', 'email', 'purchase_amount', 'timestamp']
        for field in required_fields:
            if field not in element or element[field] is None:
                errors.append(f"Missing required field: {field}")
                self.metrics.missing_required.inc()

        if errors:
            yield beam.pvalue.TaggedOutput('invalid', {'record': element, 'errors': errors})
            return

        # Email format validation
        if not self.email_pattern.match(element['email']):
            errors.append(f"Invalid email format: {element['email']}")
            self.metrics.invalid_email.inc()

        # Age range validation
        if 'age' in element and element['age'] is not None:
            if element['age'] < 0 or element['age'] > 150:
                errors.append(f"Age out of valid range: {element['age']}")
                self.metrics.invalid_age.inc()

        # Amount validation
        if element['purchase_amount'] < 0:
            errors.append(f"Negative purchase amount: {element['purchase_amount']}")

        # Timestamp validation
        try:
            ts = datetime.fromtimestamp(element['timestamp'])
            if ts > datetime.now():
                errors.append(f"Future timestamp: {element['timestamp']}")
        except (ValueError, OSError):
            errors.append(f"Invalid timestamp: {element['timestamp']}")

        if errors:
            yield beam.pvalue.TaggedOutput('invalid', {'record': element, 'errors': errors})
        else:
            self.metrics.valid_records.inc()
            yield element

class DeduplicateRecords(beam.DoFn):
    """Detect and handle duplicates"""

    def __init__(self):
        self.seen_ids = set()
        self.metrics = ValidationMetrics()

    def process(self, element):
        record_id = element['customer_id']
        if record_id in self.seen_ids:
            self.metrics.duplicate_records.inc()
            yield beam.pvalue.TaggedOutput('duplicates', element)
        else:
            self.seen_ids.add(record_id)
            yield element

def run_validation_pipeline(project, input_topic, output_table, error_table):
    """Run comprehensive data validation pipeline"""

    options = PipelineOptions(
        project=project,
        streaming=True,
        save_main_session=True
    )

    with beam.Pipeline(options=options) as pipeline:

        # Read from Pub/Sub
        raw_messages = (
            pipeline
            | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(topic=input_topic)
            | 'Parse JSON' >> beam.Map(lambda x: json.loads(x.decode('utf-8')))
        )

        # Validate records
        validation_results = (
            raw_messages
            | 'Validate Records' >> beam.ParDo(ValidateRecord()).with_outputs('invalid', main='valid')
        )

        # Deduplicate valid records
        dedup_results = (
            validation_results.valid
            | 'Deduplicate' >> beam.ParDo(DeduplicateRecords()).with_outputs('duplicates', main='unique')
        )

        # Write valid records to BigQuery
        (
            dedup_results.unique
            | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
                output_table,
                schema='customer_id:STRING,email:STRING,age:INTEGER,purchase_amount:NUMERIC,timestamp:TIMESTAMP',
                create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )

        # Write invalid records to error table
        (
            validation_results.invalid
            | 'Write Errors' >> beam.io.WriteToBigQuery(
                error_table,
                schema='record:STRING,errors:STRING,validation_timestamp:TIMESTAMP',
                create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )

        # Write duplicates to separate table
        (
            dedup_results.duplicates
            | 'Write Duplicates' >> beam.io.WriteToBigQuery(
                f"{output_table}_duplicates",
                schema='customer_id:STRING,email:STRING,age:INTEGER,purchase_amount:NUMERIC,timestamp:TIMESTAMP',
                create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )
```

### Data Profiling

**Profile Metrics Categories**:
1. **Volume Metrics**: Row counts, table sizes, growth rates
2. **Completeness Metrics**: Null percentages, missing values
3. **Distribution Metrics**: Value distributions, cardinality, outliers
4. **Format Metrics**: Data types, pattern conformance
5. **Statistical Metrics**: Mean, median, mode, std dev, quartiles
6. **Relationship Metrics**: Foreign key validity, join success rates

**Comprehensive BigQuery Profiling**:
```sql
-- Complete data profile for a table
WITH base_stats AS (
  SELECT
    COUNT(*) as total_rows,
    COUNT(DISTINCT customer_id) as distinct_customers,
    COUNTIF(customer_id IS NULL) as null_customer_ids,
    COUNTIF(email IS NULL) as null_emails,
    COUNTIF(age IS NULL) as null_ages,
    COUNTIF(age < 0 OR age > 150) as invalid_ages,
    MIN(purchase_amount) as min_purchase,
    MAX(purchase_amount) as max_purchase,
    AVG(purchase_amount) as avg_purchase,
    STDDEV(purchase_amount) as stddev_purchase,
    APPROX_QUANTILES(purchase_amount, 4) as purchase_quartiles,
    MIN(created_at) as earliest_record,
    MAX(created_at) as latest_record
  FROM `project.dataset.customers`
),
email_patterns AS (
  SELECT
    COUNTIF(REGEXP_CONTAINS(email, r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')) as valid_emails,
    COUNTIF(NOT REGEXP_CONTAINS(email, r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')) as invalid_emails
  FROM `project.dataset.customers`
  WHERE email IS NOT NULL
),
duplicate_check AS (
  SELECT
    COUNT(*) as duplicate_customers
  FROM (
    SELECT customer_id, COUNT(*) as cnt
    FROM `project.dataset.customers`
    GROUP BY customer_id
    HAVING COUNT(*) > 1
  )
)
SELECT
  -- Volume metrics
  b.total_rows,
  b.distinct_customers,
  ROUND(b.distinct_customers * 100.0 / b.total_rows, 2) as customer_uniqueness_pct,

  -- Completeness metrics
  b.null_customer_ids,
  ROUND(b.null_customer_ids * 100.0 / b.total_rows, 2) as null_customer_id_pct,
  b.null_emails,
  ROUND(b.null_emails * 100.0 / b.total_rows, 2) as null_email_pct,
  b.null_ages,
  ROUND(b.null_ages * 100.0 / b.total_rows, 2) as null_age_pct,

  -- Validity metrics
  b.invalid_ages,
  ROUND(b.invalid_ages * 100.0 / b.total_rows, 2) as invalid_age_pct,
  e.valid_emails,
  e.invalid_emails,
  ROUND(e.valid_emails * 100.0 / (e.valid_emails + e.invalid_emails), 2) as email_validity_pct,

  -- Statistical metrics
  ROUND(b.min_purchase, 2) as min_purchase,
  ROUND(b.max_purchase, 2) as max_purchase,
  ROUND(b.avg_purchase, 2) as avg_purchase,
  ROUND(b.stddev_purchase, 2) as stddev_purchase,
  ROUND(b.purchase_quartiles[OFFSET(1)], 2) as purchase_q1,
  ROUND(b.purchase_quartiles[OFFSET(2)], 2) as purchase_median,
  ROUND(b.purchase_quartiles[OFFSET(3)], 2) as purchase_q3,

  -- Temporal metrics
  b.earliest_record,
  b.latest_record,
  TIMESTAMP_DIFF(b.latest_record, b.earliest_record, DAY) as data_span_days,

  -- Uniqueness metrics
  d.duplicate_customers,
  ROUND(d.duplicate_customers * 100.0 / b.distinct_customers, 2) as duplicate_pct

FROM base_stats b, email_patterns e, duplicate_check d;
```

**Automated Profiling with Dataplex**:
```python
from google.cloud import dataplex_v1

def create_data_profile_scan(project_id, location, lake_id, zone_id, entity_id):
    """Create a Dataplex data profile scan"""

    client = dataplex_v1.DataScanServiceClient()

    parent = f"projects/{project_id}/locations/{location}"

    data_scan = dataplex_v1.DataScan(
        data=dataplex_v1.DataSource(
            entity=f"projects/{project_id}/locations/{location}/lakes/{lake_id}/zones/{zone_id}/entities/{entity_id}"
        ),
        data_profile_spec=dataplex_v1.DataProfileSpec(),
        execution_spec=dataplex_v1.DataScan.ExecutionSpec(
            trigger=dataplex_v1.Trigger(
                schedule=dataplex_v1.Trigger.Schedule(cron="0 2 * * *")  # Daily at 2 AM
            )
        )
    )

    request = dataplex_v1.CreateDataScanRequest(
        parent=parent,
        data_scan=data_scan,
        data_scan_id="customer-profile-scan"
    )

    operation = client.create_data_scan(request=request)
    response = operation.result()
    print(f"Created data profile scan: {response.name}")
    return response

def get_profile_results(project_id, location, scan_id):
    """Retrieve data profile results"""

    client = dataplex_v1.DataScanServiceClient()

    name = f"projects/{project_id}/locations/{location}/dataScans/{scan_id}"

    # Get latest scan job
    request = dataplex_v1.ListDataScanJobsRequest(parent=name, page_size=1)
    jobs = client.list_data_scan_jobs(request=request)

    for job in jobs:
        if job.data_profile_result:
            profile = job.data_profile_result
            print(f"Row count: {profile.row_count}")
            print(f"Profile time: {job.end_time}")

            # Access field profiles
            for field_name, field_profile in profile.profile.fields.items():
                print(f"\nField: {field_name}")
                print(f"  Null ratio: {field_profile.null_ratio}")
                print(f"  Distinct ratio: {field_profile.distinct_ratio}")
```

### Anomaly Detection

**Statistical Anomaly Detection**:
```sql
-- Detect anomalies using z-score method
WITH stats AS (
  SELECT
    AVG(purchase_amount) as mean,
    STDDEV(purchase_amount) as stddev
  FROM `project.dataset.purchases`
  WHERE DATE(purchase_date) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
),
z_scores AS (
  SELECT
    purchase_id,
    customer_id,
    purchase_amount,
    purchase_date,
    (purchase_amount - s.mean) / NULLIF(s.stddev, 0) as z_score
  FROM `project.dataset.purchases` p
  CROSS JOIN stats s
  WHERE DATE(purchase_date) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
)
SELECT
  purchase_id,
  customer_id,
  purchase_amount,
  purchase_date,
  z_score,
  CASE
    WHEN ABS(z_score) > 3 THEN 'High Anomaly'
    WHEN ABS(z_score) > 2 THEN 'Medium Anomaly'
    ELSE 'Normal'
  END as anomaly_severity
FROM z_scores
WHERE ABS(z_score) > 2
ORDER BY ABS(z_score) DESC;
```

**Time-Series Anomaly Detection**:
```sql
-- Detect anomalies in daily metrics using moving average
WITH daily_metrics AS (
  SELECT
    DATE(created_at) as date,
    COUNT(*) as daily_count,
    AVG(purchase_amount) as daily_avg_amount
  FROM `project.dataset.purchases`
  WHERE DATE(created_at) >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)
  GROUP BY date
),
moving_stats AS (
  SELECT
    date,
    daily_count,
    daily_avg_amount,
    AVG(daily_count) OVER (
      ORDER BY date
      ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING
    ) as moving_avg_count,
    STDDEV(daily_count) OVER (
      ORDER BY date
      ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING
    ) as moving_stddev_count
  FROM daily_metrics
)
SELECT
  date,
  daily_count,
  moving_avg_count,
  moving_stddev_count,
  daily_count - moving_avg_count as deviation,
  (daily_count - moving_avg_count) / NULLIF(moving_stddev_count, 0) as z_score,
  CASE
    WHEN ABS((daily_count - moving_avg_count) / NULLIF(moving_stddev_count, 0)) > 3
      THEN 'Anomaly Detected'
    ELSE 'Normal'
  END as status
FROM moving_stats
WHERE moving_stddev_count IS NOT NULL
ORDER BY date DESC;
```

### Dataplex Data Quality

**Create Data Quality Rules**:
```python
from google.cloud import dataplex_v1

def create_data_quality_scan(project_id, location, lake_id, zone_id, entity_id):
    """Create comprehensive data quality scan with Dataplex"""

    client = dataplex_v1.DataScanServiceClient()
    parent = f"projects/{project_id}/locations/{location}"

    # Define data quality rules
    data_quality_spec = dataplex_v1.DataQualitySpec(
        rules=[
            # Row condition rules
            dataplex_v1.DataQualityRule(
                row_condition_expectation=dataplex_v1.DataQualityRule.RowConditionExpectation(
                    sql_expression="age >= 0 AND age <= 150"
                ),
                dimension="VALIDITY",
                threshold=0.95,  # 95% of rows must pass
                name="valid_age_range"
            ),
            # Non-null expectation
            dataplex_v1.DataQualityRule(
                non_null_expectation=dataplex_v1.DataQualityRule.NonNullExpectation(
                    column="customer_id"
                ),
                dimension="COMPLETENESS",
                threshold=1.0,  # 100% non-null
                name="customer_id_not_null"
            ),
            # Uniqueness expectation
            dataplex_v1.DataQualityRule(
                uniqueness_expectation=dataplex_v1.DataQualityRule.UniquenessExpectation(
                    column="email"
                ),
                dimension="UNIQUENESS",
                threshold=0.99,  # 99% unique
                name="email_uniqueness"
            ),
            # Range expectation
            dataplex_v1.DataQualityRule(
                range_expectation=dataplex_v1.DataQualityRule.RangeExpectation(
                    column="purchase_amount",
                    min_value="0",
                    max_value="1000000",
                    strict_min_enabled=True,
                    strict_max_enabled=False
                ),
                dimension="VALIDITY",
                threshold=0.99,
                name="purchase_amount_range"
            ),
            # Set expectation (value in list)
            dataplex_v1.DataQualityRule(
                set_expectation=dataplex_v1.DataQualityRule.SetExpectation(
                    column="country_code",
                    values=["US", "UK", "CA", "AU", "DE", "FR"]
                ),
                dimension="VALIDITY",
                threshold=0.95,
                name="valid_country_codes"
            ),
            # Regex expectation
            dataplex_v1.DataQualityRule(
                regex_expectation=dataplex_v1.DataQualityRule.RegexExpectation(
                    column="email",
                    regex=r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
                ),
                dimension="VALIDITY",
                threshold=0.98,
                name="email_format"
            ),
            # Statistical range expectation
            dataplex_v1.DataQualityRule(
                statistic_range_expectation=dataplex_v1.DataQualityRule.StatisticRangeExpectation(
                    column="age",
                    statistic=dataplex_v1.DataQualityRule.StatisticRangeExpectation.Statistic.MEAN,
                    min_value="25",
                    max_value="45"
                ),
                dimension="VALIDITY",
                name="age_mean_check"
            ),
            # Table condition expectation
            dataplex_v1.DataQualityRule(
                table_condition_expectation=dataplex_v1.DataQualityRule.TableConditionExpectation(
                    sql_expression="COUNT(*) > 1000"
                ),
                dimension="COMPLETENESS",
                name="minimum_row_count"
            )
        ],
        sampling_percent=100.0,  # Sample 100% of data
        row_filter="DATE(created_at) >= CURRENT_DATE() - 7"  # Last 7 days only
    )

    data_scan = dataplex_v1.DataScan(
        data=dataplex_v1.DataSource(
            entity=f"projects/{project_id}/locations/{location}/lakes/{lake_id}/zones/{zone_id}/entities/{entity_id}"
        ),
        data_quality_spec=data_quality_spec,
        execution_spec=dataplex_v1.DataScan.ExecutionSpec(
            trigger=dataplex_v1.Trigger(
                schedule=dataplex_v1.Trigger.Schedule(cron="0 */6 * * *")  # Every 6 hours
            )
        )
    )

    request = dataplex_v1.CreateDataScanRequest(
        parent=parent,
        data_scan=data_scan,
        data_scan_id="customer-quality-scan"
    )

    operation = client.create_data_scan(request=request)
    response = operation.result()
    print(f"Created data quality scan: {response.name}")
    return response

def run_data_quality_scan_on_demand(project_id, location, scan_id):
    """Run data quality scan immediately"""

    client = dataplex_v1.DataScanServiceClient()
    name = f"projects/{project_id}/locations/{location}/dataScans/{scan_id}"

    request = dataplex_v1.RunDataScanRequest(name=name)
    response = client.run_data_scan(request=request)

    print(f"Started data quality scan job: {response.job.name}")
    return response.job

def monitor_quality_results(project_id, location, scan_id):
    """Monitor and alert on data quality results"""

    client = dataplex_v1.DataScanServiceClient()
    name = f"projects/{project_id}/locations/{location}/dataScans/{scan_id}"

    request = dataplex_v1.ListDataScanJobsRequest(parent=name, page_size=1)
    jobs = client.list_data_scan_jobs(request=request)

    for job in jobs:
        if job.data_quality_result:
            result = job.data_quality_result

            print(f"Overall score: {result.score * 100:.2f}%")
            print(f"Passed: {result.passed}")
            print(f"Rows scanned: {result.row_count}")

            # Check individual rules
            for rule_result in result.rules:
                print(f"\nRule: {rule_result.rule.name}")
                print(f"  Dimension: {rule_result.rule.dimension}")
                print(f"  Passed: {rule_result.passed}")
                print(f"  Pass ratio: {rule_result.pass_ratio * 100:.2f}%")
                print(f"  Threshold: {rule_result.rule.threshold * 100:.2f}%")

                if not rule_result.passed:
                    print(f"  ❌ FAILED: {rule_result.failing_rows_query}")

            # Send alert if overall quality is below threshold
            if result.score < 0.95:
                print(f"⚠️ ALERT: Data quality below threshold!")
                # Integrate with Cloud Monitoring or notification system
```

### Data Cleansing

**Common Operations**:
- Remove duplicates and identify canonical records
- Handle missing values (imputation, removal, forward/backward fill)
- Standardize formats (dates, addresses, phone numbers, currency)
- Correct data types and coerce values
- Fix encoding issues (UTF-8 normalization)
- Normalize values (case, whitespace, special characters)
- Resolve inconsistencies across sources

**Advanced Dataflow Cleansing Pipeline**:
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
import re
from datetime import datetime
import phonenumbers

class DataCleansingTransform(beam.DoFn):
    """Comprehensive data cleansing"""

    def __init__(self):
        self.email_pattern = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')

    def process(self, element):
        cleaned = {}

        # Email standardization
        if 'email' in element and element['email']:
            email = element['email'].lower().strip()
            cleaned['email'] = email if self.email_pattern.match(email) else None
        else:
            cleaned['email'] = None

        # Phone number standardization
        if 'phone' in element and element['phone']:
            try:
                parsed = phonenumbers.parse(element['phone'], "US")
                cleaned['phone'] = phonenumbers.format_number(
                    parsed, phonenumbers.PhoneNumberFormat.E164
                )
            except:
                cleaned['phone'] = None
        else:
            cleaned['phone'] = None

        # Name standardization
        if 'name' in element and element['name']:
            name = element['name'].strip()
            # Title case, handle multiple spaces
            name = ' '.join(name.split())
            cleaned['name'] = name.title()
        else:
            cleaned['name'] = None

        # Address normalization
        if 'address' in element and element['address']:
            address = element['address'].strip()
            address = re.sub(r'\s+', ' ', address)  # Normalize whitespace
            # Standardize abbreviations
            address = address.replace(' St.', ' Street')
            address = address.replace(' Ave.', ' Avenue')
            address = address.replace(' Dr.', ' Drive')
            cleaned['address'] = address
        else:
            cleaned['address'] = None

        # Date standardization
        if 'date_of_birth' in element:
            try:
                # Parse various date formats
                date_str = str(element['date_of_birth'])
                for fmt in ['%Y-%m-%d', '%m/%d/%Y', '%d-%m-%Y']:
                    try:
                        dt = datetime.strptime(date_str, fmt)
                        cleaned['date_of_birth'] = dt.strftime('%Y-%m-%d')
                        break
                    except ValueError:
                        continue
            except:
                cleaned['date_of_birth'] = None
        else:
            cleaned['date_of_birth'] = None

        # Numeric field standardization
        if 'age' in element:
            try:
                age = int(float(element['age']))
                cleaned['age'] = age if 0 <= age <= 150 else None
            except (ValueError, TypeError):
                cleaned['age'] = None
        else:
            cleaned['age'] = None

        # Currency standardization
        if 'purchase_amount' in element:
            try:
                amount = float(str(element['purchase_amount']).replace('$', '').replace(',', ''))
                cleaned['purchase_amount'] = round(amount, 2)
            except (ValueError, TypeError):
                cleaned['purchase_amount'] = 0.0
        else:
            cleaned['purchase_amount'] = 0.0

        # Boolean standardization
        if 'is_active' in element:
            value = str(element['is_active']).lower()
            cleaned['is_active'] = value in ['true', '1', 'yes', 'y', 't']
        else:
            cleaned['is_active'] = False

        # Copy remaining fields
        for key, value in element.items():
            if key not in cleaned:
                cleaned[key] = value

        yield cleaned

def run_cleansing_pipeline(project, input_table, output_table):
    """Run data cleansing pipeline"""

    options = PipelineOptions(
        project=project,
        save_main_session=True
    )

    with beam.Pipeline(options=options) as pipeline:
        (
            pipeline
            | 'Read from BigQuery' >> beam.io.ReadFromBigQuery(
                query=f"SELECT * FROM `{input_table}`",
                use_standard_sql=True
            )
            | 'Cleanse Data' >> beam.ParDo(DataCleansingTransform())
            | 'Remove Nulls' >> beam.Filter(lambda x: x['email'] is not None)
            | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
                output_table,
                schema='SCHEMA_AUTODETECT',
                create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
                write_disposition=beam.io.BigQueryDisposition.WRITE_TRUNCATE
            )
        )
```

### Data Monitoring

**Data Quality SLIs and SLOs**:
```python
from google.cloud import monitoring_v3
import time

def create_data_quality_slo(project_id, service_id):
    """Create SLO for data quality metrics"""

    client = monitoring_v3.ServiceMonitoringServiceClient()
    service_name = f"projects/{project_id}/services/{service_id}"

    # Define SLO: 99.5% of data quality checks should pass
    slo = monitoring_v3.ServiceLevelObjective(
        display_name="Data Quality SLO",
        goal=0.995,  # 99.5%
        rolling_period={"seconds": 86400},  # 24 hours
        service_level_indicator=monitoring_v3.ServiceLevelIndicator(
            request_based=monitoring_v3.RequestBasedSli(
                good_total_ratio=monitoring_v3.TimeSeriesRatio(
                    good_service_filter='metric.type="custom.googleapis.com/data_quality/passed" AND resource.type="global"',
                    total_service_filter='metric.type="custom.googleapis.com/data_quality/total" AND resource.type="global"'
                )
            )
        )
    )

    request = monitoring_v3.CreateServiceLevelObjectiveRequest(
        parent=service_name,
        service_level_objective=slo
    )

    created_slo = client.create_service_level_objective(request=request)
    print(f"Created SLO: {created_slo.name}")
    return created_slo

def write_quality_metric(project_id, passed, total):
    """Write data quality metrics to Cloud Monitoring"""

    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{project_id}"

    # Write passed count
    series_passed = monitoring_v3.TimeSeries()
    series_passed.metric.type = "custom.googleapis.com/data_quality/passed"
    series_passed.resource.type = "global"
    series_passed.resource.labels["project_id"] = project_id

    point = monitoring_v3.Point()
    point.value.int64_value = passed
    point.interval.end_time.seconds = int(time.time())
    series_passed.points = [point]

    # Write total count
    series_total = monitoring_v3.TimeSeries()
    series_total.metric.type = "custom.googleapis.com/data_quality/total"
    series_total.resource.type = "global"
    series_total.resource.labels["project_id"] = project_id

    point_total = monitoring_v3.Point()
    point_total.value.int64_value = total
    point_total.interval.end_time.seconds = int(time.time())
    series_total.points = [point_total]

    # Write both metrics
    client.create_time_series(
        name=project_name,
        time_series=[series_passed, series_total]
    )
```

**Data Freshness Monitoring**:
```sql
-- Monitor data freshness across tables
SELECT
  table_catalog,
  table_schema,
  table_name,
  TIMESTAMP_MILLIS(creation_time) as table_created,
  TIMESTAMP_MILLIS(CAST(last_modified_time AS INT64)) as last_modified,
  TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), TIMESTAMP_MILLIS(CAST(last_modified_time AS INT64)), HOUR) as hours_since_update,
  row_count,
  size_bytes / POW(10, 9) as size_gb,
  CASE
    WHEN TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), TIMESTAMP_MILLIS(CAST(last_modified_time AS INT64)), HOUR) > 24
      THEN 'STALE'
    WHEN TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), TIMESTAMP_MILLIS(CAST(last_modified_time AS INT64)), HOUR) > 12
      THEN 'WARNING'
    ELSE 'FRESH'
  END as freshness_status
FROM `project.dataset.__TABLES__`
WHERE table_name NOT LIKE '%_backup%'
ORDER BY hours_since_update DESC;
```

**Alerting on Data Quality**:
```python
def create_quality_alert_policy(project_id):
    """Create alert when data quality drops below threshold"""

    from google.cloud import monitoring_v3

    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    # Alert when quality drops below 95%
    alert_policy = monitoring_v3.AlertPolicy(
        display_name="Data Quality Alert",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Quality below 95%",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="custom.googleapis.com/data_quality/score" AND resource.type="global"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_LT,
                    threshold_value=0.95,
                    duration={"seconds": 300},  # 5 minutes
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_MEAN,
                        )
                    ]
                )
            )
        ],
        notification_channels=[],  # Add notification channels
        alert_strategy=monitoring_v3.AlertPolicy.AlertStrategy(
            auto_close={"seconds": 3600}  # Auto-close after 1 hour
        ),
        enabled=True
    )

    policy = client.create_alert_policy(
        name=project_name,
        alert_policy=alert_policy
    )

    print(f"Created alert policy: {policy.name}")
    return policy
```

## Data Catalog and Metadata Management

### Cloud Data Catalog Overview

**Key Features**:
- **Automatic Discovery**: Auto-discovers BigQuery, Cloud Storage, Pub/Sub assets
- **Metadata Management**: Technical and business metadata
- **Search and Discovery**: Full-text search across data assets
- **Tag Templates**: Custom metadata with typed fields
- **Policy Tags**: Column-level security integration
- **Data Lineage**: Track data flow (limited, primarily for BigQuery)

### Creating and Managing Data Catalog Entries

**Create Custom Entry**:
```python
from google.cloud import datacatalog_v1

def create_custom_entry(project_id, location, entry_group_id, entry_id):
    """Create custom data catalog entry"""

    client = datacatalog_v1.DataCatalogClient()

    # Create entry group first
    entry_group = datacatalog_v1.EntryGroup()
    entry_group.display_name = "Custom Data Assets"
    entry_group.description = "Custom data assets not auto-discovered"

    entry_group_parent = f"projects/{project_id}/locations/{location}"
    created_group = client.create_entry_group(
        parent=entry_group_parent,
        entry_group_id=entry_group_id,
        entry_group=entry_group
    )

    # Create entry
    entry = datacatalog_v1.Entry()
    entry.display_name = "Customer Master Data"
    entry.description = "Authoritative source for customer information"
    entry.type_ = datacatalog_v1.EntryType.FILESET
    entry.linked_resource = "//storage.googleapis.com/my-bucket/customer-master"

    # Set schema
    entry.schema.columns.extend([
        datacatalog_v1.ColumnSchema(
            column="customer_id",
            type_="STRING",
            description="Unique customer identifier",
            mode="REQUIRED"
        ),
        datacatalog_v1.ColumnSchema(
            column="email",
            type_="STRING",
            description="Customer email address",
            mode="REQUIRED"
        ),
        datacatalog_v1.ColumnSchema(
            column="ssn",
            type_="STRING",
            description="Social Security Number (PII)",
            mode="NULLABLE"
        )
    ])

    created_entry = client.create_entry(
        parent=created_group.name,
        entry_id=entry_id,
        entry=entry
    )

    print(f"Created entry: {created_entry.name}")
    return created_entry

def search_catalog(project_id, query):
    """Search data catalog"""

    client = datacatalog_v1.DataCatalogClient()
    scope = datacatalog_v1.SearchCatalogRequest.Scope(
        include_project_ids=[project_id]
    )

    request = datacatalog_v1.SearchCatalogRequest(
        scope=scope,
        query=query,
        page_size=100
    )

    results = client.search_catalog(request=request)

    for result in results:
        print(f"Resource: {result.relative_resource_name}")
        print(f"  Type: {result.search_result_type}")
        print(f"  Subtype: {result.search_result_subtype}")
        print(f"  Display name: {result.display_name}")
        print(f"  Description: {result.description}\n")

    return results
```

### Tag Templates and Tagging

**Create Tag Template with Business Metadata**:
```python
def create_tag_template(project_id, location, template_id):
    """Create tag template for business metadata"""

    client = datacatalog_v1.DataCatalogClient()
    parent = f"projects/{project_id}/locations/{location}"

    tag_template = datacatalog_v1.TagTemplate()
    tag_template.display_name = "Data Classification Template"

    # Add fields to template
    tag_template.fields["data_owner"] = datacatalog_v1.TagTemplateField(
        display_name="Data Owner",
        type_=datacatalog_v1.FieldType(
            primitive_type=datacatalog_v1.FieldType.PrimitiveType.STRING
        ),
        is_required=True
    )

    tag_template.fields["classification"] = datacatalog_v1.TagTemplateField(
        display_name="Data Classification",
        type_=datacatalog_v1.FieldType(
            enum_type=datacatalog_v1.FieldType.EnumType(
                allowed_values=[
                    datacatalog_v1.FieldType.EnumType.EnumValue(display_name="Public"),
                    datacatalog_v1.FieldType.EnumType.EnumValue(display_name="Internal"),
                    datacatalog_v1.FieldType.EnumType.EnumValue(display_name="Confidential"),
                    datacatalog_v1.FieldType.EnumType.EnumValue(display_name="Restricted")
                ]
            )
        ),
        is_required=True
    )

    tag_template.fields["pii_data"] = datacatalog_v1.TagTemplateField(
        display_name="Contains PII",
        type_=datacatalog_v1.FieldType(
            primitive_type=datacatalog_v1.FieldType.PrimitiveType.BOOL
        ),
        is_required=True
    )

    tag_template.fields["retention_days"] = datacatalog_v1.TagTemplateField(
        display_name="Retention Period (Days)",
        type_=datacatalog_v1.FieldType(
            primitive_type=datacatalog_v1.FieldType.PrimitiveType.DOUBLE
        ),
        is_required=False
    )

    tag_template.fields["business_glossary"] = datacatalog_v1.TagTemplateField(
        display_name="Business Glossary Terms",
        type_=datacatalog_v1.FieldType(
            primitive_type=datacatalog_v1.FieldType.PrimitiveType.RICHTEXT
        ),
        is_required=False
    )

    created_template = client.create_tag_template(
        parent=parent,
        tag_template_id=template_id,
        tag_template=tag_template
    )

    print(f"Created tag template: {created_template.name}")
    return created_template

def attach_tag_to_entry(project_id, location, entry_id, template_id, tag_values):
    """Attach tag to data catalog entry"""

    client = datacatalog_v1.DataCatalogClient()

    # Get entry
    entry_name = f"projects/{project_id}/locations/{location}/entryGroups/@bigquery/entries/{entry_id}"
    template_name = f"projects/{project_id}/locations/{location}/tagTemplates/{template_id}"

    tag = datacatalog_v1.Tag()
    tag.template = template_name

    # Set tag field values
    tag.fields["data_owner"] = datacatalog_v1.TagField(
        string_value=tag_values.get("data_owner", "")
    )

    tag.fields["classification"] = datacatalog_v1.TagField(
        enum_value=datacatalog_v1.TagField.EnumValue(
            display_name=tag_values.get("classification", "Internal")
        )
    )

    tag.fields["pii_data"] = datacatalog_v1.TagField(
        bool_value=tag_values.get("pii_data", False)
    )

    if "retention_days" in tag_values:
        tag.fields["retention_days"] = datacatalog_v1.TagField(
            double_value=tag_values["retention_days"]
        )

    created_tag = client.create_tag(
        parent=entry_name,
        tag=tag
    )

    print(f"Created tag: {created_tag.name}")
    return created_tag
```

### Policy Tags for Column-Level Security

**Create Policy Taxonomy**:
```python
def create_policy_taxonomy(project_id, location):
    """Create policy taxonomy for column-level security"""

    client = datacatalog_v1.PolicyTagManagerClient()
    parent = f"projects/{project_id}/locations/{location}"

    # Create taxonomy
    taxonomy = datacatalog_v1.Taxonomy()
    taxonomy.display_name = "Data Sensitivity Taxonomy"
    taxonomy.description = "Classification for data sensitivity levels"
    taxonomy.activated_policy_types = [
        datacatalog_v1.Taxonomy.PolicyType.FINE_GRAINED_ACCESS_CONTROL
    ]

    created_taxonomy = client.create_taxonomy(
        parent=parent,
        taxonomy=taxonomy
    )

    # Create policy tags (hierarchical)
    # Level 1: Sensitive Data
    sensitive_tag = datacatalog_v1.PolicyTag()
    sensitive_tag.display_name = "Sensitive Data"
    sensitive_tag.description = "Data requiring access controls"

    created_sensitive = client.create_policy_tag(
        parent=created_taxonomy.name,
        policy_tag=sensitive_tag
    )

    # Level 2: PII (under Sensitive Data)
    pii_tag = datacatalog_v1.PolicyTag()
    pii_tag.display_name = "PII"
    pii_tag.description = "Personally Identifiable Information"
    pii_tag.parent_policy_tag = created_sensitive.name

    created_pii = client.create_policy_tag(
        parent=created_taxonomy.name,
        policy_tag=pii_tag
    )

    # Level 2: Financial (under Sensitive Data)
    financial_tag = datacatalog_v1.PolicyTag()
    financial_tag.display_name = "Financial"
    financial_tag.description = "Financial information"
    financial_tag.parent_policy_tag = created_sensitive.name

    created_financial = client.create_policy_tag(
        parent=created_taxonomy.name,
        policy_tag=financial_tag
    )

    # Level 1: Public Data
    public_tag = datacatalog_v1.PolicyTag()
    public_tag.display_name = "Public Data"
    public_tag.description = "Publicly accessible data"

    created_public = client.create_policy_tag(
        parent=created_taxonomy.name,
        policy_tag=public_tag
    )

    print(f"Created taxonomy: {created_taxonomy.name}")
    return created_taxonomy, {
        "sensitive": created_sensitive.name,
        "pii": created_pii.name,
        "financial": created_financial.name,
        "public": created_public.name
    }

def apply_policy_tag_to_column(project_id, dataset_id, table_id, column_name, policy_tag_name):
    """Apply policy tag to BigQuery column"""

    from google.cloud import bigquery

    client = bigquery.Client(project=project_id)
    table_ref = client.dataset(dataset_id).table(table_id)
    table = client.get_table(table_ref)

    # Update schema with policy tag
    new_schema = []
    for field in table.schema:
        if field.name == column_name:
            # Add policy tag to field
            new_field = bigquery.SchemaField(
                name=field.name,
                field_type=field.field_type,
                mode=field.mode,
                description=field.description,
                fields=field.fields,
                policy_tags=bigquery.PolicyTagList(names=[policy_tag_name])
            )
            new_schema.append(new_field)
        else:
            new_schema.append(field)

    table.schema = new_schema
    updated_table = client.update_table(table, ["schema"])

    print(f"Applied policy tag to {table_id}.{column_name}")
    return updated_table
```

### Data Lineage Tracking

**Query BigQuery Lineage (Limited)**:
```python
from google.cloud import datacatalog_lineage_v1

def get_lineage_events(project_id, location):
    """Get lineage events for data assets"""

    client = datacatalog_lineage_v1.LineageClient()
    parent = f"projects/{project_id}/locations/{location}"

    # List processes (data pipelines)
    processes = client.list_processes(parent=parent)

    for process in processes:
        print(f"Process: {process.display_name}")
        print(f"  Name: {process.name}")
        print(f"  Origin: {process.origin}")

        # Get lineage events for process
        events = client.list_lineage_events(parent=process.name)
        for event in events:
            print(f"  Event: {event.name}")
            for source in event.links:
                print(f"    Source: {source.source.fully_qualified_name}")
                print(f"    Target: {source.target.fully_qualified_name}")

def create_lineage_event(project_id, location, process_id):
    """Create custom lineage event"""

    client = datacatalog_lineage_v1.LineageClient()

    # Create process first
    process = datacatalog_lineage_v1.Process()
    process.display_name = "Customer Data ETL"
    process.origin = datacatalog_lineage_v1.Process.Origin(
        source_type=datacatalog_lineage_v1.Process.Origin.SourceType.CUSTOM
    )

    parent = f"projects/{project_id}/locations/{location}"
    created_process = client.create_process(
        parent=parent,
        process=process
    )

    # Create lineage event
    event = datacatalog_lineage_v1.LineageEvent()
    event.start_time = {"seconds": int(time.time())}

    # Define source and target
    source = datacatalog_lineage_v1.EntityReference(
        fully_qualified_name=f"bigquery:project.dataset.source_table"
    )

    target = datacatalog_lineage_v1.EntityReference(
        fully_qualified_name=f"bigquery:project.dataset.target_table"
    )

    event.links.append(
        datacatalog_lineage_v1.EventLink(
            source=source,
            target=target
        )
    )

    created_event = client.create_lineage_event(
        parent=created_process.name,
        lineage_event=event
    )

    print(f"Created lineage event: {created_event.name}")
    return created_event
```

## Data Security

### Encryption Strategies

**Encryption Options**:

1. **Default Encryption (Google-managed keys)**:
   - Automatic encryption at rest for all GCP services
   - No configuration required
   - Keys managed entirely by Google
   - No additional cost

2. **Customer-Managed Encryption Keys (CMEK)**:
   - Keys stored in Cloud KMS
   - Customer controls key rotation and access
   - Supports automatic and manual rotation
   - Additional KMS costs apply
   - Used for compliance requirements

3. **Customer-Supplied Encryption Keys (CSEK)**:
   - Customer provides and manages keys
   - Keys never stored by Google
   - Customer responsibility for key availability
   - Limited to Cloud Storage and Compute Engine

**Encryption in Transit**:
- All data encrypted in transit using TLS 1.2+
- Private Google Access for internal traffic
- VPN or Cloud Interconnect for hybrid connectivity

### CMEK Implementation

**Create and Use CMEK for BigQuery**:
```python
from google.cloud import bigquery, kms_v1

def create_kms_key(project_id, location, key_ring_id, key_id):
    """Create Cloud KMS encryption key"""

    client = kms_v1.KeyManagementServiceClient()

    # Create key ring
    parent = f"projects/{project_id}/locations/{location}"
    key_ring = kms_v1.KeyRing()

    try:
        created_key_ring = client.create_key_ring(
            request={
                "parent": parent,
                "key_ring_id": key_ring_id,
                "key_ring": key_ring
            }
        )
        print(f"Created key ring: {created_key_ring.name}")
    except Exception as e:
        print(f"Key ring may already exist: {e}")
        created_key_ring_name = f"{parent}/keyRings/{key_ring_id}"

    # Create crypto key
    key_ring_name = f"{parent}/keyRings/{key_ring_id}"
    crypto_key = kms_v1.CryptoKey()
    crypto_key.purpose = kms_v1.CryptoKey.CryptoKeyPurpose.ENCRYPT_DECRYPT
    crypto_key.version_template = kms_v1.CryptoKeyVersionTemplate(
        algorithm=kms_v1.CryptoKeyVersion.CryptoKeyVersionAlgorithm.GOOGLE_SYMMETRIC_ENCRYPTION,
        protection_level=kms_v1.ProtectionLevel.SOFTWARE
    )

    # Enable automatic rotation every 90 days
    crypto_key.rotation_period = {"seconds": 90 * 24 * 60 * 60}
    crypto_key.next_rotation_time = {
        "seconds": int(time.time()) + 90 * 24 * 60 * 60
    }

    created_key = client.create_crypto_key(
        request={
            "parent": key_ring_name,
            "crypto_key_id": key_id,
            "crypto_key": crypto_key
        }
    )

    print(f"Created crypto key: {created_key.name}")
    return created_key

def create_cmek_dataset(project_id, dataset_id, kms_key_name):
    """Create BigQuery dataset with CMEK"""

    client = bigquery.Client(project=project_id)

    dataset = bigquery.Dataset(f"{project_id}.{dataset_id}")
    dataset.location = "US"
    dataset.default_encryption_configuration = bigquery.EncryptionConfiguration(
        kms_key_name=kms_key_name
    )

    created_dataset = client.create_dataset(dataset, exists_ok=True)
    print(f"Created dataset {created_dataset.dataset_id} with CMEK")
    return created_dataset

def create_cmek_table(project_id, dataset_id, table_id, kms_key_name):
    """Create BigQuery table with CMEK"""

    client = bigquery.Client(project=project_id)

    schema = [
        bigquery.SchemaField("customer_id", "STRING", mode="REQUIRED"),
        bigquery.SchemaField("ssn", "STRING", mode="NULLABLE"),
        bigquery.SchemaField("credit_card", "STRING", mode="NULLABLE"),
    ]

    table = bigquery.Table(f"{project_id}.{dataset_id}.{table_id}", schema=schema)
    table.encryption_configuration = bigquery.EncryptionConfiguration(
        kms_key_name=kms_key_name
    )

    created_table = client.create_table(table)
    print(f"Created table {created_table.table_id} with CMEK")
    return created_table

def create_cmek_bucket(project_id, bucket_name, kms_key_name):
    """Create Cloud Storage bucket with CMEK"""

    from google.cloud import storage

    client = storage.Client(project=project_id)
    bucket = client.create_bucket(bucket_name)

    bucket.default_kms_key_name = kms_key_name
    bucket.patch()

    print(f"Created bucket {bucket.name} with CMEK")
    return bucket
```

### IAM for Data Services

**BigQuery Access Control Layers**:

1. **Project-level IAM**: Controls project-wide access
2. **Dataset-level IAM**: Controls dataset access
3. **Table-level IAM**: Controls individual table access
4. **Column-level security**: Policy tags restrict column access
5. **Row-level security**: Filters limit row visibility
6. **Authorized views**: Provide controlled data access
7. **Authorized routines**: Allow specific UDF/stored procedure access

**Row-Level Security Implementation**:
```sql
-- Create row access policy based on user email
CREATE ROW ACCESS POLICY regional_access
ON `project.dataset.sales_data`
GRANT TO ("user:analyst-us@example.com")
FILTER USING (region = "US");

CREATE ROW ACCESS POLICY regional_access_eu
ON `project.dataset.sales_data`
GRANT TO ("user:analyst-eu@example.com")
FILTER USING (region = "EU");

-- Create row access policy based on data column
CREATE ROW ACCESS POLICY department_filter
ON `project.dataset.employee_data`
GRANT TO ("group:hr-team@example.com")
FILTER USING (department = "HR");

-- Create row access policy with dynamic filtering
CREATE ROW ACCESS POLICY user_owns_data
ON `project.dataset.customer_orders`
GRANT TO ("domain:example.com")
FILTER USING (customer_email = SESSION_USER());

-- List all row access policies
SELECT
  *
FROM `project.dataset.INFORMATION_SCHEMA.ROW_ACCESS_POLICIES`
WHERE table_name = 'sales_data';

-- Drop row access policy
DROP ALL ROW ACCESS POLICIES ON `project.dataset.sales_data`;
```

**Column-Level Security with Policy Tags**:
```sql
-- After applying policy tags via Python (shown earlier), query behavior:

-- User WITHOUT access to PII policy tag
SELECT customer_id, name, email, ssn
FROM `project.dataset.customers`;
-- ERROR: User does not have permission to access column ssn

-- User WITH access to PII policy tag
SELECT customer_id, name, email, ssn
FROM `project.dataset.customers`;
-- SUCCESS: Returns all columns including ssn

-- Grant access to policy tag
-- Use IAM to grant roles/datacatalog.categoryFineGrainedReader
-- on the policy tag resource
```

**Authorized Views**:
```python
def create_authorized_view(project_id, source_dataset, source_table,
                          view_dataset, view_name):
    """Create authorized view for secure data access"""

    from google.cloud import bigquery

    client = bigquery.Client(project=project_id)

    # Create view that filters sensitive data
    view_query = f"""
    SELECT
      customer_id,
      name,
      email,
      CASE
        WHEN SESSION_USER() IN ('admin@example.com', 'manager@example.com')
          THEN salary
        ELSE NULL
      END as salary,
      -- Mask SSN for non-privileged users
      CASE
        WHEN SESSION_USER() IN ('admin@example.com')
          THEN ssn
        ELSE CONCAT('XXX-XX-', RIGHT(ssn, 4))
      END as ssn_masked
    FROM `{project_id}.{source_dataset}.{source_table}`
    """

    view_ref = f"{project_id}.{view_dataset}.{view_name}"
    view = bigquery.Table(view_ref)
    view.view_query = view_query

    created_view = client.create_table(view, exists_ok=True)

    # Authorize view to access source dataset
    source_dataset_ref = client.dataset(source_dataset)
    source_dataset_obj = client.get_dataset(source_dataset_ref)

    # Add view to authorized views list
    view_reference = bigquery.DatasetReference(project_id, view_dataset).table(view_name)

    if view_reference not in source_dataset_obj.access_entries:
        entries = list(source_dataset_obj.access_entries)
        entries.append(
            bigquery.AccessEntry(
                role=None,
                entity_type="view",
                entity_id=view_reference.to_api_repr()
            )
        )
        source_dataset_obj.access_entries = entries
        client.update_dataset(source_dataset_obj, ["access_entries"])

    print(f"Created authorized view: {view_ref}")
    return created_view

def create_authorized_dataset(source_project, source_dataset,
                             target_project, target_dataset):
    """Authorize dataset to access another dataset"""

    from google.cloud import bigquery

    client = bigquery.Client(project=source_project)

    # Get source dataset
    source_dataset_ref = client.dataset(source_dataset, project=source_project)
    source_dataset_obj = client.get_dataset(source_dataset_ref)

    # Add authorized dataset
    target_dataset_ref = bigquery.DatasetReference(target_project, target_dataset)

    entries = list(source_dataset_obj.access_entries)
    entries.append(
        bigquery.AccessEntry(
            role=None,
            entity_type="dataset",
            entity_id={
                "dataset": target_dataset_ref.to_api_repr(),
                "target_types": ["VIEWS"]  # Allow only views from target dataset
            }
        )
    )

    source_dataset_obj.access_entries = entries
    updated_dataset = client.update_dataset(source_dataset_obj, ["access_entries"])

    print(f"Authorized {target_project}.{target_dataset} to access {source_project}.{source_dataset}")
    return updated_dataset
```

### Data Loss Prevention (DLP)

**DLP Capabilities**:
- **Inspection**: Detect and classify sensitive data
- **De-identification**: Mask, redact, tokenize, or encrypt sensitive data
- **Re-identification**: Reverse tokenization with proper authorization
- **Risk Analysis**: Assess re-identification risk
- **Custom InfoTypes**: Define organization-specific sensitive patterns

**Built-in InfoTypes** (100+ pre-defined):
- PII: `EMAIL_ADDRESS`, `PHONE_NUMBER`, `PERSON_NAME`, `DATE_OF_BIRTH`
- Financial: `CREDIT_CARD_NUMBER`, `IBAN_CODE`, `SWIFT_CODE`
- Government IDs: `US_SOCIAL_SECURITY_NUMBER`, `UK_NATIONAL_INSURANCE_NUMBER`
- Healthcare: `US_HEALTHCARE_NPI`, `ICD9_CODE`, `ICD10_CODE`
- Geographic: `STREET_ADDRESS`, `LOCATION`, `US_STATE`

### DLP Inspection

**Inspect Content with Multiple InfoTypes**:
```python
from google.cloud import dlp_v2
from google.cloud.dlp_v2 import types

def inspect_content_advanced(project_id, content_string):
    """Inspect content for sensitive data with detailed configuration"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    # Configure inspection
    inspect_config = types.InspectConfig(
        info_types=[
            {"name": "EMAIL_ADDRESS"},
            {"name": "PHONE_NUMBER"},
            {"name": "US_SOCIAL_SECURITY_NUMBER"},
            {"name": "CREDIT_CARD_NUMBER"},
            {"name": "PERSON_NAME"},
            {"name": "DATE_OF_BIRTH"},
            {"name": "US_PASSPORT"},
        ],
        min_likelihood=types.Likelihood.POSSIBLE,  # Minimum confidence level
        include_quote=True,  # Include matched text in results
        limits=types.InspectConfig.FindingLimits(
            max_findings_per_item=0,  # 0 = no limit
            max_findings_per_request=0
        ),
        # Custom rules for better accuracy
        rule_set=[
            types.InspectionRuleSet(
                info_types=[{"name": "EMAIL_ADDRESS"}],
                rules=[
                    types.InspectionRule(
                        exclusion_rule=types.ExclusionRule(
                            exclude_info_types=types.ExcludeInfoTypes(
                                info_types=[{"name": "PERSON_NAME"}]
                            ),
                            matching_type=types.MatchingType.MATCHING_TYPE_PARTIAL_MATCH
                        )
                    )
                ]
            )
        ]
    )

    item = types.ContentItem(value=content_string)

    response = dlp.inspect_content(
        request={
            "parent": parent,
            "inspect_config": inspect_config,
            "item": item
        }
    )

    # Process findings
    if response.result.findings:
        for finding in response.result.findings:
            print(f"Info type: {finding.info_type.name}")
            print(f"  Likelihood: {finding.likelihood.name}")
            print(f"  Quote: {finding.quote}")
            print(f"  Location: {finding.location.content_locations[0]}")
    else:
        print("No findings.")

    return response

def inspect_bigquery_table(project_id, table_project, dataset_id, table_id):
    """Inspect BigQuery table for sensitive data"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    # Configure BigQuery source
    storage_config = types.StorageConfig(
        big_query_options=types.BigQueryOptions(
            table_reference=types.BigQueryTable(
                project_id=table_project,
                dataset_id=dataset_id,
                table_id=table_id
            ),
            rows_limit=1000,  # Sample first 1000 rows
            sample_method=types.BigQueryOptions.SampleMethod.TOP,
            included_fields=[
                types.FieldId(name="email"),
                types.FieldId(name="phone"),
                types.FieldId(name="ssn"),
            ]
        )
    )

    inspect_config = types.InspectConfig(
        info_types=[
            {"name": "EMAIL_ADDRESS"},
            {"name": "PHONE_NUMBER"},
            {"name": "US_SOCIAL_SECURITY_NUMBER"},
        ],
        min_likelihood=types.Likelihood.LIKELY,
        include_quote=False  # Don't include actual values for security
    )

    # Create inspection job
    inspect_job = types.InspectJobConfig(
        storage_config=storage_config,
        inspect_config=inspect_config,
        actions=[
            types.Action(
                save_findings=types.Action.SaveFindings(
                    output_config=types.OutputStorageConfig(
                        table=types.BigQueryTable(
                            project_id=project_id,
                            dataset_id="dlp_results",
                            table_id="inspection_findings"
                        )
                    )
                )
            )
        ]
    )

    response = dlp.create_dlp_job(
        request={
            "parent": parent,
            "inspect_job": inspect_job
        }
    )

    print(f"Created inspection job: {response.name}")
    return response

def inspect_cloud_storage(project_id, bucket_name, file_pattern):
    """Inspect Cloud Storage files for sensitive data"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    # Configure Cloud Storage source
    storage_config = types.StorageConfig(
        cloud_storage_options=types.CloudStorageOptions(
            file_set=types.CloudStorageOptions.FileSet(
                url=f"gs://{bucket_name}/{file_pattern}"
            ),
            bytes_limit_per_file=1024 * 1024 * 10,  # 10 MB per file
            file_types=[
                types.FileType.TEXT_FILE,
                types.FileType.CSV,
                types.FileType.TSV,
            ],
            files_limit_percent=10,  # Sample 10% of files
            sample_method=types.CloudStorageOptions.SampleMethod.RANDOM_START
        )
    )

    inspect_config = types.InspectConfig(
        info_types=[{"name": "EMAIL_ADDRESS"}, {"name": "CREDIT_CARD_NUMBER"}],
        min_likelihood=types.Likelihood.POSSIBLE
    )

    inspect_job = types.InspectJobConfig(
        storage_config=storage_config,
        inspect_config=inspect_config
    )

    response = dlp.create_dlp_job(
        request={"parent": parent, "inspect_job": inspect_job}
    )

    print(f"Started Cloud Storage inspection: {response.name}")
    return response
```

### DLP De-identification

**De-identification Techniques**:
```python
def deidentify_with_masking(project_id, content_string):
    """Mask sensitive data with asterisks"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    inspect_config = types.InspectConfig(
        info_types=[{"name": "EMAIL_ADDRESS"}, {"name": "PHONE_NUMBER"}]
    )

    # Mask configuration
    deidentify_config = types.DeidentifyConfig(
        info_type_transformations=types.InfoTypeTransformations(
            transformations=[
                types.InfoTypeTransformations.InfoTypeTransformation(
                    info_types=[{"name": "EMAIL_ADDRESS"}],
                    primitive_transformation=types.PrimitiveTransformation(
                        character_mask_config=types.CharacterMaskConfig(
                            masking_character="*",
                            number_to_mask=0,  # Mask all characters
                        )
                    )
                ),
                types.InfoTypeTransformations.InfoTypeTransformation(
                    info_types=[{"name": "PHONE_NUMBER"}],
                    primitive_transformation=types.PrimitiveTransformation(
                        character_mask_config=types.CharacterMaskConfig(
                            masking_character="#",
                            number_to_mask=7,  # Mask first 7 digits
                            reverse_order=False
                        )
                    )
                )
            ]
        )
    )

    item = types.ContentItem(value=content_string)

    response = dlp.deidentify_content(
        request={
            "parent": parent,
            "deidentify_config": deidentify_config,
            "inspect_config": inspect_config,
            "item": item
        }
    )

    print(f"Original: {content_string}")
    print(f"Deidentified: {response.item.value}")
    return response

def deidentify_with_crypto_hash(project_id, content_string, key_name):
    """Hash sensitive data using cryptographic hash"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    inspect_config = types.InspectConfig(
        info_types=[{"name": "EMAIL_ADDRESS"}]
    )

    # Crypto hash configuration
    deidentify_config = types.DeidentifyConfig(
        info_type_transformations=types.InfoTypeTransformations(
            transformations=[
                types.InfoTypeTransformations.InfoTypeTransformation(
                    info_types=[{"name": "EMAIL_ADDRESS"}],
                    primitive_transformation=types.PrimitiveTransformation(
                        crypto_hash_config=types.CryptoHashConfig(
                            crypto_key=types.CryptoKey(
                                kms_wrapped=types.KmsWrappedCryptoKey(
                                    wrapped_key=b"YourBase64EncodedWrappedKey",
                                    crypto_key_name=key_name
                                )
                            )
                        )
                    )
                )
            ]
        )
    )

    item = types.ContentItem(value=content_string)

    response = dlp.deidentify_content(
        request={
            "parent": parent,
            "deidentify_config": deidentify_config,
            "inspect_config": inspect_config,
            "item": item
        }
    )

    return response

def deidentify_with_fpe(project_id, content_string, key_name, alphabet):
    """De-identify using Format-Preserving Encryption (FPE)"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    inspect_config = types.InspectConfig(
        info_types=[{"name": "US_SOCIAL_SECURITY_NUMBER"}]
    )

    # FPE configuration (maintains format)
    deidentify_config = types.DeidentifyConfig(
        info_type_transformations=types.InfoTypeTransformations(
            transformations=[
                types.InfoTypeTransformations.InfoTypeTransformation(
                    info_types=[{"name": "US_SOCIAL_SECURITY_NUMBER"}],
                    primitive_transformation=types.PrimitiveTransformation(
                        crypto_replace_ffx_fpe_config=types.CryptoReplaceFfxFpeConfig(
                            crypto_key=types.CryptoKey(
                                kms_wrapped=types.KmsWrappedCryptoKey(
                                    wrapped_key=b"YourBase64EncodedWrappedKey",
                                    crypto_key_name=key_name
                                )
                            ),
                            common_alphabet=alphabet,  # NUMERIC, ALPHA_NUMERIC, etc.
                            surrogate_info_type=types.InfoType(
                                name="SSN_TOKEN"
                            )
                        )
                    )
                )
            ]
        )
    )

    item = types.ContentItem(value=content_string)

    response = dlp.deidentify_content(
        request={
            "parent": parent,
            "deidentify_config": deidentify_config,
            "inspect_config": inspect_config,
            "item": item
        }
    )

    return response

def deidentify_bigquery_table(project_id, source_project, source_dataset, source_table,
                              dest_project, dest_dataset, dest_table):
    """De-identify BigQuery table and write to new table"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    # Source configuration
    storage_config = types.StorageConfig(
        big_query_options=types.BigQueryOptions(
            table_reference=types.BigQueryTable(
                project_id=source_project,
                dataset_id=source_dataset,
                table_id=source_table
            )
        )
    )

    # Transformation configuration
    deidentify_config = types.DeidentifyConfig(
        record_transformations=types.RecordTransformations(
            field_transformations=[
                # Mask email
                types.FieldTransformation(
                    fields=[types.FieldId(name="email")],
                    primitive_transformation=types.PrimitiveTransformation(
                        character_mask_config=types.CharacterMaskConfig(
                            masking_character="*"
                        )
                    )
                ),
                # Redact SSN
                types.FieldTransformation(
                    fields=[types.FieldId(name="ssn")],
                    primitive_transformation=types.PrimitiveTransformation(
                        replace_config=types.ReplaceValueConfig(
                            new_value=types.Value(string_value="[REDACTED]")
                        )
                    )
                ),
                # Date shift (for temporal anonymization)
                types.FieldTransformation(
                    fields=[types.FieldId(name="birth_date")],
                    primitive_transformation=types.PrimitiveTransformation(
                        date_shift_config=types.DateShiftConfig(
                            upper_bound_days=30,
                            lower_bound_days=30,
                        )
                    )
                )
            ]
        )
    )

    # Destination configuration
    output_config = types.OutputStorageConfig(
        table=types.BigQueryTable(
            project_id=dest_project,
            dataset_id=dest_dataset,
            table_id=dest_table
        )
    )

    # Create deidentification job
    deidentify_job = types.DeidentifyJobConfig(
        storage_config=storage_config,
        deidentify_config=deidentify_config,
        output_config=output_config
    )

    response = dlp.create_dlp_job(
        request={
            "parent": parent,
            "deidentify_job": deidentify_job
        }
    )

    print(f"Created de-identification job: {response.name}")
    return response
```

### DLP Templates

**Create Inspection Template**:
```python
def create_inspect_template(project_id, template_id):
    """Create reusable inspection template"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    inspect_config = types.InspectConfig(
        info_types=[
            {"name": "EMAIL_ADDRESS"},
            {"name": "PHONE_NUMBER"},
            {"name": "US_SOCIAL_SECURITY_NUMBER"},
            {"name": "CREDIT_CARD_NUMBER"},
        ],
        min_likelihood=types.Likelihood.LIKELY,
        limits=types.InspectConfig.FindingLimits(
            max_findings_per_request=100
        )
    )

    inspect_template = types.InspectTemplate(
        display_name="PII Inspection Template",
        description="Detects common PII types",
        inspect_config=inspect_config
    )

    response = dlp.create_inspect_template(
        request={
            "parent": parent,
            "inspect_template": inspect_template,
            "template_id": template_id
        }
    )

    print(f"Created template: {response.name}")
    return response

def create_deidentify_template(project_id, template_id, key_name):
    """Create reusable de-identification template"""

    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}"

    deidentify_config = types.DeidentifyConfig(
        info_type_transformations=types.InfoTypeTransformations(
            transformations=[
                # Tokenize with FPE for reversible de-identification
                types.InfoTypeTransformations.InfoTypeTransformation(
                    info_types=[
                        {"name": "EMAIL_ADDRESS"},
                        {"name": "PHONE_NUMBER"},
                        {"name": "US_SOCIAL_SECURITY_NUMBER"}
                    ],
                    primitive_transformation=types.PrimitiveTransformation(
                        crypto_replace_ffx_fpe_config=types.CryptoReplaceFfxFpeConfig(
                            crypto_key=types.CryptoKey(
                                kms_wrapped=types.KmsWrappedCryptoKey(
                                    wrapped_key=b"YourBase64EncodedWrappedKey",
                                    crypto_key_name=key_name
                                )
                            ),
                            common_alphabet=types.CharsToIgnore.CommonCharsToIgnore.NUMERIC,
                            surrogate_info_type=types.InfoType(name="TOKENIZED")
                        )
                    )
                )
            ]
        )
    )

    deidentify_template = types.DeidentifyTemplate(
        display_name="PII Tokenization Template",
        description="Tokenizes PII using FPE",
        deidentify_config=deidentify_config
    )

    response = dlp.create_deidentify_template(
        request={
            "parent": parent,
            "deidentify_template": deidentify_template,
            "template_id": template_id
        }
    )

    print(f"Created template: {response.name}")
    return response
```

### Audit Logging and Monitoring

**Log Types**:
- **Admin Activity**: Configuration changes (always enabled, no charge)
- **Data Access**: Read/write operations (opt-in for most services, charges apply)
- **System Events**: Google-initiated changes (always enabled, no charge)
- **Policy Denied**: Access denied events (always enabled, no charge)

**Configure Data Access Logs**:
```python
from google.cloud import logging_v2

def enable_data_access_logs(project_id):
    """Enable BigQuery data access audit logs"""

    client = logging_v2.ConfigServiceV2Client()
    parent = f"projects/{project_id}"

    # Get current IAM policy
    from google.iam.v1 import policy_pb2
    from google.cloud import resourcemanager_v3

    rm_client = resourcemanager_v3.ProjectsClient()
    project_name = f"projects/{project_id}"

    policy = rm_client.get_iam_policy(request={"resource": project_name})

    # Add audit config for BigQuery
    audit_config = policy_pb2.AuditConfig(
        service="bigquery.googleapis.com",
        audit_log_configs=[
            policy_pb2.AuditLogConfig(
                log_type=policy_pb2.AuditLogConfig.LogType.ADMIN_READ
            ),
            policy_pb2.AuditLogConfig(
                log_type=policy_pb2.AuditLogConfig.LogType.DATA_READ
            ),
            policy_pb2.AuditLogConfig(
                log_type=policy_pb2.AuditLogConfig.LogType.DATA_WRITE
            )
        ]
    )

    policy.audit_configs.append(audit_config)

    # Update policy
    updated_policy = rm_client.set_iam_policy(
        request={
            "resource": project_name,
            "policy": policy
        }
    )

    print("Enabled BigQuery data access logs")
    return updated_policy

def export_logs_to_bigquery(project_id, dataset_id):
    """Export audit logs to BigQuery for analysis"""

    client = logging_v2.ConfigServiceV2Client()
    parent = f"projects/{project_id}"

    # Create log sink
    sink = logging_v2.types.LogSink(
        name="bigquery-audit-sink",
        destination=f"bigquery.googleapis.com/projects/{project_id}/datasets/{dataset_id}",
        filter='protoPayload.serviceName="bigquery.googleapis.com"',
        include_children=True
    )

    created_sink = client.create_sink(
        request={
            "parent": parent,
            "sink": sink
        }
    )

    print(f"Created sink: {created_sink.name}")
    return created_sink
```

**Query Audit Logs**:
```sql
-- Analyze BigQuery access patterns
SELECT
  protopayload_auditlog.authenticationInfo.principalEmail as user,
  protopayload_auditlog.methodName as action,
  protopayload_auditlog.resourceName as resource,
  TIMESTAMP(timestamp) as access_time,
  protopayload_auditlog.servicedata_v1_bigquery.jobCompletedEvent.job.jobStatistics.totalBilledBytes / POW(10, 12) as tb_billed
FROM `project.dataset._AllLogs`
WHERE
  protopayload_auditlog.serviceName = "bigquery.googleapis.com"
  AND DATE(timestamp) = CURRENT_DATE()
ORDER BY access_time DESC
LIMIT 100;

-- Detect unusual access patterns
WITH access_stats AS (
  SELECT
    protopayload_auditlog.authenticationInfo.principalEmail as user,
    DATE(timestamp) as date,
    COUNT(*) as query_count,
    SUM(protopayload_auditlog.servicedata_v1_bigquery.jobCompletedEvent.job.jobStatistics.totalBilledBytes) / POW(10, 12) as total_tb_scanned
  FROM `project.dataset._AllLogs`
  WHERE
    protopayload_auditlog.serviceName = "bigquery.googleapis.com"
    AND protopayload_auditlog.methodName = "jobservice.jobcompleted"
    AND DATE(timestamp) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
  GROUP BY user, date
),
user_averages AS (
  SELECT
    user,
    AVG(query_count) as avg_query_count,
    STDDEV(query_count) as stddev_query_count,
    AVG(total_tb_scanned) as avg_tb_scanned
  FROM access_stats
  GROUP BY user
)
SELECT
  a.user,
  a.date,
  a.query_count,
  u.avg_query_count,
  (a.query_count - u.avg_query_count) / NULLIF(u.stddev_query_count, 0) as z_score
FROM access_stats a
JOIN user_averages u ON a.user = u.user
WHERE ABS((a.query_count - u.avg_query_count) / NULLIF(u.stddev_query_count, 0)) > 3
ORDER BY ABS((a.query_count - u.avg_query_count) / NULLIF(u.stddev_query_count, 0)) DESC;
```

## VPC Service Controls

### Overview

**Purpose**: Prevent data exfiltration by creating security perimeters around GCP resources

**Key Features**:
- **Service Perimeters**: Logical security boundaries
- **Access Levels**: Context-aware access policies
- **VPC Accessible Services**: Restrict which APIs can be called
- **Ingress/Egress Rules**: Control data flow across perimeter

**Supported Services**: BigQuery, Cloud Storage, Cloud Dataflow, Cloud Dataproc, Pub/Sub, and more

### Creating Service Perimeters

```python
from google.cloud import accesscontextmanager_v1

def create_access_policy(organization_id):
    """Create access context manager policy"""

    client = accesscontextmanager_v1.AccessContextManagerClient()

    policy = accesscontextmanager_v1.AccessPolicy(
        parent=f"organizations/{organization_id}",
        title="Data Security Policy"
    )

    operation = client.create_access_policy(access_policy=policy)
    created_policy = operation.result()

    print(f"Created policy: {created_policy.name}")
    return created_policy

def create_service_perimeter(policy_name, project_ids, restricted_services):
    """Create VPC Service Control perimeter for data services"""

    client = accesscontextmanager_v1.AccessContextManagerClient()

    # Create perimeter
    perimeter = accesscontextmanager_v1.ServicePerimeter(
        title="Data Platform Perimeter",
        description="Protect BigQuery and Cloud Storage data",
        perimeter_type=accesscontextmanager_v1.ServicePerimeter.PerimeterType.PERIMETER_TYPE_REGULAR,
        status=accesscontextmanager_v1.ServicePerimeterConfig(
            resources=[f"projects/{pid}" for pid in project_ids],
            restricted_services=restricted_services,
            vpc_accessible_services=accesscontextmanager_v1.VpcAccessibleServices(
                enable_restriction=True,
                allowed_services=restricted_services
            )
        )
    )

    operation = client.create_service_perimeter(
        parent=policy_name,
        service_perimeter=perimeter
    )

    created_perimeter = operation.result()
    print(f"Created perimeter: {created_perimeter.name}")
    return created_perimeter

# Example usage
restricted_services = [
    "bigquery.googleapis.com",
    "storage.googleapis.com",
    "dataflow.googleapis.com",
    "dataproc.googleapis.com"
]
```

**Configure Ingress/Egress Rules**:
```python
def add_ingress_egress_rules(perimeter_name):
    """Add ingress/egress rules to service perimeter"""

    client = accesscontextmanager_v1.AccessContextManagerClient()

    # Get existing perimeter
    perimeter = client.get_service_perimeter(name=perimeter_name)

    # Add ingress rule (allow BigQuery queries from specific source)
    ingress_rule = accesscontextmanager_v1.IngressPolicy(
        ingress_from=accesscontextmanager_v1.IngressFrom(
            sources=[
                accesscontextmanager_v1.IngressSource(
                    resource="projects/12345"  # Source project
                )
            ],
            identity_type=accesscontextmanager_v1.IngressFrom.IdentityType.ANY_IDENTITY
        ),
        ingress_to=accesscontextmanager_v1.IngressTo(
            resources=["*"],  # All resources in perimeter
            operations=[
                accesscontextmanager_v1.ApiOperation(
                    service_name="bigquery.googleapis.com",
                    method_selectors=[
                        accesscontextmanager_v1.MethodSelector(
                            method="BigQueryReadSession"
                        )
                    ]
                )
            ]
        )
    )

    # Add egress rule (allow data copy to specific external bucket)
    egress_rule = accesscontextmanager_v1.EgressPolicy(
        egress_from=accesscontextmanager_v1.EgressFrom(
            identity_type=accesscontextmanager_v1.EgressFrom.IdentityType.ANY_IDENTITY
        ),
        egress_to=accesscontextmanager_v1.EgressTo(
            resources=[
                "projects/67890"  # Destination project
            ],
            operations=[
                accesscontextmanager_v1.ApiOperation(
                    service_name="storage.googleapis.com",
                    method_selectors=[
                        accesscontextmanager_v1.MethodSelector(
                            method="google.storage.objects.create"
                        )
                    ]
                )
            ]
        )
    )

    # Update perimeter
    perimeter.status.ingress_policies.append(ingress_rule)
    perimeter.status.egress_policies.append(egress_rule)

    updated_perimeter = client.update_service_perimeter(
        service_perimeter=perimeter
    )

    print("Updated perimeter with ingress/egress rules")
    return updated_perimeter
```

## Dataplex Data Governance

### Dataplex Architecture

**Components**:
- **Lakes**: Top-level organizational construct
- **Zones**: Subdivisions for raw/curated data
- **Assets**: BigQuery datasets or Cloud Storage buckets
- **Entities**: Tables, filesets discovered within assets

```python
from google.cloud import dataplex_v1

def create_data_lake(project_id, location, lake_id):
    """Create Dataplex lake for data governance"""

    client = dataplex_v1.DataplexServiceClient()
    parent = f"projects/{project_id}/locations/{location}"

    lake = dataplex_v1.Lake(
        display_name="Enterprise Data Lake",
        description="Central data lake for all enterprise data",
        labels={"environment": "production"}
    )

    operation = client.create_lake(
        parent=parent,
        lake_id=lake_id,
        lake=lake
    )

    created_lake = operation.result()
    print(f"Created lake: {created_lake.name}")
    return created_lake

def create_zones(project_id, location, lake_id):
    """Create raw and curated zones"""

    client = dataplex_v1.DataplexServiceClient()
    lake_name = f"projects/{project_id}/locations/{location}/lakes/{lake_id}"

    # Raw zone
    raw_zone = dataplex_v1.Zone(
        type_=dataplex_v1.Zone.Type.RAW,
        display_name="Raw Data Zone",
        description="Landing zone for raw data",
        discovery_spec=dataplex_v1.Zone.DiscoverySpec(
            enabled=True,
            schedule="0 * * * *"  # Hourly discovery
        ),
        resource_spec=dataplex_v1.Zone.ResourceSpec(
            location_type=dataplex_v1.Zone.ResourceSpec.LocationType.MULTI_REGION
        )
    )

    raw_operation = client.create_zone(
        parent=lake_name,
        zone_id="raw-zone",
        zone=raw_zone
    )

    # Curated zone
    curated_zone = dataplex_v1.Zone(
        type_=dataplex_v1.Zone.Type.CURATED,
        display_name="Curated Data Zone",
        description="Zone for processed, high-quality data",
        discovery_spec=dataplex_v1.Zone.DiscoverySpec(
            enabled=True,
            schedule="0 */6 * * *"  # Every 6 hours
        ),
        resource_spec=dataplex_v1.Zone.ResourceSpec(
            location_type=dataplex_v1.Zone.ResourceSpec.LocationType.MULTI_REGION
        )
    )

    curated_operation = client.create_zone(
        parent=lake_name,
        zone_id="curated-zone",
        zone=curated_zone
    )

    raw_result = raw_operation.result()
    curated_result = curated_operation.result()

    print(f"Created raw zone: {raw_result.name}")
    print(f"Created curated zone: {curated_result.name}")

    return raw_result, curated_result

def attach_assets(project_id, location, lake_id, zone_id, dataset_id):
    """Attach BigQuery dataset as Dataplex asset"""

    client = dataplex_v1.DataplexServiceClient()
    zone_name = f"projects/{project_id}/locations/{location}/lakes/{lake_id}/zones/{zone_id}"

    asset = dataplex_v1.Asset(
        display_name="Customer Data Asset",
        description="BigQuery datasets containing customer data",
        resource_spec=dataplex_v1.Asset.ResourceSpec(
            type_=dataplex_v1.Asset.ResourceSpec.Type.BIGQUERY_DATASET,
            name=f"projects/{project_id}/datasets/{dataset_id}"
        ),
        discovery_spec=dataplex_v1.Asset.DiscoverySpec(
            enabled=True,
            schedule="0 */4 * * *"  # Every 4 hours
        )
    )

    operation = client.create_asset(
        parent=zone_name,
        asset_id="customer-data",
        asset=asset
    )

    created_asset = operation.result()
    print(f"Created asset: {created_asset.name}")
    return created_asset
```

### Data Lifecycle Management

```python
def implement_retention_policies(project_id):
    """Implement comprehensive data retention"""

    from google.cloud import bigquery, storage

    # BigQuery table expiration
    bq_client = bigquery.Client(project=project_id)

    # Set table expiration
    table_ref = bq_client.dataset("raw_data").table("events")
    table = bq_client.get_table(table_ref)
    table.expires = datetime.datetime.now() + datetime.timedelta(days=90)
    bq_client.update_table(table, ["expires"])

    # Cloud Storage lifecycle policy
    storage_client = storage.Client(project=project_id)
    bucket = storage_client.bucket("raw-data-bucket")

    # Delete objects older than 365 days
    # Move to Nearline after 30 days
    # Move to Coldline after 90 days
    rule_delete = storage.lifecycle.LifecycleRuleDelete(age=365)
    rule_nearline = storage.lifecycle.LifecycleRuleSetStorageClass(
        "NEARLINE", age=30
    )
    rule_coldline = storage.lifecycle.LifecycleRuleSetStorageClass(
        "COLDLINE", age=90
    )

    bucket.lifecycle_rules = [rule_delete, rule_nearline, rule_coldline]
    bucket.patch()

    print("Configured retention policies")
```

## Compliance Frameworks

### GDPR Compliance

**Implementation Checklist**:
```python
def gdpr_compliance_setup(project_id, eu_dataset_id):
    """Set up GDPR-compliant data infrastructure"""

    from google.cloud import bigquery, dlp_v2

    client = bigquery.Client(project=project_id)

    # 1. Use EU regions
    dataset = bigquery.Dataset(f"{project_id}.{eu_dataset_id}")
    dataset.location = "EU"  # Data residency requirement
    dataset = client.create_dataset(dataset, exists_ok=True)

    # 2. Enable audit logging for compliance
    # (configured via enable_data_access_logs function)

    # 3. Implement data deletion workflow
    delete_user_data_query = """
    CREATE OR REPLACE PROCEDURE `{project}.{dataset}.delete_user_data`(user_email STRING)
    BEGIN
      -- Delete from all tables containing user data
      DELETE FROM `{project}.{dataset}.customers` WHERE email = user_email;
      DELETE FROM `{project}.{dataset}.orders` WHERE customer_email = user_email;
      DELETE FROM `{project}.{dataset}.interactions` WHERE user_id IN (
        SELECT user_id FROM `{project}.{dataset}.customers` WHERE email = user_email
      );

      -- Log deletion for audit
      INSERT INTO `{project}.{dataset}.data_deletion_log` (email, deletion_timestamp)
      VALUES (user_email, CURRENT_TIMESTAMP());
    END;
    """.format(project=project_id, dataset=eu_dataset_id)

    client.query(delete_user_data_query).result()

    # 4. Data export capability (portability)
    export_query = """
    CREATE OR REPLACE PROCEDURE `{project}.{dataset}.export_user_data`(user_email STRING, export_bucket STRING)
    BEGIN
      EXPORT DATA OPTIONS(
        uri=CONCAT(export_bucket, '/user_data_', user_email, '_*.json'),
        format='JSON',
        overwrite=true
      ) AS
      SELECT *
      FROM `{project}.{dataset}.customers`
      WHERE email = user_email;
    END;
    """.format(project=project_id, dataset=eu_dataset_id)

    client.query(export_query).result()

    # 5. Scan for PII
    dlp = dlp_v2.DlpServiceClient()
    # (use DLP inspection from earlier examples)

    print("GDPR compliance measures implemented")
```

### HIPAA Compliance

**Implementation**:
```python
def hipaa_compliance_setup(project_id, dataset_id, kms_key_name):
    """Set up HIPAA-compliant infrastructure"""

    from google.cloud import bigquery

    client = bigquery.Client(project=project_id)

    # 1. Create encrypted dataset with CMEK
    dataset = bigquery.Dataset(f"{project_id}.{dataset_id}")
    dataset.location = "US"
    dataset.default_encryption_configuration = bigquery.EncryptionConfiguration(
        kms_key_name=kms_key_name
    )
    dataset = client.create_dataset(dataset, exists_ok=True)

    # 2. Enable all audit logs
    # (Admin Activity, Data Access, System Events)

    # 3. Row-level security for PHI
    rls_query = """
    CREATE OR REPLACE ROW ACCESS POLICY phi_access
    ON `{project}.{dataset}.patient_records`
    GRANT TO ('group:healthcare-providers@example.com')
    FILTER USING (
      authorized_user_email = SESSION_USER()
      OR 'hipaa-admin@example.com' = SESSION_USER()
    );
    """.format(project=project_id, dataset=dataset_id)

    client.query(rls_query).result()

    # 4. Column-level security with policy tags
    # (Apply PII/PHI policy tags to sensitive columns)

    # 5. VPC Service Controls
    # (Create service perimeter around healthcare data)

    print("HIPAA compliance measures implemented")
    print("Note: Ensure BAA is signed with Google Cloud")
```

## Data Security Scenarios

### Scenario 1: Healthcare Data Platform

**Requirement**: Build HIPAA-compliant analytics platform for patient records

**Solution Architecture**:
```
1. Data Ingestion:
   - Use Pub/Sub with schema validation
   - CMEK encryption at rest
   - Private IP for Dataflow workers

2. Storage:
   - BigQuery with CMEK encryption
   - EU/US region based on patient location
   - Column-level security (policy tags) for PHI
   - Row-level security for provider access

3. Access Control:
   - Authorized views for analysts
   - Row-level security by provider ID
   - Policy tags on SSN, diagnosis codes
   - Audit all data access

4. Compliance:
   - VPC Service Controls perimeter
   - Data access audit logs to BigQuery
   - Automated DLP scanning
   - 7-year retention for medical records
```

### Scenario 2: Financial Services Data Warehouse

**Requirement**: PCI-DSS compliant data warehouse for credit card transactions

**Solution**:
```
1. Data Protection:
   - DLP tokenization for credit card numbers (FPE)
   - CMEK for all datasets
   - Separate encryption keys per environment

2. Access Control:
   - Column-level security for card numbers
   - Row-level security by merchant ID
   - Authorized views for reporting
   - MFA required for production access

3. Monitoring:
   - Real-time alerts on unusual queries
   - Data access logging to SIEM
   - Query audit trail
   - Failed access attempt monitoring

4. Compliance:
   - Quarterly access reviews
   - Penetration testing
   - Encryption key rotation (90 days)
   - Network segmentation with VPC-SC
```

### Scenario 3: Multi-Region Data Residency

**Requirement**: Store EU and US customer data separately for GDPR compliance

**Solution**:
```python
def setup_multi_region_architecture(project_id):
    """Create region-specific data architecture"""

    from google.cloud import bigquery

    client = bigquery.Client(project=project_id)

    # EU Dataset
    eu_dataset = bigquery.Dataset(f"{project_id}.customers_eu")
    eu_dataset.location = "EU"
    eu_dataset.description = "EU customer data (GDPR compliant)"
    client.create_dataset(eu_dataset, exists_ok=True)

    # US Dataset
    us_dataset = bigquery.Dataset(f"{project_id}.customers_us")
    us_dataset.location = "US"
    us_dataset.description = "US customer data"
    client.create_dataset(us_dataset, exists_ok=True)

    # Routing view (respects data residency)
    routing_view = """
    CREATE OR REPLACE VIEW `{project}.customer_analytics.global_customers` AS
    SELECT *, 'EU' as region FROM `{project}.customers_eu.customers`
    UNION ALL
    SELECT *, 'US' as region FROM `{project}.customers_us.customers`;
    """.format(project=project_id)

    client.query(routing_view).result()

    # Row-level security by region
    rls_eu = """
    CREATE OR REPLACE ROW ACCESS POLICY eu_only
    ON `{project}.customer_analytics.global_customers`
    GRANT TO ('group:eu-analysts@example.com')
    FILTER USING (region = 'EU');
    """.format(project=project_id)

    rls_us = """
    CREATE OR REPLACE ROW ACCESS POLICY us_only
    ON `{project}.customer_analytics.global_customers`
    GRANT TO ('group:us-analysts@example.com')
    FILTER USING (region = 'US');
    """.format(project=project_id)

    client.query(rls_eu).result()
    client.query(rls_us).result()

    print("Multi-region architecture configured")
```

## Exam Tips and Patterns

### Data Quality Questions

**Pattern Recognition**:
- **Validation at ingestion**: Pub/Sub schemas, Dataflow ParDo
- **Continuous monitoring**: Dataplex Data Quality, custom metrics
- **Anomaly detection**: Statistical methods, ML-based detection
- **Data profiling**: Dataplex automated profiling

**Key Decision Factors**:
1. **Real-time vs Batch**: Streaming = Dataflow, Batch = Dataplex/SQL
2. **Managed vs Custom**: Dataplex for standard checks, Dataflow for complex logic
3. **Cost vs Control**: Dataplex (managed, higher cost) vs Custom (lower cost, more code)

### Security Questions

**Common Scenarios**:
1. **Column has PII**: Use policy tags for column-level security
2. **Row filtering needed**: Use row-level security policies
3. **Compliance requirement**: CMEK + audit logs + VPC-SC
4. **Data exfiltration risk**: VPC Service Controls
5. **PII detection**: Cloud DLP inspection
6. **Masking for analytics**: DLP de-identification with FPE

**Decision Matrix**:
```
| Requirement                    | Solution                                    |
|--------------------------------|---------------------------------------------|
| Hide column from some users    | Policy tags + IAM                           |
| Filter rows by user            | Row-level security                          |
| Encrypt sensitive data         | CMEK (compliance) or default (cost)         |
| Prevent data copying           | VPC Service Controls                        |
| Find PII in data lake          | DLP inspection jobs                         |
| Mask PII for development       | DLP de-identification                       |
| Track who accessed data        | Data access audit logs                      |
| Multi-region data residency    | Regional datasets + row-level security      |
```

### Compliance Questions

**Framework Mappings**:
- **GDPR**: EU regions + data deletion + audit logs + DLP
- **HIPAA**: CMEK + BAA + audit logs + VPC-SC + PHI tagging
- **PCI-DSS**: Tokenization + CMEK + network segmentation + quarterly reviews
- **SOC 2**: Audit logs + access controls + encryption + monitoring

**Red Flags (Wrong Answers)**:
- Using CSEK for production data (key management complexity)
- Not enabling data access audit logs for compliance
- Missing VPC Service Controls for sensitive data
- Using application-level encryption instead of CMEK for compliance
- Cross-region data replication without considering residency

### Best Practices for Exam

1. **Always Enable Audit Logs**: Data access logs are critical for compliance
2. **Defense in Depth**: Combine multiple security controls (CMEK + RLS + Policy Tags + VPC-SC)
3. **Least Privilege**: Start with minimal permissions, add as needed
4. **Regional Compliance**: Match data location to regulatory requirements
5. **Automated Scanning**: Use DLP and Dataplex for continuous monitoring
6. **Encryption by Default**: Use CMEK for compliance, default for cost optimization
7. **Monitor Everything**: Audit logs, data quality metrics, access patterns

### Tricky Exam Scenarios

**Scenario**: Company needs to share data with external partner while preventing data exfiltration
**Answer**: VPC Service Controls with egress rules + authorized views + audit logging

**Scenario**: Analyst needs to query production data but shouldn't see PII
**Answer**: Authorized view with DLP masking OR policy tags on PII columns

**Scenario**: Data quality issues discovered after processing, need to track source
**Answer**: Implement data lineage tracking + quality metrics at each stage + Dataplex

**Scenario**: Multi-cloud architecture, need to prevent unauthorized GCP data exports
**Answer**: VPC Service Controls + IAM conditions + audit logging + egress rules

## Key Takeaways

### Data Quality
- Use **Dataplex Data Quality** for managed, declarative quality rules
- Implement **validation at ingestion** (Pub/Sub schemas, Dataflow)
- Set up **continuous monitoring** with alerts on quality SLOs
- Track **data lineage** for root cause analysis
- Profile data regularly to understand data characteristics

### Data Security
- Apply **defense in depth**: Multiple layers of security controls
- Use **policy tags** for column-level security (easiest to manage)
- Implement **row-level security** for user-specific data filtering
- Choose **CMEK** for compliance requirements, default encryption for cost
- Enable **data access audit logs** for compliance and monitoring

### Data Governance
- Organize data with **Dataplex lakes and zones**
- Use **Data Catalog** for metadata management and discovery
- Implement **retention policies** aligned with business/legal requirements
- Apply **data classification** tags (Public, Internal, Confidential, Restricted)
- Maintain **data lineage** for impact analysis and debugging

### Compliance
- Match **data regions** to regulatory requirements (EU for GDPR)
- Implement **data deletion workflows** for right to erasure
- Use **DLP** for automated PII detection and protection
- Create **VPC Service Controls** perimeters for sensitive data
- Enable comprehensive **audit logging** for all data access
- Sign **BAAs** for HIPAA compliance (prerequisite)

## Additional Resources

- [Data Governance on GCP](https://cloud.google.com/architecture/data-governance)
- [DLP Documentation](https://cloud.google.com/dlp/docs)
- [BigQuery Security Best Practices](https://cloud.google.com/bigquery/docs/best-practices-security)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
- [Dataplex Documentation](https://cloud.google.com/dataplex/docs)
- [Compliance Resource Center](https://cloud.google.com/security/compliance)
- [Data Catalog](https://cloud.google.com/data-catalog/docs)
- [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit)
