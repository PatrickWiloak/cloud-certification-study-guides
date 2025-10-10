# ML Algorithms and Model Evaluation

## Supervised Learning Algorithms

### Linear Learner

**Use Cases**
- Binary classification
- Multi-class classification
- Regression
- Linear relationships in data

**Algorithm Types**
- Logistic regression (classification)
- Linear regression (regression)
- Supports L1 (Lasso) and L2 (Ridge) regularization

**Hyperparameters**
```python
from sagemaker import LinearLearner

linear = LinearLearner(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    predictor_type='binary_classifier',  # or 'multiclass_classifier', 'regressor'
    binary_classifier_model_selection_criteria='accuracy',

    # Key hyperparameters
    learning_rate=0.1,
    mini_batch_size=1000,
    epochs=15,
    l1=0.0,  # L1 regularization
    wd=0.0,  # L2 regularization (weight decay)
    use_bias=True,
    num_models=32,  # Train multiple models with different initializations
    num_calibration_samples=10000000
)
```

**Input Format**
- RecordIO-protobuf (recommended for best performance)
- CSV (first column is label)
- Pipe mode for large datasets

**When to Use**
- Fast training required
- Sparse data (text features, one-hot encoded)
- Linear separable data
- Baseline model
- Interpretable model needed

### XGBoost

**Characteristics**
- Gradient boosting trees
- Handles missing values
- Built-in regularization
- Feature importance
- Excellent performance

**Hyperparameters**
```python
from sagemaker.xgboost import XGBoost

xgb = XGBoost(
    entry_point='train.py',
    framework_version='1.5-1',
    hyperparameters={
        'max_depth': 5,         # Tree depth
        'eta': 0.2,             # Learning rate
        'objective': 'binary:logistic',  # or 'multi:softmax', 'reg:squarederror'
        'num_round': 100,       # Number of boosting rounds
        'subsample': 0.8,       # Row sampling
        'colsample_bytree': 0.8,  # Column sampling
        'alpha': 0,             # L1 regularization
        'lambda': 1,            # L2 regularization
        'gamma': 0,             # Minimum loss reduction for split
        'min_child_weight': 1,  # Minimum sum of instance weight in child
        'scale_pos_weight': 1,  # Balance of positive/negative weights
        'eval_metric': 'auc'    # Evaluation metric
    },
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge'
)
```

**Feature Importance**
```python
import xgboost as xgb
import matplotlib.pyplot as plt

# Train model
model = xgb.XGBClassifier()
model.fit(X_train, y_train)

# Plot feature importance
xgb.plot_importance(model, max_num_features=20)
plt.show()

# Get importance scores
importance_dict = model.get_booster().get_score(importance_type='weight')
```

**When to Use**
- Structured/tabular data
- Non-linear relationships
- Feature importance needed
- High accuracy required
- Moderate dataset size

### Random Cut Forest (RCF)

**Use Case: Anomaly Detection**
- Unsupervised algorithm
- Detects outliers in data
- Assigns anomaly scores

**Configuration**
```python
from sagemaker import RandomCutForest

rcf = RandomCutForest(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    num_samples_per_tree=256,
    num_trees=50,
    eval_metrics=['accuracy', 'precision_recall_fscore']
)

rcf.fit(rcf.record_set(training_data))

# Predict anomaly scores
predictor = rcf.deploy(initial_instance_count=1, instance_type='ml.m5.xlarge')
results = predictor.predict(test_data)
```

**When to Use**
- Fraud detection
- Network intrusion detection
- System health monitoring
- Quality control

### K-Means

**Use Case: Clustering**
- Unsupervised learning
- Groups similar data points
- Requires specifying K (number of clusters)

**Configuration**
```python
from sagemaker import KMeans

kmeans = KMeans(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    k=10,  # Number of clusters
    init_method='kmeans++',  # or 'random'
    max_iterations=100,
    tol=0.0001,
    extra_center_factor='auto'
)

kmeans.fit(kmeans.record_set(data))

# Get cluster assignments
predictor = kmeans.deploy(initial_instance_count=1, instance_type='ml.m5.xlarge')
cluster_assignments = predictor.predict(test_data)
```

**Choosing K**
```python
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# Elbow method
inertias = []
K_range = range(2, 11)

for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42)
    kmeans.fit(X)
    inertias.append(kmeans.inertia_)

plt.plot(K_range, inertias)
plt.xlabel('Number of clusters')
plt.ylabel('Inertia')
plt.show()

# Silhouette score
silhouette_scores = []
for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42)
    labels = kmeans.fit_predict(X)
    score = silhouette_score(X, labels)
    silhouette_scores.append(score)
```

**When to Use**
- Customer segmentation
- Image compression
- Document clustering
- Market basket analysis

### PCA (Principal Component Analysis)

