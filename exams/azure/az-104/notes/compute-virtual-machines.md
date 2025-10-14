# Azure Compute and Virtual Machines - AZ-104

**[📖 Azure Virtual Machines Documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/)** - Comprehensive guide to Azure VMs

## Azure Virtual Machines

**[📖 VM Sizes and Types](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes)** - Choose the right VM size for your workload

### VM Sizes and Series

**General Purpose** (B, D, DC, Dv2-5):
- Balanced CPU-to-memory ratio
- B-Series: Burstable, cost-effective for low CPU usage
- D-Series: Premium storage support
- Use Cases: Dev/test, small to medium databases, web servers

**Compute Optimized** (F, FX):
- High CPU-to-memory ratio
- F-Series: High-performance processors
- Use Cases: Batch processing, web servers, analytics

**Memory Optimized** (E, M, Mv2):
- High memory-to-CPU ratio
- E-Series: Up to 672 GiB RAM
- M-Series: Up to 4 TB RAM
- Use Cases: In-memory databases, SAP HANA, caching

**Storage Optimized** (L, Ls):
- High disk throughput and IOPS
- Local SSD storage
- Use Cases: Big data, SQL, NoSQL databases

**GPU** (NC, ND, NV):
- Graphics and compute-intensive workloads
- NC-Series: Compute and ML
- ND-Series: Deep learning training
- NV-Series: Visualization and VDI

**High Performance Compute** (H, HB, HC):
- Highest CPU performance
- RDMA networking
- Use Cases: HPC simulations, molecular modeling

### Availability Options

**[📖 Availability Options for Azure VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/availability)** - High availability strategies for virtual machines

**Availability Sets**:
- Logical grouping within a datacenter
- **Fault Domains (FD)**: Separate power and network (max 3)
- **Update Domains (UD)**: Separate update groups (max 20)
- 99.95% SLA
- No cost for the set itself
- Use Case: HA within a single datacenter

**Availability Zones**:
- Physically separate datacenters within region
- Each zone has independent power, cooling, networking
- 99.99% SLA (with 2+ VMs across zones)
- Zone-redundant services available
- Use Case: HA across datacenters

**Virtual Machine Scale Sets (VMSS)**:
- Identical VMs with auto-scaling
- Up to 1,000 VM instances (custom image: 600)
- Automatic or manual scaling
- Load balancer integration
- 99.95% SLA (single placement group)
- Use Case: Auto-scaling applications

### Disk Types

