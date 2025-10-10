# AI and ML Fundamentals for AWS GenAI

## Artificial Intelligence Overview

### Types of AI

**Narrow AI (Weak AI)**
- Designed for specific tasks
- Cannot generalize beyond training
- Examples: Image recognition, voice assistants, recommendation systems
- Current state of commercial AI

**General AI (Strong AI)**
- Human-level intelligence across all domains
- Theoretical, not yet achieved
- Can transfer knowledge between domains
- Learn and adapt to new situations

**Super AI**
- Surpasses human intelligence
- Theoretical concept
- Not currently possible

### AI vs Machine Learning vs Deep Learning

```
┌─────────────────────────────────────────────┐
│ Artificial Intelligence                      │
│  ┌────────────────────────────────────────┐ │
│  │ Machine Learning                        │ │
│  │  ┌────────────────────────────────────┐│ │
│  │  │ Deep Learning                       ││ │
│  │  │  (Neural Networks)                  ││ │
│  │  └────────────────────────────────────┘│ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

**Artificial Intelligence**
- Computer systems that mimic human intelligence
- Rule-based systems, expert systems
- Symbolic reasoning and knowledge representation

**Machine Learning**
- Subset of AI
- Learn from data without explicit programming
- Statistical learning and pattern recognition
- Improves with more data

**Deep Learning**
- Subset of ML
- Neural networks with multiple layers
- Learns hierarchical representations
- Requires large datasets and compute

## Machine Learning Fundamentals

### Types of Machine Learning

**Supervised Learning**
- Labeled training data (input-output pairs)
- Learns to map inputs to outputs
- Examples: Classification, regression
- Use cases: Spam detection, price prediction

**Classification**
- Discrete output (categories)
- Binary: Yes/No, Spam/Not Spam
- Multi-class: Animal types, product categories
- Algorithms: Decision trees, SVM, neural networks

**Regression**
- Continuous output (numbers)
- Predict values (price, temperature, sales)
- Linear regression, polynomial regression
- Algorithms: Linear regression, neural networks

**Unsupervised Learning**
- Unlabeled data
- Find patterns and structure
- Examples: Clustering, dimensionality reduction
- Use cases: Customer segmentation, anomaly detection

**Clustering**
- Group similar data points
- K-means, hierarchical clustering
- Market segmentation, document grouping

**Dimensionality Reduction**
- Reduce number of features
- Preserve important information
- PCA, t-SNE
- Visualization, noise reduction

**Reinforcement Learning**
- Learn through trial and error
- Rewards and penalties
- Agent, environment, actions, rewards
- Use cases: Game playing, robotics, autonomous vehicles

**Semi-Supervised Learning**
- Mix of labeled and unlabeled data
- Leverages large unlabeled datasets
- Reduces labeling cost
- Use cases: Speech recognition, text classification

### ML Workflow

**1. Problem Definition**
- Define business objective
- Identify success metrics
- Determine ML problem type

**2. Data Collection**
- Gather relevant data
- Multiple sources (databases, APIs, web scraping)
- Data quality assessment

**3. Data Preparation**
- Data cleaning (handle missing values, outliers)
- Feature engineering (create meaningful features)
- Data transformation (normalization, encoding)
- Train/validation/test split

**4. Model Selection**
- Choose algorithm type
- Consider problem requirements
- Baseline model first

**5. Model Training**
- Feed training data to model
- Optimize parameters
- Monitor training metrics (loss, accuracy)
- Prevent overfitting (regularization, dropout)

**6. Model Evaluation**
- Test on validation/test set
- Metrics: Accuracy, precision, recall, F1, RMSE
- Compare against baseline
- Error analysis

**7. Model Deployment**
- Deploy to production
- API endpoints
- Batch or real-time inference
- Monitoring and maintenance

**8. Model Monitoring**
- Track performance metrics
- Detect data drift
- Model degradation
- Retrain as needed

### Key Concepts

**Features**
- Input variables (independent variables)
- Characteristics used for prediction
- Feature engineering: Creating new features

**Labels**
- Output variables (dependent variables)
- What we're trying to predict
- Ground truth in training data

**Training**
- Process of learning from data
- Adjusting model parameters
- Minimizing error/loss

**Inference**
- Using trained model for predictions
- Production deployment
- Real-time or batch processing

**Overfitting**
- Model too complex for data
- Memorizes training data
- Poor generalization
- Solutions: Regularization, more data, simpler model

**Underfitting**
- Model too simple
- Cannot capture patterns
- Poor performance on training and test data
- Solutions: More features, complex model, more training

**Bias-Variance Tradeoff**
- Bias: Error from wrong assumptions (underfitting)
- Variance: Error from sensitivity to training data (overfitting)
- Goal: Balance both for good generalization

### Evaluation Metrics

**Classification Metrics**

*Confusion Matrix*
```
                 Predicted
               Positive  Negative
