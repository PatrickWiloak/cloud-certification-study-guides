# AWS Certified Security - Specialty (SCS-C02) Exam Guide

## Exam Overview

The AWS Certified Security - Specialty (SCS-C02) exam validates specialized knowledge and skills in securing AWS workloads. This certification demonstrates expertise in creating and implementing security solutions in the AWS Cloud, including incident response, threat detection, vulnerability management, and compliance.

### Exam Details
- **Exam Code**: SCS-C02
- **Duration**: 170 minutes
- **Format**: Multiple choice and multiple response
- **Number of Questions**: 65 scored questions
- **Passing Score**: 750/1000
- **Cost**: $300 USD
- **Language**: Available in multiple languages
- **Delivery**: Testing center or online proctoring
- **Validity**: 3 years
- **Prerequisites**: 2+ years hands-on experience securing AWS workloads

## Exam Domains

### Domain 1: Threat Detection and Incident Response (14% of scored content)
- Design and implement an incident response plan
- Detect security threats and anomalies by using AWS services
- Respond to compromised resources and workloads

#### Key Focus Areas:
- AWS GuardDuty configuration and response automation
- Security incident response procedures and playbooks
- Forensics and evidence collection in AWS
- Threat intelligence integration and analysis
- Automated threat detection and response
- Security operations center (SOC) implementation

### Domain 2: Security Logging and Monitoring (18% of scored content)
- Design and implement security monitoring and alerting
- Troubleshoot security monitoring and alerting
- Design and implement a logging solution
- Troubleshoot logging solutions

#### Key Focus Areas:
- CloudTrail advanced configuration and analysis
- VPC Flow Logs analysis and monitoring
- CloudWatch security metrics and alarms
- AWS Config for compliance monitoring
- Security Hub centralized findings management
- Log aggregation and correlation strategies

### Domain 3: Infrastructure Security (20% of scored content)
- Design edge security on AWS
- Design and implement a secure network infrastructure
- Troubleshoot a secure network infrastructure
- Design and implement host-based security

#### Key Focus Areas:
- VPC security architecture and network segmentation
- AWS WAF and Shield for application protection
- Network Access Control Lists (NACLs) and Security Groups
- AWS Direct Connect and VPN security
- Host-based security controls and hardening
- Container and serverless security

### Domain 4: Identity and Access Management (16% of scored content)
- Design and implement a scalable authorization and authentication system
- Troubleshoot an authorization and authentication system

#### Key Focus Areas:
- IAM policies, roles, and permission boundaries
- AWS Single Sign-On (SSO) and identity federation
- AWS Cognito for application authentication
- Cross-account access and trust relationships
- Privileged access management and automation
- Identity governance and compliance

### Domain 5: Data Protection and Cryptography (18% of scored content)
- Design and implement key management and use
- Troubleshoot key management
- Design and implement a data encryption solution at rest and in transit

#### Key Focus Areas:
- AWS Key Management Service (KMS) advanced features
- AWS CloudHSM for dedicated key management
- Encryption strategies for various AWS services
- Certificate management with AWS Certificate Manager
- Data loss prevention and data classification
- Secrets management and rotation

### Domain 6: Management and Security Governance (14% of scored content)
- Develop a strategy to centrally deploy and manage AWS security services
- Manage the security lifecycle of AWS resources
- Determine incident response strategies
- Design and implement a security governance strategy

#### Key Focus Areas:
- AWS Organizations security management
- Service Control Policies (SCPs) for governance
- AWS Control Tower for landing zone security
- Compliance frameworks and audit preparation
- Security automation and orchestration
- Risk assessment and management

## Core Security Services Deep Dive

### Identity and Access Management

#### AWS IAM Advanced Features
- **Permission Boundaries**: Delegated administration with maximum permissions
- **Access Analyzer**: Identify resources shared with external entities
- **Credential Reports**: User credential usage and status monitoring
- **Policy Simulator**: Test IAM policies before implementation
- **IAM Roles Anywhere**: Extend IAM roles to on-premises workloads
- **ABAC (Attribute-Based Access Control)**: Fine-grained access control using tags

