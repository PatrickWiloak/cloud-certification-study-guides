# AWS Networking and VPC - SAA-C03

## Amazon VPC (Virtual Private Cloud)

**[📖 Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)** - Isolated cloud resources in a virtual network

### VPC Basics

**[📖 How VPC Works](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html)** - Understanding VPC components and networking
- **CIDR Block**: IPv4 range (e.g., 10.0.0.0/16)
- **Subnets**: Subdivision of VPC CIDR into AZs
- **Route Tables**: Control traffic routing
- **Internet Gateway (IGW)**: Connect VPC to internet
- **NAT Gateway/Instance**: Private subnet internet access

### Subnet Types
**Public Subnet**:
- Route table with route to IGW (0.0.0.0/0 → IGW)
- Auto-assign public IPv4 address
- Resources accessible from internet
- Use Cases: Web servers, bastion hosts

**Private Subnet**:
- No direct route to IGW
- Use NAT for outbound internet access
- Not directly accessible from internet
- Use Cases: Databases, application servers

### NAT Solutions

**[📖 NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)** - Enable internet access for private subnets

**NAT Gateway** (Recommended):
- AWS-managed, highly available within AZ
- 5 Gbps bandwidth, scales to 45 Gbps
- Pay per hour + data processed
- Deploy in public subnet
- **Multi-AZ**: Deploy one per AZ for high availability

**NAT Instance**:
- EC2 instance with NAT AMI
- Must disable source/destination check
- Manual management required
- Lower cost but less reliable
- Use Case: Cost optimization for small workloads

## Security

