# Power Automate Flows and Automation

## Table of Contents
1. [Flow Types and Fundamentals](#flow-types-and-fundamentals)
2. [Cloud Flows](#cloud-flows)
3. [Triggers and Actions](#triggers-and-actions)
4. [Expressions and Dynamic Content](#expressions-and-dynamic-content)
5. [Control Actions](#control-actions)
6. [Error Handling and Testing](#error-handling-and-testing)
7. [Desktop Flows (RPA)](#desktop-flows-rpa)
8. [Exam Tips](#exam-tips)

---

## Flow Types and Fundamentals

### Flow Types

#### 1. Cloud Flows

**Automated Flows:**
- Triggered by events
- No user interaction needed
- Examples: New email, item created, schedule
```
Trigger: When an item is created (SharePoint)
Actions: Send email, update record, post to Teams
```

**Instant Flows (Button Flows):**
- Manually triggered
- From Power Apps, button, or manually
- Examples: Submit approval, process data
```
Trigger: Manual trigger (Power Apps, button)
Actions: User-selected operations
```

**Scheduled Flows:**
- Run on time schedule
- Recurring basis
- Examples: Daily reports, weekly cleanup
```
Trigger: Recurrence (daily at 9 AM)
Actions: Generate report, send summary
```

#### 2. Desktop Flows
- Robotic Process Automation (RPA)
- Automate desktop/web apps
- UI automation and API

#### 3. Business Process Flows
- Guide users through process
- Stages and steps
- Dataverse-based

---

## Cloud Flows

### Creating Automated Flows

**Common Triggers:**
- When an item is created (SharePoint)
- When a new email arrives (Outlook)
- When a file is created (OneDrive)
- When a new response is submitted (Forms)
- When a record is created/updated (Dataverse)

**Example: SharePoint to Email:**
```
1. Trigger: When an item is created (SharePoint)
   - Site Address: https://contoso.sharepoint.com/sites/mysite
   - List Name: Tasks

2. Action: Send an email (V2)
   - To: Manager email
   - Subject: New task created: [Title]
   - Body: [Description]
```

### Scheduled Flows

**Recurrence Trigger:**
```
Trigger: Recurrence
- Interval: 1
- Frequency: Day
- Time Zone: (UTC-08:00) Pacific Time
- At these hours: 9
- At these minutes: 0

Advanced Options:
- Start time: 2024-01-01T09:00:00Z
- On these days: Monday, Wednesday, Friday
```

**Examples:**
- Daily summary email
- Weekly report generation
- Monthly data cleanup
- Quarterly review reminders

### Instant Flows

**Manual Trigger:**
```
Trigger: Manually trigger a flow
- Add inputs:
  - Text: Employee Name
  - File: Document to process
  - Yes/No: Urgent?

Actions: Process based on inputs
```

**PowerApps Trigger:**
```
Trigger: PowerApps (V2)
- Ask in PowerApps: Item ID, Action Type

From Power Apps:
MyFlow.Run(Gallery1.Selected.ID, "Approve")
```

---

## Triggers and Actions

### SharePoint Triggers

**When an item is created:**
```
Site Address: [SharePoint site URL]
List Name: [List name]
```

**When an item is created or modified:**
```
Site Address: [SharePoint site URL]
List Name: [List name]
Advanced:
- Trigger Conditions: @equals(triggerOutputs()?['body/Status'], 'Approved')
```

**When a file is created (properties only):**
```
Site Address: [SharePoint site URL]
Library Name: Documents
Folder: /Shared Documents/Invoices
Include Nested Items: Yes
```

### Email Triggers

**When a new email arrives (V3):**
```
Folder: Inbox
Advanced:
- From: specific.sender@contoso.com
- Has Attachment: Yes
- Subject Filter: "Invoice"
- Importance: High
- Include Attachments: Yes
```

**When a new email arrives in a shared mailbox (V2):**
```
Original Mailbox Address: shared@contoso.com
Folder: Inbox
```

### Dataverse Triggers

**When a row is added, modified or deleted:**
```
Change type: Added or Modified
Table name: Accounts
Scope: Organization
Row Filter:
- Column: Status
- Value: Active
```

### Common Actions

#### SharePoint Actions

**Create item:**
```
Site Address: [URL]
List Name: Tasks
Title: [Dynamic content]
Description: [Dynamic content]
Assigned To: [User email]
```

**Update item:**
```
Site Address: [URL]
List Name: Tasks
Id: [Item ID from trigger]
Title: [Updated value]
```

**Get items:**
```
Site Address: [URL]
List Name: Products
Filter Query: Category eq 'Electronics'
Order By: Title
Top Count: 100
```

**Get file content:**
```
Site Address: [URL]
File Identifier: [File identifier from trigger]
Infer Content Type: Yes
```

#### Email Actions

**Send an email (V2):**
```
To: recipient@contoso.com;another@contoso.com
Subject: Order Confirmation - [Order Number]
Body: <p>Thank you for your order!</p><p>Order details: [Dynamic content]</p>
Advanced:
- From (Send as): shared@contoso.com
- CC: manager@contoso.com
- Importance: High
- Attachments: [File content from previous step]
```

**Send email with options:**
```
To: approver@contoso.com
Subject: Approval Required
User Options: Approve, Reject, Need More Info
```

#### Office 365 Users

**Get user profile (V2):**
```
User (UPN): [User email]
Returns: Display Name, Job Title, Department, Manager
```

**Get manager (V2):**
```
User (UPN): [User email]
Returns: Manager's email and details
```

#### Approvals

**Start and wait for an approval:**
```
Approval type: Approve/Reject - First to respond
Title: Expense Approval - [Amount]
Assigned to: [Manager email]
Details:
  Employee: [Name]
  Amount: [Amount]
  Description: [Description]

Next action based on:
Outcome: Approve or Reject
```

**Create an approval:**
```
Approval type: Approve/Reject - Everyone must approve
Title: Purchase Order Approval
Assigned to: [Manager1];[Manager2];[Finance]
```

---

## Expressions and Dynamic Content

### Dynamic Content

**Accessing Trigger Outputs:**
```
From SharePoint trigger:
- Title: triggerOutputs()?['body/Title']
- Created: triggerOutputs()?['body/Created']
- Author Email: triggerOutputs()?['body/Author/Email']
```

**Accessing Action Outputs:**
```
From "Get items" action:
- First item title: outputs('Get_items')?['body/value']?[0]?['Title']
- Count: length(outputs('Get_items')?['body/value'])
```

### Expressions

#### String Functions

```
concat('Hello, ', triggerOutputs()?['body/Title'])
// Result: "Hello, My Title"

substring(triggerOutputs()?['body/Description'], 0, 100)
// First 100 characters

replace(triggerOutputs()?['body/Title'], 'Old', 'New')
// Replace text

toLower(triggerOutputs()?['body/Status'])
toUpper(triggerOutputs()?['body/Status'])

trim(triggerOutputs()?['body/Title'])
// Remove leading/trailing spaces

split(triggerOutputs()?['body/Tags'], ',')
// Split by delimiter

join(array, ', ')
// Join array elements
```

#### Date/Time Functions

```
utcNow()
// Current UTC time: 2024-01-15T10:30:00Z

formatDateTime(utcNow(), 'yyyy-MM-dd')
// Format: 2024-01-15

formatDateTime(utcNow(), 'MM/dd/yyyy hh:mm tt')
// Format: 01/15/2024 10:30 AM

addDays(utcNow(), 7)
// Add 7 days

addHours(utcNow(), -5)
// Subtract 5 hours

startOfDay(utcNow())
// Beginning of today

dayOfWeek(utcNow())
// Day of week (0-6, Sunday=0)

ticks(utcNow())
// Timestamp in ticks
```

#### Logical Functions

```
if(equals(triggerOutputs()?['body/Status'], 'Approved'), 'Yes', 'No')

and(greater(outputs('Get_amount'), 1000), equals(Status, 'Pending'))

or(equals(Status, 'Active'), equals(Status, 'Pending'))

not(empty(triggerOutputs()?['body/Description']))

empty(triggerOutputs()?['body/Title'])
// Returns true if null or empty

coalesce(triggerOutputs()?['body/Title'], 'Default Title')
// Returns first non-null value
```

#### Collection Functions

```
length(outputs('Get_items')?['body/value'])
// Count of items

first(outputs('Get_items')?['body/value'])
// First item

last(outputs('Get_items')?['body/value'])
// Last item

contains(outputs('Get_items')?['body/value'], 'SearchValue')
// Check if contains

union(array1, array2)
// Combine arrays

intersection(array1, array2)
// Common elements
```

#### Math Functions

```
add(5, 3)  // 8
sub(10, 4)  // 6
mul(3, 4)  // 12
div(10, 2)  // 5
mod(10, 3)  // 1 (remainder)

max(5, 10, 3)  // 10
min(5, 10, 3)  // 3

rand(1, 100)  // Random number 1-100
```

---

## Control Actions

### Condition

```
Condition: Status equals Approved
If yes:
  - Send approval email
  - Update SharePoint item
If no:
  - Send rejection email
  - Log to tracking list
```

**Expression Conditions:**
```
@equals(outputs('Get_amount'), 1000)
@greater(outputs('Get_amount'), 5000)
@and(greater(Amount, 100), less(Amount, 1000))
```

### Switch

```
Switch: outputs('Get_status')
Case: Approved
  - Actions for approved
Case: Rejected
  - Actions for rejected
Case: Pending
  - Actions for pending
Default:
  - Actions for other statuses
```

### Apply to Each

```
Apply to each: outputs('Get_items')?['body/value']
  Current item: @{items('Apply_to_each')}

  Actions for each item:
  - Send email to @{items('Apply_to_each')?['AssignedTo']}
  - Update item status
  - Create task
```

**Concurrency Control:**
```
Settings > Concurrency Control
- Degree of Parallelism: 50 (default)
- Reduce to 1 for sequential processing
```

### Do Until

```
Do until: @equals(variables('Status'), 'Completed')
  Actions:
  - Get item status
  - Set variable 'Status'
  - Delay for 5 minutes

Timeout: PT1H (1 hour)
```

### Scope

```
Scope: Main Process
  - Action 1
  - Action 2
  - Action 3

Configure run after:
- Scope 1 succeeds → Run Scope 2
- Scope 1 fails → Run Error Handling Scope
```

---

## Error Handling and Testing

### Configure Run After

```
Action 2 settings > Configure run after
Run after Action 1:
- ☑ is successful
- ☐ has failed
- ☐ is skipped
- ☐ has timed out
```

**Error Handling Pattern:**
```
Scope: Try
  - Action 1
  - Action 2

Scope: Catch (Run after Try has failed)
  - Compose: @result('Try')
  - Send email notification
  - Log error
```

### Terminate Action

```
Terminate
Status: Succeeded / Failed / Cancelled
Code: HTTP status code (e.g., 200, 400, 500)
Message: "Process completed successfully" or error details
```

### Error Handling Functions

```
// Get error message from action
actions('Action_name')['error']['message']

// Get status code
actions('Action_name')['status']

// Result of scope
result('Scope_name')

// Check if action succeeded
equals(actions('Action_name')['status'], 'Succeeded')
```

### Testing Flows

**Test Options:**
1. **Manually** - Trigger with test data
2. **Using data from previous runs** - Replay
3. **Automatically** - Test each trigger event

**Test Flow:**
```
1. Click "Test" in flow designer
2. Select "Manually" or "Automatically"
3. Trigger the flow
4. Review each step's inputs/outputs
5. Check for errors
6. Validate data transformations
```

**Flow Checker:**
- Validates flow before saving
- Identifies errors and warnings
- Suggests best practices
- Shows compatibility issues

---

## Desktop Flows (RPA)

### Recording Actions

**Recording Types:**
1. **Desktop Recording** - Desktop apps
2. **Web Recording** - Web browsers
3. **Citrix Recording** - Virtual environments

**Recording Actions:**
```
1. Click button/link
2. Type into field
3. Select dropdown
4. Extract data from table
5. Save file
6. Close application
```

### Input/Output Variables

**Inputs:**
```
Text: EmployeeName
Number: RecordCount
Boolean: ProcessAll
File: InputDocument
```

**Outputs:**
```
Text: ConfirmationNumber
DataTable: ExtractedData
File: ProcessedDocument
```

### Common Actions

**UI Automation:**
```
- Click UI element
- Populate text field
- Press button
- Get details of UI element
- Wait for UI element
- Take screenshot
```

**System Actions:**
```
- Run application
- Terminate process
- Run DOS command
- Run PowerShell script
- Get environment variable
```

**File Actions:**
```
- Read from file
- Write to file
- Copy file
- Move file
- Delete file
- Get files in folder
```

**Excel Actions:**
```
- Launch Excel
- Read from Excel worksheet
- Write to Excel worksheet
- Close Excel
- Run Excel macro
```

### Error Handling in Desktop Flows

```
On error:
- Continue flow
- Go to next action
- Repeat action
- Go to label
- Exit flow run
```

**Try-Catch Pattern:**
```
Label: Start
  Action 1
  Action 2
  On error: Go to ErrorHandler

Label: ErrorHandler
  - Log error
  - Send notification
  - Clean up
```

---

## Exam Tips

### Key Concepts

**Flow Types:**
- Automated: Event-triggered
- Instant: Manual trigger
- Scheduled: Time-based
- Desktop: RPA automation

**Triggers:**
- SharePoint: Item created/modified
- Email: New email arrives
- Dataverse: Row added/modified/deleted
- Recurrence: Scheduled

**Actions:**
- SharePoint: Create/Update/Get items
- Email: Send email, send with options
- Approvals: Start and wait for approval
- Control: Condition, Switch, Apply to each

**Expressions:**
- String: concat, substring, replace
- Date: utcNow, formatDateTime, addDays
- Logical: if, equals, and, or
- Collection: length, first, contains

### Common Scenarios

**SharePoint to Email Automation:**
```
1. Trigger: When item created (SharePoint)
2. Condition: If Status equals "High Priority"
3. Action: Send email to manager
4. Action: Create task in Planner
```

**Approval Workflow:**
```
1. Trigger: When item created
2. Action: Get manager
3. Action: Start and wait for approval
4. Condition: If approved
   Yes: Update status to "Approved"
   No: Send rejection email
```

**Scheduled Report:**
```
1. Trigger: Recurrence (Daily at 9 AM)
2. Action: Get items (filter for yesterday)
3. Action: Create HTML table
4. Action: Send email with table
```

**Apply to Each Pattern:**
```
1. Action: Get items (SharePoint)
2. Apply to each: For each item
   - Send email
   - Update Dataverse
   - Log to table
```

**Error Handling:**
```
Scope: Try
  - Main actions

Scope: Catch (Run after Try fails)
  - Compose error details
  - Send notification
  - Terminate with Failed status
```

### Decision Matrix

**Flow Type:**
- Event-driven → Automated flow
- User-initiated → Instant flow
- Time-based → Scheduled flow
- UI automation → Desktop flow

**Trigger:**
- New list item → SharePoint trigger
- New email → Outlook trigger
- On schedule → Recurrence
- From app → PowerApps trigger

**Iteration:**
- Fixed list → Apply to each
- Unknown duration → Do until
- Multiple conditions → Switch

### Quick Reference

**Common Triggers:**
```
- When an item is created (SharePoint)
- When a new email arrives (Outlook)
- When a row is added (Dataverse)
- Recurrence (Schedule)
- Manually trigger a flow
- PowerApps (V2)
```

**Common Actions:**
```
- Create item (SharePoint)
- Send an email (Office 365)
- Start and wait for approval
- Compose (build JSON/text)
- Get items (SharePoint)
- Update a row (Dataverse)
```

**Expressions:**
```
// Current time
utcNow()

// Format date
formatDateTime(utcNow(), 'yyyy-MM-dd')

// Conditional
if(equals(Status, 'Active'), 'Yes', 'No')

// String manipulation
concat('Hello ', triggerOutputs()?['body/Title'])

// Get email
triggerOutputs()?['body/Author/Email']
```

### Study Focus

1. **Understand flow types** and when to use each
2. **Master triggers** - SharePoint, Email, Dataverse, Recurrence
3. **Know control actions** - Condition, Apply to each, Switch
4. **Learn expressions** - String, Date, Logical functions
5. **Understand error handling** - Configure run after, Scope
6. **Practice common scenarios** - Approval, notification, automation
7. **Know desktop flows** basics for RPA

### Final Checklist

- [ ] Flow types and use cases
- [ ] Common triggers configuration
- [ ] SharePoint actions (CRUD)
- [ ] Email actions and approvals
- [ ] Dynamic content vs expressions
- [ ] Common expression functions
- [ ] Control actions (Condition, Apply to each, Switch)
- [ ] Error handling patterns
- [ ] Configure run after
- [ ] Testing flows
- [ ] Desktop flow basics
- [ ] Best practices for performance
