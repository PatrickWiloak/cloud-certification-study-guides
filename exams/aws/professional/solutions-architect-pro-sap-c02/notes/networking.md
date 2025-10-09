# Networking

## Overview

AWS networking at the professional level involves designing complex, scalable network architectures that span multiple VPCs, regions, and hybrid environments. You must understand advanced networking concepts including Transit Gateway, PrivateLink, Direct Connect, hybrid DNS resolution, network segmentation strategies, and how to optimize for performance, security, and cost while managing organizational complexity.

Professional-level scenarios require deep knowledge of routing, BGP, VPN configurations, network security, traffic flow patterns, and integration with on-premises infrastructure. You'll need to design networks that support thousands of VPCs, multiple AWS accounts, regulatory compliance requirements, and complex connectivity patterns.

## Key Concepts

### Virtual Private Cloud (VPC) Advanced Patterns

**Multi-VPC Architecture**
- Isolated VPCs per environment (dev, test, prod) for blast radius containment
- Isolated VPCs per application for security and compliance boundaries
- Shared services VPC for common resources (Active Directory, monitoring, logging)
- Egress VPC for centralized internet and on-premises connectivity
- Network security VPC for centralized firewall inspection
- CIDR planning to avoid IP conflicts (RFC 1918 ranges, /16 to /28)

**VPC Sizing and Subnet Strategy**
- Reserve IP space for future growth (AWS reserves 5 IPs per subnet)
- Multi-AZ subnet distribution for high availability
- Public subnets for internet-facing resources (NAT Gateway, ALB, bastion hosts)
- Private subnets for application and database tiers
- Dedicated subnets for specific purposes (Lambda, EKS, Transit Gateway attachments)
- Secondary CIDR blocks for expansion without re-architecting

**IP Address Management (IPAM)**
- AWS VPC IP Address Manager for centralized IPAM across AWS Organizations
- Automated CIDR allocation from IP pools
- Prevent CIDR overlaps across accounts and regions
- Track IP usage and utilization
- Integration with on-premises IPAM systems

### AWS Transit Gateway

**Hub-and-Spoke Architecture**
- Centralized connectivity hub for VPCs and on-premises networks
- Eliminates complex VPC peering mesh topologies
- Supports thousands of VPC attachments
- Multi-AZ redundancy within a region
- Transitive routing between VPCs and on-premises

**Transit Gateway Route Tables**
- Multiple route tables for network segmentation
- Route propagation from VPN and Direct Connect attachments
- Static routes for specific traffic patterns
- Blackhole routes for traffic denial
- Route table associations and propagations

**Transit Gateway Network Manager**
- Global view of on-premises and AWS networks
- Multi-region Transit Gateway peering
- SD-WAN integration
- Centralized routing and monitoring
- Events and metrics for network health

**Advanced Patterns**
- **Segmentation**: Separate route tables for production, non-production, shared services
- **Inspection Architecture**: Route traffic through centralized firewall VPC
- **Multi-Region**: Peering between Transit Gateways in different regions
- **Egress Control**: Route all internet traffic through centralized egress VPC

### AWS Direct Connect

**Connection Types**
- Dedicated Connection: 1 Gbps, 10 Gbps, 100 Gbps physical fiber from AWS
- Hosted Connection: 50 Mbps to 10 Gbps through AWS Direct Connect Partners
- Link Aggregation Groups (LAG) for bandwidth aggregation and redundancy

**Virtual Interfaces (VIFs)**
- **Private VIF**: Access VPC resources via Virtual Private Gateway or Transit Gateway
- **Public VIF**: Access AWS public services (S3, DynamoDB) without internet traversal
- **Transit VIF**: Connect to Transit Gateway for multi-VPC access

**High Availability Architectures**
- Redundant connections from separate devices in customer network
- Connections to multiple Direct Connect locations
- VPN backup for Direct Connect failures
- BFD (Bidirectional Forwarding Detection) for fast failover
- BGP configuration with AS path prepending for path preference

**Site-to-Site VPN as Backup**
- IPSec VPN over internet as failover
- Equal-cost multi-path (ECMP) for active-active VPN
- BGP routing metrics for automatic failover
- Accelerated Site-to-Site VPN using Global Accelerator

### VPC Peering

**Characteristics**
- One-to-one connection between two VPCs
- No transitive peering (A-B and B-C does not enable A-C connectivity)
- Supports inter-region peering
- Supports cross-account peering
- No bandwidth limits, low latency
- Non-overlapping CIDR blocks required

**Use Cases vs. Transit Gateway**
- VPC Peering: Simple point-to-point connectivity, minimal cost, specific VPC pairs
- Transit Gateway: Hub-and-spoke, transitive routing, centralized management, many VPCs
- Hybrid approach: Transit Gateway for main connectivity, VPC Peering for high-bandwidth specific needs

### AWS PrivateLink

**VPC Endpoints**
- **Gateway Endpoints**: S3 and DynamoDB (free, route table-based)
- **Interface Endpoints**: Powered by PrivateLink (ENI-based, cost per hour and GB)
- Private connectivity to AWS services without internet gateway or NAT
- Access AWS services from on-premises via Direct Connect or VPN

