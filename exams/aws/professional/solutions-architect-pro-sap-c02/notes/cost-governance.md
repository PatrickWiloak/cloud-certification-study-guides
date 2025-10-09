# Cost & Governance

## Overview

Enterprise cost management and governance on AWS requires implementing financial controls, optimization strategies, organizational policies, and compliance frameworks across complex multi-account environments. At the professional level, you must design cost-efficient architectures, implement chargeback/showback models, enforce governance policies, and establish cost visibility and accountability while meeting regulatory and compliance requirements.

Professional architects must balance cost optimization with performance, implement FinOps practices, design resource tagging strategies, leverage AWS cost management tools, and establish governance frameworks using AWS Organizations, SCPs, and automated compliance checking.

## Key Concepts

### Cost Management

**AWS Cost Explorer**
- **Cost Analysis**: Visualize and analyze spending patterns
- **Forecasting**: Predict future costs based on historical data
- **Filtering**: By service, account, tag, region, instance type, purchase option
- **Grouping**: Group by service, account, tag, region, usage type
- **Rightsizing Recommendations**: Identify underutilized resources
- **Savings Plans Recommendations**: Based on usage patterns
- **Reserved Instance Recommendations**: For predictable workloads
- **Cost Anomaly Detection**: ML-based detection of unusual spending

**AWS Cost and Usage Report (CUR)**
- **Most Granular**: Line-item details of all charges
- **Delivery**: S3 bucket (CSV, Parquet)
- **Update Frequency**: Daily, hourly
- **Integration**: Athena, QuickSight, Redshift for analysis
- **Tagging**: All cost allocation tags included
- **Use Cases**: Custom cost analytics, chargeback, compliance reporting

**AWS Budgets**
- **Budget Types**:
  - Cost budgets (actual cost vs. planned)
  - Usage budgets (service usage)
  - RI/Savings Plans utilization and coverage budgets
- **Time Period**: Monthly, quarterly, annual, or custom
- **Alerts**: Email or SNS when threshold exceeded (50%, 80%, 100%)
- **Actions**: Automated actions (apply SCPs, stop instances) when budget exceeded
- **Forecasting**: Alert on forecasted spend

**AWS Cost Anomaly Detection**
- **ML-Based**: Detect unusual spending patterns
- **Cost Monitors**: Service-level, account-level, or cost allocation tag
- **Alerts**: Email or SNS when anomaly detected
- **Root Cause**: Drill down to specific services or resources
- **Use Cases**: Catch misconfigurations, runaway costs, security incidents

**AWS Compute Optimizer**
- **Rightsizing**: EC2, EBS, Lambda, ECS on Fargate, Auto Scaling groups
- **Recommendations**: Based on CloudWatch metrics (CPU, memory, network)
- **Savings**: Estimate monthly savings
- **Performance Risk**: Low, medium, high risk of performance degradation
- **Organization-Wide**: Aggregate recommendations across accounts

**Savings Plans**
- **Compute Savings Plans**: EC2, Fargate, Lambda (up to 66% savings)
- **EC2 Instance Savings Plans**: Specific instance family (up to 72% savings)
- **Commitment**: 1 or 3 years, $/hour
- **Payment**: All upfront, partial upfront, no upfront
- **Flexibility**: Change instance size, OS, tenancy, region (Compute Savings Plans)

**Reserved Instances**
- **Standard RI**: Highest savings (up to 72%), cannot change instance type
- **Convertible RI**: Can change instance family, OS, tenancy (up to 54% savings)
- **Commitment**: 1 or 3 years
- **Payment**: All upfront, partial upfront, no upfront
- **Modification**: Can modify AZ, instance size (within same family), network type
- **Marketplace**: Sell unused RIs

**Spot Instances**
- **Savings**: Up to 90% off On-Demand
- **Interruption**: 2-minute warning when AWS reclaims capacity
- **Use Cases**: Fault-tolerant workloads (batch, big data, CI/CD, rendering)
- **Spot Fleet**: Diversify across instance types and AZs for availability
- **Spot Blocks**: Reserve for 1-6 hours (deprecated, use EC2 Capacity Reservations)

### Governance

**AWS Organizations**
- **Organization Units (OUs)**: Hierarchical grouping of accounts
- **Management Account**: Billing and organization management (no workloads)
- **Member Accounts**: Workload accounts
- **Consolidated Billing**: Single payment method, volume discounts
- **Service Control Policies (SCPs)**: Preventive guardrails
- **Organizational CloudTrail**: Enable CloudTrail across all accounts
- **Tag Policies**: Standardize tags across accounts

**Service Control Policies (SCPs)**
- **Preventive Controls**: Deny/allow actions at organization, OU, or account level
- **Inheritance**: Child OUs and accounts inherit parent SCPs
- **Allow List vs. Deny List**: Default allow (deny specific), or deny all (allow specific)
- **Use Cases**: Prevent region usage, enforce encryption, deny root user, require MFA
- **Limitations**: Does not affect management account root user

**AWS Control Tower**
- **Landing Zone**: Multi-account AWS environment with best practices
- **Account Factory**: Automated account provisioning
- **Guardrails**:
  - **Preventive**: Implemented using SCPs (deny actions)
  - **Detective**: Implemented using AWS Config rules (detect non-compliance)
- **Dashboard**: Compliance status across organization
- **Account Vending**: Self-service account creation with governance

**AWS Resource Access Manager (RAM)**
- **Share Resources**: VPC subnets, Transit Gateway, Route 53 Resolver rules, License Manager configs
- **Cross-Account**: Share with specific accounts or OUs
- **No Duplication**: Single resource, multiple account access
- **Use Cases**: Shared VPCs, Transit Gateway sharing, license sharing

