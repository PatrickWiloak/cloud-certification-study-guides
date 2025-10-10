# Resource Management - AZ-104

## Resource Groups
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
- Organize subscriptions
- Hierarchy: Management groups > Subscriptions > Resource groups > Resources
- Apply policies and RBAC at any level
- Up to 6 levels deep
- Inheritance: Policies flow down

## Azure Policy
- Enforce standards and compliance
- JSON-based policy definitions
- Built-in and custom policies
- Initiatives: Group of policies

**Effects**:
- Deny: Block non-compliant resources
- Audit: Log non-compliance
- Append: Add properties
- Modify: Change properties
- DeployIfNotExists: Auto-remediate

## Resource Locks
- Prevent accidental deletion/modification
- **CanNotDelete**: Can modify, can't delete
- **ReadOnly**: Can't modify or delete
- Applies to all child resources
- Override requires removing lock first

## Tags
- Metadata key-value pairs
- Max 50 tags per resource
- Not inherited by child resources
- Use for cost tracking, automation, organization

## Azure Resource Manager (ARM)
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
- Domain-specific language for ARM
- Simpler syntax than JSON
- Transpiles to ARM JSON
- Better IntelliSense and type safety

## Moving Resources
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
