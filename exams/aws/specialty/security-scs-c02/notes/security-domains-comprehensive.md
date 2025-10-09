# AWS Security Specialty (SCS-C02) - Comprehensive Study Notes

## All Exam Domains - Essential Concepts

### Domain 1: Threat Detection and Incident Response (14%)

#### Amazon GuardDuty
- **Threat Intelligence**: Built-in threat intelligence from AWS, CrowdStrike, Proofpoint
- **Finding Types**: Recon, Instance compromise, Account compromise, Bucket compromise, Crypto mining
- **Severity Levels**: Low (0.1-3.9), Medium (4.0-6.9), High (7.0-8.9)
- **Multi-Account Setup**: Master account aggregates findings from member accounts
- **Suppression Rules**: Filter out expected behaviors and false positives
- **Automated Response**: EventBridge + Lambda for automated remediation

#### AWS Security Hub
- **Security Standards**: CIS AWS Foundations, PCI DSS, AWS Foundational Security Best Practices
- **Finding Aggregation**: Centralizes findings from GuardDuty, Inspector, Macie, IAM Access Analyzer
- **Compliance Scores**: Track compliance posture across standards
- **Custom Actions**: Automated workflows with EventBridge
- **Insights**: Custom grouping and filtering of findings

#### Incident Response Best Practices
- **Preparation**: Pre-deploy forensics tools, create playbooks, establish communication channels
- **Detection**: GuardDuty, CloudWatch, Security Hub, third-party SIEM
- **Analysis**: CloudWatch Logs Insights, Athena for log analysis, X-Ray for distributed tracing
- **Containment**: Isolate affected resources, revoke credentials, apply restrictive security groups
- **Eradication**: Remove malware, patch vulnerabilities, update compromised credentials
- **Recovery**: Restore from backups, verify system integrity, monitor for re-infection
- **Post-Incident**: Document lessons learned, update playbooks, improve detection

### Domain 2: Security Logging and Monitoring (18%)

#### AWS CloudTrail
```bash
# Create organization trail
aws cloudformation create-stack \
    --stack-name OrganizationTrail \
    --template-body file://org-trail.yaml \
    --parameters ParameterKey=S3BucketName,ParameterValue=org-cloudtrail-logs

# Enable log file validation
aws cloudtrail update-trail \
    --name MyTrail \
    --enable-log-file-validation

# CloudTrail Insights (anomaly detection)
aws cloudtrail put-insight-selectors \
    --trail-name MyTrail \
    --insight-selectors '[{"InsightType": "ApiCallRateInsight"}]'
```

**CloudTrail Best Practices**:
- Enable in all regions and all accounts
- Centralize logs in dedicated security account
- Enable log file validation for integrity
- Use CloudTrail Insights for anomaly detection
- Encrypt logs with KMS CMK
- Monitor with CloudWatch Logs and EventBridge

#### VPC Flow Logs
```python
import boto3

ec2 = boto3.client('ec2')

# Create VPC Flow Logs to CloudWatch
response = ec2.create_flow_logs(
    ResourceIds=['vpc-12345678'],
    ResourceType='VPC',
    TrafficType='ALL',  # ACCEPT, REJECT, or ALL
    LogDestinationType='cloud-watch-logs',
    LogGroupName='/aws/vpc/flowlogs',
    DeliverLogsPermissionArn='arn:aws:iam::123456789012:role/flowlogsRole'
)

# Analyze with Athena
# 1. Create table in Athena for VPC Flow Logs
# 2. Query rejected connections, top talkers, anomalous traffic
```

**VPC Flow Logs Analysis**:
- Detect port scanning and network reconnaissance
- Identify data exfiltration attempts
- Troubleshoot connectivity issues
- Meet compliance requirements

#### AWS Config
- **Configuration Items**: Track all resource configurations and changes
- **Config Rules**: Evaluate compliance (AWS managed and custom)
- **Conformance Packs**: Pre-packaged compliance frameworks
- **Remediation**: Automated fixes with SSM documents
- **Multi-Account Aggregation**: Organization-wide compliance view

### Domain 3: Infrastructure Security (20%)

#### VPC Security Architecture
```yaml
# Secure multi-tier VPC design
VPC:
  CIDR: 10.0.0.0/16
  Tiers:
    - Public Subnet (10.0.1.0/24):  # Load balancers
        - Internet Gateway
        - NAT Gateway
        - Security Groups: Allow 80/443 from internet

    - Private Subnet (10.0.10.0/24):  # Application
        - No internet access
        - Security Groups: Allow traffic from public subnet only
        - VPC Endpoints for AWS services

    - Isolated Subnet (10.0.20.0/24):  # Database
        - No route to NAT Gateway
        - Security Groups: Allow traffic from app subnet only
        - Backup via VPC Endpoint
```

#### AWS WAF Configuration
```json
{
  "Name": "WebACL",
  "Rules": [
    {
      "Name": "RateLimitRule",
      "Priority": 1,
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP"
        }
      },
      "Action": {"Block": {}}
    },
    {
      "Name": "GeoBlockRule",
      "Priority": 2,
      "Statement": {
        "GeoMatchStatement": {
          "CountryCodes": ["CN", "RU"]
        }
      },
      "Action": {"Block": {}}
    },
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 3,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      }
    }
  ]
}
```

