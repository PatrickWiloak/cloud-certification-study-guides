# Google Workspace Organizational Units and User Management

## Overview
Managing organizational structure, user accounts, and lifecycle processes in Google Workspace. This guide covers comprehensive user management, organizational unit design, group administration, and directory synchronization for the Google Workspace Administrator certification.

## Key Concepts

### Organizational Units (OUs)
Organizational Units provide a hierarchical structure for managing users and applying policies across your Google Workspace domain.

**Core Principles:**
- Hierarchical tree structure starting from root organization
- Policies and settings inherit from parent unless explicitly overridden
- Used for department, location, or role-based management
- Maximum depth of 5 levels (including root)
- Can contain users, groups, and child OUs

**Inheritance Model:**
- Child OUs inherit all settings from parent by default
- You can override inherited settings at any level
- More specific (child) settings take precedence
- "Inherit" option explicitly uses parent's setting
- Changes to parent automatically cascade unless overridden

### User Lifecycle States
Understanding user states is critical for the exam:

| State | Description | Access Level | Billing | Recovery Period |
|-------|-------------|--------------|---------|-----------------|
| Active | Full access to all services | Complete | Charged | N/A |
| Suspended | Temporary disable | No login | Charged | Indefinite |
| Deleted | Account removed | No access | Not charged | 20 days |
| Archived (via Vault) | User data retained | No access | Not charged | Per retention policy |

**Exam Tip:** Suspended users still consume licenses and are billed. Use suspension for temporary situations (security incidents, leave of absence), not long-term account disabling.

## Admin Console & gcloud Commands

### User Management Commands

#### Creating Users via gcloud

```bash
# Create a new user with basic information
gcloud identity users create john.doe@example.com \
  --given-name="John" \
  --family-name="Doe" \
  --password="TempP@ssw0rd123" \
  --organization="Engineering"

# Create user with all details
gcloud identity users create jane.smith@example.com \
  --given-name="Jane" \
  --family-name="Smith" \
  --display-name="Jane Smith" \
  --password="SecureP@ss123" \
  --organizational-unit="/Engineering/Development" \
  --recovery-email="jane.personal@gmail.com" \
  --recovery-phone="+1-555-0123"

# Bulk create users from CSV
gcloud identity users create --csv-file=users.csv
```

**CSV Format for Bulk Creation:**
```csv
Email Address,First Name,Last Name,Password,Org Unit Path
user1@example.com,John,Doe,TempPass123,/Engineering
user2@example.com,Jane,Smith,TempPass456,/Sales
```

#### Managing User Accounts

```bash
# Get user information
gcloud identity users describe john.doe@example.com

# Update user details
gcloud identity users update john.doe@example.com \
  --given-name="Jonathan" \
  --organizational-unit="/Engineering/Management"

# Suspend a user account
gcloud identity users suspend john.doe@example.com \
  --reason="Security investigation - ticket #12345"

# Unsuspend a user
gcloud identity users unsuspend john.doe@example.com

# Delete a user (soft delete - 20 day recovery)
gcloud identity users delete john.doe@example.com

# Permanently delete (cannot be recovered)
gcloud identity users delete john.doe@example.com --permanent

# Restore deleted user within 20 days
gcloud identity users undelete john.doe@example.com

# List all users
gcloud identity users list

# List suspended users
gcloud identity users list --filter="suspended=true"

# List users in specific OU
gcloud identity users list --organizational-unit="/Engineering"
```

#### Password Management

```bash
# Reset user password
gcloud identity users reset-password john.doe@example.com \
  --password="NewP@ssw0rd123"

# Force password change on next login
gcloud identity users update john.doe@example.com \
  --change-password-at-next-login

# Update password policy (via Admin API)
# Note: Password policies are typically set in Admin Console
```

**Exam Tip:** Password policies are set at the OU level in Admin Console. You cannot set per-user password policies via gcloud directly.

### Organizational Unit Commands

```bash
# Create an organizational unit
gcloud resource-manager org-units create \
  --display-name="Engineering" \
  --parent="organizations/123456789"

# Create nested OU
gcloud resource-manager org-units create \
  --display-name="Development" \
  --parent="orgUnits/engineering-ou-id"

# List organizational units
gcloud resource-manager org-units list \
  --organization=123456789

# Move user to different OU
gcloud identity users update john.doe@example.com \
  --organizational-unit="/Engineering/Development"

# Get OU details
gcloud resource-manager org-units describe orgUnits/ou-id
```

**OU Naming Best Practices:**
- Use clear, descriptive names: /Engineering, /Sales, /Finance
- Structure by function first, then location: /Sales/US, /Sales/EU
- Keep hierarchy shallow (3-4 levels maximum)
- Avoid frequent restructuring (impacts all policies)

### Group Management Commands

