# Amazon DynamoDB

## Service Overview and Purpose

Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. DynamoDB lets you offload the administrative burdens of operating and scaling a distributed database, so you don't have to worry about hardware provisioning, setup and configuration, replication, software patching, or cluster scaling.

### Key Characteristics
- **Fully Managed**: No infrastructure management required
- **Serverless**: Automatic scaling and maintenance
- **High Performance**: Single-digit millisecond latency
- **Global Tables**: Multi-region, multi-active replication
- **ACID Transactions**: Support for complex operations

## Key Features and Capabilities

### Core Features

#### Table Structure
- **Tables**: Primary data containers
- **Items**: Individual records (similar to rows)
- **Attributes**: Data elements within items (similar to columns)
- **Primary Keys**: Partition key (required) and sort key (optional)
- **Secondary Indexes**: GSI (Global) and LSI (Local) for query flexibility

#### Capacity Modes
1. **On-Demand**
   - Pay-per-request pricing
   - Automatic scaling
   - No capacity planning required
   - Ideal for unpredictable workloads

2. **Provisioned**
   - Pre-specified read/write capacity
   - Auto Scaling available
   - Reserved capacity discounts
   - Predictable performance and costs

#### Data Types
- **Scalar**: String, Number, Binary, Boolean, Null
- **Document**: List, Map
- **Set**: String Set, Number Set, Binary Set

### Advanced Features

#### Global Tables
- **Multi-Region Replication**: Automatic replication across regions
- **Multi-Active**: Read and write from any region
- **Eventual Consistency**: Cross-region consistency model
- **Conflict Resolution**: Last writer wins

#### DynamoDB Streams
- **Change Data Capture**: Real-time stream of data changes
- **Event-Driven Architecture**: Trigger Lambda functions
- **Retention**: 24-hour retention period
- **Shard Processing**: Parallel processing of changes

#### Point-in-Time Recovery (PITR)
- **Continuous Backups**: Automatic backups for 35 days
- **Granular Recovery**: Restore to any point in time
- **No Performance Impact**: Doesn't affect table performance

#### DynamoDB Accelerator (DAX)
- **In-Memory Cache**: Microsecond latency
- **Write-Through Cache**: Automatic cache management
- **Multi-AZ**: High availability caching
- **Encryption**: At rest and in transit

## Use Cases and Scenarios

### Primary Use Cases

1. **Web and Mobile Applications**
   - User profiles and preferences
   - Shopping carts and session data
   - Gaming leaderboards and player data
   - Chat and messaging applications

2. **IoT and Time-Series Data**
   - Sensor data collection
   - Device telemetry
   - Log aggregation
   - Real-time analytics

3. **Real-Time Applications**
   - Ad serving and bidding
   - Fraud detection
   - Recommendation engines
   - Live voting systems

4. **Content Management**
   - Media metadata
   - User-generated content
   - Content catalogs
   - Social media feeds

### Architecture Patterns

1. **Serverless Architecture**
   - API Gateway + Lambda + DynamoDB
   - Event-driven processing
   - Auto-scaling components
   - Cost-effective for variable workloads

2. **Microservices Data Layer**
   - Database per service
   - Independent scaling
   - Service isolation
   - Event-driven communication

3. **Real-Time Analytics**
   - DynamoDB Streams + Lambda
   - Kinesis integration
   - Real-time dashboards
   - Anomaly detection

## Pricing Models and Cost Optimization

### Pricing Components

#### On-Demand Pricing
- **Read Requests**: $0.25 per million read request units
- **Write Requests**: $1.25 per million write request units
- **Storage**: $0.25 per GB-month
- **No minimum charges**

#### Provisioned Pricing
- **Read Capacity Units (RCU)**: $0.13 per RCU-hour
- **Write Capacity Units (WCU)**: $0.65 per WCU-hour
- **Storage**: $0.25 per GB-month
- **Reserved Capacity**: Up to 76% savings

### Cost Optimization Strategies

1. **Choose Appropriate Capacity Mode**
   ```bash
   # Switch to on-demand
   aws dynamodb update-table \
     --table-name MyTable \
     --billing-mode-summary BillingMode=ON_DEMAND

   # Monitor cost metrics
   aws cloudwatch get-metric-statistics \
     --namespace AWS/DynamoDB \
     --metric-name ConsumedReadCapacityUnits \
     --dimensions Name=TableName,Value=MyTable \
     --start-time 2023-01-01T00:00:00Z \
     --end-time 2023-01-02T00:00:00Z \
     --period 3600 \
     --statistics Sum
   ```

