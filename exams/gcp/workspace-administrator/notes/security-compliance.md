# Google Workspace Security, Access, and Compliance

## Overview
Implementing authentication, access controls, content security, and compliance in Google Workspace. This comprehensive guide covers security settings, compliance features, data protection, mobile device management, and audit capabilities required for the Google Workspace Administrator exam.

## Security Settings and Authentication

### 2-Step Verification (2SV)
Google's implementation of multi-factor authentication providing additional security layer beyond passwords.

**Enforcement Levels:**
- Allow users to opt-in (default)
- Enforce 2SV for all users
- Enforce with grace period (1-4 weeks)
- New user enforcement (require at setup)

**Admin Console Configuration:**
```
Navigation: Security > Authentication > 2-Step Verification
Settings:
- Enforcement: On (with grace period)
- Allow users to trust device: Yes/No
- Frequency: Every sign-in or 30 days
- Methods: Security key, Google Authenticator, Phone, Backup codes
```

**gcloud Commands:**
```bash
# Enable 2SV enforcement (requires Directory API)
gcloud identity groups memberships modify-membership-roles \
  --group-email=security-enforced@example.com \
  --member-email=user@example.com \
  --add-roles=MEMBER

# Check 2SV status for domain
gcloud identity groups describe security-settings@example.com \
  --organization=YOUR_ORG_ID
```

**2SV Methods Priority:**
1. Security keys (FIDO U2F/WebAuthn) - Most secure
2. Google Authenticator app (TOTP)
3. Google Prompt (push notification)
4. Phone SMS/Voice (least secure, should be disabled)
5. Backup codes (recovery only)

**Best Practices:**
- Enforce 2SV for all admin accounts immediately
- Use security keys for super admins
- Disable SMS/Voice methods for high-security users
- Generate backup codes for users before travel
- Monitor 2SV enrollment rates via Security Dashboard

### Password Policies

**Admin Console Configuration:**
```
Navigation: Security > Authentication > Password management
Policy Options:
- Minimum length: 8-100 characters
- Strength: Enforce strong passwords (recommended)
- Reuse: Prevent password reuse (24 previous passwords)
- Expiration: 30, 60, 90, 180 days, or never
- Complexity: Require mix of characters
```

**Password Strength Requirements:**
- Minimum 8 characters (12+ recommended)
- Cannot contain username
- Cannot be common dictionary words
- Mix of uppercase, lowercase, numbers, symbols

**Password Management Commands:**
```bash
# Force password change for user
gam user user@example.com update password random changepassword true

# Reset password for suspended account
gam user user@example.com update password "TempPass123!" changepassword true suspended off

# Audit users with old passwords
gam print users passwordlastset > password-audit.csv
```

**Password Reset Flow:**
- Admin-initiated reset
- User self-service (requires recovery phone/email)
- Account recovery form (requires verification)
- Super admin intervention (last resort)

### Session Control and Timeout

**Session Length Settings:**
```
Navigation: Security > Access and data control > Google session control
Options:
- Session length: 12 hours, 1 day, 2 weeks, or indefinite
- Re-authentication: Require re-login after timeout
- Remember me: Allow users to stay signed in
```

**Web Session Duration:**
- Default: 14 days
- Recommended for high-security: 12 hours
- Mobile apps: 30 days
- OAuth tokens: 6 months (can be revoked)

**Session Management:**
```bash
# Revoke all sessions for user
gam user user@example.com deprovision

# Check active sessions
gam user user@example.com show tokens

# Revoke specific OAuth token
gam user user@example.com delete token <TOKEN_ID>
```

### Context-Aware Access (CAA)

Zero-trust security model allowing conditional access based on context attributes.

**Access Levels Components:**
- IP subnet ranges
- Device policy compliance
- Device encryption status
- OS version requirements
- Screen lock configuration

**Admin Console Configuration:**
```
Navigation: Security > Access and data control > Context-Aware Access
Steps:
1. Create access level (e.g., "Trusted Corporate")
   - IP ranges: 203.0.113.0/24, 198.51.100.0/24
   - Device policy: Require company-owned devices
   - OS version: Minimum Android 11, iOS 14
   - Encryption: Required

2. Assign access level to service
   - Service: Drive, Gmail, Calendar, etc.
   - Access level: Trusted Corporate
   - Action: Allow, Block, or Custom
```

**Common CAA Policies:**

**Corporate Office Access:**
```
Conditions:
- IP subnet: Corporate IP ranges
- Device: Company-managed
- Encryption: Required
Result: Full access to all services
```

**Remote Work Access:**
```
Conditions:
- IP subnet: Any
- Device: Company-managed or BeyondCorp client
- OS: Up-to-date
- 2SV: Enabled
Result: Access to Gmail, Drive, Meet (no admin console)
```

**Contractor Access:**
```
Conditions:
- IP subnet: Restricted
- Device: Any
- Org Unit: Contractors OU
Result: View-only access to shared drives
```

**BeyondCorp Integration:**
```bash
# Requires BeyondCorp Enterprise
# Device trust evaluation based on:
- Device certificates
- Endpoint verification agent
- Device compliance status
- Real-time device posture
```

## Admin Console Security Commands

### Security Center Dashboard

**Navigation Path:**
```
Admin Console > Security > Dashboard
- Security health
- Recommended actions
- Investigation tool
- Alert center
```

**Security Health Metrics:**
- 2SV enrollment rate (target: 100%)
- Suspended user accounts
- Admin activity anomalies
- External file sharing
- Data exfiltration alerts
- Spam rate and trends

### Admin Role Management

**Predefined Admin Roles:**
```
Super Admin - Full control (use sparingly)
User Management Admin - Create/manage users
Groups Admin - Manage groups only
Help Desk Admin - Reset passwords, view audit logs
Services Admin - Enable/disable services
Security Admin - Security settings, investigation tool
```

