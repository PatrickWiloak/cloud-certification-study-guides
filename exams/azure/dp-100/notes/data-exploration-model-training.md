# Data Exploration and Model Training

## Overview
This section covers data exploration, feature engineering, model training techniques including automated machine learning, hyperparameter tuning, and distributed training on Azure Machine Learning.

## Data Exploration and Analysis

### Loading and Inspecting Data

#### Working with Datasets
```python
from azureml.core import Workspace, Dataset
import pandas as pd

# Load workspace
ws = Workspace.from_config()

# Get registered dataset
dataset = Dataset.get_by_name(ws, 'my_dataset')

# Convert to pandas DataFrame
df = dataset.to_pandas_dataframe()

# Basic inspection
print(df.head())
print(df.info())
print(df.describe())
print(df.shape)
```

#### Data Profiling
```python
# Check for missing values
missing = df.isnull().sum()
missing_pct = (missing / len(df)) * 100

# Check data types
dtypes = df.dtypes

# Check unique values
for col in df.columns:
    unique_count = df[col].nunique()
    print(f"{col}: {unique_count} unique values")

# Statistical summary
summary = df.describe(include='all')
```

### Exploratory Data Analysis (EDA)

#### Distribution Analysis
```python
import matplotlib.pyplot as plt
import seaborn as sns

# Histograms for numerical features
df.hist(figsize=(15, 10), bins=50)
plt.tight_layout()

# Box plots to detect outliers
plt.figure(figsize=(12, 6))
df.boxplot()

# Distribution plots
for col in numeric_columns:
    plt.figure()
    sns.histplot(df[col], kde=True)
    plt.title(f'Distribution of {col}')
```

#### Correlation Analysis
```python
# Correlation matrix
correlation = df.corr()

# Heatmap
plt.figure(figsize=(12, 10))
sns.heatmap(correlation, annot=True, cmap='coolwarm', center=0)
plt.title('Feature Correlation Matrix')

# Strong correlations
high_corr = correlation[abs(correlation) > 0.7]
```

#### Categorical Analysis
```python
# Value counts
for col in categorical_columns:
    print(f"\n{col} distribution:")
    print(df[col].value_counts())

# Bar plots
for col in categorical_columns:
    plt.figure()
    df[col].value_counts().plot(kind='bar')
    plt.title(f'Distribution of {col}')
```

#### Target Variable Analysis
```python
# Target distribution (classification)
target_dist = df['target'].value_counts()
print(f"Class distribution:\n{target_dist}")

# Check for class imbalance
imbalance_ratio = target_dist.max() / target_dist.min()
print(f"Imbalance ratio: {imbalance_ratio}")

# Target vs features (regression)
for col in numeric_columns:
    plt.figure()
    plt.scatter(df[col], df['target'])
    plt.xlabel(col)
    plt.ylabel('Target')
```

## Data Preprocessing and Feature Engineering

### Handling Missing Values

#### Detection and Analysis
```python
# Identify missing values
missing_data = df.isnull().sum()
missing_pct = (missing_data / len(df)) * 100

# Visualize missing data
import missingno as msno
msno.matrix(df)
msno.heatmap(df)
```

#### Imputation Strategies
```python
from sklearn.impute import SimpleImputer, KNNImputer

# Mean imputation (numerical)
imputer = SimpleImputer(strategy='mean')
df[numeric_columns] = imputer.fit_transform(df[numeric_columns])

# Median imputation (robust to outliers)
imputer = SimpleImputer(strategy='median')

# Most frequent (categorical)
imputer = SimpleImputer(strategy='most_frequent')
df[categorical_columns] = imputer.fit_transform(df[categorical_columns])

# Constant value
imputer = SimpleImputer(strategy='constant', fill_value=0)

# KNN imputation
knn_imputer = KNNImputer(n_neighbors=5)
df_imputed = knn_imputer.fit_transform(df)
```

### Handling Outliers

