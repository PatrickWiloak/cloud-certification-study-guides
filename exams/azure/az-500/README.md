# Microsoft Azure Security Engineer Associate (AZ-500)

## Exam Overview

The Azure Security Engineer Associate certification validates skills and knowledge to implement security controls and threat protection, manage identity and access, and protect data, applications, and networks in cloud and hybrid environments as part of an end-to-end infrastructure.

**Exam Details:**
- **Exam Code:** AZ-500
- **Duration:** 180 minutes
- **Number of Questions:** 40-60 questions
- **Passing Score:** 700 out of 1000
- **Question Types:** Multiple choice, multiple select, drag and drop, hot area, case studies, labs
- **Cost:** $165 USD
- **Prerequisites:** Experience with Azure administration and development, security concepts

## Exam Domains

### 1. Manage Identity and Access (25-30%)
- **Configure Azure Active Directory for Workloads**
  - Create App registrations
  - Configure App registration permission scopes
  - Manage App registration permission consent
  - Configure Multi-Factor Authentication settings
  - Manage Azure AD directory groups
  - Manage Azure AD users
  - Install and configure Azure AD Connect
  - Configure authentication methods
  - Implement Conditional Access policies
  - Configure Azure AD identity protection

- **Configure Azure AD Privileged Identity Management**
  - Monitor privileged access for Azure AD Privileged Identity Management (PIM)
  - Configure Access Reviews
  - Activate Privileged Identity Management

- **Configure Azure Tenant Security**
  - Transfer Azure subscriptions between Azure AD tenants
  - Manage API access to Azure subscriptions and resources

### 2. Implement Platform Protection (35-40%)
- **Implement Advanced Network Security**
  - Secure the connectivity of hybrid networks
  - Secure the connectivity of virtual networks
  - Create and configure Azure Firewall
  - Create and configure Azure Firewall Manager
  - Create and configure Azure Application Gateway
  - Create and configure Azure Front Door
  - Create and configure Web Application Firewall (WAF)
  - Configure a resource firewall
  - Configure Network Security Groups (NSGs)
  - Implement Azure Service Endpoints
  - Implement Azure Private Endpoints
  - Implement Azure Private Links
  - Implement Azure DDoS Protection

- **Configure Advanced Security for Compute**
  - Configure Azure Endpoint Protection for VMs
  - Implement and manage security updates for VMs
  - Configure security for different types of containers
  - Implement vulnerability management for VMs and containers
  - Configure isolation for AKS
  - Configure security for serverless compute
  - Configure security for an Azure App Service
  - Configure encryption at rest
  - Configure encryption in transit

### 3. Manage Security Operations (25-30%)
- **Configure Centralized Policy Management**
  - Configure a custom security policy
  - Create a policy initiative
  - Configure security settings and auditing by using Azure Policy

- **Configure and Manage Threat Protection**
  - Configure Microsoft Defender for Servers
  - Evaluate vulnerability scans from Microsoft Defender for Servers
  - Configure Microsoft Defender for SQL
  - Use the Microsoft Threat Modeling Tool

- **Configure and Manage Security Monitoring and Automation**
  - Monitor security events by using Azure Monitor
  - Configure data retention policies
  - Configure Microsoft Sentinel
  - Assess and recommend cloud workload protection
  - Design and configure workflow automation

### 4. Secure Data and Applications (20-25%)
- **Configure Security for Storage**
  - Configure access control for storage accounts
  - Manage life cycle for storage account access keys
  - Configure Azure AD authentication for Azure Storage
  - Configure access control for Azure Files
  - Configure access control for Azure Blob Storage
  - Configure encryption for Azure Storage
  - Configure encryption for Azure SQL Database
  - Implement Azure SQL Database Always Encrypted

- **Configure Security for Databases**
  - Enable database authentication by using Azure AD
  - Configure database auditing
  - Configure dynamic data masking
  - Implement database threat detection

