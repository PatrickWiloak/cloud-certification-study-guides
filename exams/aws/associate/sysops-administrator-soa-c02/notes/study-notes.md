# SysOps Administrator Associate - Comprehensive Study Notes

## Quick Reference Guide

### CloudWatch Essentials

#### Metrics and Dimensions
```bash
# Create custom metric
aws cloudwatch put-metric-data --namespace "MyApp" \
  --metric-name PageLoadTime --value 0.312 \
  --dimensions Instance=i-1234567890abcdef0

# Create alarm
aws cloudwatch put-metric-alarm --alarm-name high-cpu \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

# Get metric statistics
aws cloudwatch get-metric-statistics --namespace AWS/EC2 \
  --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-xxx \
  --start-time 2024-01-01T00:00:00Z --end-time 2024-01-01T23:59:59Z \
  --period 3600 --statistics Average
```

#### CloudWatch Logs Insights Queries
```sql
-- Find errors in logs
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

-- Count errors by type
fields @message
| filter @message like /ERROR/
| stats count() by @message
| sort count desc

-- P99 latency
fields @timestamp, responseTime
| filter responseTime > 0
| stats pct(responseTime, 99) as p99latency

-- Average response time per endpoint
fields endpoint, responseTime
| stats avg(responseTime) as avg_time by endpoint
| sort avg_time desc
```

### Systems Manager Quick Commands

#### Session Manager
```bash
# Start session (no SSH keys needed!)
aws ssm start-session --target i-1234567890abcdef0

# Port forwarding
aws ssm start-session --target i-xxx \
  --document-name AWS-StartPortForwardingSession \
  --parameters "portNumber=3306,localPortNumber=9999"
```

#### Run Command
```bash
# Run shell script on multiple instances
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets "Key=tag:Environment,Values=Production" \
  --parameters 'commands=["df -h", "free -m"]'

# Install CloudWatch agent
aws ssm send-command \
  --document-name "AWS-ConfigureAWSPackage" \
  --targets "Key=instanceids,Values=i-xxx" \
  --parameters '{"action":["Install"],"name":["AmazonCloudWatchAgent"]}'
```

#### Parameter Store
```bash
# Store parameter
aws ssm put-parameter --name "/myapp/db/password" \
  --value "secret123" --type SecureString

# Retrieve parameter
aws ssm get-parameter --name "/myapp/db/password" --with-decryption

# Store with tags
aws ssm put-parameter --name "/prod/api/key" \
  --value "xyz" --type SecureString \
  --tags "Key=Environment,Value=Production"
```

### Auto Scaling Scenarios

#### Scaling Policies
```bash
# Target tracking policy (maintain CPU at 50%)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name target-tracking-policy \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }'

# Step scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name step-scale-out \
  --policy-type StepScaling \
  --adjustment-type PercentChangeInCapacity \
  --metric-aggregation-type Average \
  --step-adjustments '[
    {"MetricIntervalLowerBound": 0, "MetricIntervalUpperBound": 10, "ScalingAdjustment": 10},
    {"MetricIntervalLowerBound": 10, "ScalingAdjustment": 30}
  ]'

# Scheduled scaling
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 8 * * *" \
  --desired-capacity 10
```

### Load Balancer Health Checks

#### ALB Health Check Configuration
- **Healthy threshold**: 2-10 consecutive checks
- **Unhealthy threshold**: 2-10 consecutive failures
- **Interval**: 5-300 seconds (default 30)
- **Timeout**: 2-120 seconds (must be less than interval)
- **Success codes**: 200, or 200-299, or 200,202

#### Common Issues
```
Problem: Instances marked unhealthy
Solutions:
1. Check security group allows health check traffic from LB
2. Verify health check path returns 200 OK
3. Ensure timeout < interval
4. Check application is actually listening on health check port
5. Review CloudWatch metrics: HealthyHostCount, UnHealthyHostCount
```

### CloudFormation Best Practices

#### Template Structure
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Production Web Application'

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small, t3.medium]
    Description: EC2 instance type

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d1cd67c26f5fca19

Conditions:
  IsProduction: !Equals [!Ref Environment, production]

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Name
          Value: !Sub '${AWS::StackName}-web'
    CreationPolicy:
      ResourceSignal:
        Timeout: PT15M
    UpdatePolicy:
      AutoScalingReplacingUpdate:
        WillReplace: true

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref WebServer
    Export:
      Name: !Sub '${AWS::StackName}-InstanceId'