2. **Optimize Data Model**
   - Use composite primary keys
   - Minimize item size
   - Use sparse indexes
   - Compress large attributes

3. **Efficient Querying**
   - Use Query instead of Scan
   - Implement pagination
   - Use projection expressions
   - Leverage indexes appropriately

## Configuration Details and Best Practices

### Table Creation

```bash
# Create table with GSI and LSI
aws dynamodb create-table \
  --table-name ProductCatalog \
  --attribute-definitions \
    AttributeName=ProductId,AttributeType=S \
    AttributeName=Category,AttributeType=S \
    AttributeName=SubCategory,AttributeType=S \
    AttributeName=Price,AttributeType=N \
  --key-schema \
    AttributeName=ProductId,KeyType=HASH \
    AttributeName=Category,KeyType=RANGE \
  --local-secondary-indexes \
    IndexName=CategoryIndex,KeySchema='[{AttributeName=ProductId,KeyType=HASH},{AttributeName=SubCategory,KeyType=RANGE}]',Projection='{ProjectionType=ALL}' \
  --global-secondary-indexes \
    IndexName=PriceIndex,KeySchema='[{AttributeName=Category,KeyType=HASH},{AttributeName=Price,KeyType=RANGE}]',Projection='{ProjectionType=ALL}',ProvisionedThroughput='{ReadCapacityUnits=5,WriteCapacityUnits=5}' \
  --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10 \
  --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES \
  --sse-specification Enabled=true,SSEType=KMS,KMSMasterKeyId=alias/dynamodb-encryption-key \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --tags Key=Environment,Value=Production Key=Application,Value=Catalog
```

### Auto Scaling Configuration

```bash
# Enable auto scaling for table
aws application-autoscaling register-scalable-target \
  --service-namespace dynamodb \
  --resource-id table/ProductCatalog \
  --scalable-dimension dynamodb:table:ReadCapacityUnits \
  --min-capacity 5 \
  --max-capacity 100 \
  --role-arn arn:aws:iam::account:role/DynamoDBAutoscaleRole

aws application-autoscaling put-scaling-policy \
  --policy-name ProductCatalogReadScalingPolicy \
  --service-namespace dynamodb \
  --resource-id table/ProductCatalog \
  --scalable-dimension dynamodb:table:ReadCapacityUnits \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "ScaleInCooldown": 60,
    "ScaleOutCooldown": 60,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "DynamoDBReadCapacityUtilization"
    }
  }'
```

### Global Tables Setup

```bash
# Enable global tables
aws dynamodb create-global-table \
  --global-table-name ProductCatalog \
  --replication-group RegionName=us-west-2 RegionName=us-east-1 RegionName=eu-west-1

# Update global table settings
aws dynamodb update-global-table-settings \
  --global-table-name ProductCatalog \
  --global-table-billing-mode PAY_PER_REQUEST \
  --replica-settings-update '[{
    "RegionName": "us-west-2",
    "ReplicaGlobalSecondaryIndexSettingsUpdate": [{
      "IndexName": "PriceIndex",
      "ProvisionedReadCapacityUnits": 10,
      "ProvisionedWriteCapacityUnits": 10
    }]
  }]'
```

### Best Practices

1. **Data Modeling**
   - **Denormalization**: Store related data together
   - **Composite Keys**: Use partition + sort key effectively
   - **Access Patterns**: Design based on query patterns
   - **Hot Partitions**: Avoid concentrated access patterns

2. **Performance Optimization**
   - **Batch Operations**: Use BatchGetItem and BatchWriteItem
   - **Pagination**: Implement proper pagination for large results
   - **Consistent Reads**: Use when necessary, prefer eventually consistent
   - **Projection**: Only retrieve needed attributes

3. **Security**
   - **IAM Policies**: Implement least privilege access
   - **VPC Endpoints**: Use for private access
   - **Encryption**: Enable at rest and in transit
   - **Fine-Grained Access**: Use IAM conditions for item-level access

## Integration with Other AWS Services

### Application Integration

