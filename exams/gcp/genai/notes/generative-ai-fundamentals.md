# Generative AI Fundamentals on Google Cloud

## Introduction to Generative AI

### What is Generative AI?

**Definition**
- AI systems that create new content (text, images, code, audio, video)
- Learn patterns from training data
- Generate novel outputs similar to training data
- Goes beyond classification/prediction to creation

**vs Traditional AI**
- **Discriminative AI**: Classifies, predicts (Y given X)
- **Generative AI**: Creates new data (generates X)
- Both use similar underlying techniques (neural networks)

### Types of Generative Models

**Large Language Models (LLMs)**
- Generate human-like text
- Trained on massive text corpora
- Examples: PaLM 2, Gemini, GPT, Claude
- Capabilities: Writing, coding, analysis, translation

**Image Generation Models**
- Create images from text descriptions
- Diffusion models, GANs, VAEs
- Examples: Imagen, Stable Diffusion, DALL-E
- Use cases: Art, design, product visualization

**Multimodal Models**
- Process and generate multiple modalities
- Text, images, audio, video
- Example: Gemini (text, image, video understanding)
- Cross-modal understanding and generation

**Code Generation Models**
- Generate code from natural language
- Complete code snippets
- Examples: Codey, GitHub Copilot, CodeWhisperer
- Use cases: Developer productivity, code completion

## Foundation Models

### Characteristics

**Scale**
- Billions to trillions of parameters
- Trained on diverse, large-scale datasets
- Massive computational requirements
- Emergent capabilities at scale

**Transfer Learning**
- Pre-trained on broad data
- Adaptable to specific tasks
- Fine-tuning with domain data
- Few-shot and zero-shot learning

**Emergent Abilities**
- Unexpected capabilities at scale
- Reasoning, problem-solving
- In-context learning
- Chain-of-thought reasoning

### Google's Foundation Models

**PaLM 2 (Pathways Language Model)**
- State-of-the-art language understanding
- Multilingual capabilities (100+ languages)
- Reasoning and code generation
- Powers Bard and other Google products

**Capabilities**
- Advanced reasoning
- Multilingual translation
- Code generation and debugging
- Mathematical problem-solving
- Document understanding

**Gemini**
- Multimodal foundation model
- Native understanding of text, images, audio, video
- Multiple sizes: Ultra, Pro, Nano
- State-of-the-art performance

**Gemini Variants**
- **Gemini Ultra**: Most capable, complex tasks
- **Gemini Pro**: Best performance/cost balance
- **Gemini Nano**: On-device, efficient

**Imagen**
- Text-to-image generation
- Photorealistic image synthesis
- High resolution outputs
- Style control and editing

**Codey**
- Code generation and completion
- Based on PaLM 2
- Multiple programming languages
- Code explanation and debugging

## Vertex AI Generative AI Studio

### Overview

**Purpose**
- Experiment with generative AI models
- No-code/low-code environment
- Prompt engineering and testing
- Model customization

**Key Features**
- Model Garden: Access to foundation models
- Prompt design and testing
- Tuning and customization
- Deployment to production

### Language Models

**Text Generation**
- PaLM 2 for Text
- Gemini Pro for text
- Adjustable parameters (temperature, top-k, top-p)
- Safety filters and content moderation

**Parameters**
- **Temperature** (0.0-1.0): Randomness/creativity
  - Low (0.0-0.3): Deterministic, focused
  - Medium (0.4-0.7): Balanced
  - High (0.8-1.0): Creative, diverse

- **Top-k**: Consider top k tokens
  - Lower: More focused
  - Higher: More diverse

- **Top-p** (nucleus sampling): Cumulative probability
  - 0.1: Very focused
  - 0.5: Balanced
  - 0.9: Diverse

- **Max output tokens**: Length limit
- **Stop sequences**: Custom stopping conditions

**Example Configuration**
```python
from vertexai.preview.language_models import TextGenerationModel

model = TextGenerationModel.from_pretrained("text-bison@002")

response = model.predict(
    prompt="Write a short story about AI",
    temperature=0.7,
    max_output_tokens=1024,
    top_k=40,
    top_p=0.95
)

print(response.text)
```

### Chat Models

**Conversational AI**
- Multi-turn conversations
- Context maintenance
- Persona/role assignment
- Examples and instructions

**Chat Structure**
- System message: Instructions and role
- Context: Background information
- Examples: Few-shot demonstrations
- Messages: Conversation history

```python
from vertexai.preview.language_models import ChatModel

chat_model = ChatModel.from_pretrained("chat-bison@002")

chat = chat_model.start_chat(
    context="You are a helpful Python programming tutor",
    examples=[
        ["What is a list?", "A list in Python is an ordered, mutable collection..."],
    ]
)

response = chat.send_message("How do I create a dictionary?")
print(response.text)

response = chat.send_message("Can you show an example?")
print(response.text)
```