**Tagging Strategy**
- **Cost Allocation Tags**: Track costs by project, environment, department, owner
- **Automation Tags**: Used by scripts and automation (backup schedule, auto-shutdown)
- **Security Tags**: Data classification (public, confidential, restricted)
- **Compliance Tags**: Regulatory requirements (HIPAA, PCI-DSS)
- **Best Practices**:
  - Consistent naming convention (Environment, Project, Owner, CostCenter)
  - Enforce with Tag Policies in AWS Organizations
  - Activate cost allocation tags in Billing console
  - Tag everything (EC2, S3, RDS, Lambda, etc.)

**AWS Service Catalog**
- **Product Portfolio**: Pre-approved CloudFormation templates
- **Self-Service**: Users launch approved resources without IAM permissions
- **Constraints**: Launch constraints (IAM role), template constraints (allowed values)
- **TagOptions**: Automatically apply tags to launched products
- **Versioning**: Manage product versions
- **Use Cases**: Standardized deployments, governance, cost control

### Compliance and Audit

**AWS Config**
- **Resource Inventory**: Track all resources and configuration changes
- **Config Rules**: Evaluate resource configurations against desired state
  - **Managed Rules**: AWS-provided (S3 encryption, public access, approved AMIs)
  - **Custom Rules**: Lambda functions for custom logic
- **Conformance Packs**: Collection of Config rules for compliance frameworks (PCI-DSS, HIPAA, CIS)
- **Remediation**: Automated remediation using Systems Manager Automation or Lambda
- **Multi-Account Aggregation**: Aggregate compliance data across accounts

**AWS Audit Manager**
- **Continuous Auditing**: Automated evidence collection for compliance
- **Frameworks**: Pre-built for SOC 2, PCI-DSS, GDPR, HIPAA, FedRAMP, NIST
- **Assessments**: Create assessments for specific compliance requirements
- **Evidence**: Collect from CloudTrail, Config, Security Hub, Control Tower
- **Delegation**: Delegate review to subject matter experts
- **Reports**: Generate audit-ready reports

**AWS Artifact**
- **Compliance Reports**: Download AWS compliance reports (SOC 1/2/3, PCI, ISO)
- **Agreements**: Sign Business Associate Agreements (BAA) for HIPAA
- **Self-Service**: On-demand access to compliance documentation
- **Use Cases**: Provide to auditors, compliance teams, procurement

**AWS License Manager**
- **Track Licenses**: BYOL licenses for software (Windows Server, SQL Server, Oracle)
- **Enforce Limits**: Prevent over-licensing
- **Discovery**: Identify license usage across EC2, on-premises
- **Rules**: Define license rules and attach to AMIs
- **Host Resource Groups**: Dedicated Hosts with license affinity

**AWS Systems Manager**
- **Patch Manager**: Automated OS patching with compliance reporting
- **Compliance**: Check configuration compliance (patch level, software installed)
- **Inventory**: Collect metadata from instances (software, network, OS)
- **State Manager**: Define and maintain desired state
- **Change Calendar**: Block changes during blackout windows

### FinOps Practices

**Chargeback and Showback**
- **Chargeback**: Billing business units for actual AWS usage (enforced cost allocation)
- **Showback**: Reporting costs without billing (visibility only)
- **Implementation**:
  - Cost allocation tags (Department, Project, Owner)
  - Separate AWS accounts per business unit (consolidated billing)
  - Cost and Usage Report → Athena → QuickSight (dashboards per BU)
  - Third-party tools (CloudHealth, Apptio Cloudability)

**Cost Optimization Strategies**
- **Rightsizing**: Match instance size to actual usage (Compute Optimizer)
- **Scheduling**: Turn off non-production resources outside business hours
- **Reserved Capacity**: RIs or Savings Plans for predictable workloads
- **Spot Instances**: Fault-tolerant workloads (batch, ML training)
- **Storage Optimization**: S3 Intelligent-Tiering, Glacier, lifecycle policies
- **Data Transfer**: VPC endpoints, CloudFront, reduce cross-region transfer
- **Idle Resources**: Identify and delete unused (unattached EBS, old snapshots, idle load balancers)

**Cost Allocation and Tagging**
- **Activate Tags**: In Billing console (user-defined tags)
- **AWS-Generated Tags**: aws:createdBy, aws:cloudformation:stack-name
- **Tag Policies**: Enforce standardized tags across organization
- **Tag Editor**: Bulk tag resources across regions and accounts
- **Resource Groups**: Group resources by tags for management

## AWS Services Reference

### Core Services

**AWS Cost Explorer**
- Visualize and analyze costs
- Rightsizing recommendations
- Savings Plans and RI recommendations
- Forecasting

**AWS Budgets**
- Set cost and usage budgets
- Alerts and automated actions
- RI/Savings Plans utilization tracking

**AWS Cost and Usage Report (CUR)**
- Most granular cost data
- Delivered to S3
- Integration with Athena, QuickSight

**AWS Organizations**
- Multi-account management
- Consolidated billing
- Service Control Policies
- Organizational CloudTrail

### Supporting Services

**AWS Control Tower**
- Landing zone setup
- Account Factory
- Preventive and detective guardrails
- Compliance dashboard

**AWS Config**
- Resource inventory
- Configuration compliance
- Conformance packs
- Automated remediation

**AWS Compute Optimizer**
- EC2, EBS, Lambda rightsizing
- Savings estimates
- Performance risk assessment

**AWS Service Catalog**
- Pre-approved templates
- Self-service provisioning
- Governance and cost control

**AWS Audit Manager**
- Continuous compliance auditing
- Evidence collection
- Framework-based assessments

**AWS License Manager**
- License tracking and enforcement
- BYOL license management
- Dedicated Host management

