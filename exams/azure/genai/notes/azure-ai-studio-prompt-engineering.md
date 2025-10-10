# Azure AI Studio and Prompt Engineering

## Table of Contents
1. [Azure AI Studio Overview](#azure-ai-studio-overview)
2. [Prompt Engineering Fundamentals](#prompt-engineering-fundamentals)
3. [Advanced Prompt Techniques](#advanced-prompt-techniques)
4. [Azure OpenAI Playground](#azure-openai-playground)
5. [Model Selection and Configuration](#model-selection-and-configuration)
6. [Prompt Flow](#prompt-flow)
7. [RAG (Retrieval Augmented Generation)](#rag-retrieval-augmented-generation)
8. [Exam Tips](#exam-tips)

---

## Azure AI Studio Overview

### What is Azure AI Studio?
Unified development platform for building AI applications.

**Key Capabilities:**
- Model catalog and deployment
- Prompt engineering and testing
- RAG implementation
- Evaluation and monitoring
- MLOps integration
- Content safety

### Components

#### 1. AI Hub
- Central resource for AI projects
- Shared resources and configuration
- Security and networking
- Connected services (OpenAI, Search, Storage)

#### 2. AI Projects
- Development workspace
- Model deployments
- Datasets and indexes
- Evaluation runs
- Deployment endpoints

#### 3. Model Catalog
- Azure OpenAI models
- Open-source models (Llama, Mistral, etc.)
- Fine-tuned models
- Model version management

#### 4. Prompt Flow
- Visual designer for LLM apps
- Testing and evaluation
- Deployment to endpoints
- Integration with code

---

## Prompt Engineering Fundamentals

### What is Prompt Engineering?
Art and science of designing inputs to get desired outputs from LLMs.

### Basic Prompt Structure

```
[System Message]
[Context/Background]
[Instruction]
[Input Data]
[Output Format]
[Examples (optional)]
```

### System Messages

**Purpose:** Set behavior and persona of AI assistant.

**Example:**
```
You are a helpful AI assistant specializing in Azure cloud services.
You provide accurate, concise, and actionable guidance.
When uncertain, you clearly state limitations and suggest verification.
```

**Best Practices:**
- Define role and expertise
- Set tone and style
- Specify constraints
- Include safety guidelines

### Zero-Shot Prompting

Direct instruction without examples.

**Example:**
```
Classify the sentiment of this review as positive, negative, or neutral:

"The product works well but shipping took too long."
```

**When to Use:**
- Simple, well-defined tasks
- Model has strong prior knowledge
- Quick prototyping

### Few-Shot Prompting

Provide examples to guide model.

**Example:**
```
Classify the sentiment as positive, negative, or neutral:

Review: "Amazing product! Exceeded expectations."
Sentiment: Positive

Review: "Terrible quality. Would not recommend."
Sentiment: Negative

Review: "It's okay, nothing special."
Sentiment: Neutral

Review: "Fast delivery but average features."
Sentiment:
```

**Benefits:**
- Better accuracy
- Consistent format
- Task demonstration
- Reduced ambiguity

### Chain-of-Thought (CoT)

Encourage step-by-step reasoning.

**Example:**
```
Solve this problem step by step:

A store sells apples for $2 each and oranges for $3 each.
If you buy 5 apples and 4 oranges, how much do you spend?

Let's think through this:
1. Cost of apples = 5 × $2 = $10
2. Cost of oranges = 4 × $3 = $12
3. Total cost = $10 + $12 = $22

Answer: $22
```

**Trigger Phrases:**
- "Let's think step by step"
- "Let's work through this systematically"
- "Let's break this down"

---

## Advanced Prompt Techniques

### Self-Consistency

Generate multiple responses and select most consistent.

**Implementation:**
```python
# Azure OpenAI SDK
responses = []
for i in range(5):
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7
    )
    responses.append(response.choices[0].message.content)

# Select most common response
from collections import Counter
final_answer = Counter(responses).most_common(1)[0][0]
```

### Tree of Thoughts

Explore multiple reasoning paths.

**Example:**
```
Problem: Plan a 3-day trip to Seattle

Approach 1 - Weather-focused:
- Day 1: Indoor activities (museums)
- Day 2: Check weather, Pike Place if clear
- Day 3: Backup indoor options

Approach 2 - Must-see focused:
- Day 1: Space Needle, Pike Place
- Day 2: Museums, waterfront
- Day 3: Day trip to nearby areas

Evaluate approaches and select best based on:
- Time efficiency
- Weather resilience
- Coverage of key attractions
```

### Prompt Chaining

Break complex tasks into sequential prompts.

**Example:**
```python
# Step 1: Extract key information
extract_prompt = "Extract the main entities from this text: {text}"
entities = get_completion(extract_prompt)

# Step 2: Analyze relationships
analyze_prompt = f"Analyze relationships between these entities: {entities}"
relationships = get_completion(analyze_prompt)

# Step 3: Generate summary
summary_prompt = f"Create a summary using: Entities: {entities}, Relationships: {relationships}"
final_summary = get_completion(summary_prompt)
```

### ReAct (Reasoning + Acting)

Combine reasoning with action/tool use.

**Example:**
```
Question: What's the weather in Seattle and should I bring an umbrella?

Thought: I need to check current weather in Seattle
Action: search_weather("Seattle")
Observation: Cloudy, 60°F, 70% chance of rain

Thought: High chance of rain means umbrella recommended
Answer: It's currently cloudy in Seattle with 60°F and 70% chance of rain.
Yes, you should bring an umbrella.
```

### Persona Prompting

Assign specific role or expertise.

**Examples:**
```
# Expert consultant
"You are a senior Azure architect with 15 years of experience.
Analyze this architecture for scalability issues."

# Teaching assistant
"You are a patient tutor helping a beginner learn Python.
Explain this concept using simple language and examples."

# Critical reviewer
"You are a thorough code reviewer focused on security.
Review this code for potential vulnerabilities."
```

### Temperature and Top_p Control

**Temperature (0.0 - 2.0):**
- 0.0: Deterministic, focused
- 0.7: Balanced creativity
- 1.0+: More random, creative

**Top_p (0.0 - 1.0):**
- Nucleus sampling
- 0.1: Very focused
- 0.9: More diverse

**Usage:**
```python
response = client.chat.completions.create(
    model="gpt-4",
    messages=messages,
    temperature=0.7,  # Moderate creativity
    top_p=0.95,       # Consider top 95% probability tokens
    max_tokens=500
)
```

**Guidelines:**
- Factual tasks: Low temperature (0.0-0.3)
- Creative tasks: Higher temperature (0.7-1.0)
- Code generation: Low temperature
- Brainstorming: Higher temperature

---

## Azure OpenAI Playground

### Playground Types

#### 1. Chat Playground
- Interactive chat interface
- System message configuration
- Few-shot examples
- Parameter tuning
- Save and share prompts

**Features:**
- Real-time testing
- History tracking
- Export to code
- Session management

#### 2. Completions Playground
- Single-turn completions
- Prompt testing
- Parameter exploration
- Compare models

#### 3. DALL-E Playground
- Image generation
- Prompt refinement
- Style control
- Iteration testing

### Playground Features

#### System Message
```
Configure the behavior of the AI assistant:

- Role: Technical support specialist
- Tone: Professional and helpful
- Rules: Always ask for error codes, provide step-by-step solutions
```

#### Parameters

**Temperature:** Creativity level
**Max Tokens:** Response length
**Top_p:** Diversity
**Frequency Penalty:** Reduce repetition
**Presence Penalty:** Encourage new topics
**Stop Sequences:** Custom stop tokens

#### Few-Shot Examples
```json
{
  "examples": [
    {
      "user": "How do I create a VM?",
      "assistant": "To create an Azure VM: 1. Go to Azure Portal..."
    },
    {
      "user": "What's the difference between VMs and App Service?",
      "assistant": "VMs provide IaaS with full control..."
    }
  ]
}
```

#### Export Options
- Python code
- C# code
- curl command
- JSON configuration

---

## Model Selection and Configuration

### Azure OpenAI Models

#### GPT-4 Family

**GPT-4 Turbo:**
- 128K context window
- Latest knowledge (Apr 2023)
- Vision capabilities
- Best reasoning
- Higher cost

**GPT-4:**
- 8K/32K context
- Strong reasoning
- Complex tasks
- Moderate cost

**When to Use:**
- Complex reasoning required
- Long context needed
- Highest quality output
- Multi-step tasks

#### GPT-3.5 Family

**GPT-3.5 Turbo:**
- 16K context window
- Fast responses
- Good for most tasks
- Cost-effective

**GPT-3.5 Turbo Instruct:**
- Completions format
- Legacy applications
- Fine-tuning available

**When to Use:**
- High-volume applications
- Simple to moderate tasks
- Cost optimization
- Fast response needed

#### Embeddings Models

**text-embedding-ada-002:**
- 1536 dimensions
- Semantic search
- Clustering
- Recommendations

**text-embedding-3-small/large:**
- Improved quality
- Variable dimensions
- Better multilingual

### Model Configuration

#### Context Window
```python
# GPT-4 Turbo: 128K tokens
# ~96,000 words or ~300 pages

# Monitor token usage
response = client.chat.completions.create(...)
print(f"Tokens used: {response.usage.total_tokens}")
```

#### Token Limits
- Prompt + Completion ≤ Max Context
- Plan token budget
- Truncate long inputs
- Chunk large documents

#### Rate Limits
```
Tokens per minute (TPM): 150,000
Requests per minute (RPM): 1,000
```

**Handle Rate Limits:**
```python
import time
from openai import RateLimitError

try:
    response = client.chat.completions.create(...)
except RateLimitError:
    time.sleep(60)  # Wait and retry
    response = client.chat.completions.create(...)
```

---

## Prompt Flow

### What is Prompt Flow?
Visual development tool for LLM applications.

**Components:**
- Flow designer (visual)
- Python code nodes
- LLM nodes
- Tool nodes
- Evaluation

### Flow Types

#### 1. Standard Flow
General LLM application workflows.

**Nodes:**
- LLM: Call language models
- Python: Custom logic
- Prompt: Template prompts
- Tool: External tools/APIs

#### 2. Chat Flow
Conversational AI applications.

**Features:**
- Chat history management
- Session state
- Multi-turn conversations
- Context preservation

#### 3. Evaluation Flow
Assess LLM outputs.

**Metrics:**
- Groundedness
- Relevance
- Coherence
- Fluency
- Custom metrics

### Creating a Flow

**Example: Customer Support Bot**
```yaml
# flow.dag.yaml
nodes:
  - name: classify_intent
    type: llm
    source:
      type: code
      path: classify.jinja2
    inputs:
      user_query: ${inputs.question}

  - name: retrieve_context
    type: python
    source:
      type: code
      path: retrieve.py
    inputs:
      query: ${inputs.question}

  - name: generate_response
    type: llm
    source:
      type: code
      path: response.jinja2
    inputs:
      context: ${retrieve_context.output}
      query: ${inputs.question}

outputs:
  answer: ${generate_response.output}
```

### Prompt Templates

**Jinja2 Syntax:**
```jinja2
system:
You are a helpful customer support agent for {{company}}.

user:
Customer question: {{query}}

Relevant information:
{% for doc in context %}
- {{doc.content}}
{% endfor %}

Provide a helpful response based on the information above.
```

### Evaluation

**Built-in Evaluators:**
```python
from promptflow.evals.evaluators import RelevanceEvaluator, GroundednessEvaluator

# Evaluate responses
relevance_eval = RelevanceEvaluator(model_config)
result = relevance_eval(
    question="How do I reset my password?",
    answer="Click on 'Forgot Password' link...",
    context="Password reset instructions..."
)

print(f"Relevance Score: {result['relevance']}")
```

### Deployment

```bash
# Deploy flow as endpoint
pf flow build --source . --output dist --format docker

# Deploy to Azure
az ml online-deployment create \
  --name my-deployment \
  --endpoint my-endpoint \
  --model-file flow/
```

---

## RAG (Retrieval Augmented Generation)

### RAG Architecture

```
User Query
    ↓
Embedding Model (convert to vector)
    ↓
Vector Search (retrieve relevant docs)
    ↓
Retrieved Documents
    ↓
Prompt Construction (query + context)
    ↓
LLM Generation
    ↓
Response
```

### Components

#### 1. Data Ingestion
```python
# Chunk documents
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

chunks = splitter.split_documents(documents)
```

#### 2. Embedding and Indexing
```python
# Create embeddings
from openai import AzureOpenAI

client = AzureOpenAI(...)

def create_embedding(text):
    response = client.embeddings.create(
        model="text-embedding-ada-002",
        input=text
    )
    return response.data[0].embedding

# Store in vector database
from azure.search.documents import SearchClient

search_client = SearchClient(...)
for chunk in chunks:
    embedding = create_embedding(chunk.page_content)
    doc = {
        "id": chunk.metadata['id'],
        "content": chunk.page_content,
        "embedding": embedding
    }
    search_client.upload_documents([doc])
```

#### 3. Retrieval
```python
# Query vector database
query_embedding = create_embedding(user_query)

results = search_client.search(
    search_text=None,
    vector_queries=[{
        "vector": query_embedding,
        "k": 5,  # Top 5 results
        "fields": "embedding"
    }]
)

context = "\n\n".join([r['content'] for r in results])
```

#### 4. Generation
```python
# Generate response with context
prompt = f"""
Answer the question based on the context below:

Context:
{context}

Question: {user_query}

Answer:
"""

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}]
)
```

### Azure AI Search Integration

**Vector Search:**
```python
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import *

# Define index with vector field
index = SearchIndex(
    name="documents",
    fields=[
        SimpleField(name="id", type="Edm.String", key=True),
        SearchableField(name="content", type="Edm.String"),
        SearchField(
            name="embedding",
            type="Collection(Edm.Single)",
            vector_search_dimensions=1536,
            vector_search_profile_name="default-profile"
        )
    ],
    vector_search=VectorSearch(
        profiles=[VectorSearchProfile(name="default-profile", algorithm_configuration_name="default-algorithm")],
        algorithms=[HnswAlgorithmConfiguration(name="default-algorithm")]
    )
)
```

**Hybrid Search:**
```python
# Combine keyword and vector search
results = search_client.search(
    search_text=user_query,  # Keyword search
    vector_queries=[{
        "vector": query_embedding,  # Vector search
        "k": 5,
        "fields": "embedding"
    }],
    select=["id", "content"],
    top=5
)
```

### RAG Optimization

**1. Chunk Size:**
- Too small: Lose context
- Too large: Irrelevant info
- Sweet spot: 500-1500 tokens

**2. Overlap:**
- Preserve context across chunks
- Typical: 10-20% overlap

**3. Retrieval Count:**
- More docs: More context, higher cost
- Typical: 3-5 documents

**4. Reranking:**
```python
# Rerank retrieved documents
from sentence_transformers import CrossEncoder

reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')

# Score each retrieved doc
scores = reranker.predict([(user_query, doc) for doc in retrieved_docs])

# Sort and select top docs
top_docs = [doc for _, doc in sorted(zip(scores, retrieved_docs), reverse=True)][:3]
```

---

## Exam Tips

### Key Concepts

**Prompt Engineering:**
- Zero-shot: Direct instruction
- Few-shot: Provide examples
- Chain-of-thought: Step-by-step reasoning
- Temperature: Controls randomness
- System message: Sets behavior

**Azure AI Studio:**
- Model catalog: Deploy models
- Playground: Test prompts
- Prompt Flow: Build LLM apps
- Evaluation: Assess quality
- RAG: Grounding with data

**Model Selection:**
- GPT-4: Complex reasoning, long context
- GPT-3.5: Cost-effective, most tasks
- GPT-4 Turbo: Latest, vision, 128K context
- Embeddings: Semantic search, RAG

**RAG Architecture:**
- Chunk documents
- Create embeddings
- Vector search
- Augment prompt
- Generate response

### Common Scenarios

**Improving Accuracy:**
1. Add few-shot examples
2. Lower temperature
3. More specific instructions
4. Include constraints
5. Use chain-of-thought

**Reducing Hallucinations:**
1. Implement RAG
2. Lower temperature
3. Request citations
4. Add grounding data
5. Evaluate with groundedness metrics

**Cost Optimization:**
1. Use GPT-3.5 when possible
2. Optimize prompts (reduce tokens)
3. Cache common responses
4. Batch requests
5. Set appropriate max_tokens

**Building RAG System:**
1. Ingest and chunk documents
2. Create embeddings (ada-002)
3. Index in Azure AI Search
4. Implement vector search
5. Augment prompts with context
6. Generate with GPT-4/3.5

### Decision Matrix

**Model Choice:**
- Simple Q&A → GPT-3.5
- Complex reasoning → GPT-4
- Long documents → GPT-4 Turbo
- Cost-sensitive → GPT-3.5
- Vision needed → GPT-4 Turbo with Vision

**Prompt Technique:**
- Simple task → Zero-shot
- Consistent format → Few-shot
- Reasoning → Chain-of-thought
- Sequential tasks → Prompt chaining
- Tool use → ReAct

**Temperature:**
- Code generation → 0.0-0.2
- Factual Q&A → 0.0-0.3
- General chat → 0.7
- Creative writing → 0.8-1.0
- Brainstorming → 1.0+

### Quick Reference

**Prompt Structure:**
```
System: [Role and behavior]
Context: [Background information]
Instruction: [What to do]
Input: [Data to process]
Format: [Output structure]
Examples: [Few-shot demonstrations]
```

**API Parameters:**
- temperature: Randomness (0-2)
- max_tokens: Response length
- top_p: Nucleus sampling (0-1)
- frequency_penalty: Reduce repetition (-2 to 2)
- presence_penalty: New topics (-2 to 2)

**RAG Steps:**
1. Chunk → Split documents
2. Embed → Create vectors
3. Index → Store in vector DB
4. Retrieve → Search similar vectors
5. Augment → Add to prompt
6. Generate → LLM response

### Study Focus

1. **Master prompt engineering** techniques
2. **Understand model selection** criteria
3. **Know RAG architecture** components
4. **Practice with Playground** for hands-on
5. **Learn Prompt Flow** for building apps
6. **Understand token management** and costs
7. **Know evaluation metrics** for quality

### Final Checklist

- [ ] Prompt engineering techniques (zero-shot, few-shot, CoT)
- [ ] System message best practices
- [ ] Model selection criteria
- [ ] Temperature and top_p usage
- [ ] Azure AI Studio components
- [ ] Playground features and export
- [ ] Prompt Flow architecture
- [ ] RAG implementation steps
- [ ] Vector search with Azure AI Search
- [ ] Token limits and management
- [ ] Evaluation metrics
- [ ] Cost optimization strategies
