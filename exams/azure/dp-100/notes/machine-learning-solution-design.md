# Machine Learning Solution Design

## Overview
Designing and preparing a machine learning solution involves understanding requirements, selecting appropriate Azure Machine Learning resources, configuring workspaces, and establishing development environments for data science teams.

## Azure Machine Learning Workspace

### Workspace Fundamentals

#### What is an Azure ML Workspace?
- Central place to work with ML artifacts
- Top-level resource for Azure Machine Learning
- Organizes experiments, models, deployments, and compute
- Provides collaboration and resource management
- Tracks history and lineage of ML assets

#### Key Components
- **Compute Targets:** Where training and inference run
- **Datastores:** Connections to data sources
- **Datasets:** References to data for training
- **Experiments:** Organized runs of training scripts
- **Pipelines:** Automated ML workflows
- **Models:** Trained models and their metadata
- **Endpoints:** Deployed models for inference
- **Environments:** Software dependencies for runs

#### Workspace Creation
```python
from azureml.core import Workspace

# Create workspace
ws = Workspace.create(
    name='my-workspace',
    subscription_id='<subscription-id>',
    resource_group='my-resource-group',
    location='eastus',
    create_resource_group=True
)

# Connect to existing workspace
ws = Workspace.from_config()
```

### Resource Organization

#### Resource Groups
- Logical container for Azure resources
- Organize related resources together
- Apply tags for categorization and billing
- Set policies and access controls at RG level
- Best practice: Separate RG for each environment (dev, test, prod)

#### Workspace Best Practices
1. **Separate workspaces by:**
   - Environment (development, staging, production)
   - Project or team
   - Compliance requirements
   - Cost tracking needs

2. **Naming Conventions:**
   - Use descriptive names
   - Include environment indicator
   - Follow organizational standards
   - Example: `ml-workspace-prod-eastus`

3. **Access Control:**
   - Use Azure RBAC for workspace access
   - Principle of least privilege
   - Separate data scientists, engineers, and operators
   - Use Azure AD groups for team management

### Compute Resources

#### Compute Instances
- **Purpose:** Development workstation for data scientists
- **Characteristics:**
  - Fully managed VM with ML tools pre-installed
  - JupyterLab, VS Code, and RStudio
  - Single-user environment
  - Can be stopped when not in use
  - Auto-shutdown schedules to save costs

- **Use Cases:**
  - Interactive development and testing
  - Notebook-based experimentation
  - Debugging and prototyping
  - Small-scale training

- **Configuration:**
```python
from azureml.core.compute import ComputeInstance

compute_config = ComputeInstance.provisioning_configuration(
    vm_size='STANDARD_DS3_V2',
    ssh_public_access=False
)

instance = ComputeInstance.create(
    ws,
    name='my-compute-instance',
    provisioning_configuration=compute_config
)
```

#### Compute Clusters
- **Purpose:** Scalable compute for training and batch inference
- **Characteristics:**
  - Auto-scaling from 0 to max nodes
  - Multi-node distributed training
  - Automatic VM management
  - Cost-effective with auto-shutdown
  - Supports low-priority VMs for cost savings

- **Use Cases:**
  - Large-scale model training
  - Hyperparameter tuning
  - Automated ML
  - Batch scoring
  - Parallel training jobs

- **Configuration:**
```python
from azureml.core.compute import AmlCompute

compute_config = AmlCompute.provisioning_configuration(
    vm_size='STANDARD_DS3_V2',
    min_nodes=0,
    max_nodes=4,
    idle_seconds_before_scaledown=300,
    vm_priority='lowpriority'  # or 'dedicated'
)

cluster = ComputeInstance.create(
    ws,
    name='my-cluster',
    provisioning_configuration=compute_config
)
```

#### Inference Clusters (AKS)
- **Purpose:** Real-time model serving
- **Characteristics:**
  - Azure Kubernetes Service (AKS) clusters
  - High-performance, low-latency inference
  - Auto-scaling based on load
  - Production-grade deployments
  - GPU support for deep learning

- **Use Cases:**
  - Real-time predictions
  - High-throughput scoring
  - Production model deployments
  - Web services and APIs

#### Attached Compute
- **Purpose:** Use existing compute resources
- **Types:**
  - Azure Databricks
  - Azure Synapse Analytics
  - Azure Virtual Machines
  - Remote compute (on-premises or other clouds)

- **Use Cases:**
  - Leverage existing infrastructure
  - Specific tool requirements
  - Compliance or governance needs
  - Cost optimization with existing resources

### Compute Selection Guidelines

| Workload Type | Recommended Compute | Reason |
|--------------|-------------------|---------|
| Development & Debugging | Compute Instance | Interactive, pre-configured |
| Small experiments | Compute Instance | Quick setup, lower cost |
| Large-scale training | Compute Cluster | Auto-scaling, distributed training |
| Hyperparameter tuning | Compute Cluster | Parallel execution |
| Automated ML | Compute Cluster | Multiple parallel runs |
| Real-time inference | AKS Cluster | Low latency, high availability |
| Batch inference | Compute Cluster | Cost-effective for large batches |
| Spark-based ML | Databricks | Native Spark integration |

