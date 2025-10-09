# Amazon RDS (Relational Database Service)

## Service Overview and Purpose

Amazon RDS is a fully managed relational database service that makes it easy to set up, operate, and scale a relational database in the cloud. RDS provides cost-efficient and resizable capacity while automating time-consuming administration tasks such as hardware provisioning, database setup, patching, and backups.

### Key Characteristics
- **Fully Managed**: Automated management of infrastructure, patching, and backups
- **Multiple Engines**: Support for various database engines
- **High Availability**: Multi-AZ deployments for failover protection
- **Scalable**: Vertical and horizontal scaling options
- **Secure**: Encryption, network isolation, and access controls

## Key Features and Capabilities

### Supported Database Engines

1. **Amazon Aurora**
   - **MySQL Compatible**: Up to 5x performance of MySQL
   - **PostgreSQL Compatible**: Up to 3x performance of PostgreSQL
   - **Cloud Native**: Built for the cloud architecture
   - **Serverless**: Aurora Serverless for variable workloads

2. **MySQL**
   - **Versions**: 5.7, 8.0
   - **Community Edition**: Open-source MySQL
   - **Performance**: Optimized for cloud deployment
   - **Compatibility**: Full MySQL compatibility

3. **PostgreSQL**
   - **Versions**: 11, 12, 13, 14, 15
   - **Extensions**: Support for PostgreSQL extensions
   - **Advanced Features**: JSON, spatial data, full-text search
   - **ACID Compliance**: Full transactional support

4. **MariaDB**
   - **Versions**: 10.3, 10.4, 10.5, 10.6
   - **MySQL Fork**: Enhanced MySQL-compatible database
   - **Performance**: Advanced optimizer and storage engines
   - **Open Source**: Community-driven development

5. **Oracle**
   - **Versions**: 12c, 18c, 19c, 21c
   - **Editions**: Standard and Enterprise editions
   - **Licensing**: BYOL (Bring Your Own License) or License Included
   - **Enterprise Features**: Advanced security, analytics

6. **SQL Server**
   - **Versions**: 2016, 2017, 2019, 2022
   - **Editions**: Express, Web, Standard, Enterprise
   - **Licensing**: BYOL or License Included
   - **Windows Integration**: Active Directory integration

### Core Features

#### High Availability and Disaster Recovery
- **Multi-AZ Deployments**: Automatic failover to standby instance
- **Read Replicas**: Scale read workloads and disaster recovery
- **Cross-Region Replication**: Geographic disaster recovery
- **Automated Backups**: Point-in-time recovery capability
- **Manual Snapshots**: User-initiated backups

#### Performance and Scaling
- **Instance Classes**: Various CPU, memory, and network configurations
- **Storage Types**: General Purpose (gp2/gp3), Provisioned IOPS (io1/io2)
- **Storage Autoscaling**: Automatic storage capacity increases
- **Performance Insights**: Database performance monitoring
- **Enhanced Monitoring**: Real-time operating system metrics

#### Security Features
- **Encryption at Rest**: KMS-based encryption
- **Encryption in Transit**: SSL/TLS connections
- **Network Isolation**: VPC deployment
- **IAM Integration**: Database authentication via IAM
- **Database Activity Streams**: Real-time audit logging

#### Maintenance and Management
- **Automated Patching**: Operating system and database patching
- **Maintenance Windows**: Scheduled maintenance periods
- **Parameter Groups**: Database configuration management
- **Option Groups**: Database feature configuration
- **Event Notifications**: SNS integration for alerts

## Use Cases and Scenarios

### Primary Use Cases

1. **Web and Mobile Applications**
   - **E-commerce Platforms**: Product catalogs, user accounts, transactions
   - **Content Management**: Article storage, user-generated content
   - **Social Media**: User profiles, posts, relationships
   - **Gaming**: Player data, scores, achievements

2. **Enterprise Applications**
   - **ERP Systems**: Business process management
   - **CRM Applications**: Customer data management
   - **Financial Systems**: Transaction processing, reporting
   - **HR Systems**: Employee data, payroll processing

