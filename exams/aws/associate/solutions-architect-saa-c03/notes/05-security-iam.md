# AWS Security and IAM - SAA-C03

## IAM (Identity and Access Management)

**[📖 AWS IAM Documentation](https://docs.aws.amazon.com/iam/)** - Securely control access to AWS services and resources

### Core Components

**[📖 IAM Identities](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html)** - Users, groups, and roles in IAM

**Users**: Individual people or applications
- Long-term credentials (password, access keys)
- Programmatic access (access key ID + secret key)
- Console access (username + password)
- MFA enforcement

**Groups**: Collection of users
- Assign permissions to multiple users
- Users can belong to multiple groups
- No default group

**Roles**: Temporary credentials for AWS services or federated users
- No long-term credentials
- Assumed by entities (EC2, Lambda, users)
- Cross-account access
- Federation (SAML, OIDC)

**Policies**: JSON documents defining permissions
- Identity-based (attached to users, groups, roles)
- Resource-based (attached to resources like S3 buckets)
- AWS managed, customer managed, inline

### Policy Structure

**[📖 IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)** - Control access with policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow" | "Deny",
      "Action": ["service:action"],
      "Resource": ["arn"],
      "Condition": {
        "condition-operator": {"condition-key": "value"}
      }
    }
  ]
}
```

### Policy Evaluation Logic
1. **Explicit Deny**: Always wins
2. **Explicit Allow**: Required if no deny
3. **Default Deny**: Implicit deny if no allow

**Evaluation Order**:
1. Organization SCPs (deny override)
2. Resource-based policies
3. IAM permission boundaries
4. Session policies
5. Identity-based policies

### IAM Best Practices
1. Root account: Use only for account setup, enable MFA
2. Least privilege: Grant minimum permissions
3. One IAM user per person: No sharing credentials
4. Strong password policy: Length, complexity, rotation
5. Enable MFA for privileged users
6. Use roles for applications on EC2
7. Rotate credentials regularly
8. Remove unnecessary users and credentials
9. Use policy conditions for extra security
10. Monitor IAM activity with CloudTrail

### IAM Access Analyzer
- Identify resources shared with external entities
- Validates IAM policies
- Generates least-privilege policies
- Use Case: Security audits, compliance

## AWS Organizations

**[📖 AWS Organizations Documentation](https://docs.aws.amazon.com/organizations/)** - Centrally manage multiple AWS accounts

### Features
- **Consolidated Billing**: Single payment for all accounts
- **Volume Discounts**: Aggregated usage
- **OU (Organizational Units)**: Hierarchical account grouping
- **Service Control Policies (SCPs)**: Permission boundaries

**[📖 Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)** - Set permission guardrails across accounts

### Service Control Policies (SCPs)
- **Allowlist**: Explicitly allow services (default deny all)
- **Denylist**: Explicitly deny services (default allow all)
- Applied to OUs or accounts
- Does NOT apply to management account
- Limit permissions, does not grant

**SCP Example** (Deny S3):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "s3:*",
    "Resource": "*"
  }]
}
```

### Best Practices
- Separate accounts for dev, test, prod
- Centralize logging account
- Security account for GuardDuty, Security Hub
- Restrict root account access with SCPs
- Tag resources for cost allocation

## AWS IAM Identity Center (formerly SSO)

### Features
- Single sign-on to multiple AWS accounts
- Integration with Active Directory
- SAML 2.0 support
- Built-in identity store
- Application integration (Office 365, Salesforce)
- Permission sets: Collections of policies

### Use Cases
- Multi-account access management
- Workforce identity federation
- Centralized access control
- Temporary credentials for users

## Encryption

