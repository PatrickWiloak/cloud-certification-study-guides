# Responsible AI and AWS AI Services

## Responsible AI Principles

### Core Principles

**Fairness and Bias Mitigation**
- Ensure AI systems treat all users equitably
- Identify and mitigate biases in training data
- Regular auditing for discriminatory outcomes
- Diverse training datasets
- Testing across demographic groups

**Transparency and Explainability**
- Understand how AI makes decisions
- Provide explanations for AI outputs
- Document model limitations
- Clear communication about AI usage
- Interpretable model architectures when possible

**Privacy and Data Protection**
- Protect user data and personal information
- Comply with privacy regulations (GDPR, CCPA)
- Minimize data collection
- Secure data storage and transmission
- User consent and control

**Safety and Reliability**
- Ensure AI systems work as intended
- Robust testing and validation
- Fail-safe mechanisms
- Monitor for unexpected behaviors
- Graceful degradation

**Accountability**
- Clear ownership and responsibility
- Audit trails for AI decisions
- Human oversight and intervention
- Compliance with regulations
- Ethical review processes

**Security**
- Protect against adversarial attacks
- Secure model deployment
- Access controls
- Prevent misuse
- Regular security assessments

### AWS Responsible AI Framework

**Design Phase**
- Define clear use case and success metrics
- Assess potential harms and benefits
- Identify stakeholders and impact
- Plan for bias detection and mitigation
- Document ethical considerations

**Development Phase**
- Use diverse and representative datasets
- Test for bias across demographics
- Implement explainability features
- Security by design
- Privacy-preserving techniques

**Deployment Phase**
- Gradual rollout with monitoring
- Human-in-the-loop controls
- Clear user communication
- Incident response plan
- Continuous evaluation

**Operations Phase**
- Monitor for drift and degradation
- Regular bias audits
- User feedback mechanisms
- Model retraining strategy
- Documentation updates

## Bias Detection and Mitigation

### Types of Bias

**Data Bias**
- Historical bias: Past discrimination in data
- Representation bias: Underrepresented groups
- Measurement bias: Inconsistent data collection
- Aggregation bias: Inappropriate grouping

**Algorithmic Bias**
- Selection bias: Biased feature selection
- Interaction bias: User feedback loops
- Confirmation bias: Reinforcing existing patterns
- Automation bias: Over-reliance on AI

### Bias Detection Techniques

**Data Analysis**
```python
import pandas as pd
from sklearn.metrics import confusion_matrix

# Analyze data distribution across groups
def analyze_data_bias(df, protected_attribute):
    # Check representation
    distribution = df[protected_attribute].value_counts(normalize=True)
    print(f"Distribution across {protected_attribute}:")
    print(distribution)

    # Check outcome distribution
    outcome_by_group = df.groupby(protected_attribute)['outcome'].mean()
    print(f"\nOutcome rates by {protected_attribute}:")
    print(outcome_by_group)

    return distribution, outcome_by_group

# Example usage
protected_attrs = ['gender', 'race', 'age_group']
for attr in protected_attrs:
    analyze_data_bias(training_data, attr)
```

**Model Performance Analysis**
```python
from sklearn.metrics import accuracy_score, precision_score, recall_score

def evaluate_fairness(y_true, y_pred, protected_attr):
    """
    Evaluate model performance across protected groups
    """
    groups = protected_attr.unique()
    results = {}

    for group in groups:
        mask = protected_attr == group
        results[group] = {
            'accuracy': accuracy_score(y_true[mask], y_pred[mask]),
            'precision': precision_score(y_true[mask], y_pred[mask]),
            'recall': recall_score(y_true[mask], y_pred[mask]),
            'count': mask.sum()
        }

    return pd.DataFrame(results).T
```

