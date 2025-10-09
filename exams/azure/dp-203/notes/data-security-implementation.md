# Design and Implement Data Security

## Overview
This domain covers designing and implementing security policies, standards, and practices for data platforms. It represents 10-15% of the exam and focuses on protecting data throughout its lifecycle with encryption, access control, and compliance measures.

## Key Topics

### Security Policies and Standards Design
- Data encryption design (at rest and in transit)
- Data auditing strategy design
- Data masking strategy design
- Data privacy design
- Data retention policy design
- Data purging based on business requirements
- Azure RBAC and POSIX-like ACL design for Data Lake Storage Gen2

### Data Security Implementation
- Data masking implementation
- Data encryption (at rest and in motion)
- Row-level and column-level security implementation
- Azure RBAC implementation
- POSIX-like ACL implementation for Data Lake Storage Gen2
- Data retention policy implementation
- Data auditing strategy implementation
- Identity, key, and secret management across platforms
- Secure endpoint implementation (private and service)
- Resource token implementation in Azure Databricks
- DataFrame loading with sensitive information
- Encrypted data writing to tables or Parquet files
- Sensitive information management

## Services Reference

### Core Services
- Azure Active Directory
- Azure Key Vault
- Azure Data Lake Storage Gen2
- Azure SQL Database
- Azure Synapse Analytics
- Azure Databricks

### Supporting Services
- Azure Private Link
- Azure Private Endpoints
- Azure Managed Identity
- Microsoft Purview
- Azure Policy
- Customer Managed Keys (CMK)

## Best Practices

### Access Control
- Implement least privilege access principle
- Use managed identities for service-to-service access
- Combine RBAC and ACLs appropriately in Data Lake
- Regularly review and audit access permissions
- Use Azure AD groups for simplified management
- Implement just-in-time access for sensitive data

### Data Encryption
- Enable encryption at rest for all storage
- Use TLS 1.2+ for data in transit
- Implement customer-managed keys for sensitive data
- Use Always Encrypted for highly sensitive columns
- Enable transparent data encryption (TDE) for databases
- Implement column-level encryption when necessary

### Data Privacy and Compliance
- Implement dynamic data masking for PII
- Use column-level security for sensitive fields
- Apply row-level security for multi-tenant scenarios
- Tag sensitive data for classification
- Implement data retention and purging policies
- Document data lineage for compliance

### Secret Management
- Store all credentials in Key Vault
- Use secret scopes in Databricks for Key Vault integration
- Rotate secrets and keys regularly
- Never hardcode credentials in code or notebooks
- Use connection string encryption
- Implement audit logging for secret access

## Common Scenarios

### Access Control Patterns
- Data scientist access with read-only permissions
- ETL service access using managed identity
- External partner access with limited scope
- Time-bound access for contractors
- Cross-subscription data access with Private Link

### Data Protection
- PII masking in non-production environments
- Credit card number encryption in data lake
- GDPR compliance with data purging
- HIPAA compliance with encryption and auditing
- Row-level security for multi-tenant analytics

### Secure Data Pipeline
- Encrypted data ingestion from on-premises
- Secure transformation with managed identities
- Private endpoint access to data services
- Audit logging throughout pipeline
- Encryption of intermediate data files

### Databricks Security
- Credential passthrough to Data Lake
- Table access control in Unity Catalog
- Secret management with Databricks secret scopes
- Network isolation with VNet injection
- Cluster access control and policies

## Study Tips

- Understand the difference between RBAC and ACLs in Data Lake Gen2
- Practice configuring managed identities for data services
- Learn dynamic data masking rule configuration
- Hands-on experience with row-level security in SQL
- Study Always Encrypted client configuration
- Practice implementing Private Endpoints for data services
- Understand Databricks secret scopes and Key Vault integration
- Learn data classification and sensitivity labels
- Study TDE and CMK configuration
- Practice implementing audit logging for databases