#### AWS Single Sign-On (AWS SSO)
- **Multi-Account Access**: Centralized access across AWS accounts
- **Application Integration**: SAML 2.0 and custom applications
- **Permission Sets**: Reusable permission collections
- **External Identity Sources**: Active Directory and external IdP integration
- **Audit and Compliance**: Access logging and reporting

#### AWS Cognito Security
- **User Pool Security**: MFA, password policies, and account verification
- **Identity Pool Security**: Federated identity and role mapping
- **Advanced Security Features**: Risk-based authentication and adaptive authentication
- **SAML and OIDC Integration**: Enterprise identity provider federation
- **Custom Authentication Flows**: Lambda triggers for custom logic

### Data Protection and Encryption

#### AWS Key Management Service (KMS)
- **Customer Managed Keys (CMKs)**: Custom key policies and rotation
- **Key Grants**: Temporary, granular permissions for cryptographic operations
- **Multi-Region Keys**: Keys that can be used across multiple AWS regions
- **Key Origin**: AWS KMS, External, and CloudHSM key sources
- **Cross-Account Access**: Sharing encrypted resources across accounts
- **Audit and Compliance**: CloudTrail logging for all key operations

#### AWS CloudHSM
- **Dedicated Hardware**: FIPS 140-2 Level 3 validated HSMs
- **Cluster Management**: High availability and load distribution
- **Integration Patterns**: SSL/TLS termination and database encryption
- **Client SDK**: Application integration and key management
- **Backup and Recovery**: Automated and manual backup strategies
- **Compliance**: Meeting strict regulatory requirements

#### Encryption Strategies
- **Encryption at Rest**: S3, EBS, RDS, and other service encryption
- **Encryption in Transit**: TLS/SSL and VPN encryption
- **Client-Side Encryption**: Application-level encryption patterns
- **Envelope Encryption**: Hierarchical key management
- **Key Rotation**: Automated and manual key rotation strategies
- **Field-Level Encryption**: CloudFront field-level encryption

### Network Security

#### VPC Security Architecture
- **Network Segmentation**: Multi-tier architecture and microsegmentation
- **Security Groups**: Stateful firewall rules and best practices
- **Network ACLs**: Stateless subnet-level controls
- **VPC Flow Logs**: Network traffic analysis and monitoring
- **VPC Endpoints**: Secure service access without internet gateway
- **PrivateLink**: Private connectivity to AWS services and third-party services

#### AWS WAF and Shield
- **Web Application Firewall**: Custom rules and managed rule groups
- **Rate Limiting**: Request rate-based protection
- **Bot Control**: Automated bot detection and mitigation
- **IP Reputation Lists**: Known malicious IP blocking
- **Shield Standard**: Automatic DDoS protection
- **Shield Advanced**: Enhanced DDoS protection with response team

#### AWS Network Firewall
- **Stateful Inspection**: Deep packet inspection and intrusion detection
- **Domain Filtering**: DNS-based filtering and blocking
- **Custom Rules**: Suricata-compatible rule engine
- **Centralized Management**: Multi-VPC firewall management
- **Logging and Monitoring**: Comprehensive traffic and security logging

### Threat Detection and Response

#### Amazon GuardDuty
- **Machine Learning**: Behavioral analysis and anomaly detection
- **Threat Intelligence**: AWS and third-party threat feeds
- **DNS Monitoring**: Malicious domain detection
- **Finding Types**: Comprehensive threat categorization
- **Multi-Account Management**: Centralized threat detection
- **Integration**: Automated response with Lambda and Systems Manager

#### AWS Security Hub
- **Security Standards**: CIS, PCI DSS, and AWS Foundational Security Standard
- **Finding Aggregation**: Centralized security findings management
- **Custom Actions**: Automated remediation workflows
- **Integration**: Third-party security tool integration
- **Compliance Dashboards**: Continuous compliance monitoring
- **Multi-Account Architecture**: Centralized security posture management

