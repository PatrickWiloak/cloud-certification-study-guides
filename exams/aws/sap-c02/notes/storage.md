# Storage

## Overview

Enterprise storage solutions on AWS require deep understanding of performance characteristics, durability guarantees, cost models, and appropriate use cases for each storage service. At the professional level, you must design complex storage architectures that balance performance, availability, compliance, and cost across hybrid environments, multiple accounts, and global deployments.

Professional architects must navigate trade-offs between block storage, object storage, file systems, and archival solutions while implementing data lifecycle policies, replication strategies, disaster recovery patterns, and ensuring regulatory compliance.

## Key Concepts

### Storage Types and Characteristics

**Block Storage (Amazon EBS)**
- Attached to single EC2 instance (except io2 Multi-Attach)
- AZ-scoped resource with automatic replication within AZ
- Volumes: gp3, gp2, io2, io1, st1, sc1
- Snapshots stored in S3 (incremental, region-scoped)
- Use cases: Boot volumes, databases, transactional workloads

**Object Storage (Amazon S3)**
- Unlimited storage capacity, 5TB max object size
- Region-scoped with 11 9's durability
- Storage classes for different access patterns and costs
- Lifecycle policies for automated transitions
- Use cases: Data lakes, backups, static websites, media storage

**File Storage**
- **Amazon EFS**: Shared NFS for Linux, multi-AZ, elastic capacity
- **Amazon FSx**: Managed Windows File Server, Lustre, NetApp ONTAP, OpenZFS
- Concurrent access from multiple instances
- Use cases: Shared application storage, home directories, content repositories

**Archival Storage**
- **S3 Glacier**: Long-term archive with retrieval times from minutes to hours
- **S3 Glacier Deep Archive**: Lowest cost for rarely accessed data (12+ hour retrieval)
- Tape Gateway: Virtual tape library backed by S3 Glacier

### Amazon S3 Advanced Concepts

**Storage Classes**
- **S3 Standard**: Frequent access, millisecond latency, 99.99% availability
- **S3 Intelligent-Tiering**: Automatic cost optimization based on access patterns
- **S3 Standard-IA**: Infrequent access, lower storage cost, retrieval fee
- **S3 One Zone-IA**: Single AZ, 20% cheaper than Standard-IA
- **S3 Glacier Instant Retrieval**: Archive with millisecond retrieval
- **S3 Glacier Flexible Retrieval**: Archive with 1-5 minute to 12-hour retrieval
- **S3 Glacier Deep Archive**: Lowest cost, 12-48 hour retrieval
- **S3 Outposts**: S3 on AWS Outposts for local data processing

**S3 Lifecycle Policies**
- Transition objects between storage classes
- Expire objects after retention period
- Delete incomplete multipart uploads
- Clean up old versions and delete markers
- Filter by prefix and tags
- Minimum storage duration charges apply (30/90 days for IA/Glacier)

**S3 Replication**
- **Cross-Region Replication (CRR)**: Replicate to different region for DR, compliance, latency
- **Same-Region Replication (SRR)**: Replicate within region for log aggregation, backup
- **Batch Replication**: Replicate existing objects (new feature)
- Replication Time Control (RTC): 99.99% of objects replicated in 15 minutes
- Replica Modification Sync: Replicate metadata changes
- Delete marker replication (optional)
- Can replicate between accounts and storage classes

**S3 Performance Optimization**
- Request rate: 3,500 PUT/COPY/POST/DELETE, 5,500 GET/HEAD per prefix per second
- Use prefixes to parallelize requests (e.g., /2025/01/01/, /2025/01/02/)
- Multipart upload for objects >100 MB (required for >5 GB)
- S3 Transfer Acceleration for global uploads (uses CloudFront edge locations)
- Byte-range fetches for parallel downloads
- S3 Select and Glacier Select for server-side filtering

**S3 Security**
- Bucket policies for resource-based access control
- IAM policies for identity-based access control
- Access Control Lists (ACLs) - legacy, avoid for new implementations
- VPC Endpoints for private access
- S3 Access Points for simplified access management
- S3 Multi-Region Access Points for global applications
- S3 Object Ownership for bucket owner control
- Block Public Access settings (account and bucket level)

**S3 Encryption**
- **Server-Side Encryption (SSE)**:
  - SSE-S3: S3-managed keys (AES-256)
  - SSE-KMS: KMS customer-managed keys with audit trail
  - SSE-C: Customer-provided keys
  - DSSE-KMS: Dual-layer encryption with KMS
- **Client-Side Encryption**: Encrypt before upload
- Bucket default encryption settings
- Enforce encryption with bucket policies
- S3 Bucket Keys to reduce KMS costs (99% reduction in requests)

**S3 Advanced Features**
- **S3 Object Lock**: WORM (Write Once Read Many) for compliance
  - Governance mode: Protected with specific IAM permissions
  - Compliance mode: Immutable for retention period
  - Legal Hold: Indefinite protection regardless of retention
- **S3 Versioning**: Preserve all versions of objects
- **S3 Event Notifications**: Trigger Lambda, SNS, SQS, EventBridge on object events
- **S3 Batch Operations**: Perform operations on billions of objects
- **S3 Inventory**: Scheduled reports on objects and metadata
- **S3 Storage Lens**: Organization-wide visibility and recommendations
- **S3 Access Analyzer**: Identify buckets with external access

### Amazon EBS Advanced Concepts

