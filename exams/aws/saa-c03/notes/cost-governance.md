# AWS Cost Management & Governance Fundamentals

## Cost Management Overview

### AWS Pricing Model
**Pay-as-you-go**: Only pay for what you use, when you use it, with no long-term contracts or complex licensing.

### Cost Components
| Component | Description | Examples |
|-----------|-------------|----------|
| **Compute** | Processing power | EC2 instances, Lambda executions |
| **Storage** | Data storage | S3, EBS volumes, snapshots |
| **Network** | Data transfer | Cross-region transfer, internet egress |
| **Requests** | API calls and operations | S3 requests, Lambda invocations |

### Regional Pricing Differences
- **US East (N. Virginia)**: Often lowest prices, first region for new services
- **US West**: Slightly higher than US East
- **Europe**: Generally 10-20% higher than US
- **Asia Pacific**: Varies significantly by country
- **Other regions**: Can be significantly higher

## AWS Pricing Models

### On-Demand Pricing
**Standard pay-as-you-go model**

#### Characteristics
- **No upfront costs**: Start using immediately
- **No commitments**: Scale up or down anytime
- **Predictable pricing**: Fixed hourly/per-second rates
- **Full flexibility**: Change instance types, regions

#### Best For
- **Unpredictable workloads**: Variable traffic patterns
- **Short-term projects**: Development, testing, experimentation
- **First-time users**: Learning AWS services
- **Spike handling**: Temporary capacity increases

### Reserved Instances (RIs)
**Capacity reservations with significant discounts**

#### RI Types
| Type | Commitment | Discount | Flexibility |
|------|------------|----------|-------------|
| **Standard** | 1-3 years | Up to 75% | Limited modifications |
| **Convertible** | 1-3 years | Up to 54% | Can change instance family |
| **Scheduled** | 1 year | Varies | Specific time windows |

#### RI Attributes
- **Instance type**: m5.large, c5.xlarge, etc.
- **Region/AZ**: Where RI applies
- **Tenancy**: Default, dedicated, host
- **Platform**: Linux, Windows, RHEL, etc.

#### Payment Options
| Option | Upfront | Ongoing | Total Discount |
|--------|---------|---------|----------------|
| **No Upfront** | 0% | Monthly | Lowest |
| **Partial Upfront** | ~50% | Reduced monthly | Medium |
| **All Upfront** | 100% | No monthly | Highest |

### Savings Plans
**Flexible pricing model offering lower prices**

#### Savings Plans Types
| Type | Scope | Discount | Flexibility |
|------|-------|----------|-------------|
| **Compute** | EC2, Lambda, Fargate | Up to 66% | High flexibility |
| **EC2 Instance** | EC2 only | Up to 72% | Medium flexibility |

#### Compute Savings Plans
- **Applies to**: EC2, Lambda, Fargate usage
- **Flexibility**: Change instance type, size, OS, tenancy, region
- **Commitment**: $/hour for 1 or 3 years
- **Best for**: Mixed workloads, evolving architectures

#### EC2 Instance Savings Plans
- **Applies to**: EC2 usage in specific instance family and region
- **Flexibility**: Change size within family, OS, tenancy
- **Commitment**: Instance family in specific region
- **Best for**: Stable EC2 workloads

### Spot Instances
**Use spare AWS capacity at up to 90% discount**

#### How Spot Works
1. **Bid price**: Set maximum price willing to pay
2. **Spot price**: Current market price for capacity
3. **Instance runs**: When bid ≥ spot price
4. **Interruption**: 2-minute warning when capacity needed

#### Spot Best Practices
- **Fault-tolerant applications**: Handle interruptions gracefully
- **Flexible timing**: Can be interrupted and restarted
- **Diversification**: Multiple instance types and AZs
- **Spot Fleet**: Automatically request lowest-price instances

#### Spot Use Cases
- **Batch processing**: Data analysis, image rendering
- **CI/CD**: Build and test environments
- **Big data**: Hadoop, Spark clusters
- **Web scraping**: Large-scale data collection
- **Machine learning**: Training workloads

## AWS Cost Tools

### AWS Cost Explorer
**Visualize and analyze AWS costs and usage**

#### Key Features
- **Cost and usage reports**: Detailed spending analysis
- **Filtering and grouping**: By service, region, tag, etc.
- **Time ranges**: Daily, monthly, yearly views
- **Forecasting**: Predict future costs
- **Rightsizing recommendations**: Optimize instance sizes

#### Common Cost Explorer Views
```
Monthly costs by service
Daily costs for specific services
Costs grouped by linked account
Reserved Instance utilization
Savings Plans utilization
```

#### Cost Allocation Tags
**Track costs by project, department, environment**

```
Tags:
- Project: WebApp
- Environment: Production
- Department: Engineering
- Owner: john.doe@company.com
```

### AWS Budgets
**Set custom cost and usage budgets with alerts**

