# AWS Cost Optimization - SAA-C03

**[📖 AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)** - Run systems to deliver business value at the lowest price point

## EC2 Cost Optimization

**[📖 EC2 Pricing](https://aws.amazon.com/ec2/pricing/)** - Understand EC2 pricing models

### EC2 Pricing Models

**On-Demand**:
- Pay per hour or second
- No commitment
- Highest cost
- Use Case: Short-term, unpredictable workloads

**Reserved Instances (RI)**:
- 1 or 3-year commitment
- Up to 72% discount vs On-Demand
- **Standard RI**: Highest discount, cannot change instance type
- **Convertible RI**: Change instance family, lower discount
- **Payment**: All upfront, partial upfront, no upfront
- Use Case: Steady-state workloads

**[📖 Reserved Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html)** - Save up to 72% with long-term commitments

**Savings Plans**:
- 1 or 3-year commitment
- Up to 72% discount
- **Compute Savings Plan**: Flexible across instance family, region, OS
- **EC2 Instance Savings Plan**: Flexible within instance family
- Use Case: Predictable usage with flexibility

**[📖 Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html)** - Flexible pricing model for compute usage

**Spot Instances**:
- Up to 90% discount
- Can be terminated with 2-minute warning
- Spot Request: One-time or persistent
- **Spot Fleet**: Mix of Spot and On-Demand
- Use Case: Fault-tolerant, flexible workloads (batch, data analysis, CI/CD)

**Dedicated Hosts**:
- Physical server dedicated to you
- Socket/core visibility for licensing
- Most expensive option
- Use Case: Regulatory, licensing requirements

**Dedicated Instances**:
- Instances on dedicated hardware
- May share hardware with other instances in account
- Cheaper than Dedicated Hosts
- No access to underlying hardware

### Cost Optimization Strategies

**Right-Sizing**:
- Use Compute Optimizer recommendations
- Analyze CloudWatch metrics (CPU, memory, network)
- Downsize over-provisioned instances
- Match instance type to workload

**Instance Lifecycle**:
- Stop instances during off-hours
- Terminate unused instances
- Use Auto Scaling to match demand
- Schedule start/stop with Systems Manager

**Purchasing Options Mix**:
- Reserved/Savings Plans for baseline
- On-Demand for variability
- Spot for batch and fault-tolerant workloads
- Aim for 60-80% Reserved/Savings coverage

## Storage Cost Optimization

**[📖 S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)** - Optimize storage costs with the right storage class

### S3 Storage Classes

**Pricing** (per GB, cheapest to most expensive):
1. **S3 Glacier Deep Archive**: $0.00099/GB (~$1/TB)
2. **S3 Glacier Flexible Retrieval**: $0.0036/GB (~$3.60/TB)
3. **S3 Glacier Instant Retrieval**: $0.004/GB (~$4/TB)
4. **S3 One Zone-IA**: $0.01/GB (~$10/TB)
5. **S3 Standard-IA**: $0.0125/GB (~$12.50/TB)
6. **S3 Intelligent-Tiering**: $0.023/GB (~$23/TB) + monitoring
7. **S3 Standard**: $0.023/GB (~$23/TB)

**Lifecycle Policies**:
- Automatically transition objects between storage classes
- Delete old versions or expired objects
- Reduce costs for infrequently accessed data

**Example Lifecycle**:
```
0-30 days: S3 Standard
30-90 days: S3 Standard-IA
90-365 days: S3 Glacier Instant Retrieval
365+ days: S3 Glacier Deep Archive
```

**S3 Intelligent-Tiering**:
- Automatic cost optimization
- No retrieval fees for frequent/infrequent tiers
- 4 tiers: Frequent, Infrequent (30 days), Archive (90 days), Deep Archive (180 days)
- Small monitoring fee per object
- Use Case: Unknown or changing access patterns

### EBS Cost Optimization

**Volume Types**:
- **gp3**: Cheaper than gp2, provision IOPS/throughput independently
- **st1/sc1**: Cheaper for throughput-optimized workloads
- Delete old snapshots
- Resize volumes to actual usage

**Strategies**:
- Use gp3 instead of gp2 (20% cheaper)
- Delete unattached volumes
- Use snapshot lifecycle policies
- Stop instances instead of keeping running with attached EBS

### EFS Cost Optimization
- **EFS Standard-IA**: 92% cheaper than Standard
- Lifecycle management: Move to IA after 7, 14, 30, 60, or 90 days
- **One Zone storage classes**: 47% savings (if AZ redundancy not required)
- Elastic throughput for variable workloads

## Database Cost Optimization

### RDS Strategies
- **Reserved Instances**: 1 or 3-year commitment
- **Aurora Serverless v2**: Pay per second, auto-scaling
- **Right-size instances**: Use CloudWatch metrics
- **Stop dev/test instances** during off-hours
- **Read Replicas**: Offload read traffic from primary

### DynamoDB Strategies
- **On-Demand**: Unpredictable traffic (pay per request)
- **Provisioned**: Predictable traffic (pre-purchase capacity)
- **Reserved Capacity**: 1-year commitment for provisioned
- **Auto Scaling**: Match capacity to demand
- **TTL**: Automatically delete expired items
- **DAX**: Reduce read costs by caching

### Redshift Strategies
- **Reserved Nodes**: Up to 75% discount
- **Serverless**: Pay for usage, no cluster management
- **Pause/Resume**: Dev/test clusters
- **Spectrum**: Query S3 data directly (avoid loading all data)

## Data Transfer Costs

### Data Transfer Pricing
- **Inbound**: FREE (data coming into AWS)
- **Outbound to Internet**: $0.09/GB (varies by region)
- **Between AZs**: $0.01-0.02/GB
- **Between Regions**: $0.02/GB
- **CloudFront to Internet**: $0.085/GB (slightly cheaper + better performance)
- **S3 Transfer Acceleration**: Additional $0.04-0.08/GB

### Cost Optimization Strategies

**CloudFront**:
- Use CloudFront to reduce data transfer costs
- Cache at edge locations
- Regional edge caches reduce origin fetches

**S3 Transfer Acceleration**:
- Only when speed matters
- Disable if not needed

**VPC Endpoints**:
- FREE for S3 and DynamoDB
- Avoid NAT Gateway charges
- Keep traffic on AWS network

**DirectConnect**:
- Lower data transfer costs for large, consistent transfers
- $0.02/GB vs $0.09/GB for internet transfer

**Inter-Region Optimization**:
- Minimize cross-region data transfer
- Use same-region resources when possible
- CloudFront for global distribution

## Monitoring and Cost Management

**[📖 AWS Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)** - Visualize and analyze costs

### AWS Cost Explorer
- Visualize spending patterns
- Filter by service, account, tag, region
- Forecasting (up to 12 months)
- Reserved Instance recommendations
- Savings Plans recommendations

### AWS Budgets
- Set custom cost and usage budgets
- Alert when exceeding threshold
- **Budget Types**:
  - Cost budgets
  - Usage budgets
  - RI utilization budgets
  - RI coverage budgets
  - Savings Plans budgets

**[📖 AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)** - Set custom budgets and receive alerts

**Budget Actions**:
- Apply IAM policy to restrict services
- Stop EC2 or RDS instances
- Target specific users, roles, groups

### AWS Cost and Usage Report (CUR)
- Most detailed billing information
- Hourly, daily, or monthly granularity
- Delivered to S3
- Integration with Athena, QuickSight, Redshift
- Resource-level detail with tags

### Cost Allocation Tags
- Track costs by project, team, environment
- AWS-generated tags (e.g., aws:createdBy)
- User-defined tags (e.g., Project:WebApp)
- Activate tags in Billing Console
- Apply consistently across resources

### AWS Compute Optimizer
- ML-based recommendations
- Right-size EC2, Auto Scaling, EBS, Lambda, ECS/Fargate
- Performance risk assessment
- Historical utilization analysis
- Integration with Cost Explorer

**[📖 AWS Compute Optimizer](https://docs.aws.amazon.com/compute-optimizer/)** - Get recommendations to optimize AWS resources

## Architectural Cost Optimization

### Serverless Architectures
- Lambda: Pay per request and duration
- DynamoDB: Pay for read/write capacity
- S3: Pay for storage and requests
- No idle capacity costs

### Auto Scaling
- Match capacity to demand
- Scale down during low usage
- Prevent over-provisioning
- Use target tracking policies

### Caching
- **ElastiCache**: Reduce database load
- **CloudFront**: Reduce origin requests
- **DAX**: DynamoDB caching
- Lower compute and data transfer costs

### Managed Services
- RDS vs self-managed database on EC2
- EKS/ECS vs self-managed Kubernetes
- Less operational overhead
- Often cheaper total cost of ownership

## Free Tier and Trials

**[📖 AWS Free Tier](https://aws.amazon.com/free/)** - Explore AWS with free tier offerings

### AWS Free Tier

**Always Free**:
- Lambda: 1M requests/month
- DynamoDB: 25 GB storage, 25 RCU/WCU
- CloudWatch: 10 metrics, 10 alarms
- SNS: 1M publishes
- CloudFront: 1 TB out, 10M requests

**12 Months Free**:
- EC2: 750 hours/month t2.micro or t3.micro
- S3: 5 GB Standard storage
- RDS: 750 hours/month db.t2.micro
- EBS: 30 GB General Purpose (gp2/gp3)

**Trials**:
- Inspector: 90 days
- GuardDuty: 30 days
- Macie: 30 days

## Cost Optimization Best Practices

### 1. Right-Sizing
- Continuously analyze and adjust resource sizes
- Use Compute Optimizer recommendations
- Monitor CloudWatch metrics
- Review quarterly

### 2. Increase Elasticity
- Use Auto Scaling
- Serverless architectures
- Spot instances for fault-tolerant workloads
- Shut down dev/test environments

### 3. Choose Right Pricing Model
- Reserved Instances or Savings Plans for stable workloads
- Spot for batch and interruptible workloads
- On-Demand for spiky, unpredictable workloads

### 4. Optimize Storage
- Lifecycle policies for S3
- Delete old snapshots
- Use EBS gp3 instead of gp2
- Implement data retention policies

### 5. Monitor and Analyze
- Set up billing alerts
- Review Cost Explorer regularly
- Tag resources for cost allocation
- Use AWS Budgets

### 6. Architecture Efficiency
- Minimize data transfer
- Use caching (CloudFront, ElastiCache, DAX)
- Choose appropriate regions
- VPC endpoints for S3/DynamoDB

### 7. Leverage AWS Tools
- Trusted Advisor cost optimization checks
- Cost Explorer recommendations
- Compute Optimizer for rightsizing
- AWS Cost Anomaly Detection

## Exam Tips

### Common Cost Optimization Scenarios
- **Steady-state workload**: Reserved Instances or Savings Plans
- **Batch processing, fault-tolerant**: Spot Instances
- **Spiky, unpredictable**: On-Demand + Auto Scaling
- **Infrequent S3 access**: S3 IA or Intelligent-Tiering
- **Long-term archive**: S3 Glacier Deep Archive
- **Reduce data transfer**: CloudFront, VPC Endpoints
- **Right-size resources**: Compute Optimizer
- **Unused resources**: Delete unattached EBS, old snapshots, idle load balancers

### Key Decision Points
1. Workload predictability → Pricing model
2. Access pattern → Storage class
3. Fault tolerance → Spot viability
4. Data transfer volume → Transfer strategy
5. Idle periods → Auto Scaling or scheduled stop/start

### Cost vs Performance Trade-offs
- **Lower RTO/RPO**: Higher cost (multi-site vs backup)
- **Higher availability**: Higher cost (multi-AZ)
- **Better performance**: Higher cost (Provisioned IOPS, larger instances)
- Balance based on business requirements

### Common Pitfalls
- Unused Elastic IPs ($0.005/hour)
- Unattached EBS volumes
- Old EBS snapshots
- Data transfer between AZs
- Over-provisioned resources
- NAT Gateway charges (use VPC endpoints when possible)
- Load balancers with no targets
