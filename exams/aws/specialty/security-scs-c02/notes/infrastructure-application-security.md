# Infrastructure and Application Security

## VPC Security Architecture

### Defense in Depth

**Layered Security Model**
```
Internet
    │
    ▼
┌─────────────────────────────┐
│  AWS Shield / WAF / Route 53│  Layer 1: Edge Protection
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  CloudFront + WAF           │  Layer 2: Content Delivery
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  ALB + Security Groups      │  Layer 3: Load Balancer
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  Network ACLs               │  Layer 4: Subnet Level
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  Security Groups            │  Layer 5: Instance Level
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  OS Firewall + Host IDS     │  Layer 6: Operating System
└─────────────────────────────┘
    │
    ▼
┌─────────────────────────────┐
│  Application Security       │  Layer 7: Application
└─────────────────────────────┘
```

### Secure VPC Design

**Multi-Tier Architecture**
```yaml
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: Secure-Production-VPC

  # Public Subnet - DMZ
  PublicSubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: false  # Explicit elastic IPs only
      Tags:
        - Key: Name
          Value: Public-DMZ-A

  # Private Subnet - Application
  PrivateAppSubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.10.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: Private-App-A

  # Isolated Subnet - Database
  IsolatedDBSubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.20.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: Isolated-DB-A

  # VPC Flow Logs
  FlowLogRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: vpc-flow-logs.amazonaws.com
            Action: 'sts:AssumeRole'
      Policies:
        - PolicyName: CloudWatchLogPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 'logs:CreateLogGroup'
                  - 'logs:CreateLogStream'
                  - 'logs:PutLogEvents'
                Resource: '*'

  VPCFlowLog:
    Type: AWS::EC2::FlowLog
    Properties:
      ResourceType: VPC
      ResourceIds:
        - !Ref VPC
      TrafficType: ALL
      LogDestinationType: cloud-watch-logs
      LogGroupName: /aws/vpc/flowlogs/production
      DeliverLogsPermissionArn: !GetAtt FlowLogRole.Arn

  # VPC Endpoints for Private AWS Service Access
  S3Endpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcId: !Ref VPC
      ServiceName: !Sub 'com.amazonaws.${AWS::Region}.s3'
      RouteTableIds:
        - !Ref PrivateRouteTable
        - !Ref IsolatedRouteTable

  DynamoDBEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcId: !Ref VPC
      ServiceName: !Sub 'com.amazonaws.${AWS::Region}.dynamodb'
      RouteTableIds:
        - !Ref PrivateRouteTable
```

### Network Segmentation

**Security Groups - Layered Approach**
```yaml
Resources:
  # ALB Security Group
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: ALB security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
          Description: HTTPS from internet
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP from internet (redirect to HTTPS)
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          DestinationSecurityGroupId: !Ref AppSecurityGroup
          Description: To application servers

  # Application Server Security Group
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Application server security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          SourceSecurityGroupId: !Ref ALBSecurityGroup
          Description: From ALB only
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref BastionSecurityGroup
          Description: SSH from bastion
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          DestinationSecurityGroupId: !Ref DBSecurityGroup
          Description: To database
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
          Description: HTTPS for AWS API calls

  # Database Security Group
  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Database security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup
          Description: MySQL from app servers only
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 127.0.0.1/32
          Description: Deny all outbound

  # Bastion Security Group
  BastionSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Bastion host security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 203.0.113.0/24  # Corporate IP range
          Description: SSH from corporate network
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 10.0.0.0/16
          Description: SSH to internal instances
```

## AWS WAF Configuration

### Web ACL Rules

