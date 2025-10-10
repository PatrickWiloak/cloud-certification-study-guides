# Azure AI Services Overview

## Table of Contents
1. [Azure AI Services Introduction](#azure-ai-services-introduction)
2. [Computer Vision Services](#computer-vision-services)
3. [Language Services](#language-services)
4. [Speech Services](#speech-services)
5. [Decision Services](#decision-services)
6. [Azure OpenAI Service](#azure-openai-service)
7. [Service Integration and Architecture](#service-integration-and-architecture)
8. [Exam Tips](#exam-tips)

---

## Azure AI Services Introduction

### Overview
Azure AI Services (formerly Cognitive Services) are cloud-based AI capabilities that enable developers to build intelligent applications without deep AI/ML expertise.

### Key Categories
1. **Vision** - Analyze images and videos
2. **Language** - Understand and analyze text
3. **Speech** - Convert speech to text and vice versa
4. **Decision** - Make intelligent recommendations and detect anomalies
5. **Azure OpenAI** - Access to advanced language models

### Common Features
- **REST APIs** - Simple HTTP-based interfaces
- **SDKs** - Available for .NET, Python, Java, JavaScript, Go
- **Containers** - Deploy on-premises or edge
- **Custom Models** - Train with your own data
- **Multi-region** - Global availability
- **Security** - Azure AD integration, key-based auth

### Service Tiers
- **Free Tier** - Limited transactions for testing
- **Standard Tier** - Pay-as-you-go pricing
- **Commitment Tier** - Discounted rates for committed usage

---

## Computer Vision Services

### Azure Computer Vision
Analyzes images and extracts information.

#### Key Capabilities
1. **Image Analysis**
   - Object detection
   - Scene understanding
   - Brand detection
   - Face detection (not identification)
   - Color scheme extraction
   - Adult/racy content detection

2. **Optical Character Recognition (OCR)**
   - Read API for text extraction
   - Handwritten text support
   - Multiple language support
   - Document analysis

3. **Spatial Analysis**
   - People counting
   - Social distancing monitoring
   - Zone occupancy detection

#### Common Use Cases
- Catalog search by image
- Accessibility (image descriptions)
- Content moderation
- Document digitization
- Retail analytics

#### API Features
```
POST {endpoint}/vision/v3.2/analyze?visualFeatures=Categories,Tags,Description
Content-Type: application/json

{
  "url": "https://example.com/image.jpg"
}
```

### Custom Vision
Build custom image classification and object detection models.

#### Project Types
1. **Classification**
   - Multi-class (single tag per image)
   - Multi-label (multiple tags per image)

2. **Object Detection**
   - Identify and locate objects with bounding boxes

#### Training Process
1. Create project
2. Upload and tag images (minimum 5-15 per tag)
3. Train model
4. Evaluate performance (precision, recall)
5. Publish prediction endpoint
6. Iterate and improve

#### Performance Metrics
- **Precision** - Accuracy of predictions
- **Recall** - Coverage of correct items
- **AP (Average Precision)** - Overall model quality

### Face Service
Detect, recognize, and analyze human faces.

#### Capabilities
1. **Face Detection**
   - Location (bounding box)
   - Landmarks (eyes, nose, mouth)
   - Attributes (age, emotion, glasses, etc.)

2. **Face Verification**
   - Compare two faces for similarity

3. **Face Identification**
   - Match face against person group
   - Requires training data

4. **Face Grouping**
   - Organize unknown faces by similarity

#### Limited Access Features
- Face identification and verification require approval
- Complies with Responsible AI principles
- Use cases must be reviewed by Microsoft

#### Best Practices
- Use well-lit, frontal images
- Minimum face size: 36x36 pixels
- Maximum 1000 faces per image
- Store face IDs, not original images

### Form Recognizer (Document Intelligence)
Extract information from documents.

#### Model Types
1. **Prebuilt Models**
   - Invoices
   - Receipts
   - Business cards
   - ID documents
   - W-2 forms

2. **Custom Models**
   - Structured forms
   - Unstructured documents

3. **General Document Model**
   - Layout analysis
   - Key-value pairs
   - Tables

#### Key Features
- Table extraction
- Key-value pair detection
- Handwriting support
- Multi-page document support
- Confidence scores

---

## Language Services

### Azure AI Language
Comprehensive text analysis and NLP capabilities.

#### Core Features

1. **Named Entity Recognition (NER)**
   - Person, location, organization
   - Date, time, quantity
   - Email, URL, phone number
   - Custom entity recognition

2. **Key Phrase Extraction**
   - Main talking points
   - Important concepts
   - Works across 120+ languages

3. **Sentiment Analysis**
   - Positive, negative, neutral, mixed
   - Document and sentence-level analysis
   - Opinion mining (aspect-based sentiment)

4. **Language Detection**
   - Identify language of text
   - Confidence scores
   - Supports 120+ languages

5. **Entity Linking**
   - Disambiguate entities
   - Link to Wikipedia
   - Knowledge base integration

#### Advanced Features

1. **Text Analytics for Health**
   - Medical entity extraction
   - UMLS/SNOMED/ICD-10 coding
   - Relation extraction

2. **Personally Identifiable Information (PII) Detection**
   - Redact sensitive information
   - HIPAA and GDPR compliance
   - Custom PII categories

3. **Custom Text Classification**
   - Single-label classification
   - Multi-label classification
   - Minimum 10 documents per class

4. **Custom Named Entity Recognition**
   - Domain-specific entities
   - Minimum 10 labeled documents

#### Question Answering
Build FAQ and conversational Q&A systems.

**Components:**
- Knowledge base (Q&A pairs)
- Multiple sources (URLs, files, editorial)
- Active learning
- Multi-turn conversations
- Synonyms and alterations

**Development Process:**
1. Create knowledge base
2. Add Q&A pairs from sources
3. Train and test
4. Publish endpoint
5. Improve with active learning

#### Conversational Language Understanding (CLU)
Build custom natural language understanding models.

**Concepts:**
- **Intents** - User's goal (e.g., BookFlight)
- **Entities** - Key information (e.g., destination, date)
- **Utterances** - Example phrases

**Development:**
1. Define intents and entities
2. Add training utterances (10-15 per intent)
3. Train model
4. Test and evaluate
5. Deploy to prediction endpoint

### Azure AI Translator
Real-time text translation.

#### Features
- 100+ languages
- Document translation
- Custom translation models
- Transliteration
- Language detection
- Dictionary lookup
- Batch translation

#### Translation Options
1. **Text Translation API**
   - Real-time translation
   - Multiple target languages
   - Custom models

2. **Document Translation**
   - Preserve formatting
   - Batch processing
   - Multiple file formats

3. **Custom Translator**
   - Domain-specific translation
   - Terminology management
   - Translation memory

---

## Speech Services

### Azure AI Speech
Convert speech to text and text to speech.

#### Speech-to-Text (STT)
**Capabilities:**
- Real-time transcription
- Batch transcription
- Multiple languages (100+)
- Custom speech models
- Profanity filtering
- Diarization (speaker identification)

**Recognition Modes:**
- Interactive (short commands)
- Conversation (dialog)
- Dictation (continuous speech)

**Custom Speech:**
- Acoustic models (audio quality)
- Language models (domain vocabulary)
- Pronunciation models

#### Text-to-Speech (TTS)
**Features:**
- Neural voices (natural sounding)
- 400+ voices across 140+ languages
- SSML (Speech Synthesis Markup Language)
- Custom neural voice
- Emotional styles
- Speaking styles (newscast, cheerful, etc.)

**SSML Elements:**
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <prosody rate="medium" pitch="high">
            Hello, how can I help you today?
        </prosody>
    </voice>
</speak>
```

#### Speech Translation
**Capabilities:**
- Real-time speech translation
- 30+ languages
- Multiple target languages simultaneously
- Translation to text or speech

#### Speaker Recognition
**Features:**
- Text-dependent verification
- Text-independent verification
- Speaker identification

---

## Decision Services

### Azure AI Personalizer
Deliver personalized experiences using reinforcement learning.

#### How It Works
1. **Context** - Information about current situation
2. **Actions** - Available options to choose from
3. **Rank** - Service ranks actions
4. **Reward** - Feedback on chosen action
5. **Learn** - Model improves over time

#### Use Cases
- Content recommendations
- Product suggestions
- Ad placement
- Personalized UI

#### Key Concepts
- **Features** - Contextual and action characteristics
- **Exploration** - Try different options
- **Exploitation** - Use best-known option
- **Apprentice Mode** - Shadow existing system

### Anomaly Detector
Identify anomalies in time-series data.

#### Detection Modes
1. **Batch Detection**
   - Analyze entire dataset
   - Detect anomalies across all points

2. **Streaming Detection**
   - Real-time anomaly detection
   - Latest data point analysis

3. **Multivariate Anomaly Detection**
   - Analyze multiple related metrics
   - Detect complex patterns

#### Use Cases
- IoT sensor monitoring
- Business metrics monitoring
- Fraud detection
- System health monitoring

#### Configuration
- **Sensitivity** - Detection threshold (0-99)
- **Custom Interval** - Data point frequency
- **Seasonality** - Periodic patterns

### Content Moderator (Being deprecated)
Detect potentially offensive content.

**Note:** Being replaced by Azure AI Content Safety.

#### Azure AI Content Safety
**Features:**
- Text moderation
- Image moderation
- Multi-category detection (hate, violence, sexual, self-harm)
- Severity levels (0-6)
- Blocklist management
- Custom categories

---

## Azure OpenAI Service

### Overview
Access to OpenAI's advanced language models through Azure.

#### Available Models
1. **GPT-4** - Most capable, multimodal
2. **GPT-3.5** - Fast, cost-effective
3. **Embeddings** - Text vectorization
4. **DALL-E** - Image generation
5. **Whisper** - Speech recognition

#### Key Features
- Enterprise-grade security
- Private networking
- Managed infrastructure
- Content filtering
- Responsible AI tools

#### Common Patterns
1. **Completions** - Generate text
2. **Chat** - Conversational AI
3. **Embeddings** - Semantic search
4. **Fine-tuning** - Custom models

### Use Cases
- Content generation
- Code assistance
- Summarization
- Translation
- Question answering
- Chatbots

---

## Service Integration and Architecture

### Multi-Service Resource
- Single endpoint for multiple services
- Unified key management
- Consolidated billing
- Simplified deployment

### Single-Service Resource
- Dedicated endpoint per service
- Granular access control
- Separate cost tracking
- Region-specific deployment

### Authentication Methods
1. **Subscription Keys**
   - Primary and secondary keys
   - Rotate without downtime
   - Simple to implement

2. **Azure Active Directory**
   - Identity-based access
   - Managed identities
   - Role-based access control (RBAC)

### SDKs and APIs
**Available SDKs:**
- .NET (C#)
- Python
- Java
- JavaScript/Node.js
- Go

**REST API Format:**
```
POST https://{endpoint}/cognitive/service/v1.0/analyze
Ocp-Apim-Subscription-Key: {your-key}
Content-Type: application/json

{
  "parameter": "value"
}
```

### Container Deployment
**Benefits:**
- On-premises deployment
- Edge computing
- Data sovereignty
- Disconnected scenarios

**Requirements:**
- Docker runtime
- Internet for billing
- Minimum resource allocation
- License acceptance

### Best Practices
1. **Security**
   - Use Azure AD when possible
   - Rotate keys regularly
   - Use private endpoints
   - Enable logging and monitoring

2. **Performance**
   - Choose appropriate region
   - Implement retry logic
   - Cache results when appropriate
   - Use batch operations

3. **Cost Optimization**
   - Use free tier for development
   - Choose right service tier
   - Implement commitment pricing
   - Monitor usage

4. **Reliability**
   - Handle throttling (429 errors)
   - Implement circuit breakers
   - Use health checks
   - Plan for failover

---

## Exam Tips

### Key Concepts to Remember

1. **Service Selection**
   - Know which service for which scenario
   - Understand service capabilities
   - Differentiate between similar services

2. **Vision Services**
   - Computer Vision vs Custom Vision vs Face
   - OCR capabilities and limitations
   - Form Recognizer prebuilt models

3. **Language Services**
   - Sentiment analysis vs opinion mining
   - Question Answering vs CLU
   - When to use custom models

4. **Speech Services**
   - STT vs TTS vs Translation
   - Neural voices vs standard voices
   - Custom speech scenarios

5. **Decision Services**
   - Personalizer reinforcement learning
   - Anomaly Detector modes
   - Content Safety categories

6. **Authentication**
   - Subscription keys vs Azure AD
   - When to use managed identities
   - RBAC roles

7. **Deployment**
   - Container deployment scenarios
   - Multi-service vs single-service
   - Region selection considerations

### Common Exam Scenarios

1. **Content Moderation**
   - Use Azure AI Content Safety
   - Configure severity thresholds
   - Implement blocklists

2. **Document Processing**
   - Form Recognizer for structured data
   - Computer Vision OCR for simple text
   - Choose prebuilt vs custom models

3. **Language Understanding**
   - CLU for custom intents/entities
   - Question Answering for FAQ
   - Translator for multi-language

4. **Voice Applications**
   - Speech-to-Text for transcription
   - Text-to-Speech for voice output
   - Speech Translation for real-time

5. **Personalization**
   - Personalizer for dynamic content
   - Context + Actions + Reward pattern
   - Apprentice mode for learning

### Important Limits and Quotas

| Service | Free Tier | Key Limits |
|---------|-----------|------------|
| Computer Vision | 20/min | 5,000/month |
| Face | 20/min | 30,000/month |
| Language | 5,000 records | 25,000/month |
| Speech | 5 audio hours | 1 concurrent request |
| Translator | 2M chars | 10 requests/second |

### Pricing Considerations
- Free tier for development and testing
- Standard tier pay-per-use
- Commitment tiers for predictable workloads
- Custom models may have additional costs
- Container deployment requires connected billing

### Study Focus Areas

1. **Service Capabilities** (30%)
   - What each service does
   - Key features and limitations
   - When to use which service

2. **Implementation** (25%)
   - REST API usage
   - SDK implementation
   - Authentication methods

3. **Custom Models** (20%)
   - Training requirements
   - Data preparation
   - Model evaluation

4. **Deployment** (15%)
   - Container scenarios
   - Resource types
   - Security configuration

5. **Responsible AI** (10%)
   - Content filtering
   - Limited access features
   - Compliance requirements

### Quick Reference

**Computer Vision:**
- Analyze: Tags, objects, faces, OCR
- Custom Vision: Classification & object detection
- Face: Detect, verify, identify (limited access)

**Language:**
- Analyze: Sentiment, entities, key phrases
- Understand: CLU for intents/entities
- Q&A: Question Answering service
- Translate: Text and documents

**Speech:**
- Recognize: Speech-to-Text
- Synthesize: Text-to-Speech
- Translate: Speech Translation
- Identify: Speaker Recognition

**Decision:**
- Personalize: Reinforcement learning recommendations
- Detect: Time-series anomaly detection
- Moderate: Content Safety

**Azure OpenAI:**
- Generate: GPT models
- Embed: Similarity and search
- Create: DALL-E images
- Transcribe: Whisper speech

### Final Exam Tips
1. Read questions carefully - focus on requirements
2. Eliminate obviously wrong answers first
3. Consider cost, performance, and complexity
4. Remember responsible AI principles
5. Think about real-world scenarios
6. Don't overthink - trust your preparation
7. Manage time - don't get stuck on one question
8. Review flagged questions if time permits
