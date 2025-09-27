# Azure Security & Identity Fundamentals

## Azure Active Directory (Azure AD)

### What is Azure AD?
Cloud-based identity and access management service that provides single sign-on, multi-factor authentication, and identity protection.

### Azure AD vs On-Premises AD
| Feature | Azure AD | On-Premises AD |
|---------|----------|----------------|
| **Protocol** | HTTP/HTTPS (REST APIs) | LDAP, Kerberos, NTLM |
| **Structure** | Flat namespace | Hierarchical (OU structure) |
| **Authentication** | Modern auth (OAuth, SAML) | Traditional Windows auth |
| **Management** | Web-based portal | Windows-based tools |
| **Integration** | Cloud services | On-premises applications |

### Azure AD Editions
| Edition | Features | Use Case |
|---------|----------|----------|
| **Free** | Basic directory, SSO, MFA | Small organizations |
| **Premium P1** | Dynamic groups, self-service, hybrid sync | Medium enterprises |
| **Premium P2** | Identity protection, PIM, access reviews | Large enterprises |

### Core Identity Objects
- **Users**: Individual identities (employees, contractors, partners)
- **Groups**: Collections of users for permission management
- **Applications**: Software applications registered with Azure AD
- **Service Principals**: Application identities for automation
- **Devices**: Registered/joined devices for access control

## Azure RBAC (Role-Based Access Control)

### RBAC Components
- **Security Principal**: Who (user, group, service principal, managed identity)
- **Role Definition**: What permissions (actions that can be performed)
- **Scope**: Where (subscription, resource group, resource)
- **Role Assignment**: Combination of principal + role + scope

### RBAC Formula
```
Security Principal + Role Definition + Scope = Role Assignment
```

### Built-in Roles
| Role | Description | Scope | Use Case |
|------|-------------|-------|----------|
| **Owner** | Full access including delegation | Any | Administrative access |
| **Contributor** | Create and manage resources | Any | Developers, operators |
| **Reader** | View resources only | Any | Auditors, support staff |
| **User Access Administrator** | Manage user access only | Any | Security administrators |

### Service-Specific Roles
| Service | Role | Permissions |
|---------|------|-------------|
| **Storage** | Storage Blob Data Contributor | Read, write, delete blob data |
| **Virtual Machines** | Virtual Machine Contributor | Manage VMs (not access) |
| **Key Vault** | Key Vault Secrets User | Read secret contents |
| **SQL Database** | SQL DB Contributor | Manage databases (not access data) |

### Custom Roles
```json
{
  "Name": "Virtual Machine Operator",
  "Description": "Can start, stop, and restart virtual machines",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}/resourceGroups/production-rg"
  ]
}
```

### Role Assignment
```bash
# Assign role to user at resource group level
az role assignment create \
  --role "Virtual Machine Contributor" \
  --assignee user@company.com \
  --scope /subscriptions/{subscription-id}/resourceGroups/production-rg

# Assign role to group at subscription level  
az role assignment create \
  --role "Reader" \
  --assignee-object-id {group-object-id} \
  --scope /subscriptions/{subscription-id}
```

## Managed Identity

### What is Managed Identity?
Azure-managed identity that provides an automatically managed identity for applications to use when connecting to Azure resources.

### Managed Identity Types
| Type | Description | Use Case |
|------|-------------|----------|
| **System-assigned** | Tied to specific resource lifecycle | VM accessing Key Vault |
| **User-assigned** | Standalone identity resource | Multiple VMs sharing identity |

### System-Assigned Managed Identity
```bash
# Enable on VM
az vm identity assign --name myVM --resource-group myRG

# Use in application (no credentials needed)
```

```python
# Python example using managed identity
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://vault.vault.azure.net/", credential=credential)
secret = client.get_secret("database-password")
```

### User-Assigned Managed Identity
```bash
# Create user-assigned identity
az identity create --name myUserIdentity --resource-group myRG

# Assign to VM
az vm identity assign \
  --name myVM \
  --resource-group myRG \
  --identities /subscriptions/{sub-id}/resourcegroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserIdentity
```