- **Configure and Manage Key Vault**
  - Manage access to Key Vault
  - Manage permissions to secrets, certificates, and keys
  - Configure RBAC usage in Azure Key Vault
  - Manage certificates
  - Manage secrets
  - Configure key rotation
  - Backup and restore of Key Vault items

## Skills Measured in Detail

### Identity Security
- **Azure AD Configuration:** User and group management, authentication methods
- **Conditional Access:** Risk-based policies, device compliance, location-based access
- **Privileged Identity Management:** Just-in-time access, access reviews, role management
- **Identity Protection:** Risk detection, automated remediation, sign-in policies

### Network Security
- **Perimeter Security:** Azure Firewall, WAF, DDoS Protection, Front Door
- **Network Segmentation:** NSGs, service endpoints, private endpoints
- **Hybrid Connectivity:** VPN Gateway, ExpressRoute, site-to-site security
- **Application Security:** Application Gateway, API Management security

### Compute Security
- **VM Security:** Endpoint protection, patch management, disk encryption
- **Container Security:** AKS security, container scanning, runtime protection
- **Serverless Security:** Function app security, Logic Apps security
- **Application Security:** App Service security, certificate management

### Data Protection
- **Storage Security:** Access controls, encryption, lifecycle management
- **Database Security:** SQL security, encryption, auditing, threat detection
- **Key Management:** Key Vault operations, certificate management, HSM
- **Data Classification:** Sensitivity labels, data loss prevention

### Security Operations
- **Monitoring:** Security events, logging, alerting, dashboard creation
- **Threat Detection:** Microsoft Defender for Cloud, Sentinel, threat hunting
- **Incident Response:** Playbooks, automation, investigation procedures
- **Compliance:** Policy management, regulatory compliance, auditing

## Study Tips

### Recommended Study Timeline: 10-12 weeks
1. **Weeks 1-2:** Azure AD and identity management fundamentals
2. **Weeks 3-4:** Privileged Identity Management and Conditional Access
3. **Weeks 5-6:** Network security and perimeter protection
4. **Weeks 7-8:** Compute and application security
5. **Weeks 9-10:** Data protection and Key Vault management
6. **Weeks 11-12:** Security operations and monitoring

### Key Study Resources
- **Microsoft Learn Learning Paths:**
  - AZ-500: Manage identity and access in Azure Active Directory
  - AZ-500: Implement platform protection
  - AZ-500: Manage security operations
  - AZ-500: Secure data and applications

### Prerequisites Knowledge
- **Azure Administration:** AZ-104 level knowledge recommended
- **Security Fundamentals:** SC-900 knowledge helpful
- **Networking Concepts:** Understanding of TCP/IP, DNS, firewalls
- **Development Experience:** Basic understanding of application security

### Hands-on Practice Requirements
- **Security Tools:** Experience with Azure security services
- **Scripting:** PowerShell, Azure CLI for security automation
- **Monitoring:** Azure Monitor, Log Analytics, Microsoft Sentinel
- **Compliance:** Understanding of regulatory requirements

### Exam Strategy
- **Security-first Mindset:** Always consider security implications
- **Defense in Depth:** Understand layered security approaches
- **Incident Response:** Know how to investigate and respond to threats
- **Compliance Focus:** Understand regulatory and compliance requirements

### Common Gotchas
- **Conditional Access:** Policy evaluation order and exclusions
- **RBAC vs Azure AD Roles:** Different role types and scopes
- **Network Security:** NSG rule evaluation and precedence
- **Key Vault Access:** Access policies vs RBAC model differences
- **Encryption Types:** Encryption at rest vs in transit vs client-side

## Hands-on Lab Areas

### Identity and Access Labs
- Configure Azure AD Connect for hybrid identity
- Implement Conditional Access policies
- Set up Privileged Identity Management
- Configure MFA and identity protection

### Network Security Labs
- Deploy and configure Azure Firewall
- Implement Web Application Firewall
- Configure Network Security Groups
- Set up private endpoints and service endpoints

