# AWS Monitoring and Management - SAA-C03

## Amazon CloudWatch

**[📖 Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)** - Monitor AWS resources and applications

### CloudWatch Metrics

**[📖 CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html)** - Collect and track metrics

**Default Metrics** (no agent required):
- **EC2**: CPU, Network, Disk (read/write ops), Status Checks
- **EBS**: Read/Write ops, throughput
- **ELB**: Request count, latency, healthy hosts
- **Auto Scaling**: Group metrics
- **RDS**: Database connections, CPU, storage
- **S3**: Request metrics (with extra cost)

**Custom Metrics** (requires CloudWatch Agent):
- Memory utilization
- Disk space utilization
- Swap usage
- Custom application metrics

**Metric Characteristics**:
- **Namespace**: Container for metrics (AWS/EC2, AWS/RDS)
- **Dimensions**: Name/value pairs (InstanceId=i-1234)
- **Resolution**: Standard (60s), High (1s) - additional cost
- **Retention**: Up to 15 months
- **Statistics**: Sum, Average, Min, Max, Sample Count, Percentiles

### CloudWatch Alarms

**[📖 CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)** - Monitor metrics and send notifications

**States**:
- **OK**: Metric within threshold
- **ALARM**: Metric breached threshold
- **INSUFFICIENT_DATA**: Not enough data

**Actions**:
- SNS notification
- Auto Scaling action (add/remove instances)
- EC2 action (stop, terminate, reboot, recover)
- Systems Manager action

**Types**:
- **Metric Alarm**: Based on single metric
- **Composite Alarm**: Combination of multiple alarms (AND, OR logic)

### CloudWatch Logs

**[📖 CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)** - Monitor and store log files

**Components**:
- **Log Groups**: Collection of log streams (e.g., /aws/lambda/my-function)
- **Log Streams**: Sequence of log events (e.g., instance ID)
- **Log Events**: Record of activity

**Sources**:
- EC2 instances (CloudWatch agent)
- Lambda functions (automatic)
- VPC Flow Logs
- Route 53 query logs
- CloudTrail logs
- Elastic Beanstalk
- ECS containers

**Features**:
- **Metric Filters**: Create metrics from log patterns
- **Log Insights**: Query and analyze logs (SQL-like)
- **Subscription Filters**: Real-time streaming to Lambda, Kinesis, OpenSearch
- **Export**: S3, OpenSearch Service
- **Retention**: 1 day to 10 years, never expire

