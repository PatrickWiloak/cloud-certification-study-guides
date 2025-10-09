# Machine Learning Operations - GCP Professional Data Engineer

## Overview

ML integration in data pipelines, model deployment, and ML workflows for data engineers, covering BigQuery ML, Vertex AI, AutoML, and MLOps practices.

## Key Topics

1. **Pre-built ML Models** - Vision API, Natural Language API, Translation API
2. **BigQuery ML** - SQL-based machine learning
3. **Vertex AI** - Training, deployment, and serving
4. **ML Pipelines** - End-to-end ML workflows
5. **Model Monitoring** - Performance tracking, drift detection

## BigQuery ML

### Model Types
- **Linear Regression**: Numeric prediction
- **Logistic Regression**: Binary/multiclass classification
- **K-means Clustering**: Unsupervised clustering
- **Matrix Factorization**: Recommendation systems
- **Time Series**: ARIMA models for forecasting
- **Boosted Trees**: XGBoost for classification/regression
- **Deep Neural Networks**: TensorFlow-based DNN
- **AutoML Tables**: Automated model selection

### Creating Models
```sql
CREATE OR REPLACE MODEL `dataset.model_name`
OPTIONS(
  model_type='LOGISTIC_REG',
  input_label_cols=['label_column'],
  max_iterations=10
) AS
SELECT feature1, feature2, label_column
FROM `dataset.training_table`;
```

### Model Evaluation
```sql
SELECT *
FROM ML.EVALUATE(MODEL `dataset.model_name`,
  (SELECT feature1, feature2, label_column FROM `dataset.test_table`));
```

### Making Predictions
```sql
SELECT predicted_label, predicted_label_probs
FROM ML.PREDICT(MODEL `dataset.model_name`,
  (SELECT feature1, feature2 FROM `dataset.input_data`));
```

### Feature Engineering in BigQuery ML
- **Feature Crosses**: Combine features for interactions
- **Polynomial Features**: Create higher-order terms
- **Bucketizing**: Convert continuous to categorical
- **One-Hot Encoding**: Automatic for STRING columns
- **Normalization**: Automatic feature scaling
- **Feature Selection**: Identify important features with `ML.FEATURE_INFO`

### Best Practices
1. Split data into train/test sets
2. Use `ML.EVALUATE` for model assessment
3. Monitor prediction latency
4. Version models with dataset naming
5. Export models for external deployment
6. Use `ML.FEATURE_INFO` for feature importance
7. Implement cross-validation for small datasets
8. Handle class imbalance with weights

## Vertex AI Integration

### Using Pre-trained Models
**Vision API**: Image analysis, OCR, object detection
**Natural Language API**: Sentiment, entity extraction, classification
**Translation API**: Language translation
**Speech-to-Text**: Audio transcription
**Text-to-Speech**: Speech synthesis

**Example**: Sentiment analysis in data pipeline
```python
from google.cloud import language_v1

def analyze_sentiment(text):
    client = language_v1.LanguageServiceClient()
    document = language_v1.Document(content=text, type_=language_v1.Document.Type.PLAIN_TEXT)
    sentiment = client.analyze_sentiment(request={'document': document}).document_sentiment
    return sentiment.score, sentiment.magnitude
```

### AutoML
**AutoML Tables**: Structured data prediction
**AutoML Vision**: Image classification, object detection
**AutoML Natural Language**: Text classification, entity extraction
**AutoML Video**: Video intelligence

**Use Cases**:
- No ML expertise required
- Quick model development
- Good baseline models
- Custom business-specific models

### Custom Training with Vertex AI
**Training Jobs**: Submit custom training code
**Hyperparameter Tuning**: Automatic optimization
**Distributed Training**: Multi-node, multi-GPU
**Pre-built Containers**: TensorFlow, PyTorch, Scikit-learn

### Model Deployment
**Vertex AI Endpoints**: Online prediction serving
**Batch Prediction**: Large-scale batch inference
**Model Monitoring**: Drift detection, performance tracking
**A/B Testing**: Traffic splitting for model comparison

## ML Pipelines

### Pipeline Architecture
```
Data Ingestion → Feature Engineering → Model Training →
Model Evaluation → Model Deployment → Monitoring
```

