# Azure Architecture and Services - AZ-900

## Azure Global Infrastructure

### Azure Regions
- **Definition:** Geographical areas containing one or more data centers
- **Characteristics:**
  - Each region is at least 300 miles from another region
  - Connected by dedicated regional low-latency networks
  - Provide local presence for better performance and compliance
- **Key Concepts:**
  - **Region pairs:** Most regions paired with another region for disaster recovery
  - **Sovereign regions:** Special regions for compliance (US Government, China)
  - **Recommended regions:** Regions that provide the broadest range of service capabilities

### Availability Zones
- **Definition:** Physically separate data centers within an Azure region
- **Characteristics:**
  - Each zone has independent power, cooling, and networking
  - Minimum of three zones in enabled regions
  - Connected through high-speed, private fiber-optic networks
- **Benefits:**
  - Protect against data center failures
  - 99.99% VM uptime SLA when deployed across zones
  - Zone-redundant services replicate across zones automatically

### Azure Resources and Resource Groups

#### Azure Resources
- **Definition:** Manageable items available through Azure
- **Examples:** Virtual machines, storage accounts, web apps, databases, virtual networks
- **Lifecycle:** Created, managed, and deleted as individual entities

#### Resource Groups
- **Definition:** Logical containers that hold related resources for an Azure solution
- **Key Properties:**
  - All resources must belong to a resource group
  - Resources can only belong to one resource group
  - Resource groups can't be nested
  - Resources can be moved between resource groups
- **Best Practices:**
  - Group resources by lifecycle, application, or department
  - Apply consistent naming conventions
  - Use resource groups for access control and cost management

### Azure Subscriptions
- **Definition:** Logical unit of Azure services linked to an Azure account
- **Key Functions:**
  - Billing boundary (separate billing reports and invoices)
  - Access control boundary (manage access at subscription level)
  - Resource organization and management
- **Subscription Types:**
  - Free tier (12 months of popular services free)
  - Pay-as-you-go (pay for what you use)
  - Enterprise agreements (volume licensing)
  - Student subscriptions (free credits for students)

### Management Groups
- **Definition:** Containers that help manage access, policy, and compliance across multiple subscriptions
- **Hierarchy:**
  - Root management group (tenant level)
  - Management groups (organize subscriptions)
  - Subscriptions (billing and access boundaries)
  - Resource groups (logical containers)
  - Resources (actual Azure services)
- **Benefits:**
  - Apply policies and access controls at scale
  - Organize subscriptions into hierarchies
  - Inherit permissions and policies down the hierarchy

## Azure Compute Services

### Azure Virtual Machines (VMs)
- **Description:** Software emulations of physical computers (IaaS)
- **Use Cases:**
  - Total control over operating system
  - Run custom software
  - Custom hosting configurations
- **VM Families:**
  - **General purpose:** Balanced CPU-to-memory ratio (B, D, F series)
  - **Compute optimized:** High CPU-to-memory ratio (F series)
  - **Memory optimized:** High memory-to-CPU ratio (E, M series)
  - **Storage optimized:** High disk throughput and IO (L series)
  - **GPU:** Specialized for graphics and AI workloads (N series)
  - **High performance compute:** Fastest CPU with InfiniBand networking (H series)

### Azure Virtual Machine Scale Sets
- **Description:** Deploy and manage identical VMs as a group
- **Benefits:**
  - Automatic scaling based on demand or schedule
  - High availability across fault domains
  - Load balancing across VM instances
- **Use Cases:**
  - Web applications requiring auto-scaling
  - Big data workloads
  - Container orchestration

### Azure Container Instances (ACI)
- **Description:** Run containers on Azure without managing virtual machines (PaaS)
- **Benefits:**
  - Fast startup times (seconds)
  - Pay per second billing
  - No infrastructure management
- **Use Cases:**
  - Batch jobs and task automation
  - Development and testing
  - Microservices deployment

### Azure Kubernetes Service (AKS)
- **Description:** Managed Kubernetes service for container orchestration
- **Benefits:**
  - Simplified deployment and management
  - Integrated monitoring and logging
  - Automatic upgrades and patching
- **Use Cases:**
  - Microservices architecture
  - DevOps and CI/CD pipelines
  - Hybrid and multi-cloud deployments

### Azure App Service
- **Description:** Platform for building and hosting web apps, mobile backends, and APIs (PaaS)
- **Supported Languages:** .NET, Java, Ruby, Node.js, PHP, Python
- **Features:**
  - Built-in auto-scaling and load balancing
  - Continuous deployment from Git
  - Staging environments
  - Custom domains and SSL certificates
- **App Service Plans:** Define compute resources and pricing tier

### Azure Functions
- **Description:** Event-driven, serverless compute service (Functions as a Service)
- **Benefits:**
  - No infrastructure management
  - Pay only for execution time
  - Automatic scaling
  - Multiple trigger types (HTTP, timer, queue, blob)
- **Use Cases:**
  - API backends
  - Data processing
  - IoT message processing
  - File processing

## Azure Networking Services

