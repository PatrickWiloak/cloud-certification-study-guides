# Vertex AI Platform Fundamentals

## What is Vertex AI?

Vertex AI is Google Cloud's unified machine learning platform that combines AutoML and AI Platform into a unified API, client library, and user interface, providing tools for every step of the ML workflow.

### Key Benefits
- **Unified platform**: One place for all ML workflows
- **MLOps integration**: Built-in experiment tracking and model management
- **Scalable infrastructure**: Automatic scaling and optimization
- **Pre-built solutions**: AutoML and pre-trained models
- **Custom development**: Full flexibility for custom models

## Vertex AI Components

### Vertex AI Workbench
**Jupyter-based development environment**

#### Features
- **Managed notebooks**: Fully managed Jupyter instances
- **User-managed notebooks**: More control over environment
- **Built-in libraries**: Pre-installed ML frameworks
- **Data science tools**: Integrated development experience
- **Version control**: Git integration

#### Instance Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Managed** | Google-managed infrastructure | Quick start, experimentation |
| **User-managed** | Custom VM configuration | Production workloads, specific requirements |

### Vertex AI Training
**Scalable model training service**

#### Training Options
- **Custom training**: Your own training code
- **AutoML training**: Automated model development
- **Hyperparameter tuning**: Automated hyperparameter optimization
- **Distributed training**: Multi-GPU and multi-node training

#### Custom Training Example
```python
from google.cloud import aiplatform

# Initialize Vertex AI
aiplatform.init(project='my-project', location='us-central1')

# Create custom training job
job = aiplatform.CustomTrainingJob(
    display_name='my-training-job',
    script_path='train.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-gpu.1-9:latest',
    requirements=['scikit-learn', 'pandas'],
    model_serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/pytorch-gpu.1-9:latest'
)

# Run training
model = job.run(
    dataset=my_dataset,
    replica_count=1,
    machine_type='n1-standard-4',
    accelerator_type='NVIDIA_TESLA_K80',
    accelerator_count=1
)
```

### Vertex AI Endpoints
**Model serving infrastructure**

#### Deployment Options
- **Online prediction**: Real-time inference
- **Batch prediction**: Offline batch processing
- **Private endpoints**: VPC-native serving
- **Multi-model endpoints**: Host multiple models

```python
# Deploy model to endpoint
endpoint = model.deploy(
    machine_type='n1-standard-2',
    min_replica_count=1,
    max_replica_count=10,
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1
)

# Make predictions
predictions = endpoint.predict(instances=test_data)
```

### Vertex AI Pipelines
**ML workflow orchestration**

#### Pipeline Components
- **Components**: Reusable pipeline steps
- **Experiments**: Track and compare pipeline runs
- **Artifacts**: Track inputs, outputs, and metadata
- **Lineage**: Data and model lineage tracking

```python
from kfp.v2 import dsl
from kfp.v2.dsl import component, pipeline, Input, Output, Dataset, Model

@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'scikit-learn']
)
def preprocess_data(
    input_data: Input[Dataset],
    output_data: Output[Dataset]
):
    import pandas as pd
    
    # Load and preprocess data
    df = pd.read_csv(input_data.path)
    # ... preprocessing logic
    df.to_csv(output_data.path, index=False)

@component(
    base_image='python:3.9',
    packages_to_install=['scikit-learn']
)
def train_model(
    training_data: Input[Dataset],
    model: Output[Model]
):
    import pickle
    from sklearn.ensemble import RandomForestClassifier
    
    # Train model
    # ... training logic
    
    # Save model
    with open(model.path, 'wb') as f:
        pickle.dump(trained_model, f)

@pipeline(name='ml-pipeline')
def ml_pipeline():
    preprocess_task = preprocess_data(input_data=dataset)
    train_task = train_model(training_data=preprocess_task.outputs['output_data'])
```

### Vertex AI Model Registry
**Model versioning and management**

#### Features
- **Model versioning**: Track model versions
- **Model lineage**: Understand model ancestry
- **Model evaluation**: Compare model performance
- **Model aliases**: Stable references to models

```python
# Register model
model = aiplatform.Model.upload(
    display_name='my-model',
    artifact_uri='gs://my-bucket/model/',
    serving_container_image_uri='gcr.io/cloud-aiplatform/prediction/sklearn-cpu.0-24:latest',
    description='My custom model'
)

# Create model version
model_version = model.upload(
    parent_model=model,
    version_description='Updated model with new features'
)
```

## Generative AI on Vertex AI

### Generative AI Studio
**Experiment with generative AI models through unified interface**

#### Capabilities
- **Text generation**: Create and edit text content
- **Chat**: Conversational AI applications
- **Code generation**: Programming assistance
- **Multimodal**: Text and image understanding
- **Prompt design**: Test and iterate on prompts
- **Model tuning**: Fine-tune foundation models
- **Model evaluation**: Compare model outputs

#### Studio Components

**Language Section**
- Design and test prompts
- Create structured prompts with examples
- Tune language models with supervised learning
- Evaluate model quality and safety

**Vision Section**
- Image captioning and analysis
- Visual question answering
- Object detection and segmentation
- Image generation with Imagen

**Speech Section**
- Speech-to-text conversion
- Text-to-speech synthesis
- Audio understanding

**Conversation Section**
- Build conversational agents
- Multi-turn dialog management
- Context preservation
- Intent classification

### Model Garden
**Pre-trained and foundation models marketplace**

#### Available Model Categories

**Google Foundation Models**
- **PaLM 2**: text-bison, chat-bison, code-bison
- **Gemini**: gemini-pro, gemini-pro-vision, gemini-ultra
- **Imagen**: imagen-v1, imagen-v2
- **Chirp**: Speech recognition
- **BERT**: Text embeddings
- **T5**: Text-to-text generation

**Third-Party Foundation Models**
- **Meta**: Llama 2, Llama 3 (7B, 13B, 70B parameters)
- **Anthropic**: Claude 2, Claude 3
- **Cohere**: Command, Embed, Rerank
- **AI21**: Jurassic-2 models
- **Stability AI**: Stable Diffusion XL

**Open Source Models**
- **Falcon**: 7B, 40B, 180B
- **MPT**: 7B, 30B
- **FLAN-T5**: Various sizes
- **CodeLlama**: Code generation

**Task-Specific Models**
- Medical language models
- Financial analysis models
- Legal document understanding
- Domain-adapted embeddings

#### Model Selection Decision Matrix

| Use Case | Recommended Model | Alternative | Reasoning |
|----------|------------------|-------------|-----------|
| General text generation | Gemini Pro | PaLM 2 text-bison | Balance of quality and cost |
| Code generation | code-bison | CodeLlama 34B | Optimized for code syntax |
| Multi-turn conversations | Gemini Pro | chat-bison | Better context handling |
| Image understanding | Gemini Pro Vision | - | Multimodal capabilities |
| Long context (100K+ tokens) | Gemini Pro | Claude 2 | Extended context window |
| Low latency requirements | text-bison@002 | Gemini Flash | Faster response time |
| Creative writing | PaLM 2 text-bison | Gemini Pro | More creative outputs |
| Factual accuracy | Gemini Pro + Grounding | PaLM with Search | Reduced hallucination |
| Cost optimization | text-bison@001 | Llama 2 70B | Lower per-token cost |
| Enterprise compliance | Vertex AI models | - | Data residency guarantees |

### Foundation Models

#### PaLM 2 Models

**text-bison**
- Large language model for text generation
- Context window: 8,192 tokens
- Output limit: 2,048 tokens
- Strengths: Creative writing, summarization, content generation
- Versions: text-bison@001, text-bison@002
- Pricing: $0.0005 per 1K input chars, $0.0005 per 1K output chars

```python
import vertexai
from vertexai.language_models import TextGenerationModel

# Initialize Vertex AI
vertexai.init(project="my-project", location="us-central1")

# Load text-bison model
model = TextGenerationModel.from_pretrained("text-bison@002")

# Basic text generation
response = model.predict(
    prompt="Write a comprehensive product description for a smartwatch with health monitoring features",
    temperature=0.7,          # Controls randomness (0.0-1.0)
    max_output_tokens=1024,   # Maximum response length
    top_k=40,                 # Top-k sampling
    top_p=0.95,               # Nucleus sampling
    stop_sequences=["\n\n"]   # Stop generation at sequences
)

print(f"Response: {response.text}")
print(f"Safety attributes: {response.safety_attributes}")
```

**chat-bison**
- Conversational AI model with multi-turn context
- Context window: 8,192 tokens
- Maintains conversation history
- Strengths: Q&A, customer support, conversational agents
- Features: Context preservation, follow-up questions

```python
from vertexai.language_models import ChatModel

# Load chat model
chat_model = ChatModel.from_pretrained("chat-bison@002")

# Start chat session
chat = chat_model.start_chat(
    context="You are a helpful cloud architect assistant specializing in GCP",
    examples=[
        {
            "input": {"content": "What is Vertex AI?"},
            "output": {"content": "Vertex AI is Google Cloud's unified ML platform..."}
        }
    ],
    temperature=0.3,
    max_output_tokens=256,
    top_k=40,
    top_p=0.8
)

# Send messages
response1 = chat.send_message("What are the benefits of using Vertex AI?")
print(f"Response 1: {response1.text}")

# Follow-up with context
response2 = chat.send_message("How does it compare to AWS SageMaker?")
print(f"Response 2: {response2.text}")

# Get full conversation history
print(f"Message history: {chat.message_history}")
```

**code-bison**
- Specialized for code generation and completion
- Context window: 6,144 tokens
- Output limit: 1,024 tokens
- Strengths: Code generation, debugging, explanation, refactoring
- Languages: Python, Java, JavaScript, Go, C++, and more

```python
from vertexai.language_models import CodeGenerationModel

# Load code generation model
code_model = CodeGenerationModel.from_pretrained("code-bison@002")

# Generate code
response = code_model.predict(
    prefix="""
# Write a Python function that implements a retry decorator
# with exponential backoff for API calls
""",
    temperature=0.2,  # Lower temperature for more deterministic code
    max_output_tokens=512
)

print(f"Generated code:\n{response.text}")

# Code completion
completion = code_model.predict(
    prefix="""
def calculate_fibonacci(n):
    \"\"\"Calculate fibonacci number using dynamic programming\"\"\"
    if n <= 1:
        return n

    # Complete this function
""",
    suffix="""
    return fib[n]
""",
    temperature=0.1,
    max_output_tokens=256
)

print(f"Completed code:\n{completion.text}")
```

**code-chat-bison**
- Interactive code assistance and debugging
- Multi-turn conversations about code
- Supports code explanation and refactoring

```python
from vertexai.language_models import CodeChatModel

# Load code chat model
code_chat_model = CodeChatModel.from_pretrained("codechat-bison@002")

# Start code chat
code_chat = code_chat_model.start_chat()

# Ask about code
response1 = code_chat.send_message(
    "Write a Python class for a binary search tree with insert and search methods"
)
print(f"Code: {response1.text}")

# Follow up for modifications
response2 = code_chat.send_message(
    "Now add a delete method to this class"
)
print(f"Updated code: {response2.text}")

# Request explanations
response3 = code_chat.send_message(
    "Explain the time complexity of these operations"
)
print(f"Explanation: {response3.text}")
```

**Embedding Models**
- Generate vector representations of text
- Use cases: Semantic search, similarity, clustering, classification

```python
from vertexai.language_models import TextEmbeddingModel

# Load embedding model
embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")

# Generate single embedding
embedding = embedding_model.get_embeddings(["Cloud computing platforms"])[0]
print(f"Embedding vector (length {len(embedding.values)}): {embedding.values[:5]}...")

# Batch embeddings
texts = [
    "Vertex AI is a machine learning platform",
    "BigQuery is a data warehouse",
    "Cloud Storage stores objects",
    "Kubernetes runs containers"
]

embeddings = embedding_model.get_embeddings(texts)
for i, emb in enumerate(embeddings):
    print(f"Text {i}: {len(emb.values)} dimensions")

# Calculate similarity
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# Compare embeddings
emb1 = embedding_model.get_embeddings(["Machine learning"])[0].values
emb2 = embedding_model.get_embeddings(["Artificial intelligence"])[0].values
emb3 = embedding_model.get_embeddings(["Database systems"])[0].values

similarity_ml_ai = cosine_similarity(emb1, emb2)
similarity_ml_db = cosine_similarity(emb1, emb3)

print(f"ML vs AI similarity: {similarity_ml_ai:.4f}")
print(f"ML vs DB similarity: {similarity_ml_db:.4f}")
```

