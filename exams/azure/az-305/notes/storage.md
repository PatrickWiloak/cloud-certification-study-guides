# Azure Storage Fundamentals

## Azure Storage Account Basics

### What is an Azure Storage Account?
A storage account provides a unique namespace for your Azure Storage data that's accessible worldwide via HTTP or HTTPS.

### Storage Account Types
| Type | Description | Use Case | Redundancy Options |
|------|-------------|----------|-------------------|
| **General Purpose v2** | Latest storage account type | Recommended for most scenarios | All options |
| **General Purpose v1** | Legacy type | Legacy applications only | LRS, GRS, RA-GRS |
| **BlockBlobStorage** | Premium performance for blobs | High transaction rates, low latency | LRS, ZRS |
| **FileStorage** | Premium performance for files | High-performance file shares | LRS, ZRS |
| **BlobStorage** | Legacy blob-only | Legacy, use GPv2 instead | LRS, GRS, RA-GRS |

### Performance Tiers
| Tier | Description | IOPS | Throughput | Use Case |
|------|-------------|------|------------|----------|
| **Standard** | Magnetic drives | Lower | Lower | General purpose, cost-effective |
| **Premium** | SSD drives | Higher | Higher | I/O intensive applications |

### Access Tiers (Blob Storage)
| Tier | Description | Storage Cost | Access Cost | Use Case |
|------|-------------|--------------|-------------|----------|
| **Hot** | Frequently accessed | Higher | Lower | Active data |
| **Cool** | Infrequently accessed | Lower | Higher | Short-term backup, infrequent access |
| **Archive** | Rarely accessed | Lowest | Highest | Long-term backup, compliance |

## Blob Storage Deep Dive

### What is Blob Storage?
Object storage solution for the cloud, optimized for storing massive amounts of unstructured data.

### Blob Types
| Type | Description | Use Case | Max Size |
|------|-------------|----------|----------|
| **Block Blobs** | Optimized for uploading large amounts of data | Text/binary files, media | 190.7 TB |
| **Append Blobs** | Optimized for append operations | Log files, audit trails | 195 GB |
| **Page Blobs** | Optimized for random read/write | VM disks, databases | 8 TB |

### Container & Blob Hierarchy
```
Storage Account
├── Container 1
│   ├── Blob 1
│   ├── Blob 2
│   └── Virtual Directory/
│       └── Blob 3
└── Container 2
    └── Blob 4
```

### Lifecycle Management
Automatically transition blobs between access tiers based on rules:
- **Age-based**: Move data based on days since creation/modification
- **Prefix-based**: Apply rules to specific blob paths
- **Blob type-based**: Different rules for different blob types

Example Policy:
```
If blob age > 30 days → Move to Cool
If blob age > 90 days → Move to Archive  
If blob age > 2555 days → Delete
```

### Blob Security Features
- **Shared Access Signatures (SAS)**: Time-limited access tokens
- **Azure AD authentication**: Role-based access control
- **Storage Service Encryption**: Automatic encryption at rest
- **Client-side encryption**: Encrypt before uploading
- **Immutable storage**: Write-once, read-many (WORM) compliance

## File Storage (Azure Files)

### What is Azure Files?
Fully managed file shares in the cloud accessible via Server Message Block (SMB) and Network File System (NFS) protocols.

### Key Characteristics
- **SMB/NFS protocols**: Standard file sharing protocols
- **Cross-platform**: Windows, Linux, macOS
- **Cloud or on-premises**: Accessible from anywhere
- **Shared access**: Multiple VMs can mount same share
- **Azure File Sync**: Sync with on-premises file servers

### Performance Tiers
| Tier | Max IOPS | Max Throughput | Use Case |
|------|----------|----------------|----------|
| **Standard** | 1,000 | 60 MB/s | General file sharing |
| **Premium** | 100,000 | 5 GB/s | High-performance applications |

### Azure File Sync
- **Purpose**: Centralize file shares in Azure while keeping local performance
- **Components**: Storage Sync Service, Sync Group, Server Endpoint, Cloud Endpoint
- **Benefits**: Cloud backup, disaster recovery, distributed access
- **Cloud Tiering**: Keep frequently accessed files local, others in cloud

