# AWS Service Comparisons

## Compute Services

| Service | Use Case | Pros | Cons | When to Choose |
|---------|----------|------|------|----------------|
| **EC2** | Virtual servers | Full control, wide instance types | Manage OS, patching | Need custom configurations |
| **Lambda** | Event-driven functions | Serverless, automatic scaling | 15min limit, cold starts | Short, stateless workloads |
| **ECS** | Container orchestration | AWS-native, Fargate option | Learning curve | Containerized applications |
| **EKS** | Kubernetes | Industry standard | Complex, expensive | Kubernetes expertise |
| **Batch** | Batch job processing | Managed queuing, spot support | Not real-time | Large-scale batch jobs |
| **Lightsail** | Simple VPS | Easy setup, predictable pricing | Limited features | Simple web applications |

## Storage Services

| Service | Type | Use Case | Durability | Access Pattern |
|---------|------|----------|------------|----------------|
| **S3 Standard** | Object | Frequently accessed data | 99.999999999% | Regular access |
| **S3 IA** | Object | Infrequently accessed | 99.999999999% | Monthly access |
| **S3 One Zone-IA** | Object | Reproducible, infrequent | 99.999999999% | Monthly, no AZ resilience |
| **S3 Glacier** | Object | Archive, backup | 99.999999999% | Minutes to hours retrieval |
| **S3 Deep Archive** | Object | Long-term archive | 99.999999999% | 12+ hours retrieval |
| **EBS** | Block | EC2 instance storage | 99.999% | High IOPS, low latency |
| **EFS** | File | Shared file storage | 99.999999999% | Concurrent access |
| **FSx** | File | High-performance workloads | 99.999999999% | Windows, Lustre |

## Database Services

| Service | Type | Engine | Use Case | Scaling |
|---------|------|--------|----------|---------|
| **RDS** | Relational | MySQL, PostgreSQL, etc. | OLTP, complex queries | Vertical, read replicas |
| **Aurora** | Relational | MySQL, PostgreSQL compatible | High performance OLTP | Auto-scaling |
| **DynamoDB** | NoSQL | Proprietary | Single-digit ms latency | Horizontal |
| **DocumentDB** | NoSQL | MongoDB compatible | Document storage | Horizontal |
| **Redshift** | Data Warehouse | PostgreSQL compatible | OLAP, analytics | Vertical, concurrency scaling |
| **ElastiCache** | In-memory | Redis, Memcached | Caching, session store | Horizontal |
| **Neptune** | Graph | Gremlin, SPARQL | Graph relationships | Vertical |
| **Timestream** | Time-series | Proprietary | IoT, metrics | Auto-scaling |

## Networking Services

| Service | Purpose | Layer | Use Case |
|---------|---------|-------|----------|
| **ALB** | Load balancing | Layer 7 (HTTP/HTTPS) | Web applications, microservices |
| **NLB** | Load balancing | Layer 4 (TCP/UDP) | High performance, static IP |
| **CLB** | Load balancing | Layer 4/7 | Legacy applications |
| **CloudFront** | CDN | Layer 7 | Global content delivery |
| **Route 53** | DNS | Layer 7 | Domain resolution, health checks |
| **API Gateway** | API management | Layer 7 | REST/WebSocket APIs |
| **Direct Connect** | Network connection | Layer 1/2 | Dedicated network link |
| **VPN** | Network connection | Layer 3 | Encrypted connection over internet |

## Security Services

| Service | Purpose | Scope | Key Features |
|---------|---------|-------|--------------|
| **IAM** | Identity & Access | Global | Users, roles, policies |
| **Cognito** | User authentication | Regional | User pools, identity pools |
| **KMS** | Key management | Regional | Encryption key management |
| **CloudHSM** | Hardware security | AZ | Dedicated HSM instances |
| **WAF** | Web application firewall | Global/Regional | SQL injection, XSS protection |
| **Shield** | DDoS protection | Global | Standard (free), Advanced (paid) |
| **GuardDuty** | Threat detection | Regional | ML-based anomaly detection |
| **Inspector** | Security assessment | Regional | Application security |

## Monitoring & Management

| Service | Purpose | Scope | Key Metrics |
|---------|---------|-------|-------------|
| **CloudWatch** | Monitoring | Regional | Metrics, logs, alarms |
| **X-Ray** | Application tracing | Regional | Request tracing, performance |
| **CloudTrail** | API logging | Global/Regional | Governance, compliance |
| **Config** | Configuration compliance | Regional | Resource compliance |
| **Systems Manager** | System management | Regional | Patch management, automation |
| **Trusted Advisor** | Best practices | Global | Cost, security, performance |

## Integration Services

| Service | Pattern | Message Type | Durability |
|---------|---------|--------------|------------|
| **SQS** | Queue | Point-to-point | Persistent |
| **SNS** | Pub/Sub | Fanout | Transient |
| **EventBridge** | Event bus | Event-driven | Persistent |
| **Step Functions** | Workflow | State machine | Persistent |
| **SWF** | Workflow | Task coordination | Persistent |
| **MQ** | Message broker | Enterprise messaging | Persistent |
| **AppSync** | GraphQL | Real-time data | Persistent |
| **Kinesis** | Streaming | Real-time data streams | Configurable retention |

## Cost Comparison Guidelines

### Compute Cost Factors
- **On-Demand**: Highest cost, maximum flexibility
- **Reserved Instances**: 40-75% savings, 1-3 year commitment
- **Spot Instances**: Up to 90% savings, can be interrupted
- **Savings Plans**: Flexible discount, compute usage commitment

### Storage Cost Factors
- **Frequent Access**: S3 Standard > S3 IA > S3 One Zone-IA
- **Archive**: Glacier > Deep Archive (cheaper storage, higher retrieval cost)
- **Performance**: Higher IOPS = higher cost
- **Durability**: Higher durability = higher cost

### Database Cost Factors
- **Serverless**: DynamoDB On-Demand > Aurora Serverless > Traditional
- **Reserved Capacity**: DynamoDB Reserved > RDS Reserved
- **Read Scaling**: Read replicas add cost but improve performance
- **Multi-AZ**: Doubles storage and compute costs

## Service Selection Decision Tree

### For Compute:
1. **Need full OS control?** → EC2
2. **Event-driven, < 15 minutes?** → Lambda
3. **Containerized application?** → ECS/EKS
4. **Batch processing?** → AWS Batch
5. **Simple web application?** → Lightsail

### For Storage:
1. **Object storage?** → S3 (choose class by access pattern)
2. **Block storage for EC2?** → EBS
3. **Shared file system?** → EFS
4. **High-performance file system?** → FSx

### For Database:
1. **Complex relational queries?** → RDS/Aurora
2. **Simple, fast NoSQL?** → DynamoDB
3. **Analytics/Data warehouse?** → Redshift
4. **In-memory caching?** → ElastiCache
5. **Graph database?** → Neptune

### For Load Balancing:
1. **HTTP/HTTPS with advanced routing?** → ALB
2. **High performance, static IP?** → NLB
3. **Simple TCP/UDP?** → CLB or NLB
4. **Legacy application?** → CLB