#### Detection Methods
```python
# IQR method
Q1 = df.quantile(0.25)
Q3 = df.quantile(0.75)
IQR = Q3 - Q1

outliers = ((df < (Q1 - 1.5 * IQR)) | (df > (Q3 + 1.5 * IQR)))

# Z-score method
from scipy import stats
z_scores = np.abs(stats.zscore(df[numeric_columns]))
outliers_zscore = (z_scores > 3)

# Isolation Forest
from sklearn.ensemble import IsolationForest
iso_forest = IsolationForest(contamination=0.1, random_state=42)
outliers_iso = iso_forest.fit_predict(df[numeric_columns])
```

#### Handling Strategies
```python
# Remove outliers
df_no_outliers = df[~outliers.any(axis=1)]

# Cap outliers (winsorization)
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR
df_capped = df.clip(lower=lower_bound, upper=upper_bound, axis=1)

# Transform (log, sqrt)
df['log_feature'] = np.log1p(df['feature'])
```

### Feature Encoding

#### Label Encoding
```python
from sklearn.preprocessing import LabelEncoder

# For ordinal categories
le = LabelEncoder()
df['education_encoded'] = le.fit_transform(df['education'])
# Preserves order: High School < Bachelor < Master < PhD
```

#### One-Hot Encoding
```python
from sklearn.preprocessing import OneHotEncoder
import pandas as pd

# For nominal categories
encoder = OneHotEncoder(sparse=False, handle_unknown='ignore')
encoded = encoder.fit_transform(df[['category']])

# Using pandas
df_encoded = pd.get_dummies(df, columns=['category'], drop_first=True)
```

#### Target Encoding
```python
# Mean target encoding (be careful of overfitting)
target_means = df.groupby('category')['target'].mean()
df['category_encoded'] = df['category'].map(target_means)
```

### Feature Scaling

#### Standardization (Z-score normalization)
```python
from sklearn.preprocessing import StandardScaler

# Mean = 0, Std = 1
scaler = StandardScaler()
scaled_features = scaler.fit_transform(df[numeric_columns])

# Formula: (x - mean) / std
```

#### Normalization (Min-Max scaling)
```python
from sklearn.preprocessing import MinMaxScaler

# Scale to [0, 1]
scaler = MinMaxScaler()
normalized_features = scaler.fit_transform(df[numeric_columns])

# Formula: (x - min) / (max - min)

# Scale to custom range
scaler = MinMaxScaler(feature_range=(-1, 1))
```

#### Robust Scaling
```python
from sklearn.preprocessing import RobustScaler

# Robust to outliers (uses median and IQR)
scaler = RobustScaler()
robust_scaled = scaler.fit_transform(df[numeric_columns])
```

### Feature Engineering Techniques

#### Creating New Features
```python
# Date features
df['year'] = pd.to_datetime(df['date']).dt.year
df['month'] = pd.to_datetime(df['date']).dt.month
df['day_of_week'] = pd.to_datetime(df['date']).dt.dayofweek
df['is_weekend'] = df['day_of_week'].isin([5, 6]).astype(int)

# Mathematical transformations
df['feature_squared'] = df['feature'] ** 2
df['feature_log'] = np.log1p(df['feature'])
df['feature_sqrt'] = np.sqrt(df['feature'])

# Interaction features
df['feature_interaction'] = df['feature1'] * df['feature2']
df['feature_ratio'] = df['feature1'] / (df['feature2'] + 1e-8)

# Aggregations
df['total'] = df[['col1', 'col2', 'col3']].sum(axis=1)
df['average'] = df[['col1', 'col2', 'col3']].mean(axis=1)

# Binning
df['age_group'] = pd.cut(df['age'], bins=[0, 18, 35, 50, 100],
                         labels=['young', 'adult', 'middle', 'senior'])
```

