# Microsoft Azure Administrator (AZ-104)

## Exam Overview

The Azure Administrator certification validates skills and knowledge to implement, manage, and monitor an organization's Microsoft Azure environment. Azure Administrators manage cloud services that span storage, security, networking, and compute cloud capabilities, with a deep understanding of each service across the full IT lifecycle.

**Exam Details:**
- **Exam Code:** AZ-104
- **Duration:** 180 minutes
- **Number of Questions:** 40-60 questions
- **Passing Score:** 700 out of 1000
- **Question Types:** Multiple choice, multiple select, drag and drop, hot area, case studies, labs
- **Cost:** $165 USD
- **Prerequisites:** None (6+ months hands-on experience recommended)

## Exam Domains

### 1. Manage Azure Identities and Governance (15-20%)
- **Manage Azure Active Directory (Azure AD) Objects**
  - Create users and groups
  - Create administrative units
  - Manage user and group properties
  - Manage device settings and device identity
  - Perform bulk user updates
  - Manage guest accounts
  - Configure Azure AD join
  - Configure self-service password reset

- **Manage Role-Based Access Control (RBAC)**
  - Create a custom role
  - Provide access to Azure resources by assigning roles
  - Interpret access assignments
  - Manage multiple directories

- **Manage Subscriptions and Governance**
  - Configure Azure policies
  - Configure resource locks
  - Apply and manage tags on resources
  - Manage resource groups
  - Manage subscriptions
  - Manage costs
  - Configure management groups

### 2. Implement and Manage Storage (15-20%)
- **Secure Storage**
  - Configure network access to storage accounts
  - Create and configure storage accounts
  - Generate shared access signature (SAS) tokens
  - Manage access keys
  - Configure Azure AD authentication for a storage account
  - Configure access to Azure Files

- **Manage Storage**
  - Export from Azure job
  - Import into Azure job
  - Install and use Azure Storage Explorer
  - Copy data by using AZCopy
  - Implement Azure Storage replication
  - Configure blob object replication

- **Configure Azure Files and Azure Blob Storage**
  - Create an Azure file share
  - Create and configure Azure File Sync service
  - Configure Azure blob storage
  - Configure storage tiers
  - Configure blob lifecycle management

### 3. Deploy and Manage Azure Compute Resources (20-25%)
- **Automate Deployment of VMs Using ARM Templates**
  - Modify ARM templates
  - Configure VHD template
  - Deploy from template
  - Save a deployment as an ARM template
  - Deploy virtual machine extensions

- **Configure VMs**
  - Configure Azure Disk Encryption
  - Move VMs from one resource group to another
  - Manage VM sizes
  - Add data disks
  - Configure networking
  - Redeploy VMs

- **Create and Configure Containers**
  - Configure sizing and scaling for Azure Container Instances
  - Configure container groups for Azure Container Instances
  - Configure storage for Azure Kubernetes Service (AKS)
  - Configure scaling for AKS
  - Configure network connections for AKS
  - Upgrade an AKS cluster

- **Create and Configure an Azure App Service**
  - Create an App Service plan
  - Configure scaling settings in an App Service plan
  - Create an App Service
  - Secure an App Service
  - Configure custom domain names
  - Configure backup for an App Service
  - Configure networking settings
  - Configure deployment settings

### 4. Configure and Manage Virtual Networking (20-25%)
- **Implement and Manage Virtual Networking**
  - Create and configure virtual networks
  - Create and configure subnets
  - Create and configure virtual network peering
  - Configure private and public IP addresses
  - Configure user-defined network routes
  - Implement subnets
  - Configure endpoints on subnets

- **Secure Access to Virtual Networks**
  - Create security rules
  - Associate a network security group (NSG) to a subnet or network interface
  - Evaluate effective security rules
  - Implement Azure Firewall
  - Implement Azure Bastion

- **Configure Load Balancing**
  - Configure Azure Application Gateway
  - Configure an internal or public load balancer
  - Troubleshoot load balancing

- **Monitor and Troubleshoot Virtual Networking**
  - Monitor on-premises connectivity
  - Configure and use Azure Monitor for Networks
  - Use Azure Network Watcher
  - Troubleshoot external networking
  - Troubleshoot virtual network connectivity

### 5. Monitor and Back Up Azure Resources (10-15%)
- **Monitor Resources by Using Azure Monitor**
  - Configure and interpret metrics
  - Configure Azure Monitor logs
  - Query and analyze logs
  - Set up alerts and actions
  - Configure Application Insights

- **Implement Backup and Recovery**
  - Create a Recovery Services vault
  - Create a Backup vault
  - Create and configure backup policy
  - Perform backup and restore operations
  - Perform site-to-site recovery by using Azure Site Recovery
  - Configure and review backup reports

## Skills Measured in Detail

### Identity and Access Management
- **Azure AD Management:** User lifecycle, group management, device registration
- **RBAC Implementation:** Custom roles, role assignments, access reviews
- **Governance:** Policies, resource locks, cost management, compliance

### Storage Solutions
- **Storage Security:** Network restrictions, encryption, access control
- **Data Management:** Replication, lifecycle policies, archival strategies
- **File Services:** Azure Files, File Sync, SMB protocol management

### Compute Management
- **VM Operations:** Deployment, configuration, scaling, maintenance
- **Container Services:** ACI configuration, AKS management, container orchestration
- **App Services:** Web app deployment, scaling, security, custom domains

