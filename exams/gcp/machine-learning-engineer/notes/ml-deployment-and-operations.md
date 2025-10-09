# ML Deployment and Operations - GCP Professional Machine Learning Engineer

## Overview

Model deployment, serving, monitoring, MLOps practices, and production ML systems for the Professional Machine Learning Engineer certification.

## Model Deployment

### Vertex AI Endpoints
**Online Prediction**:
```python
from google.cloud import aiplatform

# Deploy model to endpoint
endpoint = aiplatform.Endpoint.create(display_name='prediction-endpoint')

model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='v1',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5,
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1
)

# Make prediction
prediction = endpoint.predict(instances=[{
    'age': 35,
    'gender': 'F',
    'subscription_months': 12
}])
```

**Traffic Splitting**:
```python
# Deploy new version
model_v2.deploy(
    endpoint=endpoint,
    deployed_model_display_name='v2',
    machine_type='n1-standard-4',
    traffic_percentage=10  # Canary 10%
)

# Adjust traffic
endpoint.update(
    traffic_split={
        'deployed_model_v1_id': 90,
        'deployed_model_v2_id': 10
    }
)
```

### Batch Prediction
```python
batch_prediction_job = model.batch_predict(
    job_display_name='batch-prediction',
    gcs_source='gs://bucket/input/*.jsonl',
    gcs_destination_prefix='gs://bucket/output/',
    machine_type='n1-standard-4',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1,
    starting_replica_count=10,
    max_replica_count=20
)

# Wait for completion
batch_prediction_job.wait()
```

## Model Monitoring

### Prediction Drift Detection
```python
from google.cloud import aiplatform

# Configure monitoring
monitoring_job = aiplatform.ModelDeploymentMonitoringJob.create(
    display_name='churn-model-monitoring',
    endpoint=endpoint,
    logging_sampling_strategy={'random_sample_config': {'sample_rate': 0.1}},
    model_monitoring_alert_config={
        'email_alert_config': {'user_emails': ['team@example.com']},
        'notification_channels': [CHANNEL_ID]
    },
    model_deployment_monitoring_objective_configs=[
        {
            'deployed_model_id': deployed_model.id,
            'objective_config': {
                'training_dataset': {
                    'target_field': 'churned',
                    'gcs_source': {'uris': ['gs://bucket/training_data.csv']}
                },
                'training_prediction_skew_detection_config': {
                    'skew_thresholds': {
                        'age': {'value': 0.1},
                        'subscription_months': {'value': 0.15}
                    }
                },
                'prediction_drift_detection_config': {
                    'drift_thresholds': {
                        'age': {'value': 0.1},
                        'subscription_months': {'value': 0.15}
                    }
                }
            }
        }
    ]
)
```

### Custom Metrics
```python
from google.cloud import monitoring_v3
import time

def log_prediction_metric(project_id, model_name, latency_ms, prediction_value):
    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{project_id}"

    series = monitoring_v3.TimeSeries()
    series.metric.type = f"custom.googleapis.com/ml/{model_name}/prediction_latency"
    series.resource.type = "global"

    point = series.points.add()
    point.value.double_value = latency_ms
    point.interval.end_time.seconds = int(time.time())

    client.create_time_series(name=project_name, time_series=[series])
```

## MLOps Pipelines

### Vertex AI Pipelines
**Pipeline Definition**:
```python
from kfp.v2 import dsl, compiler
from google.cloud import aiplatform

@dsl.component
def preprocess_data(
    input_data: dsl.Input[dsl.Dataset],
    output_data: dsl.Output[dsl.Dataset]
):
    import pandas as pd
    df = pd.read_csv(input_data.path)
    # Preprocessing logic
    df.to_csv(output_data.path, index=False)

@dsl.component
def train_model(
    training_data: dsl.Input[dsl.Dataset],
    model: dsl.Output[dsl.Model],
    learning_rate: float = 0.01
):
    # Training logic
    pass

@dsl.component
def evaluate_model(
    model: dsl.Input[dsl.Model],
    test_data: dsl.Input[dsl.Dataset],
    metrics: dsl.Output[dsl.Metrics]
):
    # Evaluation logic
    pass

@dsl.pipeline(
    name='ml-training-pipeline',
    description='End-to-end ML pipeline'
)
def ml_pipeline(
    input_data_uri: str,
    learning_rate: float = 0.01
):
    preprocess_task = preprocess_data(input_data=input_data_uri)
    train_task = train_model(
        training_data=preprocess_task.outputs['output_data'],
        learning_rate=learning_rate
    )
    evaluate_task = evaluate_model(
        model=train_task.outputs['model'],
        test_data=preprocess_task.outputs['output_data']
    )

# Compile and run
compiler.Compiler().compile(
    pipeline_func=ml_pipeline,
    package_path='pipeline.json'
)

job = aiplatform.PipelineJob(
    display_name='ml-pipeline-run',
    template_path='pipeline.json',
    parameter_values={
        'input_data_uri': 'gs://bucket/data.csv',
        'learning_rate': 0.01
    }
)
job.run()
```

