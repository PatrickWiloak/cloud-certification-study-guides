# Domain 4: Troubleshooting and Optimization (18%)

## Overview
This domain covers assisting in root cause analysis, instrumenting code for observability, and optimizing applications on AWS. Understanding monitoring, debugging, and performance optimization is essential.

## Amazon CloudWatch

**[📖 Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)** - Monitor AWS resources and applications in real-time

### CloudWatch Metrics

#### Standard Metrics
- **EC2**: CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps
- **Lambda**: Invocations, Duration, Errors, Throttles, ConcurrentExecutions
- **DynamoDB**: ConsumedReadCapacityUnits, ConsumedWriteCapacityUnits, UserErrors
- **API Gateway**: Count, Latency, 4XXError, 5XXError
- **SQS**: ApproximateNumberOfMessagesVisible, ApproximateAgeOfOldestMessage

#### Custom Metrics

**[📖 Publishing Custom Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/publishingMetrics.html)** - Publish your own metrics to CloudWatch
```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

# Put single metric
cloudwatch.put_metric_data(
    Namespace='MyApplication',
    MetricData=[
        {
            'MetricName': 'ProcessingTime',
            'Value': 125.5,
            'Unit': 'Milliseconds',
            'Timestamp': datetime.utcnow(),
            'Dimensions': [
                {'Name': 'Environment', 'Value': 'Production'},
                {'Name': 'Service', 'Value': 'OrderProcessing'}
            ]
        }
    ]
)

# Put multiple metrics (batch)
metrics = []
for i in range(10):
    metrics.append({
        'MetricName': 'RequestCount',
        'Value': i,
        'Unit': 'Count',
        'Timestamp': datetime.utcnow()
    })

cloudwatch.put_metric_data(
    Namespace='MyApplication',
    MetricData=metrics
)

# High-resolution metrics (1-second intervals)
cloudwatch.put_metric_data(
    Namespace='MyApplication',
    MetricData=[{
        'MetricName': 'HighResMetric',
        'Value': 100,
        'Unit': 'Count',
        'StorageResolution': 1  # 1 second
    }]
)
```

### CloudWatch Logs

**[📖 CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)** - Monitor, store, and access log files from AWS resources

#### Log Groups and Streams
```python
import boto3
import time

logs = boto3.client('logs')

# Create log group
logs.create_log_group(logGroupName='/aws/lambda/my-function')

# Create log stream
log_stream_name = f'stream-{int(time.time())}'
logs.create_log_stream(
    logGroupName='/aws/lambda/my-function',
    logStreamName=log_stream_name
)

# Put log events
logs.put_log_events(
    logGroupName='/aws/lambda/my-function',
    logStreamName=log_stream_name,
    logEvents=[
        {
            'message': 'Application started',
            'timestamp': int(time.time() * 1000)
        },
        {
            'message': 'Processing request',
            'timestamp': int(time.time() * 1000)
        }
    ]
)
```

#### Structured Logging
```python
import json
import logging

# Configure structured logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    # Structured log entry
    log_data = {
        'level': 'INFO',
        'message': 'Processing order',
        'order_id': '12345',
        'user_id': 'user-abc',
        'amount': 99.99,
        'timestamp': time.time()
    }
    logger.info(json.dumps(log_data))

    try:
        # Business logic
        process_order(event)
    except Exception as e:
        error_log = {
            'level': 'ERROR',
            'message': 'Order processing failed',
            'error': str(e),
            'order_id': event.get('order_id'),
            'stack_trace': traceback.format_exc()
        }
        logger.error(json.dumps(error_log))
        raise
```

### CloudWatch Logs Insights

**[📖 CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)** - Analyze log data with interactive queries

#### Query Examples
```sql
-- Find errors in Lambda logs
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100

-- Calculate average duration
fields @duration
| stats avg(@duration) as avg_duration, max(@duration) as max_duration by bin(5m)

-- Parse JSON logs
fields @timestamp, @message
| parse @message /userId: "(?<user>[^"]+)"/
| filter ispresent(user)
| stats count() by user

-- Find slow requests
fields @timestamp, @message, @duration
| filter @duration > 1000
| sort @duration desc

-- Error rate by time
fields @timestamp, @message
| filter @message like /ERROR|WARN/
| stats count() as error_count by bin(1h)

-- Analyze Lambda cold starts
fields @type, @duration, @initDuration
| filter @type = "REPORT"
| stats avg(@duration) as avg_duration,
        avg(@initDuration) as avg_cold_start,
        count(@initDuration) as cold_start_count
```

