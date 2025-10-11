# ML Deployment and Operations - GCP Professional Machine Learning Engineer

## Overview

Model deployment, serving, monitoring, MLOps practices, and production ML systems for the Professional Machine Learning Engineer certification.

## Model Deployment

### Vertex AI Endpoints

**Online Prediction Deployment**:
```python
from google.cloud import aiplatform
import json

# Initialize Vertex AI
aiplatform.init(project='my-project', location='us-central1')

# Create endpoint
endpoint = aiplatform.Endpoint.create(
    display_name='churn-prediction-endpoint',
    description='Production churn prediction service',
    labels={'team': 'ml-ops', 'env': 'prod'}
)

# Upload model
model = aiplatform.Model.upload(
    display_name='churn-model-v1',
    artifact_uri='gs://my-bucket/models/churn/v1',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest',
    serving_container_health_route='/v1/models/churn:predict',
    serving_container_predict_route='/v1/models/churn:predict',
    serving_container_ports=[8500],
    explanation_metadata=explanation_metadata,
    explanation_parameters=explanation_parameters
)

# Deploy model to endpoint with autoscaling
deployed_model = model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn-v1-prod',
    machine_type='n1-standard-4',
    min_replica_count=2,
    max_replica_count=10,
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1,
    traffic_percentage=100,
    traffic_split={'0': 100},
    service_account='ml-serving@project.iam.gserviceaccount.com',
    enable_container_logging=True,
    enable_access_logging=True,
    explanation_metadata=explanation_metadata,
    explanation_parameters=explanation_parameters,
    sync=True
)

# Make online prediction
instances = [
    {'age': 35, 'gender': 'F', 'subscription_months': 12, 'usage_hours': 45},
    {'age': 42, 'gender': 'M', 'subscription_months': 6, 'usage_hours': 22}
]

prediction = endpoint.predict(instances=instances)
print(f"Predictions: {prediction.predictions}")
print(f"Model: {prediction.deployed_model_id}")

# Get prediction with explanations
explanation = endpoint.explain(instances=instances)
print(f"Feature attributions: {explanation.explanations}")
```

**Private Endpoints with VPC Peering**:
```python
# Create private endpoint
private_endpoint = aiplatform.PrivateEndpoint.create(
    display_name='private-churn-endpoint',
    network='projects/12345/global/networks/ml-vpc',
    description='Private endpoint for internal services'
)

# Deploy to private endpoint
model.deploy(
    endpoint=private_endpoint,
    deployed_model_display_name='churn-v1-private',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5
)

# Access from VPC
# Only accessible from resources within the VPC or peered networks
prediction = private_endpoint.predict(instances=instances)
```

**Traffic Splitting and Canary Deployments**:
```python
# Deploy model v1
model_v1 = aiplatform.Model.upload(
    display_name='churn-model-v1',
    artifact_uri='gs://my-bucket/models/churn/v1',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest'
)

deployed_v1 = model_v1.deploy(
    endpoint=endpoint,
    deployed_model_display_name='v1',
    machine_type='n1-standard-4',
    min_replica_count=2,
    max_replica_count=10,
    traffic_percentage=100
)

# Deploy model v2 with canary (10% traffic)
model_v2 = aiplatform.Model.upload(
    display_name='churn-model-v2',
    artifact_uri='gs://my-bucket/models/churn/v2',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest'
)

deployed_v2 = model_v2.deploy(
    endpoint=endpoint,
    deployed_model_display_name='v2',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5,
    traffic_percentage=10  # Canary deployment
)

# Monitor v2 performance
time.sleep(3600)  # Wait 1 hour

# Gradually increase v2 traffic
endpoint.update(
    traffic_split={
        deployed_v1.id: 75,
        deployed_v2.id: 25
    }
)

# Full rollout after validation
endpoint.update(
    traffic_split={
        deployed_v1.id: 0,
        deployed_v2.id: 100
    }
)

# Undeploy old version
endpoint.undeploy(deployed_model_id=deployed_v1.id)
```

**Shadow Deployment Pattern**:
```python
# Deploy shadow model (receives traffic but doesn't serve responses)
def shadow_deployment():
    # Deploy v2 with 0% traffic
    model_v2.deploy(
        endpoint=endpoint,
        deployed_model_display_name='v2-shadow',
        machine_type='n1-standard-4',
        min_replica_count=1,
        max_replica_count=3,
        traffic_percentage=0
    )

    # Use Cloud Function to mirror traffic
    # predictions.py
    from google.cloud import aiplatform
    import logging

    def mirror_prediction(request):
        instances = request.get_json()['instances']

        # Primary prediction (v1)
        primary = endpoint.predict(instances=instances)

        # Shadow prediction (v2) - logged but not returned
        try:
            shadow = endpoint.predict(
                instances=instances,
                deployed_model_id=deployed_v2.id
            )

            # Log comparison
            logging.info(f"Primary: {primary.predictions}")
            logging.info(f"Shadow: {shadow.predictions}")

            # Calculate divergence
            divergence = calculate_divergence(primary, shadow)
            if divergence < 0.05:  # 5% threshold
                logging.info("Shadow model ready for promotion")
        except Exception as e:
            logging.error(f"Shadow prediction failed: {e}")

        return primary.predictions
```

### Batch Prediction

**Batch Prediction Jobs**:
```python
# Batch prediction on large dataset
batch_prediction_job = model.batch_predict(
    job_display_name='churn-batch-prediction',
    gcs_source='gs://my-bucket/batch-input/*.jsonl',
    gcs_destination_prefix='gs://my-bucket/batch-output/',
    model_parameters={},
    machine_type='n1-standard-16',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1,
    starting_replica_count=10,
    max_replica_count=50,
    sync=False,
    service_account='ml-batch@project.iam.gserviceaccount.com',
    generate_explanation=True,
    labels={'job_type': 'batch_inference', 'model': 'churn'}
)

# Monitor job
batch_prediction_job.wait_for_resource_creation()
print(f"Job created: {batch_prediction_job.resource_name}")

# Check status
while batch_prediction_job.state != aiplatform.gapic.JobState.JOB_STATE_SUCCEEDED:
    time.sleep(60)
    batch_prediction_job.refresh()
    print(f"State: {batch_prediction_job.state}")

# Get output
output_info = batch_prediction_job.output_info
print(f"Output location: {output_info.gcs_output_directory}")
```

**Batch Prediction with BigQuery**:
```python
from google.cloud import aiplatform_v1

# Batch predict with BigQuery input/output
batch_job = model.batch_predict(
    job_display_name='churn-bq-batch',
    bigquery_source='bq://project.dataset.input_table',
    bigquery_destination_prefix='bq://project.dataset',
    machine_type='n1-highmem-8',
    starting_replica_count=5,
    max_replica_count=20,
    model_parameters={
        'threshold': 0.7
    },
    generate_explanation=True,
    explanation_metadata=explanation_metadata
)

# Query results
from google.cloud import bigquery

client = bigquery.Client()
query = f"""
SELECT
    customer_id,
    prediction.scores[0] as churn_probability,
    prediction.classes[0] as predicted_class
FROM `{batch_job.output_info.bigquery_output_table}`
WHERE prediction.scores[0] > 0.7
ORDER BY churn_probability DESC
LIMIT 100
"""
results = client.query(query).to_dataframe()
```

**Scheduled Batch Predictions**:
```python
from google.cloud import scheduler_v1
from google.cloud import functions_v1
import json

def create_scheduled_batch_prediction():
    # Cloud Function to trigger batch prediction
    function_code = '''
from google.cloud import aiplatform
import json

def trigger_batch_prediction(request):
    aiplatform.init(project='my-project', location='us-central1')

    model = aiplatform.Model('projects/123/locations/us-central1/models/456')

    batch_job = model.batch_predict(
        job_display_name='scheduled-batch-prediction',
        gcs_source='gs://my-bucket/daily-data/*.jsonl',
        gcs_destination_prefix='gs://my-bucket/predictions/',
        machine_type='n1-standard-8',
        starting_replica_count=5
    )

    return json.dumps({'job_id': batch_job.resource_name})
'''

    # Create Cloud Scheduler job
    scheduler_client = scheduler_v1.CloudSchedulerClient()
    parent = f"projects/my-project/locations/us-central1"

    job = {
        'name': f'{parent}/jobs/daily-batch-prediction',
        'description': 'Daily batch prediction job',
        'schedule': '0 2 * * *',  # 2 AM daily
        'time_zone': 'America/New_York',
        'http_target': {
            'uri': 'https://us-central1-my-project.cloudfunctions.net/trigger-batch-prediction',
            'http_method': scheduler_v1.HttpMethod.POST,
            'headers': {'Content-Type': 'application/json'}
        }
    }

    scheduler_client.create_job(request={'parent': parent, 'job': job})
```

## Model Serving Infrastructure

### Custom Serving Containers

**TensorFlow Serving Container**:
```python
# Dockerfile for TensorFlow Serving
"""
FROM tensorflow/serving:2.12.0

# Copy model
COPY models/saved_model /models/churn/1

# Set environment variables
ENV MODEL_NAME=churn
ENV MODEL_BASE_PATH=/models

# Expose ports
EXPOSE 8500 8501

# Start TensorFlow Serving
CMD ["tensorflow_model_server", \
     "--port=8500", \
     "--rest_api_port=8501", \
     "--model_name=${MODEL_NAME}", \
     "--model_base_path=${MODEL_BASE_PATH}/${MODEL_NAME}"]
"""

# Build and push to Artifact Registry
# build_and_push.sh
"""
#!/bin/bash
PROJECT_ID="my-project"
REGION="us-central1"
IMAGE_NAME="churn-serving"
IMAGE_TAG="v1"

# Build image
docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/ml-models/${IMAGE_NAME}:${IMAGE_TAG} .

# Push to Artifact Registry
docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/ml-models/${IMAGE_NAME}:${IMAGE_TAG}
"""

# Deploy custom container
model = aiplatform.Model.upload(
    display_name='churn-custom-serving',
    artifact_uri='gs://my-bucket/models/churn/v1',
    serving_container_image_uri=f'{REGION}-docker.pkg.dev/{PROJECT_ID}/ml-models/{IMAGE_NAME}:${IMAGE_TAG}',
    serving_container_predict_route='/v1/models/churn:predict',
    serving_container_health_route='/v1/models/churn'
)
```

**PyTorch TorchServe Container**:
```python
# Dockerfile for TorchServe
"""
FROM pytorch/torchserve:0.8.0-gpu

# Copy model archive
COPY churn_model.mar /home/model-server/model-store/

# Copy config
COPY config.properties /home/model-server/

# Expose ports
EXPOSE 8080 8081 8082

# Start TorchServe
CMD ["torchserve", \
     "--start", \
     "--model-store", "/home/model-server/model-store", \
     "--models", "churn=churn_model.mar", \
     "--ts-config", "/home/model-server/config.properties"]
"""

# Create model archive
# package_model.sh
"""
torch-model-archiver \
    --model-name churn_model \
    --version 1.0 \
    --model-file model.py \
    --serialized-file churn_model.pt \
    --handler custom_handler.py \
    --export-path model-store/
"""

# custom_handler.py
"""
from ts.torch_handler.base_handler import BaseHandler
import torch
import json

class ChurnModelHandler(BaseHandler):
    def preprocess(self, data):
        # Preprocess input data
        instances = data[0].get('body')
        inputs = json.loads(instances)
        return torch.tensor(inputs['features'])

    def inference(self, data):
        # Run inference
        with torch.no_grad():
            predictions = self.model(data)
        return predictions

    def postprocess(self, data):
        # Postprocess predictions
        return [{'prediction': data.tolist()}]
"""
```