**Volume Types**
- **gp3**: General purpose SSD, 3,000-16,000 IOPS, 125-1,000 MB/s, cheaper than gp2
- **gp2**: General purpose SSD, 3 IOPS per GB (100-16,000 IOPS), baseline performance
- **io2 Block Express**: Highest performance, up to 256,000 IOPS, 4,000 MB/s, sub-millisecond latency
- **io2**: Provisioned IOPS, up to 64,000 IOPS, 1,000 MB/s, 99.999% durability
- **io1**: Provisioned IOPS, up to 64,000 IOPS, 1,000 MB/s
- **st1**: Throughput optimized HDD, 500 MB/s, big data, log processing
- **sc1**: Cold HDD, 250 MB/s, lowest cost, infrequent access

**EBS Features**
- **Snapshots**: Incremental backups to S3, cross-region copy, shared across accounts
- **Fast Snapshot Restore (FSR)**: Eliminate latency penalty on first use
- **EBS Multi-Attach**: Attach io2 volume to up to 16 instances in same AZ (clustered apps)
- **EBS Encryption**: At-rest and in-transit encryption with KMS
- **EBS Elastic Volumes**: Dynamically increase size, performance, change type without downtime
- **EBS Direct APIs**: Direct read/write to snapshots for backup/restore tools
- **Data Lifecycle Manager (DLM)**: Automated snapshot creation and retention

**Performance Optimization**
- Pre-warming no longer required for volumes restored from snapshots with FSR
- EBS-optimized instances for dedicated bandwidth
- RAID 0 for increased performance (striping)
- RAID 1 for increased durability (mirroring) - not recommended, use snapshots instead
- Instance type affects max IOPS and throughput
- CloudWatch metrics: VolumeReadOps, VolumeWriteOps, VolumeQueueLength

### Amazon EFS (Elastic File System)

**Performance Modes**
- **General Purpose**: Low latency, default, 7,000 file operations/second
- **Max I/O**: Higher latency, >7,000 operations/second, big data, media processing

**Throughput Modes**
- **Bursting**: Scales with file system size (50 MB/s per TB)
- **Provisioned**: Fixed throughput regardless of size
- **Elastic**: Automatically scales throughput (recommended)

**Storage Classes**
- **Standard**: Multi-AZ, highest availability
- **One Zone**: Single AZ, 47% cost savings
- **Infrequent Access (IA)**: Lifecycle management for cost savings
- Lifecycle policies to transition files not accessed for 7, 14, 30, 60, or 90 days

**Features**
- NFS v4.1 protocol
- Elastic capacity (automatic scaling)
- Multi-AZ redundancy (Standard class)
- Encryption at rest and in transit
- VPC-based access control with mount targets
- Access points for application-specific entry points
- Integration with AWS Backup
- Replication to another region for DR

**Use Cases**
- Shared content repositories
- Container storage (ECS, EKS persistent volumes)
- Web serving and content management
- Home directories
- Machine learning training data

### Amazon FSx Family

**FSx for Windows File Server**
- Fully managed Windows native file system
- SMB protocol support
- Active Directory integration (AWS Managed AD or self-managed)
- Multi-AZ deployment for high availability
- DFS namespaces for consolidating file shares
- Data deduplication for storage efficiency
- Storage types: SSD (latency-sensitive) and HDD (throughput-intensive)
- Backup to S3, restore to new file system
- Use cases: Windows applications, shared file storage, SQL Server, IIS

**FSx for Lustre**
- High-performance parallel file system for HPC, ML, video processing
- Sub-millisecond latencies, millions of IOPS, hundreds of GB/s throughput
- Integration with S3: Lazy loading from S3, write results back to S3
- Deployment options:
  - **Scratch**: Temporary, highest performance, no replication
  - **Persistent**: Long-term, replication, automatic failover
- Data compression (LZ4) for cost savings
- Use cases: Genomics, seismic processing, video rendering, financial modeling

**FSx for NetApp ONTAP**
- Fully managed NetApp ONTAP file system
- Multi-protocol: NFS, SMB, iSCSI
- Storage efficiency: Snapshots, clones, compression, deduplication
- Multi-AZ high availability
- Replication within and across regions
- SnapMirror for backup and DR
- Use cases: Enterprise applications, databases, VMware Cloud on AWS

**FSx for OpenZFS**
- Fully managed OpenZFS file system
- NFS v3/v4 protocol
- Up to 1 million IOPS, sub-millisecond latencies
- Point-in-time snapshots
- Data compression
- Use cases: Linux-based workloads, media processing, EDA (electronic design automation)

### AWS Storage Gateway

**File Gateway**
- NFS and SMB interface to S3
- Local cache for low-latency access to frequently used files
- Files stored as objects in S3
- Use cases: Hybrid cloud storage, backup, disaster recovery, tiering

**Volume Gateway**
- iSCSI block storage backed by S3
- **Cached Volumes**: Primary data in S3, frequently accessed data cached locally
- **Stored Volumes**: Primary data on-premises, asynchronously backed up to S3
- EBS snapshots for point-in-time backups
- Use cases: Backup, disaster recovery, migration

**Tape Gateway**
- Virtual tape library (VTL) backed by S3 and Glacier
- Compatible with existing backup software (Veeam, Veritas, Commvault)
- Eliminate physical tape infrastructure
- Use cases: Long-term archival, compliance, tape replacement

**S3 File Gateway vs. Volume Gateway**
- File Gateway: Access as files (NFS/SMB), stored as S3 objects
- Volume Gateway: Access as block storage (iSCSI), stored as S3 snapshots

### AWS Backup

**Centralized Backup Management**
- Unified backup across AWS services: EC2, EBS, RDS, DynamoDB, EFS, FSx, Storage Gateway
- Backup plans with schedules and retention rules
- Cross-region and cross-account backup
- Backup vault with access policies
- Backup vault lock for WORM compliance (immutable backups)
- Backup activity monitoring with CloudWatch and EventBridge
- AWS Organizations integration for centralized governance

