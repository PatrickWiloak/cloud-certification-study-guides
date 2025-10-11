# Google Workspace Applications Management

## Overview
Configuring and managing Google Workspace core applications including Gmail, Drive, Calendar, Meet, and collaboration tools. Application management is a critical exam domain covering service-specific settings, API controls, sharing policies, and third-party integrations.

## Core Application Administration

### Gmail Administration

#### Service-Specific Settings
Navigate to: Admin Console > Apps > Google Workspace > Gmail

**User Settings:**
- Maximum message size: 25MB (default), up to 50MB configurable
- POP and IMAP access: Enable/disable per organizational unit
- Automatic forwarding: Block or allow external forwarding
- Conversation mode: Force on/off or let users decide
- Email delegation: Maximum 25 delegates per mailbox

**Domain Settings:**
- Email allowlist/blocklist: Control which domains can send/receive
- Catch-all address: Route undelivered mail to specific account
- Default routing: Configure inbound and outbound gateways
- Compliance routing: Dual delivery for archiving/journaling
- Content compliance: Filter based on message content/attachments

**Advanced Settings:**
```
Routing > Default routing:
1. Add another rule
2. Select: Inbound or Outbound
3. Configure route: Change route, Reject message, etc.
4. Add recipient/sender conditions
5. Set envelope filter options
```

#### Email Routing Rules

**Types of Routing:**
1. **Default routing**: Applies to all messages
2. **Compliance routing**: For archiving/compliance systems
3. **Recipient address map**: Rewrite recipient addresses
4. **Objectionable content**: Filter spam/malware

**Routing Configuration:**
```
Admin Console > Apps > Gmail > Routing
- Add setting (at OU level)
- Configure conditions:
  * Envelope filter (sender, recipient domains)
  * Content conditions (subject, body, headers)
  * Attachment conditions (type, content)
- Set actions:
  * Change route (modify destination)
  * Add more recipients (Bcc functionality)
  * Modify message (headers, subject, routing)
  * Reject message or quarantine
```

**Common Routing Scenarios:**
| Scenario | Configuration | Use Case |
|----------|---------------|----------|
| Archive all email | Compliance routing + dual delivery | Legal/compliance requirements |
| Block external forwarding | Recipient address map restriction | Data loss prevention |
| Route by department | Envelope filter + change route | Department-specific gateways |
| Quarantine attachments | Content compliance + quarantine | Security scanning |

#### Content Compliance Filters

**Filter Components:**
```
Admin Console > Apps > Gmail > Compliance
1. Content compliance filter:
   - Email messages to affect (inbound/outbound/internal)
   - Add expressions (content match)
   - If expressions match: Actions to take

2. Expression types:
   - Simple content match
   - Advanced content match (regex)
   - Metadata match (headers, size)
   - Attachment content

3. Actions available:
   - Reject message
   - Quarantine message
   - Modify message (prepend subject)
   - Add custom headers
   - Route to different destination
   - Require secure transport (TLS)
```

**Example: Block SSN in Emails:**
```
Expression: Advanced content match
Pattern: \b\d{3}-\d{2}-\d{4}\b
Action: Reject message with notice
Notification: "Message contains sensitive data"
```

### Google Calendar Administration

Navigate to: Admin Console > Apps > Google Workspace > Calendar

#### Calendar Settings

**Sharing Settings:**
- External sharing: On/off for domain
- Share all information vs. only free/busy
- Default access level for new calendars
- Allow users to override sharing settings

**External Calendar Configuration:**
```
Admin Console > Apps > Calendar > Sharing settings
Options:
1. Share all information, outside domain can change
2. Share all information, outside domain can only view
3. Share only free/busy information
4. Do not allow users to share outside the domain
5. Let users decide
```

**Event Settings:**
- Video conferencing: Meet, third-party, or both
- Maximum attendees: Default 100 (can increase to 250)
- Working hours and location defaults
- Resource booking: Conference rooms, equipment
- Event attachment size limit

**Resource Management:**
- Conference room calendars: Auto-accept/decline
- Resource hierarchy: Buildings > Floors > Rooms
- Booking permissions: Internal only vs. domain
- Auto-release: Release unused rooms after X minutes

**Calendar Interop:**
- CalDAV API access: Enable for third-party clients
- iCal export/import capabilities
- Microsoft Exchange interop settings

### Google Drive Administration

Navigate to: Admin Console > Apps > Google Workspace > Drive and Docs

#### Storage Management

**Storage Allocation:**
- Business Starter: 30GB per user
- Business Standard: 2TB per user
- Business Plus: 5TB per user (unlimited with 5+ users)
- Enterprise: 5TB per user (request more as needed)