```bash
# Create a group
gcloud identity groups create dev-team@example.com \
  --display-name="Development Team" \
  --description="All development team members"

# Create group with settings
gcloud identity groups create security-team@example.com \
  --display-name="Security Team" \
  --description="Security Operations" \
  --labels="group-type=security,access-level=privileged"

# Add member to group
gcloud identity groups memberships add \
  --group-email="dev-team@example.com" \
  --member-email="john.doe@example.com" \
  --roles="MEMBER"

# Add manager to group
gcloud identity groups memberships add \
  --group-email="dev-team@example.com" \
  --member-email="jane.smith@example.com" \
  --roles="MANAGER"

# Add owner to group
gcloud identity groups memberships add \
  --group-email="dev-team@example.com" \
  --member-email="admin@example.com" \
  --roles="OWNER"

# Remove member from group
gcloud identity groups memberships remove \
  --group-email="dev-team@example.com" \
  --member-email="john.doe@example.com"

# List all groups
gcloud identity groups list

# List group memberships
gcloud identity groups memberships list \
  --group-email="dev-team@example.com"

# List groups a user belongs to
gcloud identity groups memberships search-transitive-groups \
  --member="john.doe@example.com"

# Update group settings
gcloud identity groups update dev-team@example.com \
  --display-name="Senior Development Team" \
  --description="Senior developers only"

# Delete a group
gcloud identity groups delete old-team@example.com
```

### Using Directory API (Advanced)

For programmatic access, use the Directory API with service accounts:

```bash
# List users with custom attributes using API
curl -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://admin.googleapis.com/admin/directory/v1/users?domain=example.com&maxResults=100"

# Get user details with all fields
curl -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://admin.googleapis.com/admin/directory/v1/users/john.doe@example.com?projection=full"

# Create user via API
curl -X POST \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json" \
  -d '{
    "primaryEmail": "newuser@example.com",
    "name": {"givenName": "New", "familyName": "User"},
    "password": "TempP@ssw0rd123",
    "changePasswordAtNextLogin": true,
    "orgUnitPath": "/Engineering"
  }' \
  "https://admin.googleapis.com/admin/directory/v1/users"
```

## User Lifecycle Management

### Complete Provisioning Process

#### 1. Pre-Provisioning Planning

**Requirements Gathering:**
- Determine user's department and role
- Identify required Google Workspace services
- Determine license type needed
- Identify group memberships required
- Set up recovery email/phone

**License Planning:**

| License Type | Best For | Services Included |
|--------------|----------|-------------------|
| Business Starter | Small teams | Gmail, Drive (30GB), Meet (100 participants) |
| Business Standard | Medium businesses | Gmail, Drive (2TB), Meet (150 participants) |
| Business Plus | Security-focused | All Standard + Vault, advanced security |
| Enterprise | Large organizations | Unlimited storage, advanced controls |

#### 2. User Creation Workflow

**Step-by-step process:**

```bash
# Step 1: Create user account
gcloud identity users create john.doe@example.com \
  --given-name="John" \
  --family-name="Doe" \
  --password="TempP@ss123!" \
  --organizational-unit="/Engineering/Development" \
  --recovery-email="john.personal@gmail.com" \
  --change-password-at-next-login

# Step 2: Assign license (via Admin Console or API)
# This requires Admin SDK License Manager API

# Step 3: Add to relevant groups
gcloud identity groups memberships add \
  --group-email="engineering@example.com" \
  --member-email="john.doe@example.com" \
  --roles="MEMBER"

gcloud identity groups memberships add \
  --group-email="developers@example.com" \
  --member-email="john.doe@example.com" \
  --roles="MEMBER"

# Step 4: Configure user aliases (if needed)
# Aliases must be created via Admin Console or Directory API

# Step 5: Set up delegation (if needed for executive assistants)
# Delegation is configured in Admin Console under Gmail settings
```

#### 3. User Profile Management

**Key Profile Attributes:**
- Primary email (immutable once created)
- Name (given name, family name)
- Secondary email addresses
- Phone numbers (work, mobile)
- Organizational unit
- Manager relationship
- Custom schema attributes
- Profile photo

**Updating Profiles:**

```bash
# Update basic information
gcloud identity users update john.doe@example.com \
  --given-name="Jonathan" \
  --family-name="Doe-Smith"

# Move to different OU
gcloud identity users update john.doe@example.com \
  --organizational-unit="/Engineering/Management"
```

**Custom Schemas:**
Custom schemas allow you to add organization-specific attributes:
- Employee ID
- Building/Office location
- Cost center
- Direct manager email
- Hire date
- Department code

These must be created in Admin Console first, then populated via API.

#### 4. Suspension Process

**When to Suspend:**
- Security investigation underway
- Employee on extended leave
- Suspicious account activity
- Pending termination review
- Contract employee between assignments

**Suspension Procedure:**

```bash
# Suspend with reason
gcloud identity users suspend john.doe@example.com \
  --reason="Security investigation ticket #12345 - unauthorized access attempt"

# Verify suspension
gcloud identity users describe john.doe@example.com --format="json" | grep suspended

# Document in notes (via Admin Console)
# Add internal notes explaining suspension reason and expected duration
```

**What Happens When Suspended:**
- User cannot sign in to any Google services
- Email continues to arrive (not rejected)
- User data remains intact
- Group memberships preserved
- License still consumed (still billed)
- Recovery email NOT notified automatically

**Exam Tip:** Suspension does NOT stop billing. If you need to stop billing for an extended period, consider deleting the account instead.

#### 5. Deprovisioning Process

