# SAP Architecture on AWS

## SAP on AWS Overview

### Benefits of Running SAP on AWS

**Scalability**
- Elastic infrastructure
- Scale up/down based on demand
- Handle peak workloads
- Global deployment

**Performance**
- High-performance compute instances
- Low-latency storage
- Enhanced networking
- Proximity to data centers

**Cost Optimization**
- Pay-as-you-go pricing
- Reserved Instances for predictable workloads
- Savings Plans
- No upfront infrastructure investment

**High Availability**
- Multiple Availability Zones
- Disaster recovery options
- Automated backups
- 99.99% SLA

**Security**
- Compliance certifications (SAP certified)
- Encryption at rest and in transit
- IAM for access control
- VPC isolation

## SAP-Certified EC2 Instances

### Instance Families for SAP

**Memory-Optimized (R-family)**
- R5, R6i, R6a
- High memory-to-CPU ratio
- Ideal for HANA databases
- Up to 24 TB RAM

**Compute-Optimized (C-family)**
- C5, C6i
- High CPU performance
- SAP application servers
- Batch processing

**General Purpose (M-family)**
- M5, M6i
- Balanced compute and memory
- Development/test systems
- Non-production workloads

**High Memory (U-family)**
- u-6tb1.metal, u-9tb1.metal, u-12tb1.metal
- Up to 24 TB RAM
- SAP HANA scale-up
- Single-socket performance

**Storage-Optimized (I-family)**
- i3, i3en
- High IOPS
- SAP BW on HANA
- Data warehousing

### Instance Sizing Guide

**SAP HANA Sizing**
```
SAPS (SAP Application Performance Standard)
- Measure of SAP system performance
- 100 SAPS = 2,000 fully business processed order line items/hour

Memory Sizing:
- Production: RAM = (Database size × 1.5) + (50-100 GB for OS/services)
- Non-production: RAM = Database size + overhead
- Growth buffer: Add 20-30%

Example:
- 500 GB database
- Production: 500 × 1.5 + 75 = 825 GB minimum
- Choose r5.24xlarge (768 GB) or r6i.32xlarge (1024 GB)
```

**Application Server Sizing**
```
Based on users and transactions:
- 100-500 users: m5.4xlarge to m5.8xlarge
- 500-1000 users: m5.12xlarge to m5.16xlarge
- 1000+ users: m5.24xlarge or multiple servers

SAPS requirements:
- Light users: 200-300 SAPS per user
- Medium users: 400-600 SAPS per user
- Heavy users: 800-1200 SAPS per user
```

## SAP Architecture Patterns

### Three-Tier Architecture

**Classic SAP NetWeaver Stack**
```
┌─────────────────────────────────────┐
│         Presentation Tier            │
│    (SAP GUI, Fiori, Web Browser)    │
└─────────────────────────────────────┘
                  │
┌─────────────────────────────────────┐
│         Application Tier             │
│   ┌─────────────────────────────┐   │
│   │  SAP Application Servers    │   │
│   │  (ABAP, Java, Web Dispatcher)│   │
│   └─────────────────────────────┘   │
└─────────────────────────────────────┘
                  │
┌─────────────────────────────────────┐
│          Database Tier               │
│   ┌─────────────────────────────┐   │
│   │    SAP HANA / AnyDB         │   │
│   │  (Primary + Secondary/DR)    │   │
│   └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

**AWS Implementation**
```
VPC: 10.0.0.0/16

Public Subnets (for Web Dispatcher)
├── AZ-A: 10.0.1.0/24 (Web Dispatcher, Bastion)
└── AZ-B: 10.0.2.0/24 (Web Dispatcher failover)

Private Subnets - Application Tier
├── AZ-A: 10.0.10.0/24 (SAP App Servers)
└── AZ-B: 10.0.11.0/24 (SAP App Servers)

