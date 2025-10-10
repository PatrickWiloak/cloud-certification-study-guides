# Azure Networking and Virtual Networks - AZ-104

## Virtual Networks (VNets)

### VNet Basics
- Private network in Azure
- CIDR block (e.g., 10.0.0.0/16)
- Regional resource
- Isolated by default
- Free (no charge for VNet itself)

### Address Space
- IPv4 private ranges:
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16
- Non-overlapping across peered VNets
- Can add multiple address spaces
- Cannot shrink (can only expand)

### Subnets
- Subdivision of VNet address space
- Minimum /29 (8 IPs, 5 usable)
- First 4 and last 1 IP reserved
- **Special subnets**: GatewaySubnet, AzureFirewallSubnet, AzureBastionSubnet

Reserved IPs (example 10.0.0.0/24):
- 10.0.0.0: Network address
- 10.0.0.1: Default gateway
- 10.0.0.2, 10.0.0.3: Azure DNS
- 10.0.0.255: Broadcast

### DNS in VNets
**Azure-provided DNS**:
- Default, automatic
- Name resolution within VNet
- No configuration required
- 168.63.129.16 (Azure recursive resolver)

**Custom DNS**:
- Specify custom DNS servers
- On-premises DNS integration
- Conditional forwarding
- Per-VNet or per-NIC setting

**Azure Private DNS**:
- Private DNS zones
- Name resolution across VNets
- Auto-registration with VNet link
- No internet exposure

## Network Security

### Network Security Groups (NSGs)

**Rules**:
- Priority: 100-4096 (lower = higher priority)
- Source/Destination: IP, CIDR, Service tag, ASG
- Protocol: TCP, UDP, ICMP, Any
- Port: Single, range, or *
- Action: Allow or Deny

**Default Rules**:
- AllowVNetInbound (65000)
- AllowAzureLoadBalancerInbound (65001)
- DenyAllInbound (65500)
- AllowVNetOutbound (65000)
- AllowInternetOutbound (65001)
- DenyAllOutbound (65500)

**Association**:
- Subnet level (recommended)
- NIC level
- Both (combined rules apply)

**Rule Evaluation**:
1. Inbound: Subnet NSG → NIC NSG
2. Outbound: NIC NSG → Subnet NSG
3. First match wins (priority order)

### Application Security Groups (ASGs)
- Logical grouping of VMs
- Simplify NSG rules
- Group by role (web, app, database)
- NSG rules reference ASG instead of IPs

Example:
```
Allow HTTPS from WebASG to AppASG
Allow SQL from AppASG to DBASG
```

### Service Tags
- Predefined groups of IP addresses
- Microsoft-managed
- Common tags: Internet, VirtualNetwork, AzureLoadBalancer, Storage, Sql
- Region-specific: Storage.EastUS

### Azure Firewall
- Managed firewall service
- Centralized network security
- Stateful firewall
- Built-in high availability
- Application and network rules
- Threat intelligence-based filtering

**Tiers**:
- **Standard**: Core features
- **Premium**: Advanced threat protection, TLS inspection, IDPS

**Rule Types**:
- **Application Rules**: FQDN-based filtering
- **Network Rules**: IP-based filtering
- **NAT Rules**: Inbound DNAT

### Azure Bastion
- Secure RDP/SSH without public IPs
- Deployed in dedicated subnet (AzureBastionSubnet)
- HTML5 browser-based
- Protection against port scanning
- **SKUs**: Basic, Standard

## VNet Connectivity

### VNet Peering
- Connect two VNets privately
- Traffic over Microsoft backbone
- Low latency, high bandwidth
- **Types**:
  - Regional: Same region
  - Global: Different regions

**Requirements**:
- Non-overlapping address spaces
- Bidirectional configuration
- Not transitive (A-B-C requires A-C peering)

**Features**:
- Gateway transit (use one VPN gateway)
- Allow forwarded traffic
- Service chaining

### VPN Gateway
**Site-to-Site (S2S)**:
- Connect on-premises to Azure
- IPsec/IKE VPN tunnel
- Requires public IP on both sides
- Multiple site connections supported

**Point-to-Site (P2S)**:
- Individual client to Azure
- Useful for remote workers
- Protocols: OpenVPN, IKEv2, SSTP
- Certificate or Azure AD authentication

**VNet-to-VNet**:
- Connect VNets in same or different regions
- Alternative to VNet peering (encryption in transit)

**Gateway SKUs**:
- Basic: 10 tunnels, 100 Mbps
- VpnGw1-5: Up to 10 Gbps, up to 100 tunnels
- VpnGw1-5 AZ: Zone-redundant

**Active-Active**:
- Two gateway instances
- Higher availability
- Dual tunnels

### ExpressRoute
- Private dedicated connection
- 50 Mbps to 10 Gbps
- Connectivity models:
  - Co-location at cloud exchange
  - Point-to-point Ethernet
  - Any-to-any (IPVPN)