**AWS Systems Manager**
- Patch compliance
- Inventory and configuration
- State management
- Change calendar

## Architecture Patterns

### Pattern 1: Multi-Account Cost Allocation

**Use Case**
- Enterprise with 100 AWS accounts across 10 business units
- Chargeback model for cost accountability
- Visibility into spending by BU, project, environment

**Implementation Approach**
1. **Account Structure**:
   - Management account (billing only)
   - Accounts grouped by OU (BU-Prod, BU-NonProd)
   - Consolidated billing for volume discounts
2. **Tagging Strategy**:
   - Required tags: Environment, Project, Owner, CostCenter, Department
   - Tag Policies in AWS Organizations to enforce
   - Activate cost allocation tags in billing console
3. **Cost Reporting**:
   - AWS Cost and Usage Report (CUR) → S3 (Parquet format)
   - Athena database and tables (Glue Crawler for schema)
   - QuickSight dashboards per business unit
   - Daily/weekly cost reports via email (Lambda + SES)
4. **Chargeback**:
   - Monthly cost reports per CostCenter tag
   - Finance system integration for billing
   - Variance analysis (planned vs. actual)
5. **Governance**:
   - AWS Budgets per account/BU (alerts at 80%, 100%, 120%)
   - Cost Anomaly Detection to catch runaway costs
   - SCPs to prevent expensive operations (prevent large instance types in non-prod)

**Pros/Cons**
- Pros: Cost accountability, visibility, accurate chargeback, informed decision-making
- Cons: Tagging discipline required, initial setup complexity, ongoing tag governance

### Pattern 2: Cost Optimization Pipeline

**Use Case**
- Reduce AWS spend by 30% through systematic optimization
- Automated recommendations and actions
- Continuous cost monitoring

**Implementation Approach**
1. **Compute Optimization**:
   - AWS Compute Optimizer for rightsizing recommendations
   - Lambda function (weekly) to process recommendations
   - Create Systems Manager Change Calendar approvals for non-prod
   - Automated resizing during maintenance window
2. **Storage Optimization**:
   - S3 Storage Lens for bucket-level insights
   - S3 Intelligent-Tiering for automatic tier transitions
   - Lambda to identify and delete unattached EBS volumes, old snapshots
   - RDS: Change storage type from gp2 to gp3 (20% savings)
3. **Idle Resource Cleanup**:
   - Lambda (daily) to identify idle resources:
     - Load balancers with no targets
     - EIPs not attached to instances
     - Old AMIs and snapshots
     - Unused security groups
   - SNS notification to owners (from Owner tag)
   - Auto-delete after 30 days if no response
4. **Reserved Capacity**:
   - Cost Explorer RI/Savings Plans recommendations
   - Purchase recommendations based on 1-year commitment
   - Monitor utilization (alert if <80%)
5. **Spot Instances**:
   - Identify fault-tolerant workloads (batch, dev/test)
   - Convert to Spot Fleet or Auto Scaling with mixed instances policy
   - Target 70% Spot, 30% On-Demand for reliability
6. **Monitoring**:
   - CloudWatch dashboard with cost metrics
   - Budgets with forecasted spend alerts
   - Cost Anomaly Detection for unusual patterns
   - Weekly cost optimization report to leadership

**Results**
- Compute: 25% savings from rightsizing and RIs
- Storage: 40% savings from lifecycle policies and gp3 migration
- Idle resources: 10% of total spend reclaimed
- Spot: 60% savings on eligible workloads

### Pattern 3: Governance with Control Tower

**Use Case**
- Set up secure, compliant landing zone for 200+ AWS accounts
- Enforce preventive and detective guardrails
- Automated account provisioning
- Compliance monitoring and reporting

**Implementation Approach**
1. **Landing Zone Setup**:
   - AWS Control Tower in management account
   - Foundational OUs: Security, Infrastructure, Workloads, Sandbox
   - Security OU accounts: Log Archive, Security Tooling, Audit
2. **Guardrails**:
   - **Preventive** (SCPs):
     - Deny root user access (except emergency)
     - Deny operations outside allowed regions (us-east-1, us-west-2)
     - Deny disabling CloudTrail, Config, GuardDuty
     - Deny creating unencrypted resources (S3, EBS, RDS)
   - **Detective** (Config rules):
     - S3 buckets must have versioning enabled
     - RDS instances must be encrypted
     - Public AMIs not allowed
     - IAM users must have MFA
3. **Account Factory**:
   - Self-service account creation via Service Catalog
   - Baseline configuration (VPC, IAM roles, CloudTrail, Config)
   - Automatic tagging (Owner, CostCenter, Environment)
   - SSO access provisioning with AWS IAM Identity Center
4. **Compliance Monitoring**:
   - Control Tower dashboard for guardrail compliance
   - Security Hub for aggregated security findings
   - Config Conformance Packs for regulatory frameworks (PCI-DSS, HIPAA)
   - Audit Manager for continuous evidence collection
5. **Notifications**:
   - SNS topic for non-compliant resources
   - Lambda for automated remediation or approval workflow
   - Weekly compliance report to security team

**Pros/Cons**
- Pros: Automated governance, rapid account provisioning, compliance visibility, consistent baseline
- Cons: Initial setup time, limited customization, region availability, OU structure changes are complex

### Pattern 4: Compliance Automation with Config

**Use Case**
- Healthcare company with HIPAA compliance requirements
- Continuous compliance monitoring across 50 accounts
- Automated remediation for non-compliant resources
- Audit-ready reports

**Implementation Approach**
1. **AWS Config Setup**:
   - Enable Config in all accounts and regions
   - Multi-account aggregator in security account
   - S3 bucket for Config snapshots (encrypted, versioned, Object Lock)