## Azure Key Vault

### What is Key Vault?
Cloud service for securely storing and accessing secrets, keys, and certificates.

### Key Vault Objects
| Object Type | Description | Use Case |
|-------------|-------------|----------|
| **Secrets** | Passwords, connection strings, API keys | Application configuration |
| **Keys** | Encryption keys (RSA, EC, symmetric) | Data encryption |
| **Certificates** | SSL/TLS certificates | Web application security |

### Key Vault Access Models
#### Access Policies (Legacy)
```bash
# Set access policy for user
az keyvault set-policy \
  --name myKeyVault \
  --upn user@company.com \
  --secret-permissions get set delete \
  --key-permissions get create delete
```

#### RBAC (Recommended)
```bash
# Enable RBAC on Key Vault
az keyvault update --name myKeyVault --enable-rbac-authorization true

# Assign RBAC role
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee user@company.com \
  --scope /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myKeyVault
```

### Key Vault Security Features
- **Soft delete**: Recover accidentally deleted objects
- **Purge protection**: Prevent permanent deletion during retention period
- **Network access control**: Restrict access by IP/VNet
- **Private endpoints**: VNet-integrated access
- **Audit logging**: Track all Key Vault operations

### Using Key Vault in Applications
```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

# Create client with managed identity
credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://vault.vault.azure.net/", credential=credential)

# Get secret
secret = client.get_secret("database-connection-string")
connection_string = secret.value

# Set secret
client.set_secret("api-key", "secret-value")
```

## Conditional Access

### What is Conditional Access?
Feature that brings signals together to make decisions and enforce organizational policies.

### Conditional Access Signals
- **User/group membership**: Who is accessing
- **Location**: Where they're accessing from
- **Device**: What device they're using
- **Application**: What app they're accessing
- **Risk detection**: Sign-in and user risk levels

### Common Policies
| Policy | Condition | Control |
|--------|-----------|---------|
| **MFA for admins** | Admin roles | Require MFA |
| **Block risky sign-ins** | High risk sign-in | Block access |
| **Compliant devices** | All users | Require compliant device |
| **Location-based** | Outside corporate network | Require MFA |

### Policy Example
```json
{
  "displayName": "Require MFA for Azure management",
  "conditions": {
    "users": {
      "includeRoles": ["Global Administrator", "Security Administrator"]
    },
    "cloudApps": {
      "includeApplications": ["797f4846-ba00-4fd7-ba43-dac1f8f63013"]
    }
  },
  "grantControls": {
    "operator": "AND",
    "builtInControls": ["mfa"]
  }
}
```

## Azure Security Center / Microsoft Defender for Cloud

### What is Microsoft Defender for Cloud?
Cloud security posture management (CSPM) and cloud workload protection platform (CWPP).

### Defender for Cloud Features
- **Security posture management**: Assess and improve security posture
- **Threat protection**: Detect and respond to threats
- **Compliance assessment**: Monitor regulatory compliance
- **Security recommendations**: Actionable security guidance

### Defender Plans
| Plan | Protection | Resources |
|------|------------|-----------|
| **Free** | Basic CSPM | Azure resources |
| **Enhanced Security** | Advanced threat protection | Servers, databases, storage, containers |

### Security Score
- **Percentage-based**: 0-100% security score
- **Recommendations**: Actionable items to improve score
- **Impact**: Each recommendation shows score impact
- **Tracking**: Monitor improvements over time

### Common Recommendations
- Enable disk encryption on virtual machines
- Install endpoint protection on virtual machines
- Ensure that 'HTTP Version' is the latest for web apps
- Storage accounts should restrict network access
- MFA should be enabled on accounts with write permissions

## Azure Sentinel

### What is Azure Sentinel?
Cloud-native Security Information and Event Management (SIEM) and Security Orchestration, Automation, and Response (SOAR) solution.

### Sentinel Capabilities
- **Data collection**: Collect data from various sources
- **Detection**: AI-powered threat detection
- **Investigation**: Hunt for security threats
- **Response**: Automate threat response

