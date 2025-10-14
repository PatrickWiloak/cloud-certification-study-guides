# Amazon EC2 Fundamentals for Solutions Architects

**[📖 EC2 User Guide](https://docs.aws.amazon.com/ec2/index.html)** - Complete EC2 documentation and best practices

## Instance Types and Selection

**[📖 EC2 Instance Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)** - Detailed guide to choosing the right instance type for your workload

### General Purpose Instances

#### T4g, T3, T3a, T2 - Burstable Performance
- **Use Cases**: Web servers, small databases, development environments, code repositories
- **Performance Model**: Baseline performance with ability to burst
- **CPU Credits**: Accumulate credits when below baseline, consume when bursting
- **Unlimited Mode**: T3/T4g can burst beyond credits for additional cost
- **Sizing Strategy**: Monitor CPU credit balance and utilization patterns

**[📖 Burstable Performance Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances.html)** - Understanding CPU credits and burstable performance

**T4g vs T3 vs T2**:
- T4g: ARM-based Graviton2 processors, 20% better price/performance
- T3: Latest generation Intel processors, better baseline performance than T2
- T2: Previous generation, lower baseline performance

#### M6i, M5, M5a, M4 - Balanced Performance
- **Use Cases**: Web applications, microservices, enterprise applications, small/mid-size databases
- **Characteristics**: Balanced compute, memory, and networking resources
- **vCPU Range**: 1-96 vCPUs depending on size
- **Memory Range**: 4 GiB - 384 GiB
- **Network Performance**: Up to 25 Gbps

### Compute Optimized Instances

#### C6i, C5, C5n, C4 - High Performance Computing
- **Use Cases**: CPU-intensive applications, web servers, scientific modeling, batch processing, MMO gaming
- **Characteristics**: High-performance processors, optimized for compute-intensive workloads
- **Processor Features**: Latest Intel processors with high clock speeds
- **Network Performance**: Enhanced networking capabilities

**When to Choose Compute Optimized**:
- CPU utilization consistently above 40%
- Compute-bound applications
- High-performance web servers
- Scientific and financial modeling

### Memory Optimized Instances

#### R6i, R5, R5a, R4 - Memory Optimized
- **Use Cases**: In-memory databases, real-time analytics, high-performance databases
- **Memory-to-vCPU Ratio**: Optimized for memory-intensive applications
- **Performance**: Fast processors suitable for memory-bound workloads

#### X1e, X1 - High Memory
- **Use Cases**: In-memory databases (SAP HANA), Apache Spark, Presto
- **Memory Range**: Up to 3,904 GiB of memory
- **SSD Storage**: High I/O performance with SSD-backed instance storage

#### z1d - High Frequency
- **Use Cases**: Electronic Design Automation (EDA), relational databases with high per-core licensing costs
- **Processor**: Sustained all-core frequency of up to 4.0 GHz
- **Storage**: NVMe SSD instance storage

### Storage Optimized Instances

#### I4i, I3, I3en - NVMe SSD Storage
- **Use Cases**: NoSQL databases, in-memory databases, data warehouses, search engines
- **Storage Type**: NVMe SSD instance storage
- **I/O Performance**: Very high random I/O performance
- **Network Performance**: High network performance for data-intensive applications

#### D3, D2 - Dense HDD Storage
- **Use Cases**: Distributed file systems, data processing workloads, log processing
- **Storage Type**: High density HDD storage
- **Cost**: Lower cost per GB compared to SSD options

### Accelerated Computing Instances

#### P4, P3 - GPU for Machine Learning
- **Use Cases**: Machine learning training, high-performance computing, seismic analysis
- **GPU**: NVIDIA Tesla V100 (P3) or A100 (P4) GPUs
- **Memory**: High bandwidth memory for GPU workloads
- **Networking**: 100 Gbps network performance (P4)

#### G4 - GPU for Graphics Workloads
- **Use Cases**: Game streaming, 3D visualization, video processing
- **GPU**: NVIDIA T4 GPUs
- **Applications**: Graphics workstations, media rendering

## Instance Sizing Strategy

### Right-Sizing Methodology
1. **Start Small**: Begin with smaller instances and scale up based on actual usage
2. **Monitor Metrics**: Use CloudWatch to monitor CPU, memory, network, and disk utilization
3. **Analyze Patterns**: Understand peak vs. average utilization patterns
4. **Consider Burstable**: Use T-series for variable workloads
5. **Plan for Growth**: Consider future capacity needs

### Key Metrics for Sizing
- **CPU Utilization**: Target 40-60% average utilization
- **Memory Utilization**: Monitor swap usage and available memory
- **Network Utilization**: Consider bandwidth requirements
- **Disk I/O**: Monitor IOPS and throughput requirements

## Placement Groups

**[📖 Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)** - Strategies for grouping instances to meet workload needs

### Cluster Placement Groups
- **Purpose**: Low latency, high network performance between instances
- **Use Cases**: HPC applications, tightly coupled workloads
- **Limitations**: Single AZ, limited instance types
- **Network Performance**: 10 Gbps between instances in same cluster placement group

### Partition Placement Groups
- **Purpose**: Large distributed workloads (Hadoop, Cassandra, Kafka)
- **Partitions**: Up to 7 partitions per AZ
- **Isolation**: Each partition has its own set of racks
- **Use Cases**: Large distributed and replicated workloads

### Spread Placement Groups
- **Purpose**: Critical instances that should be on separate hardware
- **Limitation**: Maximum 7 instances per AZ per group
- **Use Cases**: Small number of critical instances
- **Isolation**: Each instance on separate underlying hardware

## Instance Purchasing Options

**[📖 EC2 Pricing](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html)** - Understanding different EC2 pricing models

### On-Demand Instances
- **Billing**: Pay by the hour or second (minimum 60 seconds)
- **Use Cases**: Short-term workloads, unpredictable workloads, development/testing
- **Benefits**: No upfront payment, no long-term commitment
- **Best For**: Applications with unpredictable workloads that cannot be interrupted

### Reserved Instances
- **Standard Reserved Instances**:
  - Discount: Up to 75% compared to On-Demand
  - Flexibility: Can modify AZ, instance size (within same family), networking type
  - Term: 1 or 3 years
  - Payment: All Upfront, Partial Upfront, No Upfront

- **Convertible Reserved Instances**:
  - Discount: Up to 54% compared to On-Demand
  - Flexibility: Can change instance family, OS, tenancy, payment option
  - Term: 3 years only
  - Use Case: Workloads with changing requirements

- **Scheduled Reserved Instances**:
  - Use Case: Predictable recurring schedules
  - Commitment: Specific time windows (daily, weekly, monthly)
  - Discount: Compared to On-Demand for scheduled usage

### Spot Instances
- **Pricing**: Up to 90% discount compared to On-Demand
- **Availability**: Based on spare EC2 capacity
- **Interruption**: 2-minute warning before termination
- **Best Practices**:
  - Use Spot Fleet for diversification
  - Implement checkpointing for long-running jobs
  - Use multiple instance types and AZs
  - Set appropriate maximum price

**[📖 Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)** - Maximize savings with EC2 Spot Instances

**Ideal Workloads for Spot**:
- Batch processing jobs
- Data analysis and processing
- Image and media rendering
- Scientific computing
- CI/CD and testing

### Dedicated Instances
- **Isolation**: Run on single-tenant hardware
- **Use Cases**: Compliance requirements, licensing restrictions
- **Billing**: Additional charges apply
- **Placement**: May share hardware with other Dedicated Instances from same account

### Dedicated Hosts
- **Control**: Physical EC2 server dedicated for your use
- **Visibility**: Socket and core visibility for licensing
- **Use Cases**: Server-bound software licenses, compliance requirements
- **Management**: More control over instance placement

## Auto Scaling Strategies

**[📖 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)** - Maintain application availability with EC2 Auto Scaling

### Auto Scaling Groups (ASG)

#### Components
- **Launch Template/Configuration**: Defines instance specifications
- **Auto Scaling Group**: Manages collection of instances
- **Scaling Policies**: Rules for scaling actions
- **Health Checks**: EC2 and/or ELB health checks

#### Scaling Policies

**Target Tracking Scaling**:
- **Metric**: Maintain target value for specific metric
- **Examples**: Average CPU utilization, request count per target
- **Behavior**: ASG automatically adjusts capacity to maintain target
- **Best Practice**: Use for most common scaling scenarios

**Step Scaling**:
- **Triggers**: CloudWatch alarms trigger scaling actions
- **Steps**: Different scaling amounts based on alarm breach size
- **Flexibility**: More control over scaling behavior
- **Use Case**: When you need fine-tuned control over scaling

**Simple Scaling**:
- **Action**: Single scaling action per alarm
- **Cooldown**: Wait period before next scaling action
- **Legacy**: Older scaling type, step scaling preferred

**Predictive Scaling**:
- **Machine Learning**: Uses ML to predict future traffic
- **Proactive**: Scales before traffic increases
- **Best Practice**: Combine with reactive scaling policies

#### Health Checks
- **EC2 Health Checks**: Instance status and system status
- **ELB Health Checks**: Application-level health verification
- **Custom Health Checks**: Via CloudWatch or custom scripts
- **Grace Period**: Time to allow instance to warm up

### Best Practices for Auto Scaling

#### Design Principles
1. **Stateless Applications**: Design applications to be stateless
2. **Health Checks**: Implement meaningful health checks
3. **Graceful Shutdown**: Handle termination signals properly
4. **Multiple AZs**: Distribute instances across multiple AZs
5. **Right-Size**: Use appropriate instance types for workload

#### Monitoring and Optimization
1. **CloudWatch Metrics**: Monitor ASG and instance metrics
2. **Scaling History**: Review scaling activities and effectiveness
3. **Cost Optimization**: Balance performance and cost
4. **Testing**: Test scaling policies under various load conditions

## Security Best Practices

**[📖 EC2 Security](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security.html)** - Best practices for securing your EC2 instances

### Security Groups
- **Stateful**: Return traffic automatically allowed
- **Default Behavior**: Deny all inbound, allow all outbound
- **Best Practices**:
  - Principle of least privilege
  - Use descriptive names and descriptions
  - Regular review and cleanup
  - Use security group references instead of IP addresses

**[📖 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)** - Control traffic to your instances with security groups

### IAM Roles for EC2
- **Instance Profiles**: Attach IAM roles to EC2 instances
- **Benefits**: No need to store credentials on instances
- **Automatic Rotation**: Credentials automatically rotated
- **Best Practice**: Always use IAM roles instead of access keys

**[📖 IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)** - Grant AWS permissions to applications running on EC2

### Key Pairs
- **SSH Access**: Use key pairs for Linux instances
- **RDP Access**: Use key pairs or set passwords for Windows
- **Best Practices**:
  - Use separate key pairs for different environments
  - Regularly rotate key pairs
  - Store private keys securely
  - Use Systems Manager Session Manager when possible

### Network Security
- **VPC Placement**: Launch instances in VPC with proper subnet design
- **Network ACLs**: Additional layer of network security
- **Monitoring**: Enable VPC Flow Logs for network monitoring
- **Encryption**: Use encryption in transit and at rest

## Performance Optimization

**[📖 Enhanced Networking](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html)** - Enable high network performance for supported instance types

### Instance Optimization
- **Enhanced Networking**: Enable for supported instance types
- **SR-IOV**: Single Root I/O Virtualization for better network performance
- **Placement Groups**: Use appropriate placement group type
- **Instance Store**: Use for temporary high-performance storage

### Monitoring and Tuning
- **CloudWatch Metrics**: Monitor detailed instance metrics
- **Custom Metrics**: Send application-specific metrics
- **Performance Baselines**: Establish performance baselines
- **Load Testing**: Regular load testing to validate performance

### Storage Performance
- **EBS Optimization**: Enable EBS optimization for supported instances
- **Instance Store**: Use for temporary high-IOPS requirements
- **Network Bandwidth**: Consider network bandwidth for storage throughput

## Cost Optimization Strategies

### Instance Selection
- **Right-Sizing**: Regular review and adjustment of instance sizes
- **Instance Families**: Choose appropriate instance family for workload
- **Generation**: Use latest generation instances for better price/performance

### Purchasing Options
- **Reserved Instances**: For predictable workloads
- **Spot Instances**: For fault-tolerant workloads
- **Savings Plans**: For flexible workload patterns

### Operational Efficiency
- **Auto Scaling**: Automatically adjust capacity based on demand
- **Scheduling**: Stop/start instances for dev/test environments
- **Resource Tagging**: Implement comprehensive tagging strategy
- **Cost Monitoring**: Regular review of cost and usage reports

## Common Architecture Patterns

### Web Application Tier
- **Load Balancer**: Application Load Balancer for HTTP/HTTPS traffic
- **Auto Scaling**: Auto Scaling Group across multiple AZs
- **Instance Type**: General purpose (M5) or compute optimized (C5)
- **Health Checks**: Application-level health checks

### Application Tier
- **Private Subnets**: Deploy in private subnets for security
- **Auto Scaling**: Based on application metrics
- **Instance Type**: Based on application requirements (CPU, memory)
- **Service Discovery**: Use ELB or service mesh for service discovery

### Background Processing
- **Spot Instances**: Use Spot Instances for cost optimization
- **Queue-based**: Use SQS for decoupling and fault tolerance
- **Auto Scaling**: Scale based on queue depth
- **Instance Type**: Compute or memory optimized based on workload

## Key Takeaways

1. **Instance Selection**: Choose instance types based on workload characteristics (CPU, memory, storage, network)
2. **Purchasing Strategy**: Mix of On-Demand, Reserved, and Spot instances for cost optimization
3. **Auto Scaling**: Implement Auto Scaling for availability and cost optimization
4. **Security**: Use IAM roles, security groups, and VPC for comprehensive security
5. **Performance**: Monitor and optimize based on actual usage patterns
6. **Cost Management**: Regular review and optimization of instance usage and costs
7. **Architecture**: Design for fault tolerance using multiple AZs and Auto Scaling
8. **Monitoring**: Comprehensive monitoring for performance and cost optimization