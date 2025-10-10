# Monitoring, Logging, and Remediation (SOA-C02)

This domain covers 20% of the exam and focuses on implementing effective monitoring, logging, and automated remediation solutions.

## Amazon CloudWatch

### CloudWatch Metrics

**Standard Metrics**
- Automatically collected for most AWS services
- Default 5-minute intervals (detailed monitoring: 1-minute)
- Cannot be deleted but expire after 15 months
- Free tier: 10 custom metrics

**Custom Metrics**
- Published using PutMetricData API
- Resolution: Standard (60s) or High (1s)
- Dimensions for filtering (max 30 dimensions per metric)
- StorageResolution parameter for high-resolution metrics

**Key Metrics by Service**
- **EC2**: CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps, StatusCheckFailed
- **EBS**: VolumeReadBytes/WriteBytes, VolumeReadOps/WriteOps, VolumeQueueLength
- **RDS**: DatabaseConnections, FreeableMemory, ReadLatency, WriteLatency
- **ELB**: HealthyHostCount, UnHealthyHostCount, RequestCount, TargetResponseTime
- **Lambda**: Invocations, Duration, Errors, Throttles, ConcurrentExecutions

### CloudWatch Alarms

**Alarm States**
- **OK**: Metric within defined threshold
- **ALARM**: Metric breached threshold
- **INSUFFICIENT_DATA**: Not enough data to determine state

**Alarm Configuration**
- **Metric**: What to monitor
- **Threshold**: Static threshold or anomaly detection
- **Period**: Evaluation period length
- **Datapoints to alarm**: How many periods must breach before alarming
- **Actions**: SNS notifications, Auto Scaling, EC2 actions, Systems Manager actions

**Composite Alarms**
- Combine multiple alarms using AND/OR logic
- Reduce alarm noise
- Create complex alarm conditions

**Alarm Actions**
- **EC2 Actions**: Stop, terminate, reboot, recover
- **Auto Scaling**: Scale up/down based on metrics
- **SNS Notifications**: Email, SMS, Lambda, HTTP/S
- **Systems Manager**: Execute automation documents

### CloudWatch Logs

**Log Groups and Streams**
- Log Group: Container for log streams (e.g., /aws/lambda/function-name)
- Log Stream: Sequence of log events from same source
- Retention: 1 day to 10 years, or never expire
- Encryption: KMS encryption at rest

**Log Sources**
- EC2 instances (CloudWatch agent)
- Lambda functions (automatic)
- VPC Flow Logs
- CloudTrail logs
- Route 53 DNS queries
- RDS, ECS, EKS logs

**CloudWatch Logs Insights**
- Query language for log analysis
- Common queries: error analysis, performance metrics, security events
- Visualization with charts and graphs
- Query syntax: `fields`, `filter`, `stats`, `sort`, `limit`

**Example Queries**
```
# Find errors in last hour
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100

# Count errors by type
fields @message
| filter @message like /ERROR/
| stats count() by @message

# Average response time
fields @duration
| stats avg(@duration), max(@duration), min(@duration)
```

**Log Subscriptions**
- Stream log data to Kinesis, Lambda, or OpenSearch
- Real-time processing of log data
- Multiple subscriptions per log group (1 per destination type)

**Metric Filters**
- Extract metric data from logs
- Define patterns to match in log events
- Publish to CloudWatch metrics
- Use for custom alarms based on log content

### CloudWatch Dashboards

**Dashboard Features**
- Customizable visualizations (line, number, gauge)
- Cross-region and cross-account views
- Automatic refresh (10s, 1m, 2m, 5m, 15m)
- Sharing capabilities

**Widget Types**
- Line graphs for metrics over time
- Stacked area for cumulative metrics
- Number widgets for current values
- Text widgets for documentation
- Log widgets for Logs Insights queries

### CloudWatch Events / EventBridge

**Event Sources**
- AWS services (state changes, API calls)
- Custom applications (PutEvents API)
- Scheduled events (cron or rate expressions)

