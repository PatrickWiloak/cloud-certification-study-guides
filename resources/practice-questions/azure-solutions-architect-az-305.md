# Azure Solutions Architect Expert (AZ-305) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Design identity, governance, and monitoring solutions | 25-30% | 11 |
| Design data storage solutions | 20-25% | 9 |
| Design business continuity solutions | 10-15% | 5 |
| Design infrastructure solutions | 25-30% | 15 |

---

## Domain 1: Design Identity, Governance, and Monitoring Solutions (Questions 1-11)

### Question 1
**Scenario:** A multinational company is moving to Azure and needs to design an identity solution. They have 50,000 employees with on-premises Active Directory, multiple third-party SaaS applications requiring SSO, and need to support B2B collaboration with partners. What identity solution should they implement?

A. Azure AD with Azure AD Connect using password hash sync, Enterprise Applications for SSO, and Azure AD B2B
B. Migrate all users to cloud-only Azure AD accounts
C. Federation with AD FS only
D. Azure AD B2C for all users

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** This comprehensive approach addresses all requirements: Azure AD Connect syncs existing AD identities (password hash sync is simplest and most resilient). Enterprise Applications provide SSO to SaaS apps via SAML/OIDC. Azure AD B2B enables partner collaboration with their own identities. Cloud-only accounts (B) require recreating all users. AD FS only (C) doesn't provide SaaS SSO natively. B2C (D) is for customer-facing apps, not employees.

**Key Concept:** [Hybrid Identity](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/whatis-hybrid-identity)
</details>

### Question 2
**Scenario:** A company needs to design a governance strategy for 20 Azure subscriptions across development, test, and production environments. They need consistent policies, cost tracking by department, and centralized security management. What should they implement?

A. Manage each subscription independently
B. Management groups with Azure Policy, cost allocation tags, and Microsoft Defender for Cloud
C. Single subscription for all environments
D. Azure Blueprints only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Management groups organize subscriptions hierarchically for policy inheritance. Azure Policy enforces standards at scale. Cost allocation tags enable department-based cost tracking across subscriptions. Defender for Cloud provides unified security management. Independent management (A) doesn't scale. Single subscription (C) lacks environment isolation. Blueprints (D) are for deployment, not ongoing governance.

**Key Concept:** [Enterprise-Scale Architecture](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/architecture)
</details>

### Question 3
**Scenario:** A healthcare organization must ensure all Azure resources are compliant with HIPAA requirements. They need to detect non-compliant resources and remediate automatically where possible. What solution should they design?

A. Manual compliance audits quarterly
B. Azure Policy with HIPAA regulatory compliance initiative and auto-remediation tasks
C. Azure Security Center free tier
D. Custom PowerShell scripts

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Policy regulatory compliance initiatives include HIPAA/HITRUST controls. Policies continuously evaluate resources and can auto-remediate non-compliance (e.g., enable encryption). Compliance dashboard provides audit evidence. Manual audits (A) are not continuous. Free Security Center (C) lacks regulatory compliance features. Scripts (D) require maintenance.

