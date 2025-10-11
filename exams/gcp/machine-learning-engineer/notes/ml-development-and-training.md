# ML Development and Training - GCP Professional Machine Learning Engineer

## Overview

Comprehensive guide to ML model development, training pipelines, Vertex AI, AutoML, distributed training, TPU optimization, and model optimization for the Professional Machine Learning Engineer certification exam.

This guide covers:
- Vertex AI Training Service and custom training jobs
- AutoML for Tables, Vision, Natural Language, and Video
- TensorFlow, Keras, PyTorch, and Scikit-learn on Vertex AI
- Distributed training with GPUs and TPUs
- Hyperparameter tuning with Vertex AI Vizier
- Model optimization techniques (pruning, quantization, distillation)
- Experiment tracking and TensorBoard integration
- Pre-trained models and transfer learning
- Complete training code examples
- Real-world training scenarios

## Exam Focus Areas

**For Professional ML Engineer Exam**:
1. **Framework Selection**: When to use TensorFlow vs PyTorch vs AutoML vs BigQuery ML
2. **Training Infrastructure**: GPU vs TPU, distributed strategies, cost optimization
3. **Hyperparameter Tuning**: Vizier configuration, search algorithms, early stopping
4. **Model Optimization**: Quantization, pruning, knowledge distillation for production
5. **AutoML Decision Making**: When AutoML is appropriate vs custom training
6. **Experiment Tracking**: Vertex AI Experiments, metadata management
7. **Pre-trained Models**: When to use Vision API, NLP API vs custom training
8. **Training at Scale**: Multi-worker training, TPU pods, distribution strategies

## Model Development Frameworks

### TensorFlow on Vertex AI

**Complete TensorFlow Training Script**:
```python
# trainer/task.py - Complete TensorFlow training with distributed strategy
import tensorflow as tf
from tensorflow import keras
import argparse
import os
import json

def get_args():
    """Parse command line arguments."""
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-data', type=str, required=True)
    parser.add_argument('--val-data', type=str, required=True)
    parser.add_argument('--model-dir', type=str, required=True)
    parser.add_argument('--epochs', type=int, default=10)
    parser.add_argument('--batch-size', type=int, default=32)
    parser.add_argument('--learning-rate', type=float, default=0.001)
    parser.add_argument('--hidden-units', type=str, default='128,64')
    parser.add_argument('--dropout', type=float, default=0.2)
    return parser.parse_args()

def create_model(input_dim, hidden_units, dropout_rate):
    """Create a neural network model."""
    model = keras.Sequential([
        keras.layers.Dense(hidden_units[0], activation='relu', input_dim=input_dim),
        keras.layers.BatchNormalization(),
        keras.layers.Dropout(dropout_rate),
        keras.layers.Dense(hidden_units[1], activation='relu'),
        keras.layers.BatchNormalization(),
        keras.layers.Dropout(dropout_rate),
        keras.layers.Dense(1, activation='sigmoid')
    ])
    return model

def load_data(data_path, batch_size):
    """Load and preprocess data."""
    # Load data from GCS
    dataset = tf.data.experimental.make_csv_dataset(
        data_path,
        batch_size=batch_size,
        label_name='label',
        num_epochs=1,
        ignore_errors=True
    )

    # Preprocessing
    def preprocess(features, label):
        # Normalize features
        normalized_features = {}
        for key, value in features.items():
            normalized_features[key] = tf.cast(value, tf.float32)
        return normalized_features, label

    dataset = dataset.map(preprocess)
    return dataset

def main():
    """Main training function."""
    args = get_args()

    # Parse hidden units
    hidden_units = [int(x) for x in args.hidden_units.split(',')]

    # Setup distributed strategy
    strategy = tf.distribute.MirroredStrategy()
    print(f'Number of devices: {strategy.num_replicas_in_sync}')

    # Adjust batch size for distributed training
    global_batch_size = args.batch_size * strategy.num_replicas_in_sync

    # Load data
    with strategy.scope():
        train_dataset = load_data(args.train_data, global_batch_size)
        val_dataset = load_data(args.val_data, global_batch_size)

        # Create model
        model = create_model(
            input_dim=10,  # Adjust based on your data
            hidden_units=hidden_units,
            dropout_rate=args.dropout
        )

        # Compile model
        model.compile(
            optimizer=keras.optimizers.Adam(learning_rate=args.learning_rate),
            loss='binary_crossentropy',
            metrics=[
                'accuracy',
                keras.metrics.AUC(name='auc'),
                keras.metrics.Precision(name='precision'),
                keras.metrics.Recall(name='recall')
            ]
        )

    # Callbacks
    callbacks = [
        keras.callbacks.ModelCheckpoint(
            filepath=os.path.join(args.model_dir, 'checkpoint-{epoch:02d}'),
            save_best_only=True,
            monitor='val_auc',
            mode='max'
        ),
        keras.callbacks.EarlyStopping(
            monitor='val_loss',
            patience=3,
            restore_best_weights=True
        ),
        keras.callbacks.TensorBoard(
            log_dir=os.path.join(args.model_dir, 'logs'),
            histogram_freq=1
        ),
        keras.callbacks.ReduceLROnPlateau(
            monitor='val_loss',
            factor=0.5,
            patience=2,
            min_lr=1e-7
        )
    ]

    # Train model
    history = model.fit(
        train_dataset,
        epochs=args.epochs,
        validation_data=val_dataset,
        callbacks=callbacks,
        verbose=1
    )

    # Save final model
    model.save(os.path.join(args.model_dir, 'final_model'))

    # Save training history
    with open(os.path.join(args.model_dir, 'history.json'), 'w') as f:
        json.dump(history.history, f)

    print('Training completed successfully!')

if __name__ == '__main__':
    main()
```

**Submitting TensorFlow Training Job**:
```python
from google.cloud import aiplatform

aiplatform.init(project='your-project', location='us-central1')

# Define training job
job = aiplatform.CustomTrainingJob(
    display_name='tensorflow-training',
    script_path='trainer/task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest',
    requirements=['pandas==2.0.0', 'scikit-learn==1.3.0'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/tf2-gpu.2-12:latest'
)

# Run training
model = job.run(
    replica_count=1,
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=1,
    args=[
        '--train-data=gs://bucket/train.csv',
        '--val-data=gs://bucket/val.csv',
        '--model-dir=gs://bucket/model',
        '--epochs=50',
        '--batch-size=64',
        '--learning-rate=0.001'
    ],
    environment_variables={
        'TF_GPU_THREAD_MODE': 'gpu_private',
        'TF_GPU_THREAD_COUNT': '1'
    }
)
```

### PyTorch on Vertex AI

**Complete PyTorch Training Script**:
```python
# trainer/pytorch_task.py - PyTorch training with distributed data parallel
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP
from torch.utils.data.distributed import DistributedSampler
import argparse
import os
import json

class CustomDataset(Dataset):
    """Custom PyTorch dataset."""
    def __init__(self, data_path):
        # Load data from GCS
        self.data = self.load_data(data_path)

    def load_data(self, data_path):
        # Implement data loading
        pass

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx]

class NeuralNetwork(nn.Module):
    """PyTorch neural network."""
    def __init__(self, input_dim, hidden_units, dropout_rate):
        super(NeuralNetwork, self).__init__()
        self.network = nn.Sequential(
            nn.Linear(input_dim, hidden_units[0]),
            nn.BatchNorm1d(hidden_units[0]),
            nn.ReLU(),
            nn.Dropout(dropout_rate),
            nn.Linear(hidden_units[0], hidden_units[1]),
            nn.BatchNorm1d(hidden_units[1]),
            nn.ReLU(),
            nn.Dropout(dropout_rate),
            nn.Linear(hidden_units[1], 1),
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.network(x)

def setup_distributed():
    """Setup distributed training environment."""
    if 'WORLD_SIZE' in os.environ:
        dist.init_process_group(backend='nccl')
        torch.cuda.set_device(int(os.environ['LOCAL_RANK']))
        return True
    return False

def train_epoch(model, dataloader, criterion, optimizer, device, epoch):
    """Train for one epoch."""
    model.train()
    running_loss = 0.0
    correct = 0
    total = 0

    for batch_idx, (data, target) in enumerate(dataloader):
        data, target = data.to(device), target.to(device)

        optimizer.zero_grad()
        output = model(data)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()

        running_loss += loss.item()
        predicted = (output > 0.5).float()
        total += target.size(0)
        correct += (predicted == target).sum().item()

        if batch_idx % 100 == 0:
            print(f'Epoch: {epoch} [{batch_idx * len(data)}/{len(dataloader.dataset)}] '
                  f'Loss: {loss.item():.4f} Acc: {100. * correct / total:.2f}%')

    return running_loss / len(dataloader), correct / total

def validate(model, dataloader, criterion, device):
    """Validate the model."""
    model.eval()
    val_loss = 0.0
    correct = 0
    total = 0

    with torch.no_grad():
        for data, target in dataloader:
            data, target = data.to(device), target.to(device)
            output = model(data)
            val_loss += criterion(output, target).item()
            predicted = (output > 0.5).float()
            total += target.size(0)
            correct += (predicted == target).sum().item()

    return val_loss / len(dataloader), correct / total

def main():
    """Main training function."""
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-data', type=str, required=True)
    parser.add_argument('--val-data', type=str, required=True)
    parser.add_argument('--model-dir', type=str, required=True)
    parser.add_argument('--epochs', type=int, default=10)
    parser.add_argument('--batch-size', type=int, default=32)
    parser.add_argument('--learning-rate', type=float, default=0.001)
    parser.add_argument('--hidden-units', type=str, default='128,64')
    parser.add_argument('--dropout', type=float, default=0.2)
    args = parser.parse_args()

    # Setup distributed training
    is_distributed = setup_distributed()
    local_rank = int(os.environ.get('LOCAL_RANK', 0))
    device = torch.device(f'cuda:{local_rank}' if torch.cuda.is_available() else 'cpu')

    # Parse hidden units
    hidden_units = [int(x) for x in args.hidden_units.split(',')]

    # Load datasets
    train_dataset = CustomDataset(args.train_data)
    val_dataset = CustomDataset(args.val_data)

    # Create data loaders
    train_sampler = DistributedSampler(train_dataset) if is_distributed else None
    train_loader = DataLoader(
        train_dataset,
        batch_size=args.batch_size,
        shuffle=(train_sampler is None),
        sampler=train_sampler,
        num_workers=4
    )
    val_loader = DataLoader(val_dataset, batch_size=args.batch_size, shuffle=False)

    # Create model
    model = NeuralNetwork(input_dim=10, hidden_units=hidden_units, dropout_rate=args.dropout)
    model = model.to(device)

    if is_distributed:
        model = DDP(model, device_ids=[local_rank])

    # Loss and optimizer
    criterion = nn.BCELoss()
    optimizer = optim.Adam(model.parameters(), lr=args.learning_rate)
    scheduler = optim.lr_scheduler.ReduceLROnPlateau(optimizer, mode='min', patience=2)

    # Training loop
    best_val_loss = float('inf')
    history = {'train_loss': [], 'train_acc': [], 'val_loss': [], 'val_acc': []}

    for epoch in range(args.epochs):
        if is_distributed:
            train_sampler.set_epoch(epoch)

        train_loss, train_acc = train_epoch(model, train_loader, criterion, optimizer, device, epoch)
        val_loss, val_acc = validate(model, val_loader, criterion, device)

        scheduler.step(val_loss)

        history['train_loss'].append(train_loss)
        history['train_acc'].append(train_acc)
        history['val_loss'].append(val_loss)
        history['val_acc'].append(val_acc)

        print(f'Epoch {epoch}: Train Loss: {train_loss:.4f}, Train Acc: {train_acc:.4f}, '
              f'Val Loss: {val_loss:.4f}, Val Acc: {val_acc:.4f}')

        # Save best model
        if val_loss < best_val_loss and (not is_distributed or local_rank == 0):
            best_val_loss = val_loss
            torch.save(model.state_dict(), os.path.join(args.model_dir, 'best_model.pt'))

    # Save final model and history
    if not is_distributed or local_rank == 0:
        torch.save(model.state_dict(), os.path.join(args.model_dir, 'final_model.pt'))
        with open(os.path.join(args.model_dir, 'history.json'), 'w') as f:
            json.dump(history, f)

    if is_distributed:
        dist.destroy_process_group()

if __name__ == '__main__':
    main()
```