### Networking Infrastructure
- **Network Design:** VNet planning, subnetting, IP addressing
- **Security Implementation:** NSGs, Azure Firewall, Bastion, private endpoints
- **Load Balancing:** Application Gateway, Load Balancer, Traffic Manager
- **Troubleshooting:** Network diagnostics, monitoring, connectivity issues

### Monitoring and Recovery
- **Monitoring Setup:** Metrics, logs, alerts, dashboards
- **Backup Strategy:** Vault configuration, policy management, recovery testing
- **Disaster Recovery:** Site Recovery, cross-region replication

## Study Tips

### Recommended Study Timeline: 8-12 weeks
1. **Weeks 1-2:** Azure fundamentals review and identity management
2. **Weeks 3-4:** Storage solutions and compute resources
3. **Weeks 5-6:** Virtual networking and security
4. **Weeks 7-8:** Monitoring, backup, and governance
5. **Weeks 9-10:** ARM templates and automation
6. **Weeks 11-12:** Practice exams and hands-on labs

### Key Study Resources
- **Microsoft Learn Learning Paths:**
  - AZ-104: Prerequisites for Azure administrators
  - AZ-104: Manage identities and governance in Azure
  - AZ-104: Implement and manage storage in Azure
  - AZ-104: Deploy and manage Azure compute resources
  - AZ-104: Configure and manage virtual networks for Azure administrators
  - AZ-104: Monitor and back up Azure resources

### Hands-on Practice Requirements
- **Minimum 6 months** of hands-on experience with Azure
- **Regular use** of Azure portal, PowerShell, CLI, and ARM templates
- **Real-world scenarios** including multi-tier applications and hybrid connectivity
- **Troubleshooting experience** with Azure services and networking

### Exam Strategy
- **Hands-on Focus:** This exam includes performance-based questions and labs
- **Time Management:** Allocate time for case studies and lab scenarios
- **PowerShell/CLI:** Know common commands for each service area
- **ARM Templates:** Understand template structure and deployment
- **Troubleshooting:** Practice systematic problem-solving approaches

### Common Gotchas
- **RBAC Scope:** Understand inheritance and effective permissions
- **Network Security:** NSG rules evaluation and precedence
- **Storage Access:** Differences between access keys, SAS, and Azure AD auth
- **VM Sizing:** Know when to use different VM families and sizes
- **Load Balancer Types:** Application Gateway vs Load Balancer scenarios

## Hands-on Lab Areas

### Identity and Governance Labs
- Create and manage Azure AD users and groups
- Configure RBAC with custom roles
- Implement Azure Policy and resource locks
- Set up cost management and budgets

### Storage Labs
- Configure storage accounts with different access tiers
- Implement Azure File Sync
- Set up blob lifecycle management
- Configure storage security and networking

### Compute Labs
- Deploy VMs using ARM templates
- Configure VM scale sets with auto-scaling
- Set up AKS cluster with networking
- Deploy and configure App Services

### Networking Labs
- Design and implement virtual network architecture
- Configure VNet peering and hybrid connectivity
- Implement Azure Firewall and NSG rules
- Set up Application Gateway with SSL termination

### Monitoring Labs
- Configure Azure Monitor for infrastructure
- Set up log analytics and queries
- Create alerts and action groups
- Implement backup and recovery scenarios

## 📚 Comprehensive Study Resources

**👉 [Complete Azure Study Resources Guide](../../../.templates/resources-azure.md)**

For detailed information on courses, practice tests, hands-on labs, communities, and more, see our comprehensive Azure study resources guide which includes:
- Official Microsoft Learn paths (FREE)
- Top-rated video courses with specific instructors
- Practice test platforms with pricing and comparisons
- Hands-on lab environments and free tier details
- Community forums and study groups
- Essential tools and Azure CLI resources
- Pro tips and budget-friendly study strategies

### Quick Links (AZ-104 Specific)
- **[AZ-104 Official Exam Page](https://learn.microsoft.com/en-us/certifications/exams/az-104/)** - Registration and exam details
- **[Microsoft Learn - AZ-104 Learning Path](https://learn.microsoft.com/en-us/certifications/azure-administrator/)** - FREE official study path
- **[Azure Administrator Documentation](https://docs.microsoft.com/en-us/azure/)** - Service documentation
- **[Azure PowerShell Documentation](https://docs.microsoft.com/en-us/powershell/azure/)** - PowerShell reference
- **[Azure CLI Documentation](https://docs.microsoft.com/en-us/cli/azure/)** - CLI commands
- **[ARM Template Reference](https://docs.microsoft.com/en-us/azure/templates/)** - Infrastructure as Code
- **[Azure Free Account](https://azure.microsoft.com/en-us/free/)** - $200 free credit for hands-on practice

## Prerequisites and Next Steps

### Prerequisites
- Basic understanding of Azure fundamentals (AZ-900 recommended)
- Familiarity with networking concepts
- Basic PowerShell or CLI experience
- Understanding of virtualization concepts

### Career Path
- **Next Certifications:** AZ-305 (Solutions Architect Expert), AZ-500 (Security Engineer)
- **Role Focus:** Azure Administrator, Cloud Administrator, Infrastructure Engineer
- **Skills Development:** DevOps practices, security implementation, cost optimization

Remember: AZ-104 is a hands-on administrator role exam. Practical experience with Azure services is essential for success. Focus on understanding how services work together and practice real-world scenarios.