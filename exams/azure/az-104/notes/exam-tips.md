# AZ-104 Exam Tips and Best Practices - AZ-104

**[📖 Exam AZ-104: Microsoft Azure Administrator](https://learn.microsoft.com/en-us/certifications/exams/az-104/)** - Official exam page with requirements and skills measured

## Exam Format

**[📖 Skills Measured](https://learn.microsoft.com/en-us/certifications/resources/study-guides/az-104)** - Detailed exam objectives and study guide
- **Duration**: 180 minutes (3 hours)
- **Questions**: 40-60
- **Passing Score**: 700/1000
- **Format**: Multiple choice, multiple select, drag-drop, hot area, case studies, labs
- **Cost**: $165 USD

## Question Types

**[📖 Exam Duration and Question Types](https://learn.microsoft.com/en-us/certifications/exam-duration-question-types)** - Understand Microsoft certification exam formats

**Multiple Choice**: One correct answer
**Multiple Select**: 2+ correct answers  
**Drag and Drop**: Order steps or match items
**Hot Area**: Click regions on diagram
**Case Study**: Scenario with multiple questions
**Labs**: Perform tasks in Azure portal (simulated)

## Time Management
- Case studies: 10-15 minutes each
- Labs: 15-20 minutes each
- Regular questions: 1-2 minutes each
- Flag difficult questions
- Leave buffer for review

## Common Keywords

**Management & Governance**:
- "Prevent accidental deletion": Resource lock (CanNotDelete)
- "Enforce compliance": Azure Policy
- "Cost allocation": Tags
- "Multi-subscription management": Management groups

**Compute**:
- "High availability within datacenter": Availability Set
- "High availability across datacenters": Availability Zones
- "Auto-scale web app": VM Scale Sets or App Service
- "Zero downtime deployment": Deployment slots

**Storage**:
- "Share files between VMs": Azure Files (SMB)
- "Lowest cost long-term": Archive tier
- "Disaster recovery across regions": GRS or RA-GRS
- "Private access from VNet": Private endpoint

**Networking**:
- "Connect on-premises to Azure": VPN Gateway or ExpressRoute
- "Secure RDP/SSH without public IP": Azure Bastion
- "Layer 7 load balancing": Application Gateway
- "Global load balancing": Traffic Manager or Front Door

**Security**:
- "Least privilege": RBAC with minimum permissions
- "Delegate limited access": SAS token
- "Encrypt data at rest": Enable encryption (default for most)
- "Private connectivity to PaaS": Private endpoint

## Service Selection Patterns

### High Availability
- Availability Set: 99.95% SLA, within datacenter
- Availability Zone: 99.99% SLA, across datacenters
- Both: Best protection

### Load Balancing
- Azure Load Balancer: Layer 4, internal/public
- Application Gateway: Layer 7, WAF, SSL termination
- Front Door: Global, CDN, multi-region
- Traffic Manager: DNS-based, global

### Backup & DR
- Azure Backup: VM, Files, SQL backups
- Site Recovery: Disaster recovery, replication
- Geo-replication: Storage account level

### Monitoring
- Azure Monitor: Metrics and logs
- Application Insights: APM
- Log Analytics: Centralized logs
- Network Watcher: Network diagnostics

## PowerShell vs CLI

**When to Use**:
- Both achieve same results
- Know basic syntax for both
- Examples often show either/or

**Common Commands**:
```powershell
# PowerShell
Get-AzVM
New-AzVM
Set-AzVM
Remove-AzVM

# CLI
az vm list
az vm create
az vm update
az vm delete
```

## Hands-On Skills Required

**[📖 Azure Free Account](https://azure.microsoft.com/en-us/free/)** - Get hands-on practice with free Azure credits

**[📖 Microsoft Learn Training](https://learn.microsoft.com/en-us/certifications/azure-administrator/)** - Free official training paths for AZ-104

**Portal Tasks**:
- Create and configure VMs
- Set up VNets and subnets
- Configure NSGs
- Create storage accounts
- Set up Azure AD users
- Configure RBAC
- Create recovery services vault

**CLI/PowerShell**:
- Basic resource creation
- Start/stop VMs
- Query resource properties
- Assign roles

## Common Mistakes to Avoid

1. **Not reading full question**: Look for "NOT" or "EXCEPT"
2. **Ignoring constraints**: Cost, region, performance
3. **Over-complicating**: Start with simplest solution
4. **Missing keywords**: "Most", "least", "minimum"
5. **Confusing services**: Load Balancer vs App Gateway, Peering vs VPN

## Study Strategy

**[📖 Azure Documentation](https://learn.microsoft.com/en-us/azure/)** - Comprehensive Azure documentation and tutorials

**[📖 Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/)** - Best practices and reference architectures

### Week Before
- Practice exams (aim for 80%+)
- Review weak areas
- Hands-on practice
- Read service FAQs

### Day Before
- Light review
- Get adequate rest
- Prepare environment (for online exam)

### Exam Day
- Read carefully
- Flag and skip if unsure
- Manage time
- Review flagged questions

## Key Topics by Weight

**[📖 Exam Sandbox](https://aka.ms/examdemo)** - Practice with the Microsoft exam interface

**[📖 Azure CLI Documentation](https://learn.microsoft.com/en-us/cli/azure/)** - Command-line interface for Azure

**[📖 Azure PowerShell Documentation](https://learn.microsoft.com/en-us/powershell/azure/)** - PowerShell module for Azure

**15-20%: Identity and Governance**
- Azure AD users, groups
- RBAC (built-in and custom roles)
- Azure Policy
- Resource locks
- Tags

**15-20%: Storage**
- Storage accounts and replication
- Blob storage tiers
- Azure Files
- Backup

**20-25%: Compute**
- VMs (sizes, availability)
- VM Scale Sets
- App Service
- Containers (ACI, AKS basics)

**20-25%: Networking**
- VNets and subnets
- NSGs
- VNet peering
- VPN Gateway
- Load balancers
- Private endpoints

**10-15%: Monitoring**
- Azure Monitor
- Log Analytics
- Alerts
- Application Insights

## Final Checklist

- [ ] Completed practice exams (80%+ score)
- [ ] Hands-on with all major services
- [ ] Understand RBAC and Azure AD
- [ ] Know networking (VNet, NSG, peering, VPN)
- [ ] Storage account types and tiers
- [ ] VM availability options
- [ ] Monitoring and backup
- [ ] Cost management basics
- [ ] PowerShell/CLI basic commands

**Remember**: AZ-104 is hands-on. Focus on practical skills, not just theory!
