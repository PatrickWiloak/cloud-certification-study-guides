# AWS SysOps Administrator Associate (SOA-C02) Practice Questions

40 scenario-based practice questions covering all exam domains.

**Exam Domain Breakdown:**
- Domain 1: Monitoring, Logging, and Remediation (20%) - 8 questions
- Domain 2: Reliability and Business Continuity (16%) - 6 questions
- Domain 3: Deployment, Provisioning, and Automation (18%) - 7 questions
- Domain 4: Security and Compliance (16%) - 7 questions
- Domain 5: Networking and Content Delivery (18%) - 7 questions
- Domain 6: Cost and Performance Optimization (12%) - 5 questions

---

## Domain 1: Monitoring, Logging, and Remediation (20%)

### Question 1
**Scenario:** A SysOps administrator needs to create a CloudWatch alarm that triggers when EC2 CPU utilization exceeds 80% for 15 minutes. The alarm should send notifications to an operations team. What configuration is required?

A. Create an alarm with 15 evaluation periods of 1 minute each
B. Create an alarm with 3 evaluation periods of 5 minutes each
C. Create an alarm with 1 evaluation period of 15 minutes
D. Create an alarm with 5 evaluation periods of 3 minutes each

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** To trigger after 15 minutes of sustained high CPU, configure 3 evaluation periods of 5 minutes (3 x 5 = 15 minutes). The alarm triggers when all 3 consecutive periods breach the threshold. Single 15-minute period provides less granularity. The period and evaluation count multiply to give the total duration.

