# Model Deployment and Monitoring

## Overview
This section covers model registration, deployment strategies, real-time and batch inference, model monitoring, and MLOps practices for production machine learning solutions on Azure.

## Model Registration

### Registering Models

#### From Training Run
```python
from azureml.core import Model, Run

# In training script
run = Run.get_context()

# Save model
import joblib
joblib.dump(model, 'outputs/model.pkl')

# Register model from run
model = run.register_model(
    model_name='my_model',
    model_path='outputs/model.pkl',
    description='Random Forest classifier',
    tags={'framework': 'sklearn', 'task': 'classification'},
    model_framework=Model.Framework.SCIKITLEARN,
    model_framework_version='1.0.2'
)
```

#### From Local File
```python
# Register model from local file
model = Model.register(
    workspace=ws,
    model_name='my_model',
    model_path='./models/model.pkl',
    description='Trained model',
    tags={'version': 'v1', 'type': 'classification'}
)
```

#### From AutoML Run
```python
# Get best model from AutoML
best_run, fitted_model = automl_run.get_output()

# Register best model
model = best_run.register_model(
    model_name='automl_best_model',
    description='Best model from AutoML',
    tags={'training': 'automl'}
)
```

### Model Versioning

```python
# Register new version (same name creates new version)
model_v2 = Model.register(
    workspace=ws,
    model_name='my_model',  # Same name
    model_path='./models/model_v2.pkl',
    description='Improved model',
    tags={'version': 'v2'}
)

# Retrieve specific version
model = Model(ws, name='my_model', version=2)

# Get latest version
model = Model(ws, name='my_model')  # Gets latest by default

# List all versions
models = Model.list(ws, name='my_model')
for m in models:
    print(f"Version: {m.version}, Created: {m.created_time}")
```

### Model Metadata and Tags

```python
# Add/update tags
model.add_tags({'accuracy': '0.95', 'deployed': 'true'})

# Add properties
model.add_properties({'training_data': 'sales_2024'})

# Add description
model.description = "Production model for customer churn prediction"

# View model details
print(f"Name: {model.name}")
print(f"Version: {model.version}")
print(f"ID: {model.id}")
print(f"Created: {model.created_time}")
print(f"Tags: {model.tags}")
print(f"Properties: {model.properties}")
```

## Model Deployment

### Deployment Targets

#### Azure Container Instances (ACI)
- **Use Case:** Development/testing, low-scale deployments
- **Characteristics:**
  - Quick deployment
  - No cluster management
  - Limited scalability
  - Lower cost
  - CPU-only (no GPU)

```python
from azureml.core.webservice import AciWebservice

# ACI deployment configuration
aci_config = AciWebservice.deploy_configuration(
    cpu_cores=1,
    memory_gb=1,
    auth_enabled=True,
    enable_app_insights=True,
    collect_model_data=True
)
```

#### Azure Kubernetes Service (AKS)
- **Use Case:** Production deployments, high-scale scenarios
- **Characteristics:**
  - Auto-scaling
  - Load balancing
  - High availability
  - GPU support
  - Production-grade

```python
from azureml.core.compute import AksCompute
from azureml.core.webservice import AksWebservice

# Create AKS cluster
prov_config = AksCompute.provisioning_configuration(
    agent_count=3,
    vm_size='Standard_DS3_v2',
    cluster_purpose='DevTest'  # or 'FastProd'
)

aks_target = ComputeTarget.create(
    workspace=ws,
    name='my-aks-cluster',
    provisioning_configuration=prov_config
)

# AKS deployment configuration
aks_config = AksWebservice.deploy_configuration(
    autoscale_enabled=True,
    autoscale_min_replicas=1,
    autoscale_max_replicas=10,
    autoscale_target_utilization=70,
    cpu_cores=1,
    memory_gb=2,
    enable_app_insights=True,
    collect_model_data=True
)
```

#### Azure Machine Learning Endpoints
- **Managed Online Endpoints:** Serverless option
- **Batch Endpoints:** For batch inference

### Creating Deployment Assets