**PrivateLink for Service Providers**
- Expose services to other VPCs/accounts without VPC peering
- Network Load Balancer as target
- Endpoint services with allow-listed principals
- Private DNS names for seamless integration
- Multi-AZ redundancy

**Use Cases**
- SaaS providers exposing services to customers' VPCs
- Shared services across organizational accounts
- Third-party integrations without public internet exposure
- Microservices communication across VPCs

### Hybrid DNS Resolution

**Amazon Route 53 Resolver**
- Recursive DNS service built into every VPC
- Resolver endpoints for hybrid DNS
  - **Inbound Endpoints**: On-premises queries AWS-hosted domains
  - **Outbound Endpoints**: AWS queries on-premises domains
- Resolver rules for conditional forwarding
- Shared resolver rules across AWS Organizations

**DNS Architecture Patterns**
- Centralized DNS architecture with shared services VPC
- Route 53 Resolver rules shared via AWS RAM (Resource Access Manager)
- Conditional forwarding to on-premises DNS servers
- Private Hosted Zones for internal DNS
- Integration with Active Directory DNS

### Network Security

**Security Groups**
- Stateful firewall at instance/ENI level
- Allow rules only (implicit deny)
- Support source/destination as IP, CIDR, security group, prefix list
- Chaining security groups for micro-segmentation
- Maximum 5 security groups per ENI

**Network ACLs**
- Stateless firewall at subnet level
- Allow and deny rules
- Evaluated in numerical order
- Separate inbound and outbound rules
- Default NACL allows all traffic
- Custom NACLs deny all traffic by default

**AWS Network Firewall**
- Managed stateful firewall service
- IPS/IDS capabilities with Suricata rules
- Domain name filtering
- Centralized deployment in inspection VPC
- Multi-AZ redundancy
- Integration with AWS Firewall Manager

**AWS WAF (Web Application Firewall)**
- Layer 7 protection for CloudFront, ALB, API Gateway, AppSync
- Managed rule groups (AWS and third-party)
- Custom rules based on IP, geo, headers, body, query strings
- Rate limiting and bot control
- Centralized management with AWS Firewall Manager

### Traffic Flow and Inspection

**VPC Traffic Mirroring**
- Copy network traffic from ENI to targets for monitoring
- Targets: ENI, Network Load Balancer
- Filters for specific traffic patterns
- Use cases: IDS/IPS, troubleshooting, compliance

**Gateway Load Balancer**
- Deploy, scale, and manage third-party virtual appliances
- Transparent network gateway and load balancer
- Use cases: Firewalls, IDS/IPS, DLP, NGFW
- GENEVE protocol encapsulation
- Centralized inspection architecture

**VPC Flow Logs**
- Capture IP traffic metadata (not packet contents)
- Enabled at VPC, subnet, or ENI level
- Publish to CloudWatch Logs, S3, or Kinesis Data Firehose
- Analyze with CloudWatch Insights, Athena, or third-party tools
- Troubleshoot connectivity issues, security analysis, cost optimization

### Global Network Services

**Amazon CloudFront**
- Global CDN with 400+ edge locations
- Origin: S3, ALB, custom HTTP/HTTPS origins
- Origin failover for high availability
- Field-level encryption for sensitive data
- Lambda@Edge and CloudFront Functions for edge computing
- AWS WAF integration for DDoS protection and security

**AWS Global Accelerator**
- Anycast static IP addresses (2 IPs per accelerator)
- Traffic routed over AWS global network
- Automatic failover between regions
- Health checks and traffic dials for blue/green deployments
- Use cases: Global applications, gaming, IoT, VoIP
- Comparison to CloudFront: Global Accelerator for TCP/UDP, CloudFront for HTTP/HTTPS caching

**Amazon Route 53**
- Authoritative DNS service (hosted zones)
- Domain registration
- Health checks with failover
- Routing policies: Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multi-value
- Traffic Flow for complex routing logic
- DNSSEC for domain security
- Alias records for AWS resources (no charge for queries)

## AWS Services Reference

### Core Services

**Amazon VPC**
- Isolated virtual networks in AWS cloud
- CIDR blocks, subnets, route tables, internet gateway, NAT gateway
- VPC Flow Logs for network monitoring
- VPC sharing across accounts with AWS RAM
- Secondary CIDR blocks for expansion

**AWS Transit Gateway**
- Centralized network hub connecting VPCs and on-premises networks
- Transit Gateway attachments: VPC, VPN, Direct Connect, peering
- Route tables for traffic segmentation
- Multicast support
- Network Manager for global view

**AWS Direct Connect**
- Dedicated network connection to AWS
- 1 Gbps to 100 Gbps bandwidth
- Private and public virtual interfaces
- Link aggregation and redundancy options
- Direct Connect Gateway for multi-region access

**AWS PrivateLink**
- Private connectivity to services across VPCs and accounts
- VPC endpoints for AWS services
- Endpoint services for customer applications
- Multi-AZ redundancy with ENIs

### Supporting Services