**Comprehensive Protection**
```json
{
  "Name": "ProductionWebACL",
  "Scope": "REGIONAL",
  "DefaultAction": {
    "Allow": {}
  },
  "Rules": [
    {
      "Name": "BlockKnownBadIPs",
      "Priority": 0,
      "Statement": {
        "IPSetReferenceStatement": {
          "Arn": "arn:aws:wafv2:region:account:regional/ipset/bad-ips/a1b2c3d4"
        }
      },
      "Action": {
        "Block": {
          "CustomResponse": {
            "ResponseCode": 403
          }
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "BlockBadIPs"
      }
    },
    {
      "Name": "RateLimitPerIP",
      "Priority": 1,
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP",
          "ScopeDownStatement": {
            "NotStatement": {
              "Statement": {
                "ByteMatchStatement": {
                  "SearchString": "/healthcheck",
                  "FieldToMatch": {
                    "UriPath": {}
                  },
                  "TextTransformations": [
                    {
                      "Priority": 0,
                      "Type": "LOWERCASE"
                    }
                  ],
                  "PositionalConstraint": "STARTS_WITH"
                }
              }
            }
          }
        }
      },
      "Action": {
        "Block": {
          "CustomResponse": {
            "ResponseCode": 429,
            "CustomResponseBodyKey": "rate-limit-response"
          }
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "RateLimit"
      }
    },
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 2,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet",
          "ExcludedRules": []
        }
      },
      "OverrideAction": {
        "None": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CommonRuleSet"
      }
    },
    {
      "Name": "SQLInjectionProtection",
      "Priority": 3,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesSQLiRuleSet"
        }
      },
      "OverrideAction": {
        "None": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "SQLiProtection"
      }
    },
    {
      "Name": "BlockSpecificUserAgents",
      "Priority": 4,
      "Statement": {
        "RegexMatchStatement": {
          "RegexString": ".*(bot|crawler|scraper|scanner).*",
          "FieldToMatch": {
            "SingleHeader": {
              "Name": "user-agent"
            }
          },
          "TextTransformations": [
            {
              "Priority": 0,
              "Type": "LOWERCASE"
            }
          ]
        }
      },
      "Action": {
        "Block": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "BlockBots"
      }
    },
    {
      "Name": "GeoBlocking",
      "Priority": 5,
      "Statement": {
        "GeoMatchStatement": {
          "CountryCodes": ["CN", "RU", "KP", "IR"]
        }
      },
      "Action": {
        "Block": {}
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "GeoBlocking"
      }
    }
  ],
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "ProductionWebACL"
  }
}
```

### WAF Logging and Monitoring

**Enable Logging**
```bash
# Create Kinesis Firehose for WAF logs
aws firehose create-delivery-stream \
  --delivery-stream-name aws-waf-logs-production \
  --s3-destination-configuration \
    RoleARN=arn:aws:iam::account:role/FirehoseRole,\
    BucketARN=arn:aws:s3:::waf-logs-bucket,\
    Prefix=waflogs/,\
    CompressionFormat=GZIP

# Enable WAF logging
aws wafv2 put-logging-configuration \
  --logging-configuration \
    ResourceArn=arn:aws:wafv2:region:account:regional/webacl/ProductionWebACL/id,\
    LogDestinationConfigs=arn:aws:firehose:region:account:deliverystream/aws-waf-logs-production
```

## AWS Shield Protection

### Shield Advanced

**Enable Advanced Protection**
```bash
# Subscribe to Shield Advanced
aws shield subscribe-to-shield-advanced

# Protect resources
aws shield create-protection \
  --name ALB-Protection \
  --resource-arn arn:aws:elasticloadbalancing:region:account:loadbalancer/app/my-alb/id

aws shield create-protection \
  --name CloudFront-Protection \
  --resource-arn arn:aws:cloudfront::account:distribution/DISTRIBUTION_ID
```

**DDoS Response Team Access**
```yaml
Resources:
  DRTAccessRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: drt.shield.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSShieldDRTAccessPolicy

  DRTAccess:
    Type: AWS::Shield::DRTAccess
    Properties:
      RoleArn: !GetAtt DRTAccessRole.Arn
      LogBucketList:
        - !Ref WAFLogsBucket
        - !Ref CloudFrontLogsBucket
```

## Application Security

### Secrets Management

**AWS Secrets Manager**
```python
import boto3
import json

secrets = boto3.client('secretsmanager')

# Create secret
response = secrets.create_secret(
    Name='prod/database/credentials',
    Description='Production database credentials',
    SecretString=json.dumps({
        'username': 'admin',
        'password': 'SecurePassword123!',
        'engine': 'mysql',
        'host': 'db.example.com',
        'port': 3306,
        'dbname': 'production'
    }),
    KmsKeyId='arn:aws:kms:region:account:key/key-id',
    Tags=[
        {'Key': 'Environment', 'Value': 'Production'},
        {'Key': 'Application', 'Value': 'WebApp'}
    ]
)

# Enable automatic rotation
secrets.rotate_secret(
    SecretId='prod/database/credentials',
    RotationLambdaARN='arn:aws:lambda:region:account:function:SecretsManagerRotation',
    RotationRules={
        'AutomaticallyAfterDays': 30
    }
)

# Retrieve secret in application
def get_database_credentials():
    secret = secrets.get_secret_value(SecretId='prod/database/credentials')
    credentials = json.loads(secret['SecretString'])
    return credentials

# Use credentials
creds = get_database_credentials()
connection_string = f"mysql://{creds['username']}:{creds['password']}@{creds['host']}:{creds['port']}/{creds['dbname']}"
```

