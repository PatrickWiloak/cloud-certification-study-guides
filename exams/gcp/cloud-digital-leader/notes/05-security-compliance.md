# Security and Compliance in Google Cloud

## Cloud Security Fundamentals

### Shared Responsibility Model
The shared responsibility model defines the security responsibilities between Google Cloud and the customer.

**Google Cloud Responsibilities:**
- **Physical security:** Data center security, hardware security
- **Infrastructure security:** Host OS, hypervisor, network infrastructure
- **Service security:** API endpoints, service-to-service communication
- **Platform security:** Identity infrastructure, key management infrastructure

**Customer Responsibilities:**
- **Data security:** Data encryption, access controls, data classification
- **Identity and access management:** User authentication, authorization policies
- **Application security:** Application code, configuration, runtime security
- **Operating system:** Guest OS patches, updates, configuration (for IaaS)
- **Network security:** Firewall rules, network segmentation, traffic encryption

### Defense in Depth
Google Cloud implements multiple layers of security:

**Layer 1: Physical Security**
- Biometric access controls
- Security guards and surveillance
- Hardware destruction procedures
- Secure hardware design

**Layer 2: Infrastructure Security**
- Custom-designed servers and networking equipment
- Cryptographic signatures for hardware and software
- Secure boot process
- Regular security audits

**Layer 3: Network Security**
- Private global network
- DDoS protection
- Network segmentation
- Encrypted communication

**Layer 4: Platform Security**
- Identity and access management
- Service isolation
- Encryption key management
- Security monitoring

**Layer 5: Application Security**
- Secure coding practices
- Application-level encryption
- Security testing
- Vulnerability management

## Identity and Access Management (IAM)

### Core Concepts
**Principals:** Who can access resources
- **Users:** Individual people with Google accounts
- **Service accounts:** Applications and compute instances
- **Groups:** Collection of users and service accounts
- **Domains:** All users in a Google Workspace or Cloud Identity domain

**Resources:** What can be accessed
- **Projects:** Fundamental organizing entity
- **Folders:** Grouping of projects
- **Organizations:** Top-level container
- **Individual resources:** VMs, databases, storage buckets

**Roles:** What access is granted
- **Basic roles:** Broad access levels (Owner, Editor, Viewer)
- **Predefined roles:** Service-specific roles with granular permissions
- **Custom roles:** User-defined roles with specific permissions

### IAM Best Practices
**Principle of Least Privilege:**
- Grant minimum permissions necessary
- Regular access reviews and cleanup
- Use predefined roles when possible
- Avoid basic roles in production

**Service Account Security:**
- Use service accounts for applications
- Rotate service account keys regularly
- Use Workload Identity for GKE
- Monitor service account usage

**Access Control:**
- Use groups for user management
- Implement conditional access policies
- Enable multi-factor authentication
- Regular audit of permissions

### Advanced IAM Features
**Conditional Access:**
- Time-based access controls
- Location-based restrictions
- Device security requirements
- Resource-specific conditions

**Workload Identity:**
- Secure way for GKE pods to access GCP services
- No service account key management
- Automatic credential rotation
- Fine-grained access control

## Data Protection and Encryption

### Encryption at Rest
**Default Encryption:**
- All data encrypted by default
- AES-256 encryption standard
- Automatic key rotation
- No performance impact

**Customer-Managed Encryption Keys (CMEK):**
- Customer controls encryption keys
- Use Cloud Key Management Service (KMS)
- Granular key access controls
- Audit trail for key usage

**Customer-Supplied Encryption Keys (CSEK):**
- Customer provides encryption keys
- Customer manages key lifecycle
- Maximum control over encryption
- Higher operational complexity

### Encryption in Transit
**Network-Level Encryption:**
- TLS encryption for all communication
- Perfect Forward Secrecy
- Certificate management
- Protocol security

**Application-Level Encryption:**
- End-to-end encryption
- Client-side encryption libraries
- Field-level encryption
- Application-controlled keys

### Key Management
**Cloud Key Management Service (KMS):**
- Centralized key management
- Hardware security modules (HSMs)
- Key versioning and rotation
- Integration with GCP services

**Key Security Features:**
- Separation of duties
- Audit logging
- Cryptographic attestation
- Compliance certifications

## Network Security

### Virtual Private Cloud (VPC) Security
**Network Isolation:**
- Private IP address spaces
- Subnet-level security controls
- VPC peering for secure communication
- Shared VPC for centralized management

**Firewall Rules:**
- Stateful firewall protection
- Tag-based and service account-based rules
- Hierarchical firewall policies
- Default deny-all ingress

