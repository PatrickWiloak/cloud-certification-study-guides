# Database Security and Compliance - GCP Professional Cloud Database Engineer

## Overview

Comprehensive guide to database security, access control, encryption, compliance, and security best practices for GCP database services. This guide covers security configurations for Cloud SQL, Cloud Spanner, Firestore, Bigtable, and Memorystore.

**Key Security Principles**:
- Defense in depth with multiple security layers
- Principle of least privilege for all access
- Encryption at rest and in transit for all data
- Comprehensive audit logging and monitoring
- Compliance with industry standards (PCI-DSS, HIPAA, GDPR, SOC 2)
- Network isolation and private connectivity
- Automated security scanning and remediation

**Exam Focus Areas**:
- Implementing CMEK encryption across all database services
- Configuring IAM authentication and authorization
- Setting up private networking and VPC Service Controls
- Enabling and analyzing audit logs
- Implementing compliance frameworks
- Integrating Cloud DLP for sensitive data protection
- Security monitoring and anomaly detection

## Database Access Control and IAM

### Cloud SQL IAM Authentication

**Overview**:
- IAM database authentication eliminates password management
- Users authenticate with their Google identity
- Service accounts for application access
- Integrated with Cloud IAM policies and audit logging
- Supports PostgreSQL, MySQL, and SQL Server (limited)

**Exam Tip**: IAM authentication is preferred over traditional password-based authentication for security, auditability, and integration with organizational identity systems.

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

# Create IAM user for PostgreSQL with specific database access
gcloud sql users create user@example.com \
  --instance=prod-db \
  --type=CLOUD_IAM_USER

# Create custom role with minimal permissions
gcloud iam roles create CloudSQLDataAnalyst \
  --project=PROJECT_ID \
  --title="Cloud SQL Data Analyst" \
  --description="Read-only access to specific databases" \
  --permissions=cloudsql.instances.connect,cloudsql.instances.get \
  --stage=GA
```

**IAM Policy for Database Access**:
```yaml
# Cloud SQL instance-level IAM policy
bindings:
- role: roles/cloudsql.client
  members:
  - user:analyst@example.com
  - serviceAccount:app-backend@project.iam.gserviceaccount.com
  condition:
    title: "Production access during business hours"
    description: "Allow access only during business hours"
    expression: |
      request.time.getHours("America/New_York") >= 9 &&
      request.time.getHours("America/New_York") <= 17 &&
      request.time.getDayOfWeek("America/New_York") >= 1 &&
      request.time.getDayOfWeek("America/New_York") <= 5

- role: roles/cloudsql.admin
  members:
  - group:dba-team@example.com
  - user:senior-dba@example.com

- role: roles/cloudsql.instanceUser
  members:
  - serviceAccount:monitoring@project.iam.gserviceaccount.com

- role: roles/cloudsql.viewer
  members:
  - domain:example.com
  condition:
    title: "Read-only access"
    description: "View instance details only"
    expression: "true"

# Apply the policy
gcloud sql instances set-iam-policy INSTANCE policy.yaml
```

**Connection with IAM**:
```python
import sqlalchemy
from google.cloud.sql.connector import Connector
from google.auth import default

# Get IAM credentials
credentials, project = default()

# PostgreSQL with IAM authentication
connector = Connector()

def getconn():
    conn = connector.connect(
        "project:region:instance",
        "pg8000",
        user="user@example.com",
        password="",  # Empty for IAM auth
        db="production_db",
        enable_iam_auth=True,
    )
    return conn

# Create engine with IAM auth
engine = sqlalchemy.create_engine(
    "postgresql+pg8000://",
    creator=getconn,
)

# MySQL with IAM authentication
def getconn_mysql():
    conn = connector.connect(
        "project:region:mysql-instance",
        "pymysql",
        user="user@example.com",
        password="",
        db="app_db",
        enable_iam_auth=True,
    )
    return conn

mysql_engine = sqlalchemy.create_engine(
    "mysql+pymysql://",
    creator=getconn_mysql,
)

# Service account authentication from application
import google.auth
from google.auth.transport.requests import Request

def get_iam_token():
    """Get IAM token for database authentication"""
    creds, project = google.auth.default(
        scopes=['https://www.googleapis.com/auth/sqlservice.admin']
    )
    creds.refresh(Request())
    return creds.token

# Use in connection string
token = get_iam_token()
# Token is used automatically by Cloud SQL Connector
```

**Exam Tip**: For the exam, know that IAM authentication requires enabling the Cloud SQL Admin API, creating IAM database users, and using the Cloud SQL Connector or Proxy. Traditional username/password authentication cannot be used with IAM users.

### Cloud Spanner IAM

**Overview**:
- Fine-grained access control at instance, database, and table levels
- Role-based access control (RBAC) with predefined and custom roles
- Conditional IAM policies for time-based and attribute-based access
- Integration with VPC Service Controls for perimeter security

**Predefined Roles**:
- `roles/spanner.admin`: Full control over instances and databases
- `roles/spanner.databaseAdmin`: Database schema and data management
- `roles/spanner.databaseReader`: Read-only access to database data
- `roles/spanner.databaseUser`: Read and write access to database data
- `roles/spanner.viewer`: View instance and database metadata
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

# Custom role for specific permissions
gcloud iam roles create spannerReadOnlyCustomer \
  --project=PROJECT_ID \
  --title="Spanner Customer Reader" \
  --description="Read customer data only" \
  --permissions=spanner.databases.select,spanner.sessions.create,spanner.sessions.get \
  --stage=GA

# Database-level policy with multiple conditions
gcloud spanner databases set-iam-policy customer-db policy.yaml
```

**Spanner IAM Policy Example**:
```yaml
# policy.yaml for Spanner database
bindings:
- role: roles/spanner.databaseReader
  members:
  - user:analyst@example.com
  - group:data-analysts@example.com
  condition:
    title: "Read access to specific tables"
    description: "Analysts can read customer and order tables"
    expression: |
      resource.name.endsWith('/tables/customers') ||
      resource.name.endsWith('/tables/orders')

- role: roles/spanner.databaseUser
  members:
  - serviceAccount:backend-api@project.iam.gserviceaccount.com
  - serviceAccount:batch-processor@project.iam.gserviceaccount.com

- role: roles/spanner.databaseAdmin
  members:
  - group:database-admins@example.com
  condition:
    title: "Admin access with MFA"
    description: "Require MFA for admin operations"
    expression: |
      request.auth.claims.auth_time > (request.time - duration('1h'))

- role: roles/spanner.backupAdmin
  members:
  - serviceAccount:backup-service@project.iam.gserviceaccount.com
```

**Fine-Grained Access Control with Row-Level Security**:
```sql
-- Spanner fine-grained access control using views
CREATE VIEW customer_view AS
SELECT customer_id, name, email, region
FROM customers
WHERE region = @region_param;

-- Grant access to view only
GRANT SELECT ON TABLE customer_view TO ROLE customer_viewer;

-- Application sets session variable
SET SESSION region = 'US-WEST';

-- User sees only their region's data
SELECT * FROM customer_view;
```

**Exam Tip**: Spanner supports IAM at the instance and database levels, but not at the table or row level through IAM alone. Use views and parameterized queries for fine-grained access control within the database.

### Firestore Security Rules

**Overview**:
- Client-side security rules for web and mobile applications
- Server-side IAM for administrative access
- Rules evaluated on every request
- Support for authentication context, data validation, and custom functions
- Real-time evaluation with no caching
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

    // Time-based access
    match /promotions/{promoId} {
      allow read: if request.time < resource.data.expiryDate;
      allow write: if request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'marketing';
    }

    // Hierarchical data access
    match /organizations/{orgId} {
      allow read: if request.auth != null
        && request.auth.uid in resource.data.members;

      match /projects/{projectId} {
        allow read, write: if request.auth != null
          && exists(/databases/$(database)/documents/organizations/$(orgId))
          && request.auth.uid in get(/databases/$(database)/documents/organizations/$(orgId)).data.members;
      }
    }

    // Content owner access
    match /content/{contentId} {
      allow read: if resource.data.visibility == 'public'
        || (request.auth != null && request.auth.uid == resource.data.ownerId);
      allow write: if request.auth != null && request.auth.uid == resource.data.ownerId;
      allow delete: if request.auth != null
        && (request.auth.uid == resource.data.ownerId
        || get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin');
    }

    // Field-level validation
    match /profiles/{userId} {
      allow read: if request.auth.uid == userId;
      allow update: if request.auth.uid == userId
        && request.resource.data.email == resource.data.email  // Email cannot be changed
        && request.resource.data.createdAt == resource.data.createdAt  // Creation date immutable
        && request.resource.data.emailVerified == resource.data.emailVerified;  // Only admin can verify
    }

    // Custom functions for complex logic
    function isAdmin() {
      return request.auth != null &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }

    function isOwner(resource) {
      return request.auth != null && request.auth.uid == resource.data.ownerId;
    }

    function hasPermission(resource, permission) {
      return request.auth != null &&
        resource.data.permissions[request.auth.uid] == permission;
    }

    match /documents/{documentId} {
      allow read: if isAdmin() || isOwner(resource) || hasPermission(resource, 'read');
      allow write: if isAdmin() || isOwner(resource) || hasPermission(resource, 'write');
    }
  }
}
```

**Firestore IAM for Server-Side Access**:
```bash
# Grant Firestore user role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:backend@project.iam.gserviceaccount.com \
  --role=roles/datastore.user

# Grant Firestore viewer role (read-only)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:analyst@example.com \
  --role=roles/datastore.viewer

# Custom role for specific operations
gcloud iam roles create firestoreReader \
  --project=PROJECT_ID \
  --title="Firestore Reader" \
  --description="Read-only access to Firestore" \
  --permissions=datastore.entities.get,datastore.entities.list \
  --stage=GA
```

**Exam Tip**: Security rules apply to client SDK access (web/mobile), while IAM roles apply to server-side access through Admin SDK and REST API. For production systems, use both: security rules for client access and IAM for server/admin access.

### Bigtable IAM

**Overview**:
- Instance and table-level access control
- No row-level security (implement in application layer)
- Predefined roles for different access patterns
- Integration with VPC Service Controls
```bash
# Grant instance-level access
gcloud bigtable instances add-iam-policy-binding INSTANCE \
  --member=user:engineer@example.com \
  --role=roles/bigtable.reader

# Application access with write permissions
gcloud bigtable instances add-iam-policy-binding INSTANCE \
  --member=serviceAccount:app@project.iam.gserviceaccount.com \
  --role=roles/bigtable.user

# Admin access for table management
gcloud bigtable instances add-iam-policy-binding INSTANCE \
  --member=group:bigtable-admins@example.com \
  --role=roles/bigtable.admin

