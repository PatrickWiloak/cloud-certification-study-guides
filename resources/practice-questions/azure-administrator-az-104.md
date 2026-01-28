# Azure Administrator (AZ-104) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Manage Azure identities and governance | 20-25% | 9 |
| Implement and manage storage | 15-20% | 7 |
| Deploy and manage Azure compute resources | 20-25% | 9 |
| Implement and manage virtual networking | 15-20% | 7 |
| Monitor and maintain Azure resources | 10-15% | 8 |

---

## Domain 1: Manage Azure Identities and Governance (Questions 1-9)

### Question 1
**Scenario:** A company needs to ensure that users can only create virtual machines in the West US and East US regions. This policy must apply to all subscriptions under a management group. What should you configure?

A. Azure RBAC role assignment at the management group level
B. Azure Policy with allowed locations at the management group scope
C. Resource locks on all resource groups
D. Subscription-level budget alerts

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Policy with the "Allowed locations" built-in policy definition enforces regional restrictions. Applying it at the management group scope ensures all subscriptions inherit the policy. RBAC (A) controls who can perform actions, not where. Resource locks (C) prevent modification/deletion, not location. Budget alerts (D) are for cost management.

**Key Concept:** [Azure Policy Overview](https://learn.microsoft.com/en-us/azure/governance/policy/overview)
</details>

### Question 2
**Scenario:** A company has Microsoft 365 users who need access to Azure resources. They want to use existing identities without creating separate Azure AD accounts. The company already has Azure AD Connect synchronizing on-premises AD. What additional configuration is needed?

A. No additional configuration - Azure AD Connect already handles this
B. Configure Azure AD B2B collaboration
C. Set up Azure AD B2C
D. Install a second Azure AD Connect server

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Microsoft 365 uses Azure AD (now Entra ID) for identity. If Azure AD Connect is already synchronizing on-premises AD to Azure AD, those identities are available for both M365 and Azure resources. Users authenticate with the same credentials for both. B2B (B) is for external users. B2C (C) is for customer-facing apps. Second AD Connect (D) isn't needed.

**Key Concept:** [Azure AD Connect](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect)
</details>

### Question 3
**Scenario:** A security administrator needs to require MFA for all users accessing the Azure portal, but not for other applications. How should this be configured?

A. Enable Security Defaults
B. Create a Conditional Access policy targeting the Azure portal cloud app requiring MFA
C. Configure per-user MFA settings
D. Use Azure AD Identity Protection

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Conditional Access policies can target specific cloud apps (Microsoft Azure Management) and require MFA as a grant control. This provides granular control for portal access only. Security Defaults (A) applies MFA broadly. Per-user MFA (C) applies to all apps for that user. Identity Protection (D) is for risk-based policies.

**Key Concept:** [Conditional Access](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/overview)
</details>

### Question 4
**Scenario:** A company needs to delegate the ability to create and manage virtual machines to a team without giving them access to modify networking or storage resources. Which built-in RBAC role should be assigned?

A. Contributor
B. Virtual Machine Contributor
C. Owner
D. Virtual Machine Administrator Login

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Virtual Machine Contributor role allows managing VMs and their attached resources but not the underlying virtual network or storage account permissions. Contributor (A) has broader permissions including networking and storage. Owner (C) includes full access plus role assignment. VM Administrator Login (D) is for VM login access, not management.

**Key Concept:** [Azure Built-in Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)
</details>

### Question 5
**Scenario:** An organization wants to prevent accidental deletion of a production resource group containing critical databases. Users should still be able to modify resources within the group. What should you implement?

A. Read-only lock on the resource group
B. Delete lock on the resource group
C. Azure Policy to deny delete operations
D. Remove Contributor role from users

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** A Delete lock prevents deletion of the resource group and its resources while allowing modifications. Read-only lock (A) would prevent modifications. Azure Policy (C) could work but locks are simpler for this scenario. Removing Contributor (D) would prevent modifications.

**Key Concept:** [Azure Resource Locks](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)
</details>

### Question 6
**Scenario:** A company needs to track all changes made to Azure resources for compliance auditing. They need to know who made what change and when. What should they configure?

A. Azure Activity Log with export to Log Analytics
B. Azure Monitor Metrics
C. Resource health alerts
D. Azure Advisor recommendations

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Activity Log captures control plane operations (who did what, when) for Azure resources. Exporting to Log Analytics enables querying historical data beyond the 90-day retention and creating compliance reports. Metrics (B) are for performance data. Resource health (C) tracks availability. Advisor (D) provides recommendations, not audit trails.

**Key Concept:** [Azure Activity Log](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log)
</details>

### Question 7
**Scenario:** A company with multiple subscriptions wants to organize them into a hierarchy for policy and access management. Development subscriptions should inherit different policies than production. What should they create?

A. Multiple Azure AD tenants
B. Management groups with production and development groups
C. Resource groups in each subscription
D. Azure Blueprints

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Management groups enable hierarchical organization of subscriptions. Policies and RBAC applied to a management group are inherited by all subscriptions within it. Creating separate groups for dev and prod allows different policies. Multiple tenants (A) complicate identity management. Resource groups (C) are within subscriptions, not for organizing subscriptions. Blueprints (D) are for environment templates.

**Key Concept:** [Management Groups](https://learn.microsoft.com/en-us/azure/governance/management-groups/overview)
</details>

### Question 8
**Scenario:** A company needs to provision consistent environments for new projects including virtual networks, VMs, policies, and role assignments. They want a template that can be versioned and assigned to subscriptions. What should they use?

A. ARM templates
B. Azure Blueprints
C. Azure Resource Graph
D. Azure Policy initiatives

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Blueprints package ARM templates, policies, role assignments, and resource groups into versioned, assignable artifacts. They maintain the relationship between definition and deployment, enabling updates and compliance tracking. ARM templates (A) deploy resources but don't include policies/RBAC in a managed package. Resource Graph (C) is for querying. Policy initiatives (D) are policy-only.

**Key Concept:** [Azure Blueprints](https://learn.microsoft.com/en-us/azure/governance/blueprints/overview)
</details>

### Question 9
**Scenario:** An administrator needs to grant a consultant temporary elevated access to troubleshoot production issues. The access should automatically expire after 8 hours and require approval.

A. Add the consultant to a privileged group permanently
B. Use Azure AD Privileged Identity Management (PIM) with time-bound eligible assignment
C. Create a temporary user account that will be deleted later
D. Share administrator credentials

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** PIM provides just-in-time privileged access with time limits and approval workflows. Eligible assignments require activation, which can require approval and are time-bound (max 8 hours by default). Permanent group membership (A) doesn't provide time-bound access. Temporary accounts (C) are hard to manage. Sharing credentials (D) violates security best practices.

**Key Concept:** [Azure AD PIM](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-configure)
</details>

---

## Domain 2: Implement and Manage Storage (Questions 10-16)

### Question 10
**Scenario:** A company needs to store files that will be accessed by multiple Azure VMs across different regions. The files are accessed frequently and must support SMB protocol. What storage solution should they use?

A. Azure Blob Storage with hot tier
B. Azure Files with Premium tier and geo-replication
C. Azure Disk Storage
D. Azure Queue Storage

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Files provides fully managed SMB file shares accessible by multiple VMs. Premium tier offers high performance for frequent access. Geo-redundant storage (GRS) provides cross-region availability. Blob Storage (A) doesn't support SMB natively. Disk Storage (C) attaches to single VMs. Queue Storage (D) is for messaging.

**Key Concept:** [Azure Files](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction)
</details>

### Question 11
**Scenario:** An application stores images in Azure Blob Storage. Images are frequently accessed for the first 30 days, rarely accessed for the next 90 days, and then must be retained for 7 years for compliance. How should you configure storage to minimize cost?

A. Use hot tier for all blobs with manual archiving
B. Configure lifecycle management rules: hot → cool after 30 days → archive after 120 days
C. Use premium block blob storage
D. Store all blobs in archive tier from the start

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Lifecycle management automatically transitions blobs between tiers based on age. Hot for frequent access (first 30 days), cool for infrequent (30-120 days), archive for long-term retention (7 years). This optimizes cost automatically. Manual archiving (A) requires ongoing effort. Premium (C) is expensive for archival. Archive from start (D) would require rehydration for initial frequent access.

**Key Concept:** [Blob Lifecycle Management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)
</details>

### Question 12
**Scenario:** A company needs to migrate 50TB of file share data from an on-premises Windows Server to Azure Files. They want to minimize downtime and maintain SMB compatibility. What migration approach should they use?

A. AzCopy to copy files directly to Azure Files
B. Azure Data Box for offline transfer
C. Azure File Sync to synchronize and migrate
D. Azure Migrate for server migration

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Azure File Sync creates a sync relationship between on-premises Windows Server and Azure Files. It enables hybrid access during migration and cloud tiering. After synchronization completes, cutover involves redirecting clients. Minimal downtime. AzCopy (A) requires downtime during copy. Data Box (B) is for offline/large transfers but doesn't provide continuous sync. Azure Migrate (D) is for VMs, not file shares.

**Key Concept:** [Azure File Sync](https://learn.microsoft.com/en-us/azure/storage/file-sync/file-sync-introduction)
</details>

### Question 13
**Scenario:** A storage account contains sensitive data. The security team requires that data is encrypted with a key managed in Azure Key Vault, and the key must be rotated annually. What should you configure?

A. Microsoft-managed keys (default encryption)
B. Customer-managed keys (CMK) with Key Vault and automatic key rotation
C. Client-side encryption
D. Infrastructure encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Customer-managed keys allow you to use your own keys stored in Key Vault. Key Vault supports automatic key rotation policies. Storage accounts can be configured to use CMK for encryption at rest. Microsoft-managed (A) doesn't provide customer control over rotation. Client-side (C) requires application changes. Infrastructure encryption (D) is a secondary encryption layer, not key management.

**Key Concept:** [Customer-Managed Keys](https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-overview)
</details>

### Question 14
**Scenario:** A company needs to provide time-limited access to a specific blob for a partner organization. The partner should only be able to download the file for the next 24 hours without requiring Azure AD credentials.

A. Generate a shared access signature (SAS) with 24-hour expiry and read permissions
B. Make the blob container public
C. Create an Azure AD guest user for the partner
D. Configure anonymous read access

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** SAS provides time-limited, permission-scoped access to blobs without requiring authentication setup. You can specify expiry (24 hours) and permissions (read only). Public access (B, D) would expose the blob to everyone. Guest users (C) require Azure AD setup for external organization.

**Key Concept:** [Shared Access Signatures](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
</details>

### Question 15
**Scenario:** A storage account is configured with network rules to allow access only from a specific virtual network. An Azure Function in a different virtual network needs to access the storage account. What should you configure?

A. Add the Function's virtual network to the storage account firewall
B. Configure a private endpoint for the storage account and VNet peering between the networks
C. Disable the firewall
D. Use managed identity authentication

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Private endpoints provide private IP access to storage accounts over VNet peering or VPN. The Function's VNet can peer with the endpoint's VNet, enabling private access while maintaining firewall restrictions. Adding VNet (A) works for simple cases but private endpoints are preferred for cross-VNet access. Disabling firewall (C) removes security. Managed identity (D) is for authentication, not network access.

**Key Concept:** [Private Endpoints](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
</details>

### Question 16
**Scenario:** A company must ensure that deleted blobs can be recovered for 30 days. They also need to recover from accidental overwrites. What should they enable?

A. Soft delete for blobs with 30-day retention
B. Blob versioning
C. Point-in-time restore
D. All of the above

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** Soft delete protects against deletion (30-day retention for deleted blobs). Versioning maintains previous versions when blobs are overwritten. Point-in-time restore enables restoring the entire container to a previous state. Together they provide comprehensive protection. Each feature addresses different recovery scenarios.

**Key Concept:** [Data Protection Overview](https://learn.microsoft.com/en-us/azure/storage/blobs/data-protection-overview)
</details>

---

## Domain 3: Deploy and Manage Azure Compute Resources (Questions 17-25)

### Question 17
**Scenario:** A company needs to deploy a web application that must survive a data center failure within a region. The application uses IaaS virtual machines. What should you configure?

A. Deploy VMs in a single availability zone
B. Deploy VMs across multiple availability zones with a load balancer
C. Use a Virtual Machine Scale Set with single-instance
D. Deploy to multiple resource groups

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Availability zones are physically separate data centers within a region. Deploying VMs across zones with a zone-redundant load balancer ensures the application survives a single data center failure. Single zone (A) doesn't protect against zone failure. Single-instance VMSS (C) doesn't provide redundancy. Resource groups (D) are logical groupings, not physical separation.

**Key Concept:** [Availability Zones](https://learn.microsoft.com/en-us/azure/reliability/availability-zones-overview)
</details>

### Question 18
**Scenario:** A web application experiences variable traffic with peaks on weekends. You want to automatically scale the number of VM instances between 2 and 10 based on CPU utilization. What should you deploy?

A. Virtual machines with Azure Automation runbooks
B. Virtual Machine Scale Set with autoscale rules
C. Load balancer with health probes
D. Azure Batch

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Virtual Machine Scale Sets (VMSS) provide automatic scaling based on metrics like CPU. Autoscale rules define thresholds (e.g., scale out at 70% CPU, scale in at 30%) and instance limits (min 2, max 10). Automation runbooks (A) require custom logic. Load balancer (C) distributes traffic but doesn't scale. Azure Batch (D) is for batch processing jobs.

**Key Concept:** [VMSS Autoscale](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
</details>

### Question 19
**Scenario:** A development team needs to run containers without managing infrastructure. The containers run batch jobs that complete in a few minutes and don't need to be always running. What service should they use?

A. Azure Kubernetes Service (AKS)
B. Azure Container Instances (ACI)
C. Azure App Service
D. Virtual Machines with Docker

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Container Instances provide serverless container execution without managing infrastructure. They're ideal for batch jobs and short-lived workloads with per-second billing. AKS (A) requires cluster management. App Service (C) is for web apps, not batch jobs. VMs (D) require infrastructure management.

**Key Concept:** [Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-overview)
</details>

### Question 20
**Scenario:** A company is migrating Windows Server VMs from on-premises to Azure. They want to bring their existing Windows Server licenses with Software Assurance to reduce costs. How should they configure the VMs?

A. Select Windows Server license during VM creation
B. Enable Azure Hybrid Benefit on the VMs
C. Use Dev/Test subscription
D. Request a discount from Microsoft

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Hybrid Benefit allows using existing Windows Server licenses with Software Assurance on Azure VMs, reducing the cost to base compute rate. It can be enabled during or after VM creation. Regular license selection (A) includes Windows license cost. Dev/Test (C) is for development workloads. Discounts (D) are enterprise agreements, not license mobility.

**Key Concept:** [Azure Hybrid Benefit](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing)
</details>

### Question 21
**Scenario:** A VM is running low on disk space. The OS disk is 128GB and needs to be increased to 256GB. What steps are required?

A. Resize the disk while the VM is running
B. Stop (deallocate) the VM, resize the OS disk, start the VM, then extend the partition in the OS
C. Create a new disk and migrate data
D. Add a data disk instead

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** OS disk resizing requires the VM to be stopped (deallocated). After Azure resizes the disk, the VM is started and the partition must be extended within the OS to use the new space. Online resize (A) isn't supported for OS disks. Creating a new disk (C) is more complex. Data disk (D) doesn't solve OS disk space issues.

**Key Concept:** [Resize Managed Disks](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/expand-os-disk)
</details>

### Question 22
**Scenario:** A company wants to deploy a web application that automatically scales, supports deployment slots for zero-downtime deployments, and doesn't require managing VMs. The application is a .NET Core app. What should they use?

A. Azure App Service
B. Virtual Machine Scale Set
C. Azure Container Instances
D. Azure Functions

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** App Service is a PaaS offering for web apps with built-in autoscaling, deployment slots for staging/swapping, and no VM management. It supports .NET Core natively. VMSS (B) requires managing VMs. ACI (C) is for containers without deployment slots. Functions (D) are for event-driven workloads, not traditional web apps.

**Key Concept:** [Azure App Service](https://learn.microsoft.com/en-us/azure/app-service/overview)
</details>

### Question 23
**Scenario:** A Windows VM in Azure cannot be accessed via RDP. The VM shows as running in the portal. What should you check first?

A. Boot diagnostics and Network Security Group rules
B. Increase VM size
C. Recreate the VM
D. Check Azure status page for outages

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Boot diagnostics show console screenshots to identify OS-level issues. NSG rules might be blocking RDP (port 3389). These are the most common causes for RDP connectivity issues. Increasing size (B) is unlikely to help. Recreating (C) is drastic without diagnosis. Azure status (D) should be checked but is less likely for single VM issues.

**Key Concept:** [Troubleshoot RDP](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/troubleshoot-rdp-connection)
</details>

### Question 24
**Scenario:** A company needs to run GPU-accelerated machine learning training jobs on Azure. Jobs run for several hours and cost optimization is important. What VM configuration should they use?

A. Standard D-series VMs
B. NC-series VMs with Spot pricing
C. B-series burstable VMs
D. Fsv2-series VMs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** NC-series VMs have NVIDIA GPUs for ML/AI workloads. Spot pricing provides significant discounts (up to 90%) for interruptible workloads. ML training jobs can often checkpoint and resume, making them suitable for Spot. D-series (A) lack GPUs. B-series (C) are for variable workloads, not sustained compute. Fsv2 (D) are compute-optimized but lack GPUs.

**Key Concept:** [GPU VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes-gpu)
</details>

### Question 25
**Scenario:** A company wants to ensure their VMs are backed up with daily backups retained for 30 days and weekly backups retained for 1 year. What should they configure?

A. Azure Backup with a backup policy specifying daily and weekly retention
B. Azure Site Recovery
C. Manual snapshots using Azure Automation
D. Storage account replication

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure Backup policies define backup frequency and retention. You can configure daily backups with 30-day retention and weekly backups with 52-week (1 year) retention. Site Recovery (B) is for disaster recovery replication, not backup. Manual snapshots (C) are complex to manage. Storage replication (D) doesn't provide point-in-time recovery.

**Key Concept:** [Azure Backup Policies](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vm-backup-faq)
</details>

---

## Domain 4: Implement and Manage Virtual Networking (Questions 26-32)

### Question 26
**Scenario:** A company has two virtual networks in the same region that need to communicate. They want low latency and private IP connectivity. What should they configure?

A. VPN Gateway connections
B. VNet peering
C. ExpressRoute
D. Application Gateway

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** VNet peering provides low-latency, high-bandwidth connectivity between VNets using the Azure backbone network. Traffic stays private with no gateway required for same-region peering. VPN Gateway (A) adds latency and is for different connectivity scenarios. ExpressRoute (C) is for on-premises connectivity. Application Gateway (D) is for web traffic load balancing.

**Key Concept:** [VNet Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
</details>

### Question 27
**Scenario:** A web application needs to be accessible from the internet with high availability across zones. The application runs on VMs in a backend pool. HTTPS traffic should be terminated at the load balancer. What should you deploy?

A. Azure Load Balancer (Standard)
B. Azure Application Gateway with WAF
C. Azure Traffic Manager
D. Azure Front Door

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application Gateway is a Layer 7 load balancer that supports HTTPS termination (SSL offload), zone redundancy, and includes Web Application Firewall (WAF) for security. Standard Load Balancer (A) is Layer 4 and doesn't terminate HTTPS. Traffic Manager (C) is DNS-based, not a real load balancer. Front Door (D) is global, which may be overkill for single-region.

**Key Concept:** [Application Gateway](https://learn.microsoft.com/en-us/azure/application-gateway/overview)
</details>

### Question 28
**Scenario:** A company needs to restrict outbound internet access from VMs to only approved URLs. They want to allow access to *.microsoft.com but block all other internet traffic. What should they configure?

A. Network Security Group with FQDN rules
B. Azure Firewall with application rules
C. User Defined Routes to null route internet
D. Service endpoints

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Firewall supports application rules with FQDN filtering (*.microsoft.com). It can allow specific URLs/domains while denying other internet traffic. NSGs (A) don't support FQDN, only IP addresses. Null routing (C) blocks all internet. Service endpoints (D) are for Azure services, not FQDN filtering.

**Key Concept:** [Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/overview)
</details>

### Question 29
**Scenario:** A company's Azure VMs need to access Azure Storage privately without traffic going over the internet. They want to keep the storage account firewall enabled while allowing VNet access. What should they configure?

A. Service Endpoints for storage on the VNet subnet
B. Private Endpoints for the storage account
C. VPN Gateway
D. Disable storage firewall

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Service Endpoints extend VNet identity to Azure services, allowing traffic to route privately over the Azure backbone while storage firewall allows the VNet. This is simpler than Private Endpoints for same-region access. Private Endpoints (B) also work but are more complex for this scenario. VPN (C) is for different networks. Disabling firewall (D) removes security.

**Key Concept:** [Service Endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview)
</details>

### Question 30
**Scenario:** A company has a subnet with VMs that should not be accessible from other subnets in the same VNet. Only specific management ports should be allowed from a management subnet. What should you configure?

A. Network Security Group on the subnet with deny rules and specific allow rules
B. Azure Firewall between subnets
C. Separate VNet for isolation
D. Application Security Groups

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** NSGs can be applied to subnets to control traffic between subnets in the same VNet. Default rules allow VNet-to-VNet traffic, but explicit deny rules can block it with specific allows for management ports. Azure Firewall (B) is overkill for intra-VNet isolation. Separate VNet (C) adds complexity. ASGs (D) are for grouping VMs but work with NSGs, not instead of them.

**Key Concept:** [Network Security Groups](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
</details>

### Question 31
**Scenario:** A company wants to route all traffic from their VNet through a network virtual appliance (NVA) for inspection before going to the internet. What should they configure?

A. Service Endpoints
B. User Defined Routes (UDR) with next hop to the NVA
C. VNet peering
D. BGP routing

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** User Defined Routes override Azure's default routing. A UDR with 0.0.0.0/0 destination and NVA as next hop forces all internet-bound traffic through the NVA. Service Endpoints (A) are for Azure services. VNet peering (C) doesn't affect routing to internet. BGP (D) is for dynamic routing with gateways.

**Key Concept:** [User Defined Routes](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
</details>

### Question 32
**Scenario:** A company needs to connect their on-premises data center to Azure with dedicated, private connectivity (not over the internet). They need consistent low latency and high bandwidth. What should they implement?

A. Site-to-Site VPN
B. ExpressRoute
C. Point-to-Site VPN
D. Azure Virtual WAN with VPN

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ExpressRoute provides dedicated, private connections from on-premises to Azure through connectivity providers. It offers consistent latency, higher bandwidth (up to 100 Gbps), and doesn't traverse the public internet. VPNs (A, C, D) use internet connectivity and have variable latency.

**Key Concept:** [ExpressRoute](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
</details>

---

## Domain 5: Monitor and Maintain Azure Resources (Questions 33-40)

### Question 33
**Scenario:** A company wants to receive email alerts when their VM CPU exceeds 80% for more than 5 minutes. What should they configure?

A. Azure Monitor alert rule with metric condition and email action group
B. Azure Advisor
C. Log Analytics query
D. Azure Service Health

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure Monitor alert rules can trigger on metric conditions (CPU > 80%, aggregation over 5 minutes). Action groups define notification methods (email, SMS, webhook). Advisor (B) provides recommendations, not alerts. Log Analytics (C) is for log queries, not direct metric alerting. Service Health (D) is for Azure service issues.

**Key Concept:** [Azure Monitor Alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
</details>

### Question 34
**Scenario:** A company needs to analyze logs from multiple Azure resources including VMs, App Services, and Azure SQL. They want to write queries across all logs and create dashboards. What should they configure?

A. Send all logs to a Log Analytics workspace and use KQL queries
B. Use Azure Monitor Metrics
C. Enable diagnostic settings without a destination
D. Use Activity Log only

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Log Analytics workspace centralizes logs from multiple sources. Kusto Query Language (KQL) enables cross-resource analysis. Dashboards can be created from query results. Metrics (B) are for numeric time-series, not log analysis. Diagnostic settings (C) need a destination to be useful. Activity Log (D) is limited to control plane operations.

**Key Concept:** [Log Analytics](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
</details>

### Question 35
**Scenario:** A company needs to visualize the performance of their entire Azure environment in a single view including VM health, network topology, and application dependencies. What should they use?

A. Azure Monitor Workbooks with VM Insights
B. Azure Portal resource list
C. Resource Graph Explorer
D. Cost Management dashboard

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure Monitor Workbooks provide customizable dashboards. VM Insights includes performance monitoring and dependency mapping. Network topology visualization shows connections. Together they provide comprehensive environment visibility. Resource list (B) is basic. Resource Graph (C) is for querying resources, not monitoring. Cost Management (D) is for costs.

**Key Concept:** [VM Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/vm/vminsights-overview)
</details>

### Question 36
**Scenario:** A critical application runs on Azure. The team needs to be notified of Azure service issues that might affect their resources before they see impact. What should they configure?

A. Azure Service Health alerts for their subscriptions and regions
B. Resource health checks
C. Third-party monitoring
D. Manual Azure status page monitoring

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure Service Health provides personalized alerts for Azure service issues affecting your specific resources and regions. You can configure alerts to notify via email/SMS/webhook for service issues, planned maintenance, and health advisories. Resource health (B) shows current state, not proactive alerts. Manual monitoring (D) isn't scalable.

**Key Concept:** [Azure Service Health](https://learn.microsoft.com/en-us/azure/service-health/overview)
</details>

### Question 37
**Scenario:** A VM in Azure needs regular maintenance including Windows Updates. The company wants to control when updates are applied to avoid impacting business hours. What should they configure?

A. Azure Update Management with maintenance windows
B. Windows Update automatic updates
C. Manual patching
D. Disable updates

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Update Management (part of Azure Automation) allows scheduling update deployments during maintenance windows. You can define schedules (e.g., weekends at 2 AM) and control which updates to apply. Automatic updates (B) don't respect business hours. Manual patching (C) is labor-intensive. Disabling updates (D) is a security risk.

**Key Concept:** [Update Management](https://learn.microsoft.com/en-us/azure/automation/update-management/overview)
</details>

### Question 38
**Scenario:** A company has deployed resources but wants to ensure they follow Azure best practices for high availability, security, performance, and cost. What tool should they use?

A. Azure Advisor
B. Azure Policy
C. Azure Blueprints
D. Azure Security Center

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure Advisor analyzes deployed resources and provides recommendations across reliability, security, performance, cost, and operational excellence. It's a free service that identifies improvements. Policy (B) enforces standards. Blueprints (C) are for environment templates. Security Center (now Defender for Cloud) (D) focuses on security only.

**Key Concept:** [Azure Advisor](https://learn.microsoft.com/en-us/azure/advisor/advisor-overview)
</details>

### Question 39
**Scenario:** A company needs to diagnose intermittent VM connectivity issues. They need to capture network traffic for analysis. What tool should they use?

A. Network Watcher packet capture
B. NSG flow logs
C. Connection Monitor
D. IP flow verify

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Network Watcher packet capture records network traffic to/from a VM for detailed analysis. Captures can be filtered and downloaded for tools like Wireshark. NSG flow logs (B) show allowed/denied flows but not packet details. Connection Monitor (C) monitors connectivity status. IP flow verify (D) checks if traffic is allowed/denied.

**Key Concept:** [Packet Capture](https://learn.microsoft.com/en-us/azure/network-watcher/packet-capture-overview)
</details>

### Question 40
**Scenario:** A company wants to ensure they are alerted when their Azure spending is projected to exceed their monthly budget. What should they configure?

A. Azure Cost Management budget with forecast alerts
B. Azure Advisor cost recommendations
C. Resource tags for cost tracking
D. Pricing calculator

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Cost Management budgets allow setting spending thresholds with alert conditions based on actual or forecasted costs. Forecast alerts notify when projected spend will exceed budget. Advisor (B) provides optimization recommendations, not budget alerts. Tags (C) help categorize costs but don't alert. Pricing calculator (D) is for estimation before deployment.

**Key Concept:** [Cost Management Budgets](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | B | Identity & Governance |
| 2 | A | Identity & Governance |
| 3 | B | Identity & Governance |
| 4 | B | Identity & Governance |
| 5 | B | Identity & Governance |
| 6 | A | Identity & Governance |
| 7 | B | Identity & Governance |
| 8 | B | Identity & Governance |
| 9 | B | Identity & Governance |
| 10 | B | Storage |
| 11 | B | Storage |
| 12 | C | Storage |
| 13 | B | Storage |
| 14 | A | Storage |
| 15 | B | Storage |
| 16 | D | Storage |
| 17 | B | Compute |
| 18 | B | Compute |
| 19 | B | Compute |
| 20 | B | Compute |
| 21 | B | Compute |
| 22 | A | Compute |
| 23 | A | Compute |
| 24 | B | Compute |
| 25 | A | Compute |
| 26 | B | Networking |
| 27 | B | Networking |
| 28 | B | Networking |
| 29 | A | Networking |
| 30 | A | Networking |
| 31 | B | Networking |
| 32 | B | Networking |
| 33 | A | Monitoring |
| 34 | A | Monitoring |
| 35 | A | Monitoring |
| 36 | A | Monitoring |
| 37 | A | Monitoring |
| 38 | A | Monitoring |
| 39 | A | Monitoring |
| 40 | A | Monitoring |
