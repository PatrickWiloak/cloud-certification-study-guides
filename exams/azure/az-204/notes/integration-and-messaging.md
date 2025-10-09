# Connect to and Consume Azure Services and Third-party Services

## Overview
This domain covers implementing API Management, developing event-based solutions, and creating message-based solutions. It represents 15-20% of the exam and focuses on integration patterns and asynchronous communication.

## Key Topics

### API Management
- APIM instance creation and configuration
- API authentication configuration
- API policy definition and implementation
- API versioning and revision strategies

### Event-based Solutions
- Azure Event Grid solutions implementation
- Azure Event Hubs solutions development
- Event publishing and subscription patterns
- Event schema design and validation

### Message-based Solutions
- Azure Service Bus implementation (queues and topics)
- Azure Queue Storage queue solutions
- Message processing patterns
- Dead letter handling and retry logic

## Services Reference

### Core Services
- Azure API Management (APIM)
- Azure Event Grid
- Azure Event Hubs
- Azure Service Bus
- Azure Queue Storage

### Supporting Services
- Azure Logic Apps
- Azure Functions (triggered by events/messages)
- Azure Notification Hubs
- Azure SignalR Service

## Best Practices

### API Management
- Use products and subscriptions for API organization
- Implement rate limiting to protect backend services
- Use policies for request/response transformation
- Enable caching for frequently accessed data
- Implement proper versioning strategy
- Use named values for configuration management

### Event-driven Architecture
- Choose Event Grid for discrete events
- Use Event Hubs for high-volume streaming
- Implement idempotent event handlers
- Design events with clear schema and versioning
- Use dead letter queues for failed events
- Monitor event processing latency

### Message-based Communication
- Use Service Bus for enterprise messaging requirements
- Choose Queue Storage for simple, cost-effective queuing
- Implement message deduplication when necessary
- Set appropriate message TTL values
- Handle poison messages properly
- Use sessions for ordered message processing

## Common Scenarios

### API Management Patterns
- API gateway for microservices
- Legacy service modernization with facade pattern
- Rate limiting and throttling policies
- API monetization with subscription tiers
- Cross-origin resource sharing (CORS) enablement

### Event Processing
- Real-time notification systems with Event Grid
- IoT telemetry processing with Event Hubs
- System integration through event subscriptions
- Event-driven automation workflows
- Reactive application architectures

### Message Processing
- Order processing workflows with Service Bus
- Asynchronous task processing with queues
- Fan-out messaging with topics and subscriptions
- Request-response patterns with reply queues
- Scheduled message processing

## Study Tips

- Understand when to use events vs messages
- Practice configuring APIM policies (inbound, backend, outbound, on-error)
- Learn Event Grid event schema and filtering
- Hands-on experience with Service Bus queues and topics
- Study message session handling in Service Bus
- Practice implementing retry policies and dead letter handling
- Understand Event Hubs partition keys and consumer groups
- Learn APIM authentication options (subscription keys, OAuth, certificates)
- Study the differences between Event Grid, Event Hubs, and Service Bus
- Practice creating event handlers with Azure Functions
