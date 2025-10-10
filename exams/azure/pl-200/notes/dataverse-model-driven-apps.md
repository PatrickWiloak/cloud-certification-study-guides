# Dataverse and Model-Driven Apps

## Table of Contents
1. [Dataverse Fundamentals](#dataverse-fundamentals)
2. [Tables and Columns](#tables-and-columns)
3. [Relationships](#relationships)
4. [Business Rules](#business-rules)
5. [Model-Driven Apps](#model-driven-apps)
6. [Forms and Views](#forms-and-views)
7. [Charts and Dashboards](#charts-and-dashboards)
8. [Exam Tips](#exam-tips)

---

## Dataverse Fundamentals

### What is Dataverse?
Secure, cloud-based storage platform for Power Platform.

**Key Features:**
- Relational database
- Built-in security
- Business logic layer
- Managed storage
- API access
- Integrated with Power Platform

### Dataverse Architecture

**Components:**
- **Tables** - Data structures (like database tables)
- **Columns** - Fields in tables
- **Rows** - Records/data
- **Relationships** - Links between tables
- **Business Rules** - Logic and validation
- **Workflows** - Automation
- **Security Roles** - Access control

### Table Types

#### Standard Tables
Pre-built tables from Microsoft.

**Common Examples:**
- Account
- Contact
- Lead
- Opportunity
- Case
- Task
- Email

**Characteristics:**
- Managed by Microsoft
- Cannot delete
- Can customize
- Regular updates

#### Custom Tables
Created by users/organizations.

**Naming:**
- Prefix with publisher prefix (e.g., `contoso_`)
- Use singular form (e.g., `Project`, not `Projects`)
- Descriptive names

**Types:**
- **Standard** - Full features, ownership
- **Activity** - Time-based activities
- **Virtual** - External data source

### Table Ownership

**User or Team:**
- Owned by specific user/team
- Used for: Accounts, Contacts, Opportunities
- Row-level security
- Can reassign ownership

**Organization:**
- Owned by organization
- Used for: Configuration, Settings
- No individual ownership
- Shared across org

**None:**
- No ownership concept
- Used for: Reference data
- Simple access control

---

## Tables and Columns

### Creating Custom Tables

**Settings:**
```
Display Name: Project
Plural Name: Projects
Name (Schema): contoso_project
Primary Column: Project Name
Enable Attachments: Yes
Track changes: Yes
Advanced:
- Ownership: User or Team
- Create a new activity: Yes (for tracking)
```

**Properties:**
- **Enable for:** Queues, Business Process Flows, SharePoint integration
- **Type:** Standard, Activity, Virtual
- **Track changes** - Audit trail
- **Enable attachments** - File storage

### Column Types (Data Types)

#### Text Columns

**Single Line of Text:**
```
Display Name: Email Address
Name: emailaddress
Format: Email
Max Length: 100
IME Mode: Auto
```

**Formats:**
- Email
- Text
- Text Area
- URL
- Ticker Symbol
- Phone

**Multi Lines of Text:**
```
Display Name: Description
Name: description
Max Length: 4000
Format: Text / Rich Text
```

#### Number Columns

**Whole Number:**
```
Display Name: Employee Count
Name: employeecount
Min Value: 0
Max Value: 1000000
```

**Decimal Number:**
```
Display Name: Price
Name: price
Precision: 2
Min Value: 0.00
Max Value: 1000000.00
```

**Currency:**
```
Display Name: Annual Revenue
Name: annualrevenue
Precision: 2
Currency: USD
```

#### Choice Columns

**Choice (Local):**
```
Display Name: Status
Name: status
Choices:
  - Active (Value: 1)
  - Inactive (Value: 2)
  - Pending (Value: 3)
Default: Active
```

**Choices (Global):**
- Reusable across tables
- Centralized management
- Consistent values

**Yes/No (Boolean):**
```
Display Name: Is Approved
Name: isapproved
Default: No
```

#### Date and Time

**Date and Time:**
```
Display Name: Created On
Name: createdon
Format: Date and Time
Behavior: User Local
```

**Date Only:**
```
Display Name: Birth Date
Name: birthdate
Format: Date Only
```

**Behaviors:**
- **User Local** - Timezone conversion
- **Date Only** - No time component
- **Time-Zone Independent** - Stores as-is

#### Lookup Columns

**Lookup:**
```
Display Name: Account
Name: accountid
Related Table: Account
Related Column: Account Name
```

**Creates:**
- Lookup column
- Relationship (1:N)
- Related menu in target table

#### Other Column Types

**File:**
```
Display Name: Profile Picture
Name: profilepicture
Max Size: 10 MB
```

**Image:**
```
Display Name: Product Image
Name: productimage
Max Size: 10 MB
Primary Image: Yes
```

### Calculated and Rollup Columns

#### Calculated Column
Formula-based, calculated on retrieval.

**Example:**
```
Display Name: Full Name
Name: fullname
Data Type: Text
Formula: firstname & " " & lastname
```

**Functions Available:**
- Text: CONCATENATE, SUBSTITUTE, TRIM
- Math: ADD, SUBTRACT, MULTIPLY, DIVIDE
- Date: ADDDAYS, ADDMONTHS, ADDYEARS
- Logical: IF, AND, OR

**Limitations:**
- Cannot reference other calculated fields
- Limited function set
- No workflow triggers

#### Rollup Column
Aggregate data from related records.

**Example:**
```
Display Name: Total Opportunity Amount
Name: totalopportunityamount
Data Type: Currency
Source Entity: Opportunity
Aggregate: SUM
Aggregated Column: Estimated Revenue
Filter: Status = "Open"
```

**Aggregate Functions:**
- SUM, AVG, MIN, MAX, COUNT

**Update Frequency:**
- Manual: Calculated on demand
- Automatic: Every 1-12 hours

---

## Relationships

### Relationship Types

#### One-to-Many (1:N)
One parent record, many child records.

**Example:** Account → Contacts
```
Primary Table: Account
Related Table: Contact
Relationship Name: account_contacts
Lookup Column in Contact: Account (accountid)
Cascade Behavior:
  - Delete: Restrict (Cannot delete Account with Contacts)
  - Assign: Cascade All (Reassign Contacts with Account)
  - Share: Cascade All
  - Unshare: Cascade All
  - Reparent: Cascade All
  - Merge: Cascade All
```

**Cascade Options:**
- **Cascade All** - Apply to all related
- **Cascade Active** - Apply to active related
- **Cascade User-Owned** - Apply if same owner
- **Cascade None** - Don't cascade
- **Remove Link** - Remove relationship, keep records
- **Restrict** - Prevent if related exist

#### Many-to-One (N:1)
Many child records to one parent (reverse of 1:N).

**Example:** Contact → Account
- Created automatically with 1:N
- Lookup field in child table

#### Many-to-Many (N:N)
Many records on both sides.

**Example:** User ↔ Team
```
Current Table: User
Other Table: Team
Relationship Name: teammembership

Creates:
- Intersect table (hidden)
- Related menu on both sides
- No cascade behaviors
```

**Use Cases:**
- Tags and items
- Skills and employees
- Products and categories
- Users and teams

### Self-Referential Relationships

**Example:** Contact → Contact (Manager)
```
Primary Table: Contact
Related Table: Contact (self)
Lookup Column: Manager
Relationship Name: contact_manager_contact
```

---

## Business Rules

### Business Rule Scope

**Entity (Table) Level:**
- Applies to all forms
- Server-side execution
- API calls included

**Form Level:**
- Specific form only
- Client-side execution
- Faster performance

### Business Rule Components

#### Conditions

**Field Conditions:**
```
If Status Equals Active
AND Priority Equals High
AND Estimated Revenue Is Greater Than 10000
```

**Operators:**
- Equals, Does Not Equal
- Contains, Does Not Contain
- Is Greater Than, Is Less Than
- Begins With, Ends With
- Is Null, Is Not Null

#### Actions

**Set Field Value:**
```
When: Status = "Approved"
Then: Set Approval Date = Current Date
```

**Set Business Required:**
```
When: Amount > 10000
Then: Set Manager Approval as Business Required
```

**Set Visibility:**
```
When: Is Corporate Account = Yes
Then: Show Corporate Details Section
```

**Show Error Message:**
```
When: End Date < Start Date
Then: Show Error "End Date must be after Start Date"
```

**Lock/Unlock:**
```
When: Status = "Completed"
Then: Lock All Fields
```

**Set Default Value:**
```
When: New Record
Then: Set Status = "Draft"
```

### Example Business Rules

**Validation:**
```
Scope: All Forms
Condition: Discount Percentage > 20
Action: Show Error "Discount cannot exceed 20%"
```

**Required Fields:**
```
Scope: All Forms
Condition: Order Type = "Corporate"
Action: Set Business Required on fields:
  - Corporate Account Name
  - Purchase Order Number
  - Billing Address
```

**Auto-Calculate:**
```
Scope: Entity
Condition: Always
Actions:
  - Set Total Amount = Unit Price × Quantity
  - Set Tax = Total Amount × 0.08
  - Set Grand Total = Total Amount + Tax
```

---

## Model-Driven Apps

### Model-Driven App Components

**Site Map:**
- Navigation structure
- Areas, Groups, Subareas
- Dashboards and views

**Forms:**
- Data entry/viewing
- Main, Quick Create, Quick View
- Card, Dialog forms

**Views:**
- List of records
- Public, Personal, System
- Filters and columns

**Charts:**
- Visual data representation
- Associated with views

**Dashboards:**
- Overview page
- Charts and grids

### Creating Model-Driven Apps

**App Designer:**
```
1. Create new model-driven app
2. Add tables:
   - Account
   - Contact
   - Opportunity

3. Configure components:
   - Forms: Main form, Quick create
   - Views: Active Accounts, My Contacts
   - Charts: Opportunity pipeline
   - Dashboards: Sales dashboard

4. Design site map:
   Area: Sales
     Group: Customers
       - Accounts
       - Contacts
     Group: Pipeline
       - Opportunities
       - Quotes

5. Publish app
```

### Site Map

**Structure:**
```
App
  └─ Area (e.g., Sales, Service, Marketing)
      └─ Group (e.g., Customers, Cases, Reports)
          └─ Subarea (e.g., Accounts, Contacts)
```

**Subarea Types:**
- **Entity** - Table view
- **Dashboard** - Dashboard display
- **Web Resource** - HTML, JavaScript
- **URL** - External link

**Example:**
```
Area: Sales
  Group: Customers
    Subarea: Accounts (Entity)
    Subarea: Contacts (Entity)
  Group: Sales
    Subarea: Opportunities (Entity)
    Subarea: Quotes (Entity)
  Group: Dashboards
    Subarea: Sales Dashboard (Dashboard)
    Subarea: Pipeline Dashboard (Dashboard)
```

---

## Forms and Views

### Form Types

#### Main Form
Primary data entry form.

**Sections:**
- Header
- Body (tabs and sections)
- Footer

**Components:**
- Fields
- Subgrids
- Quick View forms
- IFrames
- Web resources
- Business Process Flows

#### Quick Create Form
Fast record creation.

**Characteristics:**
- Appears in modal dialog
- Limited fields (max 10)
- No tabs
- Used from: Lookups, Recent records, Quick Create menu

#### Quick View Form
Read-only related data.

**Example:**
In Contact form, show Account details via Quick View:
```
Related Table: Account
Fields Shown:
  - Account Name
  - Industry
  - Annual Revenue
  - Primary Contact
```

#### Card Form
Compact view for grids.

**Used in:**
- Dashboard grids
- Subgrids
- Lists (mobile)

### Form Design

**Tabs and Sections:**
```
Tab: General
  Section: Contact Information
    - Full Name
    - Email
    - Phone
  Section: Address
    - Street
    - City
    - State
    - Zip

Tab: Details
  Section: Professional
    - Job Title
    - Department
    - Manager
```

**Form Elements:**
- **1-column section** - Full width
- **2-column section** - Side-by-side
- **3-column section** - Header/footer
- **4-column section** - Compact layout

### Views

#### View Types

**Public Views:**
- Visible to all users
- Created by admins
- Org-wide queries

**Personal Views:**
- User-specific
- Private
- Custom filters

**System Views:**
- Built-in by Dataverse
- Examples: Active Accounts, All Contacts
- Cannot delete (can deactivate)

#### View Components

**Columns:**
```
- Account Name (Primary)
- Primary Contact (Lookup)
- Industry (Choice)
- Annual Revenue (Currency)
- Created On (Date)
```

**Filter Criteria:**
```
Status Equals Active
AND
Revenue Greater Than 100000
AND
Industry Equals "Technology" OR "Manufacturing"
```

**Sort Order:**
```
Primary: Account Name (Ascending)
Secondary: Annual Revenue (Descending)
```

---

## Charts and Dashboards

### Charts

#### Chart Types
- **Column** - Compare categories
- **Bar** - Horizontal comparison
- **Line** - Trends over time
- **Pie** - Proportions
- **Funnel** - Progressive reduction
- **Area** - Cumulative trends
- **Doughnut** - Proportions with total

#### Creating Charts

**Example: Opportunities by Stage**
```
Chart Name: Opportunity Pipeline
Record Type: Opportunity
View: My Open Opportunities

Chart Settings:
  - Legend Entries (Series):
      Y-Axis: Estimated Revenue (Sum)
  - Horizontal Axis:
      X-Axis: Sales Stage
  - Chart Type: Column
  - Top X: 10
```

**Example: Cases by Priority (Pie)**
```
Chart Name: Cases by Priority
Record Type: Case
View: Active Cases

Chart Settings:
  - Legend: Priority
  - Values: Case ID (Count)
  - Chart Type: Pie
```

### Dashboards

#### Dashboard Types

**System Dashboard:**
- Pre-built by system
- Org-wide
- Examples: Sales Dashboard, Service Dashboard

**User Dashboard:**
- Personal dashboards
- Customizable
- Private or shared

#### Dashboard Components

**Grid:**
```
Component: Account List
View: Active Accounts
Columns: 2
Height: 4 rows
```

**Chart:**
```
Component: Opportunity Pipeline
Chart: Opportunities by Stage
Columns: 2
Height: 4 rows
```

**IFrame:**
```
Component: External Report
URL: https://reports.contoso.com/sales
Columns: 4
Height: 6 rows
```

**Web Resource:**
```
Component: Custom Analytics
Web Resource: contoso_analytics.html
Columns: 4
Height: 6 rows
Parameters: entityid, entitytype
```

#### Dashboard Layout

**Standard Dashboard (4 columns):**
```
Row 1: [Chart 1 (2 cols)] [Chart 2 (2 cols)]
Row 2: [Grid 1 (4 cols - full width)]
Row 3: [Chart 3 (2 cols)] [List 1 (2 cols)]
```

**Example Sales Dashboard:**
```
Header: Sales Performance Dashboard

Row 1:
  - Opportunity Pipeline (Chart, 2 cols)
  - Revenue by Month (Chart, 2 cols)

Row 2:
  - Active Opportunities (Grid, 4 cols)

Row 3:
  - Recent Activities (List, 2 cols)
  - Top Accounts (Grid, 2 cols)
```

---

## Exam Tips

### Key Concepts

**Dataverse:**
- Secure, cloud-based storage
- Tables (entities), Columns (fields), Rows (records)
- Relationships: 1:N, N:1, N:N
- Standard vs Custom tables

**Table Ownership:**
- User/Team: Row-level security, reassignable
- Organization: Shared, no ownership
- None: Simple reference data

**Column Types:**
- Text: Single line, Multi-line
- Number: Whole, Decimal, Currency
- Choice: Local, Global
- Date/Time: User local, Date only
- Lookup: Creates relationship
- Calculated: Formula-based
- Rollup: Aggregate related

**Relationships:**
- 1:N: One parent, many children
- N:N: Many-to-many via intersect table
- Cascade behaviors: All, Active, User-Owned, None, Restrict

**Business Rules:**
- Entity level vs Form level
- Conditions and Actions
- Validation, Required fields, Visibility
- No-code business logic

**Model-Driven Apps:**
- Component-based
- Site map: Areas, Groups, Subareas
- Forms: Main, Quick Create, Quick View
- Views: Public, Personal, System
- Charts and Dashboards

### Common Scenarios

**Create Custom Table:**
1. Define table (name, ownership, features)
2. Add columns (primary + custom)
3. Create relationships
4. Configure forms and views
5. Add to model-driven app

**Relationship with Cascade:**
Account → Opportunities (1:N)
- Delete: Restrict (can't delete Account with Opportunities)
- Assign: Cascade All (reassign Opportunities with Account)
- Use case: Maintain data integrity

**Business Rule for Validation:**
```
Scope: Entity
Condition: Discount% > 20
Action: Show error "Max discount is 20%"
```

**Model-Driven App Site Map:**
```
Area: Sales
  Group: Customers
    Accounts
    Contacts
  Group: Sales
    Opportunities
    Quotes
```

### Decision Matrix

**Column Type:**
- Short text (< 100 chars) → Single Line of Text
- Long text (< 1000 chars) → Multi-line Text
- Predefined options → Choice
- Reference another table → Lookup
- Formula → Calculated
- Aggregate related → Rollup

**Relationship Type:**
- One parent, many children → 1:N
- Many children, one parent → N:1
- Many-to-many → N:N
- Same table → Self-referential

**Form Type:**
- Primary data entry → Main Form
- Quick add → Quick Create Form
- Show related data → Quick View Form
- Mobile compact → Card Form

**Business Rule Scope:**
- All forms + API → Entity level
- Specific form → Form level
- Best performance → Form level
- Complete validation → Entity level

### Quick Reference

**Table Settings:**
```
- Display Name: Singular
- Plural Name: Plural form
- Name: prefix_lowercase
- Primary Column: Descriptive name
- Ownership: User/Team or Organization
```

**Cascade Behaviors:**
- Cascade All: All related records
- Cascade Active: Only active records
- Cascade User-Owned: Same owner only
- Cascade None: No cascade
- Restrict: Prevent action
- Remove Link: Keep records, remove relationship

**Business Rule Actions:**
- Set Field Value
- Set Business Required
- Set Visibility
- Show Error Message
- Lock/Unlock
- Set Default Value

**Form Components:**
- Fields: Data entry
- Subgrids: Related records
- Quick View: Related data (read-only)
- Business Process Flow: Guided process
- Web Resource: Custom HTML/JS

### Study Focus

1. **Understand Dataverse** architecture and components
2. **Master column types** and when to use each
3. **Know relationship types** and cascade behaviors
4. **Learn business rules** - conditions and actions
5. **Understand model-driven apps** - components and site map
6. **Know form types** and when to use each
7. **Understand views** - public vs personal vs system
8. **Practice creating** tables, columns, relationships

### Final Checklist

- [ ] Dataverse fundamentals and architecture
- [ ] Standard vs Custom tables
- [ ] Table ownership types
- [ ] Column/field data types
- [ ] Calculated vs Rollup columns
- [ ] Relationship types (1:N, N:N)
- [ ] Cascade behaviors
- [ ] Business rule scope and actions
- [ ] Model-driven app components
- [ ] Site map structure
- [ ] Form types and use cases
- [ ] View types and configuration
- [ ] Chart types and creation
- [ ] Dashboard components
