# IBM Cloud Security Engineer (C1000-178) - Security Fundamentals & IAM

## Table of Contents
- [Security Fundamentals](#security-fundamentals)
- [Identity and Access Management (IAM)](#identity-and-access-management-iam)
- [Access Groups and Policies](#access-groups-and-policies)
- [Service IDs and API Keys](#service-ids-and-api-keys)
- [Trusted Profiles](#trusted-profiles)
- [Multi-Factor Authentication](#multi-factor-authentication)
- [Context-Based Restrictions](#context-based-restrictions)
- [Secrets Management](#secrets-management)
- [Certificate Management](#certificate-management)

---

## Security Fundamentals

### IBM Cloud Security Architecture

IBM Cloud implements a shared responsibility model:

**IBM Responsibilities**:
- Physical datacenter security
- Network infrastructure
- Hypervisor and platform security
- Service availability

**Customer Responsibilities**:
- Identity and access management
- Data encryption
- Application security
- Compliance and governance

### Security Principles

1. **Defense in Depth**
   - Multiple layers of security
   - No single point of failure
   - Redundant security controls

2. **Least Privilege**
   - Minimum necessary access
   - Time-bound permissions
   - Regular access reviews

3. **Zero Trust**
   - Never trust, always verify
   - Verify explicitly
   - Use least privileged access
   - Assume breach

### Security Zones

```
┌─────────────────────────────────────┐
│         Public Internet             │
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│      DMZ / Edge Network             │
│  - Web Application Firewall         │
│  - Load Balancers                   │
│  - API Gateway                      │
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│    Application Zone (VPC)           │
│  - Application Servers              │
│  - Microservices                    │
└─────────────────┬───────────────────┘
                  │
┌─────────────────▼───────────────────┐
│      Data Zone (Isolated VPC)       │
│  - Databases                        │
│  - Data Stores                      │
│  - Encrypted Storage                │
└─────────────────────────────────────┘
```

---

## Identity and Access Management (IAM)

### IAM Concepts

1. **Users**: Individuals with IBM Cloud accounts
2. **Service IDs**: Machine identities for applications
3. **Access Groups**: Collections of users and service IDs
4. **Policies**: Define what actions entities can perform
5. **Resources**: IBM Cloud services and objects
6. **Roles**: Collections of actions

### IAM Roles

**Platform Management Roles**:
- **Viewer**: View resources and resource groups
- **Operator**: View and perform platform actions (restart, view logs)
- **Editor**: View, create, delete, and modify resources
- **Administrator**: All platform actions + assign access to others

**Service Access Roles**:
- **Reader**: Read-only access to service data
- **Writer**: Read and write service data
- **Manager**: Full service access

**Custom Roles**:
- Define specific actions
- Granular control
- Service-specific

### Managing Users via CLI

```bash
# List users in account
ibmcloud account users

# Invite user to account
ibmcloud account user-invite user@example.com

# Remove user from account
ibmcloud account user-remove user@example.com

# Get user details
ibmcloud account user-info user@example.com

# List access policies for a user
ibmcloud iam user-policies user@example.com
```

### Creating Access Policies

```bash
# Grant user access to all resources in resource group
ibmcloud iam user-policy-create user@example.com \
  --roles Viewer,Operator \
  --resource-group-name production

# Grant access to specific service
ibmcloud iam user-policy-create user@example.com \
  --roles Administrator \
  --service-name cloud-object-storage

# Grant access to specific resource instance
ibmcloud iam user-policy-create user@example.com \
  --roles Manager \
  --service-name cloud-object-storage \
  --service-instance my-cos-instance

# Grant access with attributes
ibmcloud iam user-policy-create user@example.com \
  --roles Editor \
  --service-name is \
  --attributes "vpcId=vpc-12345"

# Time-bound policy (expires in 24 hours)
ibmcloud iam user-policy-create user@example.com \
  --roles Viewer \
  --service-name containers-kubernetes \
  --time-condition valid-for=86400
```

### IAM Policy JSON

```json
{
  "type": "access",
  "subjects": [
    {
      "attributes": [
        {
          "name": "iam_id",
          "value": "IBMid-123456"
        }
      ]
    }
  ],
  "roles": [
    {
      "role_id": "crn:v1:bluemix:public:iam::::role:Editor"
    }
  ],
  "resources": [
    {
      "attributes": [
        {
          "name": "accountId",
          "value": "account-id"
        },
        {
          "name": "serviceName",
          "value": "cloud-object-storage"
        },
        {
          "name": "resource",
          "value": "my-bucket",
          "operator": "stringEquals"
        }
      ]
    }
  ]
}
```

### Python SDK for IAM

```python
# iam-management.py
from ibm_platform_services import IamPolicyManagementV1
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator

authenticator = IAMAuthenticator(api_key)
iam_policy_service = IamPolicyManagementV1(authenticator=authenticator)

# Create access policy
policy = {
    'type': 'access',
    'subjects': [
        {
            'attributes': [
                {
                    'name': 'iam_id',
                    'value': 'IBMid-123456'
                }
            ]
        }
    ],
    'roles': [
        {
            'role_id': 'crn:v1:bluemix:public:iam::::role:Editor'
        }
    ],
    'resources': [
        {
            'attributes': [
                {
                    'name': 'accountId',
                    'value': account_id
                },
                {
                    'name': 'serviceName',
                    'value': 'cloud-object-storage'
                }
            ]
        }
    ]
}

response = iam_policy_service.create_policy(**policy)
print(f"Policy created: {response.result['id']}")

# List policies
policies = iam_policy_service.list_policies(
    account_id=account_id,
    iam_id='IBMid-123456',
    type='access'
)

for policy in policies.result['policies']:
    print(f"Policy ID: {policy['id']}")
    print(f"Roles: {[r['display_name'] for r in policy['roles']]}")

# Delete policy
iam_policy_service.delete_policy(policy_id='policy-id')
```

---

## Access Groups and Policies

### Creating Access Groups

```bash
# Create access group
ibmcloud iam access-group-create developers \
  --description "Development team members"

# Add users to access group
ibmcloud iam access-group-user-add developers user1@example.com
ibmcloud iam access-group-user-add developers user2@example.com

# Add service ID to access group
ibmcloud iam access-group-service-id-add developers ServiceId-12345

# Remove user from access group
ibmcloud iam access-group-user-remove developers user1@example.com

# List access groups
ibmcloud iam access-groups

# List members of access group
ibmcloud iam access-group-users developers
```

### Access Group Policies

```bash
# Create policy for access group
ibmcloud iam access-group-policy-create developers \
  --roles Editor,Operator \
  --service-name containers-kubernetes

# Grant access to resource group
ibmcloud iam access-group-policy-create developers \
  --roles Viewer \
  --resource-group-name development

# Grant access with conditions
ibmcloud iam access-group-policy-create developers \
  --roles Administrator \
  --service-name cloud-object-storage \
  --service-instance dev-storage \
  --attributes "bucketName=dev-*"

# List policies for access group
ibmcloud iam access-group-policies developers

# Delete access group policy
ibmcloud iam access-group-policy-delete developers policy-id

# Delete access group
ibmcloud iam access-group-delete developers
```

### Dynamic Rules for Access Groups

```bash
# Create dynamic rule based on SAML attributes
ibmcloud iam access-group-rule-create developers \
  --name "LDAP Users" \
  --expiration 12 \
  --realm-name "https://idp.example.com" \
  --conditions '[
    {
      "claim": "blueGroups",
      "operator": "CONTAINS",
      "value": "developers"
    }
  ]'

# List rules for access group
ibmcloud iam access-group-rules developers

# Delete rule
ibmcloud iam access-group-rule-delete developers rule-id
```

### Terraform for Access Groups

```hcl
# access-groups.tf
resource "ibm_iam_access_group" "developers" {
  name        = "developers"
  description = "Development team members"
}

resource "ibm_iam_access_group_members" "dev_members" {
  access_group_id = ibm_iam_access_group.developers.id

  ibm_ids = [
    "user1@example.com",
    "user2@example.com"
  ]

  iam_service_ids = [
    ibm_iam_service_id.app_service_id.id
  ]
}

resource "ibm_iam_access_group_policy" "dev_k8s_policy" {
  access_group_id = ibm_iam_access_group.developers.id
  roles           = ["Editor", "Operator"]

  resources {
    service = "containers-kubernetes"
    resource_group_id = data.ibm_resource_group.dev.id
  }
}

resource "ibm_iam_access_group_policy" "dev_cos_policy" {
  access_group_id = ibm_iam_access_group.developers.id
  roles           = ["Writer"]

  resources {
    service           = "cloud-object-storage"
    resource_instance_id = ibm_resource_instance.cos.id
  }

  resource_attributes {
    name     = "bucketName"
    value    = "dev-*"
    operator = "stringMatch"
  }
}

resource "ibm_iam_access_group_dynamic_rule" "ldap_rule" {
  access_group_id = ibm_iam_access_group.developers.id
  name            = "LDAP Developers"
  expiration      = 12
  identity_provider = "https://idp.example.com"

  conditions {
    claim    = "blueGroups"
    operator = "CONTAINS"
    value    = "developers"
  }
}
```

---

## Service IDs and API Keys

### Service IDs

Service IDs are machine identities used for applications and services.

```bash
# Create service ID
ibmcloud iam service-id-create app-service-id \
  --description "Service ID for application"

# List service IDs
ibmcloud iam service-ids

# Get service ID details
ibmcloud iam service-id app-service-id

# Update service ID
ibmcloud iam service-id-update app-service-id \
  --description "Updated description"

# Lock service ID (prevent deletion)
ibmcloud iam service-id-lock app-service-id

# Unlock service ID
ibmcloud iam service-id-unlock app-service-id

# Delete service ID
ibmcloud iam service-id-delete app-service-id
```

### Service ID Policies

```bash
# Create policy for service ID
ibmcloud iam service-policy-create app-service-id \
  --roles Writer \
  --service-name cloud-object-storage \
  --service-instance my-cos

# Grant access to Kubernetes
ibmcloud iam service-policy-create app-service-id \
  --roles Editor \
  --service-name containers-kubernetes \
  --resource-group-name production

# List policies for service ID
ibmcloud iam service-policies app-service-id

# Delete policy
ibmcloud iam service-policy-delete app-service-id policy-id
```

### API Keys

```bash
# Create API key for user
ibmcloud iam api-key-create my-api-key \
  --description "Personal API key" \
  --file key.json

# Create API key for service ID
ibmcloud iam service-api-key-create app-api-key app-service-id \
  --description "Application API key" \
  --file app-key.json

# List user API keys
ibmcloud iam api-keys

# List service ID API keys
ibmcloud iam service-api-keys app-service-id

# Lock API key (prevent usage)
ibmcloud iam api-key-lock my-api-key

# Unlock API key
ibmcloud iam api-key-unlock my-api-key

# Delete API key
ibmcloud iam api-key-delete my-api-key

# Rotate API key (create new, delete old)
ibmcloud iam api-key-create my-api-key-v2 --file new-key.json
# Update application with new key
ibmcloud iam api-key-delete my-api-key
```

### Using Service IDs in Applications

```javascript
// service-id-auth.js
const { IamAuthenticator } = require('ibm-cloud-sdk-core');
const { CloudantV1 } = require('@ibm-cloud/cloudant');

// Authenticate using service ID API key
const authenticator = new IamAuthenticator({
  apikey: process.env.SERVICE_ID_API_KEY
});

const cloudant = CloudantV1.newInstance({
  authenticator
});

async function accessCloudant() {
  try {
    const databases = await cloudant.getAllDbs();
    console.log('Databases:', databases.result);
  } catch (err) {
    console.error('Error:', err);
  }
}
```

### Terraform for Service IDs

```hcl
# service-ids.tf
resource "ibm_iam_service_id" "app_service_id" {
  name        = "app-service-id"
  description = "Service ID for application"
}

resource "ibm_iam_service_policy" "cos_policy" {
  iam_service_id = ibm_iam_service_id.app_service_id.id
  roles          = ["Writer"]

  resources {
    service              = "cloud-object-storage"
    resource_instance_id = ibm_resource_instance.cos.guid
  }
}

resource "ibm_iam_service_policy" "k8s_policy" {
  iam_service_id = ibm_iam_service_id.app_service_id.id
  roles          = ["Editor", "Manager"]

  resources {
    service           = "containers-kubernetes"
    resource_group_id = data.ibm_resource_group.prod.id
  }
}

resource "ibm_iam_service_api_key" "app_api_key" {
  name           = "app-api-key"
  iam_service_id = ibm_iam_service_id.app_service_id.iam_id
  description    = "API key for application authentication"

  # Store in Secrets Manager
  store_value = false
}

# Store API key in Secrets Manager
resource "ibm_sm_arbitrary_secret" "api_key" {
  instance_id = ibm_resource_instance.secrets_manager.guid
  region      = var.region
  name        = "app-api-key"
  description = "Application API key"
  payload     = ibm_iam_service_api_key.app_api_key.apikey
}
```

---

## Trusted Profiles

Trusted Profiles enable compute resources to authenticate without API keys.

### Creating Trusted Profiles

```bash
# Create trusted profile
ibmcloud iam trusted-profile-create app-profile \
  --description "Trusted profile for application"

# Add compute resource rule (Kubernetes)
ibmcloud iam trusted-profile-rule-create app-profile \
  --name k8s-rule \
  --type "iks" \
  --conditions '[
    {
      "claim": "namespace",
      "operator": "EQUALS",
      "value": "production"
    },
    {
      "claim": "serviceaccount",
      "operator": "EQUALS",
      "value": "myapp"
    }
  ]'

# Add compute resource rule (Code Engine)
ibmcloud iam trusted-profile-rule-create app-profile \
  --name ce-rule \
  --type "codeengine" \
  --conditions '[
    {
      "claim": "project_id",
      "operator": "EQUALS",
      "value": "project-123"
    }
  ]'

# Create policy for trusted profile
ibmcloud iam trusted-profile-policy-create app-profile \
  --roles Writer \
  --service-name cloud-object-storage

# List trusted profiles
ibmcloud iam trusted-profiles

# List rules for profile
ibmcloud iam trusted-profile-rules app-profile

# Delete trusted profile
ibmcloud iam trusted-profile-delete app-profile
```

### Using Trusted Profiles in Kubernetes

```yaml
# kubernetes-service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp
  namespace: production
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      serviceAccountName: myapp
      containers:
      - name: myapp
        image: us.icr.io/namespace/myapp:latest
        env:
        - name: IBM_CLOUD_IAM_PROFILE_ID
          value: "Profile-12345"
```

```javascript
// trusted-profile-auth.js
const { ContainerAuthenticator } = require('ibm-cloud-sdk-core');
const { CloudObjectStorageV2 } = require('ibm-cos-sdk');

// Authenticate using trusted profile
const authenticator = new ContainerAuthenticator({
  iamProfileId: process.env.IBM_CLOUD_IAM_PROFILE_ID
});

const cos = new CloudObjectStorageV2({
  authenticator,
  serviceUrl: 'https://s3.us-south.cloud-object-storage.appdomain.cloud'
});

async function listBuckets() {
  try {
    const buckets = await cos.listBuckets();
    console.log('Buckets:', buckets.Buckets);
  } catch (err) {
    console.error('Error:', err);
  }
}
```

### Terraform for Trusted Profiles

```hcl
# trusted-profiles.tf
resource "ibm_iam_trusted_profile" "app_profile" {
  name        = "app-profile"
  description = "Trusted profile for application workloads"
}

resource "ibm_iam_trusted_profile_claim_rule" "k8s_rule" {
  profile_id = ibm_iam_trusted_profile.app_profile.id
  name       = "kubernetes-rule"
  type       = "iks"

  conditions {
    claim    = "namespace"
    operator = "EQUALS"
    value    = "production"
  }

  conditions {
    claim    = "serviceaccount"
    operator = "EQUALS"
    value    = "myapp"
  }
}

resource "ibm_iam_trusted_profile_claim_rule" "ce_rule" {
  profile_id = ibm_iam_trusted_profile.app_profile.id
  name       = "code-engine-rule"
  type       = "codeengine"

  conditions {
    claim    = "project_id"
    operator = "EQUALS"
    value    = var.code_engine_project_id
  }
}

resource "ibm_iam_trusted_profile_policy" "cos_policy" {
  profile_id = ibm_iam_trusted_profile.app_profile.id
  roles      = ["Writer", "ContentReader"]

  resources {
    service              = "cloud-object-storage"
    resource_instance_id = ibm_resource_instance.cos.guid
  }
}

resource "ibm_iam_trusted_profile_policy" "cloudant_policy" {
  profile_id = ibm_iam_trusted_profile.app_profile.id
  roles      = ["Writer"]

  resources {
    service              = "cloudantnosqldb"
    resource_instance_id = ibm_resource_instance.cloudant.guid
  }
}
```

---

## Multi-Factor Authentication

### Enabling MFA

```bash
# Require MFA for all users in account
ibmcloud account update --mfa LEVEL1  # MFA for IBMid
ibmcloud account update --mfa LEVEL2  # MFA for IBMid + external authentication
ibmcloud account update --mfa LEVEL3  # MFA for all authentication methods

# Check MFA status
ibmcloud account show

# Require MFA for specific user
ibmcloud iam user-mfa-update user@example.com --mfa TOTP
```

### MFA Types

1. **TOTP (Time-based One-Time Password)**
   - Google Authenticator
   - Microsoft Authenticator
   - Authy

2. **U2F (Universal 2nd Factor)**
   - YubiKey
   - Titan Security Key
   - Other FIDO U2F devices

3. **SMS**
   - Text message codes
   - Less secure, not recommended

### Terraform for MFA

```hcl
# mfa-settings.tf
resource "ibm_iam_account_settings" "account_settings" {
  mfa = "LEVEL3"  # Require MFA for all users

  session_expiration_in_seconds = "7200"  # 2 hours
  session_invalidation_in_seconds = "86400"  # 24 hours

  restrict_create_service_id = "RESTRICTED"
  restrict_create_platform_apikey = "RESTRICTED"
}
```

---

## Context-Based Restrictions

Context-Based Restrictions (CBR) control access based on network location and other contexts.

### Network Zones

```bash
# Create network zone with IP addresses
ibmcloud cbr zone-create \
  --name office-network \
  --description "Corporate office network" \
  --addresses 203.0.113.0/24,198.51.100.0/24

# Create network zone with VPC
ibmcloud cbr zone-create \
  --name production-vpc \
  --description "Production VPC" \
  --vpc vpc-id

# Create network zone with service reference
ibmcloud cbr zone-create \
  --name code-engine \
  --description "Code Engine service" \
  --service-ref service_name=codeengine,location=us-south

# List network zones
ibmcloud cbr zones

# Update network zone
ibmcloud cbr zone-update zone-id \
  --addresses 203.0.113.0/24,198.51.100.0/24,192.0.2.0/24

# Delete network zone
ibmcloud cbr zone-delete zone-id
```

### Context-Based Restriction Rules

```bash
# Create CBR rule for Cloud Object Storage
ibmcloud cbr rule-create \
  --description "Restrict COS access to office and VPC" \
  --service-name cloud-object-storage \
  --resource-attributes "accountId=account-id,serviceInstance=instance-id" \
  --context-attributes "networkZoneId=zone-id-1,networkZoneId=zone-id-2" \
  --enforcement-mode enabled

# Create CBR rule with multiple contexts
ibmcloud cbr rule-create \
  --description "Restrict database access" \
  --service-name databases-for-postgresql \
  --resource-attributes "accountId=account-id" \
  --context-attributes "networkZoneId=zone-id,endpointType=private" \
  --enforcement-mode enabled

# List CBR rules
ibmcloud cbr rules

# Update CBR rule enforcement mode
ibmcloud cbr rule-update rule-id --enforcement-mode report

# Delete CBR rule
ibmcloud cbr rule-delete rule-id
```

### Terraform for Context-Based Restrictions

```hcl
# cbr-rules.tf
resource "ibm_cbr_zone" "office_network" {
  name        = "office-network"
  description = "Corporate office network"
  account_id  = var.account_id

  addresses {
    type = "ipAddress"
    value = "203.0.113.0/24"
  }

  addresses {
    type = "ipAddress"
    value = "198.51.100.0/24"
  }
}

resource "ibm_cbr_zone" "production_vpc" {
  name        = "production-vpc"
  description = "Production VPC"
  account_id  = var.account_id

  addresses {
    type = "vpc"
    value = ibm_is_vpc.production.crn
  }
}

resource "ibm_cbr_zone" "code_engine_service" {
  name        = "code-engine"
  description = "Code Engine service reference"
  account_id  = var.account_id

  addresses {
    type = "serviceRef"
    ref {
      account_id   = var.account_id
      service_name = "codeengine"
      location     = "us-south"
    }
  }
}

resource "ibm_cbr_rule" "cos_restriction" {
  description = "Restrict COS access to approved networks"

  contexts {
    attributes {
      name  = "networkZoneId"
      value = ibm_cbr_zone.office_network.id
    }
  }

  contexts {
    attributes {
      name  = "networkZoneId"
      value = ibm_cbr_zone.production_vpc.id
    }
  }

  resources {
    attributes {
      name  = "accountId"
      value = var.account_id
    }

    attributes {
      name  = "serviceName"
      value = "cloud-object-storage"
    }

    attributes {
      name     = "serviceInstance"
      value    = ibm_resource_instance.cos.guid
      operator = "stringEquals"
    }
  }

  enforcement_mode = "enabled"
}

resource "ibm_cbr_rule" "database_restriction" {
  description = "Restrict database access to VPC only"

  contexts {
    attributes {
      name  = "networkZoneId"
      value = ibm_cbr_zone.production_vpc.id
    }

    attributes {
      name  = "endpointType"
      value = "private"
    }
  }

  resources {
    attributes {
      name  = "accountId"
      value = var.account_id
    }

    attributes {
      name  = "serviceName"
      value = "databases-for-postgresql"
    }
  }

  enforcement_mode = "enabled"
}
```

---

## Secrets Management

### IBM Secrets Manager

```bash
# Create Secrets Manager instance
ibmcloud resource service-instance-create \
  my-secrets-manager \
  secrets-manager \
  standard \
  us-south

# Create secret group
ibmcloud secrets-manager secret-group-create \
  --instance-id $INSTANCE_ID \
  --name production \
  --description "Production secrets"

# Create arbitrary secret
ibmcloud secrets-manager secret-create \
  --instance-id $INSTANCE_ID \
  --secret-type arbitrary \
  --name database-password \
  --secret-group-id $GROUP_ID \
  --payload "super-secret-password"

# Create IAM credentials secret
ibmcloud secrets-manager secret-create \
  --instance-id $INSTANCE_ID \
  --secret-type iam_credentials \
  --name app-service-id-key \
  --secret-group-id $GROUP_ID \
  --service-id $SERVICE_ID \
  --ttl 86400

# Get secret
ibmcloud secrets-manager secret-get \
  --instance-id $INSTANCE_ID \
  --id $SECRET_ID

# List secrets
ibmcloud secrets-manager secrets-list \
  --instance-id $INSTANCE_ID

# Delete secret
ibmcloud secrets-manager secret-delete \
  --instance-id $INSTANCE_ID \
  --id $SECRET_ID
```

### Secrets Manager SDK

```javascript
// secrets-manager.js
const SecretsManagerV2 = require('@ibm-cloud/secrets-manager-sdk');
const { IamAuthenticator } = require('ibm-cloud-sdk-core');

const authenticator = new IamAuthenticator({
  apikey: process.env.IBM_CLOUD_API_KEY
});

const secretsManager = new SecretsManagerV2({
  authenticator,
  serviceUrl: process.env.SECRETS_MANAGER_URL
});

// Create arbitrary secret
async function createSecret(name, payload) {
  try {
    const response = await secretsManager.createSecret({
      secretType: 'arbitrary',
      secretGroupId: process.env.SECRET_GROUP_ID,
      name,
      payload
    });
    console.log('Secret created:', response.result.id);
    return response.result;
  } catch (err) {
    console.error('Error creating secret:', err);
    throw err;
  }
}

// Get secret
async function getSecret(secretId) {
  try {
    const response = await secretsManager.getSecret({
      id: secretId
    });
    return response.result.payload;
  } catch (err) {
    console.error('Error getting secret:', err);
    throw err;
  }
}

// Update secret
async function updateSecret(secretId, newPayload) {
  try {
    await secretsManager.updateSecret({
      id: secretId,
      secretType: 'arbitrary',
      payload: newPayload
    });
    console.log('Secret updated');
  } catch (err) {
    console.error('Error updating secret:', err);
    throw err;
  }
}

// Rotate IAM credentials
async function rotateIAMCredentials(secretId) {
  try {
    const response = await secretsManager.createSecretVersion({
      secretId,
      secretVersionPrototype: {
        rotation: {
          auto_rotate: true,
          interval: 30  // days
        }
      }
    });
    console.log('Credentials rotated:', response.result);
  } catch (err) {
    console.error('Error rotating credentials:', err);
    throw err;
  }
}

module.exports = { createSecret, getSecret, updateSecret, rotateIAMCredentials };
```

### Using Secrets in Applications

```javascript
// app-with-secrets.js
const express = require('express');
const { getSecret } = require('./secrets-manager');

const app = express();

// Load secrets on startup
let dbPassword;
let apiKey;

async function loadSecrets() {
  dbPassword = await getSecret(process.env.DB_PASSWORD_SECRET_ID);
  apiKey = await getSecret(process.env.API_KEY_SECRET_ID);
}

// Refresh secrets periodically
setInterval(async () => {
  await loadSecrets();
  console.log('Secrets refreshed');
}, 3600000); // Every hour

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(3000, async () => {
  await loadSecrets();
  console.log('Server started');
});
```

---

## Certificate Management

### IBM Certificate Manager

```bash
# Create Certificate Manager instance
ibmcloud resource service-instance-create \
  my-cert-manager \
  cloudcerts \
  free \
  us-south

# Import certificate
ibmcloud certificate-manager certificate-import \
  --instance-id $INSTANCE_ID \
  --name my-certificate \
  --certificate-file cert.pem \
  --private-key-file key.pem \
  --intermediate-file intermediate.pem

# Order Let's Encrypt certificate
ibmcloud certificate-manager certificate-order \
  --instance-id $INSTANCE_ID \
  --name my-domain-cert \
  --domains example.com,www.example.com \
  --key-algorithm RSA2048

# List certificates
ibmcloud certificate-manager certificates-list \
  --instance-id $INSTANCE_ID

# Get certificate details
ibmcloud certificate-manager certificate-get \
  --instance-id $INSTANCE_ID \
  --certificate-id $CERT_ID

# Renew certificate
ibmcloud certificate-manager certificate-renew \
  --instance-id $INSTANCE_ID \
  --certificate-id $CERT_ID

# Delete certificate
ibmcloud certificate-manager certificate-delete \
  --instance-id $INSTANCE_ID \
  --certificate-id $CERT_ID
```

### Exam Tips

1. **IAM Fundamentals**
   - Understand the difference between platform and service roles
   - Know when to use access groups vs. individual policies
   - Master policy creation with attributes and conditions
   - Study time-bound policies

2. **Service IDs**
   - Know when to use service IDs vs. API keys
   - Understand service ID locking
   - Practice API key rotation
   - Study best practices for storing credentials

3. **Trusted Profiles**
   - Understand compute resource authentication
   - Know supported compute services (IKS, ROKS, Code Engine)
   - Practice creating claim rules
   - Study advantages over API keys

4. **Access Control**
   - Master context-based restrictions
   - Understand network zones
   - Know enforcement modes (report vs. enabled)
   - Study VPC and service reference zones

5. **Secrets Management**
   - Understand different secret types
   - Know secret rotation strategies
   - Practice with Secrets Manager SDK
   - Study integration with applications

### Common Scenarios

**Scenario 1**: Grant development team access to specific resources
```bash
# Solution: Use access groups with resource-scoped policies
# Create access group, add users, assign policies with resource group
```

**Scenario 2**: Secure application authentication without API keys
```bash
# Solution: Use trusted profiles with compute resources
# Create trusted profile, add claim rules, assign policies
```

**Scenario 3**: Restrict database access to specific networks
```bash
# Solution: Implement context-based restrictions
# Create network zones, create CBR rules with zones
```
