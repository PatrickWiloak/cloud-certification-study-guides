# Security, Compliance, and Identity Concepts

## Overview
Understanding foundational security, compliance, and identity concepts is essential for implementing Microsoft cloud solutions. This covers core security principles, compliance frameworks, and identity management fundamentals.

## Security Concepts

### Shared Responsibility Model

#### Cloud Provider Responsibilities
- **Physical datacenter security**
- **Physical network security**
- **Physical host security**
- **Hypervisor** (in most cases)

#### Customer Responsibilities
- **Data governance and rights management**
- **Endpoint protection**
- **Account and access management**
- **Identity and directory infrastructure**

#### Shared Responsibilities (varies by service model)
- **Application controls**
- **Network controls**
- **Operating system**
- **Physical devices

**Service Model Breakdown:**

| Responsibility | On-Premises | IaaS | PaaS | SaaS |
|---------------|-------------|------|------|------|
| Data & Access | Customer | Customer | Customer | Customer |
| Applications | Customer | Customer | Shared | Microsoft |
| Runtime | Customer | Customer | Microsoft | Microsoft |
| OS | Customer | Customer | Microsoft | Microsoft |
| Virtualization | Customer | Microsoft | Microsoft | Microsoft |
| Servers | Customer | Microsoft | Microsoft | Microsoft |
| Storage | Customer | Microsoft | Microsoft | Microsoft |
| Networking | Customer | Shared | Microsoft | Microsoft |

### Defense in Depth

#### Layered Security Approach
1. **Physical Security:** Datacenters, buildings, access controls
2. **Identity & Access:** Authentication, authorization, MFA
3. **Perimeter:** DDoS protection, firewalls
4. **Network:** Segmentation, deny by default, secure connectivity
5. **Compute:** Secure VMs, endpoint protection, patching
6. **Application:** Secure development, no secrets in code
7. **Data:** Encryption at rest and in transit, access controls

#### Key Principles
- **Multiple layers of protection**
- **Slows down attacks**
- **No single point of failure**
- **Each layer provides additional security**

### Zero Trust Model

#### Core Principles
1. **Verify explicitly:** Always authenticate and authorize
2. **Use least privilege access:** Just-in-time, just-enough-access (JIT/JEA)
3. **Assume breach:** Minimize blast radius, segment access

#### Zero Trust Pillars
- **Identities:** Users, services, devices
- **Devices:** Managed and compliant
- **Applications:** Shadow IT discovery, appropriate permissions
- **Data:** Classification, labeling, encryption
- **Infrastructure:** Version, configuration, JIT access
- **Networks:** Segmentation, threat protection, encryption

#### Traditional vs Zero Trust

**Traditional (Trust but Verify):**
- Perimeter-based security
- Internal network is trusted
- VPN provides full access

**Zero Trust (Never Trust, Always Verify):**
- Identity-based security
- No implicit trust
- Continuous verification
- Micro-segmentation

### Encryption and Hashing

#### Encryption
- **Purpose:** Protect confidentiality of data
- **Reversible:** Can be decrypted with key
- **Types:**
  - **Symmetric:** Same key for encryption/decryption (AES)
  - **Asymmetric:** Public/private key pairs (RSA)

**Encryption at Rest:**
- Data stored on physical media
- Protects if storage is compromised
- Azure Storage Service Encryption (SSE)
- Transparent Data Encryption (TDE) for databases

**Encryption in Transit:**
- Data moving across networks
- TLS/SSL for web traffic
- IPsec for VPN connections
- Always encrypt sensitive data in transit

#### Hashing
- **Purpose:** Verify data integrity
- **Irreversible:** Cannot be "decrypted"
- **Deterministic:** Same input = same output
- **Common algorithms:** SHA-256, SHA-512, MD5 (legacy)

**Use Cases:**
- Password storage (with salt)
- File integrity verification
- Digital signatures
- Certificate validation

### Common Threats and Attack Vectors

#### Threat Categories

**Malware:**
- Viruses, worms, trojans
- Ransomware
- Spyware and adware
- Rootkits

**Phishing:**
- Email phishing
- Spear phishing (targeted)
- Whaling (executives)
- Smishing (SMS), Vishing (voice)

**Man-in-the-Middle (MITM):**
- Intercept communications
- Session hijacking
- Wi-Fi eavesdropping