### Azure Virtual Network (VNet)
- **Description:** Fundamental building block for private networks in Azure
- **Features:**
  - Isolation and segmentation
  - Internet communications
  - Communication between Azure resources
  - Communication with on-premises resources
- **Subnets:** Divide VNet into smaller segments for organization and security

### VPN Gateway
- **Description:** Send encrypted traffic between Azure VNet and on-premises location
- **Types:**
  - **Site-to-Site:** Connect on-premises network to Azure VNet
  - **Point-to-Site:** Connect individual devices to Azure VNet
  - **VNet-to-VNet:** Connect Azure VNets together
- **VPN Types:**
  - **Policy-based:** Specify IP addresses to encrypt
  - **Route-based:** Use routing to direct traffic

### Azure ExpressRoute
- **Description:** Private connection between on-premises and Azure
- **Benefits:**
  - Faster, reliable, and secure than internet connections
  - Consistent network performance
  - Lower latency
  - Higher security (traffic doesn't traverse internet)
- **Connectivity Models:**
  - Co-location at cloud exchange
  - Point-to-point Ethernet connection
  - Any-to-any (IPVPN) network

### Azure DNS
- **Description:** Host DNS domains in Azure using Azure infrastructure
- **Benefits:**
  - Reliability and performance
  - Security features
  - Ease of use
  - Customizable virtual networks with private domains

## Azure Storage Services

### Storage Account Types
- **Standard general-purpose v2:** Most scenarios (blob, file, queue, table)
- **Premium block blobs:** High transaction rates and low latency
- **Premium file shares:** Enterprise file share scenarios
- **Premium page blobs:** High-performance scenarios for page blobs

### Azure Blob Storage
- **Description:** Object storage solution for text and binary data
- **Storage Tiers:**
  - **Hot:** Frequently accessed data
  - **Cool:** Infrequently accessed (stored for at least 30 days)
  - **Archive:** Rarely accessed (stored for at least 180 days)
- **Blob Types:**
  - **Block blobs:** Text and binary data up to 4.7TB
  - **Append blobs:** Logging scenarios
  - **Page blobs:** Virtual hard disk files up to 8TB

### Azure Files
- **Description:** Fully managed file shares in the cloud (SMB and NFS protocols)
- **Use Cases:**
  - Replace or supplement on-premises file servers
  - "Lift and shift" applications
  - Shared application data
- **Benefits:**
  - Shared access across multiple VMs
  - Azure File Sync for hybrid scenarios

### Azure Queue Storage
- **Description:** Service for storing large numbers of messages
- **Features:**
  - Messages up to 64KB in size
  - Millions of messages capacity
  - Accessible via HTTP/HTTPS
- **Use Cases:**
  - Decouple application components
  - Handle workload spikes
  - Build workflow patterns

### Azure Table Storage
- **Description:** NoSQL datastore for structured, non-relational data
- **Features:**
  - Schemaless design
  - Scalable and cost-effective
  - OData-based queries
- **Use Cases:**
  - Web application user data
  - Address books
  - Device information
  - Metadata storage

### Storage Redundancy Options
- **Locally Redundant Storage (LRS):** 3 copies within single data center
- **Zone Redundant Storage (ZRS):** 3 copies across availability zones
- **Geo-Redundant Storage (GRS):** LRS + 3 copies in paired region
- **Read-Access Geo-Redundant Storage (RA-GRS):** GRS + read access to secondary region

## Azure Identity Services

### Azure Active Directory (Azure AD)
- **Description:** Cloud-based identity and access management service
- **Key Features:**
  - Authentication (verify identity)
  - Single sign-on (SSO)
  - Application management
  - Business-to-business (B2B) identity services
  - Business-to-customer (B2C) identity services
  - Device management

### Multi-Factor Authentication (MFA)
- **Description:** Security process requiring multiple forms of identification
- **Factors:**
  - Something you know (password)
  - Something you have (phone, token)
  - Something you are (biometric)
- **Implementation:** Available in Azure AD free and premium tiers

### Conditional Access
- **Description:** Tool for bringing signals together to make access decisions
- **Signals:** User, location, device, application, risk detection
- **Decisions:** Allow access, require MFA, block access
- **Controls:** Session controls, application restrictions

### Azure Role-Based Access Control (RBAC)
- **Description:** Authorization system for managing access to Azure resources
- **Components:**
  - **Security principal:** User, group, service principal, managed identity
  - **Role definition:** Collection of permissions
  - **Scope:** Set of resources the access applies to
- **Built-in Roles:**
  - Owner (full access)
  - Contributor (create and manage, can't grant access)
  - Reader (view only)
  - Service-specific roles

## Key Takeaways for AZ-900

1. **Global Infrastructure:** Regions, availability zones, and geo-redundancy provide reliability
2. **Resource Organization:** Management groups > subscriptions > resource groups > resources
3. **Compute Options:** VMs (IaaS), App Service/Functions (PaaS), Containers (flexible)
4. **Networking:** VNets provide isolation, VPN/ExpressRoute connect on-premises
5. **Storage:** Multiple types and tiers for different scenarios and costs
6. **Identity:** Azure AD centralizes identity management with RBAC for access control