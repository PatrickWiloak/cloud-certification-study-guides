# Azure Storage Solutions - AZ-104

**[📖 Azure Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/)** - Comprehensive guide to Azure Storage services

## Azure Storage Accounts

**[📖 Storage Account Overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)** - Understand Azure Storage account types and options

### Account Types
- **Standard general-purpose v2**: Blobs, Files, Queues, Tables (recommended)
- **Premium block blobs**: High transaction rates, smaller objects
- **Premium file shares**: SMB and NFS file shares
- **Premium page blobs**: VMs, disks

### Performance Tiers
- **Standard**: HDD-backed, lowest cost
- **Premium**: SSD-backed, low latency, high throughput

### Replication Options

**[📖 Azure Storage Redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)** - Choose the right replication strategy for your data

**Locally Redundant Storage (LRS)**:
- 3 copies within single datacenter
- 11 nines durability
- Lowest cost
- Protects against hardware failures
- Use Case: Non-critical, easily reproducible data

**Zone-Redundant Storage (ZRS)**:
- 3 copies across 3 availability zones
- 12 nines durability
- Available during zone failure
- Higher cost than LRS
- Use Case: High availability requirements

**Geo-Redundant Storage (GRS)**:
- LRS in primary region + LRS in secondary region
- 16 nines durability
- Automatic failover (Microsoft-initiated)
- Read access after failover
- Use Case: Regional disaster recovery

**Read-Access Geo-Redundant Storage (RA-GRS)**:
- GRS + read access to secondary region
- Read-only endpoint always available
- Use Case: Read access during primary region outage

**Geo-Zone-Redundant Storage (GZRS)**:
- ZRS in primary + LRS in secondary
- 16 nines durability
- Highest availability
- Use Case: Maximum durability and availability

**Read-Access GZRS (RA-GZRS)**:
- GZRS + read access to secondary
- Use Case: Critical applications requiring read access

### Access Tiers (Blob Storage)

**[📖 Blob Storage Access Tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)** - Optimize costs with hot, cool, and archive tiers

**Hot**:
- Frequently accessed data
- Highest storage cost, lowest access cost
- Default tier
- Optimized for active data

**Cool**:
- Infrequently accessed data (30+ days)
- Lower storage cost, higher access cost
- Minimum 30-day retention
- Use Case: Short-term backup, disaster recovery

**Archive**:
- Rarely accessed data (180+ days)
- Lowest storage cost, highest access/rehydration cost
- Minimum 180-day retention
- Offline tier, requires rehydration
- Rehydration: Standard (up to 15 hours), High priority (<1 hour)
- Use Case: Long-term backup, compliance archives

### Lifecycle Management
- Automatically transition blobs between tiers
- Delete old blobs or versions
- Rule-based policies
- Actions: Tier to cool, archive, delete

**Example Policy**:
- 0-30 days: Hot
- 30-90 days: Cool
- 90+ days: Archive
- Delete after 365 days

## Azure Blob Storage

**[📖 Introduction to Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)** - Object storage for unstructured data

### Blob Types
**Block Blobs**:
- Text and binary data
- Up to 190.7 TB
- Optimized for sequential I/O
- Use Case: Documents, media files, logs

**Append Blobs**:
- Optimized for append operations
- Log files, data streaming
- Max 195 GB

**Page Blobs**:
- Random read/write operations
- Up to 8 TB
- VHD files for VMs
- Use Case: VM disks

### Blob Access Levels
- **Container**: Public anonymous access
- **Blob**: Public anonymous read for blobs
- **Private**: No anonymous access (default)

### Blob Security

**[📖 Grant Limited Access with SAS](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)** - Secure delegated access to storage resources

**Shared Access Signature (SAS)**:
- Delegated access with specific permissions
- Time-limited
- **Account SAS**: Multiple services
- **Service SAS**: Specific service
- **User Delegation SAS**: Azure AD credentials

**Stored Access Policy**:
- Define permissions, start/end time
- Modify or revoke without regenerating SAS
- Up to 5 policies per container

**Azure AD Authentication**:
- RBAC for blob access
- Use managed identities
- Best practice for applications

### Blob Versioning
- Automatically maintain previous versions
- Recover from accidental deletion or modification
- Enabled at storage account level
- Additional storage costs for versions

### Blob Soft Delete
- Retain deleted blobs for specified period (1-365 days)
- Recover accidentally deleted data
- Separate retention for blobs and containers

### Object Replication
- Asynchronously copy blobs between storage accounts
- Same or different regions
- Requires blob versioning
- Use Cases: Minimize latency, data distribution, compliance

## Azure Files

