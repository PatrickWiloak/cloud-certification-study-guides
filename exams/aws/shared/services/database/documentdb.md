# Amazon DocumentDB

## Service Overview and Purpose

Amazon DocumentDB is a fully managed document database service that supports MongoDB workloads. DocumentDB provides the scalability, durability, and availability needed for mission-critical MongoDB applications while being fully managed by AWS.

### Key Characteristics
- **MongoDB Compatible**: Supports MongoDB 3.6, 4.0, and 5.0 APIs
- **Fully Managed**: Automated backups, patching, monitoring
- **Elastic Scaling**: Scale compute and storage independently
- **High Availability**: Multi-AZ deployments with failover
- **Secure**: Encryption, VPC isolation, audit logging

## Key Features and Capabilities

### Core Features
- **Document Storage**: Flexible schema JSON documents
- **MongoDB API Compatibility**: Use existing MongoDB drivers and tools
- **ACID Transactions**: Multi-document ACID transactions
- **Elastic Scaling**: Scale read capacity with read replicas
- **Change Streams**: Real-time data change notifications

### Architecture
- **Cluster-Based**: Primary instance and up to 15 read replicas
- **Storage**: Distributed, fault-tolerant, self-healing storage
- **Separation of Compute and Storage**: Independent scaling
- **Multi-AZ**: Automatic failover across availability zones

## Configuration and Best Practices

### Cluster Creation
```bash
aws docdb create-db-cluster \
  --db-cluster-identifier docdb-prod-cluster \
  --engine docdb \
  --engine-version 5.0.0 \
  --master-username admin \
  --master-user-password SecurePassword123! \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name docdb-subnet-group \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:05:00" \
  --storage-encrypted \
  --kms-key-id alias/docdb-encryption-key \
  --enable-cloudwatch-logs-exports audit,profiler \
  --deletion-protection \
  --tags Key=Environment,Value=Production
```

### Application Integration
```python
from pymongo import MongoClient
import ssl

# Connect to DocumentDB
client = MongoClient(
    host='docdb-cluster.cluster-xyz.us-west-2.docdb.amazonaws.com:27017',
    username='admin',
    password='password',
    ssl=True,
    ssl_ca_certs='rds-ca-2019-root.pem',
    ssl_cert_reqs=ssl.CERT_REQUIRED,
    retryWrites=False
)

# Database operations
db = client.myapp
collection = db.users

# Insert document
user_doc = {
    "user_id": "12345",
    "name": "John Doe",
    "email": "john@example.com",
    "preferences": {
        "theme": "dark",
        "notifications": True
    },
    "created_at": datetime.utcnow()
}
result = collection.insert_one(user_doc)

# Query documents
users = collection.find({"preferences.theme": "dark"})

# Update document
collection.update_one(
    {"user_id": "12345"},
    {"$set": {"preferences.theme": "light"}}
)
```

## Use Cases and Scenarios

### Primary Use Cases
1. **Content Management**: Flexible schema for varied content types
2. **Catalogs**: Product catalogs with varying attributes
3. **User Profiles**: Complex user data with nested attributes
4. **Real-Time Analytics**: Event data and time-series information
5. **Mobile and Web Apps**: JSON-based application data

### Common Exam Scenarios
- **MongoDB Migration**: Migrating existing MongoDB applications
- **Flexible Schema Requirements**: Applications needing schema flexibility
- **High Availability**: Critical applications requiring automatic failover
- **Compliance**: Applications requiring encryption and audit logging

## Exam Tips
- **Know MongoDB compatibility** and supported versions
- **Understand cluster architecture** with primary and read replicas
- **Remember security features** like encryption and VPC isolation
- **Know when to choose** DocumentDB over other database options
- **Understand backup and restore** capabilities