**GPU-Optimized Serving**:
```python
# Deploy with GPU acceleration
model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn-gpu',
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1,
    min_replica_count=1,
    max_replica_count=5,
    # GPU-specific settings
    container_env_vars={
        'TF_GPU_THREAD_MODE': 'gpu_private',
        'TF_GPU_THREAD_COUNT': '2',
        'TF_FORCE_GPU_ALLOW_GROWTH': 'true'
    }
)

# Multi-GPU deployment
model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn-multi-gpu',
    machine_type='n1-standard-16',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=4,  # 4 GPUs
    min_replica_count=1,
    max_replica_count=3
)
```

### Autoscaling Configuration

**Custom Autoscaling Rules**:
```python
# Deploy with advanced autoscaling
deployed_model = model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn-autoscaling',
    machine_type='n1-standard-4',
    min_replica_count=2,
    max_replica_count=20,
    # Autoscaling based on metrics
    autoscaling_target_cpu_utilization=70,
    autoscaling_target_accelerator_duty_cycle=80
)

# Update autoscaling parameters
from google.cloud import aiplatform_v1

client = aiplatform_v1.EndpointServiceClient()
endpoint_name = endpoint.resource_name

# Get current deployed model
deployed_model_config = {
    'id': deployed_model.id,
    'dedicated_resources': {
        'min_replica_count': 2,
        'max_replica_count': 50,
        'machine_spec': {
            'machine_type': 'n1-standard-4'
        },
        'autoscaling_metric_specs': [
            {
                'metric_name': 'aiplatform.googleapis.com/prediction/online/cpu/utilization',
                'target': 75
            },
            {
                'metric_name': 'aiplatform.googleapis.com/prediction/online/replicas',
                'target': 10
            }
        ]
    }
}
```

## Model Monitoring

### Vertex AI Model Monitoring

**Training-Serving Skew Detection**:
```python
from google.cloud import aiplatform
from google.cloud.aiplatform import ModelDeploymentMonitoringJob

# Prepare training dataset statistics
training_dataset = {
    'gcs_source': {
        'uris': ['gs://my-bucket/training_data/statistics.json']
    },
    'data_format': 'csv',
    'target_field': 'churned'
}

# Configure skew detection
skew_config = {
    'skew_thresholds': {
        'age': {'value': 0.1},
        'subscription_months': {'value': 0.15},
        'usage_hours': {'value': 0.2},
        'account_age_days': {'value': 0.1}
    },
    'attribute_skew_thresholds': {
        'all': {'value': 0.1}
    }
}

# Configure drift detection
drift_config = {
    'drift_thresholds': {
        'age': {'value': 0.15},
        'subscription_months': {'value': 0.2},
        'usage_hours': {'value': 0.25}
    },
    'attribute_drift_thresholds': {
        'all': {'value': 0.15}
    }
}

# Create monitoring job
monitoring_job = aiplatform.ModelDeploymentMonitoringJob.create(
    display_name='churn-model-monitoring',
    endpoint=endpoint.resource_name,
    logging_sampling_strategy={
        'random_sample_config': {'sample_rate': 0.2}  # Sample 20% of predictions
    },
    schedule_config={
        'monitor_interval': {'seconds': 3600}  # Check every hour
    },
    model_monitoring_alert_config={
        'email_alert_config': {
            'user_emails': ['ml-team@example.com', 'ops@example.com']
        },
        'notification_channels': [
            'projects/123/notificationChannels/456'
        ],
        'enable_logging': True
    },
    model_deployment_monitoring_objective_configs=[
        {
            'deployed_model_id': deployed_model.id,
            'objective_config': {
                'training_dataset': training_dataset,
                'training_prediction_skew_detection_config': skew_config,
                'prediction_drift_detection_config': drift_config,
                'explanation_config': {
                    'enable_feature_attributes': True,
                    'explanation_baseline': {
                        'gcs_source': {
                            'uris': ['gs://my-bucket/baselines/baseline.json']
                        }
                    }
                }
            }
        }
    ],
    predict_instance_schema_uri='gs://my-bucket/schemas/input_schema.yaml',
    analysis_instance_schema_uri='gs://my-bucket/schemas/analysis_schema.yaml',
    log_ttl={'seconds': 2592000},  # 30 days
    labels={'env': 'production', 'team': 'ml-ops'}
)

print(f"Monitoring job created: {monitoring_job.resource_name}")
```

**Prediction Drift Analysis**:
```python
from google.cloud import aiplatform_v1
from google.cloud import bigquery
import pandas as pd

def analyze_drift_metrics(project_id, endpoint_id, days=7):
    """Analyze drift metrics from monitoring job"""
    bq_client = bigquery.Client(project=project_id)

    # Query drift metrics from BigQuery
    query = f"""
    SELECT
        TIMESTAMP_TRUNC(prediction_time, HOUR) as hour,
        feature_name,
        AVG(drift_score) as avg_drift,
        MAX(drift_score) as max_drift,
        COUNT(*) as sample_count
    FROM `{project_id}.model_monitoring.drift_metrics`
    WHERE
        endpoint_id = '{endpoint_id}'
        AND prediction_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL {days} DAY)
    GROUP BY hour, feature_name
    ORDER BY hour DESC, max_drift DESC
    """

    df = bq_client.query(query).to_dataframe()

    # Identify features with high drift
    high_drift_features = df[df['max_drift'] > 0.15]['feature_name'].unique()

    print(f"Features with high drift: {high_drift_features}")
    return df

# Set up alerts for drift
def create_drift_alert():
    from google.cloud import monitoring_v3

    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{PROJECT_ID}"

    alert_policy = monitoring_v3.AlertPolicy(
        display_name="High Model Drift Alert",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Drift threshold exceeded",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="aiplatform.googleapis.com/prediction/drift_score"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.2,
                    duration={'seconds': 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={'seconds': 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_MEAN
                        )
                    ]
                )
            )
        ],
        notification_channels=[NOTIFICATION_CHANNEL_ID],
        alert_strategy=monitoring_v3.AlertPolicy.AlertStrategy(
            auto_close={'seconds': 86400}
        )
    )

    policy = client.create_alert_policy(
        name=project_name,
        alert_policy=alert_policy
    )
    return policy
```

### Feature Attribution Monitoring

**Track Feature Importance Over Time**:
```python
def monitor_feature_attributions(endpoint, baseline_data):
    """Monitor how feature attributions change over time"""
    from google.cloud import aiplatform

    # Get explanations for sample predictions
    instances = baseline_data.sample(100).to_dict('records')

    explanation_response = endpoint.explain(instances=instances)

    # Extract feature attributions
    attributions = []
    for explanation in explanation_response.explanations:
        attr_dict = {}
        for attribution in explanation.attributions:
            for feature_name, feature_attr in attribution.feature_attributes.items():
                attr_dict[feature_name] = feature_attr

        attributions.append(attr_dict)

    # Calculate average attribution per feature
    import numpy as np
    feature_importance = {}
    for feature in attributions[0].keys():
        values = [attr[feature] for attr in attributions if feature in attr]
        feature_importance[feature] = {
            'mean': np.mean(values),
            'std': np.std(values),
            'min': np.min(values),
            'max': np.max(values)
        }

    return feature_importance

# Log feature attributions to BigQuery
def log_feature_attributions(feature_importance, timestamp):
    from google.cloud import bigquery

    client = bigquery.Client()
    table_id = f"{PROJECT_ID}.model_monitoring.feature_attributions"

    rows_to_insert = [
        {
            'timestamp': timestamp,
            'feature_name': feature,
            'mean_attribution': stats['mean'],
            'std_attribution': stats['std'],
            'min_attribution': stats['min'],
            'max_attribution': stats['max']
        }
        for feature, stats in feature_importance.items()
    ]

    errors = client.insert_rows_json(table_id, rows_to_insert)
    if errors:
        print(f"Errors inserting rows: {errors}")
```

### Custom Monitoring Metrics

**Log Custom Prediction Metrics**:
```python
from google.cloud import monitoring_v3
import time
from datetime import datetime

class ModelMetricsLogger:
    def __init__(self, project_id, model_name):
        self.project_id = project_id
        self.model_name = model_name
        self.client = monitoring_v3.MetricServiceClient()
        self.project_name = f"projects/{project_id}"

    def log_prediction_latency(self, latency_ms):
        """Log prediction latency"""
        series = monitoring_v3.TimeSeries()
        series.metric.type = f"custom.googleapis.com/ml/{self.model_name}/prediction_latency"
        series.resource.type = "global"
        series.resource.labels['project_id'] = self.project_id

        point = monitoring_v3.Point()
        point.value.double_value = latency_ms
        point.interval.end_time.seconds = int(time.time())
        series.points = [point]

        self.client.create_time_series(
            name=self.project_name,
            time_series=[series]
        )

    def log_prediction_confidence(self, confidence_score):
        """Log prediction confidence score"""
        series = monitoring_v3.TimeSeries()
        series.metric.type = f"custom.googleapis.com/ml/{self.model_name}/prediction_confidence"
        series.resource.type = "global"

        point = monitoring_v3.Point()
        point.value.double_value = confidence_score
        point.interval.end_time.seconds = int(time.time())
        series.points = [point]

        self.client.create_time_series(
            name=self.project_name,
            time_series=[series]
        )

    def log_prediction_distribution(self, prediction_counts):
        """Log prediction class distribution"""
        for class_name, count in prediction_counts.items():
            series = monitoring_v3.TimeSeries()
            series.metric.type = f"custom.googleapis.com/ml/{self.model_name}/prediction_count"
            series.metric.labels['class'] = class_name
            series.resource.type = "global"

            point = monitoring_v3.Point()
            point.value.int64_value = count
            point.interval.end_time.seconds = int(time.time())
            series.points = [point]

            self.client.create_time_series(
                name=self.project_name,
                time_series=[series]
            )

# Usage
logger = ModelMetricsLogger('my-project', 'churn-model')

# In prediction handler
start_time = time.time()
prediction = endpoint.predict(instances=instances)
latency = (time.time() - start_time) * 1000

logger.log_prediction_latency(latency)
logger.log_prediction_confidence(prediction.predictions[0]['confidence'])
```

