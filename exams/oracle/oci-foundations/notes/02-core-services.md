# OCI Foundations - Core Services

## Table of Contents
- [Compute Services](#compute-services)
  - [Compute Instances](#compute-instances)
  - [Instance Shapes](#instance-shapes)
  - [Instance Operations](#instance-operations)
  - [Autoscaling](#autoscaling)
- [Storage Services](#storage-services)
  - [Block Volume](#block-volume)
  - [Object Storage](#object-storage)
  - [File Storage](#file-storage)
  - [Archive Storage](#archive-storage)
- [Networking Services](#networking-services)
  - [Virtual Cloud Network (VCN)](#virtual-cloud-network-vcn)
  - [Subnets](#subnets)
  - [Gateways](#gateways)
  - [Security Lists and NSGs](#security-lists-and-nsgs)
  - [Load Balancers](#load-balancers)
- [Database Services](#database-services)
  - [Autonomous Database](#autonomous-database)
  - [DB Systems](#db-systems)
  - [MySQL and NoSQL](#mysql-and-nosql)
- [Exam Tips](#exam-tips)

---

## Compute Services

### Compute Instances

**Definition**: Virtual machines (VMs) or bare metal servers running in OCI.

#### Instance Types

**1. Virtual Machines (VMs)**:
- Virtualized compute instances
- Share physical hardware (isolated through hypervisor)
- Quick provisioning (seconds)
- Lower cost than bare metal
- Flexible sizing

**2. Bare Metal (BM)**:
- Dedicated physical servers
- No hypervisor overhead
- Direct hardware access
- Better performance for demanding workloads
- Higher cost than VMs

**3. Dedicated Virtual Machine Hosts**:
- Single-tenant physical servers running only your VMs
- Compliance and licensing requirements
- Control over VM placement
- Pay for entire host, not individual VMs

#### Instance Lifecycle

**States**:
```
Provisioning → Running → Stopping → Stopped → Starting → Running
                  ↓
            Terminating → Terminated
```

**Key States**:
- **Provisioning**: Instance being created
- **Running**: Instance active and running (billable)
- **Stopped**: Instance powered off (storage still billable)
- **Terminated**: Instance deleted (not billable, cannot be recovered)

#### Creating Instances (Console)

**Steps**:
1. Navigate to Compute → Instances
2. Click "Create Instance"
3. Configure:
   - Name
   - Compartment
   - Availability Domain
   - Fault Domain
   - Image (OS)
   - Shape (VM size)
   - VCN and Subnet
   - SSH key (for Linux) or password (for Windows)
4. Boot volume size
5. Click "Create"

#### Creating Instances (CLI)

```bash
oci compute instance launch \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1..aaaaaa... \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus":2.0,"memoryInGBs":16.0}' \
  --display-name "web-server-01" \
  --image-id ocid1.image.oc1.phx.aaaaaa... \
  --subnet-id ocid1.subnet.oc1.phx.aaaaaa... \
  --assign-public-ip true \
  --ssh-authorized-keys-file ~/.ssh/id_rsa.pub
```

#### Instance Metadata

**Accessing Instance Metadata** (from within instance):
```bash
# Get instance OCID
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/instance/id

# Get instance region
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/instance/region

# Get instance metadata
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/instance/
```

**Use Cases**:
- Retrieve instance information programmatically
- Configure applications dynamically
- Automation scripts

### Instance Shapes

**Definition**: Template defining CPU, memory, network, and storage resources.

#### Shape Types

**1. Standard Shapes (Balanced)**:
- Balanced CPU-to-memory ratio
- General-purpose workloads
- Examples: VM.Standard.E4.Flex, VM.Standard3.Flex

**2. Dense I/O Shapes**:
- High local NVMe SSD storage
- Big data, databases, analytics
- Examples: VM.DenseIO.E4.Flex, BM.DenseIO.E4.128

**3. GPU Shapes**:
- NVIDIA GPUs
- Machine learning, HPC, graphics
- Examples: VM.GPU.A10.1, BM.GPU4.8

**4. HPC Shapes**:
- High-performance computing
- RDMA cluster networking
- Examples: BM.HPC.E4.128, BM.HPC3.64

**5. Optimized Shapes**:
- Specific workload optimization
- Examples:
  - **Compute-optimized**: Higher CPU ratio (VM.Standard.E4.Flex with more CPUs)
  - **Memory-optimized**: Higher memory ratio (VM.Standard.E4.Flex with more RAM)

#### Flexible Shapes

**Definition**: Shapes allowing custom CPU and memory allocation.

**Benefits**:
- Customize resources to exact needs
- Cost optimization
- Right-sizing workloads

**Examples**:
- **VM.Standard.E4.Flex**: 1-64 OCPUs, 1-1024 GB RAM
- **VM.Optimized3.Flex**: 1-18 OCPUs, optimized for compute
- **BM.Standard.E4.128**: Fixed 128 OCPUs

**Configuring Flex Shapes**:
```bash
oci compute instance launch \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus":4.0,"memoryInGBs":32.0}'
```

**Flex Shape Constraints**:
- Minimum: 1 OCPU, 1 GB RAM per OCPU
- Maximum: 64 GB RAM per OCPU
- Must be in whole number OCPUs

#### Changing Shapes

**When Stopped**:
- Can change to compatible shape
- Instance must be stopped
- No data loss

```bash
oci compute instance update \
  --instance-id ocid1.instance.oc1... \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus":8.0,"memoryInGBs":64.0}'
```

### Instance Operations

#### Stopping and Starting

**Stop Instance**:
```bash
oci compute instance action \
  --instance-id ocid1.instance.oc1... \
  --action STOP
```

**Start Instance**:
```bash
oci compute instance action \
  --instance-id ocid1.instance.oc1... \
  --action START
```

**Reboot Instance**:
```bash
oci compute instance action \
  --instance-id ocid1.instance.oc1... \
  --action RESET
```

#### Terminating Instances

**Terminate with Boot Volume Preservation**:
```bash
oci compute instance terminate \
  --instance-id ocid1.instance.oc1... \
  --preserve-boot-volume true
```

**Terminate and Delete Boot Volume**:
```bash
oci compute instance terminate \
  --instance-id ocid1.instance.oc1... \
  --preserve-boot-volume false
```

#### Console Connection

**Use Cases**:
- Troubleshoot boot issues
- Network configuration problems
- SSH/RDP not working

**Creating Console Connection**:
```bash
oci compute instance-console-connection create \
  --instance-id ocid1.instance.oc1... \
  --ssh-public-key-file ~/.ssh/id_rsa.pub
```

### Autoscaling

**Definition**: Automatically adjust the number of instances based on metrics.

#### Components

**1. Instance Pool**:
- Collection of identical instances
- Created from instance configuration
- Distributed across ADs and FDs

**2. Instance Configuration**:
- Template for creating instances
- Defines shape, image, VCN, etc.
- Reusable template

**3. Autoscaling Configuration**:
- Scaling rules and policies
- Threshold-based or schedule-based
- Min/max instance counts

#### Creating Instance Pool

```bash
# Create instance configuration
oci compute-management instance-configuration create \
  --compartment-id ocid1.compartment.oc1... \
  --instance-details file://instance-details.json

# Create instance pool
oci compute-management instance-pool create \
  --compartment-id ocid1.compartment.oc1... \
  --instance-configuration-id ocid1.instanceconfiguration... \
  --placement-configurations '[{"availabilityDomain":"Uocm:PHX-AD-1","primarySubnetId":"ocid1.subnet..."}]' \
  --size 3
```

#### Autoscaling Policies

**Threshold-based Scaling**:
```json
{
  "scalingPolicy": {
    "policyType": "threshold",
    "rules": [
      {
        "action": "CHANGE_COUNT_BY",
        "value": 1,
        "metric": "CPU_UTILIZATION",
        "threshold": 80,
        "operator": "GT"
      }
    ]
  }
}
```

**Schedule-based Scaling**:
- Scale up during business hours
- Scale down during off-hours
- Predictable workload patterns

---

## Storage Services

### Block Volume

**Definition**: Network-attached block storage for instances (like SAN).

#### Characteristics

- **Persistent**: Survives instance termination
- **Durable**: 99.9% durability
- **Detachable**: Can detach and reattach to different instances
- **Performance**: Up to 420 IOPS/GB, 480 MB/s throughput
- **Size**: 50 GB to 32 TB per volume
- **Encryption**: Always encrypted at rest and in transit

#### Performance Tiers

**1. Lower Cost (Basic)**:
- 2 IOPS/GB
- 240 KB/s per GB throughput
- Cost-effective for low I/O workloads

**2. Balanced**:
- 60 IOPS/GB
- 480 KB/s per GB throughput
- Default tier, most workloads

**3. Higher Performance**:
- 75-90 IOPS/GB
- 600 KB/s per GB throughput
- Performance-intensive workloads

**4. Ultra High Performance**:
- 100-225 IOPS/GB
- 1200 KB/s per GB throughput
- Highest performance tier

#### Creating Block Volumes

**Console**:
1. Navigate to Block Storage → Block Volumes
2. Click "Create Block Volume"
3. Configure:
   - Name
   - Availability Domain
   - Size (50 GB - 32 TB)
   - Performance tier
4. Click "Create"

**CLI**:
```bash
oci bv volume create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "data-volume-01" \
  --size-in-gbs 1024 \
  --vpus-per-gb 20
```

**VPUs (Volume Performance Units)**:
- 0 = Lower Cost (2 IOPS/GB)
- 10 = Balanced (60 IOPS/GB)
- 20 = Higher Performance (75 IOPS/GB)
- 30-40 = Ultra High Performance (90-225 IOPS/GB)

#### Attaching Block Volumes

**Attachment Types**:

**1. iSCSI** (Default):
- Standard protocol
- Requires iSCSI commands on instance
- Highest compatibility

**2. Paravirtualized**:
- Virtualized attachment
- Simpler configuration
- VM instances only

```bash
# Attach volume
oci compute volume-attachment attach \
  --instance-id ocid1.instance.oc1... \
  --volume-id ocid1.volume.oc1... \
  --type iscsi
```

**On Instance (iSCSI)**:
```bash
# Run iSCSI commands from console output
sudo iscsiadm -m node -o new -T <target_iqn> -p <ip_address>:3260
sudo iscsiadm -m node -o update -T <target_iqn> -n node.startup -v automatic
sudo iscsiadm -m node -T <target_iqn> -p <ip_address>:3260 -l

# Create filesystem
sudo mkfs.ext4 /dev/sdb

# Mount volume
sudo mkdir /mnt/data
sudo mount /dev/sdb /mnt/data
```

#### Block Volume Backups

**Types**:

**1. Manual Backups**:
- On-demand snapshots
- User-initiated
- Retained until deleted

**2. Policy-based Backups**:
- Automated backup schedules
- Daily, weekly, monthly, yearly
- Retention periods (days, weeks, months, years)

**Creating Manual Backup**:
```bash
oci bv backup create \
  --volume-id ocid1.volume.oc1... \
  --display-name "data-volume-backup-2024-10-10" \
  --type FULL
```

**Backup Policy**:
```bash
# Assign gold policy (daily, weekly, monthly, yearly backups)
oci bv volume update \
  --volume-id ocid1.volume.oc1... \
  --backup-policy-id ocid1.volumebackuppolicy.oc1..gold
```

**Built-in Backup Policies**:
- **Bronze**: Monthly backups, 12 months retention
- **Silver**: Weekly backups, 4 weeks retention
- **Gold**: Daily, weekly, monthly, yearly backups

#### Block Volume Clones

**Definition**: Point-in-time copy of a volume.

**Characteristics**:
- Fast creation (metadata copy)
- Independent from source
- Can be in different AD
- Cost-effective for testing

```bash
oci bv volume create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --source-volume-id ocid1.volume.oc1... \
  --display-name "data-volume-clone"
```

#### Boot Volumes

**Definition**: Block volumes containing OS for instances.

**Characteristics**:
- Created automatically with instance
- Can be preserved when instance terminated
- Can create custom images from boot volumes
- Bootable

**Boot Volume Backups**:
```bash
oci bv boot-volume-backup create \
  --boot-volume-id ocid1.bootvolume.oc1... \
  --display-name "boot-volume-backup"
```

### Object Storage

**Definition**: Scalable, durable storage for unstructured data (like AWS S3).

#### Characteristics

- **Unlimited capacity**: No size limits
- **Durability**: 99.999999999% (11 nines)
- **Regional**: Data stored in single region (can replicate)
- **Flat namespace**: Buckets contain objects
- **Access methods**: Console, CLI, API, SDK
- **Versioning**: Optional object versioning
- **Encryption**: Always encrypted

#### Storage Tiers

**1. Standard (Hot)**:
- Instant access
- Frequently accessed data
- Higher storage cost, no retrieval cost
- Use cases: Active data, websites, content distribution

**2. Infrequent Access (Cool)**:
- Instant access
- Infrequently accessed data (monthly)
- Lower storage cost, retrieval cost applies
- Minimum 31-day retention
- Use cases: Backups, data archives

**3. Archive (Cold)**:
- Restore before access (1 hour)
- Rarely accessed data (90+ days)
- Lowest storage cost, retrieval cost applies
- Minimum 90-day retention
- Use cases: Long-term archives, compliance

**Auto-Tiering**:
- Automatically moves objects to Infrequent Access after 30 days
- No retrieval fees (unlike manual Infrequent Access)
- Enabled at bucket level

#### Buckets

**Definition**: Containers for objects in Object Storage.

**Creating Buckets**:
```bash
oci os bucket create \
  --compartment-id ocid1.compartment.oc1... \
  --name "my-bucket" \
  --storage-tier Standard \
  --public-access-type NoPublicAccess
```

**Bucket Properties**:
- **Name**: Unique within namespace
- **Storage Tier**: Standard, Archive
- **Auto-Tiering**: Enable/disable
- **Versioning**: Enable/disable
- **Encryption**: Oracle-managed or customer-managed keys
- **Public Access**: Public, private, or object-level

#### Objects

**Uploading Objects**:
```bash
# Single file
oci os object put \
  --bucket-name "my-bucket" \
  --file /path/to/file.txt \
  --name "file.txt"

# Large file (multipart upload)
oci os object put \
  --bucket-name "my-bucket" \
  --file /path/to/largefile.zip \
  --name "largefile.zip" \
  --part-size 128
```

**Downloading Objects**:
```bash
oci os object get \
  --bucket-name "my-bucket" \
  --name "file.txt" \
  --file /path/to/download/file.txt
```

**Listing Objects**:
```bash
oci os object list \
  --bucket-name "my-bucket"
```

**Deleting Objects**:
```bash
oci os object delete \
  --bucket-name "my-bucket" \
  --name "file.txt"
```

#### Object Storage Features

**1. Pre-Authenticated Requests (PAR)**:
- Time-limited URLs for object access
- No OCI credentials needed
- Read or write access

```bash
oci os preauth-request create \
  --bucket-name "my-bucket" \
  --name "par-for-file" \
  --object-name "file.txt" \
  --access-type ObjectRead \
  --time-expires "2024-12-31T23:59:59Z"
```

**2. Lifecycle Policies**:
- Automatically delete or archive objects
- Based on age or other criteria

```json
{
  "rules": [
    {
      "action": "ARCHIVE",
      "isEnabled": true,
      "name": "archive-old-logs",
      "objectNameFilter": {
        "inclusionPrefixes": ["logs/"]
      },
      "timeAmount": 90,
      "timeUnit": "DAYS"
    }
  ]
}
```

**3. Object Versioning**:
- Keep multiple versions of objects
- Protect against accidental deletion
- Restore previous versions

**4. Replication**:
- Cross-region replication
- Automatic or on-demand
- Use cases: DR, compliance, latency reduction

```bash
oci os replication create \
  --bucket-name "source-bucket" \
  --destination-bucket-name "destination-bucket" \
  --destination-region "us-ashburn-1"
```

#### Object Storage Use Cases

- **Backup and Archive**: Long-term data retention
- **Content Distribution**: Static website hosting
- **Big Data**: Data lake storage
- **Application Storage**: User uploads, logs
- **Disaster Recovery**: Cross-region replication

### File Storage

**Definition**: Fully managed NFS file system (like AWS EFS).

#### Characteristics

- **Protocol**: NFSv3
- **Concurrent access**: Multiple instances can mount
- **Scalable**: Automatic scaling (8 EB max)
- **Durable**: 99.99% durability
- **Performance**: Up to 500 MB/s per file system
- **Snapshots**: Point-in-time copies

#### Components

**1. File System**:
- The actual file system
- Created in compartment and AD
- Metered storage (pay for what you use)

**2. Mount Target**:
- NFS server endpoint
- Associated with subnet
- Multiple file systems can use same mount target
- Has private IP address

**3. Export**:
- Maps file system to mount target
- Defines export path
- Controls access

#### Creating File Storage

**Create File System**:
```bash
oci fs file-system create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1...
```

**Create Mount Target**:
```bash
oci fs mount-target create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --subnet-id ocid1.subnet.oc1...
```

**Create Export**:
```bash
oci fs export create \
  --export-set-id ocid1.exportset.oc1... \
  --file-system-id ocid1.filesystem.oc1... \
  --path "/shared-data"
```

#### Mounting File Systems

**On Linux Instance**:
```bash
# Install NFS client
sudo yum install -y nfs-utils

# Create mount point
sudo mkdir /mnt/shared-data

# Mount file system
sudo mount -t nfs -o vers=3 \
  10.0.1.5:/shared-data /mnt/shared-data

# Persist mount (add to /etc/fstab)
echo "10.0.1.5:/shared-data /mnt/shared-data nfs defaults,nofail,nosuid,resvport 0 0" | sudo tee -a /etc/fstab
```

#### File Storage Snapshots

**Creating Snapshots**:
```bash
oci fs snapshot create \
  --file-system-id ocid1.filesystem.oc1... \
  --name "snapshot-2024-10-10"
```

**Restoring from Snapshot**:
- Navigate to snapshot location: `.snapshot/<snapshot-name>/`
- Copy files from snapshot location

#### File Storage Use Cases

- **Shared Application Storage**: Multiple instances accessing same data
- **Content Management**: Shared media files
- **Development Environments**: Shared code repositories
- **Big Data Analytics**: Shared datasets
- **Home Directories**: User home directories

### Archive Storage

**Definition**: Extremely low-cost storage tier within Object Storage.

#### Characteristics

- **Restore Required**: Objects must be restored before access (1 hour)
- **Lowest Cost**: Cheapest storage option
- **Minimum Retention**: 90 days
- **Retrieval Fees**: Pay for data retrieval
- **Durability**: Same as Object Storage (11 nines)

#### Use Cases

- Compliance archives (retain for years)
- Regulatory requirements
- Long-term backups
- Data rarely accessed (once per quarter/year)

#### Archive Operations

**Create Archive Bucket**:
```bash
oci os bucket create \
  --compartment-id ocid1.compartment.oc1... \
  --name "archive-bucket" \
  --storage-tier Archive
```

**Restore Archived Object**:
```bash
oci os object restore \
  --bucket-name "archive-bucket" \
  --name "archived-file.zip"
```

**Check Restore Status**:
```bash
oci os object head \
  --bucket-name "archive-bucket" \
  --name "archived-file.zip"
```

---

## Networking Services

### Virtual Cloud Network (VCN)

**Definition**: Software-defined private network in OCI (like AWS VPC).

#### Characteristics

- **Private**: Isolated from internet by default
- **Regional**: Spans all ADs in region
- **CIDR Block**: IPv4 address range (e.g., 10.0.0.0/16)
- **Subnets**: Subdivisions of VCN
- **Customizable**: Full control over network topology

#### VCN CIDR Blocks

**Valid Ranges** (RFC 1918 private addresses):
- 10.0.0.0/8 (10.0.0.0 - 10.255.255.255)
- 172.16.0.0/12 (172.16.0.0 - 172.31.255.255)
- 192.168.0.0/16 (192.168.0.0 - 192.168.255.255)

**Best Practices**:
- Use /16 for VCN (65,536 addresses)
- Avoid overlapping with on-premises networks
- Plan for growth

**Creating VCN**:
```bash
oci network vcn create \
  --compartment-id ocid1.compartment.oc1... \
  --cidr-block "10.0.0.0/16" \
  --display-name "main-vcn" \
  --dns-label "mainvcn"
```

#### VCN Components

**Default Components** (created with VCN):
- Default route table
- Default security list
- Default DHCP options

**Additional Components**:
- Subnets
- Gateways (Internet, NAT, Service, DRG)
- Route tables
- Security lists
- Network Security Groups (NSGs)
- DHCP options

### Subnets

**Definition**: Subdivision of VCN CIDR block.

#### Subnet Types

**1. Public Subnet**:
- Instances have public IP addresses
- Route to Internet Gateway
- Accessible from internet (if security rules allow)
- Use cases: Web servers, bastion hosts

**2. Private Subnet**:
- Instances have only private IP addresses
- No direct internet access
- Can access internet via NAT Gateway
- Use cases: App servers, databases

#### Regional vs AD-Specific Subnets

**Regional Subnets** (Recommended):
- Span all ADs in region
- Automatic failover across ADs
- Simplified architecture
- **Best Practice**: Use regional subnets

**AD-Specific Subnets**:
- Exist in single AD
- Legacy option
- More complex management

#### Creating Subnets

**Public Subnet**:
```bash
oci network subnet create \
  --vcn-id ocid1.vcn.oc1... \
  --compartment-id ocid1.compartment.oc1... \
  --cidr-block "10.0.1.0/24" \
  --display-name "public-subnet" \
  --route-table-id ocid1.routetable.oc1... \
  --security-list-ids '["ocid1.securitylist.oc1..."]' \
  --prohibit-public-ip-on-vnic false
```

**Private Subnet**:
```bash
oci network subnet create \
  --vcn-id ocid1.vcn.oc1... \
  --compartment-id ocid1.compartment.oc1... \
  --cidr-block "10.0.2.0/24" \
  --display-name "private-subnet" \
  --prohibit-public-ip-on-vnic true
```

### Gateways

#### Internet Gateway (IGW)

**Purpose**: Enables internet access for public subnets.

**Characteristics**:
- Allows inbound and outbound internet traffic
- One IGW per VCN (but can have multiple)
- Stateful (tracks connections)
- No bandwidth limits

**Creating Internet Gateway**:
```bash
oci network internet-gateway create \
  --compartment-id ocid1.compartment.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --is-enabled true \
  --display-name "internet-gateway"
```

**Route Table Entry**:
```
Destination: 0.0.0.0/0
Target: Internet Gateway
```

#### NAT Gateway

**Purpose**: Enables outbound internet access for private subnets.

**Characteristics**:
- Outbound only (no inbound)
- Hides private IP addresses
- Managed service (no instances needed)
- Block allocation for many instances

**Creating NAT Gateway**:
```bash
oci network nat-gateway create \
  --compartment-id ocid1.compartment.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --display-name "nat-gateway"
```

**Route Table Entry**:
```
Destination: 0.0.0.0/0
Target: NAT Gateway
```

**Use Cases**:
- Software updates from internet
- API calls to external services
- Downloading data
- Private instances needing outbound access

#### Service Gateway (SGW)

**Purpose**: Private access to Oracle Services without internet.

**Characteristics**:
- Access Object Storage, Archive Storage without internet
- Traffic doesn't traverse internet
- No internet gateway needed
- No data egress charges
- Regional (access services in same region)

**Creating Service Gateway**:
```bash
oci network service-gateway create \
  --compartment-id ocid1.compartment.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --services '[{"serviceId":"ocid1.service.oc1..."}]' \
  --display-name "service-gateway"
```

**Route Table Entry**:
```
Destination: All PHX Services in Oracle Services Network
Target: Service Gateway
```

**Supported Services**:
- Object Storage
- Archive Storage (via Object Storage service)

#### Dynamic Routing Gateway (DRG)

**Purpose**: Virtual router for private connectivity.

**Use Cases**:
- VCN-to-VCN peering (in same or different regions)
- On-premises to OCI (via FastConnect or VPN)
- Multiple VCN connectivity hub

**Creating DRG**:
```bash
oci network drg create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "main-drg"
```

**Attaching DRG to VCN**:
```bash
oci network drg-attachment create \
  --drg-id ocid1.drg.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --display-name "vcn-attachment"
```

### Security Lists and NSGs

#### Security Lists

**Definition**: Virtual firewall rules for subnets.

**Characteristics**:
- Stateful (return traffic automatically allowed)
- Applied to all VNICs in subnet
- Ingress and egress rules
- Associated with subnet

**Rule Components**:
- **Source/Destination**: CIDR block
- **Protocol**: TCP, UDP, ICMP, all
- **Port Range**: Specific ports or all
- **Action**: Allow (deny is implicit)

**Example Security List**:
```bash
# Create security list
oci network security-list create \
  --compartment-id ocid1.compartment.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --display-name "web-security-list" \
  --ingress-security-rules '[
    {
      "protocol":"6",
      "source":"0.0.0.0/0",
      "tcpOptions":{"destinationPortRange":{"min":80,"max":80}}
    },
    {
      "protocol":"6",
      "source":"0.0.0.0/0",
      "tcpOptions":{"destinationPortRange":{"min":443,"max":443}}
    }
  ]' \
  --egress-security-rules '[
    {
      "protocol":"all",
      "destination":"0.0.0.0/0"
    }
  ]'
```

#### Network Security Groups (NSGs)

**Definition**: Virtual firewall for VNICs (instance network interfaces).

**Characteristics**:
- Applied to specific VNICs (not subnets)
- More flexible than security lists
- Can reference other NSGs
- Can move VNICs between NSGs

**NSG vs Security Lists**:

| Feature | Security Lists | NSGs |
|---------|---------------|------|
| Applied to | Subnet | VNIC |
| Granularity | All instances in subnet | Specific instances |
| Rules | Source/Dest = CIDR | Source/Dest = CIDR or NSG |
| Flexibility | Less flexible | More flexible |
| Best for | Basic security | Complex rules |

**Creating NSG**:
```bash
oci network nsg create \
  --compartment-id ocid1.compartment.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --display-name "web-nsg"
```

**Adding NSG Rules**:
```bash
oci network nsg rules add \
  --nsg-id ocid1.networksecuritygroup.oc1... \
  --security-rules '[
    {
      "direction":"INGRESS",
      "protocol":"6",
      "source":"0.0.0.0/0",
      "tcpOptions":{"destinationPortRange":{"min":443,"max":443}}
    }
  ]'
```

**Best Practice**: Use NSGs for application-specific rules, security lists for subnet-level defaults.

### Load Balancers

**Definition**: Distributes traffic across multiple instances.

#### Load Balancer Types

**1. Load Balancer (Public and Private)**:
- Layer 7 (HTTP/HTTPS) and Layer 4 (TCP)
- Flexible shapes: 10 Mbps, 100 Mbps, 400 Mbps, 8 Gbps
- SSL termination
- Path-based routing
- Regional service (spans ADs)

**2. Network Load Balancer**:
- Layer 4 (TCP/UDP)
- Ultra-low latency
- Source IP preservation
- Millions of requests per second
- Regional service

#### Creating Load Balancer

```bash
oci lb load-balancer create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "web-lb" \
  --shape-name "flexible" \
  --shape-details '{"minimumBandwidthInMbps":10,"maximumBandwidthInMbps":100}' \
  --subnet-ids '["ocid1.subnet.oc1...","ocid1.subnet.oc1..."]' \
  --is-private false
```

#### Load Balancer Components

**1. Backend Set**:
- Pool of backend servers
- Health check configuration
- Session persistence

**2. Backend**:
- Individual instance/server
- IP address and port

**3. Listener**:
- Listens for traffic
- Protocol and port
- Routes to backend set

**4. Health Check**:
- Monitors backend health
- Protocol, interval, timeout

#### Health Checks

```json
{
  "protocol": "HTTP",
  "port": 80,
  "urlPath": "/health",
  "intervalInMillis": 10000,
  "timeoutInMillis": 3000,
  "retries": 3
}
```

---

## Database Services

### Autonomous Database

**Definition**: Fully managed, self-driving database service.

#### Key Features

**1. Self-Driving**:
- Automatic tuning
- Automatic indexing
- Automatic plan management

**2. Self-Securing**:
- Automatic encryption
- Automatic patching
- Automatic backups

**3. Self-Repairing**:
- Automatic failover
- 99.95% SLA
- Zero downtime patching

#### Autonomous Database Types

**1. Autonomous Transaction Processing (ATP)**:
- OLTP workloads
- High concurrency
- Short transactions
- Use cases: Applications, e-commerce, banking

**2. Autonomous Data Warehouse (ADW)**:
- OLAP workloads
- Complex queries
- Analytics and reporting
- Use cases: Business intelligence, data warehouse

**3. Autonomous JSON Database**:
- NoSQL document database
- JSON documents
- Use cases: Modern apps, microservices

#### Workload Types

**Transaction Processing**:
- Mixed workloads
- OLTP
- Short queries

**Data Warehouse**:
- Analytics
- Complex queries
- Parallel processing

#### Creating Autonomous Database

```bash
oci db autonomous-database create \
  --compartment-id ocid1.compartment.oc1... \
  --db-name "proddb" \
  --display-name "Production Database" \
  --admin-password "Welcome123!" \
  --cpu-core-count 2 \
  --data-storage-size-in-tbs 1 \
  --db-workload "OLTP" \
  --is-auto-scaling-enabled true \
  --license-model "BRING_YOUR_OWN_LICENSE"
```

#### Autonomous Database Features

**1. Auto-Scaling**:
- Automatically scale OCPUs (up to 3x)
- Based on workload demand
- No downtime

**2. Automatic Backups**:
- Daily automatic backups
- 60-day retention
- Point-in-time recovery

**3. Connection Types**:
- **TLS**: Encrypted connection (default)
- **mTLS**: Mutual TLS (wallet required)

**4. Access Types**:
- **Private Endpoint**: VCN-only access
- **Public Endpoint**: Internet access (with ACLs)

**5. License Models**:
- **License Included**: Pay for license with service
- **BYOL**: Bring Your Own License (lower cost)

#### Connecting to Autonomous Database

**Download Wallet**:
```bash
oci db autonomous-database generate-wallet \
  --autonomous-database-id ocid1.autonomousdatabase.oc1... \
  --file wallet.zip \
  --password "WalletPassword123"
```

**Connection String** (from wallet):
```
proddb_high = (description= (retry_count=20)...)
proddb_medium = (description= (retry_count=20)...)
proddb_low = (description= (retry_count=20)...)
```

**Service Levels**:
- **HIGH**: Highest resources, lowest concurrency
- **MEDIUM**: Balanced
- **LOW**: Lowest resources, highest concurrency

### DB Systems

**Definition**: Traditional Oracle Database on VMs or bare metal.

#### DB System Types

**1. Virtual Machine (VM)**:
- Cost-effective
- Quick provisioning
- Flexible sizing

**2. Bare Metal**:
- Maximum performance
- Direct hardware access
- No hypervisor overhead

**3. Exadata DB Systems**:
- Exadata infrastructure
- Highest performance
- Enterprise workloads

#### DB System Editions

- **Standard Edition**: Basic features
- **Enterprise Edition**: Advanced features
- **Enterprise Edition - High Performance**: More features
- **Enterprise Edition - Extreme Performance**: All features

#### Creating DB System

```bash
oci db system launch \
  --compartment-id ocid1.compartment.oc1... \
  --availability-domain "Uocm:PHX-AD-1" \
  --subnet-id ocid1.subnet.oc1... \
  --database-edition "ENTERPRISE_EDITION" \
  --shape "VM.Standard2.1" \
  --ssh-authorized-keys-file ~/.ssh/id_rsa.pub \
  --hostname "dbhost01" \
  --db-name "proddb" \
  --admin-password "Welcome123!" \
  --db-version "19.0.0.0" \
  --cpu-core-count 2 \
  --initial-data-storage-size-in-gb 256
```

#### DB System Features

**1. Backup Options**:
- Automatic backups
- Manual backups
- Recovery window (1-60 days)

**2. High Availability**:
- **Data Guard**: Standby database for DR
- **RAC**: Real Application Clusters (multi-node)

**3. Patching**:
- Automated patching
- Manual patching
- Scheduled maintenance

### MySQL and NoSQL

#### MySQL Database Service

**Definition**: Fully managed MySQL database.

**Features**:
- MySQL 8.0
- HeatWave (analytics engine)
- High availability
- Automatic backups
- Point-in-time recovery

**Creating MySQL DB**:
```bash
oci mysql db-system create \
  --compartment-id ocid1.compartment.oc1... \
  --shape-name "MySQL.VM.Standard.E3.1.8GB" \
  --subnet-id ocid1.subnet.oc1... \
  --admin-username "admin" \
  --admin-password "Welcome123!" \
  --data-storage-size-in-gbs 50
```

#### NoSQL Database Cloud Service

**Definition**: Fully managed NoSQL database.

**Features**:
- Key-value and document data models
- ACID transactions
- Low latency
- Automatic scaling
- On-demand and provisioned capacity

**Use Cases**:
- IoT applications
- Mobile apps
- Gaming leaderboards
- Real-time analytics

---

## Exam Tips

### Critical Concepts

**1. Compute**:
- VM vs Bare Metal: VMs share hardware, BM dedicated
- Flexible Shapes: Customize OCPUs and RAM
- Instance metadata: 169.254.169.254
- Autoscaling: Requires instance pool and configuration

**2. Storage**:
- **Block Volume**: Network-attached, persistent, 50GB-32TB
- **Object Storage**: Unlimited, regional, 11 nines durability
- **File Storage**: NFS, shared access, automatic scaling
- **Archive Storage**: Restore required, 90-day minimum

**Storage Tiers**:
- Standard: Hot, instant access
- Infrequent Access: Cool, 31-day min, retrieval fee
- Archive: Cold, 90-day min, 1-hour restore

**3. Networking**:
- **VCN**: Regional, private by default
- **Public Subnet**: Internet Gateway route, public IPs
- **Private Subnet**: No public IPs, NAT Gateway for outbound
- **Regional Subnets**: Preferred over AD-specific

**Gateway Types**:
- **IGW**: Bi-directional internet
- **NAT**: Outbound-only internet
- **SGW**: Private access to Oracle Services
- **DRG**: VCN peering, hybrid connectivity

**Security**:
- **Security Lists**: Apply to subnets
- **NSGs**: Apply to VNICs, more flexible
- Both are stateful

**4. Database**:
- **Autonomous**: Self-driving, self-securing, self-repairing
- **ATP**: Transaction processing (OLTP)
- **ADW**: Data warehouse (OLAP)
- **DB Systems**: Traditional Oracle DB on VM/BM
- **MySQL**: Fully managed MySQL

### Common Exam Questions

**Q: What's the maximum block volume size?**
A: 32 TB

**Q: Object Storage durability?**
A: 99.999999999% (11 nines)

**Q: Which gateway for outbound-only internet?**
A: NAT Gateway

**Q: Which gateway for Object Storage access without internet?**
A: Service Gateway

**Q: Autonomous Database SLA?**
A: 99.95% (or 99.99% with specific configurations)

**Q: File Storage protocol?**
A: NFSv3

**Q: Block Volume encryption?**
A: Always encrypted (at rest and in transit)

**Q: Regional vs AD-specific subnets?**
A: Regional is recommended (automatic failover)

**Q: Can you attach a block volume to multiple instances?**
A: No (except with specific cluster configurations)

**Q: What's required to restore archive objects?**
A: Restore operation (takes 1 hour)

### Quick Reference

**Compute Instance States**:
```
Provisioning → Running → Stopped → Terminated
```

**Storage Comparison**:
| Feature | Block Volume | Object Storage | File Storage |
|---------|-------------|----------------|--------------|
| Protocol | iSCSI/PV | REST API | NFS |
| Max Size | 32 TB/volume | Unlimited | 8 EB |
| Access | Single instance | Multiple clients | Multiple instances |
| Use Case | OS, databases | Archives, backups | Shared files |

**Network Security**:
- Security Lists → Subnet level
- NSGs → VNIC level
- Both stateful

**Load Balancer Types**:
- **Load Balancer**: Layer 7, flexible bandwidth
- **Network Load Balancer**: Layer 4, ultra-low latency

---

## Summary

### Key Takeaways

**Compute**:
- VMs and bare metal instances
- Flexible shapes for customization
- Instance pools for autoscaling
- Console connection for troubleshooting

**Storage**:
- Block Volume: Persistent, attached storage
- Object Storage: Unlimited, regional, durable
- File Storage: NFS shared file systems
- Archive Storage: Lowest cost, restore required

**Networking**:
- VCN: Software-defined network
- Subnets: Public (IGW) and private (NAT)
- Gateways: IGW, NAT, SGW, DRG
- Security: Security Lists and NSGs

**Database**:
- Autonomous Database: Self-managing
- DB Systems: Traditional Oracle DB
- MySQL and NoSQL options

---

**Next Steps**: Review Security, IAM, and Observability