# Viewer for monitoring
gcloud bigtable instances add-iam-policy-binding INSTANCE \
  --member=serviceAccount:monitoring@project.iam.gserviceaccount.com \
  --role=roles/bigtable.viewer
```

**Bigtable Application-Level Access Control**:
```python
from google.cloud import bigtable

def read_with_prefix_filter(project_id, instance_id, table_id, user_id):
    """Implement row-level access control in application"""
    client = bigtable.Client(project=project_id)
    instance = client.instance(instance_id)
    table = instance.table(table_id)

    # Only read rows with user's prefix
    row_prefix = f"user#{user_id}#".encode()

    partial_rows = table.read_rows(
        start_key=row_prefix,
        end_key=row_prefix + b'\xff'
    )

    for row in partial_rows:
        # User can only access their own data
        print(f"Row key: {row.row_key.decode()}")
```

**Exam Tip**: Bigtable doesn't support native row-level security. Implement access control through row key design (e.g., prefixing with user/tenant ID) and application logic. Use IAM for instance and table-level access.

### Memorystore (Redis/Memcached) Access Control

```bash
# Enable Redis AUTH
gcloud redis instances create secure-redis \
  --region=us-central1 \
  --tier=standard \
  --auth-enabled

# Get AUTH string
gcloud redis instances get-auth-string secure-redis --region=us-central1

# IAM permissions for Memorystore
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:app@project.iam.gserviceaccount.com \
  --role=roles/redis.editor

# Connect with AUTH
redis-cli -h REDIS_IP -a AUTH_STRING
```

## Database Encryption

### Encryption at Rest - Overview

**Key Concepts**:
- **Default encryption**: All GCP databases encrypted at rest by default with Google-managed keys
- **CMEK (Customer-Managed Encryption Keys)**: Customer controls key lifecycle through Cloud KMS
- **CSEK (Customer-Supplied Encryption Keys)**: Limited support, customer provides keys per operation
- **Application-layer encryption**: Encrypt data before storing in database

**When to Use CMEK**:
- Regulatory requirements for key management
- Need to control key rotation and lifecycle
- Compliance with specific security standards (PCI-DSS, HIPAA)
- Multi-tenant isolation at encryption level
- Ability to revoke access by disabling keys

**Exam Tip**: CMEK adds latency and cost but provides additional control. Default Google-managed encryption is sufficient for most use cases. Use CMEK when compliance or business requirements mandate customer control over keys.

### Cloud SQL CMEK Encryption
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

# Update existing instance to use CMEK (requires instance restart)
gcloud sql instances patch existing-instance \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/db-keyring/cryptoKeys/db-key

# Backups are automatically encrypted with same CMEK
gcloud sql backups create \
  --instance=encrypted-instance

# Verify encryption
gcloud sql instances describe encrypted-instance \
  --format="value(diskEncryptionConfiguration.kmsKeyName)"
```

**Multi-Region CMEK Setup**:
```bash
# Create key ring in multi-region location
gcloud kms keyrings create multi-region-keyring --location=us

# Create key with automatic rotation
gcloud kms keys create auto-rotate-key \
  --keyring=multi-region-keyring \
  --location=us \
  --purpose=encryption \
  --rotation-period=90d \
  --next-rotation-time=2024-12-01T00:00:00Z

# Grant Cloud SQL service account access in all regions
for region in us-central1 us-east1 us-west1; do
  PROJECT_NUMBER=$(gcloud projects describe PROJECT_ID --format="value(projectNumber)")
  gcloud kms keys add-iam-policy-binding auto-rotate-key \
    --keyring=multi-region-keyring \
    --location=us \
    --member=serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-cloud-sql.iam.gserviceaccount.com \
    --role=roles/cloudkms.cryptoKeyEncrypterDecrypter
done
```

**Exam Tip**: Cloud SQL CMEK applies to both primary instance and backups. When using CMEK, the KMS key must be in the same or multi-region location as the instance. Disabling or destroying the key makes the instance and backups inaccessible.

### Cloud Spanner CMEK
```bash
# Create instance with CMEK
gcloud spanner instances create encrypted-spanner \
  --config=regional-us-central1 \
  --nodes=1 \
  --kms-key=projects/PROJECT/locations/us-central1/keyRings/db-keyring/cryptoKeys/db-key

# Grant Spanner service account access to KMS key
PROJECT_NUMBER=$(gcloud projects describe PROJECT_ID --format="value(projectNumber)")
gcloud kms keys add-iam-policy-binding db-key \
  --keyring=db-keyring \
  --location=us-central1 \
  --member=serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-cloud-spanner.iam.gserviceaccount.com \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# Create database with inherited CMEK from instance
gcloud spanner databases create customer-db \
  --instance=encrypted-spanner \
  --ddl='CREATE TABLE Customers (
    CustomerId INT64 NOT NULL,
    Name STRING(100),
    Email STRING(100)
  ) PRIMARY KEY (CustomerId)'

# Backups also use CMEK
gcloud spanner backups create customer-backup \
  --instance=encrypted-spanner \
  --database=customer-db \
  --retention-period=7d

# Verify CMEK configuration
gcloud spanner instances describe encrypted-spanner \
  --format="value(config, encryptionConfig.kmsKeyName)"
```

**Bigtable and Firestore CMEK**:
```bash
# Bigtable instance with CMEK
gcloud bigtable instances create encrypted-bigtable \
  --cluster=encrypted-cluster \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=3 \
  --cluster-kms-key=projects/PROJECT/locations/us-central1/keyRings/db-keyring/cryptoKeys/db-key \
  --display-name="Encrypted Bigtable Instance"

# Firestore with CMEK (set at database creation)
gcloud firestore databases create \
  --location=us-central \
  --kms-key-name=projects/PROJECT/locations/us-central1/keyRings/db-keyring/cryptoKeys/db-key
```

**Exam Tip**: For Spanner, CMEK is set at the instance level and inherited by all databases and backups. For Bigtable, CMEK is set per cluster. Firestore CMEK is set at database creation and cannot be changed later.

### Encryption in Transit (SSL/TLS)

**Overview**:
- All GCP database services support TLS 1.2+ for encryption in transit
- Cloud SQL can require SSL/TLS for all connections
- Certificate-based authentication available for Cloud SQL
- Spanner, Firestore, and Bigtable use TLS by default
**Cloud SQL SSL/TLS Configuration**:
```bash
# Require SSL for Cloud SQL
gcloud sql instances patch prod-instance \
  --require-ssl

# Download server CA certificate
gcloud sql ssl-certs describe server-ca \
  --instance=prod-instance \
  --format="get(cert)" > server-ca.pem

# Create client certificate
gcloud sql ssl-certs create client-cert client-key.pem \
  --instance=prod-instance

# Download client certificate
gcloud sql ssl-certs describe client-cert \
  --instance=prod-instance \
  --format="get(cert)" > client-cert.pem

# Connect with SSL verification (PostgreSQL)
psql "sslmode=verify-ca sslrootcert=server-ca.pem \
      sslcert=client-cert.pem sslkey=client-key.pem \
      host=INSTANCE_IP dbname=DATABASE user=USER"

# Connect with SSL verification (MySQL)
mysql --ssl-ca=server-ca.pem \
      --ssl-cert=client-cert.pem \
      --ssl-key=client-key.pem \
      -h INSTANCE_IP -u USER -p DATABASE

# List all SSL certificates
gcloud sql ssl-certs list --instance=prod-instance

# Revoke client certificate
gcloud sql ssl-certs delete client-cert --instance=prod-instance
```

**SSL Modes**:
- `disable`: No SSL (not recommended for production)
- `allow`: Use SSL if available
- `prefer`: Prefer SSL, fallback to non-SSL
- `require`: Require SSL but don't verify certificate
- `verify-ca`: Require SSL and verify certificate authority
- `verify-full`: Require SSL and verify certificate + hostname

**Python Connection with SSL**:
```python
import psycopg2
import ssl

# PostgreSQL with SSL
conn = psycopg2.connect(
    host="INSTANCE_IP",
    database="production_db",
    user="dbuser",
    password="password",
    sslmode="verify-ca",
    sslrootcert="/path/to/server-ca.pem",
    sslcert="/path/to/client-cert.pem",
    sslkey="/path/to/client-key.pem"
)

# Using Cloud SQL Connector with SSL
from google.cloud.sql.connector import Connector

connector = Connector()

def getconn():
    conn = connector.connect(
        "project:region:instance",
        "pg8000",
        user="user",
        password="pass",
        db="database",
        enable_ssl=True  # SSL enabled by default
    )
    return conn

engine = sqlalchemy.create_engine(
    "postgresql+pg8000://",
    creator=getconn,
)
```

**Exam Tip**: Cloud SQL Proxy and Cloud SQL Connector automatically use encrypted connections. For direct connections to public IP, always require SSL and use certificate verification in production. Client certificates provide additional authentication beyond username/password.

### Application-Level Encryption (Field-Level Encryption)
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

# Usage in database operations
ssn_encrypted = encrypt_sensitive_data(ssn)
# Store encrypted_ssn in database
# Later retrieve and decrypt
ssn = decrypt_sensitive_data(ssn_encrypted)

# Application-level encryption for Spanner
from google.cloud import spanner
from cryptography.fernet import Fernet

def store_encrypted_data(instance_id, database_id, customer_id, ssn, credit_card):
    """Store PII with field-level encryption"""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    # Encrypt sensitive fields
    encryption_key = get_encryption_key()
    cipher = Fernet(encryption_key)

    encrypted_ssn = cipher.encrypt(ssn.encode()).decode()
    encrypted_cc = cipher.encrypt(credit_card.encode()).decode()

    with database.batch() as batch:
        batch.insert(
            table='customers',
            columns=('customer_id', 'ssn_encrypted', 'credit_card_encrypted'),
            values=[(customer_id, encrypted_ssn, encrypted_cc)]
        )

# Use Cloud KMS for envelope encryption
from google.cloud import kms

def envelope_encryption(project_id, location_id, key_ring_id, key_id, plaintext):
    """Envelope encryption using Cloud KMS"""
    client = kms.KeyManagementServiceClient()
    key_name = client.crypto_key_path(project_id, location_id, key_ring_id, key_id)

    # Generate data encryption key (DEK)
    import os
    dek = os.urandom(32)

    # Encrypt data with DEK
    cipher = Fernet(base64.urlsafe_b64encode(dek))
    encrypted_data = cipher.encrypt(plaintext.encode())

    # Encrypt DEK with KMS
    encrypt_response = client.encrypt(
        request={'name': key_name, 'plaintext': dek}
    )
    encrypted_dek = encrypt_response.ciphertext

    return encrypted_data, encrypted_dek