**Complete Offboarding Workflow:**

```bash
# Step 1: Transfer ownership of documents
# Use Data Transfer tool in Admin Console (no gcloud command)
# Transfer Drive files, Calendar events, Google+ pages

# Step 2: Set up email delegation or forwarding
# Configure in Gmail settings in Admin Console
# Options: Auto-forward, delegate access to manager

# Step 3: Back up user data (if needed)
# Use Google Vault or Takeout

# Step 4: Remove from all groups
# List current groups first
gcloud identity groups memberships search-transitive-groups \
  --member="john.doe@example.com"

# Remove from each group
gcloud identity groups memberships remove \
  --group-email="engineering@example.com" \
  --member-email="john.doe@example.com"

# Step 5: Delete user account
gcloud identity users delete john.doe@example.com

# Step 6: Verify deletion
gcloud identity users list --filter="email=john.doe@example.com"
```

**Data Retention After Deletion:**
- User data retained for 20 days (can be restored)
- After 20 days, permanently deleted
- Vault retention policies may preserve data longer
- Shared Drive files remain (ownership transferred)

**Email Aliasing Alternative:**
Instead of deleting, convert to email alias for permanent forwarding:
1. Create group: departed.user@example.com
2. Add current employee as member
3. Delete original user
4. Create alias pointing to group

#### 6. Account Recovery

**Restoring Deleted Users:**

```bash
# Restore within 20 days
gcloud identity users undelete john.doe@example.com

# Verify restoration
gcloud identity users describe john.doe@example.com

# Restore group memberships manually
# Group memberships are NOT automatically restored
```

**What Gets Restored:**
- User account and credentials
- Gmail messages (except auto-deleted)
- Drive files (owned files)
- Calendar events (owned calendars)
- Contacts

**What Doesn't Get Restored:**
- Group memberships (must re-add)
- License assignments (must reassign)
- Custom settings and preferences
- Delegated access
- OAuth tokens (user must reauthorize apps)

## Organizational Unit Design

### OU Structure Best Practices

#### Common OU Structures

**By Department (Most Common):**
```
Root Organization
├── Executive
├── Engineering
│   ├── Development
│   ├── QA
│   └── DevOps
├── Sales
│   ├── Inside Sales
│   └── Field Sales
├── Marketing
├── Finance
├── HR
└── IT
```

**By Location:**
```
Root Organization
├── Headquarters
├── North America
│   ├── US-East
│   ├── US-West
│   └── Canada
├── Europe
│   ├── UK
│   ├── Germany
│   └── France
└── Asia Pacific
    ├── Japan
    └── Australia
```

**Hybrid (Department + Location):**
```
Root Organization
├── Engineering
│   ├── US
│   ├── Europe
│   └── Asia
├── Sales
│   ├── Americas
│   └── EMEA
└── Corporate Functions
    ├── Finance
    └── HR
```

#### OU Design Principles

**1. Plan for Policy Application:**
- Group users with similar policy needs
- Minimize policy exceptions
- Consider security requirements by role
- Think about compliance boundaries

**2. Keep It Simple:**
- 3-4 levels maximum (including root)
- Avoid over-segmentation
- Easier to add OUs later than remove them
- Each OU adds administrative overhead

**3. Consider Growth:**
- Plan for company expansion
- Allow for new departments
- Leave room for reorganization
- Document OU purpose and policies

**4. Align with Business Structure:**
- Match organizational hierarchy
- Reflect reporting structure
- Support business processes
- Enable delegation to managers

#### Policy Inheritance Examples

**Scenario 1: Password Policy**
```
Root: Enforce strong passwords (8+ chars, complexity)
├── Engineering (Inherit): 8+ chars, complexity
│   └── Security Team (Override): 12+ chars, 2FA required
└── Contractors (Override): 8+ chars, 2FA required, 30-day expiry
```

**Scenario 2: Service Availability**
```
Root: All Google Workspace services enabled
├── Full-time Employees (Inherit): All services
└── Contractors (Override): Gmail, Drive, Meet only (no Vault, Cloud Search)
```

**Scenario 3: Drive Sharing**
```
Root: External sharing disabled
├── Marketing (Override): External sharing enabled
└── Finance (Inherit): External sharing disabled
```

**Exam Tip:** When troubleshooting policy issues, always check the entire OU hierarchy from root to the specific OU. A policy set higher in the tree can affect all children.

### Moving Users Between OUs

**Considerations Before Moving:**
- All policies will change to new OU's policies
- May affect service availability
- May change security settings
- Group memberships unchanged
- License assignments unchanged

**Move Command:**
```bash
# Move single user
gcloud identity users update john.doe@example.com \
  --organizational-unit="/Engineering/Management"

# Verify move
gcloud identity users describe john.doe@example.com --format="value(orgUnitPath)"

# Bulk move users from CSV
# CSV format: email,new_ou_path
while IFS=, read -r email ou_path; do
  gcloud identity users update "$email" --organizational-unit="$ou_path"
done < users_to_move.csv
```

**Common Moving Scenarios:**
- Promotion (Developer to Manager)
- Department transfer
- Role change (Employee to Contractor)
- Location change
- Temporary assignment (keep track for reversal)

