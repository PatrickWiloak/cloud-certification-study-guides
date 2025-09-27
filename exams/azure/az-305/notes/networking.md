# Azure Networking Fundamentals

## Virtual Network (VNet) Basics

### What is a Virtual Network?
A logically isolated network in Azure that provides a secure boundary for your resources. Similar to a traditional network in your datacenter.

### Key Characteristics
- **Regional scope**: VNets exist within a single region
- **Address space**: Private IP address ranges (RFC 1918)
- **Subnets**: Subdivisions of the VNet address space
- **Default connectivity**: Resources in same VNet can communicate
- **Isolated by default**: No internet access unless explicitly configured

### VNet Address Spaces
```
Common Private IP Ranges:
10.0.0.0/8     (10.0.0.0 to 10.255.255.255)
172.16.0.0/12  (172.16.0.0 to 172.31.255.255)
192.168.0.0/16 (192.168.0.0 to 192.168.255.255)

Example VNet Design:
VNet: 10.0.0.0/16
├── Web Subnet: 10.0.1.0/24
├── App Subnet: 10.0.2.0/24
├── Database Subnet: 10.0.3.0/24
└── Management Subnet: 10.0.4.0/24
```

### Reserved IP Addresses
In each subnet, Azure reserves 5 IP addresses:
- **x.x.x.0**: Network address
- **x.x.x.1**: Default gateway (Azure router)
- **x.x.x.2, x.x.x.3**: Azure DNS IPs
- **x.x.x.255**: Network broadcast address

## Subnets Deep Dive

### Subnet Types by Purpose
| Type | Purpose | Route Table | NSG | Examples |
|------|---------|-------------|-----|----------|
| **Public** | Internet-facing resources | Route to internet | Allow inbound | Web servers, load balancers |
| **Private** | Internal resources | No internet route | Restrict access | Application servers, databases |
| **DMZ** | Perimeter security | Controlled routing | Strict rules | Firewalls, reverse proxies |
| **Gateway** | VPN/ExpressRoute | Azure-managed | Azure-managed | VPN Gateway, ExpressRoute |

### Special Subnets
- **GatewaySubnet**: Required for VPN/ExpressRoute gateways
- **AzureFirewallSubnet**: Required for Azure Firewall
- **AzureBastionSubnet**: Required for Azure Bastion
- **RouteServerSubnet**: Required for Route Server

### Subnet Delegation
Allows Azure services to create service-specific resources in your subnet:
- **SQL Managed Instance**: Requires dedicated subnet
- **Azure NetApp Files**: Delegates subnet to service
- **Container Instances**: Enables VNet integration
- **App Service**: VNet integration for web apps

## Network Security Groups (NSGs)

### What are NSGs?
Virtual firewalls that filter network traffic based on security rules. Applied at subnet or network interface level.

### NSG Rule Components
| Component | Description | Example |
|-----------|-------------|---------|
| **Priority** | Rule evaluation order (100-4096) | 1000 |
| **Source** | Traffic origin | IP, CIDR, Service Tag, ASG |
| **Destination** | Traffic destination | IP, CIDR, Service Tag, ASG |
| **Protocol** | Network protocol | TCP, UDP, ICMP, Any |
| **Port Range** | Destination ports | 80, 443, 1433, * |
| **Action** | Allow or Deny | Allow |

### Default NSG Rules
```
Inbound Rules:
- Allow VNet traffic (Priority: 65000)
- Allow Azure Load Balancer (Priority: 65001)  
- Deny all (Priority: 65500)

Outbound Rules:
- Allow VNet traffic (Priority: 65000)
- Allow Internet (Priority: 65001)
- Deny all (Priority: 65500)
```

### Service Tags
Predefined groups of IP address prefixes:
- **Internet**: All internet IP addresses
- **VirtualNetwork**: All VNet address space
- **AzureLoadBalancer**: Azure load balancer IPs
- **Storage**: Azure Storage IP ranges
- **Sql**: Azure SQL IP ranges
- **AzureActiveDirectory**: Azure AD IP ranges

### Application Security Groups (ASGs)
Logical grouping of VMs for security rule application:
```
ASG Examples:
- WebServers ASG
- AppServers ASG  
- DatabaseServers ASG

NSG Rule:
Source: WebServers ASG
Destination: AppServers ASG
Port: 443
Action: Allow
```

## Load Balancing Solutions

### Azure Load Balancer (Layer 4)
**Basic network load balancer for high availability**

#### SKUs Comparison
| Feature | Basic | Standard |
|---------|-------|----------|
| **Backend pool size** | 300 | 1000 |
| **Availability Zones** | No | Yes |
| **SLA** | No | 99.99% |
| **Health probes** | TCP, HTTP | TCP, HTTP, HTTPS |
| **Security** | Open by default | Secure by default (NSG required) |

