# AWS Integration & Streaming Fundamentals

## Application Integration Overview

### Integration Patterns
| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Synchronous** | Direct service-to-service calls | Real-time response needed |
| **Asynchronous** | Message-based communication | Decoupled, fault-tolerant |
| **Event-driven** | React to state changes | Microservices, automation |
| **Workflow** | Multi-step process orchestration | Complex business processes |

### AWS Integration Services Decision Tree
```
Need reliable messaging? → SQS
Need pub/sub pattern? → SNS
Need event routing? → EventBridge
Need workflow orchestration? → Step Functions
Need streaming data? → Kinesis
Need API management? → API Gateway
```

## Amazon SQS (Simple Queue Service)

### What is SQS?
Fully managed message queuing service that enables decoupling of distributed software systems and microservices.

### Queue Types
| Feature | Standard Queue | FIFO Queue |
|---------|----------------|------------|
| **Ordering** | Best effort | Guaranteed |
| **Delivery** | At least once | Exactly once |
| **Throughput** | Unlimited | 300 TPS (without batching) |
| **Deduplication** | No | Yes |
| **Use Case** | High throughput | Order matters |

### Key SQS Concepts
- **Producer**: Sends messages to queue
- **Consumer**: Receives and processes messages
- **Message**: Data sent through queue (up to 256KB)
- **Visibility Timeout**: Time message is hidden from other consumers
- **Dead Letter Queue**: Failed message destination

### SQS Message Lifecycle
```
1. Producer sends message to queue
2. Message becomes available for consumption
3. Consumer polls and receives message
4. Message becomes invisible to other consumers
5. Consumer processes message
6. Consumer deletes message from queue
```

### Visibility Timeout
- **Purpose**: Prevent multiple consumers from processing same message
- **Default**: 30 seconds
- **Range**: 0 seconds to 12 hours
- **Best Practice**: Set slightly longer than processing time

### Dead Letter Queues (DLQ)
**Handle messages that can't be processed successfully**

#### DLQ Benefits
- **Isolate problematic messages**: Prevent blocking other messages
- **Debugging**: Analyze failed messages
- **Manual intervention**: Retry after fixing issues
- **Metrics**: Monitor failure rates

#### DLQ Configuration
- **Maximum receives**: Number of failed processing attempts
- **Redrive policy**: When to move to DLQ
- **Message retention**: How long to keep failed messages

### Long Polling vs Short Polling
| Feature | Short Polling | Long Polling |
|---------|---------------|--------------|
| **Wait time** | Returns immediately | Waits up to 20 seconds |
| **Empty responses** | Many | Fewer |
| **Cost** | Higher | Lower |
| **Latency** | Lower | Slightly higher |
| **Default** | Yes | No (must configure) |

## Amazon SNS (Simple Notification Service)

### What is SNS?
Fully managed pub/sub messaging service for decoupling microservices, distributed systems, and serverless applications.

### SNS Components
- **Topic**: Communication channel for messages
- **Publisher**: Sends messages to topic
- **Subscriber**: Receives messages from topic
- **Message**: Data sent through topic (up to 256KB)

### SNS Subscription Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Email** | Send to email addresses | Human notifications |
| **SMS** | Send text messages | Mobile alerts |
| **HTTP/HTTPS** | POST to web endpoints | Webhook integrations |
| **SQS** | Send to SQS queue | Reliable message processing |
| **Lambda** | Trigger Lambda functions | Event processing |
| **Mobile Push** | iOS, Android, Windows | Mobile app notifications |

### Message Filtering
**Deliver messages to specific subscribers based on attributes**

#### Filter Policy Example
```json
{
  "store": ["example_corp"],
  "price": [{"numeric": [">=", 100]}],
  "event": ["order-placed", "order-cancelled"]
}
```

### SNS Message Structure
```json
{
  "Type": "Notification",
  "MessageId": "22b80b92-fdea-4c2c-8f9d-bdfb0c7bf324",
  "TopicArn": "arn:aws:sns:us-east-1:123456789012:MyTopic",
  "Subject": "My First Message",
  "Message": "Hello World!",
  "Timestamp": "2012-05-02T00:54:06.655Z",
  "Attributes": {
    "store": "example_corp",
    "price": 150
  }
}
```

### SNS + SQS Fan-out Pattern
**Deliver one message to multiple queues**

```
SNS Topic
├── SQS Queue 1 (Order Processing)
├── SQS Queue 2 (Inventory Update)
├── SQS Queue 3 (Analytics)
└── Lambda Function (Email Notification)
```

## Amazon EventBridge

