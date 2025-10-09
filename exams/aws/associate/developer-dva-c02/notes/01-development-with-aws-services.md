# Domain 1: Development with AWS Services (32%)

## Overview
This domain covers developing code for applications hosted on AWS, including Lambda functions, API integrations, and data store operations. It represents the largest portion of the exam content.

## Key AWS Services

### AWS Lambda

#### Core Concepts
- **Execution Model**: Event-driven, serverless compute service
- **Runtime Support**: Node.js, Python, Java, .NET, Go, Ruby, Custom Runtime
- **Invocation Types**: Synchronous, Asynchronous, Poll-based
- **Cold Starts**: Initial invocation delay when creating new execution environment
- **Warm Containers**: Reused execution environments for subsequent invocations

#### Lambda Configuration
```bash
# Create Lambda function
aws lambda create-function \
    --function-name my-function \
    --runtime python3.9 \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip

# Update function code
aws lambda update-function-code \
    --function-name my-function \
    --zip-file fileb://function.zip

# Update function configuration
aws lambda update-function-configuration \
    --function-name my-function \
    --timeout 30 \
    --memory-size 512
```

#### Lambda Best Practices
1. **Minimize Package Size**: Smaller packages reduce cold start time
2. **Reuse Connections**: Initialize connections outside handler
3. **Use Environment Variables**: For configuration management
4. **Implement Proper Error Handling**: Use try-catch blocks
5. **Leverage Lambda Layers**: Share code across multiple functions
6. **Set Appropriate Timeouts**: Balance between completion time and cost

#### Lambda Event Sources
- **S3**: Object creation, deletion, modification events
- **DynamoDB Streams**: Table change capture
- **API Gateway**: HTTP/REST API requests
- **SNS/SQS**: Message queue and pub/sub integration
- **CloudWatch Events**: Scheduled and event-based triggers
- **Kinesis**: Real-time data stream processing

### Amazon API Gateway

#### API Types
- **REST API**: Full-featured API with caching, throttling, authorization
- **HTTP API**: Lightweight, lower cost, faster performance
- **WebSocket API**: Two-way communication for real-time applications

#### Authentication & Authorization
```javascript
// Lambda Authorizer Example
exports.handler = async (event) => {
    const token = event.authorizationToken;

    // Validate token
    if (token === 'valid-token') {
        return {
            principalId: 'user123',
            policyDocument: {
                Version: '2012-10-17',
                Statement: [{
                    Action: 'execute-api:Invoke',
                    Effect: 'Allow',
                    Resource: event.methodArn
                }]
            }
        };
    }
    throw new Error('Unauthorized');
};
```

#### API Gateway Features
- **Request Validation**: Schema-based request validation
- **Request/Response Transformation**: Modify data using mapping templates
- **Caching**: Reduce backend calls with endpoint caching
- **Throttling**: Rate limiting per API key or globally
- **CORS**: Cross-origin resource sharing configuration
- **Usage Plans**: API key management and throttling quotas

### AWS SDK Best Practices

#### Credential Management
```python
# Use IAM roles (preferred for EC2/Lambda)
import boto3
client = boto3.client('s3')

# Explicit credentials (avoid hardcoding)
client = boto3.client(
    's3',
    aws_access_key_id='ACCESS_KEY',
    aws_secret_access_key='SECRET_KEY',
    region_name='us-east-1'
)

# Use environment variables or credential files
```

#### Error Handling and Retries
```python
import boto3
from botocore.exceptions import ClientError
from botocore.config import Config

# Configure exponential backoff
config = Config(
    retries={
        'max_attempts': 3,
        'mode': 'adaptive'
    }
)

client = boto3.client('dynamodb', config=config)

try:
    response = client.put_item(
        TableName='MyTable',
        Item={'id': {'S': '123'}}
    )
except ClientError as e:
    if e.response['Error']['Code'] == 'ResourceNotFoundException':
        print("Table not found")
    elif e.response['Error']['Code'] == 'ProvisionedThroughputExceededException':
        print("Throttled - retry with backoff")
    else:
        raise
```