#### Gemini Models

**Gemini Pro**
- Next-generation multimodal model
- Context window: 32,768 tokens
- Output limit: 8,192 tokens
- Strengths: Reasoning, coding, multimodal understanding
- Features: Function calling, structured outputs, JSON mode

```python
import vertexai
from vertexai.generative_models import GenerativeModel, Part, FinishReason
import vertexai.preview.generative_models as generative_models

# Initialize
vertexai.init(project="my-project", location="us-central1")

# Load Gemini Pro
model = GenerativeModel("gemini-1.0-pro")

# Basic generation
response = model.generate_content(
    "Explain the architecture of a microservices-based e-commerce platform",
    generation_config={
        "max_output_tokens": 2048,
        "temperature": 0.7,
        "top_p": 0.95,
        "top_k": 40
    },
    safety_settings={
        generative_models.HarmCategory.HARM_CATEGORY_HATE_SPEECH: generative_models.HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        generative_models.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: generative_models.HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        generative_models.HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT: generative_models.HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
        generative_models.HarmCategory.HARM_CATEGORY_HARASSMENT: generative_models.HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
    }
)

print(f"Response: {response.text}")

# Multi-turn conversation
chat = model.start_chat()

response1 = chat.send_message("What is Kubernetes?")
print(f"Response 1: {response1.text}")

response2 = chat.send_message("How do I deploy a workload to it?")
print(f"Response 2: {response2.text}")

response3 = chat.send_message("What about scaling?")
print(f"Response 3: {response3.text}")

# Streaming responses
responses = model.generate_content(
    "Write a detailed guide to setting up a CI/CD pipeline",
    stream=True
)

print("Streaming response:")
for response in responses:
    print(response.text, end="")
```

**Gemini Pro Vision**
- Multimodal understanding (text + images)
- Supports multiple images per request
- Use cases: Image analysis, OCR, visual Q&A, diagram understanding

```python
from vertexai.generative_models import GenerativeModel, Part
import base64

# Load Gemini Pro Vision
vision_model = GenerativeModel("gemini-1.0-pro-vision")

# Image from Cloud Storage
image_part = Part.from_uri(
    uri="gs://my-bucket/architecture-diagram.png",
    mime_type="image/png"
)

response = vision_model.generate_content([
    "Analyze this architecture diagram and explain the components and data flow",
    image_part
])

print(f"Analysis: {response.text}")

# Multiple images
image1 = Part.from_uri("gs://my-bucket/before.jpg", mime_type="image/jpeg")
image2 = Part.from_uri("gs://my-bucket/after.jpg", mime_type="image/jpeg")

response = vision_model.generate_content([
    "Compare these two images and describe the differences:",
    image1,
    image2
])

print(f"Comparison: {response.text}")

# Image from local file
def load_image_from_file(file_path):
    with open(file_path, "rb") as f:
        image_data = f.read()
    return Part.from_data(data=image_data, mime_type="image/jpeg")

local_image = load_image_from_file("/path/to/screenshot.jpg")
response = vision_model.generate_content([
    "Extract and format all text from this screenshot",
    local_image
])

print(f"Extracted text: {response.text}")

# Video understanding
video_part = Part.from_uri(
    uri="gs://my-bucket/tutorial-video.mp4",
    mime_type="video/mp4"
)

response = vision_model.generate_content([
    "Summarize the key points from this tutorial video",
    video_part
])

print(f"Summary: {response.text}")
```

**Gemini Function Calling**
- Structured tool use and API integration
- Extract structured data from responses
- Enable models to call external functions

```python
from vertexai.generative_models import (
    GenerativeModel,
    Tool,
    FunctionDeclaration,
    Part
)
import json

# Define function declarations
get_weather_func = FunctionDeclaration(
    name="get_current_weather",
    description="Get the current weather in a given location",
    parameters={
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The city and state, e.g. San Francisco, CA"
            },
            "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"],
                "description": "Temperature unit"
            }
        },
        "required": ["location"]
    }
)

search_product_func = FunctionDeclaration(
    name="search_products",
    description="Search for products in inventory",
    parameters={
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "Search query"
            },
            "category": {
                "type": "string",
                "description": "Product category"
            },
            "max_results": {
                "type": "integer",
                "description": "Maximum number of results to return"
            }
        },
        "required": ["query"]
    }
)

# Create tool
tools = Tool(
    function_declarations=[get_weather_func, search_product_func]
)

# Initialize model with tools
model = GenerativeModel(
    "gemini-1.0-pro",
    tools=[tools]
)

# Send request
chat = model.start_chat()
response = chat.send_message("What's the weather like in Boston?")

# Extract function call
function_call = response.candidates[0].content.parts[0].function_call
print(f"Function: {function_call.name}")
print(f"Arguments: {dict(function_call.args)}")

# Simulate function execution
def get_current_weather(location, unit="fahrenheit"):
    # Simulate API call
    return {
        "location": location,
        "temperature": 72,
        "unit": unit,
        "conditions": "Partly cloudy"
    }

# Execute function
weather_data = get_current_weather(
    location=function_call.args["location"],
    unit=function_call.args.get("unit", "fahrenheit")
)

# Return result to model
response = chat.send_message(
    Part.from_function_response(
        name="get_current_weather",
        response={"content": weather_data}
    )
)

print(f"Final response: {response.text}")

# JSON mode for structured output
model = GenerativeModel("gemini-1.0-pro")

prompt = """
Extract the following information from this text and return as JSON:
- Company name
- Products mentioned
- Key features
- Pricing

Text: "Acme Corp offers CloudSync Pro for $99/month with features including
real-time synchronization, end-to-end encryption, and 1TB storage."
"""

response = model.generate_content(
    prompt,
    generation_config={
        "response_mime_type": "application/json"
    }
)

data = json.loads(response.text)
print(f"Structured data: {json.dumps(data, indent=2)}")
```

### Model Tuning

#### Supervised Fine-Tuning

**When to Use Fine-Tuning**
- Domain-specific terminology or jargon
- Consistent output format or style
- Specialized knowledge not in base model
- Task-specific performance improvement
- Reducing prompt length and cost

**Tuning Decision Matrix**

| Scenario | Use Fine-Tuning | Use Prompt Engineering | Use RAG |
|----------|----------------|----------------------|---------|
| Domain terminology | Yes | No | No |
| Output formatting | Yes | Maybe | No |
| Recent information | No | No | Yes |
| Large knowledge base | No | No | Yes |
| Few examples needed | No | Yes | No |
| Consistent style | Yes | Maybe | No |
| Low latency required | Yes | No | Maybe |
| Limited budget | No | Yes | Maybe |

**Text Model Tuning**

```python
from vertexai.language_models import TextGenerationModel
import pandas as pd

# Prepare training data
training_data = pd.DataFrame({
    "input_text": [
        "Classify the urgency: Customer cannot access account",
        "Classify the urgency: Request for product brochure",
        "Classify the urgency: System is completely down",
        # ... more examples
    ],
    "output_text": [
        "HIGH: Account access issue requires immediate attention",
        "LOW: Informational request, can be handled in regular queue",
        "CRITICAL: System outage affecting all users",
        # ... corresponding outputs
    ]
})

# Save to JSONL format
training_data.to_json("training_data.jsonl", orient="records", lines=True)

# Upload to Cloud Storage
from google.cloud import storage

storage_client = storage.Client()
bucket = storage_client.bucket("my-training-bucket")
blob = bucket.blob("tuning-data/training_data.jsonl")
blob.upload_from_filename("training_data.jsonl")

# Load base model
base_model = TextGenerationModel.from_pretrained("text-bison@002")

# Start tuning job
tuning_job = base_model.tune_model(
    training_data="gs://my-training-bucket/tuning-data/training_data.jsonl",
    train_steps=100,
    tuning_job_location="us-central1",
    tuned_model_location="us-central1",
    model_display_name="urgency-classifier-v1",
    tuning_evaluation_spec={
        "evaluation_data": "gs://my-training-bucket/tuning-data/eval_data.jsonl",
        "evaluation_interval": 20,
        "enable_early_stopping": True
    },
    learning_rate=0.001,
    learning_rate_multiplier=1.0,
)

print(f"Tuning job: {tuning_job.name}")
print(f"Status: {tuning_job.state}")

# Monitor tuning progress
tuning_job.wait()

# Get tuned model
tuned_model = tuning_job.get_tuned_model()

# Use tuned model
response = tuned_model.predict(
    prompt="Classify the urgency: Email server responding slowly",
    temperature=0.2,
    max_output_tokens=100
)

print(f"Classification: {response.text}")

# Compare with base model
base_response = base_model.predict(
    prompt="Classify the urgency: Email server responding slowly",
    temperature=0.2,
    max_output_tokens=100
)

print(f"Base model: {base_response.text}")
print(f"Tuned model: {response.text}")
```

**Chat Model Tuning**

```python
from vertexai.language_models import ChatModel

# Prepare chat training data with context and examples
training_data = [
    {
        "input_text": "What are your business hours?",
        "output_text": "Our support team is available Monday through Friday, 9 AM to 5 PM EST. For urgent issues outside business hours, please use our emergency hotline at 1-800-SUPPORT."
    },
    {
        "input_text": "How do I reset my password?",
        "output_text": "To reset your password: 1) Go to the login page, 2) Click 'Forgot Password', 3) Enter your email, 4) Follow the link in your email, 5) Create a new password with at least 12 characters."
    },
    # ... more examples
]

# Save training data
import json
with open("chat_training.jsonl", "w") as f:
    for item in training_data:
        f.write(json.dumps(item) + "\n")

# Upload to GCS
blob = bucket.blob("tuning-data/chat_training.jsonl")
blob.upload_from_filename("chat_training.jsonl")

# Load base chat model
base_chat_model = ChatModel.from_pretrained("chat-bison@002")

# Tune chat model
tuning_job = base_chat_model.tune_model(
    training_data="gs://my-training-bucket/tuning-data/chat_training.jsonl",
    train_steps=200,
    tuning_job_location="us-central1",
    tuned_model_location="us-central1",
    model_display_name="customer-support-chat-v1",
    context="You are a helpful customer support agent for TechCorp. Be professional, concise, and provide step-by-step instructions when appropriate.",
    learning_rate_multiplier=1.0,
    tuning_evaluation_spec={
        "evaluation_data": "gs://my-training-bucket/tuning-data/chat_eval.jsonl",
        "evaluation_interval": 50
    }
)

# Wait for completion
tuning_job.wait()

# Use tuned chat model
tuned_chat_model = tuning_job.get_tuned_model()
chat = tuned_chat_model.start_chat()

response = chat.send_message("I can't log into my account")
print(f"Tuned model response: {response.text}")
```

#### Adapter Tuning (Parameter-Efficient Fine-Tuning)

**Benefits**
- Faster training (typically 10-20x faster)
- Lower cost (smaller model to store)
- Multiple adapters for same base model
- Easy to switch between tasks

```python
# Adapter tuning for text model
adapter_tuning_job = base_model.tune_model(
    training_data="gs://my-training-bucket/tuning-data/training_data.jsonl",
    train_steps=50,  # Fewer steps needed
    adapter_size=4,  # Adapter dimensions (1, 4, 8, 16)
    tuning_job_location="us-central1",
    model_display_name="urgency-classifier-adapter-v1"
)

# Multiple adapters for different tasks
summarization_adapter = base_model.tune_model(
    training_data="gs://my-bucket/summarization_data.jsonl",
    train_steps=50,
    adapter_size=8,
    model_display_name="summarization-adapter"
)

classification_adapter = base_model.tune_model(
    training_data="gs://my-bucket/classification_data.jsonl",
    train_steps=50,
    adapter_size=8,
    model_display_name="classification-adapter"
)

# Switch between adapters at inference time
summary = summarization_adapter.get_tuned_model().predict("Long text to summarize...")
category = classification_adapter.get_tuned_model().predict("Text to classify...")
```

#### Reinforcement Learning from Human Feedback (RLHF)

**Conceptual Workflow**
1. Collect human preference data
2. Train reward model
3. Fine-tune with RL algorithm
4. Evaluate and iterate

