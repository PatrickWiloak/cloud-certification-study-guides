# AWS Lambda

## Service Overview and Purpose

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers. It automatically scales your applications by running code in response to triggers and only charges for the compute time consumed.

**Core Purpose:**
- Execute code in response to events without managing infrastructure
- Automatically scale from zero to thousands of concurrent executions
- Pay only for compute time consumed (millisecond billing)
- Integrate seamlessly with other AWS services
- Enable event-driven and serverless architectures

## Key Features and Capabilities

### Core Features
- **Event-Driven Execution**: Triggered by various AWS services and external events
- **Automatic Scaling**: Scales automatically based on incoming requests
- **Multiple Runtime Support**: Python, Node.js, Java, C#, Go, Ruby, PowerShell
- **Built-in Fault Tolerance**: Automatically handles infrastructure failures
- **Integrated Security**: Built-in AWS IAM integration
- **Monitoring**: Integrated with CloudWatch for logging and metrics
- **Versioning**: Support for function versions and aliases
- **Environment Variables**: Secure configuration management
- **Dead Letter Queues**: Error handling and retry logic
- **Concurrency Control**: Reserved and provisioned concurrency

### Supported Runtimes
- **Python**: 3.8, 3.9, 3.10, 3.11
- **Node.js**: 14.x, 16.x, 18.x
- **Java**: 8, 11, 17
- **C#**: .NET Core 3.1, .NET 6
- **Go**: 1.x
- **Ruby**: 2.7, 3.2
- **PowerShell**: Core 6.0
- **Custom Runtime**: Using runtime API

### Execution Environment
- **Memory**: 128 MB to 10,008 MB (1 MB increments)
- **CPU**: Proportional to memory allocation
- **Timeout**: Maximum 15 minutes
- **Temporary Storage**: 512 MB to 10,240 MB (/tmp directory)
- **Environment Variables**: Up to 4 KB
- **Request/Response**: 6 MB (synchronous), 256 KB (asynchronous)

## Use Cases and Scenarios

### Primary Use Cases

1. **API Backends**: RESTful APIs with API Gateway
2. **Data Processing**: ETL operations and data transformation
3. **File Processing**: Image/video processing, document conversion
4. **IoT Backends**: Process sensor data and device telemetry
5. **Chatbots**: Natural language processing and response generation
6. **Scheduled Tasks**: Cron-like functionality using CloudWatch Events
7. **Stream Processing**: Real-time data stream analysis
8. **Webhooks**: HTTP endpoint for third-party integrations
9. **Authentication**: Custom authorizers for API Gateway
10. **Microservices**: Decomposed application components

### Detailed Scenarios

#### Event-Driven Processing
```
S3 Object Upload → Lambda Function → Process Image → Store in DynamoDB
CloudWatch Log → Lambda Function → Parse Logs → Send to ElasticSearch
SQS Message → Lambda Function → Process Order → Update Database
```

#### API Development
```
API Gateway → Lambda Function → Business Logic → RDS/DynamoDB
API Gateway → Lambda Authorizer → Validate Token → Allow/Deny Request
```

#### Data Pipeline
```
Kinesis Stream → Lambda Function → Transform Data → S3/Redshift
DynamoDB Stream → Lambda Function → Sync Data → ElasticSearch
```

#### Automation
```
CloudWatch Alarm → Lambda Function → Auto-remediation → Systems Manager
EC2 State Change → Lambda Function → Backup Creation → Notification
```

## Pricing Models and Cost Optimization

### Pricing Components

1. **Request Charges**
   - $0.20 per 1 million requests
   - First 1 million requests per month are free

2. **Duration Charges**
   - Based on GB-seconds (memory × execution time)
   - Price varies by memory allocation
   - Example: $0.0000166667 per GB-second

3. **Provisioned Concurrency**
   - $0.0000097222 per GB-second for provisioned capacity
   - $0.20 per 1 million requests on provisioned capacity

### Free Tier
- 1 million free requests per month
- 400,000 GB-seconds of compute time per month

### Cost Optimization Strategies

1. **Right-size Memory Allocation**
   - Monitor execution time vs. memory
   - Use AWS Lambda Power Tuning for optimization
   - Higher memory = faster execution but higher cost

2. **Optimize Code Performance**
   - Minimize cold start time
   - Reuse connections and objects
   - Use connection pooling
   - Optimize dependencies and package size

