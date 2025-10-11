# Security and Compliance - GCP Professional Cloud Architect

## Overview

Security and compliance architecture for Google Cloud, covering IAM, network security, data protection, compliance frameworks, and security best practices for the Professional Cloud Architect certification.

This comprehensive guide covers security architecture patterns, defense-in-depth strategies, compliance frameworks, and real-world security scenarios essential for the Professional Cloud Architect exam.

## Key Exam Topics

1. **Identity and Access Management** - Authentication, authorization, service accounts, organization policies, workload identity
2. **Network Security** - VPC security, Cloud Armor, Cloud IDS, Private Service Connect, firewall hierarchies
3. **Data Protection** - Encryption (CMEK, CSEK), DLP, Secret Manager, key management
4. **Compliance** - PCI-DSS, HIPAA, SOC 2, FedRAMP, ISO 27001, GDPR
5. **Security Operations** - Security Command Center, monitoring, incident response, threat detection
6. **Zero Trust Architecture** - BeyondCorp, IAP, context-aware access
7. **VPC Service Controls** - API perimeter security, service perimeters, ingress/egress policies

## Exam Tips - Security Focus

### What to Expect
- **Scenario-based questions**: Multi-tier applications with specific compliance requirements
- **Defense-in-depth**: Questions requiring multiple security layers
- **Compliance mapping**: Matching GCP services to regulatory requirements
- **Security tradeoffs**: Performance vs security, cost vs compliance
- **Incident response**: Detecting and responding to security events

### Common Question Patterns
1. **"Most secure" solution**: Usually involves VPC Service Controls + CMEK + Private connectivity
2. **Compliance requirements**: Know which services are HIPAA/PCI-DSS eligible
3. **Service account security**: Prefer Workload Identity over keys
4. **Network isolation**: VPC Service Controls vs Private Service Connect vs Shared VPC
5. **Data protection**: When to use CMEK vs CSEK vs default encryption

### Key Decision Factors
- **HIPAA/PCI-DSS**: Requires BAA, specific eligible services, CMEK often required
- **Zero trust**: IAP + BeyondCorp + context-aware access
- **API security**: VPC Service Controls for perimeter protection
- **Secrets**: Always use Secret Manager, never environment variables or code
- **Audit requirements**: Enable all audit log types, export to immutable storage

## IAM Architecture

### Resource Hierarchy and Policy Inheritance

```
Organization
├── Folder (Business Unit)
│   ├── Folder (Environment - Production)
│   │   ├── Project (App-Prod)
│   │   └── Project (DB-Prod)
│   └── Folder (Environment - Dev)
│       ├── Project (App-Dev)
│       └── Project (DB-Dev)
```

**Policy Inheritance Rules**:
- Policies are inherited from parent to child
- Cannot restrict a permission granted at a higher level
- Use organization policies to enforce restrictions across hierarchy
- Effective policy is the union of all policies in the hierarchy
- Deny policies can override Allow policies

### IAM Best Practices

**Access Management**:
- Use groups instead of individual users (centralized management)
- Apply least privilege principle (minimum necessary permissions)
- Use predefined roles when possible (Google-maintained, secure defaults)
- Create custom roles for specific needs (fine-grained control)
- Implement organization policies (enforce security boundaries)
- Conduct regular access reviews (quarterly minimum)
- Use service accounts for applications (never use user credentials)
- Enable Cloud Audit Logs (all categories for security monitoring)

**Role Design**:
- Predefined roles: Use for common patterns (roles/compute.admin)
- Custom roles: Create for specific requirements with minimal permissions
- Basic roles (Owner, Editor, Viewer): Avoid in production (too broad)
- Service-specific roles: Most granular option (roles/storage.objectViewer)

**IAM Conditions**:
```yaml
# Time-based access
resource.type == "storage.googleapis.com/Bucket" &&
request.time < timestamp("2025-12-31T00:00:00Z")

# Resource-based access
resource.name.startsWith("projects/_/buckets/prod-")

# IP-based access
origin.ip in ["192.168.1.0/24", "10.0.0.0/8"]

# Attribute-based access
resource.labels.env == "production" &&
request.auth.claims.department == "finance"
```

### Service Account Architecture

**Identity Patterns**:

1. **Workload Identity (GKE) - PREFERRED**:
```yaml
# Kubernetes Service Account bound to GCP Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  annotations:
    iam.gke.io/gcp-service-account: app-sa@project.iam.gserviceaccount.com
```

Benefits:
- No service account keys to manage
- Automatic credential rotation
- Kubernetes-native identity
- Fine-grained pod-level permissions

2. **Workload Identity Federation (External)**:
- For AWS, Azure, or on-premises workloads
- OIDC or SAML-based authentication
- No long-lived credentials
- Attribute mapping for authorization

3. **Service Account Impersonation**:
```bash
# User impersonates service account for specific operation
gcloud compute instances create vm-name \
  --impersonate-service-account=sa@project.iam.gserviceaccount.com
```

**Service Account Patterns**:

**Application Identity**:
- Use Workload Identity for GKE (no keys)
- Use default service account for Compute Engine
- Use Application Default Credentials (ADC)

**Cross-Project Access**:
- Create service account in service project
- Grant permissions in host project
- Use service account impersonation for user actions

**Automation**:
- Dedicated service accounts for CI/CD
- Short-lived tokens via Token Service
- Audit all service account usage

**Key Management**:
- Avoid creating keys when possible
- Rotate keys every 90 days maximum
- Use Cloud KMS to encrypt keys at rest
- Monitor key usage via audit logs
- Implement key creation restrictions

### Service Account Security Controls

**Restrictions via Organization Policy**:
```yaml
# Disable service account key creation
constraints/iam.disableServiceAccountKeyCreation

# Restrict service account creation
constraints/iam.disableServiceAccountCreation

# Require external IDP for authentication
constraints/iam.allowedPolicyMemberDomains
```

**Best Practices**:
1. One service account per application/service
2. Use meaningful names (app-name-env-function)
3. Minimal permissions per principle of least privilege
4. Enable detailed audit logging for service accounts
5. Implement service account key rotation policies
6. Use Workload Identity whenever possible
7. Separate dev/test/prod service accounts
8. Document service account purpose and permissions

### Organization Policies

**Boolean Constraints**:
```yaml
# Restrict VM external IP addresses
constraints/compute.vmExternalIpAccess: false

# Require OS Login
constraints/compute.requireOsLogin: true

# Disable service account key creation
constraints/iam.disableServiceAccountKeyCreation: true

# Disable automatic IAM grants for default service accounts
constraints/iam.automaticIamGrantsForDefaultServiceAccounts: false

# Require Shielded VMs
constraints/compute.requireShieldedVm: true
```

**List Constraints**:
```yaml
# Define allowed resource locations
constraints/gcp.resourceLocations:
  allowed_values:
    - in:us-locations
    - in:eu-locations

# Restrict VM machine types
constraints/compute.vmMachineTypes:
  allowed_values:
    - e2-*
    - n2-standard-*

# Allowed IAM policy member domains
constraints/iam.allowedPolicyMemberDomains:
  allowed_values:
    - C0abcdefg  # Customer ID
```

**Advanced Constraints**:
- Enforce uniform bucket-level access
- Restrict VPC peering
- Define allowed VPC Service Controls perimeters
- Restrict public IP on Cloud SQL
- Require CMEK for specific services

### IAM Conditions - Advanced Patterns

**Time-based Access**:
```python
# Temporary contractor access
(request.time >= timestamp("2025-01-01T00:00:00Z") &&
 request.time < timestamp("2025-06-30T23:59:59Z"))
```

**Resource Attribute Access**:
```python
# Access only production resources
resource.labels.env == "production"

# Access only specific regions
resource.location in ["us-central1", "us-east1"]
```

**Request Context Access**:
```python
# Access from corporate network only
origin.ip in ["203.0.113.0/24"]

# Access during business hours
request.time.getHours("America/New_York") >= 9 &&
request.time.getHours("America/New_York") <= 17
```

**Combined Conditions**:
```python
# Production access only during business hours from corporate network
(resource.labels.env == "production") &&
(request.time.getHours("America/New_York") >= 9 &&
 request.time.getHours("America/New_York") <= 17) &&
(origin.ip in ["203.0.113.0/24"])
```

### Break-Glass Access Procedures

**Emergency Access Pattern**:
1. Dedicated break-glass accounts with Owner role
2. Stored credentials in physical safe
3. Monitored via real-time alerting
4. Automatic ticket creation on use
5. Required post-access review

**Implementation**:
```yaml
# Break-glass group with monitored access
members:
  - breakglass-1@company.com
  - breakglass-2@company.com
roles:
  - roles/owner
conditions:
  # Alert on any use
  log_filter: 'protoPayload.authenticationInfo.principalEmail=~"breakglass"'
```

## Network Security

### Defense in Depth Layers

```
Layer 7: Application Security
├── Cloud Armor (WAF, DDoS)
├── Identity-Aware Proxy (IAP)
└── Binary Authorization

Layer 4-6: Network Security
├── Firewall Rules (Hierarchical)
├── Cloud IDS (Intrusion Detection)
└── Private Service Connect

Layer 3: Network Segmentation
├── VPC Service Controls (API perimeter)
├── Shared VPC (centralized management)
└── VPC Peering (isolated connectivity)

Layer 2: Private Connectivity
├── Private Google Access
├── Private Service Connect
└── Cloud VPN/Interconnect

Layer 1: Data Protection
├── Encryption in transit (TLS)
├── Encryption at rest (CMEK)
└── DLP (data classification)
```

**Defense Strategy**:
1. **Perimeter Security**: Cloud Armor, DDoS protection, external threat mitigation
2. **Network Segmentation**: VPCs, subnets, firewall rules, micro-segmentation
3. **Private Connectivity**: Private Google Access, VPC Service Controls, no internet exposure
4. **Application Security**: IAP, Binary Authorization, authenticated access only
5. **Data Protection**: Encryption, DLP, sensitive data controls

### VPC Service Controls - API Perimeter Security

**Service Perimeter Architecture**:
```
Service Perimeter (PCI Environment)
├── Projects
│   ├── payment-processing-prod
│   ├── payment-processing-dev
│   └── cardholder-data-storage
├── Protected Services
│   ├── storage.googleapis.com
│   ├── bigquery.googleapis.com
│   ├── compute.googleapis.com
│   └── cloudkms.googleapis.com
├── Access Levels
│   ├── Corporate Network (IP ranges)
│   ├── Device Policy (managed devices)
│   └── Identity (specific users/groups)
└── Ingress/Egress Policies
    ├── Allow from trusted projects
    └── Deny all other access
```

**Key Concepts**:
- **Service Perimeter**: Logical boundary around GCP resources
- **Access Levels**: Conditions for allowed access (IP, device, identity)
- **Ingress Rules**: Control incoming requests to perimeter
- **Egress Rules**: Control outgoing requests from perimeter
- **VPC Accessible Services**: Limit which APIs can be accessed

**Access Level Examples**:
```yaml
# Corporate network access
access_level:
  name: "corp_network"
  conditions:
    - ip_subnetworks:
        - "203.0.113.0/24"
        - "198.51.100.0/24"
    - regions:
        - "US"

# Managed device access
access_level:
  name: "managed_devices"
  conditions:
    - device_policy:
        require_screen_lock: true
        require_admin_approval: true
        os_constraints:
          - os_type: DESKTOP_CHROME_OS
            minimum_version: "85.0.0"

# Combined access level
access_level:
  name: "secure_access"
  combining_function: AND
  conditions:
    - access_level: "corp_network"
    - access_level: "managed_devices"
```

**Ingress/Egress Policies**:
```yaml
# Allow ingress from specific service account
ingress_policy:
  ingress_from:
    sources:
      - resource: "projects/trusted-project"
    identities:
      - "serviceAccount:trusted-sa@project.iam.gserviceaccount.com"
  ingress_to:
    operations:
      - service_name: "storage.googleapis.com"
        method_selectors:
          - method: "google.storage.objects.get"

# Allow egress to external API
egress_policy:
  egress_from:
    identities:
      - "serviceAccount:app@project.iam.gserviceaccount.com"
  egress_to:
    resources:
      - "projects/external-project"
    operations:
      - service_name: "bigquery.googleapis.com"
```

**VPC Service Controls Best Practices**:
1. Start with dry run mode to test policies
2. Use multiple access levels for defense in depth
3. Implement separate perimeters for different compliance zones
4. Monitor VPC-SC violations in Cloud Logging
5. Use bridges for controlled cross-perimeter communication
6. Document all ingress/egress exceptions
7. Regular review of access levels and perimeter configuration

### Firewall Rules Hierarchy

**Rule Precedence** (highest to lowest):
1. **Organization-level** firewall policies (global)
2. **Folder-level** firewall policies (inherited)
3. **VPC-level** firewall rules (project-specific)
4. **Implied rules** (allow egress, deny ingress)