Private Subnets - Database Tier
├── AZ-A: 10.0.20.0/24 (HANA Primary)
└── AZ-B: 10.0.21.0/24 (HANA Secondary)

Management Subnets
├── AZ-A: 10.0.30.0/24 (SAP Solution Manager)
└── AZ-B: 10.0.31.0/24 (Backup services)
```

### S/4HANA on AWS

**Deployment Options**

1. **Greenfield (New Implementation)**
- Fresh S/4HANA installation
- Modern Fiori UI
- Simplified data model
- Cloud-native architecture

2. **Brownfield (System Conversion)**
- Convert existing ECC to S/4HANA
- Preserve customizations
- Incremental migration
- Less disruption

3. **Hybrid Approach**
- Selective data migration
- New S/4HANA with legacy integration
- Phased transition

**Reference Architecture**
```yaml
# CloudFormation for S/4HANA Base Infrastructure

Resources:
  # VPC for SAP
  SAPVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: SAP-Production-VPC

  # HANA Database Instance
  HANAPrimaryInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: r5.24xlarge
      ImageId: !Ref HANAAMIId
      SubnetId: !Ref DatabaseSubnetA
      SecurityGroupIds:
        - !Ref HANASecurityGroup
      IamInstanceProfile: !Ref HANAInstanceProfile
      BlockDeviceMappings:
        # Root volume
        - DeviceName: /dev/sda1
          Ebs:
            VolumeSize: 50
            VolumeType: gp3
            Encrypted: true
        # HANA data volume
        - DeviceName: /dev/sdb
          Ebs:
            VolumeSize: 2000
            VolumeType: gp3
            Iops: 16000
            Throughput: 1000
            Encrypted: true
        # HANA log volume
        - DeviceName: /dev/sdc
          Ebs:
            VolumeSize: 512
            VolumeType: gp3
            Iops: 10000
            Encrypted: true
        # HANA shared volume
        - DeviceName: /dev/sdd
          Ebs:
            VolumeSize: 512
            VolumeType: gp3
            Encrypted: true
      Tags:
        - Key: Name
          Value: HANA-Primary
        - Key: SAP-SID
          Value: S4H
        - Key: Backup
          Value: "true"

  # Application Server
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: m5.8xlarge
      ImageId: !Ref SAPAppAMIId
      SubnetId: !Ref AppSubnetA
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      BlockDeviceMappings:
        - DeviceName: /dev/sda1
          Ebs:
            VolumeSize: 100
            VolumeType: gp3
            Encrypted: true
        - DeviceName: /dev/sdb
          Ebs:
            VolumeSize: 200
            VolumeType: gp3
            Encrypted: true
      Tags:
        - Key: Name
          Value: SAP-AppServer-01
        - Key: SAP-SID
          Value: S4H
```

### SAP HANA Scale-Out Architecture

**Multi-Node Configuration**
```
Master Node (Coordinator)
├── Index Server
├── Name Server
└── Preprocessing Server

Worker Nodes (1-15)
├── Index Server (data partition)
├── XS Engine
└── Application Processing

Standby Nodes (Optional)
└── Automatic failover
```

**AWS Implementation**
```
Placement Group (Cluster Strategy)
├── Master Node: r5.24xlarge (AZ-A)
├── Worker 1: r5.24xlarge (AZ-A)
├── Worker 2: r5.24xlarge (AZ-A)
├── Worker 3: r5.24xlarge (AZ-A)
└── Standby: r5.24xlarge (AZ-B)

Shared Storage:
├── FSx for Lustre (high-performance)
├── EFS (for /hana/shared)
└── S3 (backups)
```

## Storage Architecture

### EBS Volumes for SAP

**Volume Types and Use Cases**

| Volume Type | IOPS | Throughput | Use Case |
|------------|------|------------|----------|
| **gp3** | 16,000 | 1,000 MB/s | HANA data, logs, general purpose |
| **io2** | 64,000 | 1,000 MB/s | High-performance HANA |
| **io2 Block Express** | 256,000 | 4,000 MB/s | Extreme performance |
| **st1** | 500 | 500 MB/s | Throughput-optimized (backups) |
| **sc1** | 250 | 250 MB/s | Cold storage (archives) |

**HANA Volume Configuration**
```bash
# Example for 1 TB HANA system