**Storage Settings:**
```
Admin Console > Apps > Drive > Features
- Shared drives: Enable/disable
- Drive File Stream/Drive for Desktop: Install permissions
- Offline access: Allow/block
- Google Photos: Integration settings
```

**Storage Monitoring:**
```
Admin Console > Reporting > Reports > App reports > Drive
View:
- Total storage used by OU
- Individual user storage consumption
- Storage growth trends
- Shared drive storage allocation
```

#### Shared Drives (Team Drives)

**Shared Drive vs. My Drive:**
| Feature | My Drive | Shared Drive |
|---------|----------|--------------|
| Ownership | Individual user | Team/organization |
| Retention | Leaves with user | Persists independently |
| Access control | Individual file/folder | Drive-level permissions |
| Membership | Not applicable | Manager, Content manager, Contributor, Commenter, Viewer |
| File limit | Unlimited | 400,000 items |
| Member limit | N/A | 600 members per drive |
| Recovery | 30-day trash | Permanent until removed |

**Shared Drive Creation:**
```
Admin Console > Apps > Drive > Manage shared drives
Options:
1. Allow users to create shared drives (Yes/No)
2. Prevent users from deleting shared drives (Recommended)
3. Hide shared drives from users who don't have access
4. Creation restrictions by OU
```

**Shared Drive Settings:**
- Default access level for new members
- External sharing allowed/blocked
- Comments by non-members: Allowed/restricted
- Downloading/copying: Can be blocked
- DLP protection: Apply to shared drives

#### Drive for Desktop Configuration

**Deployment Options:**
```
Admin Console > Apps > Drive > Desktop configuration
Settings:
1. Drive File Stream mode (stream files on-demand)
2. Mirroring mode (sync files locally)
3. Bandwidth throttling
4. Backup and Sync restrictions
5. Auto-update settings
```

**Policy Enforcement:**
- Prevent users from installing: Block via admin
- Force installation: Push via management tools
- Bandwidth limits: Upstream/downstream caps
- Team Drive access: Enable streaming access

#### File Sharing Policies

**Sharing Settings Matrix:**
```
Admin Console > Apps > Drive > Sharing settings
Configure per OU:

1. Sharing options:
   - ON: External sharing allowed
   - OFF: Internal sharing only

2. Access level options when sharing externally:
   - Public on the web (anyone with link)
   - Anyone with the link
   - Specific people only

3. Permissions granted:
   - Viewer (read only)
   - Commenter (view + comment)
   - Editor (full edit access)

4. Additional restrictions:
   - Prevent viewers from downloading
   - Prevent commenters from downloading
   - Warning when sharing outside domain
```

**Link Sharing Configuration:**
```
For files owned by domain:
- Allowlist trusted domains
- Default link sharing: Restricted/Anyone with link
- Distribution restriction: Prevent re-sharing
- Expiration dates: Force on external shares
```

**Example: Lock Down External Sharing:**
```
Admin Console > Apps > Drive > Sharing settings
1. Set sharing options: OFF (sharing outside domain disabled)
2. For necessary OUs: ON with restrictions
3. Access level: "Specific people only"
4. Prevent downloading: Enabled for viewers/commenters
5. Apply to Shared Drives: Yes
```

### Google Docs, Sheets, and Slides

Navigate to: Admin Console > Apps > Google Workspace > Drive and Docs

#### Editor Settings

**Creation and Import:**
- File conversion settings: Office files to Google format
- Office editing mode: Edit Office files directly
- Template gallery: Custom organizational templates
- Add-ons: Allow/block third-party add-ons

**Collaboration Features:**
```
Admin Console > Apps > Drive > Features and applications
Settings:
1. Creating new items in Docs, Sheets, Slides: ON/OFF
2. Creating items in other editors: Forms, Sites, etc.
3. Smart features and personalization:
   - Smart Compose in Docs
   - Smart Reply suggestions
   - Data region selection
```

**Advanced Features:**
- Macros in Sheets: Enable/disable Apps Script
- Smart Canvas features: Smart chips, checklists
- Version history: Always available (30 days minimum)
- Suggesting mode: Tracked changes for collaboration

**Offline Access:**
```
Settings:
1. Require online access: Force online-only editing
2. Allow offline access: Sync files for offline use
3. Offline access duration: Keep files cached
```

### Google Meet Administration

Navigate to: Admin Console > Apps > Google Workspace > Google Meet

#### Meeting Settings

**Meet Organization Settings:**
```
Admin Console > Apps > Meet > Meet video settings

Host management:
1. Let anyone start meetings (ON/OFF)
2. Quick access: Users can start meetings instantly
3. Host controls: Mute participants, end for all
4. Who can join meetings:
   - Domain users only
   - Specific trusted domains
   - Anyone with meeting code
```

