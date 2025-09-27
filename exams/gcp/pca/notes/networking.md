# GCP Networking Fundamentals

## VPC (Virtual Private Cloud) Basics

### What is a VPC?
A global virtual network that spans all Google Cloud regions. Unlike other clouds, GCP VPCs are global by default with regional subnets.

### Key Differences from Other Clouds
- **Global by Default**: Single VPC spans multiple regions
- **Regional Subnets**: Subnets are regional (not zonal)
- **No Internet Gateway**: Different connectivity model
- **Implicit Router**: Automatic routing between subnets
- **Projects**: VPCs belong to projects (not accounts)

### VPC Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Default** | Auto-created with project | Quick start, simple setups |
| **Auto Mode** | Google manages subnet creation | Simplified management |
| **Custom Mode** | You control all subnets | Production, specific requirements |

### Default vs Custom VPC
| Feature | Default VPC | Custom VPC |
|---------|-------------|------------|
| **Subnets** | One per region (auto-created) | You define |
| **IP Ranges** | Google-managed | You choose |
| **Firewall Rules** | Default rules included | You configure |
| **Flexibility** | Limited | Full control |

## Subnets Deep Dive

### Subnet Characteristics
- **Regional**: Span all zones in a region
- **IP Ranges**: You define CIDR blocks
- **Expandable**: Can expand IP ranges (cannot shrink)
- **Secondary Ranges**: Additional IP ranges for containers/services

### Subnet Types by Purpose
```
Primary Range: 10.0.1.0/24 (VM instances)
Secondary Range 1: 10.1.0.0/16 (GKE pods)
Secondary Range 2: 10.2.0.0/16 (GKE services)
```

### Private Google Access
- **Purpose**: Access Google services without external IPs
- **Requirement**: Private Google Access enabled on subnet
- **Benefit**: Keep VMs private while accessing Cloud Storage, BigQuery, etc.
- **Note**: Only works for Google APIs, not other internet services

### Reserved IP Addresses
Google reserves:
- **Network address**: First IP in range
- **Gateway address**: Second IP in range  
- **Second-to-last IP**: Reserved by Google
- **Broadcast address**: Last IP in range

## IP Addressing & Connectivity

### Internal vs External IPs
| Type | Scope | Cost | Assignment |
|------|-------|------|------------|
| **Internal** | Within VPC | Free | Automatic |
| **External Ephemeral** | Internet | Charged | Dynamic |
| **External Static** | Internet | Charged (higher) | Reserved |

### External IP Types
- **Ephemeral**: Changes when instance stops/starts
- **Static**: Permanent assignment, survives instance lifecycle
- **Regional**: Used within a single region
- **Global**: Used for global load balancers

### Private Service Connect
- **Purpose**: Access services privately without external IPs
- **Benefits**: Security, performance, simplified networking
- **Use Cases**: Accessing Google services, third-party services
- **Types**: Google APIs, published services, custom services

## Firewall Rules

### Firewall Fundamentals
- **Stateful**: Return traffic automatically allowed
- **Applied to Instances**: Based on tags, service accounts
- **Priority**: Lower numbers = higher priority (0-65534)
- **Default Action**: Deny unless explicitly allowed

### Rule Components
| Component | Description | Example |
|-----------|-------------|---------|
| **Direction** | Ingress or Egress | Ingress |
| **Priority** | Rule precedence | 1000 |
| **Action** | Allow or Deny | Allow |
| **Targets** | What instances | web-servers tag |
| **Source/Destination** | Traffic origin/destination | 0.0.0.0/0 |
| **Protocols/Ports** | Traffic type | tcp:80,443 |

### Default Rules
```
default-allow-internal: Allow all traffic between VPC instances
default-allow-ssh: Allow SSH (tcp:22) from anywhere  
default-allow-rdp: Allow RDP (tcp:3389) from anywhere
default-allow-icmp: Allow ICMP from anywhere
```

### Best Practices
- Use specific tags for granular control
- Follow principle of least privilege
- Use service accounts instead of tags when possible
- Regular firewall rule audits
- Document rule purposes

## Load Balancing

