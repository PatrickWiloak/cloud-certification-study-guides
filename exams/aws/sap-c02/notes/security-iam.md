# Security & IAM

## Overview

Enterprise security on AWS requires a defense-in-depth approach spanning identity and access management, data protection, infrastructure security, detective controls, and incident response. At the professional level, you must design security architectures that meet regulatory compliance requirements, implement least privilege access across multi-account organizations, protect sensitive data at scale, and establish comprehensive threat detection and response capabilities.

Professional architects must balance security requirements with operational efficiency, implement zero-trust architectures, design for compliance frameworks (HIPAA, PCI-DSS, SOC 2, FedRAMP), and establish security governance across complex organizational structures.

## Key Concepts

### Identity and Access Management (IAM)

**IAM Principals**
- **Users**: Long-term credentials for individuals (avoid for applications)
- **Groups**: Collections of users with common permissions
- **Roles**: Temporary credentials for services, federated users, cross-account access
- **Service-Linked Roles**: Predefined roles for AWS services
- **Instance Profiles**: Attach roles to EC2 instances

**IAM Policies**
- **Identity-Based Policies**: Attached to users, groups, or roles
- **Resource-Based Policies**: Attached to resources (S3 buckets, KMS keys, Lambda functions)
- **Permission Boundaries**: Maximum permissions an identity can have (delegated admin)
- **Service Control Policies (SCPs)**: Organization-wide preventive controls
- **Session Policies**: Limit permissions for assumed role sessions
- **Access Control Lists (ACLs)**: Legacy, resource-level permissions (S3, VPC)

**Policy Evaluation Logic**
1. **Explicit Deny**: Always takes precedence
2. **Organizations SCPs**: Limit maximum permissions
3. **Resource-Based Policies**: Can grant cross-account access
4. **IAM Permission Boundaries**: Limit maximum permissions for identity
5. **Identity-Based Policies**: Grant permissions
6. **Default**: Implicit deny if no explicit allow

**Advanced IAM Patterns**
- **Attribute-Based Access Control (ABAC)**: Use tags for dynamic permissions
- **Cross-Account Access**: Assume roles from other accounts
- **Resource-Based Policies**: Grant access without assuming roles (S3, Lambda, SQS, SNS)
- **IAM Roles Anywhere**: Extend IAM roles to workloads outside AWS
- **Session Tags**: Pass tags during AssumeRole for ABAC

### AWS Organizations and Multi-Account Security

**Service Control Policies (SCPs)**
- Preventive guardrails limiting permissions in member accounts
- Apply to all principals except management account root user
- Deny-list (default) or allow-list strategies
- Inheritance through organizational unit (OU) hierarchy
- Use cases: Prevent region usage, enforce encryption, restrict instance types, prevent service disablement

**Multi-Account Architecture**
- **Management Account**: Billing and organization management only
- **Security Account**: Centralized security services (GuardDuty, Security Hub, IAM Access Analyzer)
- **Log Archive Account**: Immutable logging storage
- **Audit Account**: Read-only access for auditors
- **Shared Services Account**: Active Directory, DNS, networking
- **Workload Accounts**: Application-specific, environment-specific (dev, staging, prod)

**Cross-Account Access Patterns**
- **IAM Roles**: AssumeRole for temporary credentials (preferred)
- **Resource-Based Policies**: Direct access for S3, Lambda, SQS, SNS
- **AWS RAM (Resource Access Manager)**: Share resources (Transit Gateway, subnets, Route 53 Resolver rules)
- **VPC Peering / PrivateLink**: Network-level sharing
- **AWS Organizations Delegated Administrator**: Delegate security service management

### AWS IAM Identity Center (formerly AWS SSO)

**Centralized Access Management**
- Single sign-on for AWS accounts and cloud applications
- Integration with external identity providers (SAML 2.0, Azure AD, Okta, Google Workspace)
- AWS managed identity store or Active Directory integration
- Permission sets mapped to IAM roles in member accounts
- Multi-account access through single login

**Use Cases**
- Enterprise workforce access to multiple AWS accounts
- Centralized user management and MFA enforcement
- Integration with corporate identity provider
- Audit trail with CloudTrail

### Data Protection

**Encryption at Rest**
- **EBS**: Encrypted volumes using KMS (default or customer-managed keys)
- **S3**: SSE-S3, SSE-KMS, SSE-C, client-side encryption
- **RDS/Aurora**: TDE (Transparent Data Encryption) with KMS
- **DynamoDB**: Encryption at rest with KMS
- **EFS**: KMS encryption
- **Redshift**: Cluster encryption with KMS or HSM
- **ElastiCache**: Redis encryption at rest

**Encryption in Transit**
- TLS 1.2+ for all service endpoints
- VPN (IPSec) for site-to-site and client VPN
- Direct Connect with MACsec for 100 Gbps connections
- Application-level encryption (SSL/TLS certificates from ACM)
- Private connectivity with PrivateLink (no internet exposure)

**AWS Key Management Service (KMS)**
- **AWS Managed Keys**: Created and managed by AWS (free, auto-rotated annually)
- **Customer Managed Keys**: Full control, configurable rotation, key policies
- **AWS Owned Keys**: Used by AWS services internally, not visible
- **Custom Key Store**: CloudHSM-backed keys for regulatory requirements
- **External Key Store**: Keys stored outside AWS
- Key policies for access control
- Grant mechanism for temporary permissions
- Automatic key rotation (annual for customer-managed keys)
- Envelope encryption for large data (data key encrypts data, master key encrypts data key)

**AWS CloudHSM**
- FIPS 140-2 Level 3 validated hardware security modules
- Single-tenant HSM instances in VPC
- Full control over encryption keys
- Use cases: PKI, TDE for databases, document signing
- Integration with KMS as custom key store

**AWS Certificate Manager (ACM)**
- Provision, manage, deploy SSL/TLS certificates
- Free public certificates for AWS services (ALB, CloudFront, API Gateway)
- Automatic renewal for ACM-issued certificates
- Import third-party certificates
- Private Certificate Authority for internal PKI