3. **Analytics and Reporting**
   - **Data Warehousing**: Historical data analysis
   - **Business Intelligence**: Dashboard and reporting
   - **ETL Workloads**: Data transformation processes
   - **Compliance Reporting**: Regulatory data requirements

4. **Development and Testing**
   - **Development Databases**: Application development
   - **Testing Environments**: QA and staging databases
   - **CI/CD Pipelines**: Automated testing databases
   - **Prototyping**: Rapid application development

### Architecture Patterns

1. **Three-Tier Web Architecture**
   - Web tier: Load balancer and web servers
   - Application tier: Application servers
   - Database tier: RDS Multi-AZ deployment

2. **Microservices Architecture**
   - Database per service pattern
   - Multiple RDS instances for different services
   - Event-driven communication

3. **Hybrid Cloud Architecture**
   - On-premises to RDS migration
   - Hybrid data synchronization
   - Disaster recovery in cloud

## Pricing Models and Cost Optimization

### Pricing Components

1. **Instance Costs**
   - **On-Demand**: Hourly pricing with no commitments
   - **Reserved Instances**: 1-year or 3-year commitments
   - **Instance Classes**: db.t3, db.m5, db.r5, db.x1e families
   - **Multi-AZ**: Additional cost for standby instance

2. **Storage Costs**
   - **General Purpose (gp2)**: $0.115 per GB-month
   - **General Purpose (gp3)**: $0.115 per GB-month + IOPS/throughput
   - **Provisioned IOPS**: $0.125 per GB-month + IOPS costs
   - **Magnetic**: $0.10 per GB-month (legacy)

3. **Backup Storage**
   - **Free Tier**: Backup storage equal to database size
   - **Additional**: $0.095 per GB-month for excess backup storage
   - **Cross-Region**: Additional costs for cross-region backups

4. **Data Transfer**
   - **Inbound**: Free data transfer
   - **Outbound**: Internet egress charges
   - **Cross-AZ**: $0.01 per GB for Multi-AZ data transfer

### Cost Optimization Strategies

1. **Right-Size Instances**
   ```bash
   # Monitor database performance
   aws rds describe-db-instances \
     --db-instance-identifier mydb \
     --query 'DBInstances[0].DBInstanceClass'

   # Use CloudWatch metrics for sizing decisions
   aws cloudwatch get-metric-statistics \
     --namespace AWS/RDS \
     --metric-name CPUUtilization \
     --dimensions Name=DBInstanceIdentifier,Value=mydb \
     --start-time 2023-01-01T00:00:00Z \
     --end-time 2023-01-02T00:00:00Z \
     --period 3600 \
     --statistics Average,Maximum
   ```

2. **Use Reserved Instances**
   - Analyze usage patterns
   - Purchase RIs for steady-state workloads
   - Consider partial upfront payment
   - Plan for 1-year or 3-year terms

3. **Optimize Storage**
   - Use gp3 instead of gp2 for cost savings
   - Enable storage autoscaling
   - Monitor storage utilization
   - Consider magnetic storage for dev/test

4. **Backup Optimization**
   - Set appropriate backup retention periods
   - Delete unnecessary manual snapshots
   - Use cross-region backups judiciously
   - Implement lifecycle policies

## Configuration Details and Best Practices

### RDS Instance Creation

```bash
# Create RDS instance with best practices
aws rds create-db-instance \
  --db-instance-identifier production-mysql \
  --db-instance-class db.r5.xlarge \
  --engine mysql \
  --engine-version 8.0.32 \
  --master-username admin \
  --master-user-password SecurePassword123! \
  --allocated-storage 100 \
  --storage-type gp3 \
  --iops 3000 \
  --storage-throughput 125 \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name production-subnet-group \
  --multi-az \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:05:00" \
  --storage-encrypted \
  --kms-key-id alias/rds-encryption-key \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::account:role/rds-monitoring-role \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --deletion-protection \
  --tags Key=Environment,Value=Production Key=Application,Value=WebApp
```

### Parameter Groups