**[📖 Encryption in AWS](https://docs.aws.amazon.com/whitepapers/latest/introduction-aws-security/encryption-in-aws.html)** - Data protection at rest and in transit

### Encryption at Rest

**S3 Encryption**:
- **SSE-S3**: S3-managed keys (AES-256)
- **SSE-KMS**: KMS-managed keys, audit trail
- **SSE-C**: Customer-provided keys
- **Client-side**: Encrypt before upload

**EBS Encryption**:
- KMS keys (CMK)
- Encrypted snapshots
- Enabled by default option
- Cannot change encryption state (must copy)

**RDS Encryption**:
- KMS encryption
- Includes backups, snapshots, read replicas
- Cannot enable after creation (must snapshot + restore)

### Encryption in Transit
- **TLS/SSL**: HTTPS for web traffic
- **VPN**: IPsec tunnels
- **Direct Connect**: MACsec (layer 2) or VPN over DX

### AWS KMS (Key Management Service)

**[📖 AWS KMS Documentation](https://docs.aws.amazon.com/kms/)** - Managed encryption key creation and control

**Customer Master Keys (CMKs)**:
- **AWS Managed**: Free, rotated automatically every year
- **Customer Managed**: $1/month, manual rotation
- **Symmetric (AES-256)**: Single key for encrypt/decrypt
- **Asymmetric (RSA, ECC)**: Public/private key pair

**Key Policies**:
- Primary resource-based policy
- Required for all CMKs
- Grants and IAM policies for additional access

**Envelope Encryption**:
- Encrypt data with data key
- Encrypt data key with CMK
- Performance: Large data encrypted locally
- `GenerateDataKey` API

**KMS Limits**:
- API call quotas (5,500-30,000 req/sec depending on region)
- Use envelope encryption or DEK caching to reduce calls

### AWS CloudHSM
- **Hardware Security Module**: FIPS 140-2 Level 3
- Single-tenant hardware
- Customer manages keys
- Use Cases: Regulatory compliance, contractual requirements

## AWS Certificate Manager (ACM)

### Features
- Provision, manage, deploy SSL/TLS certificates
- **Free** for public certificates
- Auto-renewal for ACM-issued certificates
- Integration: ALB, NLB, CloudFront, API Gateway

### Use Cases
- HTTPS endpoints
- End-to-end encryption
- Compliance requirements

### ACM vs IAM Certificates
- **ACM**: Integrated, auto-renewal, free
- **IAM**: Upload external certificates, manual renewal

## AWS Secrets Manager

**[📖 AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)** - Rotate, manage, and retrieve secrets throughout their lifecycle

### Features
- Store secrets (passwords, API keys, database credentials)
- Automatic rotation (Lambda integration)
- Encryption with KMS
- Fine-grained access control
- Audit with CloudTrail

### Secrets Manager vs Systems Manager Parameter Store
| Feature | Secrets Manager | Parameter Store |
|---------|----------------|----------------|
| Rotation | Automatic with Lambda | Manual |
| Integration | RDS auto-rotation | No auto-rotation |
| Cost | $0.40/secret/month | Free (standard), $0.05/param (advanced) |
| Secrets | Optimized for | General parameters |
| Cross-region | Replication | No native support |

## AWS Security Services

**[📖 AWS Security Services](https://docs.aws.amazon.com/security/)** - Comprehensive security solutions

### Amazon GuardDuty
- Intelligent threat detection
- Analyzes VPC Flow Logs, DNS logs, CloudTrail events
- Machine learning for anomaly detection
- CryptoCurrency mining detection
- Use Case: Continuous security monitoring

**[📖 Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/)** - Intelligent threat detection service

### AWS Security Hub
- Central security dashboard
- Aggregates findings from GuardDuty, Inspector, Macie, etc.
- Automated compliance checks (CIS, PCI DSS)
- Integration with EventBridge for automation

### Amazon Inspector
- Automated vulnerability assessment
- **EC2 Instances**: CVE vulnerabilities, network reachability
- **Container Images**: ECR image scanning
- **Lambda Functions**: Code vulnerabilities
- Risk scoring and remediation guidance

### Amazon Macie
- Data security and privacy service
- Discover and protect sensitive data (PII)
- S3 bucket analysis
- Machine learning and pattern matching
- Use Case: GDPR, HIPAA compliance

### AWS WAF (Web Application Firewall)
- Protect against web exploits
- Deploy on ALB, API Gateway, CloudFront, AppSync
- Rules: IP addresses, HTTP headers, body, URI strings
- Rate-based rules: DDoS protection
- Managed Rules: AWS and third-party rulesets

**[📖 AWS WAF](https://docs.aws.amazon.com/waf/)** - Protect web applications from common exploits

### AWS Shield
- **Shield Standard**: Free, DDoS protection for all AWS customers
- **Shield Advanced**: $3,000/month, enhanced protection
  - 24/7 DDoS response team
  - DDoS cost protection
  - Advanced real-time metrics
  - Integration with WAF (free)

### AWS Firewall Manager
- Centrally configure WAF rules across accounts
- Organization-wide policies
- Automatically apply to new resources
- Use Case: Multi-account security management

## Access Management

### IAM Roles for EC2
- Attach roles to EC2 instances
- Temporary credentials via instance metadata
- Automatic credential rotation
- No hardcoded credentials
- **Best Practice**: Always use roles over access keys

### IAM Roles for Lambda
- Execution role for Lambda permissions
- Resource-based policy for who can invoke
- Cross-account access

### Cross-Account Access
1. Create role in target account (trust policy)
2. Grant permissions in role
3. Assume role from source account (sts:AssumeRole)
4. Use temporary credentials

### IAM Permission Boundaries
- Set maximum permissions for IAM entity
- Does not grant permissions, only limits
- Use Case: Developers creating roles (prevent privilege escalation)

## Compliance and Governance

### AWS Config
- Track resource configuration changes
- Compliance auditing
- Config Rules: Evaluate compliance
- Remediation: Automatic or manual
- SNS notifications for changes

### AWS Audit Manager
- Continuous auditing and compliance
- Pre-built frameworks (GDPR, HIPAA, SOC 2)
- Evidence collection
- Audit reports

### AWS Artifact
- On-demand access to compliance reports
- ISO certifications, PCI, SOC reports
- BAA (Business Associate Agreement) for HIPAA

## Exam Tips

### Common Scenarios
- **Multi-account management**: AWS Organizations + SCPs
- **Federated access**: IAM Identity Center
- **Temporary credentials**: IAM Roles (never access keys for EC2)
- **Encrypt S3 with audit trail**: SSE-KMS
- **DDoS protection**: Shield Standard (free) or Shield Advanced
- **Web application protection**: AWS WAF
- **Detect compromised credentials**: GuardDuty
- **Centralized security findings**: Security Hub
- **Rotate RDS credentials automatically**: Secrets Manager
- **Compliance automation**: AWS Config

### Security Best Practices
1. Enable MFA for root and privileged users
2. Use IAM roles, not access keys
3. Implement least privilege
4. Enable CloudTrail in all regions
5. Encrypt data at rest and in transit
6. Use SCPs to enforce organizational policies
7. Regular security assessments with Inspector
8. Monitor with GuardDuty and Security Hub
9. Centralize logs in secure account
10. Implement detective and preventive controls