**Performance Monitoring Dashboard**:
```python
def create_monitoring_dashboard():
    """Create Cloud Monitoring dashboard for model metrics"""
    from google.cloud import monitoring_dashboard_v1

    client = monitoring_dashboard_v1.DashboardsServiceClient()
    project_name = f"projects/{PROJECT_ID}"

    dashboard = monitoring_dashboard_v1.Dashboard(
        display_name="ML Model Performance Dashboard",
        grid_layout=monitoring_dashboard_v1.GridLayout(
            widgets=[
                # Prediction latency
                monitoring_dashboard_v1.Widget(
                    title="Prediction Latency",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="custom.googleapis.com/ml/churn-model/prediction_latency"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={'seconds': 60},
                                            per_series_aligner=monitoring_dashboard_v1.Aggregation.Aligner.ALIGN_MEAN
                                        )
                                    )
                                )
                            )
                        ]
                    )
                ),
                # Prediction volume
                monitoring_dashboard_v1.Widget(
                    title="Prediction Volume",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="aiplatform.googleapis.com/prediction/online/prediction_count"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={'seconds': 60},
                                            per_series_aligner=monitoring_dashboard_v1.Aggregation.Aligner.ALIGN_RATE
                                        )
                                    )
                                )
                            )
                        ]
                    )
                ),
                # Drift score
                monitoring_dashboard_v1.Widget(
                    title="Model Drift Score",
                    xy_chart=monitoring_dashboard_v1.XyChart(
                        data_sets=[
                            monitoring_dashboard_v1.XyChart.DataSet(
                                time_series_query=monitoring_dashboard_v1.TimeSeriesQuery(
                                    time_series_filter=monitoring_dashboard_v1.TimeSeriesFilter(
                                        filter='metric.type="aiplatform.googleapis.com/prediction/drift_score"',
                                        aggregation=monitoring_dashboard_v1.Aggregation(
                                            alignment_period={'seconds': 300},
                                            per_series_aligner=monitoring_dashboard_v1.Aggregation.Aligner.ALIGN_MEAN
                                        )
                                    )
                                )
                            )
                        ],
                        thresholds=[
                            monitoring_dashboard_v1.Threshold(
                                value=0.15,
                                color=monitoring_dashboard_v1.Threshold.Color.YELLOW
                            ),
                            monitoring_dashboard_v1.Threshold(
                                value=0.25,
                                color=monitoring_dashboard_v1.Threshold.Color.RED
                            )
                        ]
                    )
                )
            ]
        )
    )

    response = client.create_dashboard(
        parent=project_name,
        dashboard=dashboard
    )
    return response
```

## MLOps Pipelines

### Vertex AI Pipelines with Kubeflow

**Complete End-to-End Pipeline**:
```python
from kfp.v2 import dsl, compiler
from kfp.v2.dsl import (
    Input, Output, Dataset, Model, Metrics, Artifact,
    component, pipeline
)
from google.cloud import aiplatform
from typing import NamedTuple

# Component 1: Data Validation
@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'google-cloud-bigquery', 'tensorflow-data-validation']
)
def validate_data(
    input_data_uri: str,
    validation_output: Output[Dataset],
    schema_output: Output[Artifact]
) -> NamedTuple('Outputs', [('validation_passed', bool)]):
    import pandas as pd
    import tensorflow_data_validation as tfdv
    from collections import namedtuple
    import json

    # Load data
    df = pd.read_csv(input_data_uri)

    # Generate statistics
    stats = tfdv.generate_statistics_from_dataframe(df)

    # Infer schema
    schema = tfdv.infer_schema(stats)

    # Validate data
    anomalies = tfdv.validate_statistics(stats, schema)

    # Check for anomalies
    validation_passed = len(anomalies.anomaly_info) == 0

    # Save outputs
    with open(validation_output.path, 'w') as f:
        json.dump({
            'validation_passed': validation_passed,
            'anomalies': str(anomalies)
        }, f)

    with open(schema_output.path, 'w') as f:
        f.write(str(schema))

    outputs = namedtuple('Outputs', ['validation_passed'])
    return outputs(validation_passed)

# Component 2: Feature Engineering
@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'scikit-learn', 'google-cloud-storage']
)
def feature_engineering(
    input_data_uri: str,
    train_data: Output[Dataset],
    test_data: Output[Dataset],
    feature_metadata: Output[Artifact],
    test_size: float = 0.2
):
    import pandas as pd
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler
    import json
    import pickle

    # Load data
    df = pd.read_csv(input_data_uri)

    # Feature engineering
    df['account_age_months'] = df['account_age_days'] / 30
    df['usage_per_month'] = df['usage_hours'] / df['subscription_months']
    df['is_long_term'] = (df['subscription_months'] > 12).astype(int)

    # Encode categorical features
    df = pd.get_dummies(df, columns=['gender', 'subscription_type'])

    # Split features and target
    X = df.drop('churned', axis=1)
    y = df['churned']

    # Train-test split
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=test_size, random_state=42, stratify=y
    )

    # Scale features
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)

    # Save processed data
    train_df = pd.DataFrame(X_train_scaled, columns=X_train.columns)
    train_df['churned'] = y_train.values
    train_df.to_csv(train_data.path, index=False)

    test_df = pd.DataFrame(X_test_scaled, columns=X_test.columns)
    test_df['churned'] = y_test.values
    test_df.to_csv(test_data.path, index=False)

    # Save feature metadata
    metadata = {
        'feature_names': list(X_train.columns),
        'n_features': len(X_train.columns),
        'scaler_params': {
            'mean': scaler.mean_.tolist(),
            'std': scaler.scale_.tolist()
        }
    }
    with open(feature_metadata.path, 'w') as f:
        json.dump(metadata, f)

# Component 3: Model Training
@component(
    base_image='tensorflow/tensorflow:2.12.0',
    packages_to_install=['pandas', 'scikit-learn']
)
def train_model(
    train_data: Input[Dataset],
    model_output: Output[Model],
    metrics_output: Output[Metrics],
    learning_rate: float = 0.001,
    epochs: int = 50,
    batch_size: int = 32
):
    import pandas as pd
    import tensorflow as tf
    from sklearn.metrics import roc_auc_score
    import json

    # Load training data
    df = pd.read_csv(train_data.path)
    X_train = df.drop('churned', axis=1).values
    y_train = df['churned'].values

    # Build model
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(128, activation='relu', input_shape=(X_train.shape[1],)),
        tf.keras.layers.Dropout(0.3),
        tf.keras.layers.Dense(64, activation='relu'),
        tf.keras.layers.Dropout(0.2),
        tf.keras.layers.Dense(32, activation='relu'),
        tf.keras.layers.Dense(1, activation='sigmoid')
    ])

    # Compile model
    model.compile(
        optimizer=tf.keras.optimizers.Adam(learning_rate=learning_rate),
        loss='binary_crossentropy',
        metrics=['accuracy', tf.keras.metrics.AUC()]
    )

    # Train model
    history = model.fit(
        X_train, y_train,
        epochs=epochs,
        batch_size=batch_size,
        validation_split=0.2,
        verbose=1
    )

    # Save model
    model.save(model_output.path)

    # Log metrics
    train_predictions = model.predict(X_train)
    train_auc = roc_auc_score(y_train, train_predictions)

    metrics_output.log_metric('train_accuracy', float(history.history['accuracy'][-1]))
    metrics_output.log_metric('train_auc', float(train_auc))
    metrics_output.log_metric('final_loss', float(history.history['loss'][-1]))

# Component 4: Model Evaluation
@component(
    base_image='tensorflow/tensorflow:2.12.0',
    packages_to_install=['pandas', 'scikit-learn']
)
def evaluate_model(
    model: Input[Model],
    test_data: Input[Dataset],
    metrics_output: Output[Metrics],
    threshold: float = 0.5
) -> NamedTuple('Outputs', [('accuracy', float), ('auc', float), ('deploy', bool)]):
    import pandas as pd
    import tensorflow as tf
    from sklearn.metrics import accuracy_score, roc_auc_score, confusion_matrix, classification_report
    import json
    from collections import namedtuple

    # Load test data
    df = pd.read_csv(test_data.path)
    X_test = df.drop('churned', axis=1).values
    y_test = df['churned'].values

    # Load model
    model = tf.keras.models.load_model(model.path)

    # Evaluate
    test_loss, test_accuracy, test_auc = model.evaluate(X_test, y_test)

    # Predictions
    predictions = model.predict(X_test)
    predicted_classes = (predictions > threshold).astype(int)

    # Calculate metrics
    auc = roc_auc_score(y_test, predictions)
    accuracy = accuracy_score(y_test, predicted_classes)
    cm = confusion_matrix(y_test, predicted_classes)

    # Log metrics
    metrics_output.log_metric('test_accuracy', float(accuracy))
    metrics_output.log_metric('test_auc', float(auc))
    metrics_output.log_metric('test_loss', float(test_loss))

    # Log confusion matrix
    metrics_output.log_confusion_matrix(
        categories=['Not Churned', 'Churned'],
        matrix=cm.tolist()
    )

    # Decide if model should be deployed
    deploy = auc > 0.85 and accuracy > 0.80

    outputs = namedtuple('Outputs', ['accuracy', 'auc', 'deploy'])
    return outputs(float(accuracy), float(auc), deploy)

# Component 5: Model Registration
@component(
    base_image='python:3.9',
    packages_to_install=['google-cloud-aiplatform']
)
def register_model(
    model: Input[Model],
    project_id: str,
    location: str,
    model_display_name: str,
    accuracy: float,
    auc: float
) -> str:
    from google.cloud import aiplatform

    aiplatform.init(project=project_id, location=location)

    # Upload model to Vertex AI Model Registry
    uploaded_model = aiplatform.Model.upload(
        display_name=model_display_name,
        artifact_uri=model.uri,
        serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest',
        labels={
            'framework': 'tensorflow',
            'model_type': 'binary_classification'
        }
    )

    # Add metadata
    uploaded_model.update(
        description=f"Churn prediction model - Accuracy: {accuracy:.4f}, AUC: {auc:.4f}"
    )

    return uploaded_model.resource_name

# Component 6: Model Deployment
@component(
    base_image='python:3.9',
    packages_to_install=['google-cloud-aiplatform']
)
def deploy_model(
    model_resource_name: str,
    project_id: str,
    location: str,
    endpoint_display_name: str,
    machine_type: str = 'n1-standard-4',
    min_replicas: int = 1,
    max_replicas: int = 3
) -> str:
    from google.cloud import aiplatform

    aiplatform.init(project=project_id, location=location)

    # Get or create endpoint
    endpoints = aiplatform.Endpoint.list(
        filter=f'display_name="{endpoint_display_name}"'
    )

    if endpoints:
        endpoint = endpoints[0]
    else:
        endpoint = aiplatform.Endpoint.create(
            display_name=endpoint_display_name
        )

    # Get model
    model = aiplatform.Model(model_resource_name)

    # Deploy model
    deployed_model = model.deploy(
        endpoint=endpoint,
        deployed_model_display_name=f'{endpoint_display_name}-v1',
        machine_type=machine_type,
        min_replica_count=min_replicas,
        max_replica_count=max_replicas,
        traffic_percentage=100
    )

    return endpoint.resource_name

# Complete Pipeline Definition
@pipeline(
    name='churn-prediction-training-pipeline',
    description='Complete MLOps pipeline for churn prediction',
    pipeline_root='gs://my-bucket/pipeline-root'
)
def churn_training_pipeline(
    project_id: str,
    location: str,
    input_data_uri: str,
    model_display_name: str,
    endpoint_display_name: str,
    learning_rate: float = 0.001,
    epochs: int = 50,
    batch_size: int = 32,
    test_size: float = 0.2
):
    # Step 1: Validate data
    validation_task = validate_data(input_data_uri=input_data_uri)

    # Step 2: Feature engineering (only if validation passes)
    with dsl.Condition(validation_task.outputs['validation_passed'] == True):
        feature_task = feature_engineering(
            input_data_uri=input_data_uri,
            test_size=test_size
        )

        # Step 3: Train model
        train_task = train_model(
            train_data=feature_task.outputs['train_data'],
            learning_rate=learning_rate,
            epochs=epochs,
            batch_size=batch_size
        )

        # Step 4: Evaluate model
        eval_task = evaluate_model(
            model=train_task.outputs['model_output'],
            test_data=feature_task.outputs['test_data']
        )

        # Step 5: Register model (only if evaluation passes)
        with dsl.Condition(eval_task.outputs['deploy'] == True):
            register_task = register_model(
                model=train_task.outputs['model_output'],
                project_id=project_id,
                location=location,
                model_display_name=model_display_name,
                accuracy=eval_task.outputs['accuracy'],
                auc=eval_task.outputs['auc']
            )

            # Step 6: Deploy model
            deploy_task = deploy_model(
                model_resource_name=register_task.output,
                project_id=project_id,
                location=location,
                endpoint_display_name=endpoint_display_name
            )

# Compile pipeline
compiler.Compiler().compile(
    pipeline_func=churn_training_pipeline,
    package_path='churn_pipeline.json'
)

# Run pipeline
def run_pipeline():
    aiplatform.init(project='my-project', location='us-central1')

    job = aiplatform.PipelineJob(
        display_name='churn-training-pipeline-run',
        template_path='churn_pipeline.json',
        pipeline_root='gs://my-bucket/pipeline-root',
        parameter_values={
            'project_id': 'my-project',
            'location': 'us-central1',
            'input_data_uri': 'gs://my-bucket/data/churn_data.csv',
            'model_display_name': 'churn-model',
            'endpoint_display_name': 'churn-endpoint',
            'learning_rate': 0.001,
            'epochs': 50,
            'batch_size': 32,
            'test_size': 0.2
        },
        enable_caching=True
    )

    job.run(
        service_account='pipeline-sa@my-project.iam.gserviceaccount.com'
    )

    return job
```

