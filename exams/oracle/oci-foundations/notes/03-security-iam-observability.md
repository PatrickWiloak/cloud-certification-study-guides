# OCI Foundations - Security, IAM, and Observability

## Table of Contents
- [Identity and Access Management (IAM)](#identity-and-access-management-iam)
  - [IAM Concepts](#iam-concepts)
  - [Users and Groups](#users-and-groups)
  - [Policies](#policies)
  - [Dynamic Groups](#dynamic-groups)
  - [Federation](#federation)
- [Security Services](#security-services)
  - [Cloud Guard](#cloud-guard)
  - [Security Zones](#security-zones)
  - [Vault](#vault)
  - [WAF and DDoS Protection](#waf-and-ddos-protection)
- [Observability and Management](#observability-and-management)
  - [Monitoring](#monitoring)
  - [Logging](#logging)
  - [Notifications](#notifications)
  - [Events](#events)
- [Cost Management and Pricing](#cost-management-and-pricing)
  - [Pricing Models](#pricing-models)
  - [Cost Analysis](#cost-analysis)
  - [Billing and Usage](#billing-and-usage)
- [Support and SLAs](#support-and-slas)
- [Exam Tips](#exam-tips)

---

## Identity and Access Management (IAM)

### IAM Concepts

**Definition**: Service that controls who can access which resources in OCI.

**Key Principles**:
- **Authentication**: Who are you? (User identity)
- **Authorization**: What can you do? (Permissions via policies)
- **Least Privilege**: Grant minimum necessary permissions
- **Zero Trust**: Verify every access request

**IAM Resources**:
- Users
- Groups
- Dynamic Groups
- Policies
- Identity Providers (Federation)
- Domains (Identity Domains)

#### Principal Types

**1. IAM Users**:
- Individual people or applications
- Created in IAM
- Authenticated via password or API keys

**2. Resource Principals**:
- OCI resources acting on their own
- No user credentials needed
- Examples: Functions, instances

**3. Federated Users**:
- External identity provider users
- SAML 2.0 or other federation

### Users and Groups

#### Users

**Definition**: Individual identity for person or application.

**User Types**:
- **Local Users**: Created in OCI IAM
- **Federated Users**: From external IdP

**Creating Users**:
```bash
oci iam user create \
  --name "john.doe@example.com" \
  --description "Developer" \
  --email "john.doe@example.com"
```

**User Capabilities**:
```bash
# Set user capabilities
oci iam user update-user-capabilities \
  --user-id ocid1.user.oc1... \
  --can-use-console-password true \
  --can-use-api-keys true \
  --can-use-auth-tokens true \
  --can-use-customer-secret-keys true
```

#### Credentials

**1. Console Password**:
- Web console access
- Should use MFA
- Password policies enforced

**2. API Signing Keys**:
- API/CLI/SDK access
- RSA key pair
- Public key uploaded to OCI

**Generating API Keys**:
```bash
# Generate key pair
mkdir ~/.oci
openssl genrsa -out ~/.oci/oci_api_key.pem 2048
openssl rsa -pubout -in ~/.oci/oci_api_key.pem -out ~/.oci/oci_api_key_public.pem

# Upload public key
oci iam user api-key upload \
  --user-id ocid1.user.oc1... \
  --key-file ~/.oci/oci_api_key_public.pem
```

**3. Auth Tokens**:
- Oracle Cloud Infrastructure Registry (OCIR)
- Swift API access
- Generated in console

**4. Customer Secret Keys**:
- Amazon S3 compatibility API
- Access Object Storage with S3 tools

**5. SMTP Credentials**:
- Email Delivery service
- Send emails from applications

#### Multi-Factor Authentication (MFA)

**Definition**: Additional security layer requiring second factor.

**MFA Types**:
- Time-based One-Time Password (TOTP) apps
- FIDO2 security keys
- Mobile authenticator apps (Google Authenticator, Microsoft Authenticator)

**Enabling MFA**:
1. Navigate to user profile
2. Enable MFA
3. Scan QR code with authenticator app
4. Enter verification code

**Best Practice**: Require MFA for all users, especially administrators.

#### Groups

**Definition**: Collection of users with similar access needs.

**Characteristics**:
- Users can belong to multiple groups
- Policies grant permissions to groups
- Easier management than individual user permissions

**Creating Groups**:
```bash
oci iam group create \
  --name "Developers" \
  --description "Development team members"
```

**Adding Users to Groups**:
```bash
oci iam group add-user \
  --group-id ocid1.group.oc1... \
  --user-id ocid1.user.oc1...
```

**Common Group Structure**:
```
├── Administrators (Full access)
├── NetworkAdmins (Network management)
├── DatabaseAdmins (Database management)
├── Developers (Read/write to dev compartment)
├── QA-Testers (Read/write to test compartment)
└── Auditors (Read-only access)
```

### Policies

**Definition**: Documents that define permissions for groups or resources.

#### Policy Syntax

**Basic Format**:
```
Allow <subject> to <verb> <resource-type> in <location> where <conditions>
```

**Components**:
- **Subject**: Who (group, dynamic group, any-user)
- **Verb**: What action (inspect, read, use, manage)
- **Resource-type**: What resource (instances, volumes, databases)
- **Location**: Where (tenancy, compartment)
- **Conditions**: Optional constraints

#### Permission Verbs

**Verb Hierarchy** (least to most permissive):
```
inspect < read < use < manage
```

**1. inspect**:
- List resources
- View metadata
- Cannot view contents
- Example: List all instances

**2. read**:
- inspect permissions +
- View resource details
- View configurations
- Cannot modify
- Example: View instance details

**3. use**:
- read permissions +
- Use existing resources
- Update configurations
- Cannot create or delete
- Example: Start/stop instances

**4. manage**:
- Complete control
- Create, read, update, delete
- All operations
- Example: Full instance management

#### Policy Examples

**Administrator Access**:
```
Allow group Administrators to manage all-resources in tenancy
```

**Compartment-Specific Access**:
```
Allow group Developers to manage instances in compartment Development
Allow group Developers to manage volumes in compartment Development
Allow group Developers to use virtual-network-family in compartment Development
```

**Read-Only Access**:
```
Allow group Auditors to inspect all-resources in tenancy
Allow group Auditors to read all-resources in tenancy
```

**Network Administrators**:
```
Allow group NetworkAdmins to manage virtual-network-family in tenancy
Allow group NetworkAdmins to manage load-balancers in tenancy
Allow group NetworkAdmins to manage dns in tenancy
```

**Database Administrators**:
```
Allow group DBAdmins to manage database-family in tenancy
Allow group DBAdmins to manage autonomous-database-family in tenancy
Allow group DBAdmins to inspect vcns in tenancy
```

**Instance Principal Policy** (for instances to access resources):
```
Allow dynamic-group instance-principal-group to read buckets in compartment DataCompartment
Allow dynamic-group instance-principal-group to read objects in compartment DataCompartment
```

#### Resource Types (Families)

**Common Resource Families**:
- `all-resources`: Everything
- `instance-family`: Compute instances, images, boot volumes
- `volume-family`: Block volumes, volume backups
- `object-family`: Object storage, buckets
- `virtual-network-family`: VCNs, subnets, security lists
- `database-family`: DB Systems, backups
- `autonomous-database-family`: Autonomous Databases
- `load-balancers`: Load balancers
- `file-family`: File Storage

**Individual Resource Types**:
```
instances, volumes, buckets, vcns, subnets, security-lists,
route-tables, internet-gateways, nat-gateways, service-gateways,
drgs, load-balancers, databases, autonomous-databases, etc.
```

#### Policy Conditions

**Examples**:
```
# Restrict by time
Allow group Developers to manage instances in compartment Dev
  where request.operation='LaunchInstance'

# Restrict by tag
Allow group ProjectA to manage instances in compartment Development
  where target.resource.tag.Project='ProjectA'

# Restrict by IP
Allow group RemoteWorkers to manage instances in compartment Development
  where request.networkSource.name='corporate-network'
```

#### Network Sources

**Definition**: IP-based access restrictions.

**Creating Network Source**:
```bash
oci iam network-sources create \
  --name "corporate-network" \
  --description "Corporate office IP ranges" \
  --public-source-list '["203.0.113.0/24","198.51.100.0/24"]'
```

**Using in Policy**:
```
Allow group RemoteUsers to manage instances in compartment Development
  where request.networkSource.name='corporate-network'
```

#### Policy Locations

**Tenancy-Level Policies**:
- Stored in root compartment
- Can control access across entire tenancy
- Required for cross-compartment access

**Compartment-Level Policies**:
- Stored in specific compartment
- Can only control access within that compartment and below
- Policies inherit from parent compartments

**Best Practice**: Write policies at compartment level when possible for better organization.

### Dynamic Groups

**Definition**: Groups of OCI resources (not users) that match specified rules.

**Purpose**:
- Allow instances to access other resources
- No credentials stored on instances
- Automatic membership based on rules

**Creating Dynamic Groups**:
```bash
oci iam dynamic-group create \
  --name "web-servers" \
  --description "All web server instances" \
  --matching-rule "instance.compartment.id = 'ocid1.compartment.oc1...'"
```

**Matching Rule Examples**:

**All instances in compartment**:
```
instance.compartment.id = 'ocid1.compartment.oc1...'
```

**Specific instance**:
```
instance.id = 'ocid1.instance.oc1.phx...'
```

**Any of multiple conditions**:
```
Any {instance.compartment.id = 'ocid1.compartment.oc1...',
     instance.compartment.id = 'ocid1.compartment.oc1...'}
```

**All conditions**:
```
All {instance.compartment.id = 'ocid1.compartment.oc1...',
     resource.type = 'fnfunc'}
```

**Use Case Example**:
```
# Dynamic group for web servers
Matching Rule: instance.compartment.id = 'ocid1.compartment.oc1...'

# Policy allowing web servers to access Object Storage
Allow dynamic-group web-servers to read objects in compartment data-compartment
```

### Federation

**Definition**: Integration with external identity providers (IdPs).

#### Federation Benefits

- **Centralized Identity**: Single identity source
- **SSO**: Single Sign-On experience
- **No duplicate accounts**: Users managed in IdP
- **Enterprise compliance**: Meet corporate identity policies

#### Supported Protocols

**1. SAML 2.0**:
- Standard protocol
- Most common
- Works with: Azure AD, Okta, ADFS, etc.

**2. Oracle Identity Cloud Service (IDCS)**:
- Oracle's identity platform
- Native integration with OCI

**3. Microsoft Azure AD**:
- Direct integration
- Simplified setup

#### Federation Setup (SAML)

**High-Level Steps**:
1. Export OCI metadata XML
2. Configure IdP with OCI metadata
3. Export IdP metadata XML
4. Configure OCI with IdP metadata
5. Map IdP groups to OCI groups
6. Test federation

**Creating Identity Provider**:
```bash
oci iam identity-provider create-saml2-identity-provider \
  --compartment-id <tenancy-ocid> \
  --name "corporate-idp" \
  --description "Corporate SAML Identity Provider" \
  --metadata-file file://idp-metadata.xml \
  --freeform-tags '{"Environment":"Production"}'
```

#### Group Mapping

**IdP Group → OCI Group Mapping**:
```
IdP Group "Engineering" → OCI Group "Developers"
IdP Group "IT-Operations" → OCI Group "Administrators"
IdP Group "Finance-Team" → OCI Group "FinanceReadOnly"
```

**Creating Group Mapping**:
```bash
oci iam idp-group-mapping create \
  --identity-provider-id ocid1.saml2idp.oc1... \
  --idp-group-name "Engineering" \
  --group-id ocid1.group.oc1...
```

---

## Security Services

### Cloud Guard

**Definition**: Cloud-native security monitoring and threat detection service.

#### Key Features

**1. Security Monitoring**:
- Continuous monitoring of resources
- Detects misconfigurations
- Identifies security risks
- Provides recommendations

**2. Detector Recipes**:
- Pre-configured detection rules
- Custom rules supported
- Activity, configuration, and threat detectors

**3. Responder Recipes**:
- Automated remediation actions
- Notification
- Corrective actions

**4. Target Resources**:
- Monitor entire tenancy or specific compartments
- Hierarchical monitoring

#### Cloud Guard Components

**1. Detector Recipes**:
- **Configuration Detector**: Finds misconfigurations
- **Activity Detector**: Detects suspicious activities
- **Threat Detector**: Identifies known threats

**2. Responder Recipes**:
- **Automatic**: Fix issues automatically
- **Manual**: Notify for manual resolution

**3. Targets**:
- Compartments to monitor
- Apply detector and responder recipes

**4. Problems**:
- Security issues identified
- Risk level (Critical, High, Medium, Low)
- Status (Open, Dismissed, Resolved)

#### Enabling Cloud Guard

**Console Steps**:
1. Navigate to Security → Cloud Guard
2. Click "Enable Cloud Guard"
3. Select reporting region
4. Choose target compartment
5. Select detector and responder recipes
6. Click "Enable"

**CLI**:
```bash
oci cloud-guard cloud-guard-configuration update \
  --reporting-region "us-phoenix-1" \
  --status ENABLED
```

#### Common Detectors

**Configuration Detectors**:
- Bucket is public
- Security list allows all traffic
- Instance has public IP in private subnet
- Database backup not configured

**Activity Detectors**:
- Unusual API activity
- Unauthorized API calls
- User created outside normal hours
- Policy changes by non-admin

**Threat Detectors**:
- Known malicious IPs
- Suspicious network traffic patterns
- Potential data exfiltration

#### Responders

**Automatic Actions**:
- Disable public bucket access
- Revoke problematic IAM policy
- Send notification

**Notification-Only**:
- Alert security team
- Create incident ticket
- Log for audit

### Security Zones

**Definition**: Enforce security best practices by preventing insecure configurations.

#### Characteristics

**1. Prescriptive Security**:
- Prevents violations (not just detects)
- Enforcement at resource creation
- Cannot create non-compliant resources

**2. Security Zones Recipes**:
- Pre-defined security policies
- Oracle-managed recipes
- Maximum security recipe available

**3. Scope**:
- Applied to compartments
- All resources in compartment must comply

#### Security Zones Policies

**Common Policies**:
- **Encryption**: Resources must use encryption
- **Public Access**: Buckets cannot be public
- **Network**: Security lists must have restrictive rules
- **Auditing**: Audit logging must be enabled
- **Backups**: Databases must have backup configured

**Example Violations Prevented**:
- Creating public Object Storage bucket
- Creating instance without encryption
- Disabling audit logs
- Creating overly permissive security lists

#### Creating Security Zone

**Console**:
1. Navigate to Security → Security Zones
2. Click "Create Security Zone"
3. Name and compartment
4. Select security zone recipe
5. Choose compartments to protect
6. Click "Create"

**CLI**:
```bash
oci cloud-guard security-zone create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "production-security-zone" \
  --security-zone-recipe-id ocid1.securityzonerecipe.oc1...
```

**Moving Compartments to Security Zone**:
```bash
oci cloud-guard security-zone update \
  --security-zone-id ocid1.securityzone.oc1... \
  --compartment-id ocid1.compartment.oc1...
```

#### Security Zones Best Practices

- **Production Workloads**: Always use security zones
- **Development**: Consider relaxed zones for flexibility
- **Testing**: Test deployments in non-security-zone first
- **Gradual Adoption**: Start with specific compartments

### Vault

**Definition**: Centralized key management and secrets storage service.

#### Key Features

**1. Key Management**:
- Create, import, rotate encryption keys
- Software-protected and HSM-protected keys
- FIPS 140-2 Level 3 compliance (HSM)

**2. Secrets Management**:
- Store passwords, tokens, certificates
- Versioning and rotation
- Encrypted storage

**3. Encryption Integration**:
- Block Volume encryption
- Object Storage encryption
- Database encryption
- File Storage encryption

#### Vault Components

**1. Vault**:
- Container for keys and secrets
- Regional resource
- Virtual or HSM-backed

**2. Master Encryption Key**:
- Encrypts data encryption keys
- Cannot be exported
- Rotation supported

**3. Secret**:
- Sensitive data (passwords, tokens)
- Base64-encoded
- Versioned

#### Vault Types

**1. Virtual Private Vault**:
- Software-protected keys
- Shared HSM infrastructure
- Lower cost
- FIPS 140-2 Level 3 compliant

**2. HSM Vault** (not in Foundations scope):
- Dedicated HSM partition
- Higher security
- Higher cost

#### Creating Vault

**Console**:
1. Navigate to Security → Vault
2. Click "Create Vault"
3. Name and compartment
4. Select vault type
5. Click "Create"

**CLI**:
```bash
oci kms management vault create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "production-vault" \
  --vault-type VIRTUAL_PRIVATE
```

#### Creating Master Encryption Key

```bash
oci kms management key create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "block-volume-key" \
  --key-shape '{"algorithm":"AES","length":32}' \
  --management-endpoint "https://management.vault-endpoint..."
```

**Key Shapes**:
- **AES**: 16, 24, or 32 bytes (128, 192, 256 bits)
- **RSA**: 256, 384, 512 bytes (2048, 3072, 4096 bits)
- **ECDSA**: P-256, P-384, P-521 curves

#### Using Vault Keys

**Block Volume with Customer-Managed Key**:
```bash
oci bv volume create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --kms-key-id ocid1.key.oc1... \
  --size-in-gbs 1024
```

**Object Storage Bucket with Customer-Managed Key**:
```bash
oci os bucket create \
  --name "encrypted-bucket" \
  --compartment-id ocid1.compartment.oc1... \
  --kms-key-id ocid1.key.oc1...
```

#### Secrets

**Creating Secret**:
```bash
oci vault secret create-base64 \
  --compartment-id ocid1.compartment.oc1... \
  --secret-name "database-password" \
  --vault-id ocid1.vault.oc1... \
  --key-id ocid1.key.oc1... \
  --secret-content-content "V2VsY29tZTEyMyE="
```

**Retrieving Secret**:
```bash
oci secrets secret-bundle get \
  --secret-id ocid1.vaultsecret.oc1... \
  --stage CURRENT
```

#### Key Rotation

**Manual Rotation**:
```bash
oci kms management key create-key-version \
  --key-id ocid1.key.oc1... \
  --management-endpoint "https://management.vault-endpoint..."
```

**Automatic Rotation**:
- Configure rotation schedule
- Automatic version creation
- Seamless transition

### WAF and DDoS Protection

#### Web Application Firewall (WAF)

**Definition**: Protects web applications from attacks.

**Features**:
- OWASP Top 10 protection
- Custom rules
- Bot management
- Access control
- Rate limiting

**Protection Rules**:
- SQL injection
- Cross-site scripting (XSS)
- Directory traversal
- Protocol violations

#### DDoS Protection

**Built-in Protection**:
- Layer 3/4 DDoS protection (included)
- Automatic detection and mitigation
- No configuration required

**Additional Protection**:
- Load balancer with large shapes
- WAF with rate limiting
- CloudFlare or other third-party services

---

## Observability and Management

### Monitoring

**Definition**: Metrics collection and alerting service.

#### Key Concepts

**1. Metrics**:
- Time-series data about resources
- Automatically collected by OCI
- Custom metrics supported
- 1-minute granularity (default: 5 minutes)

**2. Namespaces**:
- Logical containers for metrics
- Examples: `oci_computeagent`, `oci_blockstore`, `oci_lbaas`

**3. Dimensions**:
- Metadata tags for metrics
- Example: `availabilityDomain`, `resourceId`

**4. Alarms**:
- Trigger actions when metric crosses threshold
- Send notifications

#### Common Metrics

**Compute Metrics** (`oci_computeagent`):
- `CpuUtilization`: CPU usage percentage
- `MemoryUtilization`: Memory usage percentage
- `DiskBytesRead`: Disk read throughput
- `DiskBytesWritten`: Disk write throughput
- `NetworkBytesIn`: Network ingress
- `NetworkBytesOut`: Network egress

**Block Volume Metrics** (`oci_blockstore`):
- `VolumeReadThroughput`: Read bytes/sec
- `VolumeWriteThroughput`: Write bytes/sec
- `VolumeReadOps`: Read IOPS
- `VolumeWriteOps`: Write IOPS

**Load Balancer Metrics** (`oci_lbaas`):
- `ActiveConnections`: Current connections
- `HttpRequests`: Requests per interval
- `HttpResponseTime`: Response time
- `UnhealthyBackendServers`: Unhealthy backends

#### Viewing Metrics

**Console**:
1. Navigate to resource (instance, volume, etc.)
2. Click "Metrics" tab
3. View charts

**CLI**:
```bash
oci monitoring metric-data summarize-metrics-data \
  --compartment-id ocid1.compartment.oc1... \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1m].mean()' \
  --start-time "2024-10-10T00:00:00Z" \
  --end-time "2024-10-10T23:59:59Z"
```

#### Creating Alarms

**Console**:
1. Navigate to Monitoring → Alarm Definitions
2. Click "Create Alarm"
3. Configure:
   - Name
   - Metric namespace and name
   - Threshold
   - Trigger delay
   - Notification topic
4. Click "Save Alarm"

**CLI**:
```bash
oci monitoring alarm create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "High CPU Alarm" \
  --destinations '["ocid1.onstopic.oc1..."]' \
  --is-enabled true \
  --metric-compartment-id ocid1.compartment.oc1... \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1m].mean() > 80' \
  --severity "CRITICAL"
```

**Alarm Statuses**:
- **OK**: Metric below threshold
- **Firing**: Metric above threshold, alarm triggered
- **Reset**: Alarm was firing, now OK

#### Custom Metrics

**Publishing Custom Metrics**:
```bash
oci monitoring metric-data post \
  --metric-data '[{
    "namespace":"custom_namespace",
    "compartmentId":"ocid1.compartment.oc1...",
    "name":"application_errors",
    "dimensions":{"app":"webapp"},
    "datapoints":[{
      "timestamp":"2024-10-10T12:00:00Z",
      "value":5.0
    }]
  }]'
```

### Logging

**Definition**: Centralized log management service.

#### Log Types

**1. Service Logs** (Automatically generated):
- **Audit Logs**: IAM and API activity (enabled by default)
- **VCN Flow Logs**: Network traffic
- **Load Balancer Logs**: Access and error logs
- **Object Storage Logs**: Bucket access
- **API Gateway Logs**: API requests

**2. Custom Logs**:
- Application logs
- Custom scripts
- Third-party applications

#### Log Components

**1. Log Groups**:
- Logical container for logs
- Organize related logs

**2. Logs**:
- Individual log streams
- Associated with resource

**3. Log Sources**:
- Resource generating logs
- Service or custom source

#### Enabling Logs

**VCN Flow Logs**:
```bash
oci logging log create \
  --log-group-id ocid1.loggroup.oc1... \
  --display-name "vcn-flow-logs" \
  --log-type SERVICE \
  --configuration '{
    "source": {
      "sourceType":"OCISERVICE",
      "service":"flowlogs",
      "resource":"ocid1.subnet.oc1...",
      "category":"all"
    }
  }'
```

**Load Balancer Access Logs**:
```bash
oci logging log create \
  --log-group-id ocid1.loggroup.oc1... \
  --display-name "lb-access-logs" \
  --log-type SERVICE \
  --configuration '{
    "source": {
      "sourceType":"OCISERVICE",
      "service":"loadbalancer",
      "resource":"ocid1.loadbalancer.oc1...",
      "category":"access"
    }
  }'
```

#### Searching Logs

**Console**:
1. Navigate to Logging → Log Search
2. Enter search query
3. Select time range
4. View results

**Search Query Examples**:
```
# Find errors
search "compartment-ocid" | where type='error'

# Find specific IP
search "compartment-ocid" | where data.source='203.0.113.5'

# Time range
search "compartment-ocid" |
  where time >= '2024-10-10T00:00:00'
  and time <= '2024-10-10T23:59:59'
```

#### Audit Logs

**Definition**: Automatically enabled logs of all API calls.

**Information Captured**:
- **Who**: User or principal making request
- **What**: API operation performed
- **When**: Timestamp
- **Where**: Resource and compartment
- **Status**: Success or failure
- **Source IP**: Request origin

**Audit Log Retention**:
- 90 days in Logging service (free)
- Archive to Object Storage for longer retention

**Viewing Audit Logs**:
```bash
oci audit event list \
  --compartment-id ocid1.compartment.oc1... \
  --start-time "2024-10-10T00:00:00Z" \
  --end-time "2024-10-10T23:59:59Z"
```

### Notifications

**Definition**: Pub/sub messaging service for alerts.

#### Components

**1. Topics**:
- Communication channel
- Subscribers receive messages

**2. Subscriptions**:
- Endpoint to receive notifications
- Email, SMS, PagerDuty, Slack, HTTPS

**3. Messages**:
- Notification content
- Published to topic

#### Creating Topic

```bash
oci ons topic create \
  --compartment-id ocid1.compartment.oc1... \
  --name "alert-topic" \
  --description "Critical alerts"
```

#### Creating Subscription

**Email Subscription**:
```bash
oci ons subscription create \
  --compartment-id ocid1.compartment.oc1... \
  --topic-id ocid1.onstopic.oc1... \
  --protocol EMAIL \
  --subscription-endpoint "admin@example.com"
```

**Subscription Types**:
- **EMAIL**: Email notifications
- **HTTPS**: Webhook POST
- **PAGERDUTY**: PagerDuty integration
- **SLACK**: Slack channel
- **SMS**: Text messages (limited regions)

#### Publishing Messages

```bash
oci ons message publish \
  --topic-id ocid1.onstopic.oc1... \
  --body "High CPU alert on instance-01" \
  --title "Critical Alert"
```

#### Integration with Other Services

**Alarms → Notifications**:
- Alarm fires → publishes to topic
- Subscribers receive alert

**Events → Notifications**:
- Event triggers → publishes to topic
- Subscribers notified

### Events

**Definition**: Service that triggers actions based on state changes.

#### Event Types

**Resource Lifecycle Events**:
- Instance created/terminated
- Volume attached/detached
- Database started/stopped

**Service Events**:
- Backup completed
- Scaling action triggered
- Maintenance scheduled

#### Event Rules

**Components**:
- **Event Type**: What triggers the rule
- **Conditions**: Filter events
- **Actions**: What happens (notification, function, streaming)

**Creating Event Rule**:
```bash
oci events rule create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "instance-creation-alert" \
  --is-enabled true \
  --condition '{
    "eventType":"com.oraclecloud.computeapi.launchinstance.end"
  }' \
  --actions '{
    "actions":[{
      "actionType":"ONS",
      "isEnabled":true,
      "topicId":"ocid1.onstopic.oc1..."
    }]
  }'
```

#### Common Event Patterns

**Instance State Changes**:
```json
{
  "eventType": [
    "com.oraclecloud.computeapi.launchinstance.end",
    "com.oraclecloud.computeapi.terminateinstance.end",
    "com.oraclecloud.computeapi.stopinstance.end"
  ]
}
```

**Database Events**:
```json
{
  "eventType": [
    "com.oraclecloud.databaseservice.autonomous.database.backup.end",
    "com.oraclecloud.databaseservice.database.instance.down"
  ]
}
```

#### Event Actions

**1. Notifications**:
- Send to ONS topic
- Alert teams

**2. Functions**:
- Trigger serverless function
- Automated remediation

**3. Streaming**:
- Send to Stream
- Event processing pipeline

---

## Cost Management and Pricing

### Pricing Models

#### Pay As You Go (PAYG)

**Characteristics**:
- No upfront commitment
- Per-second billing (most services)
- Pay only for what you use
- Can cancel anytime

**Billing Units**:
- **Compute**: OCPU-hours or OCPU-seconds
- **Storage**: GB-months
- **Networking**: GB transferred
- **Database**: OCPU-hours + storage

**Example Pricing** (varies by region):
- VM.Standard.E4.Flex: ~$0.015 per OCPU-hour
- Block Volume (Balanced): ~$0.085 per GB-month
- Object Storage (Standard): ~$0.026 per GB-month
- Data Transfer Out: $0.00 (no charge in OCI)

#### Monthly Flex

**Characteristics**:
- Commit to monthly spend ($1,000+ minimum)
- Discount applied (varies by commitment)
- Universal Credits (use for any service)
- Monthly billing

**Benefits**:
- Lower unit prices than PAYG
- Flexibility to use across services
- Predictable monthly cost

#### Annual Flex

**Characteristics**:
- Commit to annual spend
- Deeper discounts than Monthly Flex
- Universal Credits valid for 1 year
- Annual billing

**Benefits**:
- Lowest unit prices
- Maximum savings
- Budget certainty

#### Bring Your Own License (BYOL)

**Characteristics**:
- Use existing Oracle licenses
- Lower compute costs
- License mobility

**Supported Products**:
- Oracle Database
- Oracle Middleware
- Oracle Applications

**Savings**:
- Up to 70% savings on compute
- Only pay for infrastructure

### Cost Analysis

#### Cost and Usage Reports

**Access**:
1. Navigate to Governance → Cost Management
2. View cost analysis charts
3. Download detailed reports

**Report Types**:
- **Cost Overview**: Daily costs by service
- **Usage Reports**: Resource usage details
- **Forecast**: Predicted future costs

#### Cost Tracking Tools

**1. Cost Analysis**:
- View costs by compartment
- Filter by tag, service, region
- Compare time periods

**2. Budgets**:
- Set spending limits per compartment
- Alert when threshold reached
- Prevent overspending

**Creating Budget**:
```bash
oci budgets budget create \
  --compartment-id ocid1.compartment.oc1... \
  --amount 1000 \
  --reset-period MONTHLY \
  --target-compartment-id ocid1.compartment.oc1... \
  --target-type COMPARTMENT \
  --display-name "Development Monthly Budget"
```

**Budget Alerts**:
- Percentage-based: Alert at 50%, 80%, 100%
- Absolute amount: Alert at specific dollar amount
- Forecasted spend: Alert if projected to exceed

**3. Cost Allocation Tags**:
- Tag resources for cost tracking
- Group costs by project, department, environment
- Required for detailed cost analysis

**Tagging Example**:
```bash
oci compute instance update \
  --instance-id ocid1.instance.oc1... \
  --defined-tags '{
    "CostCenter":{"Department":"Engineering","Project":"WebApp"}
  }'
```

#### Cost Optimization Tips

**1. Right-Sizing**:
- Use Flex shapes for exact sizing
- Monitor usage and adjust
- Stop/terminate unused resources

**2. Storage Optimization**:
- Use Auto-Tiering for Object Storage
- Archive old data
- Delete unused volumes

**3. Reserved Capacity**:
- Use Monthly/Annual Flex for predictable workloads
- Commit to steady-state capacity

**4. Resource Scheduling**:
- Stop dev/test instances after hours
- Use autoscaling for variable workloads

### Billing and Usage

#### Invoicing

**Billing Cycle**:
- Monthly invoices
- Pay by credit card or invoice
- Automatic payment (PAYG)
- Invoice payment (Monthly/Annual Flex)

**Invoice Details**:
- Usage by service
- Breakdown by compartment
- Universal Credits usage (Flex)

#### Free Tier

**Always Free Services**:
- 2 AMD VM instances (1/8 OCPU, 1 GB RAM each)
- 2 Block Volumes (100 GB total)
- 10 GB Object Storage
- 10 GB Archive Storage
- Load Balancer (1 instance, 10 Mbps)
- Autonomous Database (2 instances, 0.02 OCPU each)
- Monitoring, Notifications, Outbound Data Transfer

**30-Day Trial**:
- $300 USD free credits
- Valid for 30 days
- Use any OCI service

#### Usage Monitoring

**Real-Time Usage**:
- View current month costs
- Track service usage
- Compartment breakdown

**Historical Usage**:
- Past invoices
- Usage trends
- Year-over-year comparison

---

## Support and SLAs

### Support Tiers

**Basic Support** (Included):
- Community forums
- Documentation
- Service limits
- No direct technical support
- Best effort response

**Premier Support** (Paid):
- 24/7 phone and email support
- Severity-based response times
- Technical Account Manager (TAM) option
- Cloud Advisor
- My Oracle Support (MOS) access

**Support Response Times**:
- **Severity 1** (Critical): 15 minutes
- **Severity 2** (High): 2 hours
- **Severity 3** (Medium): 24 hours
- **Severity 4** (Low): 2 business days

### Service Level Agreements (SLAs)

**Compute SLA**: 99.9%
- Monthly uptime percentage
- Applies to multi-instance deployments

**Block Volume SLA**: 99.9%
- Availability and durability

**Object Storage SLA**: 99.9%
- Availability guarantee

**Autonomous Database SLA**: 99.95%
- Higher than traditional databases
- Can reach 99.99% with specific configurations

**Load Balancer SLA**: 99.95%
- Multi-AD deployment

**VCN/Networking SLA**: 99.9%
- Network connectivity

**SLA Credits**:
- Issued if SLA not met
- Percentage of monthly service fees
- Must be claimed within specified timeframe

---

## Exam Tips

### Critical Concepts

**1. IAM**:
- **Users**: Individual identities
- **Groups**: Collections of users
- **Policies**: Permissions (Allow group to verb resource in location)
- **Dynamic Groups**: Resource-based groups
- **Verbs**: inspect < read < use < manage

**2. Policy Syntax**:
```
Allow group <group-name> to <verb> <resource-type> in <location>
```

**3. Authentication Methods**:
- Console Password (MFA recommended)
- API Signing Keys
- Auth Tokens
- Customer Secret Keys

**4. Security Services**:
- **Cloud Guard**: Detection and response
- **Security Zones**: Prevention (blocks violations)
- **Vault**: Key and secret management

**5. Monitoring**:
- **Metrics**: Time-series data (1-min granularity)
- **Alarms**: Threshold-based alerts
- **Logs**: Service and custom logs
- **Notifications**: Pub/sub messaging

**6. Pricing**:
- **PAYG**: No commitment, per-second billing
- **Monthly Flex**: Monthly commitment, discounts
- **Annual Flex**: Annual commitment, best discounts
- **BYOL**: Use existing licenses

### Common Exam Questions

**Q: What's the difference between Security Lists and NSGs?**
A: Security Lists apply to subnets; NSGs apply to VNICs (more flexible)

**Q: Which IAM verb provides full control?**
A: manage

**Q: What's the default MFA method?**
A: TOTP (Time-based One-Time Password) via authenticator app

**Q: How long are audit logs retained?**
A: 90 days (can archive to Object Storage for longer)

**Q: What's the difference between Cloud Guard and Security Zones?**
A: Cloud Guard detects and responds; Security Zones prevents violations

**Q: Which service provides key management?**
A: Vault (KMS)

**Q: What's the monitoring metric granularity?**
A: 1 minute (default: 5 minutes for some services)

**Q: Which pricing model has no commitment?**
A: Pay As You Go (PAYG)

**Q: What's included in Basic Support?**
A: Documentation, forums, no direct technical support

**Q: What's the Autonomous Database SLA?**
A: 99.95% (can be 99.99%)

### Quick Reference

**IAM Policy Structure**:
```
Subject: group, dynamic-group, any-user
Verb: inspect, read, use, manage
Resource: instance-family, volume-family, etc.
Location: tenancy, compartment
```

**Security Service Comparison**:
| Service | Purpose | Type |
|---------|---------|------|
| Cloud Guard | Detect threats | Detective |
| Security Zones | Prevent violations | Preventive |
| Vault | Manage keys | Encryption |
| WAF | Protect web apps | Protective |

**Observability Services**:
- **Monitoring**: Metrics and alarms
- **Logging**: Log collection and search
- **Notifications**: Alert delivery
- **Events**: State change triggers

**Support Tiers**:
- **Basic**: Included, no direct support
- **Premier**: Paid, 24/7 support

---

## Summary

### Key Takeaways

**IAM**:
- Users, groups, policies control access
- Policies use: Allow <subject> to <verb> <resource> in <location>
- Dynamic groups for resource principals
- Federation for SSO and centralized identity

**Security**:
- Cloud Guard: Monitors and responds to threats
- Security Zones: Prevents insecure configurations
- Vault: Manages encryption keys and secrets
- Audit logs: Track all API activity (90 days)

**Observability**:
- Monitoring: Metrics, alarms, dashboards
- Logging: Service logs, custom logs, audit logs
- Notifications: Alert delivery (email, SMS, webhooks)
- Events: Automated actions on state changes

**Cost Management**:
- PAYG: Pay per second, no commitment
- Flex: Committed spend, discounts, Universal Credits
- BYOL: Use existing licenses, save on compute
- Budgets: Set limits, get alerts

**Support**:
- Basic: Included, community support
- Premier: Paid, 24/7 technical support
- SLAs: 99.9%-99.95% for most services

---

**Congratulations!** You've completed the OCI Foundations study notes. Review these materials, practice hands-on labs, and take practice exams to prepare for certification success!
