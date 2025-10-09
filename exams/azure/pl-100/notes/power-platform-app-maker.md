# Power Platform App Maker

## Overview
Power Platform App Makers build solutions to simplify, automate, and transform business processes using canvas apps, model-driven apps, Power Automate, and Power BI.

## Canvas Apps

### Creating Canvas Apps
- **Start from:** Blank, data source, template
- **Data sources:** SharePoint, Dataverse, SQL, Excel
- **Controls:** Buttons, forms, galleries, labels
- **Responsive design:** Adapt to screen sizes

### Formulas and Functions
```powerappsfx
// Filter gallery
Filter(Customers, Country = "USA")

// Patch record (update/create)
Patch(Customers, LookUp(Customers, ID = SelectedID), {Name: TextInput.Text})

// Navigate between screens
Navigate(DetailScreen, ScreenTransition.Fade)

// Conditional formatting
If(Value > 100, Green, Red)
```

### Connectors
- **Standard connectors:** Office 365, SharePoint, OneDrive
- **Premium connectors:** SQL Server, Salesforce, SAP
- **Custom connectors:** REST APIs

## Model-Driven Apps

### Dataverse Tables
- **Standard tables:** Account, Contact, etc.
- **Custom tables:** Business-specific entities
- **Relationships:** 1:N, N:1, N:N

### Forms and Views
- **Forms:** Display and edit records
- **Views:** List of records with filters
- **Business rules:** Client-side logic
- **JavaScript:** Advanced customization

### App Components
- **Sitemap:** Navigation structure
- **Security roles:** Control access
- **Business process flows:** Guide users
- **Dashboards:** Visualize data

## Power Automate

### Flow Types
- **Cloud flows:** Automated, instant, scheduled
- **Desktop flows:** RPA (robotic process automation)
- **Business process flows:** Guide users in model-driven apps

### Common Triggers
- **When an item is created:** SharePoint, Dataverse
- **When an email arrives:** Outlook
- **On a schedule:** Recurrence
- **For a selected item:** Manual trigger

### Actions
```yaml
# Send approval
- action: Start and wait for an approval
  inputs:
    type: Approve/Reject - First to respond
    title: Approve vacation request
    assigned_to: manager@company.com

# Create SharePoint item
- action: Create item
  inputs:
    site: https://company.sharepoint.com/sites/hr
    list: Requests
    title: $(requestTitle)
```

### Expressions
```javascript
// Get current date
utcNow()

// Add days to date
addDays(triggerBody()?['DueDate'], 7)

// Conditional
if(greater(triggerBody()?['Amount'], 1000), 'High', 'Low')
```

## Power BI Integration

### Embedding Power BI
- **Canvas apps:** Power BI tile or report visual
- **Model-driven apps:** Embed in dashboards
- **Data refresh:** Automatic or on-demand

### Creating Reports from Dataverse
- Connect Power BI Desktop to Dataverse
- Create reports and publish
- Share with app users

## Solution Design

### Business Requirements Analysis
1. **Understand process:** Current state
2. **Identify pain points:** Areas to improve
3. **Define requirements:** What success looks like
4. **Choose components:** Apps, flows, reports

### Data Modeling
1. **Identify entities:** Customers, Orders, Products
2. **Define relationships:** One-to-many, many-to-many
3. **Plan columns:** Data types and validation
4. **Security:** Who can access what

## Best Practices

### Canvas Apps
1. **Use collections** for better performance
2. **Delegate** where possible (filter, search server-side)
3. **Name controls** descriptively
4. **Use components** for reusability
5. **Test on mobile** devices

### Power Automate
1. **Error handling:** Configure run after, try-catch
2. **Scope actions:** Group related actions
3. **Concurrency:** Limit parallel runs
4. **Use variables** for clarity
5. **Naming:** Descriptive action names

### Data Design
1. **Plan schema** before building
2. **Use choice columns** instead of text
3. **Enforce** data validation
4. **Set up security** roles properly

## Study Tips

### Key Concepts
- Canvas vs model-driven apps
- Dataverse as data platform
- Power Automate flow types
- Power BI integration
- Connectors (standard vs premium)

### Common Scenarios
1. **Simple data collection** → Canvas app with SharePoint
2. **Complex business process** → Model-driven app with Dataverse
3. **Approval workflow** → Power Automate
4. **Data visualization** → Power BI embedded in app

### Remember
- Canvas apps = Flexible, custom UI
- Model-driven apps = Data-driven, automatic UI
- Dataverse = Enterprise data platform
- Power Automate = Workflow automation
- Solutions = Package and deploy together
