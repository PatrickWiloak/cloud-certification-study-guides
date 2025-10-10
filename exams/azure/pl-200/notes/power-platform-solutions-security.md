# Power Platform Solutions and Security

## Table of Contents
1. [Solutions](#solutions)
2. [Application Lifecycle Management (ALM)](#application-lifecycle-management-alm)
3. [Security Roles and Permissions](#security-roles-and-permissions)
4. [Sharing and Collaboration](#sharing-and-collaboration)
5. [Teams and Business Units](#teams-and-business-units)
6. [Data Loss Prevention (DLP)](#data-loss-prevention-dlp)
7. [Environments](#environments)
8. [Exam Tips](#exam-tips)

---

## Solutions

### What are Solutions?
Containers for Power Platform components used for:
- Packaging customizations
- Transporting between environments
- Managing dependencies
- Version control
- Application lifecycle management

### Solution Types

#### Unmanaged Solutions
**Characteristics:**
- Editable in target environment
- Can add/remove components
- Used for development
- Cannot delete components (keeps dependencies)
- Can be exported as managed or unmanaged

**Use Cases:**
- Development environment
- Active customization
- Iterative changes
- Source control integration

#### Managed Solutions
**Characteristics:**
- Read-only in target environment
- Cannot modify components
- Can be uninstalled (removes all components)
- Layer over system solution
- Used for production deployment

**Use Cases:**
- Production environments
- UAT/Staging
- Customer deployments
- AppSource apps

**Key Difference:**
```
Unmanaged → Development, editable, source control
Managed → Production, read-only, deployable package
```

### Solution Components

**Common Components:**
- Tables (entities)
- Columns (fields)
- Forms
- Views
- Charts
- Dashboards
- Business rules
- Workflows
- Power Apps (canvas, model-driven)
- Power Automate flows
- Connection references
- Environment variables
- Security roles
- Choices (option sets)

**Adding Components:**
```
1. Open solution
2. Click "Add existing" or "New"
3. Select component type
4. Add specific components
   - Add required components (dependencies)
   - Add subcomponents (child objects)
```

### Solution Layering

**Default Solution:**
- Base layer
- All customizations without solution
- Cannot export
- Background container

**Unmanaged Layer:**
- Above default solution
- Active customizations
- Multiple unmanaged solutions stack
- Top layer takes precedence

**Managed Layer:**
- Above unmanaged
- Production customizations
- Multiple managed solutions stack
- Layering order matters

**Example Layers (bottom to top):**
```
1. System Solution (base)
2. Default Solution
3. Unmanaged Solution A
4. Unmanaged Solution B
5. Managed Solution 1
6. Managed Solution 2 ← Active layer
```

### Solution Dependencies

**Types:**
- **Published** - Required components
- **Unpublished** - Current draft dependencies
- **Internal** - Within solution
- **External** - Other solutions/system

**Managing Dependencies:**
```
Check dependencies:
Solutions → Select solution → Show dependencies

Required components:
- Tables used in forms
- Columns used in views
- Lookups to other tables
- Security roles referencing tables
```

**Solution Publisher:**
```
Display Name: Contoso
Name: contoso
Prefix: contoso (for schema names)
Option Value Prefix: 10000

Results in:
- Table: contoso_project
- Column: contoso_projectcode
- Choice value: 10000 (Active)
```

---

## Application Lifecycle Management (ALM)

### ALM Environments

**Development:**
- Unmanaged solutions
- Active development
- Frequent changes
- Source control integration

**Test/UAT:**
- Managed solutions imported
- User acceptance testing
- Validation before production

**Production:**
- Managed solutions only
- End users
- No direct customization
- Monitored and supported

### Solution Export

**Export Unmanaged:**
```
1. Select solution
2. Export → Unmanaged
3. Publish customizations
4. Next, Next, Export
5. Download .zip file

Use for: Source control, backup, dev-to-dev
```

**Export Managed:**
```
1. Select solution
2. Export → Managed
3. Publish customizations
4. Next, Next, Export
5. Download .zip file

Use for: Production deployment, customer delivery
```

**Export Settings:**
```
- System settings (Auto-numbering, currencies)
- Customization (Calendar, Organization settings)
- Email templates
- Security roles
- Connections (Environment variables instead)
```

### Solution Import

**Import Process:**
```
1. Solutions → Import
2. Browse and select .zip file
3. Import solution dialog
   - Upgrade or Update (existing solutions)
   - Activate plugins and workflows
4. Configure environment variables
5. Configure connection references
6. Import

Watch for:
- Missing dependencies (error)
- Version conflicts (warning)
- Customization conflicts (warning)
```

**Import Options:**
```
Upgrade (managed solutions):
- Stage for upgrade
- Apply after import
- Old solution replaced

Update (unmanaged):
- Add new components
- Update existing
- Keep old components
```

### Environment Variables

**Purpose:** Configuration values that change between environments.

**Create:**
```
Display Name: API Endpoint URL
Name: contoso_apiendpoint
Data Type: Text
Default Value: https://api-dev.contoso.com
Current Value: (set per environment)
```

**Use in Flows:**
```
HTTP Action:
URI: environmentVariables('contoso_apiendpoint')
```

**Use in Canvas Apps:**
```powerfx
Set(ApiUrl, LookUp('Environment Variable Values',
    'Environment Variable Definition'.'Schema Name' = "contoso_apiendpoint",
    Value
));
```

**Import Process:**
```
During solution import:
1. Prompt for environment variable values
2. Set per-environment values
3. Override defaults
```

### Connection References

**Purpose:** Abstract connections from flows/apps.

**Create:**
```
Display Name: SharePoint Connection
Name: contoso_sharepointconnection
Connector: SharePoint
Connection: (selected during import)
```

**Benefits:**
- Solution aware
- No hard-coded connections
- Different connections per environment
- Easier deployment

**Import:**
```
During import:
1. Prompt for connections
2. Select existing or create new
3. Authorize if needed
```

---

## Security Roles and Permissions

### Security Role Fundamentals

**Access Levels:**
1. **None** - No access (×)
2. **User** - Own records only (●)
3. **Business Unit** - Records in user's BU (●●)
4. **Parent: Child Business Units** - User's BU + child BUs (●●●)
5. **Organization** - All records (●●●●)

**Privileges:**
- **Create** - Create new records
- **Read** - View records
- **Write** - Edit records
- **Delete** - Delete records
- **Append** - Add related records
- **Append To** - Be related to
- **Assign** - Change owner
- **Share** - Share record

### Creating Security Roles

**Copy Existing Role:**
```
Settings → Security → Security Roles
Select role (e.g., Salesperson) → Actions → Copy
Name: Custom Sales Rep
Set privileges per table
```

**Setting Privileges:**
```
Table: Account
- Create: Business Unit (●●)
- Read: Organization (●●●●)
- Write: User (●)
- Delete: None (×)
- Assign: Business Unit (●●)
- Share: Business Unit (●●)
```

### Common Security Roles

**System Administrator:**
- Full access to all tables
- Customize system
- Manage users and security

**System Customizer:**
- Customize system
- Create/edit tables, forms, views
- No user management

**Basic User:**
- Minimal access
- Read-only common tables
- Own records for tasks/activities

**Salesperson:**
- Account, Contact, Lead, Opportunity access
- Business Unit level for most
- Cannot customize

### Field-Level Security

**Enable for Column:**
```
1. Edit column
2. Advanced options
3. Enable field security: Yes
4. Save

Then configure in Security Roles:
5. Settings → Security → Field Security Profiles
6. Create profile: "Sensitive Field Access"
7. Add users/teams
8. Set permissions: Read, Update, Create
```

**Use Cases:**
- SSN, credit card numbers
- Salary information
- Confidential notes
- Trade secrets

### Record-Level Security

**Ownership:**
```
User/Team owned tables:
- Owner field
- Can assign to user/team
- Security role determines access
```

**Hierarchical Security:**
```
Enable: Settings → Security → Hierarchy Security
Configure: Manager Hierarchy or Position Hierarchy

Allows:
- Managers to access subordinate records
- Position-based access
- Independent of Business Units
```

**Access Team:**
```
Purpose: Grant specific users access to specific records

Create template:
1. Settings → Templates → Access Team Templates
2. Name: Account Collaboration Team
3. Select table: Account
4. Permissions: Read, Write, Append

Add team members:
Record → Teams → Add to access team
```

---

## Sharing and Collaboration

### Sharing Records

**Share Single Record:**
```
Record → Share
Add User/Team: [User email]
Permissions:
  ☑ Read
  ☑ Write
  ☐ Delete
  ☐ Assign
  ☐ Share (allow resharing)
```

**Share Multiple:**
```
View → Select records → Share
Add users/teams
Set permissions
```

**Sharing Cascade:**
```
Share Account with user:
Option: Share with related records
  ☑ Contacts
  ☑ Opportunities
  ☑ Cases
```

### Team Collaboration

**Owner Team:**
```
Purpose: Group ownership of records
Type: Owner
Members: [Users]
Security Roles: [Assign roles]

Usage:
- Assign records to team
- All team members access via role
- Shared workload
```

**Access Team:**
```
Purpose: Ad-hoc access to specific records
Type: Access
Template: Account Collaboration

Usage:
- Automatically created for record
- Add members dynamically
- Record-specific permissions
```

### Sharing Canvas Apps

**Share App:**
```
Power Apps → Apps → [App] → Share
Add users/co-owners:
  User/Group: [Email/AAD group]
  Permission: Can view/Can edit

Data permissions:
  ☑ Grant access to data sources
  Security roles: [Select roles for Dataverse]
```

**Co-Owner vs User:**
- **Co-Owner** - Edit app, share with others
- **User** - Run app only

---

## Teams and Business Units

### Business Units

**Purpose:**
- Organizational structure
- Data security boundary
- Reflect company hierarchy

**Hierarchy:**
```
Root Business Unit (Contoso)
  ├─ Sales BU
  │   ├─ North Sales
  │   └─ South Sales
  ├─ Service BU
  └─ Marketing BU
```

**Creating:**
```
Settings → Business Units → New
Name: North Sales
Parent: Sales BU
Save
```

**User Assignment:**
```
Settings → Users → [User] → Business Unit: North Sales
- User can be in ONE business unit
- Change: Transfer user to another BU
```

**Security Implications:**
```
Security Role: Business Unit access (●●)
User in North Sales:
  - Sees North Sales records
  - Doesn't see South Sales records
  - Unless Organization level access
```

### Teams

**Owner Team:**
```
Settings → Teams → New
Name: North Sales Team
Business Unit: North Sales
Type: Owner

Add members:
- [User 1]
- [User 2]

Assign Security Roles:
- Salesperson

Usage:
- Assign records to team
- All members have access via role
```

**Access Team:**
```
Created automatically or manually
Purpose: Record-specific access
Template-based permissions
No security roles assigned
```

**AAD Group Team:**
```
Type: AAD Security Group or Office 365 Group
Sync from Azure AD
Members managed in AAD
Permissions via security roles
```

---

## Data Loss Prevention (DLP)

### DLP Policies

**Purpose:**
- Prevent data leakage
- Control connector usage
- Enforce governance

**Connector Groups:**
1. **Business** - Corporate data (SharePoint, Dataverse)
2. **Non-Business** - External services (Twitter, Gmail)
3. **Blocked** - Prohibited connectors

**Rules:**
- Business ↔ Business: ✓ Allowed
- Non-Business ↔ Non-Business: ✓ Allowed
- Business ↔ Non-Business: × Blocked
- Any ↔ Blocked: × Blocked

### Creating DLP Policies

```
Power Platform Admin Center → Data policies → New

Policy Name: Corporate Data Protection
Scope:
  ○ All environments
  ○ Specific environments (select)
  ○ All except (exclude list)

Configure connectors:
Business:
  - SharePoint
  - Office 365 Outlook
  - Dataverse
  - SQL Server

Non-Business:
  - Twitter
  - Facebook
  - RSS

Blocked:
  - Consumer services
  - Unapproved storage

Action:
  - Block or warn
```

**Connector Patterns:**
```
Pattern: *.contoso.com
Applies to: HTTP connector
Allows: https://api.contoso.com
Blocks: https://api.external.com
```

### DLP Impact

**On Flows:**
```
Violation:
Flow uses SharePoint (Business) + Twitter (Non-Business)

Result:
- Flow suspended
- Error message
- Must remove violating connector or request exception
```

**On Apps:**
```
Violation:
App uses Dataverse (Business) + Gmail (Non-Business)

Result:
- Cannot save
- Error shown in app checker
- Must remove connector or change grouping
```

---

## Environments

### Environment Types

**Production:**
```
Purpose: Live applications
Dataverse: Yes
Storage: Paid (1GB base + 10GB per license)
DLP: Enforced
```

**Sandbox:**
```
Purpose: Testing, UAT
Dataverse: Yes
Copy from Production: Yes
Refresh: Supported
```

**Developer:**
```
Purpose: Individual development
Dataverse: Yes
Limit: One per user (with Power Apps Developer Plan)
Storage: Limited
```

**Default:**
```
Purpose: Trial, learning
Dataverse: Created on-demand
Cannot delete: Built-in
Shared: All tenant users
```

**Trial:**
```
Purpose: Evaluation (30 days)
Dataverse: Yes
Convert: To production (requires license)
```

### Environment Admin

**Creating Environment:**
```
Power Platform Admin Center → Environments → New

Name: UAT Environment
Type: Sandbox
Region: United States
Dataverse: Yes
URL: contoso-uat
Currency: USD
Language: English

Security Group (optional): [AAD group for access]
```

**Copy Environment:**
```
Source: Production
Target: New or existing sandbox
Copy type:
  - Everything (full copy)
  - Customizations and schemas only
  - Reset environment

Includes:
  - All data
  - Customizations
  - Apps
  - Flows
```

**Backup and Restore:**
```
Automated backups:
  - System backups: Daily (retained 28 days)
  - Manual backups: On-demand (up to 3)

Restore:
  Power Platform Admin Center → Environments
  → [Environment] → Backups → Restore
  Select backup point
  Confirm restore (replaces current)
```

---

## Exam Tips

### Key Concepts

**Solutions:**
- Unmanaged: Development, editable
- Managed: Production, read-only, uninstallable
- Components: Tables, apps, flows, roles
- Publisher: Prefix for schema names

**ALM:**
- Dev → Test → Production
- Export: Unmanaged (dev) or Managed (prod)
- Import: Check dependencies, set variables
- Environment variables, connection references

**Security:**
- Access levels: None, User, Business Unit, Parent-Child, Organization
- Privileges: Create, Read, Write, Delete, Append, Assign, Share
- Field-level security: Sensitive columns
- Record-level: Owner, hierarchical, access team

**Sharing:**
- Share records: Read, Write, Delete, Assign, Share
- Apps: Co-owner (edit) vs User (run)
- Teams: Owner (role-based) vs Access (record-specific)

**Business Units:**
- Organizational structure
- Security boundary
- Hierarchy: Root → Child BUs
- User: One BU only

**DLP:**
- Connector groups: Business, Non-Business, Blocked
- Rules: Block cross-group connections
- Enforce data governance

**Environments:**
- Production, Sandbox, Developer, Trial, Default
- Copy, backup, restore
- Security groups for access

### Common Scenarios

**Deploy Solution:**
1. Dev: Export as Managed
2. Import to UAT (sandbox)
3. Set environment variables, connections
4. Test
5. Import to Production
6. Monitor

**Security Configuration:**
```
Requirement: Sales reps see own accounts, managers see team's
Solution:
1. Security role: Account Read = User (●)
2. Enable Manager Hierarchy
3. Assign role to sales reps
4. Managers auto-access subordinates
```

**DLP Policy:**
```
Requirement: Prevent corporate data (Dataverse) to external (Twitter)
Solution:
1. Create DLP policy
2. Dataverse → Business group
3. Twitter → Non-Business group
4. Apply to all environments
```

**Field Security:**
```
Requirement: Only HR sees Salary field
Solution:
1. Enable field security on Salary column
2. Create field security profile "HR Salary Access"
3. Add HR team
4. Grant Read, Update permissions
```

### Decision Matrix

**Solution Type:**
- Development → Unmanaged
- Production → Managed
- Source control → Unmanaged
- Customer delivery → Managed

**Environment Type:**
- Live users → Production
- Testing → Sandbox
- Personal dev → Developer
- 30-day trial → Trial

**Team Type:**
- Group ownership → Owner Team
- Record-specific access → Access Team
- AAD-managed → AAD Group Team

**Access Level:**
- Own records → User (●)
- Department → Business Unit (●●)
- Managers + reports → Parent-Child (●●●)
- All records → Organization (●●●●)

### Quick Reference

**Solution Workflow:**
```
1. Create publisher (prefix)
2. Create solution
3. Add components
4. Export (unmanaged → managed)
5. Import to target
6. Test and deploy
```

**Security Role Privileges:**
```
× None
● User
●● Business Unit
●●● Parent-Child BUs
●●●● Organization
```

**Sharing Permissions:**
- Read: View record
- Write: Edit record
- Delete: Delete record
- Assign: Change owner
- Share: Share with others
- Append: Add related
- Append To: Be related to

**DLP Groups:**
```
Business ↔ Business: ✓
Non-Business ↔ Non-Business: ✓
Business ↔ Non-Business: ×
Any ↔ Blocked: ×
```

### Study Focus

1. **Understand solution types** - Unmanaged vs Managed
2. **Master ALM process** - Dev to production deployment
3. **Know security roles** - Access levels and privileges
4. **Learn sharing** - Records, apps, teams
5. **Understand Business Units** - Hierarchy and security
6. **Know DLP policies** - Connector groups and rules
7. **Master environments** - Types and admin tasks
8. **Practice scenario-based** questions

### Final Checklist

- [ ] Solution types and layering
- [ ] Solution components
- [ ] Publisher and prefix
- [ ] ALM environments (Dev, Test, Prod)
- [ ] Export managed vs unmanaged
- [ ] Environment variables
- [ ] Connection references
- [ ] Security role access levels
- [ ] Security role privileges
- [ ] Field-level security
- [ ] Sharing records and apps
- [ ] Owner vs Access teams
- [ ] Business Unit hierarchy
- [ ] DLP policy configuration
- [ ] Connector groups
- [ ] Environment types
- [ ] Copy and backup environments