**Use Cases**
- Automated backup for compliance (HIPAA, PCI-DSS)
- Centralized backup management across accounts
- Long-term retention with cost-effective storage
- Disaster recovery with cross-region backup

### Hybrid Storage Architectures

**AWS Outposts**
- AWS infrastructure on-premises
- S3 on Outposts for local object storage
- EBS on Outposts for local block storage
- Use cases: Data residency, low latency, local data processing

**AWS Snow Family**
- **Snowcone**: 8 TB HDD or 14 TB SSD, edge computing
- **Snowball Edge**: 80 TB or 210 TB, compute and storage
- **Snowmobile**: 100 PB, exabyte-scale data transfer
- Use cases: Large-scale data migration, edge computing, disaster recovery

**AWS DataSync**
- Automated data transfer between on-premises and AWS
- NFS, SMB, HDFS, S3, EFS, FSx
- Bandwidth throttling and scheduling
- Data verification and encryption
- Use cases: Migration, archival, disaster recovery, hybrid workflows

## AWS Services Reference

### Core Services

**Amazon S3**
- Object storage with 11 9's durability
- Storage classes for cost optimization
- Versioning, replication, lifecycle policies
- Event notifications and analytics

**Amazon EBS**
- Block storage for EC2 instances
- Multiple volume types for different workloads
- Snapshots for backup and disaster recovery
- Encryption and performance optimization

**Amazon EFS**
- Scalable NFS file system
- Multi-AZ redundancy
- Lifecycle management
- Elastic throughput

**Amazon FSx**
- Managed file systems: Windows, Lustre, NetApp ONTAP, OpenZFS
- Protocol support: SMB, NFS, iSCSI
- High performance and availability
- Enterprise features and integrations

### Supporting Services

**AWS Storage Gateway**
- Hybrid cloud storage integration
- File, volume, and tape gateway modes
- Local caching for performance
- S3-backed durable storage

**AWS Backup**
- Centralized backup management
- Cross-region and cross-account backup
- Compliance and governance
- Automated backup policies

**AWS DataSync**
- Automated data transfer
- On-premises to AWS and between AWS services
- Bandwidth management and scheduling
- Data validation

**AWS Transfer Family**
- Managed SFTP, FTPS, FTP, AS2 for S3 and EFS
- Existing workflow integration
- Authentication with existing identity providers
- Use cases: EDI, legacy application integration

**S3 Storage Lens**
- Organization-wide visibility into storage usage
- 29+ metrics and recommendations
- Free and advanced tier metrics
- Dashboard and export to S3

## Architecture Patterns

### Pattern 1: Data Lake on S3

**Use Case**
- Centralized repository for structured and unstructured data
- Analytics, ML, and business intelligence
- Cost-effective storage with lifecycle management
- Data governance and access control

**Implementation Approach**
1. **Zones**: Raw, Processed, Curated (multi-tier architecture)
2. **S3 Bucket Structure**: Separate buckets or prefixes for each zone
3. **Lifecycle Policies**: Transition to IA/Glacier based on access patterns
4. **Replication**: CRR for disaster recovery
5. **Security**: Bucket policies, IAM roles, VPC endpoints, encryption
6. **Cataloging**: AWS Glue Data Catalog for metadata
7. **Access**: S3 Access Points for different applications/teams
8. **Governance**: AWS Lake Formation for permissions and audit
9. **Analytics**: Athena, Redshift Spectrum, EMR, QuickSight

**Pros/Cons**
- Pros: Cost-effective, scalable, decoupled storage and compute, supports diverse data types
- Cons: Eventual consistency for some operations, governance complexity, learning curve

### Pattern 2: Cross-Region Disaster Recovery

**Use Case**
- Business continuity for mission-critical applications
- RTO/RPO requirements in minutes
- Geographic redundancy for natural disasters
- Compliance with data residency requirements

**Implementation Approach**
1. **Primary Region**: Production workloads with EBS-backed EC2, RDS, S3
2. **DR Region**: Standby infrastructure or continuous replication
3. **S3 Replication**: CRR with Replication Time Control for <15 minute RPO
4. **EBS Snapshots**: Automated daily snapshots, copy to DR region
5. **RDS**: Read replicas in DR region or automated backups with cross-region copy
6. **AMIs**: Copy to DR region for quick EC2 recovery
7. **Route 53**: Health checks and failover routing
8. **Automation**: Lambda or Systems Manager for failover orchestration
9. **Testing**: Regular DR drills to validate RTO/RPO

**Pros/Cons**
- Pros: Geographic redundancy, automated replication, customizable RTO/RPO
- Cons: Cross-region data transfer costs, complexity, duplicate infrastructure costs

### Pattern 3: Hybrid Cloud with Storage Gateway

**Use Case**
- Gradual cloud migration while maintaining on-premises systems
- Low-latency local access with cloud backup
- Cost-effective archival and disaster recovery
- Bandwidth-constrained environments

**Implementation Approach**
1. **File Gateway** deployed on-premises (VMware, Hyper-V, or hardware appliance)
2. **Local Cache** for frequently accessed data (low latency)
3. **S3 Buckets** as primary durable storage
4. **Lifecycle Policies** to transition to Glacier for archival
5. **AWS Backup** for centralized backup management
6. **Direct Connect or VPN** for private connectivity
7. **CloudWatch** for monitoring cache hit rate and upload/download
8. **IAM Roles** for Storage Gateway permissions to S3

**Pros/Cons**
- Pros: Seamless integration, low-latency local access, cloud scalability, cost-effective
- Cons: Bandwidth dependency, cache sizing, gateway maintenance