### Data Connectors
| Source | Type | Use Case |
|--------|------|----------|
| **Azure services** | Native | Azure AD, Azure Activity |
| **Microsoft services** | API | Office 365, Microsoft 365 Defender |
| **Third-party** | API/Syslog | AWS CloudTrail, Palo Alto Networks |
| **Custom** | REST API | Custom applications |

### Analytics Rules
```json
{
  "displayName": "Suspicious PowerShell execution",
  "description": "Detects suspicious PowerShell commands",
  "severity": "Medium",
  "query": "SecurityEvent | where EventID == 4688 and Process contains 'powershell.exe' and CommandLine contains '-enc'",
  "frequency": "PT5M",
  "period": "PT5M"
}
```

## Network Security

### Network Security Groups (NSGs)
**Virtual firewall for controlling traffic**

#### NSG Rules
| Direction | Priority | Source | Destination | Protocol | Port | Action |
|-----------|----------|--------|-------------|----------|------|--------|
| Inbound | 100 | Internet | Any | TCP | 443 | Allow |
| Inbound | 200 | VirtualNetwork | Any | TCP | 22 | Allow |
| Inbound | 65000 | Any | Any | Any | Any | Deny |

```bash
# Create NSG rule
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myNSG \
  --name allow-ssh \
  --protocol tcp \
  --priority 1000 \
  --destination-port-range 22 \
  --access allow
```

### Azure Firewall
**Managed cloud-based network security service**

#### Firewall Rules
| Type | Description | Use Case |
|------|-------------|----------|
| **Network rules** | Layer 3-4 filtering | IP, port, protocol |
| **Application rules** | Layer 7 filtering | FQDN, URL categories |
| **NAT rules** | Destination NAT | Publish internal services |

#### Rule Example
```json
{
  "name": "allow-web-traffic",
  "protocols": ["TCP"],
  "sourceAddresses": ["10.0.0.0/16"],
  "destinationFqdns": ["*.microsoft.com", "*.windows.net"],
  "destinationPorts": ["80", "443"]
}
```

### Azure DDoS Protection
**Protection against distributed denial-of-service attacks**

#### Protection Tiers
| Tier | Protection | Cost | Features |
|------|------------|------|----------|
| **Basic** | Platform-level | Free | Always-on, automatic |
| **Standard** | Resource-specific | Paid | Attack analytics, metrics, alerts |

### Azure Bastion
**Secure RDP/SSH connectivity without public IPs**

#### Benefits
- **No public IPs**: VMs don't need public IP addresses
- **Browser-based**: RDP/SSH through Azure portal
- **No client software**: No VPN or special software needed
- **Fully managed**: Microsoft-managed service

```bash
# Deploy Azure Bastion
az network bastion create \
  --name myBastion \
  --public-ip-address myBastionIP \
  --resource-group myRG \
  --vnet-name myVNet \
  --location eastus
```

## Data Protection

### Azure Information Protection (AIP)
**Classify, label, and protect documents and emails**

#### Label Types
- **Public**: No business impact if shared
- **General**: Minor business impact
- **Confidential**: Moderate business impact
- **Highly Confidential**: Severe business impact

#### Protection Actions
- **Encrypt**: Apply encryption to content
- **Watermark**: Add visual watermarks
- **Access restrictions**: Control who can access
- **Expiration**: Set content expiration dates

### Customer-Managed Keys (CMK)
**Use your own encryption keys**

#### Key Management Options
| Option | Description | Use Case |
|--------|-------------|----------|
| **Microsoft-managed** | Microsoft handles keys | Default encryption |
| **Customer-managed** | You provide keys in Key Vault | Compliance requirements |
| **Customer-controlled** | You control key lifecycle | Highest control requirements |

```bash
# Enable CMK for storage account
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --encryption-key-vault https://vault.vault.azure.net/ \
  --encryption-key-name mykey \
  --encryption-key-version 1.0
```

### Always Encrypted
**Database encryption with client-side key management**