**SageMaker Clarify for Bias Detection**
```python
from sagemaker import clarify

# Configure bias metrics
bias_config = clarify.BiasConfig(
    label_values_or_threshold=[1],
    facet_name='gender',  # Protected attribute
    facet_values_or_threshold=[0]  # Reference group
)

# Pre-training bias metrics
clarify_processor = clarify.SageMakerClarifyProcessor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    sagemaker_session=sagemaker_session
)

clarify_processor.run_pre_training_bias(
    data_config=data_config,
    data_bias_config=bias_config,
    methods='all',
    output_path='s3://bucket/bias-report'
)

# Post-training bias metrics
predictions_config = clarify.ModelPredictedLabelConfig(
    probability_threshold=0.5
)

clarify_processor.run_post_training_bias(
    data_config=data_config,
    data_bias_config=bias_config,
    model_config=model_config,
    model_predicted_label_config=predictions_config,
    methods='all',
    output_path='s3://bucket/bias-report-post'
)
```

### Bias Mitigation Strategies

**Data-Level Mitigation**
- Oversample underrepresented groups
- Synthetic data generation for minority classes
- Reweighting training samples
- Remove biased features when appropriate
- Collect more diverse data

**Algorithm-Level Mitigation**
- Fair learning algorithms
- Adversarial debiasing
- Prejudice remover regularizer
- Fairness constraints in optimization
- Calibrated equalized odds

**Post-Processing Mitigation**
- Threshold optimization per group
- Reject option classification
- Equalized odds post-processing
- Calibration techniques

## Model Explainability

### Explainability Techniques

**Feature Importance**
- Which features contribute most to predictions
- Global understanding of model behavior
- SHAP, LIME, permutation importance

**SageMaker Clarify for Explainability**
```python
from sagemaker import clarify

# Configure SHAP explainability
shap_config = clarify.SHAPConfig(
    baseline=[data_baseline],
    num_samples=100,
    agg_method='mean_abs',
    use_logit=False,
    save_local_shap_values=True
)

# Run explainability analysis
clarify_processor.run_explainability(
    data_config=data_config,
    model_config=model_config,
    explainability_config=shap_config,
    output_path='s3://bucket/explainability-report'
)
```

**Model-Specific Explainability**
```python
import shap
import lime
import lime.lime_tabular

# SHAP (SHapley Additive exPlanations)
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

# Visualize feature importance
shap.summary_plot(shap_values, X_test)
shap.force_plot(explainer.expected_value, shap_values[0], X_test.iloc[0])

# LIME (Local Interpretable Model-agnostic Explanations)
lime_explainer = lime.lime_tabular.LimeTabularExplainer(
    X_train.values,
    feature_names=X_train.columns,
    class_names=['Class 0', 'Class 1'],
    mode='classification'
)

# Explain individual prediction
exp = lime_explainer.explain_instance(
    X_test.iloc[0].values,
    model.predict_proba,
    num_features=10
)
exp.show_in_notebook()
```

### Bedrock Guardrails

**Content Filtering**
```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

# Create guardrail configuration
guardrail_config = {
    'contentFilters': [
        {
            'type': 'HATE',
            'strength': 'HIGH'
        },
        {
            'type': 'VIOLENCE',
            'strength': 'HIGH'
        },
        {
            'type': 'SEXUAL',
            'strength': 'MEDIUM'
        },
        {
            'type': 'MISCONDUCT',
            'strength': 'HIGH'
        }
    ],
    'topicFilters': [
        {
            'name': 'Financial Advice',
            'definition': 'Providing investment or financial guidance',
            'action': 'BLOCK'
        },
        {
            'name': 'Medical Diagnosis',
            'definition': 'Diagnosing medical conditions',
            'action': 'BLOCK'
        }
    ],
    'wordFilters': [
        {
            'words': ['blocked_word1', 'blocked_word2'],
            'action': 'BLOCK'
        }
    ],
    'piiFilters': [
        {
            'type': 'EMAIL',
            'action': 'MASK'
        },
        {
            'type': 'PHONE',
            'action': 'MASK'
        },
        {
            'type': 'SSN',
            'action': 'BLOCK'
        }
    ]
}

# Invoke model with guardrails
body = json.dumps({
    "prompt": f"\n\nHuman: {user_prompt}\n\nAssistant:",
    "max_tokens_to_sample": 1000
})

response = bedrock.invoke_model(
    modelId='anthropic.claude-v2',
    body=body,
    contentType='application/json',
    guardrailIdentifier='my-guardrail-id',
    guardrailVersion='1'
)

# Check if content was filtered
response_body = json.loads(response['body'].read())
if 'amazon-bedrock-guardrailAction' in response.get('ResponseMetadata', {}).get('HTTPHeaders', {}):
    print("Content was filtered by guardrails")
```

