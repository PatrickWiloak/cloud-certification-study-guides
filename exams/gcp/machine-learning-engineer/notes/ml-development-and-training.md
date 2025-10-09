# ML Development and Training - GCP Professional Machine Learning Engineer

## Overview

ML model development, training pipelines, Vertex AI, AutoML, and model optimization for the Professional Machine Learning Engineer certification.

## Vertex AI Platform

### Workbench (Notebooks)
**Managed Notebooks**:
```bash
# Create managed notebook
gcloud notebooks instances create ml-notebook \
  --vm-image-project=deeplearning-platform-release \
  --vm-image-family=tf-latest-cpu \
  --machine-type=n1-standard-4 \
  --location=us-central1-a
```

**User-Managed Notebooks**: More control, manual updates

### Custom Training
**Training Job**:
```python
from google.cloud import aiplatform

aiplatform.init(project='PROJECT_ID', location='us-central1')

job = aiplatform.CustomTrainingJob(
    display_name='custom-training',
    script_path='trainer/task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-cpu.2-11:latest',
    requirements=['pandas==1.3.0', 'scikit-learn==1.0.0'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/tf2-cpu.2-11:latest'
)

model = job.run(
    dataset=dataset,
    model_display_name='my-model',
    replica_count=1,
    machine_type='n1-standard-4',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

**Distributed Training**:
```python
# Multi-worker training
job.run(
    replica_count=5,  # Chief + 4 workers
    machine_type='n1-highmem-8',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1
)
```

### Hyperparameter Tuning
```python
from google.cloud.aiplatform import hyperparameter_tuning as hpt

hp_job = aiplatform.HyperparameterTuningJob(
    display_name='hp-tuning',
    custom_job=job,
    metric_spec={
        'accuracy': 'maximize',
    },
    parameter_spec={
        'learning_rate': hpt.DoubleParameterSpec(min=0.001, max=0.1, scale='log'),
        'batch_size': hpt.IntegerParameterSpec(min=16, max=128, scale='linear'),
        'num_layers': hpt.DiscreteParameterSpec(values=[2, 4, 6], scale='linear')
    },
    max_trial_count=20,
    parallel_trial_count=5,
    search_algorithm='bayesian'
)

hp_job.run()
```

## AutoML

### AutoML Tables
**Structured Data ML**:
```python
from google.cloud import aiplatform

dataset = aiplatform.TabularDataset.create(
    display_name='customer-churn-dataset',
    gcs_source='gs://bucket/data.csv'
)

job = aiplatform.AutoMLTabularTrainingJob(
    display_name='churn-prediction',
    optimization_prediction_type='classification',
    optimization_objective='maximize-au-prc',
    column_transformations=[
        {'numeric': {'column_name': 'age'}},
        {'categorical': {'column_name': 'gender'}},
        {'timestamp': {'column_name': 'signup_date'}}
    ]
)

model = job.run(
    dataset=dataset,
    target_column='churned',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=1000,
    model_display_name='churn-model'
)
```

### AutoML Vision
```python
# Image classification
dataset = aiplatform.ImageDataset.create(
    display_name='product-images',
    gcs_source='gs://bucket/images.csv'
)

job = aiplatform.AutoMLImageTrainingJob(
    display_name='product-classifier',
    prediction_type='classification',
    multi_label=False
)

model = job.run(
    dataset=dataset,
    model_display_name='product-classifier-model',
    training_fraction_split=0.8,
    budget_milli_node_hours=8000
)
```

### AutoML Natural Language
```python
# Text classification
dataset = aiplatform.TextDataset.create(
    display_name='support-tickets',
    gcs_source='gs://bucket/tickets.csv'
)

job = aiplatform.AutoMLTextTrainingJob(
    display_name='ticket-classifier',
    prediction_type='classification'
)

model = job.run(
    dataset=dataset,
    training_fraction_split=0.8,
    model_display_name='ticket-classifier-model'
)
```

## BigQuery ML

### Model Creation
```sql
-- Logistic regression
CREATE OR REPLACE MODEL `dataset.churn_model`
OPTIONS(
  model_type='LOGISTIC_REG',
  input_label_cols=['churned'],
  max_iterations=10,
  l1_reg=0.1,
  l2_reg=0.1
) AS
SELECT
  age,
  gender,
  subscription_months,
  support_tickets,
  churned
FROM
  `dataset.customers`
WHERE
  split_field != 'TEST';

-- XGBoost
CREATE OR REPLACE MODEL `dataset.xgboost_model`
OPTIONS(
  model_type='BOOSTED_TREE_CLASSIFIER',
  input_label_cols=['label'],
  max_iterations=50,
  booster_type='GBTREE',
  tree_method='HIST',
  max_depth=6,
  subsample=0.8,
  colsample_bytree=0.8
) AS
SELECT * FROM `dataset.training_data`;

