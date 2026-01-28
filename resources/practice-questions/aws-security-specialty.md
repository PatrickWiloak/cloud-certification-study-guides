# AWS Security Specialty (SCS-C02) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Threat Detection and Incident Response | 14% | 6 |
| Security Logging and Monitoring | 18% | 7 |
| Infrastructure Security | 20% | 8 |
| Identity and Access Management | 16% | 6 |
| Data Protection | 18% | 7 |
| Management and Security Governance | 14% | 6 |

---

## Domain 1: Threat Detection and Incident Response (Questions 1-6)

### Question 1
**Scenario:** Your company's security team received an alert from Amazon GuardDuty indicating "UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS". An EC2 instance's credentials appear to be used from an external IP address. What is the MOST appropriate immediate response?

A. Terminate the EC2 instance immediately
B. Rotate the instance's IAM role credentials by stopping and starting the instance
C. Isolate the instance by modifying its security group, preserve evidence, and investigate
D. Delete the IAM role attached to the instance

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** The proper incident response is to isolate the compromised resource while preserving forensic evidence. Modifying security groups to restrict all traffic isolates the instance without destroying evidence. Terminating (A) or stopping (B) destroys volatile memory evidence. Deleting the IAM role (D) doesn't stop the attacker who already has the credentials and destroys audit trails.

**Key Concept:** [AWS Security Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/welcome.html)
</details>

### Question 2
**Scenario:** A company needs to automatically remediate GuardDuty findings for compromised EC2 instances. The remediation should isolate instances, capture memory dumps, and notify the security team. What architecture provides this automation?

A. GuardDuty → SNS → Lambda → SSM Run Command + Security Group modification
B. GuardDuty → CloudWatch Events → Step Functions (orchestrate isolation, forensics, notification)
C. GuardDuty → S3 export → Athena analysis → manual remediation
D. GuardDuty → Security Hub → AWS Config auto-remediation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Step Functions provides orchestration for multi-step incident response workflows including parallel execution of isolation (security group changes), forensics (memory capture via SSM), and notification (SNS/SES). EventBridge (CloudWatch Events) triggers on GuardDuty findings. Option A lacks orchestration for complex workflows. Option C is not automated. Option D doesn't support custom remediation workflows for this use case.

**Key Concept:** [Automated Incident Response with Step Functions](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/automate-incident-response-and-forensics.html)
</details>

### Question 3
**Scenario:** During a security incident investigation, you need to analyze VPC Flow Logs to identify data exfiltration patterns. The logs are stored in S3 in their default format. You need to query terabytes of logs quickly and cost-effectively. Which approach is BEST?

A. Download logs to an EC2 instance and use grep/awk for analysis
B. Use Amazon Athena with partition projection on the S3 data
C. Import logs into Amazon OpenSearch Service for analysis
D. Use Amazon Macie to analyze the flow logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Athena with partition projection provides serverless, cost-effective querying of S3 data without infrastructure management. Partition projection automatically handles the time-based partitioning of flow logs. You pay only for data scanned. Option A doesn't scale. Option C requires provisioning and is expensive for ad-hoc analysis. Option D is for sensitive data discovery, not network analysis.