**Hierarchical Firewall Policy Architecture**:
```
Organization Policy
├── Priority 1000: Deny all SSH from internet
├── Priority 2000: Allow HTTPS from anywhere
└── Priority 3000: Allow internal communication

Folder Policy (Production)
├── Priority 1000: Allow SSH from bastion CIDR
├── Priority 2000: Allow database ports from app tier
└── Priority 3000: Deny all other traffic

VPC Firewall Rules
├── Priority 1000: Allow health checks
├── Priority 2000: Allow load balancer traffic
└── Priority 65535: Deny all (explicit)
```

**Firewall Rule Best Practices**:
```yaml
# Use service accounts for dynamic rules
allow-app-to-db:
  priority: 1000
  direction: INGRESS
  source_service_accounts:
    - app-tier-sa@project.iam.gserviceaccount.com
  target_service_accounts:
    - db-tier-sa@project.iam.gserviceaccount.com
  allowed:
    - protocol: tcp
      ports: ["3306"]

# Use tags for grouping
allow-web-traffic:
  priority: 1000
  direction: INGRESS
  source_ranges: ["0.0.0.0/0"]
  target_tags: ["web-server"]
  allowed:
    - protocol: tcp
      ports: ["80", "443"]

# Logging for security monitoring
allow-ssh-logging:
  priority: 1000
  direction: INGRESS
  source_ranges: ["10.0.0.0/8"]
  allowed:
    - protocol: tcp
      ports: ["22"]
  log_config:
    enable: true
    metadata: INCLUDE_ALL_METADATA
```

**Key Design Principles**:
1. Use hierarchical policies for organization-wide rules
2. Service account-based rules for dynamic environments
3. Explicit deny rules at the end of each policy
4. Enable logging on critical rules
5. Use priority gaps (100s) for future insertions
6. Document rule purpose and owner
7. Regular audit of unused rules

### Private Google Access and Private Service Connect

**Private Google Access**:
- Allows VMs without external IPs to access Google APIs
- Enabled per subnet
- Uses internal IP routing to Google services
- Supports Cloud Storage, BigQuery, other Google APIs

**Configuration**:
```bash
# Enable Private Google Access on subnet
gcloud compute networks subnets update SUBNET_NAME \
  --region=REGION \
  --enable-private-ip-google-access
```

**Private Service Connect (PSC)**:
- Consume Google services via internal IP addresses
- Publish services to consumers securely
- No internet exposure required
- Works across VPCs and organizations

**PSC Patterns**:

1. **Consume Google APIs**:
```bash
# Create PSC endpoint for Cloud Storage
gcloud compute addresses create psc-storage-endpoint \
  --global \
  --purpose=PRIVATE_SERVICE_CONNECT \
  --network=VPC_NAME \
  --addresses=10.0.0.100

gcloud dns managed-zones create psc-zone \
  --dns-name=storage.googleapis.com \
  --networks=VPC_NAME \
  --visibility=private
```

2. **Publish Internal Services**:
```bash
# Publish service via PSC
gcloud compute service-attachments create SERVICE_NAME \
  --region=REGION \
  --producer-forwarding-rule=FORWARDING_RULE \
  --connection-preference=ACCEPT_AUTOMATIC \
  --nat-subnets=NAT_SUBNET
```

**Use Cases**:
- **Private Google Access**: VMs accessing Google APIs without external IP
- **PSC for Google APIs**: Complete isolation, custom DNS, dedicated endpoints
- **PSC for Services**: Share services across VPCs/orgs without VPC peering
- **PSC for SaaS**: Consume third-party services via private endpoints

### Cloud Armor - Web Application Firewall

**Architecture**:
```
Internet → Cloud Armor → Global Load Balancer → Backend Services
           ↓
         WAF Rules
         Rate Limiting
         Geo Restrictions
         Bot Management
```

**Security Policy Levels**:
1. **Pre-configured Rules**: OWASP Top 10, ModSecurity CRS
2. **Custom Rules**: Request attributes, headers, geolocation
3. **Rate Limiting**: Per-client IP, per-session
4. **Adaptive Protection**: ML-based DDoS mitigation
5. **Bot Management**: reCAPTCHA integration

**Rule Examples**:
```yaml
# Block SQL injection attempts
rule:
  priority: 1000
  action: deny(403)
  match:
    expr:
      expression: "evaluatePreconfiguredExpr('sqli-stable')"
  description: "Block SQL injection"

# Geographic restriction
rule:
  priority: 2000
  action: deny(403)
  match:
    expr:
      expression: "origin.region_code == 'CN' || origin.region_code == 'RU'"
  description: "Block specific countries"

# Rate limiting
rule:
  priority: 3000
  action: rate_based_ban
  rate_limit_options:
    conform_action: allow
    exceed_action: deny(429)
    enforce_on_key: IP
    rate_limit_threshold:
      count: 100
      interval_sec: 60
  description: "100 requests per minute per IP"

# Custom request header blocking
rule:
  priority: 4000
  action: deny(403)
  match:
    expr:
      expression: "has(request.headers['x-internal-only'])"
  description: "Block requests with internal header"

# Allow only specific user agents
rule:
  priority: 5000
  action: allow
  match:
    expr:
      expression: "request.headers['user-agent'].contains('MyApp/1.0')"
  description: "Allow only official app"
```

**Advanced Features**:

**Adaptive Protection**:
- ML-based anomaly detection
- Automatic signature generation
- Layer 7 DDoS mitigation
- Real-time threat intelligence

**Bot Management**:
```yaml
# reCAPTCHA Enterprise integration
rule:
  priority: 1000
  action: allow
  match:
    expr:
      expression: "token.recaptcha_session.score > 0.5"
  description: "Allow legitimate users"
```

**Logging and Monitoring**:
```yaml
# Enable detailed logging
security_policy:
  name: "production-policy"
  rules: [...]
  log_level: VERBOSE
  json_parsing: STANDARD
  json_custom_config:
    content_types: ["application/json"]
```

### Cloud IDS - Intrusion Detection System

**Architecture**:
```
VPC Network → Packet Mirroring → Cloud IDS → Security Command Center
                                      ↓
                                  Threat Detection
                                  Alert Generation
```

**Capabilities**:
- Network-based threat detection
- Palo Alto Networks technology
- Covers OWASP Top 10, malware, crypto-mining
- Real-time alerts to Security Command Center
- Minimal performance impact

**Configuration**:
```bash
# Create IDS endpoint
gcloud ids endpoints create ENDPOINT_NAME \
  --network=VPC_NAME \
  --zone=ZONE \
  --severity=INFORMATIONAL

# Create packet mirroring policy
gcloud compute packet-mirrorings create POLICY_NAME \
  --region=REGION \
  --network=VPC_NAME \
  --mirrored-subnets=SUBNET_NAME \
  --collector-ilb=ILB_FORWARDING_RULE
```

**Detection Categories**:
- Malware and command-and-control (C2) traffic
- SQL injection and XSS attempts
- Denial of Service (DoS) attacks
- Data exfiltration attempts
- Lateral movement patterns
- Protocol anomalies

**Integration with SCC**:
- Automatic finding creation
- Severity classification
- Contextual threat information
- Remediation recommendations

### Packet Mirroring and Traffic Analysis

**Packet Mirroring Use Cases**:
1. **Security Analysis**: IDS/IPS inspection
2. **Compliance**: Traffic recording for audit
3. **Troubleshooting**: Network debugging
4. **Performance**: Traffic pattern analysis

**Configuration**:
```bash
# Mirror all traffic from subnet
gcloud compute packet-mirrorings create MIRROR_NAME \
  --region=REGION \
  --network=VPC_NAME \
  --mirrored-subnets=SUBNET_1,SUBNET_2 \
  --collector-ilb=COLLECTOR_ILB \
  --filter-direction=BOTH \
  --filter-protocols=tcp,udp \
  --filter-cidr-ranges=0.0.0.0/0
```

**Collector Options**:
- Internal Load Balancer (ILB)
- Cloud IDS endpoint
- Third-party security appliances
- Custom analysis tools

## Data Protection

### Encryption Strategy

**Encryption Options Comparison**:

| Feature | Default | CMEK | CSEK |
|---------|---------|------|------|
| Key Management | Google | Customer via KMS | Customer external |
| Key Rotation | Automatic | Configurable | Manual |
| Audit Logs | Basic | Detailed | Detailed |
| Services Supported | All | Most | Limited |
| Compliance | Standard | Enhanced | Maximum |
| Operational Overhead | None | Low | High |

**Data at Rest Encryption**:

1. **Default Encryption (Google-managed keys)**:
   - Automatic for all data
   - AES-256 encryption
   - No configuration required
   - Keys managed by Google
   - Suitable for most use cases

2. **CMEK (Customer-Managed Encryption Keys)**:
   - Keys stored in Cloud KMS
   - Customer controls key lifecycle
   - Cryptographic access logs
   - Key rotation policies
   - Regional key enforcement
   - Required for most compliance frameworks

3. **CSEK (Customer-Supplied Encryption Keys)**:
   - Customer provides and manages keys
   - Keys never stored in GCP
   - Maximum control and security
   - Limited service support (Compute, Storage)
   - Higher operational complexity

**Data in Transit Encryption**:
- **External**: TLS 1.3 for all public connections
- **Google Front End**: Terminates TLS at edge
- **Internal**: BoringSSL encryption (ALTS protocol)
- **Hybrid**: Cloud VPN or Cloud Interconnect with MACsec
- **Application-level**: App-specific encryption (optional)

**Encryption Architecture**:
```
Application Data
    ↓
[Application-level Encryption] (optional)
    ↓
[GCP Service Encryption]
    ↓
├── CMEK (Cloud KMS)
│   ├── Key Ring (regional)
│   ├── Crypto Keys
│   └── Key Versions
├── CSEK (Customer-provided)
│   └── External key management
└── Default (Google-managed)
    └── Automatic encryption
    ↓
[Storage Infrastructure]
    ↓
Encrypted Data at Rest
```

### Cloud KMS Architecture

**Key Hierarchy**:
```
Key Ring (Regional/Global)
├── Crypto Key (Purpose: Encryption/Decryption)
│   ├── Key Version 1 (Primary)
│   ├── Key Version 2
│   └── Key Version 3 (Destroyed)
├── Crypto Key (Purpose: MAC)
│   └── Key Versions
└── Crypto Key (Purpose: Asymmetric Signing)
    └── Key Versions
```

**Key Ring Organization Patterns**:

1. **By Environment**:
```
projects/my-project/locations/us-central1/keyRings/prod
projects/my-project/locations/us-central1/keyRings/dev
projects/my-project/locations/us-central1/keyRings/test
```

2. **By Application**:
```
projects/my-project/locations/us-central1/keyRings/payment-app
projects/my-project/locations/us-central1/keyRings/user-data-app
projects/my-project/locations/us-central1/keyRings/analytics-app
```

3. **By Data Classification**:
```
projects/my-project/locations/us-central1/keyRings/pii-data
projects/my-project/locations/us-central1/keyRings/financial-data
projects/my-project/locations/us-central1/keyRings/public-data
```

**Key Management Operations**:

```bash
# Create key ring
gcloud kms keyrings create prod-keyring \
  --location=us-central1

# Create encryption key
gcloud kms keys create prod-data-key \
  --keyring=prod-keyring \
  --location=us-central1 \
  --purpose=encryption \
  --rotation-period=90d \
  --next-rotation-time=2025-12-31T00:00:00Z

# Create key with HSM protection
gcloud kms keys create hsm-protected-key \
  --keyring=prod-keyring \
  --location=us-central1 \
  --purpose=encryption \
  --protection-level=HSM

# Grant encryption/decryption permission
gcloud kms keys add-iam-policy-binding prod-data-key \
  --keyring=prod-keyring \
  --location=us-central1 \
  --member=serviceAccount:app@project.iam.gserviceaccount.com \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# Encrypt data
gcloud kms encrypt \
  --key=prod-data-key \
  --keyring=prod-keyring \
  --location=us-central1 \
  --plaintext-file=sensitive-data.txt \
  --ciphertext-file=encrypted-data.enc
```

**Key Rotation Strategies**:

1. **Automatic Rotation** (Recommended):
```bash
# Set rotation period to 90 days
gcloud kms keys update prod-data-key \
  --keyring=prod-keyring \
  --location=us-central1 \
  --rotation-period=90d
```

2. **Manual Rotation**:
```bash
# Create new key version
gcloud kms keys versions create \
  --key=prod-data-key \
  --keyring=prod-keyring \
  --location=us-central1 \
  --primary
```

3. **Key Version Lifecycle**:
   - ENABLED: Active, can encrypt/decrypt
   - DISABLED: Can decrypt only
   - SCHEDULED_FOR_DESTRUCTION: 24-hour grace period
   - DESTROYED: Permanently deleted

**Hardware Security Module (HSM)**:
- FIPS 140-2 Level 3 validated
- Physical tamper protection
- Dedicated hardware keys
- Higher cost, maximum security
- Required for some compliance frameworks

**External Key Manager (EKM)**:
- Keys stored outside GCP
- Customer-managed key infrastructure
- Maximum control and isolation
- Supported for Compute Engine, GKE, BigQuery
- Higher complexity and latency