```

**Exam Tip**: Use application-level encryption for highly sensitive data (SSN, credit cards, health records) that requires protection even if database is compromised. Combine with CMEK for defense in depth. Store encryption keys in Secret Manager or Cloud KMS, never in the database.

## Network Security

### Private IP Configuration

**Cloud SQL Private IP**:
```bash
# Create instance with private IP only
gcloud sql instances create private-instance \
  --database-version=POSTGRES_14 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/vpc-network \
  --no-assign-ip

# Add private IP to existing instance
gcloud sql instances patch existing-instance \
  --network=projects/PROJECT/global/networks/vpc-network

# Enable Private Service Connect
gcloud services enable servicenetworking.googleapis.com

# Allocate IP range for private services
gcloud compute addresses create google-managed-services-vpc-network \
  --global \
  --purpose=VPC_PEERING \
  --prefix-length=16 \
  --network=vpc-network

# Create private connection
gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --ranges=google-managed-services-vpc-network \
  --network=vpc-network
```

**Spanner and Bigtable Private Endpoints**:
```bash
# Spanner uses Private Google Access by default
# Enable Private Google Access on subnet
gcloud compute networks subnets update subnet-name \
  --region=us-central1 \
  --enable-private-ip-google-access

# Configure Private Service Connect for Spanner
gcloud compute addresses create spanner-psc-address \
  --global \
  --purpose=PRIVATE_SERVICE_CONNECT \
  --addresses=10.0.0.5 \
  --network=vpc-network

# Bigtable private endpoint configuration
gcloud compute addresses create bigtable-psc-address \
  --region=us-central1 \
  --purpose=PRIVATE_SERVICE_CONNECT \
  --addresses=10.1.0.5 \
  --network=vpc-network
```

### Cloud SQL Proxy

```bash
# Start Cloud SQL Proxy for private IP
cloud_sql_proxy -instances=PROJECT:REGION:INSTANCE=tcp:5432 \
  -ip_address_types=PRIVATE

# Start proxy with multiple instances
cloud_sql_proxy \
  -instances=PROJECT:REGION:prod-db=tcp:5432,PROJECT:REGION:analytics-db=tcp:5433 \
  -ip_address_types=PRIVATE

# Use Unix socket
cloud_sql_proxy -dir=/cloudsql -instances=PROJECT:REGION:INSTANCE

# Connect via Unix socket
psql "host=/cloudsql/PROJECT:REGION:INSTANCE dbname=DATABASE user=USER"

# Run as Cloud Run sidecar
gcloud run deploy app \
  --image=gcr.io/PROJECT/app \
  --add-cloudsql-instances=PROJECT:REGION:INSTANCE \
  --set-env-vars=DB_HOST=/cloudsql/PROJECT:REGION:INSTANCE
```

**Cloud SQL Proxy with IAM Authentication**:
```bash
# Proxy with automatic IAM token
cloud_sql_proxy -instances=PROJECT:REGION:INSTANCE=tcp:5432 \
  -enable_iam_login

# Application connects without password
psql "host=127.0.0.1 dbname=DATABASE user=user@example.com sslmode=disable"
```

### Authorized Networks

```bash
# Add authorized network (use only for public IP)
gcloud sql instances patch instance \
  --authorized-networks=203.0.113.0/24

# Add multiple networks
gcloud sql instances patch instance \
  --authorized-networks=203.0.113.0/24,198.51.100.0/24,192.0.2.0/24

# Add network with expiration time
gcloud sql instances patch instance \
  --authorized-networks=203.0.113.50/32 \
  --authorized-networks-expiration-time=2024-12-31T23:59:59Z

# Remove all authorized networks
gcloud sql instances patch instance \
  --clear-authorized-networks
```

**Exam Tip**: Prefer private IP over authorized networks. Use authorized networks only for temporary access or when VPC connectivity isn't available. Each authorized network increases attack surface.

### VPC Service Controls

```bash
# Create access policy
gcloud access-context-manager policies create \
  --organization=ORG_ID \
  --title="Database Security Policy"

# Create access level
gcloud access-context-manager levels create database_access \
  --policy=POLICY_ID \
  --title="Database Access Level" \
  --basic-level-spec=access-level.yaml

# access-level.yaml
# combiningFunction: AND
# conditions:
# - ipSubnetworks:
#   - 10.0.0.0/16
# - members:
#   - user:dba@example.com
#   - serviceAccount:app@project.iam.gserviceaccount.com

# Create service perimeter
gcloud access-context-manager perimeters create database_perimeter \
  --policy=POLICY_ID \
  --title="Database Perimeter" \
  --resources=projects/PROJECT_NUMBER \
  --restricted-services=sqladmin.googleapis.com,spanner.googleapis.com,bigtable.googleapis.com \
  --access-levels=database_access

# Add perimeter for data exfiltration protection
gcloud access-context-manager perimeters create secure_db_perimeter \
  --policy=POLICY_ID \
  --title="Secure Database Perimeter" \
  --resources=projects/PROD_PROJECT_NUMBER \
  --restricted-services=sqladmin.googleapis.com,spanner.googleapis.com \
  --access-levels=production_access \
  --ingress-policies=ingress-policy.yaml \
  --egress-policies=egress-policy.yaml
```

**Exam Tip**: VPC Service Controls prevent data exfiltration by creating security perimeters around database services. Critical for compliance requirements (HIPAA, PCI-DSS) and preventing insider threats.

## Audit Logging and Monitoring

### Enable Comprehensive Audit Logging
**Enable Audit Logs**:
```bash
# Enable data access logs for Cloud SQL
gcloud projects get-iam-policy PROJECT_ID > policy.yaml

# Edit policy.yaml to add audit config:
# auditConfigs:
# - auditLogConfigs:
#   - logType: ADMIN_READ
#   - logType: DATA_READ
#   - logType: DATA_WRITE
#   service: cloudsql.googleapis.com
# - auditLogConfigs:
#   - logType: ADMIN_READ
#   - logType: DATA_READ
#   - logType: DATA_WRITE
#   service: spanner.googleapis.com
# - auditLogConfigs:
#   - logType: ADMIN_READ
#   - logType: DATA_READ
#   - logType: DATA_WRITE
#   service: bigtableadmin.googleapis.com

gcloud projects set-iam-policy PROJECT_ID policy.yaml

# Enable PostgreSQL query logging (pgAudit)
gcloud sql instances patch instance \
  --database-flags=cloudsql.enable_pgaudit=on,pgaudit.log=all

# Enable MySQL query logging
gcloud sql instances patch mysql-instance \
  --database-flags=general_log=on,log_output=FILE

# Enable Cloud SQL audit logging
gcloud logging sinks create sql-audit-sink \
  bigquery.googleapis.com/projects/PROJECT/datasets/audit_logs \
  --log-filter='resource.type="cloudsql_database"'
```

**Cloud Audit Logs Types**:
- **Admin Activity logs**: Always enabled, no charge, 400-day retention
- **Data Access logs**: Must be enabled, charged, captures data reads/writes
- **System Event logs**: Always enabled, no charge
- **Policy Denied logs**: Always enabled when VPC-SC is used

### Database-Specific Audit Logging

**PostgreSQL pgAudit Configuration**:
```sql
-- Enable pgAudit extension
CREATE EXTENSION pgaudit;

-- Configure audit logging
ALTER SYSTEM SET pgaudit.log = 'ddl, write, read';
ALTER SYSTEM SET pgaudit.log_catalog = 'off';
ALTER SYSTEM SET pgaudit.log_parameter = 'on';
ALTER SYSTEM SET pgaudit.log_relation = 'on';
ALTER SYSTEM SET pgaudit.log_statement_once = 'off';

-- Audit specific roles
ALTER ROLE app_user SET pgaudit.log = 'read, write';

-- Audit specific tables
CREATE TABLE sensitive_data (
    id SERIAL PRIMARY KEY,
    ssn VARCHAR(11),
    credit_card VARCHAR(20)
);

COMMENT ON TABLE sensitive_data IS 'pgaudit: read, write';
```

**MySQL Audit Plugin**:
```sql
-- Enable audit plugin
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

-- Configure audit settings via flags
-- Set via gcloud:
-- --database-flags=audit_log_policy=ALL,audit_log_format=JSON
```

**Cloud SQL Query Insights**:
```bash
# Enable Query Insights for performance and security monitoring
gcloud sql instances patch instance \
  --insights-config-query-insights-enabled \
  --insights-config-query-string-length=1024 \
  --insights-config-query-plans-per-minute=5
```

### Audit Log Analysis Queries
```sql
-- Query 1: All database access by user
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.methodName as action,
  protoPayload.resourceName as resource,
  resource.labels.database_id as database,
  protoPayload.request as details
FROM
  `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE
  resource.type = "cloudsql_database"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;

-- Query 2: Failed authentication attempts
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as attempted_user,
  protoPayload.status.message as error_message,
  protoPayload.requestMetadata.callerIp as source_ip,
  resource.labels.instance_id as instance
FROM
  `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE
  resource.type = "cloudsql_database"
  AND protoPayload.status.code != 0
  AND protoPayload.methodName LIKE '%connect%'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
ORDER BY timestamp DESC;

-- Query 3: Schema changes (DDL operations)
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.methodName as operation,
  resource.labels.database_id as database,
  JSON_EXTRACT_SCALAR(protoPayload.metadata, '$.statement') as sql_statement
FROM
  `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE
  resource.type = "cloudsql_database"
  AND protoPayload.methodName LIKE '%ddl%'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
ORDER BY timestamp DESC;

-- Query 4: Suspicious access patterns (after-hours access)
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.requestMetadata.callerIp as source_ip,
  COUNT(*) as query_count
FROM
  `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE
  resource.type = "cloudsql_database"
  AND EXTRACT(HOUR FROM timestamp) NOT BETWEEN 6 AND 18  -- Outside 6 AM - 6 PM
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY timestamp, user, source_ip
HAVING query_count > 100
ORDER BY query_count DESC;

-- Query 5: Data export operations
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.methodName as operation,
  resource.labels.instance_id as instance,
  CAST(JSON_EXTRACT_SCALAR(protoPayload.metadata, '$.exportContext.fileType') AS STRING) as export_type,
  CAST(JSON_EXTRACT_SCALAR(protoPayload.metadata, '$.exportContext.uri') AS STRING) as destination
FROM
  `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE
  protoPayload.methodName = 'cloudsql.instances.export'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
ORDER BY timestamp DESC;

-- Query 6: Spanner access patterns
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  resource.labels.database_id as database,
  protoPayload.requestMetadata.callerIp as source_ip,
  COUNT(*) as operation_count
FROM
  `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE
  resource.type = "spanner_database"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
GROUP BY timestamp, user, database, source_ip
ORDER BY operation_count DESC
LIMIT 100;

-- Query 7: IAM policy changes on databases
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as changed_by,
  protoPayload.methodName as operation,
  resource.labels.instance_id as instance,
  JSON_EXTRACT(protoPayload.request, '$.policy.bindings') as new_bindings
