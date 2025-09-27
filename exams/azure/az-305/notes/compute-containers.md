# Azure Compute & Containers Fundamentals

## Virtual Machines Basics

### What are Azure VMs?
Infrastructure-as-a-Service (IaaS) offering providing virtual machines in the cloud with full control over the operating system and applications.

### VM Series & Families
| Series | Purpose | Examples | Use Cases |
|--------|---------|----------|-----------|
| **General Purpose** | Balanced CPU-to-memory | B, D, F, E | Web servers, small-medium databases |
| **Compute Optimized** | High CPU-to-memory | F | CPU-intensive applications |
| **Memory Optimized** | High memory-to-CPU | E, M, G | Large databases, in-memory analytics |
| **Storage Optimized** | High disk throughput | L | Big data, SQL, NoSQL databases |
| **GPU** | Graphics processing | N | Machine learning, video rendering |
| **High Performance** | Fastest CPU processors | H | HPC, simulations |

### Popular VM Sizes
| Size | vCPUs | RAM | Use Case |
|------|-------|-----|----------|
| **B1s** | 1 | 1 GB | Development, testing |
| **D2s_v3** | 2 | 8 GB | Small production workloads |
| **D4s_v3** | 4 | 16 GB | Medium applications |
| **E4s_v3** | 4 | 32 GB | Memory-intensive apps |
| **F4s_v2** | 4 | 8 GB | CPU-intensive workloads |

### VM Pricing Options
| Option | Description | Discount | Use Case |
|--------|-------------|----------|----------|
| **Pay-as-you-go** | Hourly billing | None | Variable workloads |
| **Reserved Instances** | 1-3 year commitment | Up to 72% | Predictable workloads |
| **Spot Instances** | Unused capacity | Up to 90% | Fault-tolerant workloads |
| **Azure Hybrid Benefit** | Bring existing licenses | Significant | Windows/SQL Server migrations |

### Availability Options
| Option | SLA | Configuration | Use Case |
|--------|-----|---------------|----------|
| **Single VM** | 99.9% | Premium SSD required | Simple applications |
| **Availability Set** | 99.95% | Multiple fault/update domains | Traditional applications |
| **Availability Zone** | 99.99% | Across zones | Mission-critical applications |
| **Virtual Machine Scale Sets** | 99.95% | Auto-scaling group | Scalable applications |

## Azure Functions (Serverless)

### What are Azure Functions?
Event-driven serverless compute service that runs code on-demand without managing infrastructure.

### Key Characteristics
- **Event-driven**: Triggered by various Azure services
- **Automatic scaling**: From 0 to thousands of instances
- **No server management**: Azure handles infrastructure
- **Pay per execution**: Charge only for compute time used
- **Multiple languages**: C#, JavaScript, Python, Java, PowerShell

### Hosting Plans
| Plan | Scaling | Timeout | Use Case |
|------|---------|---------|----------|
| **Consumption** | Automatic | 5 minutes | Variable workloads |
| **Premium** | Pre-warmed | 30 minutes | Predictable performance |
| **Dedicated** | Manual/Auto | Unlimited | Existing App Service Plan |

### Supported Triggers
- **HTTP/HTTPS**: REST APIs, webhooks
- **Timer**: Scheduled execution (cron)
- **Blob Storage**: File upload/modification
- **Queue Storage**: Message processing
- **Event Hub**: Streaming data
- **Service Bus**: Enterprise messaging
- **Cosmos DB**: Database changes

### Function App Features
- **Deployment slots**: Staging and production environments
- **Application settings**: Environment variables and secrets
- **CORS support**: Cross-origin resource sharing
- **Authentication**: Built-in identity providers
- **Monitoring**: Application Insights integration

## App Service (Platform-as-a-Service)

### What is App Service?
Fully managed platform for building, deploying, and scaling web apps and APIs.

### App Service Plans
| Tier | Features | Use Case |
|------|----------|----------|
| **Free (F1)** | Shared infrastructure, 1GB storage | Development, testing |
| **Shared (D1)** | Shared infrastructure, custom domains | Small applications |
| **Basic (B1-B3)** | Dedicated VMs, SSL, manual scale | Production workloads |
| **Standard (S1-S3)** | Auto-scale, deployment slots | Standard production |
| **Premium (P1-P3)** | Enhanced performance, VNet integration | High-performance apps |
| **Isolated (I1-I3)** | Dedicated environment, private networking | Enterprise applications |