**Meeting Controls:**
| Setting | Options | Default | Exam Tip |
|---------|---------|---------|----------|
| Meeting length | Unlimited/Time-limited | Unlimited (Enterprise) | Starter/Standard: 24hr limit |
| Max participants | 100/150/250/500 | Varies by license | Know per-edition limits |
| Recording | ON/OFF per OU | ON | Recordings save to organizer's Drive |
| Live streaming | ON/OFF | OFF (Enterprise only) | Requires YouTube integration |
| Dial-in | ON/OFF | ON | Per-minute charges apply |
| Breakout rooms | ON/OFF | ON | Host must create manually |

**Advanced Meet Settings:**
```
Admin Console > Apps > Meet > Settings

Safety and security:
1. Moderation controls: Host-only screen sharing
2. Safety locks: Automatically mute new entrants
3. External participant restrictions
4. Anonymous users: Block or allow

Meeting artifacts:
1. Recording location: Organizer's Drive
2. Attendance tracking: CSV export available
3. Live captions: Auto-generated subtitles
4. Q&A and polls: Enable/disable
```

**Hardware and Devices:**
- Meet hardware enrollment: Conference room devices
- Calendar resource integration: Room booking
- Device management: Remote control, monitoring
- Peripheral certification: Approved hardware list

#### Meet Quality and Network

**Network Requirements:**
- Outbound bandwidth: 3.2 Mbps for HD video
- Inbound bandwidth: 2.6 Mbps for HD receiving
- Ports required: TCP 443, UDP 19302-19309
- DSCP marking: QoS traffic prioritization

### Google Chat Administration

Navigate to: Admin Console > Apps > Google Workspace > Google Chat

#### Chat Settings

**Chat Organization Settings:**
```
Admin Console > Apps > Chat > Chat settings

1. Chat history:
   - History ON: Messages retained
   - History OFF: 24-hour deletion
   - Let users decide: Per-conversation setting

2. External chat:
   - Allow users to chat outside domain: ON/OFF
   - Allowlist specific external domains
   - Warning banner for external chats

3. DM and group settings:
   - Create groups/DMs: Allow/restrict
   - Maximum group size: 8,000 members
   - Bot access: Allow/block bot integrations
```

**Spaces (Rooms) Management:**
- Create spaces: ON/OFF per OU
- External users in spaces: Allow/block
- Discovery: Spaces visible in directory
- Space managers: Assign management permissions
- Threaded conversations: Organize by topic

**Chat Apps and Bots:**
```
Admin Console > Apps > Chat > Manage Chat apps
Options:
1. Allow users to install apps from Marketplace
2. Allowlist specific apps
3. Block specific apps
4. Audit bot access and permissions
```

## Application Settings and Controls

### Service-Specific APIs

**API Access Configuration:**
```
Admin Console > Security > API controls > Manage Domain-wide Delegation

For each service:
1. Client ID (from OAuth credentials)
2. OAuth scopes required
3. Service account authorization
4. API quota and rate limits
```

**Common API Scopes:**
- Gmail: `https://www.googleapis.com/auth/gmail.settings.basic`
- Drive: `https://www.googleapis.com/auth/drive`
- Calendar: `https://www.googleapis.com/auth/calendar`
- Admin SDK: `https://www.googleapis.com/auth/admin.directory.user`

**API Quotas by Service:**
| Service | Daily Quota | Rate Limit | Exam Note |
|---------|-------------|------------|-----------|
| Gmail API | 1B queries/day | 250 quota units/user/second | Per-user quotas |
| Drive API | 1B queries/day | 1,000 requests/100 seconds/user | Exponential backoff |
| Calendar API | 1M queries/day | 500 queries/100 seconds/user | Higher for domain-wide |
| Admin SDK | 2,500 requests/domain | 1,500 queries/minute | Directory operations |

### Third-Party App Management

Navigate to: Admin Console > Security > API controls > App access control

#### OAuth App Configuration

**App Access Control Types:**
```
1. Trusted apps: Internal apps developed by domain
2. Unconfigured apps: Default (users prompted to authorize)
3. Blocked apps: Cannot access domain data
```

**Configuring App Access:**
```
Admin Console > Security > API controls > App access control

Steps:
1. Click "Configure new app"
2. Search by app name or OAuth client ID
3. Select scopes the app can access:
   - Unrestricted (all scopes)
   - Restricted (specific scopes only)
   - Blocked (no access)
4. Apply to organizational units
5. Review and save
```

