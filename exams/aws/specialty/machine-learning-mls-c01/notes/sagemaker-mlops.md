# SageMaker Platform and MLOps

## SageMaker Studio

### Studio Components

**SageMaker Studio IDE**
- Integrated Jupyter environment
- Visual interface for ML workflow
- Git integration
- Experiment tracking
- Model registry
- Pipeline visualization

**Studio Notebooks**
- Fully managed Jupyter notebooks
- Fast notebook startup (under 2 minutes)
- Elastic compute (change instance types)
- Kernel persistence
- Sharing and collaboration

**SageMaker Projects**
- MLOps templates
- CI/CD integration
- Model deployment pipelines
- Infrastructure as code
- Version control integration

```python
import sagemaker
from sagemaker import get_execution_role

role = get_execution_role()
sess = sagemaker.Session()

# Initialize SageMaker client
sm_client = boto3.client('sagemaker')

# Create Studio domain (for account setup)
sm_client.create_domain(
    DomainName='ml-platform',
    AuthMode='IAM',
    DefaultUserSettings={
        'ExecutionRole': role,
        'SecurityGroups': ['sg-xxxxx'],
        'SharingSettings': {
            'NotebookOutputOption': 'Allowed',
            'S3OutputPath': 's3://bucket/studio-output/'
        }
    },
    SubnetIds=['subnet-xxxxx'],
    VpcId='vpc-xxxxx'
)
```

## SageMaker Training

### Training Jobs

**Training Script (train.py)**
```python
import argparse
import os
import json
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import joblib

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    
    # Hyperparameters
    parser.add_argument('--n-estimators', type=int, default=100)
    parser.add_argument('--max-depth', type=int, default=10)
    parser.add_argument('--min-samples-split', type=int, default=2)
    
    # SageMaker specific arguments
    parser.add_argument('--model-dir', type=str, default=os.environ.get('SM_MODEL_DIR'))
    parser.add_argument('--train', type=str, default=os.environ.get('SM_CHANNEL_TRAIN'))
    parser.add_argument('--validation', type=str, default=os.environ.get('SM_CHANNEL_VALIDATION'))
    
    args = parser.parse_args()
    
    # Load data
    train_df = pd.read_csv(os.path.join(args.train, 'train.csv'))
    X_train = train_df.drop('target', axis=1)
    y_train = train_df['target']
    
    val_df = pd.read_csv(os.path.join(args.validation, 'validation.csv'))
    X_val = val_df.drop('target', axis=1)
    y_val = val_df['target']
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=args.n_estimators,
        max_depth=args.max_depth,
        min_samples_split=args.min_samples_split,
        random_state=42
    )
    model.fit(X_train, y_train)
    
    # Evaluate
    train_acc = accuracy_score(y_train, model.predict(X_train))
    val_acc = accuracy_score(y_val, model.predict(X_val))
    
    print(f'Train Accuracy: {train_acc:.4f}')
    print(f'Validation Accuracy: {val_acc:.4f}')
    
    # Save model
    joblib.dump(model, os.path.join(args.model_dir, 'model.joblib'))
```

**Submit Training Job**
```python
from sagemaker.sklearn import SKLearn

sklearn_estimator = SKLearn(
    entry_point='train.py',
    role=role,
    instance_type='ml.m5.xlarge',
    instance_count=1,
    framework_version='1.0-1',
    py_version='py3',
    hyperparameters={
        'n-estimators': 200,
        'max-depth': 15,
        'min-samples-split': 4
    },
    output_path='s3://bucket/model-output',
    code_location='s3://bucket/code',
    enable_sagemaker_metrics=True,
    metric_definitions=[
        {'Name': 'train:accuracy', 'Regex': 'Train Accuracy: ([0-9\\.]+)'},
        {'Name': 'validation:accuracy', 'Regex': 'Validation Accuracy: ([0-9\\.]+)'}
    ]
)

sklearn_estimator.fit({
    'train': 's3://bucket/train',
    'validation': 's3://bucket/validation'
})
```

### Distributed Training

**Data Parallel Training**
```python
from sagemaker.pytorch import PyTorch

estimator = PyTorch(
    entry_point='train_distributed.py',
    role=role,
    framework_version='1.12',
    py_version='py38',
    instance_count=4,
    instance_type='ml.p3.8xlarge',
    distribution={
        'smdistributed': {
            'dataparallel': {
                'enabled': True,
                'custom_mpi_options': '-verbose -x NCCL_DEBUG=INFO'
            }
        }
    }
)
```