### Vertex AI Pipelines
**Components**: Reusable pipeline steps
**Pipeline Definition**: Python-based DSL
**Orchestration**: Managed pipeline execution
**Lineage Tracking**: Metadata and artifact tracking

**Example Pipeline**:
```python
from kfp.v2 import dsl
from kfp.v2.dsl import component, pipeline

@component
def preprocess_data(input_path: str, output_path: str):
    # Data preprocessing logic
    pass

@component
def train_model(training_data: str, model_output: str):
    # Model training logic
    pass

@pipeline
def ml_pipeline(data_source: str):
    preprocess_task = preprocess_data(input_path=data_source)
    train_task = train_model(training_data=preprocess_task.outputs['output_path'])
```

### Integration with Data Pipelines
**Dataflow + Vertex AI**: Feature engineering in Dataflow, training in Vertex AI
**BigQuery + Vertex AI**: BigQuery for feature store, Vertex AI for training
**Cloud Composer**: Orchestrate end-to-end ML workflows
**Pub/Sub**: Trigger retraining on new data events

## MLOps Practices

### Model Versioning
- Track model versions and experiments
- Use Vertex AI Model Registry
- Maintain model lineage
- Tag models with metadata

### Model Monitoring
**Data Drift**: Input distribution changes over time
**Prediction Drift**: Output distribution changes
**Training-Serving Skew**: Differences between training and serving data
**Performance Degradation**: Accuracy decline over time

**Monitoring Solutions**:
- Vertex AI Model Monitoring for drift detection
- Cloud Monitoring for prediction latency
- Custom metrics for business KPIs
- Alerting on threshold breaches

### Continuous Training
**Triggers**:
- Scheduled retraining (e.g., weekly)
- Performance degradation alerts
- Data drift detection
- New data availability

**Automation**:
- Cloud Composer DAGs for orchestration
- Vertex AI Pipelines for ML workflows
- Cloud Functions for event-driven triggers
- Pub/Sub for notification

### A/B Testing and Rollout
**Canary Deployment**: Route small percentage to new model
**Blue-Green Deployment**: Quick switchover between models
**Traffic Splitting**: Gradual rollout of new model
**Rollback**: Quick revert to previous version if issues

## Common Scenarios

**Scenario**: Customer churn prediction pipeline
**Solution**: BigQuery for data preparation, BigQuery ML for model training, scheduled queries for daily predictions, Data Studio for visualization

**Scenario**: Real-time fraud detection
**Solution**: Pub/Sub → Dataflow (feature engineering) → Vertex AI endpoint (prediction) → Bigtable (logging)

**Scenario**: Image classification for user uploads
**Solution**: Cloud Storage trigger → Cloud Functions → Vision API (AutoML or pre-trained) → Firestore (metadata storage)

**Scenario**: Recommendation system
**Solution**: BigQuery (user behavior data) → BigQuery ML (matrix factorization) → Cloud Functions (serving API) → Memorystore (caching)

## Study Tips

1. **BigQuery ML syntax**: Practice creating, evaluating, and predicting with models
2. **Pre-built APIs**: Know capabilities and use cases for each API
3. **Vertex AI components**: Understand training, deployment, and monitoring
4. **Pipeline orchestration**: Integration with Dataflow and Composer
5. **MLOps practices**: Versioning, monitoring, and continuous training
6. **Cost optimization**: Understand pricing for training and serving
7. **Hands-on practice**: Build end-to-end ML pipelines

## Key Commands

```bash
# Vertex AI Training
gcloud ai custom-jobs create --region=us-central1 --display-name=my-job --python-package-uris=gs://bucket/trainer.tar.gz --python-module=trainer.task

# Deploy Model
gcloud ai endpoints create --region=us-central1 --display-name=my-endpoint
gcloud ai endpoints deploy-model ENDPOINT_ID --region=us-central1 --model=MODEL_ID

# Batch Prediction
gcloud ai batch-prediction-jobs create --region=us-central1 --model=MODEL_ID --input-uri=gs://bucket/input --output-uri=gs://bucket/output
```

## Additional Resources

- [BigQuery ML Documentation](https://cloud.google.com/bigquery-ml/docs)
- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [ML on GCP Guide](https://cloud.google.com/architecture/ml-on-gcp-best-practices)
- [MLOps Best Practices](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