```python
# Note: RLHF typically requires custom implementation
# This shows the conceptual approach

# Step 1: Collect preference data
preference_data = [
    {
        "prompt": "Explain quantum computing",
        "response_a": "Response A text...",
        "response_b": "Response B text...",
        "preferred": "response_b",  # Human preference
        "feedback": "More clear and concise"
    },
    # ... more preference pairs
]

# Step 2: Prepare RLHF training data
# Convert preferences to reward signals
rlhf_training_data = []
for item in preference_data:
    rlhf_training_data.append({
        "input_text": item["prompt"],
        "output_text": item[item["preferred"]],
        "reward": 1.0
    })

    # Add negative example
    not_preferred = "response_a" if item["preferred"] == "response_b" else "response_b"
    rlhf_training_data.append({
        "input_text": item["prompt"],
        "output_text": item[not_preferred],
        "reward": -0.5
    })

# Step 3: Tune with preference data
# This uses standard tuning but with preference-optimized data
tuned_model = base_model.tune_model(
    training_data="gs://my-bucket/rlhf_training.jsonl",
    train_steps=100,
    learning_rate_multiplier=0.5  # Lower LR for stability
)
```

### Grounding

**Grounding** connects model responses to external knowledge sources, reducing hallucinations and providing citations.

#### Grounding with Google Search

```python
from vertexai.preview.generative_models import (
    GenerativeModel,
    Tool,
    grounding
)

# Initialize model with Google Search grounding
model = GenerativeModel("gemini-1.0-pro")

# Create grounding tool
google_search_tool = Tool.from_google_search_retrieval(
    grounding.GoogleSearchRetrieval()
)

# Generate with grounding
response = model.generate_content(
    "What are the latest developments in quantum computing announced in 2024?",
    tools=[google_search_tool],
    generation_config={
        "temperature": 0.3,
        "max_output_tokens": 1024
    }
)

print(f"Response: {response.text}")

# Access grounding metadata
if response.candidates[0].grounding_metadata:
    grounding_metadata = response.candidates[0].grounding_metadata
    print(f"\nGrounding sources:")
    for chunk in grounding_metadata.grounding_chunks:
        print(f"  - {chunk.web.uri}")
        print(f"    Title: {chunk.web.title}")

    print(f"\nGrounding support: {grounding_metadata.grounding_support}")
```

#### Grounding with Vertex AI Search

```python
from vertexai.preview.generative_models import grounding

# Create Vertex AI Search datastore
# (Done via Console or gcloud CLI)
# gcloud alpha discovery-engine data-stores create my-datastore \
#   --location=global \
#   --collection=default_collection \
#   --industry-vertical=GENERIC

# Initialize model with Vertex AI Search grounding
vertex_search_tool = Tool.from_retrieval(
    grounding.Retrieval(
        source=grounding.VertexAISearch(
            datastore="projects/my-project/locations/global/collections/default_collection/dataStores/my-datastore"
        )
    )
)

model = GenerativeModel("gemini-1.0-pro")

# Query with grounded responses
response = model.generate_content(
    "What is our company's return policy for electronics?",
    tools=[vertex_search_tool]
)

print(f"Grounded response: {response.text}")

# Get source documents
for chunk in response.candidates[0].grounding_metadata.grounding_chunks:
    print(f"Source: {chunk.retrieved_context.uri}")
    print(f"Content: {chunk.retrieved_context.text[:200]}...")
```

#### Grounding with Custom Data Sources

```python
# Grounding using Vector Search / Matching Engine
from google.cloud import aiplatform_v1
from vertexai.preview.generative_models import GenerativeModel
import numpy as np

# Step 1: Create vector embeddings of your documents
from vertexai.language_models import TextEmbeddingModel

embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")

documents = [
    {
        "id": "doc1",
        "text": "Our return policy allows returns within 30 days...",
        "category": "policy"
    },
    {
        "id": "doc2",
        "text": "Product warranty covers manufacturing defects for 1 year...",
        "category": "warranty"
    },
    # ... more documents
]

# Generate embeddings
for doc in documents:
    embedding = embedding_model.get_embeddings([doc["text"]])[0]
    doc["embedding"] = embedding.values

# Step 2: Store in Vector Search (Matching Engine)
# Create index (done via API or Console)

# Step 3: Query-time retrieval
def retrieve_relevant_docs(query, top_k=5):
    # Get query embedding
    query_embedding = embedding_model.get_embeddings([query])[0].values

    # Search vector index
    # (Simplified - actual implementation uses Matching Engine API)
    similarities = []
    for doc in documents:
        similarity = np.dot(query_embedding, doc["embedding"]) / (
            np.linalg.norm(query_embedding) * np.linalg.norm(doc["embedding"])
        )
        similarities.append((similarity, doc))

    # Return top-k most similar
    similarities.sort(reverse=True)
    return [doc for _, doc in similarities[:top_k]]

# Step 4: Use retrieved docs as context
query = "What is the return policy?"
relevant_docs = retrieve_relevant_docs(query)

# Build context from retrieved documents
context = "\n\n".join([
    f"[Source {i+1}]\n{doc['text']}"
    for i, doc in enumerate(relevant_docs)
])

# Generate with context
model = GenerativeModel("gemini-1.0-pro")
prompt = f"""Based on the following information, answer the question.

Context:
{context}

Question: {query}

Answer:"""

response = model.generate_content(prompt)
print(f"Answer: {response.text}")
print(f"\nSources used:")
for i, doc in enumerate(relevant_docs):
    print(f"  {i+1}. {doc['id']}: {doc['text'][:100]}...")
```

### Extensions and Function Calling

#### API Integration

```python
from vertexai.generative_models import (
    GenerativeModel,
    Tool,
    FunctionDeclaration,
    Part
)
import requests

# Define function to call external API
get_stock_price_func = FunctionDeclaration(
    name="get_stock_price",
    description="Get current stock price for a ticker symbol",
    parameters={
        "type": "object",
        "properties": {
            "ticker": {
                "type": "string",
                "description": "Stock ticker symbol (e.g., GOOGL, AAPL)"
            }
        },
        "required": ["ticker"]
    }
)

get_exchange_rate_func = FunctionDeclaration(
    name="get_exchange_rate",
    description="Get currency exchange rate",
    parameters={
        "type": "object",
        "properties": {
            "from_currency": {
                "type": "string",
                "description": "Source currency code (e.g., USD)"
            },
            "to_currency": {
                "type": "string",
                "description": "Target currency code (e.g., EUR)"
            }
        },
        "required": ["from_currency", "to_currency"]
    }
)

# Create tool
financial_tool = Tool(
    function_declarations=[get_stock_price_func, get_exchange_rate_func]
)

# Initialize model
model = GenerativeModel("gemini-1.0-pro", tools=[financial_tool])

# Implementation of functions
def get_stock_price(ticker):
    # Call actual financial API
    # response = requests.get(f"https://api.example.com/stock/{ticker}")
    # return response.json()

    # Simulated response
    return {
        "ticker": ticker,
        "price": 142.50,
        "currency": "USD",
        "timestamp": "2024-01-15T10:30:00Z"
    }

def get_exchange_rate(from_currency, to_currency):
    # Call actual exchange rate API
    return {
        "from": from_currency,
        "to": to_currency,
        "rate": 0.85,
        "timestamp": "2024-01-15T10:30:00Z"
    }

# Use model with function calling
chat = model.start_chat()

response = chat.send_message(
    "What is the current price of Google stock and convert it to Euros?"
)

# Process function calls
while response.candidates[0].content.parts[0].function_call:
    function_call = response.candidates[0].content.parts[0].function_call

    # Execute appropriate function
    if function_call.name == "get_stock_price":
        result = get_stock_price(function_call.args["ticker"])
    elif function_call.name == "get_exchange_rate":
        result = get_exchange_rate(
            function_call.args["from_currency"],
            function_call.args["to_currency"]
        )

    # Send result back to model
    response = chat.send_message(
        Part.from_function_response(
            name=function_call.name,
            response={"content": result}
        )
    )

# Final natural language response
print(f"Answer: {response.text}")
```

#### Multi-Tool Orchestration

```python
# Define multiple tools for complex workflows
database_query_func = FunctionDeclaration(
    name="query_database",
    description="Query the customer database",
    parameters={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "SQL query to execute"},
            "database": {"type": "string", "description": "Database name"}
        },
        "required": ["query", "database"]
    }
)

send_email_func = FunctionDeclaration(
    name="send_email",
    description="Send an email to a customer",
    parameters={
        "type": "object",
        "properties": {
            "to": {"type": "string", "description": "Recipient email"},
            "subject": {"type": "string", "description": "Email subject"},
            "body": {"type": "string", "description": "Email body"}
        },
        "required": ["to", "subject", "body"]
    }
)

create_ticket_func = FunctionDeclaration(
    name="create_support_ticket",
    description="Create a customer support ticket",
    parameters={
        "type": "object",
        "properties": {
            "customer_id": {"type": "string"},
            "priority": {"type": "string", "enum": ["low", "medium", "high", "critical"]},
            "description": {"type": "string"}
        },
        "required": ["customer_id", "description"]
    }
)

# Create tool with multiple functions
support_tool = Tool(
    function_declarations=[database_query_func, send_email_func, create_ticket_func]
)

# Initialize model
model = GenerativeModel("gemini-1.0-pro", tools=[support_tool])

# Complex multi-step request
chat = model.start_chat()
response = chat.send_message(
    """
    Customer John Doe (john@example.com) reported that his order #12345 hasn't arrived.
    Please:
    1. Look up the order status in the database
    2. Create a high-priority support ticket
    3. Send an email to the customer acknowledging the issue
    """
)

# Process multiple function calls
function_responses = []
while True:
    if not response.candidates[0].content.parts[0].function_call:
        break

    function_call = response.candidates[0].content.parts[0].function_call
    print(f"Executing: {function_call.name}")

    # Execute function based on name
    if function_call.name == "query_database":
        result = query_database(function_call.args["query"], function_call.args["database"])
    elif function_call.name == "create_support_ticket":
        result = create_support_ticket(**dict(function_call.args))
    elif function_call.name == "send_email":
        result = send_email(**dict(function_call.args))

    # Send result back
    response = chat.send_message(
        Part.from_function_response(
            name=function_call.name,
            response={"content": result}
        )
    )

    function_responses.append({
        "function": function_call.name,
        "result": result
    })

print(f"Final response: {response.text}")
print(f"Functions executed: {len(function_responses)}")
```

### RAG (Retrieval Augmented Generation)

**RAG Architecture** combines generative AI with information retrieval to provide accurate, grounded responses with citations.

#### RAG Components

**1. Document Processing Pipeline**
```python
from google.cloud import storage, documentai_v1
from vertexai.language_models import TextEmbeddingModel
import PyPDF2
import json

# Initialize services
storage_client = storage.Client()
embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")

# Step 1: Extract text from documents
def extract_text_from_pdf(gcs_uri):
    """Extract text from PDF in Cloud Storage"""
    bucket_name, blob_name = gcs_uri.replace("gs://", "").split("/", 1)
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(blob_name)

    # Download PDF
    pdf_bytes = blob.download_as_bytes()

    # Extract text
    pdf_reader = PyPDF2.PdfReader(io.BytesIO(pdf_bytes))
    text = ""
    for page in pdf_reader.pages:
        text += page.extract_text()

    return text

# Step 2: Chunk documents
def chunk_text(text, chunk_size=500, overlap=50):
    """Split text into overlapping chunks"""
    words = text.split()
    chunks = []

    for i in range(0, len(words), chunk_size - overlap):
        chunk = " ".join(words[i:i + chunk_size])
        chunks.append(chunk)

    return chunks

# Step 3: Generate embeddings
def embed_chunks(chunks, metadata=None):
    """Generate embeddings for text chunks"""
    embedded_chunks = []

    # Batch process for efficiency
    batch_size = 250
    for i in range(0, len(chunks), batch_size):
        batch = chunks[i:i + batch_size]
        embeddings = embedding_model.get_embeddings(batch)

        for j, embedding in enumerate(embeddings):
            embedded_chunks.append({
                "text": batch[j],
                "embedding": embedding.values,
                "metadata": metadata[i + j] if metadata else {}
            })

    return embedded_chunks

# Process document
document_uri = "gs://my-bucket/documents/product-manual.pdf"
text = extract_text_from_pdf(document_uri)
chunks = chunk_text(text, chunk_size=500, overlap=50)

metadata = [
    {
        "source": document_uri,
        "chunk_id": i,
        "document_type": "product_manual"
    }
    for i in range(len(chunks))
]

embedded_chunks = embed_chunks(chunks, metadata)
print(f"Processed {len(embedded_chunks)} chunks")
```