#### Entry Script (score.py)
```python
import json
import joblib
import numpy as np
from azureml.core.model import Model

def init():
    """Called when container is initialized"""
    global model

    # Get model path
    model_path = Model.get_model_path('my_model')

    # Load model
    model = joblib.load(model_path)

    print("Model loaded successfully")

def run(raw_data):
    """Called for each inference request"""
    try:
        # Parse input data
        data = json.loads(raw_data)
        input_data = np.array(data['data'])

        # Make prediction
        predictions = model.predict(input_data)

        # Return result
        return json.dumps({
            'predictions': predictions.tolist()
        })

    except Exception as e:
        error = str(e)
        return json.dumps({"error": error})
```

#### Environment Configuration
```python
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies

# Create environment
env = Environment('deployment-env')

# Define dependencies
conda_deps = CondaDependencies.create(
    conda_packages=['scikit-learn', 'numpy'],
    pip_packages=['azureml-defaults', 'inference-schema']
)

env.python.conda_dependencies = conda_deps

# Or use curated environment
env = Environment.get(ws, name='AzureML-sklearn-1.0-ubuntu20.04-py38-cpu')
```

#### Inference Configuration
```python
from azureml.core.model import InferenceConfig

inference_config = InferenceConfig(
    entry_script='score.py',
    source_directory='./deployment',
    environment=env
)
```

### Deploying the Model

```python
from azureml.core.model import Model
from azureml.core.webservice import Webservice

# Get model
model = Model(ws, name='my_model')

# Deploy to ACI (development)
service = Model.deploy(
    workspace=ws,
    name='my-service-aci',
    models=[model],
    inference_config=inference_config,
    deployment_config=aci_config
)

service.wait_for_deployment(show_output=True)

# Deploy to AKS (production)
service = Model.deploy(
    workspace=ws,
    name='my-service-aks',
    models=[model],
    inference_config=inference_config,
    deployment_config=aks_config,
    deployment_target=aks_target
)

service.wait_for_deployment(show_output=True)

# Get service URL and keys
print(f"Service URL: {service.scoring_uri}")
print(f"Service Keys: {service.get_keys()}")
```

### Testing the Deployment

```python
import requests
import json

# Prepare test data
test_data = {
    'data': [[1.0, 2.0, 3.0, 4.0]]
}

# Get service endpoint and keys
scoring_uri = service.scoring_uri
key = service.get_keys()[0]

# Make request
headers = {
    'Content-Type': 'application/json',
    'Authorization': f'Bearer {key}'
}

response = requests.post(
    scoring_uri,
    data=json.dumps(test_data),
    headers=headers
)

print(f"Predictions: {response.json()}")

# Or use SDK
result = service.run(json.dumps(test_data))
print(f"Result: {result}")
```

## Batch Inference

### Batch Deployment with ParallelRunStep

```python
from azureml.pipeline.steps import ParallelRunConfig, ParallelRunStep
from azureml.pipeline.core import Pipeline

# Parallel run configuration
parallel_run_config = ParallelRunConfig(
    source_directory='./scripts',
    entry_script='batch_score.py',
    mini_batch_size='5',  # Number of files per batch
    error_threshold=10,
    output_action='append_row',
    environment=env,
    compute_target=compute_cluster,
    node_count=2,
    process_count_per_node=2
)

# Create parallel run step
parallel_step = ParallelRunStep(
    name='batch-scoring',
    parallel_run_config=parallel_run_config,
    inputs=[input_data],
    output=output_data,
    allow_reuse=False
)

# Create and submit pipeline
pipeline = Pipeline(workspace=ws, steps=[parallel_step])
pipeline_run = experiment.submit(pipeline)
```

#### Batch Scoring Script (batch_score.py)
```python
import os
import numpy as np
import joblib
from azureml.core import Model

def init():
    """Initialize batch scoring"""
    global model

    model_path = Model.get_model_path('my_model')
    model = joblib.load(model_path)

def run(mini_batch):
    """Score each mini-batch"""
    results = []

    for file_path in mini_batch:
        # Load data from file
        data = np.load(file_path)

        # Make predictions
        predictions = model.predict(data)

        # Store results
        results.append({
            'file': os.path.basename(file_path),
            'predictions': predictions.tolist()
        })

    return results
```

## Model Monitoring

### Application Insights

#### Enable Application Insights
```python
# Enable on existing service
service.update(enable_app_insights=True)

# View logs
logs = service.get_logs()
print(logs)

# Custom logging in score.py
import logging

def run(raw_data):
    logging.info(f"Received request: {len(raw_data)} bytes")
    # ... scoring code ...
    logging.info(f"Prediction completed")
    return result
```

