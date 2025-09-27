# Azure OpenAI Service Fundamentals

## What is Azure OpenAI Service?

Azure OpenAI Service provides REST API access to OpenAI's powerful language models including GPT-4, GPT-3.5-turbo, Codex, DALL-E, and Embeddings models through Azure's secure, enterprise-ready infrastructure.

### Key Benefits
- **Enterprise security**: Built on Azure's security and compliance framework
- **Regional availability**: Deploy in multiple Azure regions
- **Private networking**: VNet integration and private endpoints
- **Responsible AI**: Built-in content filtering and safety measures
- **Unified billing**: Integrated with Azure billing and cost management

## Available Models

### GPT-4 Models
| Model | Description | Context Length | Use Cases |
|-------|-------------|----------------|-----------|
| **GPT-4** | Most capable model | 8,192 tokens | Complex reasoning, analysis |
| **GPT-4-32k** | Extended context version | 32,768 tokens | Long document analysis |
| **GPT-4 Turbo** | Faster, more efficient | 128,000 tokens | High-throughput applications |
| **GPT-4 Vision** | Multimodal capabilities | 128,000 tokens | Image understanding and analysis |

#### GPT-4 Capabilities
- **Advanced reasoning**: Complex problem-solving and analysis
- **Code generation**: High-quality programming assistance
- **Multimodal**: Text and image understanding (Vision model)
- **Long context**: Extended conversation and document understanding
- **Instruction following**: Precise task execution

### GPT-3.5 Models
| Model | Description | Context Length | Use Cases |
|-------|-------------|----------------|-----------|
| **GPT-3.5-turbo** | Fast, cost-effective | 4,096 tokens | Chatbots, simple tasks |
| **GPT-3.5-turbo-16k** | Extended context | 16,384 tokens | Longer conversations |
| **GPT-3.5-turbo-instruct** | Completion model | 4,096 tokens | Text completion tasks |

#### GPT-3.5 Features
- **Cost-effective**: Lower cost than GPT-4
- **Fast responses**: Quick inference times
- **General purpose**: Good for most common tasks
- **Chat optimized**: Designed for conversational applications

### Codex Models
| Model | Description | Use Cases |
|-------|-------------|-----------|
| **code-davinci-002** | Most capable code model | Code generation, debugging |
| **code-cushman-001** | Faster code model | Simple code tasks |

#### Codex Capabilities
- **Code generation**: Create code from natural language
- **Code completion**: Complete partial code snippets
- **Code explanation**: Explain existing code
- **Debugging**: Identify and fix code issues
- **Multiple languages**: Support for many programming languages

### DALL-E Models
| Model | Description | Capabilities |
|-------|-------------|-------------|
| **DALL-E 3** | Latest image generation | High-quality, detailed images |
| **DALL-E 2** | Image generation | Creative image synthesis |

#### DALL-E Features
- **Text-to-image**: Generate images from text descriptions
- **High resolution**: Create detailed, high-quality images
- **Style control**: Various artistic styles and approaches
- **Safety filters**: Built-in content safety measures

### Embeddings Models
| Model | Description | Use Cases |
|-------|-------------|-----------|
| **text-embedding-ada-002** | High-performance embeddings | Semantic search, similarity |
| **text-embedding-3-small** | Smaller, efficient model | Cost-effective embeddings |
| **text-embedding-3-large** | Largest embeddings model | Highest quality representations |

## Service Configuration

### Resource Creation
```bash
# Create Azure OpenAI resource
az cognitiveservices account create \
  --name myopenai \
  --resource-group myRG \
  --location eastus \
  --kind OpenAI \
  --sku S0 \
  --custom-domain myopenai
```

### Model Deployment
```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Initialize client
credential = DefaultAzureCredential()
ml_client = MLClient(
    credential=credential,
    subscription_id="your-subscription-id",
    resource_group_name="your-resource-group",
    workspace_name="your-workspace"
)

# Deploy model
deployment = ml_client.online_deployments.create_or_update(
    deployment_name="gpt-4-deployment",
    model_name="gpt-4",
    instance_count=1,
    instance_type="Standard_DS3_v2"
)
```

## API Integration

### Python SDK Usage
```python
import openai
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

# Configure Azure OpenAI
openai.api_type = "azure"
openai.api_base = "https://myopenai.openai.azure.com/"
openai.api_version = "2024-02-15-preview"

# Use Azure AD authentication
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(), "https://cognitiveservices.azure.com/.default"
)
openai.api_key = token_provider()

# Chat completion
response = openai.ChatCompletion.create(
    engine="gpt-4",  # Your deployment name
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing"}
    ],
    temperature=0.7,
    max_tokens=1000
)

print(response.choices[0].message.content)
```

### REST API Usage
```python
import requests
import json

# API configuration
api_base = "https://myopenai.openai.azure.com/"
api_key = "your-api-key"
api_version = "2024-02-15-preview"
deployment_name = "gpt-4"

# Headers
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}"
}

# Request payload
payload = {
    "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is machine learning?"}
    ],
    "temperature": 0.7,
    "max_tokens": 800
}

# Make request
url = f"{api_base}openai/deployments/{deployment_name}/chat/completions?api-version={api_version}"
response = requests.post(url, headers=headers, data=json.dumps(payload))
result = response.json()

print(result["choices"][0]["message"]["content"])
```