**Creating Custom Admin Role:**
```
Navigation: Account > Admin roles > Create new role
Example: Limited Security Auditor
Privileges:
- Admin API Privileges > Read (no write)
- Reporting > View audit logs
- Services > View security settings
```

**gcloud Admin Role Commands:**
```bash
# List admin roles
gam print adminroles

# Assign admin role to user
gam user admin@example.com add admin role ROLE_ID

# Remove admin role
gam user admin@example.com delete admin role ROLE_ID

# Audit admin assignments
gam print admins > admin-audit.csv
```

### API Access Control

**Admin Console Configuration:**
```
Navigation: Security > Access and data control > API controls
Settings:
- API access enabled: Yes (for GAM, scripts)
- Trusted apps: Whitelist approved OAuth apps
- Unconfigured apps: Block or allow with warning
- Drive API: Control third-party access
```

**Managing OAuth Apps:**
```bash
# List OAuth tokens by user
gam user user@example.com show tokens

# List all apps with access
gam print tokens > oauth-audit.csv

# Revoke app access domain-wide
gam revoke user user@example.com token <TOKEN_ID>
```

## Compliance Features

### Data Regions and Residency

**Data Locality Requirements:**
Google Workspace stores data in multiple regions; additional products can restrict locations.

**Admin Console Configuration:**
```
Navigation: Data region > Data regions
Available Regions:
- United States
- Europe
- Asia-Pacific (limited services)

Covered Services:
- Gmail
- Drive
- Calendar (metadata only)
- Meet recordings

Not Covered:
- Google Groups (US only)
- Chat history encryption keys (global)
```

**Regional Compliance Considerations:**
- GDPR: Requires European data residency
- HIPAA: US data centers acceptable
- FedRAMP: US government regions only
- APPI (Japan): Asia-Pacific preferred

### Data Loss Prevention (DLP)

**DLP Rule Components:**
1. Content to scan (location)
2. Conditions (what to detect)
3. Actions (what to do)
4. Exceptions (allow lists)

**Predefined Content Detectors:**
```
PII Detectors:
- Credit card numbers (Visa, MC, Amex)
- Social Security Numbers (SSN)
- Driver's license numbers
- Passport numbers
- National ID numbers (various countries)

PHI Detectors:
- Patient medical record numbers
- DEA numbers (prescriptions)
- HIPAA identifiers

Financial:
- Bank account numbers (IBAN, ABA routing)
- Tax ID numbers (EIN, ITIN)

Custom Detectors:
- Regular expressions
- Dictionary matching
- Document properties
```

**Creating DLP Rules - Admin Console:**
```
Navigation: Security > Access and data control > Data protection
Steps:
1. Create rule: "Block SSN Sharing"
2. Scope: Gmail, Drive
3. Conditions:
   - Content: Contains SSN (US)
   - Confidence: Possible, Likely, Very Likely
   - Match threshold: 1 occurrence
4. Actions:
   - Gmail: Block delivery, notify admin
   - Drive: Block sharing, notify user
5. Exceptions:
   - Sender: hr@example.com
   - Recipients: Internal domain only
```

**DLP Rule Examples:**

**PCI Compliance Rule:**
```
Name: "Credit Card Protection"
Scope: Gmail, Drive
Condition: Credit card number (Visa, MC, Amex, Discover)
Threshold: 1 card number
Actions:
- Block external sharing
- Require additional authentication
- Alert security team
Exceptions: finance-team@example.com to processor@bank.com
```

**HIPAA Compliance Rule:**
```
Name: "PHI Protection"
Scope: Gmail, Drive, Chat
Condition: Medical record number OR patient name + diagnosis code
Threshold: 2 identifiers
Actions:
- Block delivery to non-HIPAA accounts
- Quarantine for review
- Log all attempts
Exceptions: Organizational Unit = "Healthcare Staff"
```

**DLP Commands (via GAM):**
```bash
# List DLP rules
gam print dlprules

# Create DLP rule (requires API)
# Use Admin Console UI for complex rules

# Export DLP audit log
gam print drivesettings > drive-dlp-settings.csv

# Monitor DLP alerts
gam report activities application = "drive" parameters = "dlp_action"
```

### Retention Policies

**Retention Rule Types:**
- Default retention (entire domain)
- Custom retention (specific data)
- Legal hold (preserve for litigation)

**Admin Console Configuration:**
```
Navigation: Google Vault > Retention
Create Retention Rule:
1. Name: "Email 7-Year Retention"
2. Entity: example.com (entire org)
3. Applies to: Gmail
4. Duration: 7 years after message date
5. Action after retention: Delete permanently or keep
```

**Retention Policy Examples:**

**Email Retention (SOX Compliance):**
```
Service: Gmail
Duration: 7 years
Scope: All users
Action: Permanent deletion after 7 years
Exceptions: Legal hold overrides
```

**Drive Document Retention:**
```
Service: Drive
Duration: 3 years from creation
Scope: Specific Organizational Unit
Action: Retain, allow user deletion
Exceptions: Files with label "Permanent"
```

**Chat/Meet Retention:**
```
Service: Google Chat, Meet recordings
Duration: 1 year
Scope: All spaces and DMs
Action: Delete after 1 year
Exceptions: Recorded meetings = 5 years
```

**Retention Best Practices:**
- Apply default retention first
- Use custom rules for exceptions
- Document retention schedule
- Test before applying to production
- Consider legal hold requirements

### eDiscovery and Google Vault

**Vault Capabilities:**
- Search and export email, Drive, Chat, Meet
- Legal hold management
- Audit trail for investigations
- Compliance reporting