**Submitting PyTorch Training Job**:
```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name='pytorch-training',
    script_path='trainer/pytorch_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-gpu.1-13:latest',
    requirements=['pandas', 'scikit-learn']
)

model = job.run(
    replica_count=4,  # Multi-worker training
    machine_type='n1-highmem-8',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=1,
    args=['--train-data=gs://bucket/train.csv', '--epochs=50']
)
```

### PyTorch Lightning on Vertex AI

**PyTorch Lightning Training Script**:
```python
# trainer/lightning_task.py - PyTorch Lightning with advanced features
import pytorch_lightning as pl
from pytorch_lightning.callbacks import ModelCheckpoint, EarlyStopping
from pytorch_lightning.loggers import TensorBoardLogger
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
import argparse

class LightningModel(pl.LightningModule):
    """PyTorch Lightning model."""
    def __init__(self, input_dim, hidden_units, learning_rate, dropout_rate):
        super().__init__()
        self.save_hyperparameters()

        self.model = nn.Sequential(
            nn.Linear(input_dim, hidden_units[0]),
            nn.BatchNorm1d(hidden_units[0]),
            nn.ReLU(),
            nn.Dropout(dropout_rate),
            nn.Linear(hidden_units[0], hidden_units[1]),
            nn.BatchNorm1d(hidden_units[1]),
            nn.ReLU(),
            nn.Dropout(dropout_rate),
            nn.Linear(hidden_units[1], 1),
            nn.Sigmoid()
        )

        self.criterion = nn.BCELoss()

    def forward(self, x):
        return self.model(x)

    def training_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self(x)
        loss = self.criterion(y_hat, y)
        acc = ((y_hat > 0.5).float() == y).float().mean()

        self.log('train_loss', loss, prog_bar=True)
        self.log('train_acc', acc, prog_bar=True)
        return loss

    def validation_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self(x)
        loss = self.criterion(y_hat, y)
        acc = ((y_hat > 0.5).float() == y).float().mean()

        self.log('val_loss', loss, prog_bar=True)
        self.log('val_acc', acc, prog_bar=True)
        return loss

    def configure_optimizers(self):
        optimizer = torch.optim.Adam(self.parameters(), lr=self.hparams.learning_rate)
        scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
            optimizer, mode='min', patience=2, factor=0.5
        )
        return {
            'optimizer': optimizer,
            'lr_scheduler': {
                'scheduler': scheduler,
                'monitor': 'val_loss'
            }
        }

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-data', type=str, required=True)
    parser.add_argument('--val-data', type=str, required=True)
    parser.add_argument('--model-dir', type=str, required=True)
    parser.add_argument('--epochs', type=int, default=10)
    parser.add_argument('--batch-size', type=int, default=32)
    parser.add_argument('--learning-rate', type=float, default=0.001)
    args = parser.parse_args()

    # Load data
    train_loader = DataLoader(...)  # Your data loader
    val_loader = DataLoader(...)

    # Create model
    model = LightningModel(
        input_dim=10,
        hidden_units=[128, 64],
        learning_rate=args.learning_rate,
        dropout_rate=0.2
    )

    # Callbacks
    checkpoint_callback = ModelCheckpoint(
        dirpath=args.model_dir,
        filename='model-{epoch:02d}-{val_loss:.2f}',
        monitor='val_loss',
        mode='min',
        save_top_k=3
    )

    early_stop_callback = EarlyStopping(
        monitor='val_loss',
        patience=3,
        mode='min'
    )

    # Logger
    logger = TensorBoardLogger(args.model_dir, name='lightning_logs')

    # Trainer
    trainer = pl.Trainer(
        max_epochs=args.epochs,
        accelerator='gpu',
        devices=-1,  # Use all available GPUs
        strategy='ddp',  # Distributed Data Parallel
        callbacks=[checkpoint_callback, early_stop_callback],
        logger=logger,
        precision=16  # Mixed precision training
    )

    # Train
    trainer.fit(model, train_loader, val_loader)

if __name__ == '__main__':
    main()
```

### Scikit-learn on Vertex AI

**Scikit-learn Training Script**:
```python
# trainer/sklearn_task.py - Scikit-learn training
import argparse
import joblib
import pandas as pd
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score
import json
import os

def load_data(data_path):
    """Load data from GCS."""
    df = pd.read_csv(data_path)
    X = df.drop('label', axis=1)
    y = df['label']
    return X, y

def create_model(model_type, random_state=42):
    """Create ML pipeline."""
    if model_type == 'random_forest':
        model = Pipeline([
            ('scaler', StandardScaler()),
            ('classifier', RandomForestClassifier(random_state=random_state))
        ])
        param_grid = {
            'classifier__n_estimators': [100, 200, 300],
            'classifier__max_depth': [10, 20, 30],
            'classifier__min_samples_split': [2, 5, 10],
            'classifier__min_samples_leaf': [1, 2, 4]
        }
    elif model_type == 'gradient_boosting':
        model = Pipeline([
            ('scaler', StandardScaler()),
            ('classifier', GradientBoostingClassifier(random_state=random_state))
        ])
        param_grid = {
            'classifier__n_estimators': [100, 200],
            'classifier__learning_rate': [0.01, 0.1, 0.2],
            'classifier__max_depth': [3, 5, 7],
            'classifier__subsample': [0.8, 1.0]
        }
    else:  # logistic_regression
        model = Pipeline([
            ('scaler', StandardScaler()),
            ('classifier', LogisticRegression(random_state=random_state, max_iter=1000))
        ])
        param_grid = {
            'classifier__C': [0.001, 0.01, 0.1, 1, 10],
            'classifier__penalty': ['l1', 'l2'],
            'classifier__solver': ['liblinear']
        }

    return model, param_grid

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-data', type=str, required=True)
    parser.add_argument('--val-data', type=str, required=True)
    parser.add_argument('--model-dir', type=str, required=True)
    parser.add_argument('--model-type', type=str, default='random_forest')
    parser.add_argument('--tune-hyperparameters', action='store_true')
    args = parser.parse_args()

    # Load data
    X_train, y_train = load_data(args.train_data)
    X_val, y_val = load_data(args.val_data)

    # Create model
    model, param_grid = create_model(args.model_type)

    # Hyperparameter tuning
    if args.tune_hyperparameters:
        print('Performing hyperparameter tuning...')
        grid_search = GridSearchCV(
            model, param_grid, cv=5, scoring='roc_auc', n_jobs=-1, verbose=2
        )
        grid_search.fit(X_train, y_train)
        model = grid_search.best_estimator_
        print(f'Best parameters: {grid_search.best_params_}')
        print(f'Best cross-validation score: {grid_search.best_score_:.4f}')
    else:
        model.fit(X_train, y_train)

    # Evaluate
    y_pred = model.predict(X_val)
    y_pred_proba = model.predict_proba(X_val)[:, 1]

    metrics = {
        'accuracy': model.score(X_val, y_val),
        'roc_auc': roc_auc_score(y_val, y_pred_proba),
        'classification_report': classification_report(y_val, y_pred, output_dict=True)
    }

    print(f'Validation Accuracy: {metrics["accuracy"]:.4f}')
    print(f'Validation ROC AUC: {metrics["roc_auc"]:.4f}')
    print('\nClassification Report:')
    print(classification_report(y_val, y_pred))

    # Save model
    model_path = os.path.join(args.model_dir, 'model.joblib')
    joblib.dump(model, model_path)

    # Save metrics
    with open(os.path.join(args.model_dir, 'metrics.json'), 'w') as f:
        json.dump(metrics, f)

    print(f'Model saved to {model_path}')

if __name__ == '__main__':
    main()
```

### Custom Container Training

**Dockerfile for Custom Training**:
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy training code
COPY trainer/ ./trainer/

# Set entrypoint
ENTRYPOINT ["python", "trainer/task.py"]
```

**Building and Using Custom Container**:
```bash
# Build container
docker build -t gcr.io/PROJECT_ID/custom-trainer:v1 .

# Push to Container Registry
docker push gcr.io/PROJECT_ID/custom-trainer:v1

# Use custom container in training job
```

```python
from google.cloud import aiplatform

job = aiplatform.CustomContainerTrainingJob(
    display_name='custom-container-training',
    container_uri='gcr.io/PROJECT_ID/custom-trainer:v1',
    model_serving_container_image_uri='gcr.io/PROJECT_ID/predictor:v1'
)

model = job.run(
    replica_count=1,
    machine_type='n1-standard-8',
    args=['--train-data=gs://bucket/train.csv', '--epochs=50']
)
```

## Vertex AI Platform

### Workbench (Notebooks)

**Managed Notebooks**:
```bash
# Create managed notebook with GPU
gcloud notebooks instances create ml-notebook \
  --vm-image-project=deeplearning-platform-release \
  --vm-image-family=tf-latest-gpu \
  --machine-type=n1-standard-8 \
  --accelerator-type=NVIDIA_TESLA_T4 \
  --accelerator-core-count=1 \
  --location=us-central1-a \
  --install-gpu-driver

# Create notebook with custom environment
gcloud notebooks instances create ml-notebook-custom \
  --vm-image-project=deeplearning-platform-release \
  --vm-image-family=pytorch-latest-gpu \
  --machine-type=n1-highmem-8 \
  --location=us-central1-a \
  --metadata='proxy-mode=service_account'
```

**User-Managed Notebooks**:
```bash
# More control, manual updates, custom configuration
gcloud compute instances create ml-notebook-user \
  --zone=us-central1-a \
  --machine-type=n1-standard-8 \
  --image-family=common-cpu \
  --image-project=deeplearning-platform-release \
  --maintenance-policy=TERMINATE \
  --boot-disk-size=100GB
```

**Notebook Features**:
- Integrated development environment
- Pre-installed ML frameworks
- Git integration
- TensorBoard integration
- Direct access to GCS, BigQuery
- Scheduled execution
- Dual authoring (Notebook and Python script)

### Custom Training Jobs

**Basic Training Job**:
```python
from google.cloud import aiplatform

aiplatform.init(project='PROJECT_ID', location='us-central1')

