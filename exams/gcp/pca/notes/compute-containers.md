# GCP Compute & Containers Fundamentals

## Compute Engine Basics

### What is Compute Engine?
Google Cloud's Infrastructure-as-a-Service (IaaS) offering providing virtual machines running on Google's infrastructure.

### Machine Types & Families
| Family | Purpose | Examples | Use Cases |
|--------|---------|----------|-----------|
| **General Purpose** | Balanced CPU, memory | e2, n1, n2, n2d | Web servers, small-medium databases |
| **Compute Optimized** | High-performance processors | c2, c2d | CPU-intensive applications, gaming |
| **Memory Optimized** | Large memory-to-vCPU ratios | m1, m2 | In-memory databases, analytics |
| **Accelerator Optimized** | GPU workloads | a2 | Machine learning, HPC |

### Machine Series Comparison
| Series | Processor | Use Case | Key Features |
|--------|-----------|----------|--------------|
| **E2** | Intel/AMD | Cost-effective general purpose | Shared-core, automatic resource optimization |
| **N1** | Intel Skylake/Broadwell | Balanced performance | Proven, reliable, wide range of sizes |
| **N2** | Intel Cascade Lake | High performance per core | Latest Intel processors, custom machine types |
| **N2D** | AMD EPYC Rome | Price-performance optimized | AMD processors, competitive pricing |
| **C2** | Intel Cascade Lake | Compute-intensive | Highest single-thread performance |

### Custom Machine Types
- **Flexibility**: Choose exact vCPU and memory combinations
- **Cost Optimization**: Pay only for what you need
- **Limitations**: Must follow Google's supported ratios
- **Use Case**: Right-sizing for specific workloads

### Preemptible & Spot VMs
| Feature | Preemptible VMs | Spot VMs |
|---------|-----------------|----------|
| **Discount** | Up to 80% | Up to 91% |
| **Termination** | Max 24 hours | Can run indefinitely |
| **Notice** | 30 seconds | 30 seconds |
| **Availability** | No guarantees | Better availability |
| **Use Case** | Batch jobs, fault-tolerant workloads | Modern replacement for preemptible |

## Cloud Functions (Serverless)

### What are Cloud Functions?
Event-driven serverless compute platform that automatically scales based on demand.

### Key Characteristics
- **Event-driven**: Triggered by HTTP requests, Cloud Storage, Pub/Sub
- **Automatic scaling**: From 0 to thousands of instances
- **No server management**: Google handles infrastructure
- **9-minute timeout**: Maximum execution time (1st gen), 60 minutes (2nd gen)
- **Stateless**: Each invocation is independent

### Supported Runtimes
- **Node.js**: JavaScript runtime
- **Python**: Popular for data processing and automation
- **Go**: High-performance applications
- **Java**: Enterprise applications
- **.NET**: Microsoft stack
- **Ruby**: Web development
- **PHP**: Web applications

### Trigger Types
- **HTTP(S)**: Direct HTTP requests
- **Cloud Storage**: File upload/delete/update
- **Pub/Sub**: Message queue triggers
- **Firestore**: Database changes
- **Firebase**: Authentication, real-time database
- **Cloud Scheduler**: Cron-like scheduling

### Generations Comparison
| Feature | 1st Gen | 2nd Gen |
|---------|---------|---------|
| **Timeout** | 9 minutes | 60 minutes |
| **Instances** | 1000 concurrent | 1000+ concurrent |
| **Memory** | Up to 8GB | Up to 32GB |
| **CPU** | Shared | Dedicated available |
| **Traffic Allocation** | Limited | Advanced |

## Cloud Run (Containers)

### What is Cloud Run?
Fully managed serverless platform for running containerized applications that automatically scales to zero.

### Key Features
- **Container-based**: Run any containerized application
- **Serverless**: No infrastructure management
- **Scale to zero**: Pay only when handling requests
- **Portable**: Standard containers work anywhere
- **HTTP/HTTPS**: RESTful APIs and web applications

### Cloud Run vs Cloud Functions
| Feature | Cloud Run | Cloud Functions |
|---------|-----------|-----------------|
| **Runtime** | Any container | Specific runtimes |
| **Languages** | Any language | Supported runtimes only |
| **Deployment** | Container images | Source code |
| **Concurrency** | Up to 1000 per instance | 1 per instance (1st gen) |
| **Cold Start** | Generally faster | Varies by runtime |
| **Use Case** | Web apps, APIs | Event processing |

### Cloud Run Services vs Jobs
| Type | Purpose | Execution | Use Case |
|------|---------|-----------|----------|
| **Services** | Handle requests | On-demand | Web applications, APIs |
| **Jobs** | Run to completion | Scheduled/triggered | Batch processing, data pipelines |

## Google Kubernetes Engine (GKE)

### What is GKE?
Managed Kubernetes service that provides a fully managed control plane and node pools.

### Cluster Types
| Type | Management Level | Use Case |
|------|------------------|----------|
| **Standard** | You manage nodes | Full control, cost optimization |
| **Autopilot** | Google manages nodes | Simplified operations, optimized |

### Standard vs Autopilot
| Feature | Standard | Autopilot |
|---------|----------|-----------|
| **Node Management** | Manual | Automatic |
| **Pricing** | Pay for all nodes | Pay for pods only |
| **Flexibility** | Full control | Opinionated setup |
| **Security** | You configure | Hardened by default |
| **Operations** | More complex | Simplified |

### Node Pool Options
- **Machine Types**: Choose appropriate VM types
- **Autoscaling**: Automatically add/remove nodes
- **Preemptible Nodes**: Cost savings for fault-tolerant workloads
- **Spot Pods**: Run pods on Spot VMs
- **Multi-zone**: Spread nodes across zones

