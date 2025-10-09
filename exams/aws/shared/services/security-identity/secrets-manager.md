# AWS Secrets Manager

## Service Overview and Purpose

AWS Secrets Manager helps you protect secrets needed to access your applications, services, and IT resources. The service enables you to easily rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle. Users and applications retrieve secrets with a call to Secrets Manager APIs, eliminating the need to hardcode sensitive information in plain text.

**Core Purpose:**
- Centralized secrets storage and management
- Automated secrets rotation and lifecycle management
- Secure access control for sensitive credentials
- Integration with AWS services and applications
- Compliance with security best practices and regulations

## Key Features and Capabilities

### Core Secret Management

1. **Secret Storage**
   - Encrypted storage of sensitive data
   - Support for text, binary, and structured data
   - Version control and history tracking
   - Cross-region replication for disaster recovery
   - Fine-grained access control

2. **Automatic Rotation**
   - Built-in rotation for RDS databases
   - Custom rotation functions for other services
   - Configurable rotation schedules
   - Zero-downtime rotation process
   - Rollback capabilities for failed rotations

3. **Integration Support**
   - Native AWS service integration
   - Application SDK support
   - CLI and console access
   - Third-party application support
   - Custom application integration

### Advanced Features

1. **Cross-Region Replication**
   - Multi-region secret availability
   - Disaster recovery support
   - Reduced latency for global applications
   - Automatic synchronization
   - Regional failover capabilities

2. **Secret Sharing**
   - Cross-account secret sharing
   - Resource-based policies
   - Organization-wide sharing
   - Granular permission control
   - Audit trail for access

3. **Lambda Integration**
   - Custom rotation functions
   - Event-driven secret management
   - Automated secret generation
   - Integration with deployment pipelines
   - Custom business logic implementation

## Use Cases and Scenarios

### Primary Use Cases

1. **Database Credentials**: Secure storage and rotation of database passwords
2. **API Keys**: Management of third-party service API keys
3. **Application Secrets**: Configuration secrets for applications
4. **SSL Certificates**: Private key and certificate storage
5. **OAuth Tokens**: Token storage and refresh management
6. **SSH Keys**: Secure storage of SSH private keys
7. **Configuration Data**: Sensitive application configuration
8. **Service Accounts**: Service account credentials management
9. **DevOps Automation**: Secrets for CI/CD pipelines
10. **Multi-tenant Applications**: Tenant-specific secret management

### Scenario Examples

- **Web Application**: Database connection strings and API keys
- **Microservices**: Service-to-service authentication credentials
- **Mobile Application**: Backend service API keys and certificates
- **Data Pipeline**: Database and service credentials for ETL processes
- **DevOps Pipeline**: Deployment credentials and configuration secrets

## Pricing Models and Cost Optimization

### Pricing Structure

1. **Secret Storage**
   - $0.40 per secret per month
   - Prorated for partial months
   - No charge for deleted secrets

2. **API Requests**
   - $0.05 per 10,000 API requests
   - All API calls including retrieve, create, update, delete
   - No charge for AWS service calls on your behalf

3. **Cross-Region Replication**
   - Additional storage charges for replica secrets
   - Standard pricing for replicated secrets
   - No additional API charges for replication

### Cost Optimization Strategies

1. **Secret Lifecycle Management**: Regular cleanup of unused secrets
2. **API Optimization**: Implement caching to reduce API calls
3. **Regional Strategy**: Optimize cross-region replication usage
4. **Secret Consolidation**: Combine related secrets when appropriate
5. **Monitoring**: Track usage patterns and optimize accordingly
6. **Automated Cleanup**: Implement automated secret deletion policies

## Configuration Details and Best Practices

### Secret Configuration

```json
{
  "Name": "prod/myapp/database",
  "Description": "Production database credentials",
  "SecretString": "{\"username\":\"admin\",\"password\":\"secure-password\",\"engine\":\"mysql\",\"host\":\"prod-db.cluster-xyz.us-east-1.rds.amazonaws.com\",\"port\":3306,\"dbname\":\"myapp\"}",
  "KmsKeyId": "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012",
  "ReplicationRegions": [
    {
      "Region": "us-west-2",
      "KmsKeyId": "arn:aws:kms:us-west-2:123456789012:key/87654321-4321-4321-4321-210987654321"
    }
  ],
  "ForceOverwriteReplicaSecret": false
}
```

