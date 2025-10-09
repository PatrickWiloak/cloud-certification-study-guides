# Secure Data and Applications

## Overview
This domain covers configuring security for storage accounts, databases, and implementing Azure Key Vault management. It represents 20-25% of the exam and focuses on protecting data at rest, in transit, and in use.

## Key Topics

### Storage Security
- Storage account access control configuration
- Storage account access key lifecycle management
- Azure AD authentication for Azure Storage
- Azure Files access control configuration
- Azure Blob Storage access control
- Azure Storage encryption configuration
- Azure SQL Database encryption

### Database Security
- Database authentication with Azure AD
- Database auditing configuration
- Dynamic data masking implementation
- Database threat detection configuration
- Always Encrypted for Azure SQL Database
- Row-level security (RLS)
- Column-level security

### Key Vault Management
- Key Vault access management
- Permissions for secrets, certificates, and keys
- RBAC usage in Azure Key Vault
- Certificate management
- Secret management
- Key rotation configuration
- Backup and restore of Key Vault items

## Services Reference

### Core Services
- Azure Storage (Blob, Files, Queue, Table)
- Azure SQL Database
- Azure Key Vault
- Azure Managed HSM
- Azure Information Protection

### Supporting Services
- Azure Disk Encryption
- Azure Storage Service Encryption (SSE)
- Transparent Data Encryption (TDE)
- Azure AD for authentication
- Customer-Managed Keys (CMK)

## Best Practices

### Storage Protection
- Enable encryption by default for all storage accounts
- Use private endpoints for storage access
- Disable public blob access when not required
- Implement SAS token expiration and IP restrictions
- Enable soft delete for blobs and containers
- Use immutable blob storage for compliance

### Database Security
- Always use Azure AD authentication when possible
- Enable Advanced Data Security for SQL databases
- Implement dynamic data masking for sensitive fields
- Use Always Encrypted for highly sensitive data
- Enable auditing for all production databases
- Implement row-level security for multi-tenant scenarios

### Key Management
- Separate Key Vaults by environment (dev, test, prod)
- Use Managed HSM for FIPS 140-2 Level 3 compliance
- Enable soft delete and purge protection
- Implement proper access policies with least privilege
- Use RBAC model for Key Vault when possible
- Rotate keys and secrets regularly
- Monitor Key Vault access and usage

### Encryption Strategy
- Use customer-managed keys for sensitive workloads
- Enable encryption in transit for all communications
- Implement double encryption for highly sensitive data
- Use Always Encrypted for column-level encryption
- Configure TDE with customer-managed keys

## Common Scenarios

### Access Control Patterns
- Blob storage access using managed identities
- Time-limited access with SAS tokens
- Azure AD authentication for SQL Database
- Service principal access to Key Vault
- User delegation SAS for fine-grained access

### Data Protection
- PII protection with dynamic data masking
- Credit card encryption with Always Encrypted
- Sensitive document storage in encrypted blob
- Compliance-required data immutability
- Data retention with lifecycle management

### Key and Secret Management
- Application secrets in Key Vault
- Certificate lifecycle management
- Database connection string protection
- API key rotation automation
- HSM-backed key protection

## Study Tips

- Understand difference between access keys and SAS tokens
- Practice configuring Azure AD authentication for storage
- Learn Always Encrypted client-side encryption process
- Hands-on experience with Key Vault access policies vs RBAC
- Study SAS token types (account, service, user delegation)
- Practice configuring dynamic data masking rules
- Understand Key Vault soft delete and purge protection
- Learn certificate renewal and rotation processes
- Study storage account firewall and virtual network rules
- Practice implementing row-level security in SQL