### CloudWatch Alarms

**[📖 Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)** - Set alarms and receive notifications

#### Creating Alarms
```python
import boto3

cloudwatch = boto3.client('cloudwatch')

# Create metric alarm
cloudwatch.put_metric_alarm(
    AlarmName='HighCPUUtilization',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Period=300,
    Statistic='Average',
    Threshold=80.0,
    ActionsEnabled=True,
    AlarmActions=[
        'arn:aws:sns:us-east-1:123456789012:AlertTopic'
    ],
    AlarmDescription='Alert when CPU exceeds 80%',
    Dimensions=[
        {'Name': 'InstanceId', 'Value': 'i-1234567890abcdef0'}
    ]
)

# Composite alarm (multiple conditions)
cloudwatch.put_composite_alarm(
    AlarmName='ApplicationHealthAlarm',
    AlarmRule='(ALARM(HighErrorRate) OR ALARM(HighLatency)) AND ALARM(LowThroughput)',
    ActionsEnabled=True,
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:CriticalAlerts']
)
```

#### Alarm States
- **OK**: Metric is within threshold
- **ALARM**: Metric has breached threshold
- **INSUFFICIENT_DATA**: Not enough data to evaluate

### CloudWatch Dashboards

```python
# Create dashboard
dashboard_body = {
    "widgets": [
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/Lambda", "Invocations", {"stat": "Sum"}],
                    [".", "Errors", {"stat": "Sum"}],
                    [".", "Duration", {"stat": "Average"}]
                ],
                "period": 300,
                "stat": "Average",
                "region": "us-east-1",
                "title": "Lambda Metrics"
            }
        },
        {
            "type": "log",
            "properties": {
                "query": "fields @timestamp, @message | filter @message like /ERROR/",
                "region": "us-east-1",
                "title": "Error Logs"
            }
        }
    ]
}

cloudwatch.put_dashboard(
    DashboardName='ApplicationDashboard',
    DashboardBody=json.dumps(dashboard_body)
)
```

## AWS X-Ray

**[📖 AWS X-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)** - Analyze and debug distributed applications

### Distributed Tracing

#### X-Ray Concepts
- **Trace**: End-to-end journey of a request
- **Segment**: Work done by a single service
- **Subsegment**: More granular view of work within a segment
- **Annotations**: Key-value pairs for indexing and filtering
- **Metadata**: Additional data not indexed

#### X-Ray SDK Integration

**[📖 X-Ray SDK for Python](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python.html)** - Instrument Python applications with X-Ray

##### Python Example
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all
import boto3

# Patch all supported libraries
patch_all()

@xray_recorder.capture('process_order')
def process_order(order_id):
    # Add annotations (indexed)
    xray_recorder.current_segment().put_annotation('order_id', order_id)
    xray_recorder.current_segment().put_annotation('environment', 'production')

    # Add metadata (not indexed, for context)
    xray_recorder.current_segment().put_metadata('order_details', {
        'items': ['item1', 'item2'],
        'total': 99.99
    })

    # Database query (automatically traced)
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('Orders')
    response = table.get_item(Key={'order_id': order_id})

    # Custom subsegment
    subsegment = xray_recorder.begin_subsegment('validate_payment')
    try:
        validate_payment(order_id)
        subsegment.put_annotation('payment_status', 'success')
    except Exception as e:
        subsegment.put_annotation('payment_status', 'failed')
        subsegment.put_metadata('error', str(e))
        raise
    finally:
        xray_receiver.end_subsegment()

    return response

def lambda_handler(event, context):
    order_id = event['order_id']
    return process_order(order_id)
```

##### Node.js Example
```javascript
const AWSXRay = require('aws-xray-sdk-core');
const AWS = AWSXRay.captureAWS(require('aws-sdk'));