**Circuits**:
- Azure private peering: VNets
- Microsoft peering: Microsoft 365, Dynamics 365, Azure Public IPs
- Premium add-on: Global connectivity

**FastPath**:
- Bypass ExpressRoute gateway
- Ultra-low latency
- UltraPerformance or ErGw3AZ SKU

### Virtual WAN
- Hub-and-spoke architecture at scale
- Centralized connectivity
- Branch-to-VNet, VNet-to-VNet
- SD-WAN integration
- **SKUs**: Basic, Standard

## Load Balancing

### Azure Load Balancer
- Layer 4 (TCP/UDP)
- Public or internal
- **SKUs**:
  - Basic: Free, single AZ
  - Standard: SLA, multi-AZ, more features

**Components**:
- Frontend IP
- Backend pool
- Health probes
- Load balancing rules
- Inbound NAT rules

**Distribution Mode**:
- 5-tuple hash (default): Source IP, source port, dest IP, dest port, protocol
- Source IP affinity (session persistence)

### Application Gateway
- Layer 7 (HTTP/HTTPS)
- URL-based routing
- SSL termination
- Web Application Firewall (WAF)
- Auto-scaling
- Zone redundancy

**Features**:
- Path-based routing: /images → pool1, /api → pool2
- Multi-site hosting: Multiple websites
- Redirection: HTTP to HTTPS
- Rewrite HTTP headers
- Custom error pages

**WAF**:
- OWASP top 10 protection
- Bot protection
- Geo-filtering
- Rate limiting

### Azure Front Door
- Global load balancer
- CDN with WAF
- SSL offload
- URL-based routing
- Health probes and failover
- Caching at edge

**Tiers**:
- Standard: Core features
- Premium: Advanced security (Private Link, managed rules)

### Traffic Manager
- DNS-based load balancing
- Global traffic distribution
- **Routing Methods**:
  - Priority: Failover
  - Weighted: Distribute by percentage
  - Performance: Lowest latency
  - Geographic: Based on user location
  - Multivalue: Return multiple healthy endpoints
  - Subnet: Based on source IP subnet

## Private Connectivity

### Private Endpoint
- Private IP in VNet for Azure resource
- Traffic over Microsoft backbone
- Supported: Storage, SQL, Cosmos DB, Key Vault, etc.
- NSG rules don't apply (use RBAC)

### Private Link Service
- Expose your service via Private Link
- Customers connect with private endpoint
- No public IP needed
- Cross-tenant supported

### Service Endpoints
- Route optimization for Azure services
- Free (vs Private Endpoint cost)
- Traffic stays on Azure backbone
- Service-level, not resource-level
- Supported: Storage, SQL, Cosmos DB, Key Vault

**Service Endpoint Policies**:
- Restrict access to specific resources
- Example: Allow only prod storage accounts

## Network Monitoring

### Network Watcher
- Regional service (one per region)
- Network diagnostic and monitoring tools

**Diagnostics**:
- IP flow verify: Check NSG rules
- Next hop: Troubleshoot routing
- VPN diagnostics: Troubleshoot VPN gateway
- Connection troubleshoot: Test connectivity

**Monitoring**:
- Topology: Visualize network
- Connection monitor: Monitor connectivity
- NSG flow logs: Traffic analysis
- Traffic analytics: Insights from flow logs

### NSG Flow Logs
- Log allowed/denied traffic
- Version 2: Additional data (bytes, packets)
- Stored in storage account
- Retention 0-365 days
- Traffic Analytics for visualization

## Exam Tips

### Common Scenarios
- **Connect VNets in same region**: VNet peering (regional)
- **Connect VNets across regions**: Global VNet peering or VPN
- **Connect on-premises to Azure**: Site-to-Site VPN or ExpressRoute
- **Remote access for users**: Point-to-Site VPN
- **Secure management without public IP**: Azure Bastion
- **Layer 4 load balancing**: Azure Load Balancer
- **Layer 7 with WAF**: Application Gateway
- **Global load balancing**: Traffic Manager or Front Door
- **Private access to PaaS**: Private Endpoint
- **Keep traffic on Azure network**: Service Endpoint
- **Troubleshoot connectivity**: Network Watcher

### Key Differences
- **Peering vs VPN**: Peering = faster, cheaper, no encryption; VPN = encrypted, gateway needed
- **Private Endpoint vs Service Endpoint**: PE = specific resource, paid; SE = service-level, free
- **NSG vs Firewall**: NSG = basic filtering; Firewall = advanced, centralized, FQDN-based
- **Load Balancer vs App Gateway**: LB = Layer 4; App Gateway = Layer 7, URL routing
- **VPN vs ExpressRoute**: VPN = over internet, cheaper; ER = private, faster, more expensive
