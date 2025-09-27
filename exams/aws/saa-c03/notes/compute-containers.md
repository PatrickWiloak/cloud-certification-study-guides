# AWS Compute & Containers Fundamentals

## EC2 (Elastic Compute Cloud) Basics

### What is EC2?
Virtual servers in the cloud that provide resizable compute capacity. Think of EC2 as renting computers in Amazon's data centers.

### Instance Types & Families
| Family | Purpose | Examples | Use Cases |
|--------|---------|----------|-----------|
| **General Purpose** | Balanced CPU, memory, networking | t3, m5, a1 | Web servers, small databases |
| **Compute Optimized** | High-performance processors | c5, c6i | CPU-intensive applications |
| **Memory Optimized** | Fast performance for large datasets | r5, x1e, z1d | In-memory databases, real-time analytics |
| **Storage Optimized** | High sequential read/write | i3, d2, h1 | Distributed file systems, data warehouses |
| **Accelerated Computing** | Hardware accelerators (GPUs) | p3, g4, f1 | Machine learning, video processing |

### Instance Sizing
- **Nano/Micro**: 1-2 vCPUs, minimal memory - Development, low-traffic sites
- **Small/Medium**: 2-4 vCPUs, moderate memory - Small production workloads
- **Large/XLarge**: 4-16 vCPUs, substantial memory - Production applications
- **2XL and beyond**: 16+ vCPUs, high memory - Enterprise workloads

### Purchasing Options
| Option | Description | Cost | Use Case |
|--------|-------------|------|----------|
| **On-Demand** | Pay per hour/second | Highest | Unpredictable workloads |
| **Reserved Instances** | 1-3 year commitment | 40-75% discount | Steady-state workloads |
| **Spot Instances** | Use spare capacity | Up to 90% discount | Fault-tolerant, flexible workloads |
| **Dedicated Hosts** | Physical server for you | Most expensive | Compliance, licensing requirements |

### Key Concepts
- **AMI (Amazon Machine Image)**: Template to launch instances
- **Security Groups**: Virtual firewall rules (stateful)
- **Key Pairs**: SSH access to Linux instances
- **User Data**: Scripts that run when instance starts
- **Instance Store**: Temporary storage attached to instance
- **Placement Groups**: Control instance placement for performance

## AWS Lambda Fundamentals

### What is Lambda?
Serverless compute service that runs code without managing servers. You pay only for compute time consumed.

### Key Characteristics
- **Event-driven**: Triggered by events from other AWS services
- **Automatic scaling**: From 0 to thousands of concurrent executions
- **No server management**: AWS handles infrastructure
- **15-minute limit**: Maximum execution time per function
- **Stateless**: Each invocation is independent

### Supported Runtimes
- **Node.js**: JavaScript runtime
- **Python**: Popular for data processing
- **Java**: Enterprise applications
- **C#/.NET**: Microsoft stack
- **Go**: High-performance applications
- **Ruby**: Web development
- **Custom Runtime**: Bring your own runtime

### Common Triggers
- **API Gateway**: HTTP requests
- **S3**: File uploads/changes
- **DynamoDB**: Database changes
- **CloudWatch Events**: Scheduled events
- **SNS/SQS**: Messages
- **Kinesis**: Streaming data

### Pricing Model
- **Requests**: $0.20 per 1M requests
- **Duration**: Based on memory allocated and execution time
- **Free Tier**: 1M requests and 400,000 GB-seconds per month

## Container Services Overview

### ECS (Elastic Container Service)
**AWS-native container orchestration service**

#### Key Components
- **Task Definition**: Blueprint for your application (like a recipe)
- **Service**: Manages desired number of tasks
- **Cluster**: Logical grouping of compute resources
- **Task**: Running instance of a task definition

#### Launch Types
| Type | Description | Use Case |
|------|-------------|----------|
| **EC2** | Run on your EC2 instances | More control, cost optimization |
| **Fargate** | Serverless containers | No server management, simpler |

#### When to Use ECS
- Need tight AWS integration
- Want AWS-native solution
- Simpler container orchestration needs
- Cost-sensitive workloads (with EC2 launch type)