**Event Patterns**
- Match event structure and content
- Filter events to specific targets
- JSON-based pattern matching

**Targets**
- Lambda functions
- SNS topics, SQS queues
- Systems Manager automation
- Step Functions
- CodePipeline, CodeBuild
- EC2 actions (start/stop instances)

**Common Use Cases**
- Auto-remediation of security findings
- Scheduled Lambda execution
- Automated backup triggers
- Instance state change notifications
- CloudTrail event monitoring

## AWS Systems Manager

### Session Manager

**Benefits**
- No SSH keys, bastion hosts, or open inbound ports
- Centralized access control with IAM
- Audit trail in CloudTrail
- Session logging to S3 or CloudWatch Logs
- Support for Linux and Windows

**Configuration**
- IAM permissions for users
- IAM role for EC2 instances
- SSM agent installed (pre-installed on AWS AMIs)
- VPC endpoints for private subnets (optional)

### Run Command

**Features**
- Execute commands across multiple instances
- Rate control: concurrent executions and error thresholds
- Command history and status tracking
- Integration with EventBridge for automation

**Common Document Types**
- AWS-RunShellScript (Linux)
- AWS-RunPowerShellScript (Windows)
- AWS-ConfigureAWSPackage
- AWS-InstallApplication

### Patch Manager

**Patch Baselines**
- Predefined baselines for each OS
- Custom baselines for specific requirements
- Auto-approval rules (by severity, classification)
- Patch groups for organizing instances

**Maintenance Windows**
- Scheduled patching windows
- Duration and cutoff time
- Rate controls (concurrency and error threshold)
- SNS notifications for patch status

**Patching Workflow**
1. Define patch baseline
2. Create maintenance window
3. Assign targets (patch groups)
4. Schedule patch task
5. Monitor compliance

**Compliance Reporting**
- Patch compliance status per instance
- Missing patches by severity
- Integration with AWS Config
- CloudWatch dashboards for visualization

### State Manager

**Purpose**
- Maintain consistent instance configuration
- Scheduled association execution
- Automatic compliance checking
- Drift detection and remediation

**Associations**
- Link SSM document to target instances
- Schedule: rate or cron expression
- Parameters for document execution
- Output location (S3, CloudWatch Logs)

### Parameter Store

**Parameter Types**
- **String**: Plain text values
- **StringList**: Comma-separated values
- **SecureString**: Encrypted with KMS

**Features**
- Hierarchical organization (/app/db/password)
- Versioning and change history
- TTL for parameter values
- CloudWatch Events on parameter changes
- Integration with other AWS services

**Standard vs Advanced**
- Standard: Up to 10,000 parameters, 4KB size, free
- Advanced: 100,000+ parameters, 8KB size, charges apply

### Automation

**Automation Documents**
- Predefined runbooks for common tasks
- Custom automation documents
- Multi-step workflows
- Integration with approval workflows

**Common Use Cases**
- AMI creation and patching
- Instance state management
- Backup automation
- Compliance remediation
- Disaster recovery procedures

## AWS Config

### Configuration Recording

**Resource Types**
- Record all supported resources or select specific types
- Global resources (IAM, CloudFront) in one region only
- Configuration snapshots and history
- Store in S3 bucket

**Configuration Items**
- Resource metadata and relationships
- Configuration changes over time
- Resource tags
- Related CloudTrail events

### Config Rules

**Managed Rules**
- Pre-built rules for common compliance checks
- Examples:
  - required-tags: Ensure resources have mandatory tags
  - encrypted-volumes: Check EBS encryption
  - rds-multi-az-support: Verify RDS Multi-AZ
  - s3-bucket-public-read-prohibited

**Custom Rules**
- Lambda function evaluates compliance
- Triggered by configuration changes or periodic
- Return compliance status (COMPLIANT, NON_COMPLIANT)

**Remediation**
- Automatic remediation with Systems Manager documents
- Manual remediation options
- Remediation retries
- Logging remediation actions