**AWS Secrets Manager**
- Store, rotate, manage secrets (API keys, database passwords)
- Automatic rotation with Lambda functions
- Integration with RDS, Redshift, DocumentDB
- Fine-grained access control with IAM
- Encryption with KMS
- Audit with CloudTrail
- Cross-region replication for DR

**AWS Systems Manager Parameter Store**
- Hierarchical storage for configuration and secrets
- Free tier for standard parameters (4 KB, no rotation)
- Advanced parameters (8 KB, parameter policies, higher throughput)
- Integration with KMS for encryption
- Versioning and expiration policies
- Cost-effective alternative to Secrets Manager for non-database secrets

### Network Security

**Security Groups**
- Stateful firewall at ENI level
- Allow rules only (implicit deny)
- Source/destination: IP, CIDR, security group, prefix list
- Protocol, port, ICMP type
- Changes take effect immediately
- Max 5 security groups per ENI
- Limit: 60 inbound, 60 outbound rules per security group

**Network ACLs (NACLs)**
- Stateless firewall at subnet level
- Allow and deny rules
- Evaluated in numerical order (1-32766)
- Separate inbound and outbound rules
- Default NACL allows all traffic
- Custom NACLs deny all by default

**AWS Network Firewall**
- Managed stateful network firewall
- IPS/IDS capabilities with Suricata-compatible rules
- Domain name filtering (allow/deny based on domain)
- Stateless and stateful rule groups
- TLS inspection (decrypt, inspect, re-encrypt)
- Centralized deployment in inspection VPC
- Logging to S3, CloudWatch, Kinesis Data Firehose

**AWS WAF (Web Application Firewall)**
- Layer 7 protection for CloudFront, ALB, API Gateway, AppSync
- Managed rule groups: AWS, third-party (F5, Fortinet)
- Custom rules: IP sets, geo-match, rate limiting, string matching, SQL injection, XSS
- Bot control and account takeover prevention
- CAPTCHA and challenge actions
- Logging to S3, CloudWatch, Kinesis Data Firehose
- Centralized management with AWS Firewall Manager

**AWS Shield**
- **Shield Standard**: Automatic DDoS protection for all AWS customers (free)
- **Shield Advanced**: Enhanced DDoS protection ($3,000/month)
  - 24/7 DDoS Response Team (DRT)
  - Cost protection (credits for scaling during attack)
  - Advanced metrics and detection
  - Integration with WAF for application-layer protection
  - Protection for Route 53, CloudFront, ALB, EIP

### Threat Detection and Monitoring

**Amazon GuardDuty**
- Intelligent threat detection using ML
- Data sources: VPC Flow Logs, CloudTrail, DNS logs, S3 data events, EKS audit logs, RDS login activity, Lambda network activity
- Threat categories: Reconnaissance, instance compromise, account compromise, bucket compromise
- Integration with Security Hub, EventBridge for automated response
- Multi-account management through Organizations delegated administrator
- Suppression rules for false positives

**AWS Security Hub**
- Centralized security findings aggregation
- Aggregates findings from GuardDuty, Inspector, Macie, IAM Access Analyzer, Firewall Manager, Systems Manager Patch Manager
- Security standards: AWS Foundational Security Best Practices, CIS AWS Foundations Benchmark, PCI-DSS
- Continuous compliance checks
- Custom insights and dashboards
- Integration with EventBridge for automation
- Cross-region aggregation

**Amazon Macie**
- Sensitive data discovery in S3 using ML
- PII, PHI, financial data, credentials
- Automated inventory of S3 buckets
- Sensitive data discovery jobs
- Policy findings for security issues (public access, unencrypted)
- Integration with Security Hub and EventBridge

**Amazon Inspector**
- Automated vulnerability assessment
- EC2 instances: OS vulnerabilities, network reachability
- Container images in ECR: Software vulnerabilities
- Lambda functions: Code vulnerabilities, package dependencies
- Continuous scanning with risk scores
- Integration with Security Hub

**Amazon Detective**
- Security investigation and root cause analysis
- Visualizations and analysis of security findings
- Data sources: VPC Flow Logs, CloudTrail, GuardDuty
- ML-based analysis of behavior over time
- Use cases: Investigate GuardDuty findings, identify root cause, scope of compromise

**AWS Config**
- Resource inventory and configuration history
- Config rules for compliance (managed and custom)
- Remediation with Systems Manager Automation or Lambda
- Conformance packs for compliance frameworks (PCI-DSS, HIPAA)
- Multi-account aggregation
- Integration with Security Hub

**AWS CloudTrail**
- API call logging for audit and compliance
- Management events: Control plane operations (CreateInstance, CreateBucket)
- Data events: Data plane operations (GetObject, PutObject)
- Insights events: Anomalous API activity detection
- Log file integrity validation
- Multi-region trail for global visibility
- Organization trail for all accounts
- Integration with CloudWatch Logs for real-time analysis
- S3 storage with SSE-KMS encryption

**VPC Flow Logs**
- IP traffic metadata for network troubleshooting and security analysis
- Capture at VPC, subnet, or ENI level
- Publish to CloudWatch Logs, S3, or Kinesis Data Firehose
- Custom format for specific fields
- Integration with GuardDuty for threat detection
- Athena for query and analysis

### Compliance and Governance

**AWS Artifact**
- Access to compliance reports and agreements
- SOC 1/2/3, PCI-DSS AOC, ISO certifications
- Business Associate Agreement (BAA) for HIPAA
- Self-service access for auditors

**AWS Audit Manager**
- Continuous audit readiness
- Pre-built frameworks: HIPAA, PCI-DSS, GDPR, SOC 2, NIST
- Automated evidence collection
- Custom frameworks and controls
- Dashboard for audit status
- Integration with Security Hub, CloudTrail, Config

**AWS Control Tower**
- Landing zone setup for multi-account environments
- Preventive guardrails (SCPs)
- Detective guardrails (Config rules)
- Account Factory for automated provisioning
- Centralized logging to Log Archive account
- Dashboard for compliance status

**Data Residency and Sovereignty**
- Region selection for compliance requirements
- SCPs to prevent operations outside allowed regions
- S3 Object Lock for immutable storage
- Encryption with customer-managed keys in specific regions
- VPC endpoints to keep traffic within region

### Incident Response