**Admin Console Configuration:**
```
Navigation: Google Vault (vault.google.com)
Requires: Vault license for users

Access Control:
- Assign "Vault Administrator" role
- Grant matter-specific access
- Audit Vault actions
```

**Creating a Matter:**
```
Steps:
1. Create matter: "Employee Investigation 2024-01"
2. Add description and case number
3. Assign permissions (legal team)
4. Place legal holds on accounts
5. Search for relevant content
6. Export results
```

**Search Query Examples:**

**Email Investigation:**
```
Matter: Contract Dispute
Service: Gmail
Accounts: user1@example.com, user2@example.com
Date Range: 2024-01-01 to 2024-03-31
Terms: "contract" AND ("amendment" OR "termination")
Exclude: Sent to legal@example.com
```

**Drive Document Search:**
```
Matter: Data Breach Investigation
Service: Drive
Organizational Unit: Engineering
Date Range: Last modified 2024-02-15 to 2024-02-20
Terms: filename:*.sql OR content:"customer database"
Include: Deleted files
```

**Vault Commands (via API):**
```bash
# Requires Vault API enabled
# Create matter (requires scripts)
python vault_create_matter.py --name "Investigation 2024"

# Export search results
python vault_export.py --matter-id MATTER_ID --format PST

# List matters
python vault_list_matters.py > matters.json
```

**Legal Hold Best Practices:**
- Place hold before notification
- Document hold scope and duration
- Regular hold reviews (quarterly)
- Notify custodians appropriately
- Release holds promptly when resolved

## Audit and Monitoring

### Admin Audit Logs

**Admin Console Audit Logs:**
```
Navigation: Reporting > Audit and investigation > Admin log events
Events Tracked:
- User creation/deletion
- Admin role assignments
- Security settings changes
- Group modifications
- Service enable/disable
- Data export requests
```

**Important Admin Events:**
- CHANGE_PASSWORD - Password reset by admin
- ASSIGN_ROLE - Admin role granted
- CREATE_USER - New account created
- SUSPEND_USER - Account suspended
- CHANGE_TWO_STEP_VERIFICATION_ENROLLMENT - 2SV modified
- DELETE_USER - Account deleted
- CHANGE_APPLICATION_SETTING - Service settings changed

**Filtering Admin Logs:**
```
Filters:
- Event name: ASSIGN_ROLE
- Actor: admin@example.com
- Date range: Last 7 days
- Organizational Unit: /Engineering
```

**gcloud Audit Commands:**
```bash
# Export admin audit logs
gam report admin start 2024-01-01 end 2024-01-31 > admin-audit.csv

# Filter by event type
gam report admin event CREATE_USER,DELETE_USER start 2024-01-01

# Get specific user's admin activity
gam report admin actor admin@example.com > admin-actions.csv

# Audit role changes
gam report admin event ASSIGN_ROLE,REVOKE_ROLE > role-changes.csv
```

### Login Audit Logs

**Login Events Tracked:**
```
Navigation: Reporting > Audit and investigation > Login audit log
Events:
- Successful logins
- Failed login attempts
- Suspicious activity
- 2SV challenges
- Password changes
- Account recovery attempts
```

**Login Event Types:**
- login_success - Successful authentication
- login_failure - Failed attempt (wrong password)
- login_challenge - 2SV verification
- logout - User logout
- account_disabled_password_leak - Compromised credential
- account_disabled_hijacked - Account hijacking detected
- account_disabled_spamming - Spam activity detected
- account_disabled_generic - Other security reason

**Investigating Suspicious Logins:**
```bash
# Export login failures
gam report login event login_failure start 2024-01-01 > failed-logins.csv

# Check logins from specific IP
gam report login filter "login_ip_address==203.0.113.45" > suspicious-ip.csv

# Audit specific user logins
gam report login user user@example.com > user-login-history.csv

# Find 2SV bypass attempts
gam report login event login_verification_bypassed > 2sv-bypass.csv
```

**Login Anomaly Indicators:**
- Logins from unusual countries
- Multiple failed attempts before success
- Login from impossible travel locations
- User agent changes (desktop to mobile rapidly)
- Login outside normal business hours

### Security Investigation Tool

**Admin Console Investigation Tool:**
```
Navigation: Security > Investigation tool
Use Cases:
- Investigate phishing campaigns
- Track data exfiltration
- Audit external sharing
- Identify compromised accounts
- Review admin activity
```

**Investigation Query Examples:**

**Data Exfiltration Search:**
```
Data Source: Drive log events
Rule: User downloaded files
Filters:
- Actor: user@example.com
- Date: Last 7 days
- Document count: > 50 files
- Visibility: Externally shared
Result: Identify bulk downloads of sensitive files
```

**Phishing Investigation:**
```
Data Source: Gmail log events
Rule: User received messages
Filters:
- Sender: Contains suspicious domain
- Subject: Contains "urgent" OR "verify account"
- Spam confidence: High
- Recipients: Multiple users
Result: Identify phishing campaign targets
```

**External Sharing Audit:**
```
Data Source: Drive log events
Rule: User changed sharing permissions
Filters:
- Visibility: Public on the web OR Anyone with link
- Date: Last 30 days
- Organizational Unit: /Finance
Result: Find improperly shared financial docs
```

**Investigation Tool Commands:**
```bash
# Use Security Investigation API
# Requires Cloud Identity Premium or Workspace Enterprise Plus

# Example: List investigation queries
gcloud alpha identity investigations list

# Run investigation query
gcloud alpha identity investigations query \
  --query-file=investigation-query.json

# Export investigation results
gcloud alpha identity investigations export INVESTIGATION_ID
```

## Data Protection

### Encryption at Rest and In Transit

**Google Workspace Encryption:**
- Encryption at rest: AES-256 (all data)
- Encryption in transit: TLS 1.2+ (all connections)
- Default encryption: Automatic, no configuration needed
- Key management: Google-managed by default

