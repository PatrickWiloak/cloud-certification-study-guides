# Vertex AI Platform Fundamentals

## What is Vertex AI?

Vertex AI is Google Cloud's unified machine learning platform that combines AutoML and AI Platform into a unified API, client library, and user interface, providing tools for every step of the ML workflow.

### Key Benefits
- **Unified platform**: One place for all ML workflows
- **MLOps integration**: Built-in experiment tracking and model management
- **Scalable infrastructure**: Automatic scaling and optimization
- **Pre-built solutions**: AutoML and pre-trained models
- **Custom development**: Full flexibility for custom models

## Vertex AI Components

### Vertex AI Workbench
**Jupyter-based development environment**

#### Features
- **Managed notebooks**: Fully managed Jupyter instances
- **User-managed notebooks**: More control over environment
- **Built-in libraries**: Pre-installed ML frameworks
- **Data science tools**: Integrated development experience
- **Version control**: Git integration

#### Instance Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Managed** | Google-managed infrastructure | Quick start, experimentation |
| **User-managed** | Custom VM configuration | Production workloads, specific requirements |

### Vertex AI Training
**Scalable model training service**

#### Training Options
- **Custom training**: Your own training code
- **AutoML training**: Automated model development
- **Hyperparameter tuning**: Automated hyperparameter optimization
- **Distributed training**: Multi-GPU and multi-node training

#### Custom Training Example
```python
from google.cloud import aiplatform

# Initialize Vertex AI
aiplatform.init(project='my-project', location='us-central1')

# Create custom training job
job = aiplatform.CustomTrainingJob(
    display_name='my-training-job',
    script_path='train.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-gpu.1-9:latest',
    requirements=['scikit-learn', 'pandas'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/pytorch-gpu.1-9:latest'
)

# Run training
model = job.run(
    dataset=my_dataset,
    replica_count=1,
    machine_type='n1-standard-4',
    accelerator_type='NVIDIA_TESLA_K80',
    accelerator_count=1
)
```

### Vertex AI Endpoints
**Model serving infrastructure**

#### Deployment Options
- **Online prediction**: Real-time inference
- **Batch prediction**: Offline batch processing
- **Private endpoints**: VPC-native serving
- **Multi-model endpoints**: Host multiple models

```python
# Deploy model to endpoint
endpoint = model.deploy(
    machine_type='n1-standard-2',
    min_replica_count=1,
    max_replica_count=10,
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1
)

# Make predictions
predictions = endpoint.predict(instances=test_data)
```

### Vertex AI Pipelines
**ML workflow orchestration**

#### Pipeline Components
- **Components**: Reusable pipeline steps
- **Experiments**: Track and compare pipeline runs
- **Artifacts**: Track inputs, outputs, and metadata
- **Lineage**: Data and model lineage tracking

```python
from kfp.v2 import dsl
from kfp.v2.dsl import component, pipeline, Input, Output, Dataset, Model

@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'scikit-learn']
)
def preprocess_data(
    input_data: Input[Dataset],
    output_data: Output[Dataset]
):
    import pandas as pd
    
    # Load and preprocess data
    df = pd.read_csv(input_data.path)
    # ... preprocessing logic
    df.to_csv(output_data.path, index=False)

@component(
    base_image='python:3.9',
    packages_to_install=['scikit-learn']
)
def train_model(
    training_data: Input[Dataset],
    model: Output[Model]
):
    import pickle
    from sklearn.ensemble import RandomForestClassifier
    
    # Train model
    # ... training logic
    
    # Save model
    with open(model.path, 'wb') as f:
        pickle.dump(trained_model, f)

@pipeline(name='ml-pipeline')
def ml_pipeline():
    preprocess_task = preprocess_data(input_data=dataset)
    train_task = train_model(training_data=preprocess_task.outputs['output_data'])
```

### Vertex AI Model Registry
**Model versioning and management**

#### Features
- **Model versioning**: Track model versions
- **Model lineage**: Understand model ancestry
- **Model evaluation**: Compare model performance
- **Model aliases**: Stable references to models

```python
# Register model
model = aiplatform.Model.upload(
    display_name='my-model',
    artifact_uri='gs://my-bucket/model/',
    serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/sklearn-cpu.0-24:latest',
    description='My custom model'
)

# Create model version
model_version = model.upload(
    parent_model=model,
    version_description='Updated model with new features'
)
```

## Generative AI on Vertex AI

### Generative AI Studio
**Experiment with generative AI models**

#### Capabilities
- **Text generation**: Create and edit text content
- **Chat**: Conversational AI applications
- **Code generation**: Programming assistance
- **Multimodal**: Text and image understanding