#### Load Balancer Types
- **Public**: Internet-facing with public IP
- **Internal**: Private IP within VNet
- **Cross-region**: Global load balancing

### Application Gateway (Layer 7)
**Web traffic load balancer with application-layer features**

#### Key Features
- **SSL termination**: Offload SSL processing
- **URL-based routing**: Route based on URL path
- **Multi-site hosting**: Host multiple websites
- **Web Application Firewall**: Built-in security protection
- **Auto-scaling**: Scale based on demand
- **Session affinity**: Cookie-based routing

#### Routing Rules
```
URL Path Routing:
contoso.com/images/* → Image server pool
contoso.com/api/* → API server pool
contoso.com/* → Web server pool

Host-based Routing:
www.contoso.com → Contoso website pool
api.contoso.com → API server pool
```

### Azure Front Door
**Global load balancer and CDN with WAF**

#### Key Features
- **Global load balancing**: Route to closest healthy backend
- **SSL offloading**: Terminate SSL at edge
- **URL rewrite/redirect**: Modify requests/responses
- **Session affinity**: Sticky sessions
- **Web Application Firewall**: Global security protection

#### Routing Methods
- **Latency-based**: Route to lowest latency backend
- **Priority**: Failover routing
- **Weighted**: Distribute traffic by percentage
- **Session affinity**: Route to same backend

### Traffic Manager
**DNS-based global load balancer**

#### Routing Methods
| Method | Description | Use Case |
|--------|-------------|----------|
| **Priority** | Failover routing | Disaster recovery |
| **Weighted** | Traffic distribution | Blue-green deployment |
| **Performance** | Lowest latency | Global applications |
| **Geographic** | Route by user location | Compliance, localization |
| **Multivalue** | Return multiple healthy endpoints | DNS-based load balancing |
| **Subnet** | Route by client IP range | Internal applications |

## Virtual Network Connectivity

### VNet Peering
**Connect VNets for private communication**

#### Peering Types
| Type | Scope | Use Case |
|------|-------|----------|
| **Regional** | Same region | Connect VNets in region |
| **Global** | Cross-region | Multi-region connectivity |

#### Key Features
- **Non-transitive**: Not routed through intermediate VNets
- **No gateways**: Direct connection between VNets
- **Bandwidth**: No bandwidth limitations
- **Cost**: Data transfer charges apply

### VPN Gateway
**Encrypted connectivity over internet**

#### VPN Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Site-to-Site** | Connect on-premises to Azure | Hybrid connectivity |
| **Point-to-Site** | Connect individual clients | Remote access |
| **VNet-to-VNet** | Connect Azure VNets | Multi-region connectivity |

#### Gateway SKUs
| SKU | Bandwidth | Tunnels | P2S Connections |
|-----|-----------|---------|-----------------|
| **Basic** | 100 Mbps | 10 | 128 |
| **VpnGw1** | 650 Mbps | 30 | 250 |
| **VpnGw2** | 1 Gbps | 30 | 500 |
| **VpnGw3** | 1.25 Gbps | 30 | 1000 |

### ExpressRoute
**Private connection to Azure through connectivity provider**

#### Connection Models
- **CloudExchange Co-location**: Datacenter with provider presence
- **Point-to-point Ethernet**: Direct connection to provider
- **Any-to-any (IPVPN)**: Connect through MPLS network
- **ExpressRoute Direct**: Direct connection to Microsoft edge

#### Circuit SKUs
| SKU | Bandwidth | Locations |
|-----|-----------|-----------|
| **Local** | Up to 10 Gbps | Single metro area |
| **Standard** | 50 Mbps - 10 Gbps | Single region |
| **Premium** | 50 Mbps - 10 Gbps | Global connectivity |

## Azure Firewall

### What is Azure Firewall?
Managed cloud-based network security service that protects Azure Virtual Network resources.

### Key Features
- **Stateful firewall**: Track connection state
- **High availability**: Built-in high availability
- **Unrestricted cloud scalability**: Scale as needed
- **FQDN filtering**: Filter by fully qualified domain names
- **Network and application rules**: Layer 3-7 filtering
- **Threat intelligence**: Microsoft threat intelligence integration

### Rule Types
| Type | Layer | Use Case |
|------|-------|----------|
| **Network rules** | Layer 3-4 | IP, port, protocol filtering |
| **Application rules** | Layer 7 | FQDN, URL filtering |
| **NAT rules** | DNAT | Publish internal services |

### Azure Firewall Manager
Centralized policy and route management for multiple Azure Firewalls:
- **Security policies**: Centralized rule management
- **Hub and spoke**: Manage multiple hubs
- **Third-party security**: Integrate partner solutions