**Secret Rotation Lambda**
```python
import boto3
import pymysql

def lambda_handler(event, context):
    token = event['Token']
    step = event['Step']
    secret_id = event['SecretId']
    
    secrets = boto3.client('secretsmanager')
    
    if step == "createSecret":
        # Generate new password
        new_password = generate_secure_password()
        
        # Create new version
        secrets.put_secret_value(
            SecretId=secret_id,
            SecretString=json.dumps({'password': new_password}),
            VersionStages=['AWSPENDING'],
            ClientRequestToken=token
        )
    
    elif step == "setSecret":
        # Update database with new password
        current = secrets.get_secret_value(SecretId=secret_id, VersionStage='AWSCURRENT')
        pending = secrets.get_secret_value(SecretId=secret_id, VersionStage='AWSPENDING')
        
        current_creds = json.loads(current['SecretString'])
        new_password = json.loads(pending['SecretString'])['password']
        
        # Connect and update password
        conn = pymysql.connect(
            host=current_creds['host'],
            user=current_creds['username'],
            password=current_creds['password']
        )
        cursor = conn.cursor()
        cursor.execute(f"ALTER USER '{current_creds['username']}' IDENTIFIED BY '{new_password}'")
        conn.commit()
        conn.close()
    
    elif step == "testSecret":
        # Test new credentials
        pending = secrets.get_secret_value(SecretId=secret_id, VersionStage='AWSPENDING')
        creds = json.loads(pending['SecretString'])
        
        # Test connection
        conn = pymysql.connect(
            host=creds['host'],
            user=creds['username'],
            password=creds['password']
        )
        conn.close()
    
    elif step == "finishSecret":
        # Finalize rotation
        secrets.update_secret_version_stage(
            SecretId=secret_id,
            VersionStage='AWSCURRENT',
            MoveToVersionId=token,
            RemoveFromVersionId=current_version
        )
```

### Parameter Store for Configuration

**Hierarchical Parameters**
```bash
# Store parameters
aws ssm put-parameter \
  --name /app/prod/database/host \
  --value "db.example.com" \
  --type String \
  --tier Standard

aws ssm put-parameter \
  --name /app/prod/database/password \
  --value "SecurePassword123!" \
  --type SecureString \
  --key-id alias/aws/ssm \
  --tier Standard

aws ssm put-parameter \
  --name /app/prod/api/key \
  --value "api-key-12345" \
  --type SecureString

# Get parameters by path
aws ssm get-parameters-by-path \
  --path /app/prod/ \
  --recursive \
  --with-decryption
```

**Application Integration**
```python
import boto3

ssm = boto3.client('ssm')

def get_config(environment='prod'):
    """Get all configuration parameters"""
    path = f'/app/{environment}/'
    
    response = ssm.get_parameters_by_path(
        Path=path,
        Recursive=True,
        WithDecryption=True
    )
    
    config = {}
    for param in response['Parameters']:
        key = param['Name'].replace(path, '')
        config[key] = param['Value']
    
    return config

# Use in application
config = get_config('prod')
db_host = config['database/host']
db_password = config['database/password']
api_key = config['api/key']
```

## Encryption

### Encryption at Rest

**S3 Encryption**
```yaml
Resources:
  EncryptedBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: secure-data-bucket
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
              KMSMasterKeyID: !GetAtt KMSKey.Arn
            BucketKeyEnabled: true
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      VersioningConfiguration:
        Status: Enabled
      LoggingConfiguration:
        DestinationBucketName: !Ref LoggingBucket
        LogFilePrefix: s3-access-logs/

  KMSKey:
    Type: AWS::KMS::Key
    Properties:
      Description: S3 bucket encryption key
      EnableKeyRotation: true
      KeyPolicy:
        Version: '2012-10-17'
        Statement:
          - Sid: Enable IAM policies
            Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${AWS::AccountId}:root'
            Action: 'kms:*'
            Resource: '*'
          - Sid: Allow S3 to use key
            Effect: Allow
            Principal:
              Service: s3.amazonaws.com
            Action:
              - 'kms:Decrypt'
              - 'kms:GenerateDataKey'
            Resource: '*'
            Condition:
              StringEquals:
                'kms:ViaService': !Sub 's3.${AWS::Region}.amazonaws.com'
```

**EBS Encryption**
```bash
# Enable encryption by default
aws ec2 enable-ebs-encryption-by-default --region us-east-1

# Specify KMS key for default encryption
aws ec2 modify-ebs-default-kms-key-id \
  --kms-key-id arn:aws:kms:region:account:key/key-id \
  --region us-east-1
```

**RDS Encryption**
```yaml
Resources:
  DBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: encrypted-database
      Engine: mysql
      EngineVersion: '8.0'
      DBInstanceClass: db.t3.medium
      AllocatedStorage: 100
      StorageEncrypted: true
      KmsKeyId: !GetAtt DBEncryptionKey.Arn
      MasterUsername: admin
      MasterUserPassword: !Sub '{{resolve:secretsmanager:${DBSecret}::password}}'
      BackupRetentionPeriod: 30
      PreferredBackupWindow: '03:00-04:00'
      EnableCloudwatchLogsExports:
        - error
        - general
        - slowquery
      DeletionProtection: true
```