**OAuth Scope Categories:**
- Highly restricted scopes: Gmail, Drive full access
- Restricted scopes: Read access to data
- Publicly available scopes: Profile, email info

#### Google Workspace Marketplace

**Marketplace App Management:**
```
Admin Console > Apps > Google Workspace Marketplace apps

Control options:
1. Allow users to install apps: ON/OFF
2. Allowlist: Pre-approved apps users can install
3. Domain installation: Admin-installed for all users
4. Data access: Review requested permissions
5. Remove access: Uninstall and revoke data access
```

**Marketplace App Installation:**
```
Process:
1. Admin reviews app in Marketplace
2. Check data access scopes requested
3. Install for domain or specific OUs
4. Users see app in app launcher
5. Monitor usage in reports
```

**App Data Access Audit:**
```
Admin Console > Security > API controls > Review third-party access

View:
- Apps with domain data access
- Number of users granted access
- OAuth scopes granted
- Last access time
- Revoke access option
```

### Data Loss Prevention (DLP)

Navigate: Admin Console > Security > Data protection > Data Loss Prevention

#### DLP Rules for Applications

**DLP Rule Components:**
```
1. Content conditions:
   - Predefined detectors (SSN, credit card, etc.)
   - Custom regex patterns
   - Document properties
   - Content size thresholds

2. Scope:
   - Gmail (sent/received email)
   - Drive (files, shared drives)
   - Chat (messages, attachments)

3. Actions:
   - Block: Prevent sharing/sending
   - Warn: Alert user before action
   - Audit: Log violation only
```

**Creating DLP Rules:**
```
Admin Console > Security > Data protection > Create rule

Example: Block sharing files with SSN
1. Name: "Block SSN in Drive"
2. Scope: Drive
3. Conditions: Predefined detector "USA Social Security Number"
4. Trigger: 1 or more matches
5. Action: Block external sharing
6. Notification: Alert user and admin
7. Apply to: Entire domain or specific OUs
```

**DLP for Gmail vs. Drive:**
| Feature | Gmail DLP | Drive DLP |
|---------|-----------|-----------|
| Scope | Messages + attachments | Files and folders |
| Actions | Block send, quarantine | Block sharing, warn |
| Scanning | Outbound, inbound, internal | On share, on upload |
| Detectors | Content + attachment | File content only |
| Remediation | Message rejection | Share restriction |

## Collaboration and Sharing Controls

### External Sharing Management

**Sharing Settings Hierarchy:**
```
Most Restrictive                    Most Permissive
│
├─ No external sharing (internal only)
├─ External sharing with allowlisted domains only
├─ External sharing with warning
├─ External sharing allowed (specific people)
├─ External sharing allowed (anyone with link)
└─ Public sharing (anyone on the web)
```

**Domain Allowlisting:**
```
Admin Console > Security > Domain allowlist

Configure:
1. Add trusted domains for sharing
2. Users can share with allowlisted domains without warning
3. Applies to Drive, Docs, Calendar
4. Separate allowlists per service possible
```

### Guest Access and External Collaboration

**Types of External Access:**
1. **Consumer accounts**: Personal Gmail/Google accounts
2. **External domain accounts**: Other Workspace/Cloud Identity
3. **Visitor sharing**: Temporary PIN-based access (Drive)

**Visitor Sharing (Drive PIN Access):**
```
Admin Console > Apps > Drive > Sharing settings
Enable visitor sharing:
1. User shares file with email address
2. Recipient receives PIN code via email
3. Access without Google account signin
4. Time-limited access (configurable)
5. Audit logs track visitor access
```

**External User Management:**
- Track external users in Drive audit logs
- Review external collaborators report
- Bulk removal of external access
- Warning banners for external shares

### Domain-Wide Sharing Policies

**Best Practice Sharing Configuration:**
```
Admin Console > Apps > Drive > Sharing settings

Recommended enterprise settings:
1. Sharing outside domain: ON (with restrictions)
2. Access level: "Specific people only" required
3. Prevent viewers from downloading: ON
4. Warning when sharing outside domain: ON
5. Allowlist trusted partner domains
6. Expiration dates: Required for external shares
7. Shared drive external sharing: More restrictive
```

## Troubleshooting Scenarios

### Scenario 1: Users Cannot Send Email to External Domain

**Symptoms:**
- Users receive bounce messages
- Error: "Address not found" or "Delivery failed"
- Only affects specific external domains

**Troubleshooting Steps:**
1. **Check email blocklist/allowlist:**
   ```
   Admin Console > Apps > Gmail > Spam, phishing, and malware
   - Review blocked senders list
   - Check if domain is blocked
   ```

