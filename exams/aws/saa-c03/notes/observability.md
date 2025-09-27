# AWS Observability Fundamentals

## Observability Overview

### What is Observability?
The ability to understand the internal state of a system by examining its external outputs, including metrics, logs, and traces.

### Three Pillars of Observability
| Pillar | Description | Use Case |
|--------|-------------|----------|
| **Metrics** | Numerical measurements over time | Performance monitoring, alerting |
| **Logs** | Event records from applications/systems | Troubleshooting, audit trails |
| **Traces** | Request journey through distributed systems | Performance analysis, debugging |

### AWS Observability Services
| Service | Purpose | Data Type |
|---------|---------|-----------|
| **CloudWatch** | Metrics, logs, alarms | Metrics, logs |
| **X-Ray** | Distributed tracing | Traces |
| **CloudTrail** | API audit logging | Audit logs |
| **AWS Config** | Configuration compliance | Configuration data |
| **Systems Manager** | Operational insights | Operational data |

## Amazon CloudWatch

### What is CloudWatch?
Monitoring and observability service that provides data and actionable insights for AWS resources and applications.

### CloudWatch Components
- **Metrics**: Time-ordered data points
- **Alarms**: Notifications based on metric thresholds
- **Logs**: Centralized log management
- **Events/EventBridge**: Event-driven automation
- **Dashboards**: Visual monitoring displays

### CloudWatch Metrics

#### Metric Fundamentals
- **Namespace**: Container for metrics (e.g., AWS/EC2, AWS/RDS)
- **Metric Name**: Name of the metric (e.g., CPUUtilization)
- **Dimensions**: Name/value pairs that identify metric (e.g., InstanceId=i-1234567890abcdef0)
- **Timestamp**: When the metric was recorded
- **Value**: The measurement value

#### Default Metrics by Service
| Service | Key Metrics | Update Frequency |
|---------|-------------|------------------|
| **EC2** | CPUUtilization, NetworkIn/Out, DiskReadOps | 5 minutes (1 min with detailed monitoring) |
| **RDS** | CPUUtilization, DatabaseConnections, ReadLatency | 1 minute |
| **S3** | BucketSizeBytes, NumberOfObjects | Daily |
| **Lambda** | Duration, Invocations, Errors | Real-time |
| **DynamoDB** | ConsumedReadCapacityUnits, UserErrors | 1 minute |

#### Custom Metrics
```bash
# AWS CLI example
aws cloudwatch put-metric-data \
  --namespace "MyApp/Queue" \
  --metric-data MetricName=QueueDepth,Value=10,Unit=Count
```

#### Metric Math
Create calculated metrics from existing metrics:
```
CPU_Credit_Balance_Trend = RATE(CPU_Credit_Balance)
Error_Rate = Errors / Invocations * 100
```

### CloudWatch Alarms

#### Alarm States
| State | Description | Meaning |
|-------|-------------|---------|
| **OK** | Metric is within threshold | System healthy |
| **ALARM** | Metric breached threshold | Action required |
| **INSUFFICIENT_DATA** | Not enough data | Cannot determine state |

#### Alarm Configuration
- **Metric**: Which metric to monitor
- **Threshold**: Value that triggers alarm
- **Comparison operator**: How to compare (>, <, >=, <=)
- **Period**: Time window for evaluation
- **Evaluation periods**: How many periods to check
- **Datapoints to alarm**: How many breaches needed

#### Alarm Actions
- **SNS notifications**: Send emails, SMS, or trigger other services
- **Auto Scaling actions**: Scale EC2 instances up/down
- **EC2 actions**: Stop, terminate, reboot instances
- **Systems Manager actions**: Execute automation documents

#### Example: High CPU Alarm
```
Metric: EC2 CPUUtilization
Threshold: 80%
Comparison: Greater than
Period: 5 minutes
Evaluation periods: 2
Datapoints to alarm: 2
Action: Send SNS notification
```

### CloudWatch Logs

#### Log Concepts
- **Log Groups**: Collection of log streams with same retention/permissions
- **Log Streams**: Sequence of log events from same source
- **Log Events**: Individual log entries with timestamp and message
- **Retention Period**: How long to keep logs (never expire to 10 years)

#### Common Log Sources
| Source | Log Group Pattern | Use Case |
|--------|-------------------|----------|
| **Lambda** | /aws/lambda/function-name | Function execution logs |
| **API Gateway** | API-Gateway-Execution-Logs_api-id/stage | API request/response logs |
| **VPC Flow Logs** | /aws/vpc/flowlogs | Network traffic analysis |
| **CloudTrail** | /aws/cloudtrail/trail-name | API audit logs |
| **EC2** | Custom | Application and system logs |

#### CloudWatch Logs Insights
SQL-like query language for log analysis:

