# Security and Compliance - GCP Professional Cloud Architect

## Overview

Security and compliance architecture for Google Cloud, covering IAM, network security, data protection, compliance frameworks, and security best practices for the Professional Cloud Architect certification.

## Key Topics

1. **Identity and Access Management** - Authentication, authorization, service accounts, organization policies
2. **Network Security** - VPC security, firewall rules, Cloud Armor, private connectivity
3. **Data Protection** - Encryption, DLP, key management, data classification
4. **Compliance** - GDPR, HIPAA, PCI DSS, SOC 2, regulatory requirements
5. **Security Operations** - Monitoring, incident response, vulnerability management

## IAM Architecture

### IAM Best Practices
- Use groups instead of individual users
- Apply least privilege principle
- Use predefined roles when possible
- Create custom roles for specific needs
- Implement organization policies
- Regular access reviews
- Use service accounts for applications
- Enable Cloud Audit Logs

### Service Account Patterns
**Application Identity**: Use Workload Identity for GKE, not service account keys
**Cross-Project Access**: Create service accounts in service project, grant access to resources
**Automation**: Use service accounts for CI/CD, infrastructure automation
**Key Management**: Rotate keys regularly, minimize key distribution

### Organization Policies
- Restrict resource sharing
- Enforce VM external IP constraints
- Require OS Login
- Restrict service account key creation
- Define allowed locations
- Enforce uniform bucket-level access

## Network Security

### Defense in Depth Layers
1. **Perimeter Security**: Cloud Armor, DDoS protection
2. **Network Segmentation**: VPCs, subnets, firewall rules
3. **Private Connectivity**: Private Google Access, VPC Service Controls
4. **Application Security**: IAP, Binary Authorization
5. **Data Protection**: Encryption, DLP

### VPC Security Controls
- VPC Service Controls for API perimeter security
- Private Service Connect for SaaS access
- Shared VPC for centralized management
- Firewall rules using service accounts
- Hierarchical firewall policies
- VPC Flow Logs for monitoring
- Packet Mirroring for analysis

### Cloud Armor
- DDoS protection for global load balancers
- WAF rules for application protection
- Rate limiting and throttling
- Geo-based access control
- IP allow/deny lists
- OWASP Top 10 protection
- Custom rules based on request attributes

## Data Protection

### Encryption Strategy
**Data at Rest**:
- Default encryption (Google-managed keys)
- CMEK (Customer-managed encryption keys)
- CSEK (Customer-supplied encryption keys)
- Key rotation policies
- Regional key storage

**Data in Transit**:
- TLS for external connections
- Google Front End termination
- BoringSSL for internal traffic
- VPN for hybrid connectivity

### Cloud KMS Architecture
- Key ring organization by environment/application
- Automatic key rotation
- Key versions and lifecycle management
- IAM permissions on keys
- Regional key location
- Hardware security modules (HSM)

### Data Loss Prevention (DLP)
- Sensitive data discovery
- Data classification
- Redaction and de-identification
- Inspection templates
- De-identification templates
- Job triggers for automation
- Integration with Cloud Storage, BigQuery

## Compliance Architecture

### Compliance Frameworks

**GDPR (General Data Protection Regulation)**:
- Data residency controls (regional resources)
- Encryption at rest and in transit
- Access controls and audit logs
- Data deletion capabilities
- DLP for PII detection
- Consent management
- Data processing agreements

**HIPAA (Health Insurance Portability and Accountability Act)**:
- HIPAA-eligible services
- Business Associate Agreement (BAA)
- Encryption requirements
- Access controls and audit trails
- Breach notification procedures
- VPC Service Controls for PHI

**PCI DSS (Payment Card Industry Data Security Standard)**:
- Network segmentation
- Encryption of cardholder data
- Access control measures
- Security monitoring and testing
- Vulnerability management
- Incident response plan

**SOC 2 (Service Organization Control 2)**:
- Security monitoring
- Change management
- Availability controls
- Confidentiality measures
- Processing integrity

### Compliance Best Practices
1. **Data Residency**: Use regional resources, organization policies
2. **Audit Logging**: Enable all audit log types, export to secure location
3. **Access Controls**: Implement strict IAM, regular reviews
4. **Encryption**: CMEK for sensitive data, key rotation
5. **Monitoring**: Continuous compliance monitoring, automated checks
6. **Documentation**: Maintain architecture and security documentation
7. **Incident Response**: Documented procedures, regular testing
8. **Third-Party Audits**: Regular security assessments

## Security Monitoring

### Security Command Center
- Asset discovery and inventory
- Vulnerability scanning
- Threat detection
- Security Health Analytics
- Web Security Scanner
- Event Threat Detection
- Centralized security dashboard

### Cloud Audit Logs
**Admin Activity**: API calls that modify configuration
**Data Access**: Read operations on user data (optional)
**System Events**: Google-initiated actions
**Policy Denied**: Requests denied by security policies

### Monitoring and Alerting
- Log-based metrics for security events
- Alert policies for suspicious activity
- Integration with SIEM systems
- Real-time threat detection
- Automated incident response
- Security dashboards

## Best Practices

### Security Architecture
1. **Zero Trust Model**: Never trust, always verify
2. **Defense in Depth**: Multiple security layers
3. **Least Privilege**: Minimum necessary access
4. **Segregation of Duties**: Separate critical roles
5. **Encryption Everywhere**: Data at rest and in transit
6. **Security by Design**: Build security into architecture
7. **Continuous Monitoring**: Real-time security visibility
8. **Incident Preparedness**: Documented response procedures

### Identity Management
1. Use Cloud Identity or Google Workspace
2. Enforce multi-factor authentication
3. Implement context-aware access (BeyondCorp)
4. Use groups for access management
5. Regular access reviews and certifications
6. Service account lifecycle management
7. Avoid service account keys when possible
8. Implement break-glass procedures

## Common Scenarios

**Scenario**: Multi-tier application with PCI DSS compliance
**Solution**: VPC Service Controls perimeter, Cloud Armor, CMEK encryption, isolated network segments, strict IAM, comprehensive audit logging

**Scenario**: Healthcare application with HIPAA requirements
**Solution**: BAA-eligible services, CMEK, VPC Service Controls, private connectivity, detailed access logging, DLP for PHI

**Scenario**: Global SaaS with data residency requirements
**Solution**: Regional resource placement, organization policies restricting locations, encryption with regional keys, compliance monitoring

**Scenario**: Hybrid environment with zero trust security
**Solution**: Identity-Aware Proxy, BeyondCorp, context-aware access, Workload Identity, VPC Service Controls, comprehensive logging

## Study Tips

1. **Understand compliance frameworks**: GDPR, HIPAA, PCI DSS requirements
2. **Practice IAM design**: Complex role hierarchies, organization policies
3. **Network security**: VPC Service Controls, private connectivity patterns
4. **Encryption strategies**: When to use CMEK vs. default encryption
5. **Security Command Center**: Features and capabilities
6. **Incident response**: Security event handling procedures
7. **DLP implementation**: Sensitive data protection patterns

## Additional Resources

- [Security Best Practices](https://cloud.google.com/security/best-practices)
- [Compliance Resources](https://cloud.google.com/security/compliance)
- [Cloud Security Command Center](https://cloud.google.com/security-command-center)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls)
- [BeyondCorp](https://cloud.google.com/beyondcorp)
