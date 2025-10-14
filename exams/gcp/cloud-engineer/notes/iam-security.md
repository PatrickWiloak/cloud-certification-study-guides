# IAM and Security - Cloud Engineer

**[📖 IAM Overview](https://cloud.google.com/iam/docs/overview)** - Identity and Access Management fundamentals

## Cloud Identity and Access Management (IAM)

### Core Concepts

**Members** (Who):
- Google Account (user)
- Service account
- Google group
- Google Workspace domain
- Cloud Identity domain
- All authenticated users (allAuthenticatedUsers)
- All users (allUsers)

**Roles** (What):
- Collection of permissions
- Permissions: service.resource.verb (e.g., compute.instances.create)

**Policy** (Binding):
- Binds members to roles on resources
- Resource hierarchy: Organization > Folder > Project > Resource
- Inheritance: Policies flow down hierarchy

### Role Types

**Primitive/Basic Roles** (Avoid in production):
- Owner: Full control including IAM
- Editor: Modify resources
- Viewer: Read-only access
- Too broad, not recommended

**Predefined Roles**:
- Google-managed, fine-grained
- Updated automatically by Google
- Examples: Compute Admin, Storage Object Viewer
- Recommended for most use cases

**Custom Roles**:
- User-defined permissions
- Project or organization level
- Use for specific requirements
- Not available for all services

**[📖 Understanding Roles](https://cloud.google.com/iam/docs/understanding-roles)** - IAM roles reference
**[📖 Custom Roles](https://cloud.google.com/iam/docs/creating-custom-roles)** - Create custom IAM roles

### Service Accounts
- Identity for applications/VMs
- Email format: NAME@PROJECT.iam.gserviceaccount.com
- Two types: User-managed, Google-managed
- No passwords, use keys or OAuth

**Best Practices**:
- One service account per service
- Grant minimal permissions
- Rotate keys regularly (if using keys)
- Use Workload Identity for GKE
- Avoid downloading keys

**[📖 Service Accounts](https://cloud.google.com/iam/docs/service-accounts)** - Service account documentation
**[📖 Service Account Best Practices](https://cloud.google.com/iam/docs/best-practices-service-accounts)** - Security best practices

**Service Account Keys**:
- JSON or P12 format
- User-managed (must rotate)
- Google-managed (automatic rotation)
- Stored securely in Secret Manager

### IAM Policy Structure
```json
{
  "bindings": [
    {
      "role": "roles/storage.objectViewer",
      "members": [
        "user:alice@example.com",
        "serviceAccount:my-sa@project.iam.gserviceaccount.com"
      ],
      "condition": {
        "expression": "resource.name.startsWith('projects/_/buckets/my-bucket')"
      }
    }
  ]
}
```

### IAM Conditions
- Fine-grained, conditional access
- Based on attributes: Time, resource, IP address
- Common Expression Language (CEL)
- Example: Grant access only during business hours

### Resource Hierarchy and Inheritance
```
Organization (policies apply to all)
  └─ Folder (policies apply to folder and below)
      └─ Project (policies apply to project resources)
          └─ Resource (resource-specific policies)
```

**Effective Policy**: Union of all policies in hierarchy
- Less permissive at higher levels
- More specific at lower levels
- Deny overrides allow (if using deny policies)

## Organization Resources

### Organization
- Root node of resource hierarchy
- Associated with Google Workspace or Cloud Identity domain
- Organization Policy Service
- Organization-level IAM roles

**Organization Roles**:
- Organization Administrator
- Billing Account Administrator
- Resource Manager roles

### Folders
- Group projects for department/team
- Apply policies to all projects in folder
- Nested folders supported
- Organizational structure

## Security Best Practices

### Least Privilege
- Grant minimum necessary permissions
- Use predefined roles over primitive
- Regular access reviews
- Remove unused permissions

### Service Account Management
1. Separate service accounts per application
2. Use Workload Identity (GKE)
3. Short-lived credentials when possible
4. Audit service account usage
5. Disable unused service accounts

### Access Controls
- Enable 2FA for user accounts
- Use groups instead of individual users
- Temporary access with conditions
- Regularly review and audit permissions

## Cloud Audit Logs

### Log Types

**Admin Activity Logs**:
- API calls that modify configuration
- Always enabled, free, 400-day retention
- Cannot be disabled
- Examples: Create VM, modify firewall rule

**Data Access Logs**:
- Read operations on user data
- Disabled by default (except BigQuery)
- Configure per service
- Can be expensive

**System Event Logs**:
- Google system events
- Always enabled, free
- Examples: VM live migration, automatic restart

**Policy Denied Logs**:
- Denied by IAM policy
- Helps troubleshoot access issues

### Viewing Audit Logs
- Logs Explorer in Cloud Console
- gcloud logging read
- Export to Cloud Storage, BigQuery, Pub/Sub
- Retention: 30-400 days depending on type

## Secret Manager
- Store API keys, passwords, certificates
- Encryption at rest and in transit
- Version management
- IAM-controlled access
- Audit logging
- Integration with Cloud Build, Cloud Run, etc.

**Best Practices**:
- Never hardcode secrets
- Use Secret Manager over environment variables
- Enable automatic replication
- Implement secret rotation
- Audit secret access

**[📖 Secret Manager](https://cloud.google.com/secret-manager/docs)** - Secure secret storage and management
**[📖 Secret Manager Best Practices](https://cloud.google.com/secret-manager/docs/best-practices)** - Security recommendations

## Key Management Service (KMS)

### Features
- Create and manage encryption keys
- Hardware Security Module (HSM) support
- Automatic key rotation
- Centralized key management

**[📖 Cloud KMS Documentation](https://cloud.google.com/kms/docs)** - Manage encryption keys
**[📖 Encryption at Rest](https://cloud.google.com/docs/security/encryption/default-encryption)** - Understanding encryption options

### Key Types
- Symmetric: Single key for encrypt/decrypt
- Asymmetric: Public/private key pair

### Protection Levels
- Software: Google-managed
- Hardware (HSM): FIPS 140-2 Level 3
- External: Customer manages keys externally

### Encryption Tiers
**Google-managed**: Default, no configuration
**Customer-managed (CMEK)**: Control key rotation, access
**Customer-supplied (CSEK)**: Full key control, customer managed

## VPC Service Controls

### Features
- Perimeter security for GCP resources
- Prevent data exfiltration
- Isolate resources within security perimeter
- Access levels based on conditions

**Service Perimeter**:
- Logical boundary around resources
- Allow resources within perimeter to communicate
- Block external access
- Ingress/egress rules

## Security Command Center (SCC)

### Features
- Centralized security and risk management
- Asset discovery and inventory
- Vulnerability scanning
- Threat detection
- Compliance monitoring

**Tiers**:
- Standard: Basic asset discovery (free)
- Premium: Advanced threat detection, compliance

**[📖 Security Command Center](https://cloud.google.com/security-command-center/docs)** - Centralized security management

## Binary Authorization
- Deploy-time security control
- Require signed container images
- Policy-based deployment
- Integration with GKE and Cloud Run

## Web Security Scanner
- Identify vulnerabilities in App Engine apps
- Automated scanning
- OWASP Top 10 detection
- Custom scans

## Exam Tips

### Common Scenarios
- **Grant access to user**: Add user to IAM policy with appropriate role
- **VM needs to access storage**: Attach service account with Storage roles
- **Temporary access**: IAM conditions with time-based expression
- **Audit who accessed data**: Enable Data Access logs
- **Store API keys securely**: Secret Manager
- **Encrypt with own keys**: CMEK (KMS)
- **Prevent data exfiltration**: VPC Service Controls
- **GKE pod authentication**: Workload Identity
- **Least privilege**: Use predefined roles, grant at lowest level

### Key Concepts
- IAM: Who (member) can do What (role) on Which resource
- Service accounts: Identity for applications
- Inheritance: Policies flow down from organization to resources
- Audit logs: Track all API activity
- Primitive roles: Avoid (Owner, Editor, Viewer)
- Predefined roles: Recommended (fine-grained)
- Custom roles: When predefined don't fit

### Important Commands
```bash
# View IAM policy
gcloud projects get-iam-policy PROJECT_ID

# Add IAM binding
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:alice@example.com \
  --role=roles/viewer

# Create service account
gcloud iam service-accounts create SA_NAME

# Grant role to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:SA_EMAIL \
  --role=roles/storage.objectViewer

# List audit logs
gcloud logging read "logName:cloudaudit.googleapis.com"
```

### Decision Matrix
- User access: Google Account + IAM role
- Application access: Service account + IAM role
- GKE pod access: Workload Identity
- Secrets: Secret Manager
- Encryption keys: KMS
- Compliance/security posture: Security Command Center
- Data perimeter: VPC Service Controls