**Denial of Service (DoS/DDoS):**
- Overwhelm services
- Resource exhaustion
- Distributed attacks from multiple sources

**SQL Injection:**
- Inject malicious SQL code
- Access/modify database
- Extract sensitive data

**Password Attacks:**
- Brute force
- Dictionary attacks
- Credential stuffing
- Password spraying

**Insider Threats:**
- Malicious employees
- Accidental data exposure
- Compromised credentials

## Identity Concepts

### Authentication vs Authorization

#### Authentication (AuthN)
- **What it is:** Proving who you are
- **Process:** Verify identity using credentials
- **Methods:**
  - Something you know (password)
  - Something you have (token, phone)
  - Something you are (biometrics)
  - Somewhere you are (location)

**Multi-Factor Authentication (MFA):**
- Requires 2+ authentication factors
- Significantly increases security
- Common factors: Password + phone app/SMS/biometric

#### Authorization (AuthZ)
- **What it is:** Determining what you can access
- **Process:** Check permissions after authentication
- **Methods:**
  - Role-Based Access Control (RBAC)
  - Attribute-Based Access Control (ABAC)
  - Access Control Lists (ACLs)

**Example Flow:**
1. User authenticates (proves identity)
2. System authorizes (checks permissions)
3. User accesses allowed resources only

### Identity as Primary Security Perimeter

#### Traditional Perimeter
- Network-based security
- Firewall at network edge
- Internal network trusted
- Physical office locations

#### Modern Perimeter (Identity)
- **Mobile workforce:** Work from anywhere
- **Cloud services:** Resources outside corporate network
- **BYOD:** Personal devices accessing corporate data
- **Identity controls access** regardless of location

#### Why Identity is the New Perimeter
- Users access from anywhere
- Resources are in multiple clouds
- Traditional network boundaries don't exist
- Identity verification is consistent across locations

### Role-Based Access Control (RBAC)

#### Core Concepts
- **Roles:** Collection of permissions
- **Assignments:** Link role to user/group/service
- **Scope:** Where role applies (subscription, resource group, resource)

#### Azure RBAC Roles
**Built-in Roles:**
- **Owner:** Full access including delegation
- **Contributor:** Create/manage resources, no access delegation
- **Reader:** View resources only
- **User Access Administrator:** Manage user access

**Custom Roles:**
- Define specific permissions
- Fine-grained access control
- Based on business needs

#### RBAC Assignment
```
Who (Security Principal) + What (Role Definition) + Where (Scope) = Access
```

**Example:**
- Who: User "Alice"
- What: Contributor role
- Where: Resource Group "Production-RG"
- Result: Alice can manage resources in Production-RG

### Identity Providers and Directory Services

#### Identity Provider (IdP)
- **Purpose:** Create, maintain, manage identity information
- **Functions:**
  - Authentication services
  - Store user credentials
  - Provide authentication tokens
- **Examples:** Azure AD, Okta, Google Identity, Facebook Login

#### Directory Services
- **Purpose:** Store and organize identity information
- **Functions:**
  - Centralized user management
  - Group management
  - Device management
  - Policy enforcement

**Active Directory (AD):**
- On-premises directory service
- LDAP-based
- Windows Server role
- Kerberos authentication

**Azure Active Directory (Azure AD):**
- Cloud-based identity service
- REST APIs
- OAuth 2.0, SAML, WS-Federation
- Modern authentication protocols

#### Federation
- **Purpose:** Link identity systems
- **Benefit:** Single identity across multiple systems
- **Process:**
  1. User authenticates with IdP
  2. IdP issues security token
  3. Relying party trusts token
  4. User accesses resources

**Example:** Use corporate credentials to access partner applications

## Compliance Concepts

### Common Compliance Standards

#### GDPR (General Data Protection Regulation)
- **Scope:** EU data protection law
- **Key Requirements:**
  - Lawful basis for processing
  - Data subject rights (access, erasure, portability)
  - Breach notification (72 hours)
  - Data protection by design
  - Privacy impact assessments

#### HIPAA (Health Insurance Portability and Accountability Act)
- **Scope:** US healthcare data
- **Key Requirements:**
  - Protected Health Information (PHI) security
  - Access controls and audit logs
  - Encryption of ePHI
  - Business Associate Agreements (BAA)
  - Incident response procedures