FROM
  `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE
  protoPayload.methodName LIKE '%SetIamPolicy%'
  AND (resource.type = "cloudsql_database" OR resource.type = "spanner_instance")
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
ORDER BY timestamp DESC;

-- Query 8: Unusual data access volumes
WITH daily_access AS (
  SELECT
    DATE(timestamp) as access_date,
    protoPayload.authenticationInfo.principalEmail as user,
    COUNT(*) as daily_queries
  FROM
    `project.dataset.cloudaudit_googleapis_com_data_access_*`
  WHERE
    resource.type = "cloudsql_database"
    AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
  GROUP BY access_date, user
),
avg_access AS (
  SELECT
    user,
    AVG(daily_queries) as avg_queries,
    STDDEV(daily_queries) as stddev_queries
  FROM daily_access
  GROUP BY user
)
SELECT
  d.access_date,
  d.user,
  d.daily_queries,
  a.avg_queries,
  (d.daily_queries - a.avg_queries) / a.stddev_queries as z_score
FROM daily_access d
JOIN avg_access a ON d.user = a.user
WHERE (d.daily_queries - a.avg_queries) / a.stddev_queries > 3  -- More than 3 standard deviations
ORDER BY z_score DESC;
```

### Security Monitoring and Alerting

```bash
# Create log-based metric for failed connections
gcloud logging metrics create failed_db_connections \
  --description="Failed database connection attempts" \
  --log-filter='resource.type="cloudsql_database"
    AND protoPayload.status.code!=0
    AND protoPayload.methodName=~".*connect.*"'

# Create alert policy for failed connections
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High Failed DB Connection Rate" \
  --condition-display-name="Failed connections > 10 in 5 min" \
  --condition-threshold-value=10 \
  --condition-threshold-duration=300s \
  --condition-aggregation-alignment-period=300s \
  --condition-aggregation-per-series-aligner=ALIGN_RATE

# Create alert for suspicious after-hours access
gcloud logging metrics create after_hours_access \
  --description="After-hours database access" \
  --log-filter='resource.type="cloudsql_database"
    AND protoPayload.methodName=~".*query.*"
    AND (EXTRACT(HOUR FROM timestamp) < 6 OR EXTRACT(HOUR FROM timestamp) > 18)'

# Security Command Center integration
gcloud services enable securitycenter.googleapis.com

# Enable Security Health Analytics
gcloud scc settings services enable \
  --organization=ORG_ID \
  --service=SECURITY_HEALTH_ANALYTICS
```

**Exam Tip**: Export audit logs to BigQuery for long-term retention and complex analysis. Create log-based metrics and alerts for security events. Use Query Insights for performance monitoring and identifying inefficient or suspicious queries.

## Compliance Frameworks

### PCI-DSS Compliance for Payment Data

**Requirements**:
1. Encrypt cardholder data at rest (CMEK required)
2. Encrypt cardholder data in transit (TLS 1.2+)
3. Implement strong access control (IAM + authentication)
4. Regularly monitor and test networks (audit logs + alerts)
5. Maintain secure systems (patching, hardening)
6. Implement network segmentation (VPC, Private IP)

**PCI-DSS Configuration Checklist**:

```bash
# 1. Enable CMEK encryption
gcloud kms keyrings create pci-keyring --location=us-central1
gcloud kms keys create payment-key \
  --keyring=pci-keyring \
  --location=us-central1 \
  --purpose=encryption \
  --rotation-period=90d

# 2. Create PCI-compliant Cloud SQL instance
gcloud sql instances create pci-compliant-db \
  --database-version=POSTGRES_14 \
  --tier=db-n1-standard-4 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/pci-vpc \
  --no-assign-ip \
  --require-ssl \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/pci-keyring/cryptoKeys/payment-key \
  --database-flags=cloudsql.enable_pgaudit=on,pgaudit.log=all \
  --backup-start-time=02:00 \
  --retained-backups-count=30 \
  --deletion-protection

# 3. Enable comprehensive audit logging
gcloud logging sinks create pci-audit-sink \
  bigquery.googleapis.com/projects/PROJECT/datasets/pci_audit_logs \
  --log-filter='resource.type="cloudsql_database" AND resource.labels.instance_id="pci-compliant-db"'

# 4. Implement VPC Service Controls
gcloud access-context-manager perimeters create pci_perimeter \
  --policy=POLICY_ID \
  --title="PCI-DSS Perimeter" \
  --resources=projects/PROJECT_NUMBER \
  --restricted-services=sqladmin.googleapis.com \
  --access-levels=pci_access_level

# 5. Create IAM policies with least privilege
gcloud sql instances set-iam-policy pci-compliant-db pci-iam-policy.yaml
```

**PCI-DSS IAM Policy**:
```yaml
# pci-iam-policy.yaml
bindings:
- role: roles/cloudsql.client
  members:
  - serviceAccount:payment-app@project.iam.gserviceaccount.com
  condition:
    title: "Access only from production VPC"
    expression: |
      origin.ip in ["10.0.0.0/16"]

- role: roles/cloudsql.admin
  members:
  - group:dba-team@example.com
  condition:
    title: "Admin with MFA and time restriction"
    expression: |
      request.auth.claims.auth_time > (request.time - duration('1h')) &&
      request.time.getHours("America/New_York") >= 9 &&
      request.time.getHours("America/New_York") <= 17
```

**Application-Level PCI Controls**:
```python
from google.cloud import kms, secretmanager
from cryptography.fernet import Fernet
import hashlib

class PCIDataHandler:
    """Handle PCI-compliant data storage and retrieval"""

    def __init__(self, project_id, kms_key_name):
        self.project_id = project_id
        self.kms_key_name = kms_key_name
        self.kms_client = kms.KeyManagementServiceClient()

    def tokenize_card(self, card_number):
        """Tokenize credit card number"""
        # Create irreversible hash as token
        token = hashlib.sha256(card_number.encode()).hexdigest()
        return token

    def encrypt_card_data(self, card_data):
        """Encrypt card data with KMS"""
        # Get DEK from KMS
        response = self.kms_client.encrypt(
            request={
                'name': self.kms_key_name,
                'plaintext': card_data.encode()
            }
        )
        return response.ciphertext

    def mask_card_number(self, card_number):
        """Mask card number for display (PCI requirement)"""
        return f"****-****-****-{card_number[-4:]}"

    def store_payment_data(self, conn, transaction_id, card_number, cvv, expiry):
        """Store payment data securely"""
        # Tokenize card number
        token = self.tokenize_card(card_number)

        # Encrypt sensitive data
        encrypted_card = self.encrypt_card_data(card_number)
        encrypted_cvv = self.encrypt_card_data(cvv)

        # Store only encrypted data and token
        with conn.cursor() as cursor:
            cursor.execute("""
                INSERT INTO payment_tokens (transaction_id, card_token, card_encrypted, cvv_encrypted, expiry)
                VALUES (%s, %s, %s, %s, %s)
            """, (transaction_id, token, encrypted_card, encrypted_cvv, expiry))

        # Return masked card for display
        return self.mask_card_number(card_number)
```

**Exam Tip**: For PCI-DSS compliance, use CMEK encryption, implement network isolation with private IP, enable comprehensive audit logging, use tokenization for card data, and implement VPC Service Controls to prevent data exfiltration.

### HIPAA Compliance for Healthcare Data

**Requirements**:
1. Business Associate Agreement (BAA) with Google Cloud
2. Encryption at rest and in transit
3. Access controls and audit logging
4. Backup and disaster recovery
5. Incident response procedures

**HIPAA-Compliant Configuration**:
```bash
# Create HIPAA-compliant database
gcloud sql instances create hipaa-db \
  --database-version=POSTGRES_14 \
  --tier=db-custom-4-26624 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/healthcare-vpc \
  --no-assign-ip \
  --require-ssl \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/hipaa-keyring/cryptoKeys/phi-key \
  --database-flags=cloudsql.enable_pgaudit=on,pgaudit.log=all \
  --backup-start-time=03:00 \
  --retained-backups-count=90 \
  --deletion-protection \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=2

# Enable audit logging for HIPAA
gcloud logging sinks create hipaa-audit-sink \
  bigquery.googleapis.com/projects/PROJECT/datasets/hipaa_audit_logs \
  --log-filter='resource.type="cloudsql_database" AND resource.labels.instance_id="hipaa-db"'

# Set retention policy (7 years for HIPAA)
bq update --default_table_expiration=220752000 PROJECT:hipaa_audit_logs
```

**PHI De-identification with Cloud DLP**:
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

# Scan database for PHI
def scan_database_for_phi(project_id, instance_connection_name):
    """Scan database tables for unprotected PHI"""
    dlp = dlp_v2.DlpServiceClient()

    # Configure DLP job
    inspect_job = {
        'inspect_config': {
            'info_types': [
                {'name': 'PERSON_NAME'},
                {'name': 'US_SOCIAL_SECURITY_NUMBER'},
                {'name': 'PHONE_NUMBER'},
                {'name': 'EMAIL_ADDRESS'},
                {'name': 'DATE_OF_BIRTH'},
                {'name': 'MEDICAL_RECORD_NUMBER'}
            ],
            'min_likelihood': dlp_v2.Likelihood.POSSIBLE,
            'limits': {'max_findings_per_request': 100}
        },
        'storage_config': {
            'cloud_sql_options': {
                'connection_name': instance_connection_name,
                'identifying_column': 'patient_id'
            }
        }
    }

    # Create DLP job
    response = dlp.create_dlp_job(
        request={
            'parent': f'projects/{project_id}',
            'inspect_job': inspect_job
        }
    )

    return response.name
```

**HIPAA Access Control**:
```python
class HIPAAAccessControl:
    """Implement HIPAA-compliant access controls"""

    def __init__(self, db_conn):
        self.conn = db_conn

    def log_phi_access(self, user_id, patient_id, access_type, reason):
        """Log every PHI access (HIPAA requirement)"""
        with self.conn.cursor() as cursor:
            cursor.execute("""
                INSERT INTO phi_access_log
                (timestamp, user_id, patient_id, access_type, reason, ip_address)
                VALUES (NOW(), %s, %s, %s, %s, %s)
            """, (user_id, patient_id, access_type, reason, get_client_ip()))

    def check_access_authorization(self, user_id, patient_id):
        """Verify user is authorized to access patient data"""
        with self.conn.cursor() as cursor:
            cursor.execute("""
                SELECT COUNT(*) FROM patient_authorizations
                WHERE user_id = %s AND patient_id = %s
                AND expiry_date > NOW()
            """, (user_id, patient_id))

            return cursor.fetchone()[0] > 0

    def access_phi(self, user_id, patient_id, reason):
        """Access PHI with proper authorization and logging"""
        # Check authorization
        if not self.check_access_authorization(user_id, patient_id):
            self.log_phi_access(user_id, patient_id, 'DENIED', reason)
            raise PermissionError("User not authorized to access this patient's data")

        # Log access
        self.log_phi_access(user_id, patient_id, 'GRANTED', reason)

        # Retrieve PHI
        with self.conn.cursor() as cursor:
            cursor.execute("""
                SELECT * FROM patient_records
                WHERE patient_id = %s
            """, (patient_id,))

            return cursor.fetchone()