```python
# Python SDK example with best practices
import boto3
from boto3.dynamodb.conditions import Key, Attr
from botocore.exceptions import ClientError

# Create DynamoDB resource
dynamodb = boto3.resource('dynamodb', region_name='us-west-2')
table = dynamodb.Table('ProductCatalog')

# Query with pagination
def query_products_by_category(category, last_evaluated_key=None):
    kwargs = {
        'KeyConditionExpression': Key('Category').eq(category),
        'Limit': 100,
        'Select': 'ALL_ATTRIBUTES'
    }

    if last_evaluated_key:
        kwargs['ExclusiveStartKey'] = last_evaluated_key

    try:
        response = table.query(**kwargs)
        return response['Items'], response.get('LastEvaluatedKey')
    except ClientError as e:
        print(f"Error querying table: {e.response['Error']['Message']}")
        return [], None

# Batch write with error handling
def batch_write_items(items):
    with table.batch_writer() as batch:
        for item in items:
            batch.put_item(Item=item)
```

### Streams and Event Processing

```python
# Lambda function for DynamoDB Streams
import json

def lambda_handler(event, context):
    for record in event['Records']:
        event_name = record['eventName']

        if event_name == 'INSERT':
            handle_new_item(record['dynamodb']['NewImage'])
        elif event_name == 'MODIFY':
            handle_updated_item(
                record['dynamodb']['OldImage'],
                record['dynamodb']['NewImage']
            )
        elif event_name == 'REMOVE':
            handle_deleted_item(record['dynamodb']['OldImage'])

    return {'statusCode': 200}

def handle_new_item(new_image):
    # Process new item
    print(f"New item created: {new_image}")

def handle_updated_item(old_image, new_image):
    # Process item update
    print(f"Item updated from {old_image} to {new_image}")

def handle_deleted_item(old_image):
    # Process item deletion
    print(f"Item deleted: {old_image}")
```

### DAX Integration

```python
# DAX client setup
import boto3
import amazon_dax_client as dax

# Create DAX client
dax_client = dax.Dax([
    'my-dax-cluster.abc123.dax-clusters.us-west-2.amazonaws.com:8111'
])

# Use DAX for high-performance reads
def get_product_with_cache(product_id, category):
    try:
        response = dax_client.get_item(
            TableName='ProductCatalog',
            Key={
                'ProductId': {'S': product_id},
                'Category': {'S': category}
            }
        )
        return response.get('Item')
    except Exception as e:
        print(f"Error getting item from DAX: {e}")
        return None
```

## Security Considerations

### Encryption

```bash
# Enable encryption at rest
aws dynamodb update-table \
  --table-name ProductCatalog \
  --sse-specification Enabled=true,SSEType=KMS,KMSMasterKeyId=alias/dynamodb-key

# Enable encryption in transit (client-side)
# Use HTTPS endpoints and AWS SDK encryption
```

### Access Control

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "dynamodb:GetItem",
      "dynamodb:Query",
      "dynamodb:Scan"
    ],
    "Resource": "arn:aws:dynamodb:region:account:table/ProductCatalog",
    "Condition": {
      "ForAllValues:StringEquals": {
        "dynamodb:Attributes": ["ProductId", "Category", "Name", "Price"]
      }
    }
  }, {
    "Effect": "Allow",
    "Action": [
      "dynamodb:PutItem",
      "dynamodb:UpdateItem"
    ],
    "Resource": "arn:aws:dynamodb:region:account:table/ProductCatalog",
    "Condition": {
      "StringEquals": {
        "dynamodb:LeadingKeys": ["${aws:username}"]
      }
    }
  }]
}
```

### VPC Endpoints

```bash
# Create VPC endpoint for DynamoDB
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-west-2.dynamodb \
  --route-table-ids rtb-12345678 \
  --policy-document '{
    "Statement": [{
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": "*"
    }]
  }'
```

## Monitoring and Troubleshooting

### CloudWatch Metrics

```bash
# Key DynamoDB metrics
# - ConsumedReadCapacityUnits/ConsumedWriteCapacityUnits
# - ProvisionedReadCapacityUnits/ProvisionedWriteCapacityUnits
# - ReadThrottleEvents/WriteThrottleEvents
# - SystemErrors/UserErrors
# - SuccessfulRequestLatency

# Create CloudWatch alarm for throttling
aws cloudwatch put-metric-alarm \
  --alarm-name "DynamoDB-Read-Throttles" \
  --alarm-description "DynamoDB read throttling detected" \
  --metric-name ReadThrottleEvents \
  --namespace AWS/DynamoDB \
  --statistic Sum \
  --period 300 \
  --threshold 0 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=TableName,Value=ProductCatalog \
  --alarm-actions arn:aws:sns:region:account:dynamodb-alerts
