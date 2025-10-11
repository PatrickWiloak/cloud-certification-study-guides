# MLOps and Production Generative AI on Google Cloud

## Generative AI MLOps Overview

### MLOps for GenAI

**Key Differences from Traditional ML**
- Prompt versioning instead of feature engineering
- Foundation model updates and versioning
- Response quality monitoring (more subjective)
- Context window management
- Token usage and cost optimization
- Safety and content moderation
- Faster iteration cycles

**MLOps Principles**
- Continuous integration and deployment
- Automated testing and validation
- Monitoring and observability
- Version control (models, prompts, data)
- Reproducibility
- Collaboration

## Vertex AI Pipelines for GenAI

### Pipeline Components

**Data Preparation**
- Training data collection
- Data cleaning and validation
- Format conversion (JSONL)
- Train/validation split
- Data versioning

**Model Tuning**
- Tuning job configuration
- Hyperparameter selection
- Training execution
- Model evaluation
- Model registration

**Deployment**
- Endpoint creation
- Model deployment
- Traffic splitting
- Canary deployments
- Rollback capability

**Monitoring**
- Response quality tracking
- Latency monitoring
- Error rate tracking
- Cost monitoring
- User feedback collection

### Example Pipeline

```python
from google.cloud import aiplatform
from kfp import dsl
from kfp.v2 import compiler

@dsl.component(
    base_image="python:3.9",
    packages_to_install=["google-cloud-aiplatform"]
)
def prepare_training_data(
    source_data: str,
    output_data: dsl.Output[dsl.Dataset]
):
    """Prepare training data for tuning"""
    import json
    from google.cloud import storage

    # Load and format data
    # Save to output_data.path

@dsl.component
def tune_model(
    training_data: dsl.Input[dsl.Dataset],
    base_model: str,
    tuned_model: dsl.Output[dsl.Model]
):
    """Tune foundation model"""
    from google.cloud import aiplatform
    from vertexai.preview.language_models import TextGenerationModel

    base = TextGenerationModel.from_pretrained(base_model)

    tuning_job = base.tune_model(
        training_data=training_data.path,
        train_steps=100,
        learning_rate=0.001
    )

    # Save model info
    tuned_model.metadata["model_id"] = tuning_job.tuned_model_name

@dsl.component
def evaluate_model(
    tuned_model: dsl.Input[dsl.Model],
    evaluation_data: dsl.Input[dsl.Dataset],
    metrics: dsl.Output[dsl.Metrics]
):
    """Evaluate tuned model"""
    # Run evaluation
    # Calculate metrics
    # Log results

@dsl.component
def deploy_model(
    tuned_model: dsl.Input[dsl.Model],
    endpoint_name: str
):
    """Deploy model to endpoint"""
    from google.cloud import aiplatform

    endpoint = aiplatform.Endpoint.create(display_name=endpoint_name)
    endpoint.deploy(tuned_model)

@dsl.pipeline(
    name="genai-tuning-pipeline",
    description="Pipeline for tuning and deploying GenAI models"
)
def genai_pipeline(
    source_data: str,
    base_model: str = "text-bison@002",
    endpoint_name: str = "genai-endpoint"
):
    prepare_task = prepare_training_data(source_data=source_data)

    tune_task = tune_model(
        training_data=prepare_task.outputs["output_data"],
        base_model=base_model
    )

    eval_task = evaluate_model(
        tuned_model=tune_task.outputs["tuned_model"],
        evaluation_data=prepare_task.outputs["output_data"]
    )

    deploy_task = deploy_model(
        tuned_model=tune_task.outputs["tuned_model"],
        endpoint_name=endpoint_name
    ).after(eval_task)

# Compile and run
compiler.Compiler().compile(
    pipeline_func=genai_pipeline,
    package_path="genai_pipeline.json"
)

pipeline = aiplatform.PipelineJob(
    display_name="genai-tuning-pipeline",
    template_path="genai_pipeline.json",
    parameter_values={
        "source_data": "gs://my-bucket/training-data.jsonl",
        "base_model": "text-bison@002"
    }
)

pipeline.run()
```

## Prompt Management

### Version Control

**Prompt Repository Structure**
```
prompts/
├── classification/
│   ├── sentiment_v1.txt
│   ├── sentiment_v2.txt
│   └── intent_detection.txt
├── generation/
│   ├── email_draft_v1.txt
│   └── summary_template.txt
└── chat/
    ├── customer_service.txt
    └── technical_support.txt
```

**Prompt Template Management**
```python
import jinja2
from google.cloud import storage

class PromptManager:
    def __init__(self, bucket_name: str):
        self.client = storage.Client()
        self.bucket = self.client.bucket(bucket_name)

    def get_prompt(self, prompt_name: str, version: str = "latest") -> str:
        """Retrieve prompt template"""
        blob = self.bucket.blob(f"prompts/{prompt_name}_{version}.txt")
        return blob.download_as_text()

    def save_prompt(self, prompt_name: str, content: str, version: str):
        """Save new prompt version"""
        blob = self.bucket.blob(f"prompts/{prompt_name}_{version}.txt")
        blob.upload_from_string(content)
        blob.metadata = {
            "created_at": datetime.now().isoformat(),
            "version": version
        }
        blob.patch()

    def render_prompt(self, template: str, **kwargs) -> str:
        """Render prompt with variables"""
        template = jinja2.Template(template)
        return template.render(**kwargs)

# Usage
pm = PromptManager("my-prompts-bucket")

template = pm.get_prompt("email_draft", version="v2")
prompt = pm.render_prompt(
    template,
    recipient="John",
    topic="Project Update",
    tone="professional"
)
```

### Prompt Testing Framework

```python
import pytest
from dataclasses import dataclass
from typing import List
from vertexai.preview.language_models import TextGenerationModel

@dataclass
class PromptTest:
    name: str
    prompt: str
    expected_keywords: List[str]
    max_length: int
    temperature: float = 0.2

class PromptTester:
    def __init__(self):
        self.model = TextGenerationModel.from_pretrained("text-bison@002")

    def run_test(self, test: PromptTest) -> dict:
        """Run single prompt test"""
        response = self.model.predict(
            test.prompt,
            temperature=test.temperature,
            max_output_tokens=test.max_length
        )

        results = {
            "passed": True,
            "response": response.text,
            "issues": []
        }

        # Check expected keywords
        for keyword in test.expected_keywords:
            if keyword.lower() not in response.text.lower():
                results["passed"] = False
                results["issues"].append(f"Missing keyword: {keyword}")

        # Check length
        if len(response.text) > test.max_length:
            results["passed"] = False
            results["issues"].append("Response too long")

        return results

# Test suite
def test_sentiment_classification():
    tester = PromptTester()

    test = PromptTest(
        name="sentiment_positive",
        prompt="Classify sentiment: 'This product is amazing!'",
        expected_keywords=["positive"],
        max_length=100
    )

    result = tester.run_test(test)
    assert result["passed"], f"Test failed: {result['issues']}"
```

## Model Deployment Patterns

### A/B Testing

```python
from google.cloud import aiplatform

class GenAIABTest:
    def __init__(self, endpoint_name: str):
        self.endpoint = aiplatform.Endpoint.list(
            filter=f'display_name="{endpoint_name}"'
        )[0]

    def deploy_models(
        self,
        model_a_id: str,
        model_b_id: str,
        traffic_split: int = 50
    ):
        """Deploy two models with traffic split"""

        # Deploy Model A
        model_a = aiplatform.Model(model_a_id)
        self.endpoint.deploy(
            model=model_a,
            deployed_model_display_name="model-a",
            traffic_percentage=traffic_split,
            machine_type="n1-standard-4"
        )

        # Deploy Model B
        model_b = aiplatform.Model(model_b_id)
        self.endpoint.deploy(
            model=model_b,
            deployed_model_display_name="model-b",
            traffic_percentage=100 - traffic_split,
            machine_type="n1-standard-4"
        )

    def update_traffic(self, model_a_traffic: int):
        """Update traffic split"""
        self.endpoint.update(
            traffic_split={
                "model-a": model_a_traffic,
                "model-b": 100 - model_a_traffic
            }
        )

# Usage
ab_test = GenAIABTest("genai-endpoint")
ab_test.deploy_models(
    model_a_id="projects/123/locations/us-central1/models/456",
    model_b_id="projects/123/locations/us-central1/models/789",
    traffic_split=50  # 50% each
)

# After analysis, shift more traffic to better model
ab_test.update_traffic(model_a_traffic=80)
```

### Canary Deployment

```python
import time
from google.cloud import aiplatform

def canary_deployment(
    endpoint_name: str,
    new_model_id: str,
    canary_percentage: int = 10,
    monitoring_duration: int = 3600  # 1 hour
):
    """Gradual canary deployment"""

    endpoint = aiplatform.Endpoint.list(
        filter=f'display_name="{endpoint_name}"'
    )[0]

    # Deploy canary with small traffic
    new_model = aiplatform.Model(new_model_id)
    endpoint.deploy(
        model=new_model,
        deployed_model_display_name="canary",
        traffic_percentage=canary_percentage,
        machine_type="n1-standard-4"
    )

    print(f"Canary deployed with {canary_percentage}% traffic")

    # Monitor for specified duration
    time.sleep(monitoring_duration)

    # Check metrics (implementation depends on monitoring setup)
    metrics = check_canary_metrics(endpoint)

    if metrics["error_rate"] < 0.01 and metrics["latency_p99"] < 1000:
        # Gradually increase traffic
        for traffic in [25, 50, 75, 100]:
            endpoint.update(
                traffic_split={
                    "canary": traffic,
                    "stable": 100 - traffic
                }
            )
            print(f"Updated canary to {traffic}%")
            time.sleep(900)  # 15 min between updates

        # Remove old model
        endpoint.undeploy_all()
        print("Canary promotion complete")
    else:
        # Rollback
        endpoint.undeploy(deployed_model_id="canary")
        print("Canary failed, rolled back")

def check_canary_metrics(endpoint) -> dict:
    """Check canary deployment metrics"""
    # Implement metric collection from Cloud Monitoring
    return {
        "error_rate": 0.005,
        "latency_p99": 850,
        "throughput": 1000
    }
```

## Monitoring and Observability

### Response Quality Monitoring

```python
from google.cloud import aiplatform, monitoring_v3
from google.cloud.monitoring_v3 import query
import time

class GenAIMonitor:
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.client = monitoring_v3.MetricServiceClient()
        self.project_name = f"projects/{project_id}"

    def log_prediction(
        self,
        prompt: str,
        response: str,
        model_id: str,
        latency: float,
        token_count: int
    ):
        """Log prediction details"""
        from google.cloud import logging

        logging_client = logging.Client()
        logger = logging_client.logger("genai-predictions")

        logger.log_struct({
            "prompt": prompt[:100],  # Truncate for privacy
            "response_length": len(response),
            "model_id": model_id,
            "latency_ms": latency,
            "token_count": token_count,
            "timestamp": time.time()
        })

    def create_custom_metric(self, metric_name: str):
        """Create custom metric for GenAI monitoring"""
        descriptor = monitoring_v3.MetricDescriptor(
            type=f"custom.googleapis.com/genai/{metric_name}",
            metric_kind=monitoring_v3.MetricDescriptor.MetricKind.GAUGE,
            value_type=monitoring_v3.MetricDescriptor.ValueType.DOUBLE,
            description=f"GenAI metric: {metric_name}"
        )

        self.client.create_metric_descriptor(
            name=self.project_name,
            metric_descriptor=descriptor
        )

    def write_metric(self, metric_name: str, value: float):
        """Write custom metric value"""
        series = monitoring_v3.TimeSeries()
        series.metric.type = f"custom.googleapis.com/genai/{metric_name}"

        now = time.time()
        point = monitoring_v3.Point({
            "interval": {"end_time": {"seconds": int(now)}},
            "value": {"double_value": value}
        })
        series.points = [point]

        self.client.create_time_series(
            name=self.project_name,
            time_series=[series]
        )

# Usage
monitor = GenAIMonitor("my-project")

# Log each prediction
start = time.time()
response = model.predict(prompt)
latency = (time.time() - start) * 1000

monitor.log_prediction(
    prompt=prompt,
    response=response.text,
    model_id="text-bison@002",
    latency=latency,
    token_count=len(response.text.split())
)

# Write custom metrics
monitor.write_metric("response_quality_score", 0.85)
monitor.write_metric("safety_filter_triggers", 2)
```