#### AWS Inspector
- **Application Assessment**: EC2 and container image vulnerability scanning
- **Agent-Based Scanning**: Comprehensive host-based assessment
- **Network Reachability**: Network configuration analysis
- **CVE Database**: Common vulnerabilities and exposures identification
- **Integration**: CI/CD pipeline integration for continuous assessment
- **Prioritization**: Risk-based vulnerability prioritization

### Logging and Monitoring

#### AWS CloudTrail
- **API Logging**: Comprehensive API call logging across all services
- **Data Events**: S3 object and Lambda function execution logging
- **Insight Events**: Unusual activity pattern detection
- **Multi-Region Logging**: Centralized logging across regions
- **Log File Integrity**: Cryptographic validation of log files
- **Organization Trails**: Account-wide logging with AWS Organizations

#### Amazon CloudWatch Security
- **Custom Metrics**: Security-specific metrics and KPIs
- **Alarms**: Automated alerting on security events
- **Log Insights**: Advanced log querying and analysis
- **Dashboards**: Real-time security monitoring dashboards
- **Cross-Account Monitoring**: Centralized monitoring across accounts
- **Integration**: Automated response and remediation

### Compliance and Governance

#### AWS Config
- **Configuration Compliance**: Resource configuration monitoring
- **Configuration History**: Historical configuration tracking
- **Remediation**: Automated compliance remediation
- **Conformance Packs**: Compliance framework templates
- **Multi-Account Strategy**: Organization-wide compliance monitoring
- **Integration**: Security Hub and other security service integration

#### AWS Control Tower
- **Landing Zone**: Multi-account secure foundation
- **Guardrails**: Preventive and detective governance controls
- **Account Factory**: Automated secure account provisioning
- **Dashboard**: Centralized compliance and governance monitoring
- **Customization**: Custom guardrails and organizational policies
- **Integration**: AWS Organizations and service integration

## Advanced Security Patterns

### Zero Trust Architecture
- **Identity Verification**: Continuous identity and device verification
- **Least Privilege Access**: Minimal access rights assignment
- **Microsegmentation**: Network and application-level segmentation
- **Encrypted Communications**: End-to-end encryption for all communications
- **Monitoring and Analytics**: Continuous monitoring and behavioral analysis
- **Dynamic Policy Enforcement**: Real-time policy adjustment based on risk

### DevSecOps Integration
- **Shift-Left Security**: Early integration of security in development
- **Automated Security Testing**: SAST, DAST, and dependency scanning
- **Infrastructure as Code Security**: CloudFormation and CDK security
- **Container Security**: Image scanning and runtime protection
- **Secrets Management**: Automated secrets rotation and management
- **Compliance as Code**: Automated compliance testing and reporting

### Incident Response Automation
- **Playbook Automation**: Automated incident response workflows
- **Evidence Collection**: Automated forensic data collection
- **Isolation Procedures**: Automated compromised resource isolation
- **Communication**: Automated stakeholder notification
- **Recovery Procedures**: Automated system recovery and validation
- **Lessons Learned**: Post-incident analysis and improvement

### Multi-Account Security Strategy
- **Security Account**: Centralized security tooling and monitoring
- **Logging Account**: Centralized log aggregation and retention
- **Shared Services**: Common security services across accounts
- **Cross-Account Roles**: Secure cross-account access patterns
- **Automation**: Account provisioning and security configuration
- **Governance**: Centralized policy management and enforcement

## Study Strategy

### Prerequisites
- **AWS Associate Certification**: Solid foundation in AWS core services
- **Security Experience**: 2+ years of hands-on security experience
- **Compliance Knowledge**: Understanding of compliance frameworks
- **Networking Fundamentals**: Strong networking and security concepts

### Phase 1: Security Fundamentals (3-4 weeks)
1. **Identity and Access Management**
   - Advanced IAM features and patterns
   - Federation and single sign-on
   - Cross-account access strategies
   - Privileged access management