**2. Vector Search with Matching Engine**

```python
from google.cloud import aiplatform
import json

# Initialize Vertex AI
aiplatform.init(project="my-project", location="us-central1")

# Create Matching Engine Index
def create_vector_index(embedded_chunks, index_name):
    """Create vector search index"""

    # Prepare data for index
    vectors = []
    for i, chunk in enumerate(embedded_chunks):
        vectors.append({
            "id": str(i),
            "embedding": chunk["embedding"],
            "restricts": [
                {"namespace": "document_type", "allow_list": [chunk["metadata"]["document_type"]]}
            ]
        })

    # Save vectors to GCS
    vectors_file = f"gs://my-bucket/vector-index/vectors.json"
    with open("vectors.json", "w") as f:
        for vector in vectors:
            f.write(json.dumps(vector) + "\n")

    # Upload to GCS
    bucket = storage_client.bucket("my-bucket")
    blob = bucket.blob("vector-index/vectors.json")
    blob.upload_from_filename("vectors.json")

    # Create index
    index = aiplatform.MatchingEngineIndex.create_tree_ah_index(
        display_name=index_name,
        contents_delta_uri=vectors_file,
        dimensions=768,  # textembedding-gecko dimensions
        approximate_neighbors_count=10,
        distance_measure_type="DOT_PRODUCT_DISTANCE",
        leaf_node_embedding_count=500,
        leaf_nodes_to_search_percent=5,
    )

    return index

# Create index
# index = create_vector_index(embedded_chunks, "document-index-v1")

# Deploy index to endpoint
def deploy_index(index):
    """Deploy index to endpoint"""
    endpoint = aiplatform.MatchingEngineIndexEndpoint.create(
        display_name="document-search-endpoint",
        public_endpoint_enabled=True
    )

    endpoint.deploy_index(
        index=index,
        deployed_index_id="doc_index_deployed",
        display_name="Document Search",
        machine_type="n1-standard-2",
        min_replica_count=1,
        max_replica_count=5
    )

    return endpoint

# endpoint = deploy_index(index)
```

**3. Query and Retrieval**

```python
def search_similar_chunks(query, endpoint, top_k=5):
    """Search for similar chunks using vector similarity"""

    # Generate query embedding
    query_embedding = embedding_model.get_embeddings([query])[0].values

    # Query index endpoint
    response = endpoint.find_neighbors(
        deployed_index_id="doc_index_deployed",
        queries=[query_embedding],
        num_neighbors=top_k
    )

    # Get matching chunks
    matches = []
    for neighbor in response[0]:
        chunk_id = int(neighbor.id)
        chunk = embedded_chunks[chunk_id]
        matches.append({
            "text": chunk["text"],
            "metadata": chunk["metadata"],
            "score": neighbor.distance
        })

    return matches

# Search for relevant information
query = "How do I troubleshoot connectivity issues?"
relevant_chunks = search_similar_chunks(query, endpoint, top_k=5)

for i, chunk in enumerate(relevant_chunks):
    print(f"\n{i+1}. Score: {chunk['score']:.4f}")
    print(f"   Text: {chunk['text'][:200]}...")
    print(f"   Source: {chunk['metadata']['source']}")
```

**4. Generate with Retrieved Context**

```python
from vertexai.generative_models import GenerativeModel

def rag_query(question, endpoint, model_name="gemini-1.0-pro"):
    """RAG: Retrieve relevant context and generate answer"""

    # Step 1: Retrieve relevant chunks
    relevant_chunks = search_similar_chunks(question, endpoint, top_k=5)

    # Step 2: Build context
    context = "\n\n".join([
        f"[Source {i+1}: {chunk['metadata']['source']}]\n{chunk['text']}"
        for i, chunk in enumerate(relevant_chunks)
    ])

    # Step 3: Generate with context
    model = GenerativeModel(model_name)

    prompt = f"""Answer the question based on the provided context. If the answer is not in the context, say "I don't have enough information to answer that."

Context:
{context}

Question: {question}

Answer:"""

    response = model.generate_content(
        prompt,
        generation_config={
            "temperature": 0.2,
            "max_output_tokens": 1024,
            "top_p": 0.8
        }
    )

    return {
        "answer": response.text,
        "sources": [chunk["metadata"] for chunk in relevant_chunks],
        "context_chunks": [chunk["text"] for chunk in relevant_chunks]
    }

# Use RAG system
result = rag_query("How do I troubleshoot connectivity issues?", endpoint)

print(f"Answer: {result['answer']}\n")
print(f"Sources:")
for i, source in enumerate(result['sources']):
    print(f"  {i+1}. {source['source']} (chunk {source['chunk_id']})")
```

**5. Advanced RAG: Hybrid Search**

```python
def hybrid_search(query, endpoint, keyword_index, top_k=10):
    """Combine vector search with keyword search"""

    # Vector search
    vector_results = search_similar_chunks(query, endpoint, top_k=top_k)

    # Keyword search (using BigQuery or Elasticsearch)
    # Simplified example
    keyword_results = []
    query_words = query.lower().split()
    for chunk in embedded_chunks:
        chunk_words = chunk["text"].lower().split()
        overlap = len(set(query_words) & set(chunk_words))
        if overlap > 0:
            keyword_results.append({
                "chunk": chunk,
                "keyword_score": overlap / len(query_words)
            })

    keyword_results.sort(key=lambda x: x["keyword_score"], reverse=True)

    # Combine and re-rank
    # Using Reciprocal Rank Fusion (RRF)
    combined_scores = {}

    for i, result in enumerate(vector_results):
        chunk_id = result["metadata"]["chunk_id"]
        combined_scores[chunk_id] = combined_scores.get(chunk_id, 0) + 1 / (i + 1)

    for i, result in enumerate(keyword_results[:top_k]):
        chunk_id = result["chunk"]["metadata"]["chunk_id"]
        combined_scores[chunk_id] = combined_scores.get(chunk_id, 0) + 1 / (i + 1)

    # Sort by combined score
    ranked_chunks = sorted(
        [(chunk_id, score) for chunk_id, score in combined_scores.items()],
        key=lambda x: x[1],
        reverse=True
    )

    return [embedded_chunks[chunk_id] for chunk_id, _ in ranked_chunks[:top_k]]

# Use hybrid search
results = hybrid_search("connectivity troubleshooting", endpoint, None, top_k=5)
```

**6. RAG with Query Rewriting**

```python
def rag_with_query_expansion(question, endpoint):
    """Improve retrieval with query expansion"""

    # Generate alternative query formulations
    model = GenerativeModel("gemini-1.0-pro")

    expansion_prompt = f"""Generate 3 alternative ways to ask this question:

Original: {question}

Alternatives (one per line):"""

    response = model.generate_content(expansion_prompt, generation_config={"temperature": 0.7})
    alternative_queries = [q.strip() for q in response.text.split("\n") if q.strip()]

    # Search with all queries
    all_chunks = []
    seen_chunks = set()

    for query in [question] + alternative_queries:
        chunks = search_similar_chunks(query, endpoint, top_k=3)
        for chunk in chunks:
            chunk_id = chunk["metadata"]["chunk_id"]
            if chunk_id not in seen_chunks:
                all_chunks.append(chunk)
                seen_chunks.add(chunk_id)

    # Build context and generate
    context = "\n\n".join([
        f"[Source {i+1}]\n{chunk['text']}"
        for i, chunk in enumerate(all_chunks[:10])
    ])

    prompt = f"""Based on the context, answer the question.

Context:
{context}

Question: {question}

Answer:"""

    response = model.generate_content(prompt)
    return response.text

# Use query expansion
answer = rag_with_query_expansion(
    "What should I do if the device won't connect?",
    endpoint
)
print(f"Answer: {answer}")
```

**7. RAG Evaluation**

```python
def evaluate_rag_system(test_questions, endpoint):
    """Evaluate RAG system quality"""

    results = []

    for question_data in test_questions:
        question = question_data["question"]
        expected_answer = question_data.get("expected_answer")

        # Get RAG result
        result = rag_query(question, endpoint)

        # Evaluate retrieval
        relevant_retrieved = any(
            source["chunk_id"] in question_data.get("relevant_chunks", [])
            for source in result["sources"]
        )

        # Evaluate answer quality (using model)
        model = GenerativeModel("gemini-1.0-pro")
        eval_prompt = f"""Evaluate if the answer correctly addresses the question.

Question: {question}
Answer: {result['answer']}
Expected: {expected_answer}

Is the answer correct and complete? (YES/NO):"""

        eval_response = model.generate_content(eval_prompt, generation_config={"temperature": 0.1})

        results.append({
            "question": question,
            "retrieval_success": relevant_retrieved,
            "answer_quality": eval_response.text.strip(),
            "num_sources": len(result["sources"])
        })

    # Calculate metrics
    retrieval_success_rate = sum(r["retrieval_success"] for r in results) / len(results)
    print(f"Retrieval Success Rate: {retrieval_success_rate:.2%}")

    return results

# Test RAG system
test_cases = [
    {
        "question": "How do I reset the device?",
        "expected_answer": "Press and hold the power button for 10 seconds",
        "relevant_chunks": [5, 12, 23]
    },
    # ... more test cases
]

# evaluation_results = evaluate_rag_system(test_cases, endpoint)
```

### Enterprise Features

#### Private Endpoints

```python
from google.cloud import aiplatform

# Initialize with VPC configuration
aiplatform.init(
    project="my-project",
    location="us-central1",
)

# Deploy model with private endpoint
def deploy_private_endpoint(model, vpc_network):
    """Deploy model to private VPC endpoint"""

    # Create private endpoint
    endpoint = aiplatform.Endpoint.create(
        display_name="private-genai-endpoint",
        network=f"projects/{PROJECT_NUMBER}/global/networks/{vpc_network}",
        encryption_spec_key_name=f"projects/{PROJECT_ID}/locations/{LOCATION}/keyRings/{KEY_RING}/cryptoKeys/{KEY}"
    )

    # Deploy model
    endpoint.deploy(
        model=model,
        deployed_model_display_name="private-model",
        machine_type="n1-standard-4",
        min_replica_count=2,
        max_replica_count=10,
        traffic_split={"0": 100},
        enable_access_logging=True
    )

    return endpoint

# Use private endpoint
private_endpoint = deploy_private_endpoint(
    model,
    vpc_network="my-vpc"
)

# Prediction from within VPC
prediction = private_endpoint.predict(instances=data)
```

#### VPC Peering and Private Service Connect

```python
# Configure VPC peering for Vertex AI
# (Done via gcloud or Console)

# gcloud compute networks peerings create vertex-ai-peering \
#     --network=my-vpc \
#     --peer-project=vertex-ai-project \
#     --peer-network=vertex-ai-network \
#     --import-custom-routes \
#     --export-custom-routes

# Use Private Service Connect
from google.cloud import aiplatform

# Create endpoint with PSC
psc_endpoint = aiplatform.Endpoint.create(
    display_name="psc-genai-endpoint",
    private_service_connect_config={
        "enable_private_service_connect": True,
        "project_allowlist": [PROJECT_ID]
    }
)

# Access via Private Service Connect attachment
# Requests stay within Google's network
```

#### Data Residency and Compliance

```python
# Specify data residency requirements
import vertexai

# Initialize with specific region
vertexai.init(
    project="my-project",
    location="europe-west4"  # Data stays in Europe
)

# Load model from specific region
from vertexai.language_models import TextGenerationModel

model = TextGenerationModel.from_pretrained(
    "text-bison@002"
)

# Predictions processed in specified region
response = model.predict(
    prompt="Sensitive data processing...",
    # Data processed in europe-west4
)

# Store tuned models with residency constraints
tuning_job = model.tune_model(
    training_data="gs://eu-bucket/training-data.jsonl",
    tuning_job_location="europe-west4",
    tuned_model_location="europe-west4",  # Model stays in EU
    model_display_name="eu-compliant-model"
)
```

#### Security and Encryption

