# Database Security and Compliance - GCP Professional Cloud Database Engineer

## Overview

Database security, access control, encryption, compliance, and security best practices for GCP database services.

## Access Control

### Cloud SQL IAM
**Database-level Access**:
```bash
# Grant Cloud SQL Client role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:dba@example.com \
  --role=roles/cloudsql.client

# Grant Cloud SQL Admin role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:admin@example.com \
  --role=roles/cloudsql.admin

# Cloud SQL IAM database authentication
gcloud sql users create user@example.com \
  --instance=INSTANCE \
  --type=CLOUD_IAM_USER

# Service account authentication
gcloud sql users create sa@project.iam.gserviceaccount.com \
  --instance=INSTANCE \
  --type=CLOUD_IAM_SERVICE_ACCOUNT
```

**Connection with IAM**:
```python
import sqlalchemy
from google.auth import default

# Get IAM credentials
credentials, project = default()

# Create engine with IAM auth
engine = sqlalchemy.create_engine(
    'postgresql+pg8000://',
    creator=lambda: connector.connect(
        "project:region:instance",
        "pg8000",
        user="user@example.com",
        password="",  # Empty for IAM
        db="database",
        enable_iam_auth=True,
    )
)
```

### Cloud Spanner IAM
```bash
# Grant database roles
gcloud spanner databases add-iam-policy-binding DATABASE \
  --instance=INSTANCE \
  --member=user:developer@example.com \
  --role=roles/spanner.databaseReader

gcloud spanner databases add-iam-policy-binding DATABASE \
  --instance=INSTANCE \
  --member=serviceAccount:app@project.iam.gserviceaccount.com \
  --role=roles/spanner.databaseUser

# Fine-grained access control with conditions
gcloud spanner databases add-iam-policy-binding DATABASE \
  --instance=INSTANCE \
  --member=user:analyst@example.com \
  --role=roles/spanner.databaseReader \
  --condition='expression=resource.name.startsWith("projects/PROJECT/instances/INSTANCE/databases/DATABASE"),title=read-only-access'
```

### Firestore Security Rules
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // User data access
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }

    // Shared data with field-level security
    match /posts/{postId} {
      allow read: if true;
      allow write: if request.auth != null
        && request.resource.data.authorId == request.auth.uid;
    }

    // Role-based access
    match /admin/{document=**} {
      allow read, write: if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }

    // Data validation
    match /orders/{orderId} {
      allow create: if request.auth != null
        && request.resource.data.keys().hasAll(['customerId', 'items', 'total'])
        && request.resource.data.total is number
        && request.resource.data.total > 0;
    }
  }
}
```

### Bigtable IAM
```bash
# Grant instance-level access
gcloud bigtable instances add-iam-policy-binding INSTANCE \
  --member=user:engineer@example.com \
  --role=roles/bigtable.reader

# Application access
gcloud bigtable instances add-iam-policy-binding INSTANCE \
  --member=serviceAccount:app@project.iam.gserviceaccount.com \
  --role=roles/bigtable.user
```

## Encryption

### Encryption at Rest
**Customer-Managed Encryption Keys (CMEK)**:
```bash
# Create KMS key
gcloud kms keyrings create db-keyring --location=us-central1
gcloud kms keys create db-key \
  --keyring=db-keyring \
  --location=us-central1 \
  --purpose=encryption

# Grant Cloud SQL service account access
gcloud kms keys add-iam-policy-binding db-key \
  --keyring=db-keyring \
  --location=us-central1 \
  --member=serviceAccount:service-PROJECT_NUMBER@gcp-sa-cloud-sql.iam.gserviceaccount.com \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# Create Cloud SQL instance with CMEK
gcloud sql instances create encrypted-instance \
  --database-version=POSTGRES_14 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/db-keyring/cryptoKeys/db-key
```

**Cloud Spanner CMEK**:
```bash
# Create instance with CMEK
gcloud spanner instances create encrypted-spanner \
  --config=regional-us-central1 \
  --nodes=1 \
  --kms-key=projects/PROJECT/locations/us-central1/keyRings/db-keyring/cryptoKeys/db-key
```

### Encryption in Transit
**SSL/TLS Configuration**:
```bash
# Require SSL for Cloud SQL
gcloud sql instances patch INSTANCE \
  --require-ssl

# Download server certificate
gcloud sql ssl-certs create client-cert client-key.pem \
  --instance=INSTANCE

# Connect with SSL
psql "sslmode=verify-ca sslrootcert=server-ca.pem \
      sslcert=client-cert.pem sslkey=client-key.pem \
      host=INSTANCE_IP dbname=DATABASE user=USER"
```

### Application-Level Encryption
```python
from cryptography.fernet import Fernet
from google.cloud import secretmanager

def get_encryption_key():
    """Get encryption key from Secret Manager"""
    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/PROJECT/secrets/db-encryption-key/versions/latest"
    response = client.access_secret_version(name=name)
    return response.payload.data

def encrypt_sensitive_data(data):
    """Encrypt data before storing in database"""
    key = get_encryption_key()
    f = Fernet(key)
    return f.encrypt(data.encode()).decode()

def decrypt_sensitive_data(encrypted_data):
    """Decrypt data after retrieving from database"""
    key = get_encryption_key()
    f = Fernet(key)
    return f.decrypt(encrypted_data.encode()).decode()