### Best Practices

1. **Naming Conventions**
   - Use hierarchical naming (environment/application/resource)
   - Consistent naming patterns across organization
   - Descriptive names for easy identification
   - Avoid sensitive information in names
   - Include versioning strategy in naming

2. **Access Control**
   - Implement least privilege access
   - Use resource-based policies for granular control
   - Regular access review and audit
   - Separate permissions for different environments
   - Use IAM roles instead of users when possible

3. **Rotation Strategy**
   - Enable automatic rotation for critical secrets
   - Test rotation procedures regularly
   - Implement rollback procedures
   - Monitor rotation success and failures
   - Document rotation dependencies

4. **Security Configuration**
   - Use customer-managed KMS keys for encryption
   - Enable CloudTrail logging for audit
   - Implement cross-region replication for critical secrets
   - Regular security reviews and updates
   - Monitor access patterns for anomalies

## Integration with Other AWS Services

### Core Integrations

1. **RDS/Aurora**: Automatic database credential rotation
2. **Lambda**: Custom rotation functions and secret retrieval
3. **ECS/EKS**: Container secret injection
4. **EC2**: Application secret retrieval
5. **CloudFormation**: Infrastructure as code secret management
6. **CodePipeline**: CI/CD secret management
7. **Systems Manager**: Parameter Store integration
8. **IAM**: Access control and permissions
9. **KMS**: Encryption key management
10. **CloudWatch**: Monitoring and alerting

### Advanced Integrations

1. **API Gateway**: Secure API authentication
2. **Cognito**: User pool and identity pool secrets
3. **SES**: SMTP credential management
4. **ElastiCache**: Redis/Memcached authentication
5. **DocumentDB**: Database credential rotation
6. **Redshift**: Data warehouse credential management
7. **Step Functions**: Workflow secret management
8. **EventBridge**: Event-driven secret operations

## Security Considerations

### Encryption and Data Protection

- **Encryption at Rest**: AES-256 encryption using KMS
- **Encryption in Transit**: TLS 1.2+ for all API communications
- **Key Management**: Customer-managed or AWS-managed KMS keys
- **Access Logging**: Comprehensive CloudTrail integration
- **Data Residency**: Regional data storage and processing

### Access Control and Monitoring

- **Fine-grained Permissions**: Resource-based and identity-based policies
- **Multi-factor Authentication**: Support for MFA requirements
- **Network Access**: VPC endpoint support for private access
- **Audit Trail**: Complete access and modification logging
- **Anomaly Detection**: CloudWatch integration for monitoring

### Compliance and Governance

- **SOC**: SOC 1, SOC 2, SOC 3 compliance
- **PCI DSS**: Payment card industry compliance
- **HIPAA**: Healthcare data protection
- **ISO 27001**: Information security management
- **FedRAMP**: Government security authorization

## Monitoring and Troubleshooting

### CloudWatch Metrics

- **SuccessfulRequestLatency**: Latency for successful requests
- **ProvisionedThroughputExceeded**: Throttling metrics
- **UserErrors**: Client-side error count
- **SystemErrors**: Service-side error count

### Common Issues and Solutions

1. **Access Denied Errors**
   - Verify IAM permissions and resource policies
   - Check cross-account access configuration
   - Validate KMS key permissions
   - Review VPC endpoint configuration

2. **Rotation Failures**
   - Check Lambda function permissions and configuration
   - Verify database connectivity and credentials
   - Review rotation function logs
   - Validate network access and security groups

3. **Performance Issues**
   - Implement client-side caching
   - Monitor API request patterns
   - Check for throttling errors
   - Optimize secret retrieval frequency

4. **Cross-Region Issues**
   - Verify replication configuration
   - Check regional service availability
   - Validate KMS key access in target regions
   - Monitor replication lag and errors