2. **Verify routing rules:**
   ```
   Admin Console > Apps > Gmail > Routing
   - Review outbound routing rules
   - Check for reject rules matching domain
   - Verify no compliance rules blocking delivery
   ```

3. **Check MX records and DNS:**
   ```
   Use Toolbox (toolbox.googleapps.com):
   - Check MX records for external domain
   - Verify DNS resolution
   - Test SMTP connectivity
   ```

4. **Review delivery issues:**
   ```
   Admin Console > Reporting > Email log search
   - Search for recipient domain
   - Review delivery status
   - Check for TLS requirements
   ```

5. **Test with Email Log Search:**
   ```
   Search criteria:
   - Recipient: user@externaldomain.com
   - Date range: Last 24 hours
   - Review message status and disposition
   ```

**Resolution:**
- Remove domain from blocklist if present
- Modify routing rules to allow delivery
- Add domain to allowlist if needed
- Contact external domain admin if their mail server is blocking

### Scenario 2: Shared Drive Not Appearing for Users

**Symptoms:**
- User is added as member but cannot see shared drive
- Shared drive missing from left navigation in Drive
- Error accessing shared drive link

**Troubleshooting Steps:**
1. **Verify Shared Drives are enabled:**
   ```
   Admin Console > Apps > Drive > Features and applications
   - Check "Shared drives" is ON for user's OU
   - Verify no OU inheritance blocking feature
   - Wait 24 hours for propagation if just enabled
   ```

2. **Check user membership:**
   ```
   In Google Drive (as admin):
   - Navigate to Shared Drives section
   - Click on problematic shared drive
   - Manage members > Verify user is listed
   - Check user's permission level (at least Commenter)
   ```

3. **Review visibility settings:**
   ```
   Admin Console > Apps > Drive > Manage shared drives
   - Verify "Hide shared drives from users who don't have access" setting
   - Check if sharing restrictions prevent visibility
   ```

4. **Test direct access:**
   ```
   - Send user direct link to shared drive
   - Have user click shared drive link in email
   - Check if drive appears after clicking
   - Refresh Drive interface (Ctrl+R)
   ```

5. **Check Drive File Stream/Desktop settings:**
   ```
   If using Drive for Desktop:
   - Settings > Preferences > Google Drive
   - Verify shared drives are syncing
   - Check "Stream files" is enabled
   ```

**Resolution:**
- Enable Shared Drives feature for user's OU
- Re-add user to shared drive membership
- Have user refresh Drive or sign out/in
- Clear browser cache if web access issue

### Scenario 3: Meet Recording Not Saving to Drive

**Symptoms:**
- Recording completes but file not in Drive
- Users cannot find recording link
- Recording notification not received

**Troubleshooting Steps:**
1. **Verify recording is enabled:**
   ```
   Admin Console > Apps > Meet > Meet video settings
   - Check "Recording" is ON for user's OU
   - Verify meeting organizer has recording enabled
   - Confirm license includes recording (not in Starter)
   ```

2. **Check Drive storage quota:**
   ```
   Admin Console > Directory > Users > Select organizer
   - View storage usage
   - Verify organizer has available storage (recordings can be large)
   - Check if storage quota exceeded
   ```

3. **Verify recording permissions:**
   ```
   Only these users can record:
   - Meeting organizer (calendar event creator)
   - Users in same domain as organizer
   - External users: Cannot record

   Check:
   - Who initiated recording
   - If user has permission in their OU
   ```

4. **Check recording location:**
   ```
   Recordings save to:
   - Organizer's My Drive > "Meet Recordings" folder

   Troubleshooting:
   - Check if folder exists in organizer's Drive
   - Verify folder not deleted or renamed
   - Check sharing permissions on folder
   ```

5. **Review Meet audit logs:**
   ```
   Admin Console > Reporting > Audit > Meet
   - Filter by: Recording started/stopped events
   - Check for errors or failures
   - Verify recording completion time
   ```

6. **Processing time consideration:**
   ```
   Recordings take time to process:
   - Short meetings: 5-15 minutes
   - Long meetings: Up to several hours
   - Check again after waiting period
   ```

**Resolution:**
- Enable recording for organizer's OU
- Increase organizer's storage quota
- Ensure correct user is recording (must be organizer or same domain)
- Wait for processing to complete (up to few hours)
- Check "Meet Recordings" folder in organizer's Drive

### Scenario 4: External Users Cannot Access Shared Files

**Symptoms:**
- External users receive "You need permission" error
- Sharing link works for internal users but not external
- External collaborators cannot view/edit files