### Continuous Training
```python
from google.cloud import aiplatform, scheduler_v1

def setup_retraining_schedule():
    # Cloud Scheduler trigger
    client = scheduler_v1.CloudSchedulerClient()
    parent = f"projects/{PROJECT_ID}/locations/{LOCATION}"

    job = {
        'name': f'{parent}/jobs/model-retraining',
        'schedule': '0 0 * * 0',  # Weekly on Sunday
        'time_zone': 'America/New_York',
        'http_target': {
            'uri': f'https://{LOCATION}-aiplatform.googleapis.com/v1/projects/{PROJECT_ID}/locations/{LOCATION}/pipelineJobs',
            'http_method': 'POST',
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({
                'pipelineSpec': {...},
                'runtimeConfig': {...}
            }).encode()
        }
    }

    client.create_job(parent=parent, job=job)
```

## Model Serving Optimization

### Model Optimization
**TensorFlow Serving**:
```python
# Optimize SavedModel
import tensorflow as tf

# Convert to TFLite
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model/')
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
tflite_model = converter.convert()

# Save optimized model
with open('model.tflite', 'wb') as f:
    f.write(tflite_model)
```

### Caching and Load Balancing
```python
from flask import Flask, request, jsonify
from functools import lru_cache
import redis

app = Flask(__name__)
cache = redis.Redis(host='memorystore-ip', port=6379)

@lru_cache(maxsize=1000)
def get_prediction_cached(features_hash):
    # Cache predictions for repeated requests
    return model.predict(features)

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    features_hash = hash(tuple(sorted(data.items())))

    # Check Redis cache
    cached = cache.get(f'pred:{features_hash}')
    if cached:
        return jsonify(json.loads(cached))

    # Make prediction
    prediction = endpoint.predict(instances=[data])

    # Cache result
    cache.setex(f'pred:{features_hash}', 3600, json.dumps(prediction))

    return jsonify(prediction)
```

## A/B Testing

### Traffic Splitting Strategy
```python
def gradual_rollout(endpoint, model_v1_id, model_v2_id):
    """Gradually shift traffic from v1 to v2"""
    rollout_schedule = [
        (95, 5),   # Day 1: 5% v2
        (90, 10),  # Day 2: 10% v2
        (75, 25),  # Day 3: 25% v2
        (50, 50),  # Day 4: 50% v2
        (25, 75),  # Day 5: 75% v2
        (0, 100)   # Day 6: 100% v2
    ]

    for v1_traffic, v2_traffic in rollout_schedule:
        # Update traffic split
        endpoint.update(traffic_split={
            model_v1_id: v1_traffic,
            model_v2_id: v2_traffic
        })

        # Monitor metrics
        metrics = get_model_metrics(endpoint, hours=24)
        if metrics['error_rate'] > 0.05:
            # Rollback if error rate too high
            endpoint.update(traffic_split={
                model_v1_id: 100,
                model_v2_id: 0
            })
            break

        time.sleep(86400)  # Wait 1 day
```

## Best Practices

### Deployment
1. Use managed endpoints for scalability
2. Implement canary deployments
3. Monitor latency and throughput
4. Set up auto-scaling
5. Use accelerators for large models
6. Implement caching for repeated requests
7. Version all models
8. Document deployment procedures

### Monitoring
1. Track prediction drift
2. Monitor model performance metrics
3. Set up alerting for anomalies
4. Log predictions for analysis
5. Monitor infrastructure metrics
6. Implement A/B testing
7. Regular model evaluation
8. Incident response procedures

### MLOps
1. Automate training pipelines
2. Version datasets and models
3. Implement CI/CD for ML
4. Feature store for consistency
5. Metadata tracking
6. Automated testing
7. Continuous monitoring
8. Regular retraining schedules

## Common Scenarios

**Scenario**: High-traffic recommendation system
**Solution**: Vertex AI endpoint with auto-scaling, Redis caching, batch prediction for precomputing, monitoring for drift

**Scenario**: Real-time fraud detection
**Solution**: Low-latency endpoint, model optimization (quantization), feature caching, real-time monitoring, automated retraining

**Scenario**: Gradual model rollout
**Solution**: Traffic splitting, A/B testing, comprehensive monitoring, automated rollback, gradual traffic shift

## Study Tips

1. Practice model deployment to endpoints
2. Configure monitoring and drift detection
3. Build end-to-end ML pipelines
4. Implement A/B testing
5. Optimize models for serving
6. Set up automated retraining
7. Work with batch and online prediction

## Additional Resources

- [Vertex AI Prediction](https://cloud.google.com/vertex-ai/docs/predictions)
- [Model Monitoring](https://cloud.google.com/vertex-ai/docs/model-monitoring)
- [Vertex AI Pipelines](https://cloud.google.com/vertex-ai/docs/pipelines)
- [MLOps Best Practices](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