```python
from google.cloud import aiplatform
from google.cloud import kms_v1

# Customer-Managed Encryption Keys (CMEK)
def deploy_with_cmek(model):
    """Deploy model with CMEK encryption"""

    # KMS key resource name
    kms_key = f"projects/{PROJECT_ID}/locations/{LOCATION}/keyRings/{KEY_RING}/cryptoKeys/{KEY}"

    # Create endpoint with CMEK
    endpoint = aiplatform.Endpoint.create(
        display_name="cmek-encrypted-endpoint",
        encryption_spec_key_name=kms_key
    )

    # Deploy model
    endpoint.deploy(
        model=model,
        deployed_model_display_name="encrypted-model",
        machine_type="n1-standard-4"
    )

    return endpoint

# VPC Service Controls
# Define security perimeter
from google.cloud import accesscontextmanager_v1

def create_service_perimeter():
    """Create VPC Service Controls perimeter"""

    client = accesscontextmanager_v1.AccessContextManagerClient()

    perimeter = {
        "name": f"accessPolicies/{POLICY_ID}/servicePerimeters/vertex_ai_perimeter",
        "title": "Vertex AI Perimeter",
        "perimeter_type": "PERIMETER_TYPE_REGULAR",
        "status": {
            "resources": [
                f"projects/{PROJECT_NUMBER}"
            ],
            "restricted_services": [
                "aiplatform.googleapis.com"
            ],
            "vpc_accessible_services": {
                "enable_restriction": True,
                "allowed_services": [
                    "aiplatform.googleapis.com",
                    "storage.googleapis.com"
                ]
            }
        }
    }

    # Create perimeter
    # operation = client.create_service_perimeter(...)
    return perimeter

# Audit logging
def enable_audit_logging():
    """Enable comprehensive audit logging"""

    # Cloud Logging configuration
    logging_config = {
        "log_type": "DATA_ACCESS",
        "exempted_members": []
    }

    # All Vertex AI API calls logged
    # View in Cloud Logging:
    # resource.type="aiplatform.googleapis.com/Endpoint"
    # protoPayload.methodName="google.cloud.aiplatform.v1.PredictionService.Predict"
```

#### Identity and Access Management

```python
# IAM roles for Vertex AI Gen AI
from google.cloud import resourcemanager_v3

def configure_iam_for_genai():
    """Configure IAM roles for Gen AI workloads"""

    # Predefined roles
    roles = {
        "roles/aiplatform.user": "Use Vertex AI resources",
        "roles/aiplatform.admin": "Full Vertex AI administration",
        "roles/aiplatform.modelUser": "Use deployed models",
        "roles/aiplatform.viewer": "View Vertex AI resources",
        "roles/aiplatform.featurestoreUser": "Access feature store",
        "roles/aiplatform.customCodeServiceAgent": "Custom code execution"
    }

    # Grant model user access to service account
    # gcloud projects add-iam-policy-binding PROJECT_ID \
    #     --member="serviceAccount:genai-app@PROJECT_ID.iam.gserviceaccount.com" \
    #     --role="roles/aiplatform.user"

    # Fine-grained permissions with custom role
    custom_permissions = [
        "aiplatform.endpoints.predict",
        "aiplatform.endpoints.get",
        "aiplatform.models.get"
    ]

    # Create custom role
    # gcloud iam roles create genaiModelUser \
    #     --project=PROJECT_ID \
    #     --permissions=aiplatform.endpoints.predict,aiplatform.endpoints.get

# Workload Identity for GKE
def configure_workload_identity():
    """Configure Workload Identity for Vertex AI access from GKE"""

    # Link Kubernetes SA to Google SA
    # gcloud iam service-accounts add-iam-policy-binding \
    #     genai-app@PROJECT_ID.iam.gserviceaccount.com \
    #     --role roles/iam.workloadIdentityUser \
    #     --member "serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"

    # Use from pod
    k8s_manifest = """
apiVersion: v1
kind: ServiceAccount
metadata:
  name: genai-app-sa
  annotations:
    iam.gke.io/gcp-service-account: genai-app@PROJECT_ID.iam.gserviceaccount.com
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: genai-app
spec:
  template:
    spec:
      serviceAccountName: genai-app-sa
      containers:
      - name: app
        image: gcr.io/PROJECT_ID/genai-app:latest
"""
```

#### Rate Limiting and Quotas

```python
# Implement rate limiting for Gen AI endpoints
from functools import wraps
import time
from collections import deque

class RateLimiter:
    """Token bucket rate limiter"""

    def __init__(self, requests_per_minute=60):
        self.requests_per_minute = requests_per_minute
        self.requests = deque()

    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()

            # Remove requests older than 1 minute
            while self.requests and self.requests[0] < now - 60:
                self.requests.popleft()

            # Check if rate limit exceeded
            if len(self.requests) >= self.requests_per_minute:
                sleep_time = 60 - (now - self.requests[0])
                if sleep_time > 0:
                    time.sleep(sleep_time)

            # Record request
            self.requests.append(time.time())

            return func(*args, **kwargs)

        return wrapper

# Use rate limiter
@RateLimiter(requests_per_minute=60)
def call_genai_model(prompt):
    model = TextGenerationModel.from_pretrained("text-bison@002")
    return model.predict(prompt)

# Monitor quotas
from google.cloud import monitoring_v3

def check_quota_usage():
    """Monitor Vertex AI API quota usage"""

    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{PROJECT_ID}"

    # Query quota metrics
    interval = monitoring_v3.TimeInterval(
        {
            "end_time": {"seconds": int(time.time())},
            "start_time": {"seconds": int(time.time()) - 3600},
        }
    )

    results = client.list_time_series(
        request={
            "name": project_name,
            "filter": 'metric.type="serviceruntime.googleapis.com/quota/rate/net_usage" AND resource.service="aiplatform.googleapis.com"',
            "interval": interval,
            "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL,
        }
    )

    for result in results:
        print(f"Quota: {result.metric.labels['quota_metric']}")
        for point in result.points:
            print(f"  Usage: {point.value.int64_value}")

# check_quota_usage()
```

### Vertex AI Gen AI Scenarios

#### Scenario 1: Enterprise Document Q&A System

```python
"""
Scenario: Build a Q&A system for internal company documentation
Requirements:
- Process 10,000+ internal documents
- Provide accurate answers with citations
- Support multiple document formats (PDF, DOCX, HTML)
- Maintain data privacy and security
- Scale to handle 1000+ concurrent users
"""

from vertexai.language_models import TextEmbeddingModel
from vertexai.generative_models import GenerativeModel
from google.cloud import storage, aiplatform
import PyPDF2
from docx import Document

class EnterpriseDocumentQA:
    def __init__(self, project_id, location="us-central1"):
        self.project_id = project_id
        self.location = location

        # Initialize models
        aiplatform.init(project=project_id, location=location)
        self.embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")
        self.generation_model = GenerativeModel("gemini-1.0-pro")

        # Storage
        self.storage_client = storage.Client()
        self.document_chunks = []

    def process_documents(self, gcs_folder_uri):
        """Process all documents in GCS folder"""

        # Extract bucket and prefix
        bucket_name = gcs_folder_uri.replace("gs://", "").split("/")[0]
        prefix = "/".join(gcs_folder_uri.replace("gs://", "").split("/")[1:])

        bucket = self.storage_client.bucket(bucket_name)
        blobs = bucket.list_blobs(prefix=prefix)

        for blob in blobs:
            if blob.name.endswith('.pdf'):
                text = self._extract_pdf(blob)
            elif blob.name.endswith('.docx'):
                text = self._extract_docx(blob)
            elif blob.name.endswith('.html'):
                text = self._extract_html(blob)
            else:
                continue

            # Chunk and embed
            chunks = self._chunk_document(text, blob.name)
            self.document_chunks.extend(chunks)

        # Create vector index
        self._create_vector_index()

    def _chunk_document(self, text, source):
        """Chunk document intelligently"""
        # Split by paragraphs
        paragraphs = text.split('\n\n')

        chunks = []
        current_chunk = ""

        for para in paragraphs:
            if len(current_chunk) + len(para) < 1000:
                current_chunk += para + "\n\n"
            else:
                if current_chunk:
                    chunks.append({
                        "text": current_chunk.strip(),
                        "source": source
                    })
                current_chunk = para + "\n\n"

        if current_chunk:
            chunks.append({"text": current_chunk.strip(), "source": source})

        return chunks

    def answer_question(self, question):
        """Answer question using RAG"""

        # Retrieve relevant chunks
        relevant_chunks = self._retrieve_chunks(question, top_k=5)

        # Build context
        context = "\n\n".join([
            f"[Document: {chunk['source']}]\n{chunk['text']}"
            for chunk in relevant_chunks
        ])

        # Generate answer
        prompt = f"""You are an AI assistant helping employees find information in company documents.
Answer the question based on the provided context. Include the document source in your answer.
If the information is not in the context, say so.

Context:
{context}

Question: {question}

Answer:"""

        response = self.generation_model.generate_content(
            prompt,
            generation_config={
                "temperature": 0.2,
                "max_output_tokens": 1024
            }
        )

        return {
            "answer": response.text,
            "sources": [chunk["source"] for chunk in relevant_chunks]
        }

# Usage
qa_system = EnterpriseDocumentQA("my-project")
qa_system.process_documents("gs://my-bucket/internal-docs/")

result = qa_system.answer_question("What is our remote work policy?")
print(f"Answer: {result['answer']}")
print(f"Sources: {result['sources']}")
```

#### Scenario 2: Code Generation and Review Assistant

```python
"""
Scenario: Build an AI assistant for code generation and review
Requirements:
- Generate code from natural language descriptions
- Review code for bugs and improvements
- Explain complex code sections
- Suggest optimizations
- Support multiple programming languages
"""

from vertexai.language_models import CodeGenerationModel, CodeChatModel
from vertexai.generative_models import GenerativeModel

class CodeAssistant:
    def __init__(self):
        self.code_model = CodeGenerationModel.from_pretrained("code-bison@002")
        self.code_chat = CodeChatModel.from_pretrained("codechat-bison@002")
        self.review_model = GenerativeModel("gemini-1.0-pro")

    def generate_code(self, description, language="python"):
        """Generate code from description"""

        prompt = f"""# Language: {language}
# Task: {description}
# Generate complete, production-ready code with error handling and documentation

"""

        response = self.code_model.predict(
            prefix=prompt,
            temperature=0.2,
            max_output_tokens=2048
        )

        return response.text

    def review_code(self, code, focus_areas=None):
        """Review code for issues and improvements"""

        focus = focus_areas if focus_areas else ["bugs", "performance", "security", "best practices"]

        prompt = f"""Review this code and provide feedback on: {', '.join(focus)}

Code:
```
{code}
```

Provide:
1. Issues found (with severity: HIGH/MEDIUM/LOW)
2. Suggested improvements
3. Security concerns
4. Performance optimization opportunities

Format as structured report:"""

        response = self.review_model.generate_content(
            prompt,
            generation_config={"temperature": 0.3, "max_output_tokens": 2048}
        )

        return response.text

    def explain_code(self, code):
        """Explain complex code"""

        chat = self.code_chat.start_chat()

        response = chat.send_message(
            f"""Explain this code in detail:

```
{code}
```

Include:
1. Purpose and functionality
2. Algorithm or approach used
3. Time and space complexity
4. Potential edge cases"""
        )

        return response.text

    def optimize_code(self, code, optimization_goal="performance"):
        """Optimize code for specific goal"""

        chat = self.code_chat.start_chat()

        response = chat.send_message(
            f"""Optimize this code for {optimization_goal}:

```
{code}
```

Provide:
1. Optimized code
2. Explanation of changes
3. Performance improvements
4. Trade-offs"""
        )

        return response.text

# Usage
assistant = CodeAssistant()

# Generate code
code = assistant.generate_code(
    "Create a rate limiter using token bucket algorithm",
    language="python"
)
print(f"Generated code:\n{code}")

# Review code
review = assistant.review_code(code)
print(f"\nCode review:\n{review}")

# Explain code
explanation = assistant.explain_code(code)
print(f"\nExplanation:\n{explanation}")
```

#### Scenario 3: Multi-Lingual Customer Support Chatbot