### Monitoring Tools

1. **CloudWatch**: Native monitoring and alerting
2. **CloudTrail**: API call auditing and compliance
3. **Config**: Configuration compliance monitoring
4. **X-Ray**: Application performance monitoring
5. **Third-party**: External monitoring and SIEM integration

## Exam-Specific Tips and Common Scenarios

### Solutions Architect Associate (SAA-C03)

- **Secret Management Architecture**: Implementing centralized secret management
- **Application Integration**: Secure application credential management
- **Cost Optimization**: Balancing security with operational costs
- **High Availability**: Cross-region replication strategies

### Solutions Architect Professional (SAP-C02)

- **Enterprise Secret Strategy**: Large-scale secret management architecture
- **Compliance Architecture**: Meeting regulatory secret management requirements
- **Automation Strategy**: Advanced secret lifecycle automation
- **Multi-Account Management**: Centralized secret management across accounts

### Security Specialty (SCS-C02)

- **Advanced Security**: Comprehensive secret security strategies
- **Compliance Management**: Regulatory compliance automation
- **Incident Response**: Secret compromise response procedures
- **Integration Security**: Secure integration with external systems

### Developer Associate (DVA-C02)

- **Application Integration**: Using Secrets Manager in application development
- **SDK Implementation**: Implementing secret retrieval in applications
- **CI/CD Integration**: Secret management in deployment pipelines
- **Error Handling**: Robust secret retrieval error handling

### Common Exam Scenarios

1. **Scenario**: Application needs secure database credential storage
   **Solution**: Use Secrets Manager with automatic rotation for RDS

2. **Scenario**: Multi-region application requires secret availability
   **Solution**: Configure cross-region replication for critical secrets

3. **Scenario**: Need to rotate API keys automatically
   **Solution**: Implement custom Lambda rotation function

4. **Scenario**: Container applications need secure secret injection
   **Solution**: Use ECS/EKS integration with Secrets Manager

## Hands-on Examples and CLI Commands

### Secret Management Operations

```bash
# Create a secret
aws secretsmanager create-secret \
  --name "prod/myapp/database" \
  --description "Production database credentials" \
  --secret-string '{
    "username": "admin",
    "password": "secure-password-123",
    "engine": "mysql",
    "host": "prod-db.cluster-xyz.us-east-1.rds.amazonaws.com",
    "port": 3306,
    "dbname": "myapp"
  }' \
  --kms-key-id "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"

# Generate random password for secret
aws secretsmanager get-random-password \
  --password-length 32 \
  --exclude-characters '"@/\' \
  --include-space false

# Update secret value
aws secretsmanager update-secret \
  --secret-id "prod/myapp/database" \
  --secret-string '{
    "username": "admin",
    "password": "new-secure-password-456",
    "engine": "mysql",
    "host": "prod-db.cluster-xyz.us-east-1.rds.amazonaws.com",
    "port": 3306,
    "dbname": "myapp"
  }'

# Retrieve secret value
aws secretsmanager get-secret-value \
  --secret-id "prod/myapp/database" \
  --query 'SecretString' \
  --output text

# List secrets
aws secretsmanager list-secrets \
  --filters Key=name,Values=prod/ \
  --max-results 20

# Describe secret
aws secretsmanager describe-secret \
  --secret-id "prod/myapp/database"

# Delete secret (with recovery window)
aws secretsmanager delete-secret \
  --secret-id "prod/myapp/database" \
  --recovery-window-in-days 30

# Cancel secret deletion
aws secretsmanager cancel-delete-secret \
  --secret-id "prod/myapp/database"

# Force delete secret (immediate)
aws secretsmanager delete-secret \
  --secret-id "prod/myapp/database" \
  --force-delete-without-recovery
```

### Automatic Rotation Configuration