**Amazon Route 53**
- DNS service for domain registration and routing
- Private hosted zones for VPC internal DNS
- Resolver endpoints for hybrid DNS
- Health checks and failover routing

**AWS Network Firewall**
- Managed stateful firewall
- IPS/IDS with Suricata-compatible rules
- Centralized firewall policy management
- TLS inspection

**AWS WAF**
- Web application firewall
- Protection against common web exploits
- Managed rules and custom rules
- Integration with CloudFront, ALB, API Gateway

**AWS Firewall Manager**
- Centrally manage firewall rules across accounts
- AWS Organizations integration
- Security Groups, WAF, Network Firewall, Route 53 Resolver DNS Firewall
- Compliance auditing and enforcement

**AWS Client VPN**
- Managed client-based VPN service
- Remote access to AWS and on-premises resources
- Active Directory and certificate-based authentication
- Split-tunnel or full-tunnel configurations

**AWS Site-to-Site VPN**
- IPsec VPN between on-premises and AWS
- Static or dynamic (BGP) routing
- Accelerated VPN using Global Accelerator
- Multiple customer gateway support

**Elastic Load Balancing**
- **Application Load Balancer (ALB)**: Layer 7, HTTP/HTTPS, WebSocket
- **Network Load Balancer (NLB)**: Layer 4, TCP/UDP/TLS, ultra-low latency
- **Gateway Load Balancer (GWLB)**: Layer 3, virtual appliance deployment
- Cross-zone load balancing
- Target groups with health checks

## Architecture Patterns

### Pattern 1: Hub-and-Spoke with Centralized Egress

**Use Case**
- Centralized internet egress for security and compliance
- Shared services (Active Directory, DNS, monitoring) accessible to all workloads
- Network segmentation between production and non-production
- Cost optimization through shared NAT Gateways and proxy servers

**Implementation Approach**
1. **Transit Gateway** as central hub
2. **Spoke VPCs** for workloads (application VPCs)
3. **Egress VPC** with NAT Gateways or proxy servers
4. **Shared Services VPC** for common infrastructure
5. **Transit Gateway Route Tables**:
   - Spoke route table: Route 0.0.0.0/0 to Egress VPC, specific routes to Shared Services
   - Egress route table: Routes to spoke VPCs
   - Shared Services route table: Routes to all spokes
6. **VPC Route Tables**: Default route pointing to Transit Gateway

**Pros/Cons**
- Pros: Centralized security controls, cost optimization, simplified management, audit compliance
- Cons: Egress VPC is single point of failure (mitigate with multi-AZ), latency for cross-region, Transit Gateway costs

### Pattern 2: Multi-Region with Transit Gateway Peering

**Use Case**
- Global applications with regional deployments
- Disaster recovery with cross-region connectivity
- Data replication between regions
- Low-latency inter-region communication

**Implementation Approach**
1. **Regional Transit Gateways** in each region (e.g., us-east-1, eu-west-1)
2. **Transit Gateway Peering** between regions
3. **Regional VPCs** attached to local Transit Gateway
4. **Cross-region routing** configured in Transit Gateway route tables
5. **Route 53** for DNS-based traffic routing
6. **Optional**: Direct Connect connections in each region for on-premises access

**Pros/Cons**
- Pros: Regional failure isolation, low inter-region latency over AWS backbone, simplified routing
- Cons: Higher costs (Transit Gateway in multiple regions, data transfer), increased complexity, cross-region data transfer charges

### Pattern 3: Centralized Network Security Inspection

**Use Case**
- Regulatory requirement for traffic inspection (IDS/IPS, DLP)
- Centralized firewall policy enforcement
- Threat detection and prevention
- Compliance auditing

**Implementation Approach**
1. **Inspection VPC** with AWS Network Firewall or third-party firewall appliances
2. **Gateway Load Balancer** for scaling firewall appliances
3. **Transit Gateway** with route tables directing traffic through inspection VPC
4. **Firewall Endpoints** in multiple AZs for high availability
5. **Route Configuration**:
   - East-west traffic (VPC to VPC) routed through inspection VPC
   - North-south traffic (internet-bound) routed through inspection VPC
6. **Logging** to centralized S3 bucket and CloudWatch

**Pros/Cons**
- Pros: Centralized security policy, deep packet inspection, compliance, scalable
- Cons: Additional latency, complexity, single inspection VPC is potential bottleneck, higher costs

### Pattern 4: Hybrid Cloud with Direct Connect and VPN Failover

**Use Case**
- Mission-critical applications requiring high-bandwidth, low-latency on-premises connectivity
- High availability with automatic failover
- Private connectivity without internet exposure

**Implementation Approach**
1. **Primary**: AWS Direct Connect (dedicated 10 Gbps connection)
2. **Backup**: Site-to-Site VPN over internet
3. **Transit Gateway** with:
   - Direct Connect Gateway attachment (via Transit VIF)
   - VPN attachment
4. **BGP Configuration**:
   - Direct Connect: Higher preference (shorter AS path or higher local preference)
   - VPN: Lower preference (longer AS path or lower local preference)
   - BFD for fast failover detection