**Automated Response**
- **EventBridge**: Route security findings to Lambda, SNS, SQS, Step Functions
- **Lambda**: Automated remediation (revoke credentials, isolate instances, block IPs)
- **Systems Manager Automation**: Runbooks for incident response
- **Step Functions**: Multi-step response workflows

**Forensics**
- EBS snapshots of compromised instances
- VPC Traffic Mirroring for packet capture
- Memory dumps using Systems Manager Run Command
- S3 access logs and CloudTrail for investigation
- Detective for visual investigation

**IR Preparation**
- IR playbooks and runbooks
- Automated isolation procedures
- Pre-created forensic VPC
- Backup and snapshot retention policies
- Communication plan (stakeholders, legal, PR)

## AWS Services Reference

### Core Services

**AWS IAM**
- Users, groups, roles, policies
- Permission boundaries and session policies
- MFA enforcement
- IAM Access Analyzer for unused access

**AWS Organizations**
- Multi-account management
- Service Control Policies (SCPs)
- Consolidated billing
- Cross-account resource sharing

**AWS IAM Identity Center**
- Single sign-on for multiple accounts
- Integration with external IdPs
- Permission sets and access portal
- MFA enforcement

**AWS KMS**
- Managed encryption key service
- Customer-managed and AWS-managed keys
- Key policies and grants
- CloudHSM integration

### Supporting Services

**Amazon GuardDuty**
- Intelligent threat detection
- Multi-account management
- Integration with Security Hub

**AWS Security Hub**
- Centralized security posture management
- Compliance standards
- Findings aggregation

**Amazon Macie**
- Sensitive data discovery in S3
- PII, PHI, financial data detection
- Policy findings for security issues

**AWS CloudTrail**
- API call logging
- CloudTrail Insights for anomalies
- Multi-region and organization trails

**AWS Config**
- Resource configuration tracking
- Compliance monitoring with Config rules
- Conformance packs

**AWS Secrets Manager**
- Secret storage and rotation
- Integration with databases
- Cross-region replication

**AWS Certificate Manager**
- SSL/TLS certificate management
- Automatic renewal
- Private CA for internal PKI

## Architecture Patterns

### Pattern 1: Multi-Account Security Architecture

**Use Case**
- Enterprise with multiple teams and environments
- Centralized security monitoring and governance
- Least privilege access with clear boundaries
- Compliance and audit requirements

**Implementation Approach**
1. **Organization Structure**:
   - Management account (billing only, no workloads)
   - Security OU: Security Tools, Log Archive, Audit accounts
   - Infrastructure OU: Network, Shared Services
   - Workload OUs: By business unit or environment
2. **Security Account**: GuardDuty, Security Hub, IAM Access Analyzer, Config aggregator
3. **Log Archive Account**: Centralized CloudTrail, VPC Flow Logs, Config logs (immutable S3 bucket)
4. **SCPs**: Prevent region usage, enforce encryption, deny root user actions (except in management account)
5. **IAM Identity Center**: Centralized SSO with permission sets
6. **Cross-Account Roles**: Auditor role, incident response role, DevOps role
7. **AWS Config**: Conformance packs deployed to all accounts
8. **Automated Response**: EventBridge rules forward Security Hub findings to Lambda for remediation

**Pros/Cons**
- Pros: Strong isolation, centralized security, clear compliance, least privilege
- Cons: Complexity, cross-account permissions management, learning curve

### Pattern 2: Zero-Trust Network Architecture

**Use Case**
- High-security environment requiring authentication and authorization for all connections
- Microsegmentation and least privilege network access
- Protection against lateral movement

**Implementation Approach**
1. **Private Subnets Only**: No internet gateway, use VPC endpoints
2. **VPC Endpoints**: S3, DynamoDB, ECR, ECS, Secrets Manager, Systems Manager
3. **PrivateLink**: Access services without VPC peering
4. **Security Groups**: Minimal allow-lists, reference other security groups (not IP ranges)
5. **Network Firewall**: Centralized inspection with strict allow-lists
6. **TLS Everywhere**: Mutual TLS for service-to-service communication
7. **AWS Certificate Manager**: Private CA for internal certificates
8. **IAM Roles**: Instance profiles for all EC2, ECS tasks, Lambda functions
9. **No Long-Term Credentials**: Rotate temporary credentials automatically
10. **GuardDuty**: Monitor for suspicious network activity

**Pros/Cons**
- Pros: Highest security, prevents lateral movement, defense in depth
- Cons: Operational complexity, debugging challenges, higher costs (PrivateLink, Network Firewall)

### Pattern 3: Automated Compliance and Remediation

**Use Case**
- Regulatory compliance requirements (PCI-DSS, HIPAA, SOC 2)
- Continuous compliance monitoring
- Automated remediation to reduce manual effort
- Audit readiness

**Implementation Approach**
1. **AWS Config**: Deploy conformance packs for compliance frameworks
2. **Config Rules**: Managed rules for encryption, public access, approved AMIs, etc.
3. **Security Hub**: Enable security standards (CIS Benchmark, AWS Foundational Best Practices)
4. **EventBridge Rules**: Trigger on Config compliance changes and Security Hub findings
5. **Lambda Functions**: Automated remediation
   - Revoke overly permissive security group rules
   - Enable encryption on non-compliant resources
   - Delete default VPCs
   - Enable MFA for IAM users
6. **Systems Manager Automation**: Runbooks for complex remediation
7. **SNS Notifications**: Alert security team for high-severity findings
8. **Audit Manager**: Continuous evidence collection for audit readiness
9. **CloudTrail**: Organization trail with log file validation

**Pros/Cons**
- Pros: Continuous compliance, reduced manual effort, faster remediation, audit readiness
- Cons: Risk of over-automation, careful testing required, event-driven complexity

### Pattern 4: Secrets Management at Scale

**Use Case**
- Hundreds of applications with database passwords, API keys, certificates
- Automatic secret rotation
- Least privilege access to secrets
- Audit trail for secret usage