### Secret Manager

**Architecture**:
```
Application → Secret Manager API → Secret Version
                                        ↓
                                   [CMEK Encrypted]
                                        ↓
                                   Secret Value
```

**Secret Management Best Practices**:

1. **Secret Organization**:
```bash
# Create secret with labels
gcloud secrets create db-password \
  --replication-policy=automatic \
  --labels=env=prod,app=payment

# Add secret version
echo -n "super-secret-password" | \
  gcloud secrets versions add db-password --data-file=-

# Grant access to service account
gcloud secrets add-iam-policy-binding db-password \
  --member=serviceAccount:app@project.iam.gserviceaccount.com \
  --role=roles/secretmanager.secretAccessor
```

2. **Replication Policies**:

**Automatic Replication**:
```bash
# Replicate to all regions
gcloud secrets create api-key \
  --replication-policy=automatic
```

**User-Managed Replication**:
```bash
# Specific regions only
gcloud secrets create pii-data-key \
  --replication-policy=user-managed \
  --locations=us-central1,us-east1
```

3. **Accessing Secrets in Applications**:

**Compute Engine**:
```bash
# Startup script
#!/bin/bash
SECRET=$(gcloud secrets versions access latest --secret=db-password)
# Use $SECRET in application
```

**Cloud Run/Cloud Functions**:
```yaml
# Environment variable from secret
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-password
        key: latest
```

**GKE**:
```yaml
# Secret Store CSI Driver
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: app-secrets
spec:
  provider: gcp
  parameters:
    secrets: |
      - resourceName: "projects/PROJECT_ID/secrets/db-password/versions/latest"
        path: "db-password"
```

4. **Secret Rotation**:
```bash
# Add new version
echo -n "new-password" | \
  gcloud secrets versions add db-password --data-file=-

# Disable old version
gcloud secrets versions disable 1 --secret=db-password

# Destroy old version
gcloud secrets versions destroy 1 --secret=db-password
```

### Certificate Authority Service

**Private CA Hierarchy**:
```
Root CA (Offline, long-lived)
    ↓
Subordinate CA (DevOps, Policy-enforcing)
    ↓
Leaf Certificates (Applications, Services)
```

**Use Cases**:
- Internal service-to-service TLS
- mTLS authentication
- Code signing certificates
- Device identity certificates
- Custom PKI infrastructure

**Configuration**:
```bash
# Create CA pool
gcloud privateca pools create prod-ca-pool \
  --location=us-central1 \
  --tier=ENTERPRISE

# Create root CA
gcloud privateca roots create root-ca \
  --pool=prod-ca-pool \
  --location=us-central1 \
  --subject="CN=Internal Root CA, O=MyCompany" \
  --key-algorithm=rsa-4096 \
  --max-chain-length=2

# Create subordinate CA
gcloud privateca subordinates create issuing-ca \
  --pool=prod-ca-pool \
  --location=us-central1 \
  --issuer-pool=prod-ca-pool \
  --issuer-location=us-central1 \
  --subject="CN=Issuing CA, O=MyCompany"

# Issue certificate
gcloud privateca certificates create \
  --issuer-pool=prod-ca-pool \
  --issuer-location=us-central1 \
  --csr=server.csr \
  --cert-output-file=server.crt \
  --validity=P30D
```

**Certificate Templates**:
```yaml
# Web server certificate template
predefined_values:
  key_usage:
    base_key_usage:
      digital_signature: true
      key_encipherment: true
    extended_key_usage:
      server_auth: true
  ca_options:
    is_ca: false
identity_constraints:
  allow_subject_passthrough: true
  allow_subject_alt_names_passthrough: true
```

### Data Loss Prevention (DLP) API

**DLP Architecture**:
```
Data Source → DLP API → Inspection → Classification
                 ↓                        ↓
            Templates              De-identification
                 ↓                        ↓
            Job Triggers            Protected Data
```

**InfoType Detection**:

**Built-in InfoTypes** (100+ predefined):
- Credit card numbers (CREDIT_CARD_NUMBER)
- Social Security numbers (US_SOCIAL_SECURITY_NUMBER)
- Email addresses (EMAIL_ADDRESS)
- Phone numbers (PHONE_NUMBER)
- Passport numbers (PASSPORT)
- Medical record numbers (MEDICAL_RECORD_NUMBER)
- IP addresses (IP_ADDRESS)

**Custom InfoTypes**:
```python
# Dictionary-based
custom_info_type = {
    "info_type": {"name": "EMPLOYEE_ID"},
    "dictionary": {
        "word_list": {"words": ["EMP-001", "EMP-002", "EMP-003"]}
    }
}

# Regex-based
custom_info_type = {
    "info_type": {"name": "CUSTOM_ID"},
    "regex": {"pattern": r"[A-Z]{3}-\d{6}"}
}

# Large custom dictionary
custom_info_type = {
    "info_type": {"name": "PROPRIETARY_TERM"},
    "dictionary": {
        "cloud_storage_path": {
            "path": "gs://bucket/dictionary.txt"
        }
    }
}
```

**Inspection Templates**:
```python
# Create inspection template
inspect_config = {
    "info_types": [
        {"name": "CREDIT_CARD_NUMBER"},
        {"name": "US_SOCIAL_SECURITY_NUMBER"},
        {"name": "EMAIL_ADDRESS"}
    ],
    "min_likelihood": "LIKELY",
    "limits": {
        "max_findings_per_request": 100
    },
    "include_quote": True
}

# Inspect content
from google.cloud import dlp_v2

dlp = dlp_v2.DlpServiceClient()
response = dlp.inspect_content(
    request={
        "parent": f"projects/{project_id}",
        "inspect_config": inspect_config,
        "item": {"value": content_string}
    }
)
```

**De-identification Methods**:

1. **Redaction** (Remove sensitive data):
```python
deidentify_config = {
    "info_type_transformations": {
        "transformations": [{
            "primitive_transformation": {"redact_config": {}}
        }]
    }
}
```

2. **Masking** (Replace with characters):
```python
deidentify_config = {
    "info_type_transformations": {
        "transformations": [{
            "primitive_transformation": {
                "character_mask_config": {
                    "masking_character": "*",
                    "number_to_mask": 4,
                    "reverse_order": True  # Mask last 4 digits
                }
            }
        }]
    }
}
# 1234-5678-9012-3456 → 1234-5678-9012-****
```

3. **Crypto-based Tokenization** (Reversible):
```python
deidentify_config = {
    "info_type_transformations": {
        "transformations": [{
            "primitive_transformation": {
                "crypto_replace_ffx_fpe_config": {
                    "crypto_key": {
                        "kms_wrapped": {
                            "wrapped_key": wrapped_key,
                            "crypto_key_name": kms_key_name
                        }
                    },
                    "alphabet": "NUMERIC"
                }
            }
        }]
    }
}
```

4. **Date Shifting** (Preserve relative time):
```python
deidentify_config = {
    "record_transformations": {
        "field_transformations": [{
            "fields": [{"name": "birth_date"}],
            "primitive_transformation": {
                "date_shift_config": {
                    "upper_bound_days": 30,
                    "lower_bound_days": -30,
                    "crypto_key": crypto_key
                }
            }
        }]
    }
}
```

**DLP Job Triggers** (Automated Scanning):
```python
# Create job trigger for Cloud Storage
job_trigger = {
    "inspect_job": {
        "storage_config": {
            "cloud_storage_options": {
                "file_set": {
                    "url": "gs://my-bucket/*"
                }
            }
        },
        "inspect_config": inspect_config,
        "actions": [{
            "save_findings": {
                "output_config": {
                    "table": {
                        "project_id": project_id,
                        "dataset_id": "dlp_findings",
                        "table_id": "findings"
                    }
                }
            }
        }]
    },
    "triggers": [{
        "schedule": {
            "recurrence_period_duration": {
                "seconds": 86400  # Daily
            }
        }
    }]
}
```

**Integration Patterns**:

1. **Cloud Storage DLP**:
   - Scan buckets for sensitive data
   - Automated redaction of uploaded files
   - Quarantine buckets for violations

2. **BigQuery DLP**:
   - Profile datasets for sensitive columns
   - De-identify data before analytics
   - Column-level access control based on findings

3. **Real-time DLP** (via Cloud Functions):
   - Pub/Sub message inspection
   - API request/response filtering
   - Stream processing integration

## Compliance Architecture

### Compliance Frameworks - Detailed Requirements

#### PCI-DSS (Payment Card Industry Data Security Standard)

**Architecture Pattern**:
```
Internet
    ↓
[Cloud Armor - WAF]
    ↓
[Load Balancer - TLS Termination]
    ↓
[VPC Service Controls Perimeter]
    ↓
├── Cardholder Data Environment (CDE)
│   ├── Application Tier (GKE/GCE)
│   │   └── No cardholder data storage
│   ├── Database Tier (Cloud SQL)
│   │   ├── CMEK encryption
│   │   ├── Private IP only
│   │   └── Automated backups (encrypted)
│   └── Logging Infrastructure
│       ├── Cloud Logging (CMEK)
│       ├── 1-year retention minimum
│       └── Immutable storage
└── Non-CDE Environment
    └── Separate VPC/Project
```

**PCI-DSS Requirements Mapping**:

| Requirement | GCP Implementation |
|-------------|-------------------|
| Build and Maintain Secure Network | VPC Service Controls, Firewall rules, Cloud Armor |
| Protect Cardholder Data | CMEK encryption, no plaintext storage, tokenization |
| Maintain Vulnerability Management | Security Command Center, OS Patch Management |
| Implement Strong Access Control | IAM, MFA, least privilege, service accounts |
| Monitor and Test Networks | Cloud Logging, VPC Flow Logs, Cloud IDS |
| Maintain Information Security Policy | Organization policies, documented procedures |

**Key Controls**:

1. **Network Segmentation** (Req 1.2):
```yaml
# Separate CDE and non-CDE
vpc_service_perimeter:
  name: "pci-cde-perimeter"
  projects:
    - cardholder-data-prod
  restricted_services:
    - storage.googleapis.com
    - compute.googleapis.com
    - sqladmin.googleapis.com
  access_levels:
    - corp_network
    - managed_devices
```

2. **Data Encryption** (Req 3.4):
```bash
# CMEK for all PCI data
gcloud sql instances create pci-db \
  --database-version=POSTGRES_14 \
  --tier=db-custom-4-16384 \
  --region=us-central1 \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/pci/cryptoKeys/db-key \
  --no-assign-ip
```

3. **Access Logging** (Req 10.2):
```bash
# Enable all audit logs
gcloud logging sinks create pci-audit-logs \
  storage.googleapis.com/pci-audit-bucket \
  --log-filter='logName:"cloudaudit.googleapis.com"'
```

4. **Quarterly Vulnerability Scans** (Req 11.2):
- Security Command Center Web Security Scanner
- Third-party ASV scans
- Penetration testing annually

#### HIPAA (Health Insurance Portability and Accountability Act)

**Architecture Pattern**:
```
Healthcare Application
    ↓
[BeyondCorp / IAP - Zero Trust Access]
    ↓
[VPC Service Controls - PHI Boundary]
    ↓
├── PHI Environment (HIPAA-eligible services only)
│   ├── Cloud Storage (CMEK, versioning)
│   ├── Cloud SQL (CMEK, automated backups)
│   ├── BigQuery (CMEK, column-level security)
│   ├── GKE (Binary Authorization, Workload Identity)
│   └── DLP (PHI detection and de-identification)
├── Audit Trail (immutable)
│   ├── Admin Activity Logs (enabled)
│   ├── Data Access Logs (enabled)
│   └── Log retention: 6 years minimum
└── Encryption
    ├── CMEK for all PHI at rest
    └── TLS 1.2+ for PHI in transit
```

**HIPAA Requirements Mapping**:

| Safeguard | GCP Implementation |
|-----------|-------------------|
| Access Control (164.312(a)(1)) | IAM, MFA, context-aware access, least privilege |
| Audit Controls (164.312(b)) | Cloud Audit Logs, 6-year retention, immutable |
| Integrity (164.312(c)(1)) | Checksums, versioning, DLP monitoring |
| Transmission Security (164.312(e)(1)) | TLS 1.2+, VPN, Private Google Access |
| Encryption (164.312(a)(2)(iv)) | CMEK for at-rest, TLS for in-transit |

**HIPAA-Eligible Services** (with signed BAA):
- Compute Engine, GKE, Cloud Run, Cloud Functions
- Cloud Storage, Cloud SQL, Firestore, Bigtable
- BigQuery, Dataflow, Dataproc
- Cloud KMS, Secret Manager, DLP
- Cloud Logging, Cloud Monitoring
- VPC, Cloud Load Balancing, Cloud CDN

**NOT HIPAA-Eligible**:
- Cloud DNS (public zones)
- App Engine Standard
- Certain Firebase services
- Third-party marketplace solutions (verify individually)

**Key Controls**:

1. **Business Associate Agreement (BAA)**:
- Required for all projects storing PHI
- Sign via Google Cloud Console
- Review covered services list