#### Query Application Insights
```python
from azureml.core import Workspace
from azureml.core.webservice import Webservice
from applicationinsights import TelemetryClient

# Get Application Insights key
service = Webservice(ws, 'my-service')
app_insights_key = service.get_app_insights_key()

# Query metrics
# (Use Azure Portal or Application Insights API)
```

### Data Drift Monitoring

#### Configure Data Drift Monitor
```python
from azureml.datadrift import DataDriftDetector

# Create data drift monitor
monitor = DataDriftDetector.create_from_datasets(
    workspace=ws,
    name='my-drift-monitor',
    baseline_data_set=baseline_dataset,
    target_data_set=target_dataset,
    compute_target='my-cluster',
    frequency='Week',
    feature_list=['feature1', 'feature2', 'feature3'],
    latency=24  # Hours of latency to allow
)

# Run drift detection
drift_run = monitor.run(
    start_date='2024-01-01',
    end_date='2024-03-31',
    feature_list=None  # Use all features
)

# Get drift results
drift_metrics = drift_run.get_metrics()
print(f"Drift coefficient: {drift_metrics['drift_coefficient']}")

# Check for alerts
if drift_metrics['drift_coefficient'] > 0.3:
    print("WARNING: Significant data drift detected!")
```

#### Enable Data Collection
```python
# Enable model data collection
aks_config = AksWebservice.deploy_configuration(
    collect_model_data=True,
    enable_app_insights=True
)

# Access collected data
from azureml.monitoring import ModelDataCollector

# Data is automatically collected to workspace blob storage
# Access via:
# - Azure Portal > Storage Account > model-data-collection container
# - Programmatically via blob storage SDK
```

### Performance Monitoring

```python
# Monitor service performance
metrics = service.get_metrics()

# Key metrics to monitor:
# - Request count
# - Request duration
# - Failed requests
# - CPU/Memory usage
# - Model data drift

# Set up alerts (via Azure Monitor)
from azure.mgmt.monitor import MonitorManagementClient
from azure.mgmt.monitor.models import *

# Create alert rule for high latency
alert_rule = AlertRuleResource(
    location='global',
    enabled=True,
    condition=ThresholdRuleCondition(
        data_source=RuleMetricDataSource(
            resource_uri=service.resource_id,
            metric_name='RequestDuration'
        ),
        operator='GreaterThan',
        threshold=1000,  # milliseconds
        window_size='PT5M'
    ),
    actions=[
        RuleEmailAction(
            send_to_service_owners=True
        )
    ]
)
```

## MLOps Practices

### CI/CD with Azure Pipelines

#### Azure DevOps Pipeline (azure-pipelines.yml)
```yaml
trigger:
  branches:
    include:
    - main

variables:
  azureml.resourceGroup: 'ml-rg'
  azureml.workspace: 'ml-workspace'

stages:
- stage: Train
  jobs:
  - job: TrainModel
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.8'

    - script: |
        pip install azureml-sdk
      displayName: 'Install dependencies'

    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Azure-Service-Connection'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          python train.py
      displayName: 'Train model'

- stage: Deploy
  dependsOn: Train
  condition: succeeded()
  jobs:
  - job: DeployModel
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Azure-Service-Connection'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          python deploy.py
      displayName: 'Deploy model'
```

### Model Retraining Pipeline

```python
from azureml.pipeline.core import Pipeline, Schedule
from azureml.pipeline.steps import PythonScriptStep

# Data preparation step
prep_step = PythonScriptStep(
    name='prepare_data',
    script_name='prepare.py',
    compute_target=compute_cluster,
    source_directory='./src'
)

# Training step
train_step = PythonScriptStep(
    name='train_model',
    script_name='train.py',
    compute_target=compute_cluster,
    source_directory='./src'
)

# Evaluation step
eval_step = PythonScriptStep(
    name='evaluate_model',
    script_name='evaluate.py',
    compute_target=compute_cluster,
    source_directory='./src'
)

# Register step
register_step = PythonScriptStep(
    name='register_model',
    script_name='register.py',
    compute_target=compute_cluster,
    source_directory='./src'
)

# Create pipeline
pipeline = Pipeline(
    workspace=ws,
    steps=[prep_step, train_step, eval_step, register_step]
)

# Publish pipeline
published_pipeline = pipeline.publish(
    name='retraining_pipeline',
    description='Automated model retraining'
)

# Schedule pipeline (weekly retraining)
schedule = Schedule.create(
    workspace=ws,
    name='weekly_retraining',
    pipeline_id=published_pipeline.id,
    experiment_name='scheduled_retraining',
    recurrence=ScheduleRecurrence(
        frequency='Week',
        interval=1,
        start_time='2024-01-01T00:00:00'
    )
)
```