### Compute Security Labs
- Configure VM disk encryption
- Implement container security in AKS
- Secure Azure App Service applications
- Configure endpoint protection

### Data Protection Labs
- Configure storage account security
- Implement Always Encrypted for SQL
- Set up Key Vault with different access models
- Configure database auditing and threat detection

### Security Operations Labs
- Configure Microsoft Defender for Cloud
- Set up Microsoft Sentinel
- Create security playbooks and automation
- Implement security monitoring and alerting

## Security Tools and Technologies

### Identity Security Tools
- **Azure Active Directory:** Identity provider and directory service
- **Azure AD Connect:** Hybrid identity synchronization
- **Conditional Access:** Risk-based access control
- **Privileged Identity Management:** Just-in-time privileged access

### Network Security Tools
- **Azure Firewall:** Cloud-native firewall service
- **Web Application Firewall:** Application layer protection
- **DDoS Protection:** Distributed denial of service protection
- **Azure Front Door:** Global application delivery network

### Security Monitoring Tools
- **Microsoft Defender for Cloud:** Cloud security posture management
- **Microsoft Sentinel:** Cloud-native SIEM/SOAR solution
- **Azure Monitor:** Comprehensive monitoring platform
- **Azure Security Center:** (Legacy) Security recommendations

### Data Protection Tools
- **Azure Key Vault:** Secret, key, and certificate management
- **Azure Information Protection:** Data classification and protection
- **Always Encrypted:** SQL database encryption
- **Storage Service Encryption:** Storage-level encryption

## Security Frameworks and Compliance

### Security Frameworks
- **NIST Cybersecurity Framework**
- **ISO 27001/27002**
- **CIS Controls**
- **OWASP Top 10**

### Compliance Standards
- **SOC 1/2/3**
- **GDPR**
- **HIPAA**
- **PCI DSS**
- **FedRAMP**

## 📚 Comprehensive Study Resources

**👉 [Complete Azure Study Resources Guide](../../../.templates/resources-azure.md)**

For detailed information on courses, practice tests, hands-on labs, communities, and more, see our comprehensive Azure study resources guide which includes:
- Official Microsoft Learn paths (FREE)
- Top-rated video courses with specific instructors
- Practice test platforms with pricing and comparisons
- Hands-on lab environments and free tier details
- Community forums and study groups
- Essential tools and Azure CLI resources
- Pro tips and budget-friendly study strategies

### Quick Links (AZ-500 Specific)
- **[AZ-500 Official Exam Page](https://learn.microsoft.com/en-us/certifications/exams/az-500/)** - Registration and exam details
- **[Microsoft Learn - AZ-500 Learning Path](https://learn.microsoft.com/en-us/certifications/azure-security-engineer/)** - FREE official study path
- **[Azure Security Documentation](https://docs.microsoft.com/en-us/azure/security/)** - Security guides
- **[Microsoft Security Blog](https://www.microsoft.com/security/blog/)** - Latest security news
- **[Azure Security Benchmark](https://docs.microsoft.com/en-us/azure/security/benchmarks/)** - Security baseline
- **[Microsoft Defender for Cloud Documentation](https://docs.microsoft.com/en-us/azure/defender-for-cloud/)** - Cloud security
- **[Microsoft Sentinel Documentation](https://docs.microsoft.com/en-us/azure/sentinel/)** - SIEM solution
- **[Azure Free Account](https://azure.microsoft.com/en-us/free/)** - $200 free credit for hands-on practice

## Prerequisites and Next Steps

### Prerequisites
- Azure Administrator knowledge (AZ-104 recommended)
- Basic security concepts understanding
- Experience with Azure networking and identity
- Understanding of compliance requirements

### Career Path
- **Next Certifications:** Expert-level certifications, specialized security certs
- **Role Focus:** Security Engineer, Security Architect, Compliance Officer
- **Skills Development:** Threat hunting, incident response, security automation

Remember: AZ-500 requires deep understanding of Azure security services and real-world security implementation experience. Focus on hands-on practice with security tools and understanding threat scenarios.