job = aiplatform.CustomTrainingJob(
    display_name='custom-training',
    script_path='trainer/task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest',
    requirements=['pandas==2.0.0', 'scikit-learn==1.3.0'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/tf2-gpu.2-12:latest'
)

model = job.run(
    replica_count=1,
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=1,
    args=['--epochs=50', '--batch-size=64'],
    environment_variables={'TF_GPU_THREAD_MODE': 'gpu_private'}
)
```

## Distributed Training

### Multi-GPU Training with TensorFlow

**TensorFlow MirroredStrategy (Single Node, Multiple GPUs)**:
```python
# trainer/multi_gpu_task.py
import tensorflow as tf
from tensorflow import keras
import os

def main():
    # MirroredStrategy for single-machine, multi-GPU training
    strategy = tf.distribute.MirroredStrategy()
    print(f'Number of devices: {strategy.num_replicas_in_sync}')

    # Global batch size = per-replica batch size * number of replicas
    per_replica_batch_size = 64
    global_batch_size = per_replica_batch_size * strategy.num_replicas_in_sync

    # Load and prepare data
    train_dataset = load_dataset('gs://bucket/train/*')
    train_dataset = train_dataset.batch(global_batch_size)

    # Create model within strategy scope
    with strategy.scope():
        model = keras.Sequential([
            keras.layers.Dense(128, activation='relu', input_shape=(784,)),
            keras.layers.Dropout(0.2),
            keras.layers.Dense(10, activation='softmax')
        ])

        model.compile(
            optimizer=keras.optimizers.Adam(learning_rate=0.001),
            loss='sparse_categorical_crossentropy',
            metrics=['accuracy']
        )

    # Train
    model.fit(train_dataset, epochs=10)

    # Save model
    model.save(os.path.join(os.environ['AIP_MODEL_DIR'], 'saved_model'))

if __name__ == '__main__':
    main()
```

**Submit Multi-GPU Training Job**:
```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name='multi-gpu-training',
    script_path='trainer/multi_gpu_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest'
)

model = job.run(
    replica_count=1,
    machine_type='n1-standard-16',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=4,  # 4 GPUs on single machine
    base_output_dir='gs://bucket/output'
)
```

### Multi-Worker Training with TensorFlow

**TensorFlow MultiWorkerMirroredStrategy**:
```python
# trainer/multi_worker_task.py
import tensorflow as tf
import json
import os

def main():
    # Setup TF_CONFIG for multi-worker training
    tf_config = json.loads(os.environ.get('TF_CONFIG', '{}'))

    # MultiWorkerMirroredStrategy for multi-machine training
    strategy = tf.distribute.MultiWorkerMirroredStrategy()
    print(f'Number of devices: {strategy.num_replicas_in_sync}')

    # Adjust batch size for distributed training
    per_replica_batch_size = 64
    global_batch_size = per_replica_batch_size * strategy.num_replicas_in_sync

    # Load data with sharding
    options = tf.data.Options()
    options.experimental_distribute.auto_shard_policy = \
        tf.data.experimental.AutoShardPolicy.DATA

    train_dataset = tf.data.Dataset.list_files('gs://bucket/train/*')
    train_dataset = train_dataset.with_options(options)
    train_dataset = train_dataset.batch(global_batch_size)

    # Create and compile model within strategy scope
    with strategy.scope():
        model = create_model()
        model.compile(
            optimizer='adam',
            loss='sparse_categorical_crossentropy',
            metrics=['accuracy']
        )

    # Callbacks for multi-worker training
    callbacks = []

    # Only save model on chief worker
    if tf_config.get('task', {}).get('type') == 'chief' or \
       tf_config.get('task', {}).get('type') is None:
        callbacks.append(
            tf.keras.callbacks.ModelCheckpoint(
                filepath=os.path.join(os.environ['AIP_MODEL_DIR'], 'checkpoint')
            )
        )

    # Train
    model.fit(train_dataset, epochs=10, callbacks=callbacks)

    # Save final model (only on chief)
    if tf_config.get('task', {}).get('type') == 'chief' or \
       tf_config.get('task', {}).get('type') is None:
        model.save(os.path.join(os.environ['AIP_MODEL_DIR'], 'final_model'))

if __name__ == '__main__':
    main()
```

**Submit Multi-Worker Training Job**:
```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name='multi-worker-training',
    script_path='trainer/multi_worker_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest'
)

# Replica count includes chief + workers
# 1 chief + 3 workers = 4 total replicas
model = job.run(
    replica_count=4,
    machine_type='n1-highmem-8',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=1,  # 1 GPU per worker
    base_output_dir='gs://bucket/output',
    environment_variables={
        'TF_CONFIG': 'auto'  # Vertex AI sets this automatically
    }
)
```

### PyTorch Distributed Training

**PyTorch DistributedDataParallel**:
```python
# trainer/pytorch_ddp_task.py
import torch
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP
from torch.utils.data.distributed import DistributedSampler
import os

def setup_distributed():
    """Initialize distributed training environment."""
    # Vertex AI sets these environment variables
    rank = int(os.environ.get('RANK', 0))
    local_rank = int(os.environ.get('LOCAL_RANK', 0))
    world_size = int(os.environ.get('WORLD_SIZE', 1))
    master_addr = os.environ.get('MASTER_ADDR', 'localhost')
    master_port = os.environ.get('MASTER_PORT', '12355')

    # Initialize process group
    dist.init_process_group(
        backend='nccl',
        init_method=f'tcp://{master_addr}:{master_port}',
        rank=rank,
        world_size=world_size
    )

    # Set device
    torch.cuda.set_device(local_rank)

    return rank, local_rank, world_size

def cleanup_distributed():
    """Clean up distributed training."""
    dist.destroy_process_group()

def main():
    # Setup distributed environment
    rank, local_rank, world_size = setup_distributed()
    device = torch.device(f'cuda:{local_rank}')

    # Create model and move to device
    model = MyModel().to(device)
    model = DDP(model, device_ids=[local_rank])

    # Create distributed sampler
    train_dataset = MyDataset()
    train_sampler = DistributedSampler(
        train_dataset,
        num_replicas=world_size,
        rank=rank,
        shuffle=True
    )

    # Create data loader
    train_loader = torch.utils.data.DataLoader(
        train_dataset,
        batch_size=64,
        sampler=train_sampler,
        num_workers=4,
        pin_memory=True
    )

    # Optimizer
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

    # Training loop
    for epoch in range(10):
        train_sampler.set_epoch(epoch)  # Important for shuffling

        model.train()
        for batch_idx, (data, target) in enumerate(train_loader):
            data, target = data.to(device), target.to(device)

            optimizer.zero_grad()
            output = model(data)
            loss = torch.nn.functional.cross_entropy(output, target)
            loss.backward()
            optimizer.step()

            if rank == 0 and batch_idx % 100 == 0:
                print(f'Epoch: {epoch}, Batch: {batch_idx}, Loss: {loss.item()}')

    # Save model (only on rank 0)
    if rank == 0:
        torch.save(model.state_dict(), 'model.pt')

    cleanup_distributed()

if __name__ == '__main__':
    main()
```

**Submit PyTorch DDP Training**:
```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name='pytorch-ddp-training',
    script_path='trainer/pytorch_ddp_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-gpu.1-13:latest'
)

model = job.run(
    replica_count=4,  # 4 workers
    machine_type='n1-highmem-8',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=1,
    base_output_dir='gs://bucket/output'
)
```

## TPU Training

### TensorFlow TPU Training

**TPU Strategy Training Script**:
```python
# trainer/tpu_task.py
import tensorflow as tf
from tensorflow import keras
import os

def main():
    # Detect TPUs
    try:
        tpu = tf.distribute.cluster_resolver.TPUClusterResolver.connect()
        print(f'Running on TPU: {tpu.cluster_spec().as_dict()["worker"]}')
        strategy = tf.distribute.TPUStrategy(tpu)
    except ValueError:
        print('Running on CPU/GPU')
        strategy = tf.distribute.get_strategy()

    print(f'Number of accelerators: {strategy.num_replicas_in_sync}')

    # Batch size per TPU core
    per_replica_batch_size = 128
    global_batch_size = per_replica_batch_size * strategy.num_replicas_in_sync

    # Load and prepare data
    def load_dataset():
        # Load from GCS
        dataset = tf.data.TFRecordDataset('gs://bucket/train.tfrecord')
        dataset = dataset.map(parse_example)
        dataset = dataset.shuffle(10000)
        dataset = dataset.batch(global_batch_size, drop_remainder=True)
        dataset = dataset.prefetch(tf.data.AUTOTUNE)
        return dataset

    train_dataset = load_dataset()

    # Create model within TPU strategy scope
    with strategy.scope():
        model = keras.Sequential([
            keras.layers.Conv2D(32, 3, activation='relu', input_shape=(224, 224, 3)),
            keras.layers.MaxPooling2D(),
            keras.layers.Conv2D(64, 3, activation='relu'),
            keras.layers.MaxPooling2D(),
            keras.layers.Conv2D(128, 3, activation='relu'),
            keras.layers.GlobalAveragePooling2D(),
            keras.layers.Dense(256, activation='relu'),
            keras.layers.Dropout(0.5),
            keras.layers.Dense(10, activation='softmax')
        ])

        # Use appropriate learning rate for TPU
        learning_rate = 0.001 * strategy.num_replicas_in_sync

        model.compile(
            optimizer=keras.optimizers.Adam(learning_rate=learning_rate),
            loss='sparse_categorical_crossentropy',
            metrics=['accuracy']
        )

    # Callbacks
    callbacks = [
        keras.callbacks.TensorBoard(log_dir=os.environ['AIP_TENSORBOARD_LOG_DIR']),
        keras.callbacks.ModelCheckpoint(
            filepath=os.path.join(os.environ['AIP_MODEL_DIR'], 'checkpoint'),
            save_best_only=True
        )
    ]

    # Train on TPU
    model.fit(
        train_dataset,
        epochs=10,
        callbacks=callbacks,
        steps_per_epoch=1000
    )

    # Save model
    model.save(os.path.join(os.environ['AIP_MODEL_DIR'], 'final_model'))

if __name__ == '__main__':
    main()
```

**Submit TPU Training Job**:
```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name='tpu-training',
    script_path='trainer/tpu_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-cpu.2-12:latest'
)

# TPU v3-8 = 8 TPU cores (4 TPU chips)
model = job.run(
    replica_count=1,
    machine_type='cloud-tpu',
    accelerator_type='TPU_V3',
    accelerator_count=8,  # 8 cores
    base_output_dir='gs://bucket/output'
)
```

**TPU Pod Training (Large Scale)**:
```python
# For TPU v3-32 (32 cores, 4 hosts with 8 cores each)
model = job.run(
    replica_count=4,  # 4 TPU hosts
    machine_type='cloud-tpu',
    accelerator_type='TPU_V3',
    accelerator_count=8,  # 8 cores per host
    base_output_dir='gs://bucket/output'
)

# Total: 4 hosts * 8 cores = 32 TPU cores (TPU v3-32)
```

### PyTorch XLA for TPU

**PyTorch TPU Training**:
```python
# trainer/pytorch_tpu_task.py
import torch
import torch_xla
import torch_xla.core.xla_model as xm
import torch_xla.distributed.parallel_loader as pl
import torch_xla.distributed.xla_multiprocessing as xmp

def train_loop(index):
    """Training loop for each TPU core."""
    # Get TPU device
    device = xm.xla_device()

    # Create model and move to TPU
    model = MyModel().to(device)

    # Wrap model for multi-core training
    model = xmp.MpModelWrapper(model)

    # Data loader
    train_loader = create_data_loader()

    # Wrap data loader for TPU
    para_loader = pl.ParallelLoader(train_loader, [device])

    # Optimizer
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

    # Training loop
    for epoch in range(10):
        for batch_idx, (data, target) in enumerate(para_loader.per_device_loader(device)):
            optimizer.zero_grad()
            output = model(data)
            loss = torch.nn.functional.cross_entropy(output, target)
            loss.backward()

            # Synchronize gradients across TPU cores
            xm.optimizer_step(optimizer)

            if batch_idx % 100 == 0:
                xm.master_print(f'Epoch: {epoch}, Batch: {batch_idx}, Loss: {loss.item()}')

    # Save model (only on master)
    if xm.is_master_ordinal():
        torch.save(model.state_dict(), 'model.pt')

def main():
    # Spawn training on all TPU cores
    xmp.spawn(train_loop, nprocs=8)  # 8 cores for TPU v3-8

if __name__ == '__main__':
    main()
