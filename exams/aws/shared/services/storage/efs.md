# Amazon EFS (Elastic File System)

## Service Overview and Purpose

Amazon EFS provides a simple, scalable, and fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. EFS is built to scale on demand to petabytes without disrupting applications, growing and shrinking automatically as you add and remove files.

### Key Characteristics
- **NFS v4.1 Protocol**: POSIX-compliant file system
- **Fully Managed**: No infrastructure management required
- **Elastic**: Automatically scales up and down
- **Multi-AZ Access**: Concurrent access from multiple AZs
- **High Durability**: 99.999999999% (11 9's) durability

## Key Features and Capabilities

### Storage Classes

1. **Standard Storage Class**
   - Default storage class
   - Multi-AZ redundancy
   - Highest level of durability and availability
   - Use case: Frequently accessed files

2. **Infrequent Access (IA) Storage Class**
   - Lower cost for infrequently accessed files
   - Same durability as Standard
   - Retrieval fees apply
   - Use case: Files accessed a few times per quarter

3. **Archive Storage Class**
   - Lowest cost storage class
   - For rarely accessed files
   - Higher retrieval fees than IA
   - Use case: Files accessed a few times per year

4. **One Zone Storage Classes**
   - **One Zone**: Single AZ storage
   - **One Zone-IA**: Single AZ with IA pricing
   - 90% lower cost than Standard
   - Use case: Backup, dev/test environments

### Performance Modes

1. **General Purpose Mode**
   - Default performance mode
   - Lowest latency per operation
   - Up to 7,000 file operations per second
   - Use case: Latency-sensitive workloads

2. **Max I/O Mode**
   - Higher performance and throughput
   - Higher latency per operation
   - Virtually unlimited performance
   - Use case: High levels of aggregate throughput

### Throughput Modes

1. **Bursting Throughput**
   - Default throughput mode
   - Throughput scales with file system size
   - Baseline: 50 MiB/s per TiB stored
   - Burst: Up to 100 MiB/s for file systems < 1TiB

2. **Provisioned Throughput**
   - Fixed throughput independent of storage
   - Pay for provisioned throughput
   - Can be higher than bursting mode
   - Use case: High throughput to storage ratio

3. **Elastic Throughput**
   - Automatically scales throughput
   - Pay only for throughput used
   - Up to 3 GiB/s for reads, 1 GiB/s for writes
   - Use case: Unpredictable throughput patterns

### Core Features

#### Lifecycle Management
- **Automatic tiering** between storage classes
- **Configurable policies** (1, 7, 14, 30, 60, 90 days)
- **Intelligent tiering** based on access patterns
- **Cost optimization** without performance impact

#### Access Control
- **POSIX permissions**
- **IAM integration**
- **VPC security groups**
- **Network ACLs**
- **EFS Access Points**

#### Backup and Restore
- **AWS Backup integration**
- **Point-in-time recovery**
- **Cross-region backup**
- **Automatic backup policies**

## Use Cases and Scenarios

### Primary Use Cases

1. **Content Management and Serving**
   - Web serving and content management
   - Media processing workflows
   - Shared content repositories
   - Content distribution

2. **Application Development**
   - Shared application data
   - Configuration files
   - Development environments
   - CI/CD pipelines

3. **Big Data Analytics**
   - Data lakes
   - Analytics workloads
   - Machine learning datasets
   - Parallel processing

4. **Backup and Archive**
   - Application backups
   - Database backups
   - Log archival
   - Compliance data retention

### Architecture Patterns

1. **Multi-Tier Web Applications**
   - Shared static content
   - User-uploaded files
   - Application logs
   - Configuration files

2. **Container Storage**
   - Kubernetes persistent volumes
   - Docker shared storage
   - Microservices data sharing
   - Stateful applications

3. **High Performance Computing**
   - Parallel workloads
   - Scientific computing
   - Financial modeling
   - Research datasets

## Pricing Models and Cost Optimization

### Pricing Components

1. **Storage Costs**
   - **Standard**: $0.30 per GB-month
   - **Infrequent Access**: $0.0125 per GB-month
   - **Archive**: $0.0045 per GB-month
   - **One Zone**: 47% lower than Standard

2. **Throughput Costs**
   - **Provisioned Throughput**: $6.00 per MiB/s-month
   - **Elastic Throughput**:
     - Read: $3.00 per GiB
     - Write: $6.00 per GiB

3. **Request Costs**
   - **IA and Archive**: Per-request charges
   - **Standard**: No request charges

### Cost Optimization Strategies

1. **Use Lifecycle Management**
   ```bash
   # Create lifecycle policy
   aws efs put-lifecycle-configuration \
     --file-system-id fs-12345678 \
     --lifecycle-policies \
       TransitionToIA=AFTER_30_DAYS,TransitionToArchive=AFTER_90_DAYS
   ```

2. **Choose Appropriate Storage Class**
   - Analyze access patterns
   - Use One Zone for non-critical data
   - Consider retrieval costs for IA/Archive

3. **Optimize Throughput Mode**
   - Monitor throughput utilization
   - Use Elastic for variable workloads
   - Provision only when needed

4. **Regional Considerations**
   - Keep data close to compute
   - Consider data transfer costs
   - Use Regional vs One Zone appropriately

## Configuration Details and Best Practices

### EFS Creation and Configuration

```bash
# Create EFS file system
aws efs create-file-system \
  --creation-token my-efs-$(date +%s) \
  --performance-mode generalPurpose \
  --throughput-mode elastic \
  --encrypted \
  --kms-key-id alias/aws/elasticfilesystem \
  --tags Key=Name,Value=MyEFS

# Create mount targets
aws efs create-mount-target \
  --file-system-id fs-12345678 \
  --subnet-id subnet-12345678 \
  --security-groups sg-12345678

# Create access point
aws efs create-access-point \
  --file-system-id fs-12345678 \
  --posix-user Uid=1000,Gid=1000 \
  --root-directory Path="/app",CreationInfo='{
    "OwnerUid": 1000,
    "OwnerGid": 1000,
    "Permissions": "755"
  }' \
  --tags Key=Name,Value=AppAccessPoint
```

### Mount Configuration

```bash
# Install EFS utilities (Amazon Linux 2)
sudo yum install -y amazon-efs-utils

# Create mount point
sudo mkdir /mnt/efs

# Mount using EFS helper (recommended)
sudo mount -t efs -o tls fs-12345678:/ /mnt/efs

# Mount using NFS
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 \
  fs-12345678.efs.region.amazonaws.com:/ /mnt/efs

# Add to /etc/fstab for persistent mounting
echo 'fs-12345678.efs.region.amazonaws.com:/ /mnt/efs efs defaults,_netdev' | sudo tee -a /etc/fstab
```

### Best Practices

1. **Security**
   - Use VPC endpoints for private access
   - Configure security groups properly
   - Enable encryption in transit and at rest
   - Use EFS Access Points for application isolation

2. **Performance**
   - Choose appropriate performance mode
   - Use regional mount targets
   - Consider client-side caching
   - Monitor performance metrics

3. **Availability**
   - Use multiple mount targets across AZs
   - Implement backup strategies
   - Monitor file system health
   - Plan for disaster recovery

4. **Cost Management**
   - Implement lifecycle policies
   - Monitor storage utilization
   - Use appropriate storage classes
   - Optimize throughput provisioning

### Access Points Configuration

```bash
# Create access point with specific permissions
aws efs create-access-point \
  --file-system-id fs-12345678 \
  --posix-user Uid=1001,Gid=1001 \
  --root-directory Path="/secure",CreationInfo='{
    "OwnerUid": 1001,
    "OwnerGid": 1001,
    "Permissions": "700"
  }' \
  --tags Key=Name,Value=SecureAccessPoint

# Mount using access point
sudo mount -t efs -o tls,accesspoint=fsap-12345678 fs-12345678:/ /mnt/secure
```

## Integration with Other AWS Services

### Compute Integration

1. **EC2 Integration**
   - Direct mounting from EC2 instances
   - Auto Scaling group integration
   - Cross-AZ access
   - Performance optimization

2. **Container Integration**
   - **ECS**: Persistent storage for containers
   - **EKS**: Kubernetes persistent volumes
   - **Fargate**: Serverless container storage
   - **Docker**: Volume mounting

3. **Serverless Integration**
   - **Lambda**: File system access for functions
   - **Batch**: Shared storage for jobs
   - **App Runner**: Application file storage

### Data Services Integration

```yaml
# EKS Persistent Volume example
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-12345678
```

### Backup Integration

```bash
# Create backup vault
aws backup create-backup-vault \
  --backup-vault-name EFSBackupVault \
  --encryption-key-arn arn:aws:kms:region:account:key/key-id

# Create backup plan for EFS
aws backup create-backup-plan \
  --backup-plan '{
    "BackupPlanName": "EFSBackupPlan",
    "Rules": [{
      "RuleName": "DailyBackups",
      "TargetBackupVault": "EFSBackupVault",
      "ScheduleExpression": "cron(0 5 ? * * *)",
      "StartWindowMinutes": 480,
      "CompletionWindowMinutes": 10080,
      "Lifecycle": {
        "DeleteAfterDays": 30,
        "MoveToColdStorageAfterDays": 7
      },
      "RecoveryPointTags": {
        "BackupType": "Automated"
      }
    }]
  }'
```

### Analytics Integration

- **EMR**: Shared data storage for big data processing
- **SageMaker**: Machine learning dataset storage
- **Athena**: Query data directly from EFS
- **Glue**: ETL job data storage

## Security Considerations

### Encryption

1. **Encryption at Rest**
   - AWS KMS integration
   - Customer-managed keys
   - Automatic key rotation
   - Regional key management

2. **Encryption in Transit**
   - TLS 1.2 encryption
   - EFS utilities integration
   - Stunnel for legacy clients
   - Certificate management

```bash
# Mount with encryption in transit
sudo mount -t efs -o tls fs-12345678:/ /mnt/efs

# Mount with access point and encryption
sudo mount -t efs -o tls,accesspoint=fsap-12345678 fs-12345678:/ /mnt/app
```

### Access Control

1. **IAM Policies**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Action": [
         "elasticfilesystem:DescribeFileSystems",
         "elasticfilesystem:DescribeAccessPoints"
       ],
       "Resource": "*"
     }, {
       "Effect": "Allow",
       "Action": [
         "elasticfilesystem:CreateAccessPoint",
         "elasticfilesystem:DeleteAccessPoint"
       ],
       "Resource": "arn:aws:elasticfilesystem:*:*:file-system/fs-12345678"
     }]
   }
   ```

2. **Resource-Based Policies**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Principal": {
         "AWS": "arn:aws:iam::account:root"
       },
       "Action": [
         "elasticfilesystem:ClientMount",
         "elasticfilesystem:ClientWrite"
       ],
       "Resource": "*",
       "Condition": {
         "Bool": {
           "aws:SecureTransport": "true"
         }
       }
     }]
   }
   ```

