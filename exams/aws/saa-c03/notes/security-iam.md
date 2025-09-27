# AWS Security & IAM Fundamentals

## IAM (Identity and Access Management) Basics

### What is IAM?
Web service that helps you securely control access to AWS resources. IAM controls who can access what resources and what actions they can perform.

### Core IAM Components
- **Users**: Individual people or services that interact with AWS
- **Groups**: Collections of users with shared permissions
- **Roles**: Temporary credentials for AWS resources or external identities
- **Policies**: Documents that define permissions (JSON format)

### IAM Entities Hierarchy
```
AWS Account (Root)
├── Users (Individual identities)
├── Groups (Collections of users)
├── Roles (Assumable identities)
└── Policies (Permission documents)
```

## Users, Groups, and Roles

### IAM Users
**Individual identities for people or services**

#### User Types
- **Human users**: Actual people who need access
- **Service users**: Applications or services that need programmatic access
- **System users**: Automated processes or scripts

#### User Access Methods
| Method | Use Case | Security |
|--------|----------|----------|
| **AWS Console**: Web browser access | Human interactive use | Username/password + MFA |
| **Access Keys**: Programmatic access | CLI, SDK, API calls | Access Key ID + Secret |
| **Temporary credentials**: Short-term access | Cross-account, federated | STS tokens |

#### Best Practices for Users
- Create individual users (don't share accounts)
- Use groups to assign permissions
- Enable MFA for all users
- Rotate access keys regularly
- Use temporary credentials when possible

### IAM Groups
**Collections of users with common permission needs**

#### Group Examples
```
Developers Group:
- Read access to development resources
- Deploy permissions for dev environments
- CloudWatch logs access

Database Admins Group:
- Full access to RDS instances
- CloudWatch metrics for databases
- Backup and snapshot permissions

Security Team Group:
- CloudTrail access
- Security-related service permissions
- Read-only access to most resources
```

#### Group Limitations
- Users can belong to multiple groups (up to 10)
- Groups cannot be nested (no groups within groups)
- Groups cannot have groups as members

### IAM Roles
**Temporary credentials that can be assumed**

#### When to Use Roles
- **EC2 instances**: Avoid storing keys on instances
- **Cross-account access**: Access resources in other AWS accounts
- **Federated users**: External identities (SAML, OIDC)
- **AWS services**: Allow services to access other services

#### Role Components
- **Trust policy**: Who can assume the role
- **Permission policy**: What the role can do
- **Role session**: Temporary credentials when assumed

#### Common Role Patterns
```
EC2 Role:
Trust Policy: EC2 service can assume this role
Permission Policy: Access to S3, CloudWatch, etc.

Cross-Account Role:
Trust Policy: Specific external account can assume
Permission Policy: Limited permissions in this account

Lambda Execution Role:
Trust Policy: Lambda service can assume this role  
Permission Policy: Write to CloudWatch Logs, access other services
```

## IAM Policies Deep Dive

### Policy Types
| Type | Managed By | Use Case |
|------|------------|----------|
| **AWS Managed**: Amazon | Common use cases, maintained by AWS |
| **Customer Managed**: You | Custom permissions for your needs |
| **Inline**: You | Embedded directly in user/group/role |

### Policy Structure
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:ExistingObjectTag/Department": "Finance"
        }
      }
    }
  ]
}
```

### Policy Elements
| Element | Description | Example |
|---------|-------------|---------|
| **Version** | Policy language version | "2012-10-17" |
| **Statement** | Array of permission statements | [...] |
| **Effect** | Allow or Deny | "Allow", "Deny" |
| **Action** | What actions are allowed/denied | "s3:GetObject" |
| **Resource** | Which resources | "arn:aws:s3:::bucket/*" |
| **Condition** | When policy applies | IP ranges, time, tags |

### Common Policy Examples

#### S3 Read-Only Access
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

#### EC2 Instance Management
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "Development"
        }
      }
    }
  ]
}
```

## Multi-Factor Authentication (MFA)

### What is MFA?
Additional layer of security requiring users to provide two or more verification factors.

### MFA Device Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Virtual MFA**: Smartphone apps | Google Authenticator, Authy | Most common, convenient |
| **Hardware MFA**: Physical tokens | YubiKey, RSA SecurID | High security environments |
| **SMS MFA**: Text message codes | Phone-based | Not recommended (security concerns) |