**Troubleshooting Steps:**
1. **Check external sharing settings:**
   ```
   Admin Console > Apps > Drive > Sharing settings
   - Verify "Sharing outside domain" is ON
   - Check allowed access levels (Anyone with link enabled?)
   - Review OU-specific settings for file owner
   ```

2. **Verify sharing restrictions:**
   ```
   Check if external domain is blocked:
   - Admin Console > Security > Domain allowlist
   - Review if only allowlisted domains permitted
   - Verify external user's domain on allowlist
   ```

3. **Review file-specific sharing:**
   ```
   As file owner or admin:
   - Right-click file > Share > Advanced settings
   - Check "Specific people" vs "Anyone with link"
   - Verify external user's email is added
   - Confirm access level (Viewer/Commenter/Editor)
   ```

4. **Check Shared Drive settings (if applicable):**
   ```
   For files in Shared Drives:
   - Shared Drive settings may override individual file settings
   - Admin Console > Apps > Drive > Shared drive settings
   - Verify external sharing enabled for Shared Drives
   - Check if Shared Drive has more restrictive settings
   ```

5. **Test with different sharing method:**
   ```
   Try alternative sharing approaches:
   - Method 1: Share with specific email (not link)
   - Method 2: Use visitor sharing (PIN code)
   - Method 3: Change to "Anyone with link"
   - Check which method works
   ```

6. **Review download restrictions:**
   ```
   Admin Console > Apps > Drive > Sharing settings
   - Check "Prevent viewers from downloading"
   - This can cause confusion for external users
   - Verify if restriction applies to external shares
   ```

**Resolution:**
- Enable external sharing for file owner's OU
- Add external user's domain to allowlist
- Share file with specific email address (not just link)
- Remove download restrictions for external users
- Use "Anyone with the link" for broader access
- Consider visitor sharing for users without Google accounts

### Scenario 5: Calendar Invites Not Reaching External Attendees

**Symptoms:**
- Internal attendees receive invites, external don't
- External users not showing on attendee list
- Calendar events missing for external participants

**Troubleshooting Steps:**
1. **Verify Calendar external sharing:**
   ```
   Admin Console > Apps > Calendar > Sharing settings
   - Check "External sharing options for primary calendars"
   - Verify not set to "Do not allow users to share outside domain"
   - Should be set to share at least free/busy info
   ```

2. **Check event sharing settings:**
   ```
   In Calendar (as organizer):
   - Open event > Edit
   - Check visibility: "Default visibility" or "Public"
   - Verify external emails added correctly
   - Confirm no typos in email addresses
   ```

3. **Review Gmail delivery:**
   ```
   Calendar invites sent via email:
   - Admin Console > Apps > Gmail > Email log search
   - Search for external attendee email
   - Check if invite was sent and delivered
   - Review any bounce messages
   ```

4. **Check external attendee's email system:**
   ```
   Common issues:
   - Spam/junk folder filtering invites
   - Non-Google calendars may not process properly
   - Firewall blocking calendar.google.com emails
   - External system not accepting .ics attachments
   ```

5. **Verify Calendar interoperability:**
   ```
   For Exchange/Outlook users:
   - Admin Console > Apps > Calendar > Interoperability
   - Check if CalDAV enabled
   - Verify iCal export settings
   - Confirm external system can process Google invites
   ```

6. **Test with different external domains:**
   ```
   Try sending to:
   - Personal Gmail account (should work)
   - Different corporate domain
   - Helps isolate if specific domain issue
   ```

**Resolution:**
- Enable external calendar sharing in admin settings
- Verify Gmail not blocking calendar invite emails
- Resend invite after confirming settings
- Ask external user to check spam folder
- Use "Send invite email" option when adding guests
- For persistent issues: Send meeting details via regular email

### Scenario 6: Third-Party App Cannot Access Drive Files

**Symptoms:**
- OAuth authorization fails for app
- App shows "Access denied" or "Insufficient permissions"
- Previously working app stops accessing files

**Troubleshooting Steps:**
1. **Check app access control settings:**
   ```
   Admin Console > Security > API controls > App access control
   - Search for app by name or Client ID
   - Verify app is not in "Blocked" state
   - Check if app is "Trusted" or "Unconfigured"
   ```

2. **Review OAuth scopes granted:**
   ```
   Admin Console > Security > API controls > App access control
   - Select the app
   - Review "Scopes" section
   - Verify app has necessary Drive scopes:
     * https://www.googleapis.com/auth/drive
     * https://www.googleapis.com/auth/drive.file
   - Check if scopes match app requirements
   ```

3. **Verify domain-wide delegation (if needed):**
   ```
   Admin Console > Security > API controls > Domain-wide delegation
   - Check if app service account is authorized
   - Verify Client ID matches
   - Confirm OAuth scopes are listed
   - Format: comma-separated scope URLs
   ```

