# Azure Fundamentals (AZ-900) Practice Questions

40 practice questions covering all exam domains.

**Exam Domain Breakdown:**
- Describe cloud concepts (25-30%) - 11 questions
- Describe Azure architecture and services (35-40%) - 15 questions
- Describe Azure management and governance (30-35%) - 14 questions

---

## Describe Cloud Concepts (25-30%)

### Question 1
**Scenario:** A company wants to avoid large upfront investments in hardware and instead pay only for the computing resources they use. Which cloud benefit describes this?

A. High availability
B. Scalability
C. Consumption-based pricing
D. Geo-distribution

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Consumption-based pricing (pay-as-you-go) means you only pay for the resources you use, eliminating large capital expenditures. High availability is about uptime. Scalability is about adjusting resources. Geo-distribution is about global presence.

**Key Concept:** [Cloud Economics](https://azure.microsoft.com/en-us/overview/what-is-cloud-computing/)
</details>

---

### Question 2
**Scenario:** A retail company's website experiences 5x normal traffic during holiday sales but returns to normal afterward. What cloud characteristic allows them to handle this efficiently?

A. Agility
B. Elasticity
C. High availability
D. Disaster recovery

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Elasticity is the ability to automatically scale resources up or down based on demand. This allows handling traffic spikes without over-provisioning. Agility is about quick deployment. High availability is about uptime. Disaster recovery is about recovering from failures.

**Key Concept:** [Elasticity in Cloud](https://docs.microsoft.com/en-us/azure/architecture/framework/scalability/design-scale)
</details>

---

### Question 3
**Scenario:** A company wants to use a service where they manage applications and data, but the cloud provider manages everything else including runtime, middleware, and operating system. Which cloud service model is this?

A. Infrastructure as a Service (IaaS)
B. Platform as a Service (PaaS)
C. Software as a Service (SaaS)
D. Function as a Service (FaaS)

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** PaaS provides a platform where users manage applications and data while the provider manages the infrastructure, OS, middleware, and runtime. IaaS requires managing the OS up. SaaS manages everything. FaaS is a subset of PaaS for functions.

**Key Concept:** [Cloud Service Models](https://azure.microsoft.com/en-us/overview/types-of-cloud-computing/)
</details>

---

### Question 4
**Scenario:** A financial services company must comply with strict data residency requirements and wants to keep some data on their own servers while using cloud services for other workloads. Which cloud model should they use?

A. Public cloud
B. Private cloud
C. Hybrid cloud
D. Community cloud

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Hybrid cloud combines on-premises infrastructure (or private cloud) with public cloud, allowing data to stay on-premises while other workloads run in the cloud. Public cloud is fully hosted by the provider. Private cloud is entirely on-premises. Community cloud is shared by organizations with common concerns.

**Key Concept:** [Hybrid Cloud](https://azure.microsoft.com/en-us/solutions/hybrid-cloud-app/)
</details>

---

### Question 5
**Scenario:** A company is comparing the total cost of running their IT infrastructure on-premises versus in the cloud. What is this type of analysis called?

A. Cost allocation
B. Total Cost of Ownership (TCO)
C. Return on Investment (ROI)
D. Break-even analysis

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Total Cost of Ownership (TCO) compares all costs of on-premises infrastructure (hardware, software, labor, facilities) against cloud costs. Azure provides a TCO Calculator for this purpose. Cost allocation is about distributing costs. ROI measures returns. Break-even analysis determines when investment pays off.

**Key Concept:** [Azure TCO Calculator](https://azure.microsoft.com/en-us/pricing/tco/calculator/)
</details>

---

### Question 6
**Scenario:** Which of the following is a characteristic of a public cloud deployment?

A. Resources are owned and operated by a single organization
B. Resources are shared among multiple organizations
C. Resources cannot be accessed over the internet
D. Resources require dedicated hardware per customer

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Public cloud resources are shared among multiple organizations (multi-tenant) and accessed over the internet. Single organization ownership describes private cloud. Public cloud is accessed over the internet. Shared infrastructure (not dedicated) reduces costs.

**Key Concept:** [Public Cloud](https://azure.microsoft.com/en-us/overview/what-is-a-public-cloud/)
</details>

---

### Question 7
**Scenario:** A company needs to ensure their application is available 99.99% of the time. Which cloud characteristic addresses this requirement?

A. Elasticity
B. Scalability
C. High availability
D. Agility

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** High availability ensures systems remain operational and accessible with minimal downtime, often measured as a percentage (99.99% uptime). Elasticity handles demand changes. Scalability is about growth. Agility is about rapid deployment.

**Key Concept:** [High Availability](https://docs.microsoft.com/en-us/azure/architecture/framework/resiliency/reliability-patterns)
</details>

---

### Question 8
**Scenario:** A startup wants to focus on developing their application without managing servers, operating systems, or infrastructure. Which cloud model best suits their needs?

A. IaaS
B. PaaS
C. SaaS
D. On-premises

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** PaaS allows developers to focus on application development while the cloud provider manages infrastructure, OS, middleware, and runtime. IaaS requires managing OS and up. SaaS provides complete applications. On-premises requires managing everything.

**Key Concept:** [PaaS Benefits](https://azure.microsoft.com/en-us/overview/what-is-paas/)
</details>

---

### Question 9
**Scenario:** Under the shared responsibility model for IaaS, who is responsible for patching and maintaining the operating system?

A. Microsoft only
B. The customer only
C. Both Microsoft and the customer
D. Neither (automated by Azure)

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** In IaaS, the customer is responsible for the operating system, including patching and maintenance. Microsoft manages the physical infrastructure. The responsibility shifts depending on the service model (IaaS, PaaS, SaaS).

**Key Concept:** [Shared Responsibility Model](https://docs.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility)
</details>

---

### Question 10
**Scenario:** A company wants to deploy new features to their application multiple times per day. Which cloud benefit enables this rapid deployment?

A. Fault tolerance
B. Disaster recovery
C. Agility
D. Predictability

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Agility in cloud computing refers to the ability to rapidly develop, test, and deploy applications. Cloud services provide tools and infrastructure that enable fast iteration. Fault tolerance handles failures. Disaster recovery handles major incidents. Predictability is about consistent performance/cost.

**Key Concept:** [Cloud Agility](https://azure.microsoft.com/en-us/overview/what-is-cloud-computing/)
</details>

---

### Question 11
**Scenario:** Which expense model represents cloud computing where you pay based on actual usage?

A. Capital Expenditure (CapEx)
B. Operational Expenditure (OpEx)
C. Fixed cost
D. Sunk cost

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud computing follows an OpEx model where you pay for services as you use them, rather than making large upfront capital investments (CapEx). This provides flexibility and reduces financial risk. Fixed and sunk costs don't describe the cloud pricing model.

**Key Concept:** [CapEx vs OpEx](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/strategy/business-outcomes/fiscal-outcomes)
</details>

---

## Describe Azure Architecture and Services (35-40%)

### Question 12
**Scenario:** A company needs to organize their Azure resources for different departments (HR, Finance, IT) with separate billing for each. What Azure feature should they use?

A. Resource Groups
B. Subscriptions
C. Management Groups
D. Azure Regions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Subscriptions provide billing boundaries and access control at a higher level. Each department can have its own subscription with separate billing. Resource Groups organize resources within a subscription. Management Groups manage multiple subscriptions. Regions are geographic locations.

**Key Concept:** [Azure Subscriptions](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources)
</details>

---

### Question 13
**Scenario:** An organization has multiple Azure subscriptions across different business units. They want to apply consistent policies across all subscriptions. What should they use?

A. Resource Groups
B. Resource Tags
C. Management Groups
D. Azure Blueprints

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Management Groups allow organizing multiple subscriptions and applying policies across them hierarchically. Resource Groups organize resources within subscriptions. Tags are for metadata. Blueprints define repeatable environments but Management Groups control policy inheritance.

**Key Concept:** [Management Groups](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview)
</details>

---

### Question 14
**Scenario:** A company needs to host virtual machines that require the highest level of isolation for security compliance. Which Azure compute option should they use?

A. Azure Virtual Machines
B. Azure Dedicated Host
C. Azure Virtual Machine Scale Sets
D. Azure Container Instances

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Dedicated Host provides physical servers dedicated to your organization, ensuring no other customers share the hardware. This addresses compliance requirements for workload isolation. Regular VMs share physical hosts. Scale Sets are for scaling. Container Instances are for containers.

**Key Concept:** [Azure Dedicated Host](https://docs.microsoft.com/en-us/azure/virtual-machines/dedicated-hosts)
</details>

---

### Question 15
**Scenario:** A developer needs to run code in response to HTTP requests without managing servers. The code runs only when triggered and billing is based on execution time. What service should they use?

A. Azure Virtual Machines
B. Azure App Service
C. Azure Functions
D. Azure Kubernetes Service

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Azure Functions is a serverless compute service that runs code on-demand in response to triggers (like HTTP requests) with consumption-based billing. VMs require server management. App Service is for web apps. AKS is for container orchestration.

**Key Concept:** [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)
</details>

---

### Question 16
**Scenario:** A company needs to store large amounts of unstructured data like documents, images, and videos. Which Azure storage service should they use?

A. Azure SQL Database
B. Azure Blob Storage
C. Azure Table Storage
D. Azure Queue Storage

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Blob Storage is designed for storing large amounts of unstructured data including documents, images, videos, and backups. SQL Database is for relational data. Table Storage is for structured NoSQL data. Queue Storage is for message queuing.

**Key Concept:** [Azure Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)
</details>

---

### Question 17
**Scenario:** An application needs to connect securely to Azure SQL Database from an Azure Virtual Network without traffic going over the public internet. What should be configured?

A. Service Endpoint
B. Public IP address
C. Load Balancer
D. Network Security Group

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Service Endpoints extend your VNet private address space to Azure services, keeping traffic on the Azure backbone network. Public IP would expose the database. Load Balancer distributes traffic. NSG controls network traffic flow but doesn't provide private connectivity.

**Key Concept:** [Virtual Network Service Endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview)
</details>

---

### Question 18
**Scenario:** A company wants to deploy a fully managed relational database with automatic backups, patching, and high availability without managing the underlying infrastructure. What service should they use?

A. SQL Server on Azure VM
B. Azure SQL Database
C. Azure Cosmos DB
D. Azure Database for PostgreSQL (single server)

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure SQL Database is a fully managed PaaS offering with automatic backups, patching, high availability, and scaling. SQL Server on VM requires managing the OS. Cosmos DB is NoSQL. PostgreSQL single server has similar management but the question specifies relational (SQL).

**Key Concept:** [Azure SQL Database](https://docs.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview)
</details>

---

### Question 19
**Scenario:** A company needs to distribute network traffic across multiple virtual machines to ensure high availability. What Azure service should they use?

A. Azure Traffic Manager
B. Azure Load Balancer
C. Azure Application Gateway
D. Azure Front Door

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Load Balancer distributes inbound traffic across VMs within a region at Layer 4 (TCP/UDP). Traffic Manager is DNS-based global load balancing. Application Gateway is Layer 7 load balancing with WAF. Front Door is global HTTP load balancing.

**Key Concept:** [Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)
</details>

---

### Question 20
**Scenario:** A company has resources in multiple Azure regions and wants to route users to the nearest region for better performance. What service should they use?

A. Azure Load Balancer
B. Azure Traffic Manager
C. Azure VPN Gateway
D. Azure ExpressRoute

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Traffic Manager uses DNS-based routing to direct users to endpoints based on routing methods like performance (nearest region), priority, or geographic. Load Balancer works within a region. VPN Gateway connects networks. ExpressRoute is private connectivity.

**Key Concept:** [Azure Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
</details>

---

### Question 21
**Scenario:** A company needs private, dedicated network connectivity between their on-premises data center and Azure that doesn't traverse the public internet. What should they use?

A. Site-to-Site VPN
B. Point-to-Site VPN
C. Azure ExpressRoute
D. VNet Peering

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Azure ExpressRoute provides private, dedicated connectivity through a connectivity provider, not traversing the public internet. Site-to-Site VPN uses encrypted tunnels over the internet. Point-to-Site VPN is for individual devices. VNet Peering connects Azure virtual networks.

**Key Concept:** [Azure ExpressRoute](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction)
</details>

---

### Question 22
**Scenario:** A web application needs to cache frequently accessed data in memory to reduce database load and improve performance. What Azure service should be used?

A. Azure Blob Storage
B. Azure Cache for Redis
C. Azure CDN
D. Azure Queue Storage

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Cache for Redis provides an in-memory data store for caching frequently accessed data, reducing database load. Blob Storage is for files. CDN caches static content at edge locations. Queue Storage is for message queuing.

**Key Concept:** [Azure Cache for Redis](https://docs.microsoft.com/en-us/azure/azure-cache-for-redis/cache-overview)
</details>

---

### Question 23
**Scenario:** A company needs to run containerized applications at scale with automatic orchestration, scaling, and management. What Azure service should they use?

A. Azure Container Instances
B. Azure Kubernetes Service
C. Azure Functions
D. Azure Virtual Machines

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Kubernetes Service (AKS) provides managed Kubernetes for orchestrating containerized applications at scale. Container Instances is for simple container workloads without orchestration. Functions is serverless code execution. VMs require manual container management.

**Key Concept:** [Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes)
</details>

---

### Question 24
**Scenario:** What is the purpose of Azure Availability Zones?

A. Connect multiple Azure regions
B. Provide isolation from datacenter failures within a region
C. Distribute traffic across regions
D. Provide content caching at edge locations

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Availability Zones are physically separate locations within an Azure region, each with independent power, cooling, and networking. This protects against datacenter-level failures. Region pairs connect regions. Traffic Manager distributes traffic. CDN provides edge caching.

**Key Concept:** [Availability Zones](https://docs.microsoft.com/en-us/azure/availability-zones/az-overview)
</details>

---

### Question 25
**Scenario:** A company needs to deploy a web application with automatic scaling, custom domains, and SSL certificates without managing virtual machines. What should they use?

A. Azure Virtual Machines
B. Azure App Service
C. Azure Container Instances
D. Azure Batch

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure App Service is a PaaS offering for web applications with built-in auto-scaling, custom domains, SSL, and deployment slots - all without managing VMs. VMs require infrastructure management. Container Instances is for simple containers. Batch is for large-scale parallel processing.

**Key Concept:** [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/overview)
</details>

---

### Question 26
**Scenario:** A company needs to store data that must be replicated across multiple regions for disaster recovery. Which storage redundancy option should they choose?

A. Locally Redundant Storage (LRS)
B. Zone-Redundant Storage (ZRS)
C. Geo-Redundant Storage (GRS)
D. Premium Storage

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Geo-Redundant Storage (GRS) replicates data to a secondary region hundreds of miles away, providing disaster recovery capability. LRS replicates within a datacenter. ZRS replicates across availability zones in one region. Premium Storage is a performance tier.

**Key Concept:** [Azure Storage Redundancy](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy)
</details>

---

## Describe Azure Management and Governance (30-35%)

### Question 27
**Scenario:** A company wants to ensure all deployed resources have a "CostCenter" tag for billing purposes. How can they enforce this automatically?

A. Azure Resource Groups
B. Azure Policy
C. Azure Blueprints
D. Azure Management Groups

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Policy can enforce tagging requirements, either denying resources without required tags or automatically adding them. Resource Groups organize resources. Blueprints define environments. Management Groups manage subscriptions. Policy enforces compliance rules.

**Key Concept:** [Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview)
</details>

---

### Question 28
**Scenario:** A company needs to track all changes to their Azure resources for compliance auditing. What Azure feature should they use?

A. Azure Monitor
B. Azure Activity Log
C. Azure Advisor
D. Azure Service Health

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Activity Log records all control plane operations (who did what and when) on resources, essential for compliance auditing. Monitor collects performance data. Advisor provides recommendations. Service Health shows Azure service status.

**Key Concept:** [Azure Activity Log](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log)
</details>

---

### Question 29
**Scenario:** An administrator needs to get personalized recommendations for improving security, reliability, performance, and reducing costs across their Azure resources. What tool should they use?

A. Azure Monitor
B. Azure Advisor
C. Azure Service Health
D. Cost Management

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Advisor analyzes your resources and provides personalized recommendations across five categories: Cost, Security, Reliability, Operational Excellence, and Performance. Monitor collects data. Service Health shows Azure status. Cost Management focuses only on costs.

**Key Concept:** [Azure Advisor](https://docs.microsoft.com/en-us/azure/advisor/advisor-overview)
</details>

---

### Question 30
**Scenario:** A company wants to estimate their monthly Azure costs before deploying resources. What tool should they use?

A. Azure TCO Calculator
B. Azure Pricing Calculator
C. Cost Management
D. Azure Advisor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Pricing Calculator helps estimate costs for specific Azure services and configurations before deployment. TCO Calculator compares on-premises vs. cloud costs. Cost Management analyzes existing spending. Advisor provides optimization recommendations.

**Key Concept:** [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)
</details>

---

### Question 31
**Scenario:** A company needs to set spending limits and receive alerts when costs exceed a threshold. What should they use?

A. Azure Pricing Calculator
B. Azure Cost Management + Billing
C. Azure Advisor
D. Azure Monitor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Cost Management + Billing provides budgets, alerts, and cost analysis for controlling and monitoring Azure spending. Pricing Calculator estimates costs. Advisor gives recommendations. Monitor tracks performance metrics.

**Key Concept:** [Azure Cost Management](https://docs.microsoft.com/en-us/azure/cost-management-billing/cost-management-billing-overview)
</details>

---

### Question 32
**Scenario:** A company wants to grant a user permission to manage virtual machines in a specific resource group but not other resources. How should they implement this?

A. Assign Owner role at subscription level
B. Assign Virtual Machine Contributor role at resource group level
C. Create a custom role at management group level
D. Assign Contributor role at subscription level

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Assigning Virtual Machine Contributor at the resource group level follows least privilege - the user can only manage VMs in that specific resource group. Subscription-level roles grant broader access. Management group level is even broader.

**Key Concept:** [Azure RBAC](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview)
</details>

---

### Question 33
**Scenario:** Which Azure service provides threat protection across hybrid cloud workloads?

A. Azure Firewall
B. Azure DDoS Protection
C. Microsoft Defender for Cloud
D. Azure Key Vault

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Microsoft Defender for Cloud (formerly Azure Security Center) provides threat protection, security posture management, and recommendations across Azure, on-premises, and other clouds. Firewall filters traffic. DDoS protects against DDoS attacks. Key Vault stores secrets.

**Key Concept:** [Microsoft Defender for Cloud](https://docs.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)
</details>

---

### Question 34
**Scenario:** A company needs to securely store and manage secrets, keys, and certificates used by applications. What Azure service should they use?

A. Azure Storage
B. Azure Key Vault
C. Azure Active Directory
D. Azure Policy

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Key Vault securely stores and manages secrets, encryption keys, and certificates with access control and auditing. Storage is for data files. Azure AD manages identities. Policy enforces compliance rules.

**Key Concept:** [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview)
</details>

---

### Question 35
**Scenario:** An organization needs to implement multi-factor authentication (MFA) for all users accessing Azure resources. What service provides this capability?

A. Azure Key Vault
B. Microsoft Entra ID (Azure AD)
C. Azure Policy
D. Azure Monitor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Microsoft Entra ID (formerly Azure Active Directory) provides identity management including multi-factor authentication. Key Vault stores secrets. Policy enforces resource compliance. Monitor tracks performance.

**Key Concept:** [Azure MFA](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-mfa-howitworks)
</details>

---

### Question 36
**Scenario:** A company wants to standardize deployments by creating a repeatable set of Azure resources including VMs, networking, and policies. What should they use?

A. Azure Policy
B. ARM Templates
C. Azure Blueprints
D. Azure Resource Groups

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Azure Blueprints packages ARM templates, policies, role assignments, and resource groups into a versioned, repeatable definition. ARM Templates define resources but not policies or roles together. Policy enforces rules. Resource Groups organize resources.

**Key Concept:** [Azure Blueprints](https://docs.microsoft.com/en-us/azure/governance/blueprints/overview)
</details>

---

### Question 37
**Scenario:** What is the primary purpose of Azure Resource Manager (ARM)?

A. Monitor resource performance
B. Deploy and manage Azure resources
C. Calculate Azure costs
D. Provide security recommendations

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Resource Manager is the deployment and management service for Azure, handling all create, update, and delete operations for resources. It provides consistent management through templates, RBAC, and tagging. Monitor tracks performance. Pricing Calculator estimates costs. Advisor gives recommendations.

**Key Concept:** [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
</details>

---

### Question 38
**Scenario:** A company needs to restrict which Azure regions can be used for resource deployment. How can they enforce this?

A. Resource Tags
B. Azure Policy
C. Network Security Groups
D. Azure Firewall

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Policy can enforce allowed locations, denying deployments to unauthorized regions. Tags are for metadata. NSGs filter network traffic. Firewall protects network boundaries.

**Key Concept:** [Azure Policy Built-in Definitions](https://docs.microsoft.com/en-us/azure/governance/policy/samples/built-in-policies)
</details>

---

### Question 39
**Scenario:** What is the maximum number of subscriptions that can be managed under a single management group?

A. 100
B. 1,000
C. 10,000
D. Unlimited

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** A management group can contain up to 10,000 subscriptions. This hierarchical structure helps organize large enterprise environments with consistent policy application.

**Key Concept:** [Management Group Limits](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview#hierarchy-of-management-groups-and-subscriptions)
</details>

---

### Question 40
**Scenario:** Which tool allows administrators to manage Azure resources using command-line scripts that can be automated?

A. Azure Portal
B. Azure CLI
C. Azure Mobile App
D. Azure Advisor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure CLI provides command-line management of Azure resources with scripts that can be automated. Azure Portal is graphical. Mobile App provides monitoring on-the-go. Advisor provides recommendations.

**Key Concept:** [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | C | Cloud Concepts |
| 2 | B | Cloud Concepts |
| 3 | B | Cloud Concepts |
| 4 | C | Cloud Concepts |
| 5 | B | Cloud Concepts |
| 6 | B | Cloud Concepts |
| 7 | C | Cloud Concepts |
| 8 | B | Cloud Concepts |
| 9 | B | Cloud Concepts |
| 10 | C | Cloud Concepts |
| 11 | B | Cloud Concepts |
| 12 | B | Architecture & Services |
| 13 | C | Architecture & Services |
| 14 | B | Architecture & Services |
| 15 | C | Architecture & Services |
| 16 | B | Architecture & Services |
| 17 | A | Architecture & Services |
| 18 | B | Architecture & Services |
| 19 | B | Architecture & Services |
| 20 | B | Architecture & Services |
| 21 | C | Architecture & Services |
| 22 | B | Architecture & Services |
| 23 | B | Architecture & Services |
| 24 | B | Architecture & Services |
| 25 | B | Architecture & Services |
| 26 | C | Architecture & Services |
| 27 | B | Management & Governance |
| 28 | B | Management & Governance |
| 29 | B | Management & Governance |
| 30 | B | Management & Governance |
| 31 | B | Management & Governance |
| 32 | B | Management & Governance |
| 33 | C | Management & Governance |
| 34 | B | Management & Governance |
| 35 | B | Management & Governance |
| 36 | C | Management & Governance |
| 37 | B | Management & Governance |
| 38 | B | Management & Governance |
| 39 | C | Management & Governance |
| 40 | B | Management & Governance |

---

## Study Resources

- [Azure Fundamentals Learning Path](https://docs.microsoft.com/en-us/learn/paths/az-900-describe-cloud-concepts/)
- [AZ-900 Exam Skills Outline](https://docs.microsoft.com/en-us/certifications/exams/az-900)
- [Azure Documentation](https://docs.microsoft.com/en-us/azure/)
- [Microsoft Learn - Free Training](https://docs.microsoft.com/en-us/learn/)
