# Identity and Access Management - GCP Professional Cloud Security Engineer

## Overview

IAM architecture, access control, service accounts, organization policies, and identity management for securing GCP resources.

## IAM Fundamentals

### IAM Model
**Components**:
- **Member**: Who (user, group, service account, domain)
- **Role**: What permissions (collection of permissions)
- **Resource**: Where (project, folder, organization, resource)

**Policy Binding**:
```json
{
  "bindings": [
    {
      "role": "roles/storage.objectViewer",
      "members": [
        "user:alice@example.com",
        "group:developers@example.com",
        "serviceAccount:sa@project.iam.gserviceaccount.com"
      ]
    }
  ]
}
```

### Role Types
**Basic Roles** (Avoid in production):
- Owner: Full access
- Editor: Modify resources
- Viewer: Read-only access

**Predefined Roles**:
- Curated by Google
- Service-specific
- Regularly updated
- Examples: `roles/storage.admin`, `roles/compute.instanceAdmin.v1`

**Custom Roles**:
```yaml
title: "Custom Network Admin"
description: "Manage networks and firewalls"
stage: "GA"
includedPermissions:
- compute.networks.*
- compute.firewalls.*
- compute.subnetworks.*
```

### IAM Best Practices
1. **Least Privilege**: Grant minimum necessary permissions
2. **Use Groups**: Assign roles to groups, not individuals
3. **Avoid Basic Roles**: Use predefined or custom roles
4. **Service Accounts**: For applications, not users
5. **Regular Reviews**: Audit IAM policies quarterly
6. **Organization Policies**: Enforce constraints
7. **Conditional Access**: Use IAM conditions for context-aware access

## Service Accounts

### Service Account Types
**User-Managed**:
- Created by you
- You control lifecycle
- Used for applications

**Google-Managed**:
- Automatically created
- Managed by Google services
- Example: App Engine default service account

### Best Practices
1. **Workload Identity**: Use for GKE instead of keys
2. **Avoid Key Creation**: Minimize service account keys
3. **Key Rotation**: Rotate keys every 90 days
4. **Dedicated Accounts**: One per application/service
5. **Naming Convention**: Descriptive names
6. **Access Scopes**: Use IAM, not access scopes
7. **Impersonation**: Use for elevated privileges

**Workload Identity Example**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    iam.gke.io/gcp-service-account: app-sa@project.iam.gserviceaccount.com
```

```bash
# Bind KSA to GSA
gcloud iam service-accounts add-iam-policy-binding \
  app-sa@project.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:project.svc.id.goog[namespace/my-app]"
```

## Organization Policies

### Common Constraints
**Resource Location**:
```yaml
constraint: constraints/gcp.resourceLocations
listPolicy:
  allowedValues:
    - in:us-locations
    - in:eu-locations
```

**VM External IP**:
```yaml
constraint: constraints/compute.vmExternalIpAccess
listPolicy:
  deniedValues:
    - "*"
```

**Service Account Key Creation**:
```yaml
constraint: constraints/iam.disableServiceAccountKeyCreation
booleanPolicy:
  enforced: true
```

### Policy Hierarchy
```
Organization
  └─ Folder (Development)
      └─ Project (dev-app-1)
  └─ Folder (Production)
      └─ Project (prod-app-1)
```

Policies inherit and merge down the hierarchy.

## Identity Federation

### Workforce Identity Federation
**Use Case**: Corporate identity provider (Okta, Azure AD)

**Configuration**:
```bash
# Create workload identity pool
gcloud iam workload-identity-pools create corporate-idp \
  --organization=ORG_ID \
  --location=global

# Create provider
gcloud iam workload-identity-pools providers create-oidc okta \
  --workload-identity-pool=corporate-idp \
  --issuer-uri=https://example.okta.com \
  --allowed-audiences=gcp-audience
```

### Workload Identity Federation
**Use Case**: External workloads (AWS, Azure, on-premises)

**Benefits**:
- No service account keys
- Short-lived credentials
- External identity provider

## Access Control Patterns

### Resource Hierarchy Access
```
Organization (Admin Group)
  └─ Folder: Production (Prod Admin Group)
      └─ Project: App1 (App1 Developers)
          └─ Resource: Cloud Storage Bucket (Data Analysts)
```

### Break-Glass Access
**Emergency Access Procedure**:
1. Dedicated emergency accounts
2. Strong authentication required
3. Monitoring and alerting
4. Regular access reviews
5. Clear activation process

**Implementation**:
```python
def request_break_glass_access(justification):
    # Log request
    log_access_request(user, justification)

    # Notify security team
    send_alert_to_security(user, justification)

    # Grant temporary access (2 hours)
    grant_temporary_role(user, "roles/owner", duration=7200)

    # Schedule automatic revocation
    schedule_access_revocation(user, 7200)
```

### Just-in-Time Access
**Temporary Privilege Elevation**:
- Time-bound access grants
- Approval workflow
- Automatic revocation
- Audit trail

## Security Best Practices

### IAM Security
1. Enable MFA for all users
2. Regular access reviews
3. Use managed groups
4. Implement least privilege
5. Monitor IAM changes
6. Avoid wildcard permissions
7. Use IAM conditions
8. Separate duties

### Monitoring and Auditing
```sql
-- BigQuery query for IAM changes
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.methodName,
  protoPayload.resourceName
FROM
  `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE
  protoPayload.serviceName = 'iam.googleapis.com'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY
  timestamp DESC;
```

## Common Scenarios

**Scenario**: Multi-tenant SaaS application
**Solution**: Service account per tenant, IAM conditions based on tenant ID, organization policies for isolation

**Scenario**: Developer access management
**Solution**: Groups for teams, predefined roles per environment, separate dev/prod access

**Scenario**: External contractor access
**Solution**: Time-bound access, specific role grants, monitoring, automatic revocation

## Study Tips

1. Understand IAM hierarchy and inheritance
2. Practice with custom roles
3. Know service account best practices
4. Implement Workload Identity
5. Configure organization policies
6. Monitor IAM changes with audit logs
7. Understand federation patterns

## Key Commands

```bash
# IAM Policy Management
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:alice@example.com \
  --role=roles/viewer

# Service Accounts
gcloud iam service-accounts create SA_NAME \
  --display-name="Service Account Display Name"

# Organization Policies
gcloud resource-manager org-policies set-policy policy.yaml \
  --organization=ORG_ID

# Audit IAM
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --format="table(bindings.role, bindings.members)"
```

## Additional Resources

- [IAM Documentation](https://cloud.google.com/iam/docs)
- [IAM Best Practices](https://cloud.google.com/iam/docs/best-practices)
- [Organization Policies](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
- [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)
