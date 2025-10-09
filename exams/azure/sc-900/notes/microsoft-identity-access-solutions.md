# Microsoft Identity and Access Management Solutions

## Overview
Microsoft provides comprehensive identity and access management solutions through Azure Active Directory (Azure AD, now Microsoft Entra ID) and related services. This covers authentication, access control, identity protection, and governance capabilities.

## Azure Active Directory (Azure AD)

### Azure AD Overview
- **What it is:** Cloud-based identity and access management service
- **Purpose:** Manage users, groups, devices, and applications
- **Key difference from on-premises AD:**
  - Cloud-native (not based on LDAP)
  - REST API access
  - Modern authentication (OAuth 2.0, SAML, OpenID Connect)
  - Multi-tenant architecture

### Azure AD Editions

#### Azure AD Free
- **Included with:**Microsoft cloud subscriptions
- **Features:**
  - User and group management
  - On-premises directory synchronization
  - Basic reports
  - Self-service password change (cloud users)
  - Single sign-on across Azure, Microsoft 365, and SaaS apps
- **User limit:** No limit
- **Best for:** Small organizations, basic needs

#### Microsoft 365 Apps (Office 365 Apps)
- **Included with:** Microsoft 365 subscriptions
- **Additional features:**
  - Company branding on sign-in pages
  - Self-service password reset for cloud users
  - Group access management
  - Application Proxy for on-premises applications

#### Azure AD Premium P1
- **Price:** Per user/month
- **Additional features:**
  - Hybrid identities (advanced synchronization)
  - Self-service group management
  - Dynamic groups
  - Self-service password reset with on-premises writeback
  - Conditional Access
  - Microsoft Identity Manager (cloud write-back)
- **Best for:** Medium to large organizations

#### Azure AD Premium P2
- **Price:** Per user/month (higher than P1)
- **Additional features (beyond P1):**
  - Azure AD Identity Protection (risk-based policies)
  - Privileged Identity Management (PIM)
  - Access Reviews
  - Entitlement Management
- **Best for:** Large enterprises, high security requirements

### User and Group Management

#### Users
- **Cloud identities:** Created in Azure AD
- **Synchronized identities:** Synced from on-premises AD
- **Guest users:** External users (B2B collaboration)
- **User attributes:** Name, email, job title, department, etc.

#### Groups
- **Security groups:** Control access to resources
- **Microsoft 365 groups:** Collaboration (Teams, SharePoint, etc.)

**Assignment methods:**
- **Assigned:** Manually add members
- **Dynamic user:** Auto-membership based on user attributes
- **Dynamic device:** Auto-membership based on device attributes

**Example dynamic membership rule:**
```
user.department -eq "Sales" -and user.country -eq "US"
```

### External Identities

#### Azure AD B2B (Business-to-Business)
- **Purpose:** Collaborate with external users
- **How it works:**
  - Invite external users as guests
  - Use their own credentials
  - Access your resources
- **Use cases:**
  - Partner collaboration
  - Vendor access
  - Consultant access
- **Licensing:** Free for guest users

#### Azure AD B2C (Business-to-Consumer)
- **Purpose:** Customer-facing applications
- **How it works:**
  - Customers create accounts
  - Social identity providers (Google, Facebook, etc.)
  - Custom branding
  - Scalable to millions of users
- **Use cases:**
  - E-commerce sites
  - Mobile apps
  - Web applications
- **Licensing:** Pay per authentication

## Authentication Capabilities

### Multi-Factor Authentication (MFA)

#### What is MFA?
- **Requires:** Two or more verification methods
- **Something you know:** Password
- **Something you have:** Phone, hardware token
- **Something you are:** Biometric (fingerprint, face)

#### MFA Methods
1. **Microsoft Authenticator app** (recommended)
   - Push notifications
   - Time-based one-time password (TOTP)
   - Passwordless sign-in

2. **SMS text message**
   - Receive code via text
   - Less secure than app

3. **Voice call**
   - Automated call with code
   - Accessibility option

4. **OATH hardware tokens**
   - Physical tokens generating codes
   - Most secure for high-value accounts