```sql
-- Create column master key
CREATE COLUMN MASTER KEY CMK1
WITH (
    KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
    KEY_PATH = 'https://vault.vault.azure.net/keys/CMK1/key-id'
);

-- Create encrypted column
CREATE TABLE Customers (
    CustomerId INT IDENTITY PRIMARY KEY,
    SSN CHAR(11) ENCRYPTED WITH (
        COLUMN_ENCRYPTION_KEY = CEK1,
        ENCRYPTION_TYPE = DETERMINISTIC,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
    ),
    Name NVARCHAR(50)
);
```

## Compliance and Governance

### Azure Policy
**Enforce organizational standards and assess compliance**

#### Policy Definition
```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        },
        {
          "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
          "notEquals": "true"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {},
  "metadata": {
    "displayName": "Storage accounts should only allow HTTPS traffic"
  }
}
```

#### Initiative (Policy Set)
```json
{
  "displayName": "Security baseline for Azure",
  "description": "Collection of security policies",
  "policyDefinitions": [
    {
      "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/storage-https-only",
      "parameters": {}
    },
    {
      "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/vm-disk-encryption",
      "parameters": {}
    }
  ]
}
```

### Azure Blueprints
**Declarative way to orchestrate deployment of resource templates, policies, and role assignments**

#### Blueprint Components
- **Resource Groups**: Organizational containers
- **ARM Templates**: Infrastructure as code
- **Policy Assignments**: Governance controls
- **Role Assignments**: Access controls

### Compliance Manager
**Assess and manage compliance across Microsoft cloud services**

#### Compliance Offerings
- **SOC 1/2/3**: Service organization controls
- **ISO 27001**: Information security management
- **PCI DSS**: Payment card industry standards
- **HIPAA**: Healthcare data protection
- **GDPR**: European data protection regulation

## Security Best Practices

### Identity and Access
1. **Use Azure AD**: Centralize identity management
2. **Enable MFA**: Multi-factor authentication for all users
3. **Implement Conditional Access**: Control access based on conditions
4. **Use managed identities**: Avoid storing credentials in code
5. **Regular access reviews**: Audit and remove unnecessary access

### Network Security
1. **Network segmentation**: Use VNets and subnets to isolate resources
2. **Just-in-time access**: Minimize exposed attack surface
3. **Monitor network traffic**: Use NSG flow logs and Azure Monitor
4. **Use Azure Firewall**: Centralized network security
5. **Implement Zero Trust**: Never trust, always verify

### Data Protection
1. **Encrypt data**: At rest and in transit
2. **Use Key Vault**: Centralized secrets management
3. **Classify data**: Understand and label sensitive data
4. **Backup regularly**: Implement backup and disaster recovery
5. **Monitor access**: Audit data access patterns

### Governance and Compliance
1. **Use Azure Policy**: Enforce organizational standards
2. **Implement tagging**: Organize and track resources
3. **Monitor compliance**: Regular compliance assessments
4. **Document procedures**: Clear security policies and procedures
5. **Regular training**: Keep staff updated on security practices

## Common Security Pitfalls

### Identity Management Issues
- **Over-privileged accounts**: Granting excessive permissions
- **Shared accounts**: Multiple people using same credentials
- **Weak authentication**: Not enabling MFA
- **Stale accounts**: Not removing access for former employees
- **Service account sprawl**: Too many service accounts with unclear purposes

### Network Security Gaps
- **Overly permissive NSGs**: Allowing unnecessary traffic
- **Public endpoints**: Exposing services to internet unnecessarily
- **Missing monitoring**: Not tracking network traffic
- **Weak segmentation**: All resources in same network segment
- **Default configurations**: Not customizing security settings

### Data Protection Issues
- **Unencrypted data**: Not encrypting sensitive information
- **Weak key management**: Poor encryption key practices
- **Missing backups**: Inadequate backup and recovery plans
- **Excessive permissions**: Too broad data access
- **Data location**: Not considering data residency requirements

### Operational Security
- **Missing monitoring**: Insufficient security monitoring and alerting
- **Slow response**: Poor incident response procedures
- **Outdated systems**: Not keeping systems and software updated
- **Poor documentation**: Lack of security policies and procedures
- **Inadequate training**: Staff not trained on security practices