#### PCI DSS (Payment Card Industry Data Security Standard)
- **Scope:** Payment card data
- **Key Requirements:**
  - Secure network and systems
  - Protect cardholder data
  - Vulnerability management
  - Strong access controls
  - Monitor and test networks
  - Information security policy

#### ISO 27001/27002
- **Scope:** Information security management
- **27001:** Requirements for ISMS
- **27002:** Best practices and guidelines
- **Coverage:** Risk assessment, security controls, continuous improvement

#### SOC Reports (Service Organization Control)
- **SOC 1:** Financial controls
- **SOC 2:** Security, availability, processing integrity, confidentiality, privacy
- **SOC 3:** Public summary of SOC 2

#### Other Important Standards
- **FedRAMP:** US federal cloud computing
- **NIST Frameworks:** Cybersecurity guidance
- **FISMA:** US federal information security
- **FERPA:** US student education records

### Data Residency and Sovereignty

#### Data Residency
- **Definition:** Physical location where data is stored
- **Importance:** Compliance with local laws
- **Azure:** Choose region for data storage

#### Data Sovereignty
- **Definition:** Data subject to laws of country where stored
- **Implications:**
  - Government access to data
  - Legal requirements vary by country
  - Cross-border data transfer restrictions

### Privacy Principles

#### Key Privacy Concepts
1. **Data Minimization:** Collect only necessary data
2. **Purpose Limitation:** Use data only for stated purpose
3. **Consent:** Obtain permission before processing
4. **Transparency:** Clear privacy policies
5. **Individual Rights:** Access, correction, deletion
6. **Accountability:** Demonstrate compliance

## Best Practices

### Security Best Practices
1. **Implement defense in depth** with multiple security layers
2. **Adopt Zero Trust principles** - never trust, always verify
3. **Use strong authentication** - require MFA for all users
4. **Encrypt sensitive data** at rest and in transit
5. **Regular security assessments** and penetration testing
6. **Keep systems patched** and updated
7. **Implement least privilege** access controls
8. **Monitor and log** all access and activities

### Identity Management Best Practices
1. **Use centralized identity** management (Azure AD)
2. **Implement conditional access** based on risk
3. **Require MFA** especially for privileged accounts
4. **Use managed identities** for Azure resources
5. **Regular access reviews** to remove unnecessary permissions
6. **Separate privileged accounts** from regular accounts
7. **Implement password policies** (complexity, expiration)

### Compliance Best Practices
1. **Understand applicable regulations** for your industry
2. **Document compliance efforts** and maintain records
3. **Regular compliance audits** and assessments
4. **Train employees** on compliance requirements
5. **Implement data classification** and handling procedures
6. **Establish incident response** procedures
7. **Use compliance management tools** (Microsoft Purview)

## Study Tips

### Key Concepts to Master
- Shared responsibility model for different service types
- Defense in depth layers
- Zero Trust principles (verify explicitly, least privilege, assume breach)
- Difference between encryption and hashing
- Authentication vs authorization
- Identity as the security perimeter
- RBAC components (principal, role, scope)
- Major compliance frameworks (GDPR, HIPAA, PCI DSS, ISO 27001)

### Common Scenarios
1. **Multi-cloud deployment** → Shared responsibility model understanding
2. **Remote workforce** → Zero Trust with identity-based security
3. **Data protection** → Encryption at rest and in transit
4. **Access control** → RBAC with least privilege
5. **Regulatory compliance** → Choose appropriate framework
6. **Password security** → MFA implementation

### Exam Focus Areas
- Shared responsibility in different cloud models
- Defense in depth concept and layers
- Zero Trust model principles
- When to use encryption vs hashing
- Authentication methods and MFA
- Authorization models (RBAC)
- Identity provider role
- Major compliance standards and their purpose
- Data residency vs data sovereignty

### Practice Questions
- What's the customer responsibility in PaaS vs IaaS?
- Which Zero Trust principle relates to minimal permissions?
- Is password storage using encryption or hashing?
- What's the difference between authentication and authorization?
- Which compliance standard applies to healthcare data?
- What is the new security perimeter in modern environments?

### Remember
- Shared responsibility varies by service model
- Zero Trust = Verify explicitly + Least privilege + Assume breach
- Encryption is reversible, hashing is not
- Authentication = Who you are, Authorization = What you can do
- Identity is the new security perimeter
- MFA significantly improves security
- Compliance requirements vary by industry and geography