### Load Balancer Types Overview
| Type | Scope | Layer | Use Case |
|------|-------|-------|----------|
| **Global External HTTP(S)** | Global | 7 | Global web applications |
| **Global External TCP Proxy** | Global | 4 | Global TCP applications |
| **Global External SSL Proxy** | Global | 4 | Global SSL applications |
| **Regional External HTTP(S)** | Regional | 7 | Regional web applications |
| **Regional External Network** | Regional | 4 | Regional TCP/UDP |
| **Internal HTTP(S)** | Regional | 7 | Internal web services |
| **Internal TCP/UDP** | Regional | 4 | Internal applications |

### Global vs Regional
| Scope | Benefits | Limitations |
|-------|----------|-------------|
| **Global** | Cross-region failover, CDN integration | HTTP(S)/TCP/SSL only |
| **Regional** | All protocols, lower latency | Single region only |

### Load Balancer Components
- **Frontend**: IP, port, protocol configuration
- **Backend Service**: Health checks, traffic distribution
- **Backend**: Instance groups or endpoints
- **Health Checks**: Monitor backend health

### URL Maps & Path-Based Routing
```
example.com/api/* → API backend service
example.com/static/* → Static content backend
example.com/* → Default web backend
```

## Cloud CDN

### What is Cloud CDN?
Global content delivery network that caches content at Google's edge locations worldwide.

### Key Features
- **Global Edge Network**: 100+ edge locations
- **Cache Modes**: CACHE_ALL_STATIC, USE_ORIGIN_HEADERS, FORCE_CACHE_ALL
- **Custom Headers**: Control caching behavior
- **Signed URLs**: Secure content delivery
- **Compression**: Automatic GZIP compression

### Cache Behavior
| Mode | Description | Use Case |
|------|-------------|----------|
| **USE_ORIGIN_HEADERS** | Follow origin cache headers | Dynamic sites with proper headers |
| **CACHE_ALL_STATIC** | Cache static content | Traditional web applications |
| **FORCE_CACHE_ALL** | Cache everything | Static sites, APIs with TTL |

### Origins Supported
- **Compute Engine instance groups**
- **Google Cloud Storage buckets**
- **External HTTP(S) origins**
- **Load balancer backends**

## VPC Connectivity Options

### VPC Peering
- **Purpose**: Connect two VPC networks privately
- **Scope**: Can peer across projects and organizations
- **Routing**: Automatic route exchange
- **Limitations**: Not transitive, no overlapping IPs
- **Cost**: No additional cost (just normal traffic charges)

### Shared VPC
- **Purpose**: Share VPC across multiple projects
- **Model**: Host project shares with service projects
- **Benefits**: Centralized network management
- **Use Case**: Enterprise multi-project environments
- **Permissions**: Network admin in host project

### VPC Network Peering vs Shared VPC
| Feature | VPC Peering | Shared VPC |
|---------|-------------|------------|
| **Administration** | Distributed | Centralized |
| **Projects** | Separate projects | Host + service projects |
| **IAM** | Independent | Unified network IAM |
| **Use Case** | Simple connectivity | Enterprise organization |

## Hybrid Connectivity

### Cloud VPN
- **Purpose**: Encrypted connection over public internet
- **Types**: Classic VPN, HA VPN
- **Tunnels**: IPSec tunnels
- **Redundancy**: HA VPN provides 99.99% SLA
- **Speed**: Up to 3 Gbps per tunnel

### Cloud Interconnect
#### Dedicated Interconnect
- **Purpose**: Physical connection to Google
- **Speeds**: 10 Gbps or 100 Gbps
- **Locations**: Google POPs (Points of Presence)
- **Benefits**: High bandwidth, predictable performance

#### Partner Interconnect
- **Purpose**: Connection through service provider
- **Speeds**: 50 Mbps to 50 Gbps
- **Availability**: More locations than Dedicated
- **Benefits**: Easier procurement, flexible bandwidth

