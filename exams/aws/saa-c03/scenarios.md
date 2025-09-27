# High-Yield Scenarios & Patterns

## Web Application Architectures

### Classic 3-Tier Web App
**Scenario**: Company needs to migrate a traditional web application to AWS with high availability.

**Solution Pattern**:
- **Presentation Tier**: ALB + EC2 Auto Scaling Group (multi-AZ)
- **Application Tier**: ALB + EC2 Auto Scaling Group in private subnets
- **Data Tier**: RDS Multi-AZ with read replicas

**Common Distractors**:
- Single AZ deployment (wrong - no HA)
- Public subnets for application tier (wrong - security)
- DynamoDB for relational data (wrong - use case mismatch)

### Serverless Web Application
**Scenario**: Startup needs cost-effective, scalable web application with minimal operational overhead.

**Solution Pattern**:
- **Frontend**: S3 + CloudFront
- **API**: API Gateway + Lambda
- **Database**: DynamoDB
- **Authentication**: Cognito

**Common Distractors**:
- EC2 instances (wrong - not serverless)
- RDS for simple data (wrong - DynamoDB more cost-effective)
- Direct Lambda invocation (wrong - use API Gateway)

## Data Storage & Backup

### Disaster Recovery Strategy
**Scenario**: Company requires RPO of 1 hour and RTO of 4 hours for critical database.

**Solution Pattern**:
- **Primary**: RDS Multi-AZ in primary region
- **DR**: Cross-region automated backups + read replica
- **Failover**: Promote read replica in DR region

**Common Distractors**:
- Single AZ RDS (wrong - doesn't meet RTO)
- Manual snapshots only (wrong - doesn't meet RPO)
- Same-region read replica (wrong - not cross-region DR)

### Archive Strategy
**Scenario**: Company needs to retain logs for 7 years with immediate access for first 90 days.

**Solution Pattern**:
- **0-90 days**: S3 Standard
- **90 days - 1 year**: S3 IA (lifecycle policy)
- **1-7 years**: S3 Glacier Deep Archive (lifecycle policy)

**Common Distractors**:
- Keep all in S3 Standard (wrong - cost inefficient)
- Move to Glacier immediately (wrong - access pattern)
- Use EBS for archival (wrong - not designed for archival)

## Security Scenarios

### Secure API Access
**Scenario**: Mobile app needs secure access to AWS APIs with user authentication.

**Solution Pattern**:
- **User Auth**: Cognito User Pool
- **AWS Access**: Cognito Identity Pool with IAM roles
- **API Security**: API Gateway with Cognito authorizer
- **Fine-grained Access**: IAM policies with conditions

**Common Distractors**:
- Hard-coded AWS credentials (wrong - security risk)
- Single IAM user for all mobile users (wrong - no granularity)
- Basic authentication (wrong - not secure enough)

### Cross-Account Access
**Scenario**: Development account needs read-only access to production S3 bucket.

**Solution Pattern**:
- **Cross-account role**: Create role in prod account
- **Trust policy**: Allow dev account to assume role
- **Permission policy**: S3 read-only permissions
- **Usage**: Dev users assume role using STS

**Common Distractors**:
- Share S3 bucket credentials (wrong - not secure)
- Create IAM user in prod account (wrong - not cross-account)
- Bucket policy with wildcard principals (wrong - too permissive)

## Performance & Scaling

### Global Application
**Scenario**: E-commerce site needs low latency globally with dynamic content.

**Solution Pattern**:
- **Global Distribution**: CloudFront with multiple origins
- **Static Content**: S3 origins in multiple regions
- **Dynamic Content**: ALB origins in multiple regions
- **Database**: Aurora Global Database or DynamoDB Global Tables

**Common Distractors**:
- Single region deployment (wrong - latency issues)
- CloudFront for dynamic content only (wrong - cache static too)
- Regional replication without global database (wrong - data consistency)

### High-Traffic Database
**Scenario**: Application experiences read-heavy traffic patterns with occasional spikes.

**Solution Pattern**:
- **Primary Database**: Aurora with multiple read replicas
- **Caching Layer**: ElastiCache (Redis) for frequently accessed data
- **Connection Pooling**: RDS Proxy to manage connections
- **Auto Scaling**: Aurora Serverless for unpredictable workloads

**Common Distractors**:
- Single RDS instance (wrong - doesn't scale reads)
- DynamoDB for complex relational queries (wrong - use case mismatch)
- Application-level caching only (wrong - not optimal)

## Cost Optimization

### Mixed Workload Optimization
**Scenario**: Company has steady-state workload with predictable batch processing.

**Solution Pattern**:
- **Steady State**: Reserved Instances for baseline capacity
- **Variable Load**: Auto Scaling with On-Demand instances
- **Batch Processing**: Spot Instances for fault-tolerant workloads
- **Storage**: S3 Intelligent Tiering for unknown access patterns

**Common Distractors**:
- All On-Demand instances (wrong - cost inefficient)
- All Spot instances (wrong - not suitable for steady state)
- Single instance type (wrong - not optimized for workload)

## Integration Patterns

### Microservices Communication
**Scenario**: Microservices architecture needs reliable, decoupled communication.

**Solution Pattern**:
- **Synchronous**: API Gateway + ALB for service-to-service
- **Asynchronous**: SQS for reliable message queuing
- **Pub/Sub**: SNS for fanout messaging patterns
- **Workflow**: Step Functions for complex orchestration

**Common Distractors**:
- Direct service-to-service calls (wrong - tight coupling)
- Database for message passing (wrong - not designed for this)
- Synchronous calls for everything (wrong - doesn't handle failures well)

### Event-Driven Architecture
**Scenario**: Need to process file uploads and trigger multiple downstream actions.

**Solution Pattern**:
- **Trigger**: S3 Event Notifications
- **Routing**: SNS topic with multiple SQS subscriptions
- **Processing**: Lambda functions triggered by SQS
- **State Management**: Step Functions for complex workflows

**Common Distractors**:
- Polling S3 for changes (wrong - inefficient)
- Direct Lambda triggers for everything (wrong - tight coupling)
- Synchronous processing (wrong - doesn't scale)

## Key Decision Factors

### Service Selection Criteria
1. **Managed vs Self-Managed**: Prefer managed services for operational excellence
2. **Cost vs Performance**: Balance based on requirements
3. **Scaling Patterns**: Consider growth patterns and traffic
4. **Integration Requirements**: How services work together
5. **Compliance Needs**: Security and regulatory requirements

### Common Anti-Patterns
- Over-engineering simple solutions
- Choosing cutting-edge over proven solutions
- Ignoring operational overhead
- Not considering vendor lock-in
- Skipping disaster recovery planning