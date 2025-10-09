# Microsoft Security and Compliance Solutions

## Overview
Microsoft provides comprehensive security and compliance solutions across Azure and Microsoft 365, including threat protection, information protection, compliance management, and unified security operations through Microsoft Defender and Microsoft Purview services.

## Basic Security Capabilities in Azure

### Microsoft Defender for Cloud

#### Overview
- **Formerly:** Azure Security Center + Azure Defender
- **Purpose:** Cloud security posture management (CSPM) and cloud workload protection (CWP)
- **Coverage:** Azure, on-premises, multi-cloud (AWS, GCP)

#### Key Features

**Secure Score:**
- Security posture assessment
- Recommendations to improve security
- Prioritized by impact
- Track progress over time
- Industry benchmarks

**Security Recommendations:**
- Actionable security improvements
- Remediations for vulnerabilities
- Quick fix options for simple fixes
- Compliance mapping

**Security Alerts:**
- Threat detection across resources
- Investigation tools
- Incident response guidance
- Integration with Azure Sentinel

**Regulatory Compliance Dashboard:**
- Map to compliance standards
- Azure Security Benchmark
- PCI DSS, ISO 27001, HIPAA, etc.
- Compliance score and reporting

#### Defender Plans
**Free tier (included):**
- Secure Score
- Security policy and recommendations
- Azure Security Benchmark

**Paid Defender plans:**
- Defender for Servers
- Defender for App Service
- Defender for Storage
- Defender for SQL
- Defender for Kubernetes
- Defender for container registries
- Defender for Key Vault
- Defender for Resource Manager
- Defender for DNS

### Azure Network Security

#### Network Security Groups (NSGs)
- **Purpose:** Filter network traffic to/from Azure resources
- **Rules contain:**
  - Priority (100-4096, lower = higher priority)
  - Source/destination (IP, CIDR, service tag, ASG)
  - Protocol (TCP, UDP, ICMP, Any)
  - Port ranges
  - Action (Allow/Deny)

**Default rules:**
- Allow VNet inbound/outbound
- Allow Azure Load Balancer inbound
- Deny all other inbound
- Allow outbound to internet

#### Azure Firewall
- **Purpose:** Managed, cloud-based network security
- **Features:**
  - Stateful firewall
  - Built-in high availability
  - Threat intelligence filtering
  - Application and network rules
  - FQDN filtering
  - Centralized logging
  - Multiple public IP addresses

**Tiers:**
- **Basic:** Small/medium businesses, <250 Mbps
- **Standard:** Enterprise workloads
- **Premium:** Advanced threat protection, TLS inspection

#### Web Application Firewall (WAF)
- **Purpose:** Protect web applications from common exploits
- **Protection against:**
  - SQL injection
  - Cross-site scripting (XSS)
  - OWASP Top 10 vulnerabilities
  - DDoS attacks

**Available on:**
- Azure Application Gateway
- Azure Front Door
- Azure CDN

### DDoS Protection

#### Azure DDoS Protection

**DDoS Infrastructure Protection (Basic):**
- Free, automatically enabled
- Protects Azure infrastructure
- Always-on monitoring

**DDoS Network Protection (Standard):**
- Paid service
- Application-specific protection
- Real-time attack metrics
- Attack analytics and logs
- Rapid response support
- Cost protection (during attacks)

**Protection capabilities:**
- Volumetric attacks
- Protocol attacks
- Resource layer attacks
- Adaptive tuning
- Always-on traffic monitoring

### Azure Key Vault

#### Overview
- **Purpose:** Securely store and access secrets
- **Benefits:**
  - Centralized secrets management
  - Secure key management
  - Certificate management
  - Hardware security module (HSM) support

#### What You Can Store

**Secrets:**
- Passwords
- Connection strings
- API keys
- Tokens
- Any sensitive string

**Keys:**
- Encryption keys
- RSA, EC keys
- Software or HSM-protected
- Key rotation support

**Certificates:**
- TLS/SSL certificates
- Auto-renewal support
- Certificate authorities integration

#### Access Control
- **Authentication:** Azure AD
- **Authorization:** RBAC or access policies
- **Monitoring:** Diagnostic logs
- **Networking:** Private endpoints, service endpoints