**Guardrail Configuration Best Practices**
- Start with high sensitivity, adjust based on use case
- Define clear topic boundaries
- Test extensively with edge cases
- Monitor guardrail triggers
- Update based on user feedback
- Document filtering rationale

## AWS AI Services Overview

### Amazon Rekognition

**Capabilities**
- Object and scene detection
- Facial analysis and recognition
- Text in images (OCR)
- Celebrity recognition
- Content moderation
- Custom labels for specific use cases

**Use Cases**
- User verification and authentication
- Content moderation for social media
- Visual search applications
- Safety and security monitoring
- Retail analytics

**Code Example**
```python
import boto3

rekognition = boto3.client('rekognition')

# Detect labels in image
response = rekognition.detect_labels(
    Image={
        'S3Object': {
            'Bucket': 'my-bucket',
            'Name': 'image.jpg'
        }
    },
    MaxLabels=10,
    MinConfidence=90,
    Features=['GENERAL_LABELS', 'IMAGE_PROPERTIES']
)

for label in response['Labels']:
    print(f"{label['Name']}: {label['Confidence']:.2f}%")

# Content moderation
moderation_response = rekognition.detect_moderation_labels(
    Image={'S3Object': {'Bucket': 'bucket', 'Name': 'image.jpg'}},
    MinConfidence=60
)

for label in moderation_response['ModerationLabels']:
    print(f"Moderation: {label['Name']} - {label['Confidence']:.2f}%")
    print(f"  Parent: {label['ParentName']}")

# Face analysis
face_response = rekognition.detect_faces(
    Image={'Bytes': image_bytes},
    Attributes=['ALL']
)

for face in face_response['FaceDetails']:
    print(f"Age range: {face['AgeRange']}")
    print(f"Gender: {face['Gender']}")
    print(f"Emotions: {face['Emotions']}")
```

### Amazon Textract

**Capabilities**
- Text detection and OCR
- Form extraction (key-value pairs)
- Table extraction
- Document analysis
- Handwriting recognition
- Query-based extraction

**Use Cases**
- Invoice processing
- Medical records digitization
- Identity document verification
- Financial document analysis
- Contract analysis

**Code Example**
```python
textract = boto3.client('textract')

# Simple text detection
response = textract.detect_document_text(
    Document={
        'S3Object': {
            'Bucket': 'bucket',
            'Name': 'document.pdf'
        }
    }
)

# Extract all text
for item in response['Blocks']:
    if item['BlockType'] == 'LINE':
        print(item['Text'])

# Analyze document with forms and tables
analysis_response = textract.analyze_document(
    Document={'S3Object': {'Bucket': 'bucket', 'Name': 'form.pdf'}},
    FeatureTypes=['FORMS', 'TABLES']
)

# Extract key-value pairs
for block in analysis_response['Blocks']:
    if block['BlockType'] == 'KEY_VALUE_SET':
        if 'KEY' in block['EntityTypes']:
            print(f"Key: {extract_text(block)}")
        else:
            print(f"Value: {extract_text(block)}")

# Analyze expense documents
expense_response = textract.analyze_expense(
    Document={'S3Object': {'Bucket': 'bucket', 'Name': 'receipt.jpg'}}
)

for expense_doc in expense_response['ExpenseDocuments']:
    for field in expense_doc['SummaryFields']:
        print(f"{field['Type']['Text']}: {field['ValueDetection']['Text']}")
```

### Amazon Comprehend

**Capabilities**
- Sentiment analysis
- Entity recognition
- Key phrase extraction
- Language detection
- Topic modeling
- Custom classification
- PII detection and redaction

**Use Cases**
- Customer feedback analysis
- Document classification
- Social media monitoring
- Compliance and redaction
- Content recommendation