### Foundation Models

#### PaLM 2 Models
| Model | Description | Use Cases |
|-------|-------------|-----------|
| **Text Bison** | Large language model for text | Content generation, summarization |
| **Chat Bison** | Conversational AI model | Chatbots, Q&A systems |
| **Code Bison** | Code generation model | Programming assistance, code completion |

#### Gemini Models
| Model | Capabilities | Use Cases |
|-------|-------------|-----------|
| **Gemini Pro** | Multimodal understanding | Text and image analysis |
| **Gemini Pro Vision** | Vision capabilities | Image understanding, OCR |

### Model Garden
**Pre-trained and foundation models**

#### Available Models
- **Google models**: PaLM, Gemini, T5, BERT
- **Third-party models**: Llama 2, Claude, Stable Diffusion
- **Task-specific models**: Specialized for specific domains
- **Open source models**: Community-contributed models

### Generative AI API Usage
```python
from vertexai.language_models import TextGenerationModel
from vertexai.language_models import ChatModel
from vertexai.vision_models import ImageGenerationModel

# Text generation
text_model = TextGenerationModel.from_pretrained("text-bison")
response = text_model.predict(
    prompt="Write a product description for a smart watch",
    temperature=0.7,
    max_output_tokens=256,
    top_k=40,
    top_p=0.8
)

# Chat
chat_model = ChatModel.from_pretrained("chat-bison")
chat = chat_model.start_chat()
response = chat.send_message("What are the benefits of renewable energy?")

# Code generation
code_model = TextGenerationModel.from_pretrained("code-bison")
response = code_model.predict(
    prompt="Write a Python function to calculate fibonacci numbers",
    temperature=0.2,
    max_output_tokens=512
)
```

## AutoML Services

### AutoML Tables
**Automated machine learning for structured data**

#### Capabilities
- **Classification**: Predict categories
- **Regression**: Predict numerical values
- **Feature engineering**: Automatic feature selection
- **Model selection**: Algorithm selection and tuning

```python
# Create AutoML tabular dataset
dataset = aiplatform.TabularDataset.create(
    display_name='my-tabular-dataset',
    gcs_source='gs://my-bucket/data.csv'
)

# Train AutoML model
job = aiplatform.AutoMLTabularTrainingJob(
    display_name='automl-tabular-training',
    optimization_prediction_type='classification',
    optimization_objective='minimize-log-loss'
)

model = job.run(
    dataset=dataset,
    target_column='target',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

### AutoML Vision
**Automated computer vision models**

#### Model Types
- **Image classification**: Categorize images
- **Object detection**: Detect and locate objects
- **Image segmentation**: Pixel-level image understanding

```python
# Create image dataset
dataset = aiplatform.ImageDataset.create(
    display_name='my-image-dataset',
    gcs_source='gs://my-bucket/images.csv'
)

# Train AutoML image classification model
job = aiplatform.AutoMLImageTrainingJob(
    display_name='automl-image-training',
    prediction_type='classification'
)