```bash
# Create custom parameter group
aws rds create-db-parameter-group \
  --db-parameter-group-name mysql-production-params \
  --db-parameter-group-family mysql8.0 \
  --description "Production MySQL 8.0 parameters"

# Modify parameters
aws rds modify-db-parameter-group \
  --db-parameter-group-name mysql-production-params \
  --parameters ParameterName=innodb_buffer_pool_size,ParameterValue="{DBInstanceClassMemory*3/4}",ApplyMethod=pending-reboot \
               ParameterName=max_connections,ParameterValue=1000,ApplyMethod=immediate \
               ParameterName=slow_query_log,ParameterValue=1,ApplyMethod=immediate

# Apply parameter group to instance
aws rds modify-db-instance \
  --db-instance-identifier production-mysql \
  --db-parameter-group-name mysql-production-params \
  --apply-immediately
```

### Security Configuration

```bash
# Create security group for RDS
aws ec2 create-security-group \
  --group-name rds-mysql-sg \
  --description "Security group for MySQL RDS instance" \
  --vpc-id vpc-12345678

# Add inbound rule for MySQL
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-87654321

# Enable encryption for existing instance (via snapshot)
aws rds create-db-snapshot \
  --db-instance-identifier production-mysql \
  --db-snapshot-identifier mysql-snapshot-$(date +%Y%m%d)

aws rds copy-db-snapshot \
  --source-db-snapshot-identifier mysql-snapshot-$(date +%Y%m%d) \
  --target-db-snapshot-identifier mysql-encrypted-snapshot \
  --kms-key-id alias/rds-encryption-key
```

### Best Practices

1. **High Availability Design**
   - **Multi-AZ**: Enable for production workloads
   - **Read Replicas**: Scale read workloads and disaster recovery
   - **Cross-Region**: For geographic disaster recovery
   - **Monitoring**: Implement comprehensive monitoring

2. **Security Hardening**
   - **Encryption**: Enable encryption at rest and in transit
   - **Network Isolation**: Deploy in private subnets
   - **Access Control**: Use IAM database authentication
   - **Auditing**: Enable database activity streams

3. **Performance Optimization**
   - **Instance Sizing**: Right-size based on workload
   - **Storage Configuration**: Choose appropriate storage type
   - **Parameter Tuning**: Optimize database parameters
   - **Monitoring**: Use Performance Insights and Enhanced Monitoring

4. **Operational Excellence**
   - **Backup Strategy**: Automated and manual backups
   - **Maintenance Planning**: Schedule maintenance windows
   - **Change Management**: Use parameter and option groups
   - **Documentation**: Maintain operational runbooks

### Read Replicas Configuration

```bash
# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier production-mysql-replica \
  --source-db-instance-identifier production-mysql \
  --db-instance-class db.r5.large \
  --publicly-accessible false \
  --auto-minor-version-upgrade true \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::account:role/rds-monitoring-role \
  --enable-performance-insights \
  --tags Key=Environment,Value=Production Key=Role,Value=ReadReplica

# Create cross-region read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier dr-mysql-replica \
  --source-db-instance-identifier arn:aws:rds:us-west-2:account:db:production-mysql \
  --db-instance-class db.r5.large \
  --region us-east-1 \
  --kms-key-id alias/rds-encryption-key-east
```

## Integration with Other AWS Services

### Application Integration

1. **Compute Services**
   ```python
   # Python application connection example
   import pymysql
   import boto3

   # Get database credentials from Secrets Manager
   secrets_client = boto3.client('secretsmanager')
   secret = secrets_client.get_secret_value(SecretId='rds-mysql-credentials')
   credentials = json.loads(secret['SecretString'])

   # Connect to RDS
   connection = pymysql.connect(
       host='production-mysql.cluster-xyz.us-west-2.rds.amazonaws.com',
       user=credentials['username'],
       password=credentials['password'],
       database='myapp',
       ssl={'ssl': True}
   )
   ```

2. **Lambda Integration**
   ```python
   # Lambda function with RDS connection
   import json
   import pymysql
   import os

   def lambda_handler(event, context):
       connection = pymysql.connect(
           host=os.environ['RDS_ENDPOINT'],
           user=os.environ['DB_USERNAME'],
           password=os.environ['DB_PASSWORD'],
           database=os.environ['DB_NAME'],
           ssl={'ssl': True}
       )

       with connection.cursor() as cursor:
           cursor.execute("SELECT COUNT(*) FROM users")
           result = cursor.fetchone()

       connection.close()

       return {
           'statusCode': 200,
           'body': json.dumps({'user_count': result[0]})
       }
   ```

