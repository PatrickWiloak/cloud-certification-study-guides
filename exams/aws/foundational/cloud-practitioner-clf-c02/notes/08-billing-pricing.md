# AWS Billing, Pricing, and Support

This section covers AWS pricing models, cost management tools, and support plans - essential knowledge for the Cloud Practitioner exam (Domain 4: Billing, Pricing, and Support - 12%).

---

## AWS Pricing Models

### EC2 Pricing Options

#### On-Demand Instances
- **Pay-as-you-go** with no upfront commitment
- Billed per second (minimum 60 seconds) for Linux, per hour for Windows
- **Best for:** Unpredictable workloads, short-term projects, testing
- **Cost:** Highest cost per hour, but most flexible

#### Reserved Instances (RIs)
- **Commit to 1 or 3 years** for significant discounts
- **Payment options:**
  - All Upfront: Maximum discount (up to 72%)
  - Partial Upfront: Medium discount
  - No Upfront: Smallest discount but flexible cash flow
- **Types:**
  - Standard RIs: Can't change instance type
  - Convertible RIs: Can change instance type (smaller discount)
- **Best for:** Steady-state, predictable workloads

#### Spot Instances
- **Up to 90% discount** on spare EC2 capacity
- AWS can reclaim with 2-minute warning
- **Best for:** Fault-tolerant, flexible workloads (batch processing, big data, testing)
- **Not suitable for:** Critical applications, databases

#### Dedicated Hosts
- **Physical server** dedicated to your use
- **Best for:** Compliance requirements, software licensing (bring your own license)
- Most expensive option

#### Savings Plans
- **Commit to hourly spend** for 1 or 3 years
- **Compute Savings Plans:** Up to 66% off, applies to EC2, Lambda, Fargate
- **EC2 Instance Savings Plans:** Up to 72% off, specific to instance family
- More flexible than Reserved Instances