**Encryption Architecture:**
```
Application Layer: Data chunks encrypted
Storage Layer: Additional encryption layer
Hardware Layer: Hardware-based encryption
Key Management: Distributed key storage
```

**Email Encryption:**
- S/MIME for Gmail (requires client certificates)
- TLS enforcement for message delivery
- Opportunistic TLS (best effort)
- Required TLS (force encrypted delivery)

**Admin Console TLS Configuration:**
```
Navigation: Apps > Google Workspace > Gmail > Compliance
Setting: Require secure transport (TLS) compliance
Options:
- Inbound: Reject unencrypted messages
- Outbound: Require TLS or bounce message
- Mutual TLS: Require certificate validation
```

**S/MIME Configuration:**
```
Navigation: Apps > Google Workspace > Gmail > User settings
S/MIME Options:
- Enable S/MIME encryption for outgoing messages
- Require S/MIME signing
- Certificate management: User-uploaded or LDAP
```

**gcloud S/MIME Commands:**
```bash
# Upload S/MIME certificate for user
gam user user@example.com add smimecert file certificate.pem default true

# List user certificates
gam user user@example.com show smimecerts

# Remove expired certificate
gam user user@example.com delete smimecert CERT_ID
```

### Customer-Managed Encryption Keys (CMEK)

**CMEK Overview:**
- Available with Workspace Enterprise Plus
- Keys stored in Google Cloud Key Management Service (KMS)
- Customer controls key lifecycle
- Additional layer beyond Google-managed encryption

**Supported Services:**
- Gmail
- Google Drive
- Google Meet recordings
- Calendar events (metadata)

**CMEK Configuration Prerequisites:**
```
1. GCP project with KMS API enabled
2. KMS key ring and encryption keys
3. Service account with appropriate permissions
4. Link GCP project to Workspace domain
```

**Setting up CMEK:**
```bash
# 1. Create GCP project and enable APIs
gcloud projects create workspace-cmek-project
gcloud services enable cloudkms.googleapis.com

# 2. Create key ring and key
gcloud kms keyrings create workspace-keyring \
  --location=us

gcloud kms keys create gmail-encryption-key \
  --location=us \
  --keyring=workspace-keyring \
  --purpose=encryption

# 3. Grant Workspace service account access
gcloud kms keys add-iam-policy-binding gmail-encryption-key \
  --location=us \
  --keyring=workspace-keyring \
  --member=serviceAccount:workspace-encryption@system.gserviceaccount.com \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# 4. Link in Admin Console
# Navigation: Security > Data protection > Encryption
# Link KMS key to Workspace service
```

**CMEK Key Rotation:**
```bash
# Automatic rotation (recommended)
gcloud kms keys update gmail-encryption-key \
  --location=us \
  --keyring=workspace-keyring \
  --rotation-period=90d \
  --next-rotation-time=2024-04-01T00:00:00Z

# Manual rotation
gcloud kms keys versions create \
  --key=gmail-encryption-key \
  --location=us \
  --keyring=workspace-keyring \
  --primary
```

**CMEK Monitoring:**
```bash
# View key usage
gcloud kms keys list --location=us --keyring=workspace-keyring

# Check key status
gcloud kms keys describe gmail-encryption-key \
  --location=us \
  --keyring=workspace-keyring

# Audit key access
gcloud logging read "resource.type=cloudkms_cryptokey" \
  --limit 50 \
  --format json > kms-audit.json
```

### Data Exfiltration Prevention

**Drive Trust Rules (DLP Integration):**
```
Navigation: Security > Access and data control > Data protection
Rules:
- Block downloads of sensitive files
- Watermark documents
- Restrict copy/paste
- Disable offline access
- Audit print actions
```

**Creating Trust Rules:**
```
Example: Confidential Financial Data
Condition: Label = "Confidential - Finance"
Actions:
- Prevent downloads
- Disable copy/paste
- Apply watermark with user email
- Block sharing outside domain
- Log all access attempts
```

**Gmail Data Exfiltration Controls:**
```
Navigation: Apps > Google Workspace > Gmail > Safety
Settings:
- Attachment compliance: Block .zip, .exe files
- Content compliance: DLP rules for email content
- Objectionable content: Block offensive material
- Approved senders: Whitelist trusted domains
```

**External Sharing Controls:**
```
Navigation: Apps > Google Drive > Sharing settings
Options:
- Sharing outside domain: Warning or block
- Publishing to web: Disabled for sensitive OUs
- Visitor sharing: Require sign-in
- Link sharing default: Restricted to domain
```

**Commands for Monitoring Exfiltration:**
```bash
# Audit external shares
gam report drive user user@example.com > drive-activity.csv

# Find publicly shared files
gam user user@example.com show filelist query "visibility='anyoneWithLink'" > public-files.csv

# Monitor bulk downloads
gam report drive filter "doc_type==file,event_type==download" > downloads.csv

# Check email forwarding rules
gam user user@example.com show forwardingaddresses
```

## Mobile Device Management

### Device Policies

**Mobile Device Management (MDM) Levels:**
- Basic Mobile Management (free)
- Advanced Mobile Management (paid)
- Endpoint verification (desktop devices)

**Admin Console MDM Configuration:**
```
Navigation: Devices > Mobile and endpoints > Settings
Options:
- Enable mobile management
- Device approval: Automatic or require approval
- Enrollment method: User-initiated or admin-provisioned
```

**Device Policy Settings:**
```
Navigation: Devices > Mobile and endpoints > Settings > Device settings
Android Policies:
- Password requirements: Length, complexity, attempts
- Screen lock timeout: 5 minutes recommended
- Encryption: Required (enforced on Android 6+)
- USB debugging: Disabled
- Unknown sources: Blocked
- Device rooting: Block rooted devices
- Camera: Allow or block
- Screen capture: Allow or block

iOS Policies:
- Passcode requirements: 6+ characters
- Touch ID/Face ID: Allowed
- iCloud backup: Blocked for company data
- Jailbreak detection: Block jailbroken devices
- VPN on-demand: Required for access
```