2. **Conformance Pack**:
   - Deploy HIPAA conformance pack (AWS-managed)
   - Custom rules for organization-specific policies
   - Examples:
     - S3 buckets must have encryption and versioning
     - RDS instances must have automated backups
     - EBS volumes must be encrypted
     - No public IPs on EC2 instances
     - CloudTrail enabled in all regions
3. **Automated Remediation**:
   - Systems Manager Automation documents for remediation
   - Examples:
     - Enable S3 versioning
     - Enable EBS encryption by default
     - Revoke overly permissive security group rules
     - Enable CloudTrail if disabled
   - Manual approval required for high-impact changes
4. **Alerting**:
   - EventBridge rule for Config compliance changes
   - SNS notification to compliance team for critical non-compliance
   - Daily summary report of compliance status
5. **Audit**:
   - Audit Manager with HIPAA framework
   - Automated evidence collection from Config, CloudTrail, Security Hub
   - Quarterly compliance reports for auditors
   - Audit-ready evidence with timestamps and responsible parties

**Compliance Metrics**
- Overall compliance score (90%+ target)
- Time to remediate non-compliance (4 hours target)
- Number of non-compliant resources by type
- Trend analysis (improving vs. declining compliance)

### Pattern 5: Serverless Cost Optimization

**Use Case**
- Serverless application with Lambda, API Gateway, DynamoDB
- Optimize costs while maintaining performance
- Right-size and optimize resource allocation

**Implementation Approach**
1. **Lambda Optimization**:
   - Lambda Power Tuning tool to find optimal memory setting
   - Analyze duration vs. memory (cost increases with memory, but faster = cheaper overall)
   - Provisioned Concurrency only for latency-critical functions (most use on-demand)
   - Reduce package size (tree-shaking, minimize dependencies)
   - Monitor unused functions (delete or disable)
2. **DynamoDB Optimization**:
   - On-demand vs. Provisioned capacity analysis
     - Predictable traffic → Provisioned (cheaper)
     - Unpredictable or spiky → On-demand
   - Enable Auto Scaling for Provisioned (target 70% utilization)
   - DynamoDB Accelerator (DAX) for read-heavy workloads (cache, reduce RCUs)
   - Lifecycle management with TTL for automatic data expiration
   - Archive old data to S3 with DynamoDB export
3. **API Gateway Optimization**:
   - HTTP API instead of REST API (60% cost savings, if features sufficient)
   - Caching at API Gateway (reduce backend invocations)
   - CloudFront in front of API Gateway (edge caching, reduce API calls)
   - Throttling to prevent runaway costs from DDoS or loops
4. **Data Transfer**:
   - VPC endpoints for Lambda to S3/DynamoDB (avoid NAT Gateway costs)
   - Same-region architecture (avoid cross-region data transfer)
   - CloudFront for static content delivery (reduce S3 egress)
5. **Monitoring**:
   - CloudWatch metrics: Lambda duration, invocations, errors, throttles
   - DynamoDB: Consumed capacity, throttled requests
   - Cost Explorer: Daily granularity for Lambda and DynamoDB
   - Budgets with alerts at 80% of forecasted monthly spend

**Cost Savings Example**
- Lambda: 30% savings from memory optimization (1024 MB → 512 MB)
- DynamoDB: 50% savings from on-demand → provisioned with Auto Scaling
- API Gateway: 60% savings from REST → HTTP API
- Data transfer: 80% savings from VPC endpoints (eliminate NAT Gateway)

### Pattern 6: Reserved Capacity Strategy

**Use Case**
- Predictable workloads running 24/7
- Maximize savings with Reserved Instances and Savings Plans
- Balance flexibility with savings

**Implementation Approach**
1. **Analysis**:
   - Cost Explorer: Last 30 days of usage
   - Identify steady-state workloads (databases, always-on web servers)
   - Forecast future usage growth (10% annual growth)
2. **Coverage Strategy**:
   - **Year 1**: 70% coverage with 1-year Savings Plans (balance savings with flexibility)
   - **Year 2-3**: Increase to 85% coverage with 3-year Savings Plans (higher savings)
   - **On-Demand**: 15-30% for burst, new workloads, short-term projects
3. **Savings Plans vs. Reserved Instances**:
   - **Compute Savings Plans**: Flexible (EC2, Fargate, Lambda), 66% savings, apply automatically
   - **EC2 Instance Savings Plans**: Specific instance family, 72% savings, less flexible
   - **Reserved Instances**: Legacy option, less flexible than Savings Plans
   - Recommendation: Compute Savings Plans for most workloads
4. **Payment Option**:
   - All Upfront: Highest savings (5-10% more than No Upfront)
   - Partial Upfront: Balance of upfront and monthly payments
   - No Upfront: No upfront cost, lowest savings
   - Recommendation: All Upfront if cash available, Partial otherwise
5. **Monitoring**:
   - AWS Budgets: Track RI/Savings Plans utilization (alert if <80%)
   - Cost Explorer: Utilization and coverage reports
   - Monthly review of recommendations (adjust commitment up/down)
6. **Governance**:
   - Centralized purchasing in management account (benefit all linked accounts)
   - Approval workflow for large commitments (>$100K)
   - Finance team sign-off on 3-year commitments

**Savings Results**
- 70% Savings Plans coverage → 46% average savings (66% * 0.70)
- 85% Savings Plans coverage → 56% average savings (66% * 0.85)
- Payback period: 6-8 months for 3-year commitment

## Best Practices

### Enterprise-Level Recommendations

**Cost Visibility**
- Enable Cost and Usage Report for granular data
- Activate cost allocation tags
- Use Cost Explorer for trend analysis and forecasting
- Create dashboards for stakeholders (executives, engineering, finance)