```

**Exam Tip**: For HIPAA compliance, sign a BAA with Google, use only HIPAA-eligible services (Cloud SQL, Spanner, Firestore, BigQuery), implement CMEK encryption, enable comprehensive audit logging with 7-year retention, use Cloud DLP for PHI detection, and implement strict access controls with audit trails.

### GDPR Compliance for EU Data

**Data Residency Requirements**:
```bash
# Create database in EU region
gcloud sql instances create gdpr-compliant-db \
  --region=europe-west1 \
  --database-version=POSTGRES_14 \
  --network=projects/PROJECT/global/networks/eu-vpc \
  --no-assign-ip

# Organization policy to enforce EU regions
cat > eu-regions-policy.yaml <<EOF
name: projects/PROJECT_ID/policies/gcp.resourceLocations
spec:
  rules:
  - values:
      allowedValues:
      - in:eu-locations
EOF

gcloud resource-manager org-policies set-policy eu-regions-policy.yaml \
  --project=PROJECT_ID

# Spanner with EU region
gcloud spanner instances create gdpr-spanner \
  --config=regional-europe-west1 \
  --nodes=1 \
  --description="GDPR compliant Spanner instance"
```

**GDPR Data Subject Rights Implementation**:
```python
from google.cloud import spanner, firestore, bigtable
from datetime import datetime
import json

class GDPRDataController:
    """Implement GDPR data subject rights"""

    def __init__(self, project_id):
        self.project_id = project_id
        self.spanner_client = spanner.Client(project=project_id)
        self.firestore_client = firestore.Client(project=project_id)
        self.bigtable_client = bigtable.Client(project=project_id)

    def export_user_data(self, user_id):
        """Right to data portability (Article 20)"""
        user_data = {}

        # Export from Cloud SQL
        with db.connect() as conn:
            cursor = conn.cursor()
            cursor.execute(
                "SELECT * FROM users WHERE user_id = %s", (user_id,)
            )
            user_data['profile'] = cursor.fetchone()

            cursor.execute(
                "SELECT * FROM user_activity WHERE user_id = %s", (user_id,)
            )
            user_data['activity'] = cursor.fetchall()

        # Export from Firestore
        user_doc = self.firestore_client.collection('users').document(user_id).get()
        if user_doc.exists:
            user_data['firestore'] = user_doc.to_dict()

        # Export from Spanner
        instance = self.spanner_client.instance('prod-instance')
        database = instance.database('user-db')
        with database.snapshot() as snapshot:
            results = snapshot.execute_sql(
                "SELECT * FROM UserPreferences WHERE UserId = @user_id",
                params={'user_id': user_id},
                param_types={'user_id': spanner.param_types.STRING}
            )
            user_data['preferences'] = [dict(row) for row in results]

        # Create portable format
        return json.dumps(user_data, indent=2, default=str)

    def delete_user_data(self, user_id):
        """Right to erasure (Article 17)"""
        deletion_log = []

        # Delete from Cloud SQL
        with db.connect() as conn:
            cursor = conn.cursor()

            tables = ['user_activity', 'user_preferences', 'users']
            for table in tables:
                cursor.execute(
                    f"DELETE FROM {table} WHERE user_id = %s", (user_id,)
                )
                deletion_log.append(f"Deleted from {table}: {cursor.rowcount} rows")

        # Delete from Firestore
        user_ref = self.firestore_client.collection('users').document(user_id)
        user_ref.delete()
        deletion_log.append(f"Deleted Firestore document: users/{user_id}")

        # Delete from Spanner
        instance = self.spanner_client.instance('prod-instance')
        database = instance.database('user-db')
        with database.batch() as batch:
            batch.delete('UserPreferences', spanner.KeySet(keys=[[user_id]]))
            batch.delete('UserActivity', spanner.KeySet(all_=True))  # Delete all with user_id
        deletion_log.append(f"Deleted from Spanner")

        # Delete from Bigtable
        instance = self.bigtable_client.instance('prod-bigtable')
        table = instance.table('user-events')
        row_key_prefix = f"user#{user_id}#".encode()
        rows = table.read_rows(start_key=row_key_prefix, end_key=row_key_prefix + b'\xff')
        for row in rows:
            row.delete()
        deletion_log.append(f"Deleted from Bigtable")

        # Log deletion for compliance
        self.log_gdpr_action(user_id, 'DELETE', deletion_log)

        return deletion_log

    def anonymize_user_data(self, user_id):
        """Anonymize instead of delete when deletion not possible"""
        with db.connect() as conn:
            cursor = conn.cursor()
            cursor.execute("""
                UPDATE users
                SET name = 'ANONYMIZED',
                    email = CONCAT('anonymized_', user_id, '@deleted.com'),
                    phone = NULL,
                    address = NULL,
                    anonymized_at = NOW()
                WHERE user_id = %s
            """, (user_id,))

    def rectify_user_data(self, user_id, corrections):
        """Right to rectification (Article 16)"""
        with db.connect() as conn:
            cursor = conn.cursor()
            for field, value in corrections.items():
                cursor.execute(f"""
                    UPDATE users
                    SET {field} = %s, updated_at = NOW()
                    WHERE user_id = %s
                """, (value, user_id))

        self.log_gdpr_action(user_id, 'RECTIFY', corrections)

    def log_gdpr_action(self, user_id, action_type, details):
        """Log GDPR-related actions for compliance"""
        with db.connect() as conn:
            cursor = conn.cursor()
            cursor.execute("""
                INSERT INTO gdpr_actions
                (timestamp, user_id, action_type, details, performed_by)
                VALUES (NOW(), %s, %s, %s, %s)
            """, (user_id, action_type, json.dumps(details), get_current_user()))
```

**Exam Tip**: For GDPR compliance, enforce EU region constraints with organization policies, implement data subject rights (access, rectification, erasure, portability), maintain audit logs, use data retention policies, and implement consent tracking.

### SOC 2 Compliance

**SOC 2 Trust Service Criteria Configuration**:
```bash
# Security: Access controls and encryption
gcloud sql instances create soc2-db \
  --database-version=POSTGRES_14 \
  --region=us-central1 \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/soc2-keyring/cryptoKeys/soc2-key \
  --require-ssl \
  --deletion-protection

# Availability: High availability configuration
gcloud sql instances patch soc2-db \
  --availability-type=REGIONAL \
  --backup-start-time=02:00 \
  --retained-backups-count=30 \
  --enable-point-in-time-recovery

# Processing Integrity: Enable query monitoring
gcloud sql instances patch soc2-db \
  --insights-config-query-insights-enabled \
  --insights-config-query-string-length=4500

# Confidentiality: VPC Service Controls
gcloud access-context-manager perimeters create soc2_perimeter \
  --policy=POLICY_ID \
  --resources=projects/PROJECT_NUMBER \
  --restricted-services=sqladmin.googleapis.com,spanner.googleapis.com

# Privacy: Data classification and DLP
gcloud dlp inspect-templates create privacy-template \
  --display-name="SOC 2 Privacy Inspection" \
  --min-likelihood=POSSIBLE \
  --include-quote=true \
  --info-types=EMAIL_ADDRESS,PHONE_NUMBER,PERSON_NAME
```

**Exam Tip**: SOC 2 focuses on five trust service criteria: Security, Availability, Processing Integrity, Confidentiality, and Privacy. Implement controls for each area using GCP database security features.

## Data Protection and Backup Security

### Backup Encryption and Retention

```bash
# Cloud SQL automated backups with CMEK
gcloud sql instances create backup-secure-db \
  --database-version=POSTGRES_14 \
  --region=us-central1 \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/backup-keyring/cryptoKeys/backup-key \
  --backup-start-time=02:00 \
  --retained-backups-count=60 \
  --retained-transaction-log-days=7 \
  --enable-point-in-time-recovery

# On-demand backup
gcloud sql backups create \
  --instance=backup-secure-db \
  --description="Pre-maintenance backup"

# Spanner backup with CMEK
gcloud spanner backups create quarterly-backup \
  --instance=prod-spanner \
  --database=customer-db \
  --retention-period=90d \
  --version-time=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# Export encrypted backup to Cloud Storage
gcloud sql export sql backup-secure-db gs://secure-backups/backup-$(date +%Y%m%d).sql \
  --database=production_db

# Encrypt exports with CSEK
gsutil -o "GSUtil:encryption_key=$(cat encryption-key.txt)" \
  cp local-backup.sql gs://secure-backups/encrypted-backup.sql
```

### Point-in-Time Recovery (PITR)

```bash
# Enable PITR for Cloud SQL
gcloud sql instances patch prod-db \
  --enable-point-in-time-recovery \
  --retained-transaction-log-days=7

# Restore to specific point in time
gcloud sql instances clone prod-db recovery-instance \
  --point-in-time='2024-12-15T10:30:00Z'

# Spanner PITR
gcloud spanner databases create recovery-db \
  --instance=prod-spanner \
  --clone=customer-db \
  --clone-time=2024-12-15T10:30:00Z
```

### Deletion Protection

```bash
# Enable deletion protection
gcloud sql instances patch critical-db \
  --deletion-protection

# Prevent accidental deletion via IAM
gcloud sql instances set-iam-policy critical-db deletion-protection-policy.yaml
```

```yaml
# deletion-protection-policy.yaml
bindings:
- role: roles/cloudsql.admin
  members:
  - group:dba-team@example.com
  condition:
    title: "Prevent deletion"
    expression: |
      api.getAttribute('cloudsql.googleapis.com/instance.delete', '') == ''
