# AWS Machine Learning Specialty - Comprehensive Study Notes

## All Exam Domains - Key Concepts

### Domain 1: Data Engineering (20%)

#### Amazon S3 for ML
```python
import boto3

s3 = boto3.client('s3')

# Upload training data with optimal storage class
s3.upload_file(
    'train.csv',
    'ml-data-bucket',
    'datasets/train.csv',
    ExtraArgs={'StorageClass': 'INTELLIGENT_TIERING'}
)

# S3 Select for efficient data retrieval
response = s3.select_object_content(
    Bucket='ml-data-bucket',
    Key='large-dataset.csv',
    Expression='SELECT * FROM S3Object WHERE age > 25',
    ExpressionType='SQL',
    InputSerialization={'CSV': {'FileHeaderInfo': 'USE'}},
    OutputSerialization={'JSON': {}}
)
```

#### AWS Glue for ETL
```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

# Initialize
glueContext = GlueContext(SparkContext.getOrCreate())
spark = glueContext.spark_session

# Read from Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database="ml_database",
    table_name="raw_data"
)

# Transform data
transformed = ApplyMapping.apply(
    frame=datasource,
    mappings=[
        ("customer_id", "string", "id", "string"),
        ("purchase_amount", "double", "amount", "double"),
        ("timestamp", "string", "date", "date")
    ]
)

# Write to S3 in Parquet format
glueContext.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={"path": "s3://output-bucket/processed/"},
    format="parquet",
    format_options={"compression": "snappy"}
)
```

#### SageMaker Feature Store
```python
import sagemaker
from sagemaker.feature_store.feature_group import FeatureGroup

# Create feature group
feature_group = FeatureGroup(
    name="customer-features",
    sagemaker_session=sagemaker_session
)

feature_group.load_feature_definitions(data_frame=features_df)

feature_group.create(
    s3_uri=f"s3://{bucket}/feature-store",
    record_identifier_name="customer_id",
    event_time_feature_name="event_time",
    role_arn=role,
    enable_online_store=True
)

# Ingest features
feature_group.ingest(data_frame=features_df, max_workers=3, wait=True)

# Query features
from sagemaker.feature_store.feature_store import FeatureStore

feature_store = FeatureStore(sagemaker_session=sagemaker_session)

query = feature_store.athena_query()
query_string = '''
SELECT *
FROM "customer-features"
WHERE age > 25
'''
query.run(query_string=query_string, output_location=f's3://{bucket}/query-results/')
results = query.as_dataframe()
```

### Domain 2: Exploratory Data Analysis (24%)

#### Data Preprocessing with SageMaker Data Wrangler
- Visual interface for data preparation
- 300+ built-in transformations
- Custom transformations with Python/PySpark
- Data quality insights
- Generate feature engineering code

#### Feature Engineering Techniques
```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.feature_selection import SelectKBest, f_classif

# Handle missing values
df['age'].fillna(df['age'].median(), inplace=True)
df['category'].fillna(df['category'].mode()[0], inplace=True)

# Outlier detection and handling
Q1 = df['amount'].quantile(0.25)
Q3 = df['amount'].quantile(0.75)
IQR = Q3 - Q1
df = df[(df['amount'] >= Q1 - 1.5*IQR) & (df['amount'] <= Q3 + 1.5*IQR)]

# Encoding categorical variables
label_encoder = LabelEncoder()
df['category_encoded'] = label_encoder.fit_transform(df['category'])

# One-hot encoding
df = pd.get_dummies(df, columns=['region'], prefix='region')

# Feature scaling
scaler = StandardScaler()
df[['age', 'income']] = scaler.fit_transform(df[['age', 'income']])

# Feature selection
selector = SelectKBest(f_classif, k=10)
X_selected = selector.fit_transform(X, y)
selected_features = X.columns[selector.get_support()]

# Time series features
df['hour'] = df['timestamp'].dt.hour
df['day_of_week'] = df['timestamp'].dt.dayofweek
df['month'] = df['timestamp'].dt.month
df['is_weekend'] = df['day_of_week'].isin([5, 6]).astype(int)

# Rolling features
df['rolling_mean_7d'] = df.groupby('customer_id')['amount'].transform(
    lambda x: x.rolling(window=7, min_periods=1).mean()
)
```

