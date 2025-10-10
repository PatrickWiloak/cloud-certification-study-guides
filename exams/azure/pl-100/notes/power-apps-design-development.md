# Power Apps Design and Development

## Table of Contents
1. [Canvas Apps Fundamentals](#canvas-apps-fundamentals)
2. [App Design and UX](#app-design-and-ux)
3. [Data Sources and Connectors](#data-sources-and-connectors)
4. [Formulas and Expressions](#formulas-and-expressions)
5. [Controls and Components](#controls-and-components)
6. [Collections and Variables](#collections-and-variables)
7. [Navigation and User Experience](#navigation-and-user-experience)
8. [Exam Tips](#exam-tips)

---

## Canvas Apps Fundamentals

### Canvas App Types

**Tablet Layout:**
- 16:9 aspect ratio (1366x768)
- Horizontal orientation
- Desktop and tablet devices
- Larger screen designs

**Phone Layout:**
- 9:16 aspect ratio (640x1136)
- Vertical orientation
- Mobile-first design
- Touch-optimized

### Creating Canvas Apps

**Starting Points:**
1. **Blank App** - Start from scratch
2. **Template** - Pre-built app structures
3. **Data Source** - Auto-generate from data
4. **SharePoint** - Form-based apps
5. **Excel** - Spreadsheet-based apps

**App Creation:**
```powerf
x
// Auto-generated from SharePoint list
// Creates Browse, Detail, Edit screens
```

### App Settings

**Display:**
- Orientation (landscape/portrait)
- Scale to fit
- Lock aspect ratio
- Lock orientation

**Advanced:**
- Explicit column selection
- Delayed load
- Use non-blocking OnStart
- Formula bar height
- Enhanced delegation

---

## App Design and UX

### Screen Design

#### Screen Templates
**Blank Screen:** Empty canvas
**Form Screen:** Edit/view records
**List Screen:** Browse data
**Success Screen:** Completion messages

#### Responsive Design
```powerfx
// Responsive width
Width: Parent.Width
Height: Parent.Height

// Conditional sizing
If(Parent.Width < 600,
    400,  // Mobile
    800   // Desktop
)

// Responsive columns
If(Parent.Width < 768,
    1,    // Mobile: 1 column
    3     // Desktop: 3 columns
)
```

### Themes and Branding

**Built-in Themes:**
- Default
- Dark
- High contrast
- Custom themes

**Custom Branding:**
```powerfx
// Custom color scheme
Set(BrandPrimary, ColorValue("#0078D4"));
Set(BrandSecondary, ColorValue("#106EBE"));
Set(TextColor, ColorValue("#323130"));

// Apply to controls
Button.Fill: BrandPrimary
Button.Color: White
Label.Color: TextColor
```

### Accessibility

**Best Practices:**
1. **Tab Index** - Set logical order
2. **Screen Reader** - Add AccessibleLabel
3. **Color Contrast** - WCAG compliance
4. **Keyboard Navigation** - Support Tab/Enter
5. **Focus Indicators** - Visual cues

**Implementation:**
```powerfx
// Accessible label for screen readers
Button.AccessibleLabel: "Submit order button"

// Tab order
Button1.TabIndex: 1
TextInput1.TabIndex: 2
Button2.TabIndex: 3

// Tooltip for help
Icon1.Tooltip: "Click for help information"
```

---

## Data Sources and Connectors

### Common Data Sources

#### Dataverse (Recommended)
```powerfx
// Connect to Dataverse table
// Add Data > Dataverse > Select table

// Filter records
Filter(Accounts, 'Account Name' = "Contoso")

// Create record
Patch(Accounts, Defaults(Accounts), {
    'Account Name': "New Company",
    'Primary Contact': LookUp(Contacts, ...)
})

// Update record
Patch(Accounts, LookUp(Accounts, ...), {
    'Account Name': "Updated Name"
})

// Delete record
Remove(Accounts, ThisItem)
```

#### SharePoint
```powerfx
// Connect to SharePoint list
// Automatic connection from integration

// Get all items
SharePointList

// Filter items
Filter(SharePointList, Status = "Active")

// Create item
Patch(SharePointList, Defaults(SharePointList), {
    Title: "New Item",
    Status: "Active"
})
```

#### SQL Server
```powerfx
// Connect via connector
// On-premises: Data Gateway required

// Query data
SQL_Server.GetItems("SELECT * FROM Products WHERE Category = 'Electronics'")

// Execute stored procedure
SQL_Server.ExecuteProcedure("sp_GetOrders", {CustomerID: 123})
```

#### Excel Online
```powerfx
// Connect to Excel table
// File must be in OneDrive/SharePoint

// Get table data
ExcelTable

// Add row
Patch(ExcelTable, Defaults(ExcelTable), {
    Name: "John Doe",
    Email: "john@contoso.com"
})
```

### Connector Actions

**Common Connectors:**
- Office 365 Outlook
- Office 365 Users
- SharePoint
- Teams
- Approvals
- Planner
- OneDrive
- Azure

**Email Example:**
```powerfx
// Send email
Office365Outlook.SendEmailV2(
    "recipient@contoso.com",
    "Subject Line",
    "Email body content",
    {
        Importance: "High",
        CC: "cc@contoso.com"
    }
)

// Send with attachment
Office365Outlook.SendEmailV2(
    recipient,
    subject,
    body,
    {
        Attachments: Table({
            Name: "report.pdf",
            ContentBytes: Binary_Content
        })
    }
)
```

**SharePoint Example:**
```powerfx
// Create folder
SharePoint.CreateFolder(
    "https://contoso.sharepoint.com/sites/mysite",
    "Documents",
    "New Folder"
)

// Upload file
SharePoint.CreateFile(
    "https://contoso.sharepoint.com/sites/mysite",
    "Documents",
    "file.txt",
    "File content"
)

// Get file content
SharePoint.GetFileContent(file_identifier)
```

### Delegation

**Delegable Functions:**
- Filter
- Search
- LookUp
- Sort
- SortByColumns
- StartsWith

**Non-Delegable:**
- Mid, Left, Right (text functions)
- CountRows (use CountIf)
- Sum, Average (use AddColumns + Sum)

**Delegation Warnings:**
```powerfx
// NON-DELEGABLE - Blue warning
Filter(LargeDataset, Left(Name, 1) = "A")  // ⚠️

// DELEGABLE - No warning
Filter(LargeDataset, StartsWith(Name, "A"))  // ✓

// Delegation limit: 500/2000 records (configurable)
```

**Best Practices:**
```powerfx
// Use server-side filtering
Filter(LargeTable, Country = "USA")  // Delegated

// Avoid client-side functions on large datasets
Filter(LargeTable, Len(Name) > 5)   // Not delegated

// Use Search for text search
Search(Products, SearchInput.Text, "ProductName")  // Delegated
```

---

## Formulas and Expressions

### Context Variables

**Set (Global Variable):**
```powerfx
// Set global variable
Set(CurrentUser, User().FullName);
Set(SelectedItem, Gallery1.Selected);
Set(TotalAmount, Sum(Cart.Price));

// Use variable
Label1.Text: CurrentUser
```

**UpdateContext (Local Variable):**
```powerfx
// Set screen-level variable
UpdateContext({IsLoading: true});
UpdateContext({
    SelectedID: Gallery1.Selected.ID,
    ShowDialog: true
});

// Use in screen controls
LoadingSpinner.Visible: IsLoading
```

### Collections

**Collect (Add Records):**
```powerfx
// Create collection
Collect(MyCollection, {Name: "John", Age: 30});

// Collect multiple
Collect(MyCollection,
    {Name: "Jane", Age: 25},
    {Name: "Bob", Age: 35}
);

// Collect from data source
Collect(LocalProducts, Filter(Products, Category = "Electronics"));
```

**ClearCollect (Replace Collection):**
```powerfx
// Replace entire collection
ClearCollect(Cart, Gallery1.AllItems);

// Clear and add new data
ClearCollect(FilteredOrders,
    Filter(Orders, Status = "Pending")
);
```

**Collection Operations:**
```powerfx
// Add item to collection
Collect(Cart, {Product: "Widget", Quantity: 2, Price: 29.99});

// Remove item from collection
Remove(Cart, LookUp(Cart, Product = "Widget"));

// Remove selected item
Remove(Cart, Gallery1.Selected);

// Clear all items
Clear(Cart);

// Update item in collection
UpdateIf(Cart,
    Product = "Widget",
    {Quantity: 5}
);
```

### Common Functions

#### Text Functions
```powerfx
// Concatenate
Concatenate("Hello", " ", "World")  // "Hello World"
"Hello" & " " & "World"            // "Hello World"

// Text conversion
Text(123.45, "[$-en-US]$#,##0.00")  // "$123.45"
Text(Today(), "mm/dd/yyyy")          // "01/15/2024"

// Text manipulation
Upper("hello")                      // "HELLO"
Lower("HELLO")                      // "hello"
Proper("john doe")                  // "John Doe"
Trim("  text  ")                    // "text"
```

#### Math Functions
```powerfx
// Arithmetic
Sum(Sales.Amount)                   // Total
Average(Sales.Amount)               // Average
Count(Sales)                        // Row count
CountRows(Filter(Sales, ...))       // Filtered count
Min(Sales.Amount), Max(Sales.Amount)

// Rounding
Round(123.456, 2)                   // 123.46
RoundUp(123.4, 0)                   // 124
RoundDown(123.9, 0)                 // 123

// Other
Abs(-5)                             // 5
Power(2, 3)                         // 8
Sqrt(16)                            // 4
```

#### Date/Time Functions
```powerfx
// Current date/time
Today()                             // Current date
Now()                               // Current date and time

// Date manipulation
DateAdd(Today(), 7, Days)           // 7 days from now
DateDiff(StartDate, EndDate, Days)  // Days between dates

// Date parts
Year(Today())                       // 2024
Month(Today())                      // 1
Day(Today())                        // 15
Weekday(Today())                    // Day of week (1-7)

// Create date
Date(2024, 1, 15)                   // January 15, 2024
Time(14, 30, 0)                     // 2:30 PM
```

#### Logical Functions
```powerfx
// If conditions
If(Score >= 90, "A", Score >= 80, "B", "C")

// Switch
Switch(Status,
    "Active", "🟢",
    "Pending", "🟡",
    "Inactive", "🔴",
    "❓"  // Default
)

// Boolean operators
And(A > 0, B < 10)
Or(Status = "A", Status = "B")
Not(IsBlank(TextInput1.Text))

// IsBlank
If(IsBlank(TextInput1.Text), "Please enter value", TextInput1.Text)
```

### Table Functions

```powerfx
// Filter
Filter(Products, Category = "Electronics" And Price < 1000)

// LookUp (First matching record)
LookUp(Customers, Email = User().Email)

// Sort
Sort(Products, Price, Descending)
SortByColumns(Products, "Category", Ascending, "Price", Descending)

// AddColumns
AddColumns(Orders,
    "Total", Quantity * Price,
    "Tax", Quantity * Price * 0.08
)

// DropColumns
DropColumns(Products, "InternalNotes", "CostPrice")

// ShowColumns
ShowColumns(Products, "ProductName", "Price", "Category")

// GroupBy
GroupBy(Sales, "Category", "GroupedSales")

// FirstN, LastN
FirstN(Products, 10)
LastN(Orders, 5)
```

---

## Controls and Components

### Input Controls

#### Text Input
```powerfx
TextInput1.Default: ""
TextInput1.HintText: "Enter your name"
TextInput1.Mode: TextMode.Email  // Email, Password, MultiLine

// Validation
TextInput1.OnChange: Set(IsValid, !IsBlank(TextInput1.Text))
```

#### Dropdown
```powerfx
Dropdown1.Items: ["Option 1", "Option 2", "Option 3"]
Dropdown1.Items: Distinct(Products, Category)
Dropdown1.Selected: First(Dropdown1.Items)

// On selection
Dropdown1.OnChange: Set(SelectedCategory, Dropdown1.Selected.Value)
```

#### Combo Box
```powerfx
// Multi-select dropdown
ComboBox1.Items: Products
ComboBox1.SelectMultiple: true
ComboBox1.SelectedItems: Filter(Products, Category = "Electronics")

// Access selected
ComboBox1.SelectedItems
```

#### Date Picker
```powerfx
DatePicker1.DefaultDate: Today()
DatePicker1.StartYear: 2020
DatePicker1.EndYear: 2030

// Format
DatePicker1.Format: "mm/dd/yyyy"
```

### Display Controls

#### Label
```powerfx
Label1.Text: "Hello, " & User().FullName
Label1.Color: Color.Blue
Label1.Font: Font.'Segoe UI'
Label1.Size: 14
Label1.FontWeight: FontWeight.Bold
```

#### Gallery
```powerfx
// Vertical gallery
Gallery1.Items: SortByColumns(Products, "ProductName")
Gallery1.TemplateSize: 100
Gallery1.TemplatePadding: 10

// Gallery template controls
ThisItem.ProductName
ThisItem.Price
ThisItem.Category

// Selected item
Gallery1.Selected
Gallery1.Selected.ID
```

#### Forms
```powerfx
// Edit form
EditForm1.DataSource: Products
EditForm1.Item: Gallery1.Selected
EditForm1.DefaultMode: FormMode.Edit

// Submit form
SubmitForm(EditForm1)

// Form modes
FormMode.New      // Create new record
FormMode.Edit     // Edit existing
FormMode.View     // Read-only

// Form validation
EditForm1.Valid   // All validations pass
EditForm1.Updates // Changed fields
EditForm1.Error   // Error message
```

### Action Controls

#### Button
```powerfx
Button1.OnSelect: SubmitForm(EditForm1);
Navigate(SuccessScreen, ScreenTransition.Fade)

Button1.DisplayMode: DisplayMode.Disabled  // Disabled state
Button1.Visible: IsFormValid               // Conditional visibility
```

#### Icon
```powerfx
Icon1.Icon: Icon.Add
Icon1.Icon: Icon.Trash
Icon1.Icon: Icon.Edit

Icon1.OnSelect: Remove(Collection, ThisItem)
Icon1.Color: If(Hover, RGBA(0, 120, 212, 1), RGBA(0, 0, 0, 0.5))
```

### Components

**Creating Reusable Components:**
```powerfx
// 1. Insert > Custom > Create component
// 2. Add controls to component
// 3. Define custom properties

// Component Input Property
Component.InputProperty: Text

// Component Output Property
Component.OutputProperty: Button1.OnSelect

// Use component
HeaderComponent.Title: "Dashboard"
HeaderComponent.OnBack: Back()
```

---

## Collections and Variables

### Variable Types

**Global Variable (Set):**
- Scope: Entire app
- Use: Shared data
```powerfx
Set(CurrentUser, User().Email);
Set(AppTheme, "Dark");
```

**Context Variable (UpdateContext):**
- Scope: Current screen
- Use: Screen-specific state
```powerfx
UpdateContext({ShowPopup: true, SelectedID: 123});
```

**Collections:**
- Scope: Global
- Use: In-memory tables
```powerfx
Collect(TempData, {Name: "John", Status: "Active"});
```

### Best Practices

**Performance:**
```powerfx
// AVOID: Multiple data calls
ForAll(Gallery.AllItems,
    Patch(DataSource, ThisRecord, {Status: "Processed"})
)

// BETTER: Batch updates
Patch(DataSource, Gallery.AllItems, {Status: "Processed"})

// Use collections for frequently accessed data
OnStart: ClearCollect(LocalProducts, Products);
```

**Variable Naming:**
```powerfx
// Clear, descriptive names
Set(varCurrentUser, User().Email);          // var prefix for variables
Set(colSelectedProducts, Gallery.Selected); // col prefix for collections
Set(gblAppSettings, {...});                 // gbl prefix for global
```

---

## Navigation and User Experience

### Screen Navigation

```powerfx
// Navigate to screen
Navigate(DetailScreen, ScreenTransition.Fade)

// Navigate with context
Navigate(DetailScreen, ScreenTransition.Cover, {
    SelectedItem: Gallery1.Selected,
    ScreenMode: "Edit"
})

// Back to previous screen
Back(ScreenTransition.UnCover)

// Navigate to specific screen in history
Back(ScreenTransition.UnCover, 2)  // Back 2 screens
```

### Screen Transitions

- **None** - Instant
- **Fade** - Fade in/out
- **Cover** - Slide over (right to left)
- **UnCover** - Slide back (left to right)

### Error Handling

```powerfx
// Try/catch pattern
If(IsError(
    Patch(DataSource, Defaults(DataSource), FormData)
),
    Notify("Error saving record: " & FirstError.Message, NotificationType.Error),
    Notify("Record saved successfully", NotificationType.Success);
    Navigate(HomeScreen)
)

// IfError function (new)
IfError(
    Patch(DataSource, Record, Updates),
    Notify("Save failed: " & FirstError.Message),
    Notify("Saved successfully")
)
```

### Notifications

```powerfx
// Notify user
Notify("Record saved successfully", NotificationType.Success)
Notify("Please fill all required fields", NotificationType.Warning)
Notify("An error occurred", NotificationType.Error)
Notify("Processing...", NotificationType.Information)

// Notify with duration
Notify("Auto-closes in 5 seconds", NotificationType.Success, 5000)
```

### Loading Indicators

```powerfx
// Show spinner during load
UpdateContext({IsLoading: true});
ClearCollect(Data, LargeDataSource);
UpdateContext({IsLoading: false});

// Spinner visibility
LoadingSpinner.Visible: IsLoading

// Disable controls while loading
Button1.DisplayMode: If(IsLoading, DisplayMode.Disabled, DisplayMode.Edit)
```

---

## Exam Tips

### Key Concepts

**Canvas Apps:**
- Tablet vs Phone layouts
- Responsive design with Parent.Width
- Delegation for large datasets
- Formula-based configuration

**Data Sources:**
- Dataverse (primary)
- SharePoint, Excel, SQL
- Connectors (Office 365, Teams, etc.)
- Delegation limitations

**Variables:**
- Set: Global variables
- UpdateContext: Screen variables
- Collections: In-memory tables

**Controls:**
- Input: Text, Dropdown, ComboBox, DatePicker
- Display: Label, Gallery, Form
- Action: Button, Icon

**Formulas:**
- Filter, LookUp, Sort
- If, Switch for logic
- Text, Date/Time functions
- Patch for CRUD operations

### Common Scenarios

**Filter Gallery:**
```powerfx
Gallery1.Items: Filter(Products,
    StartsWith(ProductName, SearchBox.Text) &&
    (DropdownCategory.Selected.Value = "All" ||
     Category = DropdownCategory.Selected.Value)
)
```

**Submit Form:**
```powerfx
// Validate and submit
If(EditForm1.Valid,
    SubmitForm(EditForm1);
    Notify("Saved successfully", NotificationType.Success);
    Navigate(HomeScreen),
    Notify("Please correct errors", NotificationType.Error)
)
```

**Master-Detail:**
```powerfx
// Gallery: Master list
Gallery1.Items: Accounts

// Form: Detail view
EditForm1.Item: Gallery1.Selected
```

**Search and Filter:**
```powerfx
Gallery1.Items: Search(Products, SearchInput.Text, "ProductName", "Description")

// Combined filter and search
Filter(Search(Products, SearchInput.Text, "ProductName"),
    Category = CategoryDropdown.Selected.Value
)
```

### Quick Reference

**Common Functions:**
- Patch: Create/Update/Delete
- Filter: Filter records
- LookUp: Find single record
- Collect: Add to collection
- Navigate: Change screens
- Notify: User notifications

**Variables:**
```powerfx
Set(gblVariable, value)              // Global
UpdateContext({varLocal: value})     // Local
Collect(colName, record)             // Collection
```

**CRUD Operations:**
```powerfx
// Create
Patch(Table, Defaults(Table), {Field: Value})

// Read
Filter(Table, Condition)
LookUp(Table, Condition)

// Update
Patch(Table, LookUp(Table, ID = 123), {Field: NewValue})

// Delete
Remove(Table, LookUp(Table, ID = 123))
```

### Study Focus

1. **Master delegation** - Know delegable vs non-delegable
2. **Understand variables** - Set vs UpdateContext vs Collections
3. **Learn controls** - Properties and events
4. **Practice formulas** - Filter, Patch, LookUp
5. **Know data sources** - Dataverse, SharePoint, connectors
6. **Understand navigation** - Screen transitions, context
7. **Learn best practices** - Performance, naming, error handling

### Final Checklist

- [ ] Canvas app creation and types
- [ ] Delegation concepts and warnings
- [ ] Variable types and scopes
- [ ] Collection operations
- [ ] Common control properties
- [ ] Gallery and Form configuration
- [ ] CRUD with Patch function
- [ ] Navigation and transitions
- [ ] Error handling patterns
- [ ] Responsive design techniques
- [ ] Data source connectors
- [ ] Best practices for performance