### EKS (Elastic Kubernetes Service)
**Managed Kubernetes service**

#### Key Features
- **Upstream Kubernetes**: Standard Kubernetes APIs
- **Managed Control Plane**: AWS manages master nodes
- **Worker Nodes**: You manage (or use managed node groups)
- **Pod Security**: Integration with AWS security services

#### When to Use EKS
- Already using Kubernetes
- Need Kubernetes-specific features
- Multi-cloud or hybrid strategy
- Complex microservices architectures

### Fargate Deep Dive
**Serverless compute for containers**

#### Benefits
- **No EC2 management**: AWS handles infrastructure
- **Right-sizing**: Pay for exact CPU/memory used
- **Security**: Each task gets its own kernel
- **Integration**: Works with ECS and EKS

#### Limitations
- **Cost**: More expensive than EC2 for steady workloads
- **Customization**: Less control over underlying infrastructure
- **Storage**: Limited ephemeral storage options

## Container Fundamentals

### What are Containers?
Lightweight, portable packages that include application code and all dependencies needed to run.

### Key Concepts
- **Docker Image**: Read-only template to create containers
- **Container**: Running instance of an image
- **Registry**: Storage for container images (ECR, Docker Hub)
- **Dockerfile**: Instructions to build an image

### Benefits of Containers
- **Consistency**: Same environment from dev to production
- **Portability**: Run anywhere containers are supported
- **Efficiency**: Share OS kernel, faster than VMs
- **Scalability**: Quick to start and stop

## Service Comparison & Decision Tree

### When to Choose What?

#### EC2 vs Lambda
```
Need full OS control? → EC2
Runs > 15 minutes? → EC2
Unpredictable traffic with burst patterns? → Lambda
Event-driven workload? → Lambda
Want to manage servers? → EC2
Want serverless? → Lambda
```

#### ECS vs EKS
```
Already using Kubernetes? → EKS
Need Kubernetes features? → EKS
Want simpler container orchestration? → ECS
Prefer AWS-native solution? → ECS
Planning multi-cloud? → EKS
Cost-sensitive? → ECS (with EC2)
```

#### EC2 vs Fargate (for containers)
```
Need custom EC2 configurations? → EC2
Want predictable pricing? → EC2 (with Reserved Instances)
Want no server management? → Fargate
Have variable workloads? → Fargate
Need specialized instance types? → EC2
```

## Practical Examples

### Basic Web Application
- **Frontend**: EC2 instances behind ALB
- **Backend API**: Lambda functions with API Gateway
- **Database**: RDS for persistence
- **Static Assets**: S3 + CloudFront

### Microservices Architecture
- **Container Platform**: EKS or ECS
- **Service Discovery**: AWS Cloud Map
- **Load Balancing**: ALB with target groups
- **Monitoring**: CloudWatch + X-Ray

### Event Processing
- **Trigger**: S3 file upload
- **Processing**: Lambda function
- **Queue**: SQS for batching
- **Storage**: DynamoDB for results

## Best Practices

### EC2 Best Practices
- Use appropriate instance types for workload
- Implement Auto Scaling for availability
- Use IAM roles instead of access keys
- Regularly patch and update AMIs
- Monitor with CloudWatch
- Use security groups as firewalls

### Lambda Best Practices
- Keep functions small and focused
- Use environment variables for configuration
- Implement proper error handling
- Optimize memory allocation for cost
- Use VPC only when necessary
- Monitor with X-Ray and CloudWatch

### Container Best Practices
- Use multi-stage Dockerfile builds
- Implement health checks
- Use least-privilege security
- Store secrets in AWS Secrets Manager
- Implement proper logging
- Use ECR for image storage

## Common Gotchas

### EC2 Pitfalls
- Forgetting to stop instances (cost)
- Not using Auto Scaling (availability)
- Incorrect security group rules
- Instance store data loss on stop

### Lambda Limitations
- 15-minute execution limit
- 10GB memory maximum
- Cold start latency
- VPC networking complexity

### Container Challenges
- Image size optimization
- Secret management
- Network configuration
- Storage persistence