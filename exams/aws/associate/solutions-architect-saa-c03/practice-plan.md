# AWS Solutions Architect Associate (SAA-C03) Study Plan

## 8-Week Comprehensive Study Schedule

### Week 1: Core Infrastructure and Compute

#### Day 1-2: Amazon EC2 Deep Dive
- [ ] Study EC2 instance types and use cases
- [ ] Learn pricing models (On-Demand, Reserved, Spot, Dedicated)
- [ ] Understand placement groups and tenancy options
- [ ] Hands-on: Launch instances across multiple AZs
- [ ] Lab: Configure Auto Scaling Groups with different scaling policies
- [ ] Review Notes: `01-ec2-fundamentals.md`

#### Day 3-4: Load Balancing and Auto Scaling
- [ ] Compare ALB, NLB, CLB, and GWLB
- [ ] Learn target groups and health checks
- [ ] Study Auto Scaling policies and CloudWatch integration
- [ ] Hands-on: Set up multi-AZ load balancer with health checks
- [ ] Lab: Implement predictive scaling with CloudWatch
- [ ] Review Notes: `02-load-balancing-auto-scaling.md`

#### Day 5-6: AWS Lambda and Serverless
- [ ] Understand Lambda execution model and limits
- [ ] Learn event sources and triggers
- [ ] Study concurrent execution and provisioned concurrency
- [ ] Hands-on: Create event-driven architecture with Lambda
- [ ] Lab: Build serverless API with API Gateway and Lambda
- [ ] Review Notes: `03-serverless-compute.md`

#### Day 7: Week 1 Review and Practice
- [ ] Complete compute services practice questions
- [ ] Build end-to-end compute architecture
- [ ] Review any weak areas identified

### Week 2: Storage Solutions

#### Day 8-9: Amazon S3 Mastery
- [ ] Study storage classes and lifecycle policies
- [ ] Learn cross-region replication and versioning
- [ ] Understand S3 Transfer Acceleration and multipart upload
- [ ] Study S3 security features and access patterns
- [ ] Hands-on: Configure S3 with lifecycle rules and CRR
- [ ] Lab: Implement S3 website hosting with CloudFront
- [ ] Review Notes: `04-s3-storage-solutions.md`

#### Day 10-11: Block and File Storage
- [ ] Compare EBS volume types and performance characteristics
- [ ] Learn EBS snapshots, encryption, and Multi-Attach
- [ ] Study EFS performance modes and throughput modes
- [ ] Understand FSx for Windows File Server and Lustre
- [ ] Hands-on: Configure encrypted EBS volumes with snapshots
- [ ] Lab: Set up EFS with multiple EC2 instances
- [ ] Review Notes: `05-block-file-storage.md`

#### Day 12-13: Hybrid Storage and Migration
- [ ] Study AWS Storage Gateway modes
- [ ] Learn AWS DataSync and Transfer Family
- [ ] Understand Snow family devices and use cases
- [ ] Study storage migration strategies
- [ ] Hands-on: Configure Storage Gateway in file gateway mode
- [ ] Lab: Set up DataSync for on-premises migration
- [ ] Review Notes: `06-hybrid-storage-migration.md`

#### Day 14: Week 2 Review and Integration
- [ ] Complete storage services practice questions
- [ ] Design multi-tier storage architecture
- [ ] Practice storage cost optimization scenarios

### Week 3: Database Solutions

#### Day 15-16: Amazon RDS and Aurora
- [ ] Study RDS engine options and Multi-AZ deployments
- [ ] Learn Read Replicas and cross-region capabilities
- [ ] Understand Aurora Global Database and Serverless
- [ ] Study automated backup and point-in-time recovery
- [ ] Hands-on: Deploy RDS with Multi-AZ and Read Replicas
- [ ] Lab: Set up Aurora Global Database
- [ ] Review Notes: `07-relational-databases.md`

#### Day 17-18: NoSQL and Caching
- [ ] Study DynamoDB partition keys and global tables
- [ ] Learn DynamoDB Accelerator (DAX) and on-demand billing
- [ ] Understand ElastiCache Redis vs Memcached
- [ ] Study caching strategies and patterns
- [ ] Hands-on: Build DynamoDB application with global tables
- [ ] Lab: Implement ElastiCache for session management
- [ ] Review Notes: `08-nosql-caching.md`