**Key Concept:** [Querying VPC Flow Logs with Athena](https://docs.aws.amazon.com/athena/latest/ug/vpc-flow-logs.html)
</details>

### Question 4
**Scenario:** An organization wants to detect when IAM credentials are used from IP addresses outside their corporate network range. They also need alerts for API calls from Tor exit nodes and known malicious IPs. Which solution provides this detection?

A. Create CloudWatch Logs metric filters on CloudTrail logs for source IP patterns
B. Enable GuardDuty with threat intelligence feeds and create custom threat lists
C. Use AWS Config rules to evaluate IAM policy compliance
D. Implement VPC Flow Logs analysis with custom Lambda functions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GuardDuty provides built-in threat intelligence including Tor exit nodes and known malicious IPs. Custom threat lists allow adding your own IP allow/deny lists. GuardDuty analyzes CloudTrail, VPC Flow Logs, and DNS logs automatically. CloudWatch metric filters (A) require complex regex and don't include threat intelligence. Config (C) is for resource compliance. VPC Flow Logs (D) don't capture IAM credential usage.

**Key Concept:** [GuardDuty Trusted IP and Threat Lists](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_upload-lists.html)
</details>

### Question 5
**Scenario:** Your security team needs to investigate a potential data breach that occurred 45 days ago. They need to analyze CloudTrail logs, VPC Flow Logs, and GuardDuty findings from that time period. How should you ensure this data is available?

A. CloudTrail, Flow Logs, and GuardDuty retain 90 days of data by default
B. Configure CloudTrail to log to S3, Flow Logs to S3/CloudWatch Logs, and export GuardDuty findings to S3
C. Use AWS Backup to retain security logs
D. Enable long-term retention in each service's console settings

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Default retention varies: CloudTrail Event History is 90 days but limited, GuardDuty findings are 90 days but archives after that, VPC Flow Logs in CloudWatch default to never expire but cost increases. For reliable long-term retention and analysis, all logs should be exported to S3 with appropriate lifecycle policies. This enables Athena queries and meets compliance requirements. AWS Backup (C) doesn't support these log types.

**Key Concept:** [AWS Security Logging Best Practices](https://docs.aws.amazon.com/prescriptive-guidance/latest/logging-monitoring-for-application-owners/aws-services-logging-monitoring.html)
</details>

### Question 6
**Scenario:** Amazon Detective shows that an IAM user has been making unusual API calls to download S3 objects at 3 AM, which is outside normal business hours. The user's regular pattern shows activity only during business hours. What additional context does Detective provide that GuardDuty does not?

A. The specific S3 objects that were accessed
B. Visualization of the user's historical behavior patterns and entity relationships
C. Real-time alerting on the suspicious activity
D. Automatic remediation of the threat

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon Detective provides behavior graphs and entity relationship visualizations that show historical patterns, related resources, and the context around findings. It helps answer "what happened" and "why" during investigations. GuardDuty provides the alert (C), but Detective provides investigation context. Detective doesn't show specific object names (A) or provide remediation (D).

**Key Concept:** [Amazon Detective Investigation Flow](https://docs.aws.amazon.com/detective/latest/userguide/detective-investigation-flow.html)
</details>

---

## Domain 2: Security Logging and Monitoring (Questions 7-13)

### Question 7
**Scenario:** A company needs to aggregate security findings from GuardDuty, Inspector, Macie, IAM Access Analyzer, and Firewall Manager across 200 AWS accounts. They need a single dashboard for prioritization and automated response. Which solution provides this?

A. Create a centralized CloudWatch dashboard pulling metrics from all accounts
B. Use AWS Security Hub with organization-wide enablement and custom insights
C. Build a custom solution using EventBridge and a central Lambda function
D. Deploy third-party SIEM and configure each service to export findings

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Security Hub provides native integration with AWS security services, organization-wide deployment via AWS Organizations, automated finding aggregation, compliance standards (CIS, PCI DSS), and custom insights for prioritization. It supports automated response via EventBridge integration. CloudWatch (A) doesn't aggregate security findings. Custom solution (C) requires significant development. Third-party SIEM (D) adds cost and complexity.

**Key Concept:** [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
</details>

### Question 8
**Scenario:** Your organization requires that all API activity across all accounts be logged, protected from tampering, and retained for 7 years for compliance. Logs must be queryable for security investigations. How should you architect this?

A. Enable CloudTrail in each account logging to local S3 buckets with Object Lock
B. Create an organization trail logging to a central S3 bucket with S3 Object Lock (Governance mode), lifecycle to Glacier after 1 year
C. Create an organization trail to a central S3 bucket in a dedicated log archive account with S3 Object Lock (Compliance mode) and SCPs preventing log deletion
D. Enable CloudTrail Insights in each account with extended retention

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Organization trails provide centralized logging across all accounts. A dedicated log archive account with restricted access follows AWS best practices. S3 Object Lock in Compliance mode (not Governance) provides WORM protection that even the root user cannot override. SCPs prevent deletion attempts. Glacier lifecycle reduces long-term costs while maintaining compliance. Option B uses Governance mode which can be overridden.

**Key Concept:** [AWS Log Archive Account](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/log-archive.html)
</details>

### Question 9
**Scenario:** A security team needs real-time alerts when root user credentials are used in any account across their AWS Organization. The alerts should include the source IP, user agent, and which API was called. What is the MOST efficient solution?

A. Create CloudWatch Alarms on CloudTrail metrics in each account
B. Configure EventBridge rules in each account forwarding to a central SNS topic
C. Use an organization trail with EventBridge rule in the management account matching root user events
D. Enable GuardDuty root user monitoring across the organization

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** An organization trail centralizes all CloudTrail events to one location. A single EventBridge rule in the management account can match root user events from any account with access to source IP, user agent, and API details from the CloudTrail event. This is simpler than rules in each account (B). CloudWatch Alarms (A) don't provide event details. GuardDuty (D) detects anomalies but doesn't alert on all root usage.

**Key Concept:** [Monitoring Root User Activity](https://docs.aws.amazon.com/IAM/latest/UserGuide/cloudtrail-integration.html)
</details>

### Question 10
**Scenario:** An application team wants to log access to sensitive DynamoDB tables containing PII. They need to capture who accessed which items, when, and from where. Standard CloudTrail only shows API calls, not item-level access. What solution provides this granularity?

A. Enable DynamoDB Streams and process with Lambda to log access
B. Enable CloudTrail data events for DynamoDB and use attribute-based access logging
C. Implement application-level logging using X-Ray and custom annotations
D. DynamoDB doesn't support item-level access logging; use a different database

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CloudTrail data events for DynamoDB capture item-level API activity including GetItem, PutItem, Query, and Scan operations. Events include the table name, key attributes accessed, and IAM identity. This provides the audit trail without application changes. DynamoDB Streams (A) captures changes, not reads. X-Ray (C) is for performance tracing, not security auditing.

**Key Concept:** [CloudTrail DynamoDB Data Events](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/logging-using-cloudtrail.html)
</details>

### Question 11
**Scenario:** You need to monitor for changes to security-critical resources (security groups, NACLs, IAM policies) across all accounts and receive alerts within 1 minute of changes occurring. Changes should be automatically reverted if they violate security policies. Which solution provides this?

A. AWS Config rules with automatic remediation using SSM Automation
B. CloudTrail events triggering Lambda functions for each change
C. GuardDuty detecting configuration changes
D. Security Hub with automated response playbooks

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** AWS Config provides near real-time configuration change detection (typically within minutes), managed and custom rules for evaluation, and automatic remediation via SSM Automation documents. This covers security groups, NACLs, and IAM policies natively. CloudTrail/Lambda (B) requires custom development and doesn't have built-in compliance evaluation. GuardDuty (C) detects threats, not general config changes. Security Hub (D) aggregates but doesn't provide config-level remediation.

**Key Concept:** [AWS Config Auto Remediation](https://docs.aws.amazon.com/config/latest/developerguide/remediation.html)
</details>

### Question 12
**Scenario:** A company is using AWS WAF to protect their web applications. Security requires that all blocked requests be logged with full request details for forensic analysis, but they want to minimize costs for allowed traffic logging. How should they configure WAF logging?

A. Enable WAF logging to CloudWatch Logs with a filter to log only blocked requests
B. Enable WAF logging to S3 with a logging filter for BLOCK actions only
C. Enable WAF logging to Kinesis Data Firehose with a filter for blocked requests, deliver to S3
D. Use CloudFront access logs instead of WAF logging

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** WAF logging with Kinesis Data Firehose supports log filtering at the source, allowing you to log only BLOCK actions. Firehose can deliver to S3 for cost-effective storage and Athena analysis. This reduces log volume and costs significantly. WAF logging to CloudWatch (A) doesn't support source filtering. Direct S3 logging (B) doesn't support filtering. CloudFront logs (D) don't include WAF rule details.

**Key Concept:** [AWS WAF Logging](https://docs.aws.amazon.com/waf/latest/developerguide/logging.html)
</details>

### Question 13
**Scenario:** Your organization needs to ensure all CloudWatch Log Groups containing security logs are encrypted with customer-managed KMS keys and have retention periods of at least 1 year. How can you continuously monitor and enforce this?

A. Create a Lambda function running daily to check log group configurations
B. Use AWS Config custom rules evaluating log group encryption and retention
C. Enable CloudTrail and alert on CreateLogGroup events missing encryption
D. Use Service Control Policies to require encryption on log group creation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Config custom rules can continuously evaluate CloudWatch Log Groups for both encryption status and retention period settings. Non-compliant resources are flagged and can trigger remediation. Lambda (A) provides point-in-time checks, not continuous monitoring. CloudTrail (C) only catches creation, not existing non-compliant resources. SCPs (D) can't enforce specific retention periods.

**Key Concept:** [AWS Config Custom Rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_develop-rules.html)
</details>

---

## Domain 3: Infrastructure Security (Questions 14-21)

### Question 14
**Scenario:** A company needs to allow HTTPS traffic from the internet to their application load balancer while ensuring all traffic between the ALB and EC2 instances in private subnets is also encrypted. The EC2 instances should not be directly accessible from the internet. How should this be architected?

A. ALB in public subnet with HTTPS listener, HTTP to instances in private subnets
B. ALB in public subnet with HTTPS listener, HTTPS to instances using ACM certificates, security groups restricting instance access to ALB only
C. NLB in public subnet with TLS passthrough to instances
D. ALB in private subnet using AWS PrivateLink for internet access

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ALB in public subnets handles internet traffic. HTTPS listeners terminate TLS, then re-encrypt to instances using certificates (ACM for ALB, self-signed or ACM-issued for instances). Security groups on instances allow traffic only from ALB's security group. Private subnets with no direct internet route ensure isolation. Option A doesn't encrypt ALB-to-instance traffic. NLB TLS passthrough (C) doesn't allow ALB-level security features. ALB can't be in private subnets for public traffic (D).

**Key Concept:** [End-to-End Encryption with ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html)
</details>

### Question 15
**Scenario:** An organization wants to ensure that EC2 instances can only be launched using approved, hardened AMIs that have been scanned and approved by the security team. How should they enforce this?

A. Create an SCP that allows ec2:RunInstances only for specific AMI IDs
B. Use AWS Config rule to detect non-compliant instances and terminate them
C. Implement IAM policies with conditions restricting ec2:RunInstances to AMIs with specific tags
D. Use EC2 Image Builder with automatic distribution and deprecate old AMIs

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** IAM policies (or SCPs) with conditions on ec2:RunInstances can restrict launches to AMIs matching specific tags (e.g., "approved:true") or owner (e.g., account ID of security team). This is preventive and flexible - new approved AMIs don't require policy updates. SCP with specific IDs (A) requires updates for each new AMI. Config (B) is detective, not preventive. Image Builder (D) helps create AMIs but doesn't prevent use of unapproved ones.

**Key Concept:** [IAM Condition Keys for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-policy-structure.html#amazon-ec2-keys)
</details>

### Question 16
**Scenario:** A security team needs to ensure no security group in their VPCs has inbound rules allowing SSH (port 22) from 0.0.0.0/0. They want automatic remediation that removes such rules while preserving other rules in the security group. What solution provides this?

A. AWS Config managed rule "restricted-ssh" with SSM Automation remediation that deletes the security group
B. AWS Config managed rule "restricted-ssh" with custom Lambda remediation that revokes only the offending rules
C. GuardDuty detecting open security groups with EventBridge remediation
D. AWS Firewall Manager security group policy with auto-remediation

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** AWS Firewall Manager security group policies can audit and auto-remediate security groups across accounts and VPCs. It can specifically remove rules allowing 0.0.0.0/0 on specified ports while preserving other rules. This is purpose-built for this use case. Option B requires custom Lambda code. Option A's remediation would delete the entire group. GuardDuty (C) doesn't detect security group configurations.

**Key Concept:** [AWS Firewall Manager Security Group Policies](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies.html)
</details>

### Question 17
**Scenario:** An organization uses AWS Network Firewall to inspect traffic between VPCs in their Transit Gateway architecture. They need to ensure all east-west traffic between production and development VPCs is inspected, but traffic within the same environment bypasses inspection for performance. How should routing be configured?

A. Attach all VPCs directly to Transit Gateway and use TGW route tables for inspection routing
B. Create separate TGW route tables for prod and dev, with routes sending cross-environment traffic through the inspection VPC
C. Use VPC peering for same-environment traffic and Transit Gateway for cross-environment
D. Deploy Network Firewall in each VPC for local inspection

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Transit Gateway route tables enable sophisticated routing policies. Create separate route tables for prod and dev VPC attachments. Configure routes so traffic destined for the other environment routes through the inspection VPC (containing Network Firewall), while same-environment traffic routes directly. This provides selective inspection without performance impact on internal traffic. Option C creates management complexity. Option D is expensive and redundant.

**Key Concept:** [Transit Gateway Network Firewall Architecture](https://docs.aws.amazon.com/network-firewall/latest/developerguide/arch-two-zone-igw.html)
</details>

### Question 18
**Scenario:** A company is running sensitive workloads on EC2 that require protection against hardware-level attacks, including protection of data in memory from AWS operators. Which EC2 feature provides this capability?

A. EC2 Dedicated Hosts with host-based encryption
B. EC2 instances with encrypted EBS volumes
C. AWS Nitro Enclaves for processing sensitive data
D. EC2 instances in a dedicated tenancy VPC

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS Nitro Enclaves create isolated compute environments with no persistent storage, no external networking, and cryptographic attestation. Even AWS operators and the parent instance cannot access data inside an enclave. This protects against hardware-level and privileged access attacks. Dedicated Hosts (A) provide physical isolation but not memory protection from operators. EBS encryption (B) protects data at rest. Dedicated tenancy (D) isolates from other customers, not AWS operators.

**Key Concept:** [AWS Nitro Enclaves](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html)
</details>

### Question 19
**Scenario:** An application running on EC2 needs to access an S3 bucket containing sensitive data. Security requires that the traffic never traverses the public internet and that access be restricted to only this specific application. How should this be implemented?

A. Use an S3 VPC endpoint (gateway type) with bucket policy restricting access to the endpoint
B. Use an S3 VPC endpoint (interface type) with security groups and bucket policy restricting access to the endpoint ID
C. Configure S3 Transfer Acceleration for private connectivity
D. Use AWS PrivateLink for S3 with Direct Connect

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Interface endpoints (powered by PrivateLink) provide private IP addresses in your VPC that can be controlled with security groups, allowing restriction to specific EC2 instances. Bucket policies can further restrict access to the vpc endpoint ID. Gateway endpoints (A) don't support security groups and use route tables for access control, making application-specific restriction harder. Transfer Acceleration (C) uses public internet. PrivateLink with Direct Connect (D) is for hybrid connectivity.

**Key Concept:** [S3 Interface Endpoints](https://docs.aws.amazon.com/AmazonS3/latest/userguide/privatelink-interface-endpoints.html)
</details>

### Question 20
**Scenario:** A web application protected by AWS WAF is experiencing a Layer 7 DDoS attack with requests that appear legitimate but are overwhelming the application. The attack is using multiple IP addresses that rotate frequently. Which WAF feature helps mitigate this?

A. IP reputation rule group blocking known bad IPs
B. Rate-based rule with IP address aggregation key
C. Rate-based rule with custom aggregation keys (URI, header values) combined with Bot Control
D. Geo-blocking rules for suspicious countries

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** When attackers rotate IPs, IP-based rate limiting is ineffective. Custom aggregation keys allow rate limiting based on other request attributes like session tokens, URI paths, or header values that remain constant across the attack. Combined with Bot Control's ML-based detection, this identifies and blocks sophisticated application-layer attacks. IP reputation (A) and IP-based rate limiting (B) don't work with rotating IPs. Geo-blocking (D) causes collateral damage to legitimate users.

**Key Concept:** [WAF Rate-Based Rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html)
</details>

### Question 21
**Scenario:** An organization needs to enforce that all new VPCs across all accounts have flow logs enabled and sent to a centralized log archive bucket. What is the MOST effective enforcement mechanism?

A. AWS Config rule with auto-remediation that creates flow logs when missing
B. SCP preventing VPC creation combined with a Service Catalog product that creates VPCs with flow logs
C. CloudFormation StackSets deploying flow logs to all VPCs nightly
D. Training developers to always enable flow logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SCPs provide preventive controls - denying ec2:CreateVpc except via Service Catalog ensures all VPCs are created through the approved product that includes flow log configuration. This prevents non-compliant resources from ever being created. Config auto-remediation (A) is reactive and creates a window of non-compliance. StackSets (C) are periodic, not continuous. Training (D) doesn't provide technical enforcement.

**Key Concept:** [Service Catalog for Governance](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/introduction.html)
</details>

---

## Domain 4: Identity and Access Management (Questions 22-27)

### Question 22
**Scenario:** A company is federating their on-premises Active Directory with AWS using AWS IAM Identity Center (successor to AWS SSO). They have 500+ employees who need different levels of access to 50 AWS accounts based on their AD group membership. What is the MOST scalable approach?

A. Create IAM users in each account matched to AD users
B. Configure IAM Identity Center with AD integration, create permission sets, and assign permission sets to AD groups for account access
C. Use SAML 2.0 federation with IAM roles in each account mapped to AD groups
D. Sync AD users to Cognito and use Cognito for AWS console access

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM Identity Center provides centralized management of multi-account access with AD integration. Permission sets define access levels, and these are assigned to AD groups for specific accounts. Users authenticate via AD, and Identity Center handles temporary credential generation. This scales to hundreds of accounts and thousands of users. IAM users (A) don't federate. SAML federation (C) requires role management in each account. Cognito (D) isn't designed for AWS console federation.

**Key Concept:** [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
</details>

### Question 23
**Scenario:** An application needs to access AWS services from both EC2 instances and on-premises servers. The on-premises servers cannot use IAM roles. The credentials must be short-lived and automatically rotated. What solution meets these requirements?

A. Create IAM users with programmatic access and rotate access keys monthly
B. Use IAM Roles Anywhere with private certificate authority for on-premises, IAM roles for EC2
C. Store long-term credentials in Secrets Manager with automatic rotation
D. Use AWS STS directly from on-premises servers with hard-coded credentials

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM Roles Anywhere allows workloads outside AWS to obtain temporary credentials by presenting X.509 certificates issued by your CA. This provides the same short-lived, automatically rotated credentials as IAM roles without storing long-term secrets. EC2 instances use instance roles. Option A has long-lived credentials. Secrets Manager (C) still stores credentials. STS requires existing credentials to call (D).

**Key Concept:** [IAM Roles Anywhere](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html)
</details>

### Question 24
**Scenario:** A company wants to implement the principle of least privilege for their Lambda functions. Each function should only have access to the specific DynamoDB tables and S3 buckets it needs. Currently, all functions share a single IAM role with broad permissions. What is the BEST approach to remediate this?

A. Create separate IAM roles for each Lambda function with resource-level permissions
B. Add IAM policy conditions based on the Lambda function name
C. Use resource-based policies on DynamoDB and S3 to restrict Lambda access
D. Implement application-level access controls in the Lambda code

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Each Lambda function should have its own execution role with permissions scoped to only the resources it needs. This follows least privilege and blast radius reduction. Resource-level permissions specify exact table/bucket ARNs. Policy conditions (B) are complex and error-prone. Resource-based policies (C) work for S3 but DynamoDB doesn't support them for Lambda. Application controls (D) don't restrict IAM permissions.

**Key Concept:** [Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
</details>

### Question 25
**Scenario:** An IAM policy is attached to a user granting full S3 access, but the user cannot access a specific bucket. The bucket has no bucket policy. The user is in an account that's part of AWS Organizations with SCPs applied. What is the MOST likely cause?

A. The IAM policy has a typo in the bucket name
B. An SCP at the OU or root level is denying S3 access
C. The bucket has server-side encryption that the user cannot access
D. S3 Block Public Access is preventing access

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SCPs set maximum permissions for accounts in an organization. Even if the IAM policy grants full S3 access, an SCP denying S3 actions would prevent access. SCPs are evaluated before IAM policies. Check SCPs at the OU level, parent OUs, and the organization root. Block Public Access (D) doesn't affect authenticated IAM users. Encryption (C) would show a different error. Typos (A) are possible but less common with existing buckets.

**Key Concept:** [SCP Evaluation Logic](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
</details>

### Question 26
**Scenario:** A security audit revealed that several IAM users have not used their credentials in over 90 days. Some have both console and programmatic access. The security team wants to automatically identify and disable inactive credentials without manual review of each user. What is the MOST efficient solution?

A. Use IAM credential reports and manually disable inactive credentials monthly
B. Use IAM Access Analyzer unused access findings with automated remediation via Lambda
C. Create a Lambda function that calls GetAccessKeyLastUsed and DisableKey APIs weekly
D. Use AWS Config rules for IAM user credential age

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM Access Analyzer's unused access analysis continuously identifies credentials (passwords and access keys) not used within a specified period. Findings can trigger EventBridge rules for automated remediation (disabling credentials via Lambda). This is a managed service approach. Credential reports (A) are manual. Custom Lambda (C) duplicates Access Analyzer functionality. Config rules (D) check age, not last used date.

**Key Concept:** [IAM Access Analyzer Unused Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-unused-access.html)
</details>

### Question 27
**Scenario:** A cross-account setup requires Account A's Lambda function to write to Account B's DynamoDB table. Account B's security team wants to ensure only this specific Lambda function from Account A can access the table, not any other resources from Account A. How should this be configured?

A. Create an IAM role in Account B that Account A's Lambda assumes, with trust policy specifying the Lambda execution role ARN
B. Use DynamoDB resource-based policy to allow Account A access
C. Create an IAM user in Account B and share credentials with Account A
D. Enable cross-account access in DynamoDB table settings

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** The trust policy on Account B's IAM role should specify the exact ARN of Account A's Lambda execution role as the principal. This ensures only that specific Lambda function can assume the role. The role's permissions policy grants DynamoDB write access. DynamoDB (B) doesn't have resource-based policies like S3. Shared credentials (C) violate security best practices. There's no cross-account setting (D) in DynamoDB.

**Key Concept:** [Cross-Account IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)
</details>

---

## Domain 5: Data Protection (Questions 28-34)

### Question 28
**Scenario:** A healthcare application stores PHI in DynamoDB. Compliance requires that all data be encrypted with customer-managed keys, the keys must be rotated annually, and key usage must be audited. Which configuration meets all requirements?

A. Use DynamoDB default encryption with AWS managed keys
B. Enable DynamoDB encryption with a customer managed KMS key, enable automatic rotation, and use CloudTrail to log key usage
C. Encrypt data in the application before storing in DynamoDB
D. Use DynamoDB encryption with AWS owned keys and application-level encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Customer managed KMS keys (CMKs) provide full control over key policies, rotation schedules, and usage auditing via CloudTrail. DynamoDB encryption with CMKs encrypts all table data at rest transparently. Automatic rotation creates new key material annually while maintaining access to data encrypted with previous versions. AWS managed keys (A) don't allow custom rotation or access policies. Application-level (C) adds complexity and doesn't encrypt indexes. AWS owned keys (D) provide no visibility.

**Key Concept:** [DynamoDB Encryption at Rest](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html)
</details>

### Question 29
**Scenario:** An application needs to store database credentials that will be used by multiple Lambda functions. The credentials should be automatically rotated every 30 days, and rotation should not cause application downtime. Which solution provides this?

A. Store credentials in Parameter Store SecureString with manual rotation
B. Store credentials in Secrets Manager with automatic rotation using alternating users strategy
C. Store credentials in environment variables and update Lambda when rotating
D. Use IAM database authentication instead of credentials

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Secrets Manager with automatic rotation handles the complexity of credential rotation. The alternating users strategy creates two database users and rotates between them, ensuring one set of credentials is always valid during rotation (zero downtime). Lambda functions using the Secrets Manager SDK automatically get current credentials. Parameter Store (A) requires custom rotation logic. Environment variables (C) require redeployment. IAM auth (D) is ideal when supported, but isn't available for all databases.

**Key Concept:** [Secrets Manager Rotation Strategies](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)
</details>

### Question 30
**Scenario:** A company needs to encrypt S3 objects with different keys for different customers' data within the same bucket. Each customer should only be able to decrypt their own data. How should this be implemented?

A. Use SSE-S3 with different prefixes for each customer
B. Use SSE-KMS with a separate customer managed key per customer, with key policies granting access only to that customer
C. Use client-side encryption with customer-provided keys
D. Create separate S3 buckets for each customer with SSE-S3

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SSE-KMS allows specifying different KMS keys for different objects. Creating a CMK per customer with key policies restricting access to that customer's IAM principals ensures cryptographic separation. Even if a customer can access the S3 object, they cannot decrypt another customer's data without their KMS key. SSE-S3 (A) uses a single key per bucket. Client-side (C) adds application complexity. Separate buckets (D) don't scale well.

**Key Concept:** [S3 Server-Side Encryption with KMS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)
</details>

### Question 31
**Scenario:** An organization needs to share encrypted EBS snapshots with a partner AWS account. The snapshots are encrypted with a customer managed KMS key. What steps are required to enable the partner to use these snapshots?

A. Share the snapshot and share the KMS key by adding the partner account to the key policy
B. Copy the snapshot using an AWS managed key, then share the copy
C. Share the snapshot, modify the KMS key policy to allow the partner's account, and create a grant for the partner to use the key
D. Export the snapshot to S3, share via cross-account bucket access

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Sharing encrypted snapshots requires both sharing the snapshot (ModifySnapshotAttribute) and providing access to the KMS key. The key policy must allow the partner account as a principal, and a grant should be created specifying the partner account with permissions to use the key for EBS operations. Option A is incomplete without the grant. Option B loses the original encryption. Snapshots can't be exported to S3 (D).

**Key Concept:** [Sharing Encrypted Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modifying-snapshot-permissions.html)
</details>

### Question 32
**Scenario:** A security team needs to discover and classify sensitive data (PII, financial data) across hundreds of S3 buckets. They need ongoing monitoring for new sensitive data uploads and integration with their security workflow. Which service provides this?

A. S3 Storage Lens with custom metrics
B. Amazon Macie with automated sensitive data discovery and Security Hub integration
C. AWS Config rules checking for S3 object tags
D. Amazon Inspector scanning S3 buckets

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon Macie is purpose-built for sensitive data discovery in S3. It uses ML and pattern matching to identify PII, PHI, financial data, and credentials. Automated discovery jobs continuously scan for sensitive data. Findings integrate with Security Hub for centralized security workflow. Storage Lens (A) is for storage analytics. Config (C) requires manual tagging. Inspector (D) scans for vulnerabilities, not data classification.

**Key Concept:** [Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)
</details>

### Question 33
**Scenario:** An application uses AWS KMS for encryption. The security team wants to ensure that specific KMS operations (like ScheduleKeyDeletion and DisableKey) can only be performed by a senior administrator role, even if other administrators have full KMS permissions. How can this be enforced?

A. Use KMS key policies to deny these operations to all except the senior admin role
B. Create an SCP denying these operations organization-wide, with an exception for the senior admin role
C. Use IAM permission boundaries to restrict KMS operations
D. Enable MFA delete on KMS keys

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** KMS key policies are resource-based policies that have explicit precedence. A policy statement explicitly denying ScheduleKeyDeletion and DisableKey to all principals except the senior admin role ARN will be enforced regardless of IAM permissions. The key policy is the primary authorization mechanism for KMS keys. SCPs (B) apply to accounts, not specific resources within accounts. Permission boundaries (C) apply to users/roles, not resources. MFA delete (D) isn't a KMS feature.

**Key Concept:** [KMS Key Policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)
</details>

### Question 34
**Scenario:** A company needs to encrypt data with keys that have never been and will never be accessible to AWS, including during encryption/decryption operations. The keys must remain in their on-premises HSM. How can they achieve this?

A. Use AWS KMS with external key store (XKS) connected to their HSM
B. Use AWS CloudHSM clusters
C. Use client-side encryption with keys stored on-premises
D. Use AWS KMS customer managed keys with imported key material

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** KMS External Key Store (XKS) allows you to use KMS with keys stored in HSMs outside of AWS. All cryptographic operations are performed by your external HSM - the key material never enters AWS. This meets "hold your own key" (HYOK) requirements. CloudHSM (B) keys are in AWS-hosted hardware. Client-side (C) doesn't integrate with AWS services that require KMS. Imported key material (D) copies keys into KMS.

**Key Concept:** [KMS External Key Store](https://docs.aws.amazon.com/kms/latest/developerguide/keystore-external.html)
</details>

---

## Domain 6: Management and Security Governance (Questions 35-40)

### Question 35
**Scenario:** An organization is implementing a multi-account strategy using AWS Control Tower. They need to ensure all accounts have a consistent security baseline including CloudTrail logging, Config enabled, and GuardDuty active. New accounts should automatically receive this configuration. What is the MOST efficient approach?

A. Create CloudFormation templates and deploy via StackSets to all accounts
B. Use Control Tower Account Factory with custom blueprints and mandatory guardrails
C. Write Lambda functions triggered by account creation events
D. Manually configure each new account following a runbook

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Control Tower Account Factory with customizations (Account Factory for Terraform or Customizations for Control Tower) allows defining a baseline that's automatically applied to new accounts. Mandatory guardrails enforce security controls. Account Factory integrates with Service Catalog for self-service provisioning with guardrails. StackSets (A) require management and don't integrate with Account Factory. Lambda (C) adds complexity. Manual (D) doesn't scale.

**Key Concept:** [Control Tower Account Factory](https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html)
</details>

### Question 36
**Scenario:** A compliance audit requires documentation that all EC2 instances have been patched within 30 days of critical security patches being released. The organization uses AWS Systems Manager for patch management. How can they generate this compliance documentation?

A. Export CloudTrail logs showing patch commands
B. Use Systems Manager Patch Manager compliance reports and AWS Config conformance packs
C. Create custom CloudWatch dashboards showing patch status
D. Manually document patches in a spreadsheet

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Patch Manager provides compliance reports showing patch status per instance, including which patches are missing and for how long. AWS Config can record patch compliance state over time. Conformance packs can validate instances meet the 30-day patching requirement. This provides auditable, historical compliance data. CloudTrail (A) shows actions, not compliance state. CloudWatch (C) is for metrics, not compliance. Manual documentation (D) isn't reliable.

**Key Concept:** [Systems Manager Patch Compliance](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-compliance-about.html)
</details>

### Question 37
**Scenario:** A security team needs to evaluate their AWS environment against CIS AWS Foundations Benchmark and PCI DSS requirements. They want automated checks, centralized findings, and the ability to track remediation progress. What solution provides this?

A. Use AWS Audit Manager to assess CIS and PCI compliance
B. Enable Security Hub with CIS AWS Foundations and PCI DSS security standards
C. Deploy third-party compliance scanning tools
D. Create custom AWS Config rules for each control

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Security Hub provides pre-built security standards including CIS AWS Foundations Benchmark and PCI DSS. It automatically runs checks, aggregates findings with severity ratings, and tracks compliance scores over time. Findings can be integrated with ticketing systems for remediation tracking. Audit Manager (A) is for audit evidence collection, not automated compliance checks. Custom Config rules (D) require significant development effort. Third-party (C) adds cost.

**Key Concept:** [Security Hub Security Standards](https://docs.aws.amazon.com/securityhub/latest/userguide/standards-reference.html)
</details>

### Question 38
**Scenario:** An organization wants to implement a "break glass" process allowing emergency access to production accounts that bypasses normal approval workflows. The access should be heavily logged and automatically revoked after 4 hours. How should this be implemented?

A. Store root account credentials in a sealed envelope
B. Create a break glass IAM role with time-limited session duration, require MFA, and enable CloudTrail logging
C. Use AWS IAM Identity Center with an emergency access permission set, time-bound, requiring MFA and manager approval
D. Provide production credentials to on-call engineers

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** IAM Identity Center supports emergency access patterns with permission sets that can be assigned on-demand. Configure with short session duration (4 hours), require MFA, and integrate with an approval workflow (even if expedited). All access is logged via CloudTrail. The permission set can be revoked manually or automatically. Option B doesn't include approval workflow. Root credentials (A) are too powerful. Sharing credentials (D) violates security best practices.

**Key Concept:** [IAM Identity Center Emergency Access](https://docs.aws.amazon.com/singlesignon/latest/userguide/emergency-access.html)
</details>

### Question 39
**Scenario:** A company uses multiple AWS services that each have their own logging capabilities (CloudTrail, VPC Flow Logs, ALB access logs, S3 access logs, RDS logs). They need a unified security analysis platform that allows correlation across all log types for threat detection. What architecture should they implement?

A. Send all logs to CloudWatch Logs and use Logs Insights for queries
B. Send all logs to S3, catalog with Glue, and query with Athena
C. Send all logs to Amazon OpenSearch Service with SIEM capabilities and dashboards
D. Send all logs to Security Hub for analysis

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** OpenSearch Service with SIEM (Security Information and Event Management) capabilities provides log aggregation, correlation, real-time analysis, alerting, and dashboards. It can ingest all AWS log types and correlate events across services for threat detection. This is the standard architecture for security operations centers. CloudWatch Insights (A) has query limits and isn't designed for SIEM workloads. Athena (B) is good for ad-hoc but not real-time correlation. Security Hub (D) aggregates findings, not raw logs.

**Key Concept:** [SIEM on AWS](https://aws.amazon.com/solutions/implementations/centralized-logging-with-opensearch/)
</details>

### Question 40
**Scenario:** An AWS environment has accumulated over 5 years of configuration drift, undocumented changes, and potential security misconfigurations. The security team wants to assess the current state against security best practices and identify the highest-risk findings to prioritize remediation. What is the fastest path to this assessment?

A. Manually review all resources in the AWS console
B. Run AWS Trusted Advisor and review security checks
C. Enable Security Hub with all security standards, review findings sorted by severity
D. Hire a third-party penetration testing firm

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Security Hub provides immediate, automated assessment against multiple security standards (CIS, PCI DSS, AWS Foundational Security Best Practices). Findings are prioritized by severity, enabling the team to focus on critical issues first. It aggregates findings from other AWS security services (GuardDuty, Inspector, Macie) for comprehensive view. Trusted Advisor (B) has limited security checks. Manual review (A) doesn't scale. Pen testing (D) focuses on exploitability, not configuration compliance.

**Key Concept:** [AWS Security Hub Best Practices](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp.html)
</details>