### Pattern 4: High-Performance Computing with FSx for Lustre

**Use Case**
- Genomics, financial modeling, video rendering, ML training
- Millions of IOPS and sub-millisecond latencies
- Large datasets in S3 that need high-performance processing
- Temporary or long-term high-performance file systems

**Implementation Approach**
1. **S3 Bucket** with input datasets
2. **FSx for Lustre** file system linked to S3 bucket
3. **Lazy Loading**: Data loaded from S3 on first access
4. **Compute Cluster**: EC2 instances (HPC-optimized, GPU) mounting Lustre file system
5. **Processing**: Parallel workloads with high I/O requirements
6. **Export**: Results written back to S3 (manual export or auto-export)
7. **Deployment Type**: Scratch for temporary, Persistent for long-term
8. **Scaling**: Add more file systems or increase throughput

**Pros/Cons**
- Pros: Extreme performance, S3 integration, cost-effective for temporary workloads
- Cons: Complex setup, Scratch type has no replication, higher cost than S3

### Pattern 5: Enterprise File Sharing with FSx for Windows

**Use Case**
- Migrating Windows file servers to AWS
- Shared drives for enterprise applications
- Active Directory integrated file permissions
- Multi-AZ high availability

**Implementation Approach**
1. **FSx for Windows File Server** in Multi-AZ configuration
2. **Active Directory** integration (AWS Managed AD or self-managed)
3. **DFS Namespaces** for unified namespace across file systems
4. **Data Deduplication** to reduce storage costs
5. **Backup**: AWS Backup for automated daily backups
6. **Replication**: FSx file replication to another region for DR
7. **Access**: SMB shares from on-premises via Direct Connect/VPN
8. **Monitoring**: CloudWatch metrics and alarms

**Pros/Cons**
- Pros: Native Windows integration, Multi-AZ HA, AD permissions, fully managed
- Cons: Higher cost than EFS, Windows-specific

### Pattern 6: Compliance and Archival with S3 Glacier

**Use Case**
- Long-term retention for regulatory compliance (7-10 years)
- Infrequently accessed data with rare retrieval needs
- Cost optimization for backup retention
- WORM requirements for immutable records

**Implementation Approach**
1. **S3 Bucket** for active data
2. **Lifecycle Policies**: Transition to Glacier Flexible Retrieval after 90 days, Deep Archive after 1 year
3. **S3 Object Lock** in Compliance mode for immutability
4. **Vault Lock** for additional protection
5. **S3 Inventory** for audit trail
6. **Retrieval**: Bulk retrieval for cost-effectiveness (5-12 hours)
7. **Notifications**: EventBridge for restore completion events
8. **Access Logging** and CloudTrail for audit

**Pros/Cons**
- Pros: Lowest storage cost ($1/TB/month Deep Archive), compliance features, durable
- Cons: Retrieval time (hours), retrieval costs, minimum storage duration (180 days)

## Best Practices

### Enterprise-Level Recommendations

**Storage Strategy**
- Match storage service to workload characteristics (performance, durability, cost)
- Implement lifecycle policies for cost optimization
- Use tiered storage approach (hot, warm, cold, archive)
- Plan for data growth and scalability
- Regular review and optimization based on access patterns

**Data Protection**
- Implement 3-2-1 backup rule: 3 copies, 2 different media, 1 offsite
- Automate backups with AWS Backup or service-native features
- Test restore procedures regularly
- Cross-region replication for disaster recovery
- Versioning for accidental deletion protection

**Cost Optimization**
- Use S3 Intelligent-Tiering for unknown or changing access patterns
- Lifecycle policies to transition and expire objects
- S3 Storage Class Analysis for data-driven decisions
- Delete incomplete multipart uploads
- Right-size EBS volumes and use gp3 over gp2
- Monitor and delete unused snapshots and volumes

### Security Considerations

**Encryption**
- Enable encryption at rest for all storage services
- Use AWS KMS for key management and audit trail
- S3 Bucket Keys to reduce KMS costs
- Enforce encryption with bucket policies and SCPs
- TLS for data in transit

**Access Control**
- Principle of least privilege for IAM policies
- S3 Access Points for application-specific access
- VPC endpoints for private access to S3 and other services
- S3 Block Public Access at account and bucket level
- Regular access reviews and removal of unused permissions

**Data Governance**
- Data classification and tagging
- S3 Object Lock for compliance and WORM
- AWS Lake Formation for data lake governance
- S3 Access Analyzer to identify external access
- CloudTrail logging for audit trail
- S3 Inventory for asset management

### Performance Tuning

**S3 Performance**
- Use prefixes to parallelize requests (3,500 PUT, 5,500 GET per prefix/second)
- Multipart upload for large files (>100 MB)
- Byte-range fetches for parallel downloads
- S3 Transfer Acceleration for global uploads
- CloudFront for frequently accessed objects
- Use S3 Select to reduce data transfer

**EBS Performance**
- Choose appropriate volume type for workload (IOPS vs. throughput)
- gp3 for cost-effective performance (adjustable IOPS and throughput)
- io2 Block Express for highest performance (256,000 IOPS)
- EBS-optimized instances for dedicated bandwidth
- Monitor CloudWatch metrics: VolumeQueueLength, BurstBalance

**File System Performance**
- EFS Elastic throughput mode for automatic scaling
- FSx for Lustre for HPC workloads (millions of IOPS)
- FSx for Windows SSD for latency-sensitive workloads
- Provisioned throughput when bursting is insufficient

## Common Scenarios

### Scenario 1: 100 PB Data Lake Migration

**Context**: Enterprise migrating 100 PB of on-premises data to S3 data lake over 6 months

