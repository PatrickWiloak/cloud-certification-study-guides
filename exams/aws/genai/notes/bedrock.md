# Amazon Bedrock Fundamentals

## What is Amazon Bedrock?

Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models (FMs) from leading AI companies through a single API, along with a broad set of capabilities to build generative AI applications.

### Key Benefits
- **Choice of foundation models**: Access models from multiple providers
- **Serverless experience**: No infrastructure to manage
- **Integrated security**: Built-in privacy and security features
- **Fine-tuning capabilities**: Customize models with your data
- **Responsible AI**: Built-in safeguards and monitoring

## Foundation Model Providers

### Anthropic (Claude Models)
| Model | Strengths | Use Cases |
|-------|-----------|-----------|
| **Claude 3 Opus** | Highest capability, complex reasoning | Research, analysis, complex problem-solving |
| **Claude 3 Sonnet** | Balanced performance and speed | Content creation, coding, conversation |
| **Claude 3 Haiku** | Fast, cost-effective | Simple tasks, content moderation |
| **Claude 2.1** | Long context (200K tokens) | Document analysis, long-form content |

#### Claude Capabilities
- **Text generation**: High-quality content creation
- **Code generation**: Programming assistance and debugging
- **Analysis**: Document and data analysis
- **Conversation**: Natural dialogue and Q&A
- **Reasoning**: Complex problem-solving and logic

### AI21 Labs (Jurassic Models)
| Model | Description | Use Cases |
|-------|-------------|-----------|
| **Jurassic-2 Ultra** | Largest, most capable | Complex text generation, analysis |
| **Jurassic-2 Mid** | Balanced performance | General text tasks, content creation |

#### Jurassic Features
- **Text completion**: Continuing and completing text
- **Summarization**: Document and content summarization
- **Question answering**: Contextual Q&A
- **Text classification**: Content categorization

### Cohere (Command and Embed Models)
| Model Type | Model | Purpose |
|------------|-------|---------|
| **Text Generation** | Command | Text generation and completion |
| **Embeddings** | Embed | Semantic search and similarity |

#### Cohere Specializations
- **Enterprise focus**: Business-oriented language tasks
- **Multilingual**: Support for multiple languages
- **Embeddings**: High-quality text representations
- **Classification**: Text categorization and routing

### Meta (Llama 2 Models)
| Model | Parameters | Use Cases |
|-------|------------|-----------|
| **Llama 2 7B** | 7 billion | Lightweight applications, edge deployment |
| **Llama 2 13B** | 13 billion | Balanced performance and efficiency |
| **Llama 2 70B** | 70 billion | Complex reasoning, high-quality generation |

#### Llama 2 Features
- **Open source foundation**: Transparent and customizable
- **Code generation**: Programming assistance
- **Instruction following**: Task-specific completions
- **Conversation**: Dialogue and chat applications

### Stability AI (Stable Diffusion)
| Model | Type | Capabilities |
|-------|------|-------------|
| **Stable Diffusion XL** | Image generation | High-resolution image creation |
| **Stable Diffusion** | Image generation | Text-to-image generation |

#### Image Generation Features
- **Text-to-image**: Create images from descriptions
- **Style control**: Artistic style customization
- **High resolution**: Detailed image generation
- **Inpainting**: Image editing and completion

### Amazon Titan Models
| Model | Type | Purpose |
|-------|------|---------|
| **Titan Text Express** | Text generation | General text tasks, content creation |
| **Titan Text Lite** | Text generation | Simple, cost-effective text tasks |
| **Titan Embeddings** | Text embeddings | Semantic search, retrieval |
| **Titan Image Generator** | Image generation | Text-to-image creation |

#### Titan Features
- **Multilingual**: Support for multiple languages
- **Safety**: Built-in content filtering
- **Customization**: Fine-tuning capabilities
- **Cost-effective**: Optimized for price-performance

## Bedrock API and Integration

