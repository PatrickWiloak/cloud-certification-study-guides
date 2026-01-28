# Azure Security Engineer (AZ-500) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Manage Identity and Access | 25-30% | 11 |
| Secure Networking | 20-25% | 9 |
| Secure Compute, Storage, and Databases | 20-25% | 9 |
| Manage Security Operations | 25-30% | 11 |

---

## Domain 1: Manage Identity and Access (Questions 1-11)

### Question 1
**Scenario:** A company wants to ensure that all administrative access to Azure resources requires multi-factor authentication, even for Global Administrators. They also need just-in-time access that requires approval for privileged roles. Which combination of Azure AD features provides this?

A. Azure AD Conditional Access with MFA requirement and Azure AD Privileged Identity Management (PIM)
B. Azure AD security defaults and role-based access control
C. Azure AD Identity Protection and access reviews
D. Azure AD B2C with MFA policies

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Conditional Access policies can require MFA for all users accessing Azure management endpoints (portal, CLI, PowerShell), including Global Administrators. PIM provides just-in-time privileged access with approval workflows, time-limited activations, and notifications. Together they provide comprehensive privileged access security. Security defaults (B) don't provide JIT access. Identity Protection (C) is for risk-based policies. B2C (D) is for external identities.

**Key Concept:** [Azure AD PIM](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-configure)
</details>

### Question 2
**Scenario:** An organization needs to grant a third-party vendor temporary access to a specific Azure subscription for a security audit. The vendor should have read-only access for 30 days, after which access should automatically expire. How should this be configured?

A. Create a guest user account with Reader role assigned at subscription scope
B. Use Azure AD B2B with PIM eligible assignment of Reader role with 30-day duration
C. Create a service principal with Reader role and rotate credentials after 30 days
D. Share subscription credentials with the vendor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure AD B2B collaboration allows inviting external users (guest accounts). PIM eligible assignments can have an end date, ensuring automatic expiration after 30 days. The vendor activates the role when needed and it expires automatically. Guest accounts without PIM (A) don't auto-expire. Service principals (C) are for applications, not human access. Sharing credentials (D) violates security best practices.