Actual Positive   TP        FN
       Negative   FP        TN
```

- **Accuracy**: (TP + TN) / Total
- **Precision**: TP / (TP + FP) - Of predicted positives, how many are correct?
- **Recall (Sensitivity)**: TP / (TP + FN) - Of actual positives, how many did we find?
- **F1 Score**: 2 × (Precision × Recall) / (Precision + Recall)
- **Specificity**: TN / (TN + FP)

**When to Use**
- High precision: Spam detection (avoid false positives)
- High recall: Disease detection (don't miss cases)
- F1 score: Balanced measure

**Regression Metrics**
- **MAE (Mean Absolute Error)**: Average absolute difference
- **MSE (Mean Squared Error)**: Average squared difference
- **RMSE (Root Mean Squared Error)**: Square root of MSE
- **R² (R-squared)**: Proportion of variance explained

## Deep Learning Fundamentals

### Neural Networks

**Architecture**
- Input layer: Receives features
- Hidden layers: Process information
- Output layer: Produces predictions
- Neurons: Individual processing units
- Weights: Connection strengths
- Activation functions: Non-linear transformations

**Activation Functions**
- **ReLU** (Rectified Linear Unit): max(0, x) - Most common
- **Sigmoid**: 1/(1+e^-x) - Binary classification
- **Tanh**: Hyperbolic tangent - Range [-1, 1]
- **Softmax**: Multi-class probabilities

**Training Process**
1. **Forward propagation**: Calculate predictions
2. **Loss calculation**: Measure error
3. **Backpropagation**: Calculate gradients
4. **Parameter update**: Adjust weights

**Optimization**
- **Gradient Descent**: Move toward minimum loss
- **Learning Rate**: Step size for updates
- **Batch Size**: Samples per update
- **Epochs**: Complete passes through data

**Convolutional Neural Networks (CNNs)**
- Specialized for image data
- Convolutional layers: Learn spatial features
- Pooling layers: Reduce dimensions
- Applications: Image classification, object detection

**Recurrent Neural Networks (RNNs)**
- Sequential data processing
- Memory of previous inputs
- LSTM, GRU: Handle long-term dependencies
- Applications: Text, time series, speech

**Transformers**
- Self-attention mechanism
- Process sequences in parallel
- Foundation for modern LLMs
- Applications: NLP, language models

### Transfer Learning

**Concept**
- Use pre-trained model as starting point
- Fine-tune on specific task
- Requires less data and compute
- Faster training

**Approaches**
1. **Feature Extraction**: Use pre-trained model as feature extractor
2. **Fine-tuning**: Update some or all layers
3. **Domain Adaptation**: Adapt to different but related domain

**Benefits**
- Reduced training time
- Less data required
- Better performance with limited data
- Leverage large pre-trained models

## Generative AI Fundamentals

### What is Generative AI?

**Definition**
- AI that creates new content
- Learns patterns from training data
- Generates novel outputs similar to training data
- Text, images, code, audio, video

**vs Discriminative AI**
- Discriminative: Classify or predict (Y given X)
- Generative: Create new data (X)
- Both use similar underlying techniques

### Types of Generative Models

**Large Language Models (LLMs)**
- Generate human-like text
- Trained on massive text corpora
- Understanding and generation capabilities
- Examples: GPT, Claude, LLaMA, PaLM

**Diffusion Models**
- Image generation
- Iterative denoising process
- High-quality, detailed images
- Examples: Stable Diffusion, DALL-E 2

**Generative Adversarial Networks (GANs)**
- Generator creates fake data
- Discriminator detects fakes
- Adversarial training process
- Applications: Image synthesis, style transfer

**Variational Autoencoders (VAEs)**
- Encode data to latent space
- Decode to generate new samples
- Probabilistic framework
- Applications: Image generation, anomaly detection

### Foundation Models

**Characteristics**
- Trained on broad data at scale
- Adaptable to wide range of tasks
- Large parameter count (billions)
- Transfer learning capabilities

**Capabilities**
- Few-shot learning: Learn from few examples
- Zero-shot learning: Perform unseen tasks
- In-context learning: Use examples in prompt
- Emergent abilities: Unexpected capabilities at scale

**Model Families**
- **Text**: GPT-4, Claude, PaLM, LLaMA
- **Vision**: CLIP, DALL-E, Stable Diffusion
- **Multimodal**: GPT-4V, Gemini, Claude 3
- **Code**: GitHub Copilot, CodeWhisperer, Codex

### Model Training Approaches

**Pre-training**
- Train on large, unlabeled dataset
- Learn general language understanding
- Self-supervised learning
- Predict next word, mask filling

**Fine-tuning**
- Adapt to specific task
- Smaller, labeled dataset
- Update model parameters
- Task-specific performance improvement

**Prompt Engineering**
- Design effective inputs
- No model retraining
- Few-shot examples
- Task instructions

**Retrieval Augmented Generation (RAG)**
- Combine model with external knowledge
- Retrieve relevant information
- Generate response with context
- Up-to-date and accurate information

## Natural Language Processing (NLP)

### NLP Tasks

**Text Classification**
- Assign categories to text
- Sentiment analysis
- Spam detection
- Topic classification

**Named Entity Recognition (NER)**
- Identify entities (person, organization, location)
- Information extraction
- Knowledge graph construction

**Question Answering**
- Answer questions from text
- Extractive: Select answer span
- Generative: Generate answer

**Text Summarization**
- Create concise summary
- Extractive: Select key sentences
- Abstractive: Generate new text

**Machine Translation**
- Translate between languages
- Neural machine translation
- Attention mechanisms

**Text Generation**
- Create coherent text
- Story generation
- Content creation
- Code generation

### Text Representation

**Bag of Words**
- Count word occurrences
- Ignore word order
- Simple but loses context

**TF-IDF**
- Term frequency-inverse document frequency
- Weight words by importance
- Less common words higher weight

**Word Embeddings**
- Dense vector representations
- Capture semantic meaning
- Word2Vec, GloVe
- Similar words have similar vectors

**Contextual Embeddings**
- Embeddings depend on context
- BERT, ELMo
- Same word, different meanings
- State of the art

## Computer Vision

### CV Tasks

**Image Classification**
- Assign category to image
- Single label or multi-label
- Pre-trained models available

**Object Detection**
- Locate and classify objects
- Bounding boxes
- YOLO, R-CNN, Faster R-CNN

**Image Segmentation**
- Pixel-level classification
- Semantic: Class per pixel
- Instance: Separate object instances

**Face Recognition**
- Identify individuals
- Face detection + recognition
- Security, authentication

**Optical Character Recognition (OCR)**
- Extract text from images
- Document digitization
- Amazon Textract

**Image Generation**
- Create new images
- Text-to-image
- Style transfer
- Image editing

## AWS AI/ML Services Overview

### AI Services (Pre-trained)
- **Amazon Rekognition**: Computer vision
- **Amazon Textract**: Document analysis
- **Amazon Comprehend**: NLP
- **Amazon Polly**: Text-to-speech
- **Amazon Transcribe**: Speech-to-text
- **Amazon Translate**: Language translation
- **Amazon Lex**: Conversational AI
- **Amazon Kendra**: Intelligent search

### ML Platform
- **Amazon SageMaker**: End-to-end ML
- **SageMaker Studio**: IDE for ML
- **SageMaker Autopilot**: AutoML
- **SageMaker JumpStart**: Pre-built models

### Generative AI
- **Amazon Bedrock**: Foundation models
- **Amazon Q**: AI assistant
- **Amazon CodeWhisperer**: Code generation

## Best Practices

### Model Development
1. **Start simple**: Baseline model first
2. **Iterate quickly**: Fast experimentation
3. **Measure everything**: Track metrics
4. **Version control**: Models and data
5. **Document assumptions**: Technical debt management

### Data Management
1. **Quality over quantity**: Clean, relevant data
2. **Data versioning**: Track changes
3. **Bias detection**: Ensure fairness
4. **Privacy**: Protect sensitive information
5. **Continuous collection**: Keep data fresh

### Production Deployment
1. **Monitoring**: Track performance
2. **A/B testing**: Compare models
3. **Gradual rollout**: Minimize risk
4. **Rollback capability**: Quick recovery
5. **Cost optimization**: Right-size resources

## Exam Tips

### Key Concepts
- Supervised vs unsupervised learning
- Classification vs regression
- Overfitting vs underfitting
- Precision vs recall tradeoffs
- Transfer learning benefits
- Foundation model capabilities
- RAG for current information
- Prompt engineering basics

### AWS Service Selection
- Pre-trained AI services: Quick start, no ML expertise
- SageMaker: Custom ML models
- Bedrock: Generative AI, foundation models
- Match service to use case and skill level

### Common Patterns
- Image analysis → Rekognition
- Document extraction → Textract
- Text analysis → Comprehend
- Chatbots → Lex + Bedrock
- Custom ML → SageMaker
- Code generation → CodeWhisperer