5. **Redundancy**: Secondary Direct Connect connection from different customer router and AWS Direct Connect location
6. **Route 53 Resolver** endpoints for hybrid DNS

**Pros/Cons**
- Pros: High availability, predictable bandwidth, low latency, automatic failover
- Cons: Higher costs, complex BGP configuration, Direct Connect provisioning time (weeks)

### Pattern 5: Multi-Account Network Architecture

**Use Case**
- AWS Organizations with multiple accounts for different teams/applications
- Centralized network management and security
- Cost allocation per account
- Compliance and isolation requirements

**Implementation Approach**
1. **Network Account**: Central account owning Transit Gateway
2. **Shared Services Account**: Active Directory, DNS, monitoring
3. **Workload Accounts**: Application-specific accounts with VPCs
4. **AWS RAM** to share Transit Gateway with workload accounts
5. **VPC Attachments** from workload VPCs to shared Transit Gateway
6. **Service Control Policies** to enforce network policies
7. **VPC Sharing** for shared subnets (optional)
8. **Centralized VPC Flow Logs** to network or security account

**Pros/Cons**
- Pros: Clear ownership, cost allocation, security isolation, centralized management
- Cons: Cross-account complexity, IAM permissions management, service quotas per account

### Pattern 6: PrivateLink Service Delivery

**Use Case**
- SaaS provider delivering services to customer VPCs
- Shared services across multiple VPCs without peering
- Private, scalable, secure service access

**Implementation Approach**
1. **Service Provider VPC** with application behind Network Load Balancer
2. **VPC Endpoint Service** exposing NLB
3. **Service Consumer VPCs** create VPC endpoints to the service
4. **Allow-listed Principals** for access control
5. **Private DNS** for seamless integration
6. **Multi-AZ** deployment for high availability

**Pros/Cons**
- Pros: No VPC peering required, no CIDR conflicts, scalable to thousands of consumers, private connectivity
- Cons: PrivateLink costs (per hour and per GB), NLB required, one-way connectivity

## Best Practices

### Enterprise-Level Recommendations