2. **Access Controls**:
```yaml
# Minimum necessary access
iam_policy:
  bindings:
    - role: roles/healthcare.datasetViewer
      members:
        - group:clinical-staff@company.com
      condition:
        title: "Business hours only"
        expression: |
          request.time.getHours("America/New_York") >= 8 &&
          request.time.getHours("America/New_York") <= 18
```

3. **PHI Detection and Protection**:
```python
# DLP template for PHI
phi_inspect_config = {
    "info_types": [
        {"name": "MEDICAL_RECORD_NUMBER"},
        {"name": "US_HEALTHCARE_NPI"},
        {"name": "PERSON_NAME"},
        {"name": "DATE_OF_BIRTH"},
        {"name": "US_SOCIAL_SECURITY_NUMBER"}
    ],
    "min_likelihood": "POSSIBLE"
}
```

4. **Breach Notification Readiness**:
- Real-time alerting on data access anomalies
- Automated incident response playbooks
- 60-day notification requirement preparation

#### SOC 2 (Service Organization Control 2)

**Trust Service Criteria**:

1. **Security** (Common Criteria):
```yaml
Controls:
  - Multi-factor authentication (Cloud Identity)
  - Network security (VPC Service Controls, firewalls)
  - Logical access controls (IAM, least privilege)
  - Change management (Cloud Build, approvals)
  - Risk assessment (Security Command Center)
```

2. **Availability**:
```yaml
Controls:
  - Multi-region architecture (GKE, Cloud Storage)
  - Automated failover (Global Load Balancer)
  - Backup and recovery (automated snapshots)
  - Performance monitoring (Cloud Monitoring)
  - Capacity planning (autoscaling)
```

3. **Processing Integrity**:
```yaml
Controls:
  - Data validation (Cloud Functions, Dataflow)
  - Error handling and logging
  - Transaction monitoring
  - Reconciliation processes
  - Quality assurance testing
```

4. **Confidentiality**:
```yaml
Controls:
  - Encryption (CMEK, TLS)
  - Access controls (IAM, VPC-SC)
  - DLP (sensitive data detection)
  - Secure data disposal
  - Non-disclosure agreements
```

5. **Privacy** (optional):
```yaml
Controls:
  - Consent management
  - Data retention policies
  - Right to access/deletion (GDPR)
  - Privacy notice and disclosure
  - Data processing agreements
```

**SOC 2 Architecture Pattern**:
```
Production Environment
├── Change Control
│   ├── CI/CD Pipeline (Cloud Build)
│   ├── Binary Authorization (only approved images)
│   ├── Deployment approvals (required)
│   └── Rollback procedures (automated)
├── Access Management
│   ├── JIT access (temporary elevated access)
│   ├── Access reviews (quarterly)
│   ├── Separation of duties
│   └── Terminated user removal (automated)
├── Monitoring
│   ├── Availability metrics (uptime checks)
│   ├── Security events (SCC findings)
│   ├── Performance metrics (latency, errors)
│   └── Audit logs (all activities)
└── Incident Response
    ├── Detection (alerting)
    ├── Response playbooks
    ├── Post-incident reviews
    └── Continuous improvement
```

#### FedRAMP (Federal Risk and Authorization Management Program)

**FedRAMP Levels**:
- **Low**: Low impact to operations, assets, or individuals
- **Moderate**: Moderate impact (most common)
- **High**: High impact to national security

**GCP FedRAMP Authorized Services**:
- Compute Engine, GKE, Cloud Run
- Cloud Storage, Cloud SQL, Firestore
- BigQuery, Dataflow, Pub/Sub
- VPC, Cloud Load Balancing
- Cloud KMS, Cloud Armor
- Cloud Logging, Cloud Monitoring

**Architecture Requirements**:

1. **Boundary Protection**:
```yaml
# FedRAMP boundary using VPC-SC
fedramp_perimeter:
  projects:
    - fedramp-workload-prod
  restricted_services: ALL
  access_levels:
    - us_only_access  # Geographic restriction
    - piv_card_required  # PIV/CAC authentication
  vpc_accessible_services:
    restriction: ENABLED
    allowed_services:
      - RESTRICTED-SERVICES-ONLY
```

2. **Strong Authentication**:
- PIV/CAC card integration (via Cloud Identity)
- MFA required for all users
- 15-minute session timeout
- Concurrent session limits

3. **Audit and Accountability**:
```bash
# Enhanced audit logging
gcloud logging sinks create fedramp-audit \
  bigquery.googleapis.com/projects/PROJECT/datasets/audit \
  --log-filter='protoPayload.serviceName="*"'

# 1-year online retention, 7-year archive
```

4. **Incident Response**:
- 1-hour detection requirement
- Incident reporting to FedRAMP PMO
- Root cause analysis documentation
- Corrective action plans

**Continuous Monitoring**:
- Monthly OSCAL documentation updates
- Quarterly vulnerability scans
- Annual assessment
- Real-time security event monitoring

#### ISO 27001 (Information Security Management)

**Control Categories**:

**A.9 Access Control**:
```yaml
Implementation:
  - IAM policies (principle of least privilege)
  - MFA enforcement (all users)
  - Access reviews (quarterly)
  - User provisioning/deprovisioning (automated)
  - Privileged access management (break-glass accounts)
```

**A.10 Cryptography**:
```yaml
Implementation:
  - CMEK for sensitive data
  - TLS 1.2+ for data in transit
  - Key rotation (90-day maximum)
  - Certificate management (Certificate Authority Service)
  - Cryptographic controls documentation
```

**A.12 Operations Security**:
```yaml
Implementation:
  - Change management (Cloud Build, approvals)
  - Capacity management (autoscaling)
  - Malware protection (Container Scanning)
  - Backup procedures (automated snapshots)
  - Logging and monitoring (Cloud Operations)
```

**A.13 Communications Security**:
```yaml
Implementation:
  - Network segmentation (VPCs, subnets)
  - Network security controls (firewalls, Cloud Armor)
  - Secure transfer (TLS, VPN)
  - Information transfer policies (DLP)
```

**A.14 System Acquisition, Development and Maintenance**:
```yaml
Implementation:
  - Secure development lifecycle (DevSecOps)
  - Security testing (vulnerability scanning)
  - Test data protection (DLP, de-identification)
  - Change control (CI/CD approvals)
```

**Architecture Pattern**:
```
ISO 27001 Compliant Architecture
├── Information Security Policy
│   └── Organization policies (enforced)
├── Risk Management
│   ├── Security Command Center (risk identification)
│   ├── Vulnerability management
│   └── Threat modeling
├── Asset Management
│   ├── Resource inventory (Asset Inventory)
│   ├── Information classification (labels)
│   └── Acceptable use policies
├── Human Resource Security
│   ├── Background checks (documented)
│   ├── Security awareness training
│   └── Termination procedures
└── Business Continuity
    ├── Backup and recovery (tested)
    ├── Redundancy (multi-region)
    └── Disaster recovery plan
```

#### GDPR (General Data Protection Regulation)

**Data Protection Principles**:

1. **Lawfulness, Fairness, Transparency**:
- Documented legal basis for processing
- Privacy notices and consent management
- Clear data collection purposes

2. **Purpose Limitation**:
- Data used only for stated purposes
- Labels for data classification
- DLP to prevent unauthorized use

3. **Data Minimization**:
- Collect only necessary data
- Regular data cleanup
- Anonymization where possible

4. **Accuracy**:
- Data validation processes
- Correction procedures
- User access to own data

5. **Storage Limitation**:
```bash
# Automated data retention
gsutil lifecycle set retention-policy.json gs://user-data-bucket

# retention-policy.json
{
  "lifecycle": {
    "rule": [{
      "action": {"type": "Delete"},
      "condition": {"age": 2555}  # 7 years
    }]
  }
}
```

6. **Integrity and Confidentiality**:
- Encryption (CMEK)
- Access controls (IAM)
- Audit logging (all access)
- Security monitoring

7. **Accountability**:
- Documentation of processing activities
- Data Protection Impact Assessments (DPIA)
- Data Processing Agreements (DPA)
- Compliance monitoring

**GDPR Architecture Pattern**:
```
GDPR-Compliant Application
├── Data Residency
│   ├── EU-only regions (europe-west1, europe-north1)
│   ├── Organization policy (gcp.resourceLocations)
│   └── Regional CMEK keys
├── Data Subject Rights
│   ├── Right to Access (export APIs)
│   ├── Right to Rectification (update procedures)
│   ├── Right to Erasure (deletion workflows)
│   ├── Right to Portability (data export)
│   └── Right to Object (opt-out mechanisms)
├── Privacy by Design
│   ├── DLP (PII detection)
│   ├── Pseudonymization (tokenization)
│   ├── Data minimization (collection limits)
│   └── Privacy impact assessments
└── Breach Notification
    ├── Detection (72-hour requirement)
    ├── Incident response procedures
    ├── Notification workflows
    └── Documentation requirements
```

**Data Subject Rights Implementation**:
```python
# Right to Access - Export user data
def export_user_data(user_id):
    """Export all data for a user (GDPR Article 15)"""
    data = {
        'profile': get_user_profile(user_id),
        'transactions': get_user_transactions(user_id),
        'logs': get_user_activity_logs(user_id)
    }
    # De-identify other users' data in logs
    dlp_deidentify(data['logs'])
    return data

# Right to Erasure - Delete user data
def delete_user_data(user_id):
    """Delete all user data (GDPR Article 17)"""
    # Mark for deletion (retain audit trail)
    mark_deleted(user_id)
    # Anonymize in backups
    schedule_backup_anonymization(user_id)
    # Remove from active systems
    delete_from_databases(user_id)
```

### Compliance Control Matrix

| Control | PCI-DSS | HIPAA | SOC 2 | FedRAMP | ISO 27001 | GDPR |
|---------|---------|-------|-------|---------|-----------|------|
| CMEK Encryption | Required | Addressable | Required | Required | Required | Recommended |
| MFA | Required | Required | Required | Required | Required | Recommended |
| VPC Service Controls | Recommended | Required | Recommended | Required | Recommended | Recommended |
| Cloud Audit Logs | Required | Required | Required | Required | Required | Required |
| Data Residency | No | No | Optional | Yes (US) | No | Yes (EU) |
| Access Reviews | Quarterly | Regular | Quarterly | Quarterly | Regular | Regular |
| Vulnerability Scanning | Quarterly | Regular | Regular | Monthly | Regular | Regular |
| Incident Response | Required | Required | Required | Required | Required | Required |
| Data Retention | 1 year | 6 years | Varies | 3 years | Varies | Documented |
| Penetration Testing | Annual | Regular | Annual | Annual | Annual | Recommended |

### Compliance Best Practices

**1. Data Residency and Sovereignty**:
```yaml
# Organization policy for location restriction
constraints/gcp.resourceLocations:
  allowed_values:
    - in:eu-locations  # GDPR
    - in:us-locations  # FedRAMP
    - in:eu-locations  # ISO 27001 EU operations

# Enforce via policy
gcloud resource-manager org-policies set-policy policy.yaml \
  --organization=ORG_ID
```

**2. Audit Logging Strategy**:
```bash
# Enable all log types
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:cloud-logs@system.gserviceaccount.com \
  --role=roles/logging.configWriter

# Export to immutable storage
gcloud logging sinks create compliance-audit-logs \
  storage.googleapis.com/compliance-audit-bucket \
  --log-filter='logName:"cloudaudit.googleapis.com"' \
  --organization=ORG_ID

# Retention lock (immutable)
gsutil retention set 2555d gs://compliance-audit-bucket
gsutil retention lock gs://compliance-audit-bucket
```

**3. Access Control Architecture**:
```yaml
# Segregation of duties
roles:
  - network_admin: Cannot modify IAM
  - security_admin: Cannot modify network
  - audit_viewer: Read-only access to logs
  - compliance_officer: Policy management only

# Break-glass procedures
emergency_access:
  accounts: [breakglass-1, breakglass-2]
  monitoring: REAL_TIME_ALERTS
  approval: OFFLINE_CREDENTIAL_REQUIRED
  review: MANDATORY_POST_ACCESS
```

**4. Encryption Strategy by Compliance**:
```yaml
PCI-DSS:
  at_rest: CMEK (required for CHD)
  in_transit: TLS 1.2+ (minimum)
  key_rotation: Every 90 days

HIPAA:
  at_rest: CMEK (addressable, recommended)
  in_transit: TLS 1.2+ (minimum)
  key_rotation: Risk-based

FedRAMP:
  at_rest: FIPS 140-2 validated (CMEK HSM)
  in_transit: TLS 1.2+, FIPS-approved
  key_rotation: Every 365 days maximum

GDPR:
  at_rest: State-of-the-art encryption
  in_transit: TLS 1.2+
  key_rotation: Risk-based assessment
```