```

#### Stack Operations
```bash
# Create stack
aws cloudformation create-stack --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t3.small \
  --capabilities CAPABILITY_IAM

# Update stack with change set
aws cloudformation create-change-set --stack-name my-stack \
  --template-body file://template-v2.yaml \
  --change-set-name my-changes

aws cloudformation execute-change-set --change-set-name my-changes \
  --stack-name my-stack

# Detect drift
aws cloudformation detect-stack-drift --stack-name my-stack
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id xxx

# Delete stack (but retain some resources)
aws cloudformation delete-stack --stack-name my-stack \
  --retain-resources WebServer Database
```

## Common Troubleshooting Scenarios

### Scenario 1: EC2 Instance Not Accessible

**Symptoms**: Cannot SSH/RDP to instance

**Troubleshooting Steps**:
1. Check instance state (running?)
2. Verify security group inbound rules (port 22 or 3389)
3. Check NACLs (both inbound and outbound - stateless!)
4. Verify route table has route to IGW for public subnet
5. Confirm instance has public IP/EIP if accessing from internet
6. Check key pair permissions (chmod 400)
7. Review VPC Flow Logs for rejected connections
8. Try Systems Manager Session Manager as alternative

```bash
# Check instance status
aws ec2 describe-instance-status --instance-ids i-xxx

# Check security groups
aws ec2 describe-security-groups --group-ids sg-xxx

# Enable VPC Flow Logs
aws ec2 create-flow-logs --resource-type VPC \
  --resource-ids vpc-xxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs
```

### Scenario 2: Auto Scaling Not Scaling

**Symptoms**: ASG not adding/removing instances despite metrics

**Troubleshooting Steps**:
1. Check CloudWatch alarm state (OK, ALARM, INSUFFICIENT_DATA)
2. Verify alarm has action attached (scaling policy ARN)
3. Review cooldown period (default 300 seconds)
4. Check if manual suspend processes
5. Verify IAM role for Auto Scaling service
6. Check instance launch failures in Activity History
7. Review service quotas (vCPU limits)

```bash
# Check ASG activity
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name my-asg --max-records 20

# Check suspended processes
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-asg \
  --query 'AutoScalingGroups[0].SuspendedProcesses'

# Resume processes
aws autoscaling resume-processes \
  --auto-scaling-group-name my-asg \
  --scaling-processes Launch Terminate
```

### Scenario 3: RDS High CPU/Storage

**High CPU**:
1. Enable Enhanced Monitoring (1-second granularity)
2. Check Performance Insights for slow queries
3. Review CloudWatch metrics: DatabaseConnections, ReadLatency, WriteLatency
4. Analyze slow query logs
5. Consider Read Replicas for read-heavy workloads
6. Upgrade instance class (vertical scaling)
7. Review application query patterns

**Storage Issues**:
```bash
# Enable storage autoscaling
aws rds modify-db-instance --db-instance-identifier mydb \
  --max-allocated-storage 1000 \
  --apply-immediately

# Check storage metrics
aws cloudwatch get-metric-statistics --namespace AWS/RDS \
  --metric-name FreeStorageSpace \
  --dimensions Name=DBInstanceIdentifier,Value=mydb \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 --statistics Average
```

### Scenario 4: S3 Access Denied

**Common Causes**:
1. Bucket policy denies access
2. IAM policy doesn't allow action
3. S3 Block Public Access enabled
4. ACLs preventing access
5. VPC endpoint policy restrictions
6. KMS key policy for encrypted objects

**Troubleshooting**:
```bash
# Check bucket policy
aws s3api get-bucket-policy --bucket my-bucket

# Check public access block
aws s3api get-public-access-block --bucket my-bucket