model = job.run(
    dataset=dataset,
    model_display_name='my-image-model',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

### AutoML Natural Language
**Automated NLP models**

#### Capabilities
- **Text classification**: Categorize text documents
- **Entity extraction**: Identify and extract entities
- **Sentiment analysis**: Analyze text sentiment

```python
# Create text dataset
dataset = aiplatform.TextDataset.create(
    display_name='my-text-dataset',
    gcs_source='gs://my-bucket/text-data.jsonl'
)

# Train AutoML text classification model
job = aiplatform.AutoMLTextTrainingJob(
    display_name='automl-text-training',
    prediction_type='classification'
)

model = job.run(
    dataset=dataset,
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

## MLOps with Vertex AI

### Experiment Tracking
```python
from google.cloud import aiplatform

# Initialize experiment
aiplatform.init(experiment='my-experiment')

# Start experiment run
with aiplatform.start_run('run-1') as run:
    # Log parameters
    run.log_params({
        'learning_rate': 0.01,
        'batch_size': 32,
        'epochs': 100
    })
    
    # Log metrics
    run.log_metrics({
        'accuracy': 0.95,
        'loss': 0.05
    })
    
    # Log artifacts
    run.log_artifact('model.pkl')
```

### Model Monitoring
```python
# Enable model monitoring
monitoring_job = aiplatform.ModelDeploymentMonitoringJob.create(
    display_name='model-monitoring',
    endpoint=endpoint,
    prediction_sampling_rate=0.1,
    monitoring_alert_config=aiplatform.monitoring.EmailAlertConfig(
        user_emails=['admin@company.com']
    ),
    monitoring_objective_configs=[
        aiplatform.monitoring.ObjectiveConfig(
            feature_name='feature1',
            feature_drift_detection_config=aiplatform.monitoring.DriftDetectionConfig(
                drift_threshold=0.05
            )
        )
    ]
)
```

### Feature Store
**Centralized feature management**

```python
# Create feature store
feature_store = aiplatform.Featurestore.create(
    featurestore_id='my-feature-store',
    location='us-central1'
)

# Create entity type
entity_type = feature_store.create_entity_type(
    entity_type_id='user',
    description='User entities'
)

# Create features
feature = entity_type.create_feature(
    feature_id='age',
    value_type='INT64',
    description='User age'
)

# Ingest feature values
entity_type.ingest_from_gcs(
    feature_ids=['age', 'income'],
    gcs_source_uris=['gs://my-bucket/features.csv'],
    entity_id_field='user_id',
    feature_time_field='timestamp'
)
```

## Data Integration

### BigQuery Integration
```python
# Train model with BigQuery data
dataset = aiplatform.TabularDataset.create_from_bigquery(
    display_name='bq-dataset',
    bq_source='bq://my-project.my_dataset.my_table'
)

# Export predictions to BigQuery
batch_job = model.batch_predict(
    job_display_name='batch-prediction',
    bigquery_source='bq://my-project.my_dataset.input_table',
    bigquery_destination_prefix='my-project.my_dataset.predictions'
)
```

### Data Labeling
```python
# Create data labeling job
labeling_job = aiplatform.DataLabelingJob.create(
    display_name='image-labeling',
    datasets=[image_dataset],
    instruction_uri='gs://my-bucket/labeling-instructions.pdf',
    inputs_schema_uri=aiplatform.schema.dataset.ioformat.image.single_label_classification,
    annotation_schema_uri=aiplatform.schema.dataset.annotation.classification
)
```

## Performance Optimization

### Compute Resources
```python
# Specify machine types and accelerators
job = aiplatform.CustomTrainingJob(
    display_name='optimized-training',
    script_path='train.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-gpu.1-9:latest'
)

model = job.run(
    replica_count=4,  # Multi-node training
    machine_type='n1-highmem-16',  # High-memory machine
    accelerator_type='NVIDIA_TESLA_V100',  # GPU acceleration
    accelerator_count=4  # Multiple GPUs
)
```

### Model Optimization
```python
# Hyperparameter tuning
tuning_job = aiplatform.HyperparameterTuningJob(
    display_name='hyperparameter-tuning',
    custom_job=custom_job,
    metric_spec={'accuracy': 'maximize'},
    parameter_spec={
        'learning_rate': hpt.DoubleParameterSpec(min=0.001, max=0.1, scale='log'),
        'batch_size': hpt.IntegerParameterSpec(min=16, max=128, scale='linear')
    },
    max_trial_count=20,
    parallel_trial_count=4
)
```

## Security and Compliance

### IAM and Security
```python
# Service account for Vertex AI
from google.oauth2 import service_account

credentials = service_account.Credentials.from_service_account_file(
    'path/to/service-account-key.json'
)

aiplatform.init(
    project='my-project',
    location='us-central1',
    credentials=credentials
)
```

### VPC and Network Security
```python
# Deploy with VPC configuration
endpoint = model.deploy(
    machine_type='n1-standard-2',
    network='projects/my-project/global/networks/my-vpc',
    encryption_spec_key_name='projects/my-project/locations/us-central1/keyRings/my-ring/cryptoKeys/my-key'
)
```

## Best Practices

### Development Workflow
1. **Start with AutoML**: Begin with automated solutions
2. **Experiment tracking**: Use Vertex AI experiments
3. **Version control**: Track model and data versions
4. **Gradual complexity**: Start simple, add complexity as needed

### Production Deployment
1. **Model validation**: Thorough testing before deployment
2. **Monitoring**: Implement comprehensive monitoring
3. **A/B testing**: Compare model versions in production
4. **Rollback strategy**: Plan for model rollbacks

### Cost Optimization
1. **Right-size resources**: Match compute to workload requirements
2. **Use preemptible instances**: For fault-tolerant training
3. **Optimize serving**: Scale endpoints based on demand
4. **Storage management**: Archive old models and data

### Performance Tuning
1. **Data preprocessing**: Optimize data pipeline performance
2. **Model architecture**: Choose efficient model designs
3. **Hardware acceleration**: Use GPUs and TPUs appropriately
4. **Batch optimization**: Optimize batch sizes for training and inference