**Architecture**
- **AWS Snowmobile** for initial 80 PB bulk transfer (10 trucks)
- **AWS DataSync** for remaining 20 PB and incremental updates
- **S3 Bucket Structure**: `/raw/`, `/processed/`, `/curated/` prefixes
- **Storage Classes**: Standard → Intelligent-Tiering (auto-optimization)
- **Lifecycle Policies**: Transition raw data to Glacier after 90 days
- **Replication**: CRR to DR region for disaster recovery
- **Security**: S3 Bucket Keys with KMS, VPC endpoints, bucket policies
- **Governance**: AWS Lake Formation for fine-grained access control
- **Cataloging**: AWS Glue Crawler for automatic schema discovery
- **Analytics**: Athena for ad-hoc queries, Redshift Spectrum for complex analytics

**Key Considerations**
- Snowmobile scheduling and logistics (10-truck coordination)
- Network bandwidth for DataSync (10 Gbps Direct Connect)
- S3 request rate planning (use prefixes for parallelization)
- Cost: Snowmobile, storage ($23/TB/month Standard, transition to Intelligent-Tiering)
- Data validation and integrity checks
- Incremental updates during migration period

### Scenario 2: Global Media Delivery Platform

**Context**: Streaming service delivering video content to 50M users globally with low latency

**Architecture**
- **S3 Standard** for source video files (multi-region)
- **CloudFront** with 400+ edge locations for low-latency delivery
- **S3 Transfer Acceleration** for content ingestion from global studios
- **S3 Multi-Region Access Points** for simplified global access
- **Lifecycle Policies**: Transition old content to Glacier Flexible Retrieval
- **Lambda@Edge** for authentication and dynamic content manipulation
- **MediaConvert** for transcoding to multiple formats/bitrates
- **Adaptive Bitrate Streaming**: HLS/DASH manifests in S3
- **Monitoring**: S3 Storage Lens, CloudFront metrics, real-time logs to Kinesis

**Key Considerations**
- Content storage in multiple regions for redundancy
- CloudFront cache hit ratio optimization (>80%)
- Data transfer costs (CloudFront to internet, S3 to CloudFront)
- Origin failover for high availability
- DRM and content protection
- Storage costs for multiple quality renditions

### Scenario 3: Financial Services Backup and Compliance

**Context**: Bank with regulatory requirements for 7-year retention, immutable backups, and disaster recovery

**Architecture**
- **AWS Backup** for centralized backup across EC2, EBS, RDS, EFS
- **Backup Vault Lock** for WORM compliance (immutable for 7 years)
- **Cross-Region Backup** to DR region (different geography)
- **S3 Object Lock** in Compliance mode for archived documents
- **Lifecycle Policies**: Transition to Glacier Deep Archive after 1 year
- **Encryption**: KMS customer-managed keys with annual rotation
- **Monitoring**: EventBridge for backup job status, CloudWatch alarms
- **Access Control**: Least privilege IAM, MFA delete for S3, backup vault access policies
- **Audit**: CloudTrail logs to immutable S3 bucket, S3 Access Analyzer

**Key Considerations**
- RPO: 15 minutes (continuous replication for critical databases)
- RTO: 4 hours (automated restore procedures)
- Compliance: SOC 2, PCI-DSS, regulatory audit requirements
- Backup testing: Quarterly restore drills
- Cost: Multi-region backup, long-term retention in Glacier Deep Archive ($1/TB/month)
- Legal hold for litigation support

### Scenario 4: HPC Genomics Pipeline

**Context**: Research institute processing genomic sequences requiring high I/O performance

**Architecture**
- **S3 Buckets** for raw sequencing data (petabytes)
- **FSx for Lustre Scratch** file system linked to S3 (temporary, highest performance)
- **EC2 Compute Cluster**: c5n.18xlarge instances with 100 Gbps networking
- **Lazy Loading**: Sequencing files loaded from S3 on first access
- **Processing**: Parallel alignment, variant calling on Lustre file system
- **Export**: Results exported to S3, Lustre file system deleted
- **Lifecycle**: Transition raw data to Glacier Flexible Retrieval after processing
- **Batch**: AWS Batch for job scheduling and resource management
- **Spot Instances**: 70% cost savings for fault-tolerant processing

**Key Considerations**
- Lustre performance: 1M+ IOPS, 100 GB/s+ throughput
- S3 integration: Lazy loading reduces data movement time
- Cost: Scratch file system only during processing (delete after)
- Spot Instances for cost savings (handle interruptions)
- Data deduplication: Many samples share reference genomes
- Security: Encryption for PHI/PII, VPC isolation, IAM roles

### Scenario 5: Hybrid Enterprise File Services

**Context**: Enterprise with 500 TB of file shares on-premises, migrating to AWS with hybrid access during transition

**Architecture**
- **AWS File Gateway** on-premises (10 TB local cache)
- **S3 Buckets** as primary storage for each file share
- **FSx for Windows File Server** in AWS for cloud-native applications
- **DataSync** for one-time bulk migration from legacy file servers
- **Active Directory** integration (on-prem trust with AWS Managed AD)
- **DFS Namespaces** for unified view of on-prem and cloud file shares
- **Direct Connect** (10 Gbps) for private, high-bandwidth connectivity
- **Lifecycle Policies**: Transition infrequently accessed files to S3 IA
- **Backup**: AWS Backup for FSx, S3 versioning for File Gateway

**Key Considerations**
- File Gateway cache sizing (10 TB for 500 TB = 2% - right-size based on working set)
- Direct Connect bandwidth for user access patterns
- Migration phases: Critical shares first, then departmental, then archive
- User training for cloud-native file access
- Performance: Local cache for hot data, S3 for cold data
- Cost: File Gateway, S3 storage, FSx, Direct Connect, data transfer