**Use Case: Dimensionality Reduction**
- Reduces number of features
- Preserves variance
- Removes correlation

**Configuration**
```python
from sagemaker import PCA

pca = PCA(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    num_components=10,  # Number of principal components
    algorithm_mode='regular',  # or 'randomized' for large datasets
    subtract_mean=True
)

pca.fit(pca.record_set(data))

# Transform data
predictor = pca.deploy(initial_instance_count=1, instance_type='ml.m5.xlarge')
transformed_data = predictor.predict(test_data)
```

**Choosing Number of Components**
```python
from sklearn.decomposition import PCA
import numpy as np

pca = PCA()
pca.fit(X)

# Cumulative explained variance
cumsum = np.cumsum(pca.explained_variance_ratio_)

# Find number of components for 95% variance
n_components = np.argmax(cumsum >= 0.95) + 1
print(f"Components needed for 95% variance: {n_components}")
```

**When to Use**
- High-dimensional data
- Visualization (2D/3D)
- Noise reduction
- Speed up training
- Feature extraction

## Deep Learning Algorithms

### Image Classification

**Use Case**
- Classify images into categories
- Transfer learning with ResNet
- Supports multi-label classification

**Configuration**
```python
from sagemaker.image_uris import retrieve

training_image = retrieve('image-classification', region, version='latest')

image_classifier = sagemaker.estimator.Estimator(
    training_image,
    role=role,
    instance_count=1,
    instance_type='ml.p3.2xlarge',
    output_path='s3://bucket/output',
    hyperparameters={
        'num_layers': 18,  # ResNet layers: 18, 34, 50, 101, 152
        'use_pretrained_model': 1,  # Transfer learning
        'num_classes': 10,
        'num_training_samples': 50000,
        'mini_batch_size': 128,
        'epochs': 30,
        'learning_rate': 0.1,
        'optimizer': 'sgd',  # or 'adam', 'rmsprop'
        'momentum': 0.9,
        'weight_decay': 0.0001,
        'augmentation_type': 'crop_color_transform'
    }
)
```

**Input Format**
- RecordIO (recommended)
- Image files (.jpg, .png) with lst file

**When to Use**
- Object recognition
- Medical image analysis
- Satellite imagery
- Quality inspection

### Object Detection

**Algorithms**
- Single Shot Detector (SSD)
- Finds and classifies objects
- Bounding box coordinates

**Configuration**
```python
object_detector = sagemaker.estimator.Estimator(
    retrieve('object-detection', region),
    role=role,
    instance_count=1,
    instance_type='ml.p3.2xlarge',
    hyperparameters={
        'base_network': 'resnet-50',  # or 'vgg-16'
        'num_classes': 20,
        'num_training_samples': 16551,
        'mini_batch_size': 32,
        'epochs': 30,
        'learning_rate': 0.001,
        'optimizer': 'sgd',
        'momentum': 0.9,
        'weight_decay': 0.0005,
        'overlap_threshold': 0.5,
        'nms_threshold': 0.45,
        'image_shape': 300,
        'label_width': 350,
        'num_training_samples': 16551
    }
)
```

### Semantic Segmentation

**Use Case**
- Pixel-level classification
- Medical imaging
- Autonomous vehicles

**Configuration**
```python
semantic_segmentation = sagemaker.estimator.Estimator(
    retrieve('semantic-segmentation', region),
    role=role,
    instance_count=1,
    instance_type='ml.p3.2xlarge',
    hyperparameters={
        'backbone': 'resnet-50',  # or 'resnet-101'
        'algorithm': 'fcn',  # or 'psp', 'deeplab'
        'use_pretrained_model': 'True',
        'crop_size': 240,
        'num_classes': 21,
        'epochs': 30,
        'learning_rate': 0.0001,
        'optimizer': 'adam',
        'mini_batch_size': 16,
        'validation_mini_batch_size': 16,
        'num_training_samples': 1464
    }
)
```

### DeepAR

**Use Case: Time Series Forecasting**
- Probabilistic forecasts
- Multiple related time series
- Handles missing values

**Configuration**
```python
deepar = sagemaker.estimator.Estimator(
    retrieve('forecasting-deepar', region),
    role=role,
    instance_count=1,
    instance_type='ml.c5.2xlarge',
    hyperparameters={
        'time_freq': 'D',  # Daily frequency
        'context_length': 30,  # How much history to use
        'prediction_length': 7,  # Forecast horizon
        'num_cells': 40,  # LSTM cells
        'num_layers': 3,
        'likelihood': 'gaussian',  # or 'beta', 'negative-binomial', 'student-T'
        'epochs': 100,
        'mini_batch_size': 32,
        'learning_rate': 0.001,
        'dropout_rate': 0.1,
        'early_stopping_patience': 10
    }
)
```