### What is EventBridge?
Serverless event bus service that connects applications using events from AWS services, SaaS applications, and custom sources.

### EventBridge Components
- **Event Source**: Where events originate
- **Event Bus**: Central routing mechanism
- **Rules**: Route events to targets
- **Targets**: Where events are delivered

### Event Bus Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Default** | AWS service events | AWS service integration |
| **Custom** | Application events | Custom applications |
| **Partner** | SaaS application events | Third-party integrations |

### Event Pattern Matching
**Route events based on content**

#### Example Event Pattern
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

### EventBridge vs SNS vs SQS
| Feature | EventBridge | SNS | SQS |
|---------|-------------|-----|-----|
| **Pattern** | Event routing | Pub/Sub | Point-to-point |
| **Filtering** | Advanced rules | Simple attributes | No filtering |
| **Targets** | 20+ AWS services | 6 protocols | N/A |
| **Schema** | Schema registry | No | No |
| **Replay** | Yes | No | No |

## AWS Step Functions

### What is Step Functions?
Visual workflow service that coordinates AWS services into serverless workflows using state machines.

### State Types
| State | Purpose | Example |
|-------|---------|---------|
| **Task** | Execute work | Call Lambda, start EC2 |
| **Choice** | Branch based on input | Route based on order type |
| **Wait** | Delay execution | Wait for external process |
| **Succeed** | End successfully | Workflow completion |
| **Fail** | End with failure | Error handling |
| **Parallel** | Execute branches concurrently | Process multiple items |
| **Map** | Iterate over array | Process list of files |

### Workflow Types
| Type | Use Case | Execution Time | Cost |
|------|----------|----------------|------|
| **Standard** | Long-running workflows | Up to 1 year | Pay per state transition |
| **Express** | High-volume, short-duration | Up to 5 minutes | Pay per execution |

### Error Handling
#### Retry Configuration
```json
{
  "Retry": [{
    "ErrorEquals": ["States.TaskFailed"],
    "IntervalSeconds": 2,
    "MaxAttempts": 3,
    "BackoffRate": 2.0
  }]
}
```

#### Catch Configuration
```json
{
  "Catch": [{
    "ErrorEquals": ["States.ALL"],
    "Next": "ErrorHandler",
    "ResultPath": "$.error"
  }]
}
```

## Amazon Kinesis

### What is Kinesis?
Platform for streaming data on AWS, offering powerful services to make it easy to load and analyze streaming data.

### Kinesis Services
| Service | Purpose | Use Case |
|---------|---------|----------|
| **Data Streams** | Real-time data streaming | Custom applications |
| **Data Firehose** | Load streaming data | Data lakes, warehouses |
| **Data Analytics** | Real-time analytics | SQL queries on streams |
| **Video Streams** | Video streaming | Video analytics, ML |

### Kinesis Data Streams

#### Key Concepts
- **Stream**: Collection of data records
- **Shard**: Base unit of capacity (1MB/sec in, 2MB/sec out)
- **Record**: Unit of data (up to 1MB)
- **Partition Key**: Determines which shard receives record
- **Sequence Number**: Unique identifier within shard

#### Stream Capacity
```
Per Shard:
- 1,000 records/second OR 1 MB/second (input)
- 2 MB/second OR 2,000 records/second (output)

Example: 10 shards = 10,000 records/second input capacity
```

#### Data Retention
- **Default**: 24 hours
- **Extended**: Up to 365 days
- **Cost**: Increases with retention period

### Kinesis Data Firehose

#### Delivery Destinations
- **Amazon S3**: Data lake storage
- **Redshift**: Data warehouse
- **OpenSearch**: Search and analytics
- **Splunk**: Log analysis
- **HTTP endpoints**: Custom destinations

#### Data Transformation
- **Lambda functions**: Transform records before delivery
- **Data format conversion**: Convert to Parquet, ORC
- **Compression**: GZIP, SNAPPY, ZIP
- **Error handling**: Failed record handling

#### Buffer Configuration
| Setting | Purpose | Values |
|---------|---------|--------|
| **Buffer size** | Amount of data to batch | 1-128 MB |
| **Buffer interval** | Time to wait | 60-900 seconds |
| **Compression** | Reduce storage costs | GZIP, SNAPPY, ZIP |

## Amazon API Gateway

### What is API Gateway?
Fully managed service for creating, publishing, maintaining, monitoring, and securing APIs at any scale.

### API Types
| Type | Protocol | Use Case |
|------|----------|----------|
| **REST API** | HTTP/HTTPS | Web services, mobile backends |
| **HTTP API** | HTTP/HTTPS | Lower cost, faster REST APIs |
| **WebSocket API** | WebSocket | Real-time applications |