## Microsoft Defender Suite

### Microsoft Defender for Endpoint

#### Overview
- **Purpose:** Enterprise endpoint security platform
- **Coverage:** Windows, macOS, Linux, iOS, Android, Windows Server

#### Capabilities

**Threat & Vulnerability Management:**
- Discover vulnerabilities
- Prioritize remediation
- Track security baselines

**Attack Surface Reduction:**
- Hardware-based isolation
- Application control
- Exploit protection
- Network protection
- Web protection

**Next-Generation Protection:**
- Antivirus and antimalware
- Cloud-delivered protection
- Behavioral analysis
- Machine learning

**Endpoint Detection and Response (EDR):**
- Advanced threat detection
- Behavioral analytics
- Automated investigation
- Response actions (isolate, block, etc.)

**Automated Investigation and Remediation:**
- AI-powered investigation
- Automatic remediation
- Playbooks for common threats
- Reduces alert fatigue

**Microsoft Threat Experts:**
- Managed threat hunting
- Expert-on-demand
- Proactive notifications

### Microsoft Defender for Office 365

#### Overview
- **Purpose:** Protect email and collaboration tools
- **Coverage:** Exchange Online, SharePoint, OneDrive, Teams

#### Protection Capabilities

**Safe Attachments:**
- Detonates attachments in sandbox
- Checks for malicious behavior
- Blocks malware before delivery
- Protects SharePoint/OneDrive files

**Safe Links:**
- Time-of-click protection
- Rewrites URLs in emails
- Checks reputation before access
- Protects against phishing

**Anti-phishing:**
- Machine learning models
- Impersonation protection
- Spoof intelligence
- Advanced anti-spam

**Anti-malware:**
- Multiple anti-malware engines
- Zero-day protection
- Attachment filtering
- Common attachment blocking

#### Plans

**Plan 1:**
- Safe Attachments
- Safe Links
- Anti-phishing policies

**Plan 2 (includes Plan 1 plus):**
- Threat investigation
- Threat trackers
- Attack simulation training
- Automated investigation and response

### Microsoft Defender for Identity

#### Overview
- **Purpose:** Detect identity-based threats
- **Monitors:** On-premises Active Directory signals
- **Deployment:** Sensor on domain controllers

#### Detection Capabilities
- Compromised credentials
- Lateral movement attempts
- Domain dominance activities
- Pass-the-hash attacks
- Pass-the-ticket attacks
- Reconnaissance activities
- Suspicious authentication patterns

#### Features
- **Real-time detection and alerting**
- **User and entity behavior analytics (UEBA)**
- **Investigation tools**
- **Integration with Defender for Cloud Apps**
- **Remediation recommendations**

### Microsoft Defender for Cloud Apps

#### Overview
- **Purpose:** Cloud Access Security Broker (CASB)
- **Coverage:** SaaS applications (Microsoft 365, third-party apps)

#### CASB Capabilities

**Discover and control shadow IT:**
- Discover cloud apps in use
- Risk assessment of apps
- App usage patterns
- Unsanctioned app detection

**Protect sensitive information:**
- Data loss prevention (DLP)
- Information protection policies
- Sensitive data discovery
- Classification and labeling

**Protect against threats:**
- Anomaly detection
- Behavioral analytics
- Malware detection
- Suspicious activity alerts

**Assess compliance:**
- Compliance assessments
- Configuration checks
- Industry standards mapping

#### Integration
- **Conditional Access App Control:** Real-time monitoring and control
- **Microsoft Defender integration:** Unified security
- **Azure Information Protection:** Label and protect data

### Microsoft 365 Defender Portal

#### Overview
- **Purpose:** Unified security operations center
- **URL:** security.microsoft.com
- **Integration:** All Defender products

#### Capabilities

**Incidents and Alerts:**
- Correlated alerts across products
- Unified incident view
- Investigation tools
- Response actions

**Threat Analytics:**
- Emerging threats
- Expert analysis
- Impact assessment
- Mitigation guidance

**Hunting:**
- Advanced hunting queries
- KQL (Kusto Query Language)
- Cross-product data
- Custom detection rules

**Action Center:**
- Pending actions
- Action history
- Automated remediation
- Manual approvals

**Reports:**
- Security trends
- Threat protection
- Device health
- User accounts

