# Azure Administrator (AZ-104) Study Plan

## 12-Week Intensive Study Schedule

### Week 1-2: Identity and Governance Foundation
**Objective:** Master Azure AD and RBAC fundamentals

#### Week 1: Azure Active Directory
- **Day 1-2:** Azure AD users and groups
  - Create and manage users (portal, PowerShell, CLI)
  - Configure user properties and settings
  - Group types and membership management
  - Lab: Set up Azure AD tenant with users and groups

- **Day 3-4:** Administrative units and device management
  - Administrative unit creation and management
  - Device join scenarios (Azure AD joined, hybrid, registered)
  - Device settings and compliance
  - Lab: Configure administrative units and device policies

- **Day 5-7:** Bulk operations and SSPR
  - Bulk user creation, deletion, and updates
  - Self-service password reset configuration
  - Authentication methods and policies
  - Lab: Implement SSPR with multiple authentication methods

#### Week 2: Role-Based Access Control
- **Day 1-3:** RBAC fundamentals
  - Built-in roles and their capabilities
  - Role assignment at different scopes
  - Effective permissions and inheritance
  - Lab: Configure RBAC for multiple scenarios

- **Day 4-5:** Custom roles
  - Custom role creation and management
  - Actions, NotActions, and scope configuration
  - PowerShell and CLI role management
  - Lab: Create custom roles for specific scenarios

- **Day 6-7:** Governance tools
  - Azure Policy implementation
  - Resource locks configuration
  - Tagging strategies and automation
  - Lab: Implement governance policies and locks

### Week 3-4: Storage Solutions
**Objective:** Master Azure Storage services and configuration

#### Week 3: Storage Accounts and Security
- **Day 1-2:** Storage account fundamentals
  - Storage account types and performance tiers
  - Replication options (LRS, ZRS, GRS, RA-GRS)
  - Storage account creation and configuration
  - Lab: Create storage accounts with different configurations

- **Day 3-4:** Storage security
  - Access keys and SAS tokens
  - Azure AD authentication for storage
  - Network access restrictions
  - Lab: Configure storage security and access controls

- **Day 5-7:** Storage services
  - Blob storage containers and lifecycle management
  - Azure Files and File Sync
  - Queue and Table storage basics
  - Lab: Implement blob lifecycle policies and Azure File Sync

#### Week 4: Advanced Storage Features
- **Day 1-2:** Data movement and migration
  - AzCopy usage and scenarios
  - Azure Storage Explorer
  - Import/Export jobs
  - Lab: Migrate data using AzCopy and Storage Explorer

- **Day 3-4:** Storage optimization
  - Storage tiers (hot, cool, archive)
  - Blob object replication
  - Performance monitoring and optimization
  - Lab: Configure storage tiers and replication

- **Day 5-7:** Integration and automation
  - Storage account integration with other services
  - PowerShell automation for storage tasks
  - Storage monitoring and alerting
  - Lab: Automate storage management tasks

### Week 5-6: Virtual Machine Management
**Objective:** Master VM deployment, configuration, and management

#### Week 5: VM Deployment and Configuration
- **Day 1-2:** VM creation and sizing
  - VM families and sizing considerations
  - VM creation methods (portal, ARM templates, PowerShell)
  - VM configuration and extensions
  - Lab: Deploy VMs using different methods

- **Day 3-4:** ARM templates
  - ARM template structure and components
  - Template deployment and management
  - Template modification and customization
  - Lab: Create and deploy custom ARM templates

- **Day 5-7:** VM management
  - VM resizing and availability sets
  - VM disk management (OS and data disks)
  - VM backup and restore
  - Lab: Manage VM lifecycle and backups

#### Week 6: Advanced VM Features
- **Day 1-2:** VM security
  - Azure Disk Encryption
  - VM security extensions
  - Update management and patch deployment
  - Lab: Implement VM security and patch management

- **Day 3-4:** VM scale sets
  - Scale set creation and configuration
  - Auto-scaling policies
  - Load balancer integration
  - Lab: Deploy and configure VM scale sets