```bash
# Configure automatic rotation for RDS
aws secretsmanager rotate-secret \
  --secret-id "prod/myapp/database" \
  --rotation-lambda-arn "arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRDSMySQLRotationSingleUser" \
  --rotation-rules '{
    "AutomaticallyAfterDays": 30
  }'

# Update rotation configuration
aws secretsmanager update-secret \
  --secret-id "prod/myapp/database" \
  --rotation-lambda-arn "arn:aws:lambda:us-east-1:123456789012:function:CustomRotationFunction" \
  --rotation-rules '{
    "AutomaticallyAfterDays": 60
  }'

# Start immediate rotation
aws secretsmanager rotate-secret \
  --secret-id "prod/myapp/database" \
  --force-rotate-immediately

# Stop automatic rotation
aws secretsmanager update-secret \
  --secret-id "prod/myapp/database" \
  --remove-rotation
```

### Cross-Region Replication

```bash
# Replicate secret to another region
aws secretsmanager replicate-secret-to-regions \
  --secret-id "prod/myapp/database" \
  --add-replica-regions Region=us-west-2,KmsKeyId=arn:aws:kms:us-west-2:123456789012:key/87654321-4321-4321-4321-210987654321

# Remove replica from region
aws secretsmanager remove-regions-from-replication \
  --secret-id "prod/myapp/database" \
  --remove-replica-regions us-west-2

# Stop replication
aws secretsmanager stop-replication-to-replica \
  --secret-id "prod/myapp/database" \
  --replica-region us-west-2
```

### Secret Versioning

```bash
# Create new version with staging label
aws secretsmanager put-secret-value \
  --secret-id "prod/myapp/database" \
  --secret-string '{
    "username": "admin",
    "password": "updated-password-789",
    "engine": "mysql",
    "host": "prod-db.cluster-xyz.us-east-1.rds.amazonaws.com",
    "port": 3306,
    "dbname": "myapp"
  }' \
  --version-stage "AWSPENDING"

# Move version to current
aws secretsmanager update-secret-version-stage \
  --secret-id "prod/myapp/database" \
  --version-stage "AWSCURRENT" \
  --move-to-version-id "12345678-1234-1234-1234-123456789012"

# Get specific version
aws secretsmanager get-secret-value \
  --secret-id "prod/myapp/database" \
  --version-id "12345678-1234-1234-1234-123456789012"

# List secret versions
aws secretsmanager list-secret-version-ids \
  --secret-id "prod/myapp/database"
```

### Resource-Based Policies

```bash
# Set resource policy for secret
aws secretsmanager put-resource-policy \
  --secret-id "prod/myapp/database" \
  --resource-policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::123456789013:role/AppRole"
        },
        "Action": "secretsmanager:GetSecretValue",
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "secretsmanager:resource/AllowRotationLambdaArn": "arn:aws:lambda:us-east-1:123456789012:function:CustomRotationFunction"
          }
        }
      }
    ]
  }'

# Get resource policy
aws secretsmanager get-resource-policy \
  --secret-id "prod/myapp/database"

# Delete resource policy
aws secretsmanager delete-resource-policy \
  --secret-id "prod/myapp/database"
```

### Application Integration Examples

#### Python SDK Example

```python
import boto3
import json
from botocore.exceptions import ClientError

def get_secret(secret_name, region_name="us-east-1"):
    """
    Retrieve secret from AWS Secrets Manager
    """
    session = boto3.session.Session()
    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )

    try:
        get_secret_value_response = client.get_secret_value(
            SecretId=secret_name
        )
    except ClientError as e:
        if e.response['Error']['Code'] == 'DecryptionFailureException':
            raise e
        elif e.response['Error']['Code'] == 'InternalServiceErrorException':
            raise e
        elif e.response['Error']['Code'] == 'InvalidParameterException':
            raise e
        elif e.response['Error']['Code'] == 'InvalidRequestException':
            raise e
        elif e.response['Error']['Code'] == 'ResourceNotFoundException':
            raise e
    else:
        secret = get_secret_value_response['SecretString']
        return json.loads(secret)

# Usage example
database_secret = get_secret("prod/myapp/database")
username = database_secret['username']
password = database_secret['password']
host = database_secret['host']
port = database_secret['port']
dbname = database_secret['dbname']

# Database connection
import pymysql
connection = pymysql.connect(
    host=host,
    user=username,
    password=password,
    database=dbname,
    port=port
)
```