# Test IAM policy simulation
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/*

# Check object ACL
aws s3api get-object-acl --bucket my-bucket --key file.txt
```

### Scenario 5: Lambda Function Timeout

**Common Issues**:
1. Insufficient timeout (default 3s, max 15 minutes)
2. VPC Lambda cold starts (use VPC endpoints or NAT)
3. External API slow response
4. Database connection pool exhaustion
5. Insufficient memory (more memory = more CPU)

**Optimization**:
```bash
# Update timeout and memory
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 300 \
  --memory-size 1024

# Enable X-Ray tracing
aws lambda update-function-configuration \
  --function-name my-function \
  --tracing-config Mode=Active

# Review CloudWatch Logs Insights
fields @timestamp, @duration, @billedDuration, @memorySize
| filter @type = "REPORT"
| stats avg(@duration), max(@duration), pct(@duration, 99)
```

## Exam Tips and Common Traps

### CloudWatch Gotchas

**Metric Math**:
- Can combine multiple metrics with operators
- Example: `m1 + m2` or `AVG([m1, m2, m3])`
- Useful for custom calculations

**Alarm States**:
- OK, ALARM, INSUFFICIENT_DATA
- Need evaluation periods to change state
- Missing data can be treated as: notBreaching, breaching, ignore, missing

**Common Trap**:
> Q: How to monitor memory and disk usage on EC2?
>
> **WRONG**: These are available by default in CloudWatch
>
> **CORRECT**: Must install CloudWatch Agent - EC2 metrics only include CPU, Network, Disk I/O (not memory or disk usage %)

### Auto Scaling Best Practices

**Scaling Policies**:
- **Target Tracking**: Easiest, maintains metric at target (e.g., CPU 50%)
- **Step Scaling**: More control, different adjustments based on alarm breach size
- **Simple Scaling**: Legacy, one adjustment per alarm
- **Scheduled**: Time-based (known traffic patterns)

**Health Checks**:
- EC2 status checks (default)
- ELB health checks (recommended with load balancer)
- Custom health checks via API

**Common Trap**:
> Q: Instances keep terminating during scale-in
>
> **Solution**: Set instance protection or increase cooldown period

### Security Groups vs NACLs

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| Level | Instance/ENI | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Order | All evaluated | Processed in order |
| Return Traffic | Automatic | Must explicitly allow |

**Common Trap**:
> Q: Added inbound rule to NACL but still can't connect
>
> **MISSING**: Outbound rule for return traffic (ephemeral ports 1024-65535)

### Systems Manager Session Manager Benefits

✅ **Advantages over SSH**:
- No bastion hosts needed
- No SSH keys to manage
- No inbound ports (outbound 443 only)
- Full audit trail in CloudTrail
- Session logging to S3/CloudWatch
- Works with instances in private subnets

**Requirements**:
1. SSM Agent installed (pre-installed on Amazon Linux 2, Ubuntu)
2. IAM instance profile with `AmazonSSMManagedInstanceCore`
3. Outbound HTTPS (443) to Systems Manager endpoints
4. User IAM permission: `ssm:StartSession`

### AWS Backup vs Native Backups

**AWS Backup Advantages**:
- Centralized management
- Cross-region copy
- Cross-account copy
- Backup compliance monitoring
- Tag-based backup policies
- Single console for all services

**Supported Services**:
- EC2, EBS, EFS, FSx
- RDS, Aurora, DynamoDB, Neptune, DocumentDB
- Storage Gateway, S3

**Common Trap**:
> Q: Need to backup on-premises VMware VMs
>
> **CORRECT**: AWS Backup supports AWS Storage Gateway, not direct VMware backup

### Cost Optimization Quick Wins

1. **Right-size EC2**: Use AWS Compute Optimizer
2. **Stop development instances**: 70% savings vs running 24/7
3. **S3 Intelligent-Tiering**: Automatic cost optimization
4. **Delete old EBS snapshots**: Incremental but still cost money
5. **Remove unused EIPs**: Charged when not attached
6. **Use RDS Reserved Instances**: Up to 69% savings
7. **Delete unattached EBS volumes**: Common after instance termination

```bash
# Find unattached volumes
aws ec2 describe-volumes \
  --filters Name=status,Values=available \
  --query 'Volumes[*].[VolumeId,Size,CreateTime]' \
  --output table

# Find unused EIPs
aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId]' \
  --output table
```

### Route 53 Health Checks

**Types**:
- **Endpoint**: Check IP/domain endpoint
- **CloudWatch Alarm**: Based on CloudWatch alarm state
- **Calculated**: Combine multiple health checks (OR, AND)

**String Matching**:
- Can check for specific string in first 5120 bytes
- Must match within 2 seconds
- Useful to verify not just server up, but application healthy

**Common Trap**:
> Q: Health check shows healthy but Route 53 not failing over
>
> **CHECK**: Failover routing policy enabled? Primary/Secondary resource records configured?

### Multi-Region Patterns

**Active-Passive**:
- Primary region handles all traffic
- Route 53 health check monitors primary
- Automatic failover to secondary region
- Use: Disaster recovery

**Active-Active**:
- Both regions handle traffic simultaneously
- Route 53 latency/geolocation routing
- Data replication between regions (DynamoDB Global Tables, Aurora Global Database, S3 Cross-Region Replication)
- Use: Global applications, low latency

### CloudFormation Drift Detection

**What It Detects**:
- Manual changes to resources
- Changes made outside CloudFormation

**What It Doesn't Detect**:
- Changes to resources not managed by CloudFormation
- Changes that don't affect resource properties

**Drift Status**:
- **IN_SYNC**: No drift
- **MODIFIED**: Resource properties changed
- **DELETED**: Resource deleted outside stack
- **NOT_CHECKED**: Resource type doesn't support drift

## Pre-Exam Checklist

### Core Services to Master
- ✅ CloudWatch: Metrics, Alarms, Logs, Insights queries
- ✅ Systems Manager: Session Manager, Run Command, Patch Manager, Parameter Store
- ✅ Auto Scaling: Policies, health checks, lifecycle hooks
- ✅ Load Balancers: ALB vs NLB, health checks, sticky sessions
- ✅ CloudFormation: Templates, stacks, drift detection
- ✅ Elastic Beanstalk: Deployment strategies, environment types
- ✅ AWS Backup: Backup plans, vault lock, cross-region copy
- ✅ Route 53: Routing policies, health checks
- ✅ VPC: Subnets, route tables, security groups, NACLs, endpoints
- ✅ IAM: Policies, roles, service control policies
- ✅ RDS: Multi-AZ, Read Replicas, backups, performance insights

### Command Line Practice
- EC2: launch, stop, terminate, describe instances
- CloudWatch: put-metric-data, put-metric-alarm, get-metric-statistics
- Systems Manager: send-command, start-session, get-parameter
- Auto Scaling: create/update ASG, put-scaling-policy
- S3: sync, cp with --sse, lifecycle configuration
- CloudFormation: create-stack, update-stack, detect-drift

### Scenario-Based Knowledge
- Troubleshoot unreachable instances (SG, NACL, routes, IGW)
- Design highly available architectures (Multi-AZ, ELB, ASG)
- Implement automated operations (EventBridge + Lambda, Systems Manager Automation)
- Optimize costs (right-sizing, Reserved Instances, S3 storage classes)
- Secure workloads (IAM roles, encryption, VPC isolation)
- Monitor and alert (CloudWatch metrics, custom metrics, composite alarms)

### Common Exam Question Patterns
1. **"Most cost-effective solution"** → Reserved Instances, Spot, S3 lifecycle
2. **"Minimal administrative overhead"** → Managed services (RDS vs EC2+DB)
3. **"Highly available"** → Multi-AZ, multiple regions, Auto Scaling
4. **"Least privilege"** → IAM policies with specific actions/resources
5. **"Automated"** → CloudFormation, Systems Manager, EventBridge
6. **"Secure"** → Encryption (KMS), network isolation (VPC), IAM roles
7. **"Monitor and alert"** → CloudWatch alarms, SNS notifications

## Final Tips

### Time Management
- 130 minutes, 65 questions = 2 minutes per question
- Flag difficult questions, come back later
- Don't overthink - choose the BEST answer, not perfect

### Question Keywords
- **"Most cost-effective"**: Consider cost first
- **"Least operational overhead"**: Choose managed services
- **"Immediate"**: No delays acceptable (synchronous replication)
- **"Eventually"**: Some delay acceptable (asynchronous replication)
- **"Compliance requirement"**: Security and audit trail priority

### AWS Well-Architected Framework
Understand the five pillars:
1. **Operational Excellence**: Automation, monitoring, documentation
2. **Security**: Defense in depth, least privilege, encryption
3. **Reliability**: Multi-AZ, backups, auto-recovery
4. **Performance Efficiency**: Right-sizing, caching, monitoring
5. **Cost Optimization**: Pay for what you use, Reserved capacity

### Before Exam Day
- ✅ Take 2-3 full practice exams
- ✅ Review all AWS service FAQs for key services
- ✅ Hands-on practice with AWS Console and CLI
- ✅ Understand when to use which service (not just what they do)
- ✅ Know troubleshooting steps for common issues
- ✅ Sleep well, eat breakfast, arrive early

Good luck! 🚀