3. **Use Provisioned Concurrency Strategically**
   - Only for functions requiring consistent low latency
   - Monitor utilization to avoid over-provisioning

4. **Implement Efficient Architectures**
   - Use appropriate trigger patterns
   - Batch processing when possible
   - Minimize external API calls

5. **Monitor and Alert**
   - Set up CloudWatch alarms for cost thresholds
   - Use AWS Cost Explorer for analysis
   - Implement cost allocation tags

## Configuration Details and Best Practices

### Function Configuration

#### Basic Configuration
```json
{
  "FunctionName": "my-function",
  "Runtime": "python3.9",
  "Handler": "lambda_function.lambda_handler",
  "Code": {
    "ZipFile": "fileb://function.zip"
  },
  "Role": "arn:aws:iam::account:role/lambda-role",
  "Description": "My Lambda function",
  "Timeout": 300,
  "MemorySize": 128,
  "Environment": {
    "Variables": {
      "ENV": "production",
      "DB_HOST": "rds-endpoint"
    }
  }
}
```

#### Advanced Configuration
```json
{
  "VpcConfig": {
    "SubnetIds": ["subnet-12345", "subnet-67890"],
    "SecurityGroupIds": ["sg-12345"]
  },
  "DeadLetterConfig": {
    "TargetArn": "arn:aws:sqs:region:account:dlq"
  },
  "TracingConfig": {
    "Mode": "Active"
  },
  "ReservedConcurrencyLimit": 100,
  "FileSystemConfigs": [{
    "Arn": "arn:aws:elasticfilesystem:region:account:access-point/fsap-12345",
    "LocalMountPath": "/mnt/efs"
  }]
}
```

### Best Practices

#### Code Organization
1. **Single Responsibility**: One function per business logic unit
2. **Stateless Design**: Don't rely on local state between invocations
3. **Idempotent Operations**: Functions should be safely retryable
4. **Error Handling**: Implement comprehensive error handling
5. **Logging**: Use structured logging for better observability

#### Performance Optimization
1. **Cold Start Reduction**
   - Minimize package size
   - Use provisioned concurrency for critical functions
   - Initialize connections outside handler
   - Use AWS Lambda Layers for shared dependencies

2. **Memory and CPU Optimization**
   - Monitor CloudWatch metrics
   - Use AWS Lambda Power Tuning tool
   - Balance memory allocation with performance

3. **Connection Management**
   - Reuse database connections
   - Use connection pooling
   - Cache configuration data

#### Security Best Practices
1. **IAM Roles**: Use least privilege principle
2. **Environment Variables**: Encrypt sensitive data
3. **VPC Configuration**: Use when accessing private resources
4. **Input Validation**: Validate all inputs
5. **Secrets Management**: Use AWS Secrets Manager or Parameter Store

#### Deployment Best Practices
1. **Version Control**: Use function versions and aliases
2. **Blue/Green Deployment**: Gradual traffic shifting
3. **Infrastructure as Code**: Use CloudFormation or CDK
4. **Automated Testing**: Unit and integration tests
5. **CI/CD Pipeline**: Automated deployment pipeline

## Integration with Other AWS Services

### Event Sources (Triggers)

1. **API Gateway**: HTTP/REST API requests
2. **S3**: Object creation, deletion, metadata changes
3. **DynamoDB**: Stream records for table changes
4. **Kinesis**: Stream and Firehose data records
5. **SQS**: Queue messages
6. **SNS**: Topic notifications
7. **CloudWatch Events/EventBridge**: Scheduled events and state changes
8. **CloudWatch Logs**: Log group events
9. **Cognito**: Authentication events
10. **ALB**: Application Load Balancer target

### Destinations and Outputs

1. **DynamoDB**: Store processed data
2. **S3**: Save files and results
3. **RDS**: Update relational databases
4. **ElastiCache**: Cache frequently accessed data
5. **SES**: Send emails
6. **SNS**: Publish notifications
7. **SQS**: Queue messages for further processing
8. **Step Functions**: Orchestrate workflows
9. **CloudWatch**: Log metrics and events
10. **X-Ray**: Distributed tracing

### Service Integration Patterns

#### API Gateway + Lambda
```
Client → API Gateway → Lambda → Database
```
- RESTful API development
- Authentication and authorization
- Request/response transformation

#### S3 + Lambda
```
File Upload → S3 Bucket → Lambda Trigger → Process File
```
- Image/video processing
- Data transformation
- File validation