### Model Invocation
```python
import boto3
import json

# Initialize Bedrock client
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

# Invoke Claude model
def invoke_claude(prompt, max_tokens=1000):
    body = json.dumps({
        "prompt": f"\n\nHuman: {prompt}\n\nAssistant:",
        "max_tokens_to_sample": max_tokens,
        "temperature": 0.7,
        "top_p": 0.9
    })
    
    response = bedrock.invoke_model(
        modelId='anthropic.claude-v2',
        body=body,
        contentType='application/json'
    )
    
    response_body = json.loads(response['body'].read())
    return response_body['completion']

# Example usage
result = invoke_claude("Explain quantum computing in simple terms")
print(result)
```

### Streaming Responses
```python
def invoke_claude_streaming(prompt):
    body = json.dumps({
        "prompt": f"\n\nHuman: {prompt}\n\nAssistant:",
        "max_tokens_to_sample": 1000,
        "temperature": 0.7
    })
    
    response = bedrock.invoke_model_with_response_stream(
        modelId='anthropic.claude-v2',
        body=body,
        contentType='application/json'
    )
    
    # Process streaming response
    for event in response['body']:
        chunk = json.loads(event['chunk']['bytes'])
        if 'completion' in chunk:
            print(chunk['completion'], end='', flush=True)
```

### Image Generation with Stable Diffusion
```python
def generate_image(prompt, style="photographic"):
    body = json.dumps({
        "text_prompts": [{"text": prompt}],
        "cfg_scale": 10,
        "seed": 0,
        "steps": 50,
        "style_preset": style
    })
    
    response = bedrock.invoke_model(
        modelId='stability.stable-diffusion-xl-v1',
        body=body,
        contentType='application/json'
    )
    
    response_body = json.loads(response['body'].read())
    return response_body['artifacts'][0]['base64']
```

## Model Customization

### Fine-Tuning
**Customize foundation models with your data**

#### When to Fine-Tune
- **Domain-specific language**: Industry or technical terminology
- **Specific writing style**: Brand voice or tone requirements
- **Task specialization**: Optimize for specific use cases
- **Performance improvement**: Better accuracy on your data

#### Fine-Tuning Process
1. **Prepare training data**: High-quality examples in JSONL format
2. **Create training job**: Configure hyperparameters
3. **Monitor training**: Track loss and validation metrics
4. **Evaluate model**: Test on held-out validation set
5. **Deploy custom model**: Use for inference

```python
# Create fine-tuning job
response = bedrock.create_model_customization_job(
    jobName='my-custom-model',
    customModelName='my-model-v1',
    baseModelIdentifier='anthropic.claude-v2',
    trainingDataConfig={
        's3Uri': 's3://my-bucket/training-data.jsonl'
    },
    validationDataConfig={
        's3Uri': 's3://my-bucket/validation-data.jsonl'
    },
    hyperParameters={
        'epochCount': '3',
        'batchSize': '1',
        'learningRate': '0.00001'
    }
)
```

### Provisioned Throughput
**Reserved capacity for consistent performance**

#### When to Use Provisioned Throughput
- **Predictable workloads**: Consistent traffic patterns
- **Performance requirements**: Guaranteed response times
- **Cost optimization**: Lower per-token costs at scale
- **Production applications**: Critical business applications

```python
# Create provisioned model throughput
response = bedrock.create_provisioned_model_throughput(
    modelUnits=1,
    modelId='anthropic.claude-v2',
    provisionedModelName='my-provisioned-model',
    commitmentDuration='OneMonth'
)
```

## Retrieval Augmented Generation (RAG)

### What is RAG?
RAG combines the power of foundation models with external knowledge sources to provide more accurate, up-to-date, and contextually relevant responses.

### RAG Architecture
```
User Query → Vector Search → Retrieved Context → Foundation Model → Enhanced Response
```

### Knowledge Bases for Amazon Bedrock
**Managed RAG implementation**