3. **VPC Security**
   - Security groups for mount targets
   - Network ACLs
   - VPC endpoints
   - Private subnet deployment

### EFS Access Points

```bash
# Create access point with IAM integration
aws efs create-access-point \
  --file-system-id fs-12345678 \
  --posix-user Uid=1000,Gid=1000 \
  --root-directory Path="/restricted",CreationInfo='{
    "OwnerUid": 1000,
    "OwnerGid": 1000,
    "Permissions": "755"
  }' \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::account:user/developer"},
      "Action": "elasticfilesystem:ClientMount",
      "Resource": "*"
    }]
  }'
```

## Monitoring and Troubleshooting

### CloudWatch Metrics

#### File System Metrics
- **TotalIOBytes**: Total bytes for file system operations
- **DataReadIOBytes/DataWriteIOBytes**: Data transfer metrics
- **ClientConnections**: Number of client connections
- **StorageBytes**: Amount of storage used by storage class
- **BurstCreditBalance**: Available burst credits

#### Performance Metrics
- **TotalIOTime**: Total time for file system operations
- **PercentIOLimit**: Percentage of I/O limit used
- **ThroughputUtilization**: Percentage of throughput used
- **MetadataIOBytes**: Metadata operation bytes

### Monitoring Setup

```bash
# Create CloudWatch alarm for burst credits
aws cloudwatch put-metric-alarm \
  --alarm-name "EFS-Low-Burst-Credits" \
  --alarm-description "EFS burst credit balance is low" \
  --metric-name BurstCreditBalance \
  --namespace AWS/EFS \
  --statistic Average \
  --period 300 \
  --threshold 1000000 \
  --comparison-operator LessThanThreshold \
  --dimensions Name=FileSystemId,Value=fs-12345678

# Monitor storage utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EFS \
  --metric-name StorageBytes \
  --dimensions Name=FileSystemId,Value=fs-12345678 Name=StorageClass,Value=Standard \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average
```

