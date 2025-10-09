# Amazon EC2 (Elastic Compute Cloud)

## Service Overview and Purpose

Amazon Elastic Compute Cloud (EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers by providing virtual servers (instances) on-demand.

**Core Purpose:**
- Provide scalable computing capacity without upfront investment
- Enable rapid deployment and scaling of applications
- Offer complete control over computing resources
- Support diverse workloads and operating systems

## Key Features and Capabilities

### Instance Types
- **General Purpose**: T4g, T3, T3a, T2, M6i, M6a, M5, M4
- **Compute Optimized**: C6i, C6a, C5, C5n, C4
- **Memory Optimized**: R6i, R6a, R5, R5a, R4, X1e, X1, z1d
- **Storage Optimized**: I4i, I3, I3en, D2, D3, D3en, H1
- **Accelerated Computing**: P4, P3, P2, G5, G4dn, G4ad, G3, F1

### Key Capabilities
- **Auto Scaling**: Automatically adjust capacity based on demand
- **Load Balancing**: Distribute traffic across multiple instances
- **Elastic Block Store (EBS)**: Persistent block storage
- **Security Groups**: Virtual firewalls for instances
- **Key Pairs**: Secure login information for instances
- **Elastic IP**: Static IP addresses for dynamic cloud computing
- **Placement Groups**: Control instance placement strategy
- **Dedicated Hosts**: Physical servers dedicated for your use
- **Spot Instances**: Bid for unused EC2 capacity at reduced costs

## Use Cases and Scenarios

### Primary Use Cases
1. **Web Applications**: Host scalable web servers and application servers
2. **Development and Testing**: Create isolated environments for development
3. **Data Processing**: Run batch processing jobs and data analytics
4. **High Performance Computing**: Scientific computing and simulations
5. **Enterprise Applications**: Host mission-critical business applications
6. **Content Delivery**: Serve static and dynamic content
7. **Backup and Disaster Recovery**: Create backup instances in different regions
8. **Gaming**: Host multiplayer game servers
9. **Machine Learning**: Train and deploy ML models
10. **Microservices**: Deploy containerized applications

### Scenario Examples
- **E-commerce Platform**: Auto-scaling web servers during traffic spikes
- **Big Data Analytics**: Processing large datasets with compute-optimized instances
- **Video Rendering**: Using GPU instances for video processing
- **Development Environment**: Quickly spin up test environments

## Pricing Models and Cost Optimization

### Pricing Models

1. **On-Demand Instances**
   - Pay for compute capacity by hour or second
   - No long-term commitments
   - Best for: Short-term, irregular workloads

2. **Reserved Instances (RIs)**
   - 1 or 3-year terms with significant discounts (up to 75%)
   - Standard RIs: Fixed instance attributes
   - Convertible RIs: Can change instance attributes
   - Scheduled RIs: Capacity reservations for predictable schedules

3. **Spot Instances**
   - Bid for unused EC2 capacity (up to 90% savings)
   - Can be interrupted with 2-minute notice
   - Best for: Fault-tolerant, flexible workloads

4. **Dedicated Hosts**
   - Physical servers dedicated to your account
   - Help meet compliance requirements
   - Can use existing server-bound software licenses

5. **Savings Plans**
   - Flexible pricing model offering lower prices
   - 1 or 3-year commitment to consistent usage
   - Up to 72% savings compared to On-Demand

### Cost Optimization Strategies

1. **Right-sizing**: Match instance types to workload requirements
2. **Auto Scaling**: Scale resources based on demand
3. **Spot Instances**: Use for fault-tolerant workloads
4. **Reserved Instances**: For predictable workloads
5. **Instance Scheduler**: Stop instances during non-business hours
6. **Monitoring**: Use CloudWatch to identify underutilized resources
7. **Storage Optimization**: Choose appropriate EBS volume types
8. **Network Optimization**: Minimize data transfer costs

## Configuration Details and Best Practices

### Launch Configuration Best Practices

1. **AMI Selection**
   - Use AWS-provided AMIs for security patches
   - Create custom AMIs for standardized deployments
   - Regularly update AMIs with latest patches

2. **Instance Type Selection**
   - Start with general-purpose instances (t3.medium)
   - Monitor performance and adjust as needed
   - Consider burstable performance instances for variable workloads

3. **Storage Configuration**
   - Use GP3 for general-purpose workloads (cost-effective)
   - Use IO2 for high IOPS requirements
   - Enable encryption at rest
   - Configure appropriate backup strategies

4. **Network Configuration**
   - Use multiple Availability Zones for high availability
   - Configure security groups with least privilege
   - Use placement groups for low latency requirements
   - Consider enhanced networking for better performance

5. **User Data Scripts**
   - Automate initial instance configuration
   - Install required software and updates
   - Configure monitoring agents
   - Set up log forwarding

### Security Best Practices

1. **Access Control**
   - Use IAM roles instead of access keys
   - Implement least privilege access
   - Regularly rotate SSH keys
   - Use AWS Systems Manager Session Manager

2. **Network Security**
   - Configure security groups with specific rules
   - Use NACLs for additional network layer security
   - Enable VPC Flow Logs for monitoring
   - Consider AWS WAF for web applications

3. **Data Protection**
   - Encrypt EBS volumes and snapshots
   - Use encrypted AMIs
   - Implement backup strategies
   - Enable detailed monitoring

## Integration with Other AWS Services

### Core Integrations

1. **Amazon VPC**: Network isolation and security
2. **Elastic Load Balancer**: Distribute traffic across instances
3. **Auto Scaling**: Automatically adjust capacity
4. **CloudWatch**: Monitoring and alerting
5. **IAM**: Access management and security
6. **Route 53**: DNS and health checks
7. **S3**: Object storage and backup
8. **RDS**: Managed database services
9. **CloudFormation**: Infrastructure as code
10. **Systems Manager**: Instance management and patching

### Advanced Integrations

1. **AWS Backup**: Centralized backup across AWS services
2. **AWS Config**: Configuration compliance monitoring
3. **GuardDuty**: Threat detection and security monitoring
4. **Inspector**: Security assessment service
5. **X-Ray**: Application performance monitoring
6. **Lambda**: Serverless computing integration
7. **ECS/EKS**: Container orchestration
8. **CodeDeploy**: Automated application deployment

## Security Considerations

### Network Security
- **Security Groups**: Stateful firewall rules at instance level
- **NACLs**: Stateless firewall rules at subnet level
- **VPC**: Private network isolation
- **VPN/Direct Connect**: Secure connectivity to on-premises

### Data Security
- **Encryption at Rest**: EBS volume encryption using KMS
- **Encryption in Transit**: SSL/TLS for data transmission
- **Key Management**: AWS KMS integration
- **Backup Encryption**: Encrypted snapshots and AMIs

### Access Security
- **IAM Roles**: Secure service-to-service communication
- **MFA**: Multi-factor authentication for console access
- **SSH Key Management**: Secure key distribution and rotation
- **Session Manager**: Secure shell access without SSH keys

### Compliance
- **SOC**: SOC 1, SOC 2, SOC 3 compliance
- **PCI DSS**: Payment card industry compliance
- **HIPAA**: Healthcare data protection
- **FedRAMP**: Government cloud computing security

## Monitoring and Troubleshooting

### CloudWatch Metrics

#### Default Metrics (5-minute intervals)
- **CPUUtilization**: Percentage of allocated compute units
- **DiskReadOps/DiskWriteOps**: Completed read/write operations
- **DiskReadBytes/DiskWriteBytes**: Bytes read/written
- **NetworkIn/NetworkOut**: Network traffic in bytes
- **NetworkPacketsIn/NetworkPacketsOut**: Network packets
- **StatusCheckFailed**: Instance and system status checks

#### Detailed Monitoring (1-minute intervals)
- Enable for production workloads
- Additional cost but better granularity
- Required for effective auto scaling

### Common Troubleshooting Scenarios

1. **Instance Launch Issues**
   - Insufficient capacity: Try different AZ or instance type
   - Security group issues: Check inbound/outbound rules
   - Subnet issues: Ensure subnet has available IP addresses
   - Key pair issues: Verify key pair exists in region

2. **Connectivity Issues**
   - Security group configuration
   - NACL configuration
   - Route table configuration
   - Instance status checks

3. **Performance Issues**
   - Instance type undersized
   - EBS volume performance limits
   - Network bandwidth limits
   - Application-level bottlenecks

4. **Storage Issues**
   - EBS volume full
   - IOPS limits reached
   - Snapshot creation failures
   - Backup restore issues

### Monitoring Tools

1. **CloudWatch**: Native AWS monitoring
2. **CloudTrail**: API call logging
3. **VPC Flow Logs**: Network traffic monitoring
4. **AWS Config**: Configuration change tracking
5. **Systems Manager**: Patch and inventory management
6. **Third-party**: Datadog, New Relic, Splunk

## Exam-Specific Tips and Common Scenarios

### Solutions Architect Associate (SAA-C03)
- **High Availability**: Multi-AZ deployments
- **Scalability**: Auto Scaling and Load Balancers
- **Cost Optimization**: Instance types and pricing models
- **Security**: Security groups and IAM roles

### Solutions Architect Professional (SAP-C02)
- **Advanced Networking**: Placement groups, enhanced networking
- **Hybrid Architectures**: Direct Connect integration
- **Large-scale Deployments**: Multi-region strategies
- **Performance Optimization**: Instance store vs. EBS

### SysOps Administrator (SOA-C02)
- **Monitoring and Alerting**: CloudWatch setup
- **Backup and Recovery**: Snapshot strategies
- **Patch Management**: Systems Manager
- **Performance Tuning**: Instance optimization

### Developer Associate (DVA-C02)
- **Application Deployment**: User data scripts
- **Development Environments**: Quick instance provisioning
- **Integration**: API Gateway, Lambda
- **Debugging**: Instance metadata and logs

### Common Exam Scenarios

1. **Scenario**: Need to host a web application with variable traffic
   **Solution**: Use Auto Scaling with ALB and t3 instances

2. **Scenario**: Require compliance with data sovereignty
   **Solution**: Use specific AWS regions and Dedicated Hosts

3. **Scenario**: Need to reduce costs for development environments
   **Solution**: Use Spot Instances and scheduled shutdowns

4. **Scenario**: High-performance computing workload
   **Solution**: Use compute-optimized instances with placement groups

## Hands-on Examples and CLI Commands

### Basic Instance Management

```bash
# List all instances
aws ec2 describe-instances

# Launch an instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1d0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678

# Stop an instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Start an instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Terminate an instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Describe instance status
aws ec2 describe-instance-status --instance-ids i-1234567890abcdef0
```

### Security Groups Management

```bash
# Create security group
aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group" \
  --vpc-id vpc-12345678

# Add inbound rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Remove inbound rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Describe security groups
aws ec2 describe-security-groups --group-ids sg-12345678
```

### EBS Volume Management

```bash
# Create EBS volume
aws ec2 create-volume \
  --size 20 \
  --volume-type gp3 \
  --availability-zone us-west-2a

# Attach volume to instance
aws ec2 attach-volume \
  --volume-id vol-12345678 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf

# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-12345678 \
  --description "My snapshot"

# List snapshots
aws ec2 describe-snapshots --owner-ids self
```

### Instance Metadata and User Data

```bash
# Get instance metadata (from within instance)
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/instance-id
curl http://169.254.169.254/latest/meta-data/public-ipv4
curl http://169.254.169.254/latest/meta-data/security-groups

# Get user data (from within instance)
curl http://169.254.169.254/latest/user-data

# Get IAM role credentials (from within instance)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name
```

### Auto Scaling Configuration

```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name my-template \
  --launch-template-data '{
    "ImageId": "ami-0c55b159cbfafe1d0",
    "InstanceType": "t3.micro",
    "KeyName": "my-key-pair",
    "SecurityGroupIds": ["sg-12345678"]
  }'

# Create Auto Scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template,Version=1 \
  --min-size 1 \
  --max-size 3 \
  --desired-capacity 2 \
  --vpc-zone-identifier subnet-12345678,subnet-87654321

# Update Auto Scaling group capacity
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --desired-capacity 3
```

### Advanced Monitoring

```bash
# Enable detailed monitoring
aws ec2 monitor-instances --instance-ids i-1234567890abcdef0

# Create CloudWatch alarm
aws cloudwatch put-metric-alarm \
  --alarm-name cpu-high \
  --alarm-description "Alarm when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --evaluation-periods 2

# Get CloudWatch metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --statistics Average \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 3600
```

### Spot Instances

```bash
# Request spot instances
aws ec2 request-spot-instances \
  --spot-price "0.05" \
  --instance-count 2 \
  --type "one-time" \
  --launch-specification '{
    "ImageId": "ami-0c55b159cbfafe1d0",
    "InstanceType": "t3.micro",
    "KeyName": "my-key-pair",
    "SecurityGroups": [{"GroupId": "sg-12345678"}],
    "SubnetId": "subnet-12345678"
  }'

# Describe spot instance requests
aws ec2 describe-spot-instance-requests

# Cancel spot instance requests
aws ec2 cancel-spot-instance-requests --spot-instance-request-ids sir-12345678
```

This comprehensive documentation covers all aspects of Amazon EC2 for AWS certification preparation, providing both theoretical knowledge and practical examples for hands-on experience.