#### DynamoDB + Lambda
```
DynamoDB Change → DynamoDB Stream → Lambda → Secondary Processing
```
- Data synchronization
- Search index updates
- Audit logging

#### SQS + Lambda
```
Producer → SQS Queue → Lambda → Process Message
```
- Asynchronous processing
- Decoupling systems
- Batch processing

## Security Considerations

### Access Control
1. **IAM Roles**: Function execution role with minimum permissions
2. **Resource-Based Policies**: Control who can invoke functions
3. **Cross-Account Access**: Secure function sharing
4. **API Gateway**: Authentication and authorization

### Data Protection
1. **Encryption at Rest**: Environment variables encryption
2. **Encryption in Transit**: HTTPS/TLS for all communications
3. **AWS KMS**: Key management for encryption
4. **VPC**: Network isolation for sensitive workloads

### Monitoring and Auditing
1. **CloudTrail**: API call logging
2. **CloudWatch Logs**: Function execution logs
3. **X-Ray**: Distributed tracing
4. **VPC Flow Logs**: Network traffic monitoring

### Security Best Practices
1. **Input Validation**: Sanitize all inputs
2. **Output Encoding**: Prevent injection attacks
3. **Secret Management**: Use AWS Secrets Manager
4. **Network Security**: Use VPC when accessing private resources
5. **Code Security**: Regular vulnerability scanning

## Monitoring and Troubleshooting

### CloudWatch Metrics

#### Standard Metrics
- **Invocations**: Number of function invocations
- **Duration**: Execution time in milliseconds
- **Errors**: Number of invocations that resulted in errors
- **Throttles**: Number of throttled invocations
- **DeadLetterErrors**: Failed async invocations sent to DLQ
- **ConcurrentExecutions**: Number of concurrent executions

#### Custom Metrics
```python
import boto3
cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp/Lambda',
    MetricData=[
        {
            'MetricName': 'ProcessedRecords',
            'Value': record_count,
            'Unit': 'Count'
        }
    ]
)
```

### Logging Best Practices

#### Structured Logging
```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(json.dumps({
        'request_id': context.aws_request_id,
        'event_type': event.get('Records', [{}])[0].get('eventName'),
        'source': event.get('Records', [{}])[0].get('eventSource')
    }))
```

#### Error Handling and Logging
```python
import traceback

def lambda_handler(event, context):
    try:
        # Function logic
        result = process_event(event)
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
    except Exception as e:
        logger.error(f"Error processing event: {str(e)}")
        logger.error(traceback.format_exc())
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Internal server error'})
        }
```

### Common Troubleshooting Scenarios

1. **Cold Start Issues**
   - Symptoms: High latency for first requests
   - Solutions: Provisioned concurrency, optimize package size

2. **Memory Issues**
   - Symptoms: Out of memory errors
   - Solutions: Increase memory allocation, optimize code

3. **Timeout Issues**
   - Symptoms: Function timeout errors
   - Solutions: Increase timeout, optimize code performance

4. **Permission Issues**
   - Symptoms: Access denied errors
   - Solutions: Check IAM roles and policies

5. **Concurrent Execution Limits**
   - Symptoms: Throttling errors
   - Solutions: Request limit increase, implement backoff

6. **VPC Configuration Issues**
   - Symptoms: Cannot access resources
   - Solutions: Check security groups, route tables, NAT gateway

## Exam-Specific Tips and Common Scenarios

### Solutions Architect Associate (SAA-C03)
- **Serverless Architectures**: Lambda + API Gateway + DynamoDB
- **Event-Driven Processing**: S3 triggers, DynamoDB streams
- **Cost Optimization**: Right-sizing and performance tuning
- **Integration Patterns**: Common trigger and destination scenarios

### Solutions Architect Professional (SAP-C02)
- **Complex Workflows**: Step Functions orchestration
- **Multi-Region Deployments**: Cross-region replication patterns
- **Performance Optimization**: Cold start mitigation strategies
- **Hybrid Architectures**: On-premises integration patterns

### Developer Associate (DVA-C02)
- **API Development**: API Gateway integration patterns
- **Debugging**: CloudWatch Logs and X-Ray tracing
- **Deployment**: Blue/green deployments and canary releases
- **Error Handling**: Dead letter queues and retry logic