**Code Example**
```python
comprehend = boto3.client('comprehend')

# Sentiment analysis
sentiment = comprehend.detect_sentiment(
    Text='I love this product! It exceeded my expectations.',
    LanguageCode='en'
)
print(f"Sentiment: {sentiment['Sentiment']}")
print(f"Scores: {sentiment['SentimentScore']}")

# Entity recognition
entities = comprehend.detect_entities(
    Text='Amazon Web Services is based in Seattle, Washington.',
    LanguageCode='en'
)

for entity in entities['Entities']:
    print(f"{entity['Text']}: {entity['Type']} ({entity['Score']:.2f})")

# PII detection and redaction
pii_response = comprehend.detect_pii_entities(
    Text='My email is john@example.com and phone is 555-1234',
    LanguageCode='en'
)

for entity in pii_response['Entities']:
    print(f"PII Type: {entity['Type']}, Score: {entity['Score']:.2f}")

# Custom classification
classification = comprehend.classify_document(
    Text=document_text,
    EndpointArn='arn:aws:comprehend:region:account:document-classifier-endpoint/name'
)

for class_result in classification['Classes']:
    print(f"{class_result['Name']}: {class_result['Score']:.2f}")
```

### Amazon Translate

**Capabilities**
- Neural machine translation
- 75+ languages
- Real-time and batch translation
- Custom terminology
- Automatic language detection
- Formality customization

**Use Cases**
- Website localization
- Customer support in multiple languages
- Document translation
- Real-time chat translation
- E-commerce product descriptions

**Code Example**
```python
translate = boto3.client('translate')

# Translate text
response = translate.translate_text(
    Text='Hello, how are you?',
    SourceLanguageCode='en',
    TargetLanguageCode='es'
)
print(response['TranslatedText'])

# Auto-detect source language
auto_response = translate.translate_text(
    Text='Bonjour, comment allez-vous?',
    SourceLanguageCode='auto',
    TargetLanguageCode='en'
)
print(f"Detected: {auto_response['SourceLanguageCode']}")
print(f"Translation: {auto_response['TranslatedText']}")

# Batch translation
start_response = translate.start_text_translation_job(
    InputDataConfig={
        'S3Uri': 's3://bucket/input/',
        'ContentType': 'text/plain'
    },
    OutputDataConfig={
        'S3Uri': 's3://bucket/output/'
    },
    DataAccessRoleArn='arn:aws:iam::account:role/TranslateRole',
    SourceLanguageCode='en',
    TargetLanguageCodes=['es', 'fr', 'de']
)

job_id = start_response['JobId']
```

### Amazon Polly

**Capabilities**
- Text-to-speech synthesis
- Natural-sounding voices
- 60+ voices in 30+ languages
- Neural TTS for most realistic voices
- SSML support for pronunciation control
- Speech marks for synchronization

**Use Cases**
- Voice assistants and chatbots
- Accessibility features
- E-learning narration
- IVR systems
- Content creation

**Code Example**
```python
polly = boto3.client('polly')

# Synthesize speech
response = polly.synthesize_speech(
    Text='Hello! Welcome to AWS Polly.',
    OutputFormat='mp3',
    VoiceId='Joanna',
    Engine='neural'
)

# Save audio file
with open('output.mp3', 'wb') as file:
    file.write(response['AudioStream'].read())

# SSML for advanced control
ssml_text = '''
<speak>
    Hello! <break time="1s"/>
    My name is <prosody rate="slow">Joanna</prosody>.
    <emphasis level="strong">Welcome</emphasis> to AWS Polly!
</speak>
'''

ssml_response = polly.synthesize_speech(
    Text=ssml_text,
    TextType='ssml',
    OutputFormat='mp3',
    VoiceId='Joanna',
    Engine='neural'
)

# Long-form narration (asynchronous)
start_response = polly.start_speech_synthesis_task(
    Text=long_text,
    OutputFormat='mp3',
    OutputS3BucketName='my-bucket',
    VoiceId='Joanna',
    Engine='neural'
)

task_id = start_response['SynthesisTask']['TaskId']
```

### Amazon Transcribe

**Capabilities**
- Speech-to-text conversion
- Real-time and batch transcription
- Speaker identification
- Custom vocabulary
- Automatic language identification
- PII redaction
- Medical transcription

