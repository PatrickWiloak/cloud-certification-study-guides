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

## Exam Tips

### Key Concepts

- Vertex AI Pipelines for GenAI workflows
- Prompt versioning and management
- A/B testing and canary deployments
- Monitoring response quality and cost
- Caching strategies for optimization
- User feedback collection
- Token usage tracking

### Common Scenarios

- **Deploy new model version** → Canary deployment with monitoring
- **Reduce costs** → Caching, model selection, prompt optimization
- **Track model performance** → Custom metrics, logging, user feedback
- **Version prompts** → Cloud Storage, version control, testing
- **Ensure quality** → A/B testing, evaluation metrics, user ratings
- **Scale globally** → Multi-region endpoints, caching, load balancing
