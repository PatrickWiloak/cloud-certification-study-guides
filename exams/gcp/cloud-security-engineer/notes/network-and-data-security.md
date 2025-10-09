# Network and Data Security - GCP Professional Cloud Security Engineer

## Overview

Network security architecture, data protection, encryption, DLP, and security controls for protecting GCP resources and data.

## Network Security

### VPC Security Controls
**Private Google Access**: Access Google APIs without external IPs
**VPC Service Controls**: Perimeter security for APIs
**Private Service Connect**: Private access to SaaS
**Shared VPC**: Centralized network management

**VPC Service Controls Example**:
```bash
# Create access policy
gcloud access-context-manager policies create --organization=ORG_ID --title="Corporate Policy"

# Create perimeter
gcloud access-context-manager perimeters create secure_perimeter \
  --policy=POLICY_ID \
  --resources=projects/PROJECT_NUMBER \
  --restricted-services=storage.googleapis.com,bigquery.googleapis.com \
  --access-levels=LEVEL_NAME
```

### Firewall Rules
**Hierarchy**:
1. Hierarchical firewall policies (org/folder level)
2. VPC firewall rules (network level)
3. Implicit deny all ingress
4. Implicit allow all egress

**Best Practices**:
```bash
# Deny-all baseline
gcloud compute firewall-rules create deny-all \
  --network=my-vpc \
  --action=deny \
  --rules=all \
  --priority=65534

# Allow specific traffic
gcloud compute firewall-rules create allow-ssh-from-iap \
  --network=my-vpc \
  --allow=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --target-service-accounts=bastion-sa@project.iam.gserviceaccount.com
```

### Cloud Armor
**DDoS Protection and WAF**:
```bash
# Create security policy
gcloud compute security-policies create my-policy

# Rate limiting rule
gcloud compute security-policies rules create 100 \
  --security-policy=my-policy \
  --expression="origin.region_code == 'CN'" \
  --action=deny-403

# OWASP rules
gcloud compute security-policies rules create 200 \
  --security-policy=my-policy \
  --expression="evaluatePreconfiguredExpr('sqli-v33-stable')" \
  --action=deny-403
```

### Identity-Aware Proxy (IAP)
**Zero Trust Access**:
```bash
# Enable IAP
gcloud iap web enable \
  --resource-type=backend-services \
  --service=SERVICE_NAME

# Grant access
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --service=SERVICE_NAME \
  --member=user:alice@example.com \
  --role=roles/iap.httpsResourceAccessor
```

## Data Protection

### Encryption
**Data at Rest**:
- Google-managed encryption keys (default)
- Customer-managed encryption keys (CMEK)
- Customer-supplied encryption keys (CSEK)

**CMEK Implementation**:
```bash
# Create key ring and key
gcloud kms keyrings create my-keyring --location=us-central1
gcloud kms keys create my-key \
  --keyring=my-keyring \
  --location=us-central1 \
  --purpose=encryption

# Grant service account access
gcloud kms keys add-iam-policy-binding my-key \
  --keyring=my-keyring \
  --location=us-central1 \
  --member=serviceAccount:service-PROJECT_NUMBER@compute-system.iam.gserviceaccount.com \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# Create encrypted disk
gcloud compute disks create encrypted-disk \
  --size=100GB \
  --kms-key=projects/PROJECT/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key
```

**Data in Transit**:
- TLS 1.2+ for external connections
- Google Front End (GFE) SSL/TLS termination
- BoringSSL for internal Google traffic
- VPN/Interconnect for hybrid

### Data Loss Prevention (DLP)
**Sensitive Data Discovery**:
```python
from google.cloud import dlp_v2

def inspect_content(project, content):
    dlp = dlp_v2.DlpServiceClient()

    inspect_config = {
        "info_types": [
            {"name": "EMAIL_ADDRESS"},
            {"name": "PHONE_NUMBER"},
            {"name": "CREDIT_CARD_NUMBER"},
            {"name": "US_SOCIAL_SECURITY_NUMBER"}
        ],
        "min_likelihood": dlp_v2.Likelihood.LIKELY,
        "limits": {"max_findings_per_request": 0}
    }

    item = {"value": content}
    parent = f"projects/{project}"

    response = dlp.inspect_content(
        request={"parent": parent, "inspect_config": inspect_config, "item": item}
    )

    return response.result.findings
```

**De-identification**:
```python
def deidentify_with_mask(project, content):
    dlp = dlp_v2.DlpServiceClient()

    deidentify_config = {
        "info_type_transformations": {
            "transformations": [
                {
                    "primitive_transformation": {
                        "character_mask_config": {
                            "masking_character": "*",
                            "number_to_mask": 0
                        }
                    }
                }
            ]
        }
    }

    response = dlp.deidentify_content(
        request={
            "parent": f"projects/{project}",
            "deidentify_config": deidentify_config,
            "item": {"value": content}
        }
    )

    return response.item.value
```