exports.handler = async (event) => {
    const segment = AWSXRay.getSegment();

    // Add annotations
    segment.addAnnotation('order_id', event.order_id);
    segment.addAnnotation('user_id', event.user_id);

    // Add metadata
    segment.addMetadata('request_details', {
        source: event.source,
        timestamp: Date.now()
    });

    // Subsegment for database operation
    const subsegment = segment.addNewSubsegment('database_query');
    try {
        const dynamodb = new AWS.DynamoDB.DocumentClient();
        const result = await dynamodb.get({
            TableName: 'Orders',
            Key: { order_id: event.order_id }
        }).promise();

        subsegment.addAnnotation('query_status', 'success');
        subsegment.close();
        return result;
    } catch (error) {
        subsegment.addError(error);
        subsegment.close();
        throw error;
    }
};
```

#### X-Ray Sampling Rules
```json
{
  "version": 2,
  "rules": [
    {
      "description": "Sample all errors",
      "host": "*",
      "http_method": "*",
      "url_path": "*",
      "fixed_target": 1,
      "rate": 1.0,
      "priority": 1000,
      "service_name": "*",
      "service_type": "*",
      "resource_ARN": "*",
      "attributes": {
        "error": "true"
      }
    },
    {
      "description": "Sample 10% of normal traffic",
      "host": "*",
      "http_method": "*",
      "url_path": "*",
      "fixed_target": 1,
      "rate": 0.1,
      "priority": 5000
    }
  ],
  "default": {
    "fixed_target": 1,
    "rate": 0.01
  }
}
```

### X-Ray Service Map

#### Analyzing Service Dependencies
- **Service Nodes**: Individual services in architecture
- **Edges**: Connections between services
- **Response Time**: Average latency between services
- **Request Rate**: Requests per second
- **Error Rate**: Percentage of failed requests

#### X-Ray Console Queries
```
# Find traces with errors
filter.error = true

# Find slow traces
responsetime > 5

# Find traces for specific user
annotation.user_id = "user-123"

# Complex query
service("my-api") AND http.status = 500 AND responsetime > 2
```

## Performance Optimization

### Lambda Optimization

**[📖 Lambda Performance Optimization](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#function-configuration)** - Optimize Lambda function performance and cost

#### Cold Start Reduction
```python
# Initialize outside handler (reused across invocations)
import boto3

# Initialize clients outside handler
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('MyTable')
s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Handler code runs on each invocation
    # Connection reuse reduces overhead
    response = table.get_item(Key={'id': event['id']})
    return response

# Use provisioned concurrency for critical functions
# Configure via console or CLI:
# aws lambda put-provisioned-concurrency-config \
#     --function-name my-function \
#     --provisioned-concurrent-executions 5
```

#### Memory and CPU Optimization
```python
# Lambda CPU is proportional to memory allocation
# Test different memory configurations for optimal performance/cost

# Example: 1024 MB might be faster AND cheaper than 512 MB
# if execution time is reduced by more than 50%

# Monitor with CloudWatch metrics
cloudwatch.get_metric_statistics(
    Namespace='AWS/Lambda',
    MetricName='Duration',
    Dimensions=[{'Name': 'FunctionName', 'Value': 'my-function'}],
    StartTime=datetime.utcnow() - timedelta(hours=1),
    EndTime=datetime.utcnow(),
    Period=3600,
    Statistics=['Average', 'Maximum']
)
```

#### Lambda Best Practices
```python
# 1. Minimize package size
# Use Lambda Layers for shared dependencies
# Remove unused dependencies

# 2. Optimize imports
import json  # Fast
# import pandas  # Slow for cold starts

# 3. Connection pooling for databases
import pymysql

connection = None

def get_db_connection():
    global connection
    if connection is None or not connection.open:
        connection = pymysql.connect(
            host=os.environ['DB_HOST'],
            user=os.environ['DB_USER'],
            password=os.environ['DB_PASSWORD'],
            database=os.environ['DB_NAME']
        )
    return connection

# 4. Use environment variables for configuration
DB_TABLE = os.environ.get('DB_TABLE', 'default-table')

# 5. Implement proper error handling
def lambda_handler(event, context):
    try:
        result = process_event(event)
        return {'statusCode': 200, 'body': json.dumps(result)}
    except ValueError as e:
        logger.error(f"Validation error: {e}")
        return {'statusCode': 400, 'body': json.dumps({'error': str(e)})}
    except Exception as e:
        logger.error(f"Unexpected error: {e}", exc_info=True)
        return {'statusCode': 500, 'body': json.dumps({'error': 'Internal server error'})}