#### AWS Shield
- **Standard**: Free DDoS protection for all AWS customers
- **Advanced**: $3,000/month with:
  - Enhanced detection and mitigation
  - 24/7 DDoS Response Team (DRT)
  - Cost protection (credits for scaling during attack)
  - Advanced metrics and reports

#### AWS Network Firewall
- **Stateful Inspection**: Deep packet inspection
- **Suricata Compatible**: Industry-standard IDS/IPS rules
- **Domain Filtering**: Block malicious domains
- **TLS Inspection**: Decrypt and inspect encrypted traffic
- **Centralized Management**: Multi-VPC and multi-account deployment

### Domain 4: Identity and Access Management (16%)

#### IAM Policy Evaluation Logic
```
1. Explicit DENY → Always wins
2. Organization SCP → Must allow
3. Resource-based policy → Can allow
4. Permission boundaries → Sets maximum permissions
5. Session policies → Further restricts permissions
6. Identity-based policy → Must allow
7. Implicit DENY → Default if no explicit allow
```

#### Advanced IAM Patterns
```json
// Permission Boundary (sets maximum permissions)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:*", "dynamodb:*", "lambda:*"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "us-west-2"]
        }
      }
    }
  ]
}

// Service Control Policy (org-wide restrictions)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["ec2:*"],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": ["t2.micro", "t2.small", "t3.micro", "t3.small"]
        }
      }
    }
  ]
}

// Attribute-Based Access Control (ABAC)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::${aws:PrincipalTag/Department}/*"
    }
  ]
}
```

#### AWS SSO (Identity Center)
- **Multi-Account Access**: Single sign-on across AWS accounts
- **Permission Sets**: Reusable permission templates
- **Identity Sources**: AWS SSO directory, Active Directory, External IdP (SAML 2.0)
- **Just-In-Time Provisioning**: Automatic user/group creation
- **Audit Logging**: CloudTrail logs all SSO activities

#### IAM Access Analyzer
- **External Access**: Identify resources shared outside organization
- **Unused Access**: Find unused permissions and credentials
- **Policy Generation**: Generate least-privilege policies from CloudTrail logs
- **Policy Validation**: Validate IAM policies before deployment

### Domain 5: Data Protection and Cryptography (18%)

#### AWS KMS Best Practices
```python
import boto3
import json

kms = boto3.client('kms')

# Create customer managed key
response = kms.create_key(
    Description='Application encryption key',
    KeyUsage='ENCRYPT_DECRYPT',
    Origin='AWS_KMS',
    MultiRegion=False,
    KeyPolicy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "Enable IAM policies",
                "Effect": "Allow",
                "Principal": {"AWS": "arn:aws:iam::123456789012:root"},
                "Action": "kms:*",
                "Resource": "*"
            },
            {
                "Sid": "Allow app role to encrypt/decrypt",
                "Effect": "Allow",
                "Principal": {"AWS": "arn:aws:iam::123456789012:role/AppRole"},
                "Action": ["kms:Encrypt", "kms:Decrypt", "kms:GenerateDataKey"],
                "Resource": "*"
            }
        ]
    })
)

# Enable automatic key rotation
kms.enable_key_rotation(KeyId=response['KeyMetadata']['KeyId'])

# Create grant for temporary access
grant = kms.create_grant(
    KeyId='key-id',
    GranteePrincipal='arn:aws:iam::123456789012:role/TemporaryRole',
    Operations=['Encrypt', 'Decrypt'],
    Constraints={
        'EncryptionContextSubset': {'Department': 'Finance'}
    }
)
```

#### Encryption Strategies
- **S3**: SSE-S3, SSE-KMS, SSE-C, client-side encryption
- **EBS**: Encrypted by default with KMS
- **RDS**: Encryption at creation, automated backups encrypted
- **DynamoDB**: Encryption at rest (AWS owned, AWS managed, or customer managed KMS)
- **In Transit**: TLS 1.2+ for all API calls, VPN for hybrid connectivity

#### AWS Secrets Manager
```python
import boto3
import json

secrets = boto3.client('secretsmanager')

# Create secret with rotation
response = secrets.create_secret(
    Name='prod/rds/credentials',
    SecretString=json.dumps({
        'username': 'admin',
        'password': 'InitialPassword123!'
    }),
    KmsKeyId='arn:aws:kms:us-east-1:123456789012:key/key-id'
)

# Configure automatic rotation
secrets.rotate_secret(
    SecretId='prod/rds/credentials',
    RotationLambdaARN='arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRDSRotation',
    RotationRules={'AutomaticallyAfterDays': 30}
)

# Retrieve secret in application
secret = secrets.get_secret_value(SecretId='prod/rds/credentials')
credentials = json.loads(secret['SecretString'])
```