**Tagging Discipline**
- Define tagging policy (required tags, naming convention)
- Enforce with Tag Policies in AWS Organizations
- Automate tagging (CloudFormation, Terraform with default tags)
- Regular tag compliance audits with Tag Editor

**Budget Management**
- Set budgets at account, service, and tag levels
- Multiple thresholds (50%, 80%, 100%, 120%)
- Forecasted budget alerts for proactive action
- Budget actions for automated cost controls

**Continuous Optimization**
- Weekly review of Compute Optimizer recommendations
- Monthly cost optimization review meeting
- Quarterly RI/Savings Plans analysis and purchase
- Automated idle resource cleanup

**Governance at Scale**
- AWS Organizations with structured OUs
- Service Control Policies for preventive controls
- AWS Config for detective controls and compliance
- Centralized logging and monitoring

### Security Considerations

**Billing Access**
- Separate IAM policies for billing access
- Read-only access for most users
- Write access only for finance team
- MFA required for payment method changes

**Cost Anomaly Alerts**
- Set up Cost Anomaly Detection
- Alert on unusual spending (potential compromise)
- Integrate with security incident response

**Prevent Accidental Spend**
- SCPs to deny expensive instance types in non-prod
- SCPs to deny operations outside allowed regions
- Require approval for large resource launches
- AWS Budgets with automated actions (apply deny SCP when budget exceeded)

### Performance Tuning

**Cost vs. Performance Trade-offs**
- Use Compute Optimizer performance risk assessment
- Test rightsizing in non-prod before prod
- Gradual rollout of changes (10% → 50% → 100%)
- Monitor application performance during changes

**Reserved Capacity Optimization**
- Start with 1-year commitment, increase to 3-year as confidence grows
- Monitor utilization (should be >80%)
- Adjust commitment quarterly based on actual usage

**Storage Lifecycle**
- S3 Intelligent-Tiering for unknown access patterns
- Lifecycle policies for predictable patterns (logs, backups)
- Delete old snapshots and AMIs (retention policy)
- Use Glacier for long-term retention (7+ years)

## Common Scenarios

### Scenario 1: Startup to Enterprise Cost Management

**Context**: Company growing from $10K/month to $500K/month AWS spend

**Phase 1: Startup ($10K/month)**
- Single AWS account
- No tagging, no cost allocation
- Monthly billing review
- **Actions**: Start tagging (Environment, Owner), enable Cost Explorer

**Phase 2: Growth ($50K/month)**
- 5 AWS accounts (prod, dev, staging, data, security)
- Basic tagging in place
- Cost visibility by account
- **Actions**:
  - Implement AWS Organizations with consolidated billing
  - Cost allocation tags (Department, Project, Environment)
  - AWS Budgets with alerts
  - Identify savings opportunities (rightsizing, RIs for databases)

**Phase 3: Scale ($200K/month)**
- 20 AWS accounts across 3 business units
- Chargeback model for BUs
- Dedicated FinOps engineer
- **Actions**:
  - Comprehensive tagging policy and enforcement (Tag Policies)
  - Cost and Usage Report → Athena → QuickSight dashboards
  - Compute Optimizer recommendations and automated rightsizing
  - Savings Plans for 50% of steady-state compute
  - Cost Anomaly Detection

**Phase 4: Enterprise ($500K/month)**
- 100+ AWS accounts across 10 business units
- Mature FinOps practice with dedicated team
- Cost optimization as part of engineering culture
- **Actions**:
  - AWS Control Tower for governance
  - Service Catalog for approved, cost-optimized templates
  - Monthly cost optimization reviews with engineering teams
  - Savings Plans coverage >70%
  - Automated cost optimization workflows
  - Executive dashboards with cost per customer, cost per transaction

**Key Metrics**
- Cost per customer
- Cost per transaction
- Unit economics (gross margin %)
- Month-over-month cost growth vs. revenue growth

### Scenario 2: Multi-BU Chargeback Implementation

**Context**: Enterprise with 10 business units, $2M/month AWS spend, implementing chargeback

**Implementation**
1. **Tagging Strategy**:
   - Required tags: Department, CostCenter, Project, Environment, Owner
   - Tag Policies to enforce across all accounts
   - Backfill tags on existing resources (Tag Editor, AWS CLI scripts)
2. **Account Structure**:
   - Separate accounts per BU-Environment (BU1-Prod, BU1-Dev, BU2-Prod, etc.)
   - Consolidated billing for volume discounts
   - OUs per BU for policy application
3. **Cost Allocation**:
   - Cost and Usage Report with hourly granularity
   - Athena for cost queries by CostCenter tag
   - QuickSight dashboards per BU (filtered by CostCenter)
   - Shared services (networking, security) allocated proportionally
4. **Chargeback Process**:
   - Monthly cost reports generated (Lambda + Athena + QuickSight)
   - Sent to BU finance leads for review
   - Integrated into corporate finance system
   - Variance analysis (planned vs. actual)
5. **Governance**:
   - Budgets per CostCenter (alert BU when 80% consumed)
   - Monthly cost review meetings with each BU
   - Shared cost optimization best practices

**Challenges**
- Tagging discipline (some resources missing tags)
- Shared resources allocation (Transit Gateway, Direct Connect)
- Cost of shared services (security, networking) allocation model
- Education on cloud cost concepts (data transfer, IOPS, API calls)

**Results**
- 100% cost visibility and accountability
- 15% cost reduction in first year (BUs optimizing their own spend)
- Faster decision-making (BUs have cost data)

### Scenario 3: Compliance for Regulated Industry

**Context**: Financial services company with SOC 2, PCI-DSS, and regulatory requirements