### SysOps Administrator (SOA-C02)
- **Monitoring Setup**: CloudWatch alarms and dashboards
- **Performance Tuning**: Memory and timeout optimization
- **Security Configuration**: IAM roles and VPC setup
- **Troubleshooting**: Common error scenarios

### Common Exam Scenarios

1. **Scenario**: Process uploaded files automatically
   **Solution**: S3 bucket notification → Lambda function

2. **Scenario**: Build serverless REST API
   **Solution**: API Gateway → Lambda → DynamoDB

3. **Scenario**: Real-time data processing
   **Solution**: Kinesis → Lambda → ElasticSearch

4. **Scenario**: Scheduled maintenance tasks
   **Solution**: CloudWatch Events → Lambda

5. **Scenario**: Cross-service communication
   **Solution**: SQS → Lambda → SNS

## Hands-on Examples and CLI Commands

### Basic Function Management

```bash
# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.9 \
  --role arn:aws:iam::account:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --description "My Lambda function"

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Update function configuration
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 300 \
  --memory-size 256

# Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  response.json

# List functions
aws lambda list-functions

# Get function configuration
aws lambda get-function-configuration \
  --function-name my-function

# Delete function
aws lambda delete-function \
  --function-name my-function
```

### Environment Variables

```bash
# Update environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables='{
    "ENV": "production",
    "DB_HOST": "rds-endpoint.amazonaws.com",
    "API_KEY": "encrypted-key"
  }'

# Get environment variables
aws lambda get-function-configuration \
  --function-name my-function \
  --query Environment.Variables
```

### Versioning and Aliases

```bash
# Publish version
aws lambda publish-version \
  --function-name my-function \
  --description "Version 1.0"

# Create alias
aws lambda create-alias \
  --function-name my-function \
  --name PROD \
  --function-version 1

# Update alias
aws lambda update-alias \
  --function-name my-function \
  --name PROD \
  --function-version 2

# List versions
aws lambda list-versions-by-function \
  --function-name my-function

# List aliases
aws lambda list-aliases \
  --function-name my-function
```

### Event Source Mappings

```bash
# Create SQS event source mapping
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:sqs:region:account:queue-name \
  --batch-size 10

# Create DynamoDB stream mapping
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:dynamodb:region:account:table/table-name/stream/timestamp \
  --starting-position LATEST

# List event source mappings
aws lambda list-event-source-mappings \
  --function-name my-function

# Delete event source mapping
aws lambda delete-event-source-mapping \
  --uuid 12345678-1234-1234-1234-123456789012
```

### Monitoring and Logging

```bash
# Get function metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=my-function \
  --statistics Sum \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 3600

# Get log streams
aws logs describe-log-streams \
  --log-group-name /aws/lambda/my-function

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name stream-name
```

### Concurrency Management

```bash
# Set reserved concurrency
aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier PROD \
  --provisioned-concurrency-limit 100

# Get concurrency configuration
aws lambda get-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier PROD

# Delete reserved concurrency
aws lambda delete-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier PROD
```

### Sample Python Lambda Functions

#### Basic Handler
```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(f"Received event: {json.dumps(event)}")

    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Hello from Lambda!',
            'request_id': context.aws_request_id
        })
    }
```

#### S3 Event Processing
```python
import json
import boto3
from urllib.parse import unquote_plus

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = unquote_plus(record['s3']['object']['key'])

        try:
            # Process the S3 object
            response = s3_client.get_object(Bucket=bucket, Key=key)
            content = response['Body'].read()

            # Your processing logic here
            print(f"Processed {key} from {bucket}")

        except Exception as e:
            print(f"Error processing {key}: {str(e)}")
            raise e

    return {'statusCode': 200}
```

#### API Gateway Integration
```python
import json
import boto3

def lambda_handler(event, context):
    # Extract request data
    http_method = event['httpMethod']
    path = event['path']
    query_params = event.get('queryStringParameters', {})
    body = json.loads(event.get('body', '{}'))

    # Process request
    if http_method == 'GET':
        result = handle_get(query_params)
    elif http_method == 'POST':
        result = handle_post(body)
    else:
        return {
            'statusCode': 405,
            'body': json.dumps({'error': 'Method not allowed'})
        }

    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(result)
    }

def handle_get(params):
    return {'message': 'GET request processed', 'params': params}

def handle_post(body):
    return {'message': 'POST request processed', 'received': body}
```

This comprehensive Lambda documentation provides detailed coverage for all AWS certification paths, including practical examples and real-world scenarios.