**Input Format**
```json
{
  "start": "2020-01-01 00:00:00",
  "target": [10.0, 12.5, 15.2, ...],
  "cat": [0, 1],  # Categorical features
  "dynamic_feat": [[1.0, 1.1, ...], [2.0, 2.1, ...]]  # Time-varying features
}
```

## NLP Algorithms

### BlazingText

**Modes**
1. **Word2Vec**: Word embeddings
2. **Text Classification**: Supervised classification

**Word2Vec Configuration**
```python
blazingtext = sagemaker.estimator.Estimator(
    retrieve('blazingtext', region),
    role=role,
    instance_count=1,
    instance_type='ml.c5.xlarge',
    hyperparameters={
        'mode': 'skipgram',  # or 'cbow', 'batch_skipgram'
        'vector_dim': 100,
        'window_size': 5,
        'negative_samples': 5,
        'min_count': 5,
        'epochs': 5,
        'learning_rate': 0.05,
        'sampling_threshold': 0.0001
    }
)
```

**Text Classification**
```python
blazingtext_classifier = sagemaker.estimator.Estimator(
    retrieve('blazingtext', region),
    role=role,
    instance_count=1,
    instance_type='ml.c5.xlarge',
    hyperparameters={
        'mode': 'supervised',
        'epochs': 10,
        'min_count': 2,
        'learning_rate': 0.05,
        'vector_dim': 10,
        'early_stopping': 'True',
        'patience': 4,
        'min_epochs': 5,
        'word_ngrams': 2
    }
)
```

### Sequence2Sequence

**Use Case**
- Machine translation
- Text summarization
- Speech recognition

**Configuration**
```python
seq2seq = sagemaker.estimator.Estimator(
    retrieve('seq2seq', region),
    role=role,
    instance_count=1,
    instance_type='ml.p3.2xlarge',
    hyperparameters={
        'num_layers_encoder': 2,
        'num_layers_decoder': 2,
        'num_embed_source': 512,
        'num_embed_target': 512,
        'encoder_rnn_type': 'gru',  # or 'lstm'
        'decoder_rnn_type': 'gru',
        'attention_type': 'mlp',
        'batch_size': 64,
        'optimizer': 'adam',
        'learning_rate': 0.0003,
        'epochs': 10
    }
)
```

## Model Evaluation Metrics

### Classification Metrics

**Confusion Matrix**
```python
from sklearn.metrics import confusion_matrix, classification_report
import seaborn as sns
import matplotlib.pyplot as plt

# Compute confusion matrix
cm = confusion_matrix(y_true, y_pred)

# Visualize
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
plt.ylabel('True Label')
plt.xlabel('Predicted Label')
plt.show()

# Detailed report
print(classification_report(y_true, y_pred))
```

**Binary Classification Metrics**
```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, roc_curve, precision_recall_curve, average_precision_score
)

# Basic metrics
accuracy = accuracy_score(y_true, y_pred)
precision = precision_score(y_true, y_pred)
recall = recall_score(y_true, y_pred)
f1 = f1_score(y_true, y_pred)

# AUC metrics
auc = roc_auc_score(y_true, y_pred_proba)
avg_precision = average_precision_score(y_true, y_pred_proba)

# ROC curve
fpr, tpr, thresholds = roc_curve(y_true, y_pred_proba)
plt.plot(fpr, tpr, label=f'AUC = {auc:.3f}')
plt.plot([0, 1], [0, 1], 'k--')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.legend()
plt.show()

# Precision-Recall curve
precision_curve, recall_curve, _ = precision_recall_curve(y_true, y_pred_proba)
plt.plot(recall_curve, precision_curve, label=f'AP = {avg_precision:.3f}')
plt.xlabel('Recall')
plt.ylabel('Precision')
plt.legend()
plt.show()
```

**Multi-Class Metrics**
```python
from sklearn.metrics import accuracy_score, precision_recall_fscore_support

# Overall accuracy
accuracy = accuracy_score(y_true, y_pred)

# Per-class metrics
precision, recall, f1, support = precision_recall_fscore_support(
    y_true, y_pred, average=None
)

# Macro average (unweighted mean)
precision_macro, recall_macro, f1_macro, _ = precision_recall_fscore_support(
    y_true, y_pred, average='macro'
)

# Weighted average (weighted by support)
precision_weighted, recall_weighted, f1_weighted, _ = precision_recall_fscore_support(
    y_true, y_pred, average='weighted'
)
```

### Regression Metrics

