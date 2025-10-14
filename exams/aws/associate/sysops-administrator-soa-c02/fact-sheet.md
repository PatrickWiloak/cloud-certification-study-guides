# AWS SysOps Administrator Associate (SOA-C02) - Fact Sheet

## Quick Reference

**Exam Code:** SOA-C02
**Duration:** 180 minutes (3 hours)
**Questions:** 65 questions
**Passing Score:** 720/1000
**Cost:** $150 USD
**Validity:** 3 years
**Delivery:** Pearson VUE
**Difficulty:** ⭐⭐⭐⭐ (Hands-on focus)

## Exam Domain Breakdown

| Domain | Weight | Key Focus |
|--------|--------|-----------|
| Monitoring, Logging & Remediation | 20% | CloudWatch, EventBridge, automation |
| Reliability & Business Continuity | 16% | Backups, HA, DR, Auto Scaling |
| Deployment, Provisioning & Automation | 18% | CloudFormation, Systems Manager, automation |
| Security & Compliance | 16% | IAM, encryption, patching, compliance |
| Networking & Content Delivery | 18% | VPC, Route 53, CloudFront, ELB |
| Cost & Performance Optimization | 12% | Cost Explorer, rightsizing, monitoring |

## Key Services by Domain

### Monitoring, Logging & Remediation (20%)

**CloudWatch**
- Metrics: Standard (5-min), detailed (1-min), custom (1-sec high-resolution)
- Alarms: Metric-based with SNS, Auto Scaling, EC2 actions
- Logs: Aggregation, Insights queries, metric filters, retention
- Dashboards: Cross-account, cross-region visualization
- Synthetics: Canary monitoring
- [📖 CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)

**CloudWatch Agent**
- Collects system-level metrics (memory, disk, processes)
- Collects logs from instances
- Configuration via SSM Parameter Store
- Unified agent (replaces old CloudWatch Logs and Monitoring agents)
- **[📖 CloudWatch Agent Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)** - Installation and configuration
- **[📖 Agent Configuration](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)** - Configuration file reference
- **[📖 Metrics Collected](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html)** - Available metrics