**Creating Device Policy:**
```
Steps:
1. Navigate to: Devices > Mobile and endpoints > Settings
2. Click "Set up" for Advanced management
3. Create policy: "Corporate iOS Devices"
4. Configure settings:
   - Password: Required, 8 characters, alphanumeric
   - Lock screen: 5-minute timeout
   - Failed attempts: Wipe after 10 attempts
   - Apps: Restrict app installation
   - Network: Require VPN for access
5. Apply to: Organizational Unit = /Employees
```

**Device Management Commands:**
```bash
# List enrolled devices
gam print mobile > devices.csv

# Get device details
gam info mobile DEVICE_ID

# Approve pending device
gam approve mobile DEVICE_ID

# Block compromised device
gam update mobile DEVICE_ID action block

# Wipe device data
gam update mobile DEVICE_ID action account_wipe
```

### App Management

**Managed Google Play (Android):**
```
Navigation: Devices > Mobile and endpoints > Apps > Managed Google Play
Capabilities:
- Approve apps for installation
- Push apps to devices
- Configure app settings
- Block specific apps
- Distribute private apps
```

**App Approval Process:**
```
Steps:
1. Navigate to Managed Google Play
2. Search for app (e.g., "Microsoft Office")
3. Review app permissions
4. Approve for organization
5. Assign to users/OUs
6. Configure app settings (if available)
7. Set installation: Required or optional
```

**iOS App Management:**
```
Navigation: Devices > Mobile and endpoints > Apps > iOS apps
Requirements:
- Apple Business Manager (ABM) integration
- VPP (Volume Purchase Program) tokens
- Device enrollment program (DEP)

Capabilities:
- Push purchased apps
- Configure managed app settings
- Remove apps remotely
```

**App Whitelist/Blacklist:**
```
Whitelist Approach (Recommended for BYOD):
- Approve only necessary apps
- Gmail, Drive, Calendar, Meet
- Required business apps
- Block all others by default

Blacklist Approach (Lenient):
- Allow most apps
- Block specific apps: Social media, games
- Monitor app usage
```

**gcloud App Commands:**
```bash
# List approved apps
gam print mobileapps

# Block specific app
gam update mobileapp APP_ID action block

# Audit app installations
gam report mobile applications > app-installations.csv
```

### Remote Wipe and Device Security

**Wipe Options:**
- Account wipe: Remove company data only (recommended)
- Full device wipe: Factory reset entire device
- Selective wipe: Remove specific apps/data

**Admin Console Remote Wipe:**
```
Navigation: Devices > Mobile and endpoints > Devices
Steps:
1. Select device
2. Click "Account wipe" or "Full device wipe"
3. Confirm action (irreversible)
4. Device will wipe on next connection
```

**Wipe Scenarios:**

**Employee Termination:**
```
Action: Account wipe
Reason: Remove company data, preserve personal data
Timing: Immediately upon termination
Follow-up: Verify wipe completed
```

**Lost/Stolen Device:**
```
Action: Full device wipe (if company-owned)
       Account wipe (if BYOD)
Reason: Prevent data breach
Timing: After unsuccessful remote lock
Follow-up: File incident report
```

**Device Commands:**
```bash
# Account wipe (remove company data only)
gam update mobile DEVICE_ID action account_wipe

# Full device wipe (factory reset)
gam update mobile DEVICE_ID action admin_remote_wipe

# Block device access
gam update mobile DEVICE_ID action block

# Check wipe status
gam info mobile DEVICE_ID
```

**Lost Device Workflow:**
1. User reports device lost
2. Admin locates device in console (if tracking enabled)
3. Attempt remote lock with message
4. If not recovered in 24 hours, initiate wipe
5. Document incident for security audit
6. Issue new device with fresh enrollment

**Device Security Best Practices:**
- Require encryption on all devices
- Enforce screen lock timeout (5 minutes max)
- Block rooted/jailbroken devices
- Monitor device compliance daily
- Regular wipe testing (non-production devices)
- Document BYOD policy clearly

## Troubleshooting Scenarios

### Scenario 1: User Unable to Enable 2SV Despite Enforcement

**Symptoms:**
- User reports cannot enable 2-Step Verification
- Receives error: "This option is unavailable for your account"
- 2SV enforcement is enabled domain-wide

**Step-by-Step Resolution:**

1. **Verify Enforcement Settings**
   ```
   Admin Console > Security > Authentication > 2-Step Verification
   Check: Enforcement status and grace period
   ```

2. **Check User's Organizational Unit**
   ```bash
   # Get user's OU
   gam info user user@example.com | grep "Org Unit Path"

   # Verify OU inherits 2SV settings
   # Admin Console > Account > Organizational units
   # Select OU > Security > Check "Inherit" vs "Override"
   ```

3. **Verify Admin Role Restrictions**
   ```
   If user is admin, super admins may need to enable 2SV first
   Check: Security > Authentication > Allow 2SV for admins
   ```

4. **Test with Different 2SV Method**
   ```
   Try: Security key instead of phone
   Some methods may be disabled by policy
   ```

5. **Clear Browser Cache and Test Incognito**
   ```
   Browser issues can prevent 2SV enrollment
   Test in incognito mode or different browser
   ```

6. **Check for Account Restrictions**
   ```bash
   # Verify account isn't suspended or locked
   gam info user user@example.com | grep -i "suspended\|locked"
   ```

**Resolution:**
Issue was OU-level override blocking 2SV. Changed OU setting from "Override: Off" to "Inherit" from parent, allowing enforcement.