### Model Versioning and Rollback

```python
# Deploy new model version
new_model = Model(ws, name='my_model', version=3)
service.update(models=[new_model])

# Rollback to previous version if issues detected
if performance_degraded():
    old_model = Model(ws, name='my_model', version=2)
    service.update(models=[old_model])
    print("Rolled back to previous model version")

# Blue-Green deployment
# Deploy new version to separate service
service_green = Model.deploy(
    workspace=ws,
    name='my-service-green',
    models=[new_model],
    inference_config=inference_config,
    deployment_config=aks_config
)

# Test green deployment
if validate_service(service_green):
    # Switch traffic to green
    service.delete()
    service_green.update(name='my-service')
else:
    # Keep blue, delete green
    service_green.delete()
```

## Best Practices

### Model Registration
1. **Use meaningful names and versions** for models
2. **Add comprehensive tags** (framework, metrics, training date)
3. **Include model metadata** (training data, parameters)
4. **Register all artifacts** (model, scaler, encoder)
5. **Document model lineage** (data, code, environment)

### Deployment
1. **Test on ACI first** before AKS deployment
2. **Use environment files** for reproducibility
3. **Enable Application Insights** for all deployments
4. **Implement health checks** in scoring script
5. **Use authentication** for production endpoints
6. **Configure auto-scaling** based on load
7. **Set up monitoring and alerts** from day one

### Monitoring
1. **Monitor both model and infrastructure** metrics
2. **Set up data drift detection** for production models
3. **Track model performance** over time
4. **Establish alert thresholds** for key metrics
5. **Review logs regularly** for errors and anomalies
6. **Implement automated retraining** when drift detected

### MLOps
1. **Automate training pipelines** for reproducibility
2. **Version control everything** (code, data, models)
3. **Implement CI/CD pipelines** for deployment
4. **Use separate environments** (dev, staging, prod)
5. **Implement gradual rollout** strategies
6. **Plan for rollback** scenarios
7. **Document deployment processes**

### Security
1. **Use managed identities** for authentication
2. **Implement RBAC** for workspace access
3. **Enable VNet integration** for private deployments
4. **Encrypt data** in transit and at rest
5. **Regularly audit** access and operations
6. **Use Key Vault** for secrets management

## Study Tips

### Key Concepts to Master
- Model registration and versioning
- Deployment targets (ACI vs AKS vs Endpoints)
- Entry script structure (init and run functions)
- Inference configuration components
- Batch inference with ParallelRunStep
- Application Insights for monitoring
- Data drift detection and monitoring
- MLOps pipeline automation

### Common Scenarios
1. **Quick testing** → Deploy to ACI
2. **Production deployment** → Deploy to AKS with autoscaling
3. **Large-scale batch scoring** → ParallelRunStep pipeline
4. **Model performance degradation** → Data drift monitoring
5. **Automated retraining** → Scheduled pipeline with drift trigger
6. **Rollback needed** → Deploy previous model version

### Exam Focus Areas
- Model registration process and versioning
- Deployment configuration for different targets
- Entry script requirements (init and run functions)
- Monitoring with Application Insights
- Data collection for drift monitoring
- Batch inference configuration
- Basic MLOps concepts
- Updating and managing deployed services

### Practice Tasks
1. Register a model with tags and metadata
2. Create entry script with init and run functions
3. Deploy model to ACI and test endpoint
4. Configure AKS deployment with autoscaling
5. Set up batch inference pipeline
6. Enable and query Application Insights
7. Configure data drift monitor
8. Update deployed service with new model version

### Remember
- ACI = Development/Testing, AKS = Production
- Entry script needs init() and run() functions
- Enable Application Insights for production monitoring
- Data drift indicates model may need retraining
- Model versioning enables easy rollback
- Batch inference uses ParallelRunStep for scale
- MLOps automates training, deployment, and monitoring
