# AWS Storage Fundamentals

## S3 (Simple Storage Service) Basics

### What is S3?
Object storage service that stores files (objects) in containers (buckets). Think of it as infinite cloud storage for any type of file.

### Core Concepts
- **Bucket**: Container for objects (globally unique name)
- **Object**: Individual files stored in S3
- **Key**: Unique identifier for an object within a bucket
- **Region**: Geographic location where bucket is created
- **Namespace**: Global namespace for bucket names

### Storage Classes Comparison

| Storage Class | Use Case | Availability | Min Storage Duration | Retrieval |
|---------------|----------|--------------|---------------------|-----------|
| **Standard** | Frequently accessed | 99.99% | None | Immediate |
| **Intelligent-Tiering** | Unknown access patterns | 99.9% | 30 days | Automatic |
| **Standard-IA** | Infrequently accessed | 99.9% | 30 days | Immediate |
| **One Zone-IA** | Non-critical, infrequent | 99.5% | 30 days | Immediate |
| **Glacier Instant** | Archive with instant access | 99.9% | 90 days | Immediate |
| **Glacier Flexible** | Archive | 99.99% | 90 days | 1-12 hours |
| **Glacier Deep Archive** | Long-term archive | 99.99% | 180 days | 12-48 hours |

### Key Features
- **Versioning**: Keep multiple versions of objects
- **Lifecycle Policies**: Automatically transition/delete objects
- **Encryption**: At rest and in transit
- **Access Control**: Bucket policies, ACLs, IAM
- **Event Notifications**: Trigger actions on object changes
- **Transfer Acceleration**: Faster uploads via CloudFront

### S3 Pricing Factors
- **Storage Amount**: GB per month
- **Requests**: GET, PUT, DELETE operations
- **Data Transfer**: Out of AWS (in is free)
- **Management Features**: Inventory, analytics, etc.

## EBS (Elastic Block Store) Fundamentals

### What is EBS?
Network-attached block storage for EC2 instances. Like a hard drive you can attach to your virtual machine.

### Volume Types

| Type | IOPS | Throughput | Size | Use Case |
|------|------|------------|------|----------|
| **gp3** | 3,000-16,000 | 125-1,000 MB/s | 1GB-16TB | General purpose, cost-effective |
| **gp2** | 3-16,000 | Up to 250 MB/s | 1GB-16TB | General purpose, burst performance |
| **io2** | Up to 64,000 | Up to 1,000 MB/s | 4GB-16TB | High IOPS, mission-critical |
| **io1** | Up to 64,000 | Up to 1,000 MB/s | 4GB-16TB | High IOPS (legacy) |
| **st1** | Up to 500 | Up to 500 MB/s | 125GB-16TB | Throughput-optimized, big data |
| **sc1** | Up to 250 | Up to 250 MB/s | 125GB-16TB | Cold storage, infrequent access |

### Key Features
- **Snapshots**: Point-in-time backups stored in S3
- **Encryption**: At rest and in transit
- **Multi-Attach**: Attach to multiple EC2 instances (io1/io2)
- **Elastic Volumes**: Modify size/type/IOPS without downtime
- **Availability**: 99.999% availability within AZ

### EBS vs Instance Store
| Feature | EBS | Instance Store |
|---------|-----|----------------|
| **Persistence** | Survives instance stop/start | Lost on stop/terminate |
| **Performance** | Network-attached, good IOPS | Local SSD, very high IOPS |
| **Backup** | Snapshots to S3 | Manual backup required |
| **Cost** | Pay for provisioned storage | Included with instance |
| **Use Case** | Root volumes, databases | Temporary storage, caching |

## EFS (Elastic File System) Fundamentals

### What is EFS?
Fully managed NFS (Network File System) for EC2. Multiple instances can mount the same file system simultaneously.

### Key Characteristics
- **POSIX-compliant**: Standard file system interface
- **Multi-AZ**: Automatically replicated across AZs
- **Scalable**: Grows and shrinks automatically
- **Concurrent Access**: Thousands of EC2 instances
- **Regional**: Available across AZs in a region

### Performance Modes
- **General Purpose**: Up to 7,000 file operations per second
- **Max I/O**: Higher performance, slightly higher latency

### Throughput Modes
- **Provisioned**: Specify exact throughput needed
- **Bursting**: Throughput scales with file system size

### Use Cases
- **Content Management**: Web serving, shared content
- **Web Serving**: Shared web content across instances
- **Data Analytics**: Shared data processing
- **Media Processing**: Shared media workflows

## FSx Family Overview

### FSx for Windows File Server
- **Purpose**: Fully managed Windows-based file system
- **Protocol**: SMB (Server Message Block)
- **Integration**: Active Directory integration
- **Use Case**: Windows-based applications, file shares