**Private Google Access:**
- Access Google services without external IPs
- Enhanced security for internal resources
- Reduced network exposure
- Cost optimization

### Advanced Network Security
**Cloud Armor:**
- DDoS protection
- Web Application Firewall (WAF)
- Adaptive protection
- Custom security policies

**VPC Flow Logs:**
- Network traffic visibility
- Security monitoring
- Compliance reporting
- Troubleshooting and optimization

**Private Service Connect:**
- Secure access to Google services
- Service-specific VPC endpoints
- Private IP connectivity
- Centralized access control

## Security Monitoring and Operations

### Cloud Security Command Center
**Centralized Security Management:**
- Asset inventory and discovery
- Security finding aggregation
- Risk assessment and prioritization
- Compliance monitoring

**Security Insights:**
- Misconfigurations detection
- Vulnerability assessment
- Anomaly detection
- Threat intelligence integration

**Integration Capabilities:**
- Third-party security tools
- SIEM integration
- Custom security findings
- API-based automation

### Cloud Logging and Monitoring
**Security Logging:**
- Admin activity logs
- Data access logs
- System event logs
- VPC flow logs

**Security Monitoring:**
- Real-time alerting
- Anomaly detection
- Threat detection
- Compliance monitoring

**Incident Response:**
- Automated response workflows
- Investigation tools
- Forensic capabilities
- Recovery procedures

## Compliance and Governance

### Compliance Certifications
**International Standards:**
- **ISO 27001:** Information security management
- **ISO 27017:** Cloud security controls
- **ISO 27018:** Cloud privacy protection
- **SOC 1/2/3:** Service organization controls

**Industry-Specific:**
- **PCI DSS:** Payment card industry
- **HIPAA:** Healthcare information
- **FedRAMP:** US government
- **FISMA:** Federal information systems

**Regional Compliance:**
- **GDPR:** European data protection
- **CCPA:** California privacy rights
- **PIPEDA:** Canadian privacy law
- **LGPD:** Brazilian data protection

### Data Governance
**Data Classification:**
- Sensitive data identification
- Data labeling and tagging
- Access control based on classification
- Retention and disposal policies

**Data Residency:**
- Geographic data storage controls
- Regional service deployment
- Cross-border data transfer restrictions
- Sovereignty requirements

**Data Lineage:**
- Data source tracking
- Transformation history
- Access audit trails
- Impact analysis

### Organizational Controls
**Policy and Procedures:**
- Security policy framework
- Standard operating procedures
- Change management processes
- Incident response procedures

**Risk Management:**
- Risk assessment methodologies
- Risk monitoring and reporting
- Mitigation strategies
- Business continuity planning

**Audit and Assurance:**
- Internal audit programs
- External audit coordination
- Compliance reporting
- Continuous monitoring

## Security Best Practices

### Application Security
**Secure Development:**
- Security by design principles
- Secure coding practices
- Code review processes
- Security testing integration

**Runtime Protection:**
- Web Application Firewall (WAF)
- API security controls
- Input validation
- Output encoding

**Vulnerability Management:**
- Regular security scanning
- Patch management processes
- Vulnerability assessment
- Penetration testing

### Infrastructure Security
**Configuration Management:**
- Security baseline configurations
- Configuration drift detection
- Automated compliance checking
- Infrastructure as code security

**Access Controls:**
- Multi-factor authentication
- Privileged access management
- Regular access reviews
- Just-in-time access

**Monitoring and Detection:**
- Security information and event management (SIEM)
- User and entity behavior analytics (UEBA)
- Threat hunting
- Incident response automation

### Data Security
**Data Protection:**
- Data encryption strategies
- Backup and recovery procedures
- Data loss prevention (DLP)
- Privacy impact assessments

**Access Management:**
- Data access controls
- Attribute-based access control (ABAC)
- Data masking and anonymization
- Audit logging

## Emergency Response and Business Continuity

### Incident Response
**Preparation:**
- Incident response plan development
- Team roles and responsibilities
- Communication procedures
- Tool and resource preparation

**Detection and Analysis:**
- Security monitoring and alerting
- Incident classification and prioritization
- Evidence collection and preservation
- Impact assessment

**Containment and Recovery:**
- Incident containment strategies
- System isolation procedures
- Recovery and restoration
- Lessons learned and improvement

### Business Continuity
**Disaster Recovery:**
- Recovery time objectives (RTO)
- Recovery point objectives (RPO)
- Backup and restoration procedures
- Geographic redundancy

**High Availability:**
- Multi-zone deployments
- Load balancing and failover
- Health monitoring
- Automatic scaling

**Crisis Management:**
- Communication plans
- Stakeholder notification
- Media relations
- Business impact minimization