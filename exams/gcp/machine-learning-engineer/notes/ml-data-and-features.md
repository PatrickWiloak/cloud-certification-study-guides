# ML Data and Features - GCP Professional Machine Learning Engineer

## Overview

Data preparation, feature engineering, Feature Store, data pipelines for ML, and data management best practices.

## Data Preparation

### Data Collection
**Sources**:
- BigQuery datasets
- Cloud Storage (CSV, JSON, Parquet, TFRecord)
- Pub/Sub streams
- Cloud SQL/Spanner databases
- External APIs

### Data Quality
**Validation**:
```python
import tensorflow_data_validation as tfdv

# Generate statistics
stats = tfdv.generate_statistics_from_csv('data.csv')

# Visualize
tfdv.visualize_statistics(stats)

# Infer schema
schema = tfdv.infer_schema(stats)

# Validate new data
new_stats = tfdv.generate_statistics_from_csv('new_data.csv')
anomalies = tfdv.validate_statistics(new_stats, schema)

# Check anomalies
tfdv.display_anomalies(anomalies)
```

### Data Splitting
```python
# Train/validation/test split
def split_dataset(df, train_ratio=0.7, val_ratio=0.15):
    train_size = int(len(df) * train_ratio)
    val_size = int(len(df) * val_ratio)

    train = df[:train_size]
    val = df[train_size:train_size+val_size]
    test = df[train_size+val_size:]

    return train, val, test

# Stratified split for imbalanced data
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

## Feature Engineering

### Feature Store
**Creating Features**:
```python
from google.cloud import aiplatform

featurestore = aiplatform.Featurestore.create('customer_features')

entity_type = featurestore.create_entity_type(
    entity_type_id='customer',
    description='Customer entity'
)

# Batch ingestion from BigQuery
entity_type.batch_create_features({
    'age': {'value_type': 'INT64', 'description': 'Customer age'},
    'ltv': {'value_type': 'DOUBLE', 'description': 'Lifetime value'},
    'is_premium': {'value_type': 'BOOL', 'description': 'Premium status'},
    'last_purchase_date': {'value_type': 'STRING', 'description': 'Last purchase'}
})

entity_type.ingest_from_bq(
    feature_ids=['age', 'ltv', 'is_premium'],
    feature_time='timestamp',
    bq_source_uri='bq://project.dataset.table',
    entity_id_field='customer_id'
)
```

**Feature Serving**:
```python
# Online serving (low latency)
features = entity_type.read(
    entity_ids=['customer_123', 'customer_456']
)

# Batch serving
batch_features = entity_type.batch_serve_to_bq(
    bq_destination_output_uri='bq://project.dataset.output_table',
    read_instances_uri='bq://project.dataset.entity_ids'
)
```

### Feature Transformation
**TensorFlow Transform**:
```python
import tensorflow_transform as tft

def preprocessing_fn(inputs):
    """Transform features"""
    # Normalize numeric
    age_normalized = tft.scale_to_z_score(inputs['age'])

    # Bucketize
    age_buckets = tft.bucketize(inputs['age'], num_buckets=5)

    # Vocabulary for categorical
    category_integerized = tft.compute_and_apply_vocabulary(
        inputs['category'], top_k=100
    )

    # Feature crossing
    age_category_cross = tft.hash(
        tf.strings.join([
            tf.as_string(age_buckets),
            tf.as_string(category_integerized)
        ]),
        hash_bucket_size=1000
    )

    return {
        'age_normalized': age_normalized,
        'age_buckets': age_buckets,
        'category': category_integerized,
        'age_category_cross': age_category_cross
    }
```

### Feature Selection
```python
# Feature importance from model
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier()
rf.fit(X_train, y_train)

# Plot feature importance
importances = rf.feature_importances_
indices = np.argsort(importances)[::-1]

plt.figure(figsize=(12, 6))
plt.bar(range(X.shape[1]), importances[indices])
plt.xlabel('Feature')
plt.ylabel('Importance')
plt.show()

# Select top features
from sklearn.feature_selection import SelectKBest, f_classif