- **Day 5-7:** Container services
  - Azure Container Instances
  - Azure Kubernetes Service basics
  - Container image management
  - Lab: Deploy containerized applications

### Week 7-8: Virtual Networking
**Objective:** Master Azure networking and connectivity

#### Week 7: Virtual Networks and Subnets
- **Day 1-2:** VNet fundamentals
  - VNet planning and design
  - Subnet creation and configuration
  - IP addressing and allocation
  - Lab: Design and implement VNet architecture

- **Day 3-4:** VNet connectivity
  - VNet peering configuration
  - VPN Gateway setup
  - ExpressRoute basics
  - Lab: Implement VNet peering and VPN connectivity

- **Day 5-7:** Network security
  - Network Security Groups (NSGs)
  - Application Security Groups
  - Service endpoints and private endpoints
  - Lab: Configure network security and service endpoints

#### Week 8: Load Balancing and Advanced Networking
- **Day 1-2:** Azure Load Balancer
  - Load balancer types and configuration
  - Backend pools and health probes
  - Load balancing rules
  - Lab: Configure load balancing for VMs

- **Day 3-4:** Application Gateway
  - Application Gateway configuration
  - SSL termination and URL routing
  - Web Application Firewall (WAF)
  - Lab: Deploy Application Gateway with SSL

- **Day 5-7:** Network troubleshooting
  - Network Watcher tools
  - Connection troubleshooting
  - Network monitoring and diagnostics
  - Lab: Troubleshoot network connectivity issues

### Week 9-10: App Services and Containers
**Objective:** Master PaaS services and container orchestration

#### Week 9: Azure App Service
- **Day 1-2:** App Service fundamentals
  - App Service plan creation and configuration
  - Web app deployment methods
  - App Service scaling and performance
  - Lab: Deploy and scale web applications

- **Day 3-4:** App Service features
  - Custom domains and SSL certificates
  - Authentication and authorization
  - Backup and restore
  - Lab: Configure custom domains and authentication

- **Day 5-7:** Advanced App Service
  - Deployment slots and blue-green deployments
  - Application settings and connection strings
  - Monitoring and diagnostics
  - Lab: Implement deployment slots and monitoring

#### Week 10: Container Services
- **Day 1-3:** Azure Kubernetes Service
  - AKS cluster creation and configuration
  - Node pools and scaling
  - Networking and storage in AKS
  - Lab: Deploy and manage AKS cluster

- **Day 4-5:** Container management
  - Container Registry
  - Container instances
  - Container security and compliance
  - Lab: Build and deploy containerized applications

- **Day 6-7:** Integration and orchestration
  - AKS integration with other Azure services
  - CI/CD for containers
  - Helm charts and application deployment
  - Lab: Implement CI/CD for container applications

### Week 11: Monitoring and Backup
**Objective:** Master monitoring, alerting, and backup strategies

#### Monitoring and Alerting
- **Day 1-2:** Azure Monitor fundamentals
  - Metrics and logs collection
  - Log Analytics workspace configuration
  - Query language (KQL) basics
  - Lab: Configure Azure Monitor and Log Analytics

- **Day 3-4:** Alerting and notifications
  - Alert rules and action groups
  - Metric and log-based alerts
  - Notification channels
  - Lab: Create comprehensive alerting solution

- **Day 5-7:** Application monitoring
  - Application Insights configuration
  - Custom telemetry and dashboards
  - Performance monitoring
  - Lab: Implement application performance monitoring

#### Backup and Recovery
- **Day 1-2:** Azure Backup
  - Recovery Services vault configuration
  - VM backup policies and schedules
  - Backup monitoring and reporting
  - Lab: Configure VM backup and recovery

- **Day 3-4:** Site Recovery
  - Azure Site Recovery setup
  - Disaster recovery planning
  - Failover and failback procedures
  - Lab: Implement disaster recovery solution

- **Day 5-7:** Data protection strategies
  - Backup best practices
  - Cross-region replication
  - Compliance and retention policies
  - Lab: Design comprehensive data protection strategy

### Week 12: Practice and Review
**Objective:** Consolidate knowledge and practice exam scenarios