**5. Continuous Compliance Monitoring**:
```python
# Automated compliance checks
compliance_checks = {
    'cmek_encryption': check_cmek_enabled,
    'public_ips': check_no_public_ips,
    'audit_logs': check_audit_logs_enabled,
    'vpc_sc_perimeter': check_vpc_sc_configured,
    'mfa_enforcement': check_mfa_required,
    'data_residency': check_resource_locations,
    'access_reviews': check_recent_reviews
}

# Daily compliance scan
for check_name, check_func in compliance_checks.items():
    result = check_func()
    if not result.compliant:
        alert_compliance_team(check_name, result)
        create_remediation_ticket(check_name, result)
```

**6. Documentation Requirements**:
- Architecture diagrams (updated quarterly)
- Data flow diagrams (all sensitive data)
- Risk assessments (annual minimum)
- Incident response procedures
- Business continuity plans
- Compliance control mapping
- Audit reports and evidence
- Third-party security reviews

## Security Monitoring

### Security Command Center (SCC)

**Architecture**:
```
GCP Resources
    ↓
[Asset Discovery] → Asset Inventory
    ↓
[Security Health Analytics] → Misconfigurations
    ↓
[Web Security Scanner] → Application Vulnerabilities
    ↓
[Event Threat Detection] → Anomalous Activity
    ↓
[Container Threat Detection] → Runtime threats
    ↓
[Cloud IDS] → Network intrusions
    ↓
Security Command Center Dashboard
    ↓
├── Findings (prioritized)
├── Assets (inventory)
├── Sources (detectors)
└── Notifications (Pub/Sub)
```

**SCC Tiers**:

| Feature | Standard (Free) | Premium |
|---------|----------------|---------|
| Asset Discovery | Yes | Yes |
| Security Health Analytics | Basic | Advanced |
| Web Security Scanner | No | Yes |
| Event Threat Detection | No | Yes |
| Container Threat Detection | No | Yes |
| Continuous Exports | No | Yes |
| Rapid Vulnerability Detection | No | Yes |

**Security Health Analytics Detectors** (Premium):
- Publicly accessible Cloud Storage buckets
- Open firewall rules
- Weak SSL/TLS policies
- Missing encryption
- Service account key age
- Over-privileged service accounts
- Unencrypted Cloud SQL instances
- Missing audit logging configuration
- Public IP on Cloud SQL
- Legacy authentication enabled

**Event Threat Detection Categories**:
- **Malware**: Bad domain, IP reputation
- **Cryptomining**: Resource hijacking
- **Brute Force**: SSH, RDP attempts
- **Data Exfiltration**: Unusual egress patterns
- **IAM Anomalies**: Unusual grants, privilege escalation
- **Persistence**: Backdoor creation attempts

**Configuration**:
```bash
# Enable SCC API
gcloud services enable securitycenter.googleapis.com

# List all findings
gcloud scc findings list ORGANIZATION_ID \
  --source=SOURCE_ID \
  --filter="state=\"ACTIVE\" AND severity=\"HIGH\""

# Create notification config
gcloud scc notifications create high-severity-findings \
  --organization=ORG_ID \
  --description="High severity findings" \
  --pubsub-topic=projects/PROJECT/topics/scc-notifications \
  --filter="severity=\"HIGH\""

# Mark finding as resolved
gcloud scc findings update FINDING_NAME \
  --source=SOURCE_ID \
  --state=INACTIVE \
  --organization=ORG_ID
```

**Integration with SIEM**:
```python
# Export findings to Splunk/QRadar
from google.cloud import pubsub_v1

subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path(PROJECT, SUBSCRIPTION)

def callback(message):
    finding = json.loads(message.data)
    # Forward to SIEM
    send_to_siem(finding)
    message.ack()

subscriber.subscribe(subscription_path, callback=callback)
```

### Cloud Audit Logs

**Log Types**:

1. **Admin Activity Logs**:
- Always enabled, no configuration required
- 400-day retention
- No storage charges
- Captures API calls that modify configuration
- Examples: Creating VMs, modifying IAM, deleting resources

2. **Data Access Logs**:
- Disabled by default (except BigQuery)
- Captures read operations on user data
- Can generate high volume
- Storage charges apply
- Examples: Reading Cloud Storage objects, querying databases

3. **System Event Logs**:
- Always enabled
- Google-initiated administrative actions
- 400-day retention
- Examples: VM live migration, auto-scaling events

4. **Policy Denied Logs**:
- Always enabled
- Captures requests denied by security policies
- Useful for troubleshooting access issues
- Examples: IAM denials, VPC-SC violations, organization policy blocks

**Audit Log Structure**:
```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "user@example.com"
    },
    "authorizationInfo": [{
      "resource": "projects/my-project/...",
      "permission": "storage.buckets.delete",
      "granted": true
    }],
    "methodName": "storage.buckets.delete",
    "serviceName": "storage.googleapis.com",
    "resourceName": "projects/_/buckets/my-bucket"
  },
  "resource": {
    "type": "gcs_bucket",
    "labels": {
      "project_id": "my-project",
      "bucket_name": "my-bucket"
    }
  },
  "severity": "NOTICE",
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Enable Data Access Logs**:
```yaml
# audit-config.yaml
auditConfigs:
- auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
  service: allServices

# Apply to project
gcloud projects set-iam-policy PROJECT_ID audit-config.yaml
```

**Log Retention and Export**:
```bash
# Export audit logs to Cloud Storage
gcloud logging sinks create audit-export \
  storage.googleapis.com/audit-logs-bucket \
  --log-filter='logName:"cloudaudit.googleapis.com"' \
  --organization=ORG_ID

# Export to BigQuery for analysis
gcloud logging sinks create audit-analysis \
  bigquery.googleapis.com/projects/PROJECT/datasets/audit_logs \
  --log-filter='logName:"cloudaudit.googleapis.com"'

# Export to Pub/Sub for real-time processing
gcloud logging sinks create audit-streaming \
  pubsub.googleapis.com/projects/PROJECT/topics/audit-stream \
  --log-filter='logName:"cloudaudit.googleapis.com"'
```

**Critical Audit Log Queries**:
```sql
-- Failed authentication attempts
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.status.message
FROM `project.dataset.cloudaudit_googleapis_com_activity`
WHERE protoPayload.status.code != 0
  AND protoPayload.methodName LIKE '%login%'
ORDER BY timestamp DESC

-- IAM policy changes
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.methodName,
  protoPayload.resourceName
FROM `project.dataset.cloudaudit_googleapis_com_activity`
WHERE protoPayload.methodName LIKE '%setIamPolicy%'
ORDER BY timestamp DESC

-- Resource deletions
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.methodName,
  resource.type,
  protoPayload.resourceName
FROM `project.dataset.cloudaudit_googleapis_com_activity`
WHERE protoPayload.methodName LIKE '%.delete'
ORDER BY timestamp DESC

-- Service account key creation (potential security risk)
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.request.name as service_account
FROM `project.dataset.cloudaudit_googleapis_com_activity`
WHERE protoPayload.methodName = 'google.iam.admin.v1.CreateServiceAccountKey'
ORDER BY timestamp DESC
```

### Monitoring and Alerting

**Security Alert Policies**:

1. **Unauthorized Access Attempts**:
```yaml
# Alert on repeated policy denials
condition:
  displayName: "Repeated authorization failures"
  conditionThreshold:
    filter: |
      resource.type="audited_resource"
      protoPayload.status.code!=0
      protoPayload.authorizationInfo.granted=false
    aggregations:
      - alignmentPeriod: 300s
        perSeriesAligner: ALIGN_RATE
    comparison: COMPARISON_GT
    thresholdValue: 10  # More than 10 denials in 5 minutes
    duration: 60s
```

2. **IAM Changes**:
```yaml
# Alert on IAM policy modifications
condition:
  displayName: "IAM policy changes"
  conditionThreshold:
    filter: |
      protoPayload.methodName="SetIamPolicy"
      protoPayload.serviceData.policyDelta.bindingDeltas.action="ADD"
```

3. **Resource Deletion**:
```yaml
# Alert on production resource deletion
condition:
  displayName: "Production resource deletion"
  conditionThreshold:
    filter: |
      protoPayload.methodName=~"*.delete"
      resource.labels.project_id="prod-project"
      severity="NOTICE"
```

4. **Service Account Key Creation**:
```yaml
# Alert on service account key creation (discouraged practice)
condition:
  displayName: "Service account key created"
  conditionThreshold:
    filter: |
      protoPayload.methodName="google.iam.admin.v1.CreateServiceAccountKey"
```

5. **Anomalous Data Access**:
```yaml
# Alert on unusual data access volume
condition:
  displayName: "High volume data access"
  conditionThreshold:
    filter: |
      logName:"cloudaudit.googleapis.com/data_access"
      protoPayload.methodName=~"*.get"
    aggregations:
      - alignmentPeriod: 600s
        perSeriesAligner: ALIGN_RATE
        groupByFields: ["protoPayload.authenticationInfo.principalEmail"]
    comparison: COMPARISON_GT
    thresholdValue: 1000  # More than 1000 reads in 10 minutes
```

**Log-Based Metrics**:
```bash
# Create metric for failed logins
gcloud logging metrics create failed_logins \
  --description="Count of failed login attempts" \
  --log-filter='protoPayload.status.code!=0 AND protoPayload.methodName=~".*login.*"'

# Create metric for admin actions
gcloud logging metrics create admin_actions \
  --description="Count of administrative actions" \
  --log-filter='logName:"cloudaudit.googleapis.com/activity"'

# Create metric for VPC-SC violations
gcloud logging metrics create vpc_sc_violations \
  --description="VPC Service Controls violations" \
  --log-filter='protoPayload.metadata.violationReason:*'
```

**Security Dashboard Setup**:
```yaml
# Cloud Monitoring dashboard for security metrics
displayName: "Security Operations Dashboard"
mosaicLayout:
  columns: 12
  tiles:
    - width: 6
      height: 4
      widget:
        title: "Failed Authentication Attempts"
        xyChart:
          dataSets:
            - timeSeriesQuery:
                timeSeriesFilter:
                  filter: 'metric.type="logging.googleapis.com/user/failed_logins"'
    - width: 6
      height: 4
      widget:
        title: "IAM Policy Changes"
        xyChart:
          dataSets:
            - timeSeriesQuery:
                timeSeriesFilter:
                  filter: 'metric.type="logging.googleapis.com/user/iam_changes"'
    - width: 12
      height: 4
      widget:
        title: "Security Command Center Findings"
        scorecard:
          timeSeriesQuery:
            timeSeriesFilter:
              filter: 'resource.type="security_finding"'
              aggregation:
                groupByFields: ["metric.severity"]
```

### Incident Response Architecture

**Detection → Response → Recovery Workflow**:
```
Detection
├── Security Command Center (findings)
├── Cloud Logging (audit logs)
├── Cloud Monitoring (alerts)
└── Third-party tools (SIEM)
    ↓
Triage
├── Severity classification
├── Impact assessment
├── Stakeholder notification
└── Incident ticket creation
    ↓
Containment
├── Isolate affected resources
├── Revoke compromised credentials
├── Block malicious IPs/domains
└── Preserve evidence
    ↓
Investigation
├── Log analysis (audit trail)
├── Threat hunting
├── Root cause identification
└── Scope determination
    ↓
Remediation
├── Remove threats
├── Patch vulnerabilities
├── Restore from backups
└── Implement compensating controls
    ↓
Recovery
├── Service restoration
├── Monitoring for recurrence
├── Post-incident review
└── Lessons learned documentation
```

**Automated Response Patterns**:

1. **Compromised Service Account**:
```python
def respond_to_compromised_sa(service_account_email):
    """Automated response to compromised service account"""
    # 1. Disable service account
    disable_service_account(service_account_email)

    # 2. Delete all keys
    delete_all_sa_keys(service_account_email)

    # 3. Revoke active tokens
    revoke_sa_tokens(service_account_email)

    # 4. Alert security team
    send_alert(f"Service account {service_account_email} disabled due to compromise")

    # 5. Create incident ticket
    create_incident(service_account_email)

    # 6. Log all actions
    audit_log_response_actions(service_account_email)
```

2. **Malicious IP Detection**:
```python
def block_malicious_ip(ip_address, reason):
    """Block malicious IP at Cloud Armor"""
    # 1. Add to Cloud Armor deny list
    add_to_cloud_armor_denylist(ip_address)

    # 2. Update firewall rules
    create_firewall_deny_rule(ip_address)

    # 3. Alert SOC
    alert_soc(f"Blocked IP {ip_address}: {reason}")

    # 4. Log to SIEM
    log_to_siem({
        'action': 'block_ip',
        'ip': ip_address,
        'reason': reason,
        'timestamp': datetime.now()
    })
```

3. **Data Exfiltration Response**:
```python
def respond_to_data_exfiltration(bucket_name, principal):
    """Response to suspected data exfiltration"""
    # 1. Enable VPC Service Controls
    enable_vpc_sc_perimeter(bucket_name)

    # 2. Revoke access for suspicious principal
    revoke_bucket_access(bucket_name, principal)

    # 3. Enable Object Versioning
    enable_bucket_versioning(bucket_name)

    # 4. Take bucket snapshot
    snapshot_bucket(bucket_name)

    # 5. Alert data owners
    notify_data_owners(bucket_name)

    # 6. Initiate forensics
    start_forensic_investigation(bucket_name, principal)