**[📖 Azure Files Documentation](https://learn.microsoft.com/en-us/azure/storage/files/)** - Cloud file shares accessible via SMB and NFS protocols

### Features
- Fully managed SMB and NFS file shares
- Mount from Windows, Linux, macOS
- Concurrent access from multiple clients
- Up to 100 TB per share (premium), 5 TB (standard)
- Azure AD authentication (domain-joined)

### Performance Tiers
**Standard (Transaction-optimized)**:
- HDD-backed
- Pay per GB stored + transactions
- Up to 10,000 IOPS per share

**Premium**:
- SSD-backed
- Provisioned performance
- Up to 100,000 IOPS per share
- Lower latency
- Use Case: Performance-sensitive workloads

### Azure File Sync

**[📖 Azure File Sync Overview](https://learn.microsoft.com/en-us/azure/storage/file-sync/file-sync-introduction)** - Centralize file shares with cloud tiering capabilities

- Sync on-premises file servers with Azure Files
- Cloud tiering: Cache frequently accessed files locally
- Multi-site sync
- Rapid disaster recovery
- Centralized backup

**Components**:
- **Storage Sync Service**: Azure resource
- **Sync Group**: Defines sync topology
- **Registered Server**: On-premises server
- **Server Endpoint**: Path on registered server
- **Cloud Endpoint**: Azure file share

**Cloud Tiering**:
- Keep frequently accessed files on-premises
- Less frequently accessed files in cloud
- Appears as full file set locally
- Reduces on-premises storage requirements

## Azure Storage Security

**[📖 Azure Storage Encryption](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)** - Data encryption at rest and in transit

### Encryption
**Encryption at Rest**:
- Enabled by default
- Microsoft-managed keys or customer-managed keys (CMK)
- 256-bit AES encryption
- Cannot be disabled

**Encryption in Transit**:
- HTTPS/TLS required
- SMB 3.0 with encryption for Azure Files
- Secure transfer required option

### Network Security
**Firewalls**:
- Restrict access to specific VNets and IP ranges
- Allow Azure services on trusted list
- Default: All networks allowed

**Private Endpoints**:
- Private IP in VNet for storage account
- All traffic over Microsoft backbone
- Use Azure Private Link
- Supported for all storage services

**Service Endpoints**:
- Direct route from VNet to storage
- Traffic stays on Azure backbone
- Free (no Private Link cost)

### Access Keys
- Full access to storage account
- Two keys (primary and secondary)
- Rotate regularly
- Store in Key Vault
- Avoid using in applications (use SAS or Azure AD)

### RBAC Roles
- **Storage Blob Data Owner**: Full access including permissions
- **Storage Blob Data Contributor**: Read, write, delete
- **Storage Blob Data Reader**: Read-only
- **Storage Queue Data Contributor**: Queue messages
- **Storage File Data SMB Share Contributor**: File share access

## Data Migration

**[📖 Choose an Azure Data Transfer Solution](https://learn.microsoft.com/en-us/azure/storage/common/storage-choose-data-transfer-solution)** - Select the best migration method for your scenario

### AzCopy

**[📖 Get Started with AzCopy](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10)** - Command-line utility for copying data to Azure Storage
- Command-line utility
- Copy data to/from storage accounts
- Blobs, Files, Tables
- Synchronize data
- Resume failed transfers

```bash
# Copy to blob
azcopy copy 'source' 'https://account.blob.core.windows.net/container?SAS' --recursive

# Sync folders
azcopy sync 'source' 'destination' --recursive
```

### Azure Storage Explorer
- GUI application
- Manage storage across accounts
- Windows, macOS, Linux
- Upload, download, manage blobs, files, queues, tables

### Azure Data Box
- Physical device for large data transfers (>40 TB)
- **Data Box Disk**: 8 TB SSD (5 disks = 40 TB)
- **Data Box**: 80 TB appliance
- **Data Box Heavy**: 770 TB appliance
- Encrypted transfer
- Use Case: Datacenter migration, disaster recovery

### Azure Import/Export Service
- Ship hard drives to Azure datacenter
- Import data to Blob storage or Azure Files
- Export data from Blob storage
- Up to 10 drives per job

## Monitoring and Diagnostics

### Storage Analytics
- Metrics for capacity, transactions
- Retention up to 1 year
- Request-level logging
- Analyze with Power BI or Azure Monitor

### Azure Monitor for Storage
- Unified monitoring across storage accounts
- Metrics: Availability, latency, transactions
- Alerts and dashboards
- Integration with Log Analytics

### Diagnostic Logs
- Log every storage request
- Sent to Log Analytics, Event Hub, or Storage Account
- Analyze access patterns, troubleshoot issues

## Best Practices

### Performance
1. Use Premium storage for high IOPS/throughput
2. Choose region close to users
3. Use CDN for frequently accessed static content
4. Optimize blob size (64 MB blocks)
5. Use block blob for large files
6. Enable CDN for global distribution

### Security
1. Use Azure AD authentication over access keys
2. Enable firewall and VNet service endpoints
3. Use SAS tokens with minimal permissions
4. Enable soft delete for blobs and containers
5. Regular access key rotation
6. Use customer-managed keys for compliance
7. Enable secure transfer required

### Cost Optimization
1. Use lifecycle management to tier data
2. Delete old blobs and versions
3. Use cool/archive tiers appropriately
4. Monitor and optimize replication strategy
5. Use reserved capacity for predictable workloads
6. Enable logging only when needed
7. Clean up incomplete multipart uploads

## Exam Tips

### Common Scenarios
- **Share files across VMs**: Azure Files with SMB
- **Static website hosting**: Blob storage with $web container
- **Large file transfer (>40 TB)**: Azure Data Box
- **Lowest cost long-term archive**: Archive tier
- **High availability**: ZRS or GZRS
- **Disaster recovery with read access**: RA-GRS or RA-GZRS
- **Private access from VNet**: Private endpoint or service endpoint
- **Sync on-premises to cloud**: Azure File Sync
- **Delegate limited access**: SAS token
- **VM disks**: Premium page blobs (managed disks)

### Key Points
- Hot/Cool/Archive differ in storage cost vs access cost
- Cannot change standard to premium (must create new account)
- Soft delete protects against accidental deletion
- Lifecycle policies automate tier transitions
- Always enable encryption at rest (enabled by default)
- Use AzCopy for command-line data transfer
- Storage Explorer for GUI management
- Azure Files supports SMB 3.0 and NFS 4.1