### Continuous Training

**Automated Retraining Pipeline**:
```python
from google.cloud import aiplatform, scheduler_v1, bigquery
import json
from datetime import datetime, timedelta

def setup_continuous_training():
    """Set up automated retraining with data drift triggers"""

    # Cloud Function to check data drift and trigger retraining
    cloud_function_code = '''
from google.cloud import aiplatform, bigquery
import json
from datetime import datetime, timedelta

def check_drift_and_retrain(request):
    """Check for data drift and trigger retraining if needed"""
    project_id = 'my-project'
    location = 'us-central1'

    # Query recent prediction statistics
    bq_client = bigquery.Client()
    query = """
    SELECT
        AVG(drift_score) as avg_drift,
        MAX(drift_score) as max_drift
    FROM `{}.model_monitoring.drift_metrics`
    WHERE prediction_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
    """.format(project_id)

    results = list(bq_client.query(query))
    avg_drift = results[0].avg_drift
    max_drift = results[0].max_drift

    # Trigger conditions
    should_retrain = False
    reason = ""

    if max_drift > 0.25:
        should_retrain = True
        reason = f"High drift detected: {max_drift:.3f}"
    elif avg_drift > 0.15:
        should_retrain = True
        reason = f"Average drift threshold exceeded: {avg_drift:.3f}"

    # Check time since last training
    last_training_query = """
    SELECT MAX(create_time) as last_training
    FROM `{}.model_registry.training_runs`
    """.format(project_id)

    last_training_results = list(bq_client.query(last_training_query))
    if last_training_results[0].last_training:
        days_since_training = (datetime.now() - last_training_results[0].last_training).days
        if days_since_training > 30:
            should_retrain = True
            reason = f"Scheduled retraining: {days_since_training} days since last training"

    if should_retrain:
        # Trigger retraining pipeline
        aiplatform.init(project=project_id, location=location)

        pipeline_job = aiplatform.PipelineJob(
            display_name=f'automated-retraining-{datetime.now().strftime("%Y%m%d-%H%M%S")}',
            template_path='gs://my-bucket/pipelines/churn_pipeline.json',
            pipeline_root='gs://my-bucket/pipeline-root',
            parameter_values={
                'project_id': project_id,
                'location': location,
                'input_data_uri': 'gs://my-bucket/data/latest_data.csv',
                'model_display_name': 'churn-model',
                'endpoint_display_name': 'churn-endpoint'
            }
        )

        pipeline_job.submit()

        return json.dumps({
            'retrained': True,
            'reason': reason,
            'job_id': pipeline_job.resource_name
        })

    return json.dumps({'retrained': False, 'reason': 'No retraining needed'})
'''

    # Create Cloud Scheduler job to check drift daily
    scheduler_client = scheduler_v1.CloudSchedulerClient()
    parent = f"projects/{PROJECT_ID}/locations/{LOCATION}"

    drift_check_job = {
        'name': f'{parent}/jobs/drift-check-retraining',
        'description': 'Daily drift check and conditional retraining',
        'schedule': '0 2 * * *',  # 2 AM daily
        'time_zone': 'America/New_York',
        'http_target': {
            'uri': f'https://{LOCATION}-{PROJECT_ID}.cloudfunctions.net/check-drift-retrain',
            'http_method': scheduler_v1.HttpMethod.POST,
            'headers': {'Content-Type': 'application/json'}
        }
    }

    scheduler_client.create_job(request={'parent': parent, 'job': drift_check_job})

    # Also create a weekly scheduled retraining
    weekly_retrain_job = {
        'name': f'{parent}/jobs/weekly-retraining',
        'description': 'Weekly scheduled model retraining',
        'schedule': '0 3 * * 0',  # 3 AM every Sunday
        'time_zone': 'America/New_York',
        'http_target': {
            'uri': f'https://{LOCATION}-aiplatform.googleapis.com/v1/projects/{PROJECT_ID}/locations/{LOCATION}/pipelineJobs',
            'http_method': scheduler_v1.HttpMethod.POST,
            'headers': {
                'Content-Type': 'application/json',
                'Authorization': f'Bearer {get_access_token()}'
            },
            'body': json.dumps({
                'displayName': f'weekly-retraining-{datetime.now().strftime("%Y%m%d")}',
                'runtimeConfig': {
                    'gcsOutputDirectory': 'gs://my-bucket/pipeline-output',
                    'parameterValues': {
                        'project_id': PROJECT_ID,
                        'location': LOCATION,
                        'input_data_uri': 'gs://my-bucket/data/weekly_data.csv'
                    }
                },
                'pipelineSpec': {
                    'pipelineInfo': {
                        'name': 'churn-training-pipeline'
                    }
                }
            }).encode()
        }
    }

    scheduler_client.create_job(request={'parent': parent, 'job': weekly_retrain_job})

def get_access_token():
    """Get OAuth access token for API calls"""
    import google.auth
    import google.auth.transport.requests

    creds, project = google.auth.default()
    auth_req = google.auth.transport.requests.Request()
    creds.refresh(auth_req)
    return creds.token
```

**Data-Driven Retraining Triggers**:
```python
from google.cloud import bigquery, pubsub_v1, aiplatform
import json

def setup_event_driven_retraining():
    """Set up retraining triggered by data quality events"""

    # Create Pub/Sub topic for retraining events
    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(PROJECT_ID, 'model-retraining-trigger')

    try:
        topic = publisher.create_topic(request={'name': topic_path})
    except Exception:
        topic = publisher.get_topic(request={'topic': topic_path})

    # Cloud Function triggered by Pub/Sub
    pubsub_function_code = '''
from google.cloud import aiplatform
import base64
import json

def retrain_on_event(event, context):
    """Trigger retraining when Pub/Sub message received"""
    pubsub_message = base64.b64decode(event['data']).decode('utf-8')
    trigger_data = json.loads(pubsub_message)

    project_id = trigger_data['project_id']
    reason = trigger_data['reason']
    data_uri = trigger_data.get('data_uri', 'gs://my-bucket/data/latest_data.csv')

    aiplatform.init(project=project_id, location='us-central1')

    # Run retraining pipeline
    pipeline_job = aiplatform.PipelineJob(
        display_name=f'event-driven-retraining-{reason}',
        template_path='gs://my-bucket/pipelines/churn_pipeline.json',
        pipeline_root='gs://my-bucket/pipeline-root',
        parameter_values={
            'project_id': project_id,
            'location': 'us-central1',
            'input_data_uri': data_uri,
            'model_display_name': 'churn-model',
            'endpoint_display_name': 'churn-endpoint',
            'learning_rate': 0.001,
            'epochs': 50
        },
        labels={
            'trigger': 'event-driven',
            'reason': reason.replace(' ', '-')
        }
    )

    pipeline_job.submit()

    return f"Retraining triggered: {pipeline_job.resource_name}"
'''

    # Monitoring alert that publishes to Pub/Sub
    from google.cloud import monitoring_v3

    alert_client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{PROJECT_ID}"

    # Alert policy for high drift
    alert_policy = monitoring_v3.AlertPolicy(
        display_name="Trigger Retraining on High Drift",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="High drift detected",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="aiplatform.googleapis.com/prediction/drift_score"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.25,
                    duration={'seconds': 3600}  # Sustained for 1 hour
                )
            )
        ],
        notification_channels=[topic_path],
        documentation=monitoring_v3.AlertPolicy.Documentation(
            content="High data drift detected. Triggering automated retraining.",
            mime_type="text/markdown"
        )
    )

    policy = alert_client.create_alert_policy(
        name=project_name,
        alert_policy=alert_policy
    )

    return policy
```

## Model Registry and Versioning

### Vertex AI Model Registry

