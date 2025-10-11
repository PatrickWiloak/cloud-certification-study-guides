# AWS Networking Services

## Amazon VPC (Virtual Private Cloud)

### Overview
Amazon VPC lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment.

### Key Concepts

#### VPC Basics
- **CIDR Block**: IP address range for the VPC (e.g., 10.0.0.0/16)
- **Region-Specific**: VPC spans all AZs in a Region
- **Default VPC**: Pre-configured VPC in each Region
- **Custom VPC**: User-created VPC with custom configuration
- **Tenancy**: Default (shared hardware) or Dedicated (dedicated hardware)

#### Subnets
- **Definition**: Subdivision of VPC IP address range
- **Availability Zone**: Each subnet in one specific AZ
- **CIDR Block**: Subset of VPC CIDR (e.g., 10.0.1.0/24)
- **Types**:
  - **Public Subnet**: Has route to Internet Gateway
  - **Private Subnet**: No direct internet access
  - **Reserved IPs**: AWS reserves 5 IPs per subnet

### Connectivity Components

#### Internet Gateway (IGW)
- **Purpose**: Connect VPC to the internet
- **Scalable**: Horizontally scaled, redundant, highly available
- **One-to-One**: One IGW per VPC
- **Route Table**: Requires route in subnet route table
- **Public IPs**: Required for internet access

#### NAT Gateway
- **Purpose**: Allow private subnet outbound internet access
- **Managed**: Fully managed by AWS
- **Availability Zone**: Deployed in specific AZ
- **Elastic IP**: Requires Elastic IP address
- **High Availability**: Deploy one per AZ for redundancy
- **Bandwidth**: Scales to 100 Gbps

#### NAT Instance (Legacy)
- **Purpose**: EC2 instance providing NAT functionality
- **Management**: Customer-managed
- **Use Case**: Cost optimization, advanced configurations
- **Limitation**: Single point of failure, limited bandwidth

### Routing

#### Route Tables
- **Definition**: Rules determining network traffic routing
- **Main Route Table**: Default for all subnets
- **Custom Route Tables**: Associate with specific subnets
- **Routes**: Destination CIDR and target (IGW, NAT, etc.)
- **Priority**: Most specific route takes precedence

#### Example Routes
- **Local**: 10.0.0.0/16 → local (VPC communication)
- **Internet**: 0.0.0.0/0 → IGW (internet access)
- **NAT**: 0.0.0.0/0 → NAT Gateway (private subnet internet)

### Security

#### Security Groups
- **Stateful**: Return traffic automatically allowed
- **Instance-Level**: Attached to EC2 instances, ENIs
- **Allow Rules Only**: Can only specify allow rules
- **Default**: Deny all inbound, allow all outbound
- **Evaluation**: All rules evaluated before decision
- **Changes**: Applied immediately

#### Network ACLs (NACLs)
- **Stateless**: Must explicitly allow return traffic
- **Subnet-Level**: Applied at subnet boundary
- **Allow and Deny**: Can specify both allow and deny rules
- **Default**: Allow all inbound and outbound
- **Order**: Rules processed in number order
- **Rule Numbers**: 1-32766 (lower numbers first)

#### Security Groups vs NACLs

| Feature | Security Groups | Network ACLs |
|---------|----------------|--------------|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and Deny |
| Evaluation | All rules | Sequential order |
| Applied to | Instances/ENIs | All subnet traffic |

### VPC Connectivity

#### VPC Peering
- **Purpose**: Connect two VPCs privately
- **Routing**: Route traffic using private IPs
- **Transitive**: Not transitive (no routing through peer)
- **Cross-Account**: Can peer across AWS accounts
- **Cross-Region**: Supports cross-region peering
- **CIDR**: Cannot have overlapping CIDR blocks

#### VPC Endpoints
- **Purpose**: Private connection to AWS services
- **Traffic**: Doesn't require IGW, NAT, VPN, or Direct Connect
- **Types**:
  - **Gateway Endpoints**: S3, DynamoDB (free)
  - **Interface Endpoints**: Many other services (hourly + data charges)
