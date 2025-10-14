# Google Cloud Professional Google Workspace Administrator - Fact Sheet

## Quick Reference

**Exam Code:** Professional Google Workspace Administrator
**Duration:** 2 hours (120 minutes)
**Questions:** ~50-60 questions
**Format:** Multiple choice and multiple select
**Passing Score:** Not officially published (~70% estimated)
**Cost:** $200 USD
**Validity:** 2 years
**Prerequisites:** Recommended 3+ years enterprise software experience, 6+ months Google Workspace administration

## Exam Domains

| Domain | Weight | Key Focus |
|--------|--------|-----------|
| Managing organizational units and users | 25% | User lifecycle, groups, OUs, authentication |
| Managing Google Workspace applications | 25% | Gmail, Drive, Meet, Calendar configuration |
| Managing access and authentication | 20% | SSO, MFA, security policies, conditional access |
| Managing content management | 15% | DLP, retention, compliance, governance |
| Managing mail routing and security | 15% | Email routing, spam protection, compliance |

## Core Google Workspace Applications

### Communication and Collaboration

**Gmail**
- Email platform with 30GB storage (Business Standard/Plus)
- Advanced security: S/MIME encryption, phishing protection
- Integrated Chat: Team messaging and spaces
- Smart features: Smart Compose, Smart Reply, nudges
- **[📖 Gmail Admin Help](https://support.google.com/a/topic/9202)** - Admin configuration
- **[📖 Gmail Settings](https://support.google.com/a/answer/2364632)** - User settings and policies

**Google Drive**
- Cloud storage: 30GB - 5TB depending on license
- Shared Drives: Team-owned shared spaces
- File sharing and collaboration
- Version history and file recovery
- **[📖 Drive Admin Help](https://support.google.com/a/topic/2490075)** - Storage management
- **[📖 Shared Drives](https://support.google.com/a/answer/7212025)** - Team drive management

**Google Meet**
- Video conferencing up to 500 participants
- Meeting recordings and live streaming
- Breakout rooms and polls
- Integration with Calendar
- **[📖 Meet Admin Help](https://support.google.com/a/topic/7290350)** - Meeting configuration
- **[📖 Meet Settings](https://support.google.com/a/answer/7303775)** - Security and features

**Google Calendar**
- Shared calendars and resource scheduling
- Working locations and availability
- Meeting rooms and resource management
- Out of office and working hours
- **[📖 Calendar Admin Help](https://support.google.com/a/topic/1034358)** - Calendar management
- **[📖 Resource Management](https://support.google.com/a/answer/1686462)** - Meeting rooms

**Google Chat**
- Direct messages and group conversations
- Spaces for team collaboration
- Bot integration and automation
- File sharing and search
- **[📖 Chat Admin Help](https://support.google.com/a/topic/9402251)** - Chat configuration

### Productivity Applications

**Google Docs, Sheets, Slides**
- Real-time collaboration
- Comments and suggestions
- Version history and recovery
- Template management
- **[📖 Editors Help](https://support.google.com/docs)** - Document editing

**Google Forms**
- Survey and form creation
- Response collection and analysis
- Quiz creation and grading
- Integration with Sheets

**Google Sites**
- Internal website creation
- Project sites and team pages
- Custom domain publishing
- Template galleries

## Administration and Management

### Google Admin Console

**Organization Structure**
- Top-level organization
- Organizational units (OUs) for segmentation
- Groups for permissions and email distribution
- Policy inheritance model
- **[📖 Admin Console Overview](https://support.google.com/a/answer/182076)** - Console navigation
- **[📖 Organizational Structure](https://support.google.com/a/answer/4352075)** - OU design

**User Management**
- User provisioning and deprovisioning
- Profile information and custom attributes
- Password policies and recovery
- License assignment
- **[📖 User Account Management](https://support.google.com/a/topic/14586)** - User admin
- **[📖 Add Users](https://support.google.com/a/answer/33310)** - User creation
- **[📖 Bulk Operations](https://support.google.com/a/answer/40057)** - CSV uploads

**Group Management**
- Google Groups for Business
- Nested groups
- Dynamic membership (requires Cloud Identity Premium)
- External members (allow/block)
- **[📖 Groups Administration](https://support.google.com/a/topic/9400082)** - Group management
- **[📖 Group Settings](https://support.google.com/a/answer/167096)** - Configuration options

**Domain Management**
- Primary and secondary domains
- Domain aliases
- Domain verification
- Custom URLs
- **[📖 Domain Management](https://support.google.com/a/topic/1409901)** - Domain admin

### Security and Access Management

**Authentication**
- Password requirements and complexity
- Password expiration policies
- 2-Step Verification (2SV) enforcement
- Security keys (FIDO U2F/U2F2)
- **[📖 Password Management](https://support.google.com/a/topic/7555707)** - Password policies
- **[📖 2-Step Verification](https://support.google.com/a/answer/175197)** - 2SV setup

**Single Sign-On (SSO)**
- SAML-based SSO configuration
- Third-party identity provider integration
- OAuth 2.0 and OpenID Connect
- SSO profile mapping
- **[📖 SSO Configuration](https://support.google.com/a/answer/60224)** - SAML setup
- **[📖 Third-party SSO](https://support.google.com/a/answer/12032922)** - IdP integration

**Mobile Device Management (MDM)**
- Basic mobile management (included)
- Advanced mobile management (Cloud Identity Premium)
- Device approval and blocking
- Remote wipe capabilities
- **[📖 Mobile Device Management](https://support.google.com/a/topic/24642)** - MDM setup
- **[📖 Device Policies](https://support.google.com/a/answer/7396025)** - Device configuration

**Context-Aware Access**
- Device trust levels
- Location-based access
- IP address allowlisting
- Access level configuration
- **[📖 Context-Aware Access](https://support.google.com/a/answer/9275380)** - Conditional access
- **[📖 Access Levels](https://support.google.com/a/answer/9261439)** - Level configuration

**Security Center**
- Security health dashboard
- Threat detection and investigation
- Security analytics
- Recommended actions
- **[📖 Security Center](https://support.google.com/a/answer/7492330)** - Security monitoring
- **[📖 Investigation Tool](https://support.google.com/a/answer/7575955)** - Security investigation

### Data Protection and Compliance

**Data Loss Prevention (DLP)**
- Predefined content detectors (SSN, credit cards, etc.)
- Custom detectors with regex patterns
- Policy rules and actions
- Scan Gmail, Drive, Chat
- **[📖 DLP Overview](https://support.google.com/a/answer/9646351)** - DLP configuration
- **[📖 DLP Rules](https://support.google.com/a/answer/7047870)** - Rule creation

**Google Vault**
- eDiscovery and legal holds
- Data retention policies
- Email, Drive, Chat retention
- Search and export for legal compliance
- **[📖 Vault Overview](https://support.google.com/vault/answer/2462365)** - Vault admin
- **[📖 Retention Policies](https://support.google.com/vault/answer/2990828)** - Retention setup
- **[📖 Legal Holds](https://support.google.com/vault/answer/2473978)** - Hold management
- **[📖 Search and Export](https://support.google.com/vault/answer/2474474)** - eDiscovery

**Sharing Settings**
- External sharing controls
- Link sharing options
- Trust rules for domains
- Whitelisting and blacklisting
- **[📖 Sharing Settings](https://support.google.com/a/answer/60781)** - Drive sharing
- **[📖 External Sharing](https://support.google.com/a/answer/60262)** - External collaboration

**Audit and Reporting**
- Admin audit logs
- User activity reports
- Login activity monitoring
- BigQuery export for advanced analysis
- **[📖 Audit Logs](https://support.google.com/a/answer/4579579)** - Log access
- **[📖 Reports](https://support.google.com/a/answer/4579451)** - Report types
- **[📖 BigQuery Export](https://support.google.com/a/answer/7233312)** - Advanced analytics

### Gmail Administration

**Mail Routing**
- Default routing and custom routes
- Dual delivery configuration
- Content compliance rules
- Catch-all address setup
- **[📖 Advanced Gmail Settings](https://support.google.com/a/answer/2364632)** - Mail routing
- **[📖 Routing Rules](https://support.google.com/a/answer/2685650)** - Route configuration
- **[📖 Content Compliance](https://support.google.com/a/answer/1346934)** - Compliance rules

**Security and Spam Protection**
- Spam filter settings
- Phishing and malware protection
- Attachment security
- Spoofing and authentication (SPF, DKIM, DMARC)
- **[📖 Spam Protection](https://support.google.com/a/answer/2364632#spam)** - Spam settings
- **[📖 Email Authentication](https://support.google.com/a/answer/33786)** - SPF, DKIM, DMARC
- **[📖 Email Security](https://support.google.com/a/answer/2364632#security)** - Security settings

**Gmail Migration**
- G Suite Migration Tool (GWSMM)
- IMAP migration
- PST file import
- Third-party migration tools
- **[📖 Data Migration](https://support.google.com/a/answer/6351475)** - Migration overview
- **[📖 GWSMM Tool](https://support.google.com/a/answer/6003169)** - Migration tool

## Cloud Identity

**Cloud Identity Free**
- User and group management
- Basic mobile device management
- SAML-based SSO
- 2-Step Verification
- **[📖 Cloud Identity Overview](https://support.google.com/cloudidentity/answer/7319251)** - Identity platform

**Cloud Identity Premium**
- Advanced mobile management
- Automated user provisioning (SCIM)
- Dynamic groups
- Context-aware access
- Security center and investigation
- **[📖 Cloud Identity Premium](https://support.google.com/cloudidentity/answer/7431902)** - Premium features

**Third-Party Integration**
- Active Directory sync (Google Cloud Directory Sync - GCDS)
- Azure AD synchronization
- Okta, Ping Identity integration
- LDAP connector
- **[📖 Directory Sync](https://support.google.com/a/answer/106368)** - GCDS setup
- **[📖 LDAP Integration](https://support.google.com/cloudidentity/answer/9048516)** - LDAP connector

## Command Line and API

**GAM (Google Workspace Admin Manager)**
- Command-line management tool
- Bulk user operations
- Automated administration
- Reporting and auditing
- **[📖 GAM Documentation](https://github.com/GAM-team/GAM)** - Open source tool

**Admin SDK APIs**
- Directory API: User and group management
- Reports API: Activity and usage reporting
- Gmail API: Email management
- Drive API: File management
- **[📖 Admin SDK](https://developers.google.com/admin-sdk)** - API overview
- **[📖 Directory API](https://developers.google.com/admin-sdk/directory)** - User management API

**Google Apps Script**
- Automation and custom workflows
- Custom functions in Sheets
- Menu and sidebar add-ons
- Integration with Workspace services
- **[📖 Apps Script](https://developers.google.com/apps-script)** - Scripting platform

## Common Administration Scenarios

### Scenario 1: New Organization Setup
**Tasks:**
- Configure domain verification
- Create organizational structure
- Set up user accounts and groups
- Configure SSO with corporate IdP
- Enable 2SV for all users
- Set up basic DLP rules

### Scenario 2: Email Migration from Exchange
**Solution:**
- Use G Suite Migration Tool (GWSMM)
- Configure dual delivery during transition
- Set up mail routing rules
- Update MX records gradually
- Monitor migration progress
- Train users on Gmail

### Scenario 3: Security Incident Response
**Actions:**
- Use Security Center to identify threats
- Investigate suspicious activity
- Reset compromised user passwords
- Revoke OAuth tokens
- Review audit logs
- Apply security policies
- Report to management

### Scenario 4: Compliance and eDiscovery
**Implementation:**
- Configure Vault retention policies
- Create legal holds for specific users
- Set up audit logging
- Configure DLP to prevent data leakage
- Export data for legal review
- Document compliance procedures

### Scenario 5: External Collaboration
**Configuration:**
- Enable external sharing with whitelisted domains
- Configure trust rules
- Set up sharing expiration
- Implement DLP for external shares
- Monitor external access with reports
- Train users on secure sharing

## Key Concepts to Master

### Organizational Design
- OU hierarchy planning
- Policy inheritance model
- Group vs OU for permissions
- Delegation of admin roles
- Service enablement per OU

### Authentication Flow
- Primary authentication (Google, SSO)
- 2-Step Verification methods
- Password policies and recovery
- Session management
- Device authentication

### Email Routing
- Inbound routing (MX records)
- Outbound routing (SMTP relays)
- Dual delivery scenarios
- Catch-all addresses
- Custom routing rules

### Data Governance
- Retention policy design
- Legal hold procedures
- DLP rule creation
- Access reviews
- Compliance reporting

### Security Best Practices
- Least privilege access
- Regular security audits
- 2SV enforcement
- Strong password policies
- Mobile device management
- Regular admin account reviews

## Essential gcloud and Admin Commands

**GAM Command Examples:**
```bash
# Create user
gam create user jdoe@example.com firstname John lastname Doe password TempPass123

# Update user
gam update user jdoe@example.com password NewPass456 changepassword on

# Create group
gam create group sales@example.com name "Sales Team"

# Add user to group
gam update group sales@example.com add member jdoe@example.com

# Get user info
gam info user jdoe@example.com

# List all users in OU
gam print users ou /Sales

# Suspend user
gam update user jdoe@example.com suspended on

# Generate reports
gam report users fields email,lastlogintime
```

**Directory API Examples:**
```python
# Python Admin SDK example
from googleapiclient.discovery import build

service = build('admin', 'directory_v1', credentials=creds)

# List users
results = service.users().list(customer='my_customer',
                                maxResults=100).execute()

# Create user
user = {
    'primaryEmail': 'jdoe@example.com',
    'name': {'givenName': 'John', 'familyName': 'Doe'},
    'password': 'TempPass123'
}
service.users().insert(body=user).execute()
```

## Licensing and Editions

### Google Workspace Editions

**Business Starter** ($6/user/month)
- Custom email
- 30GB storage per user
- Meet for up to 100 participants

**Business Standard** ($12/user/month)
- 2TB storage per user
- Meet for up to 150 participants
- Recording and attendance tracking

**Business Plus** ($18/user/month)
- 5TB storage per user
- Meet for up to 500 participants
- Enhanced security (Vault, DLP, S/MIME)

**Enterprise** (Custom pricing)
- Unlimited storage
- Advanced controls
- Premium support
- Advanced security features

**[📖 Workspace Pricing](https://workspace.google.com/pricing)** - Edition comparison

## Study Resources

### Official Google Resources

**[📖 Google Workspace Admin Help](https://support.google.com/a/)** - Complete admin documentation

**[📖 Google Cloud Skills Boost](https://www.cloudskillsboost.google/paths/18)** - Workspace administrator learning path

**[📖 Google Workspace Updates](https://workspaceupdates.googleblog.com/)** - Latest features and changes

**[📖 Google Workspace Training](https://workspace.google.com/training/)** - Official training center

### Hands-On Practice

1. **User and Group Management**
   - Create organizational units
   - Add users manually and via CSV
   - Configure groups with various settings
   - Delegate admin roles
   - Test user lifecycle

2. **Authentication Configuration**
   - Set up 2-Step Verification
   - Configure password policies
   - Test SSO with free IdP trial
   - Configure backup codes
   - Test device authentication

3. **Email Administration**
   - Configure routing rules
   - Set up content compliance
   - Test spam filters
   - Configure SPF/DKIM/DMARC
   - Practice migration scenarios

4. **Security and Compliance**
   - Create DLP rules
   - Set up Vault retention
   - Configure sharing settings
   - Review audit logs
   - Investigate security alerts

5. **Mobile Device Management**
   - Enroll devices
   - Configure device policies
   - Test remote wipe
   - Review device reports

## Exam Tips

### Common Question Topics
- Organizational unit design and inheritance
- SSO configuration and troubleshooting
- Email routing scenarios
- DLP rule creation
- Vault retention and holds
- 2SV enforcement and recovery
- Mobile device management
- Group vs OU decision making
- License assignment and management
- Domain verification and setup

### Focus Areas
- Understand OU vs Group for policy application
- Know all SSO configuration steps
- Master email routing scenarios
- Understand DLP capabilities and limitations
- Know Vault search and export process
- Be familiar with all admin console sections
- Understand license features per edition
- Know security best practices
- Understand delegation model
- Know migration strategies

### Hands-On Skills Required
- Navigate Admin Console efficiently
- Create and manage users and groups
- Configure organizational units
- Set up SSO
- Create DLP and compliance rules
- Use Vault for eDiscovery
- Configure email routing
- Troubleshoot common issues
- Use GAM or API for automation
- Generate and interpret reports

---

**Last Updated:** 2025-01-13
**Certification Focus:** Google Workspace Administration
**Total Documentation Links:** 60+

---

## Notes

This comprehensive fact sheet covers the essential aspects of Google Workspace administration. The certification validates practical skills in managing Google Workspace for organizations. Hands-on experience with the Admin Console is critical for exam success.

For the most current exam information, refer to the official Google Cloud certification page.

**Good luck on your Google Workspace Administrator exam!**