**Key Concept:** [Regulatory Compliance](https://learn.microsoft.com/en-us/azure/governance/policy/samples/hipaa-hitrust-9-2)
</details>

### Question 4
**Scenario:** A company wants to implement zero trust security for Azure resource access. Administrators should only have privileged access when needed, access should be conditional on device compliance, and all access should be logged. What should they design?

A. Permanent role assignments to administrators
B. Azure AD PIM for just-in-time access, Conditional Access with device compliance, Azure AD sign-in logs to Log Analytics
C. VPN-only access to Azure
D. Shared administrator accounts

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Zero trust requires continuous verification. PIM provides just-in-time privileged access with approval workflows. Conditional Access enforces device compliance before granting access. Sign-in logs to Log Analytics enable security monitoring and investigation. Permanent roles (A) violate least privilege. VPN (C) is network-based, not identity-based. Shared accounts (D) prevent accountability.

**Key Concept:** [Zero Trust](https://learn.microsoft.com/en-us/azure/security/fundamentals/zero-trust)
</details>

### Question 5
**Scenario:** A company needs to design a monitoring solution for 200 VMs, 50 Azure SQL databases, and 30 web applications. They need performance monitoring, alerting, and root cause analysis capabilities. What should they design?

A. Individual monitoring for each resource type
B. Azure Monitor with VM insights, SQL insights, Application Insights, and a unified Log Analytics workspace
C. Third-party monitoring only
D. Basic metrics only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Monitor provides a unified platform. VM insights monitors VM performance and dependencies. SQL insights provides database-specific monitoring. Application Insights tracks web app performance with distributed tracing. Unified Log Analytics workspace enables cross-resource correlation and root cause analysis. Individual monitoring (A) creates silos. Third-party (C) adds complexity. Basic metrics (D) lack depth.

**Key Concept:** [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
</details>

### Question 6
**Scenario:** A company's application processes sensitive financial data. They need to log all data access operations, retain logs for 7 years, and ensure logs cannot be modified or deleted. What logging architecture should they design?

A. Application logging to local files
B. Azure Diagnostic Settings to Log Analytics with immutable storage export and 7-year retention
C. Azure Activity Log only
D. Third-party SIEM

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Diagnostic settings capture data plane operations. Log Analytics enables analysis. Export to Storage Account with immutable storage (WORM) prevents modification/deletion. Lifecycle management maintains 7-year retention. Local files (A) can be modified. Activity Log (C) is control plane only. Third-party SIEM (D) may work but doesn't leverage Azure-native immutability.

**Key Concept:** [Diagnostic Settings](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)
</details>

### Question 7
**Scenario:** A company needs to design a cost management solution for multiple departments using Azure. Each department should see only their costs, central IT should see all costs, and there should be automated alerts when spending exceeds thresholds.

A. Single cost view for everyone
B. Cost Management scopes with RBAC, cost allocation rules by tags, and department-level budgets with alerts
C. Manual cost reports monthly
D. Third-party billing tool

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cost Management scopes (subscriptions, resource groups) with RBAC control who sees what costs. Cost allocation rules distribute shared costs by department tags. Budgets with alerts notify when thresholds are exceeded. Single view (A) lacks access control. Manual reports (C) aren't timely. Third-party (D) adds cost and complexity.

**Key Concept:** [Cost Management](https://learn.microsoft.com/en-us/azure/cost-management-billing/cost-management-billing-overview)
</details>

### Question 8
**Scenario:** A company is designing an external application that allows customers to sign up, sign in, and manage their profiles. They need social identity provider support (Google, Facebook) and custom branding. What identity solution should they use?

A. Azure AD with guest users
B. Azure AD B2C with custom policies and social identity providers
C. On-premises AD
D. Custom authentication system

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure AD B2C is designed for customer identity with support for social providers (Google, Facebook), local accounts, and custom branding/policies. It scales to millions of users. Guest users (A) are for B2B, not customer-facing. On-premises AD (C) doesn't support social providers. Custom authentication (D) is insecure and complex.

**Key Concept:** [Azure AD B2C](https://learn.microsoft.com/en-us/azure/active-directory-b2c/overview)
</details>

### Question 9
**Scenario:** A company needs to ensure that secrets, certificates, and encryption keys are centrally managed, access is audited, and applications can retrieve secrets securely without hardcoding them.

A. Store secrets in application configuration files
B. Azure Key Vault with RBAC, diagnostic logging, and managed identity access for applications
C. Environment variables
D. Azure App Configuration only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Key Vault provides secure storage for secrets, certificates, and keys. RBAC controls access. Diagnostic logging provides audit trails. Managed identities allow applications to authenticate to Key Vault without credentials. Config files (A) and environment variables (C) expose secrets. App Configuration (D) doesn't provide the same security features as Key Vault.

**Key Concept:** [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
</details>

### Question 10
**Scenario:** A company has multiple Azure subscriptions and wants to ensure consistent security posture across all of them. They need vulnerability assessment, security recommendations, and threat protection. What should they implement?

A. Manual security reviews
B. Microsoft Defender for Cloud with the Defender plans enabled and security policies at management group level
C. Azure Policy only
D. Third-party security tools

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Defender for Cloud provides unified security management across subscriptions. Defender plans enable advanced threat protection for various resource types. Security policies at management group level ensure consistent standards. Azure Policy (C) handles compliance but not threat detection. Manual reviews (A) don't scale.

**Key Concept:** [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)
</details>

### Question 11
**Scenario:** A company wants to design a landing zone for new Azure workloads that includes identity, networking, governance, and security foundations following best practices.

A. Create resources ad-hoc as needed
B. Implement Azure Landing Zones using Cloud Adoption Framework with management groups, policies, and hub-spoke networking
C. Single subscription with all resources
D. Separate Azure AD tenant per workload

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Landing Zones (CAF) provide a proven architecture with management group hierarchy, standardized policies, hub-spoke networking for connectivity, and security foundations. This ensures consistency and governance at scale. Ad-hoc (A) leads to inconsistency. Single subscription (C) lacks segregation. Separate tenants (D) complicate identity management.

**Key Concept:** [Azure Landing Zones](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)
</details>

---

## Domain 2: Design Data Storage Solutions (Questions 12-20)

### Question 12
**Scenario:** A company needs to migrate a 20TB on-premises SQL Server database to Azure. They require high availability, automatic failover, and the ability to read from secondary replicas for reporting. They want minimal administration.

A. SQL Server on Azure VMs with Always On Availability Groups
B. Azure SQL Database Business Critical tier with read scale-out
C. Azure SQL Managed Instance Business Critical tier with auto-failover groups
D. Azure Database for PostgreSQL

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** SQL Managed Instance provides near 100% compatibility with on-premises SQL Server, making migration easier. Business Critical tier includes built-in HA with synchronous replicas. Auto-failover groups provide cross-region failover. Read replicas handle reporting queries. SQL Database (B) has more compatibility limitations. VMs (A) require more administration. PostgreSQL (D) requires migration to different engine.

**Key Concept:** [Azure SQL Managed Instance](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview)
</details>

### Question 13
**Scenario:** An e-commerce company needs a database for their shopping cart that requires single-digit millisecond latency, automatic scaling, and multi-region writes for global availability. What should they design?

A. Azure SQL Database
B. Azure Cosmos DB with multi-region writes
C. Azure Cache for Redis
D. Azure Table Storage

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cosmos DB provides guaranteed single-digit millisecond latency at any scale. Multi-region writes (multi-master) enable writes in any region with automatic conflict resolution. Automatic scaling handles traffic spikes. Azure SQL (A) has higher latency and doesn't support multi-region writes. Redis (C) is caching, not primary storage. Table Storage (D) lacks global distribution features.

**Key Concept:** [Azure Cosmos DB](https://learn.microsoft.com/en-us/azure/cosmos-db/introduction)
</details>

### Question 14
**Scenario:** A company stores petabytes of IoT telemetry data for analytics. They need cost-effective storage, the ability to query data with SQL, and integration with Azure Synapse Analytics. What storage solution should they design?

A. Azure Blob Storage with Azure Data Lake Storage Gen2 capabilities
B. Azure SQL Database
C. Azure Files
D. Premium SSD managed disks

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** ADLS Gen2 (Blob Storage with hierarchical namespace) provides cost-effective storage for big data. It integrates natively with Synapse Analytics for SQL queries over data lake. Tiered storage reduces costs for older data. Azure SQL (B) is expensive for petabyte scale. Azure Files (C) isn't optimized for analytics. Managed disks (D) aren't for data lakes.

**Key Concept:** [Azure Data Lake Storage Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
</details>

### Question 15
**Scenario:** A media company stores video files that are accessed frequently when new but rarely accessed after 90 days. They need to optimize storage costs while maintaining quick access for recent files. What should they design?

A. Premium blob storage for all files
B. Hot tier for initial storage with lifecycle management to cool after 30 days and archive after 90 days
C. Archive tier for all files
D. Azure Files Premium

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Hot tier provides optimal performance for frequently accessed new files. Lifecycle management automatically transitions to cool (lower cost, millisecond access) after 30 days and archive (lowest cost, hours to rehydrate) after 90 days. Premium (A, D) is expensive. Archive for all (C) would have slow access for new files.

**Key Concept:** [Blob Storage Tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)
</details>

### Question 16
**Scenario:** A company needs to design a data warehouse solution for business intelligence. They have 50TB of data from multiple sources requiring complex joins and aggregations. Analysts use Power BI and need fast query performance.

A. Azure SQL Database
B. Azure Synapse Analytics dedicated SQL pool
C. Azure Cosmos DB
D. Azure Database for MySQL

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Synapse Analytics dedicated SQL pool is designed for data warehousing with MPP (massively parallel processing) architecture for complex queries. It handles 50TB efficiently with columnar storage and query optimization. Native Power BI integration provides fast visualizations. Azure SQL (A) isn't designed for data warehouse workloads. Cosmos DB (C) is for operational data. MySQL (D) isn't MPP.

**Key Concept:** [Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is)
</details>

### Question 17
**Scenario:** A company needs to ensure data stored in Azure is encrypted with keys they control. They need to rotate keys annually and have audit logs of all key usage. What should they design?

A. Microsoft-managed keys (default)
B. Customer-managed keys in Azure Key Vault with automatic rotation and diagnostic logging
C. Client-side encryption only
D. No encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Customer-managed keys (CMK) stored in Key Vault provide customer control. Key Vault supports automatic key rotation policies. Diagnostic logs capture all key operations for auditing. Storage services (Blob, SQL, etc.) can be configured to use CMKs. Microsoft-managed (A) doesn't provide customer control. Client-side (C) is additional, not replacement. No encryption (D) violates security requirements.

**Key Concept:** [Customer-Managed Keys](https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-overview)
</details>

### Question 18
**Scenario:** A company needs to replicate critical data to a secondary region for disaster recovery. The data must be readable in the secondary region during normal operations for reporting queries. What replication should they configure?

A. Locally redundant storage (LRS)
B. Zone-redundant storage (ZRS)
C. Geo-redundant storage with read access (RA-GRS)
D. Geo-zone-redundant storage (GZRS)

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** RA-GRS replicates data to a secondary region and provides read access to the secondary, enabling reporting queries during normal operations. During failover, the secondary becomes read-write. LRS (A) and ZRS (B) don't provide cross-region replication. GZRS (D) provides zone + geo redundancy but requires RA-GZRS for read access to secondary.

**Key Concept:** [Storage Redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)
</details>

### Question 19
**Scenario:** A company needs to design secure data access for an application. The application runs on Azure VMs and needs to access Azure SQL Database and Blob Storage without storing credentials in code or configuration.

A. Store connection strings in App Settings
B. Use Managed Identity for the VMs with Azure RBAC for Blob and Azure AD authentication for SQL
C. Store credentials in Key Vault and retrieve at runtime
D. Use service account with password

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Managed Identity provides an Azure AD identity for VMs without credential management. Azure RBAC assigns blob access permissions to the identity. Azure SQL supports Azure AD authentication with managed identity. No credentials to store or rotate. Key Vault (C) still requires credentials to access Key Vault initially. App Settings (A) and service accounts (D) expose credentials.

**Key Concept:** [Managed Identity](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)
</details>

### Question 20
**Scenario:** A company needs to build a real-time data pipeline that ingests millions of events per second from IoT devices, processes them, and stores results for analytics. What architecture should they design?

A. Direct database inserts
B. Azure Event Hubs for ingestion, Azure Stream Analytics for processing, Azure Synapse for storage
C. Azure Queue Storage
D. Azure Service Bus

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Event Hubs handles millions of events per second with partitioning. Stream Analytics provides real-time processing with SQL-like queries. Synapse stores processed data for analytics. This is the standard Azure streaming architecture. Direct inserts (A) can't handle the scale. Queue Storage (C) and Service Bus (D) aren't designed for high-throughput streaming ingestion.

**Key Concept:** [Azure Streaming Architecture](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/data/stream-processing-stream-analytics)
</details>

---

## Domain 3: Design Business Continuity Solutions (Questions 21-25)

### Question 21
**Scenario:** A company requires their application to have an RTO of 4 hours and RPO of 1 hour. The application runs on Azure VMs with Azure SQL Database. What disaster recovery solution should they design?

A. Daily backups only
B. Azure Site Recovery for VMs with 1-hour replication, Azure SQL geo-replication
C. Multi-region active-active deployment
D. No DR needed

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Site Recovery replicates VMs to a secondary region continuously (meeting 1-hour RPO). Recovery can be initiated within the 4-hour RTO. Azure SQL geo-replication provides continuous async replication. Daily backups (A) have 24-hour RPO. Active-active (C) exceeds requirements and costs more. DR is needed (D) to meet objectives.

**Key Concept:** [Azure Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
</details>

### Question 22
**Scenario:** A company wants to ensure their Azure VMs can be recovered to a specific point in time within the last 30 days. They need both full VM recovery and file-level recovery. What should they design?

A. Azure Backup with VM backup and 30-day retention
B. Storage account snapshots only
C. Azure Site Recovery
D. Manual scripts to copy VHDs

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Azure Backup provides application-consistent backups with 30-day (or longer) retention. It supports both full VM recovery and file-level recovery from recovery points. Managed by backup policies. Snapshots (B) don't provide managed retention or file-level recovery. Site Recovery (C) is for DR replication, not point-in-time backup. Manual scripts (D) are error-prone.

**Key Concept:** [Azure Backup](https://learn.microsoft.com/en-us/azure/backup/backup-azure-vms-introduction)
</details>

### Question 23
**Scenario:** A company needs their web application to remain available even if an entire Azure region fails. They need automatic failover with minimal data loss. What architecture should they design?

A. Single region with Availability Zones
B. Multi-region active-passive with Azure Front Door, geo-replicated databases, and Azure Site Recovery
C. Multiple VMs in single region
D. On-premises DR site

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Multi-region architecture with geo-replicated databases provides near-zero RPO. Azure Front Door provides global load balancing with automatic failover based on health probes. Azure Site Recovery replicates VMs to secondary region. This ensures availability during regional failures. Single region (A, C) doesn't survive regional failures. On-premises (D) adds complexity.

**Key Concept:** [Multi-region Architecture](https://learn.microsoft.com/en-us/azure/architecture/guide/design-principles/redundancy)
</details>

### Question 24
**Scenario:** A company needs to test their disaster recovery plan without impacting production. They want to validate that applications work correctly in the DR region. What should they use?

A. Failover production to DR region
B. Azure Site Recovery test failover to isolated network
C. Manual documentation review
D. Third-party DR testing tool

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Site Recovery test failover creates VMs in the DR region connected to an isolated network. This allows testing application functionality without affecting production or consuming replication bandwidth. After testing, cleanup removes test resources. Production failover (A) impacts users. Documentation review (C) doesn't validate actual recovery. Third-party (D) adds complexity.

**Key Concept:** [Test Failover](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-test-failover-to-azure)
</details>

### Question 25
**Scenario:** A company's database requires zero data loss (RPO = 0) during failover. They use Azure SQL Database. What should they configure?

A. Geo-replication with asynchronous replication
B. Auto-failover groups with synchronous replication in Business Critical tier
C. LRS backup
D. Manual backup and restore

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Auto-failover groups with Business Critical tier use synchronous replication within the primary region, providing RPO = 0 for zone failures. For regional failures, async geo-replication has near-zero RPO but not guaranteed zero. Note: True RPO = 0 across regions isn't possible due to physics. Business Critical provides best possible protection. Async (A) has potential data loss. Backups (C, D) have longer RPO.

**Key Concept:** [Auto-failover Groups](https://learn.microsoft.com/en-us/azure/azure-sql/database/auto-failover-group-overview)
</details>

---

## Domain 4: Design Infrastructure Solutions (Questions 26-40)

### Question 26
**Scenario:** A company needs to design a network architecture for 50 application environments that require isolation from each other but shared connectivity to on-premises. They want centralized security controls. What should they design?

A. 50 separate VNets with individual VPN connections
B. Hub-spoke topology with Azure Firewall in hub, spoke VNets for applications, VNet peering, and VPN/ExpressRoute in hub
C. Single flat VNet with NSGs
D. Azure Virtual WAN with secured hubs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Hub-spoke provides isolation (each spoke is separate) with shared services (firewall, VPN) in hub. VNet peering connects spokes to hub. Azure Firewall in hub provides centralized security. Single VPN/ExpressRoute in hub reduces cost. Separate VNets with VPNs (A) is expensive. Single VNet (C) lacks isolation. Virtual WAN (D) is also valid but hub-spoke is more traditional for this scale.

**Key Concept:** [Hub-Spoke Topology](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
</details>

### Question 27
**Scenario:** A web application needs to handle 1 million requests per second with sub-second response times. The application is stateless and runs on containers. What compute platform should they design?

A. Virtual Machines with manual scaling
B. Azure Kubernetes Service (AKS) with horizontal pod autoscaling and cluster autoscaler
C. Azure Container Instances
D. Azure Functions Consumption plan

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AKS provides container orchestration with horizontal pod autoscaling (scale pods based on metrics) and cluster autoscaler (scale nodes). It handles high throughput with low latency when properly configured. VMs (A) require manual container management. ACI (C) doesn't scale as well for consistent high load. Functions Consumption (D) has cold start issues.

**Key Concept:** [AKS Autoscaling](https://learn.microsoft.com/en-us/azure/aks/cluster-autoscaler)
</details>

### Question 28
**Scenario:** A company needs to run Windows-based batch processing jobs that run for 4-6 hours. Jobs are triggered weekly and don't need to run continuously. They want to minimize cost.

A. Always-on VMs
B. Azure Batch with low-priority VMs
C. Azure Functions
D. Azure App Service

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Batch manages large-scale batch workloads with job scheduling. Low-priority VMs (Spot) provide significant cost savings for interruptible workloads. Jobs that can checkpoint and resume are ideal. Functions (C) has 10-minute max execution (60 min for premium). Always-on VMs (A) waste money when idle. App Service (D) is for web apps.

**Key Concept:** [Azure Batch](https://learn.microsoft.com/en-us/azure/batch/batch-technical-overview)
</details>

### Question 29
**Scenario:** A company's API receives 1000 requests per second during peak hours but only 10 requests per second at night. They want to minimize cost while maintaining performance. What compute solution should they design?

A. VM Scale Set with fixed capacity for peak
B. Azure App Service with autoscaling
C. Azure API Management with Azure Functions Premium plan
D. Always-on AKS cluster

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** API Management provides API gateway capabilities. Functions Premium plan scales quickly (pre-warmed instances) based on demand and scales to zero during low traffic, minimizing cost. APIM with consumption tier is even more cost-effective for variable load. Fixed capacity (A) wastes money at night. App Service (B) works but Functions is more cost-effective for variable loads.

**Key Concept:** [Functions Premium Plan](https://learn.microsoft.com/en-us/azure/azure-functions/functions-premium-plan)
</details>

### Question 30
**Scenario:** A company needs to deploy a hybrid application with components on-premises and in Azure. The on-premises component must initiate connections to Azure (on-premises firewall blocks inbound). What connectivity should they design?

A. Site-to-Site VPN
B. Azure Relay Hybrid Connections
C. ExpressRoute
D. Public endpoints with IP whitelisting

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Relay Hybrid Connections uses outbound connections from on-premises (through firewalls) to establish bidirectional communication. No inbound firewall rules needed. Ideal for scenarios where on-premises initiates but Azure needs to send data back. S2S VPN (A) and ExpressRoute (C) require firewall configuration. Public endpoints (D) expose services.

**Key Concept:** [Azure Relay](https://learn.microsoft.com/en-us/azure/azure-relay/relay-hybrid-connections-protocol)
</details>

### Question 31
**Scenario:** A company needs a global content delivery network for their media streaming application. Content should be cached at edge locations worldwide with custom domain and HTTPS support. What should they design?

A. Azure Storage with public access
B. Azure CDN with custom domain and managed certificates
C. Azure Front Door
D. Traffic Manager

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure CDN caches content at edge locations globally for low-latency delivery. Custom domains with managed SSL certificates provide branding and security. Multiple CDN providers available (Microsoft, Verizon, Akamai). Storage public access (A) doesn't provide edge caching. Front Door (C) is more for dynamic acceleration. Traffic Manager (D) is DNS-based routing, not caching.

**Key Concept:** [Azure CDN](https://learn.microsoft.com/en-us/azure/cdn/cdn-overview)
</details>

### Question 32
**Scenario:** A company needs to design a solution for running SAP HANA with 12TB of memory. High availability is required within the region. What should they design?

A. Standard D-series VMs
B. M-series VMs with Azure Availability Zones and SAP HANA system replication
C. Azure SQL Database
D. Cosmos DB

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** M-series VMs are certified for SAP HANA with up to 12TB memory (M208ms_v2). Availability Zones provide datacenter-level redundancy. SAP HANA system replication provides database-level HA. Azure Premium Storage provides required performance. D-series (A) doesn't have enough memory. Azure SQL (C) and Cosmos DB (D) aren't SAP HANA.

**Key Concept:** [SAP on Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/workloads/sap/hana-vm-operations)
</details>

### Question 33
**Scenario:** A company wants to run containers without managing infrastructure. They need integration with Azure services, support for both HTTP and event-driven workloads, and scale-to-zero capability.

A. Azure Kubernetes Service
B. Azure Container Apps
C. Virtual Machines with Docker
D. Azure Container Instances

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Container Apps is serverless containers with managed infrastructure. It supports HTTP ingress, event-driven scaling (KEDA), scale-to-zero, and integrates with Azure services. Built on Kubernetes but abstracted. AKS (A) requires cluster management. VMs (C) require full infrastructure management. ACI (D) is simpler but lacks event-driven features.

**Key Concept:** [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/overview)
</details>

### Question 34
**Scenario:** A company needs to migrate 500 VMs from VMware to Azure within 6 months. They need minimal changes to the applications. What migration approach should they design?

A. Rebuild all applications cloud-native
B. Azure Migrate with agentless replication for assessment and migration
C. Manual VHD export and import
D. Azure Site Recovery only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Migrate provides discovery, assessment, and agentless replication for VMware VMs. It handles dependency mapping to identify migration groups. Replication enables minimal downtime cutover. Rebuilding (A) is time-consuming for 500 VMs. Manual export (C) doesn't scale. Site Recovery (D) is part of Azure Migrate's migration tools.

**Key Concept:** [Azure Migrate](https://learn.microsoft.com/en-us/azure/migrate/migrate-services-overview)
</details>

### Question 35
**Scenario:** A company needs to secure their web application against common attacks like SQL injection, XSS, and DDoS. What should they design?

A. NSG rules only
B. Azure Application Gateway with Web Application Firewall (WAF) and Azure DDoS Protection
C. On-premises firewall
D. Azure Firewall only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application Gateway WAF protects against OWASP top 10 including SQL injection and XSS. Azure DDoS Protection Standard provides DDoS mitigation. Together they provide comprehensive web application security. NSGs (A) are Layer 4, can't inspect HTTP. On-premises firewall (C) doesn't help cloud apps. Azure Firewall (D) is Layer 4, not WAF.

**Key Concept:** [Azure WAF](https://learn.microsoft.com/en-us/azure/web-application-firewall/overview)
</details>

### Question 36
**Scenario:** A company wants to implement GitOps for their AKS cluster. Configuration changes should be pulled from a Git repository and automatically applied to the cluster.

A. Manual kubectl apply commands
B. Azure Arc-enabled Kubernetes with Flux extension for GitOps
C. Azure DevOps pipelines only
D. Helm charts stored locally

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Flux (GitOps operator) continuously reconciles cluster state with Git repository. Azure Arc integration provides management and monitoring. Changes to Git automatically propagate to the cluster. True GitOps pattern. Manual commands (A) aren't automated. Pipelines (C) are push-based, not GitOps. Local Helm (D) isn't GitOps.

**Key Concept:** [GitOps with Flux](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/conceptual-gitops-flux2)
</details>

### Question 37
**Scenario:** A company needs private connectivity between Azure and their office network. They require consistent performance, low latency, and the connection should not traverse the public internet. Bandwidth need is 200 Mbps.

A. Site-to-Site VPN
B. ExpressRoute with 200 Mbps circuit
C. Point-to-Site VPN
D. Azure Bastion

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ExpressRoute provides private, dedicated connectivity not over public internet. It offers consistent latency and performance. 200 Mbps circuits are available. S2S VPN (A) uses internet. P2S VPN (C) is for individual users. Bastion (D) is for VM access.

**Key Concept:** [ExpressRoute](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
</details>

### Question 38
**Scenario:** A company needs to deploy the same application stack to 10 Azure regions for a global application. They want to manage infrastructure as code with consistent deployments.

A. Manual deployment in each region
B. Bicep templates with deployment stacks or Azure DevOps pipelines for multi-region deployment
C. Copy resources using the portal
D. Separate templates for each region

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Bicep (or ARM) templates define infrastructure as code. Deployment stacks or CI/CD pipelines can deploy the same template to multiple regions consistently. Parameters can vary per region while template stays same. Manual (A, C) doesn't scale. Separate templates (D) cause configuration drift.

**Key Concept:** [Bicep Multi-region](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-to-management-group)
</details>

### Question 39
**Scenario:** A company wants to provide developers with self-service access to create pre-approved VM configurations. VMs should be auto-shutdown at night and have budget limits per developer.

A. Give developers full subscription access
B. Azure DevTest Labs with pre-configured VM bases, auto-shutdown policies, and per-user quotas
C. Azure Blueprints
D. Manual VM creation requests

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** DevTest Labs provides self-service VM provisioning from pre-approved images. Auto-shutdown policies reduce costs. Per-user cost quotas prevent overspending. Claim-based VMs allow sharing. Full access (A) lacks controls. Blueprints (C) are for environment standards, not self-service. Manual requests (D) slow development.

**Key Concept:** [Azure DevTest Labs](https://learn.microsoft.com/en-us/azure/devtest-labs/devtest-lab-overview)
</details>

### Question 40
**Scenario:** A company is designing a microservices application. Services need to discover each other, load balance between instances, and communicate securely with mutual TLS. What should they design?

A. Hard-coded service endpoints
B. Azure Kubernetes Service with service mesh (Istio or Linkerd)
C. Azure Load Balancer
D. DNS-based discovery only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Service mesh provides service discovery, load balancing, and mTLS between services automatically. Istio or Linkerd on AKS handles communication concerns. Sidecar proxies manage traffic without application changes. Hard-coded (A) doesn't scale. Load Balancer (C) doesn't provide mTLS. DNS (D) doesn't provide load balancing or security.

**Key Concept:** [Service Mesh](https://learn.microsoft.com/en-us/azure/aks/istio-about)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | A | Identity/Governance/Monitoring |
| 2 | B | Identity/Governance/Monitoring |
| 3 | B | Identity/Governance/Monitoring |
| 4 | B | Identity/Governance/Monitoring |
| 5 | B | Identity/Governance/Monitoring |
| 6 | B | Identity/Governance/Monitoring |
| 7 | B | Identity/Governance/Monitoring |
| 8 | B | Identity/Governance/Monitoring |
| 9 | B | Identity/Governance/Monitoring |
| 10 | B | Identity/Governance/Monitoring |
| 11 | B | Identity/Governance/Monitoring |
| 12 | C | Data Storage |
| 13 | B | Data Storage |
| 14 | A | Data Storage |
| 15 | B | Data Storage |
| 16 | B | Data Storage |
| 17 | B | Data Storage |
| 18 | C | Data Storage |
| 19 | B | Data Storage |
| 20 | B | Data Storage |
| 21 | B | Business Continuity |
| 22 | A | Business Continuity |
| 23 | B | Business Continuity |
| 24 | B | Business Continuity |
| 25 | B | Business Continuity |
| 26 | B | Infrastructure |
| 27 | B | Infrastructure |
| 28 | B | Infrastructure |
| 29 | C | Infrastructure |
| 30 | B | Infrastructure |
| 31 | B | Infrastructure |
| 32 | B | Infrastructure |
| 33 | B | Infrastructure |
| 34 | B | Infrastructure |
| 35 | B | Infrastructure |
| 36 | B | Infrastructure |
| 37 | B | Infrastructure |
| 38 | B | Infrastructure |
| 39 | B | Infrastructure |
| 40 | B | Infrastructure |
