# Cloud Free Tier Guide: Maximize Free Resources for Certification Study

A comprehensive guide to leveraging free tier offerings from major cloud providers (AWS, Azure, GCP, Oracle, IBM) for certification preparation and hands-on practice.

## Table of Contents
- [AWS Free Tier](#aws-free-tier)
- [Microsoft Azure Free Tier](#microsoft-azure-free-tier)
- [Google Cloud Platform Free Tier](#google-cloud-platform-free-tier)
- [Oracle Cloud Free Tier](#oracle-cloud-free-tier)
- [IBM Cloud Free Tier](#ibm-cloud-free-tier)
- [Cost Management Strategies](#cost-management-strategies)
- [Free Tier Best Practices](#free-tier-best-practices)
- [Multi-Cloud Strategy](#multi-cloud-strategy)
- [Study Projects Using Free Tiers](#study-projects-using-free-tiers)
- [Common Pitfalls and How to Avoid Them](#common-pitfalls-and-how-to-avoid-them)

---

## AWS Free Tier

**Website:** https://aws.amazon.com/free/

**Duration:** 12 months from account creation + Always Free services

**Credit Card Required:** Yes

### Free Tier Categories

**1. 12 Months Free** (from account creation)
**2. Always Free** (no expiration)
**3. Free Trials** (short-term trials for specific services)

---

### 12 Months Free Services

#### Compute

**EC2 (Elastic Compute Cloud)**
- **Limit:** 750 hours per month of t2.micro or t3.micro instances (Linux, RHEL, or SLES)
- **What You Can Do:**
  - Run one t2.micro instance 24/7 for a full month
  - Or run multiple instances that total 750 hours
  - Practice: Deploy web servers, application servers, practice SSH access
- **Watch Out:** Data transfer limits apply (15GB out per month)

**Elastic Load Balancing**
- **Limit:** 750 hours per month
- **Limit:** 15GB data processing
- **What You Can Do:**
  - Set up Application or Network Load Balancer
  - Practice: Load balance between multiple EC2 instances

---

#### Storage

**S3 (Simple Storage Service)**
- **Limit:** 5GB of standard storage
- **Limit:** 20,000 GET requests
- **Limit:** 2,000 PUT requests
- **What You Can Do:**
  - Host static websites
  - Store files, images, backups
  - Practice: Bucket policies, lifecycle rules, versioning

**EBS (Elastic Block Store)**
- **Limit:** 30GB of General Purpose (SSD) or Magnetic storage
- **Limit:** 2 million I/Os (with Magnetic)
- **Limit:** 1GB of snapshot storage
- **What You Can Do:**
  - Attach volumes to EC2 instances
  - Practice: Volume management, snapshots, encryption

**Elastic File System (EFS)**
- **Limit:** 5GB of standard storage
- **What You Can Do:**
  - Shared file system for EC2
  - Practice: Mount EFS on multiple instances

---

#### Database

**RDS (Relational Database Service)**
- **Limit:** 750 hours per month of db.t2.micro, db.t3.micro, or db.t4g.micro instances
- **Limit:** 20GB of General Purpose (SSD) storage
- **Limit:** 20GB of backup storage
- **Supported:** MySQL, PostgreSQL, MariaDB, Oracle BYOL, SQL Server Express
- **What You Can Do:**
  - Deploy MySQL, PostgreSQL databases
  - Practice: Backups, snapshots, read replicas, parameter groups

**DynamoDB**
- **Limit:** 25GB of storage
- **Limit:** 25 read/write capacity units
- **What You Can Do:**
  - Build NoSQL applications
  - Practice: Table design, queries, indexes, streams

**ElastiCache**
- **Limit:** 750 hours per month of cache.t2micro or cache.t3.micro nodes

---

#### Networking

**VPC (Virtual Private Cloud)**
- **Free:** No charge for VPC creation
- **Free:** 1 million VPN connection hours (first 12 months)
- **What You Can Do:**
  - Create custom networks
  - Practice: Subnets, route tables, security groups, NACLs

**CloudFront (CDN)**
- **Limit:** 1TB data transfer out
- **Limit:** 10,000,000 HTTP/HTTPS requests
- **What You Can Do:**
  - Set up CDN for websites
  - Practice: Distributions, cache behaviors, origins

**Route 53 (DNS)**
- **Limit:** 50 queries per month (Hosted Zone)
- **Note:** Hosted zone costs $0.50/month after free queries

---

#### Serverless

**Lambda**
- **Limit:** 1 million free requests per month
- **Limit:** 400,000 GB-seconds of compute time per month
- **What You Can Do:**
  - Build serverless applications
  - Practice: Functions, triggers, event sources, layers
- **Example:** Lambda function running 128MB for 1 second = 0.125 GB-seconds

**API Gateway**
- **Limit:** 1 million API calls per month (first 12 months)

---

#### Other Services

**SNS (Simple Notification Service)**
- **Limit:** 1,000 publishes
- **Limit:** 100,000 HTTP/S deliveries
- **Limit:** 1,000 email deliveries

**SQS (Simple Queue Service)**
- **Limit:** 1 million requests per month

**CloudWatch**
- **Limit:** 10 custom metrics and 10 alarms
- **Limit:** 1 million API requests
- **Limit:** 5GB of log data ingestion and storage
- **Limit:** 3 dashboards with up to 50 metrics each

**AWS CodeBuild**
- **Limit:** 100 build minutes per month

**AWS CodePipeline**
- **Limit:** 1 active pipeline per month

---

### Always Free Services

**Lambda:** 1 million requests + 400,000 GB-seconds per month (forever)
**DynamoDB:** 25GB storage + 25 read/write capacity units (forever)
**SNS:** 1 million publishes (forever)
**SQS:** 1 million requests (forever)
**CloudWatch:** 10 metrics, 10 alarms, 1 million API requests (forever)
**Cognito:** 50,000 monthly active users
**AWS Glue:** 1 million objects stored in the Data Catalog
**AWS Step Functions:** 4,000 state transitions per month

---

### Free Trials

**Amazon SageMaker:** 250 hours per month of t2.medium notebook usage (2 months)
**Amazon Redshift:** 750 hours per month of dc2.large node usage (2 months)
**Amazon Inspector:** 90-day trial
**Amazon GuardDuty:** 30-day trial
**AWS Config:** Free during first 30 days

---

### Data Transfer Limits (Critical)

**Free Inbound Data Transfer:** Unlimited to AWS
**Free Outbound Data Transfer:** 15GB per month (aggregated across all services)
**Exceeding Limits:** $0.09/GB for the next 10TB

**Important:** Data transfer is one of the most common unexpected costs.

---

## Microsoft Azure Free Tier

**Website:** https://azure.microsoft.com/free/

**Duration:** 12 months + $200 credit (30 days) + Always Free services

**Credit Card Required:** Yes

**Phone Verification Required:** Yes

---

### $200 Credit (First 30 Days)

- Use for any Azure service
- Expires after 30 days or when credit exhausted
- Good for experimentation with expensive services
- Best use: Test services not in free tier (AKS clusters, larger VMs)

---

### 12 Months Free Services

#### Compute

**Virtual Machines (Linux)**
- **Limit:** 750 hours per month of B1S (1 vCPU, 1GB RAM)
- **What You Can Do:**
  - Run one B1S VM continuously
  - Practice: VM deployment, extensions, availability sets

**Virtual Machines (Windows)**
- **Limit:** 750 hours per month of B1S
- **What You Can Do:**
  - Run Windows Server
  - Practice: Active Directory, IIS, PowerShell

**Container Instances**
- **Limit:** 50 container group deployments
- **What You Can Do:**
  - Deploy containerized applications
  - Practice: Container management without Kubernetes

---

#### Storage

**Blob Storage**
- **Limit:** 5GB locally redundant storage (LRS)
- **Limit:** 20,000 read operations
- **Limit:** 10,000 write operations
- **What You Can Do:**
  - Store files, backups, static websites
  - Practice: Blob tiers, lifecycle management

**File Storage**
- **Limit:** 100GB LRS
- **What You Can Do:**
  - SMB file shares
  - Practice: File share mounting on VMs

**Managed Disks**
- **Limit:** 2 x 64GB P6 SSD disks
- **Limit:** 2 x 128GB P10 SSD disks
- **What You Can Do:**
  - Attach to VMs
  - Practice: Disk management, snapshots

---

#### Database

**SQL Database**
- **Limit:** 250GB
- **What You Can Do:**
  - Deploy SQL Server databases
  - Practice: Backups, geo-replication, elastic pools

**Cosmos DB**
- **Limit:** 25GB storage
- **Limit:** 400 request units per second
- **What You Can Do:**
  - Build multi-model NoSQL applications
  - Practice: Global distribution, consistency levels

---

#### Networking

**Load Balancer**
- **Limit:** 750 hours
- **What You Can Do:**
  - Load balance VMs
  - Practice: Health probes, load balancing rules

**Virtual Network**
- **Limit:** 50 virtual networks
- **Free:** No charge for VNets
- **What You Can Do:**
  - Create private networks
  - Practice: NSGs, peering, VPN gateways

**Public IP Address**
- **Limit:** 750 hours of static IPv4
- **What You Can Do:**
  - Assign to VMs, load balancers
  - Practice: IP management

**Bandwidth**
- **Limit:** 100GB outbound data transfer
- **What You Can Do:**
  - More generous than AWS (15GB)

---

#### Application Services

**App Service**
- **Limit:** 10 web, mobile, or API apps
- **Limit:** 1GB storage
- **What You Can Do:**
  - Deploy web applications
  - Practice: Deployment slots, scaling

**Functions**
- **Limit:** 1 million requests per month
- **Limit:** 400,000 GB-s execution time
- **What You Can Do:**
  - Build serverless applications
  - Practice: Triggers, bindings, Durable Functions

**Azure DevOps**
- **Limit:** 5 users with Basic access
- **Limit:** Unlimited private Git repos
- **Limit:** 1,800 minutes of CI/CD per month
- **What You Can Do:**
  - Set up CI/CD pipelines
  - Practice: Azure Pipelines, Repos, Boards

---

### Always Free Services

**App Service:** 10 apps with 1GB storage
**Functions:** 1 million executions per month
**Azure Cosmos DB:** 1000 request units/sec with 25GB storage (first 400 RU/s free)
**Service Bus:** 750 hours per month
**Event Grid:** 100,000 operations per month
**Azure DevOps:** 5 users
**Azure Active Directory:** 50,000 stored objects
**Notification Hubs:** 1 million pushes
**Azure Container Registry:** 1 registry with 10GB storage
**Azure Kubernetes Service (AKS):** Free cluster management (pay only for VMs)

---

### Azure for Students

**No Credit Card Required**

**$100 Credit** (12 months) for students
**Free Services:** Many free services without credit card
**Verification:** Requires .edu email or other student verification

---

## Google Cloud Platform Free Tier

**Website:** https://cloud.google.com/free

**Duration:** 90 days $300 credit + Always Free tier

**Credit Card Required:** Yes (not charged during trial)

---

### $300 Credit (90 Days)

- Use for any GCP service
- Expires after 90 days or when credit exhausted
- Won't auto-charge after expiration
- Best for: Testing expensive services (GKE, larger instances)

---

### Always Free Tier

**No Time Limit** - These services are always free within limits

#### Compute

**Compute Engine**
- **Limit:** 1 e2-micro instance per month (US regions only: us-west1, us-central1, us-east1)
- **Specs:** 0.25 vCPU, 1GB RAM
- **Limit:** 30GB-months standard persistent disk
- **Limit:** 1GB network egress per month (excluding China and Australia)
- **What You Can Do:**
  - Run small VMs 24/7
  - Practice: Instance management, metadata, startup scripts

**Cloud Functions**
- **Limit:** 2 million invocations per month
- **Limit:** 400,000 GB-seconds, 200,000 GHz-seconds compute time
- **Limit:** 5GB network egress per month
- **What You Can Do:**
  - Build serverless functions
  - Practice: HTTP functions, event-driven functions

**Cloud Run**
- **Limit:** 2 million requests per month
- **Limit:** 360,000 GB-seconds memory
- **Limit:** 180,000 vCPU-seconds
- **Limit:** 1GB network egress per month
- **What You Can Do:**
  - Deploy containerized apps
  - Practice: Serverless containers

**App Engine**
- **Limit:** 28 frontend instance hours per day
- **Limit:** 9 backend instance hours per day
- **Limit:** 1GB egress per day
- **What You Can Do:**
  - Deploy web applications
  - Practice: App deployment, versions, traffic splitting

**Google Kubernetes Engine (GKE)**
- **Free:** 1 Autopilot or Zonal cluster per month (cluster management fee waived)
- **Note:** Still pay for worker nodes

---

#### Storage

**Cloud Storage**
- **Limit:** 5GB per month (regional storage, US only)
- **Limit:** 5,000 Class A operations per month
- **Limit:** 50,000 Class B operations per month
- **Limit:** 100GB network egress per month
- **What You Can Do:**
  - Store files, host static websites
  - Practice: Buckets, lifecycle management, IAM

**Cloud Firestore**
- **Limit:** 1GB storage
- **Limit:** 50,000 reads, 20,000 writes, 20,000 deletes per day
- **What You Can Do:**
  - Build NoSQL applications
  - Practice: Collections, queries, real-time updates

**Cloud Datastore** (legacy, being replaced by Firestore)
- **Limit:** 1GB storage
- **Limit:** 50,000 reads, 20,000 writes, 20,000 deletes per day

---

#### Networking

**VPC (Virtual Private Cloud)**
- **Free:** VPC creation, subnets, firewall rules
- **Limit:** Egress as noted in other services

**Cloud Load Balancing**
- **Pricing:** Not in always free tier
- **During trial:** Use $300 credit

**Cloud NAT**
- **Pricing:** Not in always free tier

---

#### Databases

**Cloud SQL**
- **Pricing:** Not in always free tier
- **During trial:** Use $300 credit
- **Tip:** Use shared-core instances (db-f1-micro) to minimize cost

**BigQuery**
- **Limit:** 1TB of queries per month
- **Limit:** 10GB storage per month
- **What You Can Do:**
  - Analyze data with SQL
  - Practice: Datasets, tables, queries, jobs

**Cloud Bigtable**
- **Pricing:** Not in always free tier

---

#### Operations

**Cloud Monitoring**
- **Limit:** Allotment of free ingestion per month (first 50GB free per month)
- **What You Can Do:**
  - Monitor resources
  - Practice: Metrics, alerts, dashboards

**Cloud Logging**
- **Limit:** First 50GB per month free
- **What You Can Do:**
  - View logs from all GCP services
  - Practice: Log queries, exports, sinks

**Cloud Trace**
- **Limit:** First 2.5M trace spans per month free

**Cloud Debugger**
- **Free:** No charge

---

#### Other Services

**Cloud Pub/Sub**
- **Limit:** 10GB messages per month
- **What You Can Do:**
  - Messaging between services
  - Practice: Topics, subscriptions, push/pull

**Cloud Scheduler**
- **Limit:** 3 jobs per month
- **What You Can Do:**
  - Cron-like job scheduling
  - Practice: Job scheduling, Cloud Functions triggers

**Secret Manager**
- **Limit:** 6 active secret versions
- **Limit:** 10,000 access operations per month
- **What You Can Do:**
  - Store API keys, passwords
  - Practice: Secret management, IAM

**Artifact Registry**
- **Limit:** 0.5GB storage per month
- **What You Can Do:**
  - Store Docker images
  - Practice: Container registry

---

### GCP Free Trial Tips

**During 90-Day Trial:**
- No auto-charge after trial ends
- Must explicitly upgrade to paid account
- Good time to experiment with expensive services
- Create GKE clusters, use Cloud SQL, test high-compute instances

**After Trial:**
- Switch to Always Free tier
- Carefully manage which services you use
- Set up billing alerts

---

## Oracle Cloud Free Tier

**Website:** https://www.oracle.com/cloud/free/

**Duration:** Always Free (no expiration) + $300 credit (30 days)

**Credit Card Required:** Yes

**Why Consider:** Very generous always-free tier, great for learning

---

### Always Free Resources

#### Compute

**Compute Instances (AMD)**
- **Limit:** 2 AMD-based Compute VMs
- **Specs:** VM.Standard.E2.1.Micro (1 OCPU, 1GB RAM each)
- **What You Can Do:**
  - Run 2 small VMs continuously
  - Practice: Instance management, networking

**Compute Instances (ARM - Ampere A1)**
- **Limit:** 4 cores and 24GB RAM per month (Ampere A1)
- **What You Can Do:**
  - Run 4 x 1-core 6GB VMs, or 2 x 2-core 12GB VMs, or 1 x 4-core 24GB VM
  - ARM architecture for cost-effective workloads
  - Practice: ARM-based computing

**Note:** Oracle's free ARM offering is very generous compared to competitors

---

#### Storage

**Block Volume**
- **Limit:** 200GB total storage
- **What You Can Do:**
  - Attach to compute instances
  - Practice: Volume management

**Object Storage**
- **Limit:** 20GB
- **What You Can Do:**
  - Store files, backups
  - Practice: Buckets, lifecycle policies

**Archive Storage**
- **Limit:** 10GB
- **What You Can Do:**
  - Long-term archival
  - Practice: Archive management

---

#### Database

**Autonomous Database**
- **Limit:** 2 databases (20GB each)
- **Types:** Autonomous Transaction Processing (ATP) or Autonomous Data Warehouse (ADW)
- **What You Can Do:**
  - Fully managed Oracle databases
  - Practice: SQL, PL/SQL, database administration

**NoSQL Database**
- **Limit:** 133 million reads per month
- **Limit:** 133 million writes per month
- **Limit:** 25GB storage per table (3 tables max)

---

#### Networking

**Load Balancer**
- **Limit:** 1 load balancer (10Mbps bandwidth)
- **What You Can Do:**
  - Load balance between VMs
  - Practice: Load balancing rules

**Flexible Network Load Balancer**
- **Limit:** 10Mbps bandwidth

**VCN (Virtual Cloud Network)**
- **Free:** Create virtual networks
- **What You Can Do:**
  - Private networking
  - Practice: Subnets, security lists, route tables

**Outbound Data Transfer**
- **Limit:** 10TB per month
- **Note:** Much more generous than AWS (15GB) or Azure (100GB)

---

#### Other Services

**Monitoring**
- **Limit:** 500 million ingestion datapoints
- **Limit:** 1 billion retrieval datapoints

**Notifications**
- **Limit:** 1 million sent per month

**Resource Manager**
- **Free:** Infrastructure as Code (Terraform-based)

---

### $300 Credit (30 Days)

- Use for any Oracle Cloud service
- Test paid services
- Expires after 30 days or when exhausted

---

### Oracle vs Other Clouds

**Pros:**
- Very generous always-free tier (especially ARM compute)
- Great for learning Oracle Database
- High data transfer limits
- No expiration on free tier

**Cons:**
- Smaller ecosystem than AWS/Azure/GCP
- Fewer learning resources
- Less relevant for most cloud certifications (except Oracle Cloud Infrastructure)

---

## IBM Cloud Free Tier

**Website:** https://www.ibm.com/cloud/free

**Duration:** Lite account (always free) + $200 credit (30 days for new accounts)

**Credit Card Required:** No (for Lite account)

---

### Always Free (Lite Services)

#### Compute

**Cloud Foundry Apps**
- **Limit:** 256MB of instantaneous runtime memory
- **What You Can Do:**
  - Deploy applications
  - Practice: PaaS deployments

**Functions**
- **Limit:** 5 million executions per month
- **What You Can Do:**
  - Serverless functions
  - Practice: OpenWhisk-based functions

---

#### Storage

**Object Storage**
- **Limit:** 25GB per month
- **What You Can Do:**
  - Store files
  - Practice: Buckets, access control

---

#### Database

**Cloudant (NoSQL)**
- **Limit:** 1GB data storage
- **Limit:** 20 queries per second
- **What You Can Do:**
  - JSON document database
  - Practice: NoSQL queries

**Db2**
- **Limit:** 200MB storage
- **What You Can Do:**
  - Relational database
  - Practice: SQL

---

#### AI/ML

**Watson Assistant**
- **Limit:** 10,000 messages per month
- **What You Can Do:**
  - Build chatbots
  - Practice: Conversational AI

**Watson Studio**
- **Limit:** 50 capacity unit-hours per month
- **What You Can Do:**
  - Data science projects
  - Practice: Machine learning

**Watson Discovery**
- **Limit:** 1,000 documents per month
- **What You Can Do:**
  - AI-powered search
  - Practice: Document analysis

---

#### Other Services

**Kubernetes Service**
- **Limit:** 1 free cluster
- **What You Can Do:**
  - Learn Kubernetes
  - Practice: Container orchestration

**Blockchain Platform**
- **Limit:** Basic plan free
- **What You Can Do:**
  - Develop blockchain applications

---

### Why Consider IBM Cloud

**Pros:**
- No credit card required for Lite account
- Good AI/ML services in free tier
- Unique services (Watson, Blockchain)
- Great for IBM certifications

**Cons:**
- Smaller ecosystem
- Fewer learning resources
- Less popular for general cloud certifications
- Some free tier limits are restrictive

---

## Cost Management Strategies

### Set Up Billing Alerts Immediately

**AWS CloudWatch Billing Alarms:**
```bash
# Enable billing alerts in Billing Preferences first
aws cloudwatch put-metric-alarm \
  --alarm-name BillingAlert \
  --alarm-description "Alert when charges exceed $5" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1
```

**AWS Budgets:**
- Set up budget alerts in AWS Budgets console
- Free: 2 budgets
- Get alerts at 50%, 80%, 100% of budget

**Azure Cost Alerts:**
- Set up in Cost Management + Billing
- Configure budget alerts
- Free: 10 budgets

**GCP Billing Alerts:**
- Set up in Billing > Budgets & Alerts
- Configure budget thresholds
- Get email/Pub/Sub alerts

---

### Use Cost Calculators Before Deployment

**AWS Pricing Calculator:** https://calculator.aws/
**Azure Pricing Calculator:** https://azure.microsoft.com/pricing/calculator/
**GCP Pricing Calculator:** https://cloud.google.com/products/calculator

---

### Tag Everything

**AWS:**
```bash
aws ec2 create-tags --resources i-xxx --tags Key=Project,Value=CertificationStudy Key=Owner,Value=YourName
```

**Azure:**
```bash
az resource tag --tags Project=CertificationStudy Owner=YourName --resource-group myRG --name myVM --resource-type Microsoft.Compute/virtualMachines
```

**GCP:**
```bash
gcloud compute instances add-labels vm1 --labels=project=certification-study,owner=yourname
```

**Why Tag:**
- Track which resources are for study
- Identify cost by project
- Easier cleanup

---

### Use Spot/Preemptible/Low-Priority Instances

**AWS Spot Instances:**
- Up to 90% discount
- Can be interrupted
- Good for: Learning, non-critical workloads

**Azure Spot VMs:**
- Up to 90% discount
- Can be evicted
- Good for: Testing, learning

**GCP Preemptible VMs:**
- Up to 80% discount
- Maximum 24-hour runtime
- Good for: Batch jobs, learning

---

### Automate Shutdown

**AWS Lambda + CloudWatch Events:**
```python
# Lambda function to stop EC2 instances
import boto3
ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    instances = ['i-xxx', 'i-yyy']
    ec2.stop_instances(InstanceIds=instances)
    return 'Stopped instances: ' + str(instances)
```

**Azure Automation:**
- Use Azure Automation runbooks
- Schedule VM start/stop

**GCP Cloud Scheduler:**
- Schedule instance stop with Cloud Scheduler
- Use Cloud Functions to automate

---

### Monitor Costs Daily

**AWS:**
- Check AWS Cost Explorer daily
- Review Cost and Usage Reports

**Azure:**
- Check Cost Management + Billing daily
- Set up cost analysis

**GCP:**
- Check Billing Reports daily
- Use BigQuery for detailed analysis

---

### Delete Unused Resources

**Common Forgotten Resources:**

1. **Elastic IPs (AWS)** - Charged when not attached
2. **Snapshots** - Accumulate over time
3. **Load Balancers** - Hourly charges
4. **NAT Gateways** - Hourly charges
5. **VPN Connections** - Hourly charges
6. **Stopped Instances with attached EBS** - EBS still charged
7. **Old AMIs and associated snapshots**
8. **CloudWatch Logs** - Storage costs
9. **S3 Buckets** - Check for versioning

**Weekly Cleanup Routine:**
```bash
# AWS: List all resources by region
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table

# Check for unattached volumes
aws ec2 describe-volumes --filters "Name=status,Values=available" --output table

# Check for unassociated Elastic IPs
aws ec2 describe-addresses --query 'Addresses[?AssociationId==null]' --output table
```

---

## Free Tier Best Practices

### 1. Use Free Tier Exclusively for First Month

**Strategy:**
- Month 1: Only use free tier services
- Learn the limits
- Understand what's included
- Build confidence before expanding

---

### 2. Create Separate Accounts for Different Clouds

**Why:**
- Isolate free tier usage
- Avoid mixing personal and learning projects
- Easier to track certification study costs
- Can delete entire account when done

---

### 3. Use Infrastructure as Code

**Benefits:**
- Quickly tear down and rebuild environments
- No forgotten resources
- Reproducible setups
- Version control your infrastructure

**AWS CloudFormation Example:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Free tier EC2 instance

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c55b159cbfafe1f0
      Tags:
        - Key: Name
          Value: FreeTierStudy
```

**Teardown:**
```bash
aws cloudformation delete-stack --stack-name my-stack
```

---

### 4. Schedule Learning Time

**Why:**
- Free tiers reset monthly
- Plan major projects for start of month
- More hours/requests available
- Better resource management

**Example Monthly Plan:**
- Week 1: Deploy infrastructure
- Week 2: Hands-on practice
- Week 3: Advanced features
- Week 4: Cleanup, review

---

### 5. Leverage Multiple Free Tiers

**Strategy:**
- Use AWS for EC2 and S3 practice
- Use Azure for Web Apps and SQL practice
- Use GCP for BigQuery and App Engine practice
- Use Oracle for generous compute (ARM)

**Why:**
- Different strengths per cloud
- More free resources overall
- Better multi-cloud understanding
- Reduce risk of exceeding limits

---

### 6. Document Your Usage

**Track:**
- What services you've used
- How much quota consumed
- Which projects used which resources
- Lessons learned

**Tools:**
- Spreadsheet
- Notion/Obsidian
- GitHub repo with notes

---

## Multi-Cloud Strategy

### Free Tier Comparison Table

| Service | AWS | Azure | GCP | Oracle |
|---------|-----|-------|-----|--------|
| **Compute (Hours)** | 750/mo (12mo) | 750/mo (12mo) | Always free (e2-micro) | Always free (2 VMs + 4 ARM cores) |
| **Storage (GB)** | 5GB S3 (12mo) | 5GB Blob (12mo) | 5GB regional | 20GB object + 200GB block |
| **Database** | 20GB RDS (12mo) | 250GB SQL (12mo) | Limited Firestore | 2x20GB Autonomous DB |
| **Outbound Transfer** | 15GB/mo | 100GB/mo (12mo) | 1GB/mo (Compute) | 10TB/mo |
| **Serverless Requests** | 1M Lambda (always) | 1M Functions (always) | 2M Cloud Functions (always) | 5M (IBM Functions) |
| **Duration** | 12 months + always free | 12 months + always free | 90-day credit + always free | Always free |

---

### Maximizing All Free Tiers Simultaneously

**Month 1-3:** GCP Focus (use $300 credit)
- Deploy expensive resources (GKE, Cloud SQL)
- Test high-compute instances
- Learn GCP-specific services

**Month 1-12:** AWS Focus
- Primary compute on EC2
- S3 for storage
- RDS for databases
- Build complex architectures

**Month 1-12:** Azure Focus
- Web Apps for PaaS
- Azure SQL for databases
- Use $200 credit in first 30 days for testing

**Always:** Oracle Cloud
- Extra compute capacity (ARM instances)
- Additional storage
- Oracle Database practice

**Result:** Massive free resources across all clouds for certification study

---

## Study Projects Using Free Tiers

### Project 1: Three-Tier Web Application

**Architecture:**
- Frontend: Static website (S3/Blob/Cloud Storage)
- Backend: EC2/VM/Compute Engine (t2.micro/B1s/e2-micro)
- Database: RDS/SQL Database/Cloud SQL (free tier)

**Cost:** Free within limits

**Time:** 8-12 hours

**Skills:**
- Compute, storage, networking, databases
- Load balancing, security groups
- Multi-tier architecture

---

### Project 2: Serverless API

**Architecture:**
- API: Lambda/Functions/Cloud Functions
- Database: DynamoDB/Cosmos DB/Firestore
- API Gateway: API Gateway/APIM/Cloud Endpoints

**Cost:** Free (well within free tier limits)

**Time:** 6-10 hours

**Skills:**
- Serverless computing
- NoSQL databases
- API management

---

### Project 3: CI/CD Pipeline

**Architecture:**
- Code: GitHub/Azure Repos/Cloud Source Repositories
- Build: CodeBuild/Azure Pipelines/Cloud Build
- Deploy: S3/App Service/Cloud Run

**Cost:** Minimal (within free tier)

**Time:** 10-15 hours

**Skills:**
- DevOps practices
- CI/CD pipelines
- Automation

---

### Project 4: Data Analytics Pipeline

**Architecture:**
- Storage: S3/Blob Storage/Cloud Storage
- Processing: Lambda/Functions/Cloud Functions
- Analytics: Athena/SQL Database/BigQuery

**Cost:** Free (within free tier)

**Time:** 12-18 hours

**Skills:**
- Data engineering
- Analytics
- ETL processes

---

## Common Pitfalls and How to Avoid Them

### Pitfall 1: Data Transfer Costs

**Problem:** Data transfer out is often the largest unexpected cost

**AWS:** 15GB/month free
**Azure:** 100GB/month free (12 months)
**GCP:** 1GB/month free (Compute Engine)

**How to Avoid:**
- Minimize data downloads
- Use same-region resources
- Compress data before transfer
- Test on small datasets

---

### Pitfall 2: Elastic IPs (AWS)

**Problem:** Elastic IPs cost $0.005/hour when not attached to running instance

**How to Avoid:**
- Release IPs when not in use
- Use public DNS names instead
- Attach IPs only when needed

---

### Pitfall 3: Stopped Instances with Attached Storage

**Problem:** Stopped EC2 instances still incur EBS charges

**How to Avoid:**
- Terminate instances when done (not just stop)
- Create AMIs before terminating
- Use snapshots for backups, not running EBS volumes

---

### Pitfall 4: Old Snapshots

**Problem:** Snapshots accumulate and cost $0.05/GB/month (AWS)

**How to Avoid:**
- Delete old snapshots
- Use lifecycle policies
- Tag snapshots by date
- Review snapshots weekly

---

### Pitfall 5: Unused Load Balancers

**Problem:** Load balancers cost ~$0.025/hour (AWS ALB) even if not used

**How to Avoid:**
- Delete load balancers when not needed
- Use load balancers only for multi-instance scenarios
- Consider alternatives (e.g., CloudFront)

---

### Pitfall 6: NAT Gateways

**Problem:** NAT Gateways cost ~$0.045/hour (AWS) plus data processing

**How to Avoid:**
- Use NAT instances (t2.micro) instead during learning
- Delete NAT gateways when not needed
- Use public subnets where possible

---

### Pitfall 7: Forgotten Resources in Multiple Regions

**Problem:** Resources in different regions are easy to forget

**How to Avoid:**
- Stick to one region for learning
- Check all regions during cleanup
- Use AWS Config or scripts to audit

**AWS Check All Regions:**
```bash
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do
  echo "Region: $region"
  aws ec2 describe-instances --region $region --query 'Reservations[*].Instances[*].[InstanceId,State.Name]' --output table
done
```

---

### Pitfall 8: Exceeding Free Tier by Mistake

**Problem:** Running 2 t2.micro instances uses 1,500 hours/month (750 free = 750 billable)

**How to Avoid:**
- Run only 1 t2.micro continuously
- Or run multiple instances for less time
- Monitor hours used
- Set up billing alerts

---

### Pitfall 9: RDS Multi-AZ

**Problem:** Multi-AZ deployment doubles RDS costs (not in free tier)

**How to Avoid:**
- Use single-AZ for learning
- Test Multi-AZ briefly during free trial
- Understand it's not in free tier

---

### Pitfall 10: CloudWatch Logs Retention

**Problem:** CloudWatch Logs storage costs add up

**How to Avoid:**
- Set short retention periods (1-3 days for learning)
- Delete log groups when done
- Export to S3 if needed long-term

---

## Free Tier Checklist

### Before Starting

- [ ] Create new account (or use existing)
- [ ] Set up MFA on root/admin account
- [ ] Add payment method (required for AWS, Azure, GCP)
- [ ] Set up billing alerts ($5, $10, $20)
- [ ] Note free tier start date
- [ ] Choose primary region

### Weekly

- [ ] Check billing dashboard
- [ ] Review running resources
- [ ] Delete unused resources
- [ ] Check all regions for forgotten resources
- [ ] Review snapshots and backups
- [ ] Update cost tracking spreadsheet

### Monthly

- [ ] Review entire month's costs
- [ ] Identify any unexpected charges
- [ ] Plan next month's projects
- [ ] Check free tier expiration (AWS, Azure at 12 months)

### End of Certification Study

- [ ] Delete all resources
- [ ] Delete snapshots and AMIs
- [ ] Release Elastic IPs
- [ ] Delete S3 buckets (empty first)
- [ ] Delete CloudWatch log groups
- [ ] Deactivate account (or keep for future use)

---

## Recommended Free Tier Learning Path

### Month 1: Basics (AWS Focus)

**Week 1:** Setup + Core Services
- Create account, set up billing alerts
- Launch t2.micro EC2 instance
- Create S3 bucket, upload files
- Explore IAM users and roles

**Week 2:** Networking
- Create VPC, subnets
- Security groups, NACLs
- Public and private subnets
- NAT instances (not gateways)

**Week 3:** Databases
- Deploy RDS MySQL (db.t2.micro)
- Practice backups, snapshots
- Connect from EC2

**Week 4:** Serverless
- Create Lambda functions
- DynamoDB tables
- API Gateway

---

### Month 2: Intermediate (Azure Focus)

**Week 1:** Azure Basics
- Create Azure account, use $200 credit
- Deploy B1S VM
- Blob storage
- Azure AD basics

**Week 2:** Azure PaaS
- Deploy App Service
- Azure SQL Database
- Application Insights

**Week 3:** Azure Networking
- Virtual Networks
- NSGs
- Load Balancer

**Week 4:** Azure Serverless
- Azure Functions
- Logic Apps
- Cosmos DB

---

### Month 3: Advanced (GCP Focus)

**Week 1:** GCP Basics (use $300 credit)
- Create GCP account
- Deploy e2-micro instance
- Cloud Storage
- IAM and Organization

**Week 2:** GCP PaaS
- App Engine
- Cloud Functions
- Cloud Run

**Week 3:** GCP Data
- BigQuery
- Cloud SQL
- Firestore

**Week 4:** GCP Kubernetes
- Create GKE cluster
- Deploy applications
- Practice kubectl

---

### Month 4-12: Mastery + Multi-Cloud

- Build complex projects across clouds
- Implement CI/CD pipelines
- Multi-cloud architectures
- Certification exam preparation
- Hands-on labs for specific certifications

---

## Summary: Maximizing Value

**Total Free Value (First Year):**
- **AWS:** ~$450-600 (12 months free tier)
- **Azure:** ~$500-700 (12 months + $200 credit)
- **GCP:** ~$600-800 ($300 credit + always free)
- **Oracle:** ~$1200+ (always free, very generous)

**Total Potential Value:** $2,750-3,500 in free cloud resources

**Best Strategy:**
1. Start with AWS (most certifications available)
2. Add Azure and GCP in parallel
3. Use Oracle for extra compute capacity
4. Leverage IBM for AI/ML learning
5. Set up cost alerts on ALL accounts
6. Clean up religiously every week
7. Focus on free tier services
8. Use credits for expensive short-term tests

**Result:** Complete cloud certification study with minimal cost (mostly exam fees).

---

This guide provides everything you need to maximize free cloud resources for certification study. Follow the best practices, set up alerts, and clean up regularly to avoid unexpected charges.