## Group Management

### Types of Groups

#### 1. Email Lists
- Simple distribution lists
- All members receive emails sent to group
- No access control implications
- Can be external-facing or internal

```bash
# Create email list
gcloud identity groups create announcements@example.com \
  --display-name="Company Announcements" \
  --description="Company-wide announcements list"
```

#### 2. Security Groups
- Used for access control
- Grant permissions to Google Workspace services
- Can be used in Google Cloud IAM
- Should follow least privilege principle

```bash
# Create security group
gcloud identity groups create finance-access@example.com \
  --display-name="Finance System Access" \
  --description="Access to financial systems and data" \
  --labels="type=security,access-level=confidential"
```

#### 3. Collaboration Groups
- For team collaboration (Drive, Sites)
- Members can share documents
- Often tied to projects or teams

#### 4. Dynamic Groups
- Membership based on rules and user attributes
- Automatically updated
- Requires Cloud Identity Premium
- Cannot be managed via gcloud (Admin Console only)

**Dynamic Group Examples:**
- All users in Engineering OU
- All users with manager=john.doe@example.com
- All users with custom field "department=Sales"
- All users created in last 30 days

### Group Membership Roles

| Role | Can View Members | Can Post | Can Manage Members | Can Manage Settings |
|------|------------------|----------|-------------------|---------------------|
| Member | Yes | Yes | No | No |
| Manager | Yes | Yes | Yes | No |
| Owner | Yes | Yes | Yes | Yes |

**Exam Tip:** Managers can add/remove members but cannot change group settings or delete the group. Only Owners can modify settings and delete groups.

### Nested Groups

**Concept:**
Groups can contain other groups as members. This creates a hierarchy that simplifies management.

**Example Structure:**
```
all-engineering@example.com
├── developers@example.com
│   ├── frontend-devs@example.com
│   └── backend-devs@example.com
├── qa-team@example.com
└── devops-team@example.com
```

**Benefits:**
- Easier access management
- Single point for department-wide communication
- Simplified policy application
- Reduces duplicate memberships

**Adding Nested Groups:**
```bash
# Add group as member of parent group
gcloud identity groups memberships add \
  --group-email="all-engineering@example.com" \
  --member-email="developers@example.com" \
  --roles="MEMBER"

# Verify nested structure
gcloud identity groups memberships list \
  --group-email="all-engineering@example.com"
```

**Limitations:**
- Maximum nesting depth: 15 levels
- Circular membership not allowed
- Can impact performance with deep hierarchies
- Troubleshooting complex with multiple levels

### Group Settings and Policies

**Access Settings:**
- Who can join (Anyone, Invited, Request)
- Who can view members
- Who can post messages
- Who can view conversations

**Posting Policies:**
- Moderate all messages
- Moderate first-time posters
- Moderate external messages
- Reject messages from specific senders

**External Access:**
- Allow external members
- Allow posting from non-members
- Publish to web
- Archive messages

**Configuration Example (Admin Console):**
```
Group: security-team@example.com
- Who can join: Only invited users
- Who can post: Only members
- Who can view: Members only
- Moderation: None (trusted team)
- External members: Not allowed
```

## Directory Synchronization

### Google Cloud Directory Sync (GCDS)

GCDS synchronizes users, groups, and other directory data from LDAP-compatible directories (Active Directory, OpenLDAP) to Google Workspace.

#### GCDS Architecture

**Components:**
1. GCDS Application (Windows or Linux)
2. Configuration Manager (GUI tool)
3. Sync rules and filters
4. LDAP connection to source directory
5. Google Workspace API connection

**Sync Flow:**
```
Active Directory/LDAP
        ↓
GCDS reads data via LDAP
        ↓
Applies rules and filters
        ↓
Simulates changes (dry run)
        ↓
Applies changes to Google Workspace
        ↓
Generates sync log
```

#### GCDS Setup Process

**Step 1: Install GCDS**
```bash
# Download from Google (Windows or Linux)
# Install on dedicated sync server (not domain controller)

# Linux installation
sudo dpkg -i google-cloud-directory-sync_*.deb

# Verify installation
/opt/google-cloud-directory-sync/sync-cmd --version
```

**Step 2: Configure LDAP Connection**
- Server address: ldap://dc01.example.com
- Port: 389 (LDAP) or 636 (LDAPS - recommended)
- Base DN: DC=example,DC=com
- Bind account: service-account@example.com
- Authentication: Simple or Kerberos

**Step 3: Configure Google Workspace Connection**
- OAuth 2.0 authentication (recommended)
- Service account with domain-wide delegation
- Required API scopes:
  - https://www.googleapis.com/auth/admin.directory.user
  - https://www.googleapis.com/auth/admin.directory.group

**Step 4: Define Sync Rules**

**User Sync Rules:**
```xml
<!-- Example: Sync all users except service accounts -->
<rule>
  <sync>users</sync>
  <searchBase>OU=Employees,DC=example,DC=com</searchBase>
  <filter>(objectClass=user)</filter>
  <exclude>(description=Service Account)</exclude>
</rule>
```