#### Components
- **Data sources**: S3 buckets with documents
- **Vector database**: Amazon OpenSearch Serverless
- **Embeddings**: Amazon Titan Embeddings
- **Retrieval**: Semantic search and ranking

```python
# Create knowledge base
response = bedrock_agent.create_knowledge_base(
    name='my-knowledge-base',
    description='Company documentation knowledge base',
    knowledgeBaseConfiguration={
        'type': 'VECTOR',
        'vectorKnowledgeBaseConfiguration': {
            'embeddingModelArn': 'arn:aws:bedrock:us-east-1::foundation-model/amazon.titan-embed-text-v1'
        }
    },
    storageConfiguration={
        'type': 'OPENSEARCH_SERVERLESS',
        'opensearchServerlessConfiguration': {
            'collectionArn': 'arn:aws:aoss:us-east-1:123456789012:collection/kb-collection',
            'vectorIndexName': 'knowledge-base-index',
            'fieldMapping': {
                'vectorField': 'vector',
                'textField': 'text',
                'metadataField': 'metadata'
            }
        }
    }
)
```

### Custom RAG Implementation
```python
import numpy as np
from sentence_transformers import SentenceTransformer

class RAGSystem:
    def __init__(self):
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        self.knowledge_base = []
        self.embeddings = []
    
    def add_document(self, text, metadata=None):
        """Add document to knowledge base"""
        embedding = self.embedding_model.encode(text)
        self.knowledge_base.append({
            'text': text,
            'metadata': metadata or {},
            'embedding': embedding
        })
        self.embeddings.append(embedding)
    
    def search(self, query, top_k=3):
        """Search for relevant documents"""
        query_embedding = self.embedding_model.encode(query)
        
        # Calculate similarities
        similarities = []
        for doc_embedding in self.embeddings:
            similarity = np.dot(query_embedding, doc_embedding) / (
                np.linalg.norm(query_embedding) * np.linalg.norm(doc_embedding)
            )
            similarities.append(similarity)
        
        # Get top-k most similar documents
        top_indices = np.argsort(similarities)[-top_k:][::-1]
        return [self.knowledge_base[i] for i in top_indices]
    
    def generate_response(self, query):
        """Generate response using RAG"""
        # Retrieve relevant context
        relevant_docs = self.search(query)
        context = "\n".join([doc['text'] for doc in relevant_docs])
        
        # Create enhanced prompt
        prompt = f"""
        Context information:
        {context}
        
        Question: {query}
        
        Please answer the question based on the provided context.
        """
        
        # Generate response with foundation model
        return invoke_claude(prompt)
```

## Prompt Engineering

### Best Practices
1. **Be specific**: Clear, detailed instructions
2. **Provide examples**: Few-shot learning examples
3. **Structure output**: Specify desired format
4. **Set context**: Provide relevant background
5. **Use delimiters**: Clearly separate sections

### Prompt Templates

#### Task-Specific Prompts
```python
# Classification prompt
classification_prompt = """
Classify the following customer feedback into categories: positive, negative, or neutral.

Feedback: "{feedback}"

Classification: """

# Summarization prompt
summarization_prompt = """
Please provide a concise summary of the following text in 2-3 sentences:

Text: {text}

Summary: """

# Code generation prompt
code_prompt = """
Write a Python function that {description}.

Requirements:
- Include proper error handling
- Add docstrings
- Use type hints

Code: """
```

#### Few-Shot Examples
```python
few_shot_prompt = """
Classify movie reviews as positive or negative:

Review: "This movie was absolutely fantastic!"
Sentiment: Positive

Review: "I fell asleep halfway through. Boring!"
Sentiment: Negative

Review: "The acting was great but the plot was confusing."
Sentiment: Negative

Review: "{user_review}"
Sentiment: """
```

### Advanced Techniques