### Supported Platforms
- **Web Apps**: .NET, Java, PHP, Node.js, Python, Ruby
- **API Apps**: RESTful web services
- **Mobile Apps**: Mobile backend services
- **Function Apps**: Serverless functions

### Key Features
- **Auto-scaling**: Scale based on metrics
- **Deployment slots**: Blue-green deployments
- **Custom domains**: Bring your own domain
- **SSL certificates**: Free and custom certificates
- **Authentication**: Social and enterprise identity providers
- **Continuous deployment**: Git, GitHub, Azure DevOps integration

## Container Services

### Azure Container Instances (ACI)
**Simplest way to run containers**

#### Key Features
- **Serverless containers**: No VM management
- **Fast startup**: Containers start in seconds
- **Per-second billing**: Pay only for running time
- **Public/private**: Internet-accessible or VNet-connected
- **Persistent storage**: Azure Files mounting

#### Use Cases
- **Batch jobs**: Data processing tasks
- **Build agents**: CI/CD pipeline runners
- **Application testing**: Isolated test environments
- **Burst scaling**: Overflow capacity for AKS

### Azure Kubernetes Service (AKS)
**Managed Kubernetes service**

#### Key Features
- **Managed control plane**: Azure manages master nodes
- **Integrated monitoring**: Azure Monitor for containers
- **Azure AD integration**: RBAC with enterprise identity
- **Virtual Node**: Serverless pods with ACI
- **Dev Spaces**: Kubernetes development environments

#### Node Pool Types
| Type | Description | Use Case |
|------|-------------|----------|
| **System** | Run system pods | Required for cluster operation |
| **User** | Run application workloads | Your applications |
| **Spot** | Use spot VMs | Cost-effective batch workloads |

#### Scaling Options
- **Cluster Autoscaler**: Add/remove nodes based on demand
- **Horizontal Pod Autoscaler**: Scale pods based on metrics
- **Vertical Pod Autoscaler**: Adjust pod resource requests
- **Virtual Node**: Burst to ACI for unlimited scale

### Azure Container Registry (ACR)
**Private container registry**

#### Features
- **Geo-replication**: Replicate across regions
- **Security scanning**: Vulnerability assessment
- **Content trust**: Signed image validation
- **Helm charts**: Store Kubernetes applications
- **Tasks**: Build images in the cloud

#### Service Tiers
| Tier | Storage | Operations/month | Features |
|------|---------|------------------|----------|
| **Basic** | 10 GB | 10,000 | Manual builds |
| **Standard** | 100 GB | 100,000 | Webhooks, geo-replication |
| **Premium** | 500 GB | 500,000 | Advanced security, private endpoints |

## Azure Batch

### What is Azure Batch?
Cloud service for running large-scale parallel and high-performance computing (HPC) applications.

### Key Components
- **Pool**: Collection of compute nodes
- **Job**: Logical grouping of tasks
- **Task**: Individual unit of work
- **Application Package**: Software deployed to nodes

### Use Cases
- **Financial modeling**: Risk analysis, portfolio optimization
- **Digital media**: Video transcoding, 3D rendering
- **Engineering**: CAD, simulation, analysis
- **Scientific computing**: Research, data processing

### Scaling Options
- **Fixed**: Predetermined number of nodes
- **Auto-scale**: Dynamic scaling based on formulas
- **Low-priority VMs**: Use spot instances for cost savings

## Service Fabric

### What is Service Fabric?
Distributed systems platform for packaging, deploying, and managing scalable microservices.

### Application Models
- **Stateless services**: No local state storage
- **Stateful services**: Reliable local state storage
- **Actor model**: Virtual actor programming model
- **Guest executables**: Existing applications

### Key Features
- **Self-healing**: Automatic failure detection and recovery
- **Auto-scaling**: Scale services based on demand
- **Rolling upgrades**: Zero-downtime deployments
- **Service discovery**: Built-in service location
- **Health monitoring**: Application and infrastructure health

## Service Comparison & Decision Tree

### When to Choose What?