### Connectivity Decision Matrix
| Requirement | Recommendation |
|-------------|----------------|
| **High bandwidth (>3 Gbps)** | Dedicated Interconnect |
| **Predictable performance** | Dedicated/Partner Interconnect |
| **Quick deployment** | Cloud VPN |
| **Cost-sensitive** | Cloud VPN |
| **Medium bandwidth** | Partner Interconnect |

## Cloud DNS

### What is Cloud DNS?
Scalable, reliable, managed authoritative DNS service running on Google's infrastructure.

### Key Features
- **100% uptime SLA**: Highly available service
- **Global network**: Anycast DNS servers
- **DNSSEC**: Domain name system security extensions
- **Private zones**: Internal DNS resolution
- **Logging**: Query logging for analysis

### Zone Types
| Type | Purpose | Visibility |
|------|---------|------------|
| **Public** | Internet-facing domains | Global |
| **Private** | Internal resolution | VPC networks |
| **Forwarding** | Forward queries | VPC networks |
| **Peering** | Cross-project resolution | VPC networks |

### DNS Policies
- **Inbound forwarding**: External DNS can query private zones
- **Outbound forwarding**: Forward specific domains to external DNS
- **Logging**: Enable query logging for monitoring

## Identity-Aware Proxy (IAP)

### What is IAP?
Central authentication and authorization service for applications accessed via HTTPS.

### Key Benefits
- **Zero-trust security**: Verify user and device
- **No VPN required**: Direct internet access with protection
- **Google integration**: Works with Google Cloud services
- **Fine-grained access**: Per-application permissions

### How IAP Works
1. User accesses application
2. IAP checks authentication
3. If authenticated, checks authorization
4. Allows or denies access based on policy

### Use Cases
- **Internal applications**: Secure employee access
- **Partner access**: Controlled external access
- **Development environments**: Secure staging/test environments

## Network Security

### Cloud Armor
- **Purpose**: DDoS protection and WAF (Web Application Firewall)
- **Features**: Rate limiting, geo-blocking, custom rules
- **Integration**: Works with Global Load Balancers
- **Adaptive Protection**: ML-based attack detection

### Private Service Connect
- **Purpose**: Access services without exposing to internet
- **Benefits**: Simplified networking, improved security
- **Types**: Published services, private endpoints
- **Use Cases**: SaaS access, partner connectivity

### VPC Flow Logs
- **Purpose**: Network monitoring and troubleshooting
- **Data**: Source, destination, ports, protocols, packets, bytes
- **Storage**: Export to Cloud Logging, BigQuery, Pub/Sub
- **Use Cases**: Security analysis, performance monitoring, compliance

## Best Practices

### VPC Design
- Use custom VPCs for production
- Plan IP address ranges carefully
- Use regional subnets for multi-zone deployments
- Enable Private Google Access for private instances
- Implement least-privilege firewall rules

### Load Balancing
- Use health checks for all backends
- Configure appropriate session affinity
- Implement proper SSL certificate management
- Monitor backend capacity and performance
- Use Cloud CDN for static content

### Security
- Implement defense in depth
- Use IAP for internal applications
- Enable VPC Flow Logs for monitoring
- Regular security rule audits
- Use Cloud Armor for public-facing applications

## Cost Optimization

### Traffic Optimization
- Use Premium vs Standard network tier appropriately
- Minimize cross-region traffic
- Use Cloud CDN to reduce origin traffic
- Optimize load balancer configuration
- Monitor network usage patterns

### Connectivity Optimization
- Choose appropriate VPN vs Interconnect
- Right-size Interconnect bandwidth
- Use regional resources to minimize traffic
- Implement efficient data transfer patterns

## Common Pitfalls

### VPC Issues
- Overlapping IP ranges preventing peering
- Forgetting to enable Private Google Access
- Incorrect firewall rule priorities
- Not planning for future IP growth
- Mixing auto-mode and custom-mode VPCs

### Load Balancer Issues
- Health check misconfigurations
- Backend service capacity planning
- SSL certificate management problems
- Incorrect session affinity settings
- Not using appropriate load balancer type

### Connectivity Problems
- VPN tunnel configuration errors
- Routing table misconfigurations
- Firewall blocking legitimate traffic
- DNS resolution issues
- Bandwidth bottlenecks