#### Day 19-20: Specialized Databases
- [ ] Study Amazon Redshift and Redshift Spectrum
- [ ] Learn DocumentDB, Neptune, and QLDB
- [ ] Understand Timestream and other specialized databases
- [ ] Study database migration tools and strategies
- [ ] Hands-on: Set up Redshift data warehouse
- [ ] Lab: Migrate database using DMS
- [ ] Review Notes: `09-specialized-databases.md`

#### Day 21: Week 3 Review and Practice
- [ ] Complete database services practice questions
- [ ] Design polyglot persistence architecture
- [ ] Practice database performance optimization

### Week 4: Networking and Content Delivery

#### Day 22-23: VPC Architecture Deep Dive
- [ ] Study VPC design patterns and best practices
- [ ] Learn subnet types, routing, and NAT configurations
- [ ] Understand VPC peering and Transit Gateway
- [ ] Study VPC endpoints and PrivateLink
- [ ] Hands-on: Build complex multi-tier VPC
- [ ] Lab: Implement VPC peering and Transit Gateway
- [ ] Review Notes: `10-vpc-networking.md`

#### Day 24-25: DNS and Content Delivery
- [ ] Study Route 53 routing policies and health checks
- [ ] Learn CloudFront distributions and behaviors
- [ ] Understand edge locations and Regional Edge Caches
- [ ] Study Lambda@Edge and CloudFront Functions
- [ ] Hands-on: Configure Route 53 with failover routing
- [ ] Lab: Set up CloudFront with multiple origins
- [ ] Review Notes: `11-dns-content-delivery.md`

#### Day 26-27: Connectivity and Hybrid Networking
- [ ] Study AWS Direct Connect and Virtual Interfaces
- [ ] Learn VPN connections and customer gateways
- [ ] Understand AWS Global Accelerator
- [ ] Study hybrid DNS and networking patterns
- [ ] Hands-on: Configure Site-to-Site VPN
- [ ] Lab: Set up Direct Connect with BGP routing
- [ ] Review Notes: `12-hybrid-connectivity.md`

#### Day 28: Week 4 Review and Integration
- [ ] Complete networking practice questions
- [ ] Design global network architecture
- [ ] Practice network troubleshooting scenarios

### Week 5: Security and Compliance

#### Day 29-30: Identity and Access Management
- [ ] Master IAM policies, roles, and trust relationships
- [ ] Study federated access and SAML integration
- [ ] Learn AWS SSO and cross-account access
- [ ] Understand IAM best practices and policy evaluation
- [ ] Hands-on: Implement cross-account IAM roles
- [ ] Lab: Set up SAML federation with Active Directory
- [ ] Review Notes: `13-identity-access-management.md`

#### Day 31-32: Data Protection and Encryption
- [ ] Study KMS key policies and grants
- [ ] Learn CloudHSM and encryption strategies
- [ ] Understand Certificate Manager and SSL/TLS
- [ ] Study secrets management with Secrets Manager
- [ ] Hands-on: Implement end-to-end encryption
- [ ] Lab: Set up CloudHSM cluster
- [ ] Review Notes: `14-data-protection-encryption.md`

#### Day 33-34: Network Security and Monitoring
- [ ] Study Security Groups vs NACLs
- [ ] Learn AWS WAF and Shield protection
- [ ] Understand GuardDuty and Security Hub
- [ ] Study VPC Flow Logs and traffic monitoring
- [ ] Hands-on: Configure WAF with CloudFront
- [ ] Lab: Set up comprehensive security monitoring
- [ ] Review Notes: `15-network-security-monitoring.md`

#### Day 35: Week 5 Review and Practice
- [ ] Complete security practice questions
- [ ] Design defense-in-depth architecture
- [ ] Practice compliance scenario planning

### Week 6: Application Integration and Architecture Patterns

#### Day 36-37: Messaging and Queuing
- [ ] Study SQS standard vs FIFO queues
- [ ] Learn SNS topics and subscription filtering
- [ ] Understand Step Functions and workflow orchestration
- [ ] Study EventBridge and event-driven architectures
- [ ] Hands-on: Build decoupled microservices architecture
- [ ] Lab: Implement complex workflow with Step Functions
- [ ] Review Notes: `16-messaging-integration.md`

#### Day 38-39: API Management and Microservices
- [ ] Study API Gateway types and deployment models
- [ ] Learn container orchestration with ECS and EKS
- [ ] Understand service mesh and advanced networking
- [ ] Study microservices patterns and best practices
- [ ] Hands-on: Deploy containerized microservices
- [ ] Lab: Set up API Gateway with Lambda authorizers
- [ ] Review Notes: `17-api-microservices.md`