**Model Parallel Training**
```python
estimator = PyTorch(
    entry_point='train_model_parallel.py',
    role=role,
    framework_version='1.12',
    py_version='py38',
    instance_count=2,
    instance_type='ml.p3.16xlarge',
    distribution={
        'smdistributed': {
            'modelparallel': {
                'enabled': True,
                'parameters': {
                    'partitions': 2,
                    'pipeline_parallel_degree': 2,
                    'ddp': True
                }
            }
        }
    }
)
```

### Managed Spot Training

**Cost Savings with Spot Instances**
```python
from sagemaker.estimator import Estimator

estimator = Estimator(
    image_uri='training-image:latest',
    role=role,
    instance_count=10,
    instance_type='ml.p3.2xlarge',
    use_spot_instances=True,
    max_run=86400,  # 24 hours
    max_wait=90000,  # Max wait time including spot delays
    checkpoint_s3_uri='s3://bucket/checkpoints/',
    checkpoint_local_path='/opt/ml/checkpoints'
)
```

## SageMaker Processing

### Processing Jobs

**Data Processing Script**
```python
# preprocessing.py
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler, LabelEncoder
import argparse
import os

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--input-path', type=str)
    parser.add_argument('--output-path', type=str)
    args = parser.parse_args()
    
    # Read data
    df = pd.read_csv(f'{args.input_path}/data.csv')
    
    # Handle missing values
    df.fillna(df.median(), inplace=True)
    
    # Encode categorical variables
    le = LabelEncoder()
    for col in df.select_dtypes(include='object').columns:
        df[col] = le.fit_transform(df[col])
    
    # Scale features
    scaler = StandardScaler()
    numeric_cols = df.select_dtypes(include=np.number).columns
    df[numeric_cols] = scaler.fit_transform(df[numeric_cols])
    
    # Split and save
    train = df.sample(frac=0.8, random_state=42)
    val = df.drop(train.index)
    
    train.to_csv(f'{args.output_path}/train/train.csv', index=False)
    val.to_csv(f'{args.output_path}/validation/validation.csv', index=False)
```

**Run Processing Job**
```python
from sagemaker.processing import ScriptProcessor, ProcessingInput, ProcessingOutput

processor = ScriptProcessor(
    role=role,
    image_uri='sklearn-image:latest',
    instance_type='ml.m5.xlarge',
    instance_count=1,
    command=['python3']
)

processor.run(
    code='preprocessing.py',
    inputs=[
        ProcessingInput(
            source='s3://bucket/raw-data/',
            destination='/opt/ml/processing/input'
        )
    ],
    outputs=[
        ProcessingOutput(
            source='/opt/ml/processing/output/train',
            destination='s3://bucket/processed/train'
        ),
        ProcessingOutput(
            source='/opt/ml/processing/output/validation',
            destination='s3://bucket/processed/validation'
        )
    ],
    arguments=['--input-path', '/opt/ml/processing/input',
               '--output-path', '/opt/ml/processing/output']
)
```

## Hyperparameter Tuning

### Automatic Model Tuning

**Tuning Job Configuration**
```python
from sagemaker.tuner import HyperparameterTuner, IntegerParameter, ContinuousParameter, CategoricalParameter

hyperparameter_ranges = {
    'n-estimators': IntegerParameter(50, 500),
    'max-depth': IntegerParameter(5, 30),
    'min-samples-split': IntegerParameter(2, 20),
    'min-samples-leaf': IntegerParameter(1, 10),
    'learning-rate': ContinuousParameter(0.001, 0.3, scaling_type='Logarithmic')
}

objective_metric_name = 'validation:accuracy'
metric_definitions = [
    {'Name': 'validation:accuracy', 'Regex': 'Validation Accuracy: ([0-9\\.]+)'}
]

tuner = HyperparameterTuner(
    estimator=sklearn_estimator,
    objective_metric_name=objective_metric_name,
    hyperparameter_ranges=hyperparameter_ranges,
    metric_definitions=metric_definitions,
    max_jobs=50,
    max_parallel_jobs=10,
    strategy='Bayesian',
    objective_type='Maximize',
    early_stopping_type='Auto'
)

tuner.fit({'train': train_data, 'validation': val_data})

# Get best training job
best_training_job = tuner.best_training_job()
best_hyperparameters = tuner.best_estimator().hyperparameters()
```

## SageMaker Pipelines

### ML Pipeline Definition