# Data volume: 4x database size
Data Volume: 4 TB, gp3, 16,000 IOPS, 1,000 MB/s

# Log volume: 512 GB minimum
Log Volume: 512 GB, gp3, 10,000 IOPS, 500 MB/s

# Shared volume: 1x database size
Shared Volume: 1 TB, gp3, 3,000 IOPS

# Backup volume: 1-2x database size
Backup Volume: 2 TB, st1 (throughput-optimized)
```

### FSx for SAP

**FSx for NetApp ONTAP**
- NFS and SMB support
- Snapshots and cloning
- Data tiering
- Multi-AZ deployment

```yaml
Resources:
  FSxFileSystem:
    Type: AWS::FSx::FileSystem
    Properties:
      FileSystemType: ONTAP
      StorageCapacity: 1024
      SubnetIds:
        - !Ref PrivateSubnetA
        - !Ref PrivateSubnetB
      OntapConfiguration:
        DeploymentType: MULTI_AZ_1
        ThroughputCapacity: 512
        PreferredSubnetId: !Ref PrivateSubnetA
        RouteTableIds:
          - !Ref PrivateRouteTable
```

**FSx for Lustre**
- High-performance parallel file system
- Up to 1 TB/s throughput
- SAP HANA scale-out
- Data processing workloads

```yaml
Resources:
  LustreFileSystem:
    Type: AWS::FSx::FileSystem
    Properties:
      FileSystemType: LUSTRE
      StorageCapacity: 7200
      SubnetIds:
        - !Ref PrivateSubnetA
      LustreConfiguration:
        DeploymentType: PERSISTENT_2
        PerUnitStorageThroughput: 250
        DataCompressionType: LZ4
```

## High Availability Architecture

### HANA System Replication (HSR)

**Replication Modes**

1. **SYNC (Synchronous)**
- Zero data loss (RPO = 0)
- Higher latency
- Same AZ or nearby AZ

2. **SYNCMEM (Synchronous in-memory)**
- Zero data loss
- Async disk write
- Balanced performance

3. **ASYNC (Asynchronous)**
- Minimal performance impact
- Some data loss possible (RPO > 0)
- Cross-region DR

**AWS Architecture**
```
Production Site (Primary)
├── AZ-A: HANA Primary (r5.24xlarge)
└── Application Servers

DR Site (Secondary)  
├── AZ-B: HANA Secondary (r5.24xlarge)
└── Standby App Servers

Replication: SYNC or SYNCMEM
- Private subnet connectivity
- Enhanced networking
- Placement groups for low latency
```

**Pacemaker Cluster (Linux)**
```bash
# Cluster configuration for automatic failover

# Primary node
crm configure primitive rsc_SAPHana_S4H_HDB00 \
    ocf:suse:SAPHana \
    operations $id="rsc_SAPHana_S4H_HDB00-operations" \
    op start interval="0" timeout="3600" \
    op stop interval="0" timeout="3600" \
    op promote interval="0" timeout="3600" \
    op monitor interval="60" role="Master" timeout="700" \
    op monitor interval="61" role="Slave" timeout="700" \
    params SID="S4H" InstanceNumber="00" PREFER_SITE_TAKEOVER="true"

# Cluster IP resource
crm configure primitive rsc_ip_S4H_HDB00 IPaddr2 \
    params ip="10.0.20.100" \
    op monitor interval="10s" timeout="20s"
```

### Overlay IP for HA

**Elastic Network Interface (ENI) Approach**
```python
import boto3

ec2 = boto3.client('ec2')