```python
"""
Scenario: Build a multi-lingual customer support chatbot
Requirements:
- Support 20+ languages
- Access company knowledge base
- Escalate complex issues to humans
- Track conversation history
- Provide sentiment analysis
"""

from vertexai.language_models import ChatModel
from vertexai.generative_models import GenerativeModel
from google.cloud import translate_v2 as translate
from google.cloud import firestore

class MultiLingualSupportBot:
    def __init__(self, project_id):
        self.project_id = project_id
        self.chat_model = ChatModel.from_pretrained("chat-bison@002")
        self.gemini_model = GenerativeModel("gemini-1.0-pro")
        self.translate_client = translate.Client()
        self.db = firestore.Client()

    def start_conversation(self, user_id, language="en"):
        """Start new conversation"""

        conversation = {
            "user_id": user_id,
            "language": language,
            "messages": [],
            "sentiment_scores": [],
            "escalated": False
        }

        doc_ref = self.db.collection("conversations").document()
        doc_ref.set(conversation)

        return doc_ref.id

    def send_message(self, conversation_id, message):
        """Process user message"""

        # Get conversation
        doc_ref = self.db.collection("conversations").document(conversation_id)
        conversation = doc_ref.get().to_dict()

        # Detect language if needed
        if conversation["language"] == "auto":
            detection = self.translate_client.detect_language(message)
            conversation["language"] = detection["language"]

        # Translate to English for processing
        if conversation["language"] != "en":
            translated = self.translate_client.translate(
                message,
                target_language="en"
            )
            message_en = translated["translatedText"]
        else:
            message_en = message

        # Analyze sentiment
        sentiment_score = self._analyze_sentiment(message_en)
        conversation["sentiment_scores"].append(sentiment_score)

        # Check if escalation needed
        if sentiment_score < -0.5 or "speak to manager" in message_en.lower():
            conversation["escalated"] = True
            response = "I understand you're frustrated. Let me connect you with a human agent."

        else:
            # Generate response
            response = self._generate_response(message_en, conversation["messages"])

        # Translate response back
        if conversation["language"] != "en":
            response = self.translate_client.translate(
                response,
                target_language=conversation["language"]
            )["translatedText"]

        # Update conversation
        conversation["messages"].append({
            "user": message,
            "bot": response,
            "timestamp": firestore.SERVER_TIMESTAMP
        })

        doc_ref.set(conversation)

        return {
            "response": response,
            "escalated": conversation["escalated"],
            "sentiment": sentiment_score
        }

    def _generate_response(self, message, history):
        """Generate contextual response"""

        # Build conversation context
        context = "You are a helpful customer support agent. Be empathetic and professional."
        if history:
            context += "\n\nPrevious conversation:\n"
            for msg in history[-3:]:  # Last 3 exchanges
                context += f"User: {msg['user']}\nAgent: {msg['bot']}\n"

        chat = self.chat_model.start_chat(context=context)
        response = chat.send_message(message)

        return response.text

    def _analyze_sentiment(self, text):
        """Analyze message sentiment"""

        prompt = f"""Analyze the sentiment of this message.
Return a score from -1 (very negative) to +1 (very positive).

Message: {text}

Score (number only):"""

        response = self.gemini_model.generate_content(
            prompt,
            generation_config={"temperature": 0.1, "max_output_tokens": 10}
        )

        try:
            return float(response.text.strip())
        except:
            return 0.0

# Usage
bot = MultiLingualSupportBot("my-project")

# Start conversation
conv_id = bot.start_conversation("user123", language="es")

# Send messages
response1 = bot.send_message(conv_id, "Hola, tengo un problema con mi pedido")
print(f"Bot: {response1['response']}")
print(f"Sentiment: {response1['sentiment']}")

response2 = bot.send_message(conv_id, "El producto no funciona correctamente")
print(f"Bot: {response2['response']}")
```

#### Scenario 4: Content Moderation System

```python
"""
Scenario: Build an AI-powered content moderation system
Requirements:
- Detect harmful content (hate speech, violence, etc.)
- Multi-modal (text and images)
- Provide confidence scores
- Support custom policies
- Handle high throughput (10K+ requests/sec)
"""

from vertexai.generative_models import GenerativeModel
from vertexai.vision_models import ImageTextModel
import vertexai.preview.generative_models as generative_models

class ContentModerationSystem:
    def __init__(self):
        self.text_model = GenerativeModel("gemini-1.0-pro")
        self.vision_model = GenerativeModel("gemini-1.0-pro-vision")

    def moderate_text(self, text, custom_policies=None):
        """Moderate text content"""

        # Use built-in safety settings
        safety_settings = {
            generative_models.HarmCategory.HARM_CATEGORY_HATE_SPEECH: generative_models.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
            generative_models.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: generative_models.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
            generative_models.HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT: generative_models.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
            generative_models.HarmCategory.HARM_CATEGORY_HARASSMENT: generative_models.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
        }

        response = self.text_model.generate_content(
            text,
            safety_settings=safety_settings
        )

        # Extract safety ratings
        safety_ratings = {}
        for rating in response.candidates[0].safety_ratings:
            safety_ratings[rating.category.name] = {
                "probability": rating.probability.name,
                "blocked": rating.blocked
            }

        # Custom policy check
        policy_violations = []
        if custom_policies:
            policy_violations = self._check_custom_policies(text, custom_policies)

        return {
            "approved": not response.candidates[0].finish_reason.name.startswith("SAFETY"),
            "safety_ratings": safety_ratings,
            "policy_violations": policy_violations,
            "confidence": self._calculate_confidence(safety_ratings)
        }

    def moderate_image(self, image_uri, text_overlay=None):
        """Moderate image content"""

        prompt = """Analyze this image for inappropriate content.
Check for:
1. Nudity or sexual content
2. Violence or gore
3. Hate symbols
4. Illegal activities

Provide a safety rating (SAFE/CAUTION/UNSAFE) and explanation:"""

        parts = [prompt, Part.from_uri(image_uri, mime_type="image/jpeg")]
        if text_overlay:
            parts.append(f"\nText overlay: {text_overlay}")

        response = self.vision_model.generate_content(parts)

        # Parse response
        result_text = response.text.lower()
        if "unsafe" in result_text:
            rating = "UNSAFE"
        elif "caution" in result_text:
            rating = "CAUTION"
        else:
            rating = "SAFE"

        return {
            "approved": rating == "SAFE",
            "rating": rating,
            "explanation": response.text,
            "safety_ratings": {
                rating.category.name: rating.probability.name
                for rating in response.candidates[0].safety_ratings
            }
        }

    def _check_custom_policies(self, text, policies):
        """Check against custom company policies"""

        violations = []

        for policy in policies:
            prompt = f"""Check if this text violates the following policy:

Policy: {policy['description']}
Examples of violations: {policy.get('examples', [])}

Text: {text}

Does this violate the policy? (YES/NO):
Explanation:"""

            response = self.text_model.generate_content(
                prompt,
                generation_config={"temperature": 0.1}
            )

            if response.text.strip().upper().startswith("YES"):
                violations.append({
                    "policy": policy["name"],
                    "explanation": response.text
                })

        return violations

    def _calculate_confidence(self, safety_ratings):
        """Calculate overall confidence score"""

        # Simplified confidence calculation
        high_risk_count = sum(
            1 for rating in safety_ratings.values()
            if rating["probability"] in ["HIGH", "MEDIUM"]
        )

        if high_risk_count == 0:
            return 0.95
        elif high_risk_count == 1:
            return 0.75
        else:
            return 0.50

# Usage
moderator = ContentModerationSystem()

# Moderate text
result = moderator.moderate_text(
    "This is a sample message to moderate",
    custom_policies=[
        {
            "name": "no_competitor_mentions",
            "description": "Do not mention competitor products",
            "examples": ["Check out AcmeCorp instead"]
        }
    ]
)

print(f"Approved: {result['approved']}")
print(f"Confidence: {result['confidence']}")
print(f"Safety ratings: {result['safety_ratings']}")
```

## AutoML Services

### AutoML Tables
**Automated machine learning for structured data**

#### Capabilities
- **Classification**: Predict categories
- **Regression**: Predict numerical values
- **Feature engineering**: Automatic feature selection
- **Model selection**: Algorithm selection and tuning

```python
# Create AutoML tabular dataset
dataset = aiplatform.TabularDataset.create(
    display_name='my-tabular-dataset',
    gcs_source='gs://my-bucket/data.csv'
)

# Train AutoML model
job = aiplatform.AutoMLTabularTrainingJob(
    display_name='automl-tabular-training',
    optimization_prediction_type='classification',
    optimization_objective='minimize-log-loss'
)

model = job.run(
    dataset=dataset,
    target_column='target',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

### AutoML Vision
**Automated computer vision models**

#### Model Types
- **Image classification**: Categorize images
- **Object detection**: Detect and locate objects
- **Image segmentation**: Pixel-level image understanding

```python
# Create image dataset
dataset = aiplatform.ImageDataset.create(
    display_name='my-image-dataset',
    gcs_source='gs://my-bucket/images.csv'
)

# Train AutoML image classification model
job = aiplatform.AutoMLImageTrainingJob(
    display_name='automl-image-training',
    prediction_type='classification'
)