#### Practice Tests and Labs
- **Day 1-2:** Full practice test 1
  - Take comprehensive practice exam
  - Review incorrect answers
  - Identify knowledge gaps
  - Study weak areas

- **Day 3-4:** Hands-on lab scenarios
  - Multi-tier application deployment
  - Hybrid connectivity scenarios
  - Security implementation
  - Disaster recovery testing

- **Day 5-6:** Practice test 2 and final review
  - Second practice exam
  - Compare results with first test
  - Final review of key concepts
  - Create summary notes

- **Day 7:** Exam preparation
  - Final confidence building
  - Exam logistics preparation
  - Relaxation and mental preparation

## Hands-on Lab Requirements

### Essential Labs by Week
1. **Week 1-2:** Azure AD configuration, RBAC setup, governance policies
2. **Week 3-4:** Storage account management, data migration, lifecycle policies
3. **Week 5-6:** VM deployment, ARM templates, container services
4. **Week 7-8:** Network architecture, load balancing, security
5. **Week 9-10:** App services, AKS deployment, container management
6. **Week 11:** Monitoring setup, backup configuration, disaster recovery
7. **Week 12:** End-to-end scenarios, troubleshooting practice

### Real-world Scenarios
- Design and implement hybrid identity solution
- Build multi-tier web application with database
- Implement disaster recovery for critical workloads
- Configure monitoring and alerting for production environment
- Optimize costs and performance for existing resources

## Study Resources by Priority

### Primary Resources
1. **Microsoft Learn:** AZ-104 learning paths (official content)
2. **Azure Documentation:** Service-specific deep dives
3. **Hands-on Labs:** Azure portal, PowerShell, CLI practice
4. **Microsoft Official Practice Test:** Most accurate exam simulation

### Secondary Resources
1. **Cloud Academy/A Cloud Guru:** Structured courses and labs
2. **Pluralsight:** In-depth technical training
3. **YouTube:** Azure Academy, John Savill's Technical Training
4. **Books:** Official Microsoft certification guides

### Practice Resources
1. **MeasureUp:** Comprehensive practice tests
2. **Whizlabs:** Additional practice scenarios
3. **GitHub:** Azure quickstart templates and samples
4. **Microsoft Docs:** Official samples and tutorials

## Success Metrics and Milestones

### Weekly Milestones
- **Week 2:** Comfortable with identity management and RBAC
- **Week 4:** Proficient in storage configuration and management
- **Week 6:** Skilled in VM deployment and ARM templates
- **Week 8:** Expert in networking and connectivity
- **Week 10:** Competent in PaaS services and containers
- **Week 11:** Proficient in monitoring and backup
- **Week 12:** Ready for exam with 85%+ practice scores

### Key Performance Indicators
- **Practice Test Scores:** Consistently above 80%
- **Hands-on Confidence:** Complete tasks without documentation
- **Time Management:** Finish practice tests within time limits
- **Troubleshooting Skills:** Diagnose and resolve common issues

## Emergency 4-Week Plan

If you only have 4 weeks:
1. **Week 1:** Identity, RBAC, and basic storage (focus on Microsoft Learn)
2. **Week 2:** VMs, networking, and load balancing (emphasize hands-on)
3. **Week 3:** App services, monitoring, and backup (practice scenarios)
4. **Week 4:** Practice tests and review (exam preparation focus)

Focus on hands-on practice and official Microsoft Learn content for maximum efficiency.

## Final Exam Tips

### Day Before Exam
- Review summary notes and key concepts
- Avoid learning new material
- Ensure good rest and nutrition
- Verify exam logistics and location

### During Exam
- Read questions carefully, especially scenario-based ones
- Use elimination strategy for multiple choice
- Flag uncertain questions for review
- Manage time effectively (about 3 minutes per question)
- Stay calm and focused throughout

### Common Exam Scenarios
- Multi-tier application deployment and scaling
- Hybrid connectivity and identity integration
- Security implementation and compliance
- Disaster recovery and business continuity
- Cost optimization and resource management

Remember: AZ-104 is a hands-on administrator exam requiring practical experience. Prioritize lab work and real-world scenarios over theoretical study.