```

**Forensics Evidence Collection**:
```bash
# Snapshot compromised VM for forensics
gcloud compute disks snapshot DISK_NAME \
  --zone=ZONE \
  --snapshot-names=forensic-snapshot-$(date +%Y%m%d-%H%M%S) \
  --description="Forensic snapshot for incident-12345"

# Export audit logs for specific time range
gcloud logging read \
  'timestamp>="2025-01-15T00:00:00Z" AND timestamp<="2025-01-15T23:59:59Z"' \
  --format=json \
  --order=asc \
  > forensic-logs-incident-12345.json

# Preserve VPC Flow Logs
gcloud compute networks subnets update SUBNET \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-1-min \
  --logging-metadata=include-all \
  --region=REGION
```

**Incident Response Playbooks** (Key Scenarios):

1. **Ransomware Detection**:
   - Isolate affected VMs (firewall rules)
   - Disable service accounts
   - Restore from clean backups
   - Analyze malware samples
   - Report to authorities

2. **Credential Compromise**:
   - Force password reset
   - Revoke all sessions
   - Delete API keys
   - Review access logs
   - Implement MFA if not already

3. **Insider Threat**:
   - Preserve audit evidence
   - Revoke access immediately
   - Review all actions taken
   - Assess data exposure
   - Legal/HR involvement

4. **DDoS Attack**:
   - Enable Cloud Armor Adaptive Protection
   - Implement rate limiting
   - Block attacking IPs
   - Scale resources temporarily
   - Engage Google DDoS support

## Best Practices

### Security Architecture Principles
1. **Zero Trust Model**: Never trust, always verify - authenticate and authorize every request
2. **Defense in Depth**: Multiple security layers - perimeter, network, application, data
3. **Least Privilege**: Minimum necessary access - start restrictive, expand as needed
4. **Segregation of Duties**: Separate critical roles - no single user with complete control
5. **Encryption Everywhere**: Data at rest and in transit - CMEK for sensitive data
6. **Security by Design**: Build security into architecture - not an afterthought
7. **Continuous Monitoring**: Real-time security visibility - automated detection and response
8. **Incident Preparedness**: Documented response procedures - tested regularly

### Identity Management Best Practices
1. Use Cloud Identity or Google Workspace for central identity management
2. Enforce multi-factor authentication (MFA) for all users
3. Implement context-aware access (BeyondCorp Enterprise)
4. Use groups for access management (never individual users)
5. Regular access reviews and certifications (quarterly minimum)
6. Service account lifecycle management (creation, rotation, deletion)
7. Avoid service account keys when possible (use Workload Identity)
8. Implement break-glass procedures for emergency access

### Network Security Best Practices
1. Use VPC Service Controls for API-level perimeter security
2. Implement hierarchical firewall policies for consistent enforcement
3. Enable Private Google Access for resources without external IPs
4. Use Cloud Armor for application-layer DDoS protection
5. Deploy Cloud IDS for network-based threat detection
6. Enable VPC Flow Logs for network monitoring and forensics
7. Use service account-based firewall rules for dynamic environments
8. Implement network segmentation with separate VPCs

### Data Protection Best Practices
1. Use CMEK for sensitive data requiring key control
2. Implement DLP for sensitive data discovery and classification
3. Use Secret Manager for all secrets (never hardcode)
4. Enable versioning on critical data stores
5. Implement data retention and lifecycle policies
6. Use signed URLs or signed policy documents for temporary access
7. Enable object holds for immutable data requirements
8. Implement data loss prevention policies

## Complex Security Scenarios

### Scenario 1: Multi-Region E-Commerce Platform with PCI-DSS Compliance

**Requirements**:
- Global payment processing (PCI-DSS Level 1)
- 99.99% availability
- Data residency for EU customers (GDPR)
- Real-time fraud detection
- Audit trail for 7 years

**Architecture Solution**:
```
Global Layer
├── Cloud Armor (DDoS, WAF, geo-blocking)
├── Global HTTPS Load Balancer
└── Cloud CDN (static content)
    ↓
Regional Layers (US, EU, APAC)
├── VPC Service Controls Perimeter (PCI-DSS CDE)
│   ├── GKE Autopilot (application tier)
│   │   ├── Binary Authorization (only signed images)
│   │   ├── Workload Identity (no service account keys)
│   │   └── Pod Security Policies
│   ├── Cloud SQL (PostgreSQL, CMEK)
│   │   ├── Private IP only
│   │   ├── Automated backups (encrypted)
│   │   ├── Point-in-time recovery
│   │   └── Read replicas (HA)
│   ├── Cloud KMS (HSM-backed keys)
│   │   ├── Separate key rings by region
│   │   └── 90-day rotation
│   └── Payment Tokenization Service
│       └── Cloud Functions + DLP
├── Fraud Detection
│   ├── Pub/Sub (transaction stream)
│   ├── Dataflow (real-time analysis)
│   └── BigQuery ML (fraud models)
├── Audit and Compliance
│   ├── Cloud Audit Logs (all types enabled)
│   ├── Export to Cloud Storage (retention locked, 7 years)
│   ├── Security Command Center Premium
│   └── Weekly vulnerability scans
└── Monitoring
    ├── Cloud Monitoring (SLO tracking)
    ├── Cloud Logging (centralized)
    └── Alert policies (security events)
```

**Key Security Controls**:
1. VPC Service Controls isolate CDE environment
2. CMEK with HSM for cardholder data
3. Cloud Armor blocks attacks at edge
4. No cardholder data in logs (DLP redaction)
5. Quarterly vulnerability scans (ASV)
6. Annual penetration testing
7. Network segmentation (separate VPCs for CDE/non-CDE)
8. Immutable audit logs (retention lock)

**PCI-DSS Mapping**:
- Req 1: Cloud Armor, Firewall rules, VPC-SC
- Req 2: OS hardening, CIS benchmarks
- Req 3: CMEK, tokenization, no plaintext storage
- Req 4: TLS 1.2+, VPN for management
- Req 6: Vulnerability scanning, patch management
- Req 7-8: IAM, MFA, audit logging
- Req 9: Physical security (Google data centers)
- Req 10: Comprehensive audit logging
- Req 11: Quarterly scans, annual pen tests
- Req 12: Security policies, incident response

### Scenario 2: Healthcare Platform with Multi-Tenancy (HIPAA)

**Requirements**:
- Multi-tenant SaaS for hospitals
- HIPAA compliance (BAA required)
- Patient data isolation per tenant
- 6-year audit retention
- 99.95% uptime SLA
- Mobile app access

**Architecture Solution**:
```
Client Access
├── Identity-Aware Proxy (zero trust access)
├── BeyondCorp Enterprise (context-aware)
└── Certificate-based device authentication
    ↓
Application Layer
├── Cloud Run (containerized app)
│   ├── Workload Identity
│   ├── Per-tenant service accounts
│   └── mTLS between services
├── API Gateway (rate limiting, authentication)
└── Private Service Connect (no internet exposure)
    ↓
Data Layer (Per-Tenant Isolation)
├── Firestore Native (patient records)
│   ├── CMEK per tenant
│   ├── Collection-level security rules
│   └── Automatic backups
├── Cloud Storage (medical images)
│   ├── CMEK per tenant
│   ├── Uniform bucket-level access
│   ├── Object versioning enabled
│   └── DLP scanning (PHI detection)
├── BigQuery (analytics)
│   ├── Separate datasets per tenant
│   ├── Column-level security
│   ├── CMEK encryption
│   └── Authorized views (limited access)
└── VPC Service Controls Perimeter
    ├── All healthcare projects
    ├── PHI-eligible services only
    └── Access levels (device policy + IP)
    ↓
Security and Compliance
├── DLP API (PHI detection in transit)
│   ├── De-identification templates
│   ├── Inspection job triggers
│   └── Findings to Security Command Center
├── Cloud Audit Logs
│   ├── Data Access logs enabled (all services)
│   ├── Export to immutable bucket (6-year retention)
│   └── BigQuery export for analysis
├── Security Command Center Premium
│   ├── Container Threat Detection
│   ├── Event Threat Detection
│   └── Web Security Scanner
└── Monitoring
    ├── Uptime checks (SLA monitoring)
    ├── Alerting on PHI access anomalies
    └── Real-time security dashboards
```

**Tenant Isolation Strategy**:
1. Separate service accounts per tenant
2. IAM conditions (resource.labels.tenant_id)
3. CMEK keys per tenant (separate key rings)
4. VPC-SC access levels per tenant group
5. Separate Firestore collections
6. BigQuery authorized views

**HIPAA Controls**:
- Access Control: IAP, MFA, least privilege, time-based access
- Audit: All log types, 6-year retention, immutable storage
- Integrity: Checksums, versioning, DLP monitoring
- Transmission: TLS 1.2+, mTLS between services
- Encryption: CMEK for all PHI, key rotation

### Scenario 3: Financial Services with Zero Trust Architecture

**Requirements**:
- Zero trust security (BeyondCorp)
- Separation of duties (SOX compliance)
- 4-eyes principle for sensitive operations
- Real-time fraud detection
- Integration with on-premises systems

**Architecture Solution**:
```
Identity and Access
├── Cloud Identity Premium
│   ├── MFA enforcement (hardware keys)
│   ├── Context-aware access policies
│   └── Device management
├── Identity-Aware Proxy
│   ├── Per-application access policies
│   ├── Device trust verification
│   └── Location-based restrictions
├── Access Context Manager
│   ├── Device attributes (OS, encryption, corp-managed)
│   ├── IP address ranges (corporate network)
│   └── Geographic location
└── Access Approval
    ├── 4-eyes principle enforcement
    ├── Break-glass access logging
    └── Time-limited approvals
    ↓
Application Environment
├── GKE Private Cluster
│   ├── No public endpoints
│   ├── Workload Identity
│   ├── Binary Authorization (signed images only)
│   ├── Network Policies (pod-to-pod restrictions)
│   └── Service mesh (mTLS, fine-grained authz)
├── Private Service Connect
│   ├── Access Google APIs via private endpoints
│   └── No internet connectivity required
└── Cloud SQL
    ├── Private IP only
    ├── CMEK encryption
    ├── IAM database authentication
    └── Query insights (anomaly detection)
    ↓
Network Security
├── VPC Service Controls
│   ├── Perimeter around sensitive projects
│   ├── Ingress/Egress rules (strict)
│   ├── Access levels (device + IP + identity)
│   └── Dry run mode for testing
├── Cloud Interconnect (on-premises connectivity)
│   ├── 99.99% SLA (redundant connections)
│   ├── MACsec encryption
│   └── Cloud Router (BGP)
├── Hierarchical Firewall Policies
│   ├── Organization: Deny SSH from internet
│   ├── Folder: Allow bastion host access
│   └── VPC: Application-specific rules
└── Cloud NAT (for outbound only)
    ↓
Fraud Detection and Monitoring
├── Real-time Transaction Analysis
│   ├── Pub/Sub (transaction stream)
│   ├── Dataflow (streaming analysis)
│   ├── BigQuery ML (fraud models)
│   └── AutoML Tables (anomaly detection)
├── Security Operations
│   ├── Security Command Center Premium
│   ├── Event Threat Detection (privilege escalation)
│   ├── Container Threat Detection (runtime threats)
│   └── Continuous exports (SIEM integration)
└── Compliance Monitoring
    ├── Forseti Security (policy enforcement)
    ├── Config Connector (IaC validation)
    └── Policy Intelligence (IAM recommender)
```

**Zero Trust Controls**:
1. Never trust, always verify (authenticate every request)
2. Device trust verification (managed devices only)
3. Context-aware access (IP, location, device, user)
4. Least privilege access (JIT, time-limited)
5. Micro-segmentation (service mesh, network policies)
6. Continuous verification (session validation)
7. Assume breach mindset (defense in depth)
8. Comprehensive logging (all access monitored)

**Separation of Duties**:
- Network Admin: Cannot modify IAM
- Security Admin: Cannot modify network
- Audit Viewer: Read-only, cannot modify
- Application Developer: Cannot access production
- SRE: Can deploy, cannot access data
- DBA: Can manage databases, cannot read sensitive data

### Scenario 4: Global SaaS with Data Residency Requirements

**Requirements**:
- Serve customers in US, EU, APAC
- Data residency (GDPR, data sovereignty)
- Low latency (<100ms globally)
- Centralized management
- Cost optimization

**Architecture Solution**:
```
Global Edge
├── Cloud CDN (static content, 200+ edge locations)
├── Cloud Armor (DDoS protection, geo-based routing)
└── Global HTTPS Load Balancer
    ↓