model = job.run(
    dataset=dataset,
    model_display_name='my-image-model',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

### AutoML Natural Language
**Automated NLP models**

#### Capabilities
- **Text classification**: Categorize text documents
- **Entity extraction**: Identify and extract entities
- **Sentiment analysis**: Analyze text sentiment

```python
# Create text dataset
dataset = aiplatform.TextDataset.create(
    display_name='my-text-dataset',
    gcs_source='gs://my-bucket/text-data.jsonl'
)

# Train AutoML text classification model
job = aiplatform.AutoMLTextTrainingJob(
    display_name='automl-text-training',
    prediction_type='classification'
)

model = job.run(
    dataset=dataset,
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1
)
```

## MLOps with Vertex AI

### Experiment Tracking
```python
from google.cloud import aiplatform

# Initialize experiment
aiplatform.init(experiment='my-experiment')

# Start experiment run
with aiplatform.start_run('run-1') as run:
    # Log parameters
    run.log_params({
        'learning_rate': 0.01,
        'batch_size': 32,
        'epochs': 100
    })
    
    # Log metrics
    run.log_metrics({
        'accuracy': 0.95,
        'loss': 0.05
    })
    
    # Log artifacts
    run.log_artifact('model.pkl')
```

### Model Monitoring
```python
# Enable model monitoring
monitoring_job = aiplatform.ModelDeploymentMonitoringJob.create(
    display_name='model-monitoring',
    endpoint=endpoint,
    prediction_sampling_rate=0.1,
    monitoring_alert_config=aiplatform.monitoring.EmailAlertConfig(
        user_emails=['admin@company.com']
    ),
    monitoring_objective_configs=[
        aiplatform.monitoring.ObjectiveConfig(
            feature_name='feature1',
            feature_drift_detection_config=aiplatform.monitoring.DriftDetectionConfig(
                drift_threshold=0.05
            )
        )
    ]
)
```

### Feature Store
**Centralized feature management**

```python
# Create feature store
feature_store = aiplatform.Featurestore.create(
    featurestore_id='my-feature-store',
    location='us-central1'
)

# Create entity type
entity_type = feature_store.create_entity_type(
    entity_type_id='user',
    description='User entities'
)

# Create features
feature = entity_type.create_feature(
    feature_id='age',
    value_type='INT64',
    description='User age'
)

# Ingest feature values
entity_type.ingest_from_gcs(
    feature_ids=['age', 'income'],
    gcs_source_uris=['gs://my-bucket/features.csv'],
    entity_id_field='user_id',
    feature_time_field='timestamp'
)
```

## Data Integration

### BigQuery Integration
```python
# Train model with BigQuery data
dataset = aiplatform.TabularDataset.create_from_bigquery(
    display_name='bq-dataset',
    bq_source='bq://my-project.my_dataset.my_table'
)

# Export predictions to BigQuery
batch_job = model.batch_predict(
    job_display_name='batch-prediction',
    bigquery_source='bq://my-project.my_dataset.input_table',
    bigquery_destination_prefix='my-project.my_dataset.predictions'
)
```

### Data Labeling
```python
# Create data labeling job
labeling_job = aiplatform.DataLabelingJob.create(
    display_name='image-labeling',
    datasets=[image_dataset],
    instruction_uri='gs://my-bucket/labeling-instructions.pdf',
    inputs_schema_uri=aiplatform.schema.dataset.ioformat.image.single_label_classification,
    annotation_schema_uri=aiplatform.schema.dataset.annotation.classification
)
```

## Performance Optimization

### Compute Resources
```python
# Specify machine types and accelerators
job = aiplatform.CustomTrainingJob(
    display_name='optimized-training',
    script_path='train.py',
    container_uri='gcr.io/cloud-aiplatform/training/pytorch-gpu.1-9:latest'
)

model = job.run(
    replica_count=4,  # Multi-node training
    machine_type='n1-highmem-16',  # High-memory machine
    accelerator_type='NVIDIA_TESLA_V100',  # GPU acceleration
    accelerator_count=4  # Multiple GPUs
)
```

### Model Optimization
```python
# Hyperparameter tuning
tuning_job = aiplatform.HyperparameterTuningJob(
    display_name='hyperparameter-tuning',
    custom_job=custom_job,
    metric_spec={'accuracy': 'maximize'},
    parameter_spec={
        'learning_rate': hpt.DoubleParameterSpec(min=0.001, max=0.1, scale='log'),
        'batch_size': hpt.IntegerParameterSpec(min=16, max=128, scale='linear')
    },
    max_trial_count=20,
    parallel_trial_count=4
)
```

## Security and Compliance

### IAM and Security
```python
# Service account for Vertex AI
from google.oauth2 import service_account

credentials = service_account.Credentials.from_service_account_file(
    'path/to/service-account-key.json'
)

aiplatform.init(
    project='my-project',
    location='us-central1',
    credentials=credentials
)
```

### VPC and Network Security
```python
# Deploy with VPC configuration
endpoint = model.deploy(
    machine_type='n1-standard-2',
    network='projects/my-project/global/networks/my-vpc',
    encryption_spec_key_name='projects/my-project/locations/us-central1/keyRings/my-ring/cryptoKeys/my-key'
)
```

#### Scenario 5: Medical Diagnosis Assistant with RAG

```python
"""
Scenario: Build an AI assistant for medical information retrieval
Requirements:
- Access medical literature and guidelines
- Provide evidence-based answers with citations
- Comply with healthcare regulations
- Maintain audit trail
- Support medical terminology
"""

class MedicalDiagnosisAssistant:
    def __init__(self, project_id):
        self.project_id = project_id
        self.embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")
        self.model = GenerativeModel("gemini-1.0-pro")

        # Load medical knowledge base
        self.medical_kb = self._load_medical_knowledge()

    def query_medical_information(self, query, patient_context=None):
        """Query medical information with context"""

        # Retrieve relevant medical literature
        relevant_docs = self._retrieve_medical_docs(query, top_k=10)

        # Build context
        context = "Medical Literature:\n\n"
        for i, doc in enumerate(relevant_docs):
            context += f"[{i+1}] {doc['title']}\n{doc['content']}\n\n"

        if patient_context:
            context += f"\nPatient Context:\n{patient_context}\n\n"

        # Generate evidence-based response
        prompt = f"""You are a medical information assistant. Provide evidence-based information based on the medical literature provided.

IMPORTANT: Always cite sources and include disclaimers about consulting healthcare professionals.

{context}

Question: {query}

Response (with citations):"""

        response = self.model.generate_content(
            prompt,
            generation_config={
                "temperature": 0.1,  # Low temperature for medical accuracy
                "max_output_tokens": 2048
            }
        )

        # Log for audit trail
        self._log_query({
            "query": query,
            "response": response.text,
            "sources": [doc['source'] for doc in relevant_docs],
            "timestamp": datetime.now().isoformat()
        })

        return {
            "response": response.text,
            "sources": relevant_docs,
            "disclaimer": "This information is for educational purposes only. Please consult with a healthcare professional."
        }
```

#### Scenario 6: Financial Report Analysis and Summarization

```python
"""
Scenario: Build an AI system for financial report analysis
Requirements:
- Extract key metrics from financial documents
- Generate executive summaries
- Compare year-over-year performance
- Detect anomalies
- Support regulatory compliance
"""

class FinancialReportAnalyzer:
    def __init__(self):
        self.vision_model = GenerativeModel("gemini-1.0-pro-vision")
        self.text_model = GenerativeModel("gemini-1.0-pro")

    def analyze_financial_report(self, report_uri):
        """Analyze financial report PDF"""

        # Extract data from report
        prompt = """Analyze this financial report and extract:
1. Revenue and profit figures
2. Key performance indicators
3. Year-over-year changes
4. Notable trends
5. Risk factors mentioned

Provide structured output as JSON."""

        report_part = Part.from_uri(report_uri, mime_type="application/pdf")

        response = self.vision_model.generate_content(
            [prompt, report_part],
            generation_config={
                "temperature": 0.1,
                "response_mime_type": "application/json"
            }
        )

        financial_data = json.loads(response.text)

        # Generate executive summary
        summary = self._generate_executive_summary(financial_data)

        # Detect anomalies
        anomalies = self._detect_anomalies(financial_data)

        return {
            "data": financial_data,
            "summary": summary,
            "anomalies": anomalies
        }

    def _generate_executive_summary(self, financial_data):
        """Generate executive summary"""

        prompt = f"""Based on this financial data, generate a concise executive summary (3-5 paragraphs):

{json.dumps(financial_data, indent=2)}

Focus on:
- Overall financial health
- Key achievements
- Areas of concern
- Future outlook

Executive Summary:"""

        response = self.text_model.generate_content(
            prompt,
            generation_config={"temperature": 0.3}
        )

        return response.text

    def compare_reports(self, current_report_uri, previous_report_uri):
        """Compare two financial reports"""

        # Analyze both reports
        current_data = self.analyze_financial_report(current_report_uri)
        previous_data = self.analyze_financial_report(previous_report_uri)

        # Generate comparison
        prompt = f"""Compare these two financial reports and provide analysis:

Current Period:
{json.dumps(current_data['data'], indent=2)}

Previous Period:
{json.dumps(previous_data['data'], indent=2)}

Provide:
1. Key changes and trends
2. Performance improvements/declines
3. Strategic insights
4. Recommendations

Analysis:"""

        response = self.text_model.generate_content(prompt)

        return {
            "current": current_data,
            "previous": previous_data,
            "comparison": response.text
        }
```

#### Scenario 7: Real-Time Translation and Transcription Service

```python
"""
Scenario: Build a real-time translation service for meetings
Requirements:
- Real-time speech transcription
- Multi-language translation
- Speaker identification
- Meeting summarization
- Action item extraction
"""

from google.cloud import speech_v1
from google.cloud import translate_v2

class RealTimeTranslationService:
    def __init__(self):
        self.speech_client = speech_v1.SpeechClient()
        self.translate_client = translate_v2.Client()
        self.summarization_model = GenerativeModel("gemini-1.0-pro")

    def transcribe_and_translate(self, audio_uri, target_languages):
        """Transcribe audio and translate to multiple languages"""

        # Transcribe with speaker diarization
        config = speech_v1.RecognitionConfig(
            encoding=speech_v1.RecognitionConfig.AudioEncoding.LINEAR16,
            sample_rate_hertz=16000,
            language_code="en-US",
            enable_speaker_diarization=True,
            diarization_speaker_count=5,
            enable_automatic_punctuation=True,
        )

        audio = speech_v1.RecognitionAudio(uri=audio_uri)
        operation = self.speech_client.long_running_recognize(
            config=config,
            audio=audio
        )

        response = operation.result()

        # Extract transcript with speakers
        transcript = []
        for result in response.results:
            alternative = result.alternatives[0]
            for word_info in alternative.words:
                transcript.append({
                    "speaker": word_info.speaker_tag,
                    "word": word_info.word,
                    "start_time": word_info.start_time.total_seconds(),
                    "end_time": word_info.end_time.total_seconds()
                })

        # Translate to target languages
        full_transcript = " ".join([w["word"] for w in transcript])
        translations = {}

        for language in target_languages:
            result = self.translate_client.translate(
                full_transcript,
                target_language=language
            )
            translations[language] = result["translatedText"]

        # Generate meeting summary
        summary = self._generate_meeting_summary(full_transcript)

        # Extract action items
        action_items = self._extract_action_items(full_transcript)

        return {
            "transcript": transcript,
            "full_text": full_transcript,
            "translations": translations,
            "summary": summary,
            "action_items": action_items
        }

    def _generate_meeting_summary(self, transcript):
        """Generate meeting summary"""

        prompt = f"""Summarize this meeting transcript. Include:
1. Main topics discussed
2. Key decisions made
3. Open questions
4. Next steps

Transcript:
{transcript}

Summary:"""

        response = self.summarization_model.generate_content(
            prompt,
            generation_config={"temperature": 0.2}
        )

        return response.text

    def _extract_action_items(self, transcript):
        """Extract action items from transcript"""

        prompt = f"""Extract action items from this meeting transcript.
For each action item, provide:
- Task description
- Assigned to (if mentioned)
- Due date (if mentioned)
- Priority

Format as JSON array.

Transcript:
{transcript}

Action Items (JSON):"""

        response = self.summarization_model.generate_content(
            prompt,
            generation_config={
                "temperature": 0.1,
                "response_mime_type": "application/json"
            }
        )

        return json.loads(response.text)
```

#### Scenario 8: E-commerce Product Recommendation Engine

```python
"""
Scenario: Build a personalized product recommendation engine
Requirements:
- Analyze user behavior and preferences
- Generate personalized recommendations
- Create product descriptions
- Handle multi-modal product data (text, images)
- Real-time recommendations
"""

class ProductRecommendationEngine:
    def __init__(self):
        self.embedding_model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")
        self.generation_model = GenerativeModel("gemini-1.0-pro")
        self.vision_model = GenerativeModel("gemini-1.0-pro-vision")

    def generate_recommendations(self, user_id, context=None):
        """Generate personalized product recommendations"""

        # Get user profile and history
        user_profile = self._get_user_profile(user_id)
        purchase_history = self._get_purchase_history(user_id)
        browsing_history = self._get_browsing_history(user_id)

        # Generate user embedding
        user_description = f"""
User preferences: {user_profile['preferences']}
Recent purchases: {', '.join([p['name'] for p in purchase_history[:5]])}
Recent views: {', '.join([b['name'] for b in browsing_history[:10]])}
"""

        user_embedding = self.embedding_model.get_embeddings([user_description])[0].values

        # Find similar products
        candidate_products = self._find_similar_products(user_embedding, top_k=50)

        # Rank and personalize
        prompt = f"""Given this user profile, rank and select the top 10 product recommendations.

User Profile:
{user_description}

Candidate Products:
{json.dumps([p['name'] + ': ' + p['description'] for p in candidate_products], indent=2)}

Context: {context if context else 'General browsing'}

Provide:
1. Top 10 recommended products (with reasoning)
2. Personalized message for each recommendation

Format as JSON:"""

        response = self.generation_model.generate_content(
            prompt,
            generation_config={
                "temperature": 0.5,
                "response_mime_type": "application/json"
            }
        )

        recommendations = json.loads(response.text)

        return recommendations

    def generate_product_description(self, product_image_uri, product_data):
        """Generate compelling product description"""

        image_part = Part.from_uri(product_image_uri, mime_type="image/jpeg")

        prompt = f"""Create a compelling product description for e-commerce.

Product Data:
{json.dumps(product_data, indent=2)}

Generate:
1. Short title (60 characters)
2. Brief description (150 characters)
3. Full description (300 words)
4. Key features (bullet points)
5. SEO keywords

Analyze the product image and incorporate visual details.

Response (JSON):"""

        response = self.vision_model.generate_content(
            [prompt, image_part],
            generation_config={
                "temperature": 0.7,
                "response_mime_type": "application/json"
            }
        )

        return json.loads(response.text)

    def analyze_product_reviews(self, reviews):
        """Analyze product reviews for insights"""

        reviews_text = "\n\n".join([
            f"Rating: {r['rating']}/5\nReview: {r['text']}"
            for r in reviews
        ])

        prompt = f"""Analyze these product reviews and provide:
1. Overall sentiment
2. Common positive points
3. Common negative points
4. Suggestions for product improvement
5. Key themes

Reviews:
{reviews_text}

Analysis (JSON):"""

        response = self.generation_model.generate_content(
            prompt,
            generation_config={
                "temperature": 0.3,
                "response_mime_type": "application/json"
            }
        )

        return json.loads(response.text)
```

## Cost Optimization for Gen AI

### Pricing Models

**PaLM 2 Models Pricing**
| Model | Input (per 1K chars) | Output (per 1K chars) |
|-------|---------------------|----------------------|
| text-bison@001 | $0.0005 | $0.0005 |
| text-bison@002 | $0.0005 | $0.0005 |
| chat-bison@001 | $0.0005 | $0.0005 |
| chat-bison@002 | $0.0005 | $0.0005 |
| code-bison@001 | $0.0005 | $0.0005 |
| textembedding-gecko | $0.0001 | - |

**Gemini Models Pricing**
| Model | Input (per 1K chars) | Output (per 1K chars) |
|-------|---------------------|----------------------|
| gemini-1.0-pro | $0.00025 | $0.0005 |
| gemini-1.0-pro-vision | $0.00025 | $0.0005 |

### Cost Optimization Strategies

```python
# 1. Use prompt caching for repeated context
class PromptCache:
    """Cache common prompt prefixes"""

    def __init__(self):
        self.cache = {}

    def get_cached_prompt(self, context_key, user_query):
        """Reuse common context"""
        if context_key in self.cache:
            return self.cache[context_key] + user_query
        return user_query

# 2. Optimize token usage
def optimize_prompt(long_prompt):
    """Reduce unnecessary tokens"""

    # Remove extra whitespace
    optimized = " ".join(long_prompt.split())

    # Use abbreviations where appropriate
    # Summarize long examples

    return optimized

# 3. Batch requests
def batch_predictions(prompts, model):
    """Batch multiple predictions"""

    # Combine prompts
    combined_prompt = "\n\n---\n\n".join([
        f"Query {i+1}: {prompt}"
        for i, prompt in enumerate(prompts)
    ])

    # Single API call
    response = model.predict(combined_prompt)

    # Parse responses
    return response.text.split("---")

# 4. Use cheaper models when appropriate
def select_model_by_complexity(task):
    """Choose cost-effective model"""

    if task["requires_reasoning"]:
        return GenerativeModel("gemini-1.0-pro")
    elif task["requires_code"]:
        return CodeGenerationModel.from_pretrained("code-bison@002")
    else:
        return TextGenerationModel.from_pretrained("text-bison@001")

# 5. Implement response caching
from functools import lru_cache
import hashlib

@lru_cache(maxsize=1000)
def get_cached_response(prompt_hash):
    """Cache frequent queries"""
    # Return cached response if available
    pass

def query_with_cache(prompt, model):
    prompt_hash = hashlib.md5(prompt.encode()).hexdigest()

    cached = get_cached_response(prompt_hash)
    if cached:
        return cached

    response = model.predict(prompt)
    get_cached_response.cache_info()  # Monitor cache hits

    return response

# 6. Tune models to reduce prompt size
def compare_costs(base_model, tuned_model, num_requests):
    """Compare costs: prompt engineering vs tuning"""

    # Prompt engineering approach
    long_prompt_cost = (1000 * 0.0005) * num_requests

    # Tuned model approach
    tuning_cost = 100  # One-time tuning cost
    short_prompt_cost = (100 * 0.0005) * num_requests

    print(f"Prompt engineering: ${long_prompt_cost:.2f}")
    print(f"Tuned model: ${tuning_cost + short_prompt_cost:.2f}")

    return tuning_cost + short_prompt_cost < long_prompt_cost

# 7. Monitor and optimize
def track_api_costs():
    """Monitor API usage and costs"""

    # Track per-request costs
    costs = {
        "total_input_chars": 0,
        "total_output_chars": 0,
        "total_requests": 0
    }

    # Calculate estimated cost
    input_cost = (costs["total_input_chars"] / 1000) * 0.0005
    output_cost = (costs["total_output_chars"] / 1000) * 0.0005

    return input_cost + output_cost
```

## Exam Tips: Vertex AI Gen AI

### Key Exam Topics

**1. Model Selection**
- Choose appropriate model for use case (text-bison vs Gemini vs code-bison)
- Understand context window limits
- Know when to use embeddings vs generation
- Multimodal use cases with Gemini Pro Vision

**2. Prompt Engineering**
- Few-shot learning with examples
- System instructions and context
- Temperature and sampling parameters
- Stop sequences and constraints

**3. Fine-Tuning**
- When to tune vs prompt engineering
- Training data requirements (100+ examples)
- Adapter tuning vs full fine-tuning
- Evaluation and validation

**4. Grounding and RAG**
- Google Search grounding
- Vertex AI Search integration
- Custom vector search with Matching Engine
- Document chunking strategies

**5. Function Calling**
- Function declaration format
- Multi-turn function calling
- Structured outputs
- API integration patterns

**6. Enterprise Features**
- Private endpoints and VPC
- CMEK encryption
- Data residency
- IAM roles and permissions
- Audit logging

**7. Cost Optimization**
- Model pricing differences
- Prompt optimization
- Caching strategies
- When to use tuned models

### Common Exam Scenarios

**Scenario 1: Choose the Right Model**
- **Question**: "You need to analyze images and generate descriptions. Which model?"
- **Answer**: Gemini Pro Vision - multimodal capabilities
- **Why Not**: text-bison (text only), code-bison (code focused)

**Scenario 2: Reduce Hallucinations**
- **Question**: "How to reduce hallucinations in generated responses?"
- **Answer**: Use grounding with Google Search or Vertex AI Search
- **Additional**: Lower temperature (0.1-0.3), provide context, use RAG

**Scenario 3: Fine-Tuning Decision**
- **Question**: "When should you fine-tune vs use prompt engineering?"
- **Answer**: Fine-tune for:
  - Domain-specific terminology
  - Consistent output format
  - Reducing prompt length for cost
  - Task-specific performance
- **Prompt Engineering for**:
  - Few examples available
  - Changing requirements
  - Quick experimentation

**Scenario 4: Enterprise Security**
- **Question**: "Deploy Gen AI model with data residency in EU"
- **Answer**:
  ```python
  vertexai.init(project="my-project", location="europe-west4")
  tuning_job = model.tune_model(
      training_data="gs://eu-bucket/data.jsonl",
      tuning_job_location="europe-west4",
      tuned_model_location="europe-west4"
  )
  ```

**Scenario 5: RAG Implementation**
- **Question**: "Build Q&A system for company documents"
- **Answer**:
  1. Use textembedding-gecko for embeddings
  2. Store in Matching Engine
  3. Retrieve top-k relevant chunks
  4. Use Gemini Pro for generation with retrieved context
  5. Implement citation tracking

**Scenario 6: Function Calling**
- **Question**: "Enable model to query database and send emails"
- **Answer**: Use Gemini Pro with function declarations
  - Define functions with clear descriptions
  - Handle multi-turn execution
  - Return results to model for natural language response

**Scenario 7: Cost Optimization**
- **Question**: "Reduce costs for high-volume generation workload"
- **Answer**:
  - Use text-bison@001 (cheaper) if quality sufficient
  - Implement response caching
  - Batch requests where possible
  - Optimize prompts to reduce tokens
  - Consider tuning to shorten prompts

### Decision Trees for Exam

**Model Selection Decision Tree**
```
Need to process images/video?
  YES → Gemini Pro Vision
  NO → Continue

Need code generation/completion?
  YES → code-bison or codechat-bison
  NO → Continue

Need multi-turn conversation?
  YES → chat-bison or Gemini Pro
  NO → Continue

Need long context (>8K tokens)?
  YES → Gemini Pro (32K tokens)
  NO → text-bison (8K tokens)

Need function calling?
  YES → Gemini Pro
  NO → text-bison or Gemini Pro
```

**Grounding Decision Tree**
```
Need current information (news, events)?
  YES → Google Search grounding
  NO → Continue

Have custom knowledge base?
  YES → Continue
    Structured data (FAQ, policies)?
      YES → Vertex AI Search
      NO → Continue
    Unstructured documents?
      YES → RAG with Matching Engine
      NO → Prompt engineering
  NO → Use base model with prompts
```

**Enterprise Deployment Decision Tree**
```
Data residency requirements?
  YES → Deploy in specific region
  NO → Continue

Private network access only?
  YES → Private endpoints with VPC
  NO → Continue

Need encryption at rest?
  YES → CMEK with Cloud KMS
  NO → Continue

Compliance requirements (HIPAA, SOC 2)?
  YES → VPC Service Controls + audit logging
  NO → Standard deployment
```

### Exam Question Patterns

**Pattern 1: "Which model should you use..."**
- Look for: multimodal, code, conversation, context length clues
- Answer: Match capabilities to requirements

**Pattern 2: "How do you reduce hallucinations..."**
- Look for: factual accuracy, citations needed
- Answer: Grounding (Search or RAG)

**Pattern 3: "What's the most cost-effective..."**
- Look for: high volume, repeated patterns
- Answer: Tuning, caching, cheaper models, optimization

**Pattern 4: "How do you ensure data stays in EU..."**
- Look for: compliance, data residency
- Answer: Region selection, CMEK, VPC SC

**Pattern 5: "Best way to handle..."**
- Look for: specific use case description
- Answer: Match pattern to solution (RAG, function calling, etc.)

### Quick Reference Tables

**Model Capabilities Comparison**
| Feature | text-bison | chat-bison | code-bison | Gemini Pro | Gemini Pro Vision |
|---------|-----------|-----------|-----------|-----------|------------------|
| Text generation | Yes | Yes | Code | Yes | Yes |
| Multi-turn chat | No | Yes | Yes | Yes | Yes |
| Image understanding | No | No | No | No | Yes |
| Function calling | No | No | No | Yes | Yes |
| Context window | 8K | 8K | 6K | 32K | 32K |
| JSON mode | No | No | No | Yes | Yes |

**Common IAM Roles**
| Role | Permissions | Use Case |
|------|------------|----------|
| aiplatform.user | Use endpoints, models | Application access |
| aiplatform.admin | Full admin | Platform management |
| aiplatform.modelUser | Predict only | Production inference |
| aiplatform.viewer | Read-only | Monitoring, auditing |

### Key Formulas and Limits

**Token Limits**
- text-bison: 8,192 input + 2,048 output
- chat-bison: 8,192 total context
- code-bison: 6,144 input + 1,024 output
- Gemini Pro: 32,768 input + 8,192 output

**Cost Calculation**
```
Cost = (input_chars / 1000) * input_price + (output_chars / 1000) * output_price

Example:
- 5,000 input chars
- 2,000 output chars
- Gemini Pro pricing

Cost = (5,000 / 1,000) * $0.00025 + (2,000 / 1,000) * $0.0005
     = $0.00125 + $0.001
     = $0.00225 per request
```

**Tuning Data Requirements**
- Minimum: 100 examples (recommended: 500+)
- Quality > Quantity
- Balanced dataset
- Representative of production use

## Best Practices

### Development Workflow
1. **Start with AutoML**: Begin with automated solutions
2. **Experiment tracking**: Use Vertex AI experiments
3. **Version control**: Track model and data versions
4. **Gradual complexity**: Start simple, add complexity as needed

### Gen AI Development Best Practices

**1. Prompt Engineering First**
- Test with few-shot examples before tuning
- Iterate on prompts using Generative AI Studio
- Document successful prompt patterns
- Version control your prompts

**2. Evaluation and Testing**
- Create test datasets with ground truth
- Use model-based evaluation for quality
- Monitor safety scores
- A/B test different approaches

**3. Production Readiness**
- Implement rate limiting
- Add response caching
- Monitor costs and quotas
- Plan for fallbacks

**4. Security and Compliance**
- Use private endpoints for sensitive data
- Enable audit logging
- Implement data filtering
- Regular security reviews

### Production Deployment
1. **Model validation**: Thorough testing before deployment
2. **Monitoring**: Implement comprehensive monitoring
3. **A/B testing**: Compare model versions in production
4. **Rollback strategy**: Plan for model rollbacks

### Cost Optimization
1. **Right-size resources**: Match compute to workload requirements
2. **Use preemptible instances**: For fault-tolerant training
3. **Optimize serving**: Scale endpoints based on demand
4. **Storage management**: Archive old models and data
5. **Model selection**: Use cheaper models when sufficient
6. **Prompt optimization**: Reduce token usage
7. **Caching**: Implement response caching
8. **Batching**: Batch requests when possible

### Performance Tuning
1. **Data preprocessing**: Optimize data pipeline performance
2. **Model architecture**: Choose efficient model designs
3. **Hardware acceleration**: Use GPUs and TPUs appropriately
4. **Batch optimization**: Optimize batch sizes for training and inference
5. **Streaming**: Use streaming for better UX
6. **Parallel requests**: Parallelize independent operations

## Summary

Vertex AI provides a comprehensive Gen AI platform with:
- **Foundation Models**: PaLM 2, Gemini, embeddings
- **Tuning**: Supervised, adapter-based, RLHF
- **Grounding**: Search, RAG, custom sources
- **Enterprise**: Security, compliance, private deployment
- **Cost Effective**: Multiple pricing tiers, optimization options

**Key Takeaways for Exam**:
1. Know model capabilities and selection criteria
2. Understand RAG architecture and implementation
3. Master function calling and structured outputs
4. Know enterprise security features
5. Understand cost optimization strategies
6. Be familiar with common patterns and scenarios