**Governance Implementation**
1. **Landing Zone**:
   - AWS Control Tower for multi-account management
   - Foundational OUs: Security, Compliance, Production, Non-Production
   - Account Factory for standardized account creation
2. **Guardrails**:
   - **Preventive**:
     - Deny regions outside US (data residency)
     - Deny creation of unencrypted resources
     - Deny disabling CloudTrail, Config, GuardDuty
     - Deny root user access
   - **Detective**:
     - S3 buckets must have versioning and MFA Delete
     - RDS must have automated backups and encryption
     - Security groups must not allow 0.0.0.0/0 on sensitive ports
     - CloudTrail must be enabled in all regions
3. **Compliance Monitoring**:
   - AWS Config Conformance Packs (SOC 2, PCI-DSS)
   - Multi-account Config aggregator
   - Security Hub with CIS AWS Foundations Benchmark
   - Automated remediation for non-compliant resources
4. **Audit**:
   - AWS Audit Manager for continuous evidence collection
   - Quarterly compliance reports for auditors
   - CloudTrail logs to immutable S3 bucket (Object Lock, 7-year retention)
   - Audit access to read-only role in Audit account
5. **Change Management**:
   - All infrastructure changes via CloudFormation (IaC)
   - Peer review in pull requests
   - Automated testing in non-prod
   - Change approval board for production
   - Systems Manager Change Calendar for blackout windows

**Audit Results**
- Zero non-compliance findings in last audit
- 99.5% compliance score across all guardrails
- Audit preparation time reduced from 3 months to 2 weeks

### Scenario 4: Cost Optimization for ML Workloads

**Context**: Machine learning platform with $100K/month spend on training and inference

**Cost Breakdown**
- Training: $60K (60%): EC2 p3.16xlarge, p4d.24xlarge GPU instances
- Inference: $30K (30%): SageMaker endpoints, Lambda
- Storage: $10K (10%): S3, EFS

**Optimization Strategy**
1. **Training Optimization**:
   - **Spot Instances**: 70% of training on Spot (70% cost savings)
   - **Checkpointing**: Save state every 15 minutes (resume on Spot interruption)
   - **Right-Sizing**: Analyze GPU utilization with CloudWatch
     - P3.16xlarge under-utilized → P3.8xlarge (50% cost savings)
   - **Scheduling**: Run training during off-peak hours for lower Spot prices
   - **Savings Plans**: 30% on-demand for critical training (66% savings)
2. **Inference Optimization**:
   - **Multi-Model Endpoints**: Deploy multiple models on single endpoint (reduce endpoint count by 80%)
   - **Auto Scaling**: Scale endpoints based on invocation count
   - **Serverless Inference**: Use for infrequent inference (<10 requests/min)
   - **Inference Instance Right-Sizing**: ml.m5.xlarge → ml.m5.large (50% savings)
   - **Batch Transform**: For offline batch inference (cheaper than real-time endpoints)
3. **Storage Optimization**:
   - **S3 Intelligent-Tiering**: Training datasets automatically tier to IA or Archive
   - **Lifecycle Policies**: Delete intermediate training results after 30 days
   - **EFS Lifecycle**: Infrequent Access tier for cold training data
4. **Monitoring**:
   - CloudWatch metrics: GPU utilization, inference latency, endpoint invocations
   - Cost Explorer: Daily granularity for SageMaker, EC2
   - Budgets: Alert when 80% of monthly forecast

**Results**
- Training costs: $60K → $25K (58% reduction)
- Inference costs: $30K → $12K (60% reduction)
- Storage costs: $10K → $6K (40% reduction)
- Total: $100K → $43K (57% reduction, $684K annual savings)

### Scenario 5: SaaS Cost Per Tenant

**Context**: B2B SaaS with 10,000 customers, need to track cost per tenant

**Architecture**
- Multi-tenant: Shared infrastructure with TenantId in all tables
- Services: API Gateway, Lambda, DynamoDB, S3, RDS

**Cost Allocation Strategy**
1. **Tagging**:
   - TenantId tag on all resources (where possible)
   - Tier tag (Free, Standard, Premium, Enterprise)
2. **Custom Metrics**:
   - Lambda: Embedded Metric Format with TenantId dimension
   - CloudWatch custom metrics: Requests per tenant, storage per tenant
3. **Application-Level Tracking**:
   - Track API calls, compute time, storage per tenant in application
   - Write to DynamoDB or CloudWatch Logs
   - Daily aggregation with Lambda or Athena
4. **Cost Modeling**:
   - **Direct Costs**: Resources tagged with TenantId (dedicated databases for Enterprise)
   - **Shared Costs**: Allocated proportionally based on usage metrics
     - Lambda invocations per tenant / total invocations * total Lambda cost
     - DynamoDB RCUs per tenant / total RCUs * total DynamoDB cost
5. **Reporting**:
   - Monthly cost per tenant report
   - Identify unprofitable customers (cost > revenue)
   - Optimize pricing tiers based on actual costs
6. **Optimization**:
   - High-cost tenants: Investigate usage patterns, optimize, or adjust pricing
   - Low-margin tenants: Move to higher tier or implement usage limits

**Insights**
- Top 10% of customers account for 60% of costs
- Free tier customers cost $5/month average (inform conversion strategy)
- Premium tier has 40% gross margin (healthy)
- Enterprise tier has 65% gross margin (custom pricing effective)

## AWS CLI Examples