#### Compute Decision Tree
```
Need full OS control? → Virtual Machines
Containerized application? → AKS or Container Instances
Event-driven function? → Azure Functions
Web application/API? → App Service
HPC/batch processing? → Azure Batch
Microservices platform? → Service Fabric
```

#### Container Orchestration
```
Simple container deployment? → Container Instances
Need Kubernetes features? → AKS
Legacy application modernization? → Service Fabric
Hybrid on-premises/cloud? → Service Fabric
```

#### Serverless Options
```
HTTP APIs/web apps? → App Service or Functions
Event processing? → Azure Functions
Long-running processes? → Container Instances
Scheduled tasks? → Azure Functions (Timer trigger)
```

## Practical Examples

### Web Application Architecture
- **Frontend**: App Service Web App
- **API**: App Service API App or Azure Functions
- **Background processing**: Azure Functions
- **Static content**: Azure Storage + CDN
- **Database**: Azure SQL Database

### Microservices on AKS
- **Container platform**: Azure Kubernetes Service
- **Service mesh**: Istio or Linkerd
- **Load balancing**: Azure Load Balancer
- **Monitoring**: Azure Monitor for containers
- **Registry**: Azure Container Registry

### Batch Processing Pipeline
- **Orchestration**: Azure Batch
- **Data input**: Azure Storage
- **Processing**: Custom applications in containers
- **Results**: Azure Storage or databases
- **Monitoring**: Azure Monitor

### Event-Driven Architecture
- **Trigger**: Azure Storage blob upload
- **Processing**: Azure Functions
- **Messaging**: Service Bus or Event Hub
- **State**: Cosmos DB
- **Notifications**: Logic Apps

## Best Practices

### Virtual Machines
- Use appropriate VM sizes for workloads
- Implement availability sets or zones
- Use managed disks for storage
- Configure auto-shutdown for dev/test
- Monitor performance with Azure Monitor
- Use Azure Security Center recommendations

### Azure Functions
- Keep functions small and focused
- Use appropriate hosting plan for workload
- Implement proper error handling
- Use Application Insights for monitoring
- Store secrets in Key Vault
- Design for idempotency

### App Service
- Use deployment slots for staging
- Configure auto-scaling rules
- Implement health checks
- Use Application Insights for monitoring
- Store configuration in app settings
- Use custom domains and SSL certificates

### Containers (AKS/ACI)
- Use multi-stage Docker builds
- Implement resource limits and requests
- Use Azure Container Registry for images
- Implement proper logging and monitoring
- Use Kubernetes RBAC for security
- Regular security scanning of images

## Cost Optimization

### General Strategies
- **Right-sizing**: Choose appropriate VM sizes
- **Reserved Instances**: Commit to long-term usage
- **Azure Hybrid Benefit**: Use existing licenses
- **Spot Instances**: Use for fault-tolerant workloads
- **Auto-shutdown**: Automatically stop dev/test VMs

### Service-Specific Optimization
- **VMs**: Use B-series for variable workloads
- **App Service**: Choose appropriate tier for needs
- **Functions**: Optimize execution time and memory
- **AKS**: Use spot node pools for batch workloads
- **Container Instances**: Use for short-running tasks

## Monitoring & Management

### Azure Monitor
- **Metrics**: Performance and health data
- **Logs**: Detailed operational data
- **Alerts**: Proactive notifications
- **Dashboards**: Visualization and reporting
- **Application Insights**: Application performance monitoring

### Common Metrics to Monitor
- **VMs**: CPU, memory, disk, network
- **App Service**: Response time, throughput, errors
- **Functions**: Execution count, duration, errors
- **AKS**: Node health, pod status, resource usage

## Common Pitfalls

### Virtual Machine Issues
- Not using managed disks
- Insufficient planning for high availability
- Over-provisioning resources
- Not implementing proper backup strategies
- Ignoring security recommendations

### App Service Issues
- Wrong service plan tier selection
- Not using deployment slots
- Inadequate scaling configuration
- Missing SSL certificate setup
- Poor application architecture for scale

### Container Issues
- Large container images affecting startup
- Not implementing resource limits
- Poor secrets management
- Inadequate health check configuration
- Missing monitoring and logging

### Azure Functions Issues
- Cold start impact on performance
- Timeout issues with long-running processes
- Incorrect trigger configuration
- Not implementing proper retry logic
- Inappropriate hosting plan selection