**Prevention:**
- Regularly audit OU security settings inheritance
- Document any OU-level overrides with justification
- Test 2SV enrollment in each OU before full rollout

### Scenario 2: DLP Rule Not Blocking Sensitive Data Sharing

**Symptoms:**
- DLP rule configured to block SSN sharing in Gmail
- Test emails with SSNs still being delivered
- No DLP alerts generated

**Step-by-Step Resolution:**

1. **Verify DLP Rule Status**
   ```
   Admin Console > Security > Data protection > Manage rules
   Check: Rule enabled, scope includes Gmail
   Status: Active (not paused)
   ```

2. **Review Rule Conditions**
   ```
   Condition: Contains SSN
   Confidence level: Very Likely, Likely, or Possible
   Threshold: 1 occurrence

   Test case: Does test data match condition?
   Example SSN: 123-45-6789 (should trigger)
   ```

3. **Check Rule Scope and Exceptions**
   ```
   Scope: Verify includes sender's OU
   Exceptions: Check if sender is in exception list

   Common issue: Rule applies to /Finance but sender is in /Engineering
   ```

4. **Review Actions Configuration**
   ```
   Actions: Block delivery (not just "audit only")
   Notifications: Configured correctly

   Issue: If action is "Audit only," emails will still deliver
   ```

5. **Test with Simple Content**
   ```
   Send test email with clear SSN:
   Subject: Test SSN Detection
   Body: My SSN is 123-45-6789

   No other content to eliminate false negatives
   ```

6. **Check DLP Processing Time**
   ```
   DLP rules take 1-4 hours to propagate
   New rules may not apply immediately
   Check rule creation timestamp
   ```

7. **Review Audit Logs**
   ```bash
   # Check if DLP rule is evaluating
   gam report activities application = "rules" > dlp-logs.csv

   # Look for rule evaluations and outcomes
   ```

**Resolution:**
DLP rule was set to "Audit only" instead of "Block." Changed action to "Block message delivery" and tested successfully after 1-hour propagation.

**Prevention:**
- Always test DLP rules in audit mode first
- Create rules in staging OU before production
- Document expected behavior for each rule
- Set up DLP alert notifications to security team

### Scenario 3: Vault Search Returns No Results for Known Data

**Symptoms:**
- Legal matter requires email search
- User confirms emails exist in inbox
- Vault search returns 0 results

**Step-by-Step Resolution:**

1. **Verify User Has Vault License**
   ```bash
   # Check user's licenses
   gam info user user@example.com | grep -i vault

   # Should show: Google Vault or Vault (formerly Google Vault)
   ```

2. **Check Matter Scope**
   ```
   Vault > Matter > Scope
   Verify: Account user@example.com is included
   Data source: Gmail is selected
   ```

3. **Review Date Range**
   ```
   Common issue: Search date range excludes target dates
   Verify: Date range includes email send date
   Time zone: Consider UTC vs local time
   ```

4. **Test Search Terms**
   ```
   Start broad, then narrow:
   1. Remove all terms (search all mail)
   2. Add simple terms ("subject:test")
   3. Add complex terms ("from:sender@example.com AND subject:contract")

   Vault uses Gmail search operators
   ```

5. **Check Email Retention Policy**
   ```
   Admin Console > Google Vault > Retention
   Verify: Retention rule covers date range

   If email expired and deleted, not searchable
   Exception: If legal hold placed before deletion
   ```

6. **Verify Legal Hold Status**
   ```
   Vault > Matter > Holds
   Check: Hold placed on account BEFORE target date
   Hold must be active during email existence
   ```

7. **Check Account History**
   ```bash
   # Verify account existed during time period
   gam info user user@example.com | grep "Created"

   # If account created after emails, check for migration
   ```

8. **Review Data Transfer Events**
   ```bash
   # Check if data was transferred/deleted
   gam report admin event DELETE_USER,TRANSFER_DATA user user@example.com
   ```

**Resolution:**
User's account was deleted and data transferred to manager. Vault search needed to target manager's account where data was transferred, not original user.

**Prevention:**
- Place legal hold before account deletion
- Document data transfer destinations
- Use Vault retention policies proactively
- Educate HR on Vault requirements before terminations

### Scenario 4: Context-Aware Access Policy Blocking Legitimate Users

**Symptoms:**
- Users report "Access Denied" when trying to access Gmail
- Error: "Your organization's security policies prevent access"
- Users are on corporate network and compliant devices

**Step-by-Step Resolution:**

1. **Identify Affected Users**
   ```
   Admin Console > Security > Investigation tool
   Query: Login failures with CAA block reason
   Note: Common pattern (location, device, time)
   ```

2. **Review CAA Access Levels**
   ```
   Admin Console > Security > Context-Aware Access
   Check: Which access level is blocking
   Example: "Corporate Network Only"
   ```

3. **Verify Access Level Conditions**
   ```
   Access Level: Corporate Network
   Conditions:
   - IP ranges: 203.0.113.0/24
   - Device policy: Company-managed
   - OS version: Current

   Test: Does user's IP match range?
   ```

4. **Check User's Current Context**
   ```bash
   # Review login audit log for blocked attempt
   gam report login user user@example.com event login_challenge > login-details.csv

   # Note: IP address, device ID, OS version
   ```

5. **Test IP Range Match**
   ```
   User's IP: 203.0.113.150
   Configured range: 203.0.113.0/24

   Issue: Subnet mask misconfiguration?
   Test: Try accessing from known-good IP
   ```

6. **Verify Device Enrollment**
   ```
   Admin Console > Devices > Mobile and endpoints
   Search: User's device by serial number
   Check: Enrollment status and compliance

   Common issue: Device not enrolled in MDM
   ```