selector = SelectKBest(f_classif, k=10)
X_selected = selector.fit_transform(X_train, y_train)
```

## Data Pipelines

### Dataflow for ML Data Prep
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

def preprocess_features(element):
    """Transform raw data to features"""
    import json

    record = json.loads(element)

    # Feature engineering
    features = {
        'age': int(record['age']),
        'is_premium': record['subscription'] == 'premium',
        'days_since_signup': (datetime.now() -
            datetime.fromisoformat(record['signup_date'])).days,
        'total_purchases': float(record['purchase_count'])
    }

    return features

pipeline_options = PipelineOptions(
    runner='DataflowRunner',
    project='project-id',
    region='us-central1',
    temp_location='gs://bucket/temp'
)

with beam.Pipeline(options=pipeline_options) as p:
    (p
     | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(subscription=SUB)
     | 'Parse and Transform' >> beam.Map(preprocess_features)
     | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
         table='project:dataset.features',
         schema='age:INTEGER,is_premium:BOOLEAN,...'
     ))
```

### BigQuery for Feature Engineering
```sql
-- Create training dataset
CREATE OR REPLACE TABLE `dataset.training_features` AS
WITH customer_features AS (
  SELECT
    customer_id,
    DATE_DIFF(CURRENT_DATE(), signup_date, DAY) as days_since_signup,
    COUNT(DISTINCT order_id) as total_orders,
    SUM(order_amount) as total_spent,
    AVG(order_amount) as avg_order_value,
    MAX(order_date) as last_order_date
  FROM `dataset.orders`
  GROUP BY customer_id, signup_date
),
rfm_features AS (
  SELECT
    customer_id,
    DATE_DIFF(CURRENT_DATE(), MAX(order_date), DAY) as recency,
    COUNT(*) as frequency,
    SUM(order_amount) as monetary
  FROM `dataset.orders`
  GROUP BY customer_id
)
SELECT
  c.*,
  r.recency,
  r.frequency,
  r.monetary,
  CASE
    WHEN r.recency < 30 AND r.frequency > 5 THEN 1
    ELSE 0
  END as is_active_customer
FROM customer_features c
JOIN rfm_features r USING(customer_id);

-- Aggregate features with window functions
CREATE OR REPLACE TABLE `dataset.time_series_features` AS
SELECT
  customer_id,
  order_date,
  order_amount,
  AVG(order_amount) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) as avg_7day_order_amount,
  LAG(order_amount, 1) OVER (
    PARTITION BY customer_id ORDER BY order_date
  ) as prev_order_amount
FROM `dataset.orders`;
```

## Data Versioning

### DVC (Data Version Control)
```bash
# Initialize DVC
dvc init

# Track data
dvc add data/train.csv
git add data/train.csv.dvc .gitignore
git commit -m "Add training data"

# Push to remote storage
dvc remote add -d storage gs://bucket/dvc-storage
dvc push

# Pull specific version
git checkout v1.0
dvc pull
```

### Dataset Versioning in Vertex AI
```python
# Create versioned dataset
dataset_v1 = aiplatform.TabularDataset.create(
    display_name='customer-churn-v1',
    gcs_source='gs://bucket/data_v1.csv'
)

dataset_v2 = aiplatform.TabularDataset.create(
    display_name='customer-churn-v2',
    gcs_source='gs://bucket/data_v2.csv'
)

# Track lineage
dataset_v2.update(
    labels={'previous_version': dataset_v1.resource_name}
)
```

## Best Practices

### Data Quality
1. Validate data schemas
2. Check for missing values
3. Detect outliers
4. Monitor data drift
5. Document data sources
6. Version datasets
7. Implement data tests
8. Regular data audits

### Feature Engineering
1. Use Feature Store for consistency
2. Version features
3. Document feature definitions
4. Monitor feature distributions
5. Implement feature validation
6. Test feature importance
7. Automate feature pipelines
8. Cache expensive computations

### Data Pipelines
1. Make pipelines idempotent
2. Implement error handling
3. Monitor pipeline health
4. Version pipeline code
5. Test with sample data
6. Document dependencies
7. Optimize for cost
8. Enable reprocessing

## Study Tips

1. Practice feature engineering techniques
2. Work with Feature Store
3. Build data pipelines with Dataflow
4. Use BigQuery for feature aggregation
5. Implement data validation
6. Version datasets properly
7. Monitor data quality

## Additional Resources

- [Feature Store Documentation](https://cloud.google.com/vertex-ai/docs/featurestore)
- [TensorFlow Data Validation](https://www.tensorflow.org/tfx/data_validation)
- [Dataflow for ML](https://cloud.google.com/architecture/ml-inference-serving-patterns)
- [BigQuery for ML](https://cloud.google.com/bigquery/docs/bigqueryml-intro)