**[📖 VPC Security](https://docs.aws.amazon.com/vpc/latest/userguide/security.html)** - Secure your VPC with security groups and NACLs

### Security Groups (SGs)
- **Stateful**: Return traffic automatically allowed
- **Instance-level**: Applied to ENIs
- **Allow Rules Only**: Cannot deny traffic
- **Default**: Deny all inbound, allow all outbound
- **Best Practice**: Least privilege, reference other SGs

**[📖 Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)** - Control inbound and outbound traffic

### Network ACLs (NACLs)
- **Stateless**: Must allow both request and response
- **Subnet-level**: Applied to all resources in subnet
- **Allow and Deny Rules**: Explicit deny supported
- **Rule Numbers**: 1-32766, processed in order
- **Default**: Allow all inbound and outbound
- **Custom**: Deny all by default

### Security Group vs NACL
| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance (ENI) | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and Deny |
| Rule Processing | All rules evaluated | Rules in number order |
| Application | Explicit association | Automatic for subnet |

## VPC Connectivity

**[📖 VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)** - Connect VPCs in a private network

### VPC Peering
- Connect two VPCs privately
- Must have non-overlapping CIDR blocks
- **Not Transitive**: Must create direct peering
- Same or cross-region, same or cross-account
- Use Cases: Shared services, multi-account architectures

### Transit Gateway
- Hub-and-spoke connectivity
- Connect thousands of VPCs and on-premises networks
- **Transitive Routing**: VPC A ↔ TGW ↔ VPC B
- Regional resource, can peer cross-region
- Use Cases: Centralized routing, simplified network management

**[📖 Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)** - Connect VPCs and on-premises networks through a central hub

### VPC Endpoints

**[📖 VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)** - Privately connect to AWS services without internet gateway

**Interface Endpoints (PrivateLink)**:
- ENI with private IP in subnet
- Powered by AWS PrivateLink
- Supports most AWS services
- $0.01/hour per AZ + data processed

**Gateway Endpoints** (Free):
- Route table target
- Only S3 and DynamoDB
- No additional charge
- Preferred for S3/DynamoDB access

### AWS PrivateLink
- Expose services to other VPCs privately
- No VPC peering, IGW, NAT required
- Scalable, secure
- Use Case: SaaS applications, shared services

## Hybrid Connectivity

**[📖 Site-to-Site VPN](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)** - Secure connection between on-premises and AWS

### AWS Site-to-Site VPN
- Encrypted IPsec tunnels over internet
- Virtual Private Gateway (VGW) on AWS side
- Customer Gateway (CGW) on customer side
- **2 Tunnels**: Redundancy across AZs
- Up to 1.25 Gbps per tunnel
- Use Case: Quick hybrid connectivity, backup for Direct Connect

### AWS Direct Connect (DX)
- Dedicated private connection
- 1 Gbps or 10 Gbps
- Lower latency, consistent network performance
- Private and public virtual interfaces
- **Setup Time**: Weeks to months
- Use Cases: Large data transfers, hybrid architectures, regulatory requirements

**[📖 AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html)** - Dedicated network connection to AWS

### Direct Connect + VPN
- VPN over Direct Connect for encryption
- Combines benefits of both
- Use Case: Compliance requiring encryption

### AWS VPN CloudHub
- Hub-and-spoke VPN model
- Connect multiple sites together
- Uses existing VGW with multiple CGWs
- Low cost, easy to manage
- Use Case: Multiple branch offices

## Route 53

**[📖 Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)** - Scalable DNS and domain name registration

### Routing Policies

**[📖 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)** - Choose how Route 53 responds to DNS queries

**Simple**: Single resource, no health checks
**Weighted**: Distribute traffic by percentage (A/B testing, gradual migration)
**Latency**: Route to lowest latency region
**Failover**: Active-passive failover with health checks
**Geolocation**: Route based on user location
**Geoproximity**: Route based on geographic distance with bias
**Multi-value**: Return multiple values with health checks

### Health Checks
- Monitor endpoint health
- CloudWatch alarms integration
- Calculated health checks (combine multiple)
- Use Cases: Failover, multi-region high availability

### Advanced Features
- **Alias Records**: Map to AWS resources (ELB, CloudFront, S3), no charge
- **Private Hosted Zones**: DNS for VPCs
- **DNSSEC**: Domain authentication
- **Traffic Flow**: Visual editor for complex routing

## CloudFront

### Distribution Types
- **Web Distribution**: Websites, HTTP/HTTPS content
- **RTMP Distribution**: Media streaming (deprecated)

### Origin Types
- S3 bucket
- Custom origin (HTTP server, ALB, etc.)
- MediaStore/MediaPackage containers

### Key Features
- **Edge Locations**: 400+ global PoPs
- **Regional Edge Caches**: Between edge and origin
- **TTL**: Control cache duration (default 24 hours)
- **Cache Invalidation**: Remove objects before TTL expires
- **Signed URLs/Cookies**: Restrict content access

### Security
- **OAI (Origin Access Identity)**: Restrict S3 access to CloudFront only
- **Geo Restriction**: Whitelist/blacklist countries
- **SSL/TLS**: HTTPS, custom SSL certificates
- **AWS WAF Integration**: Web application firewall
- **Field-Level Encryption**: Additional encryption for sensitive data

### Performance
- HTTP/2 and HTTP/3 support
- Compression (gzip, brotli)
- Lambda@Edge: Run code at edge locations
- CloudFront Functions: Lightweight, sub-ms execution

## Load Balancers

### Application Load Balancer (ALB)
- **Layer 7**: HTTP/HTTPS
- **Path-based Routing**: /api → target group 1, /images → target group 2
- **Host-based Routing**: api.example.com vs www.example.com
- **Query String/Header Routing**: ?platform=mobile
- **WebSockets**: Bidirectional communication
- **HTTP/2**: Multiple requests over single connection
- **Fixed Hostname**: DNS name
- **Target Types**: Instances, IPs, Lambda functions
- **Use Cases**: Microservices, containerized apps

### Network Load Balancer (NLB)
- **Layer 4**: TCP/UDP/TLS
- **Ultra Performance**: Millions of requests/second, ultra-low latency
- **Static IP**: One static IP per AZ, Elastic IP support
- **Preserve Source IP**: Client IP visible to target
- **TLS Termination**: Offload SSL/TLS decryption
- **Use Cases**: Extreme performance, non-HTTP protocols, static IP requirement

### Gateway Load Balancer (GWLB)
- **Layer 3**: IP packets
- **Transparent Proxy**: Inline traffic inspection
- **GENEVE Protocol**: Encapsulation on port 6081
- **Use Cases**: Firewalls, IDS/IPS, deep packet inspection

### Classic Load Balancer (CLB)
- **Layer 4 & 7**: Legacy
- **Deprecated**: Use ALB or NLB for new applications
- **Fixed hostname**
- **Basic health checks**

### Cross-Zone Load Balancing
- Distribute traffic evenly across all targets in all AZs
- **ALB**: Enabled by default (free)
- **NLB**: Disabled by default (charged if enabled)
- **CLB**: Disabled by default (free if enabled)

### Sticky Sessions (Session Affinity)
- Route user to same target
- Cookie-based (application cookie or load balancer cookie)
- **ALB**: Application or duration-based cookies
- **NLB**: Not supported (use ALB or implement in app)
- **Use Case**: Stateful applications

## Global Accelerator

### Features
- **Anycast IP**: 2 static IPs for your application
- **AWS Global Network**: Route through AWS backbone
- **Health Checks**: Automatic failover
- **Deterministic Routing**: Consistent destination
- **Use Cases**: Gaming, IoT, VoIP

### Global Accelerator vs CloudFront
- **CloudFront**: Cacheable content (images, videos, static)
- **Global Accelerator**: Dynamic content, non-HTTP (UDP, TCP), static IP required

## Exam Tips

### Scenario Matching
- **Private connectivity to AWS services**: VPC Endpoints
- **Connect VPCs**: VPC Peering (few) or Transit Gateway (many)
- **On-premises to AWS**: Direct Connect (dedicated) or VPN (encrypted)
- **Layer 7 load balancing**: ALB
- **Ultra-low latency, static IP**: NLB
- **Global application acceleration**: Global Accelerator
- **Content delivery, caching**: CloudFront
- **Private subnet internet access**: NAT Gateway
- **Multiple VPN sites**: VPN CloudHub
- **Inline network appliances**: Gateway Load Balancer

### Security Best Practices
1. Use Security Groups for allow rules, NACLs for deny
2. Enable VPC Flow Logs for traffic analysis
3. Use VPC Endpoints to keep traffic on AWS network
4. Implement least privilege in Security Groups
5. Use CloudFront OAI for S3 bucket security
6. Enable encryption in transit (HTTPS, TLS)

### Cost Optimization
1. Use Gateway Endpoints (free) for S3/DynamoDB
2. NAT Gateway in single AZ for non-critical workloads
3. CloudFront to reduce origin load and data transfer
4. Direct Connect for large, consistent data transfers
5. ALB over multiple CLBs for consolidated routing
