# OCI Architect Associate - Compute and Storage Solutions

## Table of Contents
- [Compute Solutions](#compute-solutions)
- [Storage Solutions](#storage-solutions)
- [Image Management](#image-management)
- [Automation and Scaling](#automation-and-scaling)
- [Performance Optimization](#performance-optimization)
- [Exam Tips](#exam-tips)

---

## Compute Solutions

### Instance Configuration and Management

#### Shape Selection Strategy

**Workload Analysis**:
```
Workload Type → Recommended Shape
─────────────────────────────────
General Purpose → VM.Standard.E4.Flex
Compute-Intensive → VM.Standard.E4.Flex (high OCPU)
Memory-Intensive → VM.Standard.E4.Flex (high RAM)
GPU Workloads → VM.GPU.A10.1/2
HPC → BM.HPC.E4.128
Dense I/O → BM.DenseIO.E4.128
```

**Flex Shape Optimization**:
```bash
# Right-size for cost optimization
# Example: Web server (2 OCPU, 16 GB RAM)
oci compute instance launch \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{
    "ocpus": 2.0,
    "memoryInGBs": 16.0,
    "baselineOcpuUtilization": "BASELINE_1_8"
  }'
```

**Baseline CPU Utilization** (Flex Shapes):
- **BASELINE_1_8**: 12.5% baseline (burstable)
- **BASELINE_1_2**: 50% baseline
- **BASELINE_1_1**: 100% baseline (full performance)

**Use Cases**:
```
Burstable (1/8):
- Development/Test
- Low-utilization apps
- Cost optimization

Full Baseline (1/1):
- Production workloads
- Consistent performance
- Performance-sensitive apps
```

#### Bare Metal vs Virtual Machines

**Decision Matrix**:
| Requirement | VM | Bare Metal |
|-------------|----|-----------|
| Quick provisioning | ✓ | × |
| Cost-effective | ✓ | × |
| Licensing (per-core) | × | ✓ |
| Maximum performance | × | ✓ |
| Direct hardware access | × | ✓ |
| Flexible sizing | ✓ | × |
| Nested virtualization | × | ✓ |

**Bare Metal Use Cases**:
- Oracle Database (licensing)
- Performance-critical applications
- Containerized workloads (run own hypervisor)
- Compliance requiring physical isolation
- Big data processing

**VM Use Cases**:
- General applications
- Development and testing
- Cost-sensitive workloads
- Frequent scaling needs

### Advanced Instance Features

#### Dedicated Virtual Machine Hosts

**Definition**: Single-tenant physical servers running only your VMs.

**Architecture**:
```
Dedicated Host (Physical Server)
├── Your VM 1
├── Your VM 2
├── Your VM 3
└── Available capacity for more VMs
```

**Benefits**:
- Compliance requirements (physical isolation)
- License requirements (Oracle, Microsoft)
- Control over VM placement
- Meet regulatory requirements

**Creating Dedicated Host**:
```bash
oci compute dedicated-vm-host create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --dedicated-vm-host-shape "DVH.Standard.E4.128" \
  --display-name "prod-dedicated-host"
```

**Launching Instance on Dedicated Host**:
```bash
oci compute instance launch \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --shape "VM.Standard.E4.Flex" \
  --dedicated-vm-host-id ocid1.dedicatedvmhost.oc1... \
  --subnet-id ocid1.subnet.oc1...
```

#### Instance Metadata and Cloud-Init

**Metadata Service** (169.254.169.254):
```bash
# Get instance OCID
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/instance/id

# Get region
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/instance/region

# Get availability domain
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/instance/availabilityDomain

# Get VNIC information
curl -H "Authorization: Bearer Oracle" \
  http://169.254.169.254/opc/v2/vnics/
```

**Cloud-Init for Automation**:
```yaml
#cloud-config
# Install and configure web server

packages:
  - httpd
  - git

runcmd:
  - systemctl enable httpd
  - systemctl start httpd
  - git clone https://github.com/company/webapp.git /var/www/html/
  - echo "Instance ID: $(curl -H 'Authorization: Bearer Oracle' http://169.254.169.254/opc/v2/instance/id)" > /var/www/html/instance.txt

write_files:
  - path: /etc/httpd/conf.d/app.conf
    content: |
      <VirtualHost *:80>
        ServerName myapp.example.com
        DocumentRoot /var/www/html
      </VirtualHost>
```

**Passing Cloud-Init Data**:
```bash
oci compute instance launch \
  --shape "VM.Standard.E4.Flex" \
  --subnet-id ocid1.subnet.oc1... \
  --user-data-file cloud-init.yaml
```

#### Instance Principal Authentication

**Definition**: Allows instances to make OCI API calls without storing credentials.

**Setup**:

**1. Create Dynamic Group**:
```bash
oci iam dynamic-group create \
  --name "web-servers" \
  --description "Web server instances" \
  --matching-rule "instance.compartment.id = 'ocid1.compartment.oc1...'"
```

**2. Create Policy**:
```
Allow dynamic-group web-servers to read buckets in compartment production
Allow dynamic-group web-servers to read objects in compartment production
Allow dynamic-group web-servers to use secret-family in compartment production
```

**3. Use from Instance**:
```python
# Python SDK example
from oci.auth.signers import InstancePrincipalsSecurityTokenSigner
from oci.object_storage import ObjectStorageClient

# Authenticate using instance principal
signer = InstancePrincipalsSecurityTokenSigner()
object_storage = ObjectStorageClient(config={}, signer=signer)

# Now can access Object Storage
namespace = object_storage.get_namespace().data
objects = object_storage.list_objects(namespace, "my-bucket")
```

**Use Cases**:
- Application accessing Object Storage
- Retrieving secrets from Vault
- Calling OCI APIs from application
- Automation scripts on instances

### Boot Volume Management

#### Boot Volume Operations

**Backup and Clone**:
```bash
# Backup boot volume
oci bv boot-volume-backup create \
  --boot-volume-id ocid1.bootvolume.oc1... \
  --display-name "boot-volume-backup-2024-10-10" \
  --type FULL

# Clone boot volume (for testing)
oci bv boot-volume create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --source-boot-volume-id ocid1.bootvolume.oc1... \
  --display-name "boot-volume-clone"
```

**Custom Images from Boot Volumes**:
```bash
# Create custom image from boot volume
oci compute image create \
  --compartment-id ocid1.compartment.oc1... \
  --boot-volume-id ocid1.bootvolume.oc1... \
  --display-name "web-server-golden-image" \
  --launch-mode PARAVIRTUALIZED
```

**Boot Volume Resizing**:
```bash
# Increase boot volume size (instance must be stopped)
oci bv boot-volume update \
  --boot-volume-id ocid1.bootvolume.oc1... \
  --size-in-gbs 100

# Extend filesystem (after starting instance)
sudo /usr/libexec/oci-growfs
```

#### In-Transit Boot Volumes

**Use Case**: Detach boot volume from one instance, attach to another.

```bash
# Stop instance
oci compute instance action \
  --instance-id ocid1.instance.oc1... \
  --action STOP

# Detach boot volume (instance will be in detaching state)
oci compute boot-volume-attachment detach \
  --boot-volume-attachment-id ocid1.bootvolumeattachment.oc1...

# Create new instance with existing boot volume
oci compute instance launch \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --shape "VM.Standard.E4.Flex" \
  --subnet-id ocid1.subnet.oc1... \
  --source-boot-volume-id ocid1.bootvolume.oc1...
```

**Use Cases**:
- Hardware upgrade (change shape)
- Migration to different AD
- Troubleshooting (attach to rescue instance)

---

## Storage Solutions

### Block Volume Deep Dive

#### Performance Tuning

**Volume Performance Units (VPUs)**:
```
VPUs per GB → IOPS/GB → Throughput
──────────────────────────────────
0 (Lower Cost)  → 2    → 240 KB/s
10 (Balanced)   → 60   → 480 KB/s
20 (Higher)     → 75   → 600 KB/s
30 (Ultra High) → 90   → 1200 KB/s
40 (Ultra High) → 225  → 1200 KB/s
```

**Performance Calculation**:
```
Volume: 1000 GB, 20 VPUs/GB
IOPS: 1000 GB × 75 IOPS/GB = 75,000 IOPS
Throughput: 1000 GB × 600 KB/s = 600 MB/s
```

**Tuning Strategy**:
```bash
# Monitor current utilization
oci monitoring metric-data summarize-metrics-data \
  --compartment-id ocid1.compartment.oc1... \
  --namespace "oci_blockstore" \
  --query-text 'VolumeReadOps[1m].sum()'

# Increase performance tier
oci bv volume update \
  --volume-id ocid1.volume.oc1... \
  --vpus-per-gb 20

# Note: Can adjust VPUs dynamically (even while attached)
```

#### Volume Groups

**Definition**: Manage multiple volumes as single unit.

**Benefits**:
- Consistent backups across volumes
- Simplified management
- Application-consistent snapshots

**Use Case - Multi-Volume Database**:
```
Volume Group: "database-volumes"
├── data-volume-1 (1 TB)
├── data-volume-2 (1 TB)
├── redo-volume (500 GB)
└── archive-volume (2 TB)
```

**Creating Volume Group**:
```bash
oci bv volume-group create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --source-details '{
    "volumeIds": [
      "ocid1.volume.oc1...",
      "ocid1.volume.oc1...",
      "ocid1.volume.oc1..."
    ]
  }' \
  --display-name "database-volume-group"
```

**Backing Up Volume Group**:
```bash
# All volumes backed up together (consistent point-in-time)
oci bv volume-group-backup create \
  --volume-group-id ocid1.volumegroup.oc1... \
  --display-name "database-backup-2024-10-10" \
  --type FULL
```

#### Block Volume Replication

**Cross-AD/Cross-Region Replication**:
```
Source Volume (AD-1/Region-1)
    ↓ (Async Replication)
Replica Volume (AD-2/Region-2)
```

**Use Cases**:
- Disaster recovery
- Data migration
- Compliance (data residency)

**Setup**:
```bash
# Enable volume replication
oci bv volume update \
  --volume-id ocid1.volume.oc1.phx... \
  --block-volume-replicas '[{
    "availabilityDomain":"Uocm:IAD-AD-1",
    "displayName":"dr-replica"
  }]'
```

### Object Storage Advanced Features

#### Lifecycle Management

**Policy Example**:
```json
{
  "rules": [
    {
      "name": "archive-old-logs",
      "action": "ARCHIVE",
      "timeAmount": 90,
      "timeUnit": "DAYS",
      "isEnabled": true,
      "objectNameFilter": {
        "inclusionPrefixes": ["logs/"],
        "inclusionPatterns": ["*.log"]
      }
    },
    {
      "name": "delete-temp-files",
      "action": "DELETE",
      "timeAmount": 30,
      "timeUnit": "DAYS",
      "isEnabled": true,
      "objectNameFilter": {
        "inclusionPrefixes": ["temp/"]
      }
    },
    {
      "name": "transition-to-infrequent",
      "action": "INFREQUENT_ACCESS",
      "timeAmount": 60,
      "timeUnit": "DAYS",
      "isEnabled": true,
      "objectNameFilter": {
        "inclusionPrefixes": ["backups/"]
      }
    }
  ]
}
```

**Creating Lifecycle Policy**:
```bash
oci os object-lifecycle-policy put \
  --bucket-name "application-data" \
  --items file://lifecycle-policy.json
```

#### Object Versioning

**Enable Versioning**:
```bash
oci os bucket update \
  --bucket-name "critical-data" \
  --versioning Enabled
```

**Benefits**:
- Protect against accidental deletion
- Maintain history of changes
- Compliance requirements
- Rollback capability

**Listing Versions**:
```bash
oci os object list \
  --bucket-name "critical-data" \
  --fields name,timeCreated,versionId \
  --all
```

**Retrieving Specific Version**:
```bash
oci os object get \
  --bucket-name "critical-data" \
  --name "important-file.txt" \
  --version-id "12345678-abcd-efgh" \
  --file recovered-file.txt
```

#### Multipart Upload

**For Large Files** (>100 MB recommended):
```bash
# Initiate multipart upload
oci os multipart create \
  --bucket-name "large-files" \
  --object-name "bigfile.zip"

# Upload parts (parallel)
oci os multipart upload-part \
  --bucket-name "large-files" \
  --object-name "bigfile.zip" \
  --upload-id "upload-123" \
  --upload-part-num 1 \
  --file part1.bin

# Commit multipart upload
oci os multipart commit \
  --bucket-name "large-files" \
  --object-name "bigfile.zip" \
  --upload-id "upload-123"
```

**Benefits**:
- Parallel uploads (faster)
- Resume capability
- Upload parts independently
- Automatic for OCI CLI (--part-size option)

#### Pre-Authenticated Requests (PAR)

**Create PAR for Sharing**:
```bash
oci os preauth-request create \
  --bucket-name "shared-files" \
  --name "temp-access" \
  --access-type ObjectRead \
  --object-name "report.pdf" \
  --time-expires "2024-12-31T23:59:59Z"

# Returns URL:
# https://objectstorage.us-phoenix-1.oraclecloud.com/p/ABC123.../n/namespace/b/bucket/o/report.pdf
```

**PAR Types**:
- **ObjectRead**: Read single object
- **ObjectWrite**: Write single object
- **ObjectReadWrite**: Read and write single object
- **AnyObjectRead**: Read any object in bucket
- **AnyObjectWrite**: Write any object in bucket

**Use Cases**:
- Share files with external users
- Upload interface for customers
- Temporary access without credentials
- CDN origin

### File Storage Architecture

#### Mount Target Configuration

**Multi-AD Mount Targets**:
```
File System (Regional)
├── Mount Target 1 (Subnet-AD-1: 10.0.1.5)
├── Mount Target 2 (Subnet-AD-2: 10.0.1.6)
└── Mount Target 3 (Subnet-AD-3: 10.0.1.7)
```

**Benefits**:
- High availability
- Local access from each AD
- Load distribution

**Creating Multi-AD Setup**:
```bash
# Create file system (once)
oci fs file-system create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "shared-fs"

# Create mount target in each AD
for AD in 1 2 3; do
  oci fs mount-target create \
    --availability-domain "Uocm:PHX-AD-${AD}" \
    --compartment-id ocid1.compartment.oc1... \
    --subnet-id ocid1.subnet.oc1.ad${AD}... \
    --display-name "mount-target-ad${AD}"

  # Create export for each mount target
  oci fs export create \
    --export-set-id ocid1.exportset.oc1.ad${AD}... \
    --file-system-id ocid1.filesystem.oc1... \
    --path "/shared"
done
```

#### Export Options

**Access Control**:
```bash
oci fs export create \
  --export-set-id ocid1.exportset.oc1... \
  --file-system-id ocid1.filesystem.oc1... \
  --path "/secure-data" \
  --export-options '[{
    "source":"10.0.1.0/24",
    "access":"READ_WRITE",
    "identity-squash":"NONE",
    "require-privileged-source-port":true
  },{
    "source":"10.0.2.0/24",
    "access":"READ_ONLY",
    "identity-squash":"ALL"
  }]'
```

**Export Options Explained**:
- **source**: IP range allowed to mount
- **access**: READ_WRITE or READ_ONLY
- **identity-squash**:
  - NONE: Preserve user/group IDs
  - ROOT: Squash root to nobody
  - ALL: Squash all users
- **require-privileged-source-port**: Require mount from privileged port (<1024)

#### File Storage Snapshots

**Snapshot Strategy**:
```bash
# Create snapshot
oci fs snapshot create \
  --file-system-id ocid1.filesystem.oc1... \
  --name "daily-snapshot-$(date +%Y%m%d)"

# Automated snapshot schedule (using OCI Functions or Events)
# Trigger daily at 2 AM
```

**Accessing Snapshots**:
```bash
# Snapshots accessible from mount point
cd /mnt/shared/.snapshot/
ls -l
# drwxr-xr-x snapshot-20241010
# drwxr-xr-x snapshot-20241009
# drwxr-xr-x snapshot-20241008

# Restore file from snapshot
cp /mnt/shared/.snapshot/snapshot-20241010/important-file.txt /mnt/shared/
```

---

## Image Management

### Custom Image Creation

**From Instance**:
```bash
# Stop instance (recommended)
oci compute instance action \
  --instance-id ocid1.instance.oc1... \
  --action STOP

# Create custom image
oci compute image create \
  --compartment-id ocid1.compartment.oc1... \
  --instance-id ocid1.instance.oc1... \
  --display-name "webserver-v1.0" \
  --launch-mode PARAVIRTUALIZED
```

**From Boot Volume**:
```bash
oci compute image create \
  --compartment-id ocid1.compartment.oc1... \
  --boot-volume-id ocid1.bootvolume.oc1... \
  --display-name "database-template"
```

### Image Portability

**Export Image to Object Storage**:
```bash
oci compute image export to-object \
  --image-id ocid1.image.oc1.phx... \
  --destination-uri "https://objectstorage.us-phoenix-1.oraclecloud.com/n/namespace/b/images/o/webserver-v1.0.oci"
```

**Import Image from Object Storage**:
```bash
oci compute image import from-object \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "webserver-v1.0-imported" \
  --source-uri "https://objectstorage.us-ashburn-1.oraclecloud.com/n/namespace/b/images/o/webserver-v1.0.oci" \
  --launch-mode PARAVIRTUALIZED
```

**Cross-Region Image Copy**:
```bash
# Copy image to another region
oci compute image create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "webserver-v1.0-dr" \
  --source-image-id ocid1.image.oc1.phx... \
  --destination-region "us-ashburn-1"
```

### Bring Your Own Image (BYOI)

**Supported Formats**:
- VMDK (VMware)
- VHD (Hyper-V)
- QCOW2 (KVM)
- OCI (Oracle format)

**Import Process**:
```bash
# 1. Upload image to Object Storage
oci os object put \
  --bucket-name "images" \
  --file custom-linux.qcow2 \
  --name "custom-linux.qcow2"

# 2. Import as OCI image
oci compute image import from-object \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "custom-linux" \
  --source-uri "https://objectstorage.region.oraclecloud.com/n/namespace/b/images/o/custom-linux.qcow2" \
  --launch-mode EMULATED \
  --operating-system "Custom Linux" \
  --operating-system-version "1.0"
```

**Requirements**:
- Cloud-init or OCI utilities installed
- Proper network configuration
- DHCP enabled
- Firewall allows OCI management traffic

---

## Automation and Scaling

### Autoscaling Configuration

#### Metric-Based Scaling

**Configuration**:
```bash
# 1. Create instance configuration
oci compute-management instance-configuration create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "web-config" \
  --instance-details file://instance-details.json

# 2. Create instance pool
oci compute-management instance-pool create \
  --compartment-id ocid1.compartment.oc1... \
  --instance-configuration-id ocid1.instanceconfiguration.oc1... \
  --placement-configurations '[{
    "availabilityDomain":"Uocm:PHX-AD-1",
    "primarySubnetId":"ocid1.subnet.oc1...",
    "faultDomains":["FAULT-DOMAIN-1","FAULT-DOMAIN-2","FAULT-DOMAIN-3"]
  }]' \
  --size 3 \
  --display-name "web-pool"

# 3. Create autoscaling configuration
oci autoscaling configuration create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "web-autoscaling" \
  --auto-scaling-resources '{
    "type":"instancePool",
    "id":"ocid1.instancepool.oc1..."
  }' \
  --policies '[{
    "displayName":"scale-up-cpu",
    "policyType":"threshold",
    "capacity":{"initial":3,"min":3,"max":10},
    "rules":[{
      "action":{"type":"CHANGE_COUNT_BY","value":1},
      "metric":{"metricType":"CPU_UTILIZATION","threshold":{"operator":"GT","value":80}}
    }]
  }]'
```

**Scaling Policies**:
```json
{
  "policies": [
    {
      "displayName": "scale-up-policy",
      "policyType": "threshold",
      "capacity": {
        "initial": 3,
        "min": 3,
        "max": 10
      },
      "rules": [
        {
          "action": {
            "type": "CHANGE_COUNT_BY",
            "value": 2
          },
          "metric": {
            "metricType": "CPU_UTILIZATION",
            "threshold": {
              "operator": "GT",
              "value": 80
            }
          }
        }
      ]
    },
    {
      "displayName": "scale-down-policy",
      "policyType": "threshold",
      "capacity": {
        "initial": 3,
        "min": 3,
        "max": 10
      },
      "rules": [
        {
          "action": {
            "type": "CHANGE_COUNT_BY",
            "value": -1
          },
          "metric": {
            "metricType": "CPU_UTILIZATION",
            "threshold": {
              "operator": "LT",
              "value": 20
            }
          }
        }
      ]
    }
  ]
}
```

#### Schedule-Based Scaling

**Business Hours Scaling**:
```json
{
  "displayName": "business-hours-scaling",
  "policyType": "scheduled",
  "capacity": {
    "initial": 5,
    "min": 5,
    "max": 20
  },
  "executionSchedule": {
    "expression": "0 8 * * 1,2,3,4,5",
    "timezone": "America/New_York",
    "type": "cron"
  },
  "resourceAction": {
    "action": "CHANGE_COUNT_TO",
    "actionValue": 10
  }
}
```

**Off-Hours Scale-Down**:
```json
{
  "displayName": "off-hours-scaling",
  "policyType": "scheduled",
  "executionSchedule": {
    "expression": "0 18 * * 1,2,3,4,5",
    "timezone": "America/New_York",
    "type": "cron"
  },
  "resourceAction": {
    "action": "CHANGE_COUNT_TO",
    "actionValue": 3
  }
}
```

### Load Balancer Integration

**Automatic Backend Management**:
```bash
# Attach instance pool to load balancer backend set
oci lb backend-set create \
  --load-balancer-id ocid1.loadbalancer.oc1... \
  --name "autoscale-backend" \
  --policy "ROUND_ROBIN" \
  --health-checker '{
    "protocol":"HTTP",
    "port":80,
    "urlPath":"/health",
    "intervalInMillis":10000,
    "retries":3
  }' \
  --instance-pool-id ocid1.instancepool.oc1...
```

**Benefits**:
- Automatic backend registration
- Health check integration
- Seamless scaling (no manual intervention)
- Load distribution across new instances

---

## Performance Optimization

### Compute Performance

**CPU Performance**:
```bash
# Monitor CPU utilization
oci monitoring metric-data summarize-metrics-data \
  --compartment-id ocid1.compartment.oc1... \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1m]{instanceId="ocid1.instance..."}.mean()'
```

**Optimization Strategies**:
- Right-size shape (avoid over/under provisioning)
- Use full baseline for consistent performance
- Consider bare metal for maximum performance
- Enable hyper-threading (enabled by default on most shapes)

### Storage Performance

**Block Volume Tuning**:
```bash
# Check current IOPS usage
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_blockstore" \
  --query-text 'VolumeReadOps[1m]{resourceId="ocid1.volume..."}.sum()'

# If hitting limits, increase VPUs
oci bv volume update \
  --volume-id ocid1.volume.oc1... \
  --vpus-per-gb 30
```

**File System Performance**:
- Use multiple mount targets for load distribution
- NFS mount options for performance:
```bash
sudo mount -t nfs -o \
  vers=3,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
  10.0.1.5:/shared /mnt/shared
```

### Network Performance

**Instance Network Bandwidth**:
```
Shape → Network Bandwidth
────────────────────────
VM.Standard.E4.Flex (1 OCPU) → 1 Gbps
VM.Standard.E4.Flex (2 OCPU) → 2 Gbps
VM.Standard.E4.Flex (8+ OCPU) → 8 Gbps+
BM.Standard.E4.128 → 2x 50 Gbps
```

**Optimization**:
- Use larger shapes for more bandwidth
- Enable SR-IOV where supported
- Use FastConnect for on-premises traffic

---

## Exam Tips

### Critical Concepts

**Compute**:
- Flex shapes: Customize OCPU and memory
- Baseline CPU: 1/8 (burstable), 1/1 (full)
- Bare Metal: Best for licensing, performance
- Instance Principal: No credentials needed on instance
- Dedicated Hosts: Single-tenant physical servers

**Storage**:
- Block Volume VPUs: 0, 10, 20, 30, 40 (higher = more IOPS)
- Max block volume: 32 TB
- Object Storage tiers: Standard, Infrequent Access, Archive
- File Storage: NFSv3, automatic scaling
- Lifecycle policies: Automate tiering/deletion

**Images**:
- Custom images: From instance or boot volume
- Export/Import: Move images between regions
- BYOI: Bring custom OS images

**Autoscaling**:
- Metric-based: Scale on CPU, memory
- Schedule-based: Scale by time
- Requires: Instance configuration + Instance pool + Autoscaling config
- Integrates with load balancers automatically

### Common Exam Questions

**Q: How to allow instance to access Object Storage without credentials?**
A: Instance Principal (Dynamic Group + Policy)

**Q: Best storage for shared file access across instances?**
A: File Storage (NFS)

**Q: How to ensure consistent backups across multiple volumes?**
A: Volume Groups

**Q: Maximum block volume IOPS?**
A: Depends on size and VPUs (e.g., 1TB @ 40 VPUs = 225K IOPS)

**Q: How to restore archived object?**
A: Restore operation (takes ~1 hour)

---

## Summary

**Compute**: Shapes, instances, bare metal, dedicated hosts, automation
**Storage**: Block, Object, File - choose based on use case
**Images**: Create, share, import custom images
**Scaling**: Autoscaling with metrics or schedules
**Performance**: Right-size, monitor, optimize based on workload

---

**Next Steps**: Study Database, Security, and Operations