### MFA Best Practices
- Enable MFA for all users (especially privileged accounts)
- Use virtual MFA for most users
- Hardware MFA for highly privileged accounts
- Never disable MFA for root account
- Have backup MFA devices configured

## AWS Security Services Overview

### AWS CloudTrail
**API logging and monitoring service**

#### What CloudTrail Logs
- **Management events**: Control plane operations (creating/deleting resources)
- **Data events**: Data plane operations (S3 object access, Lambda invocations)
- **Insight events**: Unusual activity patterns

#### Key Features
- **Event history**: 90 days of management events (free)
- **Trails**: Custom logging configuration
- **Log file integrity**: Detect tampering
- **Multi-region logging**: Capture events across regions
- **Real-time delivery**: CloudWatch Logs integration

### AWS Config
**Resource configuration monitoring and compliance**

#### What Config Tracks
- **Resource inventory**: What resources exist
- **Configuration history**: How resources change over time
- **Compliance rules**: Automated compliance checking
- **Remediation**: Automatic fix for non-compliant resources

#### Common Config Rules
- **S3 bucket public access**: Detect publicly accessible buckets
- **EC2 security groups**: Check for overly permissive rules
- **IAM password policy**: Ensure strong password requirements
- **EBS encryption**: Verify volumes are encrypted

### AWS GuardDuty
**Intelligent threat detection service**

#### Detection Capabilities
- **Malicious IP addresses**: Known bad actors
- **Cryptocurrency mining**: Unauthorized mining activity
- **Compromised instances**: Unusual behavior patterns
- **Data exfiltration**: Suspicious data transfers

#### Data Sources
- **VPC Flow Logs**: Network traffic analysis
- **DNS logs**: DNS query analysis
- **CloudTrail events**: API activity analysis
- **Threat intelligence**: AWS and partner feeds

### AWS Security Hub
**Centralized security findings management**

#### Key Features
- **Multi-service findings**: Aggregates from GuardDuty, Config, Inspector, etc.
- **Security standards**: CIS, AWS Foundational, PCI DSS compliance
- **Custom insights**: Create security dashboards
- **Integration**: Third-party security tools

### AWS Systems Manager
**Operational management for EC2 and on-premises**

#### Key Capabilities
- **Patch Manager**: Automated OS patching
- **Session Manager**: Secure shell access without SSH
- **Parameter Store**: Secure configuration storage
- **Run Command**: Execute commands on multiple instances
- **Compliance**: Track system compliance

## Encryption and Key Management

### Encryption at Rest
**Data encryption when stored**

#### AWS Services with Built-in Encryption
- **S3**: Server-side encryption (SSE)
- **EBS**: Volume encryption
- **RDS**: Database encryption
- **DynamoDB**: Table encryption
- **Redshift**: Cluster encryption

### Encryption in Transit
**Data encryption when moving**

#### Common Implementations
- **HTTPS/TLS**: Web traffic encryption
- **SSL**: Database connections
- **VPN**: Network traffic encryption
- **AWS PrivateLink**: Service-to-service encryption

### AWS Key Management Service (KMS)
**Managed encryption key service**

#### Key Types
| Type | Description | Use Case |
|------|-------------|----------|
| **AWS Managed**: AWS creates and manages | Default encryption | Most services |
| **Customer Managed**: You control key policy | Custom key policies | Compliance requirements |
| **AWS Owned**: AWS owns and manages | Service-specific | Transparent to customers |

#### KMS Operations
- **Encrypt/Decrypt**: Cryptographic operations
- **GenerateDataKey**: Create data encryption keys
- **Key rotation**: Automatic annual rotation
- **Cross-region replication**: Use keys across regions

### AWS Secrets Manager
**Secure secrets storage and rotation**

#### Supported Secret Types
- **Database credentials**: RDS, Aurora, Redshift
- **API keys**: Third-party service keys
- **OAuth tokens**: Authentication tokens
- **Custom secrets**: Any sensitive text/binary data

#### Key Features
- **Automatic rotation**: Periodic credential updates
- **Fine-grained access**: IAM-based permissions
- **Audit trail**: CloudTrail integration
- **Cross-region replication**: Disaster recovery