### Streaming Responses
```python
def stream_chat_response(messages):
    response = openai.ChatCompletion.create(
        engine="gpt-4",
        messages=messages,
        temperature=0.7,
        max_tokens=1000,
        stream=True
    )
    
    for chunk in response:
        if chunk.choices[0].delta.get("content"):
            yield chunk.choices[0].delta.content

# Usage
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Write a story about AI"}
]

for content in stream_chat_response(messages):
    print(content, end='', flush=True)
```

## Advanced Features

### Function Calling
```python
import json

# Define functions
functions = [
    {
        "name": "get_weather",
        "description": "Get current weather information",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City name"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature unit"
                }
            },
            "required": ["location"]
        }
    }
]

# Chat with function calling
response = openai.ChatCompletion.create(
    engine="gpt-4",
    messages=[
        {"role": "user", "content": "What's the weather like in Seattle?"}
    ],
    functions=functions,
    function_call="auto"
)

# Handle function call
if response.choices[0].message.get("function_call"):
    function_call = response.choices[0].message.function_call
    function_name = function_call.name
    function_args = json.loads(function_call.arguments)
    
    # Execute function (implement get_weather)
    weather_result = get_weather(**function_args)
    
    # Continue conversation with function result
    messages = [
        {"role": "user", "content": "What's the weather like in Seattle?"},
        response.choices[0].message,
        {
            "role": "function",
            "name": function_name,
            "content": json.dumps(weather_result)
        }
    ]
    
    final_response = openai.ChatCompletion.create(
        engine="gpt-4",
        messages=messages
    )
```

### Image Generation with DALL-E
```python
# Generate image
response = openai.Image.create(
    engine="dall-e-3",
    prompt="A futuristic cityscape with flying cars",
    size="1024x1024",
    quality="hd",
    n=1
)

image_url = response.data[0].url
print(f"Generated image: {image_url}")

# Image variations
response = openai.Image.create_variation(
    engine="dall-e-2",
    image=open("input-image.png", "rb"),
    n=2,
    size="512x512"
)
```

### Embeddings for Semantic Search
```python
import numpy as np

def get_embedding(text):
    response = openai.Embedding.create(
        engine="text-embedding-ada-002",
        input=text
    )
    return response.data[0].embedding

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# Create knowledge base
documents = [
    "Azure is Microsoft's cloud platform",
    "Machine learning is a subset of AI",
    "Python is a programming language"
]

# Generate embeddings
embeddings = [get_embedding(doc) for doc in documents]

# Query
query = "What is Microsoft's cloud service?"
query_embedding = get_embedding(query)

# Find most similar document
similarities = [cosine_similarity(query_embedding, emb) for emb in embeddings]
best_match_idx = np.argmax(similarities)

print(f"Best match: {documents[best_match_idx]}")
print(f"Similarity: {similarities[best_match_idx]:.3f}")
```

## Responsible AI and Content Filtering

### Content Safety
Azure OpenAI includes built-in content filtering:

#### Filter Categories
- **Hate**: Content that promotes hatred or discrimination
- **Sexual**: Sexually explicit or suggestive content
- **Violence**: Content depicting violence or harm
- **Self-harm**: Content related to self-injury

#### Filter Severity Levels
- **Safe**: No harmful content detected
- **Low**: Minimal risk content
- **Medium**: Moderate risk content
- **High**: High risk content

### Custom Content Filtering
```python
# Configure content filter
response = openai.ChatCompletion.create(
    engine="gpt-4",
    messages=[{"role": "user", "content": "Your message here"}],
    content_filter_policy="default"  # or custom policy
)

# Check for content filtering
if hasattr(response.choices[0], 'content_filter_results'):
    filter_results = response.choices[0].content_filter_results
    if filter_results:
        print("Content was filtered:", filter_results)
```

### Responsible AI Practices
```python
# Implement usage monitoring
class UsageMonitor:
    def __init__(self):
        self.request_count = 0
        self.token_usage = 0
    
    def track_request(self, response):
        self.request_count += 1
        self.token_usage += response.usage.total_tokens
    
    def get_stats(self):
        return {
            "requests": self.request_count,
            "tokens": self.token_usage,
            "avg_tokens_per_request": self.token_usage / max(1, self.request_count)
        }

monitor = UsageMonitor()

# Use with requests
response = openai.ChatCompletion.create(...)
monitor.track_request(response)
```

## Performance Optimization

### Token Management
```python
import tiktoken

def count_tokens(text, model="gpt-4"):
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def optimize_prompt(prompt, max_tokens=4000):
    token_count = count_tokens(prompt)
    if token_count > max_tokens:
        # Truncate or summarize prompt
        encoding = tiktoken.encoding_for_model("gpt-4")
        tokens = encoding.encode(prompt)
        truncated_tokens = tokens[:max_tokens]
        return encoding.decode(truncated_tokens)
    return prompt
```

