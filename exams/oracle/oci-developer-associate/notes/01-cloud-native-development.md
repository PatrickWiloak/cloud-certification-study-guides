# OCI Developer Associate - Cloud Native Development

## Table of Contents
- [Functions (Serverless)](#functions-serverless)
- [API Gateway](#api-gateway)
- [Container Engine (OKE)](#container-engine-oke)
- [Container Registry (OCIR)](#container-registry-ocir)
- [Cloud Native Patterns](#cloud-native-patterns)
- [Exam Tips](#exam-tips)

---

## Functions (Serverless)

### Functions Fundamentals

**Function Anatomy**:
```python
# func.py - Basic function structure
import io
import json

def handler(ctx, data: io.BytesIO = None):
    """
    OCI Function entry point

    ctx: Function context (config, request headers, etc.)
    data: Request body (for HTTP functions)
    """
    try:
        # Parse input
        body = json.loads(data.getvalue()) if data else {}

        # Process request
        name = body.get("name", "World")
        result = {"message": f"Hello {name}!"}

        # Return response
        return result

    except Exception as e:
        return {"error": str(e)}
```

**Function Configuration (func.yaml)**:
```yaml
schema_version: 20180708
name: hello-function
version: 1.0.0
runtime: python
build_image: fnproject/python:3.9-dev
run_image: fnproject/python:3.9
entrypoint: /python/bin/fdk /function/func.py handler
memory: 256  # MB
timeout: 30  # seconds
```

**Function Deployment**:
```bash
# 1. Setup CLI context
fn use context oracle-functions

# 2. Create application
oci fn application create \
  --compartment-id ocid1.compartment... \
  --display-name "my-app" \
  --subnet-ids '["ocid1.subnet..."]'

# 3. Deploy function
fn deploy --app my-app

# 4. Invoke function
fn invoke my-app hello-function
echo '{"name":"Developer"}' | fn invoke my-app hello-function
```

### Functions with OCI Services

**Accessing Object Storage**:
```python
import io
import oci
import json

def handler(ctx, data: io.BytesIO = None):
    # Resource Principal authentication (no credentials needed)
    signer = oci.auth.signers.get_resource_principals_signer()

    # Create Object Storage client
    object_storage = oci.object_storage.ObjectStorageClient(config={}, signer=signer)
    namespace = object_storage.get_namespace().data

    # List objects in bucket
    objects = object_storage.list_objects(
        namespace,
        "my-bucket",
        limit=10
    )

    # Return object names
    return {
        "objects": [obj.name for obj in objects.data.objects]
    }
```

**Resource Principal Policy**:
```
# Allow functions to access Object Storage
Allow dynamic-group function-dynamic-group to read buckets in compartment dev
Allow dynamic-group function-dynamic-group to read objects in compartment dev
Allow dynamic-group function-dynamic-group to manage objects in compartment dev
```

**Event-Driven Functions**:
```
Trigger Sources:
├── Object Storage Events (object created/deleted)
├── Streaming (new data in stream)
├── Notifications (ONS topic messages)
├── Events Service (resource state changes)
└── API Gateway (HTTP requests)

Function → Process → Output
    └── Store in Object Storage
    └── Insert into Database
    └── Publish to Stream
    └── Send notification
```

**Event Rule Configuration**:
```terraform
# Trigger function when object uploaded
resource "oci_events_rule" "object_upload" {
  compartment_id = var.compartment_id
  display_name   = "process-uploads"
  is_enabled     = true

  condition = jsonencode({
    "eventType" : ["com.oraclecloud.objectstorage.createobject"],
    "data" : {
      "compartmentName" : ["dev"],
      "bucketName" : ["uploads"]
    }
  })

  actions {
    actions {
      action_type = "FAAS"
      is_enabled  = true
      function_id = oci_functions_function.processor.id
    }
  }
}
```

### Functions Best Practices

**Performance Optimization**:
```python
# Global variables persist between invocations
# Initialize expensive resources once
object_storage_client = None

def handler(ctx, data: io.BytesIO = None):
    global object_storage_client

    # Reuse client across invocations
    if object_storage_client is None:
        signer = oci.auth.signers.get_resource_principals_signer()
        object_storage_client = oci.object_storage.ObjectStorageClient(
            config={}, signer=signer
        )

    # Use cached client
    result = object_storage_client.list_objects(...)
    return result
```

**Error Handling**:
```python
import logging

def handler(ctx, data: io.BytesIO = None):
    logger = logging.getLogger()

    try:
        # Function logic
        result = process_data(data)
        return {"status": "success", "result": result}

    except ValueError as e:
        logger.error(f"Invalid input: {e}")
        return {"status": "error", "message": "Invalid input"}

    except Exception as e:
        logger.error(f"Unexpected error: {e}", exc_info=True)
        return {"status": "error", "message": "Internal error"}
```

---

## API Gateway

### API Gateway Configuration

**Creating Deployment**:
```terraform
resource "oci_apigateway_gateway" "main" {
  compartment_id = var.compartment_id
  endpoint_type  = "PUBLIC"
  subnet_id      = oci_core_subnet.public.id
  display_name   = "main-api-gateway"
}

resource "oci_apigateway_deployment" "api_v1" {
  gateway_id     = oci_apigateway_gateway.main.id
  compartment_id = var.compartment_id
  path_prefix    = "/v1"
  display_name   = "API v1"

  specification {
    # CORS configuration
    request_policies {
      cors {
        allowed_origins = ["https://app.example.com"]
        allowed_methods = ["GET", "POST", "PUT", "DELETE"]
        allowed_headers = ["Content-Type", "Authorization"]
        exposed_headers = ["X-Request-ID"]
        is_allow_credentials_enabled = true
        max_age_in_seconds = 3600
      }

      # Rate limiting
      rate_limiting {
        rate_in_requests_per_second = 10
        rate_key                    = "CLIENT_IP"
      }
    }

    # Routes
    routes {
      path    = "/users"
      methods = ["GET", "POST"]

      backend {
        type = "HTTP_BACKEND"
        url  = "http://backend-service:8080/api/users"
      }

      # Request policies per route
      request_policies {
        # Authentication
        authentication {
          type                        = "JWT_AUTHENTICATION"
          token_header                = "Authorization"
          token_auth_scheme           = "Bearer"
          is_anonymous_access_allowed = false

          # JWT validation
          public_keys {
            type = "REMOTE_JWKS"
            uri  = "https://auth.example.com/.well-known/jwks.json"
            max_cache_duration_in_hours = 24
          }
        }
      }
    }

    # Function backend
    routes {
      path    = "/process"
      methods = ["POST"]

      backend {
        type        = "ORACLE_FUNCTIONS_BACKEND"
        function_id = oci_functions_function.processor.id
      }
    }
  }
}
```

### Request/Response Transformation

**Header Manipulation**:
```yaml
specification:
  requestPolicies:
    headerTransformations:
      setHeaders:
        items:
          - name: X-Forwarded-For
            values:
              - ${request.headers[X-Real-IP]}
          - name: X-API-Version
            values:
              - "1.0"

      renameHeaders:
        items:
          - from: Authorization
            to: X-Auth-Token

      filterHeaders:
        items:
          - name: X-Internal-Header
```

**Body Transformation**:
```yaml
specification:
  routes:
    - path: /api/users
      requestPolicies:
        bodyValidation:
          required: true
          content:
            application/json:
              validationMode: ENFORCING
              validationSchema:
                type: object
                properties:
                  name:
                    type: string
                    minLength: 1
                  email:
                    type: string
                    format: email
                required:
                  - name
                  - email
```

### Authentication and Authorization

**JWT Authentication**:
```yaml
requestPolicies:
  authentication:
    type: JWT_AUTHENTICATION
    token_header: Authorization
    token_auth_scheme: Bearer

    public_keys:
      type: STATIC_KEYS
      keys:
        - kid: "key-1"
          format: PEM
          key: |
            -----BEGIN PUBLIC KEY-----
            ...
            -----END PUBLIC KEY-----

    audiences:
      - "https://api.example.com"

    issuers:
      - "https://auth.example.com"

    # Optional: Verify specific claims
    verify_claims:
      - key: "role"
        values: ["admin", "user"]
```

**Custom Authorizer Function**:
```python
# Custom authorizer function
import json

def handler(ctx, data: io.BytesIO = None):
    """
    Custom authorization logic
    Returns: ALLOW or DENY with context
    """
    # Parse request
    request = json.loads(data.getvalue())

    # Get token from headers
    token = request.get("token", "")

    # Custom validation logic
    if validate_token(token):
        return {
            "active": True,
            "principal": "user@example.com",
            "scope": ["read", "write"],
            "context": {
                "userId": "12345",
                "tier": "premium"
            }
        }
    else:
        return {
            "active": False
        }
```

---

## Container Engine (OKE)

### OKE Cluster Setup

**Quick Create Cluster**:
```bash
oci ce cluster create \
  --compartment-id ocid1.compartment... \
  --name "dev-cluster" \
  --kubernetes-version "v1.28.2" \
  --vcn-id ocid1.vcn... \
  --endpoint-subnet-id ocid1.subnet... \
  --service-lb-subnet-ids '["ocid1.subnet..."]'
```

**Custom Cluster (Terraform)**:
```terraform
resource "oci_containerengine_cluster" "main" {
  compartment_id     = var.compartment_id
  kubernetes_version = "v1.28.2"
  name               = "production-cluster"
  vcn_id             = oci_core_vcn.main.id

  # Kubernetes API endpoint
  endpoint_config {
    is_public_ip_enabled = true
    subnet_id            = oci_core_subnet.k8s_api.id
  }

  # Cluster options
  options {
    service_lb_subnet_ids = [oci_core_subnet.lb.id]

    add_ons {
      is_kubernetes_dashboard_enabled = true
      is_tiller_enabled               = false
    }

    kubernetes_network_config {
      pods_cidr     = "10.244.0.0/16"
      services_cidr = "10.96.0.0/16"
    }

    persistent_volume_config {
      freeform_tags = {
        "Environment" = "production"
      }
    }
  }
}
```

**Node Pool Configuration**:
```terraform
resource "oci_containerengine_node_pool" "main" {
  cluster_id         = oci_containerengine_cluster.main.id
  compartment_id     = var.compartment_id
  kubernetes_version = "v1.28.2"
  name               = "pool-1"

  # Node configuration
  node_config_details {
    size = 3  # Number of nodes

    # Multi-AD placement
    placement_configs {
      availability_domain = data.oci_identity_availability_domain.ad1.name
      subnet_id           = oci_core_subnet.workers.id
    }
    placement_configs {
      availability_domain = data.oci_identity_availability_domain.ad2.name
      subnet_id           = oci_core_subnet.workers.id
    }
    placement_configs {
      availability_domain = data.oci_identity_availability_domain.ad3.name
      subnet_id           = oci_core_subnet.workers.id
    }

    # Node labels
    node_pool_pod_network_option_details {
      cni_type = "FLANNEL_OVERLAY"
    }
  }

  # Node shape
  node_shape = "VM.Standard.E4.Flex"
  node_shape_config {
    ocpus         = 2
    memory_in_gbs = 16
  }

  # Node image
  node_source_details {
    image_id    = data.oci_core_images.node_image.images[0].id
    source_type = "IMAGE"
  }
}
```

### Kubernetes Application Deployment

**Simple Deployment**:
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: phx.ocir.io/namespace/web-app:v1.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database.host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database.password
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
```

**ConfigMap and Secrets**:
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.host: "db.example.com"
  database.port: "3306"
  app.mode: "production"
---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # Base64 encoded values
  database.password: cGFzc3dvcmQxMjM=
```

**Horizontal Pod Autoscaler**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

---

## Container Registry (OCIR)

### OCIR Setup and Usage

**Authentication**:
```bash
# Generate auth token in OCI Console
# Login to OCIR
docker login phx.ocir.io
# Username: <tenancy-namespace>/<username>
# Password: <auth-token>
```

**Image Management**:
```bash
# Build image
docker build -t my-app:v1.0 .

# Tag for OCIR
docker tag my-app:v1.0 phx.ocir.io/my-namespace/my-app:v1.0

# Push to OCIR
docker push phx.ocir.io/my-namespace/my-app:v1.0

# Pull from OCIR
docker pull phx.ocir.io/my-namespace/my-app:v1.0
```

**IAM Policies for OCIR**:
```
# Allow developers to push/pull images
Allow group Developers to manage repos in compartment dev
Allow group Developers to read repos in compartment prod

# Allow OKE to pull images
Allow dynamic-group oke-nodes to read repos in compartment prod
```

**Image Signing**:
```bash
# Sign image with Vault key
oci artifacts container image-signature sign \
  --compartment-id ocid1.compartment... \
  --image-digest sha256:abc123... \
  --kms-key-id ocid1.key.oc1... \
  --signing-algorithm SHA_256_RSA_PKCS_PSS
```

### CI/CD with OCIR

**GitLab CI Pipeline**:
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - push
  - deploy

variables:
  DOCKER_REGISTRY: phx.ocir.io
  IMAGE_NAME: $DOCKER_REGISTRY/namespace/my-app

build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA .
    - docker tag $IMAGE_NAME:$CI_COMMIT_SHORT_SHA $IMAGE_NAME:latest

test:
  stage: test
  script:
    - docker run $IMAGE_NAME:$CI_COMMIT_SHORT_SHA npm test

push:
  stage: push
  script:
    - echo $OCIR_TOKEN | docker login -u $OCIR_USERNAME --password-stdin $DOCKER_REGISTRY
    - docker push $IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - docker push $IMAGE_NAME:latest

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/web-app web=$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - kubectl rollout status deployment/web-app
```

---

## Cloud Native Patterns

### Twelve-Factor App Principles

**1. Codebase**: One codebase in version control
**2. Dependencies**: Explicitly declare dependencies (requirements.txt, package.json)
**3. Config**: Store config in environment variables
**4. Backing Services**: Treat databases as attached resources
**5. Build, Release, Run**: Strictly separate build and run stages
**6. Processes**: Execute app as stateless processes
**7. Port Binding**: Export services via port binding
**8. Concurrency**: Scale out via process model
**9. Disposability**: Fast startup and graceful shutdown
**10. Dev/Prod Parity**: Keep environments similar
**11. Logs**: Treat logs as event streams
**12. Admin Processes**: Run admin tasks as one-off processes

**Implementation Example**:
```python
# twelve-factor-app.py
import os
import logging
import sys

# 3. Config from environment
DB_HOST = os.environ.get('DB_HOST', 'localhost')
DB_PORT = int(os.environ.get('DB_PORT', 3306))
API_KEY = os.environ.get('API_KEY')  # Required

# 11. Logs to stdout
logging.basicConfig(stream=sys.stdout, level=logging.INFO)
logger = logging.getLogger(__name__)

def main():
    # 6. Stateless processing
    logger.info(f"Connecting to database at {DB_HOST}:{DB_PORT}")

    # 4. Backing services
    db = connect_to_database(DB_HOST, DB_PORT)

    # 7. Port binding
    port = int(os.environ.get('PORT', 8080))
    app.run(host='0.0.0.0', port=port)

if __name__ == '__main__':
    # 9. Graceful shutdown
    import signal
    signal.signal(signal.SIGTERM, lambda *args: sys.exit(0))

    main()
```

### Microservices Patterns

**Service Discovery**:
```
Kubernetes Service Discovery:
├── Service names as DNS (web-app.default.svc.cluster.local)
├── Environment variables (WEB_APP_SERVICE_HOST)
└── Kubernetes API

API Gateway Pattern:
Client → API Gateway → Service Discovery → Microservices
```

**Circuit Breaker**:
```python
# Circuit breaker pattern
from circuitbreaker import circuit

@circuit(failure_threshold=5, recovery_timeout=30)
def call_external_api():
    response = requests.get('https://api.example.com/data')
    if response.status_code != 200:
        raise Exception("API call failed")
    return response.json()

# If 5 failures occur, circuit opens for 30 seconds
# Prevents cascading failures
```

**Saga Pattern** (Distributed Transactions):
```
Order Service:
1. Create Order → Success
    ↓
Inventory Service:
2. Reserve Items → Success
    ↓
Payment Service:
3. Process Payment → FAILED
    ↓
Compensation (Rollback):
- Inventory Service: Release Items
- Order Service: Cancel Order
```

---

## Exam Tips

### Critical Concepts

**Functions**:
- Serverless compute, pay-per-use
- Event-driven (Object Storage, Events, API Gateway)
- Resource principal for OCI service access
- 5-minute max execution time

**API Gateway**:
- Entry point for APIs
- Authentication (JWT, custom)
- Rate limiting, CORS, transformation
- Backend types: HTTP, Functions, Stock Response

**OKE**:
- Managed Kubernetes
- Node pools (multiple allowed)
- kubectl for management
- Integrates with OCIR, Load Balancer

**OCIR**:
- Docker-compatible registry
- <region>.ocir.io
- Auth token for login
- IAM policies for access control

### Common Scenarios

**Q: Process images when uploaded to bucket?**
A: Functions triggered by Object Storage events

**Q: API needs authentication and rate limiting?**
A: API Gateway with JWT auth + rate limiting policy

**Q: Deploy containerized app with auto-scaling?**
A: OKE with HPA (Horizontal Pod Autoscaler)

**Q: Store container images privately?**
A: OCIR with IAM policies

**Q: Function needs to access Object Storage?**
A: Resource principal + IAM policy for dynamic group

---

## Summary

**Functions**: Serverless, event-driven, integrate with OCI services

**API Gateway**: API management, auth, rate limiting, transformation

**OKE**: Kubernetes service, node pools, integrations

**OCIR**: Container registry, secure, OCI-integrated

**Patterns**: Twelve-factor, microservices, event-driven

---

**Next**: Application Deployment and Integration