# Move secondary IP during failover
def failover_ip(source_eni, target_eni, secondary_ip):
    # Unassign from source
    ec2.unassign_private_ip_addresses(
        NetworkInterfaceId=source_eni,
        PrivateIpAddresses=[secondary_ip]
    )
    
    # Assign to target
    ec2.assign_private_ip_addresses(
        NetworkInterfaceId=target_eni,
        PrivateIpAddresses=[secondary_ip],
        AllowReassignment=True
    )

# Triggered by Pacemaker during failover
failover_ip('eni-primary', 'eni-secondary', '10.0.20.100')
```

## Networking for SAP

### VPC Design

**Multi-Tier SAP VPC**
```
Production VPC: 10.0.0.0/16
├── Public Subnets (Web Dispatcher, Fiori)
│   ├── AZ-A: 10.0.1.0/24
│   └── AZ-B: 10.0.2.0/24
│
├── Application Subnets
│   ├── AZ-A: 10.0.10.0/24
│   └── AZ-B: 10.0.11.0/24
│
├── Database Subnets
│   ├── AZ-A: 10.0.20.0/24
│   └── AZ-B: 10.0.21.0/24
│
└── Management Subnets
    ├── AZ-A: 10.0.30.0/24
    └── AZ-B: 10.0.31.0/24

Non-Production VPC: 10.1.0.0/16
DR VPC (Different Region): 10.2.0.0/16
```

### Connectivity Options

**Hybrid Connectivity**
1. **AWS Direct Connect**
- Dedicated 1/10/100 Gbps connection
- Low latency to on-premises
- SAP GUI access
- Data replication

2. **Site-to-Site VPN**
- Backup connectivity
- Quick setup
- Encrypted tunnel

3. **AWS Transit Gateway**
- Hub for multiple VPCs
- On-premises connectivity
- Centralized routing

**Security Groups for SAP**
```yaml
# HANA Database Security Group
HANASecurityGroup:
  Inbound:
    - Port: 3<instance>13 (HANA SQL)
      Source: Application Security Group
    - Port: 3<instance>15 (HANA Studio)
      Source: Admin Security Group
    - Port: 5<instance>13-5<instance>14 (HSR)
      Source: Secondary HANA Security Group

# Application Server Security Group
AppSecurityGroup:
  Inbound:
    - Port: 32<instance> (SAP Dispatcher)
      Source: User network
    - Port: 33<instance> (Gateway)
      Source: User network
    - Port: 44<instance> (HTTPS)
      Source: Load Balancer

# Web Dispatcher Security Group
WebDispatcherSG:
  Inbound:
    - Port: 443
      Source: 0.0.0.0/0 (or specific IPs)
    - Port: 80
      Source: 0.0.0.0/0
```

## Exam Tips

### Key Concepts
- SAP-certified instances (R, M, C, U families)
- HANA sizing: 1.5x database size for production
- Three-tier architecture: Presentation, Application, Database
- Storage: gp3 for most workloads, io2 for high performance
- HA: HANA System Replication (SYNC, SYNCMEM, ASYNC)

### Architecture Decisions
- **Small HANA (<1 TB)**: r5.24xlarge single instance
- **Large HANA (>4 TB)**: u-instances or scale-out
- **High availability**: Multi-AZ with HSR
- **DR**: Different region with ASYNC replication
- **Backup**: AWS Backint Agent to S3

### Best Practices
- Use placement groups for low latency
- Enhanced networking for better performance
- Automate with CloudFormation/Terraform
- Monitor with CloudWatch
- Regular backups to S3
- Encryption at rest and in transit
- Follow SAP sizing guidelines
- Test failover procedures

### Common Scenarios
- **Migration**: SAP DMO, SAP HANA migration
- **HA setup**: Pacemaker cluster with overlay IP
- **Scaling**: Add application servers horizontally
- **Performance**: Right-size instances, optimize storage
- **Cost**: Reserved Instances, Savings Plans