Regional Deployments (US, EU, APAC)
├── US Region (us-central1)
│   ├── GKE Autopilot (application)
│   ├── Cloud SQL (customer data)
│   ├── Cloud Storage (us-multi-region)
│   └── Cloud KMS (us-central1)
├── EU Region (europe-west1)
│   ├── GKE Autopilot (application)
│   ├── Cloud SQL (customer data)
│   ├── Cloud Storage (eu-multi-region)
│   └── Cloud KMS (europe-west1)
└── APAC Region (asia-east1)
    ├── GKE Autopilot (application)
    ├── Cloud SQL (customer data)
    ├── Cloud Storage (asia-multi-region)
    └── Cloud KMS (asia-east1)
    ↓
Data Residency Controls
├── Organization Policy
│   ├── constraints/gcp.resourceLocations (per folder)
│   ├── US Folder: allowed_values: in:us-locations
│   ├── EU Folder: allowed_values: in:eu-locations
│   └── APAC Folder: allowed_values: in:asia-locations
├── Customer Data Classification
│   ├── Labels (region, data_classification)
│   ├── Regional CMEK keys
│   └── Separate projects per region
├── Cross-Region Restrictions
│   ├── VPC Service Controls (regional perimeters)
│   ├── No data transfer between regions
│   └── Separate service accounts per region
└── Compliance Validation
    ├── Asset Inventory (resource location tracking)
    ├── Automated compliance checks
    └── Policy Denied logs (violations)
    ↓
Centralized Management
├── Shared Services (global)
│   ├── Identity (Cloud Identity)
│   ├── CI/CD (Cloud Build)
│   ├── Monitoring (Cloud Monitoring)
│   └── Logging (Cloud Logging, exported regionally)
├── Infrastructure as Code
│   ├── Terraform (region-specific modules)
│   ├── Config Connector (K8s-native)
│   └── Policy Controller (compliance checks)
└── Centralized Security
    ├── Security Command Center (organization-level)
    ├── Centralized audit logging
    └── Single pane of glass monitoring
```

**Data Residency Implementation**:
1. Organizational structure: Separate folders per region
2. Organization policies: Enforce regional constraints
3. Regional CMEK: Keys never leave region
4. Network isolation: Regional VPCs, no peering
5. Customer routing: Load balancer with geo-aware routing
6. Compliance validation: Automated checks, violations blocked

**GDPR Compliance**:
- Data residency: EU-only regions for EU customers
- Right to access: Export APIs per customer
- Right to erasure: Deletion workflows
- Right to portability: Data export in structured format
- Breach notification: <72 hour detection and reporting
- Privacy by design: DLP, pseudonymization, minimization

### Scenario 5: Serverless Microservices with Comprehensive Security

**Requirements**:
- Event-driven architecture
- Auto-scaling (0 to millions of requests)
- Secrets management
- Service-to-service authentication
- Observability and tracing

**Architecture Solution**:
```
Ingress
├── API Gateway (managed)
│   ├── OpenAPI specification
│   ├── JWT validation
│   ├── Rate limiting
│   └── Request transformation
├── Cloud Armor (backend security policy)
└── Service mesh (optional, for Cloud Run)
    ↓
Compute Layer
├── Cloud Run Services (15+ microservices)
│   ├── Private ingress (VPC-only)
│   ├── Service-specific service accounts
│   ├── IAM-based service-to-service auth
│   ├── CMEK for runtime environment
│   ├── Binary Authorization (signed containers)
│   └── VPC Connector (private network access)
├── Cloud Functions (event handlers)
│   ├── Triggered by Pub/Sub, Cloud Storage
│   ├── Service accounts per function
│   ├── VPC Connector (private access)
│   └── Secret Manager integration
└── Workflows (orchestration)
    ├── Service account for each workflow
    └── IAM-based service invocation
    ↓
Data Layer
├── Firestore (NoSQL, application data)
│   ├── Security rules (fine-grained)
│   ├── CMEK encryption
│   └── Point-in-time recovery
├── Cloud SQL (relational data)
│   ├── Private IP only
│   ├── Cloud SQL Proxy (connections)
│   ├── IAM database authentication
│   └── CMEK encryption
├── Cloud Storage (object storage)
│   ├── Uniform bucket-level access
│   ├── CMEK encryption
│   ├── Signed URLs (temporary access)
│   └── Bucket Lock (immutability)
└── Memorystore (Redis, caching)
    ├── Private IP only
    ├── In-transit encryption
    └── AUTH enabled
    ↓
Secrets and Configuration
├── Secret Manager
│   ├── Separate secret per service/environment
│   ├── Automatic replication (regional)
│   ├── Secret rotation (automated)
│   ├── Audit logging (all access)
│   └── IAM-based access control
├── Cloud KMS (encryption keys)
│   ├── Key hierarchy (per service)
│   ├── Automatic rotation (90 days)
│   └── Hardware security module (HSM)
└── Runtime Config (feature flags)
    ↓
Observability
├── Cloud Trace (distributed tracing)
│   ├── Automatic for Cloud Run
│   ├── Custom spans in code
│   └── Latency analysis
├── Cloud Logging
│   ├── Structured logging (JSON)
│   ├── Correlation IDs (request tracking)
│   ├── Error Reporting integration
│   └── Log-based metrics
├── Cloud Monitoring
│   ├── SLO/SLI tracking
│   ├── Custom metrics (business KPIs)
│   ├── Uptime checks (endpoint monitoring)
│   └── Alert policies (anomaly detection)
└── Cloud Profiler (performance analysis)
```

**Security Controls**:
1. Service-to-service auth: IAM-based (invoker role)
2. Secrets management: Never in code or env vars
3. Network security: Private ingress, VPC connectivity
4. Identity: Service accounts per service (least privilege)
5. Container security: Binary Authorization, vulnerability scanning
6. Data encryption: CMEK for data at rest
7. Audit: All service invocations logged
8. Zero trust: No implicit trust between services

### Scenario 6: Hybrid Cloud with Secure Connectivity

**Requirements**:
- On-premises datacenter integration
- Active Directory authentication
- Lift-and-shift VMs (Windows, Linux)
- Secure, high-bandwidth connectivity
- Disaster recovery in cloud

**Architecture Solution**:
```
Connectivity Options
├── Cloud Interconnect (Dedicated or Partner)
│   ├── 10 Gbps or 100 Gbps
│   ├── 99.99% SLA
│   ├── MACsec encryption (optional)
│   ├── VLAN attachments
│   └── Cloud Router (BGP)
├── Cloud VPN (HA VPN)
│   ├── 99.99% SLA (with multiple tunnels)
│   ├── IPsec encryption
│   ├── BGP dynamic routing
│   └── Backup for Interconnect
└── Private Service Connect (Google APIs access)
    ↓
Identity Integration
├── Managed Microsoft AD
│   ├── Automatic patching
│   ├── Multi-region replication
│   ├── Trust relationship with on-prem AD
│   └── LDAPS for secure communication
├── Cloud Identity Sync (G Suite/Cloud Identity)
│   ├── One-way sync from on-prem AD
│   ├── User and group provisioning
│   └── Password sync optional
└── Hybrid Join (managed devices)
    ├── Device objects in both ADs
    └── SSO experience
    ↓
Compute Layer
├── Compute Engine (lift-and-shift VMs)
│   ├── Active Directory joined
│   ├── OS Login (optional, for Linux)
│   ├── Shielded VMs (secure boot, vTPM)
│   ├── Confidential VMs (memory encryption)
│   ├── Custom images (security hardened)
│   └── Instance groups (auto-healing)
├── Sole-Tenant Nodes (physical isolation)
│   ├── Compliance requirements
│   ├── License portability (BYOL)
│   └── Affinity rules
└── Cloud SQL (managed databases)
    ├── Private IP only
    ├── AD authentication integration
    └── Automated backups (cross-region)
    ↓
Network Architecture
├── Shared VPC
│   ├── Host project (network admins)
│   ├── Service projects (application teams)
│   └── Centralized firewall management
├── Custom Subnets
│   ├── Non-overlapping with on-prem (10.0.0.0/8)
│   ├── Private Google Access enabled
│   └── VPC Flow Logs (monitoring)
├── Cloud NAT (outbound internet)
│   ├── Static IPs for allow listing
│   └── Logging enabled
└── Firewall Rules
    ├── Hierarchical policies (organization-level)
    ├── Service account-based rules
    └── Deny-all default policy
    ↓
Disaster Recovery
├── Backup Strategy
│   ├── Persistent disk snapshots (automated)
│   ├── Cross-region snapshot replication
│   ├── Cloud Storage (long-term retention)
│   └── Export on-prem backups to GCS
├── DR Site (Warm Standby)
│   ├── Minimal instances running
│   ├── Instance templates (quick scale-up)
│   ├── Load balancer ready
│   └── Database replicas (read replicas)
└── Failover Automation
    ├── Health checks (on-prem monitoring)
    ├── DNS failover (Cloud DNS)
    ├── Runbooks (Cloud Build triggers)
    └── Regular DR testing
    ↓
Security and Compliance
├── VPC Service Controls
│   ├── Perimeter around cloud resources
│   ├── Access from on-prem (ingress rule)
│   └── Prevent data exfiltration
├── Cloud Armor
│   ├── Protect internet-facing resources
│   ├── IP allowlist (known sources)
│   └── DDoS protection
├── Security Command Center
│   ├── Hybrid asset discovery
│   ├── Vulnerability detection
│   └── Compliance monitoring
└── Unified Logging
    ├── Centralized log aggregation
    ├── On-prem logs to Cloud Logging
    └── SIEM integration (Splunk, QRadar)
```

**Hybrid Security Best Practices**:
1. Defense in depth: Multiple security layers
2. Least privilege: Separate service accounts for on-prem access
3. Network segmentation: Isolated VPCs, restricted connectivity
4. Identity federation: Single sign-on (SSO)
5. Encryption: In-transit (VPN/Interconnect), at-rest (CMEK)
6. Monitoring: Unified logging and alerting
7. Compliance: Same standards on-prem and cloud
8. Disaster recovery: Regular testing, documented procedures

## Security-Focused gcloud Commands

### IAM and Security
```bash
# List all IAM bindings for a project
gcloud projects get-iam-policy PROJECT_ID --flatten="bindings[].members" --format="table(bindings.role,bindings.members)"

# Add IAM binding with condition
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:admin@example.com \
  --role=roles/compute.instanceAdmin.v1 \
  --condition='expression=request.time < timestamp("2025-12-31T00:00:00Z"),title=Temporary Access'

# List service accounts
gcloud iam service-accounts list --project=PROJECT_ID

# Create service account
gcloud iam service-accounts create SA_NAME \
  --display-name="Application Service Account" \
  --description="Service account for app"

# Delete service account keys (security best practice)
gcloud iam service-accounts keys list --iam-account=SA_EMAIL
gcloud iam service-accounts keys delete KEY_ID --iam-account=SA_EMAIL

# Impersonate service account (for testing)
gcloud compute instances list \
  --impersonate-service-account=SA_EMAIL

# Test IAM permissions
gcloud projects get-iam-policy PROJECT_ID --flatten="bindings[].members" --filter="bindings.members:user:admin@example.com"
```

### Organization Policies
```bash
# List organization policies
gcloud resource-manager org-policies list --organization=ORG_ID

# Describe specific constraint
gcloud resource-manager org-policies describe constraints/compute.vmExternalIpAccess --organization=ORG_ID

# Set organization policy (restrict VM external IPs)
gcloud resource-manager org-policies set-policy policy.yaml --organization=ORG_ID

# Example policy.yaml for external IP restriction:
cat > policy.yaml <<EOF
constraint: constraints/compute.vmExternalIpAccess
listPolicy:
  deniedValues:
  - "*"
EOF

# Delete organization policy (revert to default)
gcloud resource-manager org-policies delete constraints/compute.vmExternalIpAccess --organization=ORG_ID
```

### Cloud KMS
```bash
# Create key ring
gcloud kms keyrings create KEYRING_NAME --location=us-central1

# Create encryption key with rotation
gcloud kms keys create KEY_NAME \
  --keyring=KEYRING_NAME \
  --location=us-central1 \
  --purpose=encryption \
  --rotation-period=90d \
  --next-rotation-time=2025-12-31T00:00:00Z

# Encrypt a file
gcloud kms encrypt \
  --key=KEY_NAME \
  --keyring=KEYRING_NAME \
  --location=us-central1 \
  --plaintext-file=secret.txt \
  --ciphertext-file=secret.enc

# Decrypt a file
gcloud kms decrypt \
  --key=KEY_NAME \
  --keyring=KEYRING_NAME \
  --location=us-central1 \
  --ciphertext-file=secret.enc \
  --plaintext-file=secret.txt

# Grant permission to use key
gcloud kms keys add-iam-policy-binding KEY_NAME \
  --keyring=KEYRING_NAME \
  --location=us-central1 \
  --member=serviceAccount:SA_EMAIL \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# List key versions
gcloud kms keys versions list --key=KEY_NAME --keyring=KEYRING_NAME --location=us-central1
```

### Secret Manager
```bash
# Create secret
gcloud secrets create SECRET_NAME --replication-policy=automatic

# Add secret version
echo -n "my-secret-value" | gcloud secrets versions add SECRET_NAME --data-file=-