#### Feature Selection
```python
from sklearn.feature_selection import SelectKBest, f_classif, RFE
from sklearn.ensemble import RandomForestClassifier

# Univariate selection
selector = SelectKBest(score_func=f_classif, k=10)
X_selected = selector.fit_transform(X, y)
selected_features = X.columns[selector.get_support()]

# Recursive Feature Elimination (RFE)
estimator = RandomForestClassifier()
rfe = RFE(estimator=estimator, n_features_to_select=10)
X_rfe = rfe.fit_transform(X, y)

# Feature importance from tree-based models
rf = RandomForestClassifier()
rf.fit(X, y)
feature_importance = pd.DataFrame({
    'feature': X.columns,
    'importance': rf.feature_importances_
}).sort_values('importance', ascending=False)
```

## Model Training

### Train-Test Split

```python
from sklearn.model_selection import train_test_split

# Basic split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Stratified split (for classification)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

# Time series split (preserve temporal order)
# Don't shuffle! Use data[:train_size] and data[train_size:]
```

### Cross-Validation

```python
from sklearn.model_selection import cross_val_score, KFold, StratifiedKFold

# K-Fold Cross-Validation
kfold = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=kfold, scoring='accuracy')

# Stratified K-Fold (preserves class distribution)
skfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=skfold, scoring='accuracy')

# Time Series Split
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5)
scores = cross_val_score(model, X, y, cv=tscv)
```

### Training with Azure ML

#### ScriptRunConfig
```python
from azureml.core import ScriptRunConfig, Experiment

# Training script (train.py)
"""
import argparse
from azureml.core import Run
from sklearn.ensemble import RandomForestClassifier
import joblib

parser = argparse.ArgumentParser()
parser.add_argument('--n_estimators', type=int, default=100)
parser.add_argument('--max_depth', type=int, default=None)
args = parser.parse_args()

# Get run context
run = Run.get_context()

# Load data
dataset = run.input_datasets['training_data']
df = dataset.to_pandas_dataframe()

# Train model
model = RandomForestClassifier(
    n_estimators=args.n_estimators,
    max_depth=args.max_depth,
    random_state=42
)
model.fit(X_train, y_train)

# Log metrics
accuracy = model.score(X_test, y_test)
run.log('accuracy', accuracy)

# Save model
joblib.dump(model, 'outputs/model.pkl')
run.complete()
"""

# Submit training run
config = ScriptRunConfig(
    source_directory='./src',
    script='train.py',
    arguments=['--n_estimators', 200, '--max_depth', 10],
    compute_target='my-cluster',
    environment=env
)

experiment = Experiment(ws, 'my-experiment')
run = experiment.submit(config)
run.wait_for_completion(show_output=True)
```

## Automated Machine Learning (AutoML)

### AutoML Configuration

```python
from azureml.train.automl import AutoMLConfig

# Classification
automl_config = AutoMLConfig(
    task='classification',
    primary_metric='accuracy',
    training_data=train_dataset,
    validation_data=val_dataset,
    label_column_name='target',
    compute_target='my-cluster',

    # Experiment settings
    experiment_timeout_hours=1,
    iterations=20,
    max_concurrent_iterations=4,

    # Early stopping
    enable_early_stopping=True,

    # Featurization
    featurization='auto',  # or 'off' or custom config

    # Model selection
    blocked_models=['XGBoostClassifier'],  # Exclude specific models
    allowed_models=['RandomForest', 'LogisticRegression'],

    # Class balancing
    enable_voting_ensemble=True,
    enable_stack_ensemble=True,

    # Validation
    n_cross_validations=5
)

# Regression
automl_config = AutoMLConfig(
    task='regression',
    primary_metric='r2_score',
    training_data=train_dataset,
    label_column_name='price',
    compute_target='my-cluster'
)

# Time series forecasting
automl_config = AutoMLConfig(
    task='forecasting',
    primary_metric='normalized_root_mean_squared_error',
    training_data=train_dataset,
    label_column_name='sales',
    time_column_name='date',
    forecast_horizon=30,  # Number of periods to forecast
    grain_column_names=['store_id', 'product_id']  # Grouping columns
)
```

### Custom Featurization