5. **FIDO2 security keys**
   - Physical USB/NFC keys
   - Passwordless authentication

#### MFA Configuration
- **Per-user MFA:** Enable for specific users (legacy)
- **Conditional Access:** Policy-based MFA (recommended)
- **Security defaults:** Automatic MFA for all users (basic protection)

### Self-Service Password Reset (SSPR)

#### Overview
- **Purpose:** Users reset passwords without helpdesk
- **Benefits:**
  - Reduced helpdesk costs
  - Faster password recovery
  - Improved user experience

#### Requirements
- **Azure AD Premium P1 or P2** (for on-premises writeback)
- **User registration:** Pre-register authentication methods
- **Verification methods:** At least 2 required

#### Verification Methods
- Mobile app notification
- Mobile app code
- Email
- Mobile phone (SMS/call)
- Office phone
- Security questions (less secure)

#### On-Premises Integration
- **Password writeback:** Reset writes back to on-premises AD
- **Requires:** Azure AD Connect
- **License:** Azure AD Premium P1 or P2

### Password Protection

#### Azure AD Password Protection
- **Global banned password list:** Microsoft-maintained common passwords
- **Custom banned password list:** Organization-specific banned passwords
- **Smart lockout:** Prevents brute force attacks
- **On-premises integration:** Extend to Windows Server AD

#### Smart Lockout
- **Purpose:** Block brute force password attacks
- **How it works:**
  - Tracks failed sign-in attempts
  - Locks account after threshold
  - Uses intelligent detection (location, IP, pattern)
- **Lockout threshold:** Configurable (default: 10 failed attempts)
- **Lockout duration:** Configurable (default: 60 seconds)

### Single Sign-On (SSO)

#### What is SSO?
- **Purpose:** One authentication for multiple applications
- **Benefits:**
  - Improved user experience
  - Reduced password fatigue
  - Better security (fewer passwords)
  - Easier access management

#### SSO Methods

**Password-based SSO:**
- Azure AD stores and replays credentials
- Works with apps that don't support federation
- Browser extension required

**SAML-based SSO:**
- Federation standard
- No password storage in Azure AD
- Common for enterprise applications

**Linked SSO:**
- Link to application without SSO
- Just provides application access in My Apps portal

**Disabled:**
- No SSO configured

#### Enterprise Application Integration
- **Pre-integrated apps:** Azure AD application gallery (1000s of apps)
- **Custom apps:** Register your own applications
- **Application Proxy:** Secure access to on-premises apps

## Access Management Capabilities

### Conditional Access

#### What is Conditional Access?
- **Purpose:** Policy-based access control
- **Concept:** IF this THEN that
- **Example:** IF user is outside corp network THEN require MFA

#### Policy Components

**Assignments (IF):**
- **Users and groups:** Who the policy applies to
- **Cloud apps:** Which applications
- **Conditions:**
  - Sign-in risk (from Identity Protection)
  - Device platform (iOS, Android, Windows, etc.)
  - Locations (trusted locations, countries)
  - Client apps (browser, mobile apps, etc.)
  - Device state (compliant, hybrid joined)

**Access controls (THEN):**
- **Grant access:**
  - Require MFA
  - Require device to be compliant
  - Require hybrid Azure AD joined device
  - Require approved client app
  - Require app protection policy
  - Require password change
  - Terms of use acceptance

- **Session controls:**
  - Use app enforced restrictions
  - Use Conditional Access App Control
  - Sign-in frequency
  - Persistent browser session

**Example Policies:**

```
Policy 1: Require MFA for administrators
- Users: Admin role group
- Apps: All cloud apps
- Grant: Require MFA

Policy 2: Block access from untrusted locations
- Users: All users
- Apps: All cloud apps
- Locations: Untrusted locations
- Grant: Block access

Policy 3: Require compliant device for sensitive apps
- Users: All users
- Apps: Finance application
- Grant: Require device compliance
```

#### What-If Tool
- **Purpose:** Test conditional access policies
- **Function:** Simulate sign-in and show which policies would apply
- **Use:** Before deploying policies to production

### Azure AD Roles and RBAC