**[📖 EC2 Pricing](https://aws.amazon.com/ec2/pricing/)**

---

### S3 Pricing

S3 charges for:
1. **Storage** - Per GB/month based on storage class
2. **Requests** - PUT, GET, LIST, etc.
3. **Data Transfer** - Data out to internet (ingress is free)
4. **Management Features** - Inventory, analytics, Object Lock

#### S3 Storage Classes
| Storage Class | Use Case | Cost |
|--------------|----------|------|
| S3 Standard | Frequently accessed data | $$$ |
| S3 Intelligent-Tiering | Unknown access patterns | $$ + monitoring fee |
| S3 Standard-IA | Infrequent access (min 30 days) | $$ |
| S3 One Zone-IA | Infrequent, non-critical data | $ |
| S3 Glacier Instant Retrieval | Archive with millisecond access | $ |
| S3 Glacier Flexible Retrieval | Archive (3-5 hours retrieval) | ¢ |
| S3 Glacier Deep Archive | Long-term archive (12-48 hours) | ¢¢ (lowest) |

**[📖 S3 Pricing](https://aws.amazon.com/s3/pricing/)**

---

### Data Transfer Pricing

**Key principle:**
- **Data IN to AWS** = Generally FREE
- **Data OUT from AWS** = CHARGED (per GB)
- **Data between services in same Region** = Usually free or low cost
- **Data between Regions** = CHARGED

---

### Lambda Pricing

- **Requests:** First 1 million free, then $0.20 per million
- **Duration:** Charged per GB-second of compute time
- **Free tier:** 1 million requests and 400,000 GB-seconds per month (always free)

**[📖 Lambda Pricing](https://aws.amazon.com/lambda/pricing/)**

---

### RDS Pricing

Charges for:
- **Instance hours** - DB instance running time
- **Database storage** - Per GB/month
- **Backup storage** - Above provisioned storage
- **Data transfer** - Data out
- **Provisioned IOPS** - If using io1 storage

---

## AWS Free Tier

### Three Types of Free Offers

1. **Always Free**
   - Never expires
   - Examples: Lambda (1M requests/month), DynamoDB (25GB)

2. **12 Months Free**
   - Free for 12 months after account creation
   - Examples: EC2 (750 hours/month t2.micro), S3 (5GB), RDS (750 hours)

3. **Trials**
   - Short-term free trials for specific services
   - Examples: Redshift (2-month trial), Inspector (90-day trial)

**[📖 AWS Free Tier](https://aws.amazon.com/free/)**

---

## Cost Management Tools

### AWS Pricing Calculator
- **Estimate costs** before deploying
- Create detailed cost estimates for AWS solutions
- Compare pricing across Regions
- Export and share estimates

**[📖 AWS Pricing Calculator](https://calculator.aws/)**

---

### AWS Cost Explorer
- **Analyze** past and current AWS costs
- Visualize spending patterns over time
- Forecast future costs (up to 12 months)
- Identify cost drivers
- Available at **no additional cost**

**Key features:**
- Filter by service, linked account, tag, etc.
- View daily, monthly, or hourly granularity
- Resource Optimization recommendations

**[📖 AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)**

---

### AWS Budgets
- Set **custom budgets** for costs, usage, or reservations
- Receive **alerts** when thresholds are exceeded
- Set alerts at actual or forecasted amounts
- Track RI/Savings Plan utilization

**Budget types:**
- Cost budgets
- Usage budgets
- Reservation budgets
- Savings Plans budgets

**[📖 AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/)**

---

### AWS Cost and Usage Report (CUR)
- Most **detailed** billing data available
- Hourly, daily, or monthly reports
- Delivered to S3 bucket
- Analyze with Athena, Redshift, or QuickSight

**[📖 Cost and Usage Reports](https://aws.amazon.com/aws-cost-management/aws-cost-and-usage-reporting/)**

---

### AWS Cost Allocation Tags
- **Tag resources** to organize and track costs
- Two types:
  - **AWS-generated tags:** Automatically applied (aws: prefix)
  - **User-defined tags:** Custom tags you create
- Enable tags in Billing console for cost allocation reports

---

## AWS Organizations

### Consolidated Billing
- **Single payment method** for all accounts in organization
- **Combined usage** across accounts for volume discounts
- One bill for multiple AWS accounts

### Benefits
- **Volume discounts:** Combined usage may qualify for lower rates
- **Reserved Instance sharing:** RIs can apply across accounts
- **Simplified billing:** Single invoice and payment
- **Service Control Policies (SCPs):** Centralized permission guardrails

**[📖 AWS Organizations](https://aws.amazon.com/organizations/)**

---

## AWS Support Plans

### Basic Support (Free)
- **Included with all AWS accounts**
- 24/7 access to customer service
- AWS Trusted Advisor - 7 core checks
- AWS Personal Health Dashboard
- Documentation and forums

### Developer Support ($29/month or 3% of usage)
- Everything in Basic, plus:
- Business hours email support
- 1 primary contact
- Response times: < 24 hours (general), < 12 hours (system impaired)
- **Best for:** Development and testing

### Business Support ($100/month or 10% of usage)
- Everything in Developer, plus:
- **24/7 phone, email, chat support**
- Unlimited contacts
- Response times: < 24 hours (general), < 12 hours (system impaired), < 4 hours (production down), < 1 hour (production down critical)
- **Full Trusted Advisor checks**
- AWS Support API
- Third-party software support
- **Best for:** Production workloads

### Enterprise On-Ramp ($5,500/month)
- Everything in Business, plus:
- Technical Account Manager (TAM) pool
- Concierge Support Team
- Infrastructure Event Management
- Response time: < 30 minutes (business-critical)
- **Best for:** Production/business-critical workloads

### Enterprise Support ($15,000/month or more)
- Everything in Enterprise On-Ramp, plus:
- **Designated Technical Account Manager (TAM)**
- Response time: < 15 minutes (business-critical)
- Operations Reviews
- Training credits
- **Best for:** Mission-critical workloads

**[📖 AWS Support Plans](https://aws.amazon.com/premiumsupport/plans/)**

---

### Technical Account Manager (TAM)
- **Available only with Enterprise support**
- Provides proactive guidance and advocacy
- Helps with:
  - Architecture reviews
  - Well-Architected reviews
  - Operations reviews
  - Incident management
  - Cost optimization

---

## AWS Trusted Advisor

### What It Does
- **Automated recommendations** across five categories:
  1. **Cost Optimization** - Idle resources, Reserved Instance optimization
  2. **Performance** - Over-utilized resources
  3. **Security** - Security group rules, MFA, IAM usage
  4. **Fault Tolerance** - Backups, Multi-AZ, redundancy
  5. **Service Limits** - Approaching service quotas

### Access Levels
| Support Plan | Trusted Advisor Checks |
|-------------|----------------------|
| Basic/Developer | 7 core checks (S3 permissions, security groups, IAM, MFA, etc.) |
| Business/Enterprise | **All checks** (50+) |

**[📖 AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)**

---

## Other Billing Concepts

### Total Cost of Ownership (TCO)
- Compare on-premises vs AWS costs
- Include: Hardware, software, networking, facilities, operations
- AWS typically reduces TCO through:
  - No upfront investment
  - Pay-as-you-go pricing
  - Reduced operational overhead
  - Elasticity (right-sizing)

### AWS Professional Services
- **Paid consulting** from AWS experts
- Help with cloud adoption, migrations, architecture
- Not included in any support plan

### AWS Partner Network (APN)
- **Third-party consultants and software providers**
- Two types:
  - Consulting Partners: Professional services firms
  - Technology Partners: Software providers (ISVs)

---

## Exam Tips

### Key Points to Remember
1. **Data transfer IN is generally free**, data OUT is charged
2. **Spot Instances** = Up to 90% discount, can be interrupted
3. **Reserved Instances** = Up to 72% discount, 1 or 3 year commitment
4. **Savings Plans** = Flexible alternative to RIs
5. **Cost Explorer** = Analyze past costs and forecast future
6. **AWS Budgets** = Set alerts for spending thresholds
7. **TAM** = Only available with Enterprise support
8. **Trusted Advisor full checks** = Business/Enterprise support only
9. **Consolidated Billing** = Combined usage may qualify for volume discounts
10. **S3 Glacier Deep Archive** = Lowest cost storage

### Common Exam Questions
- Which pricing model for unpredictable workloads? **On-Demand**
- Which pricing model for steady-state workloads? **Reserved Instances**
- Which pricing model for fault-tolerant batch jobs? **Spot Instances**
- How to get alerted on spending? **AWS Budgets**
- How to analyze historical costs? **Cost Explorer**
- What support plan includes TAM? **Enterprise**
- How to get volume discounts across accounts? **AWS Organizations with Consolidated Billing**

---

## Documentation Links

- **[📖 AWS Pricing](https://aws.amazon.com/pricing/)**
- **[📖 AWS Free Tier](https://aws.amazon.com/free/)**
- **[📖 Cost Management](https://aws.amazon.com/aws-cost-management/)**
- **[📖 Support Plans Comparison](https://aws.amazon.com/premiumsupport/plans/)**
- **[📖 Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)**