### User Feedback Collection

```python
from google.cloud import firestore

class FeedbackCollector:
    def __init__(self):
        self.db = firestore.Client()

    def collect_feedback(
        self,
        request_id: str,
        rating: int,  # 1-5
        feedback_text: str = None,
        user_id: str = None
    ):
        """Collect user feedback on responses"""
        doc_ref = self.db.collection("feedback").document(request_id)
        doc_ref.set({
            "rating": rating,
            "feedback_text": feedback_text,
            "user_id": user_id,
            "timestamp": firestore.SERVER_TIMESTAMP
        })

    def get_average_rating(self, model_id: str, days: int = 7) -> float:
        """Calculate average rating for model"""
        from datetime import datetime, timedelta

        cutoff = datetime.now() - timedelta(days=days)

        docs = self.db.collection("feedback")\
            .where("model_id", "==", model_id)\
            .where("timestamp", ">=", cutoff)\
            .stream()

        ratings = [doc.to_dict()["rating"] for doc in docs]
        return sum(ratings) / len(ratings) if ratings else 0

# Usage with prediction
collector = FeedbackCollector()

# After showing response to user
collector.collect_feedback(
    request_id="req-12345",
    rating=4,
    feedback_text="Good response but could be more concise",
    user_id="user-789"
)
```

## Cost Optimization

### Token Usage Tracking

```python
from collections import defaultdict
import json

class TokenTracker:
    def __init__(self):
        self.usage = defaultdict(lambda: {
            "input_tokens": 0,
            "output_tokens": 0,
            "requests": 0
        })

    def track_usage(
        self,
        model_id: str,
        input_tokens: int,
        output_tokens: int
    ):
        """Track token usage"""
        self.usage[model_id]["input_tokens"] += input_tokens
        self.usage[model_id]["output_tokens"] += output_tokens
        self.usage[model_id]["requests"] += 1

    def estimate_cost(self, model_id: str) -> dict:
        """Estimate cost based on pricing"""
        pricing = {
            "text-bison@002": {
                "input": 0.00025,  # per 1K tokens
                "output": 0.0005
            },
            "gemini-pro": {
                "input": 0.000125,
                "output": 0.000375
            }
        }

        if model_id not in pricing:
            return {"error": "Unknown model"}

        usage = self.usage[model_id]
        prices = pricing[model_id]

        input_cost = (usage["input_tokens"] / 1000) * prices["input"]
        output_cost = (usage["output_tokens"] / 1000) * prices["output"]

        return {
            "input_cost": input_cost,
            "output_cost": output_cost,
            "total_cost": input_cost + output_cost,
            "total_tokens": usage["input_tokens"] + usage["output_tokens"],
            "requests": usage["requests"]
        }

    def export_usage(self, filepath: str):
        """Export usage data"""
        with open(filepath, 'w') as f:
            json.dump(dict(self.usage), f, indent=2)

# Usage
tracker = TokenTracker()

# Track each request
input_tokens = len(prompt.split()) * 1.3  # Rough estimate
output_tokens = len(response.text.split()) * 1.3

tracker.track_usage(
    model_id="text-bison@002",
    input_tokens=int(input_tokens),
    output_tokens=int(output_tokens)
)

# Get cost estimate
costs = tracker.estimate_cost("text-bison@002")
print(f"Estimated cost: ${costs['total_cost']:.4f}")
```

### Response Caching

```python
import hashlib
from google.cloud import memorystore_v1
import redis

class ResponseCache:
    def __init__(self, redis_host: str, redis_port: int = 6379):
        self.redis_client = redis.Redis(
            host=redis_host,
            port=redis_port,
            decode_responses=True
        )
        self.ttl = 3600  # 1 hour default

    def _generate_key(self, prompt: str, model_id: str, params: dict) -> str:
        """Generate cache key"""
        cache_input = f"{prompt}:{model_id}:{json.dumps(params, sort_keys=True)}"
        return hashlib.sha256(cache_input.encode()).hexdigest()

    def get(self, prompt: str, model_id: str, params: dict) -> str:
        """Get cached response"""
        key = self._generate_key(prompt, model_id, params)
        return self.redis_client.get(key)

    def set(
        self,
        prompt: str,
        model_id: str,
        params: dict,
        response: str,
        ttl: int = None
    ):
        """Cache response"""
        key = self._generate_key(prompt, model_id, params)
        self.redis_client.setex(
            key,
            ttl or self.ttl,
            response
        )

    def get_stats(self) -> dict:
        """Get cache statistics"""
        info = self.redis_client.info()
        return {
            "hits": info.get("keyspace_hits", 0),
            "misses": info.get("keyspace_misses", 0),
            "hit_rate": info.get("keyspace_hits", 0) /
                       (info.get("keyspace_hits", 0) + info.get("keyspace_misses", 1))
        }

# Usage
cache = ResponseCache(redis_host="10.0.0.3")

# Check cache before prediction
params = {"temperature": 0.7, "max_tokens": 256}
cached_response = cache.get(prompt, model_id, params)

if cached_response:
    response = cached_response
    print("Cache hit!")
else:
    response = model.predict(prompt, **params).text
    cache.set(prompt, model_id, params, response)
    print("Cache miss, generated new response")
```

## Best Practices

### Development

1. **Version everything**: Prompts, models, data
2. **Automated testing**: Unit tests for prompts
3. **CI/CD pipelines**: Automated deployment
4. **Experiment tracking**: Log all experiments
5. **Code reviews**: Review prompt changes

### Production

1. **Monitor continuously**: Quality, latency, cost
2. **Gradual rollouts**: Canary and A/B testing
3. **Fallback mechanisms**: Handle failures gracefully
4. **Cost controls**: Set budgets and alerts
5. **User feedback**: Collect and act on feedback

### Security

1. **Input validation**: Sanitize user inputs
2. **Output filtering**: Content moderation
3. **Access control**: IAM policies
4. **Audit logging**: Track all requests
5. **Data privacy**: PII detection and removal

## Production Deployment Strategies

### Model Serving Architecture

**Vertex AI Prediction Options**
- Online prediction (real-time)
- Batch prediction (asynchronous)
- Private endpoints (VPC)
- Multi-model endpoints
- Auto-scaling configurations

**Serving Infrastructure**
```python
from google.cloud import aiplatform
from vertexai.preview.generative_models import GenerativeModel

class GenAIServingManager:
    def __init__(self, project_id: str, location: str):
        aiplatform.init(project=project_id, location=location)
        self.project_id = project_id
        self.location = location

    def create_endpoint(
        self,
        display_name: str,
        network: str = None,
        enable_private: bool = False
    ):
        """Create prediction endpoint with optional private networking"""

        endpoint_config = {
            "display_name": display_name,
            "encryption_spec_key_name": None
        }

        if enable_private and network:
            # Private endpoint configuration
            endpoint_config["network"] = network
            endpoint_config["enable_private_service_connect"] = True

        endpoint = aiplatform.Endpoint.create(**endpoint_config)
        print(f"Created endpoint: {endpoint.resource_name}")
        return endpoint

    def deploy_with_autoscaling(
        self,
        endpoint: aiplatform.Endpoint,
        model_name: str,
        min_replicas: int = 1,
        max_replicas: int = 10,
        machine_type: str = "n1-standard-4",
        accelerator_type: str = None,
        accelerator_count: int = 0
    ):
        """Deploy model with auto-scaling configuration"""

        model = GenerativeModel(model_name)

        # Configure auto-scaling
        deployed_model = endpoint.deploy(
            model=model,
            deployed_model_display_name=f"{model_name}-deployment",
            machine_type=machine_type,
            min_replica_count=min_replicas,
            max_replica_count=max_replicas,
            accelerator_type=accelerator_type,
            accelerator_count=accelerator_count,
            # Auto-scaling metrics
            autoscaling_target_cpu_utilization=60,
            autoscaling_target_accelerator_duty_cycle=60
        )

        print(f"Deployed model with auto-scaling: {min_replicas}-{max_replicas} replicas")
        return deployed_model

    def configure_load_balancing(
        self,
        endpoint_name: str,
        model_deployments: list
    ):
        """Configure load balancing across multiple deployments"""

        endpoint = aiplatform.Endpoint.list(
            filter=f'display_name="{endpoint_name}"'
        )[0]

        # Traffic split configuration
        traffic_split = {}
        total_weight = sum(d['weight'] for d in model_deployments)

        for deployment in model_deployments:
            traffic_percentage = (deployment['weight'] / total_weight) * 100
            traffic_split[deployment['id']] = traffic_percentage

        endpoint.update(traffic_split=traffic_split)
        print(f"Updated traffic split: {traffic_split}")

# Usage
manager = GenAIServingManager(
    project_id="my-project",
    location="us-central1"
)

# Create endpoint
endpoint = manager.create_endpoint(
    display_name="genai-production-endpoint",
    network="projects/my-project/global/networks/my-vpc",
    enable_private=True
)

# Deploy with auto-scaling
manager.deploy_with_autoscaling(
    endpoint=endpoint,
    model_name="gemini-pro",
    min_replicas=2,
    max_replicas=20,
    machine_type="n1-standard-8"
)
```

### Multi-Region Deployment

```python
from google.cloud import aiplatform
from typing import List, Dict

class MultiRegionDeployment:
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.regions = ["us-central1", "europe-west4", "asia-southeast1"]

    def deploy_globally(
        self,
        model_name: str,
        endpoint_name: str
    ) -> Dict[str, str]:
        """Deploy model across multiple regions"""

        endpoints = {}

        for region in self.regions:
            aiplatform.init(project=self.project_id, location=region)

            # Create regional endpoint
            endpoint = aiplatform.Endpoint.create(
                display_name=f"{endpoint_name}-{region}"
            )

            # Deploy model
            model = GenerativeModel(model_name)
            endpoint.deploy(
                model=model,
                machine_type="n1-standard-4",
                min_replica_count=2,
                max_replica_count=10
            )

            endpoints[region] = endpoint.resource_name
            print(f"Deployed to {region}: {endpoint.resource_name}")

        return endpoints

    def route_request(self, user_location: str) -> str:
        """Route user to nearest region"""

        # Simple region mapping
        region_mapping = {
            "US": "us-central1",
            "EU": "europe-west4",
            "ASIA": "asia-southeast1"
        }

        return region_mapping.get(user_location, "us-central1")

    def health_check_all_regions(self) -> Dict[str, bool]:
        """Check health of all regional deployments"""

        health_status = {}

        for region in self.regions:
            try:
                aiplatform.init(project=self.project_id, location=region)
                endpoints = aiplatform.Endpoint.list()
                health_status[region] = len(endpoints) > 0
            except Exception as e:
                health_status[region] = False
                print(f"Health check failed for {region}: {e}")

        return health_status

# Usage
multi_region = MultiRegionDeployment("my-project")

# Deploy globally
endpoints = multi_region.deploy_globally(
    model_name="gemini-pro",
    endpoint_name="genai-global"
)

# Check health
health = multi_region.health_check_all_regions()
print(f"Regional health: {health}")
```

### Caching Strategies