**Group Sync Rules:**
```xml
<!-- Sync specific groups -->
<rule>
  <sync>groups</sync>
  <searchBase>OU=Distribution Lists,DC=example,DC=com</searchBase>
  <filter>(objectClass=group)</filter>
</rule>
```

**Attribute Mapping:**
```
LDAP Attribute → Google Workspace Field
mail → primaryEmail
givenName → name.givenName
sn → name.familyName
telephoneNumber → phones[0].value
department → organizations[0].department
```

#### GCDS Operations

**Simulation Mode (Dry Run):**
```bash
# Test sync without making changes
/opt/google-cloud-directory-sync/sync-cmd -c config.xml -simulate

# Review simulation log
cat /opt/google-cloud-directory-sync/logs/simulate.log
```

**Actual Sync:**
```bash
# Perform real synchronization
/opt/google-cloud-directory-sync/sync-cmd -c config.xml -sync

# Review sync log
cat /opt/google-cloud-directory-sync/logs/sync.log
```

**Scheduled Sync (Linux cron):**
```bash
# Run sync every 4 hours
0 */4 * * * /opt/google-cloud-directory-sync/sync-cmd -c /etc/gcds/config.xml -sync >> /var/log/gcds-cron.log 2>&1
```

#### GCDS Sync Behaviors

**What GCDS Can Do:**
- Create new users in Google Workspace
- Update existing user attributes
- Create groups and manage membership
- Disable users (suspend in Google Workspace)
- Move users between OUs
- Update email aliases

**What GCDS Cannot Do:**
- Delete users (safety feature - suspend only)
- Create OUs (must pre-create in Google Workspace)
- Sync passwords (use SSO instead)
- Modify Google-only attributes
- Sync calendar events or Drive files

**Exam Tip:** GCDS suspends users by default when they're removed from source directory. It does NOT delete them. This is a safety feature to prevent accidental data loss.

### GCDS Troubleshooting

#### Common Issues and Resolutions

**Issue 1: Sync Failing with Authentication Error**

**Symptoms:**
- "401 Unauthorized" errors
- "Invalid credentials" in logs

**Resolution Steps:**
1. Verify service account has domain-wide delegation
2. Check OAuth scopes are correctly configured
3. Verify service account email in GCDS config matches Google Cloud
4. Regenerate OAuth credentials if needed
5. Test authentication with simple API call

```bash
# Test authentication
curl -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://admin.googleapis.com/admin/directory/v1/users?domain=example.com&maxResults=1"
```

**Issue 2: Users Not Syncing**

**Symptoms:**
- LDAP query returns users
- Users don't appear in Google Workspace
- Sync log shows "0 users synced"

**Resolution Steps:**
1. Check LDAP filter syntax
2. Verify search base DN is correct
3. Check exclusion rules aren't too broad
4. Verify required attributes exist in LDAP
5. Run simulation to see what would sync
6. Check OU exists in Google Workspace

```bash
# Test LDAP query directly
ldapsearch -x -H ldap://dc01.example.com -D "bind-user@example.com" \
  -W -b "OU=Employees,DC=example,DC=com" "(objectClass=user)"
```

**Issue 3: Attribute Mapping Not Working**

**Symptoms:**
- Users sync but fields are empty
- Phone numbers, departments missing

**Resolution Steps:**
1. Verify LDAP attributes are populated in source
2. Check attribute mapping in GCDS config
3. Verify data types match (string, number, date)
4. Check for special characters causing issues
5. Review sync log for specific errors

**Issue 4: Group Membership Not Updating**

**Symptoms:**
- Groups sync but membership incorrect
- Users missing from groups

**Resolution Steps:**
1. Check "member" attribute in LDAP groups
2. Verify user email addresses match
3. Check group sync rules include membership
4. Verify nested group handling configured
5. Run simulation to see membership changes

**Issue 5: Sync Performance Issues**

**Symptoms:**
- Sync takes hours to complete
- GCDS server high CPU/memory usage

**Resolution Steps:**
1. Reduce sync frequency
2. Optimize LDAP queries with better filters
3. Exclude unnecessary attributes
4. Sync users and groups separately
5. Increase GCDS server resources
6. Use indexed LDAP attributes in filters

#### GCDS Best Practices

1. **Always Test in Simulation Mode First**
   - Review changes before applying
   - Verify mappings are correct
   - Check for unexpected deletions/suspensions

2. **Start with Small Pilot Group**
   - Sync single OU first
   - Verify results thoroughly
   - Gradually expand scope

3. **Monitor Sync Logs**
   - Set up alerts for sync failures
   - Review warnings regularly
   - Keep logs for compliance

4. **Use Dedicated Service Account**
   - Don't use admin's personal account
   - Use service account with minimal required permissions
   - Rotate credentials regularly

5. **Schedule Syncs Appropriately**
   - 4-6 hours for most organizations
   - More frequent for high-change environments
   - Avoid peak business hours

6. **Handle Exceptions Properly**
   - Document exclusion rules
   - Create process for exceptions
   - Regular review of excluded users

## Troubleshooting Scenarios

### Scenario 1: User Cannot Access Account After Password Reset

**Situation:**
User John Doe reports he cannot login after IT reset his password. He receives "Invalid credentials" error despite using the correct password IT provided.

