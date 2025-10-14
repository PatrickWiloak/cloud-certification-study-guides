# AWS Storage Solutions - SAA-C03

## Amazon S3 (Simple Storage Service)

**[📖 Amazon S3 Documentation](https://docs.aws.amazon.com/s3/index.html)** - Complete guide to S3 storage and features

### Storage Classes

**[📖 S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)** - Choose the right storage class for your data access patterns
- **S3 Standard**: Frequently accessed data, low latency, high throughput
- **S3 Intelligent-Tiering**: Automatic cost optimization, moves data between tiers
- **S3 Standard-IA**: Infrequent access, lower cost, retrieval fee
- **S3 One Zone-IA**: Single AZ, 99.5% availability, lower cost
- **S3 Glacier Instant Retrieval**: Archive with millisecond retrieval
- **S3 Glacier Flexible Retrieval**: Archive with 1-5 minute retrieval
- **S3 Glacier Deep Archive**: Lowest cost, 12-hour retrieval

### Key Features
- **Versioning**: Keep multiple versions of objects
- **Lifecycle Policies**: Automate transitions between storage classes
- **Cross-Region Replication (CRR)**: Replicate objects across regions
- **Same-Region Replication (SRR)**: Compliance and data sovereignty
- **S3 Transfer Acceleration**: Fast uploads using CloudFront edge locations
- **Multipart Upload**: For objects >100MB, required for >5GB
- **S3 Select**: Query data with SQL without retrieving entire object

**[📖 S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)** - Automatically replicate objects across AWS Regions

### Security
- **Bucket Policies**: Resource-based policies (JSON)
- **Access Control Lists (ACLs)**: Legacy access control
- **Block Public Access**: Account and bucket level protection
- **Encryption at Rest**:
  - SSE-S3: S3-managed keys
  - SSE-KMS: KMS-managed keys, audit trail
  - SSE-C: Customer-provided keys
- **Encryption in Transit**: HTTPS/TLS

**[📖 S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)** - Implement security controls for S3 buckets and objects

### Performance Optimization
- **S3 Prefixes**: 3,500 PUT/COPY/POST/DELETE, 5,500 GET/HEAD per prefix
- **Multipart Upload**: Parallel uploads for large files
- **Transfer Acceleration**: Use CloudFront edge locations
- **Byte-Range Fetches**: Download partial object

## Amazon EBS (Elastic Block Store)

**[📖 Amazon EBS Documentation](https://docs.aws.amazon.com/ebs/index.html)** - Block storage for EC2 instances

### Volume Types

**[📖 EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)** - Choose the right EBS volume for your workload
**SSD-Based (IOPS-focused)**:
- **gp3 (General Purpose SSD)**: 3,000-16,000 IOPS, 125-1,000 MB/s throughput
- **gp2 (General Purpose SSD)**: Baseline 3 IOPS/GB, burst to 3,000 IOPS
- **io2 Block Express**: Up to 256,000 IOPS, 4,000 MB/s, 99.999% durability
- **io2**: Up to 64,000 IOPS, 1,000 MB/s, 99.999% durability
- **io1**: Up to 64,000 IOPS, 1,000 MB/s

**HDD-Based (Throughput-focused)**:
- **st1 (Throughput Optimized)**: 500 IOPS, 500 MB/s, big data, log processing
- **sc1 (Cold HDD)**: 250 IOPS, 250 MB/s, infrequent access

### Key Features
- **Snapshots**: Point-in-time backups, incremental, stored in S3
- **Snapshot Archive**: 75% cost reduction, 24-72 hour retrieval
- **Fast Snapshot Restore (FSR)**: Instant restore, expensive
- **Multi-Attach (io1/io2)**: Attach volume to multiple EC2 instances
- **Encryption**: At rest and in transit, uses KMS
- **EBS-Optimized Instances**: Dedicated bandwidth for EBS

**[📖 EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)** - Back up your EBS volumes with snapshots

### Best Practices
- Use gp3 for most workloads (cheaper than gp2)
- io2 for databases requiring high IOPS
- Take regular snapshots for backup
- Delete old snapshots to save costs
- Enable encryption by default

## Amazon EFS (Elastic File System)

**[📖 Amazon EFS Documentation](https://docs.aws.amazon.com/efs/index.html)** - Scalable, elastic file storage for Linux workloads

### Features
- **Network File System (NFS)**: POSIX-compliant, shared storage
- **Multi-AZ**: Automatically replicated across AZs
- **Auto-scaling**: Grows and shrinks automatically
- **Performance Modes**:
  - General Purpose: Low latency, web serving
  - Max I/O: Higher latency, big data, media processing

### Throughput Modes
- **Bursting**: Throughput scales with size (50 MB/s per TB)
- **Provisioned**: Set throughput independent of size
- **Elastic**: Automatically scales throughput (up to 3 GB/s reads, 1 GB/s writes)

### Storage Classes
- **Standard**: Multi-AZ, frequently accessed
- **Infrequent Access (IA)**: Lower cost, lifecycle policy
- **One Zone**: Single AZ, 47% cost savings
- **One Zone-IA**: Lowest cost

### Use Cases
- Content management systems
- Web serving
- Home directories
- Shared development environments
- Container storage (ECS/EKS)

## Amazon FSx

**[📖 Amazon FSx Documentation](https://docs.aws.amazon.com/fsx/)** - Fully managed file systems built on popular file systems

### FSx for Windows File Server
- **SMB Protocol**: Windows native file system
- **Active Directory Integration**: Windows authentication
- **Performance**: Up to 2 GB/s throughput, millions of IOPS
- **Use Cases**: Windows applications, home directories, SQL Server

**[📖 FSx for Windows File Server](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)** - Fully managed Windows file storage

### FSx for Lustre
- **High Performance**: Machine learning, HPC, video processing
- **Throughput**: 100s GB/s, millions of IOPS
- **S3 Integration**: Direct access to S3 objects
- **Deployment Types**:
  - Scratch: Temporary, 200 MB/s per TiB
  - Persistent: Long-term, 50/100/200 MB/s per TiB

**[📖 FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html)** - High-performance file system for compute-intensive workloads

### FSx for NetApp ONTAP
- **NFS and SMB**: Multi-protocol support
- **Point-in-time Snapshots**: Instant recovery
- **Data Compression**: Storage efficiency

### FSx for OpenZFS
- **High Performance**: Up to 1 million IOPS
- **Point-in-time Snapshots**: Instant clones
- **Use Cases**: Linux workloads, databases

## AWS Storage Gateway

**[📖 AWS Storage Gateway](https://docs.aws.amazon.com/storagegateway/)** - Hybrid cloud storage integration

### Gateway Types

**File Gateway**:
- Store files as S3 objects
- NFS and SMB protocols
- Local cache for frequently accessed data
- Use Case: Backup, archive, cloud-native app data

**Volume Gateway**:
- Block storage using iSCSI
- **Cached Volumes**: Primary data in S3, cache on-premises
- **Stored Volumes**: Primary data on-premises, async backup to S3
- Use Case: Backup, disaster recovery

**Tape Gateway**:
- Virtual tape library (VTL)
- Backup software integration
- Store tapes in S3 Glacier
- Use Case: Replace physical tape infrastructure

### Best Practices
- Use File Gateway for NFS/SMB file shares backed by S3
- Use Volume Gateway for block storage backup
- Tape Gateway for existing backup workflows
- Deploy gateway on-premises or in EC2

## AWS DataSync

**[📖 AWS DataSync](https://docs.aws.amazon.com/datasync/)** - Automated data transfer between on-premises and AWS

### Features
- **Fast Transfer**: 10x faster than open-source tools
- **Bandwidth Throttling**: Control network usage
- **Schedule**: Hourly, daily, weekly transfers
- **Data Validation**: Verify integrity
- **Filtering**: Include/exclude patterns

### Supported Locations
- **Source**: On-premises (NFS, SMB), AWS (S3, EFS, FSx)
- **Destination**: S3, EFS, FSx
- **Use Cases**: Migration, replication, archival

### DataSync vs Storage Gateway
- **DataSync**: One-time or scheduled migrations
- **Storage Gateway**: Continuous hybrid storage access

## AWS Snow Family

### Snowcone
- **Capacity**: 8 TB HDD or 14 TB SSD
- **Use Case**: Edge computing, small data transfers
- **Portable**: 4.5 lbs, rugged
- **DataSync Agent**: Preinstalled

### Snowball Edge
- **Storage Optimized**: 80 TB HDD
- **Compute Optimized**: 42 TB HDD + 7.68 TB SSD
- **Compute with GPU**: 42 TB + GPU
- **Use Case**: Large migrations (TB-PB), edge processing

### Snowmobile
- **Capacity**: 100 PB per truck
- **Use Case**: Exabyte-scale datacenter migrations
- **Security**: GPS tracking, 24/7 monitoring

### When to Use Snow
- **Limited Bandwidth**: >1 week to transfer over network
- **High Transfer Costs**: Network egress charges
- **Disconnected Environments**: Edge locations
- **Rule of Thumb**: >10 TB = consider Snow

## Storage Comparison

### Block vs File vs Object

**Block Storage (EBS)**:
- Low latency, high IOPS
- Single instance attachment (except Multi-Attach)
- Boot volumes, databases
- Snapshots for backup

**File Storage (EFS, FSx)**:
- Shared access from multiple instances
- POSIX permissions (EFS)
- SMB/NFS protocols
- Auto-scaling capacity

**Object Storage (S3)**:
- Unlimited storage
- HTTP/HTTPS access
- 99.999999999% durability
- Static websites, backups, data lakes

### Cost Optimization Strategies
1. Use S3 Intelligent-Tiering for unknown access patterns
2. Implement S3 Lifecycle Policies to move to cheaper tiers
3. Delete incomplete multipart uploads
4. Use EFS IA for infrequently accessed files
5. Delete old EBS snapshots
6. Use gp3 instead of gp2 (20% cheaper)
7. Right-size EBS volumes

## Exam Tips

### Common Scenarios
- **Shared file storage across EC2**: EFS
- **Windows file shares**: FSx for Windows
- **High-performance computing**: FSx for Lustre
- **Boot volumes**: EBS (gp3 or gp2)
- **Database storage**: EBS (io2 for production)
- **Static website**: S3
- **Long-term archive**: S3 Glacier Deep Archive
- **Hybrid cloud storage**: Storage Gateway
- **Large data migration**: AWS DataSync or Snow Family
- **Infrequent access**: S3 IA, EFS IA, gp2 with burst credits

### Key Differences
- **EBS vs EFS**: Single instance vs multi-instance shared
- **S3 vs EBS**: Object storage vs block storage
- **DataSync vs Storage Gateway**: Migration vs ongoing access
- **Snowball vs DataSync**: Offline vs online transfer