**[📖 Azure Managed Disks Overview](https://learn.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview)** - Storage options for Azure virtual machines

**OS Disk**:
- Required for every VM
- Registered as SATA drive
- C: drive on Windows, /dev/sda on Linux
- Max size: 4 TB

**Data Disks**:
- Managed disks for application data
- Registered as SCSI drives
- Max: 32 data disks (depends on VM size)
- Max size: 32 TB

**Temporary Disk**:
- Local ephemeral storage
- D: drive on Windows, /dev/sdb on Linux
- Lost during maintenance or redeployment
- Use for swap, temp files only
- No cost (included with VM)

### Managed Disk Types

**Ultra Disk**:
- Highest performance
- 4-64 TB size
- Up to 160,000 IOPS, 4,000 MB/s
- Configurable IOPS and throughput
- No caching
- Use Case: SQL Server, SAP HANA, transaction-heavy

**Premium SSD v2**:
- High performance, lower cost than Ultra
- 1 GiB-64 TB
- Up to 80,000 IOPS, 1,200 MB/s
- Configurable performance
- Use Case: Production workloads

**Premium SSD**:
- Production workloads
- 32 GiB-32 TB
- Up to 20,000 IOPS, 900 MB/s
- Consistent low latency
- Use Case: Production VMs, databases

**Standard SSD**:
- Entry-level consistent performance
- 32 GiB-32 TB
- Up to 6,000 IOPS, 750 MB/s
- Use Case: Web servers, dev/test

**Standard HDD**:
- Lowest cost
- 32 GiB-32 TB
- Up to 2,000 IOPS, 500 MB/s
- Use Case: Backups, non-critical workloads

### VM Networking

**Virtual Network Interface Card (NIC)**:
- Each VM has at least one NIC
- Multiple NICs supported (depends on VM size)
- Each NIC attached to one subnet
- Static or dynamic IP assignment

**Accelerated Networking**:
- SR-IOV for high performance
- Lower latency, higher bandwidth
- Supported on most VM sizes
- Enable during creation or on stopped VM

**Public IP Addresses**:
- Basic SKU: Dynamic or static allocation
- Standard SKU: Static allocation, zone-redundant
- Associated with NIC or load balancer

**Network Security Groups (NSGs)**:
- Filter traffic to/from NIC or subnet
- Inbound and outbound rules
- Priority-based rule processing
- Default rules (can't be deleted)

## Azure App Service

**[📖 App Service Documentation](https://learn.microsoft.com/en-us/azure/app-service/)** - Deploy and scale web apps on Azure App Service

### App Service Plans

**[📖 App Service Pricing and Plans](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans)** - Understand App Service plan tiers and features

**Tiers**:
- **Free (F1)**: Shared compute, 1 GB RAM, 1 GB storage
- **Shared (D1)**: Shared compute, custom domains
- **Basic (B1-B3)**: Dedicated compute, manual scale
- **Standard (S1-S3)**: Auto-scale, staging slots
- **Premium (P1v3-P3v3)**: Enhanced performance, more instances
- **PremiumV2 (P1v2-P3v2)**: Previous generation
- **Isolated (I1v2-I3v2)**: Dedicated environment (App Service Environment)

**Features by Tier**:
- **Deployment Slots**: Standard and above
- **Auto-scale**: Standard and above
- **Custom Domains/SSL**: Basic and above
- **VNet Integration**: Premium and above

### Deployment Slots

**[📖 Set up Staging Environments in App Service](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)** - Deploy apps with zero downtime using deployment slots

- Separate instances with own hostnames
- Swap between slots (staging ↔ production)
- Zero downtime deployments
- Rollback capability
- Standard tier: 5 slots, Premium: 20 slots
- Slot-specific settings preserved during swap

### Auto-scaling
- Scale based on metrics (CPU, memory, HTTP queue)
- Scale based on schedule
- Minimum and maximum instance count
- Scale-out and scale-in rules
- Cool-down period between scaling operations

### Authentication and Authorization
- Built-in authentication (Easy Auth)
- Azure AD, Microsoft Account, Facebook, Google, Twitter
- Token store for authenticated sessions
- No code changes required

### Backup and Restore
- Standard tier and above
- Automatic or on-demand backups
- Retention up to 30 days
- Backup to Azure Storage account
- Partial or full restore

## Azure Container Services

**[📖 Azure Container Instances Documentation](https://learn.microsoft.com/en-us/azure/container-instances/)** - Run containers without managing servers

### Azure Container Instances (ACI)
- Serverless containers
- No VM management
- Per-second billing
- Fast startup (seconds)
- Container groups: Multi-container pods
- Windows and Linux containers
- Use Cases: Batch jobs, dev/test, simple applications

### Azure Container Registry (ACR)
- Private Docker registry
- Geo-replication
- **SKUs**: Basic, Standard, Premium
- Integration with AKS, App Service, ACI
- Vulnerability scanning (Premium)
- Content trust and signing

### Azure Kubernetes Service (AKS)

**[📖 Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/)** - Deploy and manage containerized applications with Kubernetes

- Managed Kubernetes
- Free control plane
- Automatic upgrades and patching
- Integrated monitoring and logging
- Azure AD integration
- Virtual nodes (ACI integration)

**Node Pools**:
- Multiple node pools per cluster
- Different VM sizes per pool
- System node pools: AKS system pods
- User node pools: Application pods
- Auto-scaling per node pool

**Networking**:
- **kubenet**: Basic networking, default
- **Azure CNI**: Advanced networking, VNet integration
- Network policies: Calico or Azure Network Policy

## ARM Templates and Automation

**[📖 Azure Resource Manager Templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/)** - Deploy infrastructure as code with ARM templates

### ARM Template Structure
```json
{
  "$schema": "...",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [],
  "outputs": {}
}
```

**Deployment Modes**:
- **Incremental**: Add resources, keep existing
- **Complete**: Delete resources not in template

### Bicep

**[📖 Bicep Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)** - Simplified language for deploying Azure resources

- Domain-specific language for ARM templates
- Simpler syntax than JSON
- Transpiles to ARM templates
- Better type safety

### Azure CLI
```bash
# Create VM
az vm create \
  --resource-group myRG \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Start/Stop VM
az vm start --name myVM --resource-group myRG
az vm stop --name myVM --resource-group myRG

# Deallocate (stop billing)
az vm deallocate --name myVM --resource-group myRG

# Resize VM
az vm resize --resource-group myRG --name myVM --size Standard_D4s_v3
```

### Azure PowerShell
```powershell
# Create VM
New-AzVM `
  -ResourceGroupName "myRG" `
  -Name "myVM" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -SecurityGroupName "myNSG" `
  -PublicIpAddressName "myPIP"

# Start/Stop
Start-AzVM -Name "myVM" -ResourceGroupName "myRG"
Stop-AzVM -Name "myVM" -ResourceGroupName "myRG"
```

## VM Extensions

**[📖 Virtual Machine Extensions](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/overview)** - Automate post-deployment configuration with VM extensions

### Common Extensions
- **Custom Script Extension**: Run scripts during/after deployment
- **Azure Monitor Agent**: Monitoring and diagnostics
- **Azure Disk Encryption**: Encrypt OS and data disks
- **Desired State Configuration (DSC)**: Configuration management
- **Network Watcher**: Network monitoring and diagnostics

### Custom Script Extension
- Execute PowerShell or Bash scripts
- Install software, configure settings
- Windows: PowerShell, Linux: Bash
- Download from Azure Storage or GitHub

## Best Practices

### VM Configuration
1. Use Managed Disks (default)
2. Enable Azure Backup for production VMs
3. Use Availability Zones for critical workloads
4. Implement proper NSG rules
5. Use Azure Monitor for diagnostics
6. Tag resources for cost management
7. Right-size VMs based on workload
8. Use boot diagnostics for troubleshooting

### Security
1. Use Azure AD authentication where possible
2. Enable Just-In-Time VM Access (Security Center)
3. Keep VMs patched with Update Management
4. Use Azure Disk Encryption
5. Implement network segmentation with NSGs
6. Disable public IP addresses when not needed
7. Use Azure Bastion for secure RDP/SSH access
8. Regular security assessments with Security Center

### Cost Optimization
1. Deallocate VMs when not in use (not just stop)
2. Use Reserved Instances for long-running VMs (up to 72% savings)
3. Use Spot VMs for fault-tolerant workloads (up to 90% savings)
4. Right-size VMs regularly
5. Use Azure Hybrid Benefit for Windows/SQL licenses
6. Delete unused disks and snapshots
7. Use Standard HDD for non-critical workloads

## Exam Tips

### Common Scenarios
- **High availability within datacenter**: Availability Set
- **High availability across datacenters**: Availability Zones
- **Auto-scaling web app**: VMSS or App Service
- **Zero downtime deployment**: App Service deployment slots
- **Run containers without management**: Azure Container Instances
- **Managed Kubernetes**: AKS
- **Temporary storage**: Use temp disk (D: or /dev/sdb)
- **Persistent storage**: Managed disks (data disks)
- **Best performance disk**: Ultra Disk or Premium SSD v2
- **Cost-effective disk**: Standard HDD

### Key Differences
- **Stop vs Deallocate**: Stop keeps reservation (still charged), Deallocate releases resources (no compute charge)
- **Availability Set vs Zone**: Set = within datacenter, Zone = across datacenters
- **Managed vs Unmanaged Disks**: Always use Managed (easier management, better reliability)
- **App Service vs VM**: App Service = PaaS (less management), VM = IaaS (more control)
- **ACI vs AKS**: ACI = simple containers, AKS = orchestration at scale

### PowerShell vs CLI
- Both achieve same results
- CLI: Cross-platform, simpler syntax
- PowerShell: Windows-native, object-oriented
- Know basic commands for both