#### Chain of Thought
```python
cot_prompt = """
Solve this step by step:

Question: A restaurant has 24 tables. Each table can seat 4 people. If the restaurant is 75% full, how many people are dining?

Let me think through this step by step:
1) First, I need to find the total capacity
2) Then calculate 75% of that capacity
3) 

Step 1: Total capacity = 24 tables × 4 people per table = 96 people
Step 2: 75% of 96 = 0.75 × 96 = 72 people

Therefore, 72 people are dining.

Now solve this problem:
{problem}
"""
```

#### Role-Based Prompts
```python
expert_prompt = """
You are a cybersecurity expert with 15 years of experience. 
A client asks: "{question}"

Provide a detailed, professional response that includes:
1. Technical explanation
2. Potential risks
3. Recommended solutions
4. Best practices

Response: """
```

## Monitoring and Observability

### CloudWatch Metrics
| Metric | Description | Use Case |
|--------|-------------|----------|
| **Invocations** | Number of model invocations | Usage tracking |
| **InputTokenCount** | Tokens in input | Cost monitoring |
| **OutputTokenCount** | Tokens in output | Cost monitoring |
| **InvocationLatency** | Response time | Performance monitoring |
| **InvocationClientErrors** | 4xx errors | Error tracking |
| **InvocationServerErrors** | 5xx errors | Service health |

### Cost Monitoring
```python
# Calculate token costs
def calculate_cost(input_tokens, output_tokens, model_id):
    # Pricing varies by model - example for Claude
    pricing = {
        'anthropic.claude-v2': {
            'input': 0.00001102,   # per 1K input tokens
            'output': 0.00003268   # per 1K output tokens
        }
    }
    
    if model_id in pricing:
        input_cost = (input_tokens / 1000) * pricing[model_id]['input']
        output_cost = (output_tokens / 1000) * pricing[model_id]['output']
        return input_cost + output_cost
    
    return 0
```

### Logging and Debugging
```python
import logging

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def invoke_with_logging(prompt, model_id):
    logger.info(f"Invoking model {model_id}")
    logger.debug(f"Prompt: {prompt[:100]}...")
    
    try:
        response = invoke_model(prompt, model_id)
        logger.info(f"Successful invocation, response length: {len(response)}")
        return response
    except Exception as e:
        logger.error(f"Model invocation failed: {str(e)}")
        raise
```

## Security and Compliance

### Data Privacy
- **No model training**: Your data is not used to train foundation models
- **Encryption**: Data encrypted in transit and at rest
- **VPC endpoints**: Private connectivity option
- **Access control**: IAM-based permissions

### IAM Policies
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "arn:aws:bedrock:*::foundation-model/anthropic.claude*"
        }
    ]
}
```

### Content Filtering
```python
# Enable content filtering
body = json.dumps({
    "prompt": prompt,
    "max_tokens_to_sample": 1000,
    "temperature": 0.7,
    "anthropic_version": "bedrock-2023-05-31"
})

response = bedrock.invoke_model(
    modelId='anthropic.claude-v2',
    body=body,
    contentType='application/json',
    guardrailIdentifier='my-guardrail-id',
    guardrailVersion='1'
)
```

## Best Practices

### Model Selection
1. **Evaluate multiple models**: Test different models for your use case
2. **Consider cost vs performance**: Balance quality and price
3. **Start simple**: Begin with smaller models and scale up
4. **Benchmark systematically**: Use consistent evaluation metrics

### Performance Optimization
1. **Batch requests**: Group multiple requests when possible
2. **Optimize prompts**: Minimize tokens while maintaining quality
3. **Use caching**: Cache responses for repeated queries
4. **Monitor usage**: Track metrics and optimize based on data

### Cost Management
1. **Choose appropriate models**: Don't over-provision capabilities
2. **Optimize prompt length**: Shorter prompts cost less
3. **Use provisioned throughput**: For high-volume, predictable workloads
4. **Implement request limits**: Prevent runaway costs

### Development Workflow
1. **Start with playground**: Experiment with prompts interactively
2. **Version control prompts**: Track prompt changes
3. **A/B test variations**: Compare different approaches
4. **Monitor in production**: Track performance and user satisfaction