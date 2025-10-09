# Power Platform Functional Consultant

## Overview
Functional Consultants configure Dataverse, create apps, implement business process automation, and integrate Power Platform with other services.

## Microsoft Dataverse

### Table Design
**Column types:**
- **Text:** Single line, multi-line, email, URL
- **Number:** Whole number, decimal, currency
- **Date and Time:** Date only, date and time
- **Choice:** Local, global
- **Lookup:** Reference other tables
- **File and Image:** Store attachments

### Relationships
**One-to-many (1:N):**
- Primary table has one record
- Related table has many records
- Example: Account → Contacts

**Many-to-one (N:1):**
- Reverse perspective of 1:N
- Many records relate to one

**Many-to-many (N:N):**
- Records in both tables relate to multiple records
- Example: Students ↔ Courses
- Uses intersect table

### Business Rules
- **Scope:** Entity (all forms) or specific form
- **Conditions:** If field equals value
- **Actions:**
  - Set field value
  - Set field requirement
  - Show/hide field
  - Show error message
  - Set default value

### Security
**Security roles:**
- **Organization:** Access all records
- **Business unit:** Access BU records
- **Parent: Child Business Units:** Access BU + child BU records
- **Business unit:** Own BU only
- **User:** Own records only
- **None:** No access

**Field-level security:**
- Secure sensitive fields
- Override role permissions
- Example: Salary field

**Hierarchy security:**
- Manager access to team records
- Position or manager hierarchy

## Power Apps

### Model-Driven App Configuration

**Forms:**
- **Main form:** Default form
- **Quick create:** Simplified creation
- **Quick view:** Read-only embedded form
- **Card form:** Mobile-optimized

**Business process flows:**
```javascript
// Stages and steps guide users
Stage 1: Qualify
  - Topic
  - Budget
  - Timeline
Stage 2: Develop
  - Proposal
  - Presentation
Stage 3: Close
  - Contract
  - Sign-off
```

**Dashboards:**
- System dashboards (all users)
- Personal dashboards (user-specific)
- Charts and views

### Canvas App Advanced Features
**Components:**
- Reusable UI elements
- Component properties
- Custom output properties

**Collections:**
```powerappsfx
// Create collection
ClearCollect(MyCollection, Filter(Customers, Country = "USA"))

// Add to collection
Collect(MyCollection, {Name: "New Customer"})
```

## Power Automate

### Advanced Flows

**Variables:**
```javascript
// Initialize variable
Initialize variable: varName (String)

// Set variable
Set variable: varName = "New Value"

// Append to array
Append to array variable: arrayVar
```

**Conditions and Switches:**
```javascript
// Condition
IF amount > 1000
  THEN Send for approval
  ELSE Auto-approve

// Switch
SWITCH priority
  CASE High: Notify immediately
  CASE Medium: Notify in 1 hour
  CASE Low: Add to queue
```

**Error Handling:**
- Configure run after: On failure, skip, timeout
- Scope actions
- Terminate action
- Try-catch pattern

### Desktop Flows (RPA)
- Record desktop actions
- UI elements interaction
- Excel automation
- Browser automation
- Trigger from cloud flows

## Power Virtual Agents

### Creating Chatbots
**Topics:**
- Trigger phrases: What users say
- Questions: Bot asks users
- Conditions: Branch logic
- Actions: Call flows, show messages

**Entities:**
- Pre-built: City, date, number
- Custom: Product names, categories

**Integration:**
- Call Power Automate flows
- Hand off to human agent
- Embed in websites, Teams

## Integration

### Microsoft 365
- SharePoint lists as data sources
- Teams integration (apps, bots)
- Outlook triggers in flows
- Excel as data source

### Azure Services
- Azure Functions from flows
- Azure SQL Database connector
- Azure Service Bus
- Application Insights logging

### Third-Party
- Dynamics 365
- Salesforce
- SAP
- Custom connectors (REST APIs)

## Solutions

### Solution Components
- Apps (canvas and model-driven)
- Tables and columns
- Flows
- Connection references
- Environment variables

### ALM (Application Lifecycle Management)
```bash
# Export solution
pac solution export --name MySolution --path ./solution.zip

# Import solution
pac solution import --path ./solution.zip
```

**Solution types:**
- **Unmanaged:** Development
- **Managed:** Production (can't modify)

**Deployment:**
1. Dev environment (unmanaged)
2. Export as managed
3. Import to Test environment
4. Import to Production

## Best Practices

### Dataverse
1. **Use global choices** for reusability
2. **Prefix custom tables** (e.g., contoso_)
3. **Plan relationships** before creating
4. **Implement security** at table and field level
5. **Use business rules** for simple logic

### Model-Driven Apps
1. **Hide unused fields** in forms
2. **Use business process flows** for guidance
3. **Create meaningful views** for users
4. **Optimize forms** for performance
5. **Test on mobile** devices

### Power Automate
1. **Use child flows** for reusability
2. **Implement error handling** always
3. **Avoid infinite loops** (check conditions)
4. **Use parallel branches** for performance
5. **Document complex flows**

### Solutions
1. **Use solutions** for all development
2. **Environment variables** for different environments
3. **Connection references** instead of direct connections
4. **Version control** solution files
5. **Test in separate** environment

## Study Tips

### Key Concepts
- Dataverse table design and relationships
- Security model (roles, field security, hierarchy)
- Model-driven app components (forms, views, BPF)
- Power Automate advanced features (variables, conditions, error handling)
- Solution ALM

### Common Scenarios
1. **Complex business app** → Model-driven app with Dataverse
2. **Approval workflow** → Power Automate with approvals
3. **Customer service** → Power Virtual Agents
4. **Move dev to prod** → Solutions (export/import)
5. **Data security** → Security roles + field-level security

### Remember
- Dataverse = Power Platform data storage
- Business rules = Simple logic, no code
- Business process flows = Guide users
- Solutions = Package and deploy
- Managed solutions = Production (read-only)
- Security roles = Table-level access
- Field security = Column-level access