#### Statistical Analysis
```python
import scipy.stats as stats

# Hypothesis testing
t_stat, p_value = stats.ttest_ind(group1, group2)
print(f"T-statistic: {t_stat}, P-value: {p_value}")

# Correlation analysis
correlation_matrix = df.corr()

# Chi-square test for categorical variables
chi2, p_value, dof, expected = stats.chi2_contingency(pd.crosstab(df['cat1'], df['cat2']))

# Normality test
stat, p_value = stats.normaltest(df['variable'])
```

### Domain 3: Modeling (36%)

#### Algorithm Selection Guide

**Classification**:
- Binary: XGBoost, Linear Learner, Neural Networks
- Multi-class: XGBoost, Image Classification (images), BlazingText (text)
- Text: BlazingText, Comprehend Custom Classification

**Regression**:
- Linear Learner: Linear relationships
- XGBoost: Non-linear relationships, feature importance
- Neural Networks: Complex patterns

**Clustering**:
- K-Means: Well-separated clusters
- DBSCAN: Arbitrary shaped clusters (scikit-learn)

**Dimensionality Reduction**:
- PCA: Linear dimensionality reduction
- t-SNE: Visualization (scikit-learn)

**Anomaly Detection**:
- Random Cut Forest: General anomaly detection
- IP Insights: IP address fraud

**Time Series**:
- DeepAR: Probabilistic forecasts
- Amazon Forecast: Automated time series service

#### SageMaker Training
```python
import sagemaker
from sagemaker import get_execution_role
from sagemaker.estimator import Estimator

role = get_execution_role()

# Built-in algorithm (XGBoost)
from sagemaker.xgboost import XGBoost

xgb = XGBoost(
    entry_point='train.py',
    framework_version='1.5-1',
    hyperparameters={
        'max_depth': 5,
        'eta': 0.2,
        'objective': 'binary:logistic',
        'num_round': 100
    },
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path='s3://bucket/output'
)

# Train model
xgb.fit({'train': 's3://bucket/train', 'validation': 's3://bucket/val'})

# Custom algorithm with Docker
estimator = Estimator(
    image_uri='123456789012.dkr.ecr.us-east-1.amazonaws.com/my-algorithm:latest',
    role=role,
    instance_count=1,
    instance_type='ml.p3.2xlarge',
    hyperparameters={'epochs': 10, 'batch-size': 32}
)

estimator.fit('s3://bucket/training-data')

# Distributed training
from sagemaker.pytorch import PyTorch

pytorch_estimator = PyTorch(
    entry_point='train.py',
    role=role,
    framework_version='1.12',
    py_version='py38',
    instance_count=4,  # Distributed training
    instance_type='ml.p3.8xlarge',
    distribution={'smdistributed': {'dataparallel': {'enabled': True}}}
)
```

#### Hyperparameter Tuning
```python
from sagemaker.tuner import HyperparameterTuner, IntegerParameter, ContinuousParameter

hyperparameter_ranges = {
    'max_depth': IntegerParameter(3, 10),
    'eta': ContinuousParameter(0.01, 0.5),
    'min_child_weight': IntegerParameter(1, 10),
    'subsample': ContinuousParameter(0.5, 1.0),
    'gamma': ContinuousParameter(0, 5)
}

tuner = HyperparameterTuner(
    estimator=xgb,
    objective_metric_name='validation:auc',
    hyperparameter_ranges=hyperparameter_ranges,
    max_jobs=20,
    max_parallel_jobs=3,
    strategy='Bayesian',  # or 'Random'
    early_stopping_type='Auto'
)

tuner.fit({'train': train_data, 'validation': val_data})

# Get best training job
best_training_job = tuner.best_training_job()
```

#### Model Evaluation
```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    confusion_matrix, roc_auc_score, roc_curve, classification_report
)
import matplotlib.pyplot as plt

# Classification metrics
y_pred = model.predict(X_test)
y_pred_proba = model.predict_proba(X_test)[:, 1]

print(f"Accuracy: {accuracy_score(y_test, y_pred)}")
print(f"Precision: {precision_score(y_test, y_pred)}")
print(f"Recall: {recall_score(y_test, y_pred)}")
print(f"F1 Score: {f1_score(y_test, y_pred)}")
print(f"AUC-ROC: {roc_auc_score(y_test, y_pred_proba)}")

# Confusion matrix
cm = confusion_matrix(y_test, y_pred)
print(f"Confusion Matrix:\n{cm}")

# ROC curve
fpr, tpr, thresholds = roc_curve(y_test, y_pred_proba)
plt.plot(fpr, tpr)
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')

# Regression metrics
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)

print(f"MAE: {mae}, RMSE: {rmse}, R²: {r2}")
```