### Common Issues and Solutions

1. **Mount Issues**
   - Verify security group rules (NFS port 2049)
   - Check subnet routing
   - Confirm mount target availability
   - Verify EFS utilities installation

2. **Performance Issues**
   - Monitor burst credit balance
   - Check throughput mode configuration
   - Verify client configuration
   - Consider performance mode change

3. **Permission Issues**
   - Check POSIX permissions
   - Verify IAM policies
   - Confirm access point configuration
   - Review resource-based policies

### Troubleshooting Commands

```bash
# Check mount status
mount | grep efs

# Verify connectivity to mount target
telnet fs-12345678.efs.region.amazonaws.com 2049

# Check EFS file system status
aws efs describe-file-systems --file-system-id fs-12345678

# Monitor file system performance
aws efs describe-file-systems \
  --file-system-id fs-12345678 \
  --query 'FileSystems[0].ThroughputMode'

# Check access points
aws efs describe-access-points \
  --file-system-id fs-12345678
```

## Exam-Specific Tips and Common Scenarios

### Key Exam Topics

1. **Storage Class Selection**
   - Standard vs IA vs Archive trade-offs
   - One Zone vs Regional considerations
   - Cost optimization strategies
   - Lifecycle management policies

2. **Performance Configuration**
   - General Purpose vs Max I/O modes
   - Bursting vs Provisioned vs Elastic throughput
   - Performance optimization techniques
   - Monitoring and troubleshooting