## AWS CLI Examples

```bash
# S3 - Create bucket with versioning and encryption
aws s3api create-bucket \
  --bucket my-enterprise-bucket \
  --region us-east-1 \
  --create-bucket-configuration LocationConstraint=us-east-1

aws s3api put-bucket-versioning \
  --bucket my-enterprise-bucket \
  --versioning-configuration Status=Enabled

aws s3api put-bucket-encryption \
  --bucket my-enterprise-bucket \
  --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms","KMSMasterKeyID":"arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"},"BucketKeyEnabled":true}]}'

# S3 - Enable Block Public Access
aws s3api put-public-access-block \
  --bucket my-enterprise-bucket \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# S3 - Create lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-enterprise-bucket \
  --lifecycle-configuration file://lifecycle.json

# lifecycle.json example:
# {
#   "Rules": [
#     {
#       "Id": "TransitionToIA",
#       "Status": "Enabled",
#       "Transitions": [
#         {"Days": 30, "StorageClass": "STANDARD_IA"},
#         {"Days": 90, "StorageClass": "GLACIER_IR"},
#         {"Days": 365, "StorageClass": "DEEP_ARCHIVE"}
#       ],
#       "NoncurrentVersionTransitions": [
#         {"NoncurrentDays": 30, "StorageClass": "STANDARD_IA"}
#       ],
#       "NoncurrentVersionExpiration": {"NoncurrentDays": 90}
#     }
#   ]
# }

# S3 - Enable replication
aws s3api put-bucket-replication \
  --bucket my-source-bucket \
  --replication-configuration file://replication.json

# replication.json example:
# {
#   "Role": "arn:aws:iam::123456789012:role/S3ReplicationRole",
#   "Rules": [
#     {
#       "Status": "Enabled",
#       "Priority": 1,
#       "DeleteMarkerReplication": {"Status": "Enabled"},
#       "Filter": {},
#       "Destination": {
#         "Bucket": "arn:aws:s3:::my-destination-bucket",
#         "ReplicationTime": {"Status": "Enabled", "Time": {"Minutes": 15}},
#         "Metrics": {"Status": "Enabled", "EventThreshold": {"Minutes": 15}},
#         "StorageClass": "STANDARD_IA"
#       }
#     }
#   ]
# }

# S3 - Enable Object Lock
aws s3api put-object-lock-configuration \
  --bucket my-compliance-bucket \
  --object-lock-configuration '{"ObjectLockEnabled":"Enabled","Rule":{"DefaultRetention":{"Mode":"COMPLIANCE","Years":7}}}'

# S3 - Create S3 Access Point
aws s3control create-access-point \
  --account-id 123456789012 \
  --name analytics-team-ap \
  --bucket my-enterprise-bucket \
  --vpc-configuration VpcId=vpc-12345678

# S3 - Enable S3 Inventory
aws s3api put-bucket-inventory-configuration \
  --bucket my-enterprise-bucket \
  --id DailyInventory \
  --inventory-configuration file://inventory.json

# S3 - Multipart upload for large files
aws s3api create-multipart-upload \
  --bucket my-bucket \
  --key large-file.zip \
  --server-side-encryption aws:kms

# Upload parts (repeat for each part)
aws s3api upload-part \
  --bucket my-bucket \
  --key large-file.zip \
  --part-number 1 \
  --body part1.bin \
  --upload-id <upload-id>

# Complete multipart upload
aws s3api complete-multipart-upload \
  --bucket my-bucket \
  --key large-file.zip \
  --upload-id <upload-id> \
  --multipart-upload file://parts.json

# EBS - Create encrypted volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 1000 \
  --volume-type gp3 \
  --iops 10000 \
  --throughput 500 \
  --encrypted \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=Production-DB-Volume}]'

# EBS - Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-12345678 \
  --description "Daily backup of production DB" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=Prod-DB-Snapshot},{Key=Backup,Value=Daily}]'

# EBS - Copy snapshot to another region
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-12345678 \
  --destination-region us-west-2 \
  --description "DR copy of production DB snapshot"

# EBS - Enable Fast Snapshot Restore
aws ec2 enable-fast-snapshot-restores \
  --availability-zones us-east-1a us-east-1b \
  --source-snapshot-id snap-12345678

# EBS - Modify volume (increase size and performance)
aws ec2 modify-volume \
  --volume-id vol-12345678 \
  --size 2000 \
  --volume-type gp3 \
  --iops 16000 \
  --throughput 1000

# EBS - Create Data Lifecycle Manager policy
aws dlm create-lifecycle-policy \
  --execution-role-arn arn:aws:iam::123456789012:role/AWSDataLifecycleManagerRole \
  --description "Daily snapshots with 7-day retention" \
  --state ENABLED \
  --policy-details file://dlm-policy.json

# EFS - Create file system
aws efs create-file-system \
  --performance-mode generalPurpose \
  --throughput-mode elastic \
  --encrypted \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012 \
  --tags Key=Name,Value=Shared-Application-Storage

# EFS - Create mount target
aws efs create-mount-target \
  --file-system-id fs-12345678 \
  --subnet-id subnet-12345678 \
  --security-groups sg-12345678

# EFS - Create access point
aws efs create-access-point \
  --file-system-id fs-12345678 \
  --posix-user Uid=1000,Gid=1000 \
  --root-directory Path=/app1,CreationInfo={OwnerUid=1000,OwnerGid=1000,Permissions=755} \
  --tags Key=Name,Value=App1-AccessPoint

# EFS - Enable lifecycle management
aws efs put-lifecycle-configuration \
  --file-system-id fs-12345678 \
  --lifecycle-policies TransitionToIA=AFTER_30_DAYS,TransitionToPrimaryStorageClass=AFTER_1_ACCESS

# FSx for Windows - Create file system
aws fsx create-file-system \
  --file-system-type WINDOWS \
  --storage-capacity 1024 \
  --storage-type SSD \
  --subnet-ids subnet-12345678 subnet-87654321 \
  --windows-configuration \
    ActiveDirectoryId=d-1234567890,\
ThroughputCapacity=32,\
WeeklyMaintenanceStartTime=1:00:00,\
DailyAutomaticBackupStartTime=01:00,\
AutomaticBackupRetentionDays=7,\
DeploymentType=MULTI_AZ_1 \
  --tags Key=Name,Value=Enterprise-FileServer

# FSx for Lustre - Create file system linked to S3
aws fsx create-file-system \
  --file-system-type LUSTRE \
  --storage-capacity 7200 \
  --subnet-ids subnet-12345678 \
  --lustre-configuration \
    ImportPath=s3://my-source-bucket/input,\
ExportPath=s3://my-source-bucket/output,\
DeploymentType=SCRATCH_2,\
PerUnitStorageThroughput=200 \
  --tags Key=Name,Value=HPC-Lustre-FS

# Storage Gateway - Activate gateway
aws storagegateway activate-gateway \
  --activation-key <activation-key> \
  --gateway-name "OnPrem-FileGateway" \
  --gateway-timezone "GMT-5:00" \
  --gateway-region us-east-1 \
  --gateway-type FILE_S3

# Storage Gateway - Create NFS file share
aws storagegateway create-nfs-file-share \
  --client-token $(uuidgen) \
  --gateway-arn arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-12345678 \
  --location-arn arn:aws:s3:::my-gateway-bucket/share1 \
  --role arn:aws:iam::123456789012:role/StorageGatewayRole \
  --client-list 10.0.0.0/8 \
  --squash RootSquash \
  --default-storage-class S3_INTELLIGENT_TIERING

# AWS Backup - Create backup vault
aws backup create-backup-vault \
  --backup-vault-name "Enterprise-Backup-Vault" \
  --encryption-key-arn arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012

# AWS Backup - Create backup plan
aws backup create-backup-plan \
  --backup-plan file://backup-plan.json

# backup-plan.json example:
# {
#   "BackupPlanName": "DailyBackupPlan",
#   "Rules": [
#     {
#       "RuleName": "DailyBackup",
#       "TargetBackupVaultName": "Enterprise-Backup-Vault",
#       "ScheduleExpression": "cron(0 2 * * ? *)",
#       "StartWindowMinutes": 60,
#       "CompletionWindowMinutes": 120,
#       "Lifecycle": {
#         "DeleteAfterDays": 35,
#         "MoveToColdStorageAfterDays": 7
#       },
#       "CopyActions": [
#         {
#           "DestinationBackupVaultArn": "arn:aws:backup:us-west-2:123456789012:backup-vault:DR-Vault",
#           "Lifecycle": {"DeleteAfterDays": 35}
#         }
#       ]
#     }
#   ]
# }

# AWS Backup - Assign resources to backup plan
aws backup create-backup-selection \
  --backup-plan-id <plan-id> \
  --backup-selection file://selection.json

# DataSync - Create location for on-premises NFS
aws datasync create-location-nfs \
  --server-hostname 192.168.1.10 \
  --subdirectory /exports/data \
  --on-prem-config AgentArns=arn:aws:datasync:us-east-1:123456789012:agent/agent-12345678

# DataSync - Create location for S3
aws datasync create-location-s3 \
  --s3-bucket-arn arn:aws:s3:::my-datasync-bucket \
  --s3-storage-class INTELLIGENT_TIERING \
  --s3-config BucketAccessRoleArn=arn:aws:iam::123456789012:role/DataSyncS3Role

# DataSync - Create task
aws datasync create-task \
  --source-location-arn <nfs-location-arn> \
  --destination-location-arn <s3-location-arn> \
  --name "OnPremToS3Migration" \
  --options VerifyMode=POINT_IN_TIME_CONSISTENT,OverwriteMode=ALWAYS,PreserveDeletedFiles=REMOVE \
  --schedule ScheduleExpression="cron(0 2 * * ? *)"

# DataSync - Start task execution
aws datasync start-task-execution \
  --task-arn <task-arn>
```