2. **Data Protection**
   - Encryption strategies and implementation
   - Key management best practices
   - Secrets management and rotation
   - Data classification and handling

### Phase 2: Infrastructure Security (3-4 weeks)
1. **Network Security**
   - VPC security architecture
   - Network segmentation strategies
   - Web application protection
   - Network monitoring and analysis

2. **Compute Security**
   - EC2 security hardening
   - Container and serverless security
   - Host-based security controls
   - Vulnerability management

### Phase 3: Detection and Response (2-3 weeks)
1. **Threat Detection**
   - GuardDuty configuration and tuning
   - Security monitoring and alerting
   - Threat intelligence integration
   - Behavioral analysis and anomaly detection

2. **Incident Response**
   - Incident response planning and procedures
   - Automated response and remediation
   - Forensics and evidence collection
   - Post-incident analysis and improvement

### Phase 4: Governance and Compliance (2-3 weeks)
1. **Security Governance**
   - Multi-account security strategy
   - Policy management and enforcement
   - Compliance monitoring and reporting
   - Risk assessment and management

2. **Automation and Orchestration**
   - Security automation patterns
   - Infrastructure as code security
   - DevSecOps integration
   - Continuous compliance monitoring

## Hands-on Lab Recommendations

### Essential Security Labs
1. **Multi-Account Security Architecture**
   - AWS Organizations and Control Tower setup
   - Cross-account IAM roles and policies
   - Centralized logging and monitoring
   - Security automation across accounts

2. **Advanced Threat Detection**
   - GuardDuty custom threat intelligence
   - Security Hub integration and automation
   - Incident response automation
   - Forensics data collection and analysis

3. **Data Protection Implementation**
   - End-to-end encryption implementation
   - KMS and CloudHSM integration
   - Secrets management automation
   - Data classification and protection

4. **Network Security Architecture**
   - Zero trust network implementation
   - WAF and Shield configuration
   - Network segmentation and monitoring
   - VPC security best practices

## Exam Preparation Tips

### Study Resources
- **AWS Security Whitepapers**: Comprehensive security guidance
- **AWS Well-Architected Security Pillar**: Security design principles
- **Security Partner Solutions**: Third-party security tool integration
- **Compliance Guides**: Industry-specific compliance guidance
- **Re:Invent Security Sessions**: Latest security features and best practices

### Practical Experience
- **Security Projects**: Lead or participate in security implementations
- **Incident Response**: Experience with real security incidents
- **Compliance Audits**: Participate in compliance audit preparations
- **Vulnerability Management**: Hands-on vulnerability assessment and remediation
- **Security Automation**: Build automated security solutions

### Exam Strategy
- **Scenario Analysis**: Identify security requirements and constraints
- **Defense in Depth**: Consider multiple layers of security controls
- **Compliance Requirements**: Factor in regulatory and industry requirements
- **Cost Considerations**: Balance security and cost effectiveness
- **Operational Impact**: Consider operational complexity and maintenance

## Common Exam Scenarios

### Enterprise Security Implementation
- Multi-account security architecture design
- Identity federation and access management
- Data classification and protection strategies
- Compliance framework implementation

### Incident Response and Forensics
- Automated threat detection and response
- Security incident investigation procedures
- Evidence collection and chain of custody
- Post-incident analysis and remediation

### Application Security
- Web application protection strategies
- API security and authentication
- Container and serverless security
- DevSecOps pipeline integration

### Compliance and Governance
- Regulatory compliance implementation
- Security policy automation and enforcement
- Risk assessment and management
- Audit preparation and documentation

## Next Steps After Certification

### Career Advancement
- Cloud security architect roles
- Security engineer and DevSecOps roles
- Compliance and governance specialist
- Security consulting and advisory roles

### Continuous Learning
- Other AWS specialty certifications
- Industry security certifications (CISSP, CISM, etc.)
- Security research and threat intelligence
- Emerging security technologies and practices

### Professional Development
- Security community participation
- Conference speaking and content creation
- Open-source security project contribution
- Mentoring and knowledge sharing