### Use Cases
- **Lift and shift**: Replace on-premises file servers
- **Shared application data**: Configuration files, shared content
- **Development environments**: Shared code repositories
- **Container storage**: Persistent storage for containers

## Queue Storage

### What is Queue Storage?
Message queue service for reliable messaging between application components.

### Key Features
- **Asynchronous messaging**: Decouple application components
- **At-least-once delivery**: Messages delivered reliably
- **Visibility timeout**: Hide messages during processing
- **Time-to-live**: Automatic message expiration
- **Base64 encoding**: Text-based message format

### Message Lifecycle
1. **Producer** puts message in queue
2. **Consumer** gets message (becomes invisible to others)
3. **Consumer** processes message
4. **Consumer** deletes message from queue

### Use Cases
- **Work distribution**: Distribute tasks across workers
- **Load leveling**: Buffer between fast producers and slower consumers
- **Reliable messaging**: Ensure messages aren't lost
- **Background processing**: Trigger asynchronous operations

## Table Storage

### What is Table Storage?
NoSQL key-value store for rapid development using massive semi-structured datasets.

### Data Model
- **Table**: Collection of entities
- **Entity**: Row in table (max 1MB)
- **Property**: Column in entity (max 252 properties)
- **Partition Key + Row Key**: Unique identifier for entity

### Key Characteristics
- **Schema-less**: No fixed schema required
- **Scalable**: Handles billions of entities
- **Fast**: Single-digit millisecond latency
- **Cost-effective**: Lower cost than relational databases
- **ACID transactions**: Within single partition

### Partition Strategy
- **Good partition key**: Distributes load evenly
- **Bad partition key**: Creates hot partitions
- **Examples**: 
  - Good: UserID, Date ranges
  - Bad: Single value, Sequential numbers

## Disk Storage

### Managed Disks Overview
Azure-managed virtual hard disks (VHDs) for virtual machines.

### Disk Types
| Type | Performance | Use Case |
|------|-------------|----------|
| **Ultra Disk** | Highest IOPS and throughput | Mission-critical workloads |
| **Premium SSD** | High performance | Production workloads |
| **Standard SSD** | Moderate performance | Web servers, dev/test |
| **Standard HDD** | Basic performance | Backup, non-critical |

### Disk Performance Comparison
| Type | Max IOPS | Max Throughput | Max Size |
|------|----------|----------------|----------|
| **Ultra** | 160,000 | 2,000 MB/s | 65,536 GB |
| **Premium SSD** | 20,000 | 900 MB/s | 32,767 GB |
| **Standard SSD** | 6,000 | 750 MB/s | 32,767 GB |
| **Standard HDD** | 2,000 | 500 MB/s | 32,767 GB |

### Managed Disk Features
- **Snapshots**: Point-in-time backups
- **Images**: Template for creating VMs
- **Shared disks**: Multi-VM access for clusters
- **Encryption**: Azure Disk Encryption, SSE
- **Availability zones**: Zone-redundant storage

## Redundancy Options

### Local Redundancy Options
| Type | Copies | Protection | Cost |
|------|--------|------------|------|
| **LRS** | 3 copies in single datacenter | Hardware failures | Lowest |
| **ZRS** | 3 copies across 3 zones | Zone failures | Low |

### Geo-Redundancy Options  
| Type | Copies | Protection | Read Access |
|------|--------|------------|-------------|
| **GRS** | 6 copies (3 local + 3 remote) | Regional disasters | No |
| **RA-GRS** | 6 copies (3 local + 3 remote) | Regional disasters | Yes |
| **GZRS** | 6 copies (ZRS local + LRS remote) | Zone + regional disasters | No |
| **RA-GZRS** | 6 copies (ZRS local + LRS remote) | Zone + regional disasters | Yes |

### Choosing Redundancy
```
Need read access to secondary region? → RA-GRS or RA-GZRS
Need zone redundancy? → ZRS, GZRS, or RA-GZRS
Cost-sensitive? → LRS
High availability required? → GRS or higher
```

## Storage Security

### Access Control Methods
| Method | Description | Use Case |
|--------|-------------|----------|
| **Storage Account Keys** | Full access to account | Administrative access |
| **Shared Access Signatures** | Limited access with expiration | Temporary access |
| **Azure AD** | Role-based access control | User/application access |
| **Anonymous access** | Public read access | Public websites |