```

### DynamoDB Optimization

**[📖 DynamoDB Performance Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)** - Design for optimal performance and cost

#### Partition Key Design
```python
# GOOD: Evenly distributed partition keys
# user_id as partition key (many unique values)
table.put_item(Item={'user_id': 'user-12345', 'data': 'value'})

# BAD: Hot partition (status has few values)
# Using 'status' as partition key causes hot partitions
table.put_item(Item={'status': 'active', 'user_id': 'user-12345'})

# BETTER: Composite key with good distribution
# status#user_id as partition key
table.put_item(Item={'pk': 'active#user-12345', 'data': 'value'})
```

#### Query Optimization
```python
# Use Query instead of Scan
# GOOD: Query with partition key
response = table.query(
    KeyConditionExpression='user_id = :uid',
    ExpressionAttributeValues={':uid': 'user-123'}
)

# BAD: Full table scan (expensive)
response = table.scan(
    FilterExpression='user_id = :uid',
    ExpressionAttributeValues={':uid': 'user-123'}
)

# Use projection expression to retrieve only needed attributes
response = table.query(
    KeyConditionExpression='user_id = :uid',
    ProjectionExpression='user_id, email, name',
    ExpressionAttributeValues={':uid': 'user-123'}
)
```

#### Batch Operations
```python
# Batch write (up to 25 items)
with table.batch_writer() as batch:
    for i in range(100):
        batch.put_item(Item={'id': str(i), 'data': f'value-{i}'})

# Batch get (up to 100 items)
response = dynamodb.batch_get_item(
    RequestItems={
        'MyTable': {
            'Keys': [{'id': str(i)} for i in range(10)]
        }
    }
)
```

#### DynamoDB DAX (Caching)

**[📖 DynamoDB Accelerator (DAX)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)** - In-memory caching for DynamoDB
```python
import amazondax

# DAX client for microsecond latency
dax = amazondax.AmazonDaxClient(
    endpoint_url='my-cluster.dax-clusters.us-east-1.amazonaws.com:8111'
)

# Use DAX client like DynamoDB client
response = dax.get_item(
    TableName='MyTable',
    Key={'id': {'S': '123'}}
)
```

### API Gateway Optimization

**[📖 API Gateway Caching](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-caching.html)** - Enable API caching to improve latency and reduce backend load

#### Caching
```python
# Enable caching at stage level (via console or CloudFormation)
# Cache TTL: 0-3600 seconds
# Cache size: 0.5GB - 237GB

# Cache key parameters
# GET /items?category=electronics&page=2
# Both 'category' and 'page' should be cache key parameters
```

#### Request/Response Optimization
```json
// Request validation (saves Lambda invocations)
{
  "type": "object",
  "required": ["name", "email"],
  "properties": {
    "name": {"type": "string", "minLength": 1},
    "email": {"type": "string", "pattern": "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"},
    "age": {"type": "integer", "minimum": 0}
  }
}

// Response compression (reduce transfer time)
{
  "minimumCompressionSize": 1024
}
```

### S3 Optimization

#### Transfer Optimization
```python
import boto3
from boto3.s3.transfer import TransferConfig

# Multipart upload configuration
config = TransferConfig(
    multipart_threshold=1024 * 25,  # 25 MB
    max_concurrency=10,
    multipart_chunksize=1024 * 25,
    use_threads=True
)

s3 = boto3.client('s3')
s3.upload_file(
    'large-file.zip',
    'my-bucket',
    'uploads/large-file.zip',
    Config=config
)

# S3 Transfer Acceleration (for long distances)
s3_accelerate = boto3.client(
    's3',
    config=Config(s3={'use_accelerate_endpoint': True})
)

# CloudFront for frequently accessed objects
# Configure CloudFront distribution with S3 as origin
```

## Root Cause Analysis

### Debugging Strategies

#### Lambda Debugging
```python
import logging
import json

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)

def lambda_handler(event, context):
    # Log incoming event
    logger.debug(f"Event: {json.dumps(event)}")
    logger.info(f"Function version: {context.function_version}")
    logger.info(f"Memory limit: {context.memory_limit_in_mb} MB")
    logger.info(f"Time remaining: {context.get_remaining_time_in_millis()} ms")

    try:
        result = process_event(event)
        logger.info(f"Processing successful: {result}")
        return result
    except Exception as e:
        logger.error(f"Error processing event: {str(e)}", exc_info=True)
        # Log event that caused error
        logger.error(f"Failed event: {json.dumps(event)}")
        raise