### Compliance Monitoring

**Compliance Dashboard**
- Overall compliance status
- Compliance by rule
- Non-compliant resources
- Timeline of compliance changes

**Conformance Packs**
- Collection of Config rules and remediation actions
- Predefined packs (PCI-DSS, HIPAA, etc.)
- Custom conformance packs
- Organization-wide deployment

## AWS CloudTrail

### Event Types

**Management Events**
- Control plane operations
- Creating, modifying, deleting resources
- Configuring security and routing
- Recorded by default

**Data Events**
- Data plane operations
- S3 object-level API activity
- Lambda function executions
- Not recorded by default (additional cost)

**Insights Events**
- Detect unusual activity patterns
- Analyze normal baseline activity
- Alert on anomalies (API call rate, error rate)

### Trail Configuration

**Single vs Multi-Region**
- Single region trail: Events in one region only
- Multi-region trail: Events from all regions
- Recommended: Multi-region for complete audit

**Organization Trails**
- Apply to all accounts in AWS Organization
- Centralized logging account
- Consolidated security and compliance

**Log File Integrity**
- Validate log file authenticity
- Digest files with hash values
- Detect tampering or deletion

**S3 Storage**
- Centralized log storage
- Lifecycle policies for cost optimization
- Cross-region replication for DR
- Server-side encryption

### Integration with Other Services

**CloudWatch Logs**
- Stream CloudTrail logs to CloudWatch
- Metric filters for security events
- Real-time alerting on suspicious activity

**EventBridge**
- Trigger actions based on API calls
- Near real-time event processing
- Automated response to security events

**Athena**
- Query CloudTrail logs using SQL
- Analyze access patterns
- Security investigations

## VPC Flow Logs

### Flow Log Configuration

**Scope**
- VPC level: All ENIs in VPC
- Subnet level: All ENIs in subnet
- ENI level: Specific network interface

**Traffic Types**
- ACCEPT: Accepted traffic
- REJECT: Rejected traffic
- ALL: Both accepted and rejected

**Destinations**
- CloudWatch Logs
- S3 bucket
- Kinesis Data Firehose

### Flow Log Fields

**Default Fields**
- srcaddr, dstaddr: Source and destination IP
- srcport, dstport: Source and destination port
- protocol: IANA protocol number
- packets, bytes: Packet and byte counts
- start, end: Flow start and end time
- action: ACCEPT or REJECT
- log-status: Logging status

**Custom Fields**
- vpc-id, subnet-id, instance-id
- flow-direction (ingress/egress)
- pkt-srcaddr, pkt-dstaddr (actual packet IPs)

### Use Cases

**Security Analysis**
- Identify rejected connection attempts
- Detect port scanning
- Analyze traffic patterns
- Investigate security incidents

**Network Troubleshooting**
- Diagnose connectivity issues
- Verify Security Group rules
- Check NACL configurations
- Analyze traffic flows

**Compliance**
- Network traffic audit trail
- Data transfer monitoring
- Geographic traffic analysis

## Automated Remediation

### EventBridge Rules

**Architecture**
```
CloudWatch/CloudTrail -> EventBridge -> Lambda -> Remediation
```

**Common Patterns**
1. Security Group changes -> Revert unauthorized changes
2. Non-compliant resources -> Apply compliant configuration
3. Failed health checks -> Restart/replace instances
4. Cost anomalies -> Stop over-provisioned resources

### Lambda for Remediation

**Best Practices**
- Idempotent operations (safe to retry)
- Error handling and logging
- Dead letter queue for failures
- IAM least privilege
- Timeout configuration
- Asynchronous invocation for long tasks

### Systems Manager Automation

**Advantages**
- Pre-built automation documents
- No code required for common tasks
- Approval workflows
- Multi-step orchestration
- Built-in error handling

**Integration Points**
- EventBridge triggers automation
- Config remediation actions
- Manual execution via console/API
- Scheduled execution