## Study Tips

### SAP-C02 Exam Focus Areas

**High-Priority Storage Topics**
- S3 storage class selection based on access patterns and cost
- S3 lifecycle policies for automated transitions
- Cross-region and same-region replication use cases
- EBS volume types and performance characteristics
- FSx family comparison (Windows, Lustre, ONTAP, OpenZFS)
- Storage Gateway modes (File, Volume, Tape)
- Hybrid storage architectures
- Backup and disaster recovery strategies
- Data migration tools (Snowball, DataSync, Transfer Family)
- Cost optimization for storage

**Scenario-Based Questions**
- Given access patterns, select appropriate S3 storage class
- Design DR solution with RTO/RPO requirements
- Choose between EFS, FSx, and EBS for specific workloads
- Hybrid cloud storage integration patterns
- HPC and high-performance storage solutions
- Compliance and immutable storage requirements
- Cost optimization for petabyte-scale storage
- Multi-region data replication strategies

**Common Decision Points**
- **S3 vs. EBS**: Object storage vs. block storage
- **EFS vs. FSx for Windows**: Linux NFS vs. Windows SMB
- **FSx for Lustre vs. EFS**: HPC performance vs. general-purpose
- **S3 Standard vs. Intelligent-Tiering**: Known vs. unknown access patterns
- **Glacier Flexible vs. Deep Archive**: Retrieval time vs. cost
- **Storage Gateway vs. DataSync**: Hybrid access vs. one-time migration
- **Snapshots vs. AWS Backup**: Service-native vs. centralized management