**Complete Pipeline Example**
```python
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import ProcessingStep, TrainingStep, CreateModelStep
from sagemaker.workflow.parameters import ParameterString, ParameterInteger
from sagemaker.workflow.step_collections import RegisterModel
from sagemaker.workflow.conditions import ConditionGreaterThanOrEqualTo
from sagemaker.workflow.condition_step import ConditionStep
from sagemaker.workflow.functions import JsonGet

# Parameters
input_data = ParameterString(name="InputData", default_value="s3://bucket/data")
instance_type = ParameterString(name="TrainingInstanceType", default_value="ml.m5.xlarge")
model_approval_status = ParameterString(name="ModelApprovalStatus", default_value="PendingManualApproval")

# Processing step
processor = SKLearnProcessor(
    framework_version='0.23-1',
    role=role,
    instance_type='ml.m5.xlarge',
    instance_count=1
)

processing_step = ProcessingStep(
    name="PreprocessData",
    processor=processor,
    inputs=[ProcessingInput(source=input_data, destination="/opt/ml/processing/input")],
    outputs=[
        ProcessingOutput(output_name="train", source="/opt/ml/processing/train"),
        ProcessingOutput(output_name="validation", source="/opt/ml/processing/validation"),
        ProcessingOutput(output_name="test", source="/opt/ml/processing/test")
    ],
    code="preprocessing.py"
)

# Training step
estimator = SKLearn(
    entry_point='train.py',
    role=role,
    instance_type=instance_type,
    instance_count=1,
    framework_version='1.0-1'
)

training_step = TrainingStep(
    name="TrainModel",
    estimator=estimator,
    inputs={
        "train": TrainingInput(
            s3_data=processing_step.properties.ProcessingOutputConfig.Outputs["train"].S3Output.S3Uri
        ),
        "validation": TrainingInput(
            s3_data=processing_step.properties.ProcessingOutputConfig.Outputs["validation"].S3Output.S3Uri
        )
    }
)

# Evaluation step
evaluation_processor = ScriptProcessor(
    role=role,
    image_uri='evaluation-image:latest',
    instance_type='ml.m5.xlarge',
    instance_count=1,
    command=['python3']
)

evaluation_step = ProcessingStep(
    name="EvaluateModel",
    processor=evaluation_processor,
    inputs=[
        ProcessingInput(
            source=training_step.properties.ModelArtifacts.S3ModelArtifacts,
            destination="/opt/ml/processing/model"
        ),
        ProcessingInput(
            source=processing_step.properties.ProcessingOutputConfig.Outputs["test"].S3Output.S3Uri,
            destination="/opt/ml/processing/test"
        )
    ],
    outputs=[
        ProcessingOutput(output_name="evaluation", source="/opt/ml/processing/evaluation")
    ],
    code="evaluation.py"
)

# Model registration
model_metrics = ModelMetrics(
    model_statistics=MetricsSource(
        s3_uri=Join(on='/', values=[
            evaluation_step.properties.ProcessingOutputConfig.Outputs["evaluation"].S3Output.S3Uri,
            "evaluation.json"
        ]),
        content_type="application/json"
    )
)

register_step = RegisterModel(
    name="RegisterModel",
    estimator=estimator,
    model_data=training_step.properties.ModelArtifacts.S3ModelArtifacts,
    content_types=["text/csv"],
    response_types=["text/csv"],
    inference_instances=["ml.m5.xlarge"],
    transform_instances=["ml.m5.xlarge"],
    model_package_group_name="MyModelPackageGroup",
    approval_status=model_approval_status,
    model_metrics=model_metrics
)

# Condition to check model accuracy
condition_accuracy = ConditionGreaterThanOrEqualTo(
    left=JsonGet(
        step_name=evaluation_step.name,
        property_file="evaluation",
        json_path="metrics.accuracy.value"
    ),
    right=0.85
)

condition_step = ConditionStep(
    name="CheckAccuracy",
    conditions=[condition_accuracy],
    if_steps=[register_step],
    else_steps=[]
)

# Create pipeline
pipeline = Pipeline(
    name="MLPipeline",
    parameters=[input_data, instance_type, model_approval_status],
    steps=[processing_step, training_step, evaluation_step, condition_step]
)

pipeline.upsert(role_arn=role)

# Execute pipeline
execution = pipeline.start()
```

## Model Deployment

### Real-Time Endpoints

**Deploy Model**
```python
# Deploy from training job
predictor = estimator.deploy(
    initial_instance_count=2,
    instance_type='ml.m5.xlarge',
    endpoint_name='my-model-endpoint',
    data_capture_config=DataCaptureConfig(
        enable_capture=True,
        sampling_percentage=100,
        destination_s3_uri='s3://bucket/data-capture'
    )
)

# Make predictions
result = predictor.predict(test_data)
```

