# Cost Management - AZ-104

**[📖 Cost Management and Billing Documentation](https://learn.microsoft.com/en-us/azure/cost-management-billing/)** - Monitor and optimize Azure spending

## Azure Cost Management

**[📖 What is Cost Management?](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/overview-cost-management)** - Understand your Azure costs and usage
- Analyze spending
- Create budgets
- Export cost data
- Recommendations for optimization

**Cost Analysis**:

**[📖 Quickstart: Explore and Analyze Costs](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/quick-acm-cost-analysis)** - Start analyzing your Azure costs

- View by service, resource group, tag
- Filter by time range
- Forecast future costs
- Visualize with charts

## Budgets

**[📖 Create and Manage Budgets](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets)** - Set spending alerts and controls

- Set spending limits
- Alert when threshold reached (50%, 75%, 90%, 100%)
- Action groups for notifications
- Monthly, quarterly, annual budgets

## Cost Allocation
- Tag resources
- Show-back and charge-back
- Filter cost reports by tags
- Tag policies to enforce tagging

## Azure Advisor - Cost

**[📖 Reduce Costs with Azure Advisor](https://learn.microsoft.com/en-us/azure/advisor/advisor-cost-recommendations)** - Cost optimization recommendations

- Recommendations to reduce costs
- Resize/shutdown underutilized VMs
- Reserved Instance recommendations
- Delete unattached disks
- Configure auto-shutdown

## Pricing Calculator

**[📖 Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)** - Estimate costs for Azure services

- Estimate Azure costs before deployment
- Configure services and usage
- Export estimates
- Compare scenarios

## Total Cost of Ownership (TCO) Calculator

**[📖 TCO Calculator](https://azure.microsoft.com/en-us/pricing/tco/calculator/)** - Calculate savings by migrating to Azure

- Compare on-premises vs Azure
- Include all costs (power, cooling, hardware, labor)
- Migration scenarios

## Cost Optimization Strategies

**Compute**:
- Reserved Instances (1 or 3 years, up to 72% savings)
- Azure Hybrid Benefit (Windows/SQL license reuse)
- Spot VMs (up to 90% savings)
- Auto-shutdown for dev/test
- Right-size VMs

**Storage**:
- Choose appropriate tier (Hot/Cool/Archive)
- Lifecycle management
- Delete old snapshots
- Use managed disks

**Networking**:
- Minimize data transfer between regions
- Use CDN to reduce bandwidth
- Reserved capacity for ExpressRoute

## Azure Hybrid Benefit

**[📖 Azure Hybrid Benefit](https://azure.microsoft.com/en-us/pricing/hybrid-benefit/)** - Save money with existing licenses

- Use existing Windows Server/SQL Server licenses
- Software Assurance required
- Up to 80% savings on VMs
- Up to 55% savings on SQL Database

## Reserved Instances

**[📖 Save with Azure Reservations](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/save-compute-costs-reservations)** - Reduce costs with upfront commitments

- 1-year or 3-year commitment
- Pay upfront, monthly, or no upfront
- Significant discounts (up to 72%)
- Exchange or cancel with conditions
- Auto-renewal option

## Exam Tips
- Cost Management + Billing: Analyze and control spending
- Budgets: Proactive cost alerts
- Tags: Essential for cost allocation
- Azure Advisor: Free cost optimization recommendations
- Reserved Instances: Long-term savings
- Azure Hybrid Benefit: License reuse
- Pricing Calculator: Estimate before deployment
- Right-sizing: Match resources to workload
- Lifecycle policies: Auto-tier storage