### Monitoring and Analytics Integration

```bash
# Export logs to CloudWatch
aws rds modify-db-instance \
  --db-instance-identifier production-mysql \
  --cloudwatch-logs-exports-configuration LogTypesToEnable=error,general,slow-query \
  --apply-immediately

# Create CloudWatch alarm for high CPU
aws cloudwatch put-metric-alarm \
  --alarm-name "RDS-High-CPU" \
  --alarm-description "RDS CPU utilization is high" \
  --metric-name CPUUtilization \
  --namespace AWS/RDS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=DBInstanceIdentifier,Value=production-mysql \
  --alarm-actions arn:aws:sns:region:account:rds-alerts
```

### Backup and Recovery Integration

```bash
# AWS Backup integration
aws backup create-backup-plan \
  --backup-plan '{
    "BackupPlanName": "RDS-Backup-Plan",
    "Rules": [{
      "RuleName": "DailyBackups",
      "TargetBackupVault": "RDSBackupVault",
      "ScheduleExpression": "cron(0 5 ? * * *)",
      "StartWindowMinutes": 480,
      "CompletionWindowMinutes": 10080,
      "Lifecycle": {
        "DeleteAfterDays": 30
      },
      "RecoveryPointTags": {
        "BackupType": "Daily"
      }
    }]
  }'

# Create backup selection for RDS
aws backup create-backup-selection \
  --backup-plan-id backup-plan-id \
  --backup-selection '{
    "SelectionName": "RDS-Instances",
    "IamRoleArn": "arn:aws:iam::account:role/AWSBackupDefaultServiceRole",
    "Resources": [
      "arn:aws:rds:*:*:db:production-*"
    ]
  }'
```

### Data Migration Integration

- **DMS**: Database Migration Service for migrations
- **SCT**: Schema Conversion Tool for engine migrations
- **DataSync**: For file-based database transfers
- **Glue**: ETL operations with RDS as source/target

## Security Considerations

### Encryption

1. **Encryption at Rest**
   ```bash
   # Enable encryption for new instance
   aws rds create-db-instance \
     --db-instance-identifier encrypted-mysql \
     --storage-encrypted \
     --kms-key-id alias/rds-encryption-key \
     # ... other parameters
   ```

2. **Encryption in Transit**
   ```bash
   # Force SSL connections (MySQL example)
   aws rds modify-db-parameter-group \
     --db-parameter-group-name mysql-production-params \
     --parameters ParameterName=require_secure_transport,ParameterValue=ON,ApplyMethod=pending-reboot
   ```

3. **IAM Database Authentication**
   ```bash
   # Enable IAM authentication
   aws rds modify-db-instance \
     --db-instance-identifier production-mysql \
     --enable-iam-database-authentication \
     --apply-immediately

   # Create IAM user for database access
   aws iam create-user --user-name db-user
   aws iam attach-user-policy \
     --user-name db-user \
     --policy-arn arn:aws:iam::account:policy/RDSDBAccess
   ```

### Network Security

1. **VPC Configuration**
   ```bash
   # Create DB subnet group
   aws rds create-db-subnet-group \
     --db-subnet-group-name production-subnet-group \
     --db-subnet-group-description "Production database subnet group" \
     --subnet-ids subnet-12345678 subnet-87654321 \
     --tags Key=Environment,Value=Production
   ```

2. **Security Groups**
   ```bash
   # Create restrictive security group
   aws ec2 create-security-group \
     --group-name rds-restricted-sg \
     --description "Restricted access to RDS" \
     --vpc-id vpc-12345678

   # Allow access only from application servers
   aws ec2 authorize-security-group-ingress \
     --group-id sg-12345678 \
     --protocol tcp \
     --port 3306 \
     --source-group sg-application-servers
   ```

### Access Control and Auditing

