# AWS Compute Services

## Amazon EC2 (Elastic Compute Cloud)

### Overview
Amazon EC2 provides scalable computing capacity in the AWS cloud. It eliminates the need to invest in hardware upfront and allows you to launch virtual servers (instances) as needed.

### Key Features
- **Virtual Servers**: Launch instances in minutes
- **Scalable**: Scale up or down based on demand
- **Flexible**: Choose from various instance types
- **Secure**: Integrated with VPC and security groups
- **Cost-Effective**: Pay only for what you use

### Instance Types

#### General Purpose
- **T4g, T3, T2**: Burstable performance instances
  - Use cases: Web servers, small databases, development environments
  - CPU Credits: Accumulate credits when below baseline performance

- **M6i, M5, M4**: Balanced compute, memory, and networking
  - Use cases: Web applications, microservices, enterprise applications
  - Characteristics: Balanced resources for diverse workloads

#### Compute Optimized
- **C6i, C5, C4**: High-performance processors
  - Use cases: High-performance web servers, scientific modeling, batch processing
  - Characteristics: High ratio of compute to memory

#### Memory Optimized
- **R6i, R5, R4**: Memory optimized
  - Use cases: In-memory databases, real-time analytics
  - Characteristics: High memory-to-vCPU ratio

- **X1e, X1**: High memory instances
  - Use cases: In-memory databases like SAP HANA, Apache Spark
  - Characteristics: Highest memory per vCPU ratio

- **z1d**: High frequency and NVMe SSD
  - Use cases: Electronic Design Automation (EDA), relational databases
  - Characteristics: High frequency processors with NVMe SSD

#### Storage Optimized
- **I4i, I3**: NVMe SSD storage
  - Use cases: NoSQL databases, data warehousing, search engines
  - Characteristics: High random I/O performance

- **D3, D2**: Dense HDD storage
  - Use cases: Distributed file systems, data processing workloads
  - Characteristics: High density HDD storage

#### Accelerated Computing
- **P4, P3**: GPU instances for machine learning
  - Use cases: Machine learning training, high-performance computing
  - Characteristics: NVIDIA Tesla GPUs

- **G4**: GPU instances for graphics workloads
  - Use cases: Game streaming, 3D visualization, video processing
  - Characteristics: NVIDIA T4 GPUs

- **F1**: FPGA instances
  - Use cases: Hardware acceleration, real-time video processing
  - Characteristics: Field Programmable Gate Arrays

### Pricing Models

#### On-Demand Instances
- **Description**: Pay for compute capacity by hour or second
- **Use Cases**: Short-term, irregular workloads
- **Benefits**: No long-term commitments, launch/terminate anytime
- **Billing**: Per-second billing (minimum 60 seconds)

#### Reserved Instances (RIs)
- **Description**: Commit to 1 or 3-year terms for significant discounts
- **Discount**: Up to 75% compared to On-Demand
- **Types**:
  - **Standard RIs**: Up to 75% discount, can modify AZ, instance size, networking type
  - **Convertible RIs**: Up to 54% discount, can change instance family, OS, tenancy
  - **Scheduled RIs**: For predictable recurring schedules

#### Spot Instances
- **Description**: Use spare EC2 capacity at up to 90% discount
- **Use Cases**: Fault-tolerant, flexible applications
- **Interruption**: Can be terminated with 2-minute notice
- **Best For**: Batch jobs, data analysis, image processing

#### Dedicated Hosts
- **Description**: Physical EC2 server dedicated for your use
- **Use Cases**: Licensing requirements, compliance, security
- **Benefits**: Visibility into host-level resources, BYOL (Bring Your Own License)
- **Pricing**: Per-host billing

#### Dedicated Instances
- **Description**: Instances run on hardware dedicated to single customer
- **Use Cases**: Compliance requirements, licensing restrictions
- **Difference from Dedicated Hosts**: No visibility into underlying hardware

### Storage Options

#### Instance Store (Ephemeral Storage)
- **Description**: Temporary block-level storage
- **Characteristics**: High performance, directly attached to host
- **Limitation**: Data lost when instance stops/terminates
- **Use Cases**: Temporary storage, caches, scratch data

#### Amazon EBS (Elastic Block Store)
- **Description**: Persistent block storage volumes
- **Types**:
  - **gp3/gp2**: General Purpose SSD
  - **io2/io1**: Provisioned IOPS SSD
  - **st1**: Throughput Optimized HDD
  - **sc1**: Cold HDD
- **Features**: Snapshots, encryption, Multi-Attach

### Security Features

#### Security Groups
- **Description**: Virtual firewalls controlling traffic
- **Default**: Deny all inbound, allow all outbound
- **Rules**: Based on protocol, port, source/destination
- **Stateful**: Return traffic automatically allowed