**Implementation Approach**
1. **Secrets Manager**: Store all secrets (database passwords, API keys)
2. **Automatic Rotation**: Lambda functions for RDS, Redshift, DocumentDB (built-in), custom rotation for APIs
3. **IAM Policies**: Fine-grained access per application (resource-based and identity-based)
4. **VPC Endpoints**: Private access to Secrets Manager (no internet)
5. **KMS**: Customer-managed keys for secret encryption
6. **Parameter Store**: Configuration parameters (non-sensitive)
7. **Cross-Region Replication**: DR for critical secrets
8. **CloudTrail**: Audit secret retrieval (who accessed what, when)
9. **Lambda Environment Variables**: Reference secrets ARN, retrieve at runtime (not hardcode)
10. **Container Secrets**: ECS/EKS integration with Secrets Manager

**Pros/Cons**
- Pros: No hardcoded secrets, automatic rotation, audit trail, least privilege
- Cons: Cost (Secrets Manager $0.40/secret/month + API calls), dependency on service availability

### Pattern 5: Threat Detection and Response

**Use Case**
- Proactive security monitoring for threats
- Automated response to security incidents
- Forensic investigation capabilities
- 24/7 security operations

**Implementation Approach**
1. **GuardDuty**: Enable in all accounts and regions, multi-account management
2. **Security Hub**: Aggregate findings from GuardDuty, Macie, Inspector, Config
3. **Macie**: Scan S3 buckets for sensitive data
4. **Inspector**: Vulnerability scanning for EC2, ECR, Lambda
5. **VPC Flow Logs**: Enable for all VPCs, publish to S3
6. **CloudTrail**: Management and data events, CloudTrail Insights
7. **EventBridge**: Route high/critical findings to SIEM and Lambda
8. **Automated Response Lambda**:
   - Isolate compromised EC2 (replace security group, snapshot EBS)
   - Revoke active sessions for compromised IAM user
   - Block malicious IP in WAF
   - Notify security team via SNS
9. **Detective**: Investigate findings with visual analysis
10. **SIEM Integration**: Splunk, Sumo Logic, or AWS-native (OpenSearch)

**Pros/Cons**
- Pros: Proactive threat detection, automated response, reduced MTTR, comprehensive visibility
- Cons: Alert fatigue (tune with suppressions), complexity, costs (GuardDuty, Security Hub, SIEM)

### Pattern 6: Data Encryption Everywhere

**Use Case**
- Compliance requirements for data protection (HIPAA, PCI-DSS)
- Protection against insider threats
- Encryption key management and audit

**Implementation Approach**
1. **At Rest**:
   - EBS: Encryption by default with KMS
   - S3: Bucket default encryption (SSE-KMS)
   - RDS/Aurora: TDE with KMS
   - DynamoDB: Encryption at rest
   - EFS, Redshift, ElastiCache: KMS encryption
2. **In Transit**:
   - TLS 1.2+ for all applications (ALB, CloudFront, API Gateway)
   - VPN for on-premises connectivity
   - Direct Connect with MACsec
   - PrivateLink for private service access
3. **KMS**:
   - Customer-managed keys (not AWS-managed)
   - Separate keys per environment (dev, staging, prod)
   - Separate keys per data classification (public, internal, confidential, restricted)
   - Key policies for least privilege
   - Automatic key rotation enabled
4. **SCP**: Deny creating unencrypted resources (S3 buckets, EBS volumes, RDS)
5. **Config Rules**: Check for unencrypted resources
6. **CloudTrail**: KMS key usage audit

**Pros/Cons**
- Pros: Data protection at rest and in transit, compliance, key management, audit trail
- Cons: KMS costs, performance impact (minimal), key policy complexity

## Best Practices

### Enterprise-Level Recommendations

**Least Privilege**
- Grant minimum permissions required
- Use IAM roles instead of long-term access keys
- Implement permission boundaries for delegated admin
- Regular access reviews and removal of unused permissions
- IAM Access Analyzer to identify unused access

**Defense in Depth**
- Multiple layers of security (network, application, data)
- Security groups, NACLs, WAF, Network Firewall
- Encryption at rest and in transit
- Detective and preventive controls
- Assume breach mentality

**Centralized Security Management**
- Multi-account architecture with dedicated security account
- SCPs for organization-wide preventive controls
- Centralized logging (CloudTrail, VPC Flow Logs, Config)
- Security Hub for aggregated findings
- IAM Identity Center for SSO

**Automated Security**
- Config rules for compliance checks
- Automated remediation with Lambda and Systems Manager
- GuardDuty for threat detection
- EventBridge for event-driven security
- Infrastructure as Code with security scanning

### Security Considerations

**Identity Management**
- MFA for all human users (especially root and privileged users)
- Federate with corporate IdP (SAML, OIDC) using IAM Identity Center
- Service roles for applications (no access keys)
- Temporary credentials (AssumeRole) for cross-account access
- Disable unused credentials (IAM credential reports)

**Data Protection**
- Classify data based on sensitivity
- Encrypt all data at rest (S3, EBS, RDS, DynamoDB)
- Encrypt data in transit (TLS 1.2+)
- Customer-managed KMS keys for sensitive data
- S3 Object Lock for immutable storage (compliance)
- Secrets Manager for credentials and API keys

**Network Security**
- Private subnets for application and database tiers
- VPC endpoints for AWS service access
- Security groups as primary firewall (not NACLs)
- Network Firewall for centralized inspection
- WAF for application-layer protection
- Shield Advanced for DDoS protection (critical workloads)

**Monitoring and Detection**
- GuardDuty in all accounts and regions
- Security Hub for centralized findings
- Macie for sensitive data discovery
- CloudTrail for API audit trail
- VPC Flow Logs for network traffic analysis
- Config for configuration compliance

### Cost Optimization

**KMS Optimization**
- S3 Bucket Keys to reduce KMS API calls (99% reduction)
- AWS-managed keys for non-sensitive data (free)
- Minimize cross-region KMS calls (higher latency and cost)

**GuardDuty Optimization**
- CloudTrail and DNS logs free with GuardDuty
- VPC Flow Logs add cost ($0.80/GB in us-east-1)
- S3 protection add cost ($0.40/GB scanned)
- Use VPC Flow Log sampling for cost reduction

**Secrets Manager vs. Parameter Store**
- Secrets Manager: $0.40/secret/month, rotation included
- Parameter Store: Free (standard), $0.05/parameter/month (advanced)
- Use Parameter Store for configuration, Secrets Manager for credentials