## Network Security

### VPC Security Features
- **Security Groups**: Instance-level firewall (stateful)
- **Network ACLs**: Subnet-level firewall (stateless)
- **Flow Logs**: Network traffic monitoring
- **VPC Endpoints**: Private service access

### AWS WAF (Web Application Firewall)
**Layer 7 web application protection**

#### Protection Against
- **SQL injection**: Database attack prevention
- **Cross-site scripting (XSS)**: Script injection prevention
- **DDoS attacks**: Application-layer attack mitigation
- **Bot traffic**: Automated traffic filtering

#### Rule Types
- **IP address rules**: Allow/block specific IPs
- **Geographic rules**: Block traffic from countries
- **Rate limiting**: Prevent excessive requests
- **Managed rules**: AWS and partner-maintained rules

### AWS Shield
**DDoS protection service**

#### Shield Standard (Free)
- **Network/transport layer**: Layer 3/4 protection
- **Always on**: Automatic protection
- **No configuration**: Enabled by default

#### Shield Advanced (Paid)
- **Application layer**: Layer 7 protection
- **24/7 DDoS Response Team**: Expert support
- **Cost protection**: DDoS-related charges covered
- **Advanced reporting**: Detailed attack information

## Security Best Practices

### Principle of Least Privilege
- Grant minimum permissions necessary
- Regular permission audits
- Use groups instead of individual user permissions
- Implement just-in-time access where possible

### Identity and Access Management
- Use roles instead of access keys when possible
- Enable MFA for all users
- Rotate credentials regularly
- Monitor credential usage

### Data Protection
- Encrypt sensitive data at rest and in transit
- Use KMS for key management
- Implement proper backup strategies
- Regular data classification reviews

### Network Security
- Use VPCs with proper subnet segmentation
- Implement security groups and NACLs
- Use VPC endpoints for AWS service access
- Enable VPC Flow Logs for monitoring

### Monitoring and Logging
- Enable CloudTrail in all regions
- Use Config for compliance monitoring
- Implement GuardDuty for threat detection
- Regular security assessments

## Compliance and Governance

### AWS Compliance Programs
- **SOC**: Service Organization Control reports
- **PCI DSS**: Payment Card Industry compliance
- **HIPAA**: Healthcare data protection
- **FedRAMP**: US government cloud security
- **GDPR**: European data protection regulation

### AWS Organizations
**Multi-account management service**

#### Key Features
- **Consolidated billing**: Single bill for multiple accounts
- **Service Control Policies**: Account-level permission boundaries
- **Organizational Units**: Hierarchical account grouping
- **Account creation**: Programmatic account creation

#### Common OU Structure
```
Root Organization
├── Production OU
│   ├── Prod Account 1
│   └── Prod Account 2
├── Development OU
│   ├── Dev Account 1
│   └── Dev Account 2
└── Security OU
    ├── Log Archive Account
    └── Audit Account
```

## Incident Response and Forensics

### CloudTrail for Forensics
- **Event investigation**: Who did what when
- **Data events**: File access, database queries
- **Management events**: Resource changes
- **Cross-service correlation**: Trace activity across services

### Config for Compliance
- **Resource timeline**: Track configuration changes
- **Compliance rules**: Automated checking
- **Remediation**: Automatic fixes
- **Drift detection**: Identify unauthorized changes

### GuardDuty for Threat Response
- **Threat detection**: Identify malicious activity
- **Severity levels**: Prioritize response efforts
- **Integration**: Security Hub and Lambda automation
- **Custom actions**: Automated response workflows

## Common Security Pitfalls

### IAM Misconfigurations
- Overly permissive policies (using * for actions/resources)
- Sharing access keys between users
- Not using MFA for privileged accounts
- Inline policies instead of managed policies

### Data Protection Issues
- Storing secrets in code or configuration files
- Not encrypting sensitive data
- Public S3 buckets with sensitive data
- Weak password policies

### Network Security Gaps
- Overly permissive security groups (0.0.0.0/0)
- Not using VPC endpoints for AWS services
- Missing network monitoring (Flow Logs)
- Inadequate network segmentation

### Monitoring Blind Spots
- Not enabling CloudTrail in all regions
- Insufficient log retention periods
- Not monitoring for unusual activity
- Missing automated alerting for security events