## Data Stores for Applications

### Amazon DynamoDB

#### Table Design
- **Partition Key**: Determines data distribution across partitions
- **Sort Key**: Optional, enables range queries within partition
- **Composite Keys**: Partition key + sort key for hierarchical data
- **Secondary Indexes**: GSI (Global) and LSI (Local) for alternate query patterns

#### DynamoDB Operations
```python
import boto3
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

# Put item
table.put_item(Item={'userId': '123', 'name': 'John', 'email': 'john@example.com'})

# Get item
response = table.get_item(Key={'userId': '123'})
item = response['Item']

# Query (requires partition key)
response = table.query(
    KeyConditionExpression='userId = :uid',
    ExpressionAttributeValues={':uid': '123'}
)

# Scan (full table scan - avoid in production)
response = table.scan(
    FilterExpression='age > :age',
    ExpressionAttributeValues={':age': 25}
)

# Batch operations
with table.batch_writer() as batch:
    for i in range(100):
        batch.put_item(Item={'userId': str(i), 'name': f'User{i}'})

# Transactions
client = boto3.client('dynamodb')
client.transact_write_items(
    TransactItems=[
        {'Put': {'TableName': 'Users', 'Item': {'userId': {'S': '123'}}}},
        {'Update': {'TableName': 'Orders', 'Key': {'orderId': {'S': '456'}}}},
    ]
)
```

#### DynamoDB Streams
```python
# Enable streams on table
client.update_table(
    TableName='MyTable',
    StreamSpecification={
        'StreamEnabled': True,
        'StreamViewType': 'NEW_AND_OLD_IMAGES'
    }
)

# Lambda trigger for DynamoDB Streams
def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            new_image = record['dynamodb']['NewImage']
            # Process new item
        elif record['eventName'] == 'MODIFY':
            old_image = record['dynamodb']['OldImage']
            new_image = record['dynamodb']['NewImage']
            # Process update
```

### Amazon S3

#### S3 Operations via SDK
```python
import boto3
s3 = boto3.client('s3')

# Upload file
s3.upload_file('local-file.txt', 'my-bucket', 'remote-file.txt')

# Download file
s3.download_file('my-bucket', 'remote-file.txt', 'downloaded-file.txt')

# Put object with metadata
s3.put_object(
    Bucket='my-bucket',
    Key='data.json',
    Body=json.dumps(data),
    ContentType='application/json',
    Metadata={'user': 'john', 'version': '1.0'}
)

# Generate presigned URL
url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'file.txt'},
    ExpiresIn=3600  # 1 hour
)

# Multipart upload for large files
mpu = s3.create_multipart_upload(Bucket='my-bucket', Key='large-file.zip')
upload_id = mpu['UploadId']
```

#### S3 Event Notifications
```json
{
  "Records": [
    {
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "my-bucket"
        },
        "object": {
          "key": "uploads/image.jpg",
          "size": 1024
        }
      }
    }
  ]
}
```

### Amazon RDS

#### Connection Best Practices
```python
import pymysql
import os

# Connection pooling for Lambda
connection = None

def get_connection():
    global connection
    if connection is None or not connection.open:
        connection = pymysql.connect(
            host=os.environ['DB_HOST'],
            user=os.environ['DB_USER'],
            password=os.environ['DB_PASSWORD'],
            database=os.environ['DB_NAME'],
            connect_timeout=5
        )
    return connection

def lambda_handler(event, context):
    conn = get_connection()
    with conn.cursor() as cursor:
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        result = cursor.fetchone()
    return result
```

#### RDS Proxy for Serverless
- **Connection Pooling**: Manages database connections efficiently
- **IAM Authentication**: Database authentication using IAM
- **Failover Support**: Automatic failover to read replicas
- **Reduces Cold Starts**: Maintains connection pool