**Model Versioning and Metadata**:
```python
from google.cloud import aiplatform
from datetime import datetime
import json

def register_model_with_metadata(
    model_artifact_uri,
    model_name,
    version,
    metrics,
    hyperparameters,
    training_dataset_uri
):
    """Register model with comprehensive metadata"""

    aiplatform.init(project='my-project', location='us-central1')

    # Create version-specific display name
    display_name = f'{model_name}-v{version}'

    # Upload model
    model = aiplatform.Model.upload(
        display_name=display_name,
        artifact_uri=model_artifact_uri,
        serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest',
        serving_container_predict_route='/v1/models/churn:predict',
        serving_container_health_route='/v1/models/churn',
        description=f"""
        Model Version: {version}
        Training Date: {datetime.now().isoformat()}
        Accuracy: {metrics['accuracy']:.4f}
        AUC: {metrics['auc']:.4f}
        F1 Score: {metrics['f1']:.4f}
        """,
        labels={
            'version': str(version),
            'framework': 'tensorflow',
            'model_type': 'binary_classification',
            'environment': 'production'
        }
    )

    # Store additional metadata in model's metadata field
    metadata = {
        'training': {
            'dataset_uri': training_dataset_uri,
            'training_date': datetime.now().isoformat(),
            'framework_version': '2.12.0',
            'hyperparameters': hyperparameters
        },
        'metrics': metrics,
        'validation': {
            'test_size': 0.2,
            'validation_method': 'stratified_split'
        },
        'features': {
            'input_features': ['age', 'subscription_months', 'usage_hours'],
            'feature_engineering': 'standardization',
            'categorical_encoding': 'one-hot'
        }
    }

    # Store metadata in GCS
    from google.cloud import storage
    storage_client = storage.Client()
    bucket = storage_client.bucket('my-bucket')
    blob = bucket.blob(f'model-metadata/{model_name}/v{version}/metadata.json')
    blob.upload_from_string(json.dumps(metadata, indent=2))

    # Create model card
    model_card = create_model_card(
        model_name=model_name,
        version=version,
        metrics=metrics,
        metadata=metadata
    )

    blob_card = bucket.blob(f'model-metadata/{model_name}/v{version}/model_card.md')
    blob_card.upload_from_string(model_card)

    return model

def create_model_card(model_name, version, metrics, metadata):
    """Generate comprehensive model card"""
    return f"""
# Model Card: {model_name} v{version}

## Model Details
- **Model Name**: {model_name}
- **Version**: {version}
- **Model Type**: Binary Classification
- **Framework**: TensorFlow 2.12.0
- **Training Date**: {metadata['training']['training_date']}

## Intended Use
- **Primary Use**: Customer churn prediction
- **Out-of-Scope Uses**: Not for use with populations different from training data

## Performance Metrics
- **Accuracy**: {metrics['accuracy']:.4f}
- **AUC-ROC**: {metrics['auc']:.4f}
- **F1 Score**: {metrics['f1']:.4f}
- **Precision**: {metrics['precision']:.4f}
- **Recall**: {metrics['recall']:.4f}

## Training Data
- **Dataset**: {metadata['training']['dataset_uri']}
- **Features**: {', '.join(metadata['features']['input_features'])}
- **Data Split**: 80/20 train/test

## Hyperparameters
{json.dumps(metadata['training']['hyperparameters'], indent=2)}

## Ethical Considerations
- Model should be regularly monitored for bias
- Predictions should not be the sole basis for customer actions

## Limitations
- Model performance may degrade with distribution shift
- Requires retraining every 30 days
"""

# List and compare model versions
def compare_model_versions(model_name):
    """Compare different versions of a model"""
    from google.cloud import storage

    storage_client = storage.Client()
    bucket = storage_client.bucket('my-bucket')

    # List all versions
    blobs = bucket.list_blobs(prefix=f'model-metadata/{model_name}/')

    versions_data = []
    for blob in blobs:
        if blob.name.endswith('metadata.json'):
            metadata = json.loads(blob.download_as_text())
            versions_data.append({
                'version': blob.name.split('/')[2],
                'metrics': metadata['metrics'],
                'training_date': metadata['training']['training_date']
            })

    # Sort by version
    versions_data.sort(key=lambda x: x['version'])

    # Create comparison table
    import pandas as pd
    df = pd.DataFrame(versions_data)
    print(df.to_string())

    return df
```

**Model Lineage Tracking**:
```python
from google.cloud import aiplatform, datacatalog_v1
from google.cloud.datacatalog_v1 import types

def track_model_lineage(
    model_id,
    training_dataset_id,
    feature_engineering_pipeline_id,
    parent_model_id=None
):
    """Track complete model lineage using Data Catalog"""

    # Create lineage entries
    datacatalog_client = datacatalog_v1.DataCatalogClient()
    lineage_client = datacatalog_v1.PolicyTagManagerClient()

    # Create entry for model
    model_entry = types.Entry(
        display_name=f'Model-{model_id}',
        type_=types.EntryType.MODEL,
        linked_resource=f'//aiplatform.googleapis.com/projects/{PROJECT_ID}/locations/{LOCATION}/models/{model_id}',
        schema=types.Schema(
            columns=[
                types.ColumnSchema(
                    column='input_features',
                    type_='RECORD'
                ),
                types.ColumnSchema(
                    column='predictions',
                    type_='DOUBLE'
                )
            ]
        )
    )

    # Link to training dataset
    dataset_entry = types.Entry(
        display_name=f'Dataset-{training_dataset_id}',
        type_=types.EntryType.FILESET,
        linked_resource=training_dataset_id
    )

    # Create lineage graph
    lineage_info = {
        'model_id': model_id,
        'upstream': {
            'training_dataset': training_dataset_id,
            'feature_pipeline': feature_engineering_pipeline_id,
            'parent_model': parent_model_id
        },
        'downstream': {
            'endpoints': [],
            'batch_jobs': []
        },
        'metadata': {
            'creation_time': datetime.now().isoformat(),
            'framework': 'tensorflow',
            'training_method': 'supervised'
        }
    }

    # Store lineage in Cloud Storage
    from google.cloud import storage
    storage_client = storage.Client()
    bucket = storage_client.bucket('my-bucket')
    blob = bucket.blob(f'model-lineage/{model_id}/lineage.json')
    blob.upload_from_string(json.dumps(lineage_info, indent=2))

    return lineage_info

# Query lineage
def get_model_lineage(model_id):
    """Retrieve complete model lineage"""
    from google.cloud import storage

    storage_client = storage.Client()
    bucket = storage_client.bucket('my-bucket')
    blob = bucket.blob(f'model-lineage/{model_id}/lineage.json')

    if blob.exists():
        lineage = json.loads(blob.download_as_text())
        return lineage
    return None
```

## Vertex AI Explainable AI

### Feature Attribution Methods

**Integrated Gradients**:
```python
from google.cloud import aiplatform

# Configure Integrated Gradients explanation
explanation_metadata = {
    'inputs': {
        'age': {
            'input_tensor_name': 'age',
            'encoding': 'IDENTITY',
            'modality': 'numeric',
            'input_baselines': [0.0]
        },
        'subscription_months': {
            'input_tensor_name': 'subscription_months',
            'encoding': 'IDENTITY',
            'modality': 'numeric',
            'input_baselines': [0.0]
        },
        'usage_hours': {
            'input_tensor_name': 'usage_hours',
            'encoding': 'IDENTITY',
            'modality': 'numeric',
            'input_baselines': [0.0]
        }
    },
    'outputs': {
        'churn_probability': {
            'output_tensor_name': 'dense_3'
        }
    },
    'framework': 'tensorflow',
    'tags': ['binary_classification']
}

explanation_parameters = {
    'integrated_gradients_attribution': {
        'step_count': 50,
        'smooth_grad_config': {
            'noise_sigma': 0.1,
            'noisy_sample_count': 50
        }
    },
    'top_k': 10,
    'output_indices': [0]
}

# Upload model with explanation
model = aiplatform.Model.upload(
    display_name='churn-explainable',
    artifact_uri='gs://my-bucket/models/churn/v1',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest',
    explanation_metadata=explanation_metadata,
    explanation_parameters=explanation_parameters
)

# Deploy with explanations
endpoint = aiplatform.Endpoint.create(display_name='explainable-endpoint')
model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn-explainable-v1',
    machine_type='n1-standard-4',
    explanation_metadata=explanation_metadata,
    explanation_parameters=explanation_parameters
)

# Get explanations
instances = [
    {'age': 35, 'subscription_months': 12, 'usage_hours': 45},
    {'age': 42, 'subscription_months': 6, 'usage_hours': 22}
]

explanation = endpoint.explain(instances=instances)

for i, exp in enumerate(explanation.explanations):
    print(f"\nInstance {i+1}:")
    print(f"Prediction: {explanation.predictions[i]}")
    print("Feature Attributions:")

    for attribution in exp.attributions:
        for feature_name, feature_attr in attribution.feature_attributes.items():
            print(f"  {feature_name}: {feature_attr}")
```

**Shapley Values (Sampled Shapley)**:
```python
# Configure Sampled Shapley explanation
explanation_parameters_shapley = {
    'sampled_shapley_attribution': {
        'path_count': 50
    },
    'top_k': 10
}

model_shapley = aiplatform.Model.upload(
    display_name='churn-shapley',
    artifact_uri='gs://my-bucket/models/churn/v1',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest',
    explanation_metadata=explanation_metadata,
    explanation_parameters=explanation_parameters_shapley
)

# Deploy and explain
endpoint_shapley = aiplatform.Endpoint.create(display_name='shapley-endpoint')
model_shapley.deploy(
    endpoint=endpoint_shapley,
    deployed_model_display_name='churn-shapley-v1',
    machine_type='n1-standard-4'
)

# Get Shapley explanations
shapley_explanation = endpoint_shapley.explain(instances=instances)
```

**XRAI for Image Models**:
```python
# Configure XRAI for image classification
image_explanation_metadata = {
    'inputs': {
        'image': {
            'input_tensor_name': 'image_input',
            'encoding': 'IDENTITY',
            'modality': 'image'
        }
    },
    'outputs': {
        'classification': {
            'output_tensor_name': 'dense_output'
        }
    }
}

image_explanation_parameters = {
    'xrai_attribution': {
        'step_count': 50,
        'smooth_grad_config': {
            'noise_sigma': 0.2,
            'noisy_sample_count': 50
        }
    }
}

# Upload and deploy image model with XRAI
image_model = aiplatform.Model.upload(
    display_name='image-classifier-xrai',
    artifact_uri='gs://my-bucket/models/image-classifier/v1',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest',
    explanation_metadata=image_explanation_metadata,
    explanation_parameters=image_explanation_parameters
)
```