**Key Concept:** [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
</details>

---

### Question 2
**Scenario:** An application running on EC2 writes logs to local files. The SysOps team needs to centralize these logs for analysis and alerting. What's the most efficient approach?

A. SSH into each instance and manually copy logs
B. Install and configure the CloudWatch agent
C. Use S3 sync to copy log files periodically
D. Configure VPC Flow Logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** The CloudWatch agent collects custom logs and metrics from EC2 instances, sending them to CloudWatch Logs for centralized analysis and alerting. Manual SSH doesn't scale. S3 sync adds delay. VPC Flow Logs capture network traffic, not application logs.

**Key Concept:** [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
</details>

---

### Question 3
**Scenario:** A company needs to automatically remediate EC2 instances that fail health checks. The remediation should replace unhealthy instances without manual intervention. What should be configured?

A. CloudWatch alarm with EC2 action to reboot
B. Auto Scaling group health checks with ReplaceUnhealthy process
C. Lambda function triggered by CloudWatch Events
D. Systems Manager Automation document

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Auto Scaling groups with health checks automatically terminate and replace unhealthy instances. The ReplaceUnhealthy process must be active. CloudWatch reboot action doesn't replace instances. Lambda adds complexity. Systems Manager requires manual triggers for this use case.

**Key Concept:** [Auto Scaling Health Checks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/healthcheck.html)
</details>

---

### Question 4
**Scenario:** A SysOps administrator needs to find all API calls made to delete S3 buckets in the last 30 days for a security audit. Where should they look?

A. S3 server access logs
B. CloudTrail event history
C. VPC Flow Logs
D. CloudWatch Logs Insights

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CloudTrail records all API calls including DeleteBucket operations. The Event History provides 90 days of management events. S3 access logs show object-level access, not bucket management. VPC Flow Logs capture network traffic. CloudWatch Logs would require CloudTrail to be configured to send logs there first.

**Key Concept:** [CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
</details>

---

### Question 5
**Scenario:** An RDS database is experiencing slow query performance. The SysOps team needs to identify the problematic queries without impacting production. What should they enable?

A. Enhanced Monitoring
B. Performance Insights
C. CloudWatch Logs export
D. Slow query log with manual analysis

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Performance Insights provides a dashboard to identify top SQL queries, wait events, and database load without impacting performance. Enhanced Monitoring shows OS-level metrics. CloudWatch Logs export requires configuration. Manual slow query analysis is time-consuming.

**Key Concept:** [RDS Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html)
</details>

---

### Question 6
**Scenario:** A company wants to be notified when their AWS account is approaching service quotas for EC2 instances. What should be configured?

A. AWS Config rules
B. Trusted Advisor with CloudWatch integration
C. CloudWatch billing alarms
D. AWS Service Quotas with CloudWatch alarms

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** AWS Service Quotas integrates with CloudWatch, allowing alarms when utilization approaches limits. Trusted Advisor shows current usage but limited alarm integration. Config rules check configurations. Billing alarms track costs, not quotas.

**Key Concept:** [Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/servicequotas-integration-cloudwatch.html)
</details>

---

### Question 7
**Scenario:** A Lambda function is experiencing errors. The SysOps administrator needs to trace the request path through API Gateway, Lambda, and DynamoDB to identify where failures occur. What service should they use?

A. CloudWatch Logs
B. AWS X-Ray
C. CloudTrail
D. VPC Flow Logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** X-Ray provides distributed tracing across AWS services, showing the request path and where latency or errors occur. CloudWatch Logs shows individual service logs. CloudTrail shows API calls, not request tracing. VPC Flow Logs capture network traffic.

**Key Concept:** [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)
</details>

---

### Question 8
**Scenario:** A SysOps administrator needs to create a dashboard that shows EC2 metrics from multiple AWS regions on a single view. What should they use?

A. Separate CloudWatch dashboards per region
B. CloudWatch cross-account cross-region dashboard
C. Amazon Managed Grafana
D. Third-party monitoring tool

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CloudWatch supports cross-region dashboards, allowing metrics from multiple regions on a single view. Separate dashboards per region defeats the purpose. Managed Grafana works but adds complexity for this use case. Third-party tools aren't the AWS-native solution.

**Key Concept:** [CloudWatch Cross-Region Dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_crossaccount_dashboard.html)
</details>

---

## Domain 2: Reliability and Business Continuity (16%)

### Question 9
**Scenario:** A company runs a critical web application on EC2 instances in a single Availability Zone. They need to improve availability without significant application changes. What's the recommended approach?

A. Add more instances in the same AZ
B. Deploy instances across multiple AZs with an ALB
C. Use a larger instance type
D. Enable EC2 Auto Recovery

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Deploying across multiple AZs with an Application Load Balancer provides fault tolerance against AZ failures. More instances in one AZ doesn't protect against AZ failures. Larger instances don't improve availability. Auto Recovery only helps with hardware failures in one AZ.

**Key Concept:** [High Availability](https://docs.aws.amazon.com/whitepapers/latest/real-time-communication-on-aws/high-availability-and-scalability-on-aws.html)
</details>

---

### Question 10
**Scenario:** An RDS MySQL database needs to be recoverable to any point in time within the last 7 days. What feature should be enabled?

A. Multi-AZ deployment
B. Automated backups with PITR
C. Manual snapshots
D. Read replicas

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Automated backups with Point-in-Time Recovery (PITR) allow restoring to any second within the retention period (up to 35 days). Multi-AZ provides HA, not recovery to specific times. Manual snapshots are point-in-time but not continuous. Read replicas are for read scaling.

**Key Concept:** [RDS Backup and Restore](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
</details>

---

### Question 11
**Scenario:** A company needs to ensure an S3 bucket's data can be recovered if accidentally deleted or overwritten. What should be configured?

A. Cross-region replication
B. S3 versioning
C. S3 Transfer Acceleration
D. S3 Intelligent-Tiering

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** S3 versioning preserves all versions of objects, allowing recovery of deleted or overwritten data. Cross-region replication also replicates deletes if delete markers are replicated. Transfer Acceleration speeds uploads. Intelligent-Tiering optimizes storage costs.

**Key Concept:** [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
</details>

---

### Question 12
**Scenario:** A SysOps administrator needs to create a disaster recovery strategy with RTO of 1 hour and RPO of 15 minutes for an application running on EC2 with RDS. What approach meets these requirements?

A. Backup and restore from S3
B. Pilot light with AMIs and RDS snapshots
C. Warm standby with Auto Scaling and RDS Multi-AZ
D. Active-active multi-region

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Warm standby with scaled-down infrastructure running continuously meets 1-hour RTO. RDS Multi-AZ with synchronous replication achieves 15-minute RPO (near-zero). Backup/restore takes hours. Pilot light requires provisioning time. Active-active is more than needed and expensive.

**Key Concept:** [Disaster Recovery Strategies](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)
</details>

---

### Question 13
**Scenario:** An EBS volume attached to an EC2 instance needs to be backed up automatically every day with retention for 30 days. What's the most efficient method?

A. Custom Lambda function with CreateSnapshot API
B. Amazon Data Lifecycle Manager (DLM)
C. AWS Backup
D. Manual snapshots via console

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon Data Lifecycle Manager automates EBS snapshot creation with retention policies. AWS Backup provides broader functionality but DLM is simpler for EBS-only needs. Lambda requires custom code. Manual snapshots don't scale.

**Key Concept:** [Amazon DLM](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/snapshot-lifecycle.html)
</details>

---

### Question 14
**Scenario:** A company wants to implement automatic failover for an RDS database without application changes. The failover should happen within 60 seconds of a failure. What should be configured?

A. RDS Multi-AZ
B. RDS Read Replicas
C. Aurora Global Database
D. RDS Cross-Region Read Replica

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** RDS Multi-AZ provides automatic failover to a standby instance (typically 60-120 seconds) without application changes - it uses the same endpoint. Read replicas require application changes to redirect traffic. Aurora Global Database is cross-region. Cross-region replicas require manual promotion.

**Key Concept:** [RDS Multi-AZ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
</details>

---

## Domain 3: Deployment, Provisioning, and Automation (18%)

### Question 15
**Scenario:** A SysOps administrator needs to deploy a CloudFormation stack but wants to preview changes before applying them. The stack manages production infrastructure. What should they do?

A. Deploy to a separate test account first
B. Create a change set and review it
C. Use CloudFormation drift detection
D. Use the ValidateTemplate API

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Change sets preview what changes CloudFormation will make before execution, allowing review of additions, modifications, and deletions. Test accounts don't show production changes. Drift detection shows current vs. template differences. ValidateTemplate checks syntax only.

**Key Concept:** [CloudFormation Change Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
</details>

---

### Question 16
**Scenario:** A company needs to ensure all EC2 instances are launched with a specific IAM role, regardless of how they're created. How can this be enforced?

A. Create a custom AMI with the role
B. Use AWS Config with a required-tags rule
C. Implement a Service Control Policy (SCP)
D. Use EC2 launch templates as default

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Service Control Policies can restrict EC2:RunInstances to require a specific IAM instance profile. SCPs are preventive controls at the organization level. Custom AMIs don't enforce role attachment. Config rules are detective, not preventive. Launch templates are optional.

**Key Concept:** [Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
</details>

---

### Question 17
**Scenario:** A SysOps administrator needs to install and configure the same software on 500 EC2 instances. The configuration should be applied automatically to new instances. What should they use?

A. User data scripts
B. AWS Systems Manager State Manager
C. Manual SSH and installation
D. Custom AMI only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Systems Manager State Manager maintains desired configuration state across instances automatically, including new ones. User data runs once at launch. Manual SSH doesn't scale. Custom AMIs help but don't handle configuration changes.

**Key Concept:** [Systems Manager State Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state.html)
</details>

---

### Question 18
**Scenario:** An Auto Scaling group needs to update instances to a new AMI without service interruption. The update should replace instances gradually. What's the recommended approach?

A. Update the launch template and manually terminate instances
B. Update the launch template and start an instance refresh
C. Create a new Auto Scaling group and switch traffic
D. Use rolling update with AWS CodeDeploy

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Instance refresh performs rolling updates of instances in an Auto Scaling group using the new launch template. It respects health checks and warmup time. Manual termination is error-prone. New ASG requires traffic switching. CodeDeploy is for application deployments.

**Key Concept:** [Instance Refresh](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html)
</details>

---

### Question 19
**Scenario:** A company needs to patch Windows EC2 instances during a maintenance window with automatic rollback if patching fails. What should be used?

A. Windows Update scheduled task
B. AWS Systems Manager Patch Manager
C. Custom Lambda function
D. User data script at instance launch

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Systems Manager Patch Manager provides automated patching with maintenance windows, compliance reporting, and can integrate with rollback procedures. Windows Update lacks centralized management. Lambda requires custom implementation. User data runs at launch, not for ongoing patching.

**Key Concept:** [Patch Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html)
</details>

---

### Question 20
**Scenario:** A CloudFormation stack update failed and rolled back. The SysOps administrator needs to understand why. Where should they look?

A. CloudWatch Logs
B. CloudFormation Events tab
C. CloudTrail logs
D. EC2 system logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CloudFormation Events tab shows the sequence of resource operations and failure reasons for each resource. CloudWatch Logs might have application logs but not CF-specific errors. CloudTrail shows API calls. EC2 logs are instance-specific.

**Key Concept:** [CloudFormation Troubleshooting](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html)
</details>

---

### Question 21
**Scenario:** A company wants to automatically terminate EC2 instances that have been running for more than 7 days to ensure fresh deployments. What should be configured?

A. CloudWatch alarm with EC2 terminate action
B. AWS Instance Scheduler
C. Lambda function with EventBridge scheduled rule
D. Auto Scaling max instance lifetime

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** Auto Scaling groups support max instance lifetime, automatically replacing instances that exceed the specified age. CloudWatch can't track instance age easily. Instance Scheduler is for start/stop, not termination. Lambda requires custom code.

**Key Concept:** [Instance Lifetime](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-max-instance-lifetime.html)
</details>

---

## Domain 4: Security and Compliance (16%)

### Question 22
**Scenario:** A SysOps administrator needs to ensure all S3 buckets in the account have server-side encryption enabled. They want to be alerted if any bucket is non-compliant. What should they configure?

A. S3 bucket policies on each bucket
B. AWS Config rule for s3-bucket-server-side-encryption-enabled
C. CloudTrail with CloudWatch alarms
D. IAM policy denying unencrypted uploads

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Config managed rule checks bucket encryption configuration and reports non-compliant resources with optional remediation. Bucket policies don't detect compliance. CloudTrail monitors API calls, not configuration state. IAM policies prevent actions but don't report compliance.

**Key Concept:** [AWS Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-server-side-encryption-enabled.html)
</details>

---

### Question 23
**Scenario:** An EC2 instance needs to access an S3 bucket without storing long-term credentials on the instance. What's the recommended approach?

A. Create an IAM user and store access keys in environment variables
B. Attach an IAM role to the EC2 instance
C. Store credentials in AWS Secrets Manager
D. Use S3 bucket policies with IP restrictions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM roles for EC2 provide temporary credentials that rotate automatically without storing long-term credentials. Access keys require secure storage and rotation. Secrets Manager adds complexity for this use case. Bucket policies with IP don't provide authenticated access.

**Key Concept:** [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
</details>

---

### Question 24
**Scenario:** A company needs to encrypt EBS volumes using a customer-managed KMS key. Existing unencrypted volumes need to be encrypted. What's the correct process?

A. Enable encryption on existing volumes directly
B. Create encrypted snapshots and restore to new encrypted volumes
C. Use the EBS modify volume API
D. Copy data to new encrypted volumes using rsync

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** You cannot directly encrypt an unencrypted volume. Create a snapshot, copy the snapshot with encryption enabled using the CMK, then create a new volume from the encrypted snapshot. Modify volume can't add encryption. rsync requires instance downtime and doesn't use EBS encryption.

**Key Concept:** [EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)
</details>

---

### Question 25
**Scenario:** A company needs to enforce MFA for all IAM users who access the AWS Console. How can this be implemented?

A. IAM policy with MFA condition on all actions
B. AWS Config rule to check MFA status
C. SCPs requiring MFA for console access
D. IAM user password policy

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** IAM policies with `aws:MultiFactorAuthPresent` condition can deny actions when MFA isn't used. This can be applied via permission boundaries or group policies. Config rules detect but don't enforce. SCPs work at organization level but the condition-based policy is more common. Password policy doesn't enforce MFA.

**Key Concept:** [MFA in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html)
</details>

---

### Question 26
**Scenario:** A SysOps administrator needs to rotate the access keys for a service account that's used by multiple applications. How should they perform this rotation without downtime?

A. Delete the old key, create new key, update all applications
B. Create new key, update applications, verify, then delete old key
C. Use IAM roles instead of access keys
D. Contact AWS Support for key rotation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Create the new key first (IAM allows two active keys), update applications to use the new key, verify they work, then delete the old key. This ensures no downtime. Deleting first causes immediate failures. Roles are preferred long-term but don't answer the rotation question. Support doesn't rotate keys for you.

**Key Concept:** [Access Key Rotation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_RotateAccessKey)
</details>

---

### Question 27
**Scenario:** A company needs to detect when root account credentials are used for any action. What should be configured?

A. CloudWatch Logs metric filter
B. CloudTrail with EventBridge rule for root user events
C. AWS Config rule
D. IAM Access Analyzer

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CloudTrail logs root user activity, and EventBridge rules can trigger on root user events for alerting. This is a security best practice. CloudWatch metric filters work but require additional configuration. Config doesn't track user activity. Access Analyzer finds external access risks.

**Key Concept:** [Monitoring Root Activity](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#id_root-user-monitoring)
</details>

---

### Question 28
**Scenario:** An application on EC2 needs to retrieve database credentials securely at runtime. The credentials should rotate automatically. What service should store the credentials?

A. Systems Manager Parameter Store (SecureString)
B. AWS Secrets Manager with rotation enabled
C. Environment variables
D. S3 bucket with encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Secrets Manager provides automatic rotation for database credentials with Lambda functions. Parameter Store SecureString stores secrets but doesn't rotate them automatically. Environment variables don't rotate. S3 is not designed for secrets management.

**Key Concept:** [Secrets Manager Rotation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)
</details>

---

## Domain 5: Networking and Content Delivery (18%)

### Question 29
**Scenario:** An EC2 instance in a private subnet needs to download software updates from the internet without having a public IP address. What should be configured?

A. Internet Gateway attached to the VPC
B. NAT Gateway in a public subnet with route table update
C. VPC endpoint for S3
D. Elastic IP address on the instance

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** NAT Gateway allows instances in private subnets to access the internet while remaining private. The private subnet route table needs a route to the NAT Gateway. Internet Gateway alone doesn't help private subnets. VPC endpoint is for AWS services, not general internet. Elastic IP makes the instance public.

**Key Concept:** [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
</details>

---

### Question 30
**Scenario:** A company needs to allow EC2 instances in a VPC to access S3 without traffic going over the internet. What should be configured?

A. NAT Gateway
B. VPC Gateway Endpoint for S3
C. Internet Gateway
D. Direct Connect

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Gateway VPC Endpoints for S3 route traffic directly to S3 over the AWS network, not the internet. There's no additional charge for Gateway endpoints. NAT Gateway routes traffic over the internet. Internet Gateway also uses public routes. Direct Connect is for on-premises connectivity.

**Key Concept:** [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)
</details>

---

### Question 31
**Scenario:** A web application behind an ALB needs to reject requests from specific IP addresses known to be malicious. What should be configured?

A. Security Group on the ALB
B. Network ACL on the subnet
C. AWS WAF with IP set rules
D. Route 53 routing policy

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS WAF can block specific IP addresses or ranges using IP set rules attached to the ALB. Security Groups are allow-only (no explicit deny). NACLs could work but don't integrate with threat intelligence. Route 53 doesn't filter requests.

**Key Concept:** [AWS WAF IP Rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-ipset-match.html)
</details>

---

### Question 32
**Scenario:** Two VPCs in different regions need to communicate with each other. Traffic should stay on the AWS network. What should be configured?

A. VPC Peering
B. VPN connection
C. AWS Transit Gateway with inter-region peering
D. Direct Connect

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** VPC Peering supports inter-region connectivity with traffic staying on the AWS backbone network. Transit Gateway with peering also works but is more complex for two VPCs. VPN uses encrypted tunnels but goes over internet. Direct Connect is for on-premises.

**Key Concept:** [Inter-Region VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
</details>

---

### Question 33
**Scenario:** An Application Load Balancer shows healthy targets, but users report intermittent 504 Gateway Timeout errors. What is the most likely cause?

A. Security Group blocking traffic
B. Backend instances taking longer than the ALB idle timeout
C. Insufficient ALB capacity
D. DNS resolution issues

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** 504 errors occur when the backend doesn't respond within the ALB's idle timeout (default 60 seconds). Long-running requests can cause this. Security Groups blocking would show unhealthy targets. ALB scales automatically. DNS issues would affect all requests.

**Key Concept:** [ALB Timeout Settings](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html#connection-idle-timeout)
</details>

---

### Question 34
**Scenario:** A company needs to distribute static content globally with low latency and reduce load on origin servers. What AWS service should they use?

A. Route 53 latency-based routing
B. Amazon CloudFront
C. Global Accelerator
D. Application Load Balancer

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CloudFront is a CDN that caches content at edge locations worldwide, reducing latency and origin load. Route 53 routing directs to different origins but doesn't cache. Global Accelerator optimizes routing but doesn't cache content. ALB doesn't cache.

**Key Concept:** [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
</details>

---

### Question 35
**Scenario:** A company is experiencing DNS resolution issues for their Route 53 hosted zone. How can they verify DNS is working correctly?

A. Check CloudWatch Logs
B. Use Route 53 DNS query test
C. Review VPC Flow Logs
D. Check EC2 instance system logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Route 53 provides a test record feature that shows how Route 53 responds to DNS queries. This helps verify configuration. CloudWatch Logs don't show DNS responses. VPC Flow Logs show network traffic, not DNS content. EC2 logs are instance-specific.

**Key Concept:** [Route 53 Testing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-test.html)
</details>

---

## Domain 6: Cost and Performance Optimization (12%)

### Question 36
**Scenario:** A company's EC2 instances have consistently low CPU utilization (under 10%) but can't be downsized due to memory requirements. What should they consider?

A. Use Reserved Instances for cost savings
B. Switch to memory-optimized instance types
C. Enable detailed monitoring
D. Implement Spot Instances

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Memory-optimized instances (R family) provide more memory per vCPU, allowing appropriate sizing for memory-bound workloads. Reserved Instances save costs but don't address sizing. Detailed monitoring doesn't help. Spot may not suit all workloads.

**Key Concept:** [Instance Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)
</details>

---

### Question 37
**Scenario:** A company runs batch processing jobs that can be interrupted and restarted. They want to minimize costs. What EC2 pricing model should they use?

A. On-Demand Instances
B. Reserved Instances
C. Spot Instances
D. Dedicated Hosts

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Spot Instances offer up to 90% discount for interruptible workloads. Batch processing that can handle interruptions is ideal for Spot. On-Demand is more expensive. Reserved requires commitment. Dedicated Hosts are for compliance/licensing.

**Key Concept:** [Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)
</details>

---

### Question 38
**Scenario:** AWS Trusted Advisor shows that several EBS volumes are unattached and have been for over 30 days. What action should be taken?

A. Attach them to instances
B. Create snapshots and delete the volumes
C. Convert them to lower-cost storage
D. Ignore the recommendation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Unattached volumes incur charges without providing value. Best practice is to snapshot (if data is needed) and delete the volumes. Random attachment isn't useful. EBS doesn't have storage tiers like S3. Ignoring wastes money.

**Key Concept:** [Trusted Advisor Cost Optimization](https://docs.aws.amazon.com/awssupport/latest/user/cost-optimization-checks.html)
</details>

---

### Question 39
**Scenario:** A company wants recommendations for right-sizing their EC2 instances based on actual usage patterns. What tool should they use?

A. AWS Budgets
B. AWS Cost Explorer with right-sizing recommendations
C. CloudWatch dashboards
D. AWS Pricing Calculator

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cost Explorer provides right-sizing recommendations based on CloudWatch metrics, showing potential savings from downsizing. Budgets track spending. CloudWatch shows metrics but not recommendations. Pricing Calculator estimates costs for new resources.

**Key Concept:** [Right Sizing Recommendations](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-rightsizing.html)
</details>

---

### Question 40
**Scenario:** An S3 bucket stores log files that are accessed frequently for the first 30 days, then rarely accessed. How can storage costs be optimized?

A. Delete files after 30 days
B. Use S3 Intelligent-Tiering
C. Configure lifecycle policy to transition to S3 Standard-IA after 30 days
D. Enable S3 Transfer Acceleration

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** S3 lifecycle policies automatically transition objects to cheaper storage classes based on age. Standard-IA is appropriate for infrequent access. Deleting loses data. Intelligent-Tiering adds monitoring costs for known access patterns. Transfer Acceleration speeds uploads, not storage costs.

**Key Concept:** [S3 Lifecycle Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | B | Monitoring & Remediation |
| 2 | B | Monitoring & Remediation |
| 3 | B | Monitoring & Remediation |
| 4 | B | Monitoring & Remediation |
| 5 | B | Monitoring & Remediation |
| 6 | D | Monitoring & Remediation |
| 7 | B | Monitoring & Remediation |
| 8 | B | Monitoring & Remediation |
| 9 | B | Reliability & Continuity |
| 10 | B | Reliability & Continuity |
| 11 | B | Reliability & Continuity |
| 12 | C | Reliability & Continuity |
| 13 | B | Reliability & Continuity |
| 14 | A | Reliability & Continuity |
| 15 | B | Deployment & Automation |
| 16 | C | Deployment & Automation |
| 17 | B | Deployment & Automation |
| 18 | B | Deployment & Automation |
| 19 | B | Deployment & Automation |
| 20 | B | Deployment & Automation |
| 21 | D | Deployment & Automation |
| 22 | B | Security & Compliance |
| 23 | B | Security & Compliance |
| 24 | B | Security & Compliance |
| 25 | A | Security & Compliance |
| 26 | B | Security & Compliance |
| 27 | B | Security & Compliance |
| 28 | B | Security & Compliance |
| 29 | B | Networking & Delivery |
| 30 | B | Networking & Delivery |
| 31 | C | Networking & Delivery |
| 32 | A | Networking & Delivery |
| 33 | B | Networking & Delivery |
| 34 | B | Networking & Delivery |
| 35 | B | Networking & Delivery |
| 36 | B | Cost & Performance |
| 37 | C | Cost & Performance |
| 38 | B | Cost & Performance |
| 39 | B | Cost & Performance |
| 40 | C | Cost & Performance |

---

## Study Resources

- [AWS SysOps Administrator Exam Guide](https://aws.amazon.com/certification/certified-sysops-admin-associate/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Skill Builder](https://skillbuilder.aws/)