**Auto Scaling**
```python
import boto3

asg_client = boto3.client('application-autoscaling')

# Register scalable target
asg_client.register_scalable_target(
    ServiceNamespace='sagemaker',
    ResourceId=f'endpoint/{endpoint_name}/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    MinCapacity=1,
    MaxCapacity=10
)

# Create scaling policy
asg_client.put_scaling_policy(
    PolicyName='TargetTrackingScaling',
    ServiceNamespace='sagemaker',
    ResourceId=f'endpoint/{endpoint_name}/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    PolicyType='TargetTrackingScaling',
    TargetTrackingScalingPolicyConfiguration={
        'TargetValue': 70.0,
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'SageMakerVariantInvocationsPerInstance'
        },
        'ScaleInCooldown': 300,
        'ScaleOutCooldown': 60
    }
)
```

### Multi-Model Endpoints

**Deploy Multiple Models**
```python
from sagemaker.multidatamodel import MultiDataModel

multi_model = MultiDataModel(
    name='multi-model-endpoint',
    model_data_prefix='s3://bucket/models/',
    model=sklearn_estimator.create_model(),
    sagemaker_session=sagemaker.Session()
)

predictor = multi_model.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.xlarge',
    endpoint_name='multi-model-endpoint'
)

# Predict with specific model
predictions = predictor.predict(data, target_model='model1.tar.gz')
```

### Batch Transform

**Large-Scale Batch Predictions**
```python
transformer = estimator.transformer(
    instance_count=10,
    instance_type='ml.m5.xlarge',
    strategy='MultiRecord',
    max_payload=6,
    max_concurrent_transforms=10,
    output_path='s3://bucket/batch-predictions',
    assemble_with='Line',
    accept='text/csv'
)

transformer.transform(
    data='s3://bucket/batch-input',
    content_type='text/csv',
    split_type='Line',
    join_source='Input'
)

transformer.wait()
```

## Model Monitoring

### SageMaker Model Monitor

**Data Capture**
```python
from sagemaker.model_monitor import DataCaptureConfig

data_capture_config = DataCaptureConfig(
    enable_capture=True,
    sampling_percentage=100,
    destination_s3_uri='s3://bucket/data-capture',
    capture_options=['Input', 'Output']
)
```

**Create Baseline**
```python
from sagemaker.model_monitor import DefaultModelMonitor

monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    volume_size_in_gb=20,
    max_runtime_in_seconds=3600
)

baseline_job = monitor.suggest_baseline(
    baseline_dataset='s3://bucket/validation.csv',
    dataset_format={'csv': {'header': True}},
    output_s3_uri='s3://bucket/baseline',
    wait=True
)
```

**Schedule Monitoring**
```python
from sagemaker.model_monitor import CronExpressionGenerator

monitor.create_monitoring_schedule(
    monitor_schedule_name='hourly-monitoring',
    endpoint_input=predictor.endpoint_name,
    output_s3_uri='s3://bucket/monitoring-results',
    statistics=monitor.baseline_statistics(),
    constraints=monitor.suggested_constraints(),
    schedule_cron_expression=CronExpressionGenerator.hourly(),
    enable_cloudwatch_metrics=True
)
```

## MLOps Best Practices

### Version Control
- Git for code and notebooks
- Model Registry for model versions
- Data versioning with timestamps or hashes
- Pipeline versioning

### CI/CD Integration
```yaml
# .gitlab-ci.yml
stages:
  - test
  - train
  - deploy

test:
  stage: test
  script:
    - pytest tests/

train:
  stage: train
  script:
    - python trigger_pipeline.py
  only:
    - main

deploy:
  stage: deploy
  script:
    - python deploy_model.py
  when: manual
```

### Monitoring and Alerts
- Model performance metrics
- Data drift detection
- Infrastructure metrics
- Cost tracking
- CloudWatch alarms

## Exam Tips

### SageMaker Components
- Studio: IDE for ML
- Processing: Data preprocessing
- Training: Model training
- Tuning: Hyperparameter optimization
- Pipelines: ML workflows
- Model Registry: Version control
- Endpoints: Model serving

### Deployment Options
- Real-time: Low latency predictions
- Batch: Large-scale offline predictions
- Multi-model: Multiple models on one endpoint
- Async: Long-running inference

### Cost Optimization
- Spot instances for training
- Auto-scaling for endpoints
- Right-size instances
- Multi-model endpoints
- Batch transform for batch workloads

### Best Practices
- Automate with pipelines
- Monitor model performance
- Version everything
- Use managed services
- Enable data capture
- Implement CI/CD
- Regular retraining