```

**Submit PyTorch TPU Job**:
```python
job = aiplatform.CustomTrainingJob(
    display_name='pytorch-tpu-training',
    script_path='trainer/pytorch_tpu_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-xla.1-13:latest'
)

model = job.run(
    replica_count=1,
    machine_type='cloud-tpu',
    accelerator_type='TPU_V3',
    accelerator_count=8
)
```

## Hyperparameter Tuning with Vertex AI Vizier

### Complete Hyperparameter Tuning Example

**Training Script for Hyperparameter Tuning**:
```python
# trainer/hp_tuning_task.py
import tensorflow as tf
from tensorflow import keras
import argparse
import hypertune
import os

def get_args():
    """Parse hyperparameters from command line."""
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-data', type=str, required=True)
    parser.add_argument('--val-data', type=str, required=True)
    parser.add_argument('--model-dir', type=str, required=True)
    parser.add_argument('--epochs', type=int, default=10)

    # Hyperparameters to tune
    parser.add_argument('--learning-rate', type=float, default=0.001)
    parser.add_argument('--batch-size', type=int, default=32)
    parser.add_argument('--num-layers', type=int, default=2)
    parser.add_argument('--units-per-layer', type=int, default=128)
    parser.add_argument('--dropout', type=float, default=0.2)
    parser.add_argument('--optimizer', type=str, default='adam')

    return parser.parse_args()

def create_model(num_layers, units_per_layer, dropout_rate, optimizer, learning_rate):
    """Create model with tunable architecture."""
    layers = [keras.layers.Flatten(input_shape=(28, 28))]

    for i in range(num_layers):
        layers.append(keras.layers.Dense(units_per_layer, activation='relu'))
        layers.append(keras.layers.BatchNormalization())
        layers.append(keras.layers.Dropout(dropout_rate))

    layers.append(keras.layers.Dense(10, activation='softmax'))

    model = keras.Sequential(layers)

    # Select optimizer
    if optimizer == 'adam':
        opt = keras.optimizers.Adam(learning_rate=learning_rate)
    elif optimizer == 'sgd':
        opt = keras.optimizers.SGD(learning_rate=learning_rate, momentum=0.9)
    else:
        opt = keras.optimizers.RMSprop(learning_rate=learning_rate)

    model.compile(
        optimizer=opt,
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )

    return model

def main():
    args = get_args()

    # Load data
    (x_train, y_train), (x_val, y_val) = tf.keras.datasets.mnist.load_data()
    x_train = x_train / 255.0
    x_val = x_val / 255.0

    # Create model
    model = create_model(
        num_layers=args.num_layers,
        units_per_layer=args.units_per_layer,
        dropout_rate=args.dropout,
        optimizer=args.optimizer,
        learning_rate=args.learning_rate
    )

    # Train
    history = model.fit(
        x_train, y_train,
        batch_size=args.batch_size,
        epochs=args.epochs,
        validation_data=(x_val, y_val),
        verbose=1
    )

    # Get final validation accuracy
    val_accuracy = history.history['val_accuracy'][-1]

    # Report metric to Vertex AI for hyperparameter tuning
    hpt = hypertune.HyperTune()
    hpt.report_hyperparameter_tuning_metric(
        hyperparameter_metric_tag='accuracy',
        metric_value=val_accuracy,
        global_step=args.epochs
    )

    # Save model
    model.save(os.path.join(args.model_dir, 'model'))

if __name__ == '__main__':
    main()
```

**Submit Hyperparameter Tuning Job**:
```python
from google.cloud import aiplatform
from google.cloud.aiplatform import hyperparameter_tuning as hpt

aiplatform.init(project='your-project', location='us-central1')