```sql
-- Find errors in Lambda logs
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

-- API Gateway latency analysis
fields @timestamp, @duration
| filter @type = "REPORT"
| stats avg(@duration), max(@duration), min(@duration) by bin(5m)
```

#### Log Retention and Cost
| Retention Period | Cost per GB |
|------------------|-------------|
| **1-3 days** | Lower | 
| **1 week** | Low |
| **1 month** | Medium |
| **1 year** | Higher |
| **Never expire** | Highest |

### CloudWatch Dashboards

#### Dashboard Components
- **Widgets**: Individual visualization components
- **Metrics widgets**: Line charts, stacked area, number displays
- **Log widgets**: Log query results
- **Text widgets**: Markdown documentation
- **Custom widgets**: Lambda-powered visualizations

#### Dashboard Best Practices
- **Golden signals**: Focus on latency, traffic, errors, saturation
- **Hierarchical view**: High-level overview with drill-down capability
- **Consistent time ranges**: Align widgets for correlation
- **Annotations**: Mark deployments, incidents, maintenance windows

## AWS X-Ray

### What is X-Ray?
Distributed tracing service that helps analyze and debug distributed applications.

### X-Ray Concepts
- **Trace**: Complete request journey through application
- **Segment**: Work done by single service
- **Subsegment**: More granular work within segment
- **Service Map**: Visual representation of application architecture
- **Annotations**: Key-value pairs for filtering traces
- **Metadata**: Additional context information

### X-Ray Integration
#### Supported Services
- **Lambda**: Automatic tracing with environment variable
- **API Gateway**: Enable X-Ray tracing in stage settings
- **EC2/ECS**: Install X-Ray daemon and instrument code
- **Elastic Beanstalk**: Enable in configuration
- **EKS**: Deploy X-Ray DaemonSet

#### Language Support
- **Java**: X-Ray SDK for Java
- **Node.js**: X-Ray SDK for Node.js
- **Python**: X-Ray SDK for Python
- **.NET**: X-Ray SDK for .NET
- **Go**: X-Ray SDK for Go
- **Ruby**: X-Ray SDK for Ruby

### X-Ray Sampling
Control the amount of request data recorded:

#### Sampling Rules
```json
{
  "version": 2,
  "default": {
    "fixed_target": 2,
    "rate": 0.1
  },
  "rules": [
    {
      "description": "Player moves",
      "service_name": "Scorekeep",
      "http_method": "POST", 
      "url_path": "/api/move/*",
      "fixed_target": 0,
      "rate": 0.05
    }
  ]
}
```

- **Fixed target**: Minimum traces per second
- **Rate**: Percentage of additional requests to trace

### Service Map Analysis
Visual representation showing:
- **Services**: Application components
- **Connections**: How services communicate
- **Performance**: Response times and error rates
- **Bottlenecks**: Slow services in request path

## AWS CloudTrail

### What is CloudTrail?
Service that provides governance, compliance, operational auditing, and risk auditing of AWS account activity.

### CloudTrail Events
#### Management Events
- **Control plane operations**: Creating, deleting, modifying resources
- **Examples**: RunInstances, CreateBucket, PutBucketPolicy
- **Logging**: Enabled by default for all AWS accounts

#### Data Events
- **Data plane operations**: Accessing or modifying data
- **Examples**: GetObject, PutObject, DynamoDB operations
- **Logging**: Must be explicitly enabled (additional charges)

#### Insight Events
- **Unusual activity patterns**: Detected by machine learning
- **Examples**: Spike in IAM actions, unusual API call patterns
- **Logging**: Optional feature with additional charges

### CloudTrail Configuration
#### Trail Types
| Type | Scope | Use Case |
|------|-------|----------|
| **Single region** | One AWS region | Region-specific compliance |
| **Multi-region** | All regions | Comprehensive audit trail |
| **Organization trail** | Entire AWS Organization | Centralized logging |

#### Log File Delivery
- **S3 bucket**: Where log files are stored
- **S3 key prefix**: Organize logs by date/service
- **SNS notification**: Alert when new logs arrive
- **CloudWatch Logs**: Real-time log analysis

### CloudTrail Log Analysis
#### Log Structure
```json
{
  "Records": [
    {
      "eventTime": "2023-01-15T12:00:00Z",
      "eventName": "AssumeRole",
      "eventSource": "sts.amazonaws.com",
      "userIdentity": {
        "type": "IAMUser",
        "principalId": "AIDACKCEVSQ6C2EXAMPLE",
        "arn": "arn:aws:iam::123456789012:user/johndoe",
        "accountId": "123456789012",
        "userName": "johndoe"
      },
      "sourceIPAddress": "192.0.2.0"
    }
  ]
}
```

#### Common Analysis Queries
```sql
-- Find root account usage
SELECT eventTime, sourceIPAddress, userAgent
FROM cloudtrail_logs 
WHERE userIdentity.type = 'Root'

-- Track resource deletions
SELECT eventTime, eventName, userIdentity.userName, sourceIPAddress
FROM cloudtrail_logs 
WHERE eventName LIKE '%Delete%'
```