### Encryption in Transit

**ALB with SSL/TLS**
```yaml
Resources:
  Certificate:
    Type: AWS::CertificateManager::Certificate
    Properties:
      DomainName: example.com
      SubjectAlternativeNames:
        - www.example.com
        - '*.example.com'
      ValidationMethod: DNS
      DomainValidationOptions:
        - DomainName: example.com
          HostedZoneId: !Ref HostedZone

  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref LoadBalancer
      Port: 443
      Protocol: HTTPS
      Certificates:
        - CertificateArn: !Ref Certificate
      SslPolicy: ELBSecurityPolicy-TLS-1-2-2017-01
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref TargetGroup

  # Redirect HTTP to HTTPS
  HTTPListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref LoadBalancer
      Port: 80
      Protocol: HTTP
      DefaultActions:
        - Type: redirect
          RedirectConfig:
            Protocol: HTTPS
            Port: '443'
            StatusCode: HTTP_301
```

## Compliance and Auditing

### AWS Config Rules

**Security Compliance Rules**
```yaml
Resources:
  S3PublicReadProhibited:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: s3-bucket-public-read-prohibited
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_PUBLIC_READ_PROHIBITED

  S3PublicWriteProhibited:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: s3-bucket-public-write-prohibited
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_PUBLIC_WRITE_PROHIBITED

  EBSEncryption:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: ec2-ebs-encryption-by-default
      Source:
        Owner: AWS
        SourceIdentifier: EC2_EBS_ENCRYPTION_BY_DEFAULT

  RootMFAEnabled:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: root-account-mfa-enabled
      Source:
        Owner: AWS
        SourceIdentifier: ROOT_ACCOUNT_MFA_ENABLED
      MaximumExecutionFrequency: TwentyFour_Hours

  IAMPasswordPolicy:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: iam-password-policy
      Source:
        Owner: AWS
        SourceIdentifier: IAM_PASSWORD_POLICY
      InputParameters:
        RequireUppercaseCharacters: true
        RequireLowercaseCharacters: true
        RequireNumbers: true
        MinimumPasswordLength: 14
        PasswordReusePrevention: 24
        MaxPasswordAge: 90
```

### Automated Remediation

**Auto-Remediation Actions**
```yaml
Resources:
  RemediationConfiguration:
    Type: AWS::Config::RemediationConfiguration
    Properties:
      ConfigRuleName: !Ref S3PublicReadProhibited
      TargetType: SSM_DOCUMENT
      TargetIdentifier: AWS-PublishSNSNotification
      TargetVersion: '1'
      Parameters:
        AutomationAssumeRole:
          StaticValue:
            Values:
              - !GetAtt RemediationRole.Arn
        TopicArn:
          StaticValue:
            Values:
              - !Ref SecurityAlertTopic
        Message:
          StaticValue:
            Values:
              - 'S3 bucket with public read access detected'
      Automatic: true
      MaximumAutomaticAttempts: 3
      RetryAttemptSeconds: 60
```

## Exam Tips

### Infrastructure Security
- VPC: Multi-layer defense with security groups and NACLs
- Security groups: Stateful, allow rules only
- NACLs: Stateless, allow and deny rules
- VPC Flow Logs: Monitor network traffic
- VPC Endpoints: Private AWS service access

### Web Application Protection
- WAF: Layer 7 protection (SQL injection, XSS, rate limiting)
- Shield Standard: Free DDoS protection for all
- Shield Advanced: Enhanced DDoS, 24/7 DRT, cost protection
- CloudFront: CDN with edge protection

### Application Security
- Secrets Manager: Automatic rotation, versioning
- Parameter Store: Configuration management, hierarchical
- KMS: Encryption key management, automatic rotation
- ACM: SSL/TLS certificate management

### Encryption
- At rest: S3 (SSE-KMS), EBS, RDS, DynamoDB
- In transit: HTTPS, TLS 1.2+, VPN
- KMS: Customer managed keys preferred for control
- CloudHSM: FIPS 140-2 Level 3 compliance

### Compliance
- Config Rules: Automated compliance checking
- Config Remediation: Automatic fixes
- CloudTrail: API audit logging
- GuardDuty: Threat detection
- Security Hub: Centralized security findings

### Common Scenarios
- **Protect web app**: CloudFront + WAF + Shield + ALB
- **Secure database**: Private subnet + security group + encryption + Secrets Manager
- **Compliance**: Config rules + automated remediation + CloudTrail
- **Secrets rotation**: Secrets Manager with Lambda rotation
- **Network isolation**: VPC + private subnets + VPC endpoints