# Access latest secret version
gcloud secrets versions access latest --secret=SECRET_NAME

# Grant access to service account
gcloud secrets add-iam-policy-binding SECRET_NAME \
  --member=serviceAccount:SA_EMAIL \
  --role=roles/secretmanager.secretAccessor

# List secrets
gcloud secrets list

# Destroy secret version (cannot be undone)
gcloud secrets versions destroy 1 --secret=SECRET_NAME
```

### VPC Service Controls
```bash
# Create access policy
gcloud access-context-manager policies create --organization=ORG_ID --title="My Access Policy"

# Create access level
gcloud access-context-manager levels create LEVEL_NAME \
  --title="Corporate Network" \
  --basic-level-spec=access-level.yaml \
  --policy=POLICY_ID

# Create service perimeter
gcloud access-context-manager perimeters create PERIMETER_NAME \
  --title="PCI Environment" \
  --resources=projects/PROJECT_NUMBER \
  --restricted-services=storage.googleapis.com,bigquery.googleapis.com \
  --access-levels=LEVEL_NAME \
  --policy=POLICY_ID

# Update perimeter (add ingress rule)
gcloud access-context-manager perimeters update PERIMETER_NAME \
  --set-ingress-policies=ingress-policy.yaml \
  --policy=POLICY_ID

# Dry run mode (test without enforcing)
gcloud access-context-manager perimeters update PERIMETER_NAME \
  --perimeter-type=dry-run \
  --policy=POLICY_ID
```

### Security Command Center
```bash
# List findings
gcloud scc findings list ORG_ID \
  --source=SOURCE_ID \
  --filter="state=\"ACTIVE\" AND severity=\"HIGH\""

# Create notification config
gcloud scc notifications create NOTIFICATION_ID \
  --organization=ORG_ID \
  --description="High severity alerts" \
  --pubsub-topic=projects/PROJECT/topics/scc-alerts \
  --filter="severity=\"HIGH\""

# Update finding state
gcloud scc findings update FINDING_NAME \
  --source=SOURCE_ID \
  --state=INACTIVE \
  --organization=ORG_ID

# List assets
gcloud scc assets list ORG_ID \
  --filter="securityCenterProperties.resourceType=\"google.compute.Instance\""
```

### Cloud Audit Logs
```bash
# Read audit logs (last hour)
gcloud logging read \
  'logName:"cloudaudit.googleapis.com" AND timestamp>="2025-01-15T10:00:00Z"' \
  --limit=50 \
  --format=json

# Filter by specific user
gcloud logging read \
  'protoPayload.authenticationInfo.principalEmail="user@example.com"' \
  --limit=10

# Filter by method (IAM changes)
gcloud logging read \
  'protoPayload.methodName="SetIamPolicy"' \
  --limit=10

# Export logs to BigQuery
gcloud logging sinks create audit-sink \
  bigquery.googleapis.com/projects/PROJECT/datasets/audit_logs \
  --log-filter='logName:"cloudaudit.googleapis.com"'

# Create log-based metric
gcloud logging metrics create failed_auth \
  --description="Failed authentication attempts" \
  --log-filter='protoPayload.status.code!=0 AND protoPayload.methodName=~".*login.*"'
```

### Firewall Rules
```bash
# List firewall rules
gcloud compute firewall-rules list --sort-by=PRIORITY

# Create deny rule (highest priority)
gcloud compute firewall-rules create deny-all-ssh \
  --network=VPC_NAME \
  --action=DENY \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --priority=100

# Create allow rule with service account target
gcloud compute firewall-rules create allow-app-to-db \
  --network=VPC_NAME \
  --action=ALLOW \
  --rules=tcp:5432 \
  --source-service-accounts=app-sa@PROJECT.iam.gserviceaccount.com \
  --target-service-accounts=db-sa@PROJECT.iam.gserviceaccount.com

# Enable logging on firewall rule
gcloud compute firewall-rules update RULE_NAME \
  --enable-logging \
  --logging-metadata=include-all

# Create hierarchical firewall policy
gcloud compute firewall-policies create POLICY_NAME \
  --organization=ORG_ID \
  --description="Organization-wide firewall policy"

# Add rule to hierarchical policy
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=POLICY_NAME \
  --action=deny \
  --direction=INGRESS \
  --src-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:22 \
  --organization=ORG_ID
```

### Cloud Armor
```bash
# Create security policy
gcloud compute security-policies create POLICY_NAME \
  --description="WAF and DDoS protection"

# Add rule to block SQL injection
gcloud compute security-policies rules create 1000 \
  --security-policy=POLICY_NAME \
  --expression="evaluatePreconfiguredExpr('sqli-stable')" \
  --action=deny-403

# Add rate limiting rule
gcloud compute security-policies rules create 2000 \
  --security-policy=POLICY_NAME \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600

# Attach policy to backend service
gcloud compute backend-services update BACKEND_NAME \
  --security-policy=POLICY_NAME \
  --global

# Enable adaptive protection
gcloud compute security-policies update POLICY_NAME \
  --enable-layer7-ddos-defense \
  --layer7-ddos-defense-rule-visibility=STANDARD
```

## Professional-Level Exam Tips

### Security Design Decision Framework

**When to Use CMEK vs Default Encryption**:
- **CMEK**: Compliance requirement (PCI-DSS, HIPAA), need key control, regulatory audit requirement, key access revocation
- **Default**: Cost-sensitive, standard workloads, no compliance requirement, operational simplicity

**VPC Service Controls vs Shared VPC vs VPC Peering**:
- **VPC Service Controls**: API-level security perimeter, compliance boundaries (HIPAA, PCI-DSS), data exfiltration prevention
- **Shared VPC**: Centralized network management, consistent firewall policies, cost optimization (shared NAT/VPN)
- **VPC Peering**: Simple network connectivity, transitive peering not required, no centralized control needed

**Service Account Keys vs Workload Identity**:
- **Workload Identity**: Always preferred for GKE, no key rotation burden, automatic credential management
- **Service Account Keys**: Legacy systems, external systems without Workload Identity Federation, short-lived use only

**Cloud Armor vs Cloud IDS**:
- **Cloud Armor**: Application-layer (L7) protection, DDoS mitigation, WAF rules, attached to load balancer
- **Cloud IDS**: Network-layer threat detection, IDS/IPS functionality, detects lateral movement, requires packet mirroring

### Common Exam Mistake Patterns

1. **Choosing Cloud Armor for internal traffic**: Cloud Armor only works with Cloud Load Balancing (internet-facing or internal HTTPS LB)

2. **Forgetting VPC-SC applies to APIs only**: VPC Service Controls protect Google Cloud APIs, not network traffic (use firewall rules for that)

3. **Assuming all services are HIPAA-eligible**: Always check the BAA-eligible services list (e.g., App Engine Standard is NOT eligible)

4. **Over-engineering security**: "Most secure" doesn't always mean most complex - sometimes IAP alone is sufficient

5. **Forgetting organization policies are inherited**: Child resources cannot override parent policies (policy is union of all)

6. **Assuming CMEK protects against insider threats**: CMEK protects from Google access, not from users with proper IAM permissions

7. **Using service account keys when Workload Identity available**: Exam strongly favors Workload Identity for GKE workloads

8. **Forgetting data residency vs sovereignty**: Residency = physical location, Sovereignty = legal jurisdiction (GDPR needs both)

### Question Pattern Recognition

**Pattern**: "Most secure way to..."
- Look for VPC Service Controls + CMEK + Private connectivity combination
- Prefer Workload Identity over service account keys
- Choose IAP over VPN for user access

**Pattern**: "Compliance with HIPAA/PCI-DSS requires..."
- Verify service is BAA-eligible (HIPAA) or suitable for CDE (PCI-DSS)
- CMEK is usually required
- Audit logging (all types) is mandatory
- VPC Service Controls for perimeter protection

**Pattern**: "Minimize operational overhead while..."
- Prefer managed services (Cloud Run over GKE, Cloud SQL over self-managed)
- Use default encryption unless compliance requires CMEK
- Workload Identity over service account key management
- Organization policies over manual enforcement

**Pattern**: "Zero trust architecture..."
- Identity-Aware Proxy (IAP) is key component
- BeyondCorp Enterprise
- Context-aware access (device, location, user)
- No implicit trust (authenticate every request)
- Service accounts with least privilege

**Pattern**: "Detect and respond to security incidents..."
- Security Command Center for detection
- Cloud Audit Logs for investigation
- Pub/Sub + Cloud Functions for automated response
- Immutable log storage for forensics

### Key Numbers to Remember

- **Audit log retention**: 400 days (Admin Activity, System Events)
- **Organization policy inheritance**: Cannot be overridden by child resources
- **VPC Service Controls dry run**: Test before enforcing
- **Cloud Armor rule priority**: Lower number = higher priority
- **Firewall rule priority**: Lower number = higher priority (range: 0-65535)
- **Key rotation**: 90 days recommended for CMEK
- **Service account key expiration**: No automatic expiration (manual rotation required)
- **GDPR breach notification**: 72 hours
- **HIPAA audit retention**: 6 years minimum
- **PCI-DSS audit retention**: 1 year minimum, 3 months immediately available

### Critical Security Concepts

1. **Defense in Depth**: Multiple layers of security controls (perimeter, network, application, data)
2. **Least Privilege**: Minimal permissions required (start restrictive, expand as needed)
3. **Separation of Duties**: No single user with complete control (network admin ≠ security admin)
4. **Zero Trust**: Never trust, always verify (authenticate and authorize every request)
5. **Immutability**: Audit logs, compliance data (retention lock on Cloud Storage)
6. **Encryption Hierarchy**: Application → Service → Infrastructure layers
7. **Service Perimeter**: VPC Service Controls create API-level boundaries
8. **Workload Identity**: Kubernetes-native way to access Google Cloud APIs

### Final Study Checklist

- [ ] Understand VPC Service Controls (access levels, ingress/egress policies, dry run mode)
- [ ] Know CMEK vs CSEK vs default encryption (when to use each)
- [ ] Master IAM conditions (time-based, resource-based, attribute-based)
- [ ] Understand compliance mappings (PCI-DSS, HIPAA, GDPR, FedRAMP, SOC 2, ISO 27001)
- [ ] Know Cloud Armor capabilities (WAF rules, rate limiting, adaptive protection)
- [ ] Understand Security Command Center tiers and detection capabilities
- [ ] Master service account patterns (Workload Identity, impersonation, keys)
- [ ] Know organization policy constraints and inheritance model
- [ ] Understand audit log types and retention
- [ ] Practice incident response scenarios
- [ ] Know when to use Cloud IDS vs Cloud Armor
- [ ] Understand data residency vs data sovereignty
- [ ] Master Secret Manager integration patterns
- [ ] Know firewall rule hierarchy (organization, folder, VPC)
- [ ] Understand zero trust architecture components

## Additional Resources

### Official Google Cloud Documentation
- [Security Best Practices](https://cloud.google.com/security/best-practices)
- [Compliance Resources](https://cloud.google.com/security/compliance)
- [Cloud Security Command Center](https://cloud.google.com/security-command-center)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls)
- [BeyondCorp Enterprise](https://cloud.google.com/beyondcorp-enterprise)
- [IAM Best Practices](https://cloud.google.com/iam/docs/using-iam-securely)
- [Cloud KMS Documentation](https://cloud.google.com/kms/docs)
- [Secret Manager](https://cloud.google.com/secret-manager/docs)
- [Cloud Armor](https://cloud.google.com/armor/docs)
- [Data Loss Prevention](https://cloud.google.com/dlp/docs)

### Compliance and Certifications
- [PCI-DSS on GCP](https://cloud.google.com/security/compliance/pci-dss)
- [HIPAA Compliance](https://cloud.google.com/security/compliance/hipaa)
- [GDPR Resource Center](https://cloud.google.com/privacy/gdpr)
- [FedRAMP Authorization](https://cloud.google.com/security/compliance/fedramp)
- [ISO/IEC 27001](https://cloud.google.com/security/compliance/iso-27001)
- [SOC 2 Reports](https://cloud.google.com/security/compliance/soc-2)

### Security Architecture Guides
- [Google Cloud Security Foundations Guide](https://cloud.google.com/architecture/security-foundations)
- [Enterprise Security Blueprint](https://cloud.google.com/solutions/blueprints/enterprise-security-blueprint)
- [Zero Trust Architecture](https://cloud.google.com/beyondcorp-enterprise/docs/design-guide)
- [Shared Responsibility Model](https://cloud.google.com/architecture/framework/security/shared-responsibility-shared-fate)

### Professional Cloud Architect Exam
- [Exam Guide](https://cloud.google.com/certification/guides/professional-cloud-architect)
- [Sample Questions](https://cloud.google.com/certification/sample-questions/cloud-architect)
- [Practice Exam](https://cloud.google.com/certification/practice-exam/cloud-architect)

### Additional Study Materials
- Google Cloud Architecture Framework - Security, Privacy, and Compliance
- Cloud Security Podcast by Google Cloud
- Google Cloud Security Whitepapers
- Case studies on cloud security implementations