#### Azure AD Roles
- **Scope:** Manage Azure AD resources
- **Examples:**
  - **Global Administrator:** Full access to all Azure AD features
  - **User Administrator:** Manage users and groups
  - **Security Administrator:** Manage security features
  - **Application Administrator:** Manage enterprise applications
  - **Conditional Access Administrator:** Manage CA policies
  - **Password Administrator:** Reset passwords for non-admins
  - **Helpdesk Administrator:** Reset passwords, invalidate refresh tokens

#### Azure RBAC vs Azure AD Roles

| Feature | Azure AD Roles | Azure RBAC |
|---------|---------------|------------|
| Scope | Azure AD resources | Azure resources |
| Example resources | Users, groups, apps | VMs, storage, networks |
| Example roles | User Admin, Global Admin | Owner, Contributor, Reader |
| Assignment | Azure AD portal | Azure portal, ARM templates |

#### Custom Roles
- **Azure AD Premium P1/P2 required**
- **Purpose:** Fine-grained permissions
- **Use case:** Specific organizational needs
- **Permissions:** Select from available actions

### Privileged Identity Management (PIM)

#### Overview
- **License:** Azure AD Premium P2
- **Purpose:** Manage, control, and monitor privileged access
- **Principle:** Just-in-time privileged access

#### Key Features

**Just-in-Time Access:**
- No permanent admin access
- Activate role when needed
- Time-limited activation (hours)
- Approval workflow option

**Role Activation:**
1. User requests role activation
2. Optionally approved by designated approver
3. Time-limited role assignment (e.g., 8 hours)
4. MFA required for activation
5. Justification/ticket number required

**Access Reviews:**
- Periodic review of privileged access
- Remove unnecessary assignments
- Ensure least privilege

**Audit and Alerts:**
- Track role activations
- Alert on suspicious activity
- Compliance reporting

**Eligible vs Active:**
- **Eligible:** Can activate the role
- **Active:** Currently has the role
- **Permanent:** Always has the role (try to avoid)

## Identity Protection and Governance

### Azure AD Identity Protection

#### Overview
- **License:** Azure AD Premium P2
- **Purpose:** Detect and respond to identity risks
- **Capabilities:**
  - Automated risk detection
  - Risk-based Conditional Access
  - Investigation and remediation

#### Risk Types

**User Risk:**
- Risk that identity is compromised
- **Detections:**
  - Leaked credentials (found on dark web)
  - Unusual behavior
  - Azure AD threat intelligence
- **Risk levels:** Low, Medium, High

**Sign-in Risk:**
- Risk that sign-in attempt isn't by legitimate user
- **Detections:**
  - Anonymous IP address
  - Atypical travel (impossible travel)
  - Malware-linked IP address
  - Unfamiliar sign-in properties
  - Password spray attack
  - Azure AD threat intelligence
- **Risk levels:** None, Low, Medium, High

#### Risk-Based Policies

**User Risk Policy:**
```
IF user risk is High
THEN require password change
```

**Sign-in Risk Policy:**
```
IF sign-in risk is Medium or High
THEN require MFA
```

#### Risk Investigation
- **Risky users report:** Users flagged as risky
- **Risky sign-ins report:** Sign-ins flagged as risky
- **Risk detections report:** Individual risk events
- **Actions:**
  - Confirm user compromised
  - Dismiss user risk
  - Confirm sign-in safe/compromised

### Access Reviews

#### Overview
- **License:** Azure AD Premium P2
- **Purpose:** Ensure users have appropriate access
- **Scope:** Review group membership, app access, Azure role assignments

#### Review Types
- **Group membership:** Who should be in this group?
- **Application access:** Who should access this app?
- **Azure AD roles:** Who needs this admin role?
- **Azure resource roles:** Who needs access to this resource?

#### Review Process
1. **Create review:**
   - Define scope (group, app, role)
   - Select reviewers (owners, managers, users themselves)
   - Set frequency (one-time, recurring)

2. **Conduct review:**
   - Reviewers approve or deny access
   - Provide justification
   - Automated decisions based on rules

3. **Apply results:**
   - Remove access for denied users
   - Send notifications
   - Audit trail

### Entitlement Management

