# AWS Networking Fundamentals

## VPC (Virtual Private Cloud) Basics

### What is a VPC?
A logically isolated section of AWS cloud where you can launch resources in a virtual network that you define. Think of it as your own private data center in the cloud.

### Core Components
- **CIDR Block**: IP address range for your VPC (e.g., 10.0.0.0/16)
- **Subnets**: Subdivisions of your VPC's IP range
- **Route Tables**: Rules that determine where network traffic goes
- **Internet Gateway**: Allows communication with the internet
- **NAT Gateway/Instance**: Allows outbound internet access for private subnets

### Default vs Custom VPC
| Feature | Default VPC | Custom VPC |
|---------|-------------|------------|
| **CIDR** | 172.31.0.0/16 | You choose |
| **Subnets** | One public per AZ | You create |
| **Internet Gateway** | Attached | You attach |
| **Route Table** | Routes to internet | You configure |
| **Security Group** | Default allows all | You configure |

## Subnets Deep Dive

### Public vs Private Subnets
| Type | Internet Gateway Route | Use Case | Examples |
|------|----------------------|----------|----------|
| **Public** | Yes (0.0.0.0/0 → IGW) | Internet-facing resources | Web servers, load balancers |
| **Private** | No direct route | Internal resources | Databases, application servers |

### Subnet Sizing Strategy
```
VPC: 10.0.0.0/16 (65,536 IPs)
├── Public Subnet A: 10.0.1.0/24 (256 IPs)
├── Private Subnet A: 10.0.2.0/24 (256 IPs)
├── Public Subnet B: 10.0.3.0/24 (256 IPs)
└── Private Subnet B: 10.0.4.0/24 (256 IPs)
```

### Reserved IP Addresses
In every subnet, AWS reserves 5 IP addresses:
- **10.0.1.0**: Network address
- **10.0.1.1**: VPC router
- **10.0.1.2**: DNS resolver
- **10.0.1.3**: Future use
- **10.0.1.255**: Network broadcast

## Internet Connectivity

### Internet Gateway (IGW)
- **Purpose**: Provides internet access to/from VPC
- **Scaling**: Horizontally scaled, redundant, highly available
- **Limitations**: One per VPC
- **Cost**: Free

### NAT Gateway vs NAT Instance
| Feature | NAT Gateway | NAT Instance |
|---------|-------------|--------------|
| **Management** | Fully managed | Self-managed EC2 |
| **Availability** | Highly available in AZ | Single point of failure |
| **Bandwidth** | Up to 100 Gbps | Depends on instance type |
| **Cost** | Higher | Lower (just EC2 cost) |
| **Security Groups** | Not supported | Supported |
| **Bastion Server** | Not supported | Can be configured |

### Egress-Only Internet Gateway
- **Purpose**: IPv6 outbound internet access only
- **Use Case**: Private subnets with IPv6 that need outbound access
- **Limitation**: IPv6 only

## Route Tables Fundamentals

### How Routing Works
1. Most specific route wins (longest prefix match)
2. Local routes always take precedence
3. Can't override local routes

### Route Priority Order
1. **Local routes** (VPC CIDR) - Cannot be overridden
2. **Most specific routes** (longest prefix match)
3. **Route propagation** (if enabled)
4. **Static routes** (manually added)

### Common Route Table Patterns
```
Public Subnet Route Table:
10.0.0.0/16 → Local
0.0.0.0/0 → Internet Gateway

Private Subnet Route Table:
10.0.0.0/16 → Local
0.0.0.0/0 → NAT Gateway
```

## Security Groups vs NACLs

### Security Groups (Instance-Level Firewall)
| Characteristic | Behavior |
|----------------|----------|
| **Stateful** | Return traffic automatically allowed |
| **Rules** | Allow rules only (whitelist) |
| **Scope** | Instance level |
| **Default** | Deny all inbound, allow all outbound |
| **Evaluation** | All rules evaluated |

### Network ACLs (Subnet-Level Firewall)
| Characteristic | Behavior |
|----------------|----------|
| **Stateless** | Must explicitly allow return traffic |
| **Rules** | Allow and deny rules |
| **Scope** | Subnet level |
| **Default** | Allow all traffic |
| **Evaluation** | Rules processed in order |

### Security Best Practices
```
Security Group Rules (Recommended):
- SSH (22): Only from your IP or bastion host
- HTTP (80): From anywhere (0.0.0.0/0) for web servers
- HTTPS (443): From anywhere (0.0.0.0/0) for web servers  
- Database (3306): Only from application tier security group
- Custom app ports: Only from specific security groups
```

## Load Balancing

### Application Load Balancer (ALB)
- **Layer**: 7 (HTTP/HTTPS)
- **Features**: Path-based routing, host-based routing, WebSocket
- **Target Types**: EC2, IP addresses, Lambda functions
- **Use Cases**: Microservices, container-based applications

### Network Load Balancer (NLB)
- **Layer**: 4 (TCP/UDP)
- **Features**: Ultra-high performance, static IP addresses
- **Target Types**: EC2, IP addresses, ALB
- **Use Cases**: Gaming, IoT, extreme performance needs

### Classic Load Balancer (CLB)
- **Layer**: 4 and 7
- **Features**: Basic load balancing
- **Target Types**: EC2 instances only
- **Status**: Legacy (use ALB/NLB for new applications)