#### Day 40-41: DevOps and Infrastructure as Code
- [ ] Study CloudFormation templates and best practices
- [ ] Learn CDK and infrastructure automation
- [ ] Understand CI/CD with CodePipeline and CodeDeploy
- [ ] Study blue-green and canary deployment strategies
- [ ] Hands-on: Build automated deployment pipeline
- [ ] Lab: Implement infrastructure as code with CDK
- [ ] Review Notes: `18-devops-iac.md`

#### Day 42: Week 6 Review and Integration
- [ ] Complete integration services practice questions
- [ ] Design event-driven architecture
- [ ] Practice modern application patterns

### Week 7: Monitoring, Management, and Cost Optimization

#### Day 43-44: Monitoring and Observability
- [ ] Study CloudWatch metrics, logs, and dashboards
- [ ] Learn X-Ray tracing and application insights
- [ ] Understand Systems Manager and patch management
- [ ] Study AWS Config and compliance monitoring
- [ ] Hands-on: Set up comprehensive monitoring stack
- [ ] Lab: Implement distributed tracing with X-Ray
- [ ] Review Notes: `19-monitoring-observability.md`

#### Day 45-46: Cost Management and Optimization
- [ ] Study AWS pricing models and cost calculation
- [ ] Learn Cost Explorer and budgets setup
- [ ] Understand Reserved Instances and Savings Plans
- [ ] Study cost allocation tags and billing alerts
- [ ] Hands-on: Implement cost optimization strategies
- [ ] Lab: Set up detailed cost monitoring and alerting
- [ ] Review Notes: `20-cost-optimization.md`

#### Day 47-48: Disaster Recovery and Business Continuity
- [ ] Study DR strategies (backup/restore, pilot light, warm standby, multi-site)
- [ ] Learn RPO/RTO planning and implementation
- [ ] Understand cross-region backup and replication
- [ ] Study automated disaster recovery testing
- [ ] Hands-on: Implement pilot light DR strategy
- [ ] Lab: Set up automated cross-region backups
- [ ] Review Notes: `21-disaster-recovery.md`

#### Day 49: Week 7 Review and Practice
- [ ] Complete management and monitoring practice questions
- [ ] Design enterprise-grade management strategy
- [ ] Practice cost optimization scenarios

### Week 8: Final Review and Exam Preparation

#### Day 50-51: Comprehensive Practice Exams
- [ ] Take official AWS practice exam
- [ ] Complete multiple third-party practice exams
- [ ] Identify and review weak areas
- [ ] Focus on scenario-based questions
- [ ] Review exam feedback and explanations

#### Day 52-53: Architecture Design Practice
- [ ] Practice designing solutions for common scenarios
- [ ] Review Well-Architected Framework principles
- [ ] Study real-world case studies and reference architectures
- [ ] Practice explaining architectural decisions
- [ ] Review service limits and constraints

#### Day 54-55: Final Knowledge Consolidation
- [ ] Review all notes and key concepts
- [ ] Practice whiteboarding architecture diagrams
- [ ] Review AWS service FAQs for exam-relevant services
- [ ] Take timed practice exams under exam conditions
- [ ] Focus on areas with lowest confidence

#### Day 56: Pre-Exam Preparation
- [ ] Light review of key concepts only
- [ ] Confirm exam logistics and requirements
- [ ] Prepare exam day materials and environment
- [ ] Get adequate rest and mental preparation

## Daily Study Routine (3-4 hours/day)

### Recommended Schedule
1. **60 minutes**: Read study materials and AWS documentation
2. **90 minutes**: Hands-on labs and practical exercises
3. **45 minutes**: Practice questions and review
4. **15 minutes**: Note-taking and concept reinforcement

### Weekend Extended Sessions (6-8 hours)
1. **2 hours**: Comprehensive lab exercises
2. **2 hours**: Practice exams and detailed review
3. **2 hours**: Architecture design practice
4. **1-2 hours**: Weak area remediation and additional study

## Hands-on Lab Schedule

### Week-by-Week Lab Focus

#### Week 1 Labs: Compute Foundation
1. Multi-AZ Auto Scaling architecture
2. Load balancer with health checks
3. Serverless web application
4. Container deployment on ECS

#### Week 2 Labs: Storage Solutions
1. S3 website with CloudFront CDN
2. EFS shared storage across instances
3. Storage Gateway hybrid setup
4. Data migration with DataSync

#### Week 3 Labs: Database Implementations
1. RDS Multi-AZ with Read Replicas
2. DynamoDB with DAX caching
3. Aurora Global Database setup
4. Database migration project