**Custom Explanation Visualization**:
```python
import matplotlib.pyplot as plt
import numpy as np

def visualize_feature_attributions(explanation, feature_names):
    """Visualize feature attributions from explanation"""

    # Extract attributions
    attributions = explanation.explanations[0].attributions[0]

    features = []
    values = []

    for feature_name, feature_attr in attributions.feature_attributes.items():
        features.append(feature_name)
        values.append(feature_attr)

    # Sort by absolute value
    sorted_indices = np.argsort(np.abs(values))[::-1]
    features = [features[i] for i in sorted_indices]
    values = [values[i] for i in sorted_indices]

    # Create bar plot
    plt.figure(figsize=(12, 6))
    colors = ['green' if v > 0 else 'red' for v in values]
    plt.barh(features, values, color=colors)
    plt.xlabel('Attribution Value')
    plt.title('Feature Attributions (Integrated Gradients)')
    plt.axvline(x=0, color='black', linestyle='-', linewidth=0.5)
    plt.tight_layout()
    plt.savefig('feature_attributions.png')
    plt.close()

    return features, values

# Compare attributions across methods
def compare_attribution_methods(endpoint_ig, endpoint_shapley, instances):
    """Compare Integrated Gradients vs Shapley attributions"""

    ig_explanation = endpoint_ig.explain(instances=instances)
    shapley_explanation = endpoint_shapley.explain(instances=instances)

    ig_attrs = ig_explanation.explanations[0].attributions[0].feature_attributes
    shapley_attrs = shapley_explanation.explanations[0].attributions[0].feature_attributes

    comparison = pd.DataFrame({
        'Feature': list(ig_attrs.keys()),
        'Integrated_Gradients': list(ig_attrs.values()),
        'Shapley': list(shapley_attrs.values())
    })

    comparison['Difference'] = abs(
        comparison['Integrated_Gradients'] - comparison['Shapley']
    )

    return comparison.sort_values('Difference', ascending=False)
```

## CI/CD for ML Models

### Cloud Build Pipeline for ML

**cloudbuild.yaml for ML Pipeline**:
```yaml
# cloudbuild.yaml
steps:
  # Step 1: Run unit tests
  - name: 'python:3.9'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        pip install -r requirements.txt
        pytest tests/unit/ -v --cov=src

  # Step 2: Data validation
  - name: 'python:3.9'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        pip install tensorflow-data-validation
        python scripts/validate_data.py \
          --input-data gs://my-bucket/data/latest_data.csv \
          --schema gs://my-bucket/schemas/data_schema.pbtxt

  # Step 3: Model training
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'ai'
      - 'custom-jobs'
      - 'create'
      - '--region=us-central1'
      - '--display-name=ci-model-training'
      - '--config=training_config.yaml'

  # Step 4: Model evaluation
  - name: 'python:3.9'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        python scripts/evaluate_model.py \
          --model-path gs://my-bucket/models/candidate/ \
          --test-data gs://my-bucket/data/test_data.csv \
          --metrics-output metrics.json

  # Step 5: Model validation (performance gate)
  - name: 'python:3.9'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        python scripts/validate_metrics.py \
          --metrics metrics.json \
          --min-auc 0.85 \
          --min-accuracy 0.80

  # Step 6: Build custom serving container
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'build'
      - '-t'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/ml-models/churn-serving:$SHORT_SHA'
      - '-f'
      - 'serving/Dockerfile'
      - '.'

  # Step 7: Push container
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'push'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/ml-models/churn-serving:$SHORT_SHA'

  # Step 8: Register model
  - name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        python scripts/register_model.py \
          --model-path gs://my-bucket/models/candidate/ \
          --container-image us-central1-docker.pkg.dev/$PROJECT_ID/ml-models/churn-serving:$SHORT_SHA \
          --model-name churn-model \
          --version $SHORT_SHA

  # Step 9: Deploy to staging endpoint
  - name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        python scripts/deploy_model.py \
          --model-id $SHORT_SHA \
          --endpoint staging-churn-endpoint \
          --traffic-percentage 100

  # Step 10: Integration tests on staging
  - name: 'python:3.9'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        pip install -r requirements.txt
        pytest tests/integration/ -v \
          --endpoint staging-churn-endpoint

  # Step 11: Deploy to production (canary)
  - name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        if [ "$BRANCH_NAME" = "main" ]; then
          python scripts/deploy_model.py \
            --model-id $SHORT_SHA \
            --endpoint production-churn-endpoint \
            --traffic-percentage 10 \
            --deployment-type canary
        fi

timeout: 3600s
options:
  machineType: 'N1_HIGHCPU_8'
  logging: CLOUD_LOGGING_ONLY

substitutions:
  _REGION: 'us-central1'
```

**Model Validation Script**:
```python
# scripts/validate_metrics.py
import json
import sys
import argparse

def validate_metrics(metrics_file, min_auc, min_accuracy):
    """Validate model metrics meet minimum thresholds"""

    with open(metrics_file, 'r') as f:
        metrics = json.load(f)

    validation_results = {
        'auc': metrics['auc'] >= min_auc,
        'accuracy': metrics['accuracy'] >= min_accuracy,
        'passed': True
    }

    # Check all metrics
    if metrics['auc'] < min_auc:
        print(f"FAILED: AUC {metrics['auc']:.4f} < minimum {min_auc}")
        validation_results['passed'] = False

    if metrics['accuracy'] < min_accuracy:
        print(f"FAILED: Accuracy {metrics['accuracy']:.4f} < minimum {min_accuracy}")
        validation_results['passed'] = False

    # Additional checks
    if metrics.get('precision', 1.0) < 0.75:
        print(f"WARNING: Low precision {metrics['precision']:.4f}")

    if metrics.get('recall', 1.0) < 0.70:
        print(f"WARNING: Low recall {metrics['recall']:.4f}")

    if validation_results['passed']:
        print("SUCCESS: All validation checks passed")
        sys.exit(0)
    else:
        print("FAILED: Model validation failed")
        sys.exit(1)

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--metrics', required=True)
    parser.add_argument('--min-auc', type=float, default=0.85)
    parser.add_argument('--min-accuracy', type=float, default=0.80)
    args = parser.parse_args()

    validate_metrics(args.metrics, args.min_auc, args.min_accuracy)
```

## A/B Testing and Advanced Deployment

### A/B Testing Implementation

**Complete A/B Testing Framework**:
```python
from google.cloud import aiplatform, bigquery, monitoring_v3
from datetime import datetime, timedelta
import numpy as np
from scipy import stats
import time

class ABTestManager:
    def __init__(self, project_id, location, endpoint_name):
        self.project_id = project_id
        self.location = location
        self.endpoint_name = endpoint_name

        aiplatform.init(project=project_id, location=location)
        self.endpoint = aiplatform.Endpoint.list(
            filter=f'display_name="{endpoint_name}"'
        )[0]

        self.bq_client = bigquery.Client()

    def start_ab_test(self, model_a_id, model_b_id, test_duration_days=7, initial_traffic_b=10):
        """Start A/B test with gradual rollout"""

        print(f"Starting A/B test: {model_a_id} vs {model_b_id}")

        # Deploy model B with initial traffic
        self.endpoint.update(
            traffic_split={
                model_a_id: 100 - initial_traffic_b,
                model_b_id: initial_traffic_b
            }
        )

        # Log test start
        self._log_ab_test_event({
            'event': 'test_start',
            'model_a': model_a_id,
            'model_b': model_b_id,
            'initial_traffic_b': initial_traffic_b,
            'test_duration_days': test_duration_days
        })

        return {
            'test_id': f'ab_test_{datetime.now().strftime("%Y%m%d_%H%M%S")}',
            'model_a': model_a_id,
            'model_b': model_b_id,
            'start_time': datetime.now()
        }

    def evaluate_ab_test(self, model_a_id, model_b_id, metric='auc'):
        """Evaluate A/B test results using statistical significance"""

        # Query predictions from BigQuery
        query = f"""
        WITH predictions AS (
            SELECT
                model_id,
                prediction,
                actual_label,
                prediction_time
            FROM `{self.project_id}.predictions.prediction_log`
            WHERE
                prediction_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
                AND model_id IN ('{model_a_id}', '{model_b_id}')
        )
        SELECT
            model_id,
            COUNT(*) as sample_size,
            AVG(CAST(prediction AS FLOAT64)) as avg_prediction,
            AVG(CASE WHEN prediction = actual_label THEN 1.0 ELSE 0.0 END) as accuracy,
            STDDEV(CAST(prediction AS FLOAT64)) as std_prediction
        FROM predictions
        GROUP BY model_id
        """

        results = self.bq_client.query(query).to_dataframe()

        model_a_stats = results[results['model_id'] == model_a_id].iloc[0]
        model_b_stats = results[results['model_id'] == model_b_id].iloc[0]

        # Perform t-test
        t_statistic, p_value = stats.ttest_ind_from_stats(
            mean1=model_a_stats['avg_prediction'],
            std1=model_a_stats['std_prediction'],
            nobs1=model_a_stats['sample_size'],
            mean2=model_b_stats['avg_prediction'],
            std2=model_b_stats['std_prediction'],
            nobs2=model_b_stats['sample_size']
        )

        # Calculate confidence interval
        confidence_level = 0.95
        degrees_of_freedom = model_a_stats['sample_size'] + model_b_stats['sample_size'] - 2
        critical_value = stats.t.ppf((1 + confidence_level) / 2, degrees_of_freedom)

        # Determine winner
        is_significant = p_value < 0.05
        winner = model_b_id if model_b_stats['accuracy'] > model_a_stats['accuracy'] else model_a_id

        results_dict = {
            'model_a': {
                'id': model_a_id,
                'accuracy': float(model_a_stats['accuracy']),
                'sample_size': int(model_a_stats['sample_size'])
            },
            'model_b': {
                'id': model_b_id,
                'accuracy': float(model_b_stats['accuracy']),
                'sample_size': int(model_b_stats['sample_size'])
            },
            'statistical_significance': {
                'p_value': float(p_value),
                'is_significant': bool(is_significant),
                't_statistic': float(t_statistic)
            },
            'winner': winner if is_significant else 'inconclusive',
            'improvement': float(abs(model_b_stats['accuracy'] - model_a_stats['accuracy']))
        }

        return results_dict

    def gradual_rollout(self, model_a_id, model_b_id, monitoring_window_hours=6):
        """Gradually increase traffic to winning model"""

        rollout_schedule = [
            (90, 10),   # Day 1: 10% B
            (80, 20),   # Day 2: 20% B
            (70, 30),   # Day 3: 30% B
            (50, 50),   # Day 4: 50% B
            (30, 70),   # Day 5: 70% B
            (10, 90),   # Day 6: 90% B
            (0, 100)    # Day 7: 100% B
        ]

        for traffic_a, traffic_b in rollout_schedule:
            print(f"Updating traffic split: A={traffic_a}%, B={traffic_b}%")

            # Update traffic split
            self.endpoint.update(
                traffic_split={
                    model_a_id: traffic_a,
                    model_b_id: traffic_b
                }
            )

            # Wait for monitoring window
            time.sleep(monitoring_window_hours * 3600)

            # Check health metrics
            health_check = self._check_deployment_health(model_b_id, hours=monitoring_window_hours)

            if not health_check['healthy']:
                print(f"Health check failed: {health_check['reason']}")
                print("Rolling back to model A")

                self.endpoint.update(
                    traffic_split={
                        model_a_id: 100,
                        model_b_id: 0
                    }
                )

                return {
                    'status': 'rollback',
                    'reason': health_check['reason'],
                    'rollout_stage': f'{traffic_b}%'
                }

        print("Gradual rollout completed successfully")
        return {'status': 'success', 'final_traffic': {'A': 0, 'B': 100}}

    def _check_deployment_health(self, model_id, hours=1):
        """Check deployment health metrics"""

        # Query error rate
        query = f"""
        SELECT
            COUNT(*) as total_predictions,
            COUNTIF(error IS NOT NULL) as error_count,
            AVG(latency_ms) as avg_latency,
            MAX(latency_ms) as max_latency
        FROM `{self.project_id}.predictions.prediction_log`
        WHERE
            model_id = '{model_id}'
            AND prediction_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL {hours} HOUR)
        """

        results = self.bq_client.query(query).to_dataframe().iloc[0]

        error_rate = results['error_count'] / results['total_predictions'] if results['total_predictions'] > 0 else 0
        avg_latency = results['avg_latency']

        # Health criteria
        healthy = True
        reason = ""

        if error_rate > 0.05:  # 5% error rate threshold
            healthy = False
            reason = f"High error rate: {error_rate:.2%}"
        elif avg_latency > 1000:  # 1 second latency threshold
            healthy = False
            reason = f"High latency: {avg_latency:.0f}ms"

        return {
            'healthy': healthy,
            'reason': reason,
            'metrics': {
                'error_rate': float(error_rate),
                'avg_latency': float(avg_latency),
                'total_predictions': int(results['total_predictions'])
            }
        }

    def _log_ab_test_event(self, event_data):
        """Log A/B test events to BigQuery"""

        table_id = f"{self.project_id}.ml_experiments.ab_test_log"

        row = {
            'timestamp': datetime.now().isoformat(),
            'endpoint_name': self.endpoint_name,
            **event_data
        }

        errors = self.bq_client.insert_rows_json(table_id, [row])
        if errors:
            print(f"Error logging event: {errors}")

# Usage example
ab_tester = ABTestManager(
    project_id='my-project',
    location='us-central1',
    endpoint_name='churn-prediction-endpoint'
)

# Start A/B test
test_info = ab_tester.start_ab_test(
    model_a_id='model_v1',
    model_b_id='model_v2',
    test_duration_days=7,
    initial_traffic_b=10
)

# Wait and evaluate
time.sleep(86400)  # Wait 24 hours
results = ab_tester.evaluate_ab_test('model_v1', 'model_v2')

if results['winner'] == 'model_v2' and results['statistical_significance']['is_significant']:
    # Gradually roll out winning model
    rollout_result = ab_tester.gradual_rollout('model_v1', 'model_v2')
    print(f"Rollout result: {rollout_result}")
```