**CloudWatch Logs Insights Query**:
```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

### CloudWatch Events / EventBridge

**Event Sources**:
- AWS services (EC2 state change, S3 object created)
- Custom applications
- Scheduled events (cron or rate)

**Targets**:
- Lambda functions
- SNS topics
- SQS queues
- Step Functions
- EC2 instances (Run Command)
- ECS tasks
- CodePipeline, CodeBuild
- Kinesis streams

**Use Cases**:
- Trigger Lambda on S3 upload
- Schedule automated tasks
- Respond to infrastructure changes
- Multi-account event aggregation

### CloudWatch Dashboards
- Visualize metrics and alarms
- Cross-region and cross-account
- Automatic dashboards for AWS services
- Custom dashboards with widgets

## AWS X-Ray

**[📖 AWS X-Ray Documentation](https://docs.aws.amazon.com/xray/)** - Analyze and debug distributed applications

### Features
- Distributed tracing for applications
- Service map visualization
- Request/response analysis
- Performance bottleneck identification
- Error and exception tracking

### Components
- **X-Ray SDK**: Instrument application code
- **X-Ray Daemon**: Collect and send traces
- **X-Ray API**: Retrieve trace data
- **X-Ray Console**: Visualize service map

### Concepts
- **Segments**: Data for single component (e.g., API call)
- **Subsegments**: More granular timing (DB queries)
- **Traces**: End-to-end request path
- **Sampling**: Control cost by tracing subset of requests
- **Annotations**: Indexed key-value pairs for filtering
- **Metadata**: Non-indexed additional data

### Integration
- Lambda (enable with checkbox)
- Elastic Beanstalk (configuration)
- ECS (task definition)
- API Gateway (enable in stage)
- EC2 (install X-Ray daemon)

## AWS CloudTrail

**[📖 AWS CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)** - Track user activity and API usage

### Features
- API call logging and auditing
- Records user activity and API usage
- **Management Events**: Control plane operations (default, free)
- **Data Events**: Resource operations (S3 object, Lambda invocations) - charged
- **Insights Events**: Unusual API activity detection

**[📖 CloudTrail Events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html)** - Understand CloudTrail event types

### Trail Types
- **Single-region**: Logs events in one region
- **Multi-region**: Logs events in all regions (recommended)
- **Organization Trail**: All accounts in AWS Organization

### Log Delivery
- S3 bucket (required)
- CloudWatch Logs (optional)
- EventBridge (optional)

### Use Cases
- Security analysis and compliance
- Track changes to resources
- Troubleshooting operational issues
- Security incident investigation
- Audit user and API activity

### Best Practices
- Enable in all regions
- Encrypt logs with KMS
- Enable log file validation
- Integrate with CloudWatch Logs for alarms
- Separate logging account for security

## AWS Config

**[📖 AWS Config Documentation](https://docs.aws.amazon.com/config/)** - Track resource configurations and compliance

### Features
- Resource inventory and configuration history
- Configuration change notifications
- Compliance auditing with Config Rules

### Config Rules

**AWS Managed Rules**:
- required-tags: Ensure resources have required tags
- encrypted-volumes: EBS volumes are encrypted
- rds-multi-az-support: RDS has Multi-AZ enabled
- s3-bucket-public-read-prohibited: S3 buckets not publicly readable

**Custom Rules**:
- Lambda functions to evaluate compliance
- Evaluate periodically or on configuration changes

**Remediation**:
- **Automatic**: SSM Automation documents
- **Manual**: Compliance dashboard review

### Use Cases
- Resource relationship tracking
- Compliance as code
- Security analysis
- Change management

## AWS Systems Manager

**[📖 AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)** - Unified interface to manage AWS resources

### Parameter Store
- Secure storage for configuration and secrets
- Hierarchical structure (/app/dev/db-password)
- **Standard**: Free, 4 KB value size, 10,000 parameters
- **Advanced**: $0.05/month, 8 KB size, 100,000 parameters
- Encryption with KMS
- Integration with CloudFormation, Lambda, ECS

**[📖 Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)** - Centralized store for configuration data

### Session Manager
- Secure shell access without SSH keys or bastion hosts
- Audit and log sessions
- No inbound ports required
- IAM-based access control
- Use Case: Secure EC2 access

### Run Command
- Execute commands on EC2 instances remotely
- No SSH required
- Rate control and error handling
- Use Cases: Patching, configuration, automation

### Patch Manager
- Automate OS and application patching
- Patch baselines and maintenance windows
- Compliance reporting
- Pre-defined or custom patch baselines

### Automation
- Runbooks for common tasks
- AWS-provided or custom
- Use Cases: AMI creation, disaster recovery

### State Manager
- Maintain desired configuration state
- Association with targets
- Use Case: Ensure CloudWatch agent running

## AWS Trusted Advisor

**[📖 AWS Trusted Advisor](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html)** - Real-time guidance to provision resources following best practices

### Categories
1. **Cost Optimization**: Idle resources, reserved capacity
2. **Performance**: Service limits, throughput
3. **Security**: IAM, security groups, MFA
4. **Fault Tolerance**: RDS backups, Multi-AZ
5. **Service Limits**: Approaching AWS limits

### Support Plans
- **Basic/Developer**: Core checks (7)
  - S3 bucket permissions
  - Security groups with unrestricted access
  - IAM use
  - MFA on root
  - EBS public snapshots
  - RDS public snapshots
  - Service limits
- **Business/Enterprise**: All checks (115+)
  - Plus CloudWatch integration
  - Weekly notifications
  - Programmatic access (API)

## AWS Health Dashboard

### Service Health Dashboard
- Overall AWS service status
- Public information
- Regional view
- Historical data

### Personal Health Dashboard
- Personalized view of AWS service health
- Alerts for events affecting your resources
- Proactive notifications
- Integration with EventBridge

### Use Cases
- Monitor ongoing AWS issues
- Plan for scheduled mainten ance
- Automate responses to health events

## Amazon EventBridge

### Features
- Serverless event bus
- Successor to CloudWatch Events
- Schema registry
- Event archive and replay

**Event Sources**:
- AWS services
- Custom applications (PutEvents API)
- SaaS partners (Datadog, Zendesk, etc.)

**Targets**: 20+ AWS services

**Event Buses**:
- Default: AWS services
- Custom: Your applications
- Partner: SaaS providers

### Schema Registry
- Discover event structure
- Generate code bindings
- Version management

## AWS Service Catalog

### Features
- Create and manage catalogs of approved products
- Governance and compliance
- Self-service for end users
- Version control for templates

**Components**:
- **Products**: CloudFormation templates
- **Portfolios**: Collection of products
- **Constraints**: Limits on products (launch, template)

### Use Cases
- Standardize deployments
- Governance and compliance
- Self-service IT for users

## AWS Cost Management

### AWS Cost Explorer
- Visualize and analyze costs
- Forecasting
- Filter by service, tag, account
- Savings Plans recommendations

### AWS Budgets
- Set custom cost and usage budgets
- Alerts when exceeding threshold
- Budget types: Cost, usage, reservation, Savings Plans

### AWS Cost and Usage Report
- Most detailed billing data
- Delivered to S3
- Integration with Athena, QuickSight, Redshift

### AWS Compute Optimizer
- ML-based recommendations
- Right-sizing for EC2, Auto Scaling, EBS, Lambda
- Performance risk assessment

## Exam Tips

### Monitoring Scenarios
- **Application performance issues**: X-Ray
- **Infrastructure metrics**: CloudWatch metrics
- **Log aggregation and analysis**: CloudWatch Logs
- **Compliance and configuration tracking**: AWS Config
- **API auditing**: CloudTrail
- **Cost optimization recommendations**: Trusted Advisor, Compute Optimizer
- **Automated responses to events**: EventBridge
- **Secure EC2 access**: Systems Manager Session Manager

### Best Practices
1. Enable CloudTrail in all regions
2. Use CloudWatch alarms for critical metrics
3. Centralize logs in CloudWatch Logs
4. Enable AWS Config for compliance
5. Regular review of Trusted Advisor
6. Use X-Ray for distributed applications
7. Implement automated remediation with Config
8. Tag resources for cost allocation
9. Set up billing alarms and budgets
10. Use Systems Manager for fleet management

### Key Differences
- **CloudWatch vs CloudTrail**: Metrics/Logs vs API calls
- **CloudWatch Logs vs S3**: Real-time search vs archive
- **Parameter Store vs Secrets Manager**: General config vs secrets with rotation
- **Config vs CloudTrail**: Configuration changes vs API calls