**Investigation Steps:**

1. **Verify User Account Status**
```bash
# Check if account is suspended or deleted
gcloud identity users describe john.doe@example.com --format="json"

# Look for: "suspended": true or "suspended": false
```

2. **Check for Account Suspension**
   - If suspended=true, unsuspend the account
   - Review suspension reason in Admin Console notes

3. **Verify Password Reset Was Applied**
   - Check Admin Console audit logs
   - Look for "Change password" event
   - Verify timestamp matches IT's reset time

4. **Check for 2FA Issues**
   - User may have 2FA enabled
   - Password reset doesn't disable 2FA
   - May need to reset 2FA separately

5. **Test Password Reset Again**
```bash
# Reset password and force change on login
gcloud identity users reset-password john.doe@example.com \
  --password="NewTemp123!" \
  --change-password-at-next-login
```

6. **Check Login Attempt Details**
   - Review Security > Investigation tool in Admin Console
   - Look for failed login attempts
   - Check if login from unusual location (may trigger security block)

**Resolution:**
Most commonly, account was suspended during investigation. Unsuspending the account resolved the issue.

**Prevention:**
- Always check account status before password reset
- Document suspension reason
- Create process for unsuspension approval

### Scenario 2: User Not Receiving Emails After OU Move

**Situation:**
Sarah Smith was moved from /Sales OU to /Marketing OU. She reports not receiving any emails for the past 2 hours. Sending emails works fine.

**Investigation Steps:**

1. **Verify OU Move Completed**
```bash
# Check current OU
gcloud identity users describe sarah.smith@example.com \
  --format="value(orgUnitPath)"
```

2. **Check Gmail Service Status**
   - In Admin Console, go to Apps > Google Workspace > Gmail
   - Check if Gmail is enabled for /Marketing OU
   - Compare with /Sales OU settings

3. **Check POP/IMAP Settings**
   - OU policy may have changed POP/IMAP settings
   - If user uses email client, settings may need update

4. **Review Routing Rules**
   - Check if Marketing OU has different routing rules
   - Look for content compliance rules
   - Check for spam filter differences

5. **Test Email Delivery**
```bash
# Send test email from external account
# Send test email from internal account
# Check delivery in Admin Console > Reports > Email Log Search
```

6. **Check User Quotas**
   - Different OUs may have different storage quotas
   - Verify user hasn't exceeded quota

**Resolution:**
Marketing OU had Gmail service disabled for contractors. Sarah was mistakenly moved to /Marketing/Contractors instead of /Marketing/Employees. Moving her to correct OU resolved the issue.

**Prevention:**
- Double-check target OU before moving users
- Document OU purposes and policies
- Implement approval process for OU moves
- Test service access after moves

### Scenario 3: Group Members Not Receiving Messages

**Situation:**
The team-updates@example.com group has 50 members, but only 10 are receiving messages sent to the group. No error messages when sending.

**Investigation Steps:**

1. **Verify Group Membership**
```bash
# List all current members
gcloud identity groups memberships list \
  --group-email="team-updates@example.com"

# Count members
gcloud identity groups memberships list \
  --group-email="team-updates@example.com" | wc -l
```

2. **Check Group Settings**
   - In Admin Console, go to Directory > Groups
   - Find team-updates@example.com
   - Check "Who can post" settings
   - Check "Moderation" settings

3. **Check Individual Member Settings**
```bash
# Check if specific users are suspended
for email in $(gcloud identity groups memberships list \
  --group-email="team-updates@example.com" --format="value(memberEmail)"); do
  gcloud identity users describe "$email" --format="value(suspended)"
done
```

4. **Review Message Delivery Settings**
   - Check if members opted out of group emails
   - Members can set personal delivery preferences
   - Check spam folders of affected users

5. **Check Email Routing**
   - Review Admin Console > Apps > Gmail > Routing
   - Look for rules affecting group delivery
   - Check content compliance rules

6. **Test with Different Sender**
   - Send from external account
   - Send from admin account
   - Check "Who can post" restrictions

7. **Review Audit Logs**
```bash
# Check for recent group changes
# In Admin Console > Reporting > Audit > Groups
```

**Resolution:**
Group was set to "Moderate messages from non-members" and the sender was not a member. Messages were stuck in moderation queue. Changed setting to allow posting from organization members.

**Prevention:**
- Document group posting policies
- Set up moderation notifications
- Regular audit of group settings
- Train users on group vs. individual email

### Scenario 4: GCDS Suspending Active Employees

**Situation:**
After GCDS sync runs, 20 active employees are suspended in Google Workspace. These users are still active in Active Directory.

**Investigation Steps:**

1. **Review GCDS Sync Log**
```bash
# Check most recent sync log
cat /opt/google-cloud-directory-sync/logs/sync.log | grep -i suspended

# Look for pattern in suspended users
```

2. **Check LDAP Query Results**
```bash
# Test LDAP query for affected users
ldapsearch -x -H ldap://dc01.example.com \
  -D "bind-user@example.com" -W \
  -b "OU=Employees,DC=example,DC=com" \
  "(mail=affected.user@example.com)"
```