### Key Differences from SAA-C03

**SAA-C03 Storage Knowledge**
- Basic S3 storage classes and lifecycle policies
- EBS volume types and basic performance
- EFS for shared file storage
- Simple backup with snapshots

**Additional SAP-C02 Requirements**
- Advanced S3 features: Replication Time Control, Multi-Region Access Points, Batch Operations
- S3 Object Lock and Vault Lock for compliance
- S3 Storage Lens for organization-wide optimization
- EBS Multi-Attach and io2 Block Express
- FSx family detailed comparison and use cases
- Storage Gateway integration patterns
- Hybrid storage architectures with DataSync
- AWS Backup centralized management across services and accounts
- Large-scale data migration with Snow family
- Cost optimization at petabyte scale

### Complex Scenarios to Master

**Hybrid Storage Integration**
- Storage Gateway with on-premises caching
- DataSync for incremental migrations
- Direct Connect bandwidth planning for storage traffic
- Hybrid DNS and Active Directory integration with FSx
- Multi-site file synchronization

**Disaster Recovery**
- Multi-tier recovery strategies (backup, pilot light, warm standby, active-active)
- Cross-region replication for S3, EBS, RDS
- RTO/RPO calculations and solution design
- Automated failover and recovery procedures
- DR testing and validation

**High-Performance Computing**
- FSx for Lustre with S3 integration
- Parallel file systems for genomics, financial modeling
- Instance networking (ENA, EFA) with storage
- Optimizing IOPS and throughput for large datasets

**Cost Optimization**
- S3 Storage Class Analysis and Intelligent-Tiering
- Lifecycle policies to transition and expire data
- Compression and deduplication strategies
- Data transfer cost optimization
- Right-sizing EBS volumes and switching from gp2 to gp3

**Compliance and Governance**
- Immutable backups with S3 Object Lock and Backup Vault Lock
- Encryption strategies (SSE-S3, SSE-KMS, client-side)
- Audit logging with CloudTrail and S3 Access Logging
- Data residency and sovereignty requirements
- WORM compliance for financial and healthcare

### Practice Lab Recommendations

1. **S3 Advanced Features**
   - Create lifecycle policies with multiple transitions
   - Enable cross-region replication with RTC
   - Configure S3 Object Lock in Compliance mode
   - Set up S3 Access Points and Multi-Region Access Points
   - Use S3 Batch Operations to modify metadata

2. **EBS Performance Testing**
   - Create gp3, io2, and st1 volumes
   - Benchmark IOPS and throughput with fio
   - Test snapshot creation and restoration
   - Enable Fast Snapshot Restore and measure impact
   - Modify volume size and performance without downtime

3. **FSx Comparison**
   - Create FSx for Windows with AD integration
   - Create FSx for Lustre with S3 linkage
   - Mount file systems from EC2 instances
   - Test performance characteristics
   - Compare costs and use cases

4. **Hybrid Storage with Storage Gateway**
   - Deploy File Gateway VM on-premises (or simulate)
   - Create NFS share backed by S3
   - Test local caching behavior
   - Monitor CloudWatch metrics
   - Implement lifecycle policies on backing S3 bucket

5. **DataSync Migration**
   - Create source and destination locations
   - Configure DataSync task with filtering
   - Monitor task execution and performance
   - Compare with manual S3 sync
   - Calculate data transfer costs

6. **AWS Backup Centralized Management**
   - Create backup vault with vault lock
   - Create backup plan with cross-region copy
   - Assign resources using tags
   - Perform backup and restore
   - Review backup costs and optimization

## Additional Resources

### AWS Whitepapers

- AWS Storage Services Overview
- Backup and Recovery Approaches Using AWS
- Best Practices for Amazon S3
- Amazon S3 Performance Optimization
- Cost Optimization for Storage
- Hybrid Cloud with AWS (storage chapter)

### Documentation Links

**S3 Documentation**
- Amazon S3: https://docs.aws.amazon.com/s3/
- S3 Glacier: https://docs.aws.amazon.com/glacier/
- S3 Storage Lens: https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage_lens.html

**EBS Documentation**
- Amazon EBS: https://docs.aws.amazon.com/ebs/
- EBS Volume Types: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html
- EBS Snapshots: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html

**File Systems**
- Amazon EFS: https://docs.aws.amazon.com/efs/
- FSx for Windows: https://docs.aws.amazon.com/fsx/latest/WindowsGuide/
- FSx for Lustre: https://docs.aws.amazon.com/fsx/latest/LustreGuide/
- FSx for ONTAP: https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/
- FSx for OpenZFS: https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/

**Hybrid and Migration**
- AWS Storage Gateway: https://docs.aws.amazon.com/storagegateway/
- AWS DataSync: https://docs.aws.amazon.com/datasync/
- AWS Transfer Family: https://docs.aws.amazon.com/transfer/
- AWS Snow Family: https://docs.aws.amazon.com/snowball/

**Backup and DR**
- AWS Backup: https://docs.aws.amazon.com/aws-backup/
- AWS Elastic Disaster Recovery: https://docs.aws.amazon.com/drs/

### Video Resources
- AWS re:Invent sessions on storage (search "STG" track)
- Deep Dive on Amazon S3
- EBS Performance Optimization
- FSx for Lustre for HPC Workloads