### Secret Management
**Secret Manager**:
```python
from google.cloud import secretmanager

def create_secret(project_id, secret_id, secret_value):
    client = secretmanager.SecretManagerServiceClient()
    parent = f"projects/{project_id}"

    # Create secret
    secret = client.create_secret(
        request={
            "parent": parent,
            "secret_id": secret_id,
            "secret": {"replication": {"automatic": {}}}
        }
    )

    # Add version
    version = client.add_secret_version(
        request={
            "parent": secret.name,
            "payload": {"data": secret_value.encode("UTF-8")}
        }
    )

    return version.name

def access_secret(project_id, secret_id, version_id="latest"):
    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/{project_id}/secrets/{secret_id}/versions/{version_id}"

    response = client.access_secret_version(request={"name": name})
    return response.payload.data.decode("UTF-8")
```

## Security Monitoring

### Security Command Center
**Asset Discovery and Vulnerability Scanning**:
```bash
# List findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\"" \
  --format="table(category, resourceName, eventTime)"

# Create notification
gcloud scc notifications create my-notification \
  --organization=ORG_ID \
  --pubsub-topic=projects/PROJECT/topics/scc-notifications \
  --filter="state=\"ACTIVE\" AND severity=\"HIGH\""
```

### Cloud Audit Logs
**Types**:
- Admin Activity: Free, always enabled
- Data Access: Optional, additional cost
- System Events: Free, always enabled
- Policy Denied: Free, always enabled

**Monitoring IAM Changes**:
```sql
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.methodName as action,
  protoPayload.resourceName as resource
FROM
  `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE
  protoPayload.serviceName = "iam.googleapis.com"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
ORDER BY timestamp DESC;
```

## Compliance Frameworks

### GDPR Implementation
**Requirements**:
1. Data residency (EU regions)
2. Data encryption (CMEK)
3. Access controls (IAM)
4. Audit logging (Cloud Audit Logs)
5. Data deletion (automated workflows)
6. DLP for PII detection

### HIPAA Implementation
**BAA-Eligible Services**:
- Compute Engine, GKE, Cloud Run
- Cloud Storage, Cloud SQL, BigQuery
- Cloud Pub/Sub, Dataflow
- Cloud KMS, Secret Manager

**Configuration**:
```bash
# Enable audit logs for Data Access
gcloud projects get-iam-policy PROJECT_ID > policy.yaml

# Edit policy to enable Data Access logs
# Apply updated policy
gcloud projects set-iam-policy PROJECT_ID policy.yaml

# Use CMEK
gcloud compute instances create hipaa-instance \
  --boot-disk-kms-key=projects/PROJECT/locations/LOCATION/keyRings/KEYRING/cryptoKeys/KEY
```

## Best Practices

### Network Security
1. Use VPC Service Controls for sensitive data
2. Implement private connectivity
3. Enable VPC Flow Logs
4. Use Cloud Armor for public services
5. IAP for internal applications
6. Network segmentation
7. Regular firewall audits

### Data Security
1. Encrypt sensitive data (CMEK)
2. Use DLP for discovery
3. Implement least privilege
4. Secret Manager for credentials
5. Regular data classification
6. Retention policies
7. Data masking in non-prod

### Monitoring
1. Enable all audit log types
2. Export logs to BigQuery
3. Real-time alerts for security events
4. Security Command Center
5. Regular security assessments
6. Incident response procedures

## Common Scenarios

**Scenario**: PCI DSS compliance for payment data
**Solution**: VPC Service Controls, CMEK, DLP, network segmentation, Cloud Armor, comprehensive audit logging

**Scenario**: Zero Trust architecture
**Solution**: IAP, BeyondCorp, context-aware access, Workload Identity, no external IPs, VPC Service Controls

**Scenario**: Multi-region data residency
**Solution**: Regional resources, organization policies restricting locations, CMEK with regional keys, DLP scanning

## Study Tips

1. Practice VPC Service Controls configuration
2. Understand encryption options (CMEK vs CSEK)
3. Implement DLP inspections
4. Configure Cloud Armor rules
5. Set up comprehensive audit logging
6. Know compliance requirements (GDPR, HIPAA, PCI)
7. Security monitoring with SCC

## Key Commands

```bash
# VPC Service Controls
gcloud access-context-manager perimeters create PERIMETER_NAME

# Cloud KMS
gcloud kms keys create KEY_NAME --keyring=KEYRING --location=LOCATION --purpose=encryption

# DLP (use client libraries)
# Cloud Armor
gcloud compute security-policies create POLICY_NAME

# IAP
gcloud iap web enable --resource-type=backend-services --service=SERVICE_NAME

# Audit Logs
gcloud logging read "protoPayload.serviceName=\"iam.googleapis.com\""
```

## Additional Resources

- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
- [Cloud KMS](https://cloud.google.com/kms/docs)
- [DLP Documentation](https://cloud.google.com/dlp/docs)
- [Security Command Center](https://cloud.google.com/security-command-center/docs)
- [Compliance Resources](https://cloud.google.com/security/compliance)