```

### Performance Analysis

```bash
# Enable contributor insights
aws dynamodb enable-kinesis-streaming-destination \
  --table-name ProductCatalog \
  --stream-arn arn:aws:kinesis:region:account:stream/dynamodb-stream

# Query metrics programmatically
aws cloudwatch get-metric-statistics \
  --namespace AWS/DynamoDB \
  --metric-name ConsumedReadCapacityUnits \
  --dimensions Name=TableName,Value=ProductCatalog \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T01:00:00Z \
  --period 300 \
  --statistics Sum,Average,Maximum
```

## Exam-Specific Tips and Common Scenarios

### Key Exam Topics

1. **Data Modeling**
   - Primary key design (partition + sort key)
   - GSI vs LSI differences and limitations
   - Query vs Scan operations
   - Hot partition avoidance

2. **Capacity Planning**
   - On-demand vs Provisioned mode
   - Auto Scaling configuration
   - Read/Write capacity calculations
   - Throttling and error handling

3. **Advanced Features**
   - Global Tables for multi-region
   - DynamoDB Streams for change capture
   - DAX for caching
   - Transactions and ACID compliance

### Common Exam Scenarios

1. **High-Scale Web Application**
   - Use on-demand for unpredictable traffic
   - Implement DAX for read-heavy workloads
   - Use Global Tables for global users
   - Design efficient access patterns

2. **Real-Time Analytics**
   - Enable DynamoDB Streams
   - Process changes with Lambda
   - Aggregate data in real-time
   - Store time-series data efficiently

3. **Multi-Region Applications**
   - Configure Global Tables
   - Handle eventual consistency
   - Optimize for local reads
   - Plan for conflict resolution

### Exam Tips

- **Know the differences** between GSI and LSI
- **Understand capacity modes** and when to use each
- **Remember Global Tables** provide multi-region, multi-active replication
- **Know DynamoDB Streams** enable real-time change processing
- **Understand DAX** provides microsecond latency caching

## Hands-on Examples and CLI Commands

### Complete DynamoDB Implementation

```bash
# 1. Create table with streams and encryption
aws dynamodb create-table \
  --table-name UserProfiles \
  --attribute-definitions \
    AttributeName=UserId,AttributeType=S \
    AttributeName=LastLoginDate,AttributeType=S \
    AttributeName=Email,AttributeType=S \
  --key-schema \
    AttributeName=UserId,KeyType=HASH \
  --global-secondary-indexes \
    IndexName=EmailIndex,KeySchema='[{AttributeName=Email,KeyType=HASH}]',Projection='{ProjectionType=ALL}',ProvisionedThroughput='{ReadCapacityUnits=5,WriteCapacityUnits=5}' \
    IndexName=LoginDateIndex,KeySchema='[{AttributeName=LastLoginDate,KeyType=HASH}]',Projection='{ProjectionType=KEYS_ONLY}',ProvisionedThroughput='{ReadCapacityUnits=5,WriteCapacityUnits=5}' \
  --billing-mode PROVISIONED \
  --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10 \
  --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES \
  --sse-specification Enabled=true,SSEType=KMS \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true \
  --tags Key=Environment,Value=Production

# 2. Load sample data
aws dynamodb batch-write-item \
  --request-items '{
    "UserProfiles": [
      {
        "PutRequest": {
          "Item": {
            "UserId": {"S": "user001"},
            "Email": {"S": "user001@example.com"},
            "Name": {"S": "John Doe"},
            "LastLoginDate": {"S": "2023-01-01"}
          }
        }
      }
    ]
  }'

# 3. Query data efficiently
aws dynamodb query \
  --table-name UserProfiles \
  --key-condition-expression "UserId = :userId" \
  --expression-attribute-values '{":userId": {"S": "user001"}}' \
  --projection-expression "UserId, Email, #n" \
  --expression-attribute-names '{"#n": "Name"}'

# 4. Update item with condition
aws dynamodb update-item \
  --table-name UserProfiles \
  --key '{"UserId": {"S": "user001"}}' \
  --update-expression "SET LastLoginDate = :date, LoginCount = LoginCount + :inc" \
  --condition-expression "attribute_exists(UserId)" \
  --expression-attribute-values '{
    ":date": {"S": "2023-01-02"},
    ":inc": {"N": "1"}
  }' \
  --return-values ALL_NEW
```

This comprehensive DynamoDB documentation covers all essential aspects needed for AWS certification exams.