**Multi-Armed Bandit Approach**:
```python
import numpy as np

class ThompsonSamplingMAB:
    """Thompson Sampling for multi-armed bandit A/B testing"""

    def __init__(self, n_models):
        self.n_models = n_models
        self.successes = np.ones(n_models)  # Beta prior
        self.failures = np.ones(n_models)

    def select_model(self):
        """Select model using Thompson Sampling"""
        samples = np.random.beta(self.successes, self.failures)
        return np.argmax(samples)

    def update(self, model_id, reward):
        """Update belief about model performance"""
        if reward > 0:
            self.successes[model_id] += 1
        else:
            self.failures[model_id] += 1

    def get_probabilities(self):
        """Get current model selection probabilities"""
        return self.successes / (self.successes + self.failures)

# Implement MAB-based traffic routing
def mab_traffic_router():
    """Route traffic using multi-armed bandit"""
    mab = ThompsonSamplingMAB(n_models=2)

    # Cloud Function for routing
    function_code = '''
from google.cloud import aiplatform
import numpy as np
import json

# Load MAB state from Cloud Storage
mab = load_mab_state()

def route_prediction(request):
    """Route prediction request to model selected by MAB"""
    instances = request.get_json()['instances']

    # Select model
    model_id = mab.select_model()

    # Make prediction
    endpoint = get_endpoint()
    prediction = endpoint.predict(
        instances=instances,
        deployed_model_id=get_model_ids()[model_id]
    )

    # Calculate reward (simplified)
    reward = 1 if prediction.predictions[0] > 0.8 else 0

    # Update MAB
    mab.update(model_id, reward)
    save_mab_state(mab)

    return json.dumps({
        'prediction': prediction.predictions[0],
        'model_id': model_id
    })
'''

    return function_code
```

## Production MLOps Scenarios

### Scenario 1: Real-Time Fraud Detection System

**Requirements**:
- Sub-100ms latency
- 99.9% availability
- Real-time monitoring
- Automated retraining

**Implementation**:
```python
# 1. Optimized model deployment
from google.cloud import aiplatform

# Deploy with GPU for low latency
model = aiplatform.Model.upload(
    display_name='fraud-detection-optimized',
    artifact_uri='gs://my-bucket/models/fraud/optimized',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-gpu.2-12:latest'
)

endpoint = aiplatform.Endpoint.create(
    display_name='fraud-detection-prod',
    network='projects/123/global/networks/private-vpc'  # Private endpoint
)

model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='fraud-v1',
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1,
    min_replica_count=3,  # High availability
    max_replica_count=20,
    autoscaling_target_cpu_utilization=60,  # Aggressive scaling
    enable_access_logging=True
)

# 2. Feature caching with Memorystore
from google.cloud import redis_v1
import redis

redis_client = redis.Redis(
    host='10.0.0.3',  # Memorystore IP
    port=6379,
    decode_responses=True
)

def get_features_with_cache(customer_id):
    """Get features with Redis caching"""
    cache_key = f'features:{customer_id}'

    # Check cache
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)

    # Fetch from Firestore/Bigtable
    features = fetch_features_from_db(customer_id)

    # Cache for 5 minutes
    redis_client.setex(cache_key, 300, json.dumps(features))

    return features

# 3. Real-time prediction with monitoring
def predict_fraud(transaction_data):
    """Make fraud prediction with monitoring"""
    start_time = time.time()

    # Get cached features
    features = get_features_with_cache(transaction_data['customer_id'])

    # Combine with transaction data
    instances = [{**transaction_data, **features}]

    # Predict
    prediction = endpoint.predict(instances=instances)

    # Calculate latency
    latency_ms = (time.time() - start_time) * 1000

    # Log metrics
    log_prediction_metrics(
        latency=latency_ms,
        prediction=prediction.predictions[0],
        customer_id=transaction_data['customer_id']
    )

    # Alert if high risk
    if prediction.predictions[0] > 0.9:
        send_alert(transaction_data, prediction.predictions[0])

    return prediction.predictions[0]

# 4. Automated retraining on new fraud patterns
def setup_fraud_retraining():
    """Retrain when new fraud patterns detected"""

    # Query recent fraud cases
    query = """
    SELECT COUNT(*) as new_fraud_count
    FROM `project.fraud.transactions`
    WHERE
        is_fraud = TRUE
        AND transaction_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
    """

    # Trigger retraining if threshold exceeded
    if new_fraud_count > 100:
        trigger_retraining_pipeline()
```

**Key Design Decisions**:
- Private VPC endpoint for security
- GPU acceleration for low latency
- Redis caching for frequently accessed features
- Aggressive autoscaling (60% CPU target)
- Real-time monitoring and alerting

### Scenario 2: Large-Scale Recommendation System

**Requirements**:
- Serve millions of users
- Personalized recommendations
- Batch precomputation + online serving
- Cost optimization

**Implementation**:
```python
# 1. Batch precompute recommendations daily
def daily_recommendation_batch():
    """Precompute recommendations for all users"""

    # Batch prediction on all users
    batch_job = model.batch_predict(
        job_display_name='daily-recommendations',
        bigquery_source='bq://project.dataset.user_features',
        bigquery_destination_prefix='bq://project.dataset',
        machine_type='n1-highmem-32',
        starting_replica_count=50,
        max_replica_count=100,
        accelerator_type='NVIDIA_TESLA_T4',
        accelerator_count=1
    )

    batch_job.wait()

    # Store results in Bigtable for fast lookup
    store_recommendations_in_bigtable(
        batch_job.output_info.bigquery_output_table
    )

# 2. Online serving for real-time personalization
def serve_recommendations(user_id, context):
    """Serve recommendations with real-time context"""

    # Get precomputed recommendations from Bigtable
    base_recs = get_from_bigtable(user_id)

    # Apply real-time reranking
    instances = [{
        'user_id': user_id,
        'base_recs': base_recs,
        'context': context  # Real-time signals
    }]

    # Reranking model (lightweight)
    reranked = reranking_endpoint.predict(instances=instances)

    return reranked.predictions[0]

# 3. Two-tier deployment architecture
# Tier 1: Candidate generation (batch)
candidate_model = aiplatform.Model('candidate-generation-model')
batch_job = candidate_model.batch_predict(...)  # Daily

# Tier 2: Real-time reranking (online)
reranking_model = aiplatform.Model('reranking-model')
reranking_model.deploy(
    endpoint=reranking_endpoint,
    machine_type='n1-standard-4',
    min_replica_count=10,
    max_replica_count=50
)

# 4. Cost optimization with spot VMs for batch
from google.cloud import aiplatform_v1

batch_job_config = {
    'model_name': candidate_model.resource_name,
    'input_config': {...},
    'output_config': {...},
    'dedicated_resources': {
        'machine_spec': {
            'machine_type': 'n1-highmem-32'
        },
        'starting_replica_count': 50,
        'max_replica_count': 100
    },
    'manual_batch_tuning_parameters': {
        'batch_size': 64
    }
}
```

**Key Design Decisions**:
- Two-tier architecture: batch + online
- Bigtable for fast precomputed lookups
- Lightweight reranking model for real-time
- Batch with spot VMs for cost optimization

### Scenario 3: Medical Diagnosis Model with Explainability

**Requirements**:
- High accuracy
- Explainable predictions
- Compliance and audit trail
- Model governance