# Create custom training job
job = aiplatform.CustomTrainingJob(
    display_name='hp-tuning-job',
    script_path='trainer/hp_tuning_task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest',
    requirements=['cloudml-hypertune'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/tf2-gpu.2-12:latest'
)

# Define hyperparameter tuning job
hp_job = aiplatform.HyperparameterTuningJob(
    display_name='mnist-hp-tuning',
    custom_job=job,
    metric_spec={
        'accuracy': 'maximize',  # Metric to optimize
    },
    parameter_spec={
        'learning_rate': hpt.DoubleParameterSpec(
            min=0.0001, max=0.1, scale='log'
        ),
        'batch_size': hpt.DiscreteParameterSpec(
            values=[32, 64, 128], scale='linear'
        ),
        'num_layers': hpt.IntegerParameterSpec(
            min=1, max=4, scale='linear'
        ),
        'units_per_layer': hpt.DiscreteParameterSpec(
            values=[64, 128, 256], scale='linear'
        ),
        'dropout': hpt.DoubleParameterSpec(
            min=0.1, max=0.5, scale='linear'
        ),
        'optimizer': hpt.CategoricalParameterSpec(
            values=['adam', 'sgd', 'rmsprop']
        )
    },
    max_trial_count=30,
    parallel_trial_count=5,
    max_failed_trial_count=5,
    search_algorithm='bayesian',  # 'random', 'grid', or 'bayesian'
    measurement_selection='best'  # Use best measurement for early stopping
)

# Run hyperparameter tuning
hp_job.run(
    machine_type='n1-standard-4',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1
)

# Get best trial
best_trial = hp_job.trials[0]
print(f'Best trial: {best_trial.id}')
print(f'Best accuracy: {best_trial.final_measurement.metrics[0].value}')
print(f'Best hyperparameters: {best_trial.parameters}')
```

**Advanced Hyperparameter Tuning with Early Stopping**:
```python
# Configure early stopping
hp_job = aiplatform.HyperparameterTuningJob(
    display_name='advanced-hp-tuning',
    custom_job=job,
    metric_spec={
        'accuracy': 'maximize',
        'loss': 'minimize'  # Can optimize multiple metrics
    },
    parameter_spec={
        'learning_rate': hpt.DoubleParameterSpec(min=0.0001, max=0.1, scale='log'),
        'batch_size': hpt.DiscreteParameterSpec(values=[32, 64, 128]),
        'num_layers': hpt.IntegerParameterSpec(min=1, max=4)
    },
    max_trial_count=50,
    parallel_trial_count=10,
    search_algorithm='bayesian',
    # Early stopping configuration
    measurement_selection='best',
    enable_trial_early_stopping=True
)

hp_job.run(
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_V100',
    accelerator_count=1,
    timeout=7200  # 2 hours timeout
)
```

**Conditional Hyperparameter Tuning**:
```python
# Tune different parameters based on model type
from google.cloud.aiplatform import hyperparameter_tuning as hpt

# Parameter spec with conditional parameters
parameter_spec = {
    'model_type': hpt.CategoricalParameterSpec(values=['cnn', 'rnn', 'transformer']),
    # CNN-specific parameters
    'num_conv_layers': hpt.IntegerParameterSpec(min=1, max=5, scale='linear',
                                                 parent_values={'model_type': ['cnn']}),
    'kernel_size': hpt.DiscreteParameterSpec(values=[3, 5, 7],
                                             parent_values={'model_type': ['cnn']}),
    # RNN-specific parameters
    'rnn_units': hpt.IntegerParameterSpec(min=64, max=512, scale='linear',
                                          parent_values={'model_type': ['rnn']}),
    'rnn_type': hpt.CategoricalParameterSpec(values=['lstm', 'gru'],
                                             parent_values={'model_type': ['rnn']}),
    # Common parameters
    'learning_rate': hpt.DoubleParameterSpec(min=0.0001, max=0.1, scale='log'),
    'dropout': hpt.DoubleParameterSpec(min=0.1, max=0.5, scale='linear')
}

hp_job = aiplatform.HyperparameterTuningJob(
    display_name='conditional-hp-tuning',
    custom_job=job,
    metric_spec={'accuracy': 'maximize'},
    parameter_spec=parameter_spec,
    max_trial_count=40,
    parallel_trial_count=8,
    search_algorithm='bayesian'
)

hp_job.run()
```

**Retrieve and Analyze Results**:
```python
# Get tuning job results
hp_job = aiplatform.HyperparameterTuningJob.get('hp-job-id')

# Get all trials
trials = hp_job.trials

# Sort by accuracy
sorted_trials = sorted(trials,
                       key=lambda x: x.final_measurement.metrics[0].value,
                       reverse=True)

# Best trial
best_trial = sorted_trials[0]
print(f'Best Trial ID: {best_trial.id}')
print(f'Best Accuracy: {best_trial.final_measurement.metrics[0].value}')
print('Best Hyperparameters:')
for param in best_trial.parameters:
    print(f'  {param.parameter_id}: {param.value}')

# Analyze hyperparameter importance
import pandas as pd
import matplotlib.pyplot as plt

# Create dataframe of trials
trial_data = []
for trial in trials:
    params = {p.parameter_id: p.value for p in trial.parameters}
    params['accuracy'] = trial.final_measurement.metrics[0].value
    trial_data.append(params)

df = pd.DataFrame(trial_data)

# Correlation analysis
correlations = df.corr()['accuracy'].drop('accuracy').sort_values(ascending=False)
print('\nHyperparameter Correlations with Accuracy:')
print(correlations)
```

## AutoML - Automated Machine Learning

### When to Use AutoML vs Custom Training

**Use AutoML when**:
- Quick proof of concept needed
- Limited ML expertise in team
- Standard problems (classification, regression, object detection)
- Need baseline model quickly
- Data is clean and well-structured
- Budget and time constraints

**Use Custom Training when**:
- Complex model architectures required
- Need fine-grained control over training
- Using specialized loss functions
- Implementing research papers
- Need specific optimization strategies
- Working with unique data formats

### AutoML Tables (Structured Data)

**Complete AutoML Tables Example - Classification**:
```python
from google.cloud import aiplatform

aiplatform.init(project='your-project', location='us-central1')

# Create tabular dataset
dataset = aiplatform.TabularDataset.create(
    display_name='customer-churn-dataset',
    gcs_source='gs://bucket/customer_data.csv',
    labels={'env': 'prod', 'team': 'ml'}
)

# Create AutoML training job
job = aiplatform.AutoMLTabularTrainingJob(
    display_name='churn-prediction-automl',
    optimization_prediction_type='classification',
    optimization_objective='maximize-au-prc',  # Options: maximize-au-roc, maximize-au-prc,
                                                 # maximize-log-loss, maximize-precision-at-recall,
                                                 # maximize-recall-at-precision
    # Column specifications
    column_specs={
        'customer_id': 'auto',  # Let AutoML detect
        'age': 'numeric',
        'gender': 'categorical',
        'account_balance': 'numeric',
        'signup_date': 'timestamp',
        'last_purchase': 'timestamp',
        'email': 'text',
        'country': 'categorical',
        'churned': 'categorical'  # Target
    },
    # Advanced settings
    predefined_split_column_name=None,  # Use 'split' column if you have one
    weight_column_name=None,  # For weighted samples
    budget_milli_node_hours=1000,  # 1 node hour = 1000 milli node hours
    model_type='CLOUD',  # or 'MOBILE_TF_LITE' for mobile deployment
    disable_early_stopping=False
)

# Run training
model = job.run(
    dataset=dataset,
    target_column='churned',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    model_display_name='churn-prediction-model',
    sync=True  # Wait for completion
)

# Get model evaluation
model_evaluation = model.list_model_evaluations()[0]
print(f'Model Evaluation:')
print(f'  AU-ROC: {model_evaluation.metrics["auRoc"]}')
print(f'  AU-PRC: {model_evaluation.metrics["auPrc"]}')
print(f'  Log Loss: {model_evaluation.metrics["logLoss"]}')

# Feature importance
feature_attributions = model_evaluation.metrics.get('featureAttributions', [])
for feature in feature_attributions[:10]:
    print(f'{feature["featureName"]}: {feature["attributionValue"]}')
```

**AutoML Tables - Regression**:
```python
# Regression problem
job = aiplatform.AutoMLTabularTrainingJob(
    display_name='price-prediction',
    optimization_prediction_type='regression',
    optimization_objective='minimize-rmse',  # minimize-mae, minimize-rmsle
    column_specs={
        'square_feet': 'numeric',
        'bedrooms': 'numeric',
        'location': 'categorical',
        'year_built': 'numeric',
        'price': 'numeric'  # Target
    }
)

model = job.run(
    dataset=dataset,
    target_column='price',
    budget_milli_node_hours=1000,
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)

# Get regression metrics
evaluation = model.list_model_evaluations()[0]
print(f'RMSE: {evaluation.metrics["rootMeanSquaredError"]}')
print(f'MAE: {evaluation.metrics["meanAbsoluteError"]}')
print(f'R²: {evaluation.metrics["rSquared"]}')
```

**AutoML Tables with Custom Split**:
```python
# Dataset with predefined split column
# CSV format: feature1,feature2,...,target,split
# split values: 'TRAIN', 'VALIDATION', 'TEST'

dataset = aiplatform.TabularDataset.create(
    display_name='custom-split-dataset',
    gcs_source='gs://bucket/data_with_split.csv'
)

job = aiplatform.AutoMLTabularTrainingJob(
    display_name='custom-split-training',
    optimization_prediction_type='classification',
    optimization_objective='maximize-au-roc',
    predefined_split_column_name='split'  # Use custom split
)

model = job.run(
    dataset=dataset,
    target_column='label',
    model_display_name='custom-split-model'
)
```

**AutoML Tables - Budget Recommendations**:
```
Budget Guidelines (milli node hours):
- Quick experiment: 1000 (1 hour)
- Small dataset (<1GB): 1000-3000 (1-3 hours)
- Medium dataset (1-10GB): 3000-8000 (3-8 hours)
- Large dataset (>10GB): 8000-20000 (8-20 hours)
- Production model: 10000-100000 (10-100 hours)

Note: More budget allows AutoML to try more algorithms and ensembles
```

### AutoML Vision

**Image Classification - Single Label**:
```python
# Prepare image dataset
# CSV format: gs://bucket/image.jpg,label
# Or JSONL format for more metadata

dataset = aiplatform.ImageDataset.create(
    display_name='product-classification',
    gcs_source='gs://bucket/image_data.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.single_label_classification
)

# Create AutoML Vision training job
job = aiplatform.AutoMLImageTrainingJob(
    display_name='product-classifier',
    prediction_type='classification',
    multi_label=False,
    model_type='CLOUD',  # or 'CLOUD_1', 'MOBILE_TF_LOW_LATENCY_1',
                         # 'MOBILE_TF_VERSATILE_1', 'MOBILE_TF_HIGH_ACCURACY_1'
    base_model=None  # Optional: Use pre-trained model
)

model = job.run(
    dataset=dataset,
    model_display_name='product-classifier',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=8000,  # Recommended: 8000-20000 for vision
    disable_early_stopping=False
)

# Evaluate model
evaluation = model.list_model_evaluations()[0]
print(f'Confidence Threshold Metrics:')
for threshold_metric in evaluation.metrics['confidenceMetrics']:
    print(f"Threshold: {threshold_metric['confidenceThreshold']}")
    print(f"  Precision: {threshold_metric['precision']}")
    print(f"  Recall: {threshold_metric['recall']}")
    print(f"  F1: {threshold_metric['f1Score']}")

# Confusion matrix
confusion_matrix = evaluation.metrics['confusionMatrix']
print(f'\nConfusion Matrix:')
print(confusion_matrix)
```

**Image Classification - Multi-Label**:
```python
# Multi-label: Each image can have multiple labels
# CSV format: gs://bucket/image.jpg,label1,label2,label3

dataset = aiplatform.ImageDataset.create(
    display_name='multi-label-classification',
    gcs_source='gs://bucket/multi_label_data.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.multi_label_classification
)

job = aiplatform.AutoMLImageTrainingJob(
    display_name='multi-label-classifier',
    prediction_type='classification',
    multi_label=True  # Enable multi-label
)

model = job.run(
    dataset=dataset,
    budget_milli_node_hours=10000
)
```

**Object Detection**:
```python
# Object detection dataset
# JSONL format with bounding boxes
# {
#   "imageGcsUri": "gs://bucket/image.jpg",
#   "boundingBoxAnnotations": [
#     {"displayName": "cat", "xMin": 0.1, "yMin": 0.2, "xMax": 0.5, "yMax": 0.8},
#     {"displayName": "dog", "xMin": 0.6, "yMin": 0.3, "xMax": 0.9, "yMax": 0.7}
#   ]
# }

dataset = aiplatform.ImageDataset.create(
    display_name='object-detection',
    gcs_source='gs://bucket/object_detection.jsonl',
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.bounding_box
)

job = aiplatform.AutoMLImageTrainingJob(
    display_name='object-detector',
    prediction_type='object_detection',
    model_type='CLOUD_HIGH_ACCURACY_1'  # or 'CLOUD_LOW_LATENCY_1', 'MOBILE_TF_LOW_LATENCY_1'
)

model = job.run(
    dataset=dataset,
    budget_milli_node_hours=20000,  # Object detection needs more budget
    model_display_name='object-detector-model'
)

# Evaluate object detection
evaluation = model.list_model_evaluations()[0]
print(f'Mean Average Precision (mAP): {evaluation.metrics["meanAveragePrecision"]}')
```

**Image Segmentation**:
```python
# Image segmentation dataset
dataset = aiplatform.ImageDataset.create(
    display_name='image-segmentation',
    gcs_source='gs://bucket/segmentation.jsonl',
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.image_segmentation
)

job = aiplatform.AutoMLImageTrainingJob(
    display_name='segmentation-model',
    prediction_type='image_segmentation',
    model_type='CLOUD_HIGH_ACCURACY_1'
)

model = job.run(
    dataset=dataset,
    budget_milli_node_hours=24000
)
```

### AutoML Natural Language

**Text Classification**:
```python
# Text classification dataset
# CSV format: text,label
# Or JSONL: {"textContent": "...", "classificationAnnotation": {"displayName": "label"}}

dataset = aiplatform.TextDataset.create(
    display_name='sentiment-analysis',
    gcs_source='gs://bucket/reviews.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.text.single_label_classification
)

job = aiplatform.AutoMLTextTrainingJob(
    display_name='sentiment-classifier',
    prediction_type='classification',
    multi_label=False
)

model = job.run(
    dataset=dataset,
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    model_display_name='sentiment-model'
)

# Evaluate
evaluation = model.list_model_evaluations()[0]
print(f'AU-ROC: {evaluation.metrics["auRoc"]}')
print(f'AU-PRC: {evaluation.metrics["auPrc"]}')
```

**Multi-Label Text Classification**:
```python
# Multi-label text classification
dataset = aiplatform.TextDataset.create(
    display_name='topic-classification',
    gcs_source='gs://bucket/topics.jsonl',
    import_schema_uri=aiplatform.schema.dataset.ioformat.text.multi_label_classification
)

job = aiplatform.AutoMLTextTrainingJob(
    display_name='topic-classifier',
    prediction_type='classification',
    multi_label=True
)

model = job.run(dataset=dataset)
```

**Entity Extraction**:
```python
# Named entity recognition dataset
# JSONL format with text spans and entity types

dataset = aiplatform.TextDataset.create(
    display_name='entity-extraction',
    gcs_source='gs://bucket/entities.jsonl',
    import_schema_uri=aiplatform.schema.dataset.ioformat.text.extraction
)

job = aiplatform.AutoMLTextTrainingJob(
    display_name='entity-extractor',
    prediction_type='extraction'
)

model = job.run(
    dataset=dataset,
    model_display_name='entity-extraction-model'
)
```

**Sentiment Analysis**:
```python
# Sentiment analysis (specific type of classification)
dataset = aiplatform.TextDataset.create(
    display_name='product-sentiment',
    gcs_source='gs://bucket/product_reviews.csv'
)

job = aiplatform.AutoMLTextTrainingJob(
    display_name='sentiment-analyzer',
    prediction_type='sentiment',
    sentiment_max=10  # Sentiment scale (e.g., 0-10)
)

model = job.run(dataset=dataset)
```

### AutoML Video

**Video Classification**:
```python
# Video classification dataset
# CSV format: gs://bucket/video.mp4,label,start_time,end_time

dataset = aiplatform.VideoDataset.create(
    display_name='action-recognition',
    gcs_source='gs://bucket/videos.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.video.classification
)

job = aiplatform.AutoMLVideoTrainingJob(
    display_name='action-classifier',
    prediction_type='classification',
    model_type='CLOUD'
)

model = job.run(
    dataset=dataset,
    training_fraction_split=0.8,
    test_fraction_split=0.2,
    model_display_name='action-recognition-model'
)
```

**Video Object Tracking**:
```python
# Video object tracking
dataset = aiplatform.VideoDataset.create(
    display_name='object-tracking',
    gcs_source='gs://bucket/tracking.jsonl',
    import_schema_uri=aiplatform.schema.dataset.ioformat.video.object_tracking
)

job = aiplatform.AutoMLVideoTrainingJob(
    display_name='object-tracker',
    prediction_type='object_tracking',
    model_type='CLOUD'
)

model = job.run(dataset=dataset)
```

**Video Action Recognition**:
```python
# Action recognition in videos
dataset = aiplatform.VideoDataset.create(
    display_name='action-recognition',
    gcs_source='gs://bucket/actions.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.video.action_recognition
)

job = aiplatform.AutoMLVideoTrainingJob(
    display_name='action-recognizer',
    prediction_type='action_recognition',
    model_type='CLOUD'
)

model = job.run(dataset=dataset)
```

### AutoML Model Export

**Export AutoML Model for Edge Deployment**:
```python
# Train model for mobile/edge deployment
job = aiplatform.AutoMLImageTrainingJob(
    display_name='mobile-classifier',
    prediction_type='classification',
    model_type='MOBILE_TF_LOW_LATENCY_1'  # Mobile-optimized model
)

model = job.run(dataset=dataset, budget_milli_node_hours=8000)

# Export model for TensorFlow Lite
model.export_model(
    export_format_id='tf-saved-model',  # or 'tflite', 'edgetpu-tflite', 'tf-js'
    artifact_destination='gs://bucket/exported_model/',
    sync=True
)

# Export for Core ML (iOS)
model.export_model(
    export_format_id='core-ml',
    artifact_destination='gs://bucket/coreml_model/'
)
```

## Experiment Tracking and TensorBoard

### Vertex AI Experiments

**Complete Experiment Tracking Example**:
```python
from google.cloud import aiplatform
import tensorflow as tf

aiplatform.init(
    project='your-project',
    location='us-central1',
    experiment='image-classification-experiments'
)

# Start experiment run
aiplatform.start_run(run='run-001')

# Log parameters
aiplatform.log_params({
    'learning_rate': 0.001,
    'batch_size': 32,
    'num_layers': 3,
    'dropout': 0.2,
    'optimizer': 'adam'
})

# Train model
model = create_and_train_model()

# Log metrics
aiplatform.log_metrics({
    'accuracy': 0.95,
    'loss': 0.12,
    'val_accuracy': 0.93,
    'val_loss': 0.15,
    'auc': 0.97
})

# Log model
aiplatform.log_model(model, 'trained-model')

# Log time series metrics during training
for epoch in range(10):
    train_loss, train_acc = train_one_epoch()
    aiplatform.log_time_series_metrics({
        'train_loss': train_loss,
        'train_accuracy': train_acc
    })

# End run
aiplatform.end_run()

# Compare runs
df = aiplatform.get_experiment_df()
print(df.sort_values('metric.accuracy', ascending=False))
```

**TensorBoard Integration**:
```python
# trainer/task.py with TensorBoard logging
import tensorflow as tf
from tensorflow import keras
import os

def main():
    # TensorBoard callback
    tensorboard_callback = keras.callbacks.TensorBoard(
        log_dir=os.environ.get('AIP_TENSORBOARD_LOG_DIR', './logs'),
        histogram_freq=1,
        write_graph=True,
        write_images=True,
        update_freq='epoch',
        profile_batch='10,20',  # Profile batches 10-20
        embeddings_freq=1
    )

    # Custom TensorBoard logging
    file_writer = tf.summary.create_file_writer(
        os.environ.get('AIP_TENSORBOARD_LOG_DIR') + '/custom'
    )

    model = create_model()
    model.compile(optimizer='adam', loss='mse')

    # Log custom metrics
    for epoch in range(10):
        for batch, (x, y) in enumerate(train_dataset):
            loss = model.train_on_batch(x, y)

            with file_writer.as_default():
                tf.summary.scalar('batch_loss', loss, step=epoch * len(train_dataset) + batch)

                # Log images
                if batch % 100 == 0:
                    tf.summary.image('training_images', x[:4], step=batch)

                # Log histograms
                for layer in model.layers:
                    for weight in layer.weights:
                        tf.summary.histogram(weight.name, weight, step=batch)

    model.fit(train_dataset, epochs=10, callbacks=[tensorboard_callback])
```

**Create TensorBoard Instance**:
```python
from google.cloud import aiplatform

# Create TensorBoard instance
tensorboard = aiplatform.Tensorboard.create(
    display_name='my-tensorboard',
    labels={'env': 'prod'}
)

# Use TensorBoard with training
job = aiplatform.CustomTrainingJob(
    display_name='training-with-tensorboard',
    script_path='trainer/task.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-gpu.2-12:latest'
)

model = job.run(
    tensorboard=tensorboard.resource_name,
    service_account='sa@project.iam.gserviceaccount.com'
)
```

## Pre-trained Models and Transfer Learning

### Vision API (Pre-trained Models)

**When to use Vision API vs Custom Training**:
- Vision API: Standard tasks (label detection, face detection, OCR, logo detection), no training data
- AutoML Vision: Custom labels, moderate dataset (100+ images per label)
- Custom Training: Complex architectures, large datasets, specific requirements

**Vision API Usage**:
```python
from google.cloud import vision

client = vision.ImageAnnotatorClient()

# Image from GCS
image = vision.Image()
image.source.image_uri = 'gs://bucket/image.jpg'

# Label detection
response = client.label_detection(image=image)
labels = response.label_annotations
for label in labels:
    print(f'{label.description}: {label.score}')

# Object localization
response = client.object_localization(image=image)
objects = response.localized_object_annotations
for obj in objects:
    print(f'{obj.name}: {obj.score}')
    print(f'Bounds: {obj.bounding_poly}')

# Face detection
response = client.face_detection(image=image)
faces = response.face_annotations

# OCR (Text detection)
response = client.text_detection(image=image)
texts = response.text_annotations

# Landmark detection
response = client.landmark_detection(image=image)

# Logo detection
response = client.logo_detection(image=image)
```

### Natural Language API

**Text Analysis with Pre-trained Models**:
```python
from google.cloud import language_v1

client = language_v1.LanguageServiceClient()

# Analyze sentiment
document = language_v1.Document(
    content='I love this product! It is amazing.',
    type_=language_v1.Document.Type.PLAIN_TEXT
)
sentiment = client.analyze_sentiment(document=document).document_sentiment
print(f'Sentiment score: {sentiment.score}')  # -1.0 to 1.0
print(f'Sentiment magnitude: {sentiment.magnitude}')  # 0.0+

# Entity analysis
entities = client.analyze_entities(document=document).entities
for entity in entities:
    print(f'Entity: {entity.name}')
    print(f'Type: {entity.type_.name}')
    print(f'Salience: {entity.salience}')

# Syntax analysis
syntax = client.analyze_syntax(document=document)
for token in syntax.tokens:
    print(f'{token.text.content}: {token.part_of_speech.tag.name}')

# Entity sentiment
entity_sentiment = client.analyze_entity_sentiment(document=document)

# Content classification
categories = client.classify_text(document=document).categories
for category in categories:
    print(f'{category.name}: {category.confidence}')
```

### Speech-to-Text API

**Speech Recognition**:
```python
from google.cloud import speech

client = speech.SpeechClient()

# Audio from GCS
audio = speech.RecognitionAudio(uri='gs://bucket/audio.wav')

config = speech.RecognitionConfig(
    encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
    sample_rate_hertz=16000,
    language_code='en-US',
    enable_automatic_punctuation=True,
    enable_word_time_offsets=True,
    enable_word_confidence=True,
    model='default'  # or 'phone_call', 'video', 'command_and_search'
)

# Synchronous recognition (< 60 seconds)
response = client.recognize(config=config, audio=audio)

# Long audio (> 60 seconds)
operation = client.long_running_recognize(config=config, audio=audio)
response = operation.result(timeout=300)

for result in response.results:
    print(f'Transcript: {result.alternatives[0].transcript}')
    print(f'Confidence: {result.alternatives[0].confidence}')
```

### Transfer Learning with Pre-trained Models

**TensorFlow Hub Models**:
```python
import tensorflow as tf
import tensorflow_hub as hub

# Load pre-trained model from TensorFlow Hub
mobilenet = hub.KerasLayer(
    'https://tfhub.dev/google/tf2-preview/mobilenet_v2/feature_vector/4',
    trainable=False,  # Freeze base model
    input_shape=(224, 224, 3)
)

# Build model with transfer learning
model = tf.keras.Sequential([
    mobilenet,
    tf.keras.layers.Dropout(0.5),
    tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Train only the top layers
model.fit(train_dataset, epochs=5)

# Fine-tune: Unfreeze some layers
mobilenet.trainable = True
model.compile(
    optimizer=tf.keras.optimizers.Adam(1e-5),  # Lower learning rate
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Fine-tune the model
model.fit(train_dataset, epochs=10)
```

**BERT for NLP**:
```python
import tensorflow as tf
import tensorflow_hub as hub
import tensorflow_text as text

# Load BERT preprocessor and encoder
preprocessor = hub.KerasLayer(
    'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3'
)
encoder = hub.KerasLayer(
    'https://tfhub.dev/tensorflow/bert_en_uncased_L-12_H-768_A-12/4',
    trainable=True
)

# Build text classification model
text_input = tf.keras.layers.Input(shape=(), dtype=tf.string)
preprocessed_text = preprocessor(text_input)
outputs = encoder(preprocessed_text)

# Use pooled_output for classification
pooled_output = outputs['pooled_output']
dropout = tf.keras.layers.Dropout(0.1)(pooled_output)
output = tf.keras.layers.Dense(2, activation='softmax')(dropout)

model = tf.keras.Model(inputs=text_input, outputs=output)

model.compile(
    optimizer=tf.keras.optimizers.Adam(2e-5),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

model.fit(train_dataset, epochs=3)
```

## Model Optimization Techniques

### Quantization

**Post-Training Quantization**:
```python
import tensorflow as tf

# Load trained model
model = tf.keras.models.load_model('saved_model/')

# Convert to TensorFlow Lite with default quantization
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model/')
converter.optimizations = [tf.lite.Optimize.DEFAULT]
quantized_tflite_model = converter.convert()

# Save quantized model
with open('model_quantized.tflite', 'wb') as f:
    f.write(quantized_tflite_model)

# Full integer quantization (requires representative dataset)
def representative_dataset():
    for data in train_dataset.take(100):
        yield [tf.cast(data, tf.float32)]

converter.representative_dataset = representative_dataset
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.int8
converter.inference_output_type = tf.int8
int8_tflite_model = converter.convert()

# Compare model sizes
import os
original_size = os.path.getsize('model.tflite')
quantized_size = os.path.getsize('model_quantized.tflite')
print(f'Size reduction: {(1 - quantized_size/original_size) * 100:.2f}%')
```

**Quantization-Aware Training**:
```python
import tensorflow as tf
import tensorflow_model_optimization as tfmot

# Create model
model = create_model()

# Apply quantization-aware training
quantize_model = tfmot.quantization.keras.quantize_model

q_aware_model = quantize_model(model)

# Compile and train
q_aware_model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

q_aware_model.fit(train_dataset, epochs=10, validation_data=val_dataset)

# Convert to TFLite
converter = tf.lite.TFLiteConverter.from_keras_model(q_aware_model)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
quantized_model = converter.convert()
```

### Model Pruning

**Magnitude-Based Pruning**:
```python
import tensorflow_model_optimization as tfmot

# Define pruning schedule
pruning_params = {
    'pruning_schedule': tfmot.sparsity.keras.PolynomialDecay(
        initial_sparsity=0.0,
        final_sparsity=0.5,  # 50% of weights will be zero
        begin_step=1000,
        end_step=5000,
        frequency=100
    )
}

# Apply pruning to model
model = create_model()
pruned_model = tfmot.sparsity.keras.prune_low_magnitude(model, **pruning_params)

# Compile with pruning callbacks
pruned_model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Train with pruning callbacks
callbacks = [
    tfmot.sparsity.keras.UpdatePruningStep(),
    tfmot.sparsity.keras.PruningSummaries(log_dir='./logs')
]

pruned_model.fit(
    train_dataset,
    epochs=10,
    validation_data=val_dataset,
    callbacks=callbacks
)

# Strip pruning wrappers for export
final_model = tfmot.sparsity.keras.strip_pruning(pruned_model)
final_model.save('pruned_model')

# Convert to TFLite
converter = tf.lite.TFLiteConverter.from_keras_model(final_model)
converter.optimizations = [tf.lite.Optimize.DEFAULT]
pruned_tflite_model = converter.convert()
```

**Layer-Specific Pruning**:
```python
def apply_pruning_to_dense(layer):
    """Apply pruning only to Dense layers."""
    if isinstance(layer, tf.keras.layers.Dense):
        return tfmot.sparsity.keras.prune_low_magnitude(layer, **pruning_params)
    return layer

model = tf.keras.models.clone_model(
    original_model,
    clone_function=apply_pruning_to_dense
)
```

### Knowledge Distillation

**Complete Knowledge Distillation Example**:
```python
import tensorflow as tf

class Distiller(tf.keras.Model):
    """Knowledge distillation wrapper."""
    def __init__(self, student, teacher):
        super().__init__()
        self.teacher = teacher
        self.student = student

    def compile(self, optimizer, metrics, student_loss_fn, distillation_loss_fn,
                alpha=0.1, temperature=3):
        super().compile(optimizer=optimizer, metrics=metrics)
        self.student_loss_fn = student_loss_fn
        self.distillation_loss_fn = distillation_loss_fn
        self.alpha = alpha
        self.temperature = temperature

    def train_step(self, data):
        x, y = data

        # Forward pass of teacher
        teacher_predictions = self.teacher(x, training=False)

        with tf.GradientTape() as tape:
            # Forward pass of student
            student_predictions = self.student(x, training=True)

            # Compute losses
            student_loss = self.student_loss_fn(y, student_predictions)

            # Distillation loss (soft targets)
            distillation_loss = self.distillation_loss_fn(
                tf.nn.softmax(teacher_predictions / self.temperature, axis=1),
                tf.nn.softmax(student_predictions / self.temperature, axis=1)
            )

            # Combined loss
            loss = self.alpha * student_loss + (1 - self.alpha) * distillation_loss

        # Compute gradients
        trainable_vars = self.student.trainable_variables
        gradients = tape.gradient(loss, trainable_vars)

        # Update weights
        self.optimizer.apply_gradients(zip(gradients, trainable_vars))

        # Update metrics
        self.compiled_metrics.update_state(y, student_predictions)

        # Return metrics
        results = {m.name: m.result() for m in self.metrics}
        results.update({'student_loss': student_loss, 'distillation_loss': distillation_loss})
        return results

# Load teacher model (large, pre-trained)
teacher = tf.keras.models.load_model('large_teacher_model')
teacher.trainable = False

# Create smaller student model
student = create_small_model()

# Create distiller
distiller = Distiller(student=student, teacher=teacher)

distiller.compile(
    optimizer=tf.keras.optimizers.Adam(),
    metrics=[tf.keras.metrics.SparseCategoricalAccuracy()],
    student_loss_fn=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),
    distillation_loss_fn=tf.keras.losses.KLDivergence(),
    alpha=0.1,
    temperature=10
)

# Train student
distiller.fit(train_dataset, epochs=10, validation_data=val_dataset)

# Extract student model
student_model = distiller.student
student_model.save('distilled_student_model')
```

### TensorFlow Lite Conversion

**Complete TFLite Conversion with Optimization**:
```python
import tensorflow as tf

# Load model
model = tf.keras.models.load_model('saved_model/')

# Basic conversion
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model/')
tflite_model = converter.convert()

# Optimized conversion with all techniques
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model/')

# Enable all optimizations
converter.optimizations = [tf.lite.Optimize.DEFAULT]

# Representative dataset for quantization
def representative_dataset():
    for data in train_dataset.take(100):
        yield [tf.cast(data[0], tf.float32)]

converter.representative_dataset = representative_dataset

# Integer-only quantization
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.uint8
converter.inference_output_type = tf.uint8

# Convert
optimized_tflite_model = converter.convert()

# Save model
with open('model_optimized.tflite', 'wb') as f:
    f.write(optimized_tflite_model)

# Inference with TFLite
interpreter = tf.lite.Interpreter(model_content=optimized_tflite_model)
interpreter.allocate_tensors()

input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# Run inference
input_data = tf.constant([[1.0, 2.0, 3.0]], dtype=tf.float32)
interpreter.set_tensor(input_details[0]['index'], input_data)
interpreter.invoke()
output_data = interpreter.get_tensor(output_details[0]['index'])
```

### Mixed Precision Training

**Automatic Mixed Precision**:
```python
import tensorflow as tf

# Enable mixed precision
policy = tf.keras.mixed_precision.Policy('mixed_float16')
tf.keras.mixed_precision.set_global_policy(policy)

# Create model (automatically uses mixed precision)
model = create_model()

# Compile with loss scaling
optimizer = tf.keras.optimizers.Adam()
optimizer = tf.keras.mixed_precision.LossScaleOptimizer(optimizer)

model.compile(
    optimizer=optimizer,
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Train (automatically faster with mixed precision)
model.fit(train_dataset, epochs=10)
```

## Training Scenarios and Complete Examples

### Scenario 1: Large-Scale Image Classification with TPU

**Problem**: Train ResNet50 on ImageNet-like dataset with 1M images

```python
# trainer/imagenet_tpu.py
import tensorflow as tf
from tensorflow import keras
import os

def create_dataset(file_pattern, batch_size, is_training=True):
    """Create TFRecord dataset."""
    dataset = tf.data.TFRecordDataset.list_files(file_pattern)

    def parse_example(example):
        features = tf.io.parse_single_example(
            example,
            features={
                'image': tf.io.FixedLenFeature([], tf.string),
                'label': tf.io.FixedLenFeature([], tf.int64)
            }
        )
        image = tf.io.decode_jpeg(features['image'], channels=3)
        image = tf.image.resize(image, [224, 224])
        image = tf.cast(image, tf.float32) / 255.0
        return image, features['label']

    dataset = dataset.map(parse_example, num_parallel_calls=tf.data.AUTOTUNE)

    if is_training:
        dataset = dataset.shuffle(10000)
        dataset = dataset.repeat()

    dataset = dataset.batch(batch_size, drop_remainder=True)
    dataset = dataset.prefetch(tf.data.AUTOTUNE)
    return dataset

def main():
    # TPU setup
    resolver = tf.distribute.cluster_resolver.TPUClusterResolver()
    tf.config.experimental_connect_to_cluster(resolver)
    tf.tpu.experimental.initialize_tpu_system(resolver)
    strategy = tf.distribute.TPUStrategy(resolver)

    # Batch size scaled for TPU
    per_replica_batch_size = 128
    global_batch_size = per_replica_batch_size * strategy.num_replicas_in_sync

    # Create datasets
    train_dataset = create_dataset('gs://bucket/train/*.tfrecord', global_batch_size)
    val_dataset = create_dataset('gs://bucket/val/*.tfrecord', global_batch_size, False)

    # Create model with TPU strategy
    with strategy.scope():
        base_model = keras.applications.ResNet50(
            weights='imagenet',
            include_top=False,
            input_shape=(224, 224, 3)
        )

        model = keras.Sequential([
            base_model,
            keras.layers.GlobalAveragePooling2D(),
            keras.layers.Dense(1000, activation='softmax')
        ])

        # Scale learning rate
        learning_rate = 0.1 * (global_batch_size / 256)

        model.compile(
            optimizer=keras.optimizers.SGD(learning_rate=learning_rate, momentum=0.9),
            loss='sparse_categorical_crossentropy',
            metrics=['accuracy', 'top_k_categorical_accuracy']
        )

    # Train
    model.fit(
        train_dataset,
        epochs=90,
        steps_per_epoch=1000,
        validation_data=val_dataset,
        validation_steps=100,
        callbacks=[
            keras.callbacks.LearningRateScheduler(lambda epoch: learning_rate * (0.1 ** (epoch // 30))),
            keras.callbacks.TensorBoard(log_dir=os.environ['AIP_TENSORBOARD_LOG_DIR'])
        ]
    )

    model.save(os.path.join(os.environ['AIP_MODEL_DIR'], 'resnet50_model'))

if __name__ == '__main__':
    main()
```

**Submit TPU Training**:
```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name='imagenet-resnet50-tpu',
    script_path='trainer/imagenet_tpu.py',
    container_uri='gcr.io/cloud-aiplatform/training/tf-cpu.2-12:latest'
)

model = job.run(
    replica_count=1,
    machine_type='cloud-tpu',
    accelerator_type='TPU_V3',
    accelerator_count=8,
    base_output_dir='gs://bucket/models/imagenet'
)
```

### Scenario 2: NLP Sentiment Analysis with BERT Fine-Tuning

**Complete BERT Fine-Tuning Pipeline**:
```python
# trainer/bert_sentiment.py
import tensorflow as tf
import tensorflow_hub as hub
import tensorflow_text as text
import argparse

def create_bert_model(num_classes):
    """Create BERT model for classification."""
    text_input = tf.keras.layers.Input(shape=(), dtype=tf.string, name='text')

    preprocessor = hub.KerasLayer(
        'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3'
    )
    encoder = hub.KerasLayer(
        'https://tfhub.dev/tensorflow/bert_en_uncased_L-12_H-768_A-12/4',
        trainable=True,
        name='bert'
    )

    encoder_inputs = preprocessor(text_input)
    outputs = encoder(encoder_inputs)
    pooled_output = outputs['pooled_output']

    dropout = tf.keras.layers.Dropout(0.1)(pooled_output)
    output = tf.keras.layers.Dense(num_classes, activation='softmax')(dropout)

    model = tf.keras.Model(inputs=text_input, outputs=output)
    return model

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--train-data', type=str, required=True)
    parser.add_argument('--val-data', type=str, required=True)
    parser.add_argument('--model-dir', type=str, required=True)
    parser.add_argument('--epochs', type=int, default=3)
    parser.add_argument('--batch-size', type=int, default=32)
    args = parser.parse_args()

    # Load data
    train_df = pd.read_csv(args.train_data)
    val_df = pd.read_csv(args.val_data)

    train_dataset = tf.data.Dataset.from_tensor_slices((
        train_df['text'].values,
        train_df['label'].values
    )).batch(args.batch_size)

    val_dataset = tf.data.Dataset.from_tensor_slices((
        val_df['text'].values,
        val_df['label'].values
    )).batch(args.batch_size)

    # Create model
    model = create_bert_model(num_classes=2)

    # Compile
    model.compile(
        optimizer=tf.keras.optimizers.Adam(learning_rate=2e-5),
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )

    # Train
    history = model.fit(
        train_dataset,
        epochs=args.epochs,
        validation_data=val_dataset,
        callbacks=[
            tf.keras.callbacks.EarlyStopping(patience=2),
            tf.keras.callbacks.ModelCheckpoint(
                filepath=args.model_dir + '/checkpoint',
                save_best_only=True
            )
        ]
    )

    # Save model
    model.save(args.model_dir + '/bert_sentiment_model')

if __name__ == '__main__':
    main()
```

### Scenario 3: Time Series Forecasting with LSTM

**Complete LSTM Training for Time Series**:
```python
# trainer/lstm_forecast.py
import tensorflow as tf
from tensorflow import keras
import numpy as np

def create_sequences(data, seq_length):
    """Create sequences for LSTM."""
    X, y = [], []
    for i in range(len(data) - seq_length):
        X.append(data[i:i+seq_length])
        y.append(data[i+seq_length])
    return np.array(X), np.array(y)

def create_lstm_model(seq_length, n_features):
    """Create LSTM model."""
    model = keras.Sequential([
        keras.layers.LSTM(128, return_sequences=True, input_shape=(seq_length, n_features)),
        keras.layers.Dropout(0.2),
        keras.layers.LSTM(64, return_sequences=True),
        keras.layers.Dropout(0.2),
        keras.layers.LSTM(32),
        keras.layers.Dropout(0.2),
        keras.layers.Dense(1)
    ])
    return model

def main():
    # Load time series data
    data = load_data('gs://bucket/timeseries.csv')

    # Prepare sequences
    seq_length = 60
    X, y = create_sequences(data, seq_length)

    # Split data
    split = int(0.8 * len(X))
    X_train, X_test = X[:split], X[split:]
    y_train, y_test = y[:split], y[split:]

    # Create model
    model = create_lstm_model(seq_length, n_features=1)

    # Compile
    model.compile(
        optimizer='adam',
        loss='mse',
        metrics=['mae']
    )

    # Train
    history = model.fit(
        X_train, y_train,
        epochs=50,
        batch_size=64,
        validation_data=(X_test, y_test),
        callbacks=[
            keras.callbacks.EarlyStopping(patience=5),
            keras.callbacks.ReduceLROnPlateau(patience=3)
        ]
    )

    model.save('lstm_forecast_model')

if __name__ == '__main__':
    main()
```

## Exam Tips and Decision Framework

### Framework Selection Decision Tree

```
Is the problem standard (classification, regression, object detection)?
├─ YES: Do you have labeled data?
│  ├─ YES: Is data size < 100MB and task is tabular?
│  │  └─ Use BigQuery ML
│  ├─ YES: Is data size < 10GB and need quick results?
│  │  └─ Use AutoML
│  └─ NO: Use pre-trained APIs (Vision, NLP, Speech)
└─ NO: Need custom architecture?
   └─ Use Custom Training (TensorFlow/PyTorch)

Need to scale training?
├─ Large model (>1B parameters): Use TPU Pods
├─ Medium model (<1B parameters): Use Multi-GPU
└─ Small model: Single GPU or CPU

Need model optimization?
├─ Mobile deployment: Quantization + TFLite
├─ Edge deployment: Pruning + Quantization
├─ Fast inference: TensorRT optimization
└─ Cost reduction: Knowledge distillation
```

### Professional ML Engineer Exam Tips

**Training Infrastructure Choices**:
1. **CPU**: Simple models, small datasets, prototyping
2. **Single GPU (T4/V100)**: Most deep learning tasks, standard training
3. **Multi-GPU (4x V100)**: Large batches, faster training
4. **TPU v2/v3**: TensorFlow models, very large batches, matrix operations
5. **TPU Pods**: Massive scale training (ImageNet, BERT-large)

**AutoML Budget Guidelines (Exam)**:
- Tables: 1000-10000 milli node hours
- Vision (classification): 8000-20000 milli node hours
- Vision (object detection): 20000-50000 milli node hours
- NLP: 3000-10000 milli node hours
- Video: 10000-50000 milli node hours

**Hyperparameter Tuning Decisions**:
- **Grid Search**: Small parameter space, exhaustive search needed
- **Random Search**: Moderate parameter space, exploration
- **Bayesian (Vizier)**: Large parameter space, expensive training, best performance

**When to Use Each Service**:
- **BigQuery ML**: SQL users, data already in BigQuery, simple models
- **AutoML**: Quick POC, no ML expertise, standard problems
- **Vertex AI Training**: Custom code, specific requirements, research
- **Pre-trained APIs**: No training data, standard tasks, immediate results

**Cost Optimization Tips**:
1. Use preemptible instances for fault-tolerant training
2. Start with AutoML for baseline, then custom if needed
3. Use appropriate accelerators (don't over-provision)
4. Implement checkpointing for long-running jobs
5. Use hyperparameter tuning with early stopping
6. Optimize data pipelines to reduce I/O bottlenecks

**Common Exam Scenarios**:
1. **Scenario**: Need to classify 100K images into 50 categories
   - **Answer**: AutoML Vision (8000 milli node hours budget)

2. **Scenario**: Train BERT-large model on custom dataset
   - **Answer**: Vertex AI Custom Training with TPU v3-8

3. **Scenario**: Real-time inference on mobile devices
   - **Answer**: Train with AutoML MOBILE_TF_LOW_LATENCY_1, export to TFLite

4. **Scenario**: Tabular data in BigQuery, SQL team
   - **Answer**: BigQuery ML for training and prediction

5. **Scenario**: Need to reduce model size by 80% for edge deployment
   - **Answer**: Quantization + Pruning + Knowledge Distillation

6. **Scenario**: Train on proprietary data that cannot leave premises
   - **Answer**: Not covered by Professional ML Engineer (use on-premises solutions)

7. **Scenario**: Optimize hyperparameters for complex model
   - **Answer**: Vertex AI Hyperparameter Tuning with Bayesian search

8. **Scenario**: Standard object detection (cars, people, animals)
   - **Answer**: Vision API object localization (no training needed)

## BigQuery ML

### Model Creation
```sql
-- Logistic regression
CREATE OR REPLACE MODEL `dataset.churn_model`
OPTIONS(
  model_type='LOGISTIC_REG',
  input_label_cols=['churned'],
  max_iterations=10,
  l1_reg=0.1,
  l2_reg=0.1
) AS
SELECT
  age,
  gender,
  subscription_months,
  support_tickets,
  churned
FROM
  `dataset.customers`
WHERE
  split_field != 'TEST';

-- XGBoost
CREATE OR REPLACE MODEL `dataset.xgboost_model`
OPTIONS(
  model_type='BOOSTED_TREE_CLASSIFIER',
  input_label_cols=['label'],
  max_iterations=50,
  booster_type='GBTREE',
  tree_method='HIST',
  max_depth=6,
  subsample=0.8,
  colsample_bytree=0.8
) AS
SELECT * FROM `dataset.training_data`;

-- Deep Neural Network
CREATE OR REPLACE MODEL `dataset.dnn_model`
OPTIONS(
  model_type='DNN_CLASSIFIER',
  hidden_units=[128, 64, 32],
  activation_fn='RELU',
  dropout=0.2,
  batch_size=32,
  max_iterations=100,
  early_stop=TRUE
) AS
SELECT * FROM `dataset.training_data`;
```

### Model Evaluation
```sql
SELECT *
FROM ML.EVALUATE(MODEL `dataset.churn_model`,
  (SELECT * FROM `dataset.customers` WHERE split_field = 'TEST'));

-- Feature importance
SELECT *
FROM ML.FEATURE_IMPORTANCE(MODEL `dataset.churn_model`);

-- Confusion matrix
SELECT *
FROM ML.CONFUSION_MATRIX(MODEL `dataset.churn_model`,
  (SELECT * FROM `dataset.test_data`));
```

### Predictions
```sql
-- Batch prediction
SELECT
  customer_id,
  predicted_churned,
  predicted_churned_probs
FROM
  ML.PREDICT(MODEL `dataset.churn_model`,
    (SELECT * FROM `dataset.new_customers`));

-- Explain predictions
SELECT *
FROM ML.EXPLAIN_PREDICT(MODEL `dataset.churn_model`,
  (SELECT * FROM `dataset.test_data` LIMIT 10));
```

## Feature Engineering

### Feature Store
```python
from google.cloud import aiplatform

# Create feature store
featurestore = aiplatform.Featurestore.create(
    featurestore_id='customer_features',
    online_store_fixed_node_count=1
)

# Create entity type
entity_type = featurestore.create_entity_type(
    entity_type_id='customer',
    description='Customer features'
)

# Create features
entity_type.create_feature(
    feature_id='age',
    value_type='INT64'
)
entity_type.create_feature(
    feature_id='total_purchases',
    value_type='DOUBLE'
)

# Ingest features
entity_type.ingest_from_bq(
    feature_ids=['age', 'total_purchases'],
    feature_time='feature_timestamp',
    bq_source_uri='bq://project.dataset.table'
)

# Online serving
features = entity_type.read(entity_ids=['customer_123'])
```

### Feature Preprocessing
```python
import tensorflow as tf
from tensorflow import feature_column

# Numeric features
age = feature_column.numeric_column('age')
income = feature_column.numeric_column('income', normalizer_fn=lambda x: x / 100000)

# Categorical features
gender = feature_column.categorical_column_with_vocabulary_list(
    'gender', ['M', 'F', 'Other'])
gender_embedding = feature_column.embedding_column(gender, dimension=3)

# Bucketized features
age_buckets = feature_column.bucketized_column(
    age, boundaries=[18, 25, 35, 45, 55, 65])

# Crossed features
age_gender_cross = feature_column.crossed_column(
    [age_buckets, gender], hash_bucket_size=1000)

# Feature layer
feature_layer = tf.keras.layers.DenseFeatures([
    age, income, gender_embedding, age_buckets
])
```

## Model Optimization

### Quantization
```python
import tensorflow as tf

# Post-training quantization
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model/')
converter.optimizations = [tf.lite.Optimize.DEFAULT]
quantized_model = converter.convert()

# Quantization-aware training
model = tf.keras.models.Sequential([...])
model = tfmot.quantization.keras.quantize_model(model)
model.compile(...)
model.fit(...)
```

### Model Pruning
```python
import tensorflow_model_optimization as tfmot

pruning_params = {
    'pruning_schedule': tfmot.sparsity.keras.PolynomialDecay(
        initial_sparsity=0.0,
        final_sparsity=0.5,
        begin_step=1000,
        end_step=5000
    )
}

model = tfmot.sparsity.keras.prune_low_magnitude(model, **pruning_params)
model.compile(...)
model.fit(...)
```

## Best Practices for Production ML

### Training Optimization
1. **Data Pipeline**: Use TFRecord, implement prefetching, parallelize data loading
2. **Batch Size**: Use powers of 2, scale with number of replicas
3. **Learning Rate**: Scale with batch size, use warmup for large batches
4. **Checkpointing**: Save every N steps, keep best models only
5. **Early Stopping**: Prevent overfitting, save training costs
6. **Mixed Precision**: Enable for faster training on modern GPUs
7. **Distributed Training**: Use appropriate strategy for infrastructure
8. **Experiment Tracking**: Log all parameters, metrics, and artifacts

### AutoML Best Practices
1. **Data Quality**: Clean data thoroughly, balance classes, handle missing values
2. **Budget Management**: Start small for testing, increase for production
3. **Model Selection**: Use appropriate model type for deployment target
4. **Export Strategy**: Export for custom serving when needed
5. **Monitoring**: Track AutoML model performance over time
6. **Iteration**: Use AutoML for baseline, custom training for optimization

### Cost Optimization
1. **Use Preemptible VMs**: 70% cost savings for fault-tolerant training
2. **Right-size Infrastructure**: Don't over-provision resources
3. **Implement Checkpointing**: Enable resumable training
4. **Optimize Data Pipelines**: Reduce training time
5. **Use Early Stopping**: In hyperparameter tuning
6. **Clean Up Resources**: Delete unused models and datasets
7. **Monitor Spending**: Set budgets and alerts

### Security Best Practices
1. **VPC Service Controls**: Isolate training data
2. **Encryption**: CMEK for data at rest
3. **IAM**: Minimal permissions for service accounts
4. **Private IP**: For training jobs
5. **Audit Logs**: Track all access
6. **Data Residency**: Ensure compliance requirements

## Study Tips for Professional ML Engineer Exam

### Hands-On Practice Required
1. **Vertex AI Custom Training**: TensorFlow, PyTorch, custom containers
2. **AutoML**: Tables, Vision, NLP models
3. **Hyperparameter Tuning**: Configure Vizier, analyze results
4. **TPU Training**: Set up TPU jobs, use TPUStrategy
5. **Model Optimization**: Quantization, pruning, distillation
6. **BigQuery ML**: Create and evaluate models with SQL

### Key Decision Frameworks to Master
1. **Service Selection**: AutoML vs Custom vs BigQuery ML vs Pre-trained APIs
2. **Hardware Selection**: CPU vs GPU vs TPU, when to use each
3. **Distributed Training**: Which strategy for which scenario
4. **Hyperparameter Tuning**: Grid vs Random vs Bayesian search
5. **Model Optimization**: Which techniques for which deployment targets
6. **Cost Optimization**: When to use preemptible VMs, spot pricing

### Common Exam Scenarios
1. Image classification with limited ML expertise - AutoML Vision
2. Train BERT-large on custom data - TPU v3 with Custom Training
3. Mobile deployment with size constraints - Quantization + TFLite
4. SQL team with BigQuery data - BigQuery ML
5. Multi-machine multi-GPU training - MultiWorkerMirroredStrategy
6. Standard object detection - Vision API (no training)
7. Optimize 10+ hyperparameters - Bayesian with Vizier
8. Reduce model size 80% - Quantization + Pruning + Distillation

### Quick Reference Tables

**AutoML Budget Guidelines**:
- Tables: 1000-10000 milli node hours
- Vision Classification: 8000-20000 milli node hours
- Object Detection: 20000-50000 milli node hours
- NLP: 3000-10000 milli node hours
- Video: 10000-50000 milli node hours

**Hardware Costs (approximate)**:
- CPU: $0.05-0.10/hour
- GPU T4: $0.35-0.40/hour
- GPU V100: $2.48/hour
- GPU A100: $3.67/hour
- TPU v2: $4.50/hour
- TPU v3: $8.00/hour

**When to Use Each Service**:
- BigQuery ML: SQL users, data in BigQuery, simple models
- AutoML: Quick POC, limited expertise, standard problems
- Custom Training: Specific requirements, complex architectures, research
- Pre-trained APIs: No training data, standard tasks, immediate results

## Additional Resources

- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [AutoML Documentation](https://cloud.google.com/automl/docs)
- [BigQuery ML](https://cloud.google.com/bigquery-ml/docs)
- [ML Best Practices](https://cloud.google.com/architecture/ml-on-gcp-best-practices)