### Domain 4: ML Implementation and Operations (20%)

#### Model Deployment
```python
# Real-time endpoint
predictor = xgb.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    endpoint_name='xgboost-endpoint'
)

# Make predictions
result = predictor.predict(test_data)

# Batch transform
transformer = xgb.transformer(
    instance_count=1,
    instance_type='ml.m5.xlarge',
    output_path='s3://bucket/batch-predictions'
)

transformer.transform(
    data='s3://bucket/batch-input',
    content_type='text/csv',
    split_type='Line'
)

# Multi-model endpoint
from sagemaker.multidatamodel import MultiDataModel

multi_model = MultiDataModel(
    name='multi-model-endpoint',
    model_data_prefix='s3://bucket/models/',
    model=xgb.create_model(),
    sagemaker_session=sagemaker_session
)

multi_model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge'
)
```

#### A/B Testing
```python
from sagemaker import ProductionVariant

variant1 = ProductionVariant(
    model_name='model-v1',
    instance_type='ml.m5.xlarge',
    initial_instance_count=1,
    variant_name='Variant1',
    initial_weight=0.5
)

variant2 = ProductionVariant(
    model_name='model-v2',
    instance_type='ml.m5.xlarge',
    initial_instance_count=1,
    variant_name='Variant2',
    initial_weight=0.5
)

sagemaker_client = boto3.client('sagemaker')
sagemaker_client.create_endpoint_config(
    EndpointConfigName='ab-test-config',
    ProductionVariants=[variant1, variant2]
)

sagemaker_client.create_endpoint(
    EndpointName='ab-test-endpoint',
    EndpointConfigName='ab-test-config'
)
```

#### Model Monitoring
```python
from sagemaker.model_monitor import DataCaptureConfig, DefaultModelMonitor

# Enable data capture
data_capture_config = DataCaptureConfig(
    enable_capture=True,
    sampling_percentage=100,
    destination_s3_uri='s3://bucket/data-capture'
)

predictor = model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    data_capture_config=data_capture_config
)

# Create baseline
my_default_monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    volume_size_in_gb=20,
    max_runtime_in_seconds=3600
)

my_default_monitor.suggest_baseline(
    baseline_dataset='s3://bucket/validation.csv',
    dataset_format={'csv': {'header': True}},
    output_s3_uri='s3://bucket/baselining',
    wait=True
)

# Schedule monitoring
my_default_monitor.create_monitoring_schedule(
    monitor_schedule_name='model-monitoring-schedule',
    endpoint_input=predictor.endpoint_name,
    output_s3_uri='s3://bucket/monitoring-results',
    statistics=my_default_monitor.baseline_statistics(),
    constraints=my_default_monitor.suggested_constraints(),
    schedule_cron_expression='cron(0 * * * ? *)'  # Hourly
)
```

#### SageMaker Pipelines
```python
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import ProcessingStep, TrainingStep
from sagemaker.workflow.parameters import ParameterString

# Define parameters
input_data = ParameterString(name="InputData", default_value="s3://bucket/input")
instance_type = ParameterString(name="InstanceType", default_value="ml.m5.xlarge")

# Processing step
from sagemaker.sklearn.processing import SKLearnProcessor

sklearn_processor = SKLearnProcessor(
    framework_version='0.23-1',
    role=role,
    instance_type=instance_type,
    instance_count=1
)

processing_step = ProcessingStep(
    name="PreprocessData",
    processor=sklearn_processor,
    inputs=[ProcessingInput(source=input_data, destination="/opt/ml/processing/input")],
    outputs=[ProcessingOutput(output_name="train", source="/opt/ml/processing/train")],
    code="preprocessing.py"
)

# Training step
training_step = TrainingStep(
    name="TrainModel",
    estimator=xgb,
    inputs={"train": processing_step.properties.ProcessingOutputConfig.Outputs["train"].S3Output.S3Uri}
)

# Create pipeline
pipeline = Pipeline(
    name="MLPipeline",
    parameters=[input_data, instance_type],
    steps=[processing_step, training_step]
)

pipeline.upsert(role_arn=role)
execution = pipeline.start()
```

## AWS AI Services

