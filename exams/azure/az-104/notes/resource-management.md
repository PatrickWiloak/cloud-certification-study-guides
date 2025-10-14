# Resource Management - AZ-104

**[📖 Azure Resource Manager Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/)** - Deployment and management service for Azure

## Resource Groups

**[📖 Manage Resource Groups](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal)** - Organize and manage Azure resources
- Logical container for resources
- All resources must be in a resource group
- Cannot be nested
- Resources can be in different regions
- Free (no charge)

**Lifecycle**:
- Deleting RG deletes all resources
- Move resources between RGs
- Lock RG to prevent deletion

## Management Groups

**[📖 Management Groups Overview](https://learn.microsoft.com/en-us/azure/governance/management-groups/overview)** - Organize subscriptions at scale

- Organize subscriptions
- Hierarchy: Management groups > Subscriptions > Resource groups > Resources
- Apply policies and RBAC at any level
- Up to 6 levels deep
- Inheritance: Policies flow down

## Azure Policy

**[📖 Azure Policy Overview](https://learn.microsoft.com/en-us/azure/governance/policy/overview)** - Enforce organizational standards and assess compliance

- Enforce standards and compliance
- JSON-based policy definitions
- Built-in and custom policies
- Initiatives: Group of policies

**Effects**:

**[📖 Azure Policy Effects](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effects)** - Understand policy enforcement options

- Deny: Block non-compliant resources
- Audit: Log non-compliance
- Append: Add properties
- Modify: Change properties
- DeployIfNotExists: Auto-remediate

## Resource Locks

**[📖 Lock Resources to Prevent Changes](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)** - Protect critical resources from accidental changes

- Prevent accidental deletion/modification
- **CanNotDelete**: Can modify, can't delete
- **ReadOnly**: Can't modify or delete
- Applies to all child resources
- Override requires removing lock first

## Tags

**[📖 Use Tags to Organize Resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources)** - Apply metadata for organization and billing

- Metadata key-value pairs
- Max 50 tags per resource
- Not inherited by child resources
- Use for cost tracking, automation, organization

## Azure Resource Manager (ARM)

**[📖 ARM Templates Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/)** - Implement infrastructure as code with templates

- Deployment and management layer
- Consistent management interface
- Template-based deployments
- Declarative syntax (JSON)

**ARM Templates**:
- Parameters: Input values
- Variables: Reusable values
- Resources: What to deploy
- Outputs: Return values

## Bicep

**[📖 Bicep Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)** - Modern declarative language for Azure resources

- Domain-specific language for ARM
- Simpler syntax than JSON
- Transpiles to ARM JSON
- Better IntelliSense and type safety

## Moving Resources

**[📖 Move Resources to New Resource Group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-resource-group-and-subscription)** - Move resources between groups and subscriptions

- Within subscription: Same or different RG
- Across subscriptions: Different RG
- Validation before move
- Some resources can't be moved (AKS, App Service with cert)

## Exam Tips
- Resource groups: Logical grouping, lifecycle management
- Management groups: Multi-subscription organization
- Policies: Enforce compliance
- Locks: Prevent accidents (CanNotDelete common)
- Tags: Cost allocation and organization
- ARM templates: Infrastructure as code
- Bicep: Easier ARM authoring