```python
from azureml.automl.core.featurization import FeaturizationConfig

featurization_config = FeaturizationConfig()

# Drop columns
featurization_config.drop_columns = ['id', 'timestamp']

# Impute missing values
featurization_config.add_column_purpose('categorical_column', 'Categorical')
featurization_config.add_transformer_params('Imputer', ['numeric_column'],
                                           {"strategy": "median"})

# Block transformations
featurization_config.blocked_transformers = ['LabelEncoder']

automl_config = AutoMLConfig(
    task='classification',
    training_data=train_dataset,
    label_column_name='target',
    featurization=featurization_config
)
```

### Retrieving AutoML Results

```python
# Submit AutoML run
automl_run = experiment.submit(automl_config)
automl_run.wait_for_completion(show_output=True)

# Get best model
best_run, fitted_model = automl_run.get_output()

print(f"Best run ID: {best_run.id}")
print(f"Best model: {fitted_model}")

# Get all child runs
children = list(automl_run.get_children())
metricslist = {}
for run in children:
    properties = run.get_properties()
    metrics = run.get_metrics()
    metricslist[run.id] = metrics

# View feature importance
feature_importance = fitted_model.get_feature_importance_dict()
```

## Hyperparameter Tuning

### HyperDrive Configuration

```python
from azureml.train.hyperdrive import (
    HyperDriveConfig,
    RandomParameterSampling,
    BanditPolicy,
    PrimaryMetricGoal,
    choice,
    uniform,
    loguniform
)

# Define parameter space
param_sampling = RandomParameterSampling({
    '--learning_rate': loguniform(-6, -1),  # 10^-6 to 10^-1
    '--n_estimators': choice(50, 100, 200, 500),
    '--max_depth': choice(5, 10, 20, 50, None),
    '--min_samples_split': choice(2, 5, 10),
    '--batch_size': choice(16, 32, 64, 128)
})

# Early termination policy
early_termination_policy = BanditPolicy(
    evaluation_interval=2,
    slack_factor=0.1,  # Terminate if metric < best_metric * (1 - slack_factor)
    delay_evaluation=5  # Start policy after 5 intervals
)

# HyperDrive configuration
hyperdrive_config = HyperDriveConfig(
    run_config=script_config,
    hyperparameter_sampling=param_sampling,
    policy=early_termination_policy,
    primary_metric_name='accuracy',
    primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
    max_total_runs=20,
    max_concurrent_runs=4
)

# Submit HyperDrive run
hyperdrive_run = experiment.submit(hyperdrive_config)
hyperdrive_run.wait_for_completion(show_output=True)

# Get best run
best_run = hyperdrive_run.get_best_run_by_primary_metric()
best_run_metrics = best_run.get_metrics()
best_parameters = best_run.get_details()['runDefinition']['arguments']
```

### Parameter Sampling Methods

```python
# Grid Sampling - exhaustive search
from azureml.train.hyperdrive import GridParameterSampling

grid_sampling = GridParameterSampling({
    '--learning_rate': choice(0.001, 0.01, 0.1),
    '--batch_size': choice(16, 32, 64)
})

# Bayesian Sampling - intelligent search
from azureml.train.hyperdrive import BayesianParameterSampling

bayesian_sampling = BayesianParameterSampling({
    '--learning_rate': uniform(0.0001, 0.1),
    '--momentum': uniform(0.9, 0.99)
})
```

### Early Termination Policies

```python
# Bandit Policy - aggressive early stopping
bandit_policy = BanditPolicy(
    evaluation_interval=1,
    slack_factor=0.1
)

# Median Stopping Policy
from azureml.train.hyperdrive import MedianStoppingPolicy
median_policy = MedianStoppingPolicy(
    evaluation_interval=1,
    delay_evaluation=5
)

# Truncation Selection Policy
from azureml.train.hyperdrive import TruncationSelectionPolicy
truncation_policy = TruncationSelectionPolicy(
    evaluation_interval=1,
    truncation_percentage=20,  # Terminate bottom 20%
    delay_evaluation=5
)
```