**EventBridge**
- Event-driven automation
- AWS service events + custom applications
- Schedule-based rules (cron expressions)
- Targets: Lambda, Step Functions, SQS, SNS, EC2 actions
- [📖 EventBridge Documentation](https://docs.aws.amazon.com/eventbridge/)
- **[📖 Event Patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)** - Pattern matching
- **[📖 Schedule Expressions](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html)** - Cron and rate expressions
- **[📖 EventBridge Targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)** - Available targets

**Systems Manager Automation**
- Runbooks for common operational tasks
- AWS-managed automation documents (100+)
- Custom automation with YAML/JSON
- Approval steps for sensitive operations
- Change Calendar for maintenance windows
- **[📖 SSM Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html)** - Runbook overview
- **[📖 Automation Documents](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-documents.html)** - Document reference
- **[📖 Change Calendar](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-change-calendar.html)** - Maintenance windows

**CloudTrail**
- API auditing for compliance
- Management + data events
- Insights for anomaly detection
- Organization trails
- [📖 CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)
- **[📖 CloudTrail Events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html#cloudtrail-concepts-events)** - Event types
- **[📖 CloudTrail Insights](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-insights-events-with-cloudtrail.html)** - Anomaly detection

### Reliability & Business Continuity (16%)

**High Availability**
- Multi-AZ deployments
- Elastic Load Balancing (ALB, NLB, GWLB)
- Auto Scaling groups with health checks
- RDS Multi-AZ automatic failover
- Aurora with read replicas
- **[📖 HA Architecture](https://docs.aws.amazon.com/whitepapers/latest/real-time-communication-on-aws/high-availability-and-scalability-on-aws.html)** - Best practices
- **[📖 RDS Multi-AZ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)** - Database HA
- **[📖 Aurora Read Replicas](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html)** - Replication

**Disaster Recovery**
- **Backup & Restore:** Lowest cost, RTO hours-days
- **Pilot Light:** Core systems running, RTO 10s of minutes
- **Warm Standby:** Scaled-down environment, RTO minutes
- **Multi-Region Active-Active:** Highest cost, RTO seconds
- **[📖 DR Strategies](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)** - Comprehensive guide

**AWS Backup**
- Centralized backup across 35+ services
- Backup plans with lifecycle rules
- Cross-region and cross-account copies
- Backup vault with encryption
- [📖 Backup Documentation](https://docs.aws.amazon.com/aws-backup/)

**Auto Scaling**
- Dynamic scaling: Target tracking, step, simple
- Scheduled scaling
- Predictive scaling (ML-based)
- Health checks: EC2, ELB
- Lifecycle hooks for custom actions
- [📖 Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)
- **[📖 Dynamic Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html)** - Scaling types
- **[📖 Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html)** - Custom actions
- **[📖 Predictive Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-predictive-scaling.html)** - ML-based scaling

### Deployment, Provisioning & Automation (18%)

**CloudFormation**
- Infrastructure as Code (JSON/YAML)
- Stacks for resource management
- StackSets for multi-account/region
- Change sets to preview updates
- Drift detection
- [📖 CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- **[📖 StackSets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html)** - Multi-account deployment
- **[📖 Change Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)** - Preview changes
- **[📖 Drift Detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)** - Configuration drift

**Systems Manager**
- **Session Manager:** Secure shell without SSH keys
- **Run Command:** Execute at scale
- **Patch Manager:** Automated patching with maintenance windows
- **Parameter Store:** Configuration management
- **State Manager:** Enforce desired configuration
- **Inventory:** Collect metadata from instances
- [📖 Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)
- **[📖 Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)** - Secure shell access
- **[📖 Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/execute-remote-commands.html)** - Remote execution
- **[📖 Patch Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html)** - OS patching
- **[📖 Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)** - Configuration data
- **[📖 State Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state.html)** - Desired state
- **[📖 Inventory](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-inventory.html)** - Instance metadata

**Elastic Beanstalk**
- PaaS for applications
- Deployment options: All-at-once, rolling, rolling with batch, immutable, blue/green
- Configuration with .ebextensions
- [📖 Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- **[📖 Deployment Policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html)** - Deployment strategies
- **[📖 .ebextensions](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html)** - Configuration files

**OpsWorks**
- Chef and Puppet managed configuration
- Stacks, layers, instances
- Lifecycle events with recipes
- **[📖 AWS OpsWorks](https://docs.aws.amazon.com/opsworks/latest/userguide/)** - Configuration management
- **[📖 OpsWorks Stacks](https://docs.aws.amazon.com/opsworks/latest/userguide/workingstacks.html)** - Stack configuration

### Security & Compliance (16%)

**IAM**
- Users, groups, roles, policies
- Least privilege principle
- MFA enforcement
- Access Analyzer for permission analysis
- [📖 IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- **[📖 IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)** - Policy syntax
- **[📖 IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)** - Role delegation
- **[📖 Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)** - Permission analysis

**Encryption**
- KMS for key management
- EBS encryption (default per region)
- S3 encryption (SSE-S3, SSE-KMS, SSE-C)
- RDS/Aurora encryption at rest
- In-transit via TLS/SSL
- **[📖 AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/)** - Key management
- **[📖 EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)** - Volume encryption
- **[📖 S3 Encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)** - Encryption options

**Patch Management**
- Systems Manager Patch Manager
- Patch baselines (OS-specific)
- Maintenance windows for scheduling
- Patch compliance reporting
- **[📖 Patch Manager Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html)** - Patching workflow
- **[📖 Patch Baselines](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-patch-baselines.html)** - Baseline rules
- **[📖 Maintenance Windows](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-maintenance.html)** - Scheduling

**AWS Config**
- Resource configuration tracking
- Compliance rules (managed + custom)
- Remediation actions
- [📖 Config Documentation](https://docs.aws.amazon.com/config/)
- **[📖 Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config.html)** - Compliance evaluation
- **[📖 Remediation Actions](https://docs.aws.amazon.com/config/latest/developerguide/remediation.html)** - Automated fixes

### Networking & Content Delivery (18%)

**VPC**
- Subnets (public/private)
- Route tables
- Internet Gateway, NAT Gateway
- Security Groups (stateful)
- NACLs (stateless)
- VPC Flow Logs
- [📖 VPC Documentation](https://docs.aws.amazon.com/vpc/)
- **[📖 VPC Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)** - Subnet configuration
- **[📖 Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)** - Instance firewall
- **[📖 Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)** - Subnet firewall
- **[📖 VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)** - Traffic logging

**Route 53**
- DNS service
- Routing policies: Simple, weighted, latency, failover, geolocation, geoproximity, multivalue
- Health checks with failover
- [📖 Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- **[📖 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)** - Policy types
- **[📖 Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)** - Failover configuration

**CloudFront**
- Global CDN
- Origin: S3, ALB, custom HTTP
- Edge caching with TTL
- Signed URLs/cookies for private content
- [📖 CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- **[📖 Cache Behavior](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValuesCacheBehavior)** - Caching configuration
- **[📖 Signed URLs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html)** - Private content

**Elastic Load Balancing**
- **ALB:** Layer 7, HTTP/HTTPS, host/path routing
- **NLB:** Layer 4, TCP/UDP, ultra-low latency, static IPs
- Health checks
- Target groups
- Cross-zone load balancing
- [📖 ELB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- **[📖 ALB Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)** - Application Load Balancer
- **[📖 NLB Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/)** - Network Load Balancer
- **[📖 Health Checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)** - Target health

### Cost & Performance Optimization (12%)

**Cost Management**
- Cost Explorer: Analyze spending
- Budgets: Set alerts
- Savings Plans: Up to 72% savings
- Reserved Instances: 1 or 3 year
- Spot Instances: Up to 90% savings
- [📖 Cost Management Documentation](https://docs.aws.amazon.com/cost-management/)
- **[📖 Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)** - Cost analysis
- **[📖 AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)** - Budget alerts
- **[📖 Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/)** - Flexible pricing

**Rightsizing**
- Compute Optimizer recommendations
- CloudWatch metrics analysis
- AWS Trusted Advisor checks
- [📖 Compute Optimizer Documentation](https://docs.aws.amazon.com/compute-optimizer/)
- **[📖 Trusted Advisor](https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor.html)** - Best practice checks
- **[📖 Rightsizing Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html)** - Instance resizing

**S3 Optimization**
- Storage classes: Standard, IA, One Zone-IA, Glacier, Deep Archive
- Intelligent-Tiering for automatic optimization
- Lifecycle policies
- Request metrics for optimization
- **[📖 S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)** - Class comparison
- **[📖 S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)** - Lifecycle rules
- **[📖 S3 Intelligent-Tiering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering.html)** - Automatic optimization

**Performance Monitoring**
- CloudWatch metrics for bottlenecks
- X-Ray for distributed tracing
- VPC Flow Logs for network analysis
- EBS IOPS and throughput optimization
- **[📖 X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/)** - Application tracing
- **[📖 EBS Performance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)** - Volume types and performance

## Common SysOps Tasks

### Instance Management
- Launch instances with user data
- Configure CloudWatch Agent for detailed monitoring
- Implement auto-recovery for instance failures
- Schedule instance start/stop with Lambda + EventBridge
- Apply patches with Systems Manager

### Backup Strategy
- Automated EBS snapshots with lifecycle policies
- S3 versioning and lifecycle rules
- RDS automated backups and manual snapshots
- AWS Backup for centralized management
- Cross-region backup copies for DR

### Security Hardening
- Enable EBS encryption by default
- Enforce S3 encryption with bucket policies
- Implement least privilege IAM policies
- Enable MFA for privileged users
- Regular security audits with Config and Security Hub

### Network Troubleshooting
- VPC Flow Logs to analyze traffic
- Reachability Analyzer for path testing
- Security Group and NACL rule verification
- Route table configuration check
- DNS resolution with Route 53 query logging

### Monitoring & Alerting
- CloudWatch alarms for critical metrics
- SNS notifications for alerts
- EventBridge rules for automated responses
- CloudWatch Logs Insights for log analysis
- Custom metrics for application monitoring

## Exam Tips

### Hands-On Focus
- SOA-C02 includes lab-based questions
- Must demonstrate actual AWS console/CLI skills
- Practice in real AWS environment essential

### Common Scenarios
- Troubleshoot failing Auto Scaling groups
- Restore from backups after data loss
- Optimize costs for EC2 and storage
- Configure CloudWatch alarms and dashboards
- Implement automated patching
- Resolve network connectivity issues
- Set up cross-region DR

### Question Keywords
- **"Automate"** → Systems Manager, EventBridge, Lambda
- **"Monitor"** → CloudWatch, X-Ray, VPC Flow Logs
- **"Cost-effective"** → Savings Plans, Spot, rightsizing, S3 lifecycle
- **"High availability"** → Multi-AZ, Auto Scaling, ELB
- **"Secure"** → Encryption, IAM roles, least privilege
- **"Troubleshoot"** → CloudWatch Logs, VPC Flow Logs, CloudTrail

## Essential Documentation

### Core Resources
- [📖 SysOps Administrator Learning Path](https://aws.amazon.com/training/learn-about/sysops/)
- [📖 AWS Systems Manager User Guide](https://docs.aws.amazon.com/systems-manager/)
- [📖 Monitoring Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring_best_practices.html)
- [📖 Well-Architected Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)

### Hands-On Labs
- [🧪 AWS Systems Manager Workshops](https://workshops.aws/categories/Systems%20Manager)
- [🧪 Operational Excellence Labs](https://wellarchitectedlabs.com/operational-excellence/)

## Final Checklist

### Knowledge
- [ ] Configure CloudWatch monitoring and alarms
- [ ] Implement Auto Scaling with health checks
- [ ] Design backup and DR strategies
- [ ] Troubleshoot networking issues
- [ ] Optimize costs using various AWS tools
- [ ] Automate operations with Systems Manager
- [ ] Implement security best practices
- [ ] Deploy with CloudFormation

### Skills
- [ ] AWS Console proficiency
- [ ] AWS CLI experience
- [ ] Systems administration experience
- [ ] Networking fundamentals
- [ ] Scripting (Python, Bash, PowerShell)

### Preparation
- [ ] 1+ year AWS SysOps experience
- [ ] Hands-on with all core services
- [ ] Practiced lab scenarios
- [ ] Completed practice exams (80%+)

---

**Pro Tip:** SOA-C02 is the most hands-on AWS Associate exam. You MUST have practical experience - you'll need to perform tasks in a live AWS environment during the exam. Focus on automation, monitoring, and troubleshooting!

**Good luck!** 🚀