# Usage
ssn_encrypted = encrypt_sensitive_data(ssn)
# Store encrypted_ssn in database
# Later retrieve and decrypt
ssn = decrypt_sensitive_data(ssn_encrypted)
```

## Compliance

### Audit Logging
**Enable Audit Logs**:
```bash
# Enable data access logs for Cloud SQL
gcloud projects get-iam-policy PROJECT_ID > policy.yaml

# Edit policy.yaml to add audit config:
# auditConfigs:
# - auditLogConfigs:
#   - logType: DATA_READ
#   - logType: DATA_WRITE
#   service: cloudsql.googleapis.com

gcloud projects set-iam-policy PROJECT_ID policy.yaml
```

**Query Audit Logs**:
```sql
-- BigQuery query for database access
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.methodName as action,
  protoPayload.resourceName as resource,
  protoPayload.request as details
FROM
  `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE
  resource.type = "cloudsql_database"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;
```

### GDPR Compliance
**Data Residency**:
```bash
# Use EU regions
gcloud sql instances create eu-instance \
  --region=europe-west1 \
  --database-version=POSTGRES_14

# Organization policy to restrict regions
cat > region-policy.yaml <<EOF
constraint: constraints/gcp.resourceLocations
listPolicy:
  allowedValues:
    - in:eu-locations
EOF

gcloud resource-manager org-policies set-policy region-policy.yaml \
  --organization=ORG_ID
```

**Data Deletion**:
```python
def delete_user_data(user_id):
    """Delete all user data (GDPR right to erasure)"""
    # Cloud SQL
    with db.connect() as conn:
        conn.execute(
            "DELETE FROM user_data WHERE user_id = %s", (user_id,)
        )
        conn.execute(
            "DELETE FROM user_activity WHERE user_id = %s", (user_id,)
        )

    # Firestore
    user_ref = firestore_db.collection('users').document(user_id)
    user_ref.delete()

    # Bigtable
    table.delete_rows(row_keys=[f"user#{user_id}".encode()])

    # Log deletion for audit
    log_data_deletion(user_id, timestamp=datetime.utcnow())
```

### HIPAA Compliance
**BAA Requirements**:
- Sign Business Associate Agreement with Google
- Use HIPAA-eligible services
- Enable audit logging
- Use CMEK for encryption
- Implement access controls

**PHI Protection**:
```python
from google.cloud import dlp_v2

def de_identify_phi(project, content):
    """De-identify PHI using Cloud DLP"""
    dlp = dlp_v2.DlpServiceClient()

    # Define de-identification config
    deidentify_config = {
        "info_type_transformations": {
            "transformations": [
                {
                    "info_types": [
                        {"name": "PERSON_NAME"},
                        {"name": "US_SOCIAL_SECURITY_NUMBER"},
                        {"name": "PHONE_NUMBER"}
                    ],
                    "primitive_transformation": {
                        "crypto_hash_config": {
                            "crypto_key": {
                                "kms_wrapped": {
                                    "wrapped_key": WRAPPED_KEY,
                                    "crypto_key_name": KMS_KEY_NAME
                                }
                            }
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

## Security Best Practices

### Network Security
```bash
# Use private IP
gcloud sql instances patch INSTANCE \
  --network=projects/PROJECT/global/networks/vpc \
  --no-assign-ip

# Authorized networks for public IP
gcloud sql instances patch INSTANCE \
  --authorized-networks=203.0.113.0/24

# Cloud SQL Proxy for secure connections
cloud_sql_proxy -instances=PROJECT:REGION:INSTANCE=tcp:5432
```

### Access Management
1. Use IAM for authentication
2. Principle of least privilege
3. Separate admin and user accounts
4. Service accounts for applications
5. Regular access reviews
6. MFA for admin access
7. Audit all access
8. Time-bound access grants

### Data Protection
1. Enable encryption at rest (CMEK)
2. Require SSL/TLS in transit
3. Sensitive data in Secret Manager
4. Application-level encryption for PII
5. Regular security assessments
6. Data classification
7. DLP scanning
8. Backup encryption

### Compliance
1. Enable comprehensive audit logging
2. Regular compliance audits
3. Data retention policies
4. Documented procedures
5. Incident response plan
6. Regular training
7. Third-party assessments
8. Compliance reporting

## Common Scenarios

**Scenario**: Multi-tenant SaaS with data isolation
**Solution**: Separate databases per tenant or row-level security, CMEK per tenant, audit logging, IAM per tenant

**Scenario**: Healthcare application (HIPAA)
**Solution**: BAA with Google, CMEK, audit logs, DLP for PHI, private networking, strict access controls

**Scenario**: EU customer data (GDPR)
**Solution**: EU regions, data deletion workflows, audit logging, consent tracking, data portability

## Study Tips

1. Practice IAM configuration for each service
2. Implement CMEK encryption
3. Configure audit logging
4. Set up Firestore security rules
5. Understand compliance requirements
6. Work with DLP for sensitive data
7. Network security configurations

## Additional Resources

- [Cloud SQL Security](https://cloud.google.com/sql/docs/postgres/security)
- [Cloud Spanner IAM](https://cloud.google.com/spanner/docs/iam)
- [Firestore Security Rules](https://firebase.google.com/docs/firestore/security/get-started)
- [CMEK Documentation](https://cloud.google.com/kms/docs/cmek)
- [Compliance Resources](https://cloud.google.com/security/compliance)