```bash
# Cost Explorer - Get cost and usage
aws ce get-cost-and-usage \
  --time-period Start=2025-01-01,End=2025-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost UsageQuantity \
  --group-by Type=DIMENSION,Key=SERVICE \
  --query 'ResultsByTime[0].Groups[].[Keys[0],Metrics.BlendedCost.Amount]' \
  --output table

# Cost Explorer - Get rightsizing recommendations
aws ce get-rightsizing-recommendation \
  --service "AmazonEC2" \
  --page-size 10 \
  --query 'RightsizingRecommendations[*].[CurrentInstance.ResourceId,RightsizingType,ModifyRecommendationDetail.TargetInstances[0].ResourceDetails.EC2ResourceDetails.InstanceType,ModifyRecommendationDetail.TargetInstances[0].EstimatedMonthlySavings]'

# Cost Explorer - Get Savings Plans recommendations
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP \
  --term-in-years ONE_YEAR \
  --payment-option NO_UPFRONT \
  --lookback-period-in-days SIXTY_DAYS

# Budgets - Create budget
aws budgets create-budget \
  --account-id 123456789012 \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json

# budget.json:
# {
#   "BudgetName": "Monthly-Production-Budget",
#   "BudgetLimit": {"Amount": "10000", "Unit": "USD"},
#   "TimeUnit": "MONTHLY",
#   "BudgetType": "COST",
#   "CostFilters": {"TagKeyValue": ["Environment$Production"]}
# }

# Budgets - Create budget action
aws budgets create-budget-action \
  --account-id 123456789012 \
  --budget-name Monthly-Production-Budget \
  --notification-type ACTUAL \
  --action-type APPLY_SCP_POLICY \
  --action-threshold ActionThresholdValue=100,ActionThresholdType=PERCENTAGE \
  --definition SspActionDefinition={PolicyId=p-12345,TargetIds=[ou-abc-123]} \
  --approval-model AUTOMATIC \
  --subscribers file://subscribers.json \
  --execution-role-arn arn:aws:iam::123456789012:role/BudgetActionRole

# Cost Anomaly Detection - Create monitor
aws ce create-anomaly-monitor \
  --anomaly-monitor MonitorName=ServiceMonitor,MonitorType=DIMENSIONAL,MonitorDimension=SERVICE

# Cost Anomaly Detection - Create subscription
aws ce create-anomaly-subscription \
  --anomaly-subscription SubscriptionName=DailyAlerts,MonitorArnList=arn:aws:ce::123456789012:anomalymonitor/abc-123,Subscribers=[{Address=team@example.com,Type=EMAIL,Status=CONFIRMED}],Threshold={Amount=100,Type=PERCENTAGE},Frequency=DAILY

# Compute Optimizer - Get EC2 recommendations
aws compute-optimizer get-ec2-instance-recommendations \
  --filters name=Finding,values=Underprovisioned,Overprovisioned \
  --query 'instanceRecommendations[*].[instanceArn,finding,currentInstanceType,recommendationOptions[0].instanceType,recommendationOptions[0].estimatedMonthlySavings.value]' \
  --output table

# Compute Optimizer - Get Lambda recommendations
aws compute-optimizer get-lambda-function-recommendations \
  --query 'lambdaFunctionRecommendations[*].[functionArn,finding,currentMemorySize,memorySizeRecommendationOptions[0].memorySize,memorySizeRecommendationOptions[0].estimatedMonthlySavings.value]' \
  --output table

# Organizations - Create organization
aws organizations create-organization --feature-set ALL

# Organizations - Create OU
aws organizations create-organizational-unit \
  --parent-id r-abc123 \
  --name Production

# Organizations - Create account
aws organizations create-account \
  --email prod-account@example.com \
  --account-name "Production Account" \
  --role-name OrganizationAccountAccessRole

# Organizations - Attach SCP
aws organizations attach-policy \
  --policy-id p-12345678 \
  --target-id ou-abc-xyz123

# Organizations - Create SCP
aws organizations create-policy \
  --name DenyRegions \
  --description "Deny operations outside allowed regions" \
  --type SERVICE_CONTROL_POLICY \
  --content file://deny-regions-scp.json

# deny-regions-scp.json:
# {
#   "Version": "2012-10-17",
#   "Statement": [{
#     "Effect": "Deny",
#     "Action": "*",
#     "Resource": "*",
#     "Condition": {
#       "StringNotEquals": {
#         "aws:RequestedRegion": ["us-east-1", "us-west-2"]
#       }
#     }
#   }]
# }

# Organizations - Create tag policy
aws organizations create-policy \
  --name RequiredTags \
  --description "Require Environment and Owner tags" \
  --type TAG_POLICY \
  --content file://tag-policy.json

# Config - Start configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/ConfigRole
aws configservice start-configuration-recorder --configuration-recorder-name default

# Config - Deploy conformance pack
aws configservice put-conformance-pack \
  --conformance-pack-name HIPAA-Compliance \
  --template-s3-uri s3://config-templates/hipaa-pack.yaml \
  --delivery-s3-bucket config-delivery-bucket

# Config - Get compliance summary
aws configservice describe-compliance-by-config-rule \
  --compliance-types NON_COMPLIANT \
  --query 'ComplianceByConfigRules[*].[ConfigRuleName,Compliance.ComplianceType]' \
  --output table

# Service Catalog - Create portfolio
aws servicecatalog create-portfolio \
  --display-name "Approved-Infrastructure" \
  --description "Pre-approved infrastructure templates" \
  --provider-name "Cloud Platform Team"

# Service Catalog - Create product
aws servicecatalog create-product \
  --name "Web-Application-Stack" \
  --owner "Platform Team" \
  --product-type CLOUD_FORMATION_TEMPLATE \
  --provisioning-artifact-parameters file://product-artifact.json

# Resource Groups - Create resource group
aws resource-groups create-group \
  --name Production-WebServers \
  --resource-query '{"Type":"TAG_FILTERS_1_0","Query":"{\"ResourceTypeFilters\":[\"AWS::EC2::Instance\"],\"TagFilters\":[{\"Key\":\"Environment\",\"Values\":[\"Production\"]},{\"Key\":\"Role\",\"Values\":[\"WebServer\"]}]}"}' \
  --tags Environment=Production,ManagedBy=Platform-Team

# Tag Editor - Get resources by tag
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Environment,Values=Production \
  --resource-type-filters ec2:instance \
  --query 'ResourceTagMappingList[*].[ResourceARN,Tags]' \
  --output table

# Tag Editor - Tag resources
aws resourcegroupstaggingapi tag-resources \
  --resource-arn-list arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0 \
  --tags Project=WebApp,Owner=john.doe@example.com
```