### Load Balancer Comparison
| Feature | ALB | NLB | CLB |
|---------|-----|-----|-----|
| **Protocol** | HTTP/HTTPS | TCP/UDP/TLS | HTTP/HTTPS/TCP |
| **Performance** | High | Ultra-high | Moderate |
| **Static IP** | No | Yes | No |
| **WebSocket** | Yes | Yes | No |
| **Path Routing** | Yes | No | No |
| **Cost** | Moderate | Higher | Lower |

## VPC Connectivity Options

### VPC Peering
- **Purpose**: Connect two VPCs privately
- **Limitations**: Not transitive, no overlapping CIDR blocks
- **Cost**: Data transfer charges only
- **Use Case**: Connect VPCs in same/different regions or accounts

### Transit Gateway
- **Purpose**: Central hub to connect multiple VPCs
- **Benefits**: Transitive routing, simplified network topology
- **Scale**: Thousands of VPCs and on-premises connections
- **Cost**: Hourly charge + data processing charges

### VPC Endpoints
#### Gateway Endpoints (Free)
- **Services**: S3, DynamoDB only
- **Implementation**: Route table entries
- **Traffic**: Stays within AWS network

#### Interface Endpoints (VPC PrivateLink)
- **Services**: Most AWS services
- **Implementation**: Elastic Network Interface (ENI)
- **Cost**: Hourly charge + data processing

### Direct Connect
- **Purpose**: Dedicated network connection to AWS
- **Benefits**: Consistent performance, reduced costs, private connectivity
- **Options**: Dedicated (1/10 Gbps) or Hosted (50 Mbps to 10 Gbps)
- **Use Case**: Hybrid architectures, large data transfers

### Site-to-Site VPN
- **Purpose**: Encrypted connection over internet to AWS
- **Components**: Virtual Private Gateway, Customer Gateway
- **Redundancy**: Two VPN tunnels for high availability
- **Use Case**: Temporary or backup connectivity

## DNS and Route 53 Basics

### Route 53 Core Features
- **Domain Registration**: Register and manage domains
- **DNS Hosting**: Host DNS records for domains
- **Health Checking**: Monitor endpoint health
- **Traffic Routing**: Route users based on policies

### Routing Policies
| Policy | Description | Use Case |
|--------|-------------|----------|
| **Simple** | Single resource record | Basic DNS resolution |
| **Weighted** | Route based on assigned weights | Blue/green deployments, testing |
| **Latency-based** | Route to lowest latency region | Global applications |
| **Failover** | Route to backup when primary fails | Disaster recovery |
| **Geolocation** | Route based on user's location | Compliance, localization |
| **Geoproximity** | Route based on location with bias | Traffic engineering |
| **Multivalue** | Return multiple healthy records | Simple load balancing |

## CloudFront (Content Delivery Network)

### What is CloudFront?
Global content delivery network that caches content at edge locations worldwide for low latency delivery.

### Key Concepts
- **Origin**: Source of content (S3, ALB, EC2, custom)
- **Distribution**: Configuration for content delivery
- **Edge Locations**: Global cache locations (400+ worldwide)
- **Cache Behaviors**: Rules for how content is cached
- **TTL**: How long content stays cached

### Origin Types
| Origin Type | Use Case | Features |
|-------------|----------|----------|
| **S3 Bucket** | Static websites, files | Origin Access Control (OAC) |
| **ALB/NLB** | Dynamic content | Custom headers, SSL |
| **EC2 Instance** | Application servers | Direct connection |
| **Custom Origin** | Any HTTP server | On-premises, other clouds |

### Caching Strategies
- **Static Content**: Long TTL (days/months)
- **Dynamic Content**: Short TTL (seconds/minutes) or no cache
- **API Responses**: Cache based on query parameters
- **Personalized Content**: Use cookies/headers in cache key

## Practical Network Architectures

### Basic Web Application
```
Internet → ALB (Public Subnets) → EC2 (Private Subnets) → RDS (Private Subnets)
                ↓
            CloudFront (for static content from S3)
```

### Multi-Tier Application
```
Internet → CloudFront → ALB → Auto Scaling Group → Database
           (Static)      ↓      (App Servers)        ↓
                    WAF Filter   Private Subnets   Multi-AZ RDS
```

### Hybrid Architecture
```
On-Premises ←→ Direct Connect/VPN ←→ VPC ←→ Other AWS Services
                                    ↓
                              Private Subnets
```

## Best Practices

### VPC Design
- Plan CIDR blocks carefully (avoid overlaps)
- Use multiple AZs for high availability
- Separate public and private subnets
- Use descriptive naming conventions
- Document your network design

### Security
- Follow principle of least privilege
- Use security groups as primary defense
- NACLs for additional subnet-level protection
- Enable VPC Flow Logs for monitoring
- Regularly audit security group rules

### Cost Optimization
- Use VPC endpoints to avoid NAT Gateway costs
- Consider data transfer costs in architecture
- Right-size NAT Gateways
- Use CloudFront for global content delivery
- Monitor and optimize Route 53 queries

## Common Pitfalls

### VPC Issues
- CIDR block conflicts preventing VPC peering
- Forgetting to update route tables
- Security group rules too permissive
- Not planning for future growth
- Mixing public and private resources incorrectly

### Load Balancer Issues
- Wrong load balancer type for use case
- Health check misconfigurations
- Target group registration issues
- SSL certificate problems
- Cross-zone load balancing not enabled

### Connectivity Problems
- Missing route table entries
- Security group blocking traffic
- NACL deny rules
- NAT Gateway in wrong subnet
- DNS resolution issues