```python
import hashlib
import json
from google.cloud import memorystore_v1, redis_v1
from typing import Optional, Dict, Any
import time

class AdvancedResponseCache:
    def __init__(
        self,
        redis_host: str,
        redis_port: int = 6379,
        enable_semantic_cache: bool = False
    ):
        import redis
        self.redis_client = redis.Redis(
            host=redis_host,
            port=redis_port,
            decode_responses=True
        )
        self.enable_semantic = enable_semantic_cache
        self.cache_hits = 0
        self.cache_misses = 0

    def _generate_cache_key(
        self,
        prompt: str,
        model_id: str,
        parameters: Dict[str, Any]
    ) -> str:
        """Generate deterministic cache key"""

        # Normalize parameters
        normalized_params = json.dumps(parameters, sort_keys=True)
        cache_string = f"{prompt}|{model_id}|{normalized_params}"

        return hashlib.sha256(cache_string.encode()).hexdigest()

    def _semantic_similarity_key(self, prompt: str) -> str:
        """Generate semantic similarity key using embeddings"""

        if not self.enable_semantic:
            return None

        # Use text embeddings for semantic matching
        from vertexai.language_models import TextEmbeddingModel

        model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")
        embedding = model.get_embeddings([prompt])[0]

        # Create approximate key from embedding
        return hashlib.sha256(str(embedding.values[:10]).encode()).hexdigest()

    def get(
        self,
        prompt: str,
        model_id: str,
        parameters: Dict[str, Any]
    ) -> Optional[str]:
        """Get cached response with exact and semantic matching"""

        # Try exact match first
        exact_key = self._generate_cache_key(prompt, model_id, parameters)
        cached = self.redis_client.get(exact_key)

        if cached:
            self.cache_hits += 1
            # Update access metadata
            self.redis_client.hincrby(f"{exact_key}:meta", "hits", 1)
            return cached

        # Try semantic match if enabled
        if self.enable_semantic:
            semantic_key = self._semantic_similarity_key(prompt)
            if semantic_key:
                cached = self.redis_client.get(semantic_key)
                if cached:
                    self.cache_hits += 1
                    return cached

        self.cache_misses += 1
        return None

    def set(
        self,
        prompt: str,
        model_id: str,
        parameters: Dict[str, Any],
        response: str,
        ttl: int = 3600,
        metadata: Dict[str, Any] = None
    ):
        """Cache response with metadata"""

        key = self._generate_cache_key(prompt, model_id, parameters)

        # Store response
        self.redis_client.setex(key, ttl, response)

        # Store metadata
        meta = metadata or {}
        meta.update({
            "cached_at": time.time(),
            "model_id": model_id,
            "prompt_length": len(prompt),
            "response_length": len(response),
            "hits": 0
        })

        # Store as hash
        self.redis_client.hset(f"{key}:meta", mapping=meta)
        self.redis_client.expire(f"{key}:meta", ttl)

        # Semantic cache if enabled
        if self.enable_semantic:
            semantic_key = self._semantic_similarity_key(prompt)
            if semantic_key:
                self.redis_client.setex(semantic_key, ttl, response)

    def invalidate_by_model(self, model_id: str):
        """Invalidate all cache entries for a model"""

        # Scan for keys with model_id in metadata
        for key in self.redis_client.scan_iter(match="*:meta"):
            meta = self.redis_client.hgetall(key)
            if meta.get("model_id") == model_id:
                cache_key = key.replace(":meta", "")
                self.redis_client.delete(cache_key, key)

    def get_statistics(self) -> Dict[str, Any]:
        """Get comprehensive cache statistics"""

        info = self.redis_client.info()

        total_requests = self.cache_hits + self.cache_misses
        hit_rate = self.cache_hits / total_requests if total_requests > 0 else 0

        return {
            "cache_hits": self.cache_hits,
            "cache_misses": self.cache_misses,
            "hit_rate": hit_rate,
            "memory_used": info.get("used_memory_human", "N/A"),
            "total_keys": self.redis_client.dbsize(),
            "evicted_keys": info.get("evicted_keys", 0)
        }

    def optimize_cache(self):
        """Run cache optimization"""

        # Remove least recently used entries if memory is high
        info = self.redis_client.info()
        memory_usage = info.get("used_memory", 0)
        max_memory = info.get("maxmemory", float('inf'))

        if memory_usage / max_memory > 0.9:
            # Redis handles LRU eviction automatically
            print("Cache memory usage high, Redis LRU eviction active")

# Usage
cache = AdvancedResponseCache(
    redis_host="10.0.0.3",
    enable_semantic_cache=True
)

# Check cache before prediction
params = {
    "temperature": 0.7,
    "max_output_tokens": 256,
    "top_p": 0.95
}

cached_response = cache.get(prompt, "gemini-pro", params)

if cached_response:
    response = cached_response
    print("Cache hit!")
else:
    # Generate new response
    response = model.predict(prompt, **params).text

    # Cache with metadata
    cache.set(
        prompt=prompt,
        model_id="gemini-pro",
        parameters=params,
        response=response,
        ttl=7200,
        metadata={
            "user_id": "user-123",
            "application": "customer-support"
        }
    )
    print("Cache miss, cached new response")

# Get statistics
stats = cache.get_statistics()
print(f"Cache statistics: {stats}")
```

## Advanced Monitoring and Observability

### Token Usage and Cost Monitoring

```python
from google.cloud import monitoring_v3, bigquery
from dataclasses import dataclass
from datetime import datetime, timedelta
from typing import List, Dict
import time

@dataclass
class TokenUsage:
    prompt_tokens: int
    completion_tokens: int
    total_tokens: int
    model_id: str
    timestamp: datetime
    user_id: str
    request_id: str

class ComprehensiveTokenMonitor:
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.monitoring_client = monitoring_v3.MetricServiceClient()
        self.bq_client = bigquery.Client(project=project_id)
        self.project_name = f"projects/{project_id}"

        # Token pricing per 1K tokens (USD)
        self.pricing = {
            "gemini-pro": {
                "input": 0.000125,
                "output": 0.000375
            },
            "gemini-pro-vision": {
                "input": 0.00025,
                "output": 0.00047
            },
            "text-bison@002": {
                "input": 0.00025,
                "output": 0.0005
            },
            "chat-bison@002": {
                "input": 0.00025,
                "output": 0.0005
            },
            "text-unicorn@001": {
                "input": 0.0005,
                "output": 0.0015
            }
        }

    def track_token_usage(self, usage: TokenUsage):
        """Track token usage with multiple backends"""

        # 1. Write to Cloud Monitoring
        self._write_to_monitoring(usage)

        # 2. Write to BigQuery for analysis
        self._write_to_bigquery(usage)

        # 3. Check against quotas
        self._check_quotas(usage)

    def _write_to_monitoring(self, usage: TokenUsage):
        """Write token metrics to Cloud Monitoring"""

        metrics = [
            ("prompt_tokens", usage.prompt_tokens),
            ("completion_tokens", usage.completion_tokens),
            ("total_tokens", usage.total_tokens)
        ]

        for metric_name, value in metrics:
            series = monitoring_v3.TimeSeries()
            series.metric.type = f"custom.googleapis.com/genai/{metric_name}"
            series.metric.labels["model_id"] = usage.model_id
            series.metric.labels["user_id"] = usage.user_id

            series.resource.type = "global"
            series.resource.labels["project_id"] = self.project_id

            now = time.time()
            point = monitoring_v3.Point({
                "interval": {
                    "end_time": {"seconds": int(now)}
                },
                "value": {"int64_value": value}
            })
            series.points = [point]

            self.monitoring_client.create_time_series(
                name=self.project_name,
                time_series=[series]
            )

    def _write_to_bigquery(self, usage: TokenUsage):
        """Write usage data to BigQuery for analysis"""

        table_id = f"{self.project_id}.genai_monitoring.token_usage"

        # Calculate cost
        model_pricing = self.pricing.get(usage.model_id, {})
        input_cost = (usage.prompt_tokens / 1000) * model_pricing.get("input", 0)
        output_cost = (usage.completion_tokens / 1000) * model_pricing.get("output", 0)

        rows_to_insert = [{
            "request_id": usage.request_id,
            "timestamp": usage.timestamp.isoformat(),
            "model_id": usage.model_id,
            "user_id": usage.user_id,
            "prompt_tokens": usage.prompt_tokens,
            "completion_tokens": usage.completion_tokens,
            "total_tokens": usage.total_tokens,
            "input_cost": input_cost,
            "output_cost": output_cost,
            "total_cost": input_cost + output_cost
        }]

        errors = self.bq_client.insert_rows_json(table_id, rows_to_insert)
        if errors:
            print(f"BigQuery insert errors: {errors}")

    def _check_quotas(self, usage: TokenUsage):
        """Check if usage exceeds quotas"""

        # Query recent usage
        query = f"""
        SELECT
            SUM(total_tokens) as total_tokens,
            SUM(total_cost) as total_cost
        FROM `{self.project_id}.genai_monitoring.token_usage`
        WHERE timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
        AND user_id = @user_id
        """

        job_config = bigquery.QueryJobConfig(
            query_parameters=[
                bigquery.ScalarQueryParameter("user_id", "STRING", usage.user_id)
            ]
        )

        results = self.bq_client.query(query, job_config=job_config).result()

        for row in results:
            daily_tokens = row.total_tokens
            daily_cost = row.total_cost

            # Check against limits
            if daily_tokens > 1000000:  # 1M tokens per day
                print(f"WARNING: User {usage.user_id} exceeded daily token limit")

            if daily_cost > 100:  # $100 per day
                print(f"WARNING: User {usage.user_id} exceeded daily cost limit")

    def get_cost_report(
        self,
        start_date: datetime,
        end_date: datetime,
        group_by: str = "model_id"
    ) -> List[Dict]:
        """Generate cost report"""

        query = f"""
        SELECT
            {group_by},
            SUM(prompt_tokens) as total_prompt_tokens,
            SUM(completion_tokens) as total_completion_tokens,
            SUM(total_tokens) as total_tokens,
            SUM(input_cost) as total_input_cost,
            SUM(output_cost) as total_output_cost,
            SUM(total_cost) as total_cost,
            COUNT(*) as request_count
        FROM `{self.project_id}.genai_monitoring.token_usage`
        WHERE timestamp BETWEEN @start_date AND @end_date
        GROUP BY {group_by}
        ORDER BY total_cost DESC
        """

        job_config = bigquery.QueryJobConfig(
            query_parameters=[
                bigquery.ScalarQueryParameter("start_date", "TIMESTAMP", start_date),
                bigquery.ScalarQueryParameter("end_date", "TIMESTAMP", end_date)
            ]
        )

        results = self.bq_client.query(query, job_config=job_config).result()

        report = []
        for row in results:
            report.append({
                group_by: row[0],
                "total_tokens": row.total_tokens,
                "total_cost": round(row.total_cost, 2),
                "request_count": row.request_count,
                "avg_cost_per_request": round(row.total_cost / row.request_count, 4)
            })

        return report

    def create_cost_alert(
        self,
        alert_name: str,
        threshold: float,
        notification_channels: List[str]
    ):
        """Create alert for cost threshold"""

        from google.cloud import monitoring_v3

        alert_client = monitoring_v3.AlertPolicyServiceClient()

        # Define alert condition
        condition = monitoring_v3.AlertPolicy.Condition(
            display_name=f"GenAI cost exceeds ${threshold}",
            condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                filter='metric.type="custom.googleapis.com/genai/daily_cost"',
                comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                threshold_value=threshold,
                duration={"seconds": 300},  # 5 minutes
                aggregations=[
                    monitoring_v3.Aggregation(
                        alignment_period={"seconds": 300},
                        per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_SUM
                    )
                ]
            )
        )

        # Create alert policy
        policy = monitoring_v3.AlertPolicy(
            display_name=alert_name,
            conditions=[condition],
            combiner=monitoring_v3.AlertPolicy.ConditionCombinerType.AND,
            notification_channels=notification_channels,
            enabled=True
        )

        created_policy = alert_client.create_alert_policy(
            name=self.project_name,
            alert_policy=policy
        )

        print(f"Created alert policy: {created_policy.name}")

# Usage
monitor = ComprehensiveTokenMonitor("my-project")

# Track usage
usage = TokenUsage(
    prompt_tokens=150,
    completion_tokens=300,
    total_tokens=450,
    model_id="gemini-pro",
    timestamp=datetime.now(),
    user_id="user-123",
    request_id="req-abc-123"
)

monitor.track_token_usage(usage)

# Generate cost report
report = monitor.get_cost_report(
    start_date=datetime.now() - timedelta(days=30),
    end_date=datetime.now(),
    group_by="model_id"
)

for entry in report:
    print(f"Model: {entry['model_id']}")
    print(f"  Total cost: ${entry['total_cost']}")
    print(f"  Requests: {entry['request_count']}")
    print(f"  Avg per request: ${entry['avg_cost_per_request']}")
```

