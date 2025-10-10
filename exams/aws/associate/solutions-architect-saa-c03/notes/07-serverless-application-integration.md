# AWS Serverless and Application Integration - SAA-C03

## AWS Lambda

### Core Concepts
- Event-driven, serverless compute
- Pay per request and compute time
- Automatic scaling (up to 1,000 concurrent by default)
- No server management
- Supports multiple languages (Node.js, Python, Java, Go, .NET, Ruby, custom runtimes)

### Limits
- **Memory**: 128 MB to 10,240 MB (1 MB increments)
- **Timeout**: Max 15 minutes
- **Deployment Package**: 50 MB (zipped), 250 MB (unzipped)
- **Environment Variables**: 4 KB total
- **Concurrent Executions**: 1,000 (default, can request increase)
- **Burst Concurrency**: 500-3,000 depending on region

### Invocation Types

**Synchronous** (Wait for response):
- CLI, SDK, API Gateway
- User waits for result
- Error handling by caller

**Asynchronous** (Fire and forget):
- S3, SNS, EventBridge, SES
- Lambda retries 2 times
- Dead Letter Queue (DLQ) for failures

**Event Source Mapping** (Poll-based):
- Kinesis, DynamoDB Streams, SQS
- Lambda polls source
- Batch processing
- Retries until success or data expires

### Lambda Execution Environment
- **/tmp**: 512 MB - 10 GB ephemeral storage
- Environment variables for configuration
- Layers for shared code libraries (up to 5 layers)
- Container image support (up to 10 GB)

### Performance Optimization

**Cold Start**:
- First invocation or scaling up
- Initialize runtime and code
- Reduce: Provisioned concurrency, smaller package, minimize dependencies

**Provisioned Concurrency**:
- Pre-initialized execution environments
- Eliminates cold starts
- Additional cost
- Use for latency-sensitive applications

**Best Practices**:
- Minimize deployment package size
- Use environment variables for configuration
- Reuse connections (database, HTTP)
- Increase memory for CPU-bound tasks
- Use /tmp for temporary files

### Lambda@Edge
- Run Lambda at CloudFront edge locations
- Customize content delivery
- 4 CloudFront events:
  - Viewer Request: After CloudFront receives request
  - Origin Request: Before forwarding to origin
  - Origin Response: After CloudFront receives response
  - Viewer Response: Before returning to viewer
- **Limits**: 128 MB memory, 5-30s timeout

## Amazon API Gateway

### Types

**REST API**:
- RESTful architecture
- API keys, usage plans, request throttling
- Caching
- Import from OpenAPI 3.0
- Regional, edge-optimized, or private

**HTTP API**:
- Lower latency, lower cost (70% cheaper)
- OIDC and OAuth 2.0 authorization
- No caching, usage plans, or API keys
- Use when: Simple proxy, cost optimization

**WebSocket API**:
- Bi-directional communication
- Real-time applications (chat, gaming)
- Connection state management

### Deployment and Stages
- **Stages**: Different environments (dev, test, prod)
- **Stage Variables**: Environment-specific configuration
- **Canary Deployments**: Gradual rollout (% traffic)

### Security
- **IAM Permissions**: SigV4 signing
- **Cognito User Pools**: User authentication
- **Lambda Authorizers**: Custom authorization logic
- **Resource Policies**: Control access to API
- **API Keys**: Simple usage tracking (not for security)

### Throttling
- **Account Limit**: 10,000 requests/second
- **Burst**: 5,000 requests
- **Stage/Method Limit**: Custom limits
- **Usage Plans**: Associate with API keys

### Caching
- Cache responses for specified TTL
- Reduce backend calls
- Improve latency
- Per-stage cache (0.5 GB to 237 GB)
- Can be invalidated

### Integration Types
- **Lambda Function**: Proxy or custom mapping
- **HTTP Endpoint**: Proxy to external API
- **AWS Service**: Invoke AWS services (S3, DynamoDB, etc.)
- **Mock**: Return static response

## Amazon SQS (Simple Queue Service)

### Standard Queue
- **Unlimited Throughput**: Unlimited TPS
- **At-Least-Once Delivery**: May deliver duplicates
- **Best-Effort Ordering**: Messages may be out of order
- **Use Case**: Decouple applications, buffer requests

### FIFO Queue
- **Ordered**: Exactly-once processing
- **Limited Throughput**: 300 TPS (3,000 with batching)
- **Deduplication**: 5-minute window
- **Message Groups**: Ordering within group
- **Use Case**: Order processing, event sequencing

### Key Features
- **Message Retention**: 1 minute to 14 days (default 4 days)
- **Message Size**: Up to 256 KB
- **Visibility Timeout**: 0 seconds to 12 hours (default 30s)
- **Long Polling**: Reduce empty responses (1-20s)
- **Dead Letter Queue**: Failed messages after max receives
- **Delay Queue**: Delay delivery up to 15 minutes
- **Message Attributes**: Metadata (up to 10)

### Visibility Timeout
- Message hidden from other consumers during processing
- If not deleted, becomes visible again after timeout
- Use ChangeMessageVisibility to extend

### Dead Letter Queue (DLQ)
- Receive messages after max receive count
- Analyze failed messages
- Separate queue with same type (standard/FIFO)