**Use Cases**
- Meeting transcription
- Subtitle generation
- Call center analytics
- Voice commands
- Medical documentation

**Code Example**
```python
transcribe = boto3.client('transcribe')

# Start transcription job
job_name = 'my-transcription-job'
transcribe.start_transcription_job(
    TranscriptionJobName=job_name,
    Media={'MediaFileUri': 's3://bucket/audio.mp3'},
    MediaFormat='mp3',
    LanguageCode='en-US',
    Settings={
        'ShowSpeakerLabels': True,
        'MaxSpeakerLabels': 2,
        'VocabularyName': 'my-custom-vocabulary'
    },
    ContentRedaction={
        'RedactionType': 'PII',
        'RedactionOutput': 'redacted'
    }
)

# Check job status
status = transcribe.get_transcription_job(TranscriptionJobName=job_name)
print(f"Status: {status['TranscriptionJob']['TranscriptionJobStatus']}")

# Real-time transcription (using WebSocket)
from amazon_transcribe.client import TranscribeStreamingClient
from amazon_transcribe.handlers import TranscriptResultStreamHandler

async def transcribe_streaming():
    client = TranscribeStreamingClient(region='us-east-1')

    stream = await client.start_stream_transcription(
        language_code='en-US',
        media_sample_rate_hz=16000,
        media_encoding='pcm'
    )

    async for event in stream.output_stream:
        for result in event.transcript_result_stream:
            for alt in result.alternatives:
                print(alt.transcript)
```

### Amazon Lex

**Capabilities**
- Conversational AI chatbots
- Natural language understanding
- Automatic speech recognition integration
- Multi-turn conversations
- Slot filling and validation
- Integration with Lambda for business logic
- Multi-language support

**Use Cases**
- Customer service chatbots
- Voice assistants
- FAQ automation
- Appointment booking
- Order tracking

**Code Example**
```python
lex = boto3.client('lexv2-runtime')

# Send text input to bot
response = lex.recognize_text(
    botId='bot-id',
    botAliasId='bot-alias-id',
    localeId='en_US',
    sessionId='user-session-123',
    text='I want to book a hotel'
)

print(f"Intent: {response['interpretations'][0]['intent']['name']}")
print(f"Response: {response['messages'][0]['content']}")

# Voice input
with open('audio.pcm', 'rb') as audio_file:
    voice_response = lex.recognize_utterance(
        botId='bot-id',
        botAliasId='bot-alias-id',
        localeId='en_US',
        sessionId='user-session-123',
        requestContentType='audio/l16; rate=16000; channels=1',
        inputStream=audio_file
    )
```

### Amazon Forecast

**Capabilities**
- Time series forecasting
- AutoML for algorithm selection
- Multiple forecasting algorithms
- Related time series support
- Holiday and event handling
- Probabilistic forecasts with quantiles

**Use Cases**
- Demand forecasting
- Inventory planning
- Resource capacity planning
- Financial forecasting
- Energy consumption prediction

**Code Example**
```python
forecast = boto3.client('forecast')

# Create dataset
dataset_response = forecast.create_dataset(
    DatasetName='sales_data',
    Domain='RETAIL',
    DatasetType='TARGET_TIME_SERIES',
    DataFrequency='D',
    Schema={
        'Attributes': [
            {'AttributeName': 'timestamp', 'AttributeType': 'timestamp'},
            {'AttributeName': 'item_id', 'AttributeType': 'string'},
            {'AttributeName': 'demand', 'AttributeType': 'float'}
        ]
    }
)

# Create auto predictor
predictor_response = forecast.create_auto_predictor(
    PredictorName='sales_predictor',
    ForecastHorizon=30,
    ForecastFrequency='D',
    ForecastTypes=['0.50', '0.70', '0.90']
)

# Generate forecast
forecast_response = forecast.create_forecast(
    ForecastName='sales_forecast',
    PredictorArn=predictor_arn
)

# Query forecast
query_response = forecast.query_forecast(
    ForecastArn=forecast_arn,
    Filters={
        'item_id': 'item_123'
    }
)
```