### Latency and Quality Monitoring

```python
from google.cloud import monitoring_v3, logging_v2
import time
from dataclasses import dataclass
from typing import Optional, List
import statistics

@dataclass
class PredictionMetrics:
    request_id: str
    model_id: str
    latency_ms: float
    prompt_length: int
    response_length: int
    timestamp: datetime
    error: Optional[str] = None
    quality_score: Optional[float] = None
    user_feedback: Optional[int] = None

class QualityAndLatencyMonitor:
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.monitoring_client = monitoring_v3.MetricServiceClient()
        self.logging_client = logging_v2.Client()
        self.logger = self.logging_client.logger("genai-quality-metrics")
        self.project_name = f"projects/{project_id}"

        # Store recent metrics for percentile calculation
        self.recent_latencies = []
        self.max_recent = 1000

    def track_prediction(self, metrics: PredictionMetrics):
        """Track prediction metrics"""

        # Add to recent latencies
        self.recent_latencies.append(metrics.latency_ms)
        if len(self.recent_latencies) > self.max_recent:
            self.recent_latencies.pop(0)

        # Log to Cloud Logging
        self._log_prediction(metrics)

        # Write to Cloud Monitoring
        self._write_latency_metric(metrics)

        if metrics.quality_score:
            self._write_quality_metric(metrics)

        # Check for anomalies
        self._check_anomalies(metrics)

    def _log_prediction(self, metrics: PredictionMetrics):
        """Log prediction details"""

        log_entry = {
            "request_id": metrics.request_id,
            "model_id": metrics.model_id,
            "latency_ms": metrics.latency_ms,
            "prompt_length": metrics.prompt_length,
            "response_length": metrics.response_length,
            "timestamp": metrics.timestamp.isoformat(),
            "quality_score": metrics.quality_score,
            "user_feedback": metrics.user_feedback
        }

        if metrics.error:
            log_entry["error"] = metrics.error
            self.logger.log_struct(log_entry, severity="ERROR")
        else:
            self.logger.log_struct(log_entry, severity="INFO")

    def _write_latency_metric(self, metrics: PredictionMetrics):
        """Write latency metric to Cloud Monitoring"""

        series = monitoring_v3.TimeSeries()
        series.metric.type = "custom.googleapis.com/genai/prediction_latency"
        series.metric.labels["model_id"] = metrics.model_id

        series.resource.type = "global"
        series.resource.labels["project_id"] = self.project_id

        now = time.time()
        point = monitoring_v3.Point({
            "interval": {"end_time": {"seconds": int(now)}},
            "value": {"double_value": metrics.latency_ms}
        })
        series.points = [point]

        self.monitoring_client.create_time_series(
            name=self.project_name,
            time_series=[series]
        )

    def _write_quality_metric(self, metrics: PredictionMetrics):
        """Write quality score metric"""

        series = monitoring_v3.TimeSeries()
        series.metric.type = "custom.googleapis.com/genai/quality_score"
        series.metric.labels["model_id"] = metrics.model_id

        series.resource.type = "global"
        series.resource.labels["project_id"] = self.project_id

        now = time.time()
        point = monitoring_v3.Point({
            "interval": {"end_time": {"seconds": int(now)}},
            "value": {"double_value": metrics.quality_score}
        })
        series.points = [point]

        self.monitoring_client.create_time_series(
            name=self.project_name,
            time_series=[series]
        )

    def _check_anomalies(self, metrics: PredictionMetrics):
        """Check for anomalies in metrics"""

        if len(self.recent_latencies) < 10:
            return

        # Calculate percentiles
        p50 = statistics.median(self.recent_latencies)
        p95 = statistics.quantiles(self.recent_latencies, n=20)[18]  # 95th percentile

        # Check if current latency is anomalous
        if metrics.latency_ms > p95 * 2:
            self.logger.log_struct({
                "message": "Latency anomaly detected",
                "request_id": metrics.request_id,
                "latency_ms": metrics.latency_ms,
                "p95": p95,
                "threshold": p95 * 2
            }, severity="WARNING")

        # Check quality score if available
        if metrics.quality_score and metrics.quality_score < 0.5:
            self.logger.log_struct({
                "message": "Low quality score detected",
                "request_id": metrics.request_id,
                "quality_score": metrics.quality_score
            }, severity="WARNING")

    def get_latency_percentiles(
        self,
        model_id: str,
        hours: int = 24
    ) -> Dict[str, float]:
        """Calculate latency percentiles"""

        # Query Cloud Monitoring for historical data
        now = time.time()
        seconds = int(now)
        nanos = int((now - seconds) * 10 ** 9)

        interval = monitoring_v3.TimeInterval({
            "end_time": {"seconds": seconds, "nanos": nanos},
            "start_time": {"seconds": seconds - (hours * 3600)}
        })

        # Query latency metrics
        results = self.monitoring_client.list_time_series(
            request={
                "name": self.project_name,
                "filter": f'metric.type="custom.googleapis.com/genai/prediction_latency" AND metric.label.model_id="{model_id}"',
                "interval": interval,
                "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL
            }
        )

        latencies = []
        for result in results:
            for point in result.points:
                latencies.append(point.value.double_value)

        if not latencies:
            return {}

        latencies.sort()

        return {
            "p50": statistics.median(latencies),
            "p95": statistics.quantiles(latencies, n=20)[18],
            "p99": statistics.quantiles(latencies, n=100)[98],
            "min": min(latencies),
            "max": max(latencies),
            "avg": statistics.mean(latencies)
        }

    def create_slo(
        self,
        slo_name: str,
        latency_threshold_ms: float,
        target_percentage: float = 95.0
    ):
        """Create Service Level Objective for latency"""

        from google.cloud import service_management

        # SLO: 95% of requests should complete within threshold
        slo_config = {
            "name": slo_name,
            "description": f"{target_percentage}% of requests < {latency_threshold_ms}ms",
            "target": target_percentage / 100,
            "window": "rolling_30_days",
            "metric": "custom.googleapis.com/genai/prediction_latency",
            "threshold": latency_threshold_ms
        }

        print(f"SLO configuration: {slo_config}")
        # Actual implementation would use Cloud Monitoring SLO API

        return slo_config

# Usage
quality_monitor = QualityAndLatencyMonitor("my-project")

# Track prediction
start_time = time.time()
response = model.predict(prompt)
latency = (time.time() - start_time) * 1000

metrics = PredictionMetrics(
    request_id="req-xyz-789",
    model_id="gemini-pro",
    latency_ms=latency,
    prompt_length=len(prompt),
    response_length=len(response.text),
    timestamp=datetime.now(),
    quality_score=0.87  # Calculated separately
)

quality_monitor.track_prediction(metrics)

# Get latency statistics
stats = quality_monitor.get_latency_percentiles("gemini-pro", hours=24)
print(f"Latency percentiles: {stats}")

# Create SLO
quality_monitor.create_slo(
    slo_name="genai-latency-slo",
    latency_threshold_ms=500,
    target_percentage=95.0
)
```

## Safety and Content Moderation

### Content Filtering and Safety

```python
from vertexai.preview.generative_models import (
    GenerativeModel,
    HarmCategory,
    HarmBlockThreshold,
    SafetySetting
)
from google.cloud import dlp_v2
from typing import List, Dict, Tuple
import re

class SafetyAndModerationManager:
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.dlp_client = dlp_v2.DlpServiceClient()
        self.parent = f"projects/{project_id}"

        # Safety settings for Gemini
        self.safety_settings = [
            SafetySetting(
                category=HarmCategory.HARM_CATEGORY_HATE_SPEECH,
                threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE
            ),
            SafetySetting(
                category=HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
                threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE
            ),
            SafetySetting(
                category=HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT,
                threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE
            ),
            SafetySetting(
                category=HarmCategory.HARM_CATEGORY_HARASSMENT,
                threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE
            )
        ]

    def detect_pii(self, text: str) -> Dict:
        """Detect PII using Cloud DLP"""

        # Configure DLP inspection
        inspect_config = {
            "info_types": [
                {"name": "EMAIL_ADDRESS"},
                {"name": "PHONE_NUMBER"},
                {"name": "CREDIT_CARD_NUMBER"},
                {"name": "US_SOCIAL_SECURITY_NUMBER"},
                {"name": "PERSON_NAME"},
                {"name": "STREET_ADDRESS"},
                {"name": "PASSPORT"},
                {"name": "MEDICAL_RECORD_NUMBER"}
            ],
            "min_likelihood": dlp_v2.Likelihood.POSSIBLE,
            "include_quote": True
        }

        item = {"value": text}

        response = self.dlp_client.inspect_content(
            request={
                "parent": self.parent,
                "inspect_config": inspect_config,
                "item": item
            }
        )

        findings = []
        for finding in response.result.findings:
            findings.append({
                "info_type": finding.info_type.name,
                "likelihood": dlp_v2.Likelihood(finding.likelihood).name,
                "quote": finding.quote,
                "location": finding.location.byte_range
            })

        return {
            "has_pii": len(findings) > 0,
            "findings": findings
        }

    def redact_pii(self, text: str) -> str:
        """Redact PII from text using Cloud DLP"""

        inspect_config = {
            "info_types": [
                {"name": "EMAIL_ADDRESS"},
                {"name": "PHONE_NUMBER"},
                {"name": "CREDIT_CARD_NUMBER"},
                {"name": "US_SOCIAL_SECURITY_NUMBER"},
                {"name": "PERSON_NAME"}
            ]
        }

        # Redaction configuration
        deidentify_config = {
            "info_type_transformations": {
                "transformations": [{
                    "primitive_transformation": {
                        "replace_with_info_type_config": {}
                    }
                }]
            }
        }

        item = {"value": text}

        response = self.dlp_client.deidentify_content(
            request={
                "parent": self.parent,
                "deidentify_config": deidentify_config,
                "inspect_config": inspect_config,
                "item": item
            }
        )

        return response.item.value

    def check_toxicity(self, text: str) -> Dict:
        """Check text toxicity using Perspective API"""

        # Note: Requires Perspective API setup
        # This is a simplified example

        from googleapiclient import discovery

        try:
            client = discovery.build(
                "commentanalyzer",
                "v1alpha1",
                developerKey="YOUR_API_KEY",
                discoveryServiceUrl="https://commentanalyzer.googleapis.com/$discovery/rest?version=v1alpha1"
            )

            analyze_request = {
                'comment': {'text': text},
                'requestedAttributes': {
                    'TOXICITY': {},
                    'SEVERE_TOXICITY': {},
                    'IDENTITY_ATTACK': {},
                    'INSULT': {},
                    'PROFANITY': {},
                    'THREAT': {}
                }
            }

            response = client.comments().analyze(body=analyze_request).execute()

            scores = {}
            for attribute, data in response['attributeScores'].items():
                scores[attribute] = data['summaryScore']['value']

            return {
                "is_toxic": scores.get('TOXICITY', 0) > 0.7,
                "scores": scores
            }
        except Exception as e:
            print(f"Toxicity check error: {e}")
            return {"is_toxic": False, "scores": {}, "error": str(e)}

    def moderate_content(
        self,
        prompt: str,
        response: str
    ) -> Tuple[bool, List[str]]:
        """Comprehensive content moderation"""

        issues = []

        # Check prompt for PII
        prompt_pii = self.detect_pii(prompt)
        if prompt_pii["has_pii"]:
            issues.append(f"Prompt contains PII: {[f['info_type'] for f in prompt_pii['findings']]}")

        # Check response for PII
        response_pii = self.detect_pii(response)
        if response_pii["has_pii"]:
            issues.append(f"Response contains PII: {[f['info_type'] for f in response_pii['findings']]}")

        # Check response toxicity
        toxicity = self.check_toxicity(response)
        if toxicity.get("is_toxic"):
            issues.append(f"Response is toxic: {toxicity['scores']}")

        # Check for banned patterns
        banned_patterns = [
            r'password\s*[:=]\s*\S+',
            r'api[_-]key\s*[:=]\s*\S+',
            r'secret\s*[:=]\s*\S+'
        ]

        for pattern in banned_patterns:
            if re.search(pattern, response, re.IGNORECASE):
                issues.append(f"Response contains sensitive pattern: {pattern}")

        is_safe = len(issues) == 0
        return is_safe, issues

    def safe_predict(
        self,
        model: GenerativeModel,
        prompt: str,
        redact_prompt: bool = True
    ) -> Dict:
        """Make prediction with safety checks"""

        # Moderate input
        if redact_prompt:
            safe_prompt = self.redact_pii(prompt)
        else:
            pii_check = self.detect_pii(prompt)
            if pii_check["has_pii"]:
                return {
                    "success": False,
                    "error": "Prompt contains PII",
                    "findings": pii_check["findings"]
                }
            safe_prompt = prompt

        try:
            # Generate with safety settings
            response = model.generate_content(
                safe_prompt,
                safety_settings=self.safety_settings
            )

            # Check safety attributes
            if response.candidates[0].finish_reason != 1:  # FINISH_REASON_STOP
                return {
                    "success": False,
                    "error": "Response blocked by safety filters",
                    "finish_reason": response.candidates[0].finish_reason,
                    "safety_ratings": response.candidates[0].safety_ratings
                }

            response_text = response.text

            # Moderate output
            is_safe, issues = self.moderate_content(safe_prompt, response_text)

            if not is_safe:
                return {
                    "success": False,
                    "error": "Response failed moderation",
                    "issues": issues
                }

            return {
                "success": True,
                "response": response_text,
                "safety_ratings": response.candidates[0].safety_ratings
            }

        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }

# Usage
safety_manager = SafetyAndModerationManager("my-project")

# Initialize model
model = GenerativeModel("gemini-pro")

# Safe prediction with PII detection and content moderation
result = safety_manager.safe_predict(
    model=model,
    prompt="My email is john@example.com. Can you help me?",
    redact_prompt=True
)

if result["success"]:
    print(f"Response: {result['response']}")
    print(f"Safety ratings: {result['safety_ratings']}")
else:
    print(f"Blocked: {result['error']}")
    if "issues" in result:
        print(f"Issues: {result['issues']}")
```

