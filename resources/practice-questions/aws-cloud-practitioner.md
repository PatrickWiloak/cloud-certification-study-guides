# AWS Cloud Practitioner (CLF-C02) Practice Questions

40 practice questions covering all exam domains.

**Exam Domain Breakdown:**
- Domain 1: Cloud Concepts (24%) - 10 questions
- Domain 2: Security and Compliance (30%) - 10 questions
- Domain 3: Cloud Technology and Services (34%) - 12 questions
- Domain 4: Billing, Pricing, and Support (12%) - 8 questions

---

## Domain 1: Cloud Concepts (24%)

### Question 1
**Scenario:** A startup wants to launch a new application but doesn't want to invest in physical servers or predict capacity needs upfront. They want to pay only for the resources they actually use.

A. Traditional on-premises data center
B. Colocation hosting
C. Cloud computing
D. Dedicated hosting

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Cloud computing enables on-demand access to computing resources with pay-as-you-go pricing, eliminating the need for upfront capital investment. On-premises, colocation, and dedicated hosting all require capacity planning and upfront investment.

**Key Concept:** [What is Cloud Computing?](https://aws.amazon.com/what-is-cloud-computing/)
</details>

---

### Question 2
**Scenario:** A company wants to run their applications in AWS but maintain complete control over the operating system, including patches and security updates.

A. Software as a Service (SaaS)
B. Platform as a Service (PaaS)
C. Infrastructure as a Service (IaaS)
D. Function as a Service (FaaS)

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** IaaS provides virtualized computing resources where the customer manages the operating system, middleware, and applications. SaaS provides complete applications. PaaS manages the OS for you. FaaS abstracts infrastructure entirely.

**Key Concept:** [Cloud Computing Models](https://aws.amazon.com/types-of-cloud-computing/)
</details>

---

### Question 3
**Scenario:** A company is evaluating whether to migrate to AWS. They currently spend $500,000 annually on data center operations including staff, power, cooling, and hardware maintenance. Which cloud benefit addresses this?

A. High availability
B. Elasticity
C. Trade capital expense for variable expense
D. Global reach

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Cloud computing allows companies to replace large capital investments (CapEx) in data centers with variable operational expenses (OpEx) that scale with usage. This eliminates the need for upfront hardware purchases and ongoing maintenance costs.

**Key Concept:** [Six Advantages of Cloud Computing](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/six-advantages-of-cloud-computing.html)
</details>

---

### Question 4
**Scenario:** An e-commerce website experiences 10x normal traffic during holiday sales but returns to baseline afterward. What cloud computing benefit helps handle this situation cost-effectively?

A. Fault tolerance
B. Elasticity
C. Disaster recovery
D. Security

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Elasticity is the ability to automatically scale resources up during high demand and scale down during low demand. This ensures you only pay for what you use while meeting demand spikes.

**Key Concept:** [Elasticity](https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.concept.elasticity.en.html)
</details>

---

### Question 5
**Scenario:** A company needs to deploy their application closer to users in Asia-Pacific to reduce latency. Which AWS concept enables this?

A. Availability Zones
B. Edge Locations
C. AWS Regions
D. Local Zones

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS Regions are geographic locations around the world where AWS has data centers. Deploying to an Asia-Pacific region brings the application closer to users in that area. Availability Zones are within regions. Edge Locations are for content delivery. Local Zones extend regions for specific cities.

**Key Concept:** [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)
</details>

---

### Question 6
**Scenario:** A company requires their application to remain available even if a single data center experiences a failure. What should they use?

A. Single Availability Zone deployment
B. Multiple Availability Zones in one Region
C. Edge Locations
D. AWS Outposts

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Availability Zones are physically separate data centers within a Region with independent power, cooling, and networking. Deploying across multiple AZs provides high availability and fault tolerance against single data center failures.

**Key Concept:** [Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
</details>

---

### Question 7
**Scenario:** Which pillar of the AWS Well-Architected Framework focuses on protecting information, systems, and assets while delivering business value through risk assessment and mitigation?

A. Operational Excellence
B. Security
C. Reliability
D. Performance Efficiency

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** The Security pillar focuses on protecting data, systems, and assets through risk assessment and mitigation strategies. Operational Excellence focuses on operations. Reliability focuses on recovery and meeting demand. Performance Efficiency focuses on using resources efficiently.

**Key Concept:** [Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
</details>

---

### Question 8
**Scenario:** A company wants to avoid guessing infrastructure capacity needs and instead scale resources based on actual demand. Which cloud benefit does this describe?

A. Agility
B. Elasticity
C. Economy of scale
D. Global deployment

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Elasticity allows you to scale computing resources up or down based on actual demand, eliminating the need to guess capacity. Agility refers to speed of deploying resources. Economy of scale relates to cost benefits from AWS's size.

**Key Concept:** [Benefits of Cloud Computing](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/six-advantages-of-cloud-computing.html)
</details>

---

### Question 9
**Scenario:** Which deployment model combines on-premises infrastructure with cloud resources, allowing data and applications to be shared between them?

A. Public cloud
B. Private cloud
C. Hybrid cloud
D. Multi-cloud

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Hybrid cloud combines on-premises (private) infrastructure with public cloud resources, allowing organizations to keep sensitive data on-premises while leveraging cloud scalability. Multi-cloud uses multiple public cloud providers.

**Key Concept:** [Hybrid Cloud](https://aws.amazon.com/hybrid/)
</details>

---

### Question 10
**Scenario:** A company can deploy updates to their application in minutes instead of weeks. Which cloud benefit enables this?

A. Cost optimization
B. Elasticity
C. Agility
D. Global reach

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Agility in cloud computing refers to the ability to quickly provision resources, deploy applications, and iterate on changes. This reduces time from weeks to minutes for deployments and updates.

**Key Concept:** [Cloud Agility](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/six-advantages-of-cloud-computing.html)
</details>

---

## Domain 2: Security and Compliance (30%)

### Question 11
**Scenario:** Under the AWS Shared Responsibility Model, who is responsible for patching the operating system on an Amazon EC2 instance?

A. AWS
B. The customer
C. Shared between AWS and customer
D. Third-party security provider

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Under the Shared Responsibility Model, customers are responsible for "security IN the cloud," which includes patching guest operating systems on EC2 instances. AWS is responsible for "security OF the cloud" - the underlying infrastructure.

**Key Concept:** [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
</details>

---

### Question 12
**Scenario:** A company needs to grant temporary access to an external auditor to review their AWS resources. What is the MOST secure approach?

A. Create a permanent IAM user with full access
B. Share the root account credentials
C. Create an IAM role with specific permissions
D. Create an IAM user with no MFA

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** IAM roles provide temporary credentials and can be assumed when needed. This follows the principle of least privilege and avoids creating permanent credentials. Never share root credentials, and always use MFA.

**Key Concept:** [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
</details>

---

### Question 13
**Scenario:** Which AWS service provides a centralized view of security alerts and compliance status across multiple AWS accounts?

A. AWS CloudTrail
B. Amazon GuardDuty
C. AWS Security Hub
D. Amazon Inspector

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS Security Hub provides a comprehensive view of security alerts and compliance status across AWS accounts. CloudTrail logs API calls. GuardDuty detects threats. Inspector assesses vulnerabilities in EC2 instances.

**Key Concept:** [AWS Security Hub](https://aws.amazon.com/security-hub/)
</details>

---

### Question 14
**Scenario:** A company needs to encrypt data stored in Amazon S3. Who is responsible for enabling this encryption?

A. AWS enables it automatically
B. The customer must enable it
C. Third-party tools are required
D. S3 data cannot be encrypted

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** While AWS provides encryption capabilities (SSE-S3, SSE-KMS, SSE-C), the customer is responsible for enabling and configuring encryption under the Shared Responsibility Model. Note: AWS now enables SSE-S3 by default for new buckets, but customers still manage encryption settings.

**Key Concept:** [S3 Encryption](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-encryption.html)
</details>

---

### Question 15
**Scenario:** Which service should be used to manage encryption keys for AWS services?

A. AWS Secrets Manager
B. AWS Key Management Service (KMS)
C. AWS Certificate Manager
D. Amazon Cognito

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS KMS is designed to create and manage encryption keys used to encrypt data across AWS services. Secrets Manager stores secrets like passwords. Certificate Manager manages SSL/TLS certificates. Cognito handles user authentication.

**Key Concept:** [AWS KMS](https://aws.amazon.com/kms/)
</details>

---

### Question 16
**Scenario:** Which service provides intelligent threat detection by analyzing AWS CloudTrail logs, VPC Flow Logs, and DNS logs?

A. AWS WAF
B. Amazon GuardDuty
C. AWS Shield
D. Amazon Macie

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon GuardDuty uses machine learning to analyze logs and detect threats like compromised instances, reconnaissance, and account compromise. WAF protects web applications. Shield protects against DDoS. Macie discovers sensitive data.

**Key Concept:** [Amazon GuardDuty](https://aws.amazon.com/guardduty/)
</details>

---

### Question 17
**Scenario:** A developer needs to store database credentials securely and have them automatically rotated. Which service should they use?

A. AWS Systems Manager Parameter Store
B. AWS Secrets Manager
C. AWS KMS
D. Amazon S3 with encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Secrets Manager is specifically designed for storing, rotating, and managing secrets like database credentials. It supports automatic rotation. Parameter Store stores configuration data but has limited rotation capabilities. KMS manages encryption keys, not secrets.

**Key Concept:** [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
</details>

---

### Question 18
**Scenario:** Which practice should be followed for the AWS root account?

A. Use it for daily administrative tasks
B. Share it among the IT team
C. Enable MFA and avoid using it for routine tasks
D. Delete it after creating IAM users

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** The root account has unrestricted access and should be protected with MFA. Use IAM users for routine tasks. Never share root credentials. The root account cannot be deleted as it's tied to the AWS account.

**Key Concept:** [Root Account Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
</details>

---

### Question 19
**Scenario:** What type of attack does AWS Shield protect against?

A. SQL injection
B. Cross-site scripting (XSS)
C. Distributed Denial of Service (DDoS)
D. Man-in-the-middle

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS Shield provides protection against DDoS attacks. Shield Standard is free and protects against common attacks. Shield Advanced provides additional protection and 24/7 support. WAF protects against SQL injection and XSS.

**Key Concept:** [AWS Shield](https://aws.amazon.com/shield/)
</details>

---

### Question 20
**Scenario:** Which AWS service records API calls made in your AWS account for auditing purposes?

A. Amazon CloudWatch
B. AWS CloudTrail
C. AWS Config
D. AWS X-Ray

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS CloudTrail records API calls made in your account, providing an audit trail of who did what and when. CloudWatch monitors metrics. Config tracks resource configurations. X-Ray traces application requests.

**Key Concept:** [AWS CloudTrail](https://aws.amazon.com/cloudtrail/)
</details>

---

## Domain 3: Cloud Technology and Services (34%)

### Question 21
**Scenario:** A company needs to run a web server that requires a specific operating system and custom software. Which AWS service should they use?

A. AWS Lambda
B. Amazon EC2
C. Amazon S3
D. Amazon RDS

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon EC2 provides virtual servers where you control the operating system and can install custom software. Lambda is serverless (no OS access). S3 is storage. RDS is managed database service.

**Key Concept:** [Amazon EC2](https://aws.amazon.com/ec2/)
</details>

---

### Question 22
**Scenario:** A developer needs to run code in response to events without managing servers. Which service should they use?

A. Amazon EC2
B. Amazon ECS
C. AWS Lambda
D. Amazon Lightsail

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS Lambda runs code in response to events without provisioning or managing servers. You pay only for compute time consumed. EC2 requires server management. ECS runs containers. Lightsail is simplified compute.

**Key Concept:** [AWS Lambda](https://aws.amazon.com/lambda/)
</details>

---

### Question 23
**Scenario:** A company needs to store and retrieve any amount of data from anywhere on the web. Which service provides this capability?

A. Amazon EBS
B. Amazon S3
C. Amazon EFS
D. AWS Storage Gateway

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon S3 is object storage designed to store and retrieve any amount of data from anywhere. It's highly durable (99.999999999%). EBS is block storage for EC2. EFS is file storage. Storage Gateway connects on-premises to cloud storage.

**Key Concept:** [Amazon S3](https://aws.amazon.com/s3/)
</details>

---

### Question 24
**Scenario:** A company needs block storage for an EC2 instance that persists independently of the instance lifecycle. Which service should they use?

A. Amazon S3
B. Instance Store
C. Amazon EBS
D. Amazon Glacier

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Amazon EBS provides persistent block storage that exists independently of EC2 instances. Instance Store is ephemeral (data lost when instance stops). S3 is object storage. Glacier is archival storage.

**Key Concept:** [Amazon EBS](https://aws.amazon.com/ebs/)
</details>

---

### Question 25
**Scenario:** Which database service is fully managed and compatible with MySQL and PostgreSQL?

A. Amazon DynamoDB
B. Amazon Aurora
C. Amazon Redshift
D. Amazon ElastiCache

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon Aurora is a fully managed relational database compatible with MySQL and PostgreSQL, offering up to 5x better performance. DynamoDB is NoSQL. Redshift is data warehousing. ElastiCache is in-memory caching.

**Key Concept:** [Amazon Aurora](https://aws.amazon.com/rds/aurora/)
</details>

---

### Question 26
**Scenario:** A company needs a NoSQL database that provides single-digit millisecond performance at any scale. Which service should they use?

A. Amazon RDS
B. Amazon DynamoDB
C. Amazon Redshift
D. Amazon Aurora

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon DynamoDB is a fully managed NoSQL database providing single-digit millisecond latency at any scale. RDS and Aurora are relational databases. Redshift is for data warehousing analytics.

**Key Concept:** [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)
</details>

---

### Question 27
**Scenario:** Which service provides a virtual private network within AWS where you can launch resources in a logically isolated section?

A. AWS Direct Connect
B. Amazon VPC
C. AWS VPN
D. Amazon Route 53

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon VPC (Virtual Private Cloud) lets you provision a logically isolated section of AWS where you can launch resources in a virtual network you define. Direct Connect is dedicated network connection. VPN creates encrypted tunnels. Route 53 is DNS.

**Key Concept:** [Amazon VPC](https://aws.amazon.com/vpc/)
</details>

---

### Question 28
**Scenario:** Which service distributes incoming application traffic across multiple targets like EC2 instances?

A. Amazon Route 53
B. AWS Auto Scaling
C. Elastic Load Balancing
D. Amazon CloudFront

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Elastic Load Balancing automatically distributes incoming traffic across multiple targets. Route 53 is DNS. Auto Scaling adjusts capacity. CloudFront is a CDN for content delivery.

**Key Concept:** [Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/)
</details>

---

### Question 29
**Scenario:** A company wants to deliver content with low latency to users worldwide. Which service should they use?

A. Amazon S3
B. Amazon CloudFront
C. AWS Global Accelerator
D. Amazon Route 53

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon CloudFront is a CDN that caches content at edge locations worldwide, reducing latency for end users. S3 stores content but doesn't cache it at edge. Global Accelerator improves availability. Route 53 is DNS.

**Key Concept:** [Amazon CloudFront](https://aws.amazon.com/cloudfront/)
</details>

---

### Question 30
**Scenario:** Which service provides DNS and domain name registration?

A. Amazon CloudFront
B. Amazon Route 53
C. AWS Direct Connect
D. Elastic Load Balancing

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon Route 53 is a highly available DNS service that also provides domain registration. CloudFront is CDN. Direct Connect is dedicated network. ELB distributes traffic.

**Key Concept:** [Amazon Route 53](https://aws.amazon.com/route53/)
</details>

---

### Question 31
**Scenario:** A company needs to send notification messages to multiple subscribers including email, SMS, and HTTP endpoints. Which service should they use?

A. Amazon SQS
B. Amazon SNS
C. Amazon SES
D. AWS Step Functions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Amazon SNS (Simple Notification Service) is a pub/sub service that sends messages to multiple subscribers across various protocols. SQS is message queuing. SES is for sending emails. Step Functions orchestrates workflows.

**Key Concept:** [Amazon SNS](https://aws.amazon.com/sns/)
</details>

---

### Question 32
**Scenario:** Which service helps automatically adjust compute capacity to maintain steady, predictable performance at the lowest possible cost?

A. Elastic Load Balancing
B. AWS Auto Scaling
C. Amazon CloudWatch
D. AWS Lambda

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Auto Scaling monitors applications and automatically adjusts capacity to maintain performance. ELB distributes traffic but doesn't scale resources. CloudWatch monitors but doesn't scale. Lambda scales automatically but is serverless.

**Key Concept:** [AWS Auto Scaling](https://aws.amazon.com/autoscaling/)
</details>

---

## Domain 4: Billing, Pricing, and Support (12%)

### Question 33
**Scenario:** A company runs EC2 instances continuously for 3 years. Which pricing option would provide the MOST cost savings?

A. On-Demand Instances
B. Spot Instances
C. Reserved Instances (3-year term)
D. Dedicated Hosts

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Reserved Instances with a 3-year term provide up to 72% discount compared to On-Demand for steady-state workloads. Spot Instances are cheaper but can be interrupted. On-Demand has no commitment but highest cost. Dedicated Hosts are for compliance needs.

**Key Concept:** [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
</details>

---

### Question 34
**Scenario:** Which AWS support plan provides access to a Technical Account Manager (TAM)?

A. Basic
B. Developer
C. Business
D. Enterprise

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** Only the Enterprise Support plan includes a designated Technical Account Manager who provides proactive guidance. Basic has no technical support. Developer and Business have increasing support but no TAM.

**Key Concept:** [AWS Support Plans](https://aws.amazon.com/premiumsupport/plans/)
</details>

---

### Question 35
**Scenario:** Which tool helps estimate the cost of your AWS architecture before deployment?

A. AWS Cost Explorer
B. AWS Pricing Calculator
C. AWS Budgets
D. Cost and Usage Report

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Pricing Calculator helps estimate costs for AWS services before you deploy. Cost Explorer analyzes existing costs. Budgets sets alerts for spending. Cost and Usage Report provides detailed billing data.

**Key Concept:** [AWS Pricing Calculator](https://calculator.aws/)
</details>

---

### Question 36
**Scenario:** A company wants to be alerted when their AWS spending exceeds a certain threshold. Which service should they use?

A. AWS Cost Explorer
B. AWS Budgets
C. AWS Pricing Calculator
D. AWS Trusted Advisor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Budgets allows you to set custom spending thresholds and receive alerts when costs exceed them. Cost Explorer analyzes past spending. Pricing Calculator estimates future costs. Trusted Advisor provides best practice recommendations.

**Key Concept:** [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/)
</details>

---

### Question 37
**Scenario:** Which service provides recommendations to help reduce costs, improve security, and optimize performance?

A. AWS Cost Explorer
B. AWS CloudWatch
C. AWS Trusted Advisor
D. AWS Config

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** AWS Trusted Advisor inspects your AWS environment and provides recommendations across five categories: cost optimization, performance, security, fault tolerance, and service limits. Cost Explorer shows costs. CloudWatch monitors. Config tracks configurations.

**Key Concept:** [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)
</details>

---

### Question 38
**Scenario:** How does AWS charge for data transfer?

A. Data transfer into AWS is charged; data transfer out is free
B. Data transfer out of AWS is charged; data transfer in is generally free
C. Both data transfer in and out are charged equally
D. All data transfer is free

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS generally does not charge for data transfer into AWS (ingress), but charges for data transfer out of AWS (egress) to the internet. Data transfer between AWS services in the same region may also incur charges.

**Key Concept:** [AWS Pricing](https://aws.amazon.com/pricing/)
</details>

---

### Question 39
**Scenario:** A company uses multiple AWS services and wants to combine billing across all accounts. Which feature enables this?

A. AWS Cost Explorer
B. AWS Organizations with Consolidated Billing
C. AWS Budgets
D. AWS Cost and Usage Report

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Organizations with Consolidated Billing combines usage across all accounts in an organization into a single bill, potentially qualifying for volume discounts. Cost Explorer analyzes costs. Budgets sets alerts. Cost and Usage Report provides detailed data.

**Key Concept:** [Consolidated Billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html)
</details>

---

### Question 40
**Scenario:** Which AWS service or feature is available at no additional cost and helps you understand your AWS costs and usage?

A. AWS Cost Explorer (basic)
B. AWS Trusted Advisor (full checks)
C. Technical Account Manager
D. AWS Professional Services

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** AWS Cost Explorer is available to all AWS customers at no additional cost and provides visualization of costs and usage patterns. Full Trusted Advisor checks require Business or Enterprise support. TAM requires Enterprise support. Professional Services is paid consulting.

**Key Concept:** [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | C | Cloud Concepts |
| 2 | C | Cloud Concepts |
| 3 | C | Cloud Concepts |
| 4 | B | Cloud Concepts |
| 5 | C | Cloud Concepts |
| 6 | B | Cloud Concepts |
| 7 | B | Cloud Concepts |
| 8 | B | Cloud Concepts |
| 9 | C | Cloud Concepts |
| 10 | C | Cloud Concepts |
| 11 | B | Security |
| 12 | C | Security |
| 13 | C | Security |
| 14 | B | Security |
| 15 | B | Security |
| 16 | B | Security |
| 17 | B | Security |
| 18 | C | Security |
| 19 | C | Security |
| 20 | B | Security |
| 21 | B | Technology |
| 22 | C | Technology |
| 23 | B | Technology |
| 24 | C | Technology |
| 25 | B | Technology |
| 26 | B | Technology |
| 27 | B | Technology |
| 28 | C | Technology |
| 29 | B | Technology |
| 30 | B | Technology |
| 31 | B | Technology |
| 32 | B | Technology |
| 33 | C | Billing |
| 34 | D | Billing |
| 35 | B | Billing |
| 36 | B | Billing |
| 37 | C | Billing |
| 38 | B | Billing |
| 39 | B | Billing |
| 40 | A | Billing |