```

**Exam Tip**: Always enable deletion protection for production databases. Use CMEK for backup encryption. Enable PITR for Cloud SQL for faster recovery. Test recovery procedures regularly as part of DR planning.

## Comprehensive Security Scenarios

### Scenario 1: Multi-Tenant SaaS with Data Isolation

**Requirements**:
- Separate data per tenant
- Ensure no cross-tenant data access
- Tenant-level encryption keys
- Per-tenant audit logging
- Cost allocation per tenant

**Solution Architecture**:
```bash
# Option 1: Separate database per tenant
for tenant in tenant_a tenant_b tenant_c; do
  # Create tenant-specific KMS key
  gcloud kms keys create ${tenant}-key \
    --keyring=tenant-keyring \
    --location=us-central1 \
    --purpose=encryption

  # Create tenant database
  gcloud sql instances create ${tenant}-db \
    --database-version=POSTGRES_14 \
    --tier=db-custom-2-7680 \
    --region=us-central1 \
    --network=projects/PROJECT/global/networks/tenant-vpc \
    --no-assign-ip \
    --require-ssl \
    --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/tenant-keyring/cryptoKeys/${tenant}-key \
    --labels=tenant=${tenant},environment=production

  # Tenant-specific IAM
  gcloud sql instances set-iam-policy ${tenant}-db ${tenant}-iam-policy.yaml

  # Tenant-specific audit log sink
  gcloud logging sinks create ${tenant}-audit-sink \
    bigquery.googleapis.com/projects/PROJECT/datasets/${tenant}_audit_logs \
    --log-filter="resource.type=\"cloudsql_database\" AND resource.labels.instance_id=\"${tenant}-db\""
done