### Custom Safety Classifiers

```python
from google.cloud import aiplatform
from vertexai.preview.language_models import TextEmbeddingModel
import numpy as np
from typing import List, Dict

class CustomSafetyClassifier:
    def __init__(self, project_id: str, location: str):
        self.project_id = project_id
        self.location = location
        aiplatform.init(project=project_id, location=location)
        self.embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")

        # Custom safety categories with examples
        self.safety_categories = {
            "financial_advice": [
                "invest all your money in",
                "guaranteed returns of",
                "buy this stock immediately"
            ],
            "medical_advice": [
                "you should take this medication",
                "this will cure your disease",
                "stop taking your prescribed"
            ],
            "legal_advice": [
                "you should sue",
                "this is definitely legal",
                "sign this contract"
            ],
            "dangerous_instructions": [
                "how to make explosives",
                "how to hack into",
                "how to bypass security"
            ]
        }

        # Pre-compute embeddings for categories
        self.category_embeddings = self._compute_category_embeddings()

    def _compute_category_embeddings(self) -> Dict[str, np.ndarray]:
        """Pre-compute embeddings for safety categories"""

        embeddings = {}

        for category, examples in self.safety_categories.items():
            # Get embeddings for all examples
            example_embeddings = []
            for example in examples:
                embedding = self.embedding_model.get_embeddings([example])[0]
                example_embeddings.append(embedding.values)

            # Average embeddings for category
            embeddings[category] = np.mean(example_embeddings, axis=0)

        return embeddings

    def classify_safety(
        self,
        text: str,
        threshold: float = 0.75
    ) -> Dict[str, any]:
        """Classify text against custom safety categories"""

        # Get text embedding
        text_embedding = self.embedding_model.get_embeddings([text])[0]
        text_vector = np.array(text_embedding.values)

        # Compare with category embeddings
        similarities = {}
        for category, cat_embedding in self.category_embeddings.items():
            # Cosine similarity
            similarity = np.dot(text_vector, cat_embedding) / (
                np.linalg.norm(text_vector) * np.linalg.norm(cat_embedding)
            )
            similarities[category] = float(similarity)

        # Check if any category exceeds threshold
        flagged_categories = [
            cat for cat, sim in similarities.items()
            if sim > threshold
        ]

        return {
            "is_safe": len(flagged_categories) == 0,
            "flagged_categories": flagged_categories,
            "all_similarities": similarities,
            "max_similarity": max(similarities.values()) if similarities else 0
        }

    def add_safety_category(
        self,
        category_name: str,
        examples: List[str]
    ):
        """Add new custom safety category"""

        self.safety_categories[category_name] = examples

        # Compute embedding for new category
        example_embeddings = []
        for example in examples:
            embedding = self.embedding_model.get_embeddings([example])[0]
            example_embeddings.append(embedding.values)

        self.category_embeddings[category_name] = np.mean(example_embeddings, axis=0)

        print(f"Added safety category: {category_name}")

    def train_custom_classifier(
        self,
        training_data: List[Dict],
        model_display_name: str
    ):
        """Train custom AutoML text classification model for safety"""

        # Create dataset
        dataset = aiplatform.TextDataset.create(
            display_name=f"{model_display_name}-dataset",
            gcs_source="gs://my-bucket/safety-training-data.jsonl"
        )

        # Train AutoML model
        job = aiplatform.AutoMLTextTrainingJob(
            display_name=f"{model_display_name}-job",
            prediction_type="classification",
            multi_label=True
        )

        model = job.run(
            dataset=dataset,
            training_fraction_split=0.8,
            validation_fraction_split=0.1,
            test_fraction_split=0.1
        )

        print(f"Trained custom safety classifier: {model.resource_name}")
        return model

# Usage
safety_classifier = CustomSafetyClassifier(
    project_id="my-project",
    location="us-central1"
)

# Classify text
text = "You should invest all your savings in cryptocurrency for guaranteed returns"
result = safety_classifier.classify_safety(text, threshold=0.75)

if not result["is_safe"]:
    print(f"Unsafe content detected!")
    print(f"Flagged categories: {result['flagged_categories']}")
    print(f"Similarities: {result['all_similarities']}")
else:
    print("Content is safe")

# Add custom category
safety_classifier.add_safety_category(
    category_name="political_misinformation",
    examples=[
        "the election was stolen",
        "voting machines were hacked",
        "fake ballots were counted"
    ]
)
```

## Evaluation Frameworks

### Automated Evaluation Metrics

```python
from typing import List, Dict
import numpy as np
from collections import Counter
import re

class GenAIEvaluationMetrics:
    """Comprehensive evaluation metrics for Gen AI responses"""

    @staticmethod
    def bleu_score(
        reference: str,
        candidate: str,
        n_gram: int = 4
    ) -> float:
        """Calculate BLEU score"""

        from nltk.translate.bleu_score import sentence_bleu, SmoothingFunction

        reference_tokens = reference.lower().split()
        candidate_tokens = candidate.lower().split()

        smoothie = SmoothingFunction().method4
        score = sentence_bleu(
            [reference_tokens],
            candidate_tokens,
            smoothing_function=smoothie
        )

        return score

    @staticmethod
    def rouge_scores(
        reference: str,
        candidate: str
    ) -> Dict[str, float]:
        """Calculate ROUGE scores (ROUGE-1, ROUGE-2, ROUGE-L)"""

        from rouge_score import rouge_scorer

        scorer = rouge_scorer.RougeScorer(
            ['rouge1', 'rouge2', 'rougeL'],
            use_stemmer=True
        )

        scores = scorer.score(reference, candidate)

        return {
            'rouge1': scores['rouge1'].fmeasure,
            'rouge2': scores['rouge2'].fmeasure,
            'rougeL': scores['rougeL'].fmeasure
        }

    @staticmethod
    def exact_match(reference: str, candidate: str) -> bool:
        """Check if candidate exactly matches reference"""
        return reference.strip().lower() == candidate.strip().lower()

    @staticmethod
    def f1_score(reference: str, candidate: str) -> float:
        """Calculate token-level F1 score"""

        ref_tokens = set(reference.lower().split())
        cand_tokens = set(candidate.lower().split())

        if len(cand_tokens) == 0:
            return 0.0

        common = ref_tokens & cand_tokens

        if len(common) == 0:
            return 0.0

        precision = len(common) / len(cand_tokens)
        recall = len(common) / len(ref_tokens)

        f1 = 2 * (precision * recall) / (precision + recall)
        return f1

    @staticmethod
    def semantic_similarity(
        text1: str,
        text2: str,
        model_name: str = "textembedding-gecko@003"
    ) -> float:
        """Calculate semantic similarity using embeddings"""

        from vertexai.language_models import TextEmbeddingModel

        model = TextEmbeddingModel.from_pretrained(model_name)

        embeddings = model.get_embeddings([text1, text2])
        vec1 = np.array(embeddings[0].values)
        vec2 = np.array(embeddings[1].values)

        # Cosine similarity
        similarity = np.dot(vec1, vec2) / (
            np.linalg.norm(vec1) * np.linalg.norm(vec2)
        )

        return float(similarity)

    @staticmethod
    def coherence_score(text: str) -> float:
        """Calculate coherence score based on linguistic features"""

        # Simple coherence indicators
        sentences = re.split(r'[.!?]+', text)
        sentences = [s.strip() for s in sentences if s.strip()]

        if len(sentences) < 2:
            return 1.0

        score = 0.0

        # Check for transition words
        transitions = {
            'however', 'therefore', 'furthermore', 'moreover',
            'additionally', 'consequently', 'nevertheless', 'thus',
            'hence', 'accordingly', 'also', 'besides'
        }

        transition_count = sum(
            1 for sent in sentences
            if any(trans in sent.lower() for trans in transitions)
        )

        # Coherence based on transitions
        score += (transition_count / len(sentences)) * 0.5

        # Check for consistent length (not too variable)
        lengths = [len(s.split()) for s in sentences]
        avg_length = np.mean(lengths)
        variance = np.var(lengths)

        if avg_length > 0:
            cv = np.sqrt(variance) / avg_length  # Coefficient of variation
            score += max(0, 1 - cv) * 0.5

        return min(score, 1.0)

    @staticmethod
    def factual_consistency(
        source: str,
        generated: str,
        model_name: str = "gemini-pro"
    ) -> Dict[str, any]:
        """Check factual consistency using LLM-as-judge"""

        from vertexai.preview.generative_models import GenerativeModel

        model = GenerativeModel(model_name)

        prompt = f"""You are an expert fact-checker. Compare the generated text with the source text and determine if the generated text contains any factual inconsistencies or hallucinations.

Source text:
{source}

Generated text:
{generated}

Provide your analysis in this format:
Consistent: [YES/NO]
Confidence: [0-1]
Issues: [List any factual inconsistencies found]
Explanation: [Brief explanation]"""

        response = model.generate_content(prompt)

        # Parse response
        response_text = response.text

        is_consistent = "consistent: yes" in response_text.lower()

        return {
            "is_consistent": is_consistent,
            "analysis": response_text
        }

class EvaluationSuite:
    """Complete evaluation suite for Gen AI models"""

    def __init__(self):
        self.metrics = GenAIEvaluationMetrics()

    def evaluate_response(
        self,
        prompt: str,
        response: str,
        reference: str = None,
        source_context: str = None
    ) -> Dict[str, any]:
        """Comprehensive evaluation of a single response"""

        results = {
            "prompt": prompt,
            "response": response,
            "metrics": {}
        }

        # Basic metrics
        results["metrics"]["response_length"] = len(response.split())
        results["metrics"]["coherence"] = self.metrics.coherence_score(response)

        # Reference-based metrics
        if reference:
            results["metrics"]["exact_match"] = self.metrics.exact_match(
                reference, response
            )
            results["metrics"]["f1_score"] = self.metrics.f1_score(
                reference, response
            )
            results["metrics"]["bleu"] = self.metrics.bleu_score(
                reference, response
            )
            results["metrics"]["rouge"] = self.metrics.rouge_scores(
                reference, response
            )
            results["metrics"]["semantic_similarity"] = self.metrics.semantic_similarity(
                reference, response
            )

        # Factual consistency check
        if source_context:
            results["metrics"]["factual_consistency"] = self.metrics.factual_consistency(
                source_context, response
            )

        # Calculate overall score
        metric_values = [
            v for v in results["metrics"].values()
            if isinstance(v, (int, float))
        ]

        if metric_values:
            results["overall_score"] = np.mean(metric_values)

        return results

    def evaluate_dataset(
        self,
        evaluation_data: List[Dict]
    ) -> Dict[str, any]:
        """Evaluate model on entire dataset"""

        all_results = []

        for item in evaluation_data:
            result = self.evaluate_response(
                prompt=item["prompt"],
                response=item["response"],
                reference=item.get("reference"),
                source_context=item.get("source_context")
            )
            all_results.append(result)

        # Aggregate metrics
        aggregated = {
            "num_examples": len(all_results),
            "metrics": {}
        }

        # Collect all metric names
        metric_names = set()
        for result in all_results:
            metric_names.update(result["metrics"].keys())

        # Calculate average for each metric
        for metric_name in metric_names:
            values = []
            for result in all_results:
                val = result["metrics"].get(metric_name)
                if isinstance(val, (int, float)):
                    values.append(val)

            if values:
                aggregated["metrics"][metric_name] = {
                    "mean": np.mean(values),
                    "std": np.std(values),
                    "min": np.min(values),
                    "max": np.max(values)
                }

        aggregated["individual_results"] = all_results

        return aggregated

# Usage
evaluator = EvaluationSuite()

# Evaluate single response
result = evaluator.evaluate_response(
    prompt="Summarize the benefits of cloud computing",
    response="Cloud computing offers scalability, cost savings, and flexibility for businesses.",
    reference="Cloud computing provides scalable resources, reduced costs, and operational flexibility.",
    source_context="Cloud computing enables organizations to access computing resources on demand..."
)

print(f"Overall score: {result['overall_score']:.3f}")
print(f"Metrics: {result['metrics']}")

# Evaluate dataset
evaluation_data = [
    {
        "prompt": "What is machine learning?",
        "response": "Machine learning is a subset of AI...",
        "reference": "Machine learning is an AI technique..."
    },
    # More examples...
]

dataset_results = evaluator.evaluate_dataset(evaluation_data)
print(f"Dataset performance: {dataset_results['metrics']}")
```