3. **Verify GCDS Filters**
   - Open GCDS Configuration Manager
   - Review LDAP search base
   - Check exclusion rules
   - Verify required attributes exist

4. **Check for OU Changes in AD**
   - Users may have moved to OU outside sync scope
   - Check if search base includes their current location

5. **Review Attribute Issues**
   - Check if required field missing in AD
   - GCDS may skip users with missing required attributes

6. **Run Simulation to Test**
```bash
# Run simulation to see what would happen
/opt/google-cloud-directory-sync/sync-cmd -c config.xml -simulate

# Review simulation log
cat /opt/google-cloud-directory-sync/logs/simulate.log
```

7. **Unsuspend Affected Users**
```bash
# Create list of affected users
# affected_users.txt contains one email per line

while read email; do
  gcloud identity users unsuspend "$email"
  echo "Unsuspended $email"
done < affected_users.txt
```

**Resolution:**
LDAP search base was changed from "DC=example,DC=com" to "OU=Employees,DC=example,DC=com". 20 users were in "OU=Contractors" which was no longer in scope. GCDS suspended them thinking they were removed from directory. Fixed by updating search base to include both OUs.

**Prevention:**
- Always run simulation before actual sync
- Alert on large numbers of suspensions
- Document LDAP search base and filters
- Test changes in non-production first
- Set up multiple search rules for different OUs

### Scenario 5: User Cannot Access Shared Drive After Department Change

**Situation:**
Mike Jones transferred from Engineering to Sales. He can no longer access the "Product Specs" Shared Drive he needs for his new role, but still has access to old Engineering drives.

**Investigation Steps:**

1. **Verify User's Current OU**
```bash
# Check current organizational unit
gcloud identity users describe mike.jones@example.com \
  --format="value(orgUnitPath)"
```

2. **Check Shared Drive Membership**
   - In Google Drive, open Shared Drive settings
   - Check if mike.jones@example.com is listed
   - Check if his old groups are still members

3. **Verify Group Memberships**
```bash
# List all groups user belongs to
gcloud identity groups memberships search-transitive-groups \
  --member="mike.jones@example.com"
```

4. **Check Shared Drive Access Permissions**
   - Shared Drives grant access to:
     - Individual users
     - Groups
   - Verify if "Product Specs" uses group-based access

5. **Add User to Required Groups**
```bash
# Add to Sales team group
gcloud identity groups memberships add \
  --group-email="sales-team@example.com" \
  --member-email="mike.jones@example.com" \
  --roles="MEMBER"
```

6. **Verify Access After Group Addition**
   - Wait 5-10 minutes for propagation
   - Have user refresh browser
   - Check Shared Drive appears in Drive UI

7. **Review OU-based Restrictions**
   - Check if /Sales OU has Drive sharing restrictions
   - Compare with /Engineering OU policies

**Resolution:**
"Product Specs" Shared Drive granted access to sales-team@example.com group. Mike was removed from this group during his department change. Re-adding him to the group restored access.

**Prevention:**
- Document required group memberships by role
- Create onboarding checklist for department changes
- Use nested groups for cross-functional access
- Automate group assignments based on OU
- Regular access reviews

### Scenario 6: Bulk User Import Failed Partially

**Situation:**
Imported 500 new users via CSV upload. Admin Console shows "250 succeeded, 250 failed" but no clear error message for failed users.

**Investigation Steps:**

1. **Download Failed Users Report**
   - In Admin Console > Users > Add multiple users
   - Download error report CSV
   - Review specific error messages

2. **Common Failure Reasons:**
   - Email format invalid (missing @, special chars)
   - Duplicate email addresses
   - OU path doesn't exist
   - First or last name missing
   - Password doesn't meet complexity requirements
   - License quota exceeded

3. **Check CSV Format**
```csv
# Correct format:
First Name,Last Name,Email Address,Password,Org Unit Path
John,Doe,john.doe@example.com,TempPass123!,/Engineering
```

4. **Validate Email Addresses**
```bash
# Check for common issues in CSV
grep -E "^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$" users.csv
```

5. **Verify OU Paths Exist**
```bash
# List all OUs to verify paths
gcloud resource-manager org-units list

# Check if OUs in CSV match existing OUs
```

6. **Check License Availability**
   - In Admin Console > Billing > Licenses
   - Verify available licenses for user type
   - Purchase more licenses if needed

7. **Fix CSV and Retry Failed Users**
```bash
# Extract failed users from error report
# Fix issues in CSV
# Upload corrected CSV
```

8. **Alternative: Use gcloud for Failed Users**
```bash
# If CSV continues to fail, use gcloud
while IFS=, read -r first last email password ou; do
  gcloud identity users create "$email" \
    --given-name="$first" \
    --family-name="$last" \
    --password="$password" \
    --organizational-unit="$ou" \
    --change-password-at-next-login
done < failed_users.csv
```

**Resolution:**
Review of error report showed:
- 150 users failed: OU "/Engineering/Dev" didn't exist (missing hyphen, should be "/Engineering/Development")
- 100 users failed: Passwords didn't include special characters (requirement from company policy)

Fixed CSV with correct OU paths and stronger passwords, successfully imported all users on second attempt.