### Batch Processing
```python
import asyncio
import aiohttp

async def process_batch(prompts, max_concurrent=5):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def process_single(prompt):
        async with semaphore:
            response = await openai.ChatCompletion.acreate(
                engine="gpt-4",
                messages=[{"role": "user", "content": prompt}]
            )
            return response.choices[0].message.content
    
    tasks = [process_single(prompt) for prompt in prompts]
    results = await asyncio.gather(*tasks)
    return results

# Usage
prompts = ["Prompt 1", "Prompt 2", "Prompt 3"]
results = asyncio.run(process_batch(prompts))
```

### Caching Strategies
```python
import hashlib
import json
from functools import wraps

def cache_openai_response(cache_dict={}):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Create cache key from arguments
            cache_key = hashlib.md5(
                json.dumps(str(args) + str(kwargs), sort_keys=True).encode()
            ).hexdigest()
            
            if cache_key in cache_dict:
                return cache_dict[cache_key]
            
            result = func(*args, **kwargs)
            cache_dict[cache_key] = result
            return result
        return wrapper
    return decorator

@cache_openai_response()
def cached_completion(prompt):
    return openai.ChatCompletion.create(
        engine="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
```

## Security and Compliance

### Authentication Methods
```python
# API Key authentication
openai.api_key = "your-api-key"

# Azure AD authentication
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
token = credential.get_token("https://cognitiveservices.azure.com/.default")
openai.api_key = token.token
```

### Network Security
```python
# Configure for private endpoints
openai.api_base = "https://myopenai.privatelink.openai.azure.com/"

# Use with VNet integration
import requests

session = requests.Session()
session.verify = "/path/to/ca-bundle.crt"  # Custom CA if needed

# Configure proxy if required
session.proxies = {
    'http': 'http://proxy.company.com:8080',
    'https': 'https://proxy.company.com:8080'
}
```

### Data Privacy
```python
# Implement data handling policies
class DataHandler:
    def __init__(self):
        self.sensitive_patterns = [
            r'\b\d{3}-\d{2}-\d{4}\b',  # SSN
            r'\b\d{4}\s?\d{4}\s?\d{4}\s?\d{4}\b',  # Credit card
            r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'  # Email
        ]
    
    def sanitize_input(self, text):
        import re
        sanitized = text
        for pattern in self.sensitive_patterns:
            sanitized = re.sub(pattern, '[REDACTED]', sanitized)
        return sanitized
    
    def secure_request(self, prompt):
        sanitized_prompt = self.sanitize_input(prompt)
        response = openai.ChatCompletion.create(
            engine="gpt-4",
            messages=[{"role": "user", "content": sanitized_prompt}]
        )
        return response
```

## Monitoring and Observability

### Usage Tracking
```python
import logging
from datetime import datetime

class OpenAILogger:
    def __init__(self):
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger(__name__)
    
    def log_request(self, model, prompt_tokens, completion_tokens, cost=None):
        self.logger.info(f"""
        OpenAI Request:
        Timestamp: {datetime.now()}
        Model: {model}
        Prompt Tokens: {prompt_tokens}
        Completion Tokens: {completion_tokens}
        Total Tokens: {prompt_tokens + completion_tokens}
        Estimated Cost: ${cost or 'N/A'}
        """)

logger = OpenAILogger()

# Use with requests
response = openai.ChatCompletion.create(...)
logger.log_request(
    model="gpt-4",
    prompt_tokens=response.usage.prompt_tokens,
    completion_tokens=response.usage.completion_tokens
)
```

### Azure Monitor Integration
```python
from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace

# Configure Azure Monitor
configure_azure_monitor(
    connection_string="InstrumentationKey=your-key"
)

tracer = trace.get_tracer(__name__)

def traced_completion(prompt):
    with tracer.start_as_current_span("openai_completion") as span:
        span.set_attribute("prompt_length", len(prompt))
        span.set_attribute("model", "gpt-4")
        
        response = openai.ChatCompletion.create(
            engine="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        
        span.set_attribute("completion_tokens", response.usage.completion_tokens)
        span.set_attribute("total_tokens", response.usage.total_tokens)
        
        return response
```

## Best Practices

### Prompt Engineering
1. **Be specific**: Provide clear, detailed instructions
2. **Use examples**: Include few-shot examples for complex tasks
3. **Set context**: Establish role and expectations
4. **Iterative refinement**: Test and improve prompts systematically

### Cost Optimization
1. **Choose appropriate models**: Use GPT-3.5 for simpler tasks
2. **Optimize token usage**: Minimize unnecessary tokens
3. **Implement caching**: Cache responses for repeated queries
4. **Monitor usage**: Track and analyze spending patterns

### Security Guidelines
1. **Input validation**: Sanitize user inputs
2. **Output filtering**: Review generated content
3. **Access control**: Implement proper authentication
4. **Data handling**: Follow data privacy regulations

### Performance Best Practices
1. **Async processing**: Use async for high-throughput scenarios
2. **Batch requests**: Group multiple requests when possible
3. **Connection pooling**: Reuse HTTP connections
4. **Error handling**: Implement robust retry logic