# Option 2: Shared database with row-level security
# PostgreSQL Row-Level Security (RLS)
CREATE TABLE tenant_data (
    id SERIAL PRIMARY KEY,
    tenant_id VARCHAR(50) NOT NULL,
    data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

-- Create policy for tenant isolation
CREATE POLICY tenant_isolation_policy ON tenant_data
    USING (tenant_id = current_setting('app.current_tenant')::TEXT);

-- Application sets tenant context
SET app.current_tenant = 'tenant_a';

-- Users can only see their tenant's data
SELECT * FROM tenant_data;  -- Only returns tenant_a data
```

**Exam Tip**: For multi-tenant architectures, separate databases provide strongest isolation but higher cost. Shared database with RLS is cost-effective but requires careful implementation. Use labels for cost allocation and tenant identification.

### Scenario 2: Financial Services with PCI-DSS Compliance

**Requirements**:
- Store payment card data securely
- Meet PCI-DSS requirements
- Cardholder Data Environment (CDE) isolation
- Tokenization for card numbers
- Comprehensive audit logging

**Solution**:
```bash
# Create isolated VPC for CDE
gcloud compute networks create pci-cde-vpc \
  --subnet-mode=custom

gcloud compute networks subnets create pci-cde-subnet \
  --network=pci-cde-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24 \
  --enable-private-ip-google-access

# Create PCI-compliant database in CDE
gcloud sql instances create pci-payment-db \
  --database-version=POSTGRES_14 \
  --tier=db-n1-highmem-4 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/pci-cde-vpc \
  --no-assign-ip \
  --require-ssl \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/pci-keyring/cryptoKeys/payment-key \
  --database-flags=cloudsql.enable_pgaudit=on,pgaudit.log=all,log_connections=on,log_disconnections=on \
  --backup-start-time=02:00 \
  --retained-backups-count=90 \
  --deletion-protection

# VPC Service Controls for CDE perimeter
gcloud access-context-manager perimeters create pci_cde_perimeter \
  --policy=POLICY_ID \
  --title="PCI CDE Perimeter" \
  --resources=projects/PCI_PROJECT_NUMBER \
  --restricted-services=sqladmin.googleapis.com,storage.googleapis.com \
  --access-levels=pci_cde_access \
  --enable-vpc-accessible-services \
  --vpc-allowed-services=sqladmin.googleapis.com

# Firewall rules for CDE
gcloud compute firewall-rules create deny-all-ingress-pci-cde \
  --network=pci-cde-vpc \
  --action=DENY \
  --rules=all \
  --source-ranges=0.0.0.0/0 \
  --priority=1000

gcloud compute firewall-rules create allow-app-to-db-pci-cde \
  --network=pci-cde-vpc \
  --action=ALLOW \
  --rules=tcp:5432 \
  --source-tags=payment-app \
  --target-tags=payment-db \
  --priority=500
```

**Payment Data Handler**:
```python
import hashlib
from google.cloud import kms, secretmanager

class PCICompliantPaymentHandler:
    def __init__(self, project_id):
        self.project_id = project_id
        self.kms_client = kms.KeyManagementServiceClient()
        self.kms_key_name = f"projects/{project_id}/locations/us-central1/keyRings/pci-keyring/cryptoKeys/payment-key"

    def tokenize_pan(self, pan):
        """Create token for Primary Account Number (PAN)"""
        # Use one-way hash for tokenization
        token = hashlib.sha256(f"{pan}{SECRET_SALT}".encode()).hexdigest()
        return token

    def store_card_data(self, pan, cvv, expiry, cardholder_name):
        """Store card data with PCI compliance"""
        # Tokenize PAN
        token = self.tokenize_pan(pan)

        # Encrypt full PAN with KMS (for refunds/disputes only)
        encrypted_pan = self.kms_client.encrypt(
            request={
                'name': self.kms_key_name,
                'plaintext': pan.encode()
            }
        ).ciphertext

        # Never store CVV (PCI requirement)
        # Store only last 4 digits of PAN and token
        last_four = pan[-4:]

        with db.connect() as conn:
            cursor = conn.cursor()
            cursor.execute("""
                INSERT INTO payment_tokens
                (token, last_four, encrypted_pan, expiry, cardholder_name, created_at)
                VALUES (%s, %s, %s, %s, %s, NOW())
                RETURNING token_id
            """, (token, last_four, encrypted_pan, expiry, cardholder_name))

            token_id = cursor.fetchone()[0]

        # Log card storage event
        self.log_pci_event('CARD_STORED', token_id)

        return token

    def process_payment(self, token, amount):
        """Process payment using token"""
        # Retrieve encrypted PAN
        with db.connect() as conn:
            cursor = conn.cursor()
            cursor.execute("""
                SELECT encrypted_pan, expiry
                FROM payment_tokens
                WHERE token = %s
            """, (token,))

            result = cursor.fetchone()
            if not result:
                raise ValueError("Invalid token")

            encrypted_pan, expiry = result

        # Decrypt PAN for payment processor
        pan = self.kms_client.decrypt(
            request={
                'name': self.kms_key_name,
                'ciphertext': encrypted_pan
            }
        ).plaintext.decode()

        # Log payment processing
        self.log_pci_event('PAYMENT_PROCESSED', token, amount)

        # Send to payment processor (implementation specific)
        return self.call_payment_processor(pan, expiry, amount)

    def log_pci_event(self, event_type, *args):
        """Log PCI-relevant events for audit"""
        with db.connect() as conn:
            cursor = conn.cursor()
            cursor.execute("""
                INSERT INTO pci_audit_log
                (timestamp, event_type, details, user_id, ip_address)
                VALUES (NOW(), %s, %s, %s, %s)
            """, (event_type, str(args), get_current_user(), get_client_ip()))
```

### Scenario 3: Healthcare Platform with HIPAA Compliance

**Requirements**:
- Store PHI (Protected Health Information)
- HIPAA compliance requirements
- BAA with Google Cloud
- Minimum necessary access
- Comprehensive audit trails

**Solution**:
```bash
# Create HIPAA-compliant infrastructure
gcloud sql instances create hipaa-ehr-db \
  --database-version=POSTGRES_14 \
  --tier=db-custom-8-32768 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/healthcare-vpc \
  --no-assign-ip \
  --require-ssl \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/hipaa-keyring/cryptoKeys/phi-key \
  --database-flags=cloudsql.enable_pgaudit=on,pgaudit.log=all,log_statement=ddl,log_connections=on \
  --availability-type=REGIONAL \
  --backup-start-time=03:00 \
  --retained-backups-count=365 \
  --deletion-protection

# Create Spanner for patient records
gcloud spanner instances create hipaa-patient-records \
  --config=regional-us-central1 \
  --nodes=3 \
  --kms-key=projects/PROJECT/locations/us-central1/keyRings/hipaa-keyring/cryptoKeys/phi-key \
  --description="HIPAA-compliant patient records"

gcloud spanner databases create patient_db \
  --instance=hipaa-patient-records \
  --ddl-file=patient_schema.sql
```

**Patient Data Access Control**:
```python
from google.cloud import spanner
from enum import Enum
import logging

class AccessReason(Enum):
    TREATMENT = "treatment"
    PAYMENT = "payment"
    OPERATIONS = "healthcare_operations"
    RESEARCH = "research_irb_approved"
    EMERGENCY = "emergency"

class HIPAAPatientDataAccess:
    def __init__(self, spanner_instance_id, database_id):
        self.spanner_client = spanner.Client()
        self.instance = self.spanner_client.instance(spanner_instance_id)
        self.database = self.instance.database(database_id)

    def access_patient_record(self, provider_id, patient_id, reason: AccessReason):
        """Access patient record with HIPAA compliance"""
        # Verify provider authorization
        if not self.verify_provider_access(provider_id, patient_id, reason):
            self.log_access_denial(provider_id, patient_id, reason)
            raise PermissionError("Provider not authorized for this patient")

        # Check for break-the-glass emergency access
        if reason == AccessReason.EMERGENCY:
            self.alert_compliance_team(provider_id, patient_id)

        # Log access (HIPAA requirement)
        self.log_phi_access(provider_id, patient_id, reason)

        # Retrieve minimum necessary information
        with self.database.snapshot() as snapshot:
            results = snapshot.execute_sql("""
                SELECT patient_id, name, dob, medical_record_number,
                       current_medications, allergies, active_diagnoses
                FROM patient_records
                WHERE patient_id = @patient_id
            """, params={'patient_id': patient_id},
                param_types={'patient_id': spanner.param_types.STRING})

            patient_data = list(results)[0]

        return patient_data

    def verify_provider_access(self, provider_id, patient_id, reason):
        """Verify provider has legitimate relationship with patient"""
        with self.database.snapshot() as snapshot:
            # Check if provider is assigned to patient
            results = snapshot.execute_sql("""
                SELECT COUNT(*) as count
                FROM patient_provider_relationships
                WHERE provider_id = @provider_id
                AND patient_id = @patient_id
                AND relationship_type = @reason
                AND relationship_end_date IS NULL
            """, params={
                'provider_id': provider_id,
                'patient_id': patient_id,
                'reason': reason.value
            }, param_types={
                'provider_id': spanner.param_types.STRING,
                'patient_id': spanner.param_types.STRING,
                'reason': spanner.param_types.STRING
            })

            count = list(results)[0][0]
            return count > 0 or reason == AccessReason.EMERGENCY

    def log_phi_access(self, provider_id, patient_id, reason):
        """Log PHI access for HIPAA audit trail"""
        with self.database.batch() as batch:
            batch.insert(
                table='phi_access_log',
                columns=('log_id', 'timestamp', 'provider_id', 'patient_id',
                        'access_reason', 'ip_address', 'user_agent'),
                values=[(
                    generate_uuid(),
                    spanner.COMMIT_TIMESTAMP,
                    provider_id,
                    patient_id,
                    reason.value,
                    get_client_ip(),
                    get_user_agent()
                )]
            )

    def de_identify_for_research(self, patient_ids):
        """De-identify PHI for research purposes"""
        from google.cloud import dlp_v2

        dlp = dlp_v2.DlpServiceClient()

        # Configure de-identification
        deidentify_config = {
            'record_transformations': {
                'field_transformations': [
                    {
                        'fields': [{'name': 'name'}, {'name': 'address'}],
                        'primitive_transformation': {
                            'replace_config': {
                                'new_value': {'string_value': '[REDACTED]'}
                            }
                        }
                    },
                    {
                        'fields': [{'name': 'dob'}],
                        'primitive_transformation': {
                            'date_shift_config': {
                                'upper_bound_days': 50,
                                'lower_bound_days': -50
                            }
                        }
                    }
                ]
            }
        }

        # Apply de-identification to dataset
        # Implementation specific to data format
        pass
```

### Scenario 4: Global E-Commerce with Multi-Region Compliance

**Requirements**:
- Store customer data in appropriate regions (GDPR, data residency)
- PCI-DSS for payment processing
- Data replication for performance
- Region-specific compliance

**Solution**:
```bash
# US region for US customers
gcloud sql instances create ecommerce-us-db \
  --database-version=POSTGRES_14 \
  --tier=db-n1-highmem-4 \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/global-vpc \
  --no-assign-ip \
  --require-ssl \
  --disk-encryption-key=projects/PROJECT/locations/us-central1/keyRings/us-keyring/cryptoKeys/us-key

# EU region for EU customers (GDPR)
gcloud sql instances create ecommerce-eu-db \
  --database-version=POSTGRES_14 \
  --tier=db-n1-highmem-4 \
  --region=europe-west1 \
  --network=projects/PROJECT/global/networks/global-vpc \
  --no-assign-ip \
  --require-ssl \
  --disk-encryption-key=projects/PROJECT/locations/europe-west1/keyRings/eu-keyring/cryptoKeys/eu-key

# Global Spanner for order processing
gcloud spanner instances create global-orders \
  --config=nam-eur-asia1 \
  --processing-units=1000 \
  --kms-key=projects/PROJECT/locations/us-central1/keyRings/global-keyring/cryptoKeys/orders-key

# Database schema with region-specific data placement
CREATE TABLE customers (
    customer_id VARCHAR(36) PRIMARY KEY,
    region VARCHAR(10) NOT NULL,  -- US, EU, ASIA
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP,
    data_residency_region VARCHAR(20)
) INTERLEAVE IN PARENT customer_regions;

# Application routing logic
def get_customer_database(customer_region):
    """Route to appropriate regional database"""
    db_mapping = {
        'US': 'ecommerce-us-db',
        'EU': 'ecommerce-eu-db',
        'ASIA': 'ecommerce-asia-db'
    }
    return connect_to_database(db_mapping[customer_region])
```

### Scenario 5: SaaS Application with API Key Management

**Requirements**:
- Secure API key storage
- API key rotation
- Usage auditing
- Rate limiting per key

**Solution**:
```python
from google.cloud import secretmanager, firestore
import hashlib
import secrets

class APIKeyManager:
    def __init__(self, project_id):
        self.project_id = project_id
        self.secret_client = secretmanager.SecretManagerServiceClient()
        self.firestore_client = firestore.Client(project=project_id)

    def create_api_key(self, customer_id, key_name, permissions):
        """Create new API key for customer"""
        # Generate cryptographically secure API key
        api_key = f"sk_{secrets.token_urlsafe(32)}"

        # Hash API key for storage
        key_hash = hashlib.sha256(api_key.encode()).hexdigest()

        # Store hash in Firestore with metadata
        api_key_doc = self.firestore_client.collection('api_keys').document(key_hash)
        api_key_doc.set({
            'customer_id': customer_id,
            'key_name': key_name,
            'permissions': permissions,
            'created_at': firestore.SERVER_TIMESTAMP,
            'last_used_at': None,
            'usage_count': 0,
            'rate_limit': 1000,  # requests per hour
            'is_active': True
        })

        # Store full key in Secret Manager (for emergency recovery)
        secret_name = f"api-key-{key_hash[:16]}"
        parent = f"projects/{self.project_id}"
        self.secret_client.create_secret(
            request={
                'parent': parent,
                'secret_id': secret_name,
                'secret': {'replication': {'automatic': {}}}
            }
        )

        self.secret_client.add_secret_version(
            request={
                'parent': f"{parent}/secrets/{secret_name}",
                'payload': {'data': api_key.encode()}
            }
        )

        # Return API key (only shown once)
        return api_key

    def validate_api_key(self, api_key):
        """Validate API key and log usage"""
        key_hash = hashlib.sha256(api_key.encode()).hexdigest()

        api_key_doc = self.firestore_client.collection('api_keys').document(key_hash).get()

        if not api_key_doc.exists:
            self.log_invalid_key_attempt(api_key)
            return None

        key_data = api_key_doc.to_dict()

        if not key_data['is_active']:
            return None

        # Check rate limit
        if not self.check_rate_limit(key_hash, key_data):
            raise RateLimitExceeded("API rate limit exceeded")

        # Update usage statistics
        api_key_doc.reference.update({
            'last_used_at': firestore.SERVER_TIMESTAMP,
            'usage_count': firestore.Increment(1)
        })

        return key_data

    def rotate_api_key(self, old_key_hash):
        """Rotate API key (create new, deprecate old)"""
        old_key_doc = self.firestore_client.collection('api_keys').document(old_key_hash).get()
        old_key_data = old_key_doc.to_dict()

        # Create new key with same permissions
        new_key = self.create_api_key(
            old_key_data['customer_id'],
            old_key_data['key_name'] + '_rotated',
            old_key_data['permissions']
        )

        # Deprecate old key (don't delete for audit trail)
        old_key_doc.reference.update({
            'is_active': False,
            'deprecated_at': firestore.SERVER_TIMESTAMP,
            'replaced_by': hashlib.sha256(new_key.encode()).hexdigest()
        })

        return new_key
```

### Scenario 6: Real-Time Analytics with Data Masking

**Requirements**:
- Real-time analytics on customer data
- Mask PII for analysts
- Different access levels for different roles
- Performance requirements

**Solution**:
```sql
-- PostgreSQL dynamic data masking views
CREATE VIEW customer_analytics_masked AS
SELECT
    customer_id,
    CASE
        WHEN current_setting('app.user_role', TRUE) = 'admin'
        THEN email
        ELSE regexp_replace(email, '(.{2}).*(@.*)', '\1***\2')
    END AS email,
    CASE
        WHEN current_setting('app.user_role', TRUE) = 'admin'
        THEN phone
        ELSE '***-***-' || right(phone, 4)
    END AS phone,
    total_purchases,
    total_spent,
    last_purchase_date,
    customer_segment
FROM customers;

-- Spanner with fine-grained access views
CREATE VIEW analyst_customer_view AS
SELECT
    customer_id,
    -- Anonymized identifiers
    FARM_FINGERPRINT(email) AS email_hash,
    FARM_FINGERPRINT(phone) AS phone_hash,
    -- Aggregate data only
    purchase_count,
    total_revenue,
    avg_order_value,
    customer_lifetime_value,
    -- Geographic data at region level only
    SUBSTR(postal_code, 1, 3) AS postal_code_prefix,
    state
FROM customer_details;

-- Grant access to masked view only
GRANT SELECT ON customer_analytics_masked TO ROLE analyst;
REVOKE SELECT ON customers FROM ROLE analyst;
```

**Python Data Masking**:
```python
from google.cloud import bigquery

class DataMaskingService:
    def __init__(self, project_id):
        self.bq_client = bigquery.Client(project=project_id)

    def query_with_masking(self, query, user_role):
        """Execute query with role-based data masking"""
        # Apply masking based on role
        if user_role == 'analyst':
            masked_query = self.apply_analyst_masking(query)
        elif user_role == 'data_scientist':
            masked_query = self.apply_data_scientist_masking(query)
        else:
            masked_query = query

        return self.bq_client.query(masked_query).result()

    def apply_analyst_masking(self, query):
        """Apply masking for analyst role"""
        # Use BigQuery's data masking functions
        masking_rules = {
            'email': "REGEXP_REPLACE(email, r'(.{{2}}).*(@.*)', r'\\1***\\2')",
            'ssn': "CONCAT('***-**-', SUBSTR(ssn, -4))",
            'credit_card': "CONCAT('****-****-****-', SUBSTR(credit_card, -4))"
        }

        # Apply masking to query (simplified)
        for field, mask_expr in masking_rules.items():
            query = query.replace(field, f"{mask_expr} AS {field}")

        return query
```

### Scenario 7: Incident Response and Forensics

**Requirements**:
- Detect security incidents
- Forensic analysis capabilities
- Automated response
- Compliance reporting

**Solution**:
```python
from google.cloud import logging_v2, monitoring_v3
import json

class DatabaseSecurityIncidentResponse:
    def __init__(self, project_id):
        self.project_id = project_id
        self.logging_client = logging_v2.Client(project=project_id)
        self.monitoring_client = monitoring_v3.MetricServiceClient()

    def detect_sql_injection_attempt(self):
        """Detect potential SQL injection in query logs"""
        filter_str = '''
        resource.type="cloudsql_database"
        AND (
            textPayload=~".*(\\'|\\-\\-|;DROP|UNION SELECT).*"
            OR jsonPayload.statement=~".*(\\'|\\-\\-|;DROP|UNION SELECT).*"
        )
        '''

        entries = self.logging_client.list_entries(filter_=filter_str)

        incidents = []
        for entry in entries:
            incident = {
                'timestamp': entry.timestamp,
                'user': entry.labels.get('principalEmail'),
                'query': entry.json_payload.get('statement'),
                'source_ip': entry.http_request.remote_ip if entry.http_request else None,
                'severity': 'HIGH'
            }
            incidents.append(incident)

            # Trigger automated response
            self.respond_to_sql_injection(incident)

        return incidents

    def detect_unauthorized_access(self):
        """Detect unauthorized data access attempts"""
        filter_str = '''
        resource.type="cloudsql_database"
        AND protoPayload.status.code!=0
        AND protoPayload.methodName=~".*connect.*"
        '''

        entries = self.logging_client.list_entries(filter_=filter_str, max_results=100)

        # Detect brute force attempts
        failed_attempts = {}
        for entry in entries:
            user = entry.proto_payload.authentication_info.principal_email
            failed_attempts[user] = failed_attempts.get(user, 0) + 1

        # Alert on multiple failed attempts
        for user, count in failed_attempts.items():
            if count > 10:
                self.alert_security_team({
                    'type': 'BRUTE_FORCE',
                    'user': user,
                    'failed_attempts': count,
                    'severity': 'CRITICAL'
                })

                # Automated response: temporarily block user
                self.block_user_access(user, duration_minutes=30)

    def detect_data_exfiltration(self):
        """Detect large data exports"""
        filter_str = '''
        resource.type="cloudsql_database"
        AND protoPayload.methodName="cloudsql.instances.export"
        '''

        entries = self.logging_client.list_entries(filter_=filter_str)

        for entry in entries:
            export_size = entry.proto_payload.metadata.get('exportContext', {}).get('dataSize')

            if export_size and int(export_size) > 10 * 1024 * 1024 * 1024:  # > 10 GB
                self.alert_security_team({
                    'type': 'LARGE_DATA_EXPORT',
                    'user': entry.proto_payload.authentication_info.principal_email,
                    'size_gb': int(export_size) / (1024**3),
                    'destination': entry.proto_payload.metadata.get('exportContext', {}).get('uri'),
                    'severity': 'HIGH'
                })

    def respond_to_sql_injection(self, incident):
        """Automated response to SQL injection attempt"""
        # 1. Block the user
        self.block_user_access(incident['user'], duration_minutes=60)

        # 2. Alert security team
        self.alert_security_team(incident)

        # 3. Create incident ticket
        self.create_incident_ticket(incident)

        # 4. Preserve forensic evidence
        self.preserve_forensic_evidence(incident)

    def generate_forensic_report(self, incident_id, start_time, end_time):
        """Generate comprehensive forensic report"""
        report = {
            'incident_id': incident_id,
            'timeline': [],
            'affected_data': [],
            'user_actions': [],
            'recommendations': []
        }

        # Collect all relevant logs
        filter_str = f'''
        resource.type="cloudsql_database"
        AND timestamp>="{start_time}"
        AND timestamp<="{end_time}"
        '''

        entries = self.logging_client.list_entries(filter_=filter_str)

        for entry in entries:
            report['timeline'].append({
                'timestamp': str(entry.timestamp),
                'action': entry.proto_payload.method_name,
                'user': entry.proto_payload.authentication_info.principal_email,
                'details': str(entry.proto_payload)
            })

        # Analyze impact
        report['recommendations'] = self.generate_remediation_steps(report)

        return report
```

### Scenario 8: Zero-Trust Database Access

**Requirements**:
- Verify every access request
- Context-aware access control
- Device trust
- Continuous authentication

**Solution**:
```python
from google.cloud import iap
from google.auth.transport import requests
import jwt

class ZeroTrustDatabaseAccess:
    def __init__(self, project_id):
        self.project_id = project_id

    def verify_iap_jwt(self, iap_jwt, expected_audience):
        """Verify IAP JWT token"""
        try:
            decoded_jwt = jwt.decode(
                iap_jwt,
                algorithms=['ES256'],
                audience=expected_audience,
                options={'verify_signature': True}
            )

            return decoded_jwt
        except Exception as e:
            raise PermissionError(f"Invalid IAP JWT: {e}")

    def check_device_trust(self, device_id):
        """Verify device is trusted"""
        # Integrate with endpoint verification
        # Check device certificates, encryption status, etc.
        pass

    def evaluate_access_context(self, user_id, resource, context):
        """Context-aware access control"""
        score = 0

        # Check user risk score
        user_risk = self.get_user_risk_score(user_id)
        if user_risk < 0.3:
            score += 40

        # Check location
        if context.get('ip_address') in self.get_trusted_ip_ranges():
            score += 20

        # Check time of access
        if self.is_business_hours(context.get('timestamp')):
            score += 10

        # Check device trust
        if context.get('device_trusted'):
            score += 20

        # Check recent authentication
        if context.get('auth_age_minutes', 999) < 60:
            score += 10

        # Require score >= 70 for access
        return score >= 70

    def enforce_step_up_authentication(self, user_id, resource_sensitivity):
        """Require additional authentication for sensitive resources"""
        if resource_sensitivity == 'HIGH':
            # Require MFA
            if not self.verify_mfa(user_id):
                raise PermissionError("MFA required for high-sensitivity resources")

        if resource_sensitivity == 'CRITICAL':
            # Require MFA + approval
            if not self.verify_mfa(user_id) or not self.check_approval(user_id, resource):
                raise PermissionError("MFA and approval required for critical resources")

    def continuous_authentication(self, session_id):
        """Continuously verify user throughout session"""
        session = self.get_session(session_id)

        # Re-verify every 15 minutes
        if (datetime.now() - session['last_verification']).minutes > 15:
            # Check if user behavior matches baseline
            if not self.verify_behavior_pattern(session['user_id']):
                self.terminate_session(session_id)
                raise PermissionError("Anomalous behavior detected")

            session['last_verification'] = datetime.now()
```

## Professional Database Engineer Exam Tips

### Security Decision Framework

**When choosing encryption options**:
1. **Google-managed encryption (default)**: Suitable for most use cases, no additional cost or latency
2. **CMEK**: Required for:
   - Regulatory compliance (PCI-DSS, HIPAA)
   - Key lifecycle control needed
   - Multi-tenant key isolation
   - Audit requirements for key usage
3. **Application-level encryption**: Required for:
   - Field-level encryption needs
   - End-to-end encryption requirements
   - Zero-knowledge architecture

**When choosing network configuration**:
1. **Private IP only**: Preferred for production, requires VPC setup
2. **Private IP + Cloud SQL Proxy**: Best balance of security and ease of use
3. **Public IP + Authorized Networks**: Temporary access only
4. **Public IP + Cloud SQL Proxy**: Development/testing scenarios

**When implementing access control**:
1. Use IAM authentication over password-based whenever possible
2. Implement least privilege with custom IAM roles
3. Use IAM conditions for time-based and attribute-based access
4. Separate admin and application service accounts
5. Regular access reviews (quarterly minimum)

**When enabling audit logging**:
1. Always enable Admin Activity logs (free, enabled by default)
2. Enable Data Access logs for:
   - Production databases
   - Databases with sensitive data
   - Compliance requirements (PCI, HIPAA, SOC 2)
3. Export to BigQuery for long-term retention and analysis
4. Create log-based metrics for security events
5. Set up alerting for critical security events

**Compliance framework selection**:
- **PCI-DSS**: Payment card data - CMEK, network isolation, VPC-SC, audit logs
- **HIPAA**: Healthcare data - BAA, CMEK, 7-year audit retention, DLP
- **GDPR**: EU personal data - EU regions, data subject rights, consent tracking
- **SOC 2**: SaaS trust - All five trust service criteria controls

**Exam patterns to recognize**:
1. Scenario asks for "most secure" → Usually private IP + CMEK + IAM auth + VPC-SC
2. Scenario mentions "compliance" → Enable comprehensive audit logging first
3. Scenario mentions "cost optimization" → Consider default encryption vs CMEK trade-off
4. Scenario mentions "multi-tenant" → Consider separate databases or RLS
5. Scenario mentions "minimal access" → Use IAM conditions and custom roles
6. Scenario asks about "encryption in transit" → Require SSL/TLS, use Cloud SQL Proxy
7. Scenario mentions "sensitive data" → Consider application-level encryption + DLP
8. Scenario asks about "incident response" → Export audit logs to BigQuery, create alerts

### Common Exam Question Patterns

**Pattern 1**: "A healthcare company needs to store patient records..."
- Answer involves: HIPAA, BAA, CMEK, audit logs, DLP, private networking

**Pattern 2**: "A payment processing application requires..."
- Answer involves: PCI-DSS, CMEK, tokenization, VPC-SC, network isolation

**Pattern 3**: "A company with EU customers must..."
- Answer involves: GDPR, EU regions, data subject rights, organization policies

**Pattern 4**: "Database access needs to be restricted to..."
- Answer involves: IAM conditions, custom roles, time-based access, IP restrictions

**Pattern 5**: "Detecting unauthorized access to databases..."
- Answer involves: Audit logs, log-based metrics, alerting, Security Command Center

**Pattern 6**: "Multi-tenant SaaS application with data isolation..."
- Answer involves: Separate databases vs RLS, per-tenant CMEK, tenant-specific IAM

### Key Services and Features to Know

**Cloud SQL Security**:
- IAM database authentication (PostgreSQL, MySQL)
- Private IP and VPC peering
- Cloud SQL Proxy
- SSL/TLS with client certificates
- CMEK for encryption at rest
- Automated backups with encryption
- pgAudit for PostgreSQL
- Deletion protection
- Point-in-time recovery

**Cloud Spanner Security**:
- IAM at instance and database level
- Fine-grained access with IAM conditions
- CMEK encryption
- Automatic encryption in transit
- VPC Service Controls support
- Private Service Connect (PSC)
- Audit logging
- Backup encryption with CMEK

**Firestore Security**:
- Security rules for client access
- IAM for server-side access
- CMEK encryption
- Rules versioning
- Test framework for security rules
- Hierarchical security
- Custom authentication claims

**Bigtable Security**:
- IAM at instance level
- Application-level row security
- CMEK encryption per cluster
- VPC Service Controls
- Private networking via VPC
- Replication encryption

**Security Tools**:
- Cloud KMS for CMEK management
- Secret Manager for secrets
- Cloud DLP for sensitive data detection
- VPC Service Controls for perimeter security
- Security Command Center for findings
- Cloud Armor for DDoS protection
- Cloud Audit Logs for compliance

### Study Approach

1. **Hands-on practice** (Critical):
   - Create databases with CMEK
   - Configure IAM authentication
   - Set up private IP and Cloud SQL Proxy
   - Enable and query audit logs
   - Implement VPC Service Controls
   - Practice with Cloud DLP

2. **Scenario practice**:
   - Work through all 8 scenarios in this guide
   - Understand when to use each security control
   - Practice making trade-off decisions

3. **Compliance frameworks**:
   - Understand requirements for PCI-DSS, HIPAA, GDPR, SOC 2
   - Know which GCP services are eligible for each compliance framework
   - Understand BAA requirements

4. **Security patterns**:
   - Defense in depth
   - Least privilege
   - Zero trust architecture
   - Encryption at rest and in transit
   - Comprehensive audit logging

## Additional Resources

- [Cloud SQL Security Best Practices](https://cloud.google.com/sql/docs/postgres/security)
- [Cloud Spanner IAM Documentation](https://cloud.google.com/spanner/docs/iam)
- [Firestore Security Rules Guide](https://firebase.google.com/docs/firestore/security/get-started)
- [Customer-Managed Encryption Keys (CMEK)](https://cloud.google.com/kms/docs/cmek)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
- [Cloud DLP Documentation](https://cloud.google.com/dlp/docs)
- [GCP Compliance Resource Center](https://cloud.google.com/security/compliance)
- [Professional Cloud Database Engineer Exam Guide](https://cloud.google.com/certification/cloud-database-engineer)
- [Database Security Whitepapers](https://cloud.google.com/security/encryption-at-rest)