## Microsoft Sentinel

### Overview
- **Purpose:** Cloud-native SIEM (Security Information and Event Management) and SOAR (Security Orchestration, Automated Response)
- **Scope:** Enterprise-wide security analytics

### Key Capabilities

#### Data Connectors
- **Microsoft services:** Azure, Microsoft 365, Defender products
- **Third-party:** Palo Alto, Cisco, AWS, GCP
- **Industry standards:** Syslog, CEF, REST API
- **Custom:** Logic Apps, Functions

#### Analytics Rules
- **Scheduled queries:** Run on schedule, detect patterns
- **Microsoft security:** Alerts from Defender, etc.
- **Fusion:** ML-based multi-stage attack detection
- **Anomaly:** Behavioral analytics
- **Custom:** KQL-based detection rules

#### Incidents and Investigations
- **Incident creation:** From alert groups
- **Investigation graph:** Visual entity relationships
- **Automation:** Playbooks for response
- **Collaboration:** Comments, tagging, assignment

#### Hunting
- **KQL queries:** Search across data
- **Hunting queries:** Pre-built searches
- **Bookmarks:** Save interesting results
- **Notebooks:** Jupyter notebooks for analysis

#### Automation (SOAR)
- **Playbooks:** Logic Apps for automation
- **Automated responses:** Block IPs, disable users, etc.
- **Incident enrichment:** Get additional context
- **Integration:** ServiceNow, Jira, Teams

### Workbooks
- **Purpose:** Visualize and monitor data
- **Built-in templates:** Azure AD, Office 365, etc.
- **Custom workbooks:** Create your own
- **Interactive:** Drill-down and filtering

## Microsoft Purview Compliance

### Compliance Management

#### Microsoft Purview Compliance Portal
- **URL:** compliance.microsoft.com
- **Purpose:** Manage compliance across Microsoft 365
- **Capabilities:**
  - Compliance Manager
  - Data lifecycle management
  - Information protection
  - Insider risk management
  - eDiscovery

#### Compliance Manager
- **Purpose:** Assess and improve compliance posture
- **Compliance Score:** Overall compliance measurement
- **Improvement Actions:** Steps to improve compliance
- **Assessments:** Map to regulations (GDPR, ISO 27001, etc.)
- **Templates:** Pre-built regulatory templates

### Information Protection

#### Microsoft Purview Information Protection

**Sensitivity Labels:**
- **Purpose:** Classify and protect data
- **Application:** Manual or automatic
- **Scope:** Documents, emails, containers (Teams, groups)
- **Protection:**
  - Encryption
  - Content marking (headers, footers, watermarks)
  - Access restrictions
  - Prevent external sharing

**Label Policies:**
- **Default label:** Applied automatically
- **Mandatory labeling:** Require users to label
- **Help links:** Custom guidance
- **Scope:** Users and groups

**Data Classification:**
- **Know your data:** Discover sensitive content
- **Sensitive info types:** Credit cards, SSN, etc.
- **Trainable classifiers:** ML-based classification
- **Content explorer:** View classified content

#### Data Loss Prevention (DLP)

**Purpose:**
- Prevent sensitive data from leaving organization
- Compliance with regulations
- Protect confidential information

**Locations:**
- Exchange Online
- SharePoint Online
- OneDrive for Business
- Teams chats and channels
- Devices (Windows, macOS)
- Non-Microsoft cloud apps

**Policy Components:**
- **Conditions:** What to look for (sensitive info types, labels)
- **Actions:** What to do (block, notify, encrypt)
- **Exceptions:** When not to apply
- **User notifications:** Educate users
- **Incident reports:** Alert admins

#### Records Management

**Purpose:**
- Manage high-value content lifecycle
- Regulatory and legal requirements
- Retention and disposition

**Retention Labels:**
- **Retention period:** How long to keep
- **Disposition:** Delete or review after period
- **Mark as record:** Immutable
- **Regulatory record:** Even stricter controls

**File Plans:**
- **Structure:** Organize retention labels
- **Metadata:** Department, category, etc.
- **Import/export:** Manage at scale

### Insider Risk Management

#### Microsoft Purview Insider Risk Management