1. **Database Activity Streams**
   ```bash
   # Enable database activity streams
   aws rds start-activity-stream \
     --resource-arn arn:aws:rds:region:account:db:production-mysql \
     --mode async \
     --kms-key-id alias/das-encryption-key \
     --kinesis-stream-name rds-activity-stream
   ```

2. **Performance Insights Encryption**
   ```bash
   # Enable Performance Insights with encryption
   aws rds modify-db-instance \
     --db-instance-identifier production-mysql \
     --enable-performance-insights \
     --performance-insights-kms-key-id alias/performance-insights-key \
     --apply-immediately
   ```

## Monitoring and Troubleshooting

### CloudWatch Metrics

#### Database Metrics
- **CPUUtilization**: CPU usage percentage
- **DatabaseConnections**: Number of connections
- **FreeableMemory**: Available memory
- **FreeStorageSpace**: Available storage
- **ReadLatency/WriteLatency**: I/O latency
- **ReadIOPS/WriteIOPS**: I/O operations per second

#### Engine-Specific Metrics
- **BinLogDiskUsage**: MySQL binary log usage
- **ReplicaLag**: Read replica lag
- **SwapUsage**: Swap file usage
- **NetworkReceiveThroughput/NetworkTransmitThroughput**: Network I/O

### Enhanced Monitoring

```bash
# Enable Enhanced Monitoring
aws rds modify-db-instance \
  --db-instance-identifier production-mysql \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::account:role/rds-monitoring-role \
  --apply-immediately

# Create CloudWatch dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "RDS-Monitoring" \
  --dashboard-body '{
    "widgets": [{
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/RDS", "CPUUtilization", "DBInstanceIdentifier", "production-mysql"],
          [".", "DatabaseConnections", ".", "."],
          [".", "FreeableMemory", ".", "."]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-west-2",
        "title": "RDS Performance Metrics"
      }
    }]
  }'
```

### Performance Insights

```bash
# Enable Performance Insights
aws rds modify-db-instance \
  --db-instance-identifier production-mysql \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --performance-insights-kms-key-id alias/performance-insights-key \
  --apply-immediately

# Query Performance Insights data
aws pi get-resource-metrics \
  --service-type RDS \
  --identifier production-mysql \
  --metric-queries MetricQuery='{
    "Metric": "db.SQL.SELECT.AvgLatency.avg",
    "GroupBy": {"Group": "db.sql_tokenized.statement"}
  }' \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T01:00:00Z \
  --period-in-seconds 300
```

### Common Issues and Solutions

1. **Connection Issues**
   - **Security Groups**: Verify port 3306/5432 is open
   - **Subnet Groups**: Ensure proper subnet configuration
   - **DNS Resolution**: Check endpoint accessibility
   - **Authentication**: Verify credentials and IAM policies

2. **Performance Issues**
   - **Instance Sizing**: Check CPU and memory utilization
   - **Storage Performance**: Monitor IOPS and throughput
   - **Parameter Tuning**: Optimize database parameters
   - **Query Optimization**: Use Performance Insights

3. **Storage Issues**
   - **Space Management**: Monitor free storage space
   - **Autoscaling**: Enable storage autoscaling
   - **Backup Storage**: Monitor backup storage usage
   - **Log Management**: Configure log retention

### Troubleshooting Commands

```bash
# Check instance status
aws rds describe-db-instances \
  --db-instance-identifier production-mysql \
  --query 'DBInstances[0].DBInstanceStatus'

# View recent events
aws rds describe-events \
  --source-identifier production-mysql \
  --source-type db-instance \
  --start-time 2023-01-01T00:00:00Z

# Check parameter group status
aws rds describe-db-instances \
  --db-instance-identifier production-mysql \
  --query 'DBInstances[0].DBParameterGroups'

# Monitor backup status
aws rds describe-db-snapshots \
  --db-instance-identifier production-mysql \
  --snapshot-type automated \
  --max-records 5
```

## Exam-Specific Tips and Common Scenarios

### Key Exam Topics

1. **Engine Selection**
   - MySQL vs PostgreSQL vs Oracle vs SQL Server
   - Aurora benefits and use cases
   - Licensing considerations (BYOL vs LI)
   - Performance characteristics