#### Budget Types
| Type | Purpose | Example |
|------|---------|---------|
| **Cost** | Monitor spending | Alert when monthly cost > $1000 |
| **Usage** | Monitor service usage | Alert when EC2 hours > 1000 |
| **RI Utilization** | Monitor RI usage | Alert when utilization < 80% |
| **RI Coverage** | Monitor RI coverage | Alert when coverage < 90% |

#### Budget Configuration
- **Period**: Monthly, quarterly, annually
- **Budget amount**: Fixed or variable by month
- **Filters**: Service, account, tag, region
- **Alerts**: Email, SNS when thresholds exceeded

#### Budget Best Practices
- **Start with high-level budgets**: Overall account spending
- **Drill down**: Department, project, environment budgets
- **Set multiple thresholds**: 50%, 80%, 100% of budget
- **Action budgets**: Automatically stop instances when exceeded

### AWS Cost and Usage Report (CUR)
**Most comprehensive cost and usage data**

#### CUR Features
- **Detailed data**: Line-item usage and costs
- **Hourly granularity**: Detailed time breakdown
- **Resource-level details**: Individual resource costs
- **Custom analysis**: Use with BI tools
- **S3 delivery**: Automatically delivered to S3

#### CUR Use Cases
- **Detailed cost analysis**: Understand exactly where money is spent
- **Chargeback/showback**: Allocate costs to business units
- **Custom reporting**: Build internal dashboards
- **Cost optimization**: Identify optimization opportunities

### AWS Trusted Advisor
**Automated recommendations for cost optimization**

#### Cost Optimization Checks
- **Low utilization EC2 instances**: Underutilized resources
- **Idle load balancers**: Unused load balancers
- **Underutilized EBS volumes**: Oversized storage
- **RDS idle DB instances**: Unused databases
- **Route 53 latency resource record sets**: Optimization opportunities

#### Support Plan Requirements
| Check Type | Basic | Developer | Business | Enterprise |
|------------|-------|-----------|----------|------------|
| **Core checks** | ✓ | ✓ | ✓ | ✓ |
| **Full checks** | - | - | ✓ | ✓ |
| **API access** | - | - | ✓ | ✓ |
| **Weekly refresh** | - | - | ✓ | ✓ |

## Cost Optimization Strategies

### Right-Sizing
**Match resource capacity to actual usage**

#### EC2 Right-Sizing
- **Monitor utilization**: CloudWatch metrics over time
- **Identify opportunities**: Consistently low CPU/memory usage
- **Test smaller instances**: Verify performance impact
- **Automate**: Use AWS Compute Optimizer recommendations

#### Right-Sizing Process
1. **Baseline current usage**: Collect 2-4 weeks of metrics
2. **Identify candidates**: Resources with low utilization
3. **Test changes**: Non-production environments first
4. **Implement gradually**: Minimize impact on operations
5. **Monitor results**: Verify performance and cost savings

### Storage Optimization

#### S3 Storage Classes
**Choose appropriate storage class for access patterns**

| Class | Use Case | Cost | Retrieval |
|-------|----------|------|-----------|
| **Standard** | Frequently accessed | Highest | Immediate |
| **IA** | Infrequently accessed | Lower | Immediate |
| **One Zone-IA** | Non-critical, infrequent | Lowest active tier | Immediate |
| **Glacier** | Archive | Very low | Hours |
| **Deep Archive** | Long-term archive | Lowest | 12+ hours |

#### S3 Lifecycle Policies
```json
{
  "Rules": [
    {
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90, 
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ]
    }
  ]
}
```

#### EBS Optimization
- **Delete unused volumes**: Unattached volumes still incur costs
- **Snapshot management**: Delete old/unnecessary snapshots
- **Volume types**: Use gp3 instead of gp2 for cost savings
- **Snapshot lifecycle**: Automate snapshot deletion

### Network Optimization
- **CloudFront**: Reduce data transfer costs with CDN
- **Regional optimization**: Process data close to users
- **VPC endpoints**: Avoid NAT Gateway costs for AWS services
- **Direct Connect**: For large, consistent data transfers

### Compute Optimization

#### Auto Scaling
**Automatically adjust capacity based on demand**

```json
{
  "TargetTrackingScalingPolicies": [
    {
      "TargetValue": 70.0,
      "PredefinedMetricSpecification": {
        "PredefinedMetricType": "ASGAverageCPUUtilization"
      }
    }
  ]
}
```

#### Lambda Optimization
- **Right-size memory**: Affects CPU allocation and cost
- **Optimize execution time**: Reduce duration charges
- **Provisioned concurrency**: For consistent performance
- **Cold start optimization**: Minimize initialization time

## Governance and Compliance

### AWS Organizations
**Centrally manage multiple AWS accounts**

#### Organizational Units (OUs)
```
Root Organization
├── Production OU
│   ├── Web App Production
│   └── Database Production
├── Development OU
│   ├── Dev Environment
│   └── Test Environment
└── Shared Services OU
    ├── Logging Account
    └── Security Account
```