4. **Check API access restrictions:**
   ```
   Admin Console > Security > API controls > Settings
   - Verify "Drive API" is enabled
   - Check if unrestricted API access allowed
   - Review if specific apps allowlisted
   ```

5. **Review user-level app authorization:**
   ```
   Users may need to re-authorize:
   - User account > Security > Third-party apps with account access
   - Check if app is listed
   - Remove and re-authorize if present
   - Try OAuth flow again
   ```

6. **Test with different user account:**
   ```
   Try app access with:
   - Admin account
   - Regular user in different OU
   - Helps determine if OU-specific restriction
   ```

**Resolution:**
- Change app from "Blocked" to "Trusted" in app access control
- Grant necessary OAuth scopes to app
- Configure domain-wide delegation if app requires it
- Enable Drive API in API controls
- Have users re-authorize app access
- Wait up to 24 hours for access changes to propagate

## Common Admin Console Commands and Configurations

### Gmail Admin Tasks

**Configure Email Delegation:**
```
Admin Console > Apps > Gmail > User settings
1. Enable "Email delegation"
2. Users can add delegates via Settings > Accounts
3. Maximum 25 delegates per account
4. Delegates can read, send, delete (as delegator)
5. Cannot delete email permanently (admin can)
```

**Set Up Catch-All Address:**
```
Admin Console > Apps > Gmail > Default routing
1. Add setting for organizational unit
2. For unrecognized addresses: Change route
3. Route to: catchall@domain.com
4. Save configuration
5. Test with non-existent address
```

**Configure Email Archiving (Dual Delivery):**
```
Admin Console > Apps > Gmail > Compliance > Content compliance
1. Create new rule: "Archive All Email"
2. Messages to affect: All email
3. Add action: "Add more recipients"
4. Additional recipient: archive@domain.com
5. Do not change route: Checked
6. Save rule (applies in 1-24 hours)
```

### Drive Admin Tasks

**Bulk Transfer Drive Ownership:**
```
Admin Console > Apps > Google Workspace > Drive and Docs
Transfer tool for offboarding:
1. Admin Console > Directory > Users > Deleted users
2. Select user > Transfer data
3. Choose destination user for Drive files
4. Select "Transfer ownership"
5. Monitor transfer status
6. Note: Can take 24+ hours for large accounts
```

**Create Organization-Wide Shared Drive:**
```
As admin with Drive privileges:
1. Google Drive > Shared drives > New
2. Name the shared drive
3. Add members (can add groups)
4. Set permissions:
   - Manager: Full control
   - Content manager: Add/edit/move files
   - Contributor: Add files only
   - Commenter: View and comment
   - Viewer: View only
5. Configure sharing settings
```

**Restore Deleted Files (Admin Recovery):**
```
Admin Console > Apps > Drive > Manage data
Use Google Vault (if available):
1. Navigate to vault.google.com
2. Search for matter or create new
3. Add user's Drive account
4. Search by date/keywords
5. Export or restore files

Without Vault (within 25 days):
1. Contact Google Support
2. Provide user email and file details
3. Support can attempt recovery
```

### Meet Admin Tasks

**Configure Default Meeting Settings:**
```
Admin Console > Apps > Meet > Meet video settings
1. Host management:
   - Who can create meetings: Domain users
   - Quick access: ON (instant meetings)
   - Host controls: Enabled
2. Meeting features:
   - Recording: ON (save to Drive)
   - Attendance tracking: ON
   - Live streaming: ON (if licensed)
3. Safety settings:
   - Moderate new participants: OFF (auto-admit domain)
   - External participants: Require knock
```

**Set Up Meet Hardware:**
```
Admin Console > Devices > Chrome devices > Meet hardware
1. Enroll device using provisioning code
2. Associate with calendar resource (room)
3. Configure settings:
   - Auto-join scheduled meetings
   - Display calendar on screen
   - Peripheral devices
4. Apply device policy
5. Monitor device status
```

## Exam Tips and Common Pitfalls

### Application-Specific Traps

**Gmail Administration:**
- Routing rules process in order: First match wins, order matters
- Content compliance can take up to 24 hours to propagate
- Email delegation: Delegates see "sent by" not "sent as"
- Maximum message size: 50MB including headers (actual ~25MB)
- SMTP relay requires IP allowlisting and authentication

**Drive Management:**
- Shared drive ownership: No individual owner, persists after member leaves
- My Drive: Files owned by user, leaves with user unless transferred
- Shared drive limits: 400,000 items, 600 members per drive
- Storage quotas: Shared drives count against domain pool, not individual
- Shared with me: Not the same as Shared Drives (common confusion)