## Responsible AI Best Practices

### Development Guidelines

**Data Collection and Preparation**
1. Diverse and representative datasets
2. Document data sources and collection methods
3. Regular data quality audits
4. Remove biased or sensitive features when appropriate
5. Implement data versioning

**Model Development**
1. Baseline fairness metrics before training
2. Regular bias testing during development
3. Explainability as a requirement, not afterthought
4. Multiple model evaluation metrics
5. A/B testing for production deployment

**Testing and Validation**
1. Test across diverse scenarios and demographics
2. Adversarial testing for robustness
3. Edge case identification and testing
4. User acceptance testing
5. Security and privacy testing

### Operational Best Practices

**Monitoring**
```python
# CloudWatch metrics for model monitoring
cloudwatch = boto3.client('cloudwatch')

# Track fairness metrics
cloudwatch.put_metric_data(
    Namespace='AI/Fairness',
    MetricData=[
        {
            'MetricName': 'DisparateImpact',
            'Value': disparate_impact_score,
            'Unit': 'None',
            'Dimensions': [
                {'Name': 'ModelName', 'Value': 'credit-model'},
                {'Name': 'ProtectedAttribute', 'Value': 'gender'}
            ]
        }
    ]
)

# Alert on bias detection
alarm = cloudwatch.put_metric_alarm(
    AlarmName='BiasDetected',
    MetricName='DisparateImpact',
    Namespace='AI/Fairness',
    Threshold=0.8,
    ComparisonOperator='LessThanThreshold',
    EvaluationPeriods=1,
    AlarmActions=['arn:aws:sns:region:account:topic']
)
```

**Documentation**
- Model cards documenting intended use
- Known limitations and biases
- Performance across different groups
- Training data characteristics
- Evaluation methodology
- Update history

**Governance**
- Regular ethical reviews
- Diverse stakeholder input
- Clear escalation procedures
- Compliance tracking
- Audit trails

## Exam Tips

### Key Concepts to Remember

**Responsible AI**
- Fairness: Equal treatment across groups
- Transparency: Explainable decisions
- Privacy: Data protection and consent
- Safety: Reliable and secure operation
- Accountability: Clear responsibility

**Bias Detection**
- Pre-training bias in data
- Post-training bias in predictions
- SageMaker Clarify for analysis
- Fairness metrics (disparate impact, equal opportunity)

**Explainability**
- SHAP for feature importance
- LIME for local explanations
- SageMaker Clarify integration
- Model-specific interpretation

**AWS AI Services**
- Rekognition: Computer vision
- Textract: Document extraction
- Comprehend: NLP and text analysis
- Translate: Language translation
- Polly: Text-to-speech
- Transcribe: Speech-to-text
- Lex: Conversational AI
- Forecast: Time series prediction

### Common Scenarios

**Content Moderation**
- Use Rekognition for image/video moderation
- Comprehend for text sentiment and toxicity
- Bedrock Guardrails for generative AI
- Custom classifiers for domain-specific content

**Document Processing**
- Textract for extraction
- Comprehend for analysis and classification
- Translate for multi-language support
- Store results in searchable format (OpenSearch)

**Customer Service Automation**
- Lex for chatbot conversations
- Transcribe for call transcription
- Comprehend for sentiment analysis
- Translate for multi-language support
- Polly for voice responses

**Accessibility**
- Transcribe for captions
- Polly for audio content
- Textract for document accessibility
- Translate for language accessibility

### Service Selection Guide

**When to use which service:**
- Images/Video → Rekognition
- Documents (forms, tables) → Textract
- Text analysis → Comprehend
- Translation → Translate
- Voice output → Polly
- Voice input → Transcribe
- Chatbots → Lex + Bedrock
- Forecasting → Forecast
- Custom ML → SageMaker
- Generative AI → Bedrock

**Integration patterns:**
- Document pipeline: S3 → Textract → Comprehend → DynamoDB
- Voice assistant: Transcribe → Lex → Lambda → Polly
- Content moderation: S3 → Rekognition → Lambda → SNS
- Multi-language chat: Transcribe → Translate → Lex → Translate → Polly