3. **Security and Access Control**
   - Encryption options and requirements
   - IAM vs resource-based policies
   - Access Points use cases
   - VPC security configurations

### Common Exam Scenarios

1. **Container Storage**
   - EKS persistent volumes
   - ECS task storage
   - Fargate container storage
   - Docker volume mounting

2. **Multi-AZ Applications**
   - Shared application data
   - Cross-AZ file access
   - High availability configurations
   - Disaster recovery planning

3. **Cost Optimization**
   - Lifecycle policy implementation
   - Storage class selection
   - Throughput mode optimization
   - Regional vs One Zone decisions

4. **Backup and Recovery**
   - AWS Backup integration
   - Point-in-time recovery
   - Cross-region backup strategies
   - Compliance requirements

### Exam Tips

- **Know the differences** between storage classes and their use cases
- **Understand performance modes** and when to use each
- **Remember encryption options** and their implications
- **Know integration patterns** with EC2, containers, and Lambda
- **Understand cost factors** and optimization strategies

## Hands-on Examples and CLI Commands

### File System Operations

```bash
# Create encrypted EFS with lifecycle management
aws efs create-file-system \
  --creation-token production-efs-$(date +%s) \
  --performance-mode generalPurpose \
  --throughput-mode elastic \
  --encrypted \
  --kms-key-id alias/my-efs-key \
  --tags Key=Name,Value=ProductionEFS Key=Environment,Value=Production

# Set lifecycle policy
aws efs put-lifecycle-configuration \
  --file-system-id fs-12345678 \
  --lifecycle-policies \
    TransitionToIA=AFTER_30_DAYS,TransitionToArchive=AFTER_90_DAYS

# Create replication configuration
aws efs create-replication-configuration \
  --source-file-system-id fs-12345678 \
  --destinations Region=us-east-1,KmsKeyId=alias/my-efs-key-east
```

### Mount Target Management

```bash
# Create mount targets in multiple subnets
for subnet in subnet-12345678 subnet-87654321 subnet-11111111; do
  aws efs create-mount-target \
    --file-system-id fs-12345678 \
    --subnet-id $subnet \
    --security-groups sg-12345678
done

# Modify mount target security groups
aws efs modify-mount-target-security-groups \
  --mount-target-id fsmt-12345678 \
  --security-groups sg-12345678 sg-87654321
```

### Performance Testing

```bash
# Install EFS utilities and fio
sudo yum install -y amazon-efs-utils fio

# Mount EFS with optimal settings
sudo mkdir /mnt/efs-test
sudo mount -t efs -o tls,fsc fs-12345678:/ /mnt/efs-test

# Test sequential write performance
sudo fio --name=efs-sequential-write \
  --ioengine=libaio \
  --iodepth=16 \
  --rw=write \
  --bs=1M \
  --direct=1 \
  --size=1G \
  --numjobs=4 \
  --runtime=60 \
  --group_reporting \
  --directory=/mnt/efs-test

# Test random read performance
sudo fio --name=efs-random-read \
  --ioengine=libaio \
  --iodepth=32 \
  --rw=randread \
  --bs=4k \
  --direct=1 \
  --size=1G \
  --numjobs=4 \
  --runtime=60 \
  --group_reporting \
  --directory=/mnt/efs-test
```

### Application Integration

```bash
# Configure EFS for web application
sudo mkdir -p /var/www/shared
sudo mount -t efs -o tls,accesspoint=fsap-12345678 fs-12345678:/ /var/www/shared
sudo chown www-data:www-data /var/www/shared

# Set up automatic mounting
echo 'fs-12345678.efs.region.amazonaws.com:/ /var/www/shared efs tls,accesspoint=fsap-12345678,_netdev 0 0' \
  | sudo tee -a /etc/fstab
```

This comprehensive EFS documentation covers all essential aspects for AWS certification exams, providing both theoretical knowledge and practical implementation examples.