#### Overview
- **License:** Azure AD Premium P2
- **Purpose:** Automate access request workflows
- **Concept:** Access packages for resources

#### Key Concepts

**Access Package:**
- Bundle of resources (groups, apps, SharePoint sites)
- Policies for requesting access
- Approval workflows
- Time-limited assignments

**Catalog:**
- Container for access packages
- Organized by department or project

**Connected Organizations:**
- External organizations
- B2B collaboration at scale

#### Workflow Example
1. User requests access package
2. Manager approves request
3. User added to groups/apps
4. Access granted for 90 days
5. Access automatically removed after expiration
6. Access review before renewal

### Terms of Use

#### Overview
- **Purpose:** Present legal disclaimers before access
- **Requirements:** Consent to terms
- **Use cases:**
  - Acceptable use policies
  - Privacy statements
  - Compliance requirements

#### Configuration
- Upload PDF document
- Display terms on sign-in
- Require acceptance
- Re-acceptance on schedule
- Per-application terms
- Conditional Access integration

## Best Practices

### Identity Management
1. **Use cloud-only identities** when possible
2. **Implement directory synchronization** for hybrid scenarios
3. **Use Azure AD Connect** for on-premises integration
4. **Regular access reviews** for groups and roles
5. **Leverage dynamic groups** for automatic membership
6. **Plan B2B carefully** with appropriate access controls

### Authentication Security
1. **Enforce MFA for all users** (at minimum, admins)
2. **Use Microsoft Authenticator** as primary MFA method
3. **Enable SSPR** to reduce helpdesk burden
4. **Implement password protection** with custom banned lists
5. **Use passwordless** authentication when possible
6. **Monitor smart lockout** events

### Access Control
1. **Use Conditional Access** instead of per-user MFA
2. **Start with report-only mode** for new CA policies
3. **Block legacy authentication** protocols
4. **Require compliant/managed devices** for sensitive apps
5. **Define trusted locations** accurately
6. **Use What-If tool** before deploying policies

### Privileged Access
1. **Implement PIM** for all privileged roles
2. **No permanent Global Administrators** (use eligible)
3. **Require MFA for role activation**
4. **Require justification** for activation
5. **Regular access reviews** of privileged access
6. **Separate accounts** for admin and regular use

### Identity Protection
1. **Enable Identity Protection** (P2 licenses)
2. **Configure risk-based policies**
3. **Investigate risky sign-ins** promptly
4. **Educate users** about leaked credentials
5. **Use risk data** in Conditional Access
6. **Monitor risk trends** regularly

## Study Tips

### Key Concepts to Master
- Azure AD editions and their features
- External identities (B2B vs B2C)
- MFA methods and configuration
- SSPR process and requirements
- Conditional Access policy components
- PIM just-in-time access
- Identity Protection risk types
- Access Reviews purpose and process

### Common Scenarios
1. **External collaboration** → Azure AD B2B
2. **Customer-facing app** → Azure AD B2C
3. **Password reset without helpdesk** → SSPR
4. **Location-based access control** → Conditional Access
5. **Temporary admin access** → PIM
6. **Detect compromised accounts** → Identity Protection
7. **Audit group membership** → Access Reviews

### Exam Focus Areas
- Azure AD edition features (Free vs P1 vs P2)
- B2B vs B2C scenarios
- MFA methods and when to use each
- Conditional Access policy creation
- Azure AD roles vs Azure RBAC
- PIM capabilities (requires P2)
- Identity Protection risk types
- Which features require which licenses

### Practice Questions
- Which Azure AD edition is required for PIM?
- What's the difference between B2B and B2C?
- What are the components of a Conditional Access policy?
- Which MFA method is most secure?
- What license is needed for SSPR with writeback?
- What does Identity Protection user risk detect?
- How does PIM implement just-in-time access?

### Remember
- Free = Basic features, P1 = Conditional Access, P2 = PIM + Identity Protection
- B2B = Partners/vendors, B2C = Customers
- MFA significantly reduces account compromise
- Conditional Access = Policy-based access control
- PIM = Just-in-time privileged access (requires P2)
- Identity Protection = Automated risk detection (requires P2)
- Always use least privilege principle