### LLM-as-Judge Evaluation

```python
from vertexai.preview.generative_models import GenerativeModel
from typing import List, Dict
import json

class LLMJudge:
    """Use LLM as a judge for evaluating Gen AI outputs"""

    def __init__(self, judge_model: str = "gemini-pro"):
        self.judge = GenerativeModel(judge_model)

    def evaluate_quality(
        self,
        prompt: str,
        response: str,
        criteria: List[str] = None
    ) -> Dict[str, any]:
        """Evaluate response quality using LLM judge"""

        if criteria is None:
            criteria = [
                "Relevance",
                "Accuracy",
                "Completeness",
                "Clarity",
                "Helpfulness"
            ]

        evaluation_prompt = f"""You are an expert evaluator. Evaluate the following AI response based on these criteria: {', '.join(criteria)}.

User Prompt:
{prompt}

AI Response:
{response}

For each criterion, provide:
1. Score (1-5, where 5 is best)
2. Brief justification

Also provide:
- Overall score (1-5)
- Key strengths
- Areas for improvement

Format your response as JSON with this structure:
{{
    "criteria_scores": {{
        "Relevance": {{"score": X, "justification": "..."}},
        ...
    }},
    "overall_score": X,
    "overall_justification": "...",
    "strengths": ["...", "..."],
    "improvements": ["...", "..."]
}}"""

        judge_response = self.judge.generate_content(evaluation_prompt)

        try:
            # Extract JSON from response
            json_match = re.search(r'\{.*\}', judge_response.text, re.DOTALL)
            if json_match:
                evaluation = json.loads(json_match.group())
                return evaluation
            else:
                return {"error": "Could not parse judge response", "raw": judge_response.text}
        except json.JSONDecodeError:
            return {"error": "Invalid JSON from judge", "raw": judge_response.text}

    def compare_responses(
        self,
        prompt: str,
        response_a: str,
        response_b: str,
        model_a_name: str = "Model A",
        model_b_name: str = "Model B"
    ) -> Dict[str, any]:
        """Compare two responses using LLM judge"""

        comparison_prompt = f"""You are an expert evaluator comparing two AI responses to the same prompt.

User Prompt:
{prompt}

{model_a_name} Response:
{response_a}

{model_b_name} Response:
{response_b}

Compare these responses on:
1. Relevance to the prompt
2. Accuracy of information
3. Clarity and coherence
4. Completeness
5. Overall quality

Provide:
- Winner: [A/B/Tie]
- Confidence: [Low/Medium/High]
- Detailed comparison
- Specific advantages of each response

Format as JSON:
{{
    "winner": "A|B|Tie",
    "confidence": "Low|Medium|High",
    "scores": {{
        "response_a": X,
        "response_b": Y
    }},
    "comparison": "...",
    "response_a_advantages": ["...", "..."],
    "response_b_advantages": ["...", "..."]
}}"""

        judge_response = self.judge.generate_content(comparison_prompt)

        try:
            json_match = re.search(r'\{.*\}', judge_response.text, re.DOTALL)
            if json_match:
                comparison = json.loads(json_match.group())
                return comparison
            else:
                return {"error": "Could not parse comparison", "raw": judge_response.text}
        except json.JSONDecodeError:
            return {"error": "Invalid JSON", "raw": judge_response.text}

    def evaluate_batch(
        self,
        prompts: List[str],
        responses: List[str],
        references: List[str] = None
    ) -> List[Dict]:
        """Evaluate a batch of responses"""

        results = []

        for i, (prompt, response) in enumerate(zip(prompts, responses)):
            evaluation = self.evaluate_quality(prompt, response)

            # Add reference comparison if available
            if references and i < len(references):
                reference_eval = self.compare_responses(
                    prompt=prompt,
                    response_a=response,
                    response_b=references[i],
                    model_a_name="Generated",
                    model_b_name="Reference"
                )
                evaluation["reference_comparison"] = reference_eval

            results.append(evaluation)

        return results

    def calculate_win_rate(
        self,
        comparison_results: List[Dict],
        model_name: str
    ) -> float:
        """Calculate win rate for a model from comparison results"""

        wins = sum(
            1 for result in comparison_results
            if result.get("winner", "").upper() in ["A", model_name.upper()]
        )

        return wins / len(comparison_results) if comparison_results else 0.0

# Usage
judge = LLMJudge(judge_model="gemini-pro")

# Evaluate single response
prompt = "Explain quantum computing in simple terms"
response = "Quantum computing uses quantum mechanics principles..."

evaluation = judge.evaluate_quality(prompt, response)
print(f"Overall score: {evaluation.get('overall_score')}")
print(f"Strengths: {evaluation.get('strengths')}")
print(f"Improvements: {evaluation.get('improvements')}")

# Compare two model responses
response_a = "Model A's response..."
response_b = "Model B's response..."

comparison = judge.compare_responses(
    prompt=prompt,
    response_a=response_a,
    response_b=response_b,
    model_a_name="gemini-pro",
    model_b_name="text-bison"
)

print(f"Winner: {comparison.get('winner')}")
print(f"Confidence: {comparison.get('confidence')}")
```

## Complete Production MLOps Pipeline

### End-to-End Pipeline with Monitoring

```python
from google.cloud import aiplatform, storage, bigquery, firestore
from kfp import dsl, compiler
from vertexai.preview.generative_models import GenerativeModel
from typing import Dict, List
import time
import json

@dsl.component(
    base_image="python:3.10",
    packages_to_install=[
        "google-cloud-aiplatform",
        "google-cloud-storage",
        "google-cloud-bigquery",
        "vertexai"
    ]
)
def prepare_training_data(
    source_bucket: str,
    source_prefix: str,
    output_jsonl_path: dsl.OutputPath(str),
    train_split: float = 0.8
):
    """Prepare and split training data"""
    from google.cloud import storage
    import json
    import random

    client = storage.Client()
    bucket = client.bucket(source_bucket)
    blobs = list(bucket.list_blobs(prefix=source_prefix))

    data = []
    for blob in blobs:
        if blob.name.endswith('.json'):
            content = json.loads(blob.download_as_text())
            data.append(content)

    # Shuffle and split
    random.shuffle(data)
    split_idx = int(len(data) * train_split)
    train_data = data[:split_idx]

    # Write training data in JSONL format
    with open(output_jsonl_path, 'w') as f:
        for item in train_data:
            f.write(json.dumps(item) + '\n')

    print(f"Prepared {len(train_data)} training examples")

@dsl.component(
    base_image="python:3.10",
    packages_to_install=["google-cloud-aiplatform", "vertexai"]
)
def tune_foundation_model(
    training_data_path: dsl.InputPath(str),
    base_model: str,
    tuning_steps: int,
    learning_rate: float,
    tuned_model_output: dsl.OutputPath(str)
):
    """Tune foundation model"""
    from vertexai.preview.language_models import TextGenerationModel
    import json

    # Load base model
    model = TextGenerationModel.from_pretrained(base_model)

    # Start tuning job
    tuning_job = model.tune_model(
        training_data=training_data_path,
        train_steps=tuning_steps,
        learning_rate=learning_rate,
        tuning_job_location="us-central1"
    )

    # Wait for completion
    print(f"Tuning job started: {tuning_job.tuning_job_id}")
    tuning_job.wait()

    # Save model info
    model_info = {
        "tuned_model_name": tuning_job.tuned_model_name,
        "tuning_job_id": tuning_job.tuning_job_id,
        "base_model": base_model
    }

    with open(tuned_model_output, 'w') as f:
        json.dump(model_info, f)

    print(f"Tuning complete: {tuning_job.tuned_model_name}")

@dsl.component(
    base_image="python:3.10",
    packages_to_install=["google-cloud-aiplatform", "vertexai", "numpy"]
)
def evaluate_tuned_model(
    tuned_model_path: dsl.InputPath(str),
    eval_data_path: dsl.InputPath(str),
    metrics_output: dsl.OutputPath(str)
):
    """Evaluate tuned model"""
    from vertexai.preview.language_models import TextGenerationModel
    import json
    import numpy as np

    # Load model info
    with open(tuned_model_path, 'r') as f:
        model_info = json.load(f)

    model = TextGenerationModel.from_pretrained(model_info["tuned_model_name"])

    # Load evaluation data
    with open(eval_data_path, 'r') as f:
        eval_data = [json.loads(line) for line in f]

    # Run evaluation
    scores = []
    for item in eval_data[:100]:  # Sample for speed
        try:
            response = model.predict(
                item["input_text"],
                max_output_tokens=256
            )

            # Simple scoring (in practice, use more sophisticated metrics)
            reference = item.get("output_text", "")
            score = len(set(response.text.split()) & set(reference.split())) / max(
                len(set(response.text.split())), 1
            )
            scores.append(score)
        except Exception as e:
            print(f"Evaluation error: {e}")
            continue

    # Calculate metrics
    metrics = {
        "avg_score": float(np.mean(scores)),
        "std_score": float(np.std(scores)),
        "min_score": float(np.min(scores)),
        "max_score": float(np.max(scores)),
        "num_evaluated": len(scores)
    }

    with open(metrics_output, 'w') as f:
        json.dump(metrics, f)

    print(f"Evaluation metrics: {metrics}")

@dsl.component(
    base_image="python:3.10",
    packages_to_install=["google-cloud-aiplatform"]
)
def deploy_to_endpoint(
    tuned_model_path: dsl.InputPath(str),
    endpoint_name: str,
    traffic_percentage: int,
    project_id: str,
    location: str
):
    """Deploy model to endpoint"""
    from google.cloud import aiplatform
    import json

    aiplatform.init(project=project_id, location=location)

    # Load model info
    with open(tuned_model_path, 'r') as f:
        model_info = json.load(f)

    # Get or create endpoint
    endpoints = aiplatform.Endpoint.list(
        filter=f'display_name="{endpoint_name}"'
    )

    if endpoints:
        endpoint = endpoints[0]
        print(f"Using existing endpoint: {endpoint.resource_name}")
    else:
        endpoint = aiplatform.Endpoint.create(display_name=endpoint_name)
        print(f"Created new endpoint: {endpoint.resource_name}")

    # Deploy model
    model = aiplatform.Model(model_info["tuned_model_name"])

    endpoint.deploy(
        model=model,
        deployed_model_display_name=f"deployment-{int(time.time())}",
        machine_type="n1-standard-4",
        min_replica_count=1,
        max_replica_count=5,
        traffic_percentage=traffic_percentage
    )

    print(f"Deployed model with {traffic_percentage}% traffic")

@dsl.component(
    base_image="python:3.10",
    packages_to_install=["google-cloud-monitoring"]
)
def setup_monitoring(
    project_id: str,
    endpoint_name: str,
    alert_threshold_latency: float,
    alert_threshold_error_rate: float
):
    """Setup monitoring and alerts"""
    from google.cloud import monitoring_v3

    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    # Create latency alert
    latency_policy = monitoring_v3.AlertPolicy(
        display_name=f"{endpoint_name}-high-latency",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="High latency",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter=f'metric.type="custom.googleapis.com/genai/prediction_latency"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=alert_threshold_latency,
                    duration={"seconds": 300}
                )
            )
        ],
        combiner=monitoring_v3.AlertPolicy.ConditionCombinerType.AND
    )

    client.create_alert_policy(
        name=project_name,
        alert_policy=latency_policy
    )

    print(f"Created monitoring alerts for {endpoint_name}")

@dsl.pipeline(
    name="genai-production-mlops-pipeline",
    description="Complete Gen AI MLOps pipeline with monitoring"
)
def genai_production_pipeline(
    project_id: str,
    location: str,
    source_bucket: str,
    source_prefix: str,
    base_model: str = "text-bison@002",
    tuning_steps: int = 100,
    learning_rate: float = 0.001,
    endpoint_name: str = "genai-production",
    traffic_percentage: int = 100,
    alert_threshold_latency: float = 1000.0,
    alert_threshold_error_rate: float = 0.05
):
    """End-to-end Gen AI MLOps pipeline"""

    # Step 1: Prepare training data
    prepare_task = prepare_training_data(
        source_bucket=source_bucket,
        source_prefix=source_prefix,
        train_split=0.8
    )

    # Step 2: Tune model
    tune_task = tune_foundation_model(
        training_data_path=prepare_task.outputs["output_jsonl_path"],
        base_model=base_model,
        tuning_steps=tuning_steps,
        learning_rate=learning_rate
    )

    # Step 3: Evaluate model
    eval_task = evaluate_tuned_model(
        tuned_model_path=tune_task.outputs["tuned_model_output"],
        eval_data_path=prepare_task.outputs["output_jsonl_path"]
    )

    # Step 4: Deploy to endpoint
    deploy_task = deploy_to_endpoint(
        tuned_model_path=tune_task.outputs["tuned_model_output"],
        endpoint_name=endpoint_name,
        traffic_percentage=traffic_percentage,
        project_id=project_id,
        location=location
    ).after(eval_task)  # Only deploy if evaluation passes

    # Step 5: Setup monitoring
    monitor_task = setup_monitoring(
        project_id=project_id,
        endpoint_name=endpoint_name,
        alert_threshold_latency=alert_threshold_latency,
        alert_threshold_error_rate=alert_threshold_error_rate
    ).after(deploy_task)

# Compile and run pipeline
def run_production_pipeline(
    project_id: str,
    location: str = "us-central1"
):
    """Compile and execute production pipeline"""

    # Compile pipeline
    compiler.Compiler().compile(
        pipeline_func=genai_production_pipeline,
        package_path="genai_production_pipeline.json"
    )

    # Initialize Vertex AI
    aiplatform.init(project=project_id, location=location)

    # Create and run pipeline job
    pipeline_job = aiplatform.PipelineJob(
        display_name="genai-production-mlops",
        template_path="genai_production_pipeline.json",
        parameter_values={
            "project_id": project_id,
            "location": location,
            "source_bucket": "my-training-data",
            "source_prefix": "genai-training/",
            "base_model": "text-bison@002",
            "tuning_steps": 100,
            "learning_rate": 0.001,
            "endpoint_name": "genai-production",
            "traffic_percentage": 10,  # Start with canary
            "alert_threshold_latency": 1000.0,
            "alert_threshold_error_rate": 0.05
        },
        enable_caching=True
    )

    pipeline_job.run(sync=False)
    print(f"Pipeline running: {pipeline_job.resource_name}")

    return pipeline_job

# Execute pipeline
if __name__ == "__main__":
    job = run_production_pipeline(
        project_id="my-project",
        location="us-central1"
    )
```