## Application Integration Services

### Amazon SQS

#### Queue Types
- **Standard Queue**: At-least-once delivery, best-effort ordering
- **FIFO Queue**: Exactly-once processing, strict ordering

#### SQS Operations
```python
import boto3
sqs = boto3.client('sqs')

# Send message
response = sqs.send_message(
    QueueUrl='https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue',
    MessageBody='Hello World',
    MessageAttributes={
        'Priority': {'StringValue': 'High', 'DataType': 'String'}
    }
)

# Receive messages
messages = sqs.receive_message(
    QueueUrl=queue_url,
    MaxNumberOfMessages=10,
    WaitTimeSeconds=20,  # Long polling
    VisibilityTimeout=30
)

# Delete message after processing
sqs.delete_message(
    QueueUrl=queue_url,
    ReceiptHandle=message['ReceiptHandle']
)

# Send batch messages
entries = [
    {'Id': '1', 'MessageBody': 'Message 1'},
    {'Id': '2', 'MessageBody': 'Message 2'}
]
sqs.send_message_batch(QueueUrl=queue_url, Entries=entries)
```

### Amazon SNS

#### SNS Publishing
```python
import boto3
sns = boto3.client('sns')

# Publish to topic
response = sns.publish(
    TopicArn='arn:aws:sns:us-east-1:123456789012:MyTopic',
    Message='Hello World',
    Subject='Notification',
    MessageAttributes={
        'type': {'DataType': 'String', 'StringValue': 'alert'}
    }
)

# Fan-out pattern: SNS → multiple SQS queues
# Subscribe SQS queues to SNS topic for parallel processing
```

## Common Development Patterns

### Serverless REST API Pattern
```
API Gateway → Lambda → DynamoDB
- API Gateway handles HTTP requests
- Lambda processes business logic
- DynamoDB stores data
```

### Event-Driven Processing Pattern
```
S3 Upload → Lambda → Processing → S3/DynamoDB
- S3 event triggers Lambda
- Lambda processes file
- Results stored in S3 or DynamoDB
```

### Microservices Pattern
```
API Gateway → Lambda → SQS → Lambda Workers
- Asynchronous processing
- Decoupled services
- Scalable architecture
```

## Study Tips

1. **Hands-on Practice**: Build serverless applications using Lambda + API Gateway
2. **Understand Limits**: Know Lambda timeout (15 min), memory (10GB), payload limits
3. **Master DynamoDB**: Understand partition keys, indexes, and query patterns
4. **SDK Proficiency**: Practice error handling, pagination, and async operations
5. **Integration Patterns**: Study event-driven and microservices architectures
6. **Performance**: Learn Lambda optimization, connection pooling, caching

## Common Exam Scenarios

- Designing serverless applications with Lambda and API Gateway
- Implementing authentication with Cognito and Lambda authorizers
- Optimizing Lambda functions for performance and cost
- Choosing between DynamoDB and RDS for data persistence
- Handling S3 events and file processing workflows
- Implementing message queuing with SQS for decoupled architectures
- Using SDK with proper error handling and retry logic

## CLI Quick Reference

```bash
# Lambda
aws lambda invoke --function-name my-function output.json
aws lambda list-functions
aws lambda get-function --function-name my-function

# DynamoDB
aws dynamodb put-item --table-name Users --item '{"id":{"S":"123"}}'
aws dynamodb get-item --table-name Users --key '{"id":{"S":"123"}}'
aws dynamodb query --table-name Users --key-condition-expression "id = :id"

# S3
aws s3 cp file.txt s3://my-bucket/
aws s3 ls s3://my-bucket/
aws s3 sync ./local-dir s3://my-bucket/prefix/

# SQS
aws sqs send-message --queue-url URL --message-body "Hello"
aws sqs receive-message --queue-url URL --wait-time-seconds 20
```