### Key GKE Features
- **Workload Identity**: Secure pod access to Google Cloud services
- **Binary Authorization**: Ensure only trusted images are deployed
- **Pod Security Standards**: Enforce security policies
- **Cluster Autoscaler**: Scale nodes based on demand
- **Vertical Pod Autoscaler**: Right-size pod resource requests

## App Engine (Platform-as-a-Service)

### What is App Engine?
Fully managed serverless platform for deploying applications with automatic scaling and traffic management.

### Environments
| Environment | Language Support | Flexibility | Use Case |
|-------------|------------------|-------------|----------|
| **Standard** | Specific runtimes | Limited | Simple web apps, APIs |
| **Flexible** | Any runtime in container | More flexible | Complex applications |

### Standard Environment Features
- **Automatic scaling**: Scale to zero or handle traffic spikes
- **Free tier**: Generous free quotas
- **Sandboxed**: Secure runtime environment
- **Fast startup**: Quick instance startup times
- **Integrated services**: Built-in integration with Google services

### Flexible Environment Features
- **Custom runtimes**: Any language or framework
- **SSH access**: Debug running instances
- **Docker containers**: Custom container images
- **More resources**: Higher memory and CPU limits
- **Local disk**: Persistent local storage

## Container Registry & Artifact Registry

### Container Registry (Legacy)
- **Purpose**: Store and manage Docker container images
- **Storage**: Built on Cloud Storage
- **Status**: Being replaced by Artifact Registry

### Artifact Registry (Current)
- **Purpose**: Universal package manager for containers and language packages
- **Formats**: Docker, Maven, npm, Python, Go modules
- **Features**: Regional storage, IAM integration, vulnerability scanning
- **Security**: Container analysis, binary authorization

## Service Comparison & Decision Tree

### When to Choose What?

#### Compute Engine vs App Engine vs Cloud Run vs Cloud Functions
```
Need full OS control? → Compute Engine
Traditional web application? → App Engine
Containerized application? → Cloud Run
Event-driven function? → Cloud Functions
```

#### Container Orchestration Decision
```
Need Kubernetes features? → GKE
Want Google to manage everything? → GKE Autopilot
Simple container deployment? → Cloud Run
Traditional PaaS approach? → App Engine
```

#### Serverless Options
```
HTTP API/web app? → Cloud Run or App Engine
Event processing? → Cloud Functions
Scheduled jobs? → Cloud Run Jobs
Batch processing? → Cloud Functions or Cloud Run Jobs
```

## Practical Examples

### Web Application Architecture
- **Frontend**: App Engine or Cloud Run
- **API**: Cloud Run or Cloud Functions
- **Background Jobs**: Cloud Functions or Cloud Run Jobs
- **Static Assets**: Cloud Storage + Cloud CDN

### Microservices on GKE
- **Container Platform**: GKE Autopilot
- **Service Mesh**: Istio (available as GKE add-on)
- **Load Balancing**: Google Cloud Load Balancer
- **Monitoring**: Google Cloud Operations

### Event-Driven Processing
- **Trigger**: Cloud Storage file upload
- **Processing**: Cloud Functions
- **Queue**: Pub/Sub for complex workflows
- **Storage**: Cloud Storage or Firestore

### Batch Processing
- **Orchestration**: Cloud Composer (Airflow)
- **Processing**: Cloud Run Jobs or Compute Engine
- **Data**: Cloud Storage for input/output
- **Monitoring**: Cloud Monitoring

## Best Practices

### Compute Engine
- Use appropriate machine types for workloads
- Implement managed instance groups for scaling
- Use service accounts for API access
- Regular OS patching and updates
- Monitor with Cloud Monitoring
- Use sustained use discounts

### Cloud Functions
- Keep functions small and focused
- Use environment variables for configuration
- Implement proper error handling and retries
- Optimize for cold start performance
- Use appropriate memory allocation
- Monitor function performance

### Cloud Run
- Design for stateless operation
- Use appropriate concurrency settings
- Implement health checks
- Use secrets for sensitive configuration
- Monitor container resource usage
- Configure appropriate timeout values

### GKE
- Use Workload Identity for secure access
- Implement resource requests and limits
- Use namespaces for organization
- Regular cluster and node updates
- Monitor cluster and application metrics
- Implement proper RBAC

## Cost Optimization

### General Strategies
- **Right-sizing**: Choose appropriate machine types and sizes
- **Sustained Use Discounts**: Automatic discounts for consistent usage
- **Committed Use Discounts**: 1-3 year commitments for predictable workloads
- **Preemptible/Spot Instances**: Up to 80-91% savings for fault-tolerant workloads

### Service-Specific Optimization
- **Compute Engine**: Use custom machine types, sustained use discounts
- **Cloud Functions**: Optimize memory allocation and execution time
- **Cloud Run**: Configure appropriate concurrency and CPU allocation
- **GKE**: Use Autopilot for automatic optimization, preemptible nodes

## Common Gotchas

### Compute Engine
- Instance charges continue until stopped
- Persistent disks have separate charges
- External IP addresses cost extra
- Regional persistent disks cost more but offer better durability

### Cloud Functions
- Cold start latency for infrequently used functions
- 9-minute timeout limit (1st gen)
- Memory and CPU limitations
- Event trigger configuration complexity

### Cloud Run
- Request timeout limits
- CPU allocation only during request processing (by default)
- Container image size affects cold start
- Concurrency settings impact performance

### GKE
- Node pools continue running unless autoscaled to zero
- Complex networking configurations
- Persistent volume management
- Cluster upgrade coordination