### API Gateway Features
- **Request/Response transformation**: Modify requests and responses
- **Authentication**: API keys, AWS IAM, Cognito, Lambda authorizers
- **Rate limiting**: Throttle requests
- **Caching**: Improve performance
- **Monitoring**: CloudWatch integration
- **CORS**: Cross-origin resource sharing

### Integration Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Lambda Proxy** | Forward entire request to Lambda | Serverless backends |
| **Lambda Custom** | Transform request before Lambda | Custom request handling |
| **HTTP Proxy** | Forward to HTTP endpoint | Existing web services |
| **HTTP Custom** | Transform before HTTP call | Legacy system integration |
| **AWS Service** | Direct AWS service integration | S3, DynamoDB access |
| **Mock** | Return mock response | Testing, prototyping |

### Stages and Deployments
- **Stage**: Named reference to deployment
- **Stage variables**: Environment-specific configuration
- **Canary deployments**: Gradual rollout to subset of traffic
- **Blue/green deployments**: Switch between two stages

## Amazon MQ

### What is Amazon MQ?
Managed message broker service for Apache ActiveMQ and RabbitMQ.

### When to Use Amazon MQ
- **Legacy applications**: Existing JMS, AMQP, MQTT, OpenWire, STOMP protocols
- **Enterprise messaging**: Complex routing, message transformation
- **Standards compliance**: Industry standard protocols
- **Migration**: Lift-and-shift existing message brokers

### MQ vs SQS/SNS
| Feature | Amazon MQ | SQS/SNS |
|---------|-----------|---------|
| **Protocols** | Industry standards | AWS proprietary |
| **Management** | Some infrastructure | Fully managed |
| **Scale** | Instance-based | Serverless |
| **Cost** | Instance charges | Pay per use |
| **Migration** | Easy from existing | Requires code changes |

## Integration Patterns and Best Practices

### Decoupling Patterns

#### Queue-Based Pattern (SQS)
```
Producer → SQS Queue → Consumer
Benefits: Reliability, scaling, fault tolerance
```

#### Pub/Sub Pattern (SNS)
```
Publisher → SNS Topic → Multiple Subscribers
Benefits: Fan-out, loose coupling
```

#### Event-Driven Pattern (EventBridge)
```
Event Source → EventBridge → Rule → Target
Benefits: Event routing, filtering, replay
```

### Error Handling Strategies

#### Retry with Exponential Backoff
```
Attempt 1: Immediate
Attempt 2: 2 seconds
Attempt 3: 4 seconds
Attempt 4: 8 seconds
```

#### Dead Letter Queues
- Move failed messages to separate queue
- Analyze failure patterns
- Manual or automated recovery

#### Circuit Breaker Pattern
- Stop calling failing service
- Fail fast to prevent cascading failures
- Retry after cooldown period

### Monitoring and Observability

#### Key Metrics to Monitor
- **Message volume**: Number of messages processed
- **Processing time**: Time to process messages
- **Error rates**: Failed message percentage
- **Queue depth**: Number of messages waiting
- **Consumer lag**: Time between send and receive

#### CloudWatch Integration
- **Built-in metrics**: All services provide basic metrics
- **Custom metrics**: Application-specific measurements
- **Alarms**: Proactive monitoring
- **Dashboards**: Visualization and reporting

## Cost Optimization

### SQS Cost Factors
- **Requests**: Number of API calls
- **Data transfer**: Cross-region charges
- **Long polling**: Reduces request costs

### SNS Cost Factors
- **Notifications**: Per notification charges
- **Data transfer**: Varies by destination type
- **SMS**: Higher cost than other delivery methods

### Kinesis Cost Factors
- **Shard hours**: Number of shards × time
- **PUT payload units**: Data ingestion charges
- **Extended retention**: Additional storage costs

### API Gateway Cost Factors
- **API calls**: Number of requests
- **Data transfer**: Response payload size
- **Caching**: Additional caching charges

## Common Integration Pitfalls

### SQS Issues
- Not handling duplicate messages (idempotency)
- Incorrect visibility timeout settings
- Missing dead letter queue configuration
- Poor error handling and retry logic

### SNS Issues
- Not implementing message filtering
- Inadequate error handling for failed deliveries
- Missing monitoring for delivery failures
- Not considering message size limits

### Step Functions Issues
- Not implementing proper error handling
- Overly complex state machines
- Not using parallel states for independent tasks
- Inadequate monitoring and logging

### API Gateway Issues
- Not implementing proper authentication
- Missing rate limiting and throttling
- Inadequate error handling and responses
- Not optimizing for cost (caching, compression)