### FSx for Lustre
- **Purpose**: High-performance file system for compute-intensive workloads
- **Integration**: Works with S3
- **Performance**: Sub-millisecond latencies, millions of IOPS
- **Use Case**: Machine learning, high-performance computing (HPC)

### FSx for NetApp ONTAP
- **Purpose**: Fully managed NetApp ONTAP file system
- **Protocol**: NFS, SMB, iSCSI
- **Features**: Snapshots, cloning, replication
- **Use Case**: Enterprise applications, databases

### FSx for OpenZFS
- **Purpose**: Fully managed OpenZFS file system
- **Features**: Point-in-time snapshots, compression
- **Performance**: Up to 1 million IOPS
- **Use Case**: Linux-based workloads, databases

## Storage Gateway (Hybrid Cloud)

### What is Storage Gateway?
Hybrid cloud storage service connecting on-premises to AWS cloud storage.

### Gateway Types

#### File Gateway
- **Interface**: NFS/SMB
- **Storage**: Files stored as S3 objects
- **Use Case**: File shares, content distribution

#### Volume Gateway
- **Stored Volumes**: Primary data on-premises, async backup to S3
- **Cached Volumes**: Primary data in S3, cache frequently accessed on-premises
- **Use Case**: Backup, disaster recovery

#### Tape Gateway
- **Interface**: Virtual Tape Library (VTL)
- **Storage**: Virtual tapes in S3 and Glacier
- **Use Case**: Replace physical tape infrastructure

## Storage Decision Framework

### Choosing Storage Type
```
Object storage for web apps? → S3
Block storage for EC2? → EBS
Shared file storage? → EFS
Windows file shares? → FSx for Windows
High-performance computing? → FSx for Lustre
Hybrid connectivity? → Storage Gateway
```

### S3 Storage Class Selection
```
Unknown access pattern? → Intelligent-Tiering
Frequent access? → Standard
Monthly access? → Standard-IA
Rarely accessed, cost-sensitive? → One Zone-IA
Archive with instant access? → Glacier Instant Retrieval
Archive, hours retrieval OK? → Glacier Flexible Retrieval
Long-term archive? → Glacier Deep Archive
```

### EBS Volume Type Selection
```
General purpose workload? → gp3
Need guaranteed IOPS? → io2
Big data, throughput-focused? → st1
Backup, cold storage? → sc1
Cost-sensitive, general use? → gp2
```

## Practical Examples

### Web Application Storage
- **Static Assets**: S3 + CloudFront CDN
- **Application Files**: EBS for EC2 instances
- **User Uploads**: S3 with lifecycle policies
- **Database**: EBS for database storage
- **Logs**: S3 for long-term storage

### Content Management System
- **Shared Content**: EFS for multiple web servers
- **Media Storage**: S3 for images, videos
- **Database**: EBS for database files
- **Backup**: S3 Glacier for long-term retention

### Data Analytics Pipeline
- **Raw Data**: S3 for data lake
- **Processing**: FSx for Lustre for high-performance compute
- **Results**: S3 with different storage classes
- **Working Data**: EBS for compute instances

## Best Practices

### S3 Best Practices
- Use appropriate storage classes
- Enable versioning for important data
- Implement lifecycle policies
- Use server-side encryption
- Monitor access patterns
- Optimize request patterns

### EBS Best Practices
- Choose right volume type for workload
- Enable encryption for sensitive data
- Regular snapshots for backup
- Monitor IOPS and throughput
- Use gp3 for cost optimization
- Size volumes appropriately

### EFS Best Practices
- Use appropriate performance mode
- Monitor file system utilization
- Use security groups for access control
- Consider regional vs one-zone for cost
- Use encryption for sensitive data

## Cost Optimization Strategies

### S3 Cost Optimization
- Use S3 Intelligent-Tiering for unknown patterns
- Implement lifecycle policies
- Delete incomplete multipart uploads
- Use S3 Storage Lens for insights
- Consider Requester Pays for shared data

### EBS Cost Optimization
- Right-size volumes regularly
- Use gp3 instead of gp2
- Delete unused snapshots
- Use Elastic Volumes to resize
- Consider st1/sc1 for appropriate workloads

### General Storage Optimization
- Regular storage audits
- Automate data lifecycle management
- Use compression when possible
- Monitor and alert on unusual usage
- Choose regions based on data locality

## Common Pitfalls

### S3 Common Issues
- Forgetting about storage class minimums
- Not understanding consistency model
- Incorrect bucket policies
- Missing lifecycle policies
- Not monitoring costs

### EBS Common Issues
- Over-provisioning IOPS
- Not taking regular snapshots
- Incorrect volume type selection
- Not encrypting sensitive data
- Not monitoring performance metrics

### EFS Common Issues
- Using when simpler storage would work
- Not optimizing performance mode
- Unexpected costs from throughput
- Security group misconfigurations
- Not considering regional vs one-zone