## SNS for Notifications

### Topic Types

**Standard Topics**
- At-least-once delivery
- Best-effort ordering
- Higher throughput

**FIFO Topics**
- Exactly-once delivery
- Strict message ordering
- Deduplication

### Subscriptions

**Protocol Options**
- Email/Email-JSON
- HTTP/HTTPS endpoints
- SMS
- SQS queues
- Lambda functions
- Mobile push notifications

**Subscription Filter Policies**
- JSON-based message filtering
- Reduce unnecessary invocations
- Cost optimization

### Integration Patterns

**CloudWatch Alarms**
- Direct integration with alarms
- Alarm state change notifications
- Composite alarm aggregation

**EventBridge**
- SNS as event target
- Fan-out to multiple subscribers
- Message transformation

## Best Practices

### Monitoring Strategy

1. **Define Key Metrics**: Identify critical business and technical metrics
2. **Set Appropriate Thresholds**: Avoid alarm fatigue
3. **Use Composite Alarms**: Reduce false positives
4. **Implement Anomaly Detection**: Catch unusual patterns
5. **Create Dashboards**: Visualize system health

### Logging Strategy

1. **Centralize Logs**: Single location for all logs
2. **Retention Policies**: Balance compliance and cost
3. **Log Aggregation**: Use subscriptions for analysis
4. **Structured Logging**: JSON format for easier parsing
5. **Sensitive Data**: Avoid logging PII, credentials

### Remediation Strategy

1. **Automate Common Issues**: Reduce MTTR
2. **Testing**: Test remediation in non-production
3. **Rollback Capability**: Plan for remediation failures
4. **Notification**: Alert on remediation actions
5. **Documentation**: Maintain runbook for manual intervention

### Cost Optimization

1. **Metric Retention**: Adjust based on needs
2. **Log Retention**: Use lifecycle policies
3. **Custom Metrics**: Only essential metrics
4. **Alarm Optimization**: Consolidate related alarms
5. **Data Transfer**: Use VPC endpoints to avoid charges

## Exam Tips

### Key Concepts to Remember

- CloudWatch agent required for memory and disk metrics
- Standard metrics: 5 minutes; Detailed: 1 minute
- High-resolution custom metrics: Up to 1 second
- Logs Insights: Ad-hoc query; Metric filters: Ongoing monitoring
- EventBridge: Event-driven automation (formerly CloudWatch Events)
- Session Manager: No bastion hosts or SSH keys needed
- Parameter Store: Free for standard, charged for advanced
- Config Rules: Evaluate compliance; Remediation: Fix issues
- CloudTrail: API audit trail; Management events free
- VPC Flow Logs: Network traffic analysis

### Common Scenarios

**Scenario**: Monitor custom application metrics
**Solution**: Install CloudWatch agent, publish custom metrics, create alarms

**Scenario**: Automated security group remediation
**Solution**: CloudTrail -> EventBridge -> Lambda -> Revert changes

**Scenario**: Patch EC2 instances monthly
**Solution**: Patch Manager with maintenance window and patch baseline

**Scenario**: Alert on repeated failed SSH attempts
**Solution**: VPC Flow Logs -> Metric filter -> CloudWatch alarm -> SNS

**Scenario**: Compliance reporting for encrypted EBS volumes
**Solution**: AWS Config rule (encrypted-volumes) with dashboard

### Troubleshooting

**Issue**: Metrics not appearing
- Check CloudWatch agent status and configuration
- Verify IAM permissions
- Check metric namespace and dimensions

**Issue**: Alarms stuck in INSUFFICIENT_DATA
- Not enough data points
- Metric not being published
- Period too short for available data

**Issue**: Session Manager connection fails
- SSM agent not running
- Missing IAM role on instance
- No route to Systems Manager endpoints

**Issue**: CloudTrail events missing
- Trail not enabled in region
- S3 bucket permissions incorrect
- Event type not captured (data events)