#### Key Pairs
- **Description**: Public/private key pairs for secure login
- **SSH**: Used for Linux instances
- **RDP**: Used for Windows instances
- **Best Practice**: Use separate key pairs for different environments

#### IAM Roles for EC2
- **Description**: Assign permissions to EC2 instances
- **Benefits**: No need to store credentials on instances
- **Temporary**: Automatically rotated credentials
- **Best Practice**: Use roles instead of access keys

### High Availability and Scaling

#### Auto Scaling
- **Description**: Automatically adjust number of instances
- **Components**:
  - **Launch Template/Configuration**: Instance specifications
  - **Auto Scaling Group**: Collection of instances
  - **Scaling Policies**: Rules for scaling actions
- **Benefits**: Cost optimization, high availability, fault tolerance

#### Elastic Load Balancing
- **Application Load Balancer (ALB)**: Layer 7 load balancing
- **Network Load Balancer (NLB)**: Layer 4 load balancing
- **Classic Load Balancer (CLB)**: Legacy load balancer
- **Gateway Load Balancer (GWLB)**: Third-party virtual appliances

#### Placement Groups
- **Cluster**: Low latency, high network performance
- **Partition**: Large distributed/replicated workloads
- **Spread**: Small number of critical instances

## AWS Lambda

### Overview
AWS Lambda is a serverless compute service that runs code without provisioning or managing servers. You pay only for the compute time consumed.

### Key Features
- **Serverless**: No server management required
- **Event-Driven**: Triggered by events from other AWS services
- **Automatic Scaling**: Scales automatically based on demand
- **Pay-per-Use**: Pay only for compute time used
- **Multiple Languages**: Support for various programming languages

### Supported Runtimes
- **Python**: 3.7, 3.8, 3.9, 3.10, 3.11
- **Node.js**: 14.x, 16.x, 18.x, 20.x
- **Java**: 8, 11, 17, 21
- **C#**: .NET Core 3.1, .NET 6, .NET 8
- **Go**: 1.x
- **Ruby**: 2.7, 3.2
- **Custom Runtime**: Using Runtime API

### Key Concepts

#### Function
- **Definition**: Code and configuration that processes events
- **Components**: Code, runtime, handler, environment variables
- **Limits**: 15-minute maximum execution time, 10GB memory

#### Event Sources
- **AWS Services**: S3, DynamoDB, Kinesis, SNS, SQS, API Gateway
- **Direct Invocation**: AWS CLI, SDKs, Lambda console
- **Schedule**: CloudWatch Events/EventBridge cron expressions

#### Execution Environment
- **Container**: Secure, isolated execution environment
- **Cold Start**: First invocation or after period of inactivity
- **Warm Start**: Reuse existing execution environment
- **Provisioned Concurrency**: Pre-initialized execution environments

### Pricing Model
- **Requests**: First 1 million requests per month free, then $0.20 per 1M requests
- **Duration**: Charged per 1ms, based on memory allocated
- **Free Tier**: 1M free requests and 400,000 GB-seconds per month

### Use Cases
- **Real-time File Processing**: Process files uploaded to S3
- **Data Transformation**: Transform data in streams
- **Web Applications**: Serverless web backends
- **Chatbots**: Process messaging platform events
- **Scheduled Tasks**: Automated backup, cleanup operations

### Best Practices
- **Optimize Memory**: Right-size memory allocation
- **Minimize Cold Starts**: Keep functions warm, use provisioned concurrency
- **Environment Variables**: Store configuration outside code
- **Error Handling**: Implement proper error handling and retries
- **Monitoring**: Use CloudWatch for monitoring and logging

## Amazon ECS (Elastic Container Service)

### Overview
Amazon ECS is a fully managed container orchestration service that helps you deploy, manage, and scale containerized applications.

### Key Components

#### Task Definition
- **Description**: Blueprint for your application
- **Contents**: Container images, CPU/memory requirements, networking
- **Versioning**: Immutable, new versions created for changes

#### Service
- **Description**: Runs and maintains desired number of tasks
- **Features**: Load balancing, auto scaling, rolling deployments
- **Health Checks**: Automatic replacement of unhealthy tasks

#### Cluster
- **Description**: Logical grouping of compute resources
- **Types**: EC2 clusters, Fargate clusters
- **Scaling**: Add/remove instances based on demand

### Launch Types

#### EC2 Launch Type
- **Description**: Run containers on EC2 instances
- **Management**: You manage the underlying EC2 instances
- **Use Cases**: More control over infrastructure, cost optimization
- **Responsibility**: Instance patching, scaling, monitoring

#### Fargate Launch Type
- **Description**: Serverless compute for containers
- **Management**: AWS manages the underlying infrastructure
- **Use Cases**: Focus on application, not infrastructure
- **Benefits**: No instance management, automatic scaling