#### Amazon Macie
- **Data Discovery**: Automatically discover S3 buckets and sensitive data
- **Sensitive Data Types**: PII, financial data, credentials, intellectual property
- **Custom Data Identifiers**: Regex-based custom patterns
- **Findings**: Alert on publicly accessible buckets, unencrypted buckets, sensitive data
- **Integration**: Security Hub, EventBridge for automated response

### Domain 6: Management and Security Governance (14%)

#### AWS Organizations Security
```python
import boto3

orgs = boto3.client('organizations')

# Create organizational structure
root_id = orgs.list_roots()['Roots'][0]['Id']

# Security OU
security_ou = orgs.create_organizational_unit(
    ParentId=root_id,
    Name='Security'
)

# Workloads OU
workloads_ou = orgs.create_organizational_unit(
    ParentId=root_id,
    Name='Workloads'
)

# Apply SCP to restrict regions
scp_policy = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Deny",
        "Action": "*",
        "Resource": "*",
        "Condition": {
            "StringNotEquals": {
                "aws:RequestedRegion": ["us-east-1", "us-west-2", "eu-west-1"]
            }
        }
    }]
}

policy_response = orgs.create_policy(
    Content=json.dumps(scp_policy),
    Description='Restrict to approved regions',
    Name='RegionRestriction',
    Type='SERVICE_CONTROL_POLICY'
)

# Attach SCP to OU
orgs.attach_policy(
    PolicyId=policy_response['Policy']['PolicySummary']['Id'],
    TargetId=workloads_ou['OrganizationalUnit']['Id']
)
```

#### AWS Control Tower
- **Landing Zone**: Multi-account AWS environment setup
- **Account Factory**: Automated account provisioning with guardrails
- **Guardrails**:
  - **Preventive**: SCPs that prevent actions
  - **Detective**: Config rules that detect non-compliance
- **Dashboard**: Centralized compliance and governance view
- **Drift Detection**: Detect manual changes to landing zone

#### Compliance Automation
```yaml
# AWS Config Conformance Pack (CIS AWS Foundations Benchmark)
Parameters:
  AccessKeysRotatedParameterMaxAccessKeyAge:
    Default: '90'
Resources:
  IamPasswordPolicy:
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

  CloudTrailEnabled:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: cloud-trail-enabled
      Source:
        Owner: AWS
        SourceIdentifier: CLOUD_TRAIL_ENABLED
```

## Common Exam Scenarios

### Scenario 1: Automated Incident Response
**Question**: Design automated response to GuardDuty finding indicating compromised EC2 instance.

**Solution**:
1. GuardDuty generates finding
2. EventBridge rule triggers Lambda
3. Lambda performs:
   - Isolate instance (restrictive security group)
   - Create EBS snapshot for forensics
   - Terminate instance
   - Revoke IAM credentials
   - Send SNS notification to security team
   - Create Security Hub finding

### Scenario 2: Multi-Account Security Logging
**Question**: Design centralized logging for 50+ AWS accounts.

**Solution**:
1. Dedicated security account for log aggregation
2. Organization-wide CloudTrail to central S3 bucket
3. VPC Flow Logs from all accounts to CloudWatch Logs with cross-account subscription
4. GuardDuty master-member setup
5. Security Hub aggregation
6. Athena/OpenSearch for log analysis
7. S3 Object Lock for log retention compliance

### Scenario 3: Data Encryption at Rest
**Question**: Implement encryption for multi-tier application with compliance requirements.

**Solution**:
1. KMS customer managed key per environment
2. Enable automatic key rotation
3. S3 buckets: SSE-KMS with bucket key
4. RDS: Enable encryption at creation with KMS CMK
5. EBS volumes: Encrypted by default policy
6. DynamoDB: Customer managed KMS key
7. CloudTrail logs all KMS API calls
8. Key policies enforce least privilege

## Study Tips

1. **Hands-On Practice**: Build multi-account security architecture in your own AWS account
2. **Security Hub Focus**: Understand all supported standards and integration patterns
3. **IAM Policy Mastery**: Practice policy evaluation logic with IAM Policy Simulator
4. **Incident Response**: Create and test incident response playbooks
5. **Encryption Everywhere**: Know encryption options for every AWS service
6. **Compliance Frameworks**: Familiarize with CIS, PCI DSS, HIPAA requirements
7. **Network Security**: Design secure VPC architectures with multiple layers

## CLI Quick Reference

```bash
# GuardDuty
aws guardduty create-detector --enable
aws guardduty list-findings --detector-id abc123

# Security Hub
aws securityhub enable-security-hub
aws securityhub get-findings --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}'

# CloudTrail
aws cloudtrail create-trail --name OrgTrail --s3-bucket-name trail-bucket --is-organization-trail
aws cloudtrail start-logging --name OrgTrail

# Config
aws configservice put-config-rule --config-rule file://rule.json
aws configservice describe-compliance-by-config-rule

# KMS
aws kms create-key --description "App encryption key"
aws kms enable-key-rotation --key-id key-id
aws kms encrypt --key-id alias/my-key --plaintext "secret"

# Organizations
aws organizations create-policy --type SERVICE_CONTROL_POLICY --name RestrictRegions --content file://scp.json
aws organizations attach-policy --policy-id p-123 --target-id ou-456
```