**Prevention:**
- Validate CSV format before upload
- Test with 5-10 users first
- Pre-create all OUs referenced in CSV
- Use consistent OU naming
- Document password requirements
- Use template CSV from Google

## Exam Tips and Common Pitfalls

### Key Exam Topics

**High-Priority Topics (Expect Multiple Questions):**
1. OU inheritance model and policy application
2. User lifecycle states (active, suspended, deleted)
3. GCDS behavior and limitations
4. Group types and use cases
5. User recovery process and time limits
6. License management and billing

**Medium-Priority Topics:**
1. Directory API usage
2. Custom schemas
3. Email aliases vs. groups
4. Delegated administration
5. Organizational unit design patterns

### Common Exam Traps

**Trap 1: Suspended vs. Deleted**
- Question implies "remove access immediately"
- Both suspend and delete work, but different implications
- Remember: Suspended users still billed
- Deleted users have 20-day recovery period

**Correct Answer:** If question emphasizes "temporary" or "investigation," choose suspend. If emphasizes "stop billing" or "terminated employee," choose delete.

**Trap 2: GCDS Cannot Delete Users**
- Exam may ask "what happens when user removed from AD"
- GCDS suspends by default, does NOT delete
- This is intentional safety feature

**Correct Answer:** GCDS will suspend the user, not delete them. Manual deletion required if desired.

**Trap 3: OU Move Changes All Policies**
- Question shows user moved to new OU
- User loses access to service
- Need to identify if OU policy difference caused issue

**Correct Answer:** Always check OU-level service availability and settings when troubleshooting after OU moves.

**Trap 4: Group Membership Is Not Restored**
- User deleted and restored within 20 days
- Question asks what needs to be manually reconfigured

**Correct Answer:** Group memberships are NOT automatically restored. Must manually re-add user to groups.

**Trap 5: Maximum OU Depth**
- Question shows complex 7-level OU structure
- Asks if valid

**Correct Answer:** Google Workspace supports maximum 5 OU levels including root. 7 levels would fail.

### Exam Time Management

**Question Types and Time Allocation:**
- Multiple choice (70%): 1-2 minutes each
- Scenario-based (20%): 3-5 minutes each
- Configuration/command (10%): 2-3 minutes each

**Strategy:**
1. Answer easy questions first (flag hard ones)
2. Read scenario questions carefully (details matter)
3. Eliminate obviously wrong answers
4. Watch for "EXCEPT" and "NOT" in questions
5. Leave 10 minutes for review

### Hands-On Practice Checklist

Before the exam, practice these tasks:
- [ ] Create and manage users via Admin Console
- [ ] Create OU structure and apply policies
- [ ] Move users between OUs and verify policy changes
- [ ] Create, configure, and manage groups
- [ ] Add nested groups and verify membership
- [ ] Suspend and restore user accounts
- [ ] Delete and undelete users within 20 days
- [ ] Set up GCDS with test AD (if possible)
- [ ] Review audit logs and reports
- [ ] Configure email routing and compliance rules
- [ ] Set up delegated administration
- [ ] Use Directory API with service account

## Best Practices Summary

### Enterprise-Scale User Management

**1. Automate Everything Possible**
- Use GCDS for directory synchronization
- Script bulk operations
- Implement provisioning workflows
- Use Directory API for integrations

**2. Plan for Scale**
- Design OU structure for growth
- Use groups liberally (easier to manage)
- Implement role-based access
- Document processes thoroughly

**3. Security First**
- Enforce strong password policies
- Enable 2-Step Verification
- Regular access reviews
- Monitor suspicious activity
- Use security groups for sensitive data

**4. Disaster Recovery**
- Regular backups of user data (Vault)
- Document recovery procedures
- Test restoration processes
- Know the 20-day deletion window
- Maintain offline admin access

**5. Compliance and Auditing**
- Enable audit logging
- Regular compliance reviews
- Retention policies in Vault
- User access reports
- Document policy exceptions

**6. User Experience**
- Clear onboarding process
- Self-service password reset
- Quick issue resolution
- Training materials
- Support documentation

**7. Change Management**
- Test in pilot group first
- Communicate changes in advance
- Document all changes
- Have rollback plan
- Monitor after changes

### Google-Recommended Patterns

**User Provisioning:**
1. Create user in appropriate OU
2. Assign license
3. Add to required groups
4. Configure service access
5. Send welcome email with instructions

**User Deprovisioning:**
1. Transfer file ownership
2. Configure email forwarding/delegation
3. Remove from sensitive groups
4. Back up user data if needed
5. Delete or suspend account
6. Document in ticket system

**OU Structure:**
- Start simple, grow as needed
- Use functional grouping (department)
- Consider compliance boundaries
- Plan for delegated administration
- Document policy inheritance

**Group Strategy:**
- Use groups for access control
- Nested groups for hierarchies
- Dynamic groups for automatic membership
- Clear naming conventions
- Regular membership audits

This comprehensive guide covers organizational unit management and user lifecycle operations for Google Workspace Administrator certification. Focus on understanding the inheritance model, user states, GCDS behavior, and troubleshooting methodologies for exam success.