## Azure Bastion

### What is Azure Bastion?
Fully managed PaaS service providing secure RDP and SSH connectivity to VMs without exposing public IPs.

### Key Benefits
- **No public IPs**: VMs don't need public IP addresses
- **SSL protection**: RDP/SSH over HTTPS
- **Azure portal integration**: Connect through web browser
- **No client software**: No VPN client or special software needed
- **Hardened service**: Microsoft-managed and hardened

### Bastion SKUs
| SKU | Features | Max Sessions |
|-----|----------|--------------|
| **Basic** | RDP/SSH connectivity | 25 |
| **Standard** | File upload/download, native client support | 50 |

## Private Link & Private Endpoints

### Private Link
Service that provides private connectivity to Azure PaaS services over a private endpoint.

### Private Endpoints
Network interface that connects privately to Azure services:
- **Private IP**: Gets IP from your VNet
- **Secure connectivity**: Traffic stays on Microsoft backbone
- **Disable public access**: Optional public access disable
- **DNS integration**: Automatic private DNS zone creation

### Supported Services
- **Storage**: Blob, File, Queue, Table, Data Lake
- **Databases**: SQL Database, Cosmos DB, MySQL, PostgreSQL
- **Platform services**: Key Vault, Event Hub, Service Bus
- **AI services**: Cognitive Services, OpenAI

## DNS in Azure

### Azure DNS
Hosting service for DNS domains using Azure infrastructure.

### DNS Zone Types
| Type | Purpose | Use Case |
|------|---------|----------|
| **Public** | Internet-resolvable domains | Public websites |
| **Private** | VNet-resolvable domains | Internal services |

### DNS Resolution in VNet
1. **Azure-provided DNS**: Default DNS for VNet (168.63.129.16)
2. **Custom DNS**: Specify your own DNS servers
3. **Hybrid DNS**: Forward specific domains to on-premises

### Private DNS Zones
- **VNet linking**: Link private zones to VNets
- **Auto-registration**: Automatically register VM records
- **Split-brain DNS**: Different records for internal/external

## Network Monitoring & Diagnostics

### Network Watcher
Suite of tools to monitor and diagnose networking issues:

#### Key Tools
- **Topology**: Visualize network topology
- **Connection Monitor**: Monitor connectivity between endpoints
- **IP Flow Verify**: Check if traffic is allowed/denied
- **Next Hop**: Determine next hop for routing
- **VPN Diagnostics**: Troubleshoot VPN connectivity
- **Packet Capture**: Capture network traffic
- **NSG Flow Logs**: Log NSG rule decisions

### Connection Monitor
Monitor connectivity between Azure and on-premises resources:
- **Synthetic monitoring**: Proactive connectivity testing
- **Multi-hop topology**: Understand network paths
- **Performance metrics**: Latency, packet loss, jitter
- **Alerting**: Proactive issue detection

## Best Practices

### VNet Design
- Plan IP address spaces to avoid conflicts
- Use hub-and-spoke topology for multiple VNets
- Implement proper subnet segmentation
- Use service endpoints and private endpoints for security
- Document network architecture and IP allocations

### Security
- Implement defense in depth with NSGs and Azure Firewall
- Use just-in-time VM access
- Minimize public IP address usage
- Enable DDoS protection for production workloads
- Regular security rule audits

### Connectivity
- Use ExpressRoute for production hybrid connectivity
- Implement redundancy for critical connections
- Monitor network performance and connectivity
- Use Azure Bastion for secure VM access
- Plan for disaster recovery connectivity

## Cost Optimization

### Bandwidth Optimization
- Use Azure CDN to reduce bandwidth costs
- Minimize cross-region traffic
- Use VNet peering instead of VPN for Azure-to-Azure
- Right-size VPN Gateway and ExpressRoute circuits
- Monitor data transfer costs

### Resource Optimization
- Use Standard Load Balancer only when needed
- Right-size Application Gateway instances
- Use Traffic Manager for DNS-based load balancing
- Optimize NSG rule placement
- Regular cleanup of unused network resources

## Common Pitfalls

### VNet Issues
- IP address space conflicts preventing peering
- Incorrect subnet sizing for future growth
- Missing service endpoints causing internet traffic
- Poor subnet segmentation affecting security
- Not planning for hybrid connectivity requirements

### Security Issues
- Overly permissive NSG rules
- Missing network security monitoring
- Not using private endpoints for PaaS services
- Inadequate DDoS protection
- Poor credential management for VPN connections

### Connectivity Problems
- ExpressRoute single point of failure
- VPN Gateway capacity planning issues
- DNS resolution problems in hybrid scenarios
- Application Gateway health probe misconfigurations
- Load balancer backend pool health issues