## Study Tips

### SAP-C02 Exam Focus Areas

**High-Priority Topics**
- Multi-account cost allocation with AWS Organizations
- Tagging strategies for chargeback/showback
- Cost optimization techniques (rightsizing, RIs, Savings Plans, Spot)
- AWS Cost Explorer and Cost and Usage Report
- Service Control Policies (SCPs) for governance
- AWS Control Tower for landing zones
- AWS Config for compliance monitoring
- AWS Budgets for cost control
- Reserved Instances vs. Savings Plans decision-making

**Scenario-Based Questions**
- Design multi-account governance architecture
- Implement chargeback model for business units
- Optimize costs for specific workload types
- Enforce compliance requirements with SCPs and Config
- Choose between RIs and Savings Plans
- Set up budgets and automated actions
- Tag strategy for cost allocation

**Common Decision Points**
- **RIs vs. Savings Plans**: Flexibility vs. maximum savings
- **Compute vs. EC2 Instance Savings Plans**: Broad vs. specific
- **1-year vs. 3-year**: Flexibility vs. savings
- **On-Demand vs. Reserved vs. Spot**: Workload characteristics
- **Multi-account vs. single account**: Scale and isolation vs. simplicity
- **SCPs vs. IAM policies**: Organization-wide vs. account-specific
- **Preventive vs. Detective controls**: Prevent vs. detect and remediate

### Key Differences from SAA-C03

**SAA-C03 Knowledge**
- Basic AWS Organizations
- Simple budgets and Cost Explorer
- Reserved Instances basics
- Basic tagging

**Additional SAP-C02 Requirements**
- Advanced Organizations (SCPs, tag policies, multi-account strategies)
- AWS Control Tower for landing zones
- Comprehensive cost allocation and chargeback models
- AWS Cost and Usage Report with Athena analysis
- Savings Plans vs. RIs decision matrix
- Cost optimization at scale (Compute Optimizer, automated actions)
- AWS Config Conformance Packs for compliance
- AWS Audit Manager for continuous auditing
- Service Catalog for governance
- AWS License Manager for BYOL
- FinOps practices and cost culture

### Complex Scenarios to Master

**Multi-Account Governance**
- Landing zone with Control Tower
- OU structure design
- SCP inheritance and policy evaluation
- Cross-account resource sharing with RAM
- Centralized logging and security

**Cost Allocation and Chargeback**
- Tagging strategy and enforcement
- Cost and Usage Report analysis
- Chargeback model design
- Shared cost allocation
- Unit economics calculation

**Compliance Automation**
- Config Conformance Packs
- Automated remediation
- Audit Manager for evidence collection
- Regulatory frameworks (HIPAA, PCI-DSS, SOC 2)

**Cost Optimization**
- Rightsizing analysis and implementation
- Reserved capacity strategy (RIs vs. Savings Plans)
- Spot Instance integration
- Storage lifecycle optimization
- Idle resource identification and cleanup

**Serverless Cost Optimization**
- Lambda memory and duration optimization
- DynamoDB capacity modes (on-demand vs. provisioned)
- API Gateway HTTP vs. REST API
- VPC endpoints to avoid NAT costs

### Practice Lab Recommendations

1. **Multi-Account Setup**
   - Create AWS Organization
   - Create OUs and accounts
   - Apply SCPs to deny regions
   - Enable consolidated billing
   - View cost by account

2. **Tagging and Cost Allocation**
   - Define tagging policy
   - Tag resources with Tag Editor
   - Activate cost allocation tags
   - View costs grouped by tags in Cost Explorer

3. **Budgets and Alerts**
   - Create budget for account
   - Set thresholds (50%, 80%, 100%)
   - Configure SNS notifications
   - Test budget exceeded scenario

4. **Rightsizing**
   - Review Compute Optimizer recommendations
   - Right-size an over-provisioned instance
   - Monitor performance after change
   - Calculate cost savings

5. **Savings Plans**
   - Analyze usage in Cost Explorer
   - Review Savings Plans recommendations
   - Purchase Savings Plans (use free tier or minimal commitment)
   - Monitor utilization

6. **AWS Config Compliance**
   - Enable AWS Config
   - Deploy conformance pack (CIS Benchmark)
   - Create custom Config rule
   - View compliance dashboard
   - Set up automated remediation

## Additional Resources

### AWS Whitepapers
- AWS Cost Optimization Pillar - Well-Architected Framework
- Organizing Your AWS Environment Using Multiple Accounts
- AWS Tagging Best Practices
- Introduction to AWS Economics
- AWS Control Tower User Guide
- AWS FinOps with AWS

### Documentation Links
- AWS Cost Management: https://docs.aws.amazon.com/cost-management/
- AWS Organizations: https://docs.aws.amazon.com/organizations/
- AWS Control Tower: https://docs.aws.amazon.com/controltower/
- AWS Config: https://docs.aws.amazon.com/config/
- AWS Budgets: https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html

### Video Resources
- AWS re:Invent Cost Optimization sessions
- AWS FinOps Best Practices
- Multi-Account Governance with Organizations
- AWS Control Tower Deep Dive