**Purpose:**
- Detect risky user activities
- Insider threats and data leaks
- Balance security and privacy

**Risk Indicators:**
- Data exfiltration
- Data leakage to unauthorized apps
- Offensive language
- Security policy violations
- Departing employee activities

**Workflow:**
1. **Configure policies** based on templates
2. **Detect activities** matching risk indicators
3. **Create alerts** for investigation
4. **Investigate** with detailed timelines
5. **Take action** (educate, escalate, etc.)

**Privacy:**
- Pseudonymization of users
- Role-based access to identities
- Audit trails

#### Communication Compliance

**Purpose:**
- Monitor communications for policy violations
- Inappropriate content
- Regulatory compliance

**Detection:**
- Offensive language
- Sensitive information sharing
- Harassment
- Regulatory violations

**Channels:**
- Email (Exchange)
- Teams chats and channels
- Yammer
- Third-party platforms (with connectors)

#### Information Barriers

**Purpose:**
- Prevent communication between groups
- Prevent conflicts of interest
- Regulatory requirements (Chinese Wall)

**Example:**
- Traders can't communicate with analysts
- Legal teams separated by client

### eDiscovery and Audit

#### eDiscovery Solutions

**Content Search:**
- Search across Microsoft 365
- Export search results
- Basic eDiscovery needs

**eDiscovery (Standard):**
- Cases and custodians
- Legal holds
- Search and export
- Included in E3 licenses

**eDiscovery (Premium):**
- Advanced analytics
- Machine learning
- Predictive coding
- Custodian communications
- Review sets with redaction
- Requires E5 license

#### Audit

**Audit (Standard):**
- 90-day retention
- Search audit logs
- Export to CSV
- Included in E1/E3

**Audit (Premium):**
- 1-year retention (10 years with add-on)
- High-value events (mailbox access, etc.)
- Intelligent insights
- API access
- Requires E5

## Best Practices

### Security
1. **Enable Defender for Cloud** for all Azure subscriptions
2. **Implement recommendations** from Secure Score
3. **Use NSGs and Azure Firewall** for network security
4. **Enable DDoS Protection Standard** for public resources
5. **Store secrets in Key Vault** never in code
6. **Deploy Defender for Endpoint** on all devices
7. **Configure Safe Links and Attachments** for email
8. **Use Sentinel** for centralized security monitoring

### Compliance
1. **Use Compliance Manager** to track compliance posture
2. **Implement sensitivity labels** across organization
3. **Configure DLP policies** for sensitive data
4. **Enable audit logging** for all services
5. **Implement retention policies** for data governance
6. **Regular access reviews** for compliance
7. **Train users** on classification and protection

### Information Protection
1. **Classify data** before protecting
2. **Use automatic labeling** where possible
3. **Encrypt sensitive data** with labels
4. **Monitor DLP alerts** and tune policies
5. **Implement least privilege** access
6. **Regular data discovery** scans

## Study Tips

### Key Concepts
- Defender for Cloud vs Defender for Endpoint (cloud vs endpoint)
- SIEM (Sentinel) vs CASB (Defender for Cloud Apps)
- Sensitivity labels vs retention labels
- DLP vs Information Protection
- eDiscovery Standard vs Premium
- Different Defender products and their purposes

### Common Scenarios
1. **Cloud security posture** → Defender for Cloud
2. **Endpoint protection** → Defender for Endpoint
3. **Email security** → Defender for Office 365
4. **Cloud app security** → Defender for Cloud Apps
5. **Centralized SIEM** → Microsoft Sentinel
6. **Data classification** → Sensitivity labels
7. **Prevent data leaks** → DLP policies
8. **Legal hold** → eDiscovery

### Exam Focus
- Purpose of each Defender product
- Sentinel capabilities (SIEM/SOAR)
- Sensitivity labels vs retention labels
- DLP policy components
- Compliance Manager functionality
- Insider risk management capabilities
- eDiscovery editions

### Remember
- Defender for Cloud = Azure resources security
- Defender for Endpoint = Device protection
- Defender for Office 365 = Email & collaboration
- Sentinel = Cloud SIEM/SOAR
- Purview = Compliance and governance
- Sensitivity labels = Classify and protect
- DLP = Prevent data loss
- Compliance Manager = Assess compliance posture