### Encryption Options
- **Encryption at rest**: Automatic with Microsoft-managed keys
- **Customer-managed keys**: Use your own encryption keys
- **Client-side encryption**: Encrypt before sending to Azure
- **Encryption in transit**: HTTPS/TLS for all connections

### Network Security
- **Firewall rules**: Restrict access by IP address
- **Virtual network service endpoints**: Access from specific VNets
- **Private endpoints**: Private IP access from VNet
- **Secure transfer**: Require HTTPS for all operations

## Azure Storage Services Integration

### Content Delivery Network (CDN)
- **Purpose**: Cache blob content globally for low latency
- **Providers**: Microsoft, Verizon, Akamai
- **Benefits**: Reduced latency, bandwidth savings, global scale
- **Use cases**: Static websites, download distribution, streaming

### Azure Import/Export
- **Purpose**: Transfer large amounts of data using physical drives
- **Import**: Ship drives to Azure datacenter
- **Export**: Azure ships drives with your data
- **Use cases**: Initial cloud migration, backup offloading, disaster recovery

### AzCopy
- **Purpose**: Command-line utility for copying data to/from storage
- **Features**: Cross-platform, resume capability, synchronization
- **Performance**: Optimized for high throughput
- **Use cases**: Data migration, backup, synchronization

## Storage Decision Framework

### Choosing Storage Type
```
Unstructured data/files? → Blob Storage
Shared file system needed? → Azure Files
Message queuing? → Queue Storage
NoSQL key-value data? → Table Storage
Virtual machine disks? → Managed Disks
```

### Blob Access Tier Selection
```
Accessed frequently? → Hot tier
Accessed monthly? → Cool tier
Long-term archive? → Archive tier
Unknown pattern? → Start with Hot, use lifecycle management
```

### Performance Tier Selection
```
High IOPS/throughput needed? → Premium
Cost-sensitive? → Standard
Ultra-high performance? → Ultra Disk (for VMs)
```

## Practical Examples

### Static Website Hosting
- **Website files**: Blob Storage ($web container)
- **Custom domain**: Configure custom domain and SSL
- **CDN**: Azure CDN for global distribution
- **Backup**: Geo-redundant storage for disaster recovery

### Data Lake Architecture
- **Raw data**: Blob Storage (hierarchical namespace)
- **Processing**: Azure Data Factory, Databricks
- **Analytics**: Azure Synapse Analytics
- **Archive**: Lifecycle policies to Archive tier

### Enterprise File Sharing
- **Primary storage**: Azure Files premium tier
- **Hybrid sync**: Azure File Sync for on-premises
- **Backup**: Azure Backup for file shares
- **Security**: Azure AD authentication, encryption

### Application Backend
- **User uploads**: Blob Storage with SAS tokens
- **Session state**: Table Storage or Redis Cache
- **Background jobs**: Queue Storage for task distribution
- **Logs**: Blob Storage append blobs

## Best Practices

### Performance Optimization
- Choose appropriate performance tier for workload
- Use multiple storage accounts to scale beyond limits
- Implement proper partitioning strategy for Table Storage
- Use CDN for globally distributed content
- Monitor performance metrics and adjust accordingly

### Security Best Practices
- Use Azure AD authentication when possible
- Implement least privilege access with RBAC
- Use SAS tokens with minimal required permissions
- Enable firewall rules and VNet integration
- Regularly rotate storage account keys
- Enable encryption for sensitive data

### Cost Optimization
- Use appropriate access tiers for blob storage
- Implement lifecycle management policies
- Use Azure Storage reserved capacity for predictable workloads
- Monitor and clean up unused snapshots and old data
- Choose appropriate redundancy level for requirements

## Common Pitfalls

### Blob Storage Issues
- Not using lifecycle policies for cost optimization
- Incorrect access tier selection
- Poor container/blob naming strategy
- Not implementing proper security controls
- Ignoring storage analytics and monitoring

### File Storage Issues
- Wrong performance tier for workload requirements
- Inadequate network bandwidth for file sync
- Not configuring proper backup strategies
- Authentication issues with domain-joined scenarios
- Performance bottlenecks with large files

### General Storage Issues
- Not planning for storage account limits
- Inadequate redundancy for business requirements
- Poor key management practices
- Not monitoring storage costs and usage
- Incorrect region selection affecting performance