- **Security**: Use security groups and endpoint policies

#### AWS PrivateLink
- **Purpose**: Private connectivity to services in other VPCs
- **Technology**: Uses Interface VPC Endpoints
- **Security**: Traffic doesn't traverse internet
- **Use Case**: SaaS applications, shared services

### Hybrid Connectivity

#### AWS Site-to-Site VPN
- **Purpose**: Encrypted connection over the internet
- **Components**:
  - **Virtual Private Gateway (VGW)**: AWS side
  - **Customer Gateway (CGW)**: Customer side
- **Redundancy**: Two VPN tunnels for high availability
- **IPsec**: Uses IPsec protocol
- **Quick Setup**: Minutes to set up

#### AWS Direct Connect
- **Purpose**: Dedicated network connection to AWS
- **Bandwidth**: 1 Gbps or 10 Gbps (hosted: 50 Mbps to 10 Gbps)
- **Latency**: Lower and more consistent than internet
- **Security**: Private connection, doesn't traverse internet
- **Setup Time**: Weeks to months
- **Use Cases**: Large data transfers, hybrid cloud, compliance

#### AWS Transit Gateway
- **Purpose**: Central hub to connect VPCs and on-premises networks
- **Scalability**: Connect thousands of VPCs
- **Simplification**: Reduces complex peering relationships
- **Routing**: Central routing configuration
- **Attachments**: VPCs, VPN, Direct Connect

## Amazon Route 53

### Overview
Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. It connects user requests to infrastructure running in AWS and outside of AWS.

### Key Features
- **Domain Registration**: Register domain names
- **DNS Routing**: Route internet traffic to resources
- **Health Checking**: Monitor resource health
- **100% SLA**: Only AWS service with 100% availability SLA
- **Global Service**: Operates at AWS edge locations worldwide

