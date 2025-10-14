# Identity and Governance - AZ-104

**[📖 Azure AD Documentation](https://learn.microsoft.com/en-us/azure/active-directory/)** - Official Azure Active Directory documentation

## Azure Active Directory Management

**[📖 Manage Users in Azure AD](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/add-users-azure-active-directory)** - Learn how to create and manage users in Azure Active Directory

### User Management

#### Creating and Managing Users
```powershell
# PowerShell - Create user
New-AzureADUser -DisplayName "John Doe" -UserPrincipalName "john@contoso.com" -AccountEnabled $true -PasswordProfile $PasswordProfile

# Azure CLI - Create user
az ad user create --display-name "John Doe" --user-principal-name "john@contoso.com" --password "TempPassword123!"
```

#### User Types
- **Member Users:** Users who belong to the organization
- **Guest Users:** External users invited to access resources (B2B)
- **Cloud-only Users:** Created directly in Azure AD
- **Synced Users:** Synchronized from on-premises AD

#### User Properties
- **Basic Information:** Name, username, contact info
- **Job Information:** Title, department, manager
- **Settings:** Usage location, license assignments
- **Authentication:** Password policies, MFA settings

### Group Management

**[📖 Azure AD Groups](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/how-to-manage-groups)** - Comprehensive guide to managing groups in Azure AD

#### Group Types
- **Security Groups:** Manage access to resources
- **Microsoft 365 Groups:** Collaboration groups with shared resources
- **Distribution Groups:** Email distribution (Exchange Online)

#### Membership Types
- **Assigned:** Manual membership assignment
- **Dynamic User:** Rule-based membership using user attributes
- **Dynamic Device:** Rule-based membership using device attributes

#### Dynamic Group Rules
```
# Example rules
user.department -eq "Sales"
user.city -eq "Seattle" -and user.jobTitle -contains "Manager"
device.deviceOSType -eq "Windows"
```

### Administrative Units

#### Purpose
- Divide Azure AD organization into smaller units
- Delegate administrative permissions
- Restrict scope of administrative roles

#### Use Cases
- Geographic divisions (US, Europe, Asia)
- Business units (Sales, Engineering, Marketing)
- Subsidiary organizations

#### Configuration
1. Create administrative unit
2. Add users/groups to the unit
3. Assign scoped administrators
4. Configure role permissions

### Device Management

#### Device Join Types
- **Azure AD Joined:** Corporate-owned devices managed by organization
- **Hybrid Azure AD Joined:** On-premises domain joined + Azure AD registered
- **Azure AD Registered:** Personal devices with organizational access

#### Device Settings
- Device registration settings
- Enterprise State Roaming
- Device writeback to on-premises
- Maximum devices per user

#### Device Compliance
- Compliance policies for device access
- Conditional access integration
- Device enrollment restrictions

### Bulk Operations

#### Bulk User Operations
- Bulk create users (CSV upload)
- Bulk invite guest users
- Bulk delete users
- Download user lists

#### PowerShell for Bulk Operations
```powershell
# Bulk create users from CSV
$users = Import-Csv "users.csv"
foreach ($user in $users) {
    New-AzureADUser -DisplayName $user.DisplayName -UserPrincipalName $user.UPN -AccountEnabled $true
}
```

### Self-Service Password Reset (SSPR)

**[📖 Deploy Azure AD Self-Service Password Reset](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-sspr-deployment)** - Complete deployment guide for SSPR

#### Configuration Requirements
- Azure AD Premium P1 or P2 license
- Authentication methods configuration
- Registration requirements
- Notification settings

#### Authentication Methods
- Mobile phone (SMS/Call)
- Office phone
- Email address
- Security questions
- Microsoft Authenticator app

#### Implementation Steps
1. Enable SSPR for target users
2. Configure authentication methods
3. Set registration requirements
4. Configure password writeback (hybrid)
5. Test SSPR functionality

## Role-Based Access Control (RBAC)

**[📖 Azure RBAC Documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)** - Complete overview of Azure role-based access control

### RBAC Components

#### Security Principal
- **User:** Individual person
- **Group:** Collection of users
- **Service Principal:** Application identity
- **Managed Identity:** Azure-managed application identity

#### Role Definition
- **Actions:** Allowed operations
- **NotActions:** Denied operations (exclusions from Actions)
- **DataActions:** Data plane operations
- **NotDataActions:** Denied data operations

#### Scope
- **Management Group:** Multiple subscriptions
- **Subscription:** Azure subscription
- **Resource Group:** Logical container
- **Resource:** Individual Azure resource

### Built-in Roles

#### Common Management Roles
```json
{
  "Owner": "Full access including access management",
  "Contributor": "Full access except access management",
  "Reader": "View all resources but no changes",
  "User Access Administrator": "Manage user access to Azure resources"
}
```

#### Service-specific Roles
- Virtual Machine Contributor
- Storage Account Contributor
- Network Contributor
- SQL DB Contributor
- Website Contributor

### Custom Roles

**[📖 Create Custom RBAC Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles)** - Step-by-step guide to creating custom Azure roles

#### Creating Custom Roles
```json
{
  "Name": "Virtual Machine Operator",
  "Id": null,
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/subscription-id"
  ]
}
```

#### PowerShell Example
```powershell
# Create custom role
$role = Get-AzRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "Virtual Machine Operator"
$role.Description = "Can monitor and restart virtual machines"
$role.Actions.RemoveRange(0,$role.Actions.Count)
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
New-AzRoleDefinition -Role $role
```

### Role Assignments

#### Assignment Process
1. Select security principal (who)
2. Select role definition (what permissions)
3. Select scope (where)

#### Azure CLI Example
```bash
# Assign role to user at resource group scope
az role assignment create \
  --assignee user@contoso.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"
```

#### Checking Effective Permissions
```powershell
# View role assignments for user
Get-AzRoleAssignment -SignInName user@contoso.com
```

### Multiple Directory Management

#### Scenarios
- Multi-tenant organizations
- Mergers and acquisitions
- Testing and development environments
- Geographic requirements

#### Guest User Management
```powershell
# Invite guest user
New-AzureADMSInvitation -InvitedUserEmailAddress "guest@external.com" -InviteRedirectUrl "https://portal.azure.com"
```

## Subscription and Governance

**[📖 Azure Subscriptions Overview](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/initial-subscriptions)** - Best practices for Azure subscription design

### Azure Subscriptions

#### Subscription Types
- **Free Tier:** 12 months free services + always free services
- **Pay-As-You-Go:** Pay for what you use
- **Enterprise Agreement:** Volume licensing for large organizations
- **CSP (Cloud Solution Provider):** Partner-managed subscriptions

#### Subscription Limits
- Resource limits per subscription
- API call limits
- Regional availability
- Service-specific quotas

#### Moving Subscriptions
```powershell
# Move subscription between directories
Select-AzSubscription -SubscriptionId "subscription-id"
# Use Azure portal or contact support for directory transfer
```

### Management Groups

**[📖 Management Groups Documentation](https://learn.microsoft.com/en-us/azure/governance/management-groups/overview)** - Organize resources with Azure management groups

#### Hierarchy Structure
```
Root Management Group
├── Production Management Group
│   ├── Production Subscription 1
│   └── Production Subscription 2
├── Development Management Group
│   ├── Dev Subscription 1
│   └── Test Subscription 2
└── Sandbox Management Group
    └── Sandbox Subscription
```

#### Management Group Operations
```powershell
# Create management group
New-AzManagementGroup -GroupName "Production" -DisplayName "Production Environment"

# Move subscription to management group
New-AzManagementGroupSubscription -GroupName "Production" -SubscriptionId "subscription-id"
```

### Azure Policy

**[📖 Azure Policy Documentation](https://learn.microsoft.com/en-us/azure/governance/policy/overview)** - Enforce organizational standards with Azure Policy

#### Policy Structure
```json
{
  "if": {
    "field": "location",
    "notIn": ["eastus", "westus", "centralus"]
  },
  "then": {
    "effect": "deny"
  }
}
```

#### Policy Effects
- **Deny:** Block resource creation/update
- **Audit:** Log non-compliant resources
- **AuditIfNotExists:** Audit if related resource doesn't exist
- **DeployIfNotExists:** Deploy resource if it doesn't exist
- **Modify:** Change resource properties
- **Disabled:** Turn off policy evaluation

#### Built-in Policies
- Allowed locations
- Allowed virtual machine sizes
- Require tags on resources
- Audit VMs without managed disks
- Enforce HTTPS for storage accounts

#### Policy Initiatives (Sets)
```powershell
# Create policy initiative
$policySet = @{
    Name = "SecurityBaseline"
    DisplayName = "Security Baseline Initiative"
    PolicyDefinitions = @(
        @{PolicyDefinitionId = "/providers/Microsoft.Authorization/policyDefinitions/policy1-id"},
        @{PolicyDefinitionId = "/providers/Microsoft.Authorization/policyDefinitions/policy2-id"}
    )
}
New-AzPolicySetDefinition @policySet
```

### Resource Locks

**[📖 Lock Resources to Prevent Changes](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)** - Protect resources from accidental deletion or modification

#### Lock Types
- **ReadOnly:** Authorized users can read but not delete or modify
- **Delete:** Authorized users can read and modify but not delete

#### Lock Inheritance
- Locks applied at parent scope inherited by child resources
- Child locks can be more restrictive, not less

#### Managing Locks
```powershell
# Create resource lock
New-AzResourceLock -LockName "DontDelete" -LockLevel Delete -ResourceGroupName "myRG"

# Remove resource lock
Remove-AzResourceLock -LockName "DontDelete" -ResourceGroupName "myRG"
```

### Resource Tagging

**[📖 Use Tags to Organize Azure Resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources)** - Best practices for resource tagging and cost management

#### Tagging Strategy
```json
{
  "Environment": "Production",
  "CostCenter": "IT",
  "Owner": "john.doe@contoso.com",
  "Project": "WebApp-v2",
  "ExpirationDate": "2024-12-31"
}
```

#### Tag Policies
```json
{
  "if": {
    "field": "tags['Environment']",
    "exists": "false"
  },
  "then": {
    "effect": "deny"
  }
}
```

#### Bulk Tagging Operations
```powershell
# Apply tags to all resources in resource group
$tags = @{Environment="Production"; CostCenter="IT"}
$resources = Get-AzResource -ResourceGroupName "myRG"
foreach ($resource in $resources) {
    Set-AzResource -ResourceId $resource.ResourceId -Tag $tags -Force
}
```

### Cost Management

**[📖 Azure Cost Management and Billing](https://learn.microsoft.com/en-us/azure/cost-management-billing/)** - Monitor and control Azure spending with cost management tools

#### Cost Analysis Tools
- **Azure Cost Management + Billing:** Native cost analysis
- **Azure Advisor:** Cost optimization recommendations
- **Azure Pricing Calculator:** Estimate costs for new resources
- **TCO Calculator:** Compare on-premises vs cloud costs

#### Budget Configuration
```powershell
# Create budget
$budget = @{
    Name = "MonthlyBudget"
    Amount = 1000
    TimeGrain = "Monthly"
    TimePeriod = @{
        StartDate = "2024-01-01"
        EndDate = "2024-12-31"
    }
    Notifications = @{
        Actual_GreaterThan_80_Percent = @{
            Enabled = $true
            Operator = "GreaterThan"
            Threshold = 80
            ContactEmails = @("admin@contoso.com")
        }
    }
}
```

#### Cost Optimization Strategies
- Right-sizing virtual machines
- Reserved instances for predictable workloads
- Azure Hybrid Benefit for Windows/SQL licenses
- Automated shutdown for dev/test environments
- Storage tier optimization

## Key Takeaways for AZ-104

1. **Identity Management:** Master Azure AD user/group management and SSPR
2. **RBAC Implementation:** Understand built-in roles and custom role creation
3. **Governance Tools:** Policy, locks, and tagging for compliance and organization
4. **Cost Control:** Budgets, analysis, and optimization strategies
5. **Administrative Efficiency:** Bulk operations and PowerShell automation
6. **Security Focus:** Principle of least privilege and proper access controls