**Common Metrics**
```python
from sklearn.metrics import (
    mean_absolute_error, mean_squared_error, r2_score,
    mean_absolute_percentage_error, explained_variance_score
)
import numpy as np

# MAE: Mean Absolute Error
mae = mean_absolute_error(y_true, y_pred)

# MSE: Mean Squared Error
mse = mean_squared_error(y_true, y_pred)

# RMSE: Root Mean Squared Error
rmse = np.sqrt(mse)

# R-squared
r2 = r2_score(y_true, y_pred)

# MAPE: Mean Absolute Percentage Error
mape = mean_absolute_percentage_error(y_true, y_pred)

# Explained variance
exp_var = explained_variance_score(y_true, y_pred)

print(f"MAE: {mae:.4f}")
print(f"RMSE: {rmse:.4f}")
print(f"R²: {r2:.4f}")
print(f"MAPE: {mape:.4f}%")
```

**Residual Analysis**
```python
residuals = y_true - y_pred

# Residual plot
plt.scatter(y_pred, residuals)
plt.axhline(y=0, color='r', linestyle='--')
plt.xlabel('Predicted Values')
plt.ylabel('Residuals')
plt.show()

# Histogram of residuals
plt.hist(residuals, bins=30, edgecolor='black')
plt.xlabel('Residual Value')
plt.ylabel('Frequency')
plt.show()

# Q-Q plot
import scipy.stats as stats
stats.probplot(residuals, dist="norm", plot=plt)
plt.show()
```

### Cross-Validation

**K-Fold Cross-Validation**
```python
from sklearn.model_selection import cross_val_score, cross_validate

# Simple cross-validation
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"Accuracy: {scores.mean():.4f} (+/- {scores.std() * 2:.4f})")

# Multiple metrics
scoring = ['accuracy', 'precision', 'recall', 'f1', 'roc_auc']
scores = cross_validate(model, X, y, cv=5, scoring=scoring)

for metric in scoring:
    print(f"{metric}: {scores[f'test_{metric}'].mean():.4f}")
```

**Stratified K-Fold**
```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

for train_idx, val_idx in skf.split(X, y):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]

    model.fit(X_train, y_train)
    score = model.score(X_val, y_val)
    print(f"Fold score: {score:.4f}")
```

### Model Comparison

**Statistical Tests**
```python
from scipy import stats

# Paired t-test
model1_scores = cross_val_score(model1, X, y, cv=10)
model2_scores = cross_val_score(model2, X, y, cv=10)

t_stat, p_value = stats.ttest_rel(model1_scores, model2_scores)
print(f"t-statistic: {t_stat:.4f}, p-value: {p_value:.4f}")

if p_value < 0.05:
    print("Significant difference between models")
else:
    print("No significant difference")
```

## Algorithm Selection Guide

### Decision Framework

**Problem Type**
- **Binary Classification**: Linear Learner, XGBoost, Neural Networks
- **Multi-class Classification**: XGBoost, Image Classification, BlazingText
- **Regression**: Linear Learner, XGBoost, Neural Networks
- **Clustering**: K-Means
- **Anomaly Detection**: Random Cut Forest, IP Insights
- **Dimensionality Reduction**: PCA
- **Time Series**: DeepAR, Amazon Forecast
- **NLP**: BlazingText, Sequence2Sequence, Comprehend
- **Computer Vision**: Image Classification, Object Detection, Rekognition

**Data Characteristics**
- **Structured/Tabular**: XGBoost, Linear Learner
- **Images**: Image Classification, Object Detection
- **Text**: BlazingText, Comprehend
- **Time Series**: DeepAR, Forecast
- **High-dimensional**: PCA first, then other algorithms
- **Sparse data**: Linear Learner, Factorization Machines

**Performance Requirements**
- **Fast training**: Linear Learner, K-Means
- **High accuracy**: XGBoost, Deep Learning
- **Interpretability**: Linear Learner, Decision Trees
- **Scalability**: Built-in algorithms, distributed training

## Exam Tips

### Algorithm Selection
- XGBoost: Default for tabular data
- Linear Learner: Fast, sparse data, linear relationships
- Image Classification: Computer vision tasks
- DeepAR: Time series forecasting
- BlazingText: Text classification, word embeddings
- RCF: Anomaly detection
- K-Means: Clustering
- PCA: Dimensionality reduction

### Metrics Selection
- **Imbalanced data**: Precision, Recall, F1, AUC
- **Balanced data**: Accuracy
- **Business cost of errors**: Weighted precision/recall
- **Regression**: RMSE (penalizes large errors), MAE (robust to outliers)
- **Ranking**: AUC, Average Precision

### Common Scenarios
- High false positives: Increase precision (higher threshold)
- High false negatives: Increase recall (lower threshold)
- Overfitting: Regularization, more data, simpler model
- Underfitting: More features, complex model, more training
- Imbalanced classes: Resampling, class weights, F1 score

### Best Practices
- Start with simple baseline
- Try multiple algorithms
- Cross-validation for robust estimates
- Monitor multiple metrics
- Consider business context
- Understand algorithm assumptions
- Use built-in algorithms when possible