**Calendar Features:**
- External sharing: Must be enabled at domain level first
- Resource calendars: Auto-accept requires specific configuration
- Working location: Different from working hours
- Calendar delegation: Different from event delegation
- Free/busy vs. full details: Choose based on privacy needs

**Meet Limitations:**
- Recording: Only organizer or same-domain users can record
- License requirements: Starter cannot record, Standard limited
- Live streaming: Requires Enterprise edition + YouTube
- Breakout rooms: Must be created by host, cannot be pre-assigned
- Dial-in: Incurs per-minute charges, not included in license

**Chat Configuration:**
- History OFF: Messages deleted after 24 hours (cannot be recovered)
- External chat: Shows warning banner, logs in audit
- Spaces vs. DMs: Different retention and management
- Maximum group size: 8,000 members, but performance degrades
- Bot permissions: Review carefully, can access all conversation data

### Common Exam Scenarios

**Scenario Type 1: Sharing Not Working**
- First check: Is external sharing enabled for OU?
- Second check: Is domain allowlisted/blocked?
- Third check: Are file-level settings more restrictive?
- Remember: Most restrictive setting wins (OU > domain > file)

**Scenario Type 2: Service Not Available**
- Check: Is service enabled for user's OU?
- Verify: License includes the feature (edition-specific)
- Consider: Propagation time (up to 24 hours)
- Review: OU hierarchy and inheritance

**Scenario Type 3: API/Integration Issues**
- Verify: App access control settings
- Check: OAuth scopes granted
- Confirm: API quotas not exceeded
- Review: Domain-wide delegation if needed

**Scenario Type 4: Email Routing Problems**
- Order matters: Routing rules process top to bottom
- Compliance rules: Run in addition to routing rules
- Catch-all: Only works for unrecognized addresses
- Test: Email log search is your best friend

### Feature Comparison Quick Reference

**Sharing Levels (Most to Least Permissive):**
1. Public on web (anyone, searchable)
2. Anyone with link (no sign-in required)
3. Domain with link (domain users only)
4. Specific people (named individuals)

**Permission Levels (Most to Least Access):**
1. Owner (Drive): Full control, can delete
2. Manager (Shared Drive): Full control, manage members
3. Content Manager (Shared Drive): Add, edit, move, delete content
4. Contributor (Shared Drive): Add files, edit own files
5. Editor: Full edit access
6. Commenter: View and comment
7. Viewer: Read-only access

**Storage Counting Rules:**
- My Drive files: Count against user quota
- Shared Drive files: Count against domain pool
- Shared with me: Counts against owner's quota
- Google Docs/Sheets/Slides: Don't count (unlimited)
- Photos: Original quality counts, compressed doesn't (Photos settings)

### Critical Settings to Know

**Must-Know Admin Paths:**
```
Gmail settings: Apps > Google Workspace > Gmail
Drive settings: Apps > Google Workspace > Drive and Docs
Calendar: Apps > Google Workspace > Calendar
Meet: Apps > Google Workspace > Google Meet
Chat: Apps > Google Workspace > Google Chat
Security: Security > API controls / Data protection
Marketplace: Apps > Google Workspace Marketplace apps
```

**Time-Based Considerations:**
- Setting propagation: Up to 24 hours for most changes
- Email routing rules: Can take 1 hour to 24 hours
- OU changes: Usually within minutes, up to 1 hour
- User creation: Immediate, but service access up to 24 hours
- License assignment: Immediate, but features up to 24 hours

**Troubleshooting Order:**
1. Check if feature is enabled (service ON/OFF)
2. Verify license includes feature (edition check)
3. Review OU-specific settings (inheritance)
4. Check user-level permissions
5. Review audit logs for errors
6. Consider propagation time
7. Test with different user/OU

## Additional Resources

**Google Workspace Admin Help:**
- Admin Console: admin.google.com
- Admin Toolbox: toolbox.googleapps.com (MX, dig, SMTP tests)
- Status Dashboard: google.com/appsstatus
- Google Vault: vault.google.com (eDiscovery and archiving)

**Useful Reports:**
```
Admin Console > Reporting > Reports > App reports
- Drive: Storage, sharing, content
- Gmail: Email logs, spam reports
- Meet: Meeting usage, recordings
- Calendar: Resource utilization
- User: Login activity, 2FA status
```

**Best Practices for Exam:**
- Know the Admin Console navigation paths by heart
- Understand OU inheritance and override behavior
- Practice troubleshooting with methodical approach
- Remember license edition limitations
- Be aware of propagation times for settings
- Review audit logs for verification and troubleshooting