## AWS Config

### What is Config?
Service that assesses, audits, and evaluates configurations of AWS resources for compliance.

### Config Components
- **Configuration Items (CIs)**: Point-in-time configuration snapshots
- **Configuration History**: Timeline of resource changes
- **Configuration Snapshots**: Complete resource inventory
- **Config Rules**: Compliance evaluation logic
- **Remediation**: Automatic fixes for non-compliant resources

### Config Rules
#### AWS Managed Rules
Common compliance checks:
- **s3-bucket-public-access-prohibited**: S3 buckets shouldn't be public
- **ec2-security-group-attached-to-eni**: Security groups should be attached
- **rds-storage-encrypted**: RDS storage should be encrypted
- **iam-password-policy**: IAM password complexity requirements

#### Custom Rules
Lambda functions that evaluate resource compliance:
```python
def lambda_handler(event, context):
    # Get the configuration item
    config_item = event['configurationItem']
    
    # Evaluation logic
    compliance_type = 'COMPLIANT'
    if not is_compliant(config_item):
        compliance_type = 'NON_COMPLIANT'
    
    return {
        'complianceType': compliance_type,
        'annotation': 'Resource compliance evaluation'
    }
```

### Config Remediation
Automatic fixes for non-compliant resources:
- **SSM documents**: Predefined remediation actions
- **Custom Lambda**: Custom remediation logic
- **Manual remediation**: Require human intervention

## Monitoring Best Practices

### Golden Signals
Focus on these four key metrics:
1. **Latency**: How long requests take
2. **Traffic**: How much demand on system
3. **Errors**: Rate of failed requests
4. **Saturation**: How "full" your service is

### Alerting Strategy
#### Alert Levels
| Level | Criteria | Response |
|-------|----------|----------|
| **Critical** | Service down, data loss | Immediate response required |
| **Warning** | Performance degradation | Response within hours |
| **Info** | Unusual but not problematic | Review during business hours |

#### Alert Fatigue Prevention
- **Actionable alerts**: Only alert on what requires action
- **Proper thresholds**: Avoid false positives
- **Alert escalation**: Route to appropriate teams
- **Alert correlation**: Group related alerts

### Dashboard Design
#### Executive Dashboard
- **High-level KPIs**: Business metrics
- **Service health**: Overall system status
- **Cost metrics**: Spending trends
- **Security alerts**: Security incidents

#### Operational Dashboard
- **Service metrics**: Application performance
- **Infrastructure metrics**: Server health
- **Error rates**: Failure analysis
- **Deployment status**: Release tracking

### Log Management
#### Log Aggregation Strategy
- **Centralized logging**: All logs in one place
- **Structured logging**: JSON format for easier parsing
- **Log levels**: DEBUG, INFO, WARN, ERROR, FATAL
- **Correlation IDs**: Track requests across services

#### Log Retention Policy
| Log Type | Retention Period | Reason |
|----------|------------------|--------|
| **Security logs** | 2+ years | Compliance requirements |
| **Application logs** | 30-90 days | Troubleshooting |
| **Debug logs** | 7-14 days | Development debugging |
| **Access logs** | 1 year | Audit requirements |

## Cost Optimization for Observability

### CloudWatch Cost Factors
- **Custom metrics**: $0.30 per metric per month
- **API requests**: $0.01 per 1,000 requests
- **Alarms**: $0.10 per alarm per month
- **Logs ingestion**: $0.50 per GB
- **Log storage**: $0.03 per GB per month

### Cost Optimization Strategies
- **Metric filtering**: Only collect needed metrics
- **Log retention**: Set appropriate retention periods
- **Sampling**: Use sampling for high-volume traces
- **Reserved capacity**: For predictable log volumes
- **Cross-region replication**: Only when necessary

## Common Observability Pitfalls

### Monitoring Issues
- **Alert fatigue**: Too many false positive alerts
- **Blind spots**: Missing monitoring for critical components
- **Reactive monitoring**: Only alerting after problems occur
- **Poor correlation**: Can't connect metrics to business impact

### Logging Problems
- **Log verbosity**: Too much or too little logging
- **Unstructured logs**: Difficult to parse and analyze
- **Missing context**: Logs don't provide enough information
- **Log sprawl**: Logs scattered across multiple systems

### Tracing Challenges
- **High overhead**: Performance impact from tracing
- **Incomplete traces**: Missing spans in request path
- **Sampling issues**: Missing important low-volume requests
- **Complex correlation**: Difficulty tracking requests across services

### Performance Impact
- **Over-instrumentation**: Too much observability overhead
- **Synchronous logging**: Blocking application performance
- **Large payloads**: Expensive metric/log transmission
- **Inefficient queries**: Slow log analysis queries