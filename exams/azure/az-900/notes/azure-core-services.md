# Azure Core Services

## Table of Contents
1. [Compute Services](#compute-services)
2. [Storage Services](#storage-services)
3. [Networking Services](#networking-services)
4. [Database Services](#database-services)
5. [Identity and Access Management](#identity-and-access-management)
6. [Management and Governance](#management-and-governance)
7. [Service Selection Guide](#service-selection-guide)
8. [Exam Tips](#exam-tips)

---

## Compute Services

### Azure Virtual Machines (VMs)
Infrastructure as a Service (IaaS) - Full control over the operating system.

#### Key Features
- Windows or Linux operating systems
- Multiple VM series (General purpose, Compute optimized, Memory optimized, etc.)
- Custom software installation
- Full administrative access
- Lift-and-shift migration scenarios

#### VM Sizes and Series
**General Purpose (B, D series):**
- Balanced CPU-to-memory ratio
- Web servers, small databases
- Development and test environments

**Compute Optimized (F series):**
- High CPU-to-memory ratio
- Web servers with high traffic
- Batch processing
- Application servers

**Memory Optimized (E, M series):**
- High memory-to-CPU ratio
- Relational databases
- In-memory analytics
- Large caches

**Storage Optimized (L series):**
- High disk throughput and I/O
- Big Data applications
- SQL and NoSQL databases

**GPU Enabled (N series):**
- Graphics rendering
- AI and machine learning
- Video encoding

#### Availability Options
1. **Availability Sets**
   - Protection from hardware failures within datacenter
   - 99.95% SLA
   - Fault domains (physical separation)
   - Update domains (planned maintenance)

2. **Availability Zones**
   - Protection from datacenter failures
   - 99.99% SLA
   - Physically separate datacenters within region
   - Independent power, cooling, networking

3. **Virtual Machine Scale Sets**
   - Automatic scaling (up to 1000 VMs)
   - Load balancer integration
   - Identical VMs
   - Auto-scaling rules based on metrics

#### Use Cases
- Legacy application migration
- Custom software requiring specific OS
- Full control needed
- Specialized configurations

### Azure App Service
Platform as a Service (PaaS) for web applications.

#### Features
- Multiple programming languages (.NET, Java, Node.js, Python, PHP)
- Built-in scaling (manual and automatic)
- CI/CD integration
- Deployment slots
- Custom domains and SSL
- Authentication and authorization

#### App Service Plans
**Free/Shared:** Development and testing
**Basic:** Dedicated compute, manual scale
**Standard:** Production workloads, auto-scale, staging slots
**Premium:** Enhanced performance, more scale, advanced features
**Isolated:** Dedicated environment, maximum scale

#### Types of Apps
1. **Web Apps** - Host websites and web applications
2. **API Apps** - Host RESTful APIs
3. **Web Apps for Containers** - Run containerized applications
4. **Static Web Apps** - Host static content with serverless APIs

#### Use Cases
- Web applications
- REST APIs
- Mobile backends
- Microservices

### Azure Container Services

#### Azure Container Instances (ACI)
Simplest way to run containers in Azure.

**Features:**
- No orchestration needed
- Per-second billing
- Fast startup
- Hypervisor isolation
- Custom sizes

**Use Cases:**
- Simple containerized applications
- Task automation
- Build and test jobs
- Event-driven applications

#### Azure Kubernetes Service (AKS)
Managed Kubernetes orchestration service.

**Features:**
- Automated upgrades and patching
- Integrated logging and monitoring
- Horizontal scaling
- Identity and access management
- Virtual network integration

**Components:**
- **Control Plane** - Free, managed by Azure
- **Node Pools** - Worker nodes (VMs)
- **Pods** - Container instances
- **Services** - Network access to pods

**Use Cases:**
- Microservices architectures
- Complex containerized applications
- Multi-container applications
- DevOps workflows

### Azure Functions
Serverless compute - run code without managing infrastructure.

#### Key Characteristics
- Event-driven execution
- Pay per execution (consumption plan)
- Automatic scaling
- Supports multiple languages
- Stateless or stateful (Durable Functions)

#### Triggers and Bindings
**Common Triggers:**
- HTTP requests
- Timer (scheduled)
- Azure Queue Storage
- Azure Service Bus
- Azure Event Hub
- Blob Storage

**Bindings:**
- Input bindings - Read data
- Output bindings - Write data

#### Hosting Plans
1. **Consumption Plan**
   - Pay per execution
   - Automatic scaling
   - 5-minute timeout (default)

2. **Premium Plan**
   - Pre-warmed instances
   - Longer execution duration
   - Virtual network connectivity

3. **App Service Plan**
   - Dedicated resources
   - Unlimited execution time
   - Predictable pricing

#### Use Cases
- API backends
- Scheduled tasks
- Event processing
- Integration workflows
- Real-time file processing

### Azure Virtual Desktop
Desktop and application virtualization service.

**Features:**
- Windows 10/11 multi-session
- Microsoft 365 Apps optimization
- FSLogix profile containers
- Security and compliance
- Hybrid identity support

**Use Cases:**
- Remote work
- Contractor/partner access
- Branch offices
- Compliance scenarios

---

## Storage Services

### Azure Storage Account
Foundation for Azure storage services.

#### Storage Account Types
1. **Standard General-purpose v2** - Most scenarios
2. **Premium Block Blobs** - High transaction rates
3. **Premium File Shares** - Enterprise file shares
4. **Premium Page Blobs** - VM disks

#### Performance Tiers
- **Standard** - HDD-based, cost-effective
- **Premium** - SSD-based, low latency

#### Redundancy Options

**Locally Redundant (LRS):**
- 3 copies within single datacenter
- 99.999999999% (11 9's) durability
- Lowest cost
- Protection from server/drive failures

**Zone-Redundant (ZRS):**
- 3 copies across availability zones
- 99.9999999999% (12 9's) durability
- Protection from datacenter failures
- Higher availability

**Geo-Redundant (GRS):**
- 6 copies (3 local + 3 secondary region)
- 99.99999999999999% (16 9's) durability
- Protection from regional disasters
- Secondary is read-only (unless RA-GRS)

**Geo-Zone-Redundant (GZRS):**
- Combines ZRS and GRS
- Maximum durability and availability

### Blob Storage
Object storage for unstructured data.

#### Blob Types
1. **Block Blobs** - Text and binary data, up to 190.7 TB
2. **Append Blobs** - Optimized for append operations (logs)
3. **Page Blobs** - Random access files, VHD files (up to 8 TB)

#### Access Tiers
**Hot:**
- Frequent access
- Highest storage cost
- Lowest access cost

**Cool:**
- Infrequent access (30+ days)
- Lower storage cost
- Higher access cost
- 30-day minimum

**Cold:**
- Rare access (90+ days)
- Even lower storage cost
- Even higher access cost
- 90-day minimum

**Archive:**
- Rarely accessed (180+ days)
- Lowest storage cost
- Highest access cost
- Rehydration time (hours)
- 180-day minimum

#### Use Cases
- Backups and archives
- Media files (images, videos)
- Application data
- Data lakes
- Static website hosting

### Azure Files
Fully managed file shares in the cloud.

**Features:**
- SMB and NFS protocols
- Accessible from Windows, Linux, macOS
- Azure File Sync (hybrid scenarios)
- Snapshots for backup
- Azure AD authentication

**Performance Tiers:**
- **Standard** - HDD-based
- **Premium** - SSD-based

**Use Cases:**
- Lift-and-shift of file shares
- Shared application configuration
- Development and debugging tools
- Containerized applications

### Queue Storage
Message queue for asynchronous communication.

**Features:**
- Store millions of messages
- Messages up to 64 KB
- TTL up to 7 days
- REST API access

**Use Cases:**
- Decoupling application components
- Asynchronous processing
- Task queuing

### Table Storage
NoSQL key-value store for structured data.

**Features:**
- Schemaless design
- Partition key + row key
- Fast queries
- Cost-effective

**Use Cases:**
- Web application data
- Address books
- Device information
- Metadata storage

### Disk Storage
Block-level storage for VMs.

**Disk Types:**
1. **Ultra Disk** - Highest performance, mission-critical
2. **Premium SSD v2** - Balance of performance and cost
3. **Premium SSD** - Production workloads
4. **Standard SSD** - Web servers, light apps
5. **Standard HDD** - Backup, non-critical

**Disk Roles:**
- OS Disk - Operating system
- Data Disk - Application data
- Temporary Disk - Ephemeral storage

---

## Networking Services

### Virtual Network (VNet)
Fundamental networking building block in Azure.

#### Features
- Private IP address space
- Subnets for segmentation
- Internet connectivity
- VNet peering
- Service endpoints
- Private endpoints

#### Address Space
- RFC 1918 private addresses
- 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- CIDR notation
- Non-overlapping spaces for peering

#### Subnets
- Logical segmentation
- Network security groups (NSGs)
- Route tables
- Service delegation

#### Use Cases
- Isolated network for Azure resources
- Hybrid connectivity
- Network segmentation
- Secure communication

### Network Security Group (NSG)
Filter network traffic to and from Azure resources.

**Components:**
- Security rules
- Source/destination
- Protocol (TCP, UDP, Any)
- Port range
- Action (Allow/Deny)
- Priority (100-4096)

**Default Rules:**
- Allow VNet traffic
- Allow Azure Load Balancer
- Deny all inbound internet traffic
- Allow all outbound internet traffic

**Application:**
- Subnet level
- Network interface level
- Both (rules combined)

### Azure Load Balancer
Distribute network traffic across multiple resources.

**Types:**
1. **Public Load Balancer**
   - Internet-facing
   - Public IP address
   - Distribute incoming internet traffic

2. **Internal Load Balancer**
   - Private IP address
   - Internal traffic distribution
   - Multi-tier applications

**SKUs:**
- **Basic** - Small scale, free
- **Standard** - Production, zone-redundant, SLA

**Features:**
- Health probes
- Port forwarding
- Outbound rules
- High availability

### Azure VPN Gateway
Encrypted connection over public internet.

**Types:**
1. **Site-to-Site (S2S)**
   - Connect on-premises network to Azure
   - IPsec/IKE VPN tunnel
   - Requires VPN device on-premises

2. **Point-to-Site (P2S)**
   - Individual computers to Azure
   - Certificate or Azure AD authentication
   - Remote access

3. **VNet-to-VNet**
   - Connect Azure VNets
   - Different regions or subscriptions

**Gateway SKUs:**
- Basic, VpnGw1, VpnGw2, VpnGw3, etc.
- Different throughput and features

### Azure ExpressRoute
Private, dedicated connection to Azure.

**Features:**
- Not over public internet
- Higher reliability
- Lower latency
- Higher bandwidth (50 Mbps to 100 Gbps)
- Predictable performance

**Connectivity Models:**
- CloudExchange co-location
- Point-to-point Ethernet
- Any-to-any (IPVPN)
- ExpressRoute Direct

**Use Cases:**
- Large data transfers
- Compliance requirements
- Disaster recovery
- Hybrid cloud scenarios

### Azure DNS
Host DNS domains in Azure.

**Features:**
- Anycast network
- Fast DNS responses
- Azure integration
- Alias records
- Private DNS zones

**Record Types:**
- A (IPv4 address)
- AAAA (IPv6 address)
- CNAME (canonical name)
- MX (mail exchange)
- TXT (text)
- Alias (Azure resources)

### Azure Application Gateway
Web traffic load balancer (Layer 7).

**Features:**
- URL-based routing
- SSL termination
- Web Application Firewall (WAF)
- Cookie-based session affinity
- Autoscaling

**Components:**
- Frontend IP
- Backend pools
- HTTP settings
- Listeners
- Rules

### Azure Content Delivery Network (CDN)
Distribute content to edge locations worldwide.

**Benefits:**
- Reduced latency
- Lower bandwidth costs
- Handle traffic spikes
- DDoS protection

**Use Cases:**
- Static website acceleration
- Video streaming
- File downloads
- Dynamic site acceleration

---

## Database Services

### Azure SQL Database
Fully managed relational database (PaaS).

#### Features
- Based on SQL Server engine
- Automatic backups (7-35 days)
- Point-in-time restore
- High availability (99.99% SLA)
- Automatic patching and updates
- Elastic pools for resource sharing

#### Deployment Options
1. **Single Database** - Isolated database with dedicated resources
2. **Elastic Pool** - Share resources among multiple databases
3. **Managed Instance** - Near 100% SQL Server compatibility

#### Purchasing Models
**DTU (Database Transaction Unit):**
- Bundled measure (CPU, memory, I/O)
- Basic, Standard, Premium tiers
- Simple, predictable pricing

**vCore:**
- Choose CPU, memory, storage independently
- General Purpose, Business Critical, Hyperscale
- Hybrid benefit for cost savings

#### Use Cases
- Cloud-native applications
- SaaS applications
- Microservices data storage
- Modernizing SQL Server workloads

### Azure Cosmos DB
Globally distributed, multi-model NoSQL database.

#### Key Features
- Turnkey global distribution
- Single-digit millisecond latency
- Multiple consistency levels
- Multiple APIs (SQL, MongoDB, Cassandra, Gremlin, Table)
- Automatic and instant scalability

#### Consistency Levels
1. **Strong** - Linearizability, highest consistency
2. **Bounded Staleness** - Configurable lag
3. **Session** - Consistent within session
4. **Consistent Prefix** - Updates in order
5. **Eventual** - Eventually consistent, lowest latency

#### Use Cases
- IoT and telemetry
- Retail and e-commerce
- Gaming
- Web and mobile applications

### Azure Database for MySQL/PostgreSQL
Fully managed open-source databases.

**Features:**
- Community editions
- Automatic backups
- High availability options
- Scaling (up and out)
- Advanced threat protection

**Deployment Options:**
- Single Server - Basic, General Purpose, Memory Optimized
- Flexible Server - More configuration options
- Hyperscale (PostgreSQL only) - Horizontal scaling

### Azure SQL Managed Instance
Most SQL Server features with PaaS benefits.

**Features:**
- SQL Server compatibility (near 100%)
- VNet integration
- Instance-level features
- Automated backups and patching

**Use Cases:**
- Lift-and-shift SQL Server
- Applications requiring SQL Server features
- Existing SQL Server investments

### Azure Synapse Analytics
Unified analytics service (formerly SQL Data Warehouse).

**Components:**
- SQL pools (data warehouse)
- Spark pools (big data)
- Data Explorer (time series)
- Pipelines (data integration)

**Use Cases:**
- Data warehousing
- Big data analytics
- Real-time analytics
- Unified analytics platform

---

## Identity and Access Management

### Azure Active Directory (Azure AD)
Cloud-based identity and access management service.

#### Key Features
- User and group management
- Single Sign-On (SSO)
- Multi-Factor Authentication (MFA)
- Conditional Access
- Application management
- Device management

#### Azure AD Editions
**Free:**
- User and group management
- Basic SSO
- Self-service password change

**Premium P1:**
- Conditional Access
- Dynamic groups
- Self-service password reset
- Hybrid identity features

**Premium P2:**
- Identity Protection
- Privileged Identity Management (PIM)
- Access reviews

#### Identity Models
1. **Cloud-only** - Identities exist only in Azure AD
2. **Hybrid** - Synchronized with on-premises AD
3. **External identities** - B2B collaboration

### Multi-Factor Authentication (MFA)
Additional security layer beyond passwords.

**Methods:**
- Mobile app notification
- Mobile app verification code
- Phone call
- Text message

**When to Use:**
- High-privilege accounts
- Remote access
- Sensitive operations
- Compliance requirements

### Role-Based Access Control (RBAC)
Fine-grained access management.

**Built-in Roles:**
- **Owner** - Full access including access management
- **Contributor** - Create and manage resources, no access management
- **Reader** - View resources only
- **User Access Administrator** - Manage user access

**Custom Roles:**
- Define specific permissions
- JSON-based definitions
- Reusable across scopes

**Scope Levels:**
1. Management group
2. Subscription
3. Resource group
4. Resource

---

## Management and Governance

### Azure Resource Manager (ARM)
Deployment and management service for Azure.

**Features:**
- Consistent management layer
- Declarative templates
- Resource grouping
- Access control (RBAC)
- Tags for organization
- Policies for compliance

**Benefits:**
- Deploy, update, delete resources as group
- Consistent state
- Dependencies handled
- Repeatable deployments

### Resource Groups
Logical container for Azure resources.

**Characteristics:**
- Resources exist in only one resource group
- Resources can be in different regions
- Can move resources between groups
- Lifecycle management
- Access control boundary

**Best Practices:**
- Group by lifecycle
- Group by permissions
- Group by billing
- Use naming conventions

### Azure Monitor
Comprehensive monitoring solution.

**Components:**
1. **Metrics** - Numerical time-series data
2. **Logs** - Text-based event data
3. **Alerts** - Notifications based on conditions
4. **Dashboards** - Visualizations

**Features:**
- Application Insights (APM)
- Log Analytics
- Metrics Explorer
- Action groups
- Smart detection

### Azure Policy
Enforce standards and assess compliance.

**Features:**
- Built-in policies
- Custom policies
- Policy assignments
- Compliance reporting
- Remediation tasks

**Common Policies:**
- Allowed resource types
- Allowed locations
- Required tags
- SKU restrictions
- Naming conventions

### Azure Blueprints
Repeatable set of Azure resources.

**Components:**
- Resource groups
- ARM templates
- Policy assignments
- Role assignments

**Use Cases:**
- Environment standardization
- Compliance requirements
- Rapid deployment
- Governance at scale

---

## Service Selection Guide

### Compute Decision Tree

**Need full OS control?**
- Yes → Virtual Machines

**Running containers?**
- Orchestration needed? → AKS
- Simple containers? → Container Instances

**Event-driven code?**
- Yes → Azure Functions

**Web application?**
- Yes → App Service

### Storage Decision Tree

**Data Type?**
- **Files** → Azure Files
- **Objects** → Blob Storage
- **Messages** → Queue Storage
- **Tables** → Table Storage or Cosmos DB
- **VM Disks** → Managed Disks

**Access Frequency?**
- Hot → Hot tier
- Cool → Cool/Cold tier
- Archive → Archive tier

### Database Decision Tree

**Data Structure?**
- **Relational** → Azure SQL Database
- **NoSQL** → Cosmos DB
- **Open source** → MySQL/PostgreSQL
- **Analytical** → Synapse Analytics

**SQL Server features needed?**
- Yes → SQL Managed Instance

**Global distribution required?**
- Yes → Cosmos DB

### Networking Decision Tree

**Connectivity Type?**
- **Internet to Azure** → Public Load Balancer
- **Internal Azure** → Internal Load Balancer
- **On-premises to Azure** → VPN Gateway or ExpressRoute
- **Web traffic routing** → Application Gateway

**Hybrid Connectivity?**
- **Over internet** → VPN Gateway
- **Private connection** → ExpressRoute

---

## Exam Tips

### Key Concepts to Remember

#### Compute Services
- **VMs** - Full control, IaaS, lift-and-shift
- **App Service** - PaaS, web apps, auto-scale
- **AKS** - Container orchestration
- **Functions** - Serverless, event-driven, pay per execution

#### Storage Services
- **Blob** - Object storage, hot/cool/archive tiers
- **Files** - SMB file shares
- **Redundancy** - LRS (local), ZRS (zonal), GRS (geo), GZRS (both)

#### Networking
- **VNet** - Private network in Azure
- **NSG** - Traffic filtering rules
- **VPN Gateway** - Encrypted connection over internet
- **ExpressRoute** - Private dedicated connection
- **Load Balancer** - Layer 4 load balancing
- **Application Gateway** - Layer 7 web traffic load balancing

#### Databases
- **SQL Database** - PaaS relational database
- **Cosmos DB** - NoSQL, global distribution, multi-model
- **SQL Managed Instance** - Near 100% SQL Server compatibility

#### Identity
- **Azure AD** - Cloud identity service
- **MFA** - Additional authentication factor
- **RBAC** - Permission management
- **Roles** - Owner (full), Contributor (manage resources), Reader (view)

### Common Exam Scenarios

**Scenario: High Availability Web Application**
- App Service with Standard/Premium tier
- Multiple instances across availability zones
- Azure SQL Database with geo-replication
- Application Gateway with WAF

**Scenario: Hybrid Connectivity**
- VPN Gateway for encrypted internet connection
- ExpressRoute for private, dedicated connection
- Choose based on bandwidth, latency, security needs

**Scenario: Cost Optimization**
- Azure Functions for sporadic workloads
- Cool/Archive storage tiers for infrequent access
- Reserved instances for predictable VM workloads
- Spot VMs for flexible, interruptible workloads

**Scenario: Global Application**
- Cosmos DB for multi-region database
- CDN for content distribution
- Traffic Manager for global load balancing
- Azure Front Door for web application acceleration

**Scenario: Disaster Recovery**
- Geo-redundant storage (GRS/GZRS)
- Azure Site Recovery for VM replication
- Azure Backup for data protection
- Multiple region deployment

### Service Comparison Tables

#### Compute Comparison
| Service | Type | Control | Scaling | Use Case |
|---------|------|---------|---------|----------|
| VM | IaaS | Full | Manual/VMSS | Legacy apps, full control |
| App Service | PaaS | Limited | Auto/Manual | Web apps, APIs |
| AKS | PaaS | Moderate | Auto | Microservices, containers |
| Functions | Serverless | Minimal | Automatic | Event-driven, scheduled |

#### Storage Redundancy SLA
| Type | Copies | SLA | Use Case |
|------|--------|-----|----------|
| LRS | 3 (local) | 99.9% | Cost-effective |
| ZRS | 3 (zones) | 99.9% | Zone resilience |
| GRS | 6 (geo) | 99.99% | Disaster recovery |
| GZRS | 6 (zone+geo) | 99.99% | Maximum availability |

### Quick Facts

**VM Availability:**
- Single VM with Premium SSD: 99.9%
- Availability Set: 99.95%
- Availability Zones: 99.99%

**Storage Limits:**
- Blob: 190.7 TB per block blob
- Files: 100 TiB per share
- Queue message: 64 KB
- Table entity: 1 MB

**Networking:**
- VNet address space: RFC 1918
- NSG rules per NSG: 1000
- ExpressRoute: 50 Mbps to 100 Gbps

**Database:**
- SQL Database max size: 4 TB (DTU), 100 TB (vCore Hyperscale)
- Cosmos DB: Unlimited storage
- Automatic backups: 7-35 days

### Study Approach

1. **Understand, Don't Memorize**
   - Know why to choose each service
   - Understand trade-offs
   - Think about real scenarios

2. **Compare and Contrast**
   - VMs vs App Service vs Functions
   - VPN Gateway vs ExpressRoute
   - SQL Database vs Cosmos DB

3. **Know the Limits**
   - SLAs for each service
   - Size and throughput limits
   - Cost implications

4. **Practice Scenarios**
   - Architecture design questions
   - Service selection based on requirements
   - Cost optimization strategies

5. **Focus on Core Services**
   - Compute, Storage, Networking, Databases
   - These are most frequently tested
   - Understand fundamentals deeply

### Final Checklist

- [ ] Understand all compute service types and use cases
- [ ] Know storage types, tiers, and redundancy options
- [ ] Differentiate networking services and when to use each
- [ ] Compare database options and selection criteria
- [ ] Understand Azure AD, MFA, and RBAC
- [ ] Know management tools (ARM, Resource Groups, Monitor)
- [ ] Remember SLAs for key services
- [ ] Practice scenario-based questions
- [ ] Review service comparison tables
- [ ] Understand hybrid connectivity options