#### Service Control Policies (SCPs)
**Set permission guardrails for accounts**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ec2:TerminateInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    }
  ]
}
```

### AWS Control Tower
**Set up and govern secure, multi-account AWS environment**

#### Key Features
- **Landing Zone**: Pre-configured multi-account environment
- **Guardrails**: Preventive and detective controls
- **Account Factory**: Automated account provisioning
- **Dashboard**: Centralized governance view

#### Guardrails Types
| Type | Purpose | Examples |
|------|---------|----------|
| **Mandatory** | Cannot be disabled | CloudTrail enabled |
| **Strongly Recommended** | Should be enabled | MFA for root user |
| **Elective** | Optional controls | S3 bucket encryption |

### Cost Allocation and Chargeback

#### Cost Allocation Tags
**Track costs by business dimensions**

```
Cost Allocation Tags:
- Project: mobile-app
- Environment: production
- Department: engineering
- Owner: team-mobile
- CostCenter: 12345
```

#### Chargeback Models
| Model | Description | Use Case |
|-------|-------------|----------|
| **Direct charge** | Exact AWS costs | Dedicated resources |
| **Proportional** | Split based on usage | Shared resources |
| **Fixed allocation** | Predetermined percentages | Overhead costs |
| **Activity-based** | Based on business metrics | Transaction-based |

### AWS Config for Governance
**Track resource compliance and changes**

#### Compliance Rules
- **Required tags**: Ensure all resources are tagged
- **Approved instance types**: Limit to cost-effective sizes
- **Regional restrictions**: Control where resources deployed
- **Security configurations**: Enforce security baselines

## Cost Monitoring and Alerting

### CloudWatch Billing Alarms
**Basic cost monitoring**

```json
{
  "AlarmName": "Monthly-Spend-Alarm",
  "MetricName": "EstimatedCharges",
  "Namespace": "AWS/Billing",
  "Statistic": "Maximum",
  "Period": 86400,
  "EvaluationPeriods": 1,
  "Threshold": 1000,
  "ComparisonOperator": "GreaterThanThreshold"
}
```

### Budget Alerts
**More sophisticated cost controls**

#### Alert Types
- **Actual costs**: Based on current spending
- **Forecasted costs**: Predicted future spending
- **Usage**: Resource utilization thresholds

#### Automated Actions
- **SNS notifications**: Email/SMS alerts
- **Lambda functions**: Custom automation
- **IAM policy attachments**: Restrict permissions
- **EC2/RDS actions**: Stop instances automatically

### Cost Anomaly Detection
**Machine learning-based cost monitoring**

#### Features
- **Automatic baseline**: Learn normal spending patterns
- **Anomaly alerts**: Unusual spending notifications
- **Root cause analysis**: Identify what caused spikes
- **Threshold customization**: Adjust sensitivity

## Best Practices Summary

### Cost Optimization
1. **Right-size resources**: Match capacity to actual needs
2. **Use Reserved Instances/Savings Plans**: For predictable workloads
3. **Leverage Spot Instances**: For fault-tolerant applications
4. **Optimize storage**: Use appropriate storage classes and lifecycle policies
5. **Monitor and analyze**: Regular cost reviews and optimization

### Governance
1. **Multi-account strategy**: Separate environments and business units
2. **Tagging strategy**: Consistent, comprehensive resource tagging
3. **Access controls**: Least privilege access principles
4. **Cost allocation**: Clear chargeback/showback processes
5. **Compliance monitoring**: Automated governance checks

### Monitoring
1. **Set up budgets**: Proactive cost management
2. **Enable Cost Explorer**: Regular cost analysis
3. **Use Trusted Advisor**: Automated optimization recommendations
4. **Implement alerting**: Early warning systems
5. **Regular reviews**: Monthly cost optimization sessions

## Common Cost Pitfalls

### Forgotten Resources
- **Stopped instances**: EBS volumes continue charging
- **Unused load balancers**: Hourly charges with no traffic
- **Idle RDS instances**: Database charges without connections
- **Old snapshots**: Accumulating storage costs
- **Unattached EBS volumes**: Storage without compute

### Over-Provisioning
- **Large instance types**: More capacity than needed
- **High IOPS storage**: Performance beyond requirements
- **Reserved Instance mismatches**: Wrong instance types/regions
- **Excessive data transfer**: Inefficient data movement

### Poor Architecture Decisions
- **Cross-region traffic**: Expensive data transfer
- **Inefficient caching**: Repeated expensive operations
- **Monolithic applications**: Difficult to optimize
- **Always-on development**: Dev/test resources running 24/7

### Monitoring Gaps
- **No budget alerts**: Surprise bills
- **Lack of cost allocation**: Can't identify optimization opportunities
- **Infrequent reviews**: Missing cost optimization opportunities
- **No owner accountability**: No one responsible for costs