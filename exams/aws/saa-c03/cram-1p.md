# SAA-C03 One-Page Cram Sheet

## 🏗️ Well-Architected Pillars
**Operational Excellence** | **Security** | **Reliability** | **Performance** | **Cost Optimization** | **Sustainability**

## 💾 Storage Decision Tree
- **S3**: Object storage, static websites, backup, data archiving
- **EBS**: Block storage for EC2, high IOPS, snapshots
- **EFS**: Shared file storage, NFS, multi-AZ
- **FSx**: High-performance file systems (Windows, Lustre)

## 🖥️ Compute Options
- **EC2**: Virtual servers, full control, wide instance types
- **Lambda**: Serverless, event-driven, 15min max, stateless
- **ECS**: Container orchestration, Docker
- **EKS**: Managed Kubernetes
- **Fargate**: Serverless containers

## 🗄️ Database Selection
- **RDS**: OLTP, MySQL/PostgreSQL/SQL Server/Oracle, Multi-AZ
- **Aurora**: Cloud-native, faster than RDS, auto-scaling
- **DynamoDB**: NoSQL, single-digit ms, serverless, DAX for caching
- **Redshift**: Data warehouse, OLAP, columnar
- **ElastiCache**: In-memory, Redis/Memcached

## 🌐 Networking Essentials
- **VPC**: Isolated network, 10.0.0.0/16 default
- **Subnets**: Public (route to IGW), Private (route to NAT)
- **Security Groups**: Stateful, whitelist, instance-level
- **NACLs**: Stateless, subnet-level, allow/deny rules
- **Route Tables**: Direct traffic, one per subnet

## 🔒 Security Quick Hits
- **IAM**: Users, Groups, Roles, Policies (JSON)
- **Principle of Least Privilege**: Minimum required permissions
- **MFA**: Multi-factor authentication for privileged access
- **CloudTrail**: API logging, governance, compliance
- **Config**: Resource compliance monitoring

## ⚖️ Load Balancing
- **ALB**: HTTP/HTTPS, layer 7, advanced routing
- **NLB**: TCP/UDP, layer 4, ultra-high performance
- **CLB**: Legacy, layer 4/7, simple use cases
- **Target Groups**: Health checks, routing targets

## 📈 Auto Scaling
- **Scaling Policies**: Target tracking, step, simple
- **Metrics**: CPU, memory, custom CloudWatch metrics
- **Cooldown**: Prevent rapid scaling events
- **Health Checks**: EC2 + ELB health checks

## 🚀 High Availability Patterns
- **Multi-AZ**: Different availability zones in same region
- **Multi-Region**: Geographic distribution, latency reduction
- **Auto Scaling**: Horizontal scaling for resilience
- **Load Balancing**: Distribute traffic across healthy instances
- **Database**: RDS Multi-AZ, Aurora replicas

## 💰 Cost Optimization
- **Right Sizing**: Match instance size to workload
- **Reserved Instances**: 1-3 year commitment, up to 75% savings
- **Spot Instances**: Unused capacity, up to 90% discount
- **S3 Storage Classes**: IA, One Zone-IA, Glacier for archival
- **CloudWatch**: Monitor and optimize resource usage

## 🎯 Exam Keywords & Services
**High Availability** → Multi-AZ, ELB, Auto Scaling
**Scalability** → Auto Scaling Groups, ELB, CloudFront
**Security** → IAM, VPC, Security Groups, encryption
**Cost** → Reserved Instances, Spot, right-sizing
**Performance** → CloudFront, ElastiCache, read replicas
**Serverless** → Lambda, API Gateway, DynamoDB, S3
**Monitoring** → CloudWatch, CloudTrail, Config
**Integration** → SQS, SNS, SWF, Step Functions