## Production Scenarios and Best Practices

### Scenario 1: Customer Support Chatbot

**Requirements:**
- Real-time response generation
- PII detection and redaction
- Quality monitoring
- Cost optimization
- 24/7 availability

**Implementation:**

```python
from google.cloud import aiplatform, dlp_v2
from vertexai.preview.generative_models import GenerativeModel
import redis
import time

class CustomerSupportChatbot:
    def __init__(self, project_id: str, location: str):
        self.project_id = project_id
        self.location = location

        # Initialize model
        self.model = GenerativeModel("gemini-pro")

        # Initialize cache
        self.cache = redis.Redis(host="10.0.0.3", decode_responses=True)

        # Initialize DLP for PII
        self.dlp_client = dlp_v2.DlpServiceClient()

        # System prompt
        self.system_prompt = """You are a helpful customer support assistant.
- Be professional and empathetic
- Provide accurate information
- If you don't know something, say so
- Never share customer PII
- Keep responses concise (< 200 words)"""

    def detect_and_redact_pii(self, text: str) -> tuple:
        """Detect and redact PII"""

        inspect_config = {
            "info_types": [
                {"name": "EMAIL_ADDRESS"},
                {"name": "PHONE_NUMBER"},
                {"name": "CREDIT_CARD_NUMBER"},
                {"name": "PERSON_NAME"}
            ]
        }

        deidentify_config = {
            "info_type_transformations": {
                "transformations": [{
                    "primitive_transformation": {
                        "replace_with_info_type_config": {}
                    }
                }]
            }
        }

        response = self.dlp_client.deidentify_content(
            request={
                "parent": f"projects/{self.project_id}",
                "deidentify_config": deidentify_config,
                "inspect_config": inspect_config,
                "item": {"value": text}
            }
        )

        return response.item.value, len(response.overview.transformation_summaries) > 0

    def generate_response(
        self,
        user_message: str,
        conversation_history: list = None
    ) -> dict:
        """Generate chatbot response with safety and monitoring"""

        start_time = time.time()

        # Check cache
        cache_key = f"chatbot:{user_message}"
        cached = self.cache.get(cache_key)

        if cached:
            return {
                "response": cached,
                "cached": True,
                "latency_ms": (time.time() - start_time) * 1000
            }

        # Redact PII from user message
        safe_message, had_pii = self.detect_and_redact_pii(user_message)

        # Build context
        context = self.system_prompt
        if conversation_history:
            for turn in conversation_history[-3:]:  # Last 3 turns
                context += f"\nUser: {turn['user']}\nAssistant: {turn['assistant']}"

        # Generate response
        full_prompt = f"{context}\nUser: {safe_message}\nAssistant:"

        response = self.model.generate_content(
            full_prompt,
            generation_config={
                "max_output_tokens": 256,
                "temperature": 0.7,
                "top_p": 0.9
            }
        )

        response_text = response.text

        # Verify no PII in response
        safe_response, response_had_pii = self.detect_and_redact_pii(response_text)

        # Cache response
        self.cache.setex(cache_key, 3600, safe_response)

        latency = (time.time() - start_time) * 1000

        return {
            "response": safe_response,
            "cached": False,
            "latency_ms": latency,
            "had_input_pii": had_pii,
            "had_output_pii": response_had_pii,
            "safety_ratings": response.candidates[0].safety_ratings
        }

# Usage
chatbot = CustomerSupportChatbot(
    project_id="my-project",
    location="us-central1"
)

result = chatbot.generate_response(
    user_message="I need help with my order #12345. My email is john@example.com",
    conversation_history=[]
)

print(f"Response: {result['response']}")
print(f"Latency: {result['latency_ms']:.2f}ms")
print(f"PII detected: {result['had_input_pii']}")
```

### Scenario 2: Document Summarization at Scale

**Requirements:**
- Batch processing of documents
- Quality assurance
- Cost tracking
- Progress monitoring

**Implementation:**

```python
from google.cloud import storage, bigquery
from vertexai.preview.generative_models import GenerativeModel
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

class DocumentSummarizationPipeline:
    def __init__(self, project_id: str):
        self.project_id = project_id
        self.model = GenerativeModel("gemini-pro")
        self.storage_client = storage.Client()
        self.bq_client = bigquery.Client()

        # Initialize tracking table
        self._setup_tracking_table()

    def _setup_tracking_table(self):
        """Setup BigQuery tracking table"""

        schema = [
            bigquery.SchemaField("document_id", "STRING"),
            bigquery.SchemaField("timestamp", "TIMESTAMP"),
            bigquery.SchemaField("status", "STRING"),
            bigquery.SchemaField("input_tokens", "INTEGER"),
            bigquery.SchemaField("output_tokens", "INTEGER"),
            bigquery.SchemaField("cost", "FLOAT"),
            bigquery.SchemaField("latency_ms", "FLOAT"),
            bigquery.SchemaField("summary_length", "INTEGER")
        ]

        table_id = f"{self.project_id}.genai_monitoring.summarization_tracking"

        try:
            table = bigquery.Table(table_id, schema=schema)
            self.bq_client.create_table(table, exists_ok=True)
        except Exception as e:
            print(f"Table setup error: {e}")

    def summarize_document(
        self,
        document_text: str,
        document_id: str,
        max_summary_length: int = 200
    ) -> dict:
        """Summarize single document"""

        start_time = time.time()

        prompt = f"""Summarize the following document in {max_summary_length} words or less.
Be concise and capture the main points.

Document:
{document_text[:4000]}  # Limit input length

Summary:"""

        try:
            response = self.model.generate_content(
                prompt,
                generation_config={
                    "max_output_tokens": max_summary_length * 2,
                    "temperature": 0.3
                }
            )

            summary = response.text

            # Calculate costs (approximate)
            input_tokens = len(prompt.split()) * 1.3
            output_tokens = len(summary.split()) * 1.3
            cost = (input_tokens / 1000 * 0.000125) + (output_tokens / 1000 * 0.000375)

            latency = (time.time() - start_time) * 1000

            # Track in BigQuery
            self._track_summary(
                document_id=document_id,
                status="SUCCESS",
                input_tokens=int(input_tokens),
                output_tokens=int(output_tokens),
                cost=cost,
                latency_ms=latency,
                summary_length=len(summary)
            )

            return {
                "document_id": document_id,
                "summary": summary,
                "success": True,
                "cost": cost,
                "latency_ms": latency
            }

        except Exception as e:
            self._track_summary(
                document_id=document_id,
                status="FAILED",
                input_tokens=0,
                output_tokens=0,
                cost=0,
                latency_ms=0,
                summary_length=0
            )

            return {
                "document_id": document_id,
                "success": False,
                "error": str(e)
            }

    def _track_summary(self, **kwargs):
        """Track summarization in BigQuery"""

        from datetime import datetime

        row = {
            "document_id": kwargs["document_id"],
            "timestamp": datetime.utcnow().isoformat(),
            "status": kwargs["status"],
            "input_tokens": kwargs["input_tokens"],
            "output_tokens": kwargs["output_tokens"],
            "cost": kwargs["cost"],
            "latency_ms": kwargs["latency_ms"],
            "summary_length": kwargs["summary_length"]
        }

        table_id = f"{self.project_id}.genai_monitoring.summarization_tracking"
        errors = self.bq_client.insert_rows_json(table_id, [row])

        if errors:
            print(f"Tracking error: {errors}")

    def batch_summarize(
        self,
        bucket_name: str,
        prefix: str,
        max_workers: int = 10
    ) -> dict:
        """Batch summarize documents from Cloud Storage"""

        bucket = self.storage_client.bucket(bucket_name)
        blobs = list(bucket.list_blobs(prefix=prefix))

        results = []
        total_cost = 0

        with ThreadPoolExecutor(max_workers=max_workers) as executor:
            futures = {}

            for blob in blobs:
                if blob.name.endswith('.txt'):
                    document_text = blob.download_as_text()
                    document_id = blob.name

                    future = executor.submit(
                        self.summarize_document,
                        document_text,
                        document_id
                    )
                    futures[future] = document_id

            for future in as_completed(futures):
                result = future.result()
                results.append(result)

                if result["success"]:
                    total_cost += result["cost"]

                    # Save summary back to GCS
                    summary_blob = bucket.blob(f"summaries/{futures[future]}.summary.txt")
                    summary_blob.upload_from_string(result["summary"])

                print(f"Processed {len(results)}/{len(futures)} documents")

        return {
            "total_documents": len(results),
            "successful": sum(1 for r in results if r["success"]),
            "failed": sum(1 for r in results if not r["success"]),
            "total_cost": total_cost,
            "results": results
        }

# Usage
pipeline = DocumentSummarizationPipeline("my-project")

# Batch summarize all documents
results = pipeline.batch_summarize(
    bucket_name="my-documents",
    prefix="documents/",
    max_workers=10
)

print(f"Summarized {results['successful']} documents")
print(f"Total cost: ${results['total_cost']:.4f}")
```