### Amazon Rekognition
```python
rekognition = boto3.client('rekognition')

# Detect labels
response = rekognition.detect_labels(
    Image={'S3Object': {'Bucket': 'bucket', 'Name': 'image.jpg'}},
    MaxLabels=10,
    MinConfidence=90
)

# Detect faces
response = rekognition.detect_faces(
    Image={'Bytes': image_bytes},
    Attributes=['ALL']
)

# Custom Labels
response = rekognition.detect_custom_labels(
    ProjectVersionArn='arn:aws:rekognition:region:account:project/project-name/version/1',
    Image={'S3Object': {'Bucket': 'bucket', 'Name': 'test.jpg'}},
    MinConfidence=70
)
```

### Amazon Comprehend
```python
comprehend = boto3.client('comprehend')

# Sentiment analysis
response = comprehend.detect_sentiment(
    Text='I love this product!',
    LanguageCode='en'
)

# Entity recognition
response = comprehend.detect_entities(
    Text='Amazon Web Services is located in Seattle.',
    LanguageCode='en'
)

# Custom classification
response = comprehend.classify_document(
    Text=document_text,
    EndpointArn='arn:aws:comprehend:region:account:document-classifier-endpoint/endpoint-name'
)
```

### Amazon Forecast
```python
forecast = boto3.client('forecast')

# Create dataset
response = forecast.create_dataset(
    DatasetName='sales_forecast',
    Domain='RETAIL',
    DatasetType='TARGET_TIME_SERIES',
    DataFrequency='D',
    Schema={
        'Attributes': [
            {'AttributeName': 'timestamp', 'AttributeType': 'timestamp'},
            {'AttributeName': 'target_value', 'AttributeType': 'float'},
            {'AttributeName': 'item_id', 'AttributeType': 'string'}
        ]
    }
)

# Create predictor (AutoML)
response = forecast.create_auto_predictor(
    PredictorName='sales_predictor',
    ForecastHorizon=30,
    ForecastFrequency='D'
)

# Generate forecast
response = forecast.create_forecast(
    ForecastName='sales_forecast_output',
    PredictorArn=predictor_arn
)
```

## Common Exam Scenarios

### Scenario 1: Real-time Fraud Detection
**Solution**:
- Kinesis Data Streams for real-time transaction ingestion
- Lambda for preprocessing
- SageMaker real-time endpoint with XGBoost or Random Cut Forest
- DynamoDB for feature lookup
- Step Functions for workflow orchestration

### Scenario 2: Image Classification at Scale
**Solution**:
- S3 for image storage
- SageMaker Image Classification algorithm
- Transfer learning with ResNet
- Hyperparameter tuning for optimization
- Batch Transform for large-scale inference
- Model Monitor for drift detection

### Scenario 3: NLP Document Analysis
**Solution**:
- Textract for document text extraction
- Comprehend for entity recognition and sentiment
- Custom Comprehend classifier for domain-specific categories
- SageMaker endpoint for custom models
- QuickSight for visualization

## Study Tips

1. **Hands-on Practice**: Build 3-5 complete ML projects end-to-end
2. **Algorithm Comparison**: Know when to use each algorithm
3. **SageMaker Mastery**: Deep understanding of all SageMaker features
4. **AI Services**: Practice with all AWS AI services
5. **MLOps**: Understand production deployment and monitoring
6. **Cost Optimization**: Know Spot instances, instance types, and savings strategies

## Quick Reference

### SageMaker Built-in Algorithm Use Cases
- **XGBoost**: General-purpose classification/regression
- **Linear Learner**: Linear relationships, sparse data
- **DeepAR**: Time series forecasting
- **Image Classification**: Computer vision tasks
- **BlazingText**: Text classification, word embeddings
- **K-Means**: Clustering, data segmentation
- **PCA**: Dimensionality reduction
- **Random Cut Forest**: Anomaly detection

### Instance Type Selection
- **Training**: ml.p3 (GPU) for deep learning, ml.m5 for general ML
- **Inference**: ml.m5 for CPU, ml.g4dn for GPU, ml.inf1 for cost-effective inference
- **Processing**: ml.m5 or ml.c5 for data processing

### Cost Optimization
- Use Spot instances for training (70% savings)
- Right-size instances based on workload
- Multi-model endpoints for many models
- Batch Transform instead of real-time for batch predictions
- Automatic model tuning with early stopping
- Inference recommender for optimal configuration