## Datastores and Datasets

### Datastores

#### Overview
- Abstraction over Azure storage services
- Store connection information securely
- No data duplication, just references
- Support for various storage types
- Credential management handled by Azure ML

#### Supported Storage Types
- **Azure Blob Storage:** Unstructured data, scalable
- **Azure File Share:** File-based storage with SMB protocol
- **Azure Data Lake Gen2:** Big data analytics, hierarchical namespace
- **Azure SQL Database:** Structured relational data
- **Azure Database for PostgreSQL/MySQL:** Open-source databases
- **Azure Databricks File System (DBFS):** Databricks storage

#### Default Datastores
- **workspaceblobstore:** Default blob storage
- **workspacefilestore:** Default file share
- Created automatically with workspace

#### Registering Datastores
```python
from azureml.core import Datastore

# Register blob datastore
blob_datastore = Datastore.register_azure_blob_container(
    workspace=ws,
    datastore_name='my_blob_datastore',
    container_name='my-container',
    account_name='mystorageaccount',
    account_key='<account-key>'
)

# Register Data Lake Gen2
datalake_datastore = Datastore.register_azure_data_lake_gen2(
    workspace=ws,
    datastore_name='my_datalake',
    filesystem='my-filesystem',
    account_name='mydatalakeaccount',
    tenant_id='<tenant-id>',
    client_id='<client-id>',
    client_secret='<client-secret>'
)
```

### Datasets

#### Overview
- References to data in datastores
- Versioned and tracked
- Reusable across experiments
- Support data profiling and monitoring
- Two types: FileDataset and TabularDataset

#### FileDataset
- References one or more files
- Suitable for computer vision, audio, unstructured data
- Downloads or mounts files
- Preserves file structure

```python
from azureml.core import Dataset

# Create from datastore path
file_dataset = Dataset.File.from_files(
    path=(datastore, 'images/**/*.jpg')
)

# Register dataset
file_dataset = file_dataset.register(
    workspace=ws,
    name='image_files',
    description='Training images',
    create_new_version=True
)
```

#### TabularDataset
- Represents data in tabular format
- Parses files into DataFrame
- Supports CSV, Parquet, JSON, SQL query results
- Enables data profiling and transformation

```python
# Create from delimited files
tabular_dataset = Dataset.Tabular.from_delimited_files(
    path=(datastore, 'data/*.csv')
)

# Create from SQL query
tabular_dataset = Dataset.Tabular.from_sql_query(
    query="SELECT * FROM my_table WHERE date > '2024-01-01'",
    query_timeout=10,
    datastore=sql_datastore
)

# Register dataset
tabular_dataset = tabular_dataset.register(
    workspace=ws,
    name='sales_data',
    description='Sales transactions',
    create_new_version=True
)
```

#### Dataset Versioning
- Track changes to data over time
- Reproducibility of experiments
- Compare model performance across versions
- Automatic versioning with register()

#### Data Drift Monitoring
- Detect changes in input data distribution
- Compare baseline vs target dataset
- Alerts when drift exceeds threshold
- Schedule automated drift detection

## Experiment Design

### Experiments and Runs

#### Experiments
- Logical container for related runs
- Organizes training iterations
- Compares multiple approaches
- Best practice: One experiment per project or model type

```python
from azureml.core import Experiment

# Create or get experiment
experiment = Experiment(workspace=ws, name='my-experiment')

# Submit a run
run = experiment.submit(config)
```

#### Runs
- Single execution of training code
- Captures metrics, outputs, and logs
- Tracks code, environment, and compute
- Can be child runs (for hyperparameter tuning)

```python
from azureml.core import Run

# In training script
run = Run.get_context()

# Log metrics
run.log('accuracy', 0.95)
run.log('loss', 0.23)

# Log multiple values for iteration
for i in range(100):
    run.log('training_loss', loss_values[i])

# Upload files
run.upload_file('outputs/model.pkl', 'model.pkl')

# Complete run
run.complete()
```

### Environment Management

#### Azure ML Environments
- Specify Python packages and versions
- Docker container configuration
- Conda or pip dependencies
- Ensures reproducibility across runs
- Curated environments provided by Azure

#### Creating Environments
```python
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies

# Create from conda specification
env = Environment('my-env')
conda_dep = CondaDependencies.create(
    conda_packages=['scikit-learn', 'pandas'],
    pip_packages=['azureml-defaults']
)
env.python.conda_dependencies = conda_dep

# Create from existing conda file
env = Environment.from_conda_specification(
    name='my-env',
    file_path='./conda.yml'
)

# Use curated environment
env = Environment.get(workspace=ws, name='AzureML-sklearn-1.0-ubuntu20.04-py38-cpu')

# Register environment
env.register(workspace=ws)
```