**Key Concept:** [Azure AD B2B with PIM](https://docs.microsoft.com/azure/active-directory/external-identities/b2b-quickstart-invite-portal)
</details>

### Question 3
**Scenario:** A company's security policy requires that users can only authenticate from corporate-managed devices. They use a mix of Windows, macOS, and iOS devices managed through Microsoft Intune. How should they enforce this requirement?

A. Configure Conditional Access requiring compliant device or Hybrid Azure AD joined device
B. Block all mobile device access in Azure AD settings
C. Require hardware tokens for all authentication
D. Use IP-based access restrictions for corporate networks only

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Conditional Access can require devices to be marked as compliant in Intune or be Hybrid Azure AD joined (for domain-joined Windows devices). This covers all managed device types. The device state is evaluated at sign-in. Blocking mobile (B) prevents legitimate iOS access. Hardware tokens (C) verify user identity, not device state. IP restrictions (D) don't work for remote workers on managed devices.

**Key Concept:** [Conditional Access Device Conditions](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-conditions#device-state)
</details>

### Question 4
**Scenario:** An application needs to access Azure Key Vault to retrieve secrets. The application runs on an Azure VM. Security requires that no credentials are stored in the application code or configuration. What is the BEST approach?

A. Store Key Vault access keys in application settings
B. Use system-assigned managed identity for the VM, grant the identity access to Key Vault
C. Use Azure AD application registration with client secret stored in environment variables
D. Configure Key Vault firewall to allow the VM's IP address

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** System-assigned managed identity provides an automatically managed identity in Azure AD for the VM. The identity can be granted access to Key Vault via access policies or RBAC. The application uses the Azure SDK which automatically obtains tokens from the metadata endpoint - no credentials in code. Stored credentials (A, C) can be compromised. IP allowlisting (D) doesn't provide authentication.

**Key Concept:** [Managed Identities for Azure Resources](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)
</details>

### Question 5
**Scenario:** A company discovered that user accounts are being compromised through password spray attacks. They need to detect and block these attacks automatically while minimizing impact on legitimate users. Which Azure AD feature addresses this?

A. Azure AD Password Protection with custom banned passwords
B. Azure AD Identity Protection with sign-in risk policies
C. Azure AD Conditional Access with named locations
D. Azure AD smart lockout combined with Identity Protection

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** Smart lockout protects against brute-force and password spray attacks by locking out attackers while allowing legitimate users to continue signing in. It learns user patterns. Identity Protection detects risky sign-ins (including password spray patterns) and can require MFA or block access based on risk level. Password Protection (A) prevents weak passwords but doesn't detect attacks. Named locations (C) don't detect attack patterns.

**Key Concept:** [Azure AD Smart Lockout](https://docs.microsoft.com/azure/active-directory/authentication/howto-password-smart-lockout)
</details>

### Question 6
**Scenario:** An organization wants to implement single sign-on for a legacy on-premises web application that uses header-based authentication. The application cannot be modified. How can they integrate this with Azure AD?

A. Use Azure AD Application Proxy with header-based authentication
B. Configure SAML federation with the application
C. Use Azure AD B2C custom policies
D. Implement OpenID Connect in the application

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure AD Application Proxy can publish on-premises applications to external users via Azure AD authentication. For header-based apps, the Proxy (with PingAccess or custom connectors) translates Azure AD authentication to HTTP headers the legacy app expects. No app modification required. SAML (B) and OIDC (D) require app changes. B2C (C) is for external customer identities.

**Key Concept:** [Application Proxy Header-Based SSO](https://docs.microsoft.com/azure/active-directory/app-proxy/application-proxy-configure-single-sign-on-with-headers)
</details>

### Question 7
**Scenario:** A security audit found that several Azure AD users have excessive permissions through direct role assignments at various resource scopes. The team needs to review all access permissions, identify over-privileged users, and remediate. Which Azure feature helps with this?

A. Azure AD Access Reviews for Azure resource roles
B. Azure Policy compliance reports
C. Azure Advisor security recommendations
D. Azure Monitor activity logs

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure AD Access Reviews can be configured for Azure resource roles (RBAC). Reviewers assess whether users still need their role assignments. Reviews can be scheduled recurring, with automatic actions to remove access if not approved. This provides systematic remediation of over-provisioned access. Policy (B) checks resource configurations. Advisor (C) provides recommendations but not access review workflows. Activity logs (D) show actions, not current permissions.

**Key Concept:** [Access Reviews for Azure Resources](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)
</details>

### Question 8
**Scenario:** A company is implementing a Zero Trust strategy and wants to continuously evaluate user and session risk, requiring step-up authentication when risk increases during a session. How should they configure this?

A. Require MFA for all sign-ins via Conditional Access
B. Configure Continuous Access Evaluation (CAE) with Conditional Access session controls
C. Use PIM with short activation periods
D. Implement session timeout policies

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Continuous Access Evaluation enables near real-time policy and security enforcement during active sessions. When combined with Conditional Access session controls, it can detect risk changes (location, device compliance, user risk) and require re-authentication or block access mid-session. MFA at sign-in only (A) doesn't evaluate ongoing risk. PIM (C) is for privileged roles. Session timeouts (D) are time-based, not risk-based.

**Key Concept:** [Continuous Access Evaluation](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation)
</details>

### Question 9
**Scenario:** An organization needs to ensure that service principals used by CI/CD pipelines have the minimum required permissions and their credentials are rotated regularly. They also need to audit all actions performed by these service principals. What combination of practices should they implement?

A. Use managed identities where possible, use certificates instead of secrets for service principals, assign custom roles with minimum permissions, enable Azure AD audit logs
B. Use service principals with secrets, assign built-in Contributor role, review logs monthly
C. Use personal accounts for CI/CD pipelines with MFA
D. Create one shared service principal for all pipelines with Owner role

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Managed identities eliminate credential management entirely. When SPs are needed, certificates are more secure than secrets and can be auto-rotated. Custom roles with minimum permissions follow least privilege. Azure AD audit logs and Azure Activity Logs capture all SP actions. Shared SPs (D) and broad roles (B) violate least privilege. Personal accounts (C) lack automation support and auditing.

**Key Concept:** [Service Principal Best Practices](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal)
</details>

### Question 10
**Scenario:** A company wants to prevent users from accidentally granting third-party applications excessive permissions through OAuth consent. They want to allow only pre-approved applications and require admin approval for others. How should this be configured?

A. Disable all app registrations in Azure AD
B. Configure user consent settings to allow consent only for apps from verified publishers for low-risk permissions, require admin consent for all others
C. Block all third-party applications
D. Allow user consent for all permissions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure AD consent settings can be configured to allow users to consent only to apps from verified publishers requesting low-risk (delegated) permissions. All other consent requests require admin approval through the admin consent workflow. This balances security with usability. Blocking all apps (A, C) prevents legitimate integrations. Allowing all consent (D) enables OAuth phishing attacks.

**Key Concept:** [Configure User Consent](https://docs.microsoft.com/azure/active-directory/manage-apps/configure-user-consent)
</details>

### Question 11
**Scenario:** During a security incident, you need to immediately revoke all active sessions for a compromised user account while keeping the account active for investigation. What is the FASTEST way to invalidate all sessions?

A. Disable the user account
B. Reset the user's password
C. Use the "Revoke sessions" option in Azure AD which invalidates refresh tokens
D. Remove all application assignments

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** "Revoke sessions" in Azure AD immediately invalidates all refresh tokens for the user. Combined with Continuous Access Evaluation, this provides near-instant session termination for CAE-enabled applications. The user account remains active for investigation. Disabling (A) prevents investigation. Password reset (B) works but is slower to propagate. Removing app assignments (D) doesn't invalidate existing sessions.

**Key Concept:** [Revoke User Access](https://docs.microsoft.com/azure/active-directory/enterprise-users/users-revoke-access)
</details>

---

## Domain 2: Secure Networking (Questions 12-20)

### Question 12
**Scenario:** A company needs to allow Azure App Service to access a SQL database and Storage account in a VNet while preventing the App Service from accessing the public internet. The App Service is on a Premium plan. What architecture should be implemented?

A. Configure VNet integration for App Service with service endpoints for SQL and Storage
B. Configure VNet integration for App Service with private endpoints for SQL and Storage, and route all traffic through a NAT Gateway
C. Configure VNet integration with regional VNet integration, private endpoints for PaaS services, and a route table forcing all traffic through Azure Firewall with no public routes
D. Use App Service Environment in an isolated VNet

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Regional VNet integration allows App Service to access resources in the VNet. Private endpoints provide private connectivity to SQL and Storage. A route table with 0.0.0.0/0 pointing to Azure Firewall (with no default internet route) forces all egress through the firewall where internet access can be blocked. Service endpoints (A) still allow internet egress. Option B with NAT Gateway still allows internet access. ASE (D) is more expensive than needed.

**Key Concept:** [App Service VNet Integration](https://docs.microsoft.com/azure/app-service/overview-vnet-integration)
</details>

### Question 13
**Scenario:** An organization wants to inspect all traffic between VNets in their hub-and-spoke topology, including traffic from spokes to shared services in the hub. They also need URL filtering and threat intelligence-based filtering. What Azure service provides these capabilities?

A. Network Security Groups with application security groups
B. Azure Firewall Premium with TLS inspection and IDPS
C. Azure Web Application Firewall
D. Network Virtual Appliance with custom rules

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Firewall Premium provides URL filtering, threat intelligence-based filtering, TLS inspection (to inspect encrypted traffic), and intrusion detection and prevention (IDPS). In hub-and-spoke, route tables in spoke VNets point to Azure Firewall in the hub for east-west traffic inspection. NSGs (A) are stateful but layer 4 only. WAF (C) is for web application protection, not general network filtering. NVAs (D) require more management.

**Key Concept:** [Azure Firewall Premium](https://docs.microsoft.com/azure/firewall/premium-features)
</details>

### Question 14
**Scenario:** A web application behind Azure Application Gateway needs protection against SQL injection, XSS, and OWASP Top 10 vulnerabilities. Security also requires logging of all blocked requests for forensic analysis. What configuration is needed?

A. Enable Azure DDoS Protection Standard
B. Configure Web Application Firewall (WAF) policy with OWASP rule sets in Prevention mode, enable diagnostic logging
C. Use Network Security Groups with deny rules
D. Configure Application Gateway health probes

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** WAF on Application Gateway provides protection against OWASP Top 10 including SQL injection and XSS. Prevention mode blocks malicious requests. WAF policies with OWASP 3.2/3.1 Core Rule Sets cover common web vulnerabilities. Diagnostic logging to Log Analytics or Storage captures blocked request details. DDoS (A) protects against volumetric attacks, not application attacks. NSGs (C) are layer 4. Health probes (D) are for backend health.

**Key Concept:** [Azure WAF on Application Gateway](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview)
</details>

### Question 15
**Scenario:** A company needs to ensure that management traffic to Azure VMs (RDP, SSH) never traverses the public internet. Administrators work remotely and need access without VPN. What solution provides secure management access?

A. Open NSG rules for RDP/SSH from administrator IP addresses
B. Deploy Azure Bastion in the VNet for browser-based RDP/SSH access
C. Configure public IPs on VMs with Just-in-Time VM access
D. Use Azure Serial Console for all management

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Bastion provides secure RDP/SSH access through the Azure portal over TLS. VMs don't need public IPs, and management traffic stays within Azure's backbone. No VPN client required. NSG rules with public IPs (A) expose management ports to internet. JIT (C) still requires public IP or other connectivity. Serial Console (D) has limited functionality and is for troubleshooting, not daily management.

**Key Concept:** [Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview)
</details>

### Question 16
**Scenario:** An organization has multiple VNets across regions that need to communicate securely. They also have ExpressRoute connectivity to on-premises. Traffic between regions should be encrypted. What architecture provides this?

A. VNet peering between all VNets with ExpressRoute circuits
B. Azure Virtual WAN with encrypted VNet connections and ExpressRoute integration
C. Site-to-site VPN between each VNet pair
D. ExpressRoute Global Reach only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Virtual WAN provides hub-and-spoke at global scale with encryption for branch-to-VNet and VNet-to-VNet over the Microsoft backbone. It integrates ExpressRoute for on-premises connectivity. Encryption is handled natively. VNet peering (A) doesn't encrypt traffic natively. S2S VPN between each pair (C) doesn't scale. Global Reach (D) connects ExpressRoute circuits but doesn't encrypt Azure traffic.

**Key Concept:** [Azure Virtual WAN](https://docs.microsoft.com/azure/virtual-wan/virtual-wan-about)
</details>

### Question 17
**Scenario:** A security team needs to log and analyze all DNS queries from VMs in their Azure VNets for threat detection. They want to detect communication to known malicious domains. What is the BEST solution?

A. Configure custom DNS servers that log queries
B. Enable Azure Firewall DNS Proxy with logging and threat intelligence
C. Use Azure DNS Private Resolver with diagnostic logs
D. Capture DNS traffic with Network Watcher packet capture

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Firewall can act as a DNS Proxy, logging all DNS queries from VMs. Combined with threat intelligence, it can detect and optionally block queries to known malicious domains. All queries are logged to Log Analytics for analysis. Custom DNS (A) requires more infrastructure. Private Resolver (C) is for hybrid DNS, not security logging. Packet capture (D) is not practical for continuous monitoring.

**Key Concept:** [Azure Firewall DNS Settings](https://docs.microsoft.com/azure/firewall/dns-settings)
</details>

### Question 18
**Scenario:** A company needs to protect their public-facing web application from DDoS attacks while maintaining detailed attack analytics and having access to the DDoS Rapid Response team during attacks. Which Azure DDoS Protection tier should they use?

A. Azure DDoS Infrastructure Protection (Basic)
B. Azure DDoS Network Protection
C. Azure DDoS IP Protection
D. Azure Web Application Firewall only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** DDoS Network Protection provides enhanced DDoS mitigation including attack analytics, alerting, and access to the DDoS Rapid Response (DRR) team during active attacks. It protects all public IPs in the VNet. IP Protection (C) provides per-IP protection without DRR access. Basic (A) is automatic but provides no analytics or DRR. WAF (D) doesn't protect against volumetric DDoS attacks.

**Key Concept:** [Azure DDoS Protection](https://docs.microsoft.com/azure/ddos-protection/ddos-protection-overview)
</details>

### Question 19
**Scenario:** An organization needs to ensure that only traffic from their corporate IP ranges can access Azure Storage accounts, but they also need Azure Data Factory (running in an Azure VNet) to access the same storage. How should network access be configured?

A. Storage firewall with corporate IP ranges only
B. Storage firewall allowing corporate IP ranges and VNet service endpoint from Data Factory's subnet
C. Private endpoint for Data Factory and public access for corporate IPs
D. No network restrictions with SAS tokens for all access

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Storage firewall can combine IP-based rules (for corporate access) with VNet service endpoints (for Azure service access). Add corporate IP ranges to the firewall and enable service endpoint for the Data Factory subnet, then allow that subnet in storage firewall. Data Factory traffic goes through the service endpoint, not public internet. Private endpoints (C) would work but service endpoints are simpler here. No restrictions (D) violates security requirements.

**Key Concept:** [Azure Storage Firewalls](https://docs.microsoft.com/azure/storage/common/storage-network-security)
</details>

### Question 20
**Scenario:** A company wants to use Azure Front Door for their global web application. They need to ensure that backend App Services only accept traffic from Front Door and not from direct internet access. How should this be implemented?

A. Configure App Service IP restrictions for Front Door's IP ranges
B. Use App Service access restrictions with the Front Door service tag and header-based filtering for X-Azure-FDID
C. Deploy App Service in a VNet with NSG rules
D. Use client certificates from Front Door to App Service

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** App Service access restrictions can use the AzureFrontDoor.Backend service tag to allow only Front Door IPs. However, since Front Door IPs are shared across customers, you must also filter on the X-Azure-FDID header containing your specific Front Door instance ID. This ensures only your Front Door can access your backend. IP ranges only (A) allows any Front Door instance. VNet deployment (C) is unnecessary complexity. Client certs (D) aren't supported this way.

**Key Concept:** [Restrict Backend Access to Front Door](https://docs.microsoft.com/azure/frontdoor/front-door-faq#how-do-i-lock-down-the-access-to-my-backend-to-only-azure-front-door)
</details>

---

## Domain 3: Secure Compute, Storage, and Databases (Questions 21-29)

### Question 21
**Scenario:** A company stores sensitive customer data in Azure Blob Storage. Compliance requires that data be encrypted with customer-managed keys, keys must be stored in a FIPS 140-2 Level 3 validated HSM, and key operations must be logged. What configuration meets these requirements?

A. Storage Service Encryption with Microsoft-managed keys
B. Storage Service Encryption with customer-managed keys in Azure Key Vault (Premium tier with HSM-backed keys)
C. Storage Service Encryption with customer-managed keys in Azure Key Vault Managed HSM
D. Client-side encryption with application-managed keys

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Azure Key Vault Managed HSM provides dedicated, FIPS 140-2 Level 3 validated HSMs under customer control. Storage encryption with CMK stored in Managed HSM meets the HSM validation requirement. All key operations are logged. Key Vault Premium (B) uses multi-tenant HSMs which are Level 2, not Level 3. Microsoft-managed keys (A) don't provide customer control. Client-side (D) adds complexity and doesn't integrate with managed HSM automatically.

**Key Concept:** [Azure Key Vault Managed HSM](https://docs.microsoft.com/azure/key-vault/managed-hsm/overview)
</details>

### Question 22
**Scenario:** An organization needs to ensure that Azure SQL Database connections are always encrypted in transit, use the strongest available TLS version, and reject connections from applications using older TLS versions. How should this be configured?

A. Enable Transparent Data Encryption (TDE) on the database
B. Configure minimum TLS version to 1.2 on the SQL Server, which is enforced for all connections
C. Use Always Encrypted for sensitive columns
D. Configure SQL firewall rules

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Setting minimum TLS version to 1.2 at the Azure SQL Server level enforces encryption in transit with modern TLS for all database connections. Connections using TLS 1.0 or 1.1 are rejected. Azure SQL always encrypts connections; this setting controls the minimum acceptable version. TDE (A) is encryption at rest. Always Encrypted (C) is for column-level encryption. Firewall rules (D) control network access, not encryption.

**Key Concept:** [Azure SQL TLS](https://docs.microsoft.com/azure/azure-sql/database/connectivity-settings#minimal-tls-version)
</details>

### Question 23
**Scenario:** A healthcare application needs to store patient data in Azure SQL Database. Developers should not be able to see patient names and SSNs even when querying the database directly for troubleshooting. The application should transparently encrypt and decrypt this data. What feature provides this?

A. Transparent Data Encryption (TDE)
B. Dynamic Data Masking
C. Always Encrypted with secure enclaves
D. Row-Level Security

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Always Encrypted ensures sensitive data is encrypted client-side and remains encrypted in the database. Database administrators and developers cannot see plaintext data even with full database access. Secure enclaves enable server-side operations on encrypted data (comparisons, pattern matching) for richer query support. TDE (A) encrypts at rest but data is visible to authorized users. Masking (B) can be bypassed by database admins. RLS (D) filters rows, not encrypts columns.

**Key Concept:** [Always Encrypted with Secure Enclaves](https://docs.microsoft.com/azure/azure-sql/database/always-encrypted-with-secure-enclaves-landing)
</details>

### Question 24
**Scenario:** An organization wants to scan their Azure VMs for vulnerabilities, missing OS patches, and misconfigurations. They need recommendations prioritized by severity and integration with their existing Security operations workflow in Microsoft Defender for Cloud. What should they enable?

A. Microsoft Defender for Servers with vulnerability assessment integration
B. Azure Update Management only
C. Third-party vulnerability scanner
D. Azure Advisor recommendations

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Microsoft Defender for Servers (part of Defender for Cloud) provides vulnerability assessment powered by Qualys or Microsoft Defender Vulnerability Management, integrated directly into Defender for Cloud. Findings appear as security recommendations prioritized by severity. It also assesses OS configurations against security baselines. Update Management (B) handles patches but not vulnerability assessment. Third-party (C) adds complexity. Advisor (D) provides limited security insights.

**Key Concept:** [Defender for Servers](https://docs.microsoft.com/azure/defender-for-cloud/defender-for-servers-introduction)
</details>

### Question 25
**Scenario:** A company running containerized workloads in AKS needs to ensure that containers cannot run as root, cannot use privileged mode, and can only pull images from their private Azure Container Registry. How should these security policies be enforced?

A. Document the requirements and train developers
B. Use Azure Policy for Kubernetes with built-in policies for container security
C. Configure network policies in the AKS cluster
D. Use AKS node pools with Windows containers only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Policy for Kubernetes applies Gatekeeper policies to AKS clusters. Built-in policies include: don't allow privileged containers, don't allow root user, only allow images from specified registries. Policies can audit or deny non-compliant pods at admission time. Training (A) doesn't enforce. Network policies (C) control network traffic, not container privileges. Windows containers (D) don't address the security requirements.

**Key Concept:** [Azure Policy for AKS](https://docs.microsoft.com/azure/governance/policy/concepts/policy-for-kubernetes)
</details>

### Question 26
**Scenario:** An organization wants to protect Azure VM disks with customer-managed encryption keys and ensure the encryption keys are automatically rotated. They also need to ensure VMs cannot be started without access to the encryption key. What configuration provides this?

A. Azure Disk Encryption (ADE) with BitLocker/DM-Crypt
B. Server-Side Encryption (SSE) with customer-managed keys in Key Vault with automatic key rotation
C. Confidential VMs with confidential OS disk encryption
D. Storage Service Encryption with Microsoft-managed keys

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Server-Side Encryption with CMK provides transparent disk encryption using keys stored in Key Vault. Key Vault supports automatic key rotation, and Azure automatically re-encrypts disks with new key versions. If the key is disabled or deleted, the VM cannot start. ADE (A) uses guest-level encryption and doesn't auto-rotate keys. Confidential VMs (C) are for enhanced isolation but have different key management. Microsoft-managed keys (D) don't provide customer control.

**Key Concept:** [Azure Disk Encryption with CMK](https://docs.microsoft.com/azure/virtual-machines/disk-encryption-overview)
</details>

### Question 27
**Scenario:** A company needs to ensure that Azure Functions can only access specific secrets in Key Vault, following the principle of least privilege. The Function should not have access to other secrets in the vault or other Key Vaults. How should access be configured?

A. Grant the Function's managed identity Key Vault Administrator role at the vault level
B. Use Key Vault access policies granting Get permission for specific secret names to the Function's managed identity
C. Use Key Vault RBAC with Key Vault Secrets User role scoped to specific secrets
D. Store the secrets in Function application settings instead

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Key Vault RBAC allows role assignments at individual secret scope, enabling true least privilege - the Function can only access specific secrets, not all secrets in the vault. Key Vault Secrets User role grants read access to secret values. Access policies (B) are vault-wide and don't support per-secret granularity. Administrator role (A) grants full control. App settings (D) don't use Key Vault security features.

**Key Concept:** [Key Vault RBAC](https://docs.microsoft.com/azure/key-vault/general/rbac-guide)
</details>

### Question 28
**Scenario:** An organization uses Azure Container Registry for storing container images. They need to ensure only approved, vulnerability-scanned images can be deployed to production AKS clusters. How should this be implemented?

A. Enable Microsoft Defender for Containers for vulnerability scanning and use Azure Policy to allow only signed images from the registry
B. Manually review images before deployment
C. Use only public images from Docker Hub
D. Disable image scanning to reduce deployment time

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Defender for Containers scans images in ACR for vulnerabilities. Azure Policy for Kubernetes can enforce that only images from specific registries are allowed and can require image signatures (using Notation or similar). This creates a gated pipeline where only scanned, approved images reach production. Manual review (B) doesn't scale. Public images (C) and disabling scanning (D) reduce security.

**Key Concept:** [Defender for Containers](https://docs.microsoft.com/azure/defender-for-cloud/defender-for-containers-introduction)
</details>

### Question 29
**Scenario:** A database administrator needs temporary elevated access to an Azure SQL Database to troubleshoot a production issue. Normal operations should not have this level of access. Access should be time-limited and fully audited. How should this be managed?

A. Grant the DBA permanent db_owner access
B. Use Azure AD PIM with eligible assignment for SQL db_owner role, requiring activation and approval
C. Share the SQL admin password for emergency access
D. Create a temporary SQL login and delete after use

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** PIM for Azure resources can manage database access through Azure AD authentication. The DBA has an eligible assignment that requires activation, can include approval workflow, is time-limited, and is fully logged in Azure AD audit logs. Permanent access (A) violates least privilege. Shared passwords (C) lack accountability. Temporary logins (D) require manual cleanup and may not be audited as completely.

**Key Concept:** [PIM for Azure AD Roles](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-how-to-add-role-to-user)
</details>

---

## Domain 4: Manage Security Operations (Questions 30-40)

### Question 30
**Scenario:** An organization wants to centralize security alerts from Azure resources, third-party security solutions, and custom log sources. They need correlation, automated investigation, and playbook-based response. What Azure solution provides this SIEM/SOAR capability?

A. Azure Monitor with action groups
B. Microsoft Defender for Cloud only
C. Microsoft Sentinel with data connectors, analytics rules, and playbooks
D. Azure Log Analytics with custom queries

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Microsoft Sentinel is Azure's cloud-native SIEM/SOAR. Data connectors ingest from Azure services, Microsoft 365, third-party sources, and custom logs. Analytics rules detect threats and create incidents. Playbooks (Logic Apps) automate response. Built-in ML detects anomalies. Defender for Cloud (B) provides workload protection but not full SIEM. Monitor (A) and Log Analytics (D) provide logging but not security orchestration.

**Key Concept:** [Microsoft Sentinel](https://docs.microsoft.com/azure/sentinel/overview)
</details>

### Question 31
**Scenario:** A security analyst needs to investigate a potential compromise where an attacker may have moved laterally through the Azure environment. They need to understand the sequence of events, affected resources, and scope of the incident. Which Sentinel feature helps with this investigation?

A. Workbooks for visualization
B. Investigation graph showing entity relationships and timeline
C. Watchlists for IOC tracking
D. Notebooks for advanced hunting

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Sentinel's Investigation Graph provides visual exploration of incidents, showing entity relationships (users, IPs, hosts, resources), timeline of events, and connections between related alerts. Analysts can expand entities to discover the full scope of compromise. This directly supports lateral movement investigation. Workbooks (A) are for dashboards. Watchlists (C) track indicators. Notebooks (D) are for advanced analysis but require more expertise.

**Key Concept:** [Sentinel Investigation](https://docs.microsoft.com/azure/sentinel/investigate-cases)
</details>

### Question 32
**Scenario:** An organization wants to automatically respond to high-severity alerts by isolating affected VMs, notifying the security team, and creating a ticket in ServiceNow. The response should occur within seconds of alert generation. How should this be configured in Sentinel?

A. Create automation rules that trigger a playbook (Logic App) performing isolation, notification, and ServiceNow integration
B. Manually review each alert and take action
C. Use scheduled queries to check for alerts hourly
D. Export alerts to a SIEM and respond from there

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Automation rules in Sentinel can automatically trigger playbooks based on incident properties (severity, threat type). Playbooks built with Logic Apps can call Azure APIs to isolate VMs (change NSG rules, disable NICs), send notifications (email, Teams), and integrate with ServiceNow via connectors. Response is near real-time. Manual review (B) and hourly queries (C) are too slow. Exporting to another SIEM (D) adds unnecessary complexity.

**Key Concept:** [Sentinel Automation](https://docs.microsoft.com/azure/sentinel/automation)
</details>

### Question 33
**Scenario:** A company needs to monitor for signs of compromise across their Azure AD tenant, including impossible travel, leaked credentials, and atypical behavior. Detections should feed into their Sentinel workspace. What should be configured?

A. Azure AD audit logs only
B. Azure AD Identity Protection with Sentinel connector
C. Azure AD sign-in logs with custom analytics rules
D. Microsoft Defender for Identity

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure AD Identity Protection provides built-in detections for identity risks including impossible travel, anonymous IP use, leaked credentials (from dark web monitoring), and atypical behavior using ML. The Sentinel connector ingests these risk detections as alerts. Custom rules (C) would require building these detections from scratch. Audit logs (A) don't provide risk scoring. Defender for Identity (D) is for on-premises AD/hybrid environments.

**Key Concept:** [Identity Protection Integration with Sentinel](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-investigate-risk)
</details>

### Question 34
**Scenario:** A security team wants to proactively hunt for threats in their Azure environment using custom queries against log data. They need to save and share successful hunting queries with the team and convert confirmed threats into detection rules. Which Sentinel feature supports this workflow?

A. Workbooks
B. Hunting queries with bookmarks and promotion to analytics rules
C. Threat Intelligence indicators
D. Entity behavior analytics

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Sentinel Hunting provides a query workspace with built-in and custom KQL queries against Log Analytics data. Interesting findings can be bookmarked for investigation. Successful hunting queries can be promoted to analytics rules for ongoing automated detection. Queries can be shared via the content hub. Workbooks (A) are for visualization. TI (C) tracks known indicators. UEBA (D) is for behavior profiling.

**Key Concept:** [Sentinel Hunting](https://docs.microsoft.com/azure/sentinel/hunting)
</details>

### Question 35
**Scenario:** An organization uses multiple Microsoft security products (Defender for Endpoint, Defender for Office 365, Defender for Identity, Defender for Cloud Apps). They want unified incident management and cross-product correlation. How should they consolidate these signals?

A. Configure each product to send alerts to separate Sentinel workspaces
B. Use Microsoft 365 Defender portal for integrated XDR experience across Microsoft Defender products
C. Build custom integrations between each product
D. Use only one Defender product

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Microsoft 365 Defender provides unified XDR (Extended Detection and Response) across all Microsoft Defender products. It automatically correlates alerts into incidents, provides cross-product investigation, and unified hunting. The portal combines endpoint, email, identity, and cloud app security. Separate Sentinel workspaces (A) fragment data. Custom integrations (C) duplicate built-in functionality. Single product (D) loses visibility.

**Key Concept:** [Microsoft 365 Defender](https://docs.microsoft.com/microsoft-365/security/defender/microsoft-365-defender)
</details>

### Question 36
**Scenario:** A security analyst receives a Sentinel incident indicating potential data exfiltration from a Storage account. They need to quickly determine what data was accessed, by whom, from where, and whether this is anomalous. What logs should they analyze?

A. Azure Activity logs for the Storage account
B. Azure Storage diagnostic logs (StorageRead, StorageWrite) combined with Azure AD sign-in logs
C. NSG flow logs for the Storage account's network
D. Azure Monitor metrics for Storage throughput

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Storage diagnostic logs (specifically StorageBlobLogs or StorageFileLogs) capture detailed read/write operations including which blobs were accessed, caller IP, authentication type. Correlating with Azure AD sign-in logs identifies the user/principal. This combination answers who, what, when, and where. Activity logs (A) show management operations, not data access. NSG flow logs (C) show network flows, not storage operations. Metrics (D) show aggregate throughput only.

**Key Concept:** [Azure Storage Logging](https://docs.microsoft.com/azure/storage/common/storage-analytics-logging)
</details>

### Question 37
**Scenario:** An organization wants to ensure compliance with security baselines across all their Azure subscriptions. They need continuous monitoring, compliance scoring, and the ability to remediate non-compliant resources automatically. What Azure feature provides this?

A. Azure Blueprints
B. Microsoft Defender for Cloud with security policies, secure score, and auto-remediation
C. Azure Advisor
D. Azure Resource Graph

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Defender for Cloud provides security policies (built on Azure Policy) that continuously assess resources against security baselines. Secure Score quantifies security posture. Recommendations identify non-compliant resources, and many have "Fix" buttons for auto-remediation. Regulatory compliance dashboards track against standards. Blueprints (A) deploy resources but don't continuously monitor. Advisor (C) has limited security coverage. Resource Graph (D) queries resources but doesn't assess compliance.

**Key Concept:** [Defender for Cloud Recommendations](https://docs.microsoft.com/azure/defender-for-cloud/recommendations-reference)
</details>

### Question 38
**Scenario:** A security team needs to detect unusual user behavior that might indicate compromised accounts, such as a user suddenly accessing resources they've never accessed before or working at unusual hours. What Sentinel capability provides this detection?

A. Scheduled analytics rules with custom queries
B. User and Entity Behavior Analytics (UEBA)
C. Threat Intelligence matching
D. Fusion advanced multistage attack detection

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** UEBA builds behavioral profiles for users and entities based on historical activity patterns. It detects anomalies like first-time access, unusual hours, impossible travel, and peer group deviations. UEBA enriches investigations with context about whether behavior is anomalous for that entity. Custom rules (A) require known patterns. TI (C) matches known indicators. Fusion (D) correlates alerts but relies on other detections.

**Key Concept:** [Sentinel UEBA](https://docs.microsoft.com/azure/sentinel/identify-threats-with-entity-behavior-analytics)
</details>

### Question 39
**Scenario:** During a security incident, a SOC analyst needs to understand all resources a specific user account has accessed in the last 30 days, including Azure resources, Microsoft 365 applications, and any lateral movement attempts. What is the MOST efficient way to get this information?

A. Query Azure AD sign-in logs, Azure Activity logs, and Microsoft 365 audit logs separately
B. Use Sentinel Entity pages which aggregate activity across connected data sources for a user entity
C. Export all logs to Excel for manual analysis
D. Interview the user about their activities

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Sentinel Entity pages provide a unified view of an entity (user, host, IP) aggregating data from all connected sources. For a user entity, this shows Azure AD sign-ins, Azure resource access, M365 activity, associated alerts, and relationship to other entities. This provides rapid triage without manual correlation. Separate queries (A) are time-consuming. Manual analysis (C, D) doesn't scale.

**Key Concept:** [Sentinel Entity Pages](https://docs.microsoft.com/azure/sentinel/entity-pages)
</details>

### Question 40
**Scenario:** An organization wants to block known malicious IP addresses and domains across their Azure environment including Azure Firewall, NSGs, and Sentinel for detection. They need this threat intelligence updated automatically from multiple sources. How should this be implemented?

A. Manually update block lists weekly
B. Configure Threat Intelligence Platform data connector in Sentinel with automated indicator import to Azure Firewall
C. Use only the built-in Microsoft threat intelligence
D. Subscribe to threat feeds via RSS

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Sentinel's Threat Intelligence Platform (TIP) connector imports indicators from TAXII servers, STIX files, and various threat intelligence platforms. These indicators can be used in Sentinel analytics for detection and automatically exported to Azure Firewall for blocking. This provides unified TI management with automated updates. Manual updates (A) are slow and error-prone. Built-in TI only (C) limits coverage. RSS (D) isn't a standard TI format.

**Key Concept:** [Sentinel Threat Intelligence](https://docs.microsoft.com/azure/sentinel/understand-threat-intelligence)
</details>