2. **High Availability**
   - Multi-AZ vs Read Replicas
   - Cross-region disaster recovery
   - Failover mechanisms
   - RTO/RPO considerations

3. **Security Configuration**
   - Encryption at rest and in transit
   - IAM database authentication
   - VPC security and network isolation
   - Database activity streams

### Common Exam Scenarios

1. **Migration from On-Premises**
   - Choose appropriate engine and version
   - Plan for minimal downtime migration
   - Use DMS for data migration
   - Configure security and networking

2. **High Availability Requirements**
   - Multi-AZ for automatic failover
   - Read replicas for read scaling
   - Cross-region replicas for DR
   - Backup and recovery strategies

3. **Performance Optimization**
   - Right-size instances based on workload
   - Choose appropriate storage type
   - Configure parameter groups
   - Use Performance Insights for monitoring

4. **Cost Optimization**
   - Use Reserved Instances for steady workloads
   - Right-size instances based on utilization
   - Optimize storage configuration
   - Manage backup retention

### Exam Tips

- **Know the differences** between Multi-AZ and Read Replicas
- **Understand engine-specific** features and limitations
- **Remember encryption** requirements and implementation
- **Know backup and recovery** options and RPO/RTO
- **Understand cost factors** and optimization strategies

## Hands-on Examples and CLI Commands

### Complete RDS Setup

```bash
# 1. Create subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name production-db-subnet-group \
  --db-subnet-group-description "Production database subnet group" \
  --subnet-ids subnet-12345678 subnet-87654321 subnet-11111111 \
  --tags Key=Environment,Value=Production

# 2. Create parameter group
aws rds create-db-parameter-group \
  --db-parameter-group-name mysql-8-0-production \
  --db-parameter-group-family mysql8.0 \
  --description "Production MySQL 8.0 parameter group"

# 3. Create option group (if needed)
aws rds create-option-group \
  --option-group-name mysql-8-0-production-options \
  --engine-name mysql \
  --major-engine-version 8.0 \
  --option-group-description "Production MySQL 8.0 options"

# 4. Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier production-mysql-primary \
  --db-instance-class db.r5.2xlarge \
  --engine mysql \
  --engine-version 8.0.32 \
  --master-username dbadmin \
  --master-user-password $(aws secretsmanager get-random-password --password-length 32 --exclude-characters '"@/\' --query 'RandomPassword' --output text) \
  --allocated-storage 500 \
  --storage-type gp3 \
  --iops 12000 \
  --storage-throughput 500 \
  --storage-encrypted \
  --kms-key-id alias/rds-encryption-key \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name production-db-subnet-group \
  --db-parameter-group-name mysql-8-0-production \
  --backup-retention-period 14 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:06:00" \
  --multi-az \
  --auto-minor-version-upgrade \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::account:role/rds-monitoring-role \
  --enable-performance-insights \
  --performance-insights-retention-period 31 \
  --performance-insights-kms-key-id alias/performance-insights-key \
  --enable-cloudwatch-logs-exports error,general,slow-query \
  --deletion-protection \
  --copy-tags-to-snapshot \
  --tags Key=Environment,Value=Production Key=Application,Value=WebApp Key=Owner,Value=DBA-Team

# 5. Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier production-mysql-replica-1 \
  --source-db-instance-identifier production-mysql-primary \
  --db-instance-class db.r5.xlarge \
  --publicly-accessible false \
  --auto-minor-version-upgrade true \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::account:role/rds-monitoring-role \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --tags Key=Environment,Value=Production Key=Role,Value=ReadReplica

# 6. Create cross-region replica for DR
aws rds create-db-instance-read-replica \
  --db-instance-identifier dr-mysql-replica \
  --source-db-instance-identifier arn:aws:rds:us-west-2:account:db:production-mysql-primary \
  --db-instance-class db.r5.large \
  --region us-east-1 \
  --kms-key-id alias/rds-encryption-key-east \
  --tags Key=Environment,Value=Production Key=Role,Value=DisasterRecovery
```

This comprehensive RDS documentation covers all essential aspects needed for AWS certification exams, providing both theoretical knowledge and practical implementation examples.