### Scenario 3: Multi-Model RAG System

**Requirements:**
- Vector search integration
- Multiple model fallback
- Citation tracking
- Response verification

**Implementation:**

```python
from google.cloud import aiplatform
from vertexai.preview.language_models import TextEmbeddingModel
from vertexai.preview.generative_models import GenerativeModel
from typing import List, Dict
import numpy as np

class RAGSystem:
    def __init__(self, project_id: str, location: str):
        self.project_id = project_id
        self.location = location

        # Initialize models
        self.embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")
        self.primary_model = GenerativeModel("gemini-pro")
        self.fallback_model = GenerativeModel("gemini-pro")

        # Initialize vector search
        aiplatform.init(project=project_id, location=location)
        self.index_endpoint = None  # Configure with your vector search endpoint

    def embed_query(self, query: str) -> List[float]:
        """Generate embedding for query"""

        embedding = self.embedding_model.get_embeddings([query])[0]
        return embedding.values

    def retrieve_context(
        self,
        query: str,
        top_k: int = 5
    ) -> List[Dict]:
        """Retrieve relevant context from vector search"""

        # Get query embedding
        query_embedding = self.embed_query(query)

        # Search vector database
        # This is a simplified example - implement with your vector DB
        retrieved_docs = [
            {
                "content": "Sample document content...",
                "score": 0.85,
                "source": "doc_123.pdf",
                "page": 5
            }
            # More results...
        ]

        return retrieved_docs[:top_k]

    def generate_with_context(
        self,
        query: str,
        context_docs: List[Dict],
        include_citations: bool = True
    ) -> Dict:
        """Generate response with retrieved context"""

        # Build context string
        context_str = "\n\n".join([
            f"[{i+1}] {doc['content']} (Source: {doc['source']}, Page: {doc['page']})"
            for i, doc in enumerate(context_docs)
        ])

        # Build prompt
        prompt = f"""Answer the question based on the provided context. If the answer is not in the context, say so.
{"Include citations [1], [2], etc. in your response." if include_citations else ""}

Context:
{context_str}

Question: {query}

Answer:"""

        try:
            # Try primary model
            response = self.primary_model.generate_content(
                prompt,
                generation_config={
                    "max_output_tokens": 512,
                    "temperature": 0.2,
                    "top_p": 0.9
                }
            )

            answer = response.text
            model_used = "gemini-pro (primary)"

        except Exception as e:
            print(f"Primary model failed: {e}, trying fallback")

            # Fallback model
            response = self.fallback_model.generate_content(prompt)
            answer = response.text
            model_used = "gemini-pro (fallback)"

        # Extract citations
        citations = []
        if include_citations:
            import re
            citation_refs = re.findall(r'\[(\d+)\]', answer)
            citations = [
                context_docs[int(ref)-1]
                for ref in citation_refs
                if 0 < int(ref) <= len(context_docs)
            ]

        return {
            "answer": answer,
            "citations": citations,
            "model_used": model_used,
            "context_docs": context_docs,
            "num_sources": len(context_docs)
        }

    def verify_answer(
        self,
        query: str,
        answer: str,
        context_docs: List[Dict]
    ) -> Dict:
        """Verify answer against context"""

        context_str = "\n\n".join([doc['content'] for doc in context_docs])

        verification_prompt = f"""Verify if the answer is supported by the context.

Context:
{context_str}

Question: {query}

Answer: {answer}

Is the answer fully supported by the context? Respond with:
- SUPPORTED: If fully supported
- PARTIALLY_SUPPORTED: If partially supported
- NOT_SUPPORTED: If not supported
- Cannot be determined from context

Also explain your reasoning briefly."""

        verification = self.primary_model.generate_content(verification_prompt)

        return {
            "verification_result": verification.text,
            "is_supported": "SUPPORTED" in verification.text.upper()
        }

    def query(
        self,
        question: str,
        top_k: int = 5,
        verify: bool = True
    ) -> Dict:
        """Complete RAG query pipeline"""

        # Retrieve context
        context_docs = self.retrieve_context(question, top_k=top_k)

        # Generate answer
        result = self.generate_with_context(
            query=question,
            context_docs=context_docs,
            include_citations=True
        )

        # Verify if requested
        if verify:
            verification = self.verify_answer(
                query=question,
                answer=result["answer"],
                context_docs=context_docs
            )
            result["verification"] = verification

        return result

# Usage
rag_system = RAGSystem(
    project_id="my-project",
    location="us-central1"
)

# Query with RAG
result = rag_system.query(
    question="What are the key benefits of cloud computing?",
    top_k=5,
    verify=True
)

print(f"Answer: {result['answer']}")
print(f"Sources used: {result['num_sources']}")
print(f"Citations: {len(result['citations'])}")
print(f"Verified: {result['verification']['is_supported']}")
```

## Exam Tips and Best Practices

### Gen AI MLOps Key Concepts

**1. Production Deployment:**
- Use Vertex AI Endpoints for serving
- Implement auto-scaling (1-10+ replicas typical)
- Enable caching for repeated queries (can reduce costs 50%+)
- Deploy across multiple regions for low latency
- Use private endpoints for sensitive workloads

**2. Monitoring Requirements:**
```
Track these metrics:
- Latency (P50, P95, P99)
- Token usage (input/output)
- Cost per request
- Error rates
- Safety filter triggers
- User satisfaction scores
```

**3. Cost Optimization:**
- Cache common queries (Redis/Memorystore)
- Use smaller models when possible (text-bison vs gemini-pro)
- Implement request batching
- Set quota limits per user
- Monitor and alert on cost thresholds
- Consider tuned models vs base models (10x cost difference)

**4. Safety and Compliance:**
- Always use safety settings in production
- Implement PII detection (Cloud DLP)
- Log all requests for audit
- Redact sensitive information
- Use content filtering
- Track safety attribute violations

**5. Model Versioning:**
- Version prompts in Cloud Storage
- Use A/B testing for prompt changes
- Implement canary deployments (10% → 50% → 100%)
- Keep rollback capability
- Track model performance by version

**6. Evaluation Frameworks:**
- Use BLEU/ROUGE for summarization tasks
- Implement LLM-as-judge for quality
- Collect human feedback
- Track user satisfaction
- Monitor task-specific metrics
- Use semantic similarity for retrieval

### Common Exam Scenarios

**Scenario: Reduce Gen AI costs**
Answer: Implement response caching, use smaller models for simple tasks, batch requests, set per-user quotas, optimize prompts to reduce token usage

**Scenario: Deploy new model version safely**
Answer: Use canary deployment starting at 10% traffic, monitor latency and error rates, gradually increase to 50% then 100%, keep rollback capability

**Scenario: Monitor model quality in production**
Answer: Track user feedback scores, implement automated evaluation metrics, use LLM-as-judge for quality assessment, monitor safety filter triggers, log all requests to BigQuery

**Scenario: Ensure content safety**
Answer: Configure safety settings (BLOCK_MEDIUM_AND_ABOVE), use Cloud DLP for PII detection, implement toxicity checking, add custom safety classifiers, audit all responses

**Scenario: Scale globally with low latency**
Answer: Deploy to multiple regions (us-central1, europe-west4, asia-southeast1), use load balancing, implement regional caching, optimize model selection by region

**Scenario: Track and control costs**
Answer: Implement token tracking to BigQuery, set up cost alerts in Cloud Monitoring, use budget notifications, cache frequently asked questions, implement rate limiting

**Scenario: Implement RAG system**
Answer: Use Vertex AI Vector Search for retrieval, implement embedding pipeline, cite sources in responses, verify factual consistency, handle missing context gracefully

**Scenario: Handle PII in prompts**
Answer: Use Cloud DLP to detect PII, redact before sending to model, audit PII detection, comply with data privacy regulations, implement data retention policies

### Production Checklist

**Before Production:**
- [ ] Safety settings configured
- [ ] Monitoring dashboard created
- [ ] Cost alerts set up
- [ ] PII detection implemented
- [ ] Caching configured
- [ ] Rate limiting enabled
- [ ] Fallback models configured
- [ ] Evaluation metrics defined
- [ ] Canary deployment plan ready
- [ ] Rollback procedure documented

**In Production:**
- [ ] Monitor latency (target < 1s P95)
- [ ] Track cost per request
- [ ] Collect user feedback
- [ ] Review safety violations
- [ ] Analyze cache hit rate
- [ ] Monitor error rates
- [ ] Check model version performance
- [ ] Review audit logs
- [ ] Update prompts based on feedback
- [ ] Optimize costs monthly

### Key Services for Gen AI MLOps

**Vertex AI:**
- Model Garden (foundation models)
- Model Tuning (supervised fine-tuning)
- Endpoints (model serving)
- Pipelines (MLOps workflows)
- Experiments (tracking)
- Feature Store (for RAG)
- Vector Search (embeddings)

**Monitoring:**
- Cloud Monitoring (metrics)
- Cloud Logging (request logs)
- BigQuery (analytics)
- Firestore (user feedback)

**Safety:**
- Cloud DLP (PII detection)
- Perspective API (toxicity)
- Safety attributes (built-in)
- Custom classifiers (AutoML)

**Cost Management:**
- Budget alerts
- Quota management
- Resource monitoring
- BigQuery cost analysis

### Exam Tips

1. **Understand token pricing**: Input vs output tokens have different costs, Gemini is cheaper than PaLM 2
2. **Know safety levels**: BLOCK_NONE, BLOCK_ONLY_HIGH, BLOCK_MEDIUM_AND_ABOVE, BLOCK_LOW_AND_ABOVE
3. **Caching strategies**: Exact match (Redis) vs semantic similarity (vector search)
4. **Deployment patterns**: Canary (gradual), A/B testing (comparison), Blue/Green (instant switch)
5. **Monitoring metrics**: Latency P95 < 1s, error rate < 1%, user satisfaction > 4.0/5.0
6. **Cost optimization**: Cache hit rate > 30%, smaller models for 70% of requests, batch when possible
7. **Model selection**: Use Gemini for multimodal, PaLM for text generation, embeddings for search
8. **Prompt management**: Version control, A/B testing, automated testing, template management
9. **Evaluation**: BLEU/ROUGE for generation, semantic similarity for retrieval, LLM-as-judge for quality
10. **Compliance**: PII detection mandatory, audit logging required, data retention policies, model cards

### Remember

- **Always** implement safety settings in production
- **Always** monitor costs and set budgets
- **Always** version prompts and models
- **Always** collect user feedback
- **Always** implement caching for cost optimization
- **Never** skip PII detection for user-facing applications
- **Never** deploy without canary testing
- **Never** forget to set up monitoring before production
- **Prefer** managed services (Vertex AI) over custom solutions
- **Prefer** smaller models when accuracy allows