7. **Check for Recent Policy Changes**
   ```bash
   # Audit CAA policy changes
   gam report admin event CHANGE_APPLICATION_SETTING filter "context-aware" > caa-changes.csv

   # Look for recent modifications
   ```

8. **Test with Temporary Exemption**
   ```
   Create test OU: /CAA-Exempt
   Move user to test OU temporarily
   Disable CAA for test OU
   Test access: If successful, confirms CAA issue
   ```

**Resolution:**
Corporate office IP range was updated but CAA policy still had old subnet. Updated access level with new IP range: 203.0.114.0/24. Users could access within 5 minutes.

**Prevention:**
- Document IP ranges in network inventory
- Subscribe to network change notifications
- Test CAA policies in staging before production
- Create emergency override OU for rapid response
- Monitor CAA blocks daily in Investigation tool

### Scenario 5: Mobile Device Wipe Fails to Execute

**Symptoms:**
- Admin initiated remote wipe for lost device
- Device status shows "Wipe pending" for 3+ days
- Device has not connected to network

**Step-by-Step Resolution:**

1. **Verify Device Status**
   ```
   Admin Console > Devices > Mobile and endpoints > Devices
   Check: Last sync time, wipe status
   ```

2. **Check Device Connectivity**
   ```
   Device must connect to internet for wipe command
   Status: Last sync = 3 days ago
   Issue: Device offline, wipe cannot execute
   ```

3. **Review Wipe Type**
   ```
   Wipe types:
   - Account wipe: Requires device connection
   - Full wipe: Requires device connection

   No remote wipe works if device is offline
   ```

4. **Check Network Requirements**
   ```bash
   # Device must reach Google servers
   Domains: android.googleapis.com, google.com
   Ports: 443 (HTTPS), 5228-5230 (Google Cloud Messaging)

   If device on restricted network, wipe may be blocked
   ```

5. **Verify User Account Status**
   ```bash
   # Check if user account is still active
   gam info user user@example.com | grep -i "suspended"

   # Suspended account prevents device communication
   ```

6. **Check for SIM Card Removal**
   ```
   If SIM removed, device has no cellular data
   If Wi-Fi disabled, device cannot connect
   Wipe will execute when connectivity restored
   ```

7. **Review Device Settings**
   ```
   Android: Settings > Accounts > Remove Google Account
   If user removed account, wipe cannot execute

   iOS: Settings > [Name] > Sign Out
   Similar issue on iOS devices
   ```

8. **Implement Alternative Security Measures**
   ```
   While waiting for wipe:
   1. Suspend user account (prevents cloud access)
   2. Reset password (logs out active sessions)
   3. Revoke OAuth tokens
   4. Report device to carrier (lock IMEI)
   5. File police report if stolen
   ```

   ```bash
   # Suspend account
   gam update user user@example.com suspended on

   # Reset password
   gam update user user@example.com password random

   # Revoke all tokens
   gam user user@example.com deprovision
   ```

9. **Document for Security Audit**
   ```
   Record:
   - Device details (IMEI, serial number)
   - Wipe initiation time
   - Last known location (if tracking enabled)
   - Alternative measures taken
   - Resolution date
   ```

**Resolution:**
Device was turned off and never reconnected. Implemented account suspension, password reset, and token revocation to protect data. Wipe remained pending. Device would wipe if powered on and connected.

**Prevention:**
- Enable device tracking before loss
- Educate users to keep devices powered and connected
- Implement "require check-in" policies (weekly)
- Use lost mode/lock screen message before wipe
- Consider full disk encryption for defense in depth

### Scenario 6: Compliance Retention Policy Conflict

**Symptoms:**
- User reports cannot delete old emails from 10 years ago
- Error: "This message cannot be deleted due to retention policy"
- Company policy is 7-year retention

**Step-by-Step Resolution:**

1. **Review Active Retention Policies**
   ```
   Google Vault > Retention
   List all rules:
   - Default retention: 7 years
   - Custom retention: Check for conflicts
   - Legal holds: Active matters
   ```

2. **Check for Legal Holds**
   ```
   Vault > Holds
   Search: user@example.com

   If legal hold active, overrides deletion
   Hold must be released before deletion possible
   ```

3. **Identify Conflicting Rules**
   ```
   Common conflicts:
   - Multiple retention rules on same data
   - Legal hold overrides retention
   - Custom rule extends default retention

   Resolution: Longest retention wins
   ```

4. **Review Rule Hierarchy**
   ```
   Priority order:
   1. Legal hold (highest priority)
   2. Custom retention rules
   3. Default retention rules
   4. User deletion (lowest priority)
   ```

5. **Check Matter Status**
   ```bash
   # If legal matter closed, release hold
   Vault > Matters > Matter name > Holds
   Click: Release hold (if matter closed)

   Note: Requires legal approval
   ```

6. **Verify Retention Rule Dates**
   ```
   Rule: "Email 7-Year Retention"
   Message date: 2013-01-15
   Current date: 2024-01-15 (11 years)

   Calculation: Should be deletable after 2020-01-15
   Issue: Why is retention still active?
   ```

7. **Check for Extended Retention**
   ```
   Admin Console > Vault > Retention > Custom rules
   Look for: Extended retention on specific OU or labels

   Example: Finance OU = 10 years (overrides 7-year default)
   ```

8. **Review Audit Log for Rule Changes**
   ```bash
   # Check retention rule modifications
   gam report admin event CHANGE_DATA_RETENTION_SETTINGS > retention-changes.csv

   # Look for recent rule additions or modifications
   ```

**Resolution:**
User was in Finance OU with custom 10-year retention rule (regulatory requirement). This overrode the default 7-year policy. No conflict; retention working as designed. Educated user on regulatory requirements.

