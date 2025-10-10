# OCI Architect Professional - Advanced Services and Optimization

## Table of Contents
- [Container and Kubernetes](#container-and-kubernetes)
- [Serverless Architecture](#serverless-architecture)
- [Data and Analytics](#data-and-analytics)
- [Machine Learning and AI](#machine-learning-and-ai)
- [Cost Optimization](#cost-optimization)
- [Performance Optimization](#performance-optimization)
- [Exam Tips](#exam-tips)

---

## Container and Kubernetes

### OKE Advanced Architecture

**Production-Grade OKE Cluster**:
```
OKE Cluster (Regional)
├── Control Plane (Oracle-Managed)
│   ├── HA across 3 ADs
│   ├── Kubernetes API Server
│   ├── etcd (distributed config)
│   └── Controller Manager
├── Worker Nodes (Customer-Managed)
│   ├── Node Pool 1: General workloads (E4.Flex)
│   ├── Node Pool 2: GPU workloads (GPU shapes)
│   └── Node Pool 3: Memory-intensive (E4.Flex high RAM)
└── Add-ons
    ├── Kubernetes Dashboard
    ├── Metrics Server
    ├── Cluster Autoscaler
    └── OCI Cloud Controller Manager
```

**Multi-Node Pool Strategy**:
```terraform
# General purpose node pool
resource "oci_containerengine_node_pool" "general" {
  cluster_id         = oci_containerengine_cluster.main.id
  compartment_id     = var.compartment_id
  kubernetes_version = "v1.28.2"
  name               = "general-pool"

  node_config_details {
    placement_configs {
      availability_domain = data.oci_identity_availability_domain.ad1.name
      subnet_id           = oci_core_subnet.workers_ad1.id
    }
    placement_configs {
      availability_domain = data.oci_identity_availability_domain.ad2.name
      subnet_id           = oci_core_subnet.workers_ad2.id
    }
    placement_configs {
      availability_domain = data.oci_identity_availability_domain.ad3.name
      subnet_id           = oci_core_subnet.workers_ad3.id
    }

    size = 9  # 3 per AD
  }

  node_shape = "VM.Standard.E4.Flex"
  node_shape_config {
    ocpus         = 4
    memory_in_gbs = 32
  }

  # Auto-scaling
  initial_node_labels {
    key   = "workload"
    value = "general"
  }
}

# GPU node pool
resource "oci_containerengine_node_pool" "gpu" {
  cluster_id = oci_containerengine_cluster.main.id
  name       = "gpu-pool"

  node_shape = "VM.GPU.A10.1"

  initial_node_labels {
    key   = "workload"
    value = "gpu"
  }

  node_config_details {
    size = 3  # Smaller, expensive nodes
  }
}
```

**Pod Scheduling Strategy**:
```yaml
# Node affinity for GPU workloads
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ml-training
  template:
    metadata:
      labels:
        app: ml-training
    spec:
      # Node selector for GPU nodes
      nodeSelector:
        workload: gpu

      # Pod anti-affinity (spread across nodes)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - ml-training
              topologyKey: kubernetes.io/hostname

      containers:
      - name: training
        image: ml-training:v1.0
        resources:
          limits:
            nvidia.com/gpu: 1  # Request 1 GPU
            memory: 32Gi
          requests:
            nvidia.com/gpu: 1
            memory: 16Gi
```

### Service Mesh (Istio)

**Microservices with Service Mesh**:
```
API Gateway (Entry Point)
    ↓
Istio Ingress Gateway
    ↓
Service Mesh (Istio)
├── Service A ←→ Service B
├── Service C ←→ Service D
└── Service E
    ↓
Features:
├── Traffic Management (canary, blue/green)
├── Security (mTLS between services)
├── Observability (distributed tracing)
└── Resilience (circuit breakers, retries)
```

**Canary Deployment**:
```yaml
# VirtualService for canary
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        user:
          exact: "beta-tester"
    route:
    - destination:
        host: reviews
        subset: v2  # New version for beta testers
  - route:
    - destination:
        host: reviews
        subset: v1  # Existing version
      weight: 90
    - destination:
        host: reviews
        subset: v2  # New version
      weight: 10  # 10% of traffic to v2
```

---

## Serverless Architecture

### Functions Deep Dive

**Event-Driven Architecture**:
```
Event Sources:
├── Object Storage: Object created/deleted
├── Queue: Message available
├── Streaming: New data in stream
├── API Gateway: HTTP request
└── Events Service: Resource state change
    ↓
Functions (Serverless Compute)
    ↓
Process and Store Results
```

**Function Development**:
```python
# Function: Process uploaded images
import io
import oci
from PIL import Image

def handler(ctx, data: io.BytesIO = None):
    # Get event details
    event = ctx.Config().get("EVENT_DATA")

    # Authenticate using resource principal
    signer = oci.auth.signers.get_resource_principals_signer()
    object_storage = oci.object_storage.ObjectStorageClient(config={}, signer=signer)

    # Get uploaded image
    namespace = event["namespace"]
    bucket = event["bucketName"]
    object_name = event["objectName"]

    obj = object_storage.get_object(namespace, bucket, object_name)
    image = Image.open(io.BytesIO(obj.data.content))

    # Create thumbnail
    image.thumbnail((200, 200))

    # Save thumbnail
    thumb_buffer = io.BytesIO()
    image.save(thumb_buffer, format='JPEG')
    thumb_buffer.seek(0)

    object_storage.put_object(
        namespace,
        bucket,
        f"thumbnails/{object_name}",
        thumb_buffer
    )

    return {
        "status": "success",
        "original": object_name,
        "thumbnail": f"thumbnails/{object_name}"
    }
```

**Function Configuration**:
```terraform
# Function application
resource "oci_functions_application" "image_processor" {
  compartment_id = var.compartment_id
  display_name   = "image-processor"
  subnet_ids     = [oci_core_subnet.functions.id]

  config = {
    "STORAGE_NAMESPACE" = data.oci_objectstorage_namespace.ns.namespace
  }
}

# Function
resource "oci_functions_function" "thumbnail" {
  application_id = oci_functions_application.image_processor.id
  display_name   = "create-thumbnail"
  image          = "${var.region}.ocir.io/${var.namespace}/thumbnail:latest"
  memory_in_mbs  = 512
  timeout_in_seconds = 120
}

# Event rule to trigger function
resource "oci_events_rule" "image_upload" {
  compartment_id = var.compartment_id
  display_name   = "image-upload-trigger"
  is_enabled     = true

  condition = jsonencode({
    "eventType" : ["com.oraclecloud.objectstorage.createobject"],
    "data" : {
      "bucketName" : ["user-uploads"]
    }
  })

  actions {
    actions {
      action_type = "FAAS"
      is_enabled  = true
      function_id = oci_functions_function.thumbnail.id
    }
  }
}
```

### API Gateway Advanced

**Request/Response Transformation**:
```yaml
# API Deployment spec
specification:
  routes:
    - path: /api/v1/users
      methods:
        - GET
      backend:
        type: HTTP_BACKEND
        url: http://backend:8080/legacy/getUsers

      requestPolicies:
        # Add API key to backend request
        headerTransformations:
          setHeaders:
            items:
              - name: X-API-Key
                values:
                  - ${request.headers[Authorization]}
                ifExists: OVERWRITE

        # Transform request
        bodyTransformation:
          filterQueryParams:
            - name: limit
              type: QUERY_PARAMETER

      responsePolicies:
        # Transform legacy response to modern format
        headerTransformations:
          setHeaders:
            items:
              - name: Content-Type
                values:
                  - application/json
              - name: X-API-Version
                values:
                  - "2.0"
```

**Rate Limiting and Throttling**:
```yaml
requestPolicies:
  rateLimiting:
    rateInRequestsPerSecond: 100  # Global limit
    rateKey: ${request.headers[X-Client-ID]}  # Per client

  # Quota policy
  usagePlans:
    - name: free-tier
      quotaInRequests: 1000  # 1000 requests per month
      resetPolicy: CALENDAR_MONTH
    - name: premium-tier
      quotaInRequests: 1000000  # 1M requests per month
      resetPolicy: CALENDAR_MONTH
```

---

## Data and Analytics

### Data Integration

**ETL Pipeline**:
```
Source Systems
├── On-Premises Database (via FastConnect)
├── SaaS Applications (via API)
└── File Uploads (Object Storage)
    ↓
OCI Data Integration
├── Extract
├── Transform (cleanse, aggregate, enrich)
└── Load
    ↓
Target
├── Autonomous Data Warehouse
├── Object Storage (Data Lake)
└── Analytics Cloud
```

**Data Flow Pattern**:
```
Real-Time Analytics:
Stream → Analytics → Dashboard

Batch Analytics:
Object Storage → Data Integration → ADW → Analytics Cloud

Hybrid:
Stream → Object Storage (landing)
      → Data Integration (batch processing)
      → ADW (warehouse)
      → Analytics Cloud (visualization)
```

### Data Lake Architecture

**Zone-Based Data Lake**:
```
Object Storage Buckets:

├── Raw Zone (raw-data)
│   ├── Ingested as-is
│   ├── No transformation
│   └── Lifecycle: 90 days → Infrequent Access

├── Refined Zone (refined-data)
│   ├── Cleaned and validated
│   ├── Standardized formats (Parquet)
│   └── Lifecycle: 365 days → Archive

├── Curated Zone (curated-data)
│   ├── Business-ready datasets
│   ├── Aggregated and joined
│   └── Lifecycle: Permanent

└── Sandbox Zone (sandbox)
    ├── Experimental analysis
    ├── Temporary datasets
    └── Lifecycle: 30 days → Delete
```

**Big Data Processing**:
```
Object Storage (Data Lake)
    ↓
Data Flow (Apache Spark)
├── Distributed processing
├── Scale out to 1000s of nodes
└── Process petabytes
    ↓
Results → ADW or Object Storage
```

### Data Catalog

**Metadata Management**:
```terraform
resource "oci_datacatalog_catalog" "main" {
  compartment_id = var.compartment_id
  display_name   = "enterprise-data-catalog"
}

# Crawl Object Storage for metadata
resource "oci_datacatalog_data_asset" "object_storage" {
  catalog_id     = oci_datacatalog_catalog.main.id
  display_name   = "data-lake"
  type_key       = "objectStorage"

  properties = {
    "default.namespace" = data.oci_objectstorage_namespace.ns.namespace
  }
}
```

**Benefits**:
- Discover available datasets
- Understand data lineage
- Search for data assets
- Governance and compliance

---

## Machine Learning and AI

### Data Science Platform

**ML Workflow**:
```
Data Preparation:
Object Storage → Data Labeling → Labeled Dataset

Model Development:
Notebooks (Data Science) → Train Model → Evaluate

Model Deployment:
Model Catalog → Model Deployment → API Endpoint

Inference:
API Gateway → Model Endpoint → Predictions
```

**Model Training**:
```python
# Notebook example
import ads
from ads.dataset.dataset_browser import DatasetBrowser

# Authenticate
ads.set_auth('resource_principal')

# Load data from Object Storage
ds = DatasetBrowser.open(
    "oci://bucket@namespace/training-data.csv",
    target="target_column"
)

# AutoML training
ml_model = ads.model.framework.automl.AutoML(
    ds.X, ds.y,
    task='classification',
    algorithm='LightGBM'
)

ml_model.train()

# Evaluate
ml_model.evaluate()

# Save to Model Catalog
model_id = ml_model.save_to_model_catalog(
    compartment_id=compartment_id,
    project_id=project_id,
    display_name="fraud-detection-v1"
)
```

**Model Deployment**:
```python
# Deploy model
from ads.model.deployment import ModelDeployment

deployment = ModelDeployment(
    model_id=model_id,
    compartment_id=compartment_id,
    project_id=project_id
)

deployment.deploy(
    display_name="fraud-detection-api",
    instance_shape="VM.Standard.E4.Flex",
    instance_count=2,
    bandwidth_mbps=10
)

# Get endpoint URL
endpoint = deployment.url
```

### AI Services

**Document Understanding**:
```python
# Extract text from invoices
from oci.ai_document import AIServiceDocumentClient
from oci.ai_document.models import *

ai_client = AIServiceDocumentClient(config)

# Analyze document
analyze_document_response = ai_client.analyze_document(
    AnalyzeDocumentDetails(
        features=[
            DocumentTextExtractionFeature(),
            DocumentKeyValueExtractionFeature(),
            DocumentTableExtractionFeature()
        ],
        document=InlineDocumentContent(
            data=base64_encoded_image
        )
    )
)

# Extract invoice data
for kv_pair in analyze_document_response.data.key_value_pairs:
    print(f"{kv_pair.key}: {kv_pair.value}")
```

**Vision AI**:
```python
# Object detection in images
from oci.ai_vision import AIServiceVisionClient

vision_client = AIServiceVisionClient(config)

# Analyze image
response = vision_client.analyze_image(
    AnalyzeImageDetails(
        features=[
            ImageObjectDetectionFeature(),
            ImageClassificationFeature()
        ],
        image=InlineImageDetails(
            data=base64_encoded_image
        )
    )
)

# Get detected objects
for obj in response.data.image_objects:
    print(f"Object: {obj.name}, Confidence: {obj.confidence}")
```

---

## Cost Optimization

### Advanced Cost Strategies

**Reserved Capacity**:
```terraform
# Capacity reservation for predictable workloads
resource "oci_core_compute_capacity_reservation" "prod" {
  compartment_id     = var.compartment_id
  availability_domain = data.oci_identity_availability_domain.ad1.name
  display_name       = "prod-reserved-capacity"

  instance_reservation_configs {
    instance_shape = "VM.Standard.E4.Flex"

    instance_shape_config {
      ocpus         = 4
      memory_in_gbs = 32
    }

    reserved_count = 20  # Reserve 20 instances
  }
}

# Instances automatically use reservation (lower cost)
resource "oci_core_instance" "app" {
  # ... other config ...
  capacity_reservation_id = oci_core_compute_capacity_reservation.prod.id
}
```

**Spot Instances**:
```terraform
# Use spot instances for fault-tolerant workloads
resource "oci_core_instance_configuration" "spot" {
  compartment_id = var.compartment_id
  display_name   = "spot-config"

  instance_details {
    instance_type = "compute"

    launch_details {
      # ... other config ...

      # Spot instance (up to 50% discount)
      is_pv_encryption_in_transit_enabled = true
      preemptible_instance_config {
        preemption_action {
          type                 = "TERMINATE"
          preserve_boot_volume = false
        }
      }
    }
  }
}
```

**Storage Optimization**:
```
Object Storage Tiers:
├── Standard: $0.026/GB/month (hot data)
├── Infrequent Access: $0.01/GB/month (warm data)
└── Archive: $0.0025/GB/month (cold data)

Block Volume VPUs:
├── 0 VPUs: $0.085/GB/month (low performance)
├── 10 VPUs: $0.085/GB/month (standard)
└── 20+ VPUs: Additional cost (high performance)

Optimization Strategy:
1. Auto-tier Object Storage (automatic)
2. Lifecycle policies (move to archive after 90 days)
3. Delete unused block volumes
4. Lower VPUs for non-critical workloads
```

**Cost Anomaly Detection**:
```python
# Alert on unexpected cost increases
def check_cost_anomaly():
    # Get current month costs
    current_costs = get_current_month_costs()

    # Get average of previous 3 months
    avg_costs = get_average_costs(months=3)

    # Alert if >20% increase
    if current_costs > avg_costs * 1.2:
        send_alert(
            severity="HIGH",
            message=f"Cost increase detected: {current_costs} vs {avg_costs}"
        )
```

---

## Performance Optimization

### Application Performance Management (APM)

**Distributed Tracing**:
```
User Request
    ↓
API Gateway (span 1) - 50ms
    ↓
Service A (span 2) - 200ms
    ├─→ Service B (span 3) - 150ms
    │   └─→ Database (span 4) - 100ms
    └─→ Cache (span 5) - 10ms
    ↓
Response

APM Analysis:
- Total latency: 510ms
- Bottleneck: Service A → Service B (150ms)
- Optimization target: Service B database queries
```

**APM Configuration**:
```terraform
resource "oci_apm_apm_domain" "main" {
  compartment_id = var.compartment_id
  display_name   = "production-apm"
  description    = "APM for production applications"
  is_free_tier   = false
}

# Data keys for different applications
resource "oci_apm_data_key" "app_a" {
  apm_domain_id = oci_apm_apm_domain.main.id
  name          = "app-a-key"
  type          = "PRIVATE"
}
```

**Application Instrumentation**:
```java
// Java application with APM agent
java -javaagent:/path/to/apm-java-agent.jar \
     -Dcom.oracle.apm.agent.data_upload_endpoint=https://... \
     -Dcom.oracle.apm.agent.private_data_key=PRIVATE_DATA_KEY \
     -Dcom.oracle.apm.agent.service_name=my-app \
     -jar my-app.jar
```

### Database Performance

**Autonomous Database Insights**:
```sql
-- Performance Hub queries
-- Top SQL by CPU time
SELECT sql_id,
       cpu_time/1000000 AS cpu_seconds,
       executions,
       cpu_time/executions/1000 AS avg_cpu_ms
FROM v$sql
WHERE cpu_time > 5000000
ORDER BY cpu_time DESC
FETCH FIRST 10 ROWS ONLY;

-- Wait events
SELECT event,
       total_waits,
       time_waited/100 AS time_waited_seconds
FROM v$system_event
WHERE wait_class != 'Idle'
ORDER BY time_waited DESC
FETCH FIRST 20 ROWS ONLY;
```

**SQL Tuning**:
```
Autonomous DB Auto-Tuning:
├── Automatic indexing
│   └── Creates indexes for frequently queried columns
├── Automatic SQL plan management
│   └── Prevents plan regressions
├── Automatic statistics gathering
│   └── Keeps optimizer statistics current
└── SQL quarantine
    └── Blocks poorly performing SQL
```

---

## Exam Tips

### Critical Concepts

**Containers/K8s**:
- OKE: Managed Kubernetes, multi-node pools
- Service Mesh: Traffic management, security, observability
- Scheduling: Node affinity, pod anti-affinity

**Serverless**:
- Functions: Event-driven, auto-scale, pay-per-use
- API Gateway: Transform, rate-limit, authenticate
- Use cases: Image processing, webhooks, data processing

**Data/Analytics**:
- Data Lake: Zone-based (raw, refined, curated)
- ADW: OLAP workloads, auto-scaling, analytics
- Data Integration: ETL, data pipelines

**ML/AI**:
- Data Science: Notebooks, model training, deployment
- AI Services: Vision, Document, Language (pre-trained)
- Model Catalog: Version control for models

**Cost**:
- Reserved capacity: Discount for commitment
- Spot instances: Up to 50% discount (preemptible)
- Storage tiers: Auto-tier, lifecycle policies
- Right-sizing: Monitor and adjust

**Performance**:
- APM: Distributed tracing, bottleneck detection
- Auto-tuning: Autonomous DB automatic optimization
- Caching: Reduce latency, improve throughput

### Common Scenarios

**Q: Microservices with canary deployments?**
A: OKE + Istio service mesh

**Q: Image processing when uploaded to bucket?**
A: Functions triggered by Object Storage events

**Q: Lowest cost for batch processing (fault-tolerant)?**
A: Spot instances

**Q: Find bottlenecks in distributed application?**
A: APM with distributed tracing

**Q: Data lake for analytics?**
A: Object Storage (zones) + Data Integration + ADW

---

## Summary

**Containers**: OKE, service mesh, multi-node pools, scheduling

**Serverless**: Functions, API Gateway, event-driven architecture

**Data**: Data lake, ETL, ADW, Data Integration

**ML/AI**: Data Science platform, AI services, model deployment

**Cost**: Reserved capacity, spot instances, storage tiers, monitoring

**Performance**: APM, auto-tuning, caching, optimization

---

**Congratulations on completing OCI Architect Professional!**