### Integration with AWS Services
- **Application Load Balancer**: Distribute traffic across containers
- **CloudWatch**: Monitoring and logging
- **IAM**: Task-level permissions
- **ECR**: Container image registry
- **VPC**: Network isolation and security

### Use Cases
- **Microservices**: Break applications into smaller services
- **Batch Processing**: Run batch jobs on containers
- **Machine Learning**: Deploy ML models in containers
- **Web Applications**: Host scalable web applications

## Amazon EKS (Elastic Kubernetes Service)

### Overview
Amazon EKS is a managed Kubernetes service that makes it easy to run Kubernetes on AWS without needing to install and operate your own Kubernetes control plane.

### Key Features
- **Managed Control Plane**: AWS manages Kubernetes masters
- **High Availability**: Control plane across multiple AZs
- **Security**: Integrated with IAM and VPC
- **Compatibility**: Certified Kubernetes conformant
- **Automatic Updates**: Managed updates and patches

### Components

#### Control Plane
- **Management**: Fully managed by AWS
- **High Availability**: Runs across multiple AZs
- **Scaling**: Automatic scaling based on demand
- **Updates**: Managed updates and patches

#### Worker Nodes
- **Options**: EC2 instances, Fargate, or both
- **Management**: Customer managed (EC2) or AWS managed (Fargate)
- **Auto Scaling**: Kubernetes Cluster Autoscaler
- **Node Groups**: Managed groups of EC2 instances

#### Fargate for EKS
- **Serverless**: Run pods without managing nodes
- **Isolation**: Each pod runs in its own compute environment
- **Scaling**: Automatic scaling based on pod requirements
- **Billing**: Pay per pod resource consumption

### Use Cases
- **Container Orchestration**: Advanced container management
- **Hybrid Deployments**: Consistent Kubernetes across environments
- **Microservices**: Service mesh and advanced networking
- **Machine Learning**: Kubernetes-native ML workflows

### Integration with AWS Services
- **ALB**: Application Load Balancer for Ingress
- **IAM**: Service accounts for pods (IRSA)
- **VPC CNI**: Native VPC networking
- **ECR**: Container image registry
- **CloudWatch**: Container insights and logging

## AWS Batch

### Overview
AWS Batch enables you to run batch computing workloads on the AWS Cloud by dynamically provisioning the optimal quantity and type of compute resources.

### Key Components

#### Job Definition
- **Description**: Template for job execution
- **Parameters**: vCPUs, memory, IAM role, container image
- **Types**: Single-job, multi-node parallel jobs

#### Job Queue
- **Description**: Queue where jobs are submitted
- **Priority**: Numerical priority for job execution
- **Association**: Linked to compute environments

#### Compute Environment
- **Managed**: AWS manages EC2 instances
- **Unmanaged**: You manage EC2 instances
- **Fargate**: Serverless compute option
- **Spot**: Use Spot instances for cost optimization

### Use Cases
- **Financial Risk Modeling**: Monte Carlo simulations
- **Drug Discovery**: Molecular modeling
- **Image/Video Processing**: Media rendering
- **Scientific Computing**: Genomics, weather modeling

## AWS Lightsail

### Overview
Amazon Lightsail is designed to be the easiest way to launch and manage a virtual private server with AWS. It includes everything you need to jumpstart your project.

### Key Features
- **Simple**: Easy-to-use interface
- **Predictable Pricing**: Fixed monthly pricing
- **Pre-configured**: Templates for common applications
- **Integrated**: CDN, DNS, load balancer included
- **Scalable**: Can migrate to EC2 when needed

### Use Cases
- **Simple Web Applications**: WordPress, LAMP stack
- **Development/Test Environments**: Quick environment setup
- **Small Business Websites**: Cost-effective hosting
- **Learning/Experimentation**: AWS introduction

### Included Features
- **Compute**: Virtual private server
- **Storage**: SSD-based storage
- **Networking**: Data transfer allowances
- **DNS**: DNS management
- **Static IP**: Static IP addresses
- **Monitoring**: Basic CloudWatch metrics

## Key Takeaways

1. **EC2**: Foundation of AWS compute, flexible virtual servers with multiple pricing options
2. **Lambda**: Serverless computing, event-driven, pay-per-use model
3. **ECS**: Container orchestration, choice between EC2 and Fargate launch types
4. **EKS**: Managed Kubernetes service with full Kubernetes compatibility
5. **Batch**: Managed batch computing for large-scale parallel workloads
6. **Lightsail**: Simplified VPS solution with predictable pricing
7. **Choose Right Service**: Based on requirements for control, management, and complexity
8. **Cost Optimization**: Multiple pricing models and options available for different use cases