**Prevention:**
- Document all retention policies by OU
- Create retention matrix (service x OU x duration)
- Train users on retention requirements
- Use Vault reporting for retention coverage
- Annual retention policy review

## Exam Tips and Common Pitfalls

### Security Exam Focus Areas

**High-Priority Topics:**
1. **2-Step Verification Enforcement** (20% of security questions)
   - Enforcement methods and grace periods
   - 2SV methods hierarchy (security key > authenticator > SMS)
   - Troubleshooting enrollment failures
   - Admin role requirements for 2SV

2. **Context-Aware Access** (15% of security questions)
   - Access level components (IP, device, encryption)
   - Policy assignment to services
   - BeyondCorp integration
   - Troubleshooting CAA blocks

3. **DLP Rule Configuration** (15% of security questions)
   - Predefined vs custom detectors
   - Condition threshold settings
   - Action types (block, audit, alert)
   - Exceptions and allow lists

4. **Admin Roles and Least Privilege** (10% of security questions)
   - Predefined role capabilities
   - Custom role creation
   - Super admin restrictions
   - Role audit procedures

### Common Exam Traps

**Trap 1: 2SV Enforcement Timing**
```
Question: How long does it take for 2SV enforcement to apply?
Wrong: Immediately for all users
Right: Immediately for new signins, but grace period for existing sessions
Key: Grace period can be 1-4 weeks
```

**Trap 2: DLP Detection Confidence**
```
Question: DLP rule with "Very Likely" confidence will catch all SSNs?
Wrong: Yes, it catches all SSNs
Right: No, only SSNs detected with high confidence; may miss some
Key: Use "Possible" confidence for maximum coverage (more false positives)
```

**Trap 3: Vault Retention vs Legal Hold**
```
Question: User can delete emails after retention period expires?
Wrong: Yes, if retention period expired
Right: No, if legal hold is active (holds override retention)
Key: Legal holds take precedence over retention policies
```

**Trap 4: CAA Device Requirements**
```
Question: CAA requires all devices to be company-owned?
Wrong: Yes, company-owned is required
Right: No, CAA can work with BYOD using BeyondCorp Enterprise
Key: Device policy can require "managed" not necessarily "owned"
```

**Trap 5: Mobile Device Wipe Types**
```
Question: Full device wipe is recommended for BYOD?
Wrong: Yes, protects company data
Right: No, account wipe is recommended (removes company data only)
Key: Full wipe removes personal data; privacy concerns with BYOD
```

**Trap 6: CMEK Key Management**
```
Question: CMEK keys are stored in Google Workspace?
Wrong: Yes, in Workspace Admin Console
Right: No, keys stored in Google Cloud KMS (separate service)
Key: Requires GCP project and KMS configuration
```

### Best Practices for Exam

**Study Techniques:**
1. **Practice in Admin Console** (hands-on required)
   - Create test domain or use trial
   - Configure each security setting
   - Test DLP rules with sample data
   - Practice Vault searches

2. **Memorize Command Paths**
   ```
   2SV: Security > Authentication > 2-Step Verification
   DLP: Security > Access and data control > Data protection
   CAA: Security > Access and data control > Context-Aware Access
   Vault: vault.google.com (separate interface)
   Audit: Reporting > Audit and investigation
   ```

3. **Understand Service Dependencies**
   ```
   CMEK requires: GCP project + Cloud KMS
   CAA requires: Cloud Identity Premium or Workspace Enterprise Plus
   Advanced MDM requires: Endpoint Management add-on
   Vault requires: Vault license per user
   ```

4. **Know Time Delays**
   ```
   DLP propagation: 1-4 hours
   CAA policy changes: 5-15 minutes
   Retention rule changes: 24 hours
   2SV enforcement: Immediate for new logins
   ```

### Compliance Frameworks to Know

**HIPAA (Healthcare):**
- BAA required (Business Associate Agreement)
- DLP for PHI (Protected Health Information)
- Audit logs retention: 6 years
- Encryption at rest and in transit

**GDPR (European Union):**
- Data residency: Europe region
- Right to deletion (Vault considerations)
- Data portability (Takeout)
- Consent management

**SOX (Financial):**
- Email retention: 7 years
- Audit log preservation
- Access controls and segregation of duties
- Change management documentation

**FERPA (Education):**
- Student data protection
- Parental access controls
- Data retention policies
- Third-party sharing restrictions

### Quick Reference Table

| Security Feature | Admin Console Path | License Requirement | Propagation Time |
|-----------------|-------------------|---------------------|------------------|
| 2-Step Verification | Security > Authentication | All editions | Immediate |
| Context-Aware Access | Security > Access and data control | Enterprise Plus | 5-15 minutes |
| DLP Rules | Security > Data protection | Enterprise, Plus | 1-4 hours |
| Vault Retention | vault.google.com | Vault license | 24 hours |
| CMEK | Security > Encryption | Enterprise Plus | Immediate |
| Advanced MDM | Devices > Mobile | Endpoint Management | Immediate |
| Security Investigation | Security > Investigation tool | Enterprise Plus | Real-time |
| Admin Audit Logs | Reporting > Audit | All editions | Real-time |

### Final Exam Preparation Checklist

- [ ] Configure 2SV enforcement in test domain
- [ ] Create DLP rule for PII detection
- [ ] Set up Context-Aware Access policy
- [ ] Perform Vault search and export
- [ ] Review admin audit logs for security events
- [ ] Configure mobile device policy
- [ ] Practice remote device wipe
- [ ] Set up retention policy
- [ ] Use Security Investigation tool
- [ ] Configure S/MIME certificates
- [ ] Test CAA with different access levels
- [ ] Review compliance framework requirements (HIPAA, GDPR, SOX)
- [ ] Memorize admin console navigation paths
- [ ] Understand service license requirements
- [ ] Practice troubleshooting scenarios