### Code Models

**Code Generation**
- Codey for code generation
- Code completion
- Code explanation
- Unit test generation

**Code Chat**
- Interactive coding assistance
- Debugging help
- Code review
- Best practices

```python
from vertexai.preview.language_models import CodeGenerationModel

code_model = CodeGenerationModel.from_pretrained("code-bison@002")

response = code_model.predict(
    prefix="Write a Python function to calculate fibonacci numbers"
)

print(response.text)
```

### Image Generation (Imagen)

**Text-to-Image**
- Generate images from descriptions
- Style control
- Aspect ratio selection
- Negative prompts

**Image Editing**
- Inpainting: Fill masked regions
- Outpainting: Extend images
- Style transfer
- Image-to-image translation

```python
from vertexai.preview.vision_models import ImageGenerationModel

imagen = ImageGenerationModel.from_pretrained("imagegeneration@005")

images = imagen.generate_images(
    prompt="A futuristic city at sunset, cyberpunk style",
    number_of_images=4,
    aspect_ratio="1:1",
    safety_filter_level="block_few",
    person_generation="allow_adult"
)

for i, image in enumerate(images):
    image.save(f"generated_image_{i}.png")
```

## Prompt Engineering

### Principles

**1. Be Clear and Specific**
- Detailed instructions
- Specify format and style
- Include constraints
- Define success criteria

**2. Provide Context**
- Background information
- Domain knowledge
- User intent
- Expected output format

**3. Use Examples**
- Few-shot learning
- Show desired format
- Demonstrate reasoning
- Edge case handling

**4. Iterate and Refine**
- Test variations
- Analyze outputs
- Adjust parameters
- A/B testing

### Prompt Patterns

**Zero-Shot**
```
Task: Summarize the following article in 3 sentences.

Article: [article text]

Summary:
```

**Few-Shot**
```
Classify customer reviews as positive, negative, or neutral.

Review: "This product is amazing!"
Sentiment: Positive

Review: "It broke after one day."
Sentiment: Negative

Review: "It's okay, nothing special."
Sentiment: Neutral

Review: "I absolutely love this purchase!"
Sentiment:
```

**Chain-of-Thought**
```
Solve this problem step by step:

Problem: A store has 48 apples. They sell 3/4 of them. How many are left?

Solution:
Let me work through this step by step:
1. First, find 3/4 of 48
2. Calculate: 3/4 × 48 = 36
3. Subtract from original: 48 - 36 = 12
Therefore, 12 apples are left.

Now solve: A class has 32 students. 5/8 are girls. How many boys?
```

**Role-Based**
```
You are an expert cybersecurity analyst with 15 years of experience.

A client asks: "What are the best practices for cloud security?"

Provide a professional response including:
1. Key security principles
2. Specific recommendations
3. Common pitfalls to avoid
4. Tools and services to consider
```

**Structured Output**
```
Extract information from the following text and return as JSON:

Text: "John Smith works at Google in Mountain View, CA. His email is john@google.com"

Output format:
{
  "name": "",
  "company": "",
  "location": "",
  "email": ""
}
```

### Advanced Techniques

**System Instructions**
- Define model behavior
- Set constraints
- Specify output format
- Safety guidelines

**Context Window Management**
- Track token usage
- Summarize long conversations
- Sliding window approach
- Context compression

**Temperature Tuning**
- Low temperature: Factual, consistent
- High temperature: Creative, diverse
- Task-dependent optimization

**Negative Prompts** (for image generation)
- Specify what to avoid
- Reduce unwanted elements
- Improve quality

## Model Tuning and Customization

### Tuning Methods

**Prompt Tuning (Soft Prompts)**
- Learn task-specific prompts
- No model weight changes
- Fast and efficient
- Limited data required

**Adapter Tuning**
- Add small adapter layers
- Freeze base model weights
- Task-specific adapters
- Parameter-efficient

**Fine-Tuning**
- Update model weights
- Requires more data and compute
- Best performance on specific tasks
- Can overfit with small data

### When to Tune

**Prompt Engineering First**
- Quick iteration
- No training required
- Zero-shot or few-shot
- Cost-effective

**Tune When**
- Specific domain terminology
- Consistent output format
- Brand voice requirements
- Performance improvement needed

### Tuning Process

**1. Prepare Data**
```json
{"input_text": "Translate: Hello", "output_text": "Hola"}
{"input_text": "Translate: Goodbye", "output_text": "Adiós"}
```

**2. Create Tuning Job**
```python
from vertexai.preview.language_models import TextGenerationModel

base_model = TextGenerationModel.from_pretrained("text-bison@002")

tuning_job = base_model.tune_model(
    training_data="gs://my-bucket/training_data.jsonl",
    train_steps=100,
    learning_rate=0.001,
    tuned_model_location="gs://my-bucket/tuned_models"
)
```