**Implementation**:
```python
# 1. Deploy with full explainability
explanation_metadata = {
    'inputs': {
        'age': {'input_tensor_name': 'age', 'encoding': 'IDENTITY', 'modality': 'numeric'},
        'symptoms': {'input_tensor_name': 'symptoms', 'encoding': 'BAG_OF_FEATURES', 'modality': 'categorical'},
        'test_results': {'input_tensor_name': 'test_results', 'encoding': 'IDENTITY', 'modality': 'numeric'}
    },
    'outputs': {
        'diagnosis': {'output_tensor_name': 'diagnosis_output'}
    }
}

explanation_parameters = {
    'integrated_gradients_attribution': {
        'step_count': 50
    },
    'top_k': 5
}

model.deploy(
    endpoint=endpoint,
    explanation_metadata=explanation_metadata,
    explanation_parameters=explanation_parameters
)

# 2. Complete audit trail
def make_diagnosis_with_audit(patient_data):
    """Make diagnosis with complete audit trail"""

    # Get prediction with explanation
    explanation = endpoint.explain(instances=[patient_data])

    prediction = explanation.predictions[0]
    attributions = explanation.explanations[0].attributions[0]

    # Log to audit trail
    audit_entry = {
        'timestamp': datetime.now().isoformat(),
        'patient_id': patient_data['patient_id'],
        'prediction': prediction,
        'confidence': prediction['confidence'],
        'model_version': deployed_model.id,
        'feature_attributions': dict(attributions.feature_attributes),
        'clinician_id': patient_data['clinician_id']
    }

    store_audit_entry(audit_entry)

    return {
        'diagnosis': prediction['diagnosis'],
        'confidence': prediction['confidence'],
        'explanation': format_explanation_for_clinician(attributions)
    }

# 3. Model governance and approval workflow
def deploy_medical_model_with_governance(model, approvers):
    """Deploy model with approval workflow"""

    # Upload model to registry
    registered_model = register_model_with_metadata(
        model_artifact_uri=model.artifact_uri,
        model_name='diagnosis-model',
        version='v2',
        metrics=model.evaluation_metrics,
        hyperparameters=model.hyperparameters,
        training_dataset_uri=model.training_data_uri
    )

    # Create model card
    model_card = create_comprehensive_model_card(registered_model)

    # Request approval
    approval_request = {
        'model_id': registered_model.resource_name,
        'model_card': model_card,
        'approvers': approvers,
        'approval_required_by': ['medical_director', 'compliance_officer']
    }

    send_approval_request(approval_request)

    # Wait for approval
    wait_for_approval(approval_request['model_id'])

    # Deploy after approval
    registered_model.deploy(endpoint=endpoint)

# 4. Continuous monitoring for model drift
monitoring_job = aiplatform.ModelDeploymentMonitoringJob.create(
    display_name='medical-model-monitoring',
    endpoint=endpoint,
    logging_sampling_strategy={'random_sample_config': {'sample_rate': 1.0}},  # 100% sampling
    model_deployment_monitoring_objective_configs=[{
        'deployed_model_id': deployed_model.id,
        'objective_config': {
            'training_dataset': training_dataset,
            'training_prediction_skew_detection_config': {
                'skew_thresholds': {'age': {'value': 0.05}}  # Strict thresholds
            },
            'prediction_drift_detection_config': {
                'drift_thresholds': {'age': {'value': 0.05}}
            }
        }
    }]
)
```

**Key Design Decisions**:
- Integrated Gradients for explainability
- Complete audit trail for compliance
- Approval workflow for governance
- 100% prediction sampling for monitoring
- Strict drift detection thresholds

### Scenario 4: Multi-Model Ensemble Deployment

**Requirements**:
- Combine multiple models
- Weighted ensemble
- A/B test different ensemble strategies

**Implementation**:
```python
# 1. Deploy multiple models
models = []
for i, model_uri in enumerate(['gs://bucket/model1', 'gs://bucket/model2', 'gs://bucket/model3']):
    model = aiplatform.Model.upload(
        display_name=f'ensemble-model-{i+1}',
        artifact_uri=model_uri,
        serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest'
    )

    model.deploy(
        endpoint=endpoint,
        deployed_model_display_name=f'model-{i+1}',
        machine_type='n1-standard-4',
        traffic_percentage=0  # No direct traffic
    )

    models.append(model)

# 2. Ensemble prediction service
from flask import Flask, request, jsonify
import numpy as np

app = Flask(__name__)

@app.route('/predict', methods=['POST'])
def ensemble_predict():
    """Weighted ensemble prediction"""
    instances = request.json['instances']

    # Get predictions from all models
    predictions = []
    for i, model in enumerate(models):
        pred = endpoint.predict(
            instances=instances,
            deployed_model_id=model.id
        )
        predictions.append(pred.predictions)

    # Weighted average
    weights = [0.5, 0.3, 0.2]  # Based on validation performance
    ensemble_pred = np.average(predictions, axis=0, weights=weights)

    return jsonify({
        'ensemble_prediction': ensemble_pred.tolist(),
        'individual_predictions': predictions
    })

# 3. A/B test different ensemble strategies
ensemble_strategies = {
    'weighted_avg': lambda preds: np.average(preds, axis=0, weights=[0.5, 0.3, 0.2]),
    'max_vote': lambda preds: np.argmax(np.bincount(np.array(preds).flatten().astype(int))),
    'stacking': lambda preds: stacking_model.predict(preds)
}

# Route traffic to different strategies
@app.route('/predict_ab', methods=['POST'])
def ab_ensemble_predict():
    """A/B test ensemble strategies"""
    instances = request.json['instances']
    user_id = request.json['user_id']

    # Get predictions from all models
    predictions = get_all_model_predictions(instances)

    # Select strategy based on user_id (consistent hashing)
    strategy = select_strategy_for_user(user_id)

    # Apply ensemble strategy
    final_prediction = ensemble_strategies[strategy](predictions)

    # Log for A/B analysis
    log_ab_prediction(user_id, strategy, final_prediction)

    return jsonify({'prediction': final_prediction.tolist()})
```

## Exam Tips and Decision Framework

### Deployment Strategy Selection

**When to use Online Prediction**:
- Real-time predictions needed (< 1 second latency)
- Low to medium prediction volume (< 1M per day)
- Dynamic feature values
- Interactive applications

**When to use Batch Prediction**:
- Large volume predictions (> 1M per day)
- Predictions can be precomputed
- Cost optimization priority
- Non-time-sensitive use cases

**When to use Private Endpoints**:
- Security requirements (HIPAA, PCI-DSS)
- Internal applications only
- VPC-peered architecture
- No public internet exposure needed

### Monitoring Strategy Selection

**Training-Serving Skew Detection**:
- Use when: New model deployed
- Monitor: First 7-14 days after deployment
- Threshold: 0.1-0.15 (10-15% skew)
- Action: Retrain if exceeded

**Prediction Drift Detection**:
- Use when: Model in production
- Monitor: Continuous
- Threshold: 0.15-0.20 (15-20% drift)
- Action: Automated retraining pipeline

**Feature Attribution Monitoring**:
- Use when: Model interpretability required
- Monitor: Weekly snapshots
- Threshold: Major shifts in top features
- Action: Investigate data quality issues

### MLOps Pattern Selection

**CI/CD Pipeline**:
- Automated tests (unit, integration)
- Data validation (TFDV)
- Model validation (performance gates)
- Canary deployment (10% → 100%)
- Automated rollback on failure

**Continuous Training**:
- Scheduled: Weekly/monthly retraining
- Event-driven: Drift-triggered retraining
- Hybrid: Scheduled + drift-triggered

**A/B Testing Strategy**:
- Statistical: For business metrics (conversion, revenue)
- Multi-armed bandit: For exploration-exploitation
- Champion-challenger: For gradual rollout

### Exam Question Patterns

**Pattern 1: "Which deployment strategy?"**
- Keywords: latency, cost, volume, security
- Decision tree:
  - Real-time + low volume → Online prediction
  - Batch + high volume → Batch prediction
  - Security/private → Private endpoint
  - Gradual rollout → Canary deployment

**Pattern 2: "How to detect model degradation?"**
- Keywords: drift, skew, monitoring
- Answer options:
  - Training-serving skew: New deployment
  - Prediction drift: Production monitoring
  - Feature attribution: Interpretability
  - Custom metrics: Business KPIs

**Pattern 3: "How to optimize serving?"**
- Keywords: latency, throughput, cost
- Options:
  - GPU: Large models, low latency
  - Autoscaling: Variable traffic
  - Caching: Repeated predictions
  - Batch: High volume, precompute

**Pattern 4: "How to ensure model quality?"**
- Keywords: validation, testing, governance
- Components:
  - Data validation (TFDV)
  - Model evaluation (performance gates)
  - A/B testing (production validation)
  - Monitoring (ongoing quality)

### Key Formulas and Thresholds

**Skew/Drift Detection**:
- L-infinity distance: max|P(feature) - Q(feature)|
- Threshold: 0.1-0.15 for skew, 0.15-0.20 for drift

**A/B Test Sample Size**:
```
n = (Z_α/2 + Z_β)² × (σ_A² + σ_B²) / (μ_A - μ_B)²
```
- Typical: p < 0.05, power = 0.80

**Autoscaling Formula**:
```
replicas = ceil(current_metric / target_metric × current_replicas)
```
- Min/max bounds apply

## Best Practices Checklist

### Deployment
- [ ] Use managed endpoints for production
- [ ] Implement canary deployments (10% → 25% → 50% → 100%)
- [ ] Set up autoscaling with appropriate thresholds
- [ ] Use accelerators (GPUs) for large models
- [ ] Implement private endpoints for sensitive data
- [ ] Version all models with semantic versioning
- [ ] Document deployment procedures
- [ ] Set up automated rollback

### Monitoring
- [ ] Enable prediction logging (sampled)
- [ ] Set up training-serving skew detection
- [ ] Configure prediction drift monitoring
- [ ] Create custom metrics dashboards
- [ ] Set up alerting (PagerDuty, email)
- [ ] Monitor infrastructure metrics (CPU, memory, latency)
- [ ] Track business metrics (accuracy, precision, recall)
- [ ] Implement explanation monitoring

### MLOps
- [ ] Automate training pipelines (Vertex AI Pipelines)
- [ ] Version datasets and features
- [ ] Implement CI/CD with Cloud Build
- [ ] Use Feature Store for consistency
- [ ] Track model lineage and metadata
- [ ] Implement automated testing (unit, integration)
- [ ] Set up continuous monitoring
- [ ] Schedule regular retraining
- [ ] Maintain model registry with model cards

### Cost Optimization
- [ ] Use batch prediction for high volume
- [ ] Implement caching for repeated requests
- [ ] Right-size machine types
- [ ] Use preemptible VMs for training
- [ ] Set appropriate autoscaling bounds
- [ ] Monitor and optimize serving costs
- [ ] Use committed use discounts

## Study Tips for Professional ML Engineer

1. **Hands-On Practice**:
   - Deploy at least 3 models to Vertex AI endpoints
   - Configure monitoring and drift detection
   - Build end-to-end Kubeflow pipeline
   - Implement A/B testing with traffic splitting

2. **Key Concepts to Master**:
   - Difference between online and batch prediction
   - Training-serving skew vs prediction drift
   - Canary deployment vs blue-green deployment
   - Integrated Gradients vs Shapley values
   - CI/CD for ML pipelines

3. **Exam Focus Areas**:
   - Deployment strategy selection (40% of deployment questions)
   - Monitoring and drift detection (30%)
   - MLOps pipelines and automation (20%)
   - Explainability and governance (10%)

4. **Common Pitfalls to Avoid**:
   - Deploying directly to production (always use staging)
   - Not monitoring for drift (set up day one)
   - Insufficient autoscaling headroom (plan for 2x peak)
   - Not logging predictions (needed for retraining)

## Additional Resources

- [Vertex AI Prediction Documentation](https://cloud.google.com/vertex-ai/docs/predictions/overview)
- [Model Monitoring Guide](https://cloud.google.com/vertex-ai/docs/model-monitoring/overview)
- [Vertex AI Pipelines Documentation](https://cloud.google.com/vertex-ai/docs/pipelines/introduction)
- [MLOps: Continuous delivery and automation pipelines](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
- [Kubeflow Pipelines SDK](https://www.kubeflow.org/docs/components/pipelines/)
- [Vertex AI Explainable AI](https://cloud.google.com/vertex-ai/docs/explainable-ai/overview)