## Distributed Training

### Data Parallel Training

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import MpiConfiguration

# MPI configuration for distributed training
mpi_config = MpiConfiguration(node_count=2, process_count_per_node=2)

config = ScriptRunConfig(
    source_directory='./src',
    script='train_distributed.py',
    compute_target='gpu-cluster',
    environment=env,
    distributed_job_config=mpi_config
)
```

### PyTorch Distributed Training

```python
# In training script (train_distributed.py)
import torch
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel
from azureml.core import Run

# Get run context
run = Run.get_context()

# Initialize distributed training
dist.init_process_group(backend='nccl')

# Model setup
model = YourModel()
model = model.to(device)
model = DistributedDataParallel(model)

# Data loader with distributed sampler
train_sampler = torch.utils.data.distributed.DistributedSampler(train_dataset)
train_loader = torch.utils.data.DataLoader(
    train_dataset,
    batch_size=batch_size,
    sampler=train_sampler
)

# Training loop
for epoch in range(epochs):
    train_sampler.set_epoch(epoch)
    for batch in train_loader:
        # Training code
        loss = train_step(batch)

        # Log only from rank 0
        if dist.get_rank() == 0:
            run.log('loss', loss.item())
```

## Best Practices

### Data Preparation
1. **Understand your data first** - thorough EDA before modeling
2. **Handle missing values appropriately** - consider why data is missing
3. **Detect and handle outliers** - but don't always remove them
4. **Encode categorical variables correctly** - ordinal vs nominal
5. **Scale features when necessary** - especially for distance-based algorithms

### Feature Engineering
1. **Create domain-specific features** - leverage business knowledge
2. **Test feature importance** - remove redundant features
3. **Avoid data leakage** - don't use future information
4. **Document transformations** - for reproducibility
5. **Version your features** - track feature evolution

### Model Training
1. **Start simple** - baseline models first
2. **Use cross-validation** - for robust performance estimates
3. **Track all experiments** - log metrics, parameters, and artifacts
4. **Monitor for overfitting** - validation set performance
5. **Use appropriate metrics** - match business objectives

### AutoML Usage
1. **Set reasonable timeouts** - balance exploration vs cost
2. **Configure appropriate validation** - cross-validation or hold-out
3. **Review feature engineering** - AutoML featurization may need customization
4. **Interpret results** - understand what AutoML selected
5. **Test on unseen data** - verify generalization

### Hyperparameter Tuning
1. **Define reasonable parameter ranges** - based on prior knowledge
2. **Use early termination** - save compute resources
3. **Choose appropriate sampling** - Bayesian for expensive training
4. **Monitor resource usage** - avoid wasted compute
5. **Validate best parameters** - retrain and verify

## Study Tips

### Key Concepts to Master
- Data profiling and exploratory analysis techniques
- Missing value and outlier handling strategies
- Feature encoding methods and when to use each
- Feature scaling and normalization
- AutoML configuration and capabilities
- HyperDrive parameter sampling and early termination
- Distributed training concepts

### Common Scenarios
1. **Imbalanced dataset** → Use stratified split, AutoML with class balancing
2. **Large-scale training** → Distributed training with HyperDrive
3. **Quick model development** → AutoML with custom featurization
4. **Optimal hyperparameters** → HyperDrive with Bayesian sampling
5. **Feature selection** → Use AutoML or feature importance from tree models

### Exam Focus Areas
- Data preprocessing techniques
- Feature engineering best practices
- AutoML configuration options
- HyperDrive setup and parameter tuning
- Cross-validation strategies
- Handling class imbalance
- Distributed training concepts

### Remember
- EDA is essential before modeling
- Proper data preprocessing significantly impacts model performance
- AutoML automates algorithm selection and hyperparameter tuning
- HyperDrive optimizes hyperparameters for a specific algorithm
- Cross-validation provides more reliable performance estimates
- Track all experiments for reproducibility