-- Deep Neural Network
CREATE OR REPLACE MODEL `dataset.dnn_model`
OPTIONS(
  model_type='DNN_CLASSIFIER',
  hidden_units=[128, 64, 32],
  activation_fn='RELU',
  dropout=0.2,
  batch_size=32,
  max_iterations=100,
  early_stop=TRUE
) AS
SELECT * FROM `dataset.training_data`;
```

### Model Evaluation
```sql
SELECT *
FROM ML.EVALUATE(MODEL `dataset.churn_model`,
  (SELECT * FROM `dataset.customers` WHERE split_field = 'TEST'));

-- Feature importance
SELECT *
FROM ML.FEATURE_IMPORTANCE(MODEL `dataset.churn_model`);

-- Confusion matrix
SELECT *
FROM ML.CONFUSION_MATRIX(MODEL `dataset.churn_model`,
  (SELECT * FROM `dataset.test_data`));
```

### Predictions
```sql
-- Batch prediction
SELECT
  customer_id,
  predicted_churned,
  predicted_churned_probs
FROM
  ML.PREDICT(MODEL `dataset.churn_model`,
    (SELECT * FROM `dataset.new_customers`));

-- Explain predictions
SELECT *
FROM ML.EXPLAIN_PREDICT(MODEL `dataset.churn_model`,
  (SELECT * FROM `dataset.test_data` LIMIT 10));
```

## Feature Engineering

### Feature Store
```python
from google.cloud import aiplatform

# Create feature store
featurestore = aiplatform.Featurestore.create(
    featurestore_id='customer_features',
    online_store_fixed_node_count=1
)

# Create entity type
entity_type = featurestore.create_entity_type(
    entity_type_id='customer',
    description='Customer features'
)

# Create features
entity_type.create_feature(
    feature_id='age',
    value_type='INT64'
)
entity_type.create_feature(
    feature_id='total_purchases',
    value_type='DOUBLE'
)

# Ingest features
entity_type.ingest_from_bq(
    feature_ids=['age', 'total_purchases'],
    feature_time='feature_timestamp',
    bq_source_uri='bq://project.dataset.table'
)

# Online serving
features = entity_type.read(entity_ids=['customer_123'])
```

### Feature Preprocessing
```python
import tensorflow as tf
from tensorflow import feature_column

# Numeric features
age = feature_column.numeric_column('age')
income = feature_column.numeric_column('income', normalizer_fn=lambda x: x / 100000)

# Categorical features
gender = feature_column.categorical_column_with_vocabulary_list(
    'gender', ['M', 'F', 'Other'])
gender_embedding = feature_column.embedding_column(gender, dimension=3)

# Bucketized features
age_buckets = feature_column.bucketized_column(
    age, boundaries=[18, 25, 35, 45, 55, 65])

# Crossed features
age_gender_cross = feature_column.crossed_column(
    [age_buckets, gender], hash_bucket_size=1000)

# Feature layer
feature_layer = tf.keras.layers.DenseFeatures([
    age, income, gender_embedding, age_buckets
])
```

## Model Optimization

### Quantization
```python
import tensorflow as tf

# Post-training quantization
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model/')
converter.optimizations = [tf.lite.Optimize.DEFAULT]
quantized_model = converter.convert()

# Quantization-aware training
model = tf.keras.models.Sequential([...])
model = tfmot.quantization.keras.quantize_model(model)
model.compile(...)
model.fit(...)
```

### Model Pruning
```python
import tensorflow_model_optimization as tfmot

pruning_params = {
    'pruning_schedule': tfmot.sparsity.keras.PolynomialDecay(
        initial_sparsity=0.0,
        final_sparsity=0.5,
        begin_step=1000,
        end_step=5000
    )
}

model = tfmot.sparsity.keras.prune_low_magnitude(model, **pruning_params)
model.compile(...)
model.fit(...)
```

## Best Practices

### Training
1. Use Vertex AI for managed training
2. Implement distributed training for large models
3. Hyperparameter tuning for optimization
4. Monitor training metrics
5. Use early stopping
6. Save checkpoints regularly
7. Version datasets and models
8. Document experiments

### AutoML
1. Evaluate business problem suitability
2. Prepare high-quality training data
3. Set appropriate budget
4. Use for baseline models
5. Export for custom deployment if needed
6. Monitor AutoML recommendations
7. Iterate on data quality

### Feature Engineering
1. Use Feature Store for consistency
2. Version features
3. Monitor feature drift
4. Implement feature validation
5. Document feature definitions
6. Reuse features across models
7. Test feature importance

## Study Tips

1. Practice with Vertex AI training jobs
2. Implement AutoML for different data types
3. Use BigQuery ML for SQL-based ML
4. Configure hyperparameter tuning
5. Work with Feature Store
6. Optimize models for production
7. Understand distributed training

## Additional Resources

- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [AutoML Documentation](https://cloud.google.com/automl/docs)
- [BigQuery ML](https://cloud.google.com/bigquery-ml/docs)
- [ML Best Practices](https://cloud.google.com/architecture/ml-on-gcp-best-practices)