## ML Workflow Types

### Script-Based Training

#### ScriptRunConfig
- Run training scripts on compute targets
- Specify script, compute, environment
- Pass arguments to script
- Most flexible training approach

```python
from azureml.core import ScriptRunConfig

config = ScriptRunConfig(
    source_directory='./src',
    script='train.py',
    arguments=['--learning-rate', 0.01, '--epochs', 10],
    compute_target='my-cluster',
    environment=env
)

run = experiment.submit(config)
```

### Automated Machine Learning (AutoML)

#### Overview
- Automatically tries multiple algorithms
- Hyperparameter optimization
- Feature engineering
- Model selection
- Suitable for classification, regression, forecasting

#### Configuration
```python
from azureml.train.automl import AutoMLConfig

automl_config = AutoMLConfig(
    task='classification',
    primary_metric='accuracy',
    training_data=train_dataset,
    validation_data=val_dataset,
    label_column_name='target',
    compute_target='my-cluster',
    iterations=20,
    max_concurrent_iterations=4,
    experiment_timeout_hours=1,
    enable_early_stopping=True,
    featurization='auto'
)

run = experiment.submit(automl_config)
```

### ML Pipelines

#### Overview
- Multi-step ML workflows
- Reusable and reproducible
- Schedule and automate
- Parallel execution of steps
- Data dependency management

#### Pipeline Steps
- **PythonScriptStep:** Run Python script
- **EstimatorStep:** Run training with estimator
- **DataTransferStep:** Move data between stores
- **DatabricksStep:** Run Databricks notebook/JAR
- **ParallelRunStep:** Batch inference on large datasets

```python
from azureml.pipeline.steps import PythonScriptStep
from azureml.pipeline.core import Pipeline

# Define steps
prep_step = PythonScriptStep(
    name='prepare_data',
    script_name='prepare.py',
    compute_target='my-cluster',
    source_directory='./src'
)

train_step = PythonScriptStep(
    name='train_model',
    script_name='train.py',
    compute_target='my-cluster',
    source_directory='./src'
)

# Create pipeline
pipeline = Pipeline(workspace=ws, steps=[prep_step, train_step])

# Submit pipeline
pipeline_run = experiment.submit(pipeline)
```

## Best Practices

### Workspace Management
1. **Use separate workspaces for environments** (dev, test, prod)
2. **Implement proper RBAC** for team access control
3. **Tag resources** for cost tracking and organization
4. **Use managed identities** for secure authentication
5. **Enable diagnostic logging** for monitoring and troubleshooting

### Compute Optimization
1. **Use auto-shutdown** for compute instances
2. **Set appropriate min/max nodes** for clusters
3. **Consider low-priority VMs** for non-critical workloads
4. **Right-size VM SKUs** for workload requirements
5. **Use spot instances** for cost savings on interruptible workloads

### Data Management
1. **Register datasets** for reusability and versioning
2. **Use datastores** for centralized data access
3. **Implement data validation** before training
4. **Monitor data drift** in production
5. **Document data sources** and transformations

### Experiment Organization
1. **Use meaningful experiment names** related to project
2. **Tag runs** with metadata for easy filtering
3. **Log all relevant metrics** and hyperparameters
4. **Version environments** for reproducibility
5. **Document experiment goals** and findings

### Security and Compliance
1. **Use Azure Key Vault** for secrets
2. **Enable workspace encryption** for sensitive data
3. **Implement network isolation** with private endpoints
4. **Follow data governance** policies
5. **Audit access** and operations regularly

## Study Tips

### Key Concepts to Master
- Azure ML workspace components and hierarchy
- Differences between compute types and when to use each
- Datastores vs datasets and their purposes
- Environment management for reproducibility
- Experiment and run organization
- Pipeline concepts and benefits

### Common Scenarios
1. **Setting up development environment** → Compute Instance
2. **Large-scale hyperparameter tuning** → Compute Cluster with AutoML
3. **Production model deployment** → AKS Cluster
4. **Batch scoring large datasets** → Compute Cluster with ParallelRunStep
5. **Reproducible training** → Registered environments and datasets

### Exam Focus Areas
- Workspace creation and configuration
- Compute target selection for different scenarios
- Datastore and dataset registration
- Environment specification and management
- ScriptRunConfig usage
- AutoML configuration
- Basic pipeline creation

### Practice Tasks
1. Create and configure an Azure ML workspace
2. Set up different compute types
3. Register datastores and datasets
4. Create and register environments
5. Submit training runs with ScriptRunConfig
6. Configure AutoML experiments
7. Build simple ML pipelines

### Remember
- Workspace = Top-level organization for ML assets
- Compute Instance = Development, Compute Cluster = Training
- Datastores = Connection info, Datasets = Data references
- Environments = Dependencies for reproducibility
- Experiments = Logical grouping, Runs = Individual executions
- Pipelines = Multi-step workflows for automation