#### Lambda Rotation Function Example

```python
import boto3
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """
    Custom rotation function for API keys
    """
    service = boto3.client('secretsmanager')

    # Extract rotation parameters
    secret_arn = event['Step1']['SecretArn']
    token = event['Step1']['ClientRequestToken']
    step = event['Step1']['Step']

    # Handle rotation steps
    if step == "createSecret":
        create_secret(service, secret_arn, token)
    elif step == "setSecret":
        set_secret(service, secret_arn, token)
    elif step == "testSecret":
        test_secret(service, secret_arn, token)
    elif step == "finishSecret":
        finish_secret(service, secret_arn, token)

    return {"statusCode": 200}

def create_secret(service, secret_arn, token):
    """Create new secret version"""
    try:
        # Get current secret
        current_secret = service.get_secret_value(
            SecretId=secret_arn,
            VersionStage="AWSCURRENT"
        )

        # Generate new API key (example)
        new_api_key = generate_new_api_key()

        # Create new version
        service.put_secret_value(
            SecretId=secret_arn,
            ClientRequestToken=token,
            SecretString=json.dumps({"api_key": new_api_key}),
            VersionStages=["AWSPENDING"]
        )

        logger.info("Created new secret version")

    except Exception as e:
        logger.error(f"Error creating secret: {str(e)}")
        raise e

def set_secret(service, secret_arn, token):
    """Configure new secret in external service"""
    # Implementation depends on external service API
    logger.info("Setting secret in external service")

def test_secret(service, secret_arn, token):
    """Test new secret functionality"""
    # Implementation depends on service requirements
    logger.info("Testing new secret")

def finish_secret(service, secret_arn, token):
    """Finalize rotation by updating version stages"""
    service.update_secret_version_stage(
        SecretId=secret_arn,
        VersionStage="AWSCURRENT",
        MoveToVersionId=token
    )
    logger.info("Finished secret rotation")

def generate_new_api_key():
    """Generate new API key"""
    # Implementation for API key generation
    import secrets
    return secrets.token_urlsafe(32)
```

#### ECS Task Definition with Secrets

```json
{
  "family": "myapp-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "myapp:latest",
      "secrets": [
        {
          "name": "DB_USERNAME",
          "valueFrom": "prod/myapp/database:username::"
        },
        {
          "name": "DB_PASSWORD",
          "valueFrom": "prod/myapp/database:password::"
        },
        {
          "name": "API_KEY",
          "valueFrom": "prod/myapp/api-keys:third_party_key::"
        }
      ],
      "environment": [
        {
          "name": "DB_HOST",
          "value": "prod-db.cluster-xyz.us-east-1.rds.amazonaws.com"
        }
      ]
    }
  ]
}
```

### Monitoring and Alerting

```bash
# Create CloudWatch alarm for rotation failures
aws cloudwatch put-metric-alarm \
  --alarm-name "SecretsManagerRotationFailures" \
  --alarm-description "Alert on secret rotation failures" \
  --metric-name "RotationFailed" \
  --namespace "AWS/SecretsManager" \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:secrets-alerts"

# Monitor secret access patterns
aws logs filter-log-events \
  --log-group-name "CloudTrail/SecretsManager" \
  --filter-pattern "{ $.eventName = GetSecretValue }" \
  --start-time $(date -d '1 hour ago' +%s)000

# Check secret usage statistics
aws cloudwatch get-metric-statistics \
  --namespace "AWS/SecretsManager" \
  --metric-name "SuccessfulRequestLatency" \
  --dimensions Name=SecretName,Value=prod/myapp/database \
  --statistics Average \
  --start-time $(date -d '24 hours ago' --iso-8601) \
  --end-time $(date --iso-8601) \
  --period 3600
```

This comprehensive Secrets Manager documentation provides detailed coverage of secret management capabilities essential for AWS certification preparation and real-world secure application development.