#### Week 4 Labs: Network Architecture
1. Complex multi-tier VPC design
2. Transit Gateway connectivity
3. CloudFront global distribution
4. Hybrid connectivity setup

#### Week 5 Labs: Security Implementation
1. End-to-end encryption setup
2. Cross-account access configuration
3. WAF and security monitoring
4. Compliance architecture design

#### Week 6 Labs: Integration Patterns
1. Event-driven microservices
2. API Gateway with security
3. Step Functions workflow
4. Infrastructure as Code deployment

#### Week 7 Labs: Operations and Management
1. Comprehensive monitoring setup
2. Cost optimization implementation
3. Disaster recovery testing
4. Automated operations pipeline

#### Week 8 Labs: Exam Simulation
1. Time-pressured architecture design
2. Complex scenario implementation
3. Troubleshooting exercises
4. Performance optimization challenges

## Progress Tracking

### Weekly Milestones
- [ ] Week 1: Master compute services and scaling
- [ ] Week 2: Understand storage solutions and patterns
- [ ] Week 3: Design database architectures
- [ ] Week 4: Implement network and content delivery
- [ ] Week 5: Secure applications and data
- [ ] Week 6: Integrate applications and services
- [ ] Week 7: Monitor, manage, and optimize
- [ ] Week 8: Ready for exam with 85%+ practice scores

### Practice Exam Targets
- [ ] Week 3 end: Score 65%+ on practice exams
- [ ] Week 5 end: Score 75%+ on practice exams
- [ ] Week 7 end: Score 85%+ on practice exams
- [ ] Week 8 end: Score 90%+ consistently on all practice exams

## 📚 Comprehensive Study Resources

**👉 [Complete AWS Study Resources Guide](../../../../.templates/resources-aws.md)**

For detailed information on courses, practice tests, hands-on labs, communities, and more, see our comprehensive AWS study resources guide which includes:
- AWS Skill Builder free courses and exam prep
- Top-rated video courses (Stephane Maarek, Adrian Cantrill, etc.)
- Practice test platforms with pricing (Tutorials Dojo, Whizlabs)
- AWS Free Tier details and hands-on lab guidance
- Community forums (r/AWSCertifications) and study groups
- Essential AWS CLI and tools
- Pro tips and budget-friendly strategies ($45-85 total)

### Quick Links (SAA-C03 Specific)
- **[SAA-C03 Official Exam Page](https://aws.amazon.com/certification/certified-solutions-architect-associate/)** - Registration and exam details
- **[AWS Skill Builder - SAA-C03 Exam Prep](https://skillbuilder.aws/)** - FREE official exam preparation
- **[AWS Documentation](https://docs.aws.amazon.com/)** - Complete service documentation
- **[AWS Free Tier](https://aws.amazon.com/free/)** - 12 months free + always-free services

## Free Hands-On Labs

Practice with real AWS environments using these free resources:

### Official AWS Labs
- **[AWS Skill Builder Labs](https://skillbuilder.aws/search?searchText=&learningTypes%5B%5D=lab)** - Official hands-on labs (free tier)
- **[AWS Workshops](https://workshops.aws/)** - Self-paced workshops covering all major services
- **[AWS Well-Architected Labs](https://wellarchitectedlabs.com/)** - Architecture-focused practical exercises

### Free Tier Practice
- **[AWS Free Tier](https://aws.amazon.com/free/)** - 12 months free + always-free services
- Use free tier to build real architectures from this study plan
- Set up billing alerts to avoid unexpected charges

### Community Labs
- **[AWS Samples on GitHub](https://github.com/aws-samples)** - Reference architectures and code
- **[Serverless Land](https://serverlessland.com/)** - Serverless patterns and tutorials

---

## Final Exam Checklist

### One Week Before
- [ ] Complete all practice exams with target scores
- [ ] Review weak areas identified in practice
- [ ] Confirm exam appointment and technical requirements
- [ ] Prepare exam day environment and materials

### Day Before Exam
- [ ] Light review of key formulas and limits
- [ ] Ensure technology setup works (for online proctoring)
- [ ] Prepare identification and workspace
- [ ] Get adequate sleep and nutrition

### Exam Day
- [ ] Arrive early or log in 30 minutes before
- [ ] Bring required identification documents
- [ ] Stay calm and manage time effectively (2 minutes per question)
- [ ] Read questions carefully and identify key requirements
- [ ] Use elimination strategy for multiple choice questions