# Enable active tracing in Lambda configuration
# X-Ray will automatically trace AWS SDK calls
```

#### Common Issues and Solutions

##### Timeout Issues
```python
# Monitor execution time
import time

start_time = time.time()
# ... process logic ...
elapsed_time = time.time() - start_time

if elapsed_time > context.get_remaining_time_in_millis() / 1000 - 5:
    # Approaching timeout, log warning
    logger.warning(f"Approaching timeout: {elapsed_time}s elapsed")
```

##### Memory Issues
```python
# Monitor memory usage
import resource

def get_memory_usage():
    return resource.getrusage(resource.RUSAGE_SELF).ru_maxrss / 1024

logger.info(f"Memory usage: {get_memory_usage()} MB")
```

##### Throttling Issues
```python
from botocore.exceptions import ClientError
import time

def retry_with_backoff(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except ClientError as e:
            if e.response['Error']['Code'] == 'ProvisionedThroughputExceededException':
                if attempt < max_retries - 1:
                    wait_time = (2 ** attempt) + (random.randint(0, 1000) / 1000.0)
                    logger.warning(f"Throttled, retrying in {wait_time}s")
                    time.sleep(wait_time)
                else:
                    raise
            else:
                raise
```

## Cost Optimization

### Lambda Cost Optimization
```python
# 1. Right-size memory allocation
# Monitor duration and memory used
# Optimize for best duration/cost ratio

# 2. Use ARM/Graviton2 processors
# Up to 34% better price performance

# 3. Reduce execution time
# - Optimize code
# - Use compiled languages (Go, Rust) for performance-critical paths
# - Implement caching

# 4. Use appropriate timeout
# Don't set unnecessarily high timeouts
```

### DynamoDB Cost Optimization
```python
# 1. Use on-demand billing for unpredictable workloads
table = dynamodb.create_table(
    TableName='MyTable',
    BillingMode='PAY_PER_REQUEST',
    # ...
)

# 2. Use provisioned capacity for predictable workloads
# with auto-scaling

# 3. Use projection expressions to reduce read costs
response = table.query(
    KeyConditionExpression='pk = :pk',
    ProjectionExpression='id, name, email',  # Only retrieve needed fields
    ExpressionAttributeValues={':pk': 'user-123'}
)

# 4. Use DynamoDB Streams instead of polling
```

## Study Tips

1. **CloudWatch Mastery**: Practice creating metrics, logs, alarms, and dashboards
2. **X-Ray Integration**: Implement distributed tracing in sample applications
3. **Performance Testing**: Use AWS Lambda Power Tuning tool
4. **Log Analysis**: Practice CloudWatch Logs Insights queries
5. **Troubleshooting**: Build scenarios with common errors and debug them
6. **Optimization**: Measure before and after optimization changes

## Common Exam Scenarios

- Debugging Lambda timeout or memory issues
- Implementing distributed tracing for microservices
- Creating CloudWatch alarms for application monitoring
- Optimizing DynamoDB query performance and costs
- Analyzing logs with CloudWatch Logs Insights
- Implementing custom metrics for business KPIs
- Root cause analysis using X-Ray service map

## CLI Quick Reference

```bash
# CloudWatch Logs
aws logs create-log-group --log-group-name /aws/lambda/my-function
aws logs tail /aws/lambda/my-function --follow
aws logs filter-log-events --log-group-name /aws/lambda/my-function --filter-pattern "ERROR"

# CloudWatch Metrics
aws cloudwatch put-metric-data --namespace MyApp --metric-name RequestCount --value 1
aws cloudwatch get-metric-statistics --namespace AWS/Lambda --metric-name Duration

# X-Ray
aws xray get-trace-summaries --start-time 2024-01-01T00:00:00 --end-time 2024-01-02T00:00:00
aws xray get-trace-graph --trace-ids trace-id

# Lambda
aws lambda get-function-configuration --function-name my-function
aws lambda invoke --function-name my-function --log-type Tail output.json
```