**Security Hub Optimization**
- Consolidated findings from all security services
- Archive findings that don't require action
- Cost: $0.0010 per finding ingested (first 10K free per account/region/month)

### Performance Tuning

**IAM Policy Optimization**
- Minimize policy size (6,144 characters for user policies)
- Use managed policies for reusability
- Cache AssumeRole credentials (1-12 hours)
- Avoid excessive permission checks in hot path

**KMS Performance**
- Envelope encryption for large data (encrypt data key, not data)
- Cache data keys in application
- Shared request quotas (5,500 symmetric requests/second in us-east-1)
- Request quota increases for high-throughput workloads

**Secrets Manager Caching**
- Cache secrets in application (don't retrieve every request)
- Secrets Manager client-side caching library
- Balance security (rotation frequency) with performance (cache TTL)

## Common Scenarios

### Scenario 1: PCI-DSS Compliant E-Commerce Platform

**Context**: E-commerce platform processing credit card payments, requiring PCI-DSS Level 1 compliance

**Security Architecture**
1. **Multi-Account Structure**:
   - Cardholder Data Environment (CDE) in dedicated account
   - Non-CDE workloads in separate accounts
   - Security and Log Archive accounts
2. **Network Segmentation**:
   - CDE in isolated VPC with no internet gateway
   - Network Firewall for all CDE traffic inspection
   - PrivateLink for AWS service access
   - VPC Flow Logs for all traffic
3. **Encryption**:
   - All data encrypted at rest (EBS, S3, RDS) with customer-managed KMS keys
   - TLS 1.2 for all data in transit
   - Field-level encryption for credit card numbers
4. **Access Control**:
   - IAM Identity Center with MFA for all users
   - SCPs prevent disabling encryption, CloudTrail, Config
   - Just-in-time access for CDE (temporary credentials)
   - No long-term credentials in CDE
5. **Monitoring**:
   - GuardDuty, Security Hub, Config in all accounts
   - CloudTrail organization trail with log file validation
   - Automated alerts for unauthorized access attempts
   - File integrity monitoring with Systems Manager or third-party
6. **Compliance**:
   - Config conformance pack for PCI-DSS
   - Quarterly vulnerability scans with Inspector
   - Annual penetration testing
   - Audit Manager for continuous compliance
7. **SCPs**:
   - Deny operations outside allowed regions
   - Deny creation of unencrypted resources
   - Deny public S3 access in CDE account
   - Deny root user access (except emergency)

**Key Considerations**
- Quarterly ASV scans and annual penetration testing
- Compensating controls documentation
- Change management with approval workflow
- Separation of duties for database access
- Vendor management for third-party integrations

### Scenario 2: HIPAA Healthcare Application

**Context**: Healthcare SaaS platform storing Protected Health Information (PHI) for 5M patients

**Security Architecture**
1. **Business Associate Agreement (BAA)** with AWS
2. **HIPAA-Eligible Services Only**: EC2, EBS, S3, RDS, DynamoDB, ECS, Lambda, etc.
3. **Encryption**:
   - All PHI encrypted at rest with customer-managed KMS keys
   - TLS 1.3 for data in transit
   - Client-side encryption for highly sensitive data
4. **Access Control**:
   - Role-based access control (RBAC) with IAM
   - Minimum necessary access principle
   - MFA for all access to PHI
   - Audit logging for all PHI access (CloudTrail data events)
   - Automatic session timeout (15 minutes)
5. **Network Security**:
   - Private subnets only, VPC endpoints for AWS services
   - Application Load Balancer with AWS WAF
   - DDoS protection with Shield Standard
6. **Monitoring**:
   - GuardDuty for threat detection
   - Macie for PHI discovery and classification
   - Config for HIPAA compliance checks
   - Automated alerts for security findings
7. **Backup and DR**:
   - Encrypted backups with 6-year retention
   - Cross-region replication for disaster recovery
   - S3 Object Lock for immutable backups
8. **Incident Response**:
   - Breach notification procedures (60 days)
   - Forensic VPC for investigation
   - EBS snapshots of compromised instances
9. **Audit**:
   - Audit Manager with HIPAA framework
   - Annual HIPAA compliance assessment
   - Risk analysis and security risk assessment

**Key Considerations**
- Patient consent management
- Right to access (patients can request their data)
- Data retention and disposal policies
- Workforce training on HIPAA requirements
- Subcontractor BAAs for third-party services

### Scenario 3: Multi-Tenant SaaS with Data Isolation

**Context**: B2B SaaS platform with 1,000 enterprise customers requiring strict data isolation

**Security Architecture**
1. **Isolation Strategy**:
   - **Silo Model**: Separate account per customer (highest isolation, high cost)
   - **Pool Model**: Shared infrastructure with logical isolation (lower cost, lower isolation)
   - **Hybrid**: Dedicated accounts for top-tier customers, shared for standard customers
2. **Tenant Identification**:
   - Tenant ID in JWT token
   - Verified at API Gateway with Lambda authorizer
   - Passed to all downstream services
3. **Data Isolation**:
   - **DynamoDB**: Partition key includes tenant ID
   - **S3**: Separate buckets per tenant or prefix-based isolation
   - **RDS**: Separate databases per tenant or row-level security (PostgreSQL)
   - IAM policies with ABAC (tenant ID in principal tag)
4. **Compute Isolation**:
   - Lambda: Separate functions per tenant or tenant context in environment
   - ECS: Separate task definitions per tenant or shared with tenant context
   - API Gateway: Resource policies limiting access
5. **Access Control**:
   - Customer IdP federation (SAML, OIDC)
   - Cognito user pools per tenant
   - Token includes tenant ID claim
   - All API calls validate tenant context
6. **Monitoring Per Tenant**:
   - CloudWatch metrics with tenant dimension
   - Per-tenant dashboards for customers
   - GuardDuty findings tagged with tenant ID
7. **Cross-Tenant Isolation Testing**:
   - Automated tests attempting cross-tenant access
   - Regular penetration testing
   - IAM Access Analyzer for unintended access

**Key Considerations**
- Trade-off: Isolation vs. cost vs. operational complexity
- Noisy neighbor problem (resource limits per tenant)
- Tenant offboarding and data deletion
- Cost allocation per tenant
- SLA differentiation by tenant tier

### Scenario 4: Insider Threat Protection

**Context**: Financial services firm protecting against insider threats and privileged user abuse

**Security Architecture**
1. **Least Privilege**:
   - IAM permission boundaries for all users
   - Just-in-time access with time-limited credentials
   - Break-glass procedures for emergency access
2. **Separation of Duties**:
   - Separate accounts for dev, staging, prod
   - Cross-account access requires approval
   - No individual has both read and write access to production data
3. **MFA Everywhere**:
   - Hardware MFA tokens (YubiKey) for privileged users
   - MFA required for all AWS Console access
   - MFA required for AssumeRole to production
4. **Audit Trail**:
   - CloudTrail with log file validation
   - Logs sent to immutable S3 bucket in separate account
   - Athena for log analysis
   - CloudWatch Insights for real-time analysis
5. **Data Access Monitoring**:
   - CloudTrail data events for S3 (GetObject, PutObject)
   - Macie for sensitive data access patterns
   - GuardDuty for anomalous credential usage
   - Detective for user behavior analysis
6. **DLP (Data Loss Prevention)**:
   - S3 Block Public Access at account level
   - SCPs preventing public access
   - S3 Object Lock for critical data (WORM)
   - VPC endpoints (no internet access)
7. **Privilege Escalation Prevention**:
   - SCPs preventing IAM policy modification
   - Permission boundaries preventing privilege escalation
   - Config rules for IAM policy changes
   - Automated alerts for new IAM users/roles
8. **Automated Response**:
   - Revoke sessions on suspicious activity
   - Snapshot resources before isolation
   - Notify security team and management
   - Initiate incident response workflow

**Key Considerations**
- Balance security with operational efficiency
- User training on security policies
- Regular access reviews and audits
- Termination procedures (immediate credential revocation)
- Vendor and contractor access management

### Scenario 5: Regulated Financial Services (FINRA, SEC)

**Context**: Investment firm subject to FINRA, SEC regulations requiring comprehensive audit trails and data retention

**Security Architecture**
1. **Immutable Audit Logs**:
   - CloudTrail organization trail to S3 with Object Lock (Compliance mode, 7 years)
   - VPC Flow Logs to S3 with Object Lock
   - Application logs to S3 with Object Lock
   - MFA Delete enabled on log buckets
2. **Data Retention**:
   - 7-year retention for communications (emails, chats)
   - 7-year retention for trading records
   - S3 lifecycle to Glacier Deep Archive after 1 year
   - Legal hold for litigation support
3. **Access Control**:
   - Four-eyes principle (dual authorization) for critical operations
   - Approval workflow using Step Functions and SNS
   - IAM Identity Center with corporate AD integration
   - No root user access (except break-glass)
4. **Encryption**:
   - Customer-managed KMS keys with CloudHSM
   - Separate keys per data classification
   - Key policies preventing deletion (DenyDelete)
   - Annual key rotation with audit trail
5. **Network Security**:
   - Private connectivity only (Direct Connect, no internet)
   - VPC endpoints for all AWS services
   - Network Firewall for egress filtering (allow-list)
   - WAF for public-facing websites
6. **Compliance Monitoring**:
   - Config conformance packs (custom for FINRA/SEC)
   - Security Hub custom standards
   - Audit Manager with custom framework
   - Quarterly compliance reports
7. **Surveillance**:
   - Communication monitoring (email, chat) for insider trading
   - Trade surveillance systems
   - GuardDuty for threat detection
   - Macie for detecting sensitive data leaks
8. **Change Management**:
   - All infrastructure as code (CloudFormation)
   - Peer review for all changes
   - Automated testing in non-production
   - Change approval board for production
   - Automated rollback procedures

**Key Considerations**
- Regulatory examinations and audits
- E-discovery for litigation
- Preservation of electronic records
- Supervisory procedures for compliance
- Business continuity and disaster recovery (COOP)

## AWS CLI Examples

```bash
# IAM - Create user with MFA requirement
aws iam create-user --user-name john.doe

aws iam create-login-profile \
  --user-name john.doe \
  --password 'TempPassword123!' \
  --password-reset-required

aws iam attach-user-policy \
  --user-name john.doe \
  --policy-arn arn:aws:iam::aws:policy/IAMUserChangePassword

# IAM - Create role for cross-account access
aws iam create-role \
  --role-name CrossAccountAuditor \
  --assume-role-policy-document file://trust-policy.json

# trust-policy.json:
# {
#   "Version": "2012-10-17",
#   "Statement": [{
#     "Effect": "Allow",
#     "Principal": {"AWS": "arn:aws:iam::111111111111:root"},
#     "Action": "sts:AssumeRole",
#     "Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}
#   }]
# }

aws iam attach-role-policy \
  --role-name CrossAccountAuditor \
  --policy-arn arn:aws:iam::aws:policy/SecurityAudit

# IAM - Create permission boundary
aws iam create-policy \
  --policy-name DeveloperBoundary \
  --policy-document file://boundary.json

aws iam put-user-permission-boundary \
  --user-name developer1 \
  --permissions-boundary arn:aws:iam::123456789012:policy/DeveloperBoundary

# IAM - Enable MFA for user
aws iam enable-mfa-device \
  --user-name john.doe \
  --serial-number arn:aws:iam::123456789012:mfa/john.doe \
  --authentication-code-1 123456 \
  --authentication-code-2 789012

# IAM - Get credential report
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 -d > credential-report.csv

# IAM Access Analyzer - Create analyzer
aws accessanalyzer create-analyzer \
  --analyzer-name OrganizationAnalyzer \
  --type ORGANIZATION \
  --tags Key=Environment,Value=Production

# Organizations - Create SCP
aws organizations create-policy \
  --name DenyUnencryptedStorage \
  --description "Prevent creation of unencrypted S3 buckets and EBS volumes" \
  --type SERVICE_CONTROL_POLICY \
  --content file://scp.json

# scp.json example:
# {
#   "Version": "2012-10-17",
#   "Statement": [{
#     "Effect": "Deny",
#     "Action": ["s3:PutObject"],
#     "Resource": "*",
#     "Condition": {
#       "StringNotEquals": {"s3:x-amz-server-side-encryption": ["AES256", "aws:kms"]}
#     }
#   }, {
#     "Effect": "Deny",
#     "Action": "ec2:RunInstances",
#     "Resource": "arn:aws:ec2:*:*:volume/*",
#     "Condition": {"Bool": {"ec2:Encrypted": "false"}}
#   }]
# }

aws organizations attach-policy \
  --policy-id p-12345678 \
  --target-id ou-abc123-xyz456

# KMS - Create customer-managed key
aws kms create-key \
  --description "Production data encryption key" \
  --key-policy file://key-policy.json \
  --tags TagKey=Environment,TagValue=Production

aws kms create-alias \
  --alias-name alias/production-data \
  --target-key-id <key-id>

# KMS - Enable automatic key rotation
aws kms enable-key-rotation --key-id <key-id>

# KMS - Encrypt data
echo "Sensitive data" > plaintext.txt
aws kms encrypt \
  --key-id alias/production-data \
  --plaintext fileb://plaintext.txt \
  --query CiphertextBlob \
  --output text | base64 -d > encrypted.bin

# Secrets Manager - Create secret
aws secretsmanager create-secret \
  --name prod/db/password \
  --description "Production database password" \
  --secret-string '{"username":"admin","password":"SecureP@ssw0rd!"}' \
  --kms-key-id alias/production-data \
  --tags Key=Environment,Value=Production

# Secrets Manager - Enable automatic rotation
aws secretsmanager rotate-secret \
  --secret-id prod/db/password \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRotation \
  --rotation-rules AutomaticallyAfterDays=30

# Secrets Manager - Retrieve secret
aws secretsmanager get-secret-value \
  --secret-id prod/db/password \
  --query SecretString \
  --output text

# Systems Manager Parameter Store - Create parameter
aws ssm put-parameter \
  --name /app/config/api-endpoint \
  --value "https://api.example.com" \
  --type String \
  --tags Key=Environment,Value=Production

aws ssm put-parameter \
  --name /app/config/api-key \
  --value "secret-api-key-12345" \
  --type SecureString \
  --key-id alias/production-data

# GuardDuty - Enable GuardDuty
aws guardduty create-detector --enable

# GuardDuty - Create filter for findings
aws guardduty create-filter \
  --detector-id <detector-id> \
  --name HighSeverityFindings \
  --action ARCHIVE \
  --finding-criteria file://criteria.json

# Security Hub - Enable Security Hub
aws securityhub enable-security-hub \
  --enable-default-standards

# Security Hub - Enable specific standards
aws securityhub batch-enable-standards \
  --standards-subscription-requests '[
    {"StandardsArn":"arn:aws:securityhub:us-east-1::standards/cis-aws-foundations-benchmark/v/1.4.0"},
    {"StandardsArn":"arn:aws:securityhub:::ruleset/pci-dss/v/3.2.1"}
  ]'

# Security Hub - Get findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}],"WorkflowStatus":[{"Value":"NEW","Comparison":"EQUALS"}]}' \
  --query 'Findings[*].[Title,ProductArn,CreatedAt]' \
  --output table

# Macie - Enable Macie
aws macie2 enable-macie

# Macie - Create classification job
aws macie2 create-classification-job \
  --job-type ONE_TIME \
  --name "S3-PII-Discovery" \
  --s3-job-definition '{
    "bucketDefinitions": [{
      "accountId": "123456789012",
      "buckets": ["my-sensitive-data-bucket"]
    }]
  }' \
  --managed-data-identifier-ids recommended

# Config - Start configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/ConfigRole

aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=config-bucket-123456789012

aws configservice start-configuration-recorder \
  --configuration-recorder-name default

# Config - Create conformance pack
aws configservice put-conformance-pack \
  --conformance-pack-name PCI-DSS-Compliance \
  --template-s3-uri s3://config-templates/pci-dss-pack.yaml \
  --delivery-s3-bucket config-bucket-123456789012

# CloudTrail - Create organization trail
aws cloudtrail create-trail \
  --name OrganizationTrail \
  --s3-bucket-name cloudtrail-logs-123456789012 \
  --is-organization-trail \
  --is-multi-region-trail \
  --enable-log-file-validation \
  --kms-key-id alias/cloudtrail-key

aws cloudtrail start-logging --name OrganizationTrail

# CloudTrail - Enable CloudTrail Insights
aws cloudtrail put-insight-selectors \
  --trail-name OrganizationTrail \
  --insight-selectors '[{"InsightType":"ApiCallRateInsight"}]'

# WAF - Create IP set
aws wafv2 create-ip-set \
  --name BlockedIPs \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses 192.0.2.0/24 203.0.113.0/24 \
  --region us-east-1

# WAF - Create web ACL
aws wafv2 create-web-acl \
  --name ProductionWAF \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://waf-rules.json \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=ProductionWAFMetric \
  --region us-east-1

# WAF - Associate with ALB
aws wafv2 associate-web-acl \
  --web-acl-arn <web-acl-arn> \
  --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890abcdef \
  --region us-east-1

# VPC Flow Logs - Enable with encryption
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::vpc-flow-logs-123456789012/vpc-12345678/ \
  --log-format '${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}' \
  --destination-options FileFormat=parquet,HiveCompatiblePartitions=true,PerHourPartition=false

# S3 - Block public access (account-level)
aws s3control put-public-access-block \
  --account-id 123456789012 \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# S3 - Enable Object Lock
aws s3api put-object-lock-configuration \
  --bucket compliance-bucket \
  --object-lock-configuration '{"ObjectLockEnabled":"Enabled","Rule":{"DefaultRetention":{"Mode":"COMPLIANCE","Years":7}}}'

# ACM - Request certificate
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com \
  --validation-method DNS \
  --tags Key=Environment,Value=Production

# Inspector - Create assessment target
aws inspector create-assessment-target \
  --assessment-target-name "Production-EC2-Instances" \
  --resource-group-arn arn:aws:inspector:us-east-1:123456789012:resourcegroup/0-abc123
```

## Study Tips

### SAP-C02 Exam Focus Areas

**High-Priority Security Topics**
- Multi-account security architecture with AWS Organizations and SCPs
- IAM policy evaluation logic and advanced patterns (ABAC, permission boundaries)
- Cross-account access patterns (roles vs. resource-based policies)
- Encryption strategies (at-rest and in-transit) with KMS
- GuardDuty, Security Hub, Macie for threat detection
- Automated compliance with Config and Config rules
- Secrets management with Secrets Manager
- VPC security (security groups, NACLs, Network Firewall, WAF)
- CloudTrail for audit and compliance
- Incident response and forensics

**Scenario-Based Questions**
- Design security architecture for compliance (HIPAA, PCI-DSS)
- Implement least privilege access across multi-account organization
- Choose encryption strategy based on regulatory requirements
- Design automated threat detection and response
- Troubleshoot IAM permission issues
- Implement data loss prevention (DLP)
- Design secrets management for containerized applications
- Multi-tenant data isolation strategies

**Common Decision Points**
- **SCPs vs. IAM policies**: Preventive guardrails vs. permissions
- **Roles vs. Resource-based policies**: Cross-account access patterns
- **Secrets Manager vs. Parameter Store**: Rotation vs. cost
- **KMS vs. CloudHSM**: Managed vs. dedicated HSM
- **Security Groups vs. NACLs**: Stateful vs. stateless
- **GuardDuty vs. Security Hub**: Threat detection vs. aggregation
- **AWS-managed keys vs. Customer-managed keys**: Convenience vs. control

### Key Differences from SAA-C03

**SAA-C03 Security Knowledge**
- Basic IAM (users, groups, roles, policies)
- S3 encryption and bucket policies
- Security groups and NACLs
- KMS basics
- CloudTrail for auditing

**Additional SAP-C02 Requirements**
- Advanced IAM (ABAC, permission boundaries, session policies, policy evaluation logic)
- AWS Organizations and SCPs for multi-account governance
- IAM Identity Center for enterprise SSO
- Advanced KMS (key policies, grants, custom key store, external key store)
- Comprehensive threat detection (GuardDuty, Macie, Detective, Inspector)
- Security Hub for centralized security management
- Automated compliance and remediation with Config
- Secrets Manager with automatic rotation
- Network security at scale (Network Firewall, WAF, Shield Advanced)
- Compliance frameworks (HIPAA, PCI-DSS, SOC 2)
- Incident response and forensics
- Zero-trust architectures

### Complex Scenarios to Master

**Multi-Account Security**
- Organization structure for security isolation
- SCPs for preventive controls
- Cross-account access patterns (roles, resource-based policies, RAM)
- Centralized logging and monitoring
- IAM Identity Center for SSO

**Compliance and Governance**
- PCI-DSS, HIPAA, SOC 2, FedRAMP requirements
- Automated compliance with Config conformance packs
- Immutable audit logs with S3 Object Lock
- Audit Manager for continuous audit readiness
- Data residency and sovereignty

**Threat Detection and Response**
- GuardDuty findings and automated response
- Security Hub standards and custom insights
- Macie for sensitive data discovery
- Detective for root cause analysis
- Incident response automation with EventBridge and Lambda

**Data Protection**
- Encryption everywhere (at-rest and in-transit)
- KMS key management strategies
- Secrets rotation and management
- DLP with Macie and VPC endpoints
- S3 Object Lock for WORM compliance

**Zero-Trust Architecture**
- Private subnets and VPC endpoints
- PrivateLink for service access
- Mutual TLS for service-to-service communication
- No long-term credentials
- Microsegmentation with security groups

### Practice Lab Recommendations

1. **Multi-Account Security Setup**
   - Create AWS Organization with OUs
   - Implement SCPs (deny regions, enforce encryption)
   - Set up IAM Identity Center with permission sets
   - Create cross-account roles
   - Test policy evaluation logic

2. **Threat Detection and Response**
   - Enable GuardDuty in multi-account setup
   - Enable Security Hub with multiple standards
   - Generate test findings (GuardDuty sample findings)
   - Create EventBridge rules for automated response
   - Build Lambda functions for remediation

3. **Secrets Management**
   - Store database password in Secrets Manager
   - Enable automatic rotation
   - Access secret from Lambda function
   - Test VPC endpoint for private access
   - Compare with Parameter Store

4. **Data Encryption**
   - Create customer-managed KMS key
   - Encrypt S3 bucket with KMS
   - Enable S3 Bucket Keys
   - Encrypt EBS volume and RDS database
   - Test key policies and grants

5. **Compliance Automation**
   - Enable AWS Config
   - Deploy conformance pack (CIS Benchmark)
   - Create custom Config rule
   - Build automated remediation with Lambda
   - Review compliance dashboard

6. **Network Security**
   - Configure security groups and NACLs
   - Deploy Network Firewall with rules
   - Set up WAF on ALB
   - Enable VPC Flow Logs
   - Analyze traffic with Athena

## Additional Resources

### AWS Whitepapers
- AWS Security Best Practices
- AWS Well-Architected Framework - Security Pillar
- Organizing Your AWS Environment Using Multiple Accounts
- AWS Key Management Service Best Practices
- AWS Incident Response Guide
- NIST Cybersecurity Framework in AWS
- AWS HIPAA Compliance Whitepaper
- PCI DSS on AWS

### Documentation Links
- AWS IAM: https://docs.aws.amazon.com/iam/
- AWS Organizations: https://docs.aws.amazon.com/organizations/
- AWS IAM Identity Center: https://docs.aws.amazon.com/singlesignon/
- AWS KMS: https://docs.aws.amazon.com/kms/
- AWS Secrets Manager: https://docs.aws.amazon.com/secretsmanager/
- Amazon GuardDuty: https://docs.aws.amazon.com/guardduty/
- AWS Security Hub: https://docs.aws.amazon.com/securityhub/
- AWS CloudTrail: https://docs.aws.amazon.com/cloudtrail/
- AWS Config: https://docs.aws.amazon.com/config/
- AWS WAF: https://docs.aws.amazon.com/waf/

### Video Resources
- AWS re:Invent Security sessions (search "SEC" track)
- IAM Policy Evaluation Deep Dive
- Multi-Account Security Strategy
- Threat Detection and Response on AWS