### DNS Record Types
- **A Record**: Maps domain to IPv4 address
- **AAAA Record**: Maps domain to IPv6 address
- **CNAME**: Alias for another domain (can't use for root)
- **Alias**: AWS-specific, maps to AWS resources
- **MX**: Mail exchange servers
- **TXT**: Text records (SPF, DKIM, etc.)
- **NS**: Name server records
- **SOA**: Start of authority record

### Routing Policies

#### Simple Routing
- **Use Case**: Single resource for domain
- **Example**: example.com → single web server
- **No Health Checks**: Doesn't support health checks
- **Multiple IPs**: Can return multiple values (random selection)

#### Weighted Routing
- **Use Case**: Distribute traffic by weight
- **Weight**: 0-255 (relative to total)
- **Example**: 70% to new version, 30% to old version
- **A/B Testing**: Test new features with small percentage

#### Latency Routing
- **Use Case**: Route to lowest latency resource
- **Measurement**: Based on historical latency data
- **Example**: EU users → eu-west-1, US users → us-east-1
- **Global Applications**: Improve user experience globally

#### Failover Routing
- **Use Case**: Active-passive failover
- **Primary**: Route to primary resource
- **Secondary**: Failover to secondary if primary unhealthy
- **Health Checks**: Requires health checks on primary
- **Disaster Recovery**: Simple DR configuration

#### Geolocation Routing
- **Use Case**: Route based on user location
- **Granularity**: Continent, country, or state (US)
- **Default**: Specify default location for unmatched
- **Use Cases**: Content localization, regulatory compliance

#### Geoproximity Routing
- **Use Case**: Route based on geography with bias
- **Bias**: Shift traffic between regions (-99 to +99)
- **Resources**: Can specify latitude/longitude
- **Traffic Flow**: Use Route 53 Traffic Flow for visualization

#### Multi-Value Answer Routing
- **Use Case**: Return multiple values/IP addresses
- **Health Checks**: Only return healthy resources
- **Random**: Client chooses from returned values
- **Not LB**: Simple multi-value response, not load balancing

### Health Checks
- **Endpoint Monitoring**: Monitor endpoint health (IP or domain)
- **CloudWatch Alarm**: Monitor CloudWatch alarms
- **Calculated**: Combine multiple health checks
- **Frequency**: 30-second or 10-second intervals
- **Threshold**: Number of health checkers that must pass
- **Integration**: Integrated with routing policies

### Use Cases
- **Domain Registration**: Register and manage domains
- **DNS Hosting**: Host DNS for domains
- **Traffic Management**: Route users to optimal endpoints
- **Service Discovery**: Internal DNS for microservices
- **Hybrid DNS**: Integrate with on-premises DNS

## Elastic Load Balancing (ELB)

### Overview
Elastic Load Balancing automatically distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses, in one or more Availability Zones.

### Types of Load Balancers

#### Application Load Balancer (ALB)
- **Layer**: Layer 7 (Application layer - HTTP/HTTPS)
- **Routing**: Content-based routing
- **Features**:
  - Host-based routing (host header)
  - Path-based routing (URL path)
  - HTTP header routing
  - Query string parameter routing
- **Targets**: EC2 instances, IP addresses, Lambda functions, containers
- **WebSockets**: Supports WebSocket and HTTP/2
- **Use Cases**: Microservices, container-based applications

#### Network Load Balancer (NLB)
- **Layer**: Layer 4 (Transport layer - TCP/UDP)
- **Performance**: Ultra-high performance, low latency
- **Throughput**: Millions of requests per second
- **Static IP**: Static IP addresses per AZ
- **Elastic IP**: Can assign Elastic IPs
- **Use Cases**: Extreme performance, static IPs, TCP/UDP traffic
- **Preservation**: Preserves source IP address

#### Gateway Load Balancer (GWLB)
- **Purpose**: Deploy, scale, manage third-party virtual appliances
- **Protocol**: Layer 3 Gateway + Layer 4 Load Balancing
- **Use Cases**: Firewalls, intrusion detection/prevention, deep packet inspection
- **Target**: Virtual appliances in VPC
- **GENEVE**: Uses GENEVE protocol (port 6081)

#### Classic Load Balancer (CLB) - Legacy
- **Support**: Layer 4 and Layer 7
- **Status**: Previous generation
- **Recommendation**: Use ALB or NLB instead
- **Features**: Basic load balancing

### Key Features

#### Cross-Zone Load Balancing
- **Definition**: Distribute traffic evenly across all registered targets in all enabled AZs
- **ALB**: Always enabled, no charges
- **NLB/GWLB**: Disabled by default, charges apply when enabled
- **Benefit**: Even distribution despite uneven target distribution

#### Connection Draining/Deregistration Delay
- **Purpose**: Complete in-flight requests before deregistering
- **Duration**: 1-3600 seconds (default 300)
- **Status**: Target enters "draining" state
- **New Requests**: No new requests sent to draining target

#### SSL/TLS Termination
- **Offloading**: Terminate SSL/TLS at load balancer
- **Certificate**: Use AWS Certificate Manager (ACM)
- **Backend**: Unencrypted traffic to targets (optional)
- **SNI**: Server Name Indication for multiple certificates

#### Health Checks
- **Protocol**: HTTP, HTTPS, TCP, or gRPC
- **Path**: Specify health check path
- **Interval**: Check frequency (5-300 seconds)
- **Threshold**: Healthy/Unhealthy threshold count
- **Response**: Expected response code(s)

### Sticky Sessions (Session Affinity)
- **Definition**: Route requests from same client to same target
- **ALB**: Application-based or duration-based cookies
- **CLB/NLB**: Duration-based cookies
- **Use Case**: Maintain user session state
- **Duration**: 1 second to 7 days

### Use Cases
- **High Availability**: Distribute traffic across AZs
- **Auto Scaling**: Integrate with Auto Scaling groups
- **Microservices**: Content-based routing (ALB)
- **Gaming**: Low latency, high throughput (NLB)
- **Security Appliances**: Deploy virtual appliances (GWLB)

## Amazon CloudFront

### Overview
Amazon CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds.

### Key Concepts

#### Distributions
- **Web Distribution**: For websites, web applications
- **RTMP Distribution**: For media streaming (deprecated)
- **Origin**: Source of content (S3, ELB, EC2, custom origin)
- **Behaviors**: Rules for handling requests

#### Edge Locations
- **Global Network**: 600+ edge locations worldwide
- **Caching**: Cache content closer to users
- **Low Latency**: Reduce latency by serving from nearby location
- **Regional Edge Cache**: Larger cache between edge and origin

### Key Features

#### Caching
- **TTL**: Time to Live for cached content
- **Cache Behaviors**: Different settings for different paths
- **Query Strings**: Cache based on query parameters
- **Headers**: Cache based on headers
- **Cookies**: Cache based on cookies
- **Invalidation**: Remove content from cache before TTL

#### Security
- **HTTPS**: Require HTTPS for content delivery
- **Certificate**: Use ACM certificates
- **Signed URLs**: Restrict access with signed URLs
- **Signed Cookies**: Restrict access with signed cookies
- **OAI**: Origin Access Identity for S3 origins
- **Field-Level Encryption**: Encrypt sensitive data

#### Origin Configuration
- **Primary Origin**: Main content source
- **Origin Groups**: Failover to secondary origin
- **Custom Headers**: Add headers to origin requests
- **Origin Shield**: Additional caching layer

#### Lambda@Edge
- **Purpose**: Run code at edge locations
- **Events**: Viewer request/response, origin request/response
- **Use Cases**: A/B testing, authentication, SEO, bot mitigation
- **Languages**: Node.js, Python

### Use Cases
- **Static Website**: Deliver static content globally
- **Video Streaming**: Stream video content with low latency
- **API Acceleration**: Speed up API calls
- **Software Distribution**: Distribute software updates
- **Dynamic Content**: Accelerate dynamic content delivery

### CloudFront vs S3 Cross-Region Replication

| Feature | CloudFront | S3 CRR |
|---------|-----------|--------|
| Type | CDN caching | Replication |
| Scope | Global edge network | Specific regions |
| Content | Static and dynamic | Objects only |
| Updates | TTL-based | Near real-time |
| Use Case | Content delivery | Compliance, low latency |

## Amazon API Gateway

### Overview
Amazon API Gateway is a fully managed service that makes it easy to create, publish, maintain, monitor, and secure APIs at any scale.

### API Types

#### REST API
- **Architecture**: RESTful APIs
- **Features**: Full feature set, caching, request/response transformation
- **Protocols**: HTTP/HTTPS
- **Pricing**: Pay per request

#### HTTP API
- **Architecture**: RESTful APIs
- **Features**: Streamlined, lower cost
- **Performance**: Lower latency than REST API
- **Use Case**: Simple HTTP backends, serverless backends
- **Pricing**: Up to 71% cheaper than REST API

#### WebSocket API
- **Architecture**: Two-way communication
- **Use Cases**: Chat applications, real-time dashboards, streaming
- **Connections**: Persistent connections
- **Routes**: Route messages based on content

### Key Features

#### Integration Types
- **Lambda**: Invoke Lambda functions
- **HTTP**: HTTP endpoints (public or private)
- **AWS Services**: Direct integration with AWS services
- **Mock**: Return mock responses
- **VPC Link**: Private integration with VPC resources

#### Security
- **IAM**: AWS IAM authentication
- **Cognito**: Amazon Cognito user pools
- **Lambda Authorizer**: Custom authorization logic
- **API Keys**: Simple API key authentication
- **Resource Policies**: Control access to API

#### Performance
- **Caching**: Cache responses to reduce backend load
- **Throttling**: Rate limiting and burst limiting
- **Request Validation**: Validate requests before backend
- **Response Compression**: Compress responses automatically

#### Deployment
- **Stages**: Different environments (dev, test, prod)
- **Canary**: Canary deployments for gradual rollout
- **Versions**: API versioning support
- **Documentation**: Generate API documentation

### Use Cases
- **Serverless APIs**: Backend for serverless applications
- **Microservices**: API for microservices architecture
- **Mobile Backend**: Backend for mobile applications
- **Partner Integration**: Expose APIs to partners
- **Internal APIs**: APIs for internal services

## AWS Global Accelerator

### Overview
AWS Global Accelerator is a networking service that improves the availability and performance of applications with local or global users by using the AWS global network infrastructure.

### Key Features
- **Anycast IP**: Static IP addresses as fixed entry point
- **Global Network**: Route traffic over AWS global network
- **Health Checks**: Automatic health checking and failover
- **Performance**: Improve application performance up to 60%
- **DDoS Protection**: Built-in AWS Shield protection

### How It Works
1. **Static IPs**: Get two static anycast IP addresses
2. **Edge Locations**: Users connect to nearest edge location
3. **AWS Network**: Traffic routed over AWS global network
4. **Endpoints**: Route to optimal application endpoint

### Components
- **Static IP Addresses**: Two anycast IPv4 addresses
- **Accelerator**: The Global Accelerator resource
- **Listener**: Processes inbound connections
- **Endpoint Groups**: Groups of endpoints (ALB, NLB, EC2, Elastic IP)
- **Traffic Dials**: Control traffic to endpoint groups (0-100%)

### Use Cases
- **Global Applications**: Improve performance for global users
- **Failover**: Fast regional failover
- **Gaming**: Low-latency gaming experiences
- **IoT**: IoT device communication
- **VoIP**: Voice over IP applications

### Global Accelerator vs CloudFront

| Feature | Global Accelerator | CloudFront |
|---------|-------------------|-----------|
| Use Case | TCP/UDP applications | HTTP/HTTPS content |
| Caching | No caching | Caches content |
| Static IP | Provides static IPs | No static IPs |
| Protocol | Layer 3/4 | Layer 7 |
| Failover | Fast failover | Cache-based delivery |

## AWS VPN

### Types of VPN

#### Site-to-Site VPN
- **Purpose**: Connect on-premises network to AWS
- **Components**: Virtual Private Gateway, Customer Gateway
- **Redundancy**: Two tunnels for high availability
- **Encryption**: IPsec encryption
- **Bandwidth**: Up to 1.25 Gbps per tunnel

#### Client VPN
- **Purpose**: Remote user access to AWS and on-premises
- **Protocol**: OpenVPN protocol
- **Authentication**: Active Directory, mutual certificate, SSO
- **Split Tunnel**: Route only specific traffic through VPN
- **Use Case**: Remote workers, contractors

#### CloudHub
- **Purpose**: Connect multiple sites together via VPN
- **Topology**: Hub-and-spoke model
- **Communication**: Sites can communicate with each other
- **Use Case**: Multiple branch offices

## Key Networking Best Practices

1. **VPC Design**
   - Plan CIDR blocks carefully (no overlaps)
   - Use multiple AZs for high availability
   - Separate public and private subnets
   - Use VPC Flow Logs for network monitoring

2. **Security**
   - Implement defense in depth (Security Groups + NACLs)
   - Use least privilege access
   - Enable VPC Flow Logs
   - Encrypt data in transit

3. **High Availability**
   - Deploy across multiple AZs
   - Use load balancers for distribution
   - Implement health checks
   - Plan for failover scenarios

4. **Performance**
   - Use CloudFront for global content delivery
   - Implement caching strategies
   - Consider Direct Connect for large transfers
   - Use Global Accelerator for global apps

5. **Cost Optimization**
   - Use NAT Gateway efficiently (one per AZ)
   - Optimize data transfer costs
   - Use VPC endpoints for AWS service access
   - Monitor and analyze traffic patterns

## Key Takeaways

1. **VPC**: Foundation of AWS networking, provides network isolation and control
2. **Route 53**: Managed DNS service with multiple routing policies for traffic management
3. **ELB**: Three types (ALB, NLB, GWLB) for different use cases and protocols
4. **CloudFront**: Global CDN for low-latency content delivery
5. **API Gateway**: Create and manage APIs for applications and services
6. **Global Accelerator**: Improve global application performance using AWS network
7. **Hybrid Connectivity**: Multiple options (Site-to-Site VPN, Direct Connect) for connecting to on-premises
8. **Security Layers**: Multiple security controls (Security Groups, NACLs, WAF) for comprehensive protection