**3. Evaluate**
- Test on validation set
- Compare with base model
- Measure task-specific metrics

**4. Deploy**
- Deploy tuned model
- A/B test against base model
- Monitor performance

## Responsible AI

### AI Principles

**Fairness**
- Avoid bias in outputs
- Representative training data
- Bias detection and mitigation
- Inclusive design

**Safety**
- Content filtering
- Harmful content detection
- Toxicity screening
- Safety thresholds

**Privacy**
- Data protection
- PII detection and removal
- User consent
- Secure processing

**Transparency**
- Explainability
- Model cards
- Documentation
- Limitations disclosure

### Safety Filters

**Content Categories**
- Hate speech
- Harassment
- Sexually explicit
- Dangerous content
- Violence

**Filter Levels**
- Block none
- Block few
- Block some (default)
- Block most

```python
response = model.predict(
    prompt="Write a story",
    safety_settings={
        "HARM_CATEGORY_HATE_SPEECH": "BLOCK_MEDIUM_AND_ABOVE",
        "HARM_CATEGORY_DANGEROUS_CONTENT": "BLOCK_ONLY_HIGH",
    }
)
```

### Bias Mitigation

**Detection**
- Test diverse inputs
- Evaluate across demographics
- Identify stereotypes
- Check representation

**Mitigation**
- Balanced training data
- Debiasing techniques
- Post-processing filters
- Human review

## Integration and Deployment

### API Integration

**REST API**
```bash
curl -X POST \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json" \
  https://us-central1-aiplatform.googleapis.com/v1/projects/PROJECT_ID/locations/us-central1/publishers/google/models/text-bison:predict \
  -d '{
    "instances": [{
      "prompt": "What is machine learning?"
    }],
    "parameters": {
      "temperature": 0.7,
      "maxOutputTokens": 256
    }
  }'
```

**Python SDK**
```python
import vertexai
from vertexai.preview.language_models import TextGenerationModel

vertexai.init(project="my-project", location="us-central1")

model = TextGenerationModel.from_pretrained("text-bison@002")
response = model.predict("Explain quantum computing")
```

### Deployment Options

**Serverless (Vertex AI)**
- Managed infrastructure
- Auto-scaling
- Pay-per-use
- Low maintenance

**Containers (GKE)**
- Custom control
- Self-managed scaling
- Kubernetes orchestration
- Advanced configurations

**Edge Deployment**
- On-device inference
- Gemini Nano
- Low latency
- Privacy-preserving

### Production Patterns

**Caching**
- Cache common queries
- Reduce latency and cost
- Memorystore integration
- TTL configuration

**Rate Limiting**
- Prevent abuse
- Cost control
- Fair usage
- API quotas

**Fallback Strategies**
- Multiple model tiers
- Graceful degradation
- Error handling
- Retry logic

**Monitoring**
- Request/response logging
- Latency tracking
- Error rates
- Cost monitoring

## Best Practices

### Development

1. **Start with Prompt Engineering**
   - Iterate quickly
   - Test variations
   - Document effective prompts

2. **Use Appropriate Model Size**
   - Gemini Nano: Simple tasks, on-device
   - Gemini Pro: General use, best balance
   - Gemini Ultra: Complex reasoning

3. **Optimize Parameters**
   - Temperature for creativity vs consistency
   - Token limits for cost control
   - Safety filters for content moderation

4. **Test Thoroughly**
   - Diverse inputs
   - Edge cases
   - Bias evaluation
   - Safety testing

### Production

1. **Monitor Performance**
   - Response quality
   - Latency metrics
   - Error rates
   - User feedback

2. **Cost Optimization**
   - Appropriate model selection
   - Caching strategies
   - Batch processing
   - Token usage optimization

3. **Safety and Compliance**
   - Content filtering
   - PII detection
   - Audit logging
   - Compliance requirements

4. **Version Control**
   - Track model versions
   - Prompt versioning
   - A/B testing
   - Rollback capability

## Exam Tips

### Key Concepts

- Understand generative AI fundamentals vs traditional ML
- Know when to use each foundation model (PaLM, Gemini, Codey, Imagen)
- Prompt engineering techniques and best practices
- Model tuning options and when to use them
- Responsible AI principles and safety filters
- Vertex AI Generative AI Studio features

### Service Selection

- **Text generation**: PaLM 2, Gemini Pro
- **Multimodal**: Gemini (text + images)
- **Code**: Codey
- **Images**: Imagen
- **Chat**: Chat-bison, Gemini Pro
- **Simple tasks**: Gemini Nano (on-device)

### Common Scenarios

- Chatbot implementation → Chat models with context
- Content generation → Text generation with prompts
- Code assistance → Codey models
- Image creation → Imagen with prompt engineering
- Multilingual translation → PaLM 2 or Gemini
- Summarization → Text generation with appropriate prompts
- Custom terminology → Model tuning
- Safety requirements → Safety filters and responsible AI practices