**CIDR Planning**
- Plan IP address space before creating VPCs (RFC 1918: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- Use /16 for VPCs (65,536 IPs) for flexibility
- Reserve IP ranges for future growth
- Document CIDR allocations to prevent overlaps
- Use AWS VPC IPAM for centralized management

**High Availability**
- Deploy resources across multiple Availability Zones
- Use multiple Direct Connect connections from different locations
- Configure VPN backup for Direct Connect
- Implement health checks and automated failover
- Test failover scenarios regularly

**Scalability**
- Use Transit Gateway instead of VPC peering mesh for >5 VPCs
- Plan for service quotas (VPCs per region, routes per route table, etc.)
- Design for horizontal scaling (add more VPCs/subnets rather than expanding existing)
- Monitor growth and request quota increases proactively

**Documentation**
- Network diagrams showing VPCs, subnets, route tables, gateways
- CIDR allocation spreadsheet
- Routing tables and traffic flows
- Security group and NACL rules
- DNS architecture and zone delegation

### Security Considerations

**Defense in Depth**
- Security Groups for instance-level security
- NACLs for subnet-level security
- AWS Network Firewall or third-party firewalls for traffic inspection
- AWS WAF for application-layer protection
- VPC Flow Logs for network monitoring

**Least Privilege Network Access**
- Restrict security group rules to minimum required
- Use prefix lists for managed IP ranges (AWS services, on-premises)
- Deny unnecessary protocols and ports
- Regular audit of security group rules (unused, overly permissive)

**Network Segmentation**
- Separate VPCs for different environments (dev, test, prod)
- Separate subnets for different tiers (web, app, database)
- Use Transit Gateway route tables for micro-segmentation
- Implement network isolation for PCI, HIPAA, or other compliance requirements

**Encryption**
- TLS for data in transit
- VPN for on-premises connectivity
- Direct Connect with MACsec for link-layer encryption (100 Gbps connections)
- PrivateLink for private connectivity without internet exposure

### Cost Optimization

**Data Transfer Costs**
- Minimize cross-region data transfer (use regional services)
- Use VPC endpoints for S3/DynamoDB (avoid NAT Gateway costs)
- Use PrivateLink for AWS services (avoid data processing charges of NAT)
- Direct Connect for predictable, high-volume on-premises traffic
- CloudFront for content delivery (reduce origin data transfer)

**NAT Gateway Optimization**
- Use VPC endpoints to bypass NAT for AWS services
- Consolidate NAT Gateways in shared egress VPC
- Schedule NAT Gateways in non-production environments
- Monitor and right-size NAT Gateway usage

**Transit Gateway Costs**
- Charged per attachment-hour and per GB processed
- Consolidate VPCs where possible to reduce attachments
- Monitor data transfer through Transit Gateway
- Compare costs vs. VPC peering for specific scenarios

**Elastic Load Balancer Costs**
- Application Load Balancer: Per hour + LCU (Load Balancer Capacity Units)
- Network Load Balancer: Per hour + NLCU
- Gateway Load Balancer: Per hour + GLCU
- Right-size based on traffic patterns
- Use target group features to reduce number of load balancers

### Performance Tuning

**Enhanced Networking**
- Enable Enhanced Networking (ENA) for up to 100 Gbps
- Use Elastic Fabric Adapter (EFA) for HPC workloads (MPI, NCCL)
- Choose network-optimized instance types for high throughput

**Placement Groups**
- Cluster placement group for low-latency, high-bandwidth
- Spread placement group for high availability
- Partition placement group for distributed workloads (Hadoop, Cassandra)

**Network Performance**
- Same-AZ communication for lowest latency
- VPC peering for high-bandwidth VPC-to-VPC (vs. Transit Gateway)
- Direct Connect for high-bandwidth, low-latency on-premises connectivity
- Global Accelerator for global traffic optimization over AWS network

**Monitoring**
- VPC Flow Logs for traffic analysis
- CloudWatch metrics for NAT Gateway, Load Balancers, Transit Gateway
- VPC Reachability Analyzer for connectivity troubleshooting
- Network Insights for path analysis

## Common Scenarios

### Scenario 1: 100-VPC Enterprise Network

**Context**: Global enterprise with 100 VPCs across 5 regions, requiring centralized security and on-premises connectivity

**Architecture**
- **Regional Transit Gateways** in each region (5 total)
- **Transit Gateway Peering** between regions for cross-region communication
- **Direct Connect** connections in 2 primary regions with VPN backup
- **Centralized Egress VPCs** in each region with NAT Gateways and inspection
- **Shared Services VPC** in each region (AD, DNS, monitoring)
- **AWS Network Firewall** in inspection VPC for east-west and north-south traffic
- **Route 53 Resolver** with outbound endpoints for on-premises DNS queries
- **VPC Flow Logs** centralized to S3 in security account
- **AWS IPAM** for centralized IP address management

**Key Considerations**
- Transit Gateway route table segmentation (production, non-production, shared services)
- BGP configuration for Direct Connect with route filtering
- Multi-region failover and disaster recovery
- Cost optimization through shared egress and right-sizing
- Security through centralized inspection and logging

### Scenario 2: Hybrid Cloud for Lift-and-Shift Migration

**Context**: Migrating 200 applications from on-premises to AWS over 12 months, requiring hybrid connectivity during migration

**Architecture**
- **AWS Direct Connect** (10 Gbps) with redundant connection
- **Site-to-Site VPN** as backup
- **Transit Gateway** connecting migration VPCs and Direct Connect
- **AWS Directory Service** (Managed Microsoft AD) with trust to on-premises AD
- **Route 53 Resolver** inbound/outbound endpoints for hybrid DNS
- **VPN Client** for administrator access
- **AWS Migration Hub** for tracking migration progress
- **Phased migration approach**: Rehost first, then optimize

**Key Considerations**
- Bandwidth planning for data transfer (use AWS DataSync for file servers)
- IP address overlap avoidance (use AWS IPAM)
- Active Directory integration and GPO management
- DNS resolution for both on-premises and AWS resources
- Security group configuration for on-premises IP ranges
- Cost management during hybrid period (double infrastructure)

### Scenario 3: Multi-Region SaaS with PrivateLink

**Context**: SaaS provider serving enterprise customers via private connectivity in 3 regions

**Architecture**
- **Regional Deployments** in us-east-1, eu-west-1, ap-southeast-1
- **Network Load Balancers** in each region fronting ECS/EKS services
- **VPC Endpoint Services** in each region
- **PrivateLink** allowing customers to create VPC endpoints
- **Route 53** latency-based routing to direct customers to nearest region
- **CloudFront** for edge caching (public API)
- **AWS WAF** on CloudFront and ALB for security
- **AWS Shield Advanced** for DDoS protection

**Key Considerations**
- Customer onboarding process for VPC endpoint creation
- Allow-listing customer AWS principals
- Private DNS configuration for seamless access
- High availability with multi-AZ NLB deployment
- Monitoring customer connectivity and troubleshooting
- Cost model (pass PrivateLink costs to customers or absorb)

### Scenario 4: Financial Services Compliance Network

**Context**: Financial services company with PCI-DSS and regulatory requirements for network segmentation and inspection

**Architecture**
- **Isolated VPCs** for cardholder data environment (CDE)
- **Transit Gateway** with strict route tables preventing CDE cross-contamination
- **Dedicated Direct Connect** for PCI environment (separate from corporate)
- **AWS Network Firewall** with IPS rules for all CDE traffic
- **VPC Flow Logs** sent to immutable S3 bucket in compliance account
- **AWS PrivateLink** for accessing AWS services without internet exposure
- **Private subnets only** in CDE (no internet gateway)
- **Network ACLs** denying all by default, explicit allows only
- **Security Groups** with strict allow-lists based on IP and ports

**Key Considerations**
- PCI-DSS network segmentation requirements
- Quarterly PCI scans and penetration testing
- Encryption requirements (TLS 1.2+, MACsec for Direct Connect)
- Audit logging and retention (7 years)
- Change management and approval process
- Regular network diagram updates and validation

### Scenario 5: IoT Data Ingestion at Scale

**Context**: IoT platform ingesting data from 1M devices globally, requiring low latency and high throughput

**Architecture**
- **AWS IoT Core** for device connectivity (MQTT)
- **AWS Global Accelerator** providing anycast IPs for device connections
- **Regional VPCs** with IoT Core endpoints in 6 regions
- **Network Load Balancers** for custom protocols
- **PrivateLink** for backend service access
- **Kinesis Data Streams** for ingestion pipeline
- **VPC endpoints** for Kinesis to avoid NAT Gateway bottleneck
- **Lambda functions** for real-time processing
- **DynamoDB** for device state management

**Key Considerations**
- Device connection distribution across regions
- Network throughput planning (1M devices * data rate)
- VPC endpoint scaling for Kinesis (multiple endpoints per AZ)
- CloudWatch metrics for connection counts and throughput
- Cost optimization (data transfer, VPC endpoints, NAT)
- Security: TLS mutual auth, device certificates, IoT policies

## AWS CLI Examples

```bash
# VPC - Create VPC with DNS support
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Production-VPC}]' \
  --amazon-provided-ipv6-cidr-block \
  --query 'Vpc.VpcId' \
  --output text

# VPC - Add secondary CIDR block
aws ec2 associate-vpc-cidr-block \
  --vpc-id vpc-12345678 \
  --cidr-block 10.1.0.0/16

# VPC - Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-12345678 \
  --enable-dns-hostnames

# Transit Gateway - Create Transit Gateway
aws ec2 create-transit-gateway \
  --description "Enterprise-Hub" \
  --options "AmazonSideAsn=64512,AutoAcceptSharedAttachments=disable,DefaultRouteTableAssociation=disable,DefaultRouteTablePropagation=disable,DnsSupport=enable,VpnEcmpSupport=enable" \
  --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=Main-TGW}]'

# Transit Gateway - Create attachment to VPC
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-12345678 \
  --vpc-id vpc-12345678 \
  --subnet-ids subnet-11111111 subnet-22222222 \
  --options "DnsSupport=enable,Ipv6Support=disable" \
  --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=Prod-VPC-Attachment}]'

# Transit Gateway - Create route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Production-Routes}]'

# Transit Gateway - Create static route
aws ec2 create-transit-gateway-route \
  --destination-cidr-block 10.0.0.0/16 \
  --transit-gateway-route-table-id tgw-rtb-12345678 \
  --transit-gateway-attachment-id tgw-attach-12345678

# Transit Gateway - Peer with another region
aws ec2 create-transit-gateway-peering-attachment \
  --transit-gateway-id tgw-12345678 \
  --peer-transit-gateway-id tgw-87654321 \
  --peer-account-id 123456789012 \
  --peer-region eu-west-1 \
  --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=US-EU-Peering}]'

# Direct Connect - Create virtual interface
aws directconnect create-private-virtual-interface \
  --connection-id dxcon-12345678 \
  --new-private-virtual-interface \
    virtualInterfaceName=Production-VIF,\
vlan=100,\
asn=65000,\
authKey=secretkey123,\
amazonAddress=169.254.0.1/30,\
customerAddress=169.254.0.2/30,\
addressFamily=ipv4,\
virtualGatewayId=vgw-12345678

# Direct Connect - Create Transit VIF
aws directconnect create-transit-virtual-interface \
  --connection-id dxcon-12345678 \
  --new-transit-virtual-interface \
    virtualInterfaceName=Transit-VIF,\
vlan=200,\
asn=65000,\
directConnectGatewayId=dxgw-12345678,\
addressFamily=ipv4

# VPC Peering - Create peering connection
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-11111111 \
  --peer-vpc-id vpc-22222222 \
  --peer-region us-west-2 \
  --peer-owner-id 123456789012 \
  --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=VPC1-VPC2-Peer}]'

# VPC Peering - Accept peering connection
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-12345678 \
  --region us-west-2

# VPC Endpoint - Create interface endpoint for S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678 \
  --vpc-endpoint-type Gateway

# VPC Endpoint - Create interface endpoint for EC2
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.ec2 \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-11111111 subnet-22222222 \
  --security-group-ids sg-12345678 \
  --private-dns-enabled

# VPC Endpoint Service - Create endpoint service
aws ec2 create-vpc-endpoint-service-configuration \
  --network-load-balancer-arns arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/net/my-nlb/1234567890abcdef \
  --acceptance-required \
  --tag-specifications 'ResourceType=vpc-endpoint-service,Tags=[{Key=Name,Value=My-Service}]'

# Route 53 Resolver - Create inbound endpoint
aws route53resolver create-resolver-endpoint \
  --name "OnPrem-to-AWS-DNS" \
  --security-group-ids sg-12345678 \
  --direction INBOUND \
  --ip-addresses SubnetId=subnet-11111111,Ip=10.0.1.10 SubnetId=subnet-22222222,Ip=10.0.2.10

# Route 53 Resolver - Create outbound endpoint
aws route53resolver create-resolver-endpoint \
  --name "AWS-to-OnPrem-DNS" \
  --security-group-ids sg-12345678 \
  --direction OUTBOUND \
  --ip-addresses SubnetId=subnet-11111111 SubnetId=subnet-22222222

# Route 53 Resolver - Create resolver rule
aws route53resolver create-resolver-rule \
  --creator-request-id $(uuidgen) \
  --name "OnPrem-Corp-Domain" \
  --rule-type FORWARD \
  --domain-name corp.example.com \
  --resolver-endpoint-id rslvr-out-12345678 \
  --target-ips Ip=192.168.1.10,Port=53 Ip=192.168.1.11,Port=53

# Route 53 Resolver - Associate rule with VPC
aws route53resolver associate-resolver-rule \
  --resolver-rule-id rslvr-rr-12345678 \
  --vpc-id vpc-12345678 \
  --name "Associate-Production-VPC"

# VPC Flow Logs - Enable flow logs to S3
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::my-flow-logs-bucket/vpc-flow-logs/ \
  --log-format '${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}' \
  --tag-specifications 'ResourceType=vpc-flow-log,Tags=[{Key=Name,Value=Production-VPC-Logs}]'

# VPC Flow Logs - Enable flow logs to CloudWatch
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/VPCFlowLogsRole

# VPC Reachability Analyzer - Analyze path
aws ec2 create-network-insights-path \
  --source i-11111111 \
  --destination i-22222222 \
  --protocol tcp \
  --destination-port 443 \
  --tag-specifications 'ResourceType=network-insights-path,Tags=[{Key=Name,Value=Instance-to-Instance}]'

aws ec2 start-network-insights-analysis \
  --network-insights-path-id nip-12345678

# AWS Network Firewall - Create firewall policy
aws network-firewall create-firewall-policy \
  --firewall-policy-name "Enterprise-Firewall-Policy" \
  --firewall-policy '{"StatelessDefaultActions":["aws:forward_to_sfe"],"StatelessFragmentDefaultActions":["aws:forward_to_sfe"],"StatefulRuleGroupReferences":[{"ResourceArn":"arn:aws:network-firewall:us-east-1:123456789012:stateful-rulegroup/my-rules"}]}'

# AWS Network Firewall - Create firewall
aws network-firewall create-firewall \
  --firewall-name "Production-Firewall" \
  --firewall-policy-arn arn:aws:network-firewall:us-east-1:123456789012:firewall-policy/Enterprise-Firewall-Policy \
  --vpc-id vpc-12345678 \
  --subnet-mappings SubnetId=subnet-11111111 SubnetId=subnet-22222222

# Site-to-Site VPN - Create customer gateway
aws ec2 create-customer-gateway \
  --type ipsec.1 \
  --public-ip 203.0.113.1 \
  --bgp-asn 65000 \
  --tag-specifications 'ResourceType=customer-gateway,Tags=[{Key=Name,Value=OnPrem-Router}]'

# Site-to-Site VPN - Create VPN connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-12345678 \
  --transit-gateway-id tgw-12345678 \
  --options TunnelOptions='[{TunnelInsideCidr=169.254.10.0/30,PreSharedKey=secretkey123},{TunnelInsideCidr=169.254.11.0/30,PreSharedKey=secretkey456}]' \
  --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=OnPrem-VPN}]'

# Global Accelerator - Create accelerator
aws globalaccelerator create-accelerator \
  --name "Global-App-Accelerator" \
  --ip-address-type IPV4 \
  --enabled

# Global Accelerator - Create listener
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::123456789012:accelerator/12345678-1234-1234-1234-123456789012 \
  --protocol TCP \
  --port-ranges FromPort=443,ToPort=443

# Global Accelerator - Create endpoint group
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::123456789012:accelerator/12345678-1234-1234-1234-123456789012/listener/12345678 \
  --endpoint-group-region us-east-1 \
  --endpoint-configurations EndpointId=arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890abcdef,Weight=128
```

## Study Tips

### SAP-C02 Exam Focus Areas

**High-Priority Networking Topics**
- Transit Gateway architecture and routing (extremely common)
- Direct Connect configurations and high availability patterns
- Hybrid DNS with Route 53 Resolver
- PrivateLink vs. VPC Peering vs. Transit Gateway decision-making
- Multi-region networking and disaster recovery
- VPC endpoint types and use cases
- Network security layering (SG, NACL, Network Firewall, WAF)
- VPC Flow Logs analysis and troubleshooting
- BGP routing and path selection

**Scenario Types**
- Design network for multi-account organization
- Troubleshoot connectivity issues (VPC Flow Logs, Reachability Analyzer)
- Optimize costs for data transfer and network services
- Implement security controls for compliance (PCI, HIPAA)
- Design hybrid connectivity with failover
- Multi-region architecture with low latency

**Common Question Patterns**
- "Most cost-effective solution" → Often VPC endpoints, right-sized Direct Connect
- "Highest performance" → Enhanced networking, placement groups, Direct Connect
- "Lowest latency" → Same-AZ, Direct Connect, Global Accelerator
- "Highest availability" → Multi-AZ, multiple Direct Connect, VPN backup
- "Most secure" → Private subnets, PrivateLink, Network Firewall, encryption

### Key Differences from SAA-C03

**SAA-C03 Networking Knowledge**
- Basic VPC concepts (subnets, route tables, internet gateway, NAT)
- Simple VPC peering
- Basic security groups and NACLs
- ALB and NLB basics
- CloudFront and Route 53 fundamentals

**Additional SAP-C02 Requirements**
- Complex Transit Gateway architectures with multiple route tables
- Direct Connect with BGP routing and failover configurations
- Advanced PrivateLink patterns (endpoint services, cross-account)
- Hybrid DNS resolution with Route 53 Resolver
- Multi-region networking with Transit Gateway peering
- Network security inspection architectures
- Global Accelerator for global applications
- VPC Flow Logs analysis and VPC Reachability Analyzer
- AWS Network Firewall and Gateway Load Balancer
- Cost optimization strategies for network services

### Complex Scenarios to Master

**Transit Gateway Architecture**
- Hub-and-spoke with segmentation (production, non-production, shared services)
- Centralized egress through dedicated VPC
- Centralized security inspection
- Multi-region with Transit Gateway peering
- Inter-region routing and traffic patterns

**Hybrid Connectivity**
- Direct Connect with VPN failover using BGP
- Active-active Direct Connect with ECMP
- Multi-region Direct Connect architecture
- Direct Connect Gateway for multi-VPC access
- MACsec encryption for 100 Gbps connections

**Network Security**
- Defense in depth: SG → NACL → Network Firewall → WAF
- Centralized inspection VPC with Gateway Load Balancer
- AWS Network Firewall rule groups and policies
- VPC Traffic Mirroring for IDS/IPS
- GuardDuty VPC Flow Log analysis

**Cost Optimization**
- VPC endpoints to eliminate NAT Gateway costs
- Direct Connect vs. VPN cost analysis
- Transit Gateway vs. VPC peering cost comparison
- Data transfer cost reduction strategies
- Right-sizing NAT Gateways and load balancers

### Practice Lab Recommendations

**Hands-On Labs**
1. **Build Transit Gateway Hub-and-Spoke**
   - Create 5 VPCs in different accounts
   - Set up Transit Gateway with RAM sharing
   - Configure segmented route tables
   - Test connectivity and routing

2. **Hybrid DNS Resolution**
   - Set up Route 53 Resolver inbound/outbound endpoints
   - Create resolver rules for conditional forwarding
   - Simulate on-premises DNS server
   - Test bidirectional DNS resolution

3. **PrivateLink Service**
   - Create NLB-backed service in provider VPC
   - Set up VPC endpoint service
   - Create endpoint in consumer VPC
   - Test private connectivity and DNS

4. **Network Security Inspection**
   - Deploy AWS Network Firewall
   - Configure stateful and stateless rules
   - Route traffic through firewall endpoints
   - Analyze firewall logs in CloudWatch

5. **VPC Flow Logs Analysis**
   - Enable VPC Flow Logs to S3
   - Query logs with Athena
   - Create CloudWatch Insights queries
   - Troubleshoot connectivity issues

6. **Multi-Region Failover**
   - Deploy application in two regions
   - Configure Route 53 health checks and failover
   - Simulate regional failure
   - Validate automatic failover

## Additional Resources

### AWS Whitepapers

**Essential Reading**
- Building a Scalable and Secure Multi-VPC AWS Network Infrastructure
- Amazon VPC Connectivity Options
- AWS Direct Connect Best Practices
- Hybrid Cloud with AWS
- AWS Security Best Practices

### Architecture Guides

- AWS Transit Gateway Network Patterns
- AWS PrivateLink Architecture Guide
- Centralized Network Inspection
- Multi-Region Architecture Best Practices
- AWS Networking Immersion Day (workshop)

### Documentation Links

**Core Networking**
- Amazon VPC: https://docs.aws.amazon.com/vpc/
- AWS Transit Gateway: https://docs.aws.amazon.com/vpc/latest/tgw/
- AWS Direct Connect: https://docs.aws.amazon.com/directconnect/
- AWS PrivateLink: https://docs.aws.amazon.com/vpc/latest/privatelink/

**Security & Monitoring**
- AWS Network Firewall: https://docs.aws.amazon.com/network-firewall/
- VPC Flow Logs: https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
- VPC Reachability Analyzer: https://docs.aws.amazon.com/vpc/latest/reachability/

**Global Services**
- Amazon Route 53: https://docs.aws.amazon.com/route53/
- AWS Global Accelerator: https://docs.aws.amazon.com/global-accelerator/
- Amazon CloudFront: https://docs.aws.amazon.com/cloudfront/

**Tools**
- AWS VPC IPAM: https://docs.aws.amazon.com/vpc/latest/ipam/
- Transit Gateway Network Manager: https://docs.aws.amazon.com/vpc/latest/tgw/network-manager.html

### Video Resources
- AWS re:Invent sessions on networking (search "NET" track)
- AWS Networking Fundamentals (AWS Skill Builder)
- Advanced Networking Workshop