## Amazon SNS (Simple Notification Service)

### Pub/Sub Model
- **Topic**: Communication channel
- **Publishers**: Send messages to topic
- **Subscribers**: Receive messages from topic
- **Fan-out**: One message to multiple subscribers

### Subscription Types
- **SQS**: Decouple, persist, process asynchronously
- **Lambda**: Event-driven processing
- **HTTP/HTTPS**: Webhook delivery
- **Email/Email-JSON**: Notifications
- **SMS**: Text messages
- **Mobile Push**: iOS, Android, Fire OS

### Message Filtering
- Subscribers receive filtered subset
- JSON policy on message attributes
- Reduce unnecessary processing

### Message Attributes
- Metadata for routing and filtering
- Up to 10 attributes

### SNS + SQS Fan-out Pattern
- Publish once to SNS
- Multiple SQS queues subscribe
- Each queue processes independently
- Guarantees delivery to each subscriber
- Use Case: Parallel async processing

### FIFO Topics
- Ordering and deduplication
- Limited to FIFO SQS subscribers
- 300 TPS (3,000 with batching)

## AWS Step Functions

### Features
- Orchestrate serverless workflows
- Visual workflow editor
- State machines (JSON)
- Error handling and retries
- Parallel and choice states

### Workflow Types

**Standard Workflows**:
- Long-running (up to 1 year)
- Exactly-once execution
- At-most-once state transitions
- Full execution history
- Use Case: Long-running processes

**Express Workflows**:
- Short-duration (up to 5 minutes)
- At-least-once execution
- High event rate (100,000+ per second)
- Cheaper for high-volume
- Use Case: IoT, streaming, mobile backends

### States
- **Task**: Single unit of work (Lambda, ECS, etc.)
- **Choice**: Conditional logic
- **Parallel**: Execute branches in parallel
- **Wait**: Delay for duration or timestamp
- **Succeed/Fail**: Terminal states
- **Pass**: No-op, data transformation
- **Map**: Iterate over array

### Error Handling
- **Retry**: Exponential backoff
- **Catch**: Handle errors, transition to fallback

### Use Cases
- Microservice orchestration
- ETL workflows
- Order processing
- Video processing pipelines

## Amazon EventBridge

### Event Bus
- Default bus for AWS services
- Custom bus for applications
- Partner bus for SaaS

### Event Rules
- Event pattern matching
- Schedule (cron, rate)
- Filter and route to targets

### Targets (20+ services)
- Lambda, Step Functions
- SQS, SNS, Kinesis
- EC2 Run Command
- CodePipeline, CodeBuild
- ECS task

### Schema Registry
- Infer schema from events
- Versioning
- Code bindings generation

### Use Cases
- Event-driven architectures
- Cross-account event routing
- Schedule-based automation
- SaaS integration

## Amazon AppSync

### Features
- Managed GraphQL service
- Real-time data synchronization
- Offline support
- Fine-grained access control

### Data Sources
- DynamoDB, Lambda, HTTP endpoints
- RDS (via Lambda)
- OpenSearch

### Resolvers
- Map GraphQL operations to data sources
- VTL (Velocity Template Language) or JavaScript
- Pipeline resolvers for complex operations

### Security
- API keys
- IAM
- Cognito User Pools
- OpenID Connect

## Amazon MQ

### Features
- Managed message broker
- Apache ActiveMQ and RabbitMQ
- JMS, AMQP, MQTT, STOMP, WebSocket protocols
- Use Case: Migrate existing applications using traditional protocols

### MQ vs SQS/SNS
- **Amazon MQ**: Existing apps, specific protocols, lift-and-shift
- **SQS/SNS**: Cloud-native, scalable, no protocol requirements

## Exam Tips

### Scenario Matching
- **Decouple applications**: SQS
- **Distribute messages to multiple consumers**: SNS
- **Guaranteed message order**: SQS FIFO
- **Real-time notifications**: SNS
- **Fan-out pattern**: SNS + multiple SQS
- **Execute code without servers**: Lambda
- **RESTful API**: API Gateway REST API
- **Low-cost API**: API Gateway HTTP API
- **Orchestrate multiple services**: Step Functions
- **Event-driven architecture**: EventBridge
- **Traditional message broker**: Amazon MQ
- **GraphQL API**: AppSync

### Lambda Best Practices
1. Keep functions small and focused
2. Use environment variables for configuration
3. Implement proper error handling and logging
4. Use layers for shared dependencies
5. Monitor with CloudWatch and X-Ray
6. Use Provisioned Concurrency for latency-sensitive apps
7. Set appropriate timeout and memory
8. Use DLQ for asynchronous invocations
9. Implement idempotency for retries
10. Minimize cold start impact

### Integration Patterns
- **Synchronous**: API Gateway → Lambda
- **Asynchronous**: S3 → Lambda, SNS → Lambda
- **Event Streaming**: Kinesis → Lambda
- **Queue Processing**: SQS → Lambda
- **Scheduled**: EventBridge → Lambda
- **Fan-out**: SNS → multiple SQS → Lambda
- **Orchestration**: EventBridge → Step Functions → Lambda
