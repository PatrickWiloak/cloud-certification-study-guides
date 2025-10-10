# OCI Architect Associate - Architecture Design and Networking

## Table of Contents
- [Architecture Design Principles](#architecture-design-principles)
- [High Availability Architecture](#high-availability-architecture)
- [Disaster Recovery](#disaster-recovery)
- [VCN Architecture](#vcn-architecture)
- [Advanced Networking](#advanced-networking)
- [Load Balancing](#load-balancing)
- [Hybrid Cloud Connectivity](#hybrid-cloud-connectivity)
- [Network Security](#network-security)
- [Exam Tips](#exam-tips)

---

## Architecture Design Principles

### Well-Architected Framework

**Five Pillars**:

#### 1. Operational Excellence
**Principles**:
- Automate operations
- Monitor and observe systems
- Learn from failures
- Iterate and improve

**OCI Services**:
- Resource Manager (Infrastructure as Code)
- Monitoring and Logging
- Events and Notifications
- Cloud Guard

**Best Practices**:
- Use Infrastructure as Code (Terraform, Resource Manager)
- Implement comprehensive monitoring
- Automate deployments
- Document architecture and runbooks
- Practice incident response

#### 2. Security
**Principles**:
- Implement defense in depth
- Apply least privilege
- Encrypt data at rest and in transit
- Audit and monitor all activities

**OCI Security Layers**:
```
Physical Security (Data Centers)
    ↓
Network Security (VCN, Security Lists, NSGs)
    ↓
Identity Security (IAM, MFA, Federation)
    ↓
Application Security (WAF, DDoS)
    ↓
Data Security (Encryption, Vault)
    ↓
Monitoring (Audit Logs, Cloud Guard)
```

**Best Practices**:
- Enable MFA for all users
- Use Security Zones for critical workloads
- Encrypt all sensitive data
- Implement network segmentation
- Enable audit logging
- Regular security reviews

#### 3. Reliability
**Principles**:
- Design for failure
- Recover quickly
- Scale horizontally
- Test recovery procedures

**High Availability Design**:
```
Region: us-phoenix-1
├── AD-1
│   ├── FD-1: Web-1, App-1
│   ├── FD-2: Web-2, App-2
│   └── FD-3: Web-3, App-3
├── AD-2
│   ├── FD-1: Web-4, App-4
│   ├── FD-2: Web-5, App-5
│   └── FD-3: Web-6, App-6
└── AD-3
    ├── FD-1: Web-7, App-7, DB-Primary
    ├── FD-2: Web-8, App-8
    └── FD-3: Web-9, App-9, DB-Standby
```

**Best Practices**:
- Deploy across multiple ADs
- Use regional subnets
- Implement health checks
- Configure auto-recovery
- Regular backup testing
- Chaos engineering

#### 4. Performance Efficiency
**Principles**:
- Use appropriate resources
- Monitor and optimize
- Scale as needed
- Leverage serverless

**Performance Optimization**:
- **Compute**: Right-size instances, use Flex shapes
- **Storage**: Choose appropriate tier and IOPS
- **Network**: Use FastConnect for low latency
- **Database**: Use Autonomous Database, caching

**Best Practices**:
- Benchmark workloads
- Use CDN for static content
- Implement caching layers
- Regular performance testing
- Monitor and adjust resources

#### 5. Cost Optimization
**Principles**:
- Pay only for what you use
- Right-size resources
- Use commitment discounts
- Eliminate waste

**Cost Optimization Strategies**:
- Use Flex shapes for exact sizing
- Implement auto-scaling
- Use Monthly/Annual Flex for steady workloads
- Archive old data to lower tiers
- Tag resources for cost tracking
- Schedule non-production instances
- Use BYOL where applicable

### Design Patterns

#### Multi-Tier Architecture

**Three-Tier Pattern**:
```
Internet
    ↓
Load Balancer (Public Subnet)
    ↓
Web Tier (Private Subnet)
    ↓
Application Tier (Private Subnet)
    ↓
Database Tier (Private Subnet)
```

**Network Design**:
```
VCN: 10.0.0.0/16
├── Public Subnet: 10.0.1.0/24 (Load Balancer)
│   └── Route: 0.0.0.0/0 → Internet Gateway
├── Private Subnet 1: 10.0.10.0/24 (Web Tier)
│   └── Route: 0.0.0.0/0 → NAT Gateway
├── Private Subnet 2: 10.0.20.0/24 (App Tier)
│   └── Route: 0.0.0.0/0 → NAT Gateway
└── Private Subnet 3: 10.0.30.0/24 (Database)
    └── Route: OCI Services → Service Gateway
```

**Security Implementation**:
```
Load Balancer NSG:
- Ingress: 0.0.0.0/0:443 (HTTPS)
- Egress: Web-NSG:80

Web Tier NSG:
- Ingress: LB-NSG:80
- Egress: App-NSG:8080

App Tier NSG:
- Ingress: Web-NSG:8080
- Egress: DB-NSG:1521

Database NSG:
- Ingress: App-NSG:1521
- Egress: Service Gateway (backups)
```

#### Microservices Architecture

**Container-Based Pattern**:
```
API Gateway
    ↓
Container Engine for Kubernetes (OKE)
├── Service Mesh
├── Microservice 1 (Pods across ADs)
├── Microservice 2 (Pods across ADs)
└── Microservice 3 (Pods across ADs)
    ↓
Autonomous Database / NoSQL
```

**Key Components**:
- **OKE**: Managed Kubernetes for containers
- **API Gateway**: Entry point for APIs
- **Service Mesh**: Service-to-service communication
- **Container Registry**: Store container images
- **Autonomous Database**: Managed database backend

#### Event-Driven Architecture

**Serverless Pattern**:
```
Event Source (Object Storage, Queue, Stream)
    ↓
Events Service
    ↓
Functions (Serverless)
    ↓
Process and Store Results
```

**Use Cases**:
- Image processing on upload
- Real-time data processing
- Automated workflows
- IoT data ingestion

---

## High Availability Architecture

### Multi-AD Deployment

**Three-AD Strategy**:

**Active-Active Across ADs**:
```
Load Balancer (Regional)
    ↓
┌────────────┬────────────┬────────────┐
│   AD-1     │   AD-2     │   AD-3     │
├────────────┼────────────┼────────────┤
│  Web-1,2,3 │  Web-4,5,6 │  Web-7,8,9 │
│  App-1,2,3 │  App-4,5,6 │  App-7,8,9 │
│  DB-Primary│  DB-Standby│  DB-Backup │
└────────────┴────────────┴────────────┘
```

**Benefits**:
- Survives entire AD failure
- Automatic failover
- No performance degradation
- Maximum availability

**Implementation**:
```terraform
# Regional subnet spans all ADs
resource "oci_core_subnet" "web_subnet" {
  cidr_block     = "10.0.10.0/24"
  vcn_id         = oci_core_vcn.main.id
  compartment_id = var.compartment_id
  # Regional subnet (not AD-specific)
}

# Instance pool distributes across ADs
resource "oci_core_instance_pool" "web_pool" {
  compartment_id = var.compartment_id
  instance_configuration_id = oci_core_instance_configuration.web.id

  placement_configurations {
    availability_domain = data.oci_identity_availability_domain.ad1.name
    primary_subnet_id   = oci_core_subnet.web_subnet.id
  }

  placement_configurations {
    availability_domain = data.oci_identity_availability_domain.ad2.name
    primary_subnet_id   = oci_core_subnet.web_subnet.id
  }

  placement_configurations {
    availability_domain = data.oci_identity_availability_domain.ad3.name
    primary_subnet_id   = oci_core_subnet.web_subnet.id
  }

  size = 9  # 3 per AD
}
```

### Fault Domains

**FD Distribution**:
```
AD-1
├── FD-1: Web-1, App-1, DB-Replica-1
├── FD-2: Web-2, App-2, DB-Replica-2
└── FD-3: Web-3, App-3, DB-Replica-3
```

**Automatic FD Placement**:
- OCI automatically distributes instances across FDs
- Manual override available if needed
- Load balancers use all FDs

**Best Practices**:
- Let OCI auto-distribute across FDs
- Use regional subnets for automatic AD/FD distribution
- Deploy odd number of instances (3, 5, 7) for quorum
- Test failover scenarios

### Database High Availability

#### Data Guard

**Active Data Guard Configuration**:
```
Primary Database (AD-1)
    ↓ (Synchronous Replication)
Standby Database (AD-2)
    ↓ (Asynchronous Replication)
Far Sync Instance (AD-3)
```

**Features**:
- Automatic failover
- Zero data loss (Maximum Protection mode)
- Read replicas for reporting
- Rolling upgrades

**Creating Data Guard**:
```bash
oci db data-guard-association create \
  --database-id ocid1.database.oc1... \
  --creation-type ExistingDbSystem \
  --database-admin-password "Welcome123!" \
  --protection-mode MAXIMUM_PERFORMANCE \
  --transport-type ASYNC \
  --peer-db-system-id ocid1.dbsystem.oc1...
```

#### RAC (Real Application Clusters)

**Two-Node RAC**:
```
Load Balancer
    ↓
┌─────────────┬─────────────┐
│   RAC-1     │   RAC-2     │
│   (AD-1)    │   (AD-2)    │
└─────────────┴─────────────┘
        ↓           ↓
    Shared Storage (ASM)
```

**Benefits**:
- Active-Active clustering
- Automatic failover
- Load balancing
- High availability and scalability

**Use Cases**:
- Mission-critical applications
- High transaction volumes
- Zero downtime requirements

#### Autonomous Database HA

**Built-in Features**:
- Automatic failover (99.95% SLA)
- Rolling patches (zero downtime)
- Automatic backups
- Point-in-time recovery

**Configuration**:
```bash
oci db autonomous-database create \
  --compartment-id ocid1.compartment.oc1... \
  --db-name "proddb" \
  --cpu-core-count 2 \
  --data-storage-size-in-tbs 1 \
  --is-auto-scaling-enabled true \
  --is-free-tier false
```

---

## Disaster Recovery

### DR Strategies

**RTO and RPO**:
- **RTO** (Recovery Time Objective): How long to recover
- **RPO** (Recovery Point Objective): How much data loss acceptable

**DR Strategy Matrix**:
| Strategy | RPO | RTO | Cost | Complexity |
|----------|-----|-----|------|------------|
| Backup/Restore | Hours | Hours | Low | Low |
| Pilot Light | Minutes-Hours | Hours | Medium | Medium |
| Warm Standby | Minutes | Minutes | Medium-High | Medium |
| Hot Standby | Seconds | Seconds | High | High |

#### Backup and Restore

**Lowest Cost DR**:
```
Primary Region              DR Region
├── Active Resources       ├── No resources
├── Daily Backups          │   (create on demand)
└── Replicate Backups  ──→ └── Backup Storage
```

**RTO**: 4-24 hours
**RPO**: 24 hours

**Implementation**:
- Automated backups in primary region
- Cross-region backup replication
- Documented restore procedures
- Regular DR drills

#### Pilot Light

**Minimal Active DR**:
```
Primary Region              DR Region
├── Full Production        ├── Core Services Only
│   - Compute (Active)     │   - Database (Standby)
│   - Database (Primary)   │   - Minimal Compute
│   - Load Balancer        │   - IAC Templates Ready
└── Data Replication   ──→ └── Data Replicated
```

**RTO**: 1-4 hours
**RPO**: Minutes to 1 hour

**Implementation**:
- Database replication (Data Guard)
- Core infrastructure running
- Application servers stopped
- Infrastructure as Code ready to deploy
- Regular failover testing

#### Warm Standby

**Reduced Capacity DR**:
```
Primary Region              DR Region
├── Full Capacity          ├── Reduced Capacity
│   - 100% Resources       │   - 30-50% Resources
│   - Active Traffic       │   - No Traffic (Ready)
│   - Database (Primary)   │   - Database (Standby)
└── Real-time Sync     ──→ └── Real-time Sync
```

**RTO**: 10-30 minutes
**RPO**: Seconds to minutes

**Implementation**:
- Active-Passive setup
- Continuous replication
- Scaled-down DR infrastructure
- DNS or load balancer failover
- Auto-scaling configured for quick scale-up

#### Hot Standby (Active-Active)

**Full Capacity DR**:
```
Primary Region              DR Region
├── Full Capacity          ├── Full Capacity
│   - 100% Resources       │   - 100% Resources
│   - Active Traffic (50%) │   - Active Traffic (50%)
│   - Database (Primary)   │   - Database (Primary)
└── Bi-directional Sync ←→ └── Bi-directional Sync
```

**RTO**: 0-5 minutes (automatic)
**RPO**: Near-zero

**Implementation**:
- Multi-region active-active
- Traffic Manager for global routing
- Database bi-directional replication
- Conflict resolution strategy
- Automated failover

### Cross-Region Replication

#### Object Storage Replication

**Setup**:
```bash
oci os replication create \
  --bucket-name "source-bucket" \
  --destination-bucket-name "dr-bucket" \
  --destination-region "us-ashburn-1"
```

**Replication Policies**:
```json
{
  "name": "replicate-all",
  "destination": {
    "bucketName": "dr-bucket",
    "region": "us-ashburn-1"
  },
  "sourceObjectNameFilters": ["*"]
}
```

#### Block Volume Cross-Region Backup

**Backup Policy with Cross-Region Copy**:
```bash
# Create backup
oci bv backup create \
  --volume-id ocid1.volume.oc1.phx... \
  --display-name "volume-backup"

# Copy to DR region
oci bv backup copy \
  --backup-id ocid1.volumebackup.oc1.phx... \
  --destination-region "us-ashburn-1"
```

#### Database Cross-Region Standby

**Data Guard Across Regions**:
```bash
oci db data-guard-association create \
  --database-id ocid1.database.oc1.phx... \
  --creation-type NewDbSystem \
  --database-admin-password "Welcome123!" \
  --protection-mode MAXIMUM_PERFORMANCE \
  --transport-type ASYNC \
  --availability-domain "Uocm:IAD-AD-1" \
  --display-name "dr-standby" \
  --hostname "drdb01" \
  --subnet-id ocid1.subnet.oc1.iad...
```

### Failover Procedures

**DNS-Based Failover**:
```
Primary Region (Active)
    ↓
DNS (TTL: 60s) ─── Monitor Health
    ↓
DR Region (Standby) ← Failover if primary fails
```

**Traffic Manager Configuration**:
```terraform
resource "oci_dns_steering_policy" "failover" {
  compartment_id = var.compartment_id
  display_name   = "dr-failover"
  template       = "FAILOVER"

  rules {
    rule_type = "HEALTH"
    cases {
      answer_data {
        answer_condition = "answer.isHealthy"
        value            = 100
      }
    }
  }

  answers {
    name  = "primary"
    rdata = var.primary_lb_ip
    pool  = "primary-pool"
  }

  answers {
    name  = "dr"
    rdata = var.dr_lb_ip
    pool  = "dr-pool"
  }
}
```

---

## VCN Architecture

### VCN Design Patterns

#### Hub-and-Spoke Topology

**Centralized Services**:
```
Hub VCN (Shared Services)
├── Bastion Hosts
├── Monitoring Tools
├── Security Services
└── NAT/Service Gateways

Spoke VCNs (Workloads)
├── Production VCN   ─→ Peered to Hub
├── Development VCN  ─→ Peered to Hub
└── Testing VCN      ─→ Peered to Hub
```

**Benefits**:
- Centralized security and monitoring
- Shared services reduce costs
- Simplified management
- Isolated workloads

**Implementation**:
```bash
# Hub VCN
oci network vcn create \
  --cidr-block "10.0.0.0/16" \
  --display-name "hub-vcn" \
  --compartment-id ocid1.compartment.oc1...

# Spoke VCN (Production)
oci network vcn create \
  --cidr-block "10.1.0.0/16" \
  --display-name "prod-vcn" \
  --compartment-id ocid1.compartment.oc1...

# Spoke VCN (Development)
oci network vcn create \
  --cidr-block "10.2.0.0/16" \
  --display-name "dev-vcn" \
  --compartment-id ocid1.compartment.oc1...

# Local Peering Gateway (Hub)
oci network local-peering-gateway create \
  --vcn-id ocid1.vcn.oc1..hub \
  --display-name "hub-lpg" \
  --compartment-id ocid1.compartment.oc1...

# Local Peering Gateway (Spoke)
oci network local-peering-gateway create \
  --vcn-id ocid1.vcn.oc1..prod \
  --display-name "prod-lpg" \
  --compartment-id ocid1.compartment.oc1...

# Connect peering
oci network local-peering-gateway connect \
  --local-peering-gateway-id ocid1.localpeeringgateway..hub \
  --peer-id ocid1.localpeeringgateway..prod
```

#### Transit Routing with DRG

**DRG as Hub**:
```
                    DRG (Hub)
                       ↓
      ┌────────────────┼────────────────┐
      ↓                ↓                ↓
   VCN-1            VCN-2            VCN-3
      ↓                ↓                ↓
On-Premises ← FastConnect/VPN → DRG
```

**Advanced DRG Features** (DRG v2):
- Route tables for granular routing
- Import/export route distributions
- Connect VCNs, FastConnect, VPN, Remote Peering

**DRG Route Tables**:
```bash
# Create DRG route table
oci network drg-route-table create \
  --drg-id ocid1.drg.oc1... \
  --display-name "vcn-route-table"

# Add route rule
oci network drg-route-table-route-rule create \
  --drg-route-table-id ocid1.drgroutetable.oc1... \
  --destination "10.1.0.0/16" \
  --destination-type CIDR_BLOCK \
  --next-hop-drg-attachment-id ocid1.drgattachment.oc1...
```

### Subnet Design

**Best Practices**:

**1. Regional Subnets** (Recommended):
```
Regional Subnet: 10.0.1.0/24
├── Spans all ADs automatically
├── Instances distributed across ADs
└── Simplified failover
```

**2. Subnet Sizing**:
```
VCN: 10.0.0.0/16 (65,536 addresses)
├── Public Subnet: 10.0.0.0/24 (256 - 5 reserved = 251 usable)
├── Private Subnet 1: 10.0.10.0/24
├── Private Subnet 2: 10.0.20.0/24
├── Database Subnet: 10.0.30.0/24
└── Reserved for growth: 10.0.40.0/22 (1,024 addresses)
```

**Reserved Addresses per Subnet**:
- First address: Network address (.0)
- Second address: VCN router (.1)
- Third address: DNS (.2)
- Last two addresses: Reserved by OCI
- **Total reserved**: 5 addresses per subnet

**3. Security Layers**:
```
Public Subnet (DMZ):
- Load Balancers
- Bastion Hosts
- NAT Instances

Private Subnet (Application):
- Application Servers
- Web Servers
- Container Hosts

Private Subnet (Data):
- Databases
- File Storage Mount Targets
- Sensitive Data Services
```

---

## Advanced Networking

### Load Balancing

#### Load Balancer Architecture

**Regional Load Balancer**:
```
Internet
    ↓
Load Balancer (Regional - spans all ADs)
    ↓
┌──────────┬──────────┬──────────┐
│   AD-1   │   AD-2   │   AD-3   │
├──────────┼──────────┼──────────┤
│Backend-1 │Backend-4 │Backend-7 │
│Backend-2 │Backend-5 │Backend-8 │
│Backend-3 │Backend-6 │Backend-9 │
└──────────┴──────────┴──────────┘
```

**Key Features**:
- Automatic failover across ADs
- Health checks remove unhealthy backends
- SSL termination
- Session persistence
- Path-based routing

#### Load Balancer Types

**1. Load Balancer (Flexible)**:
- Layer 7 (HTTP/HTTPS) and Layer 4 (TCP)
- Bandwidth: 10 Mbps to 8 Gbps
- Advanced routing
- SSL/TLS termination
- Web Application Firewall (WAF) integration

**2. Network Load Balancer**:
- Layer 4 only (TCP/UDP/ICMP)
- Ultra-low latency (<1ms)
- Preserve source IP
- Regional HA
- Millions RPS capacity

**Comparison**:
| Feature | Load Balancer | Network Load Balancer |
|---------|--------------|----------------------|
| OSI Layer | 4 & 7 | 4 |
| Protocols | HTTP/S, TCP | TCP, UDP, ICMP |
| Source IP | NAT'd | Preserved |
| SSL Termination | Yes | No |
| Path Routing | Yes | No |
| Latency | Normal | Ultra-low |
| Use Case | Web apps | Gaming, IoT, high-perf |

#### Load Balancer Configuration

**Creating Load Balancer**:
```terraform
resource "oci_load_balancer_load_balancer" "web_lb" {
  compartment_id = var.compartment_id
  display_name   = "web-load-balancer"
  shape          = "flexible"

  shape_details {
    minimum_bandwidth_in_mbps = 10
    maximum_bandwidth_in_mbps = 100
  }

  subnet_ids = [
    oci_core_subnet.public_subnet_ad1.id,
    oci_core_subnet.public_subnet_ad2.id
  ]

  is_private = false
}
```

**Backend Set with Health Check**:
```terraform
resource "oci_load_balancer_backend_set" "web_backend" {
  load_balancer_id = oci_load_balancer_load_balancer.web_lb.id
  name             = "web-backend-set"
  policy           = "ROUND_ROBIN"  # or LEAST_CONNECTIONS, IP_HASH

  health_checker {
    protocol          = "HTTP"
    port              = 80
    url_path          = "/health"
    interval_ms       = 10000
    timeout_in_millis = 3000
    retries           = 3
    return_code       = 200
  }

  session_persistence_configuration {
    cookie_name      = "lb-session"
    disable_fallback = false
  }
}
```

**Listener Configuration**:
```terraform
resource "oci_load_balancer_listener" "https_listener" {
  load_balancer_id         = oci_load_balancer_load_balancer.web_lb.id
  name                     = "https-listener"
  default_backend_set_name = oci_load_balancer_backend_set.web_backend.name
  port                     = 443
  protocol                 = "HTTP"

  ssl_configuration {
    certificate_name        = "web-cert"
    verify_peer_certificate = false
    verify_depth            = 5
  }

  connection_configuration {
    idle_timeout_in_seconds = 300
  }
}
```

**Path-Based Routing**:
```terraform
resource "oci_load_balancer_path_route_set" "api_routes" {
  load_balancer_id = oci_load_balancer_load_balancer.web_lb.id
  name             = "api-routes"

  path_routes {
    path                      = "/api/*"
    path_match_type           = "PREFIX_MATCH"
    backend_set_name          = oci_load_balancer_backend_set.api_backend.name
  }

  path_routes {
    path                      = "/images/*"
    path_match_type           = "PREFIX_MATCH"
    backend_set_name          = oci_load_balancer_backend_set.static_backend.name
  }
}
```

#### SSL/TLS Configuration

**SSL Termination at Load Balancer**:
```
Client ← HTTPS → Load Balancer ← HTTP → Backend Servers
```

**Benefits**:
- Offload SSL processing from backends
- Centralized certificate management
- Inspect and modify requests

**End-to-End SSL**:
```
Client ← HTTPS → Load Balancer ← HTTPS → Backend Servers
```

**Benefits**:
- Maximum security
- Encrypted all the way to backend
- Compliance requirements

**Certificate Management**:
```bash
# Upload SSL certificate
oci lb certificate create \
  --certificate-name "web-cert" \
  --load-balancer-id ocid1.loadbalancer.oc1... \
  --ca-certificate-file ca-cert.pem \
  --private-key-file private-key.pem \
  --public-certificate-file public-cert.pem
```

### Traffic Management

**DNS Traffic Management** (Traffic Steering):

**Failover Policy**:
```
Priority 1: Primary Region (Active)
    ↓ (if unhealthy)
Priority 2: DR Region (Standby)
```

**Load Distribution Policy**:
```
50% → Region 1
50% → Region 2
```

**Geolocation Steering**:
```
US Traffic   → us-phoenix-1
EU Traffic   → eu-frankfurt-1
APAC Traffic → ap-tokyo-1
```

**Configuration**:
```terraform
resource "oci_dns_steering_policy" "geo_steering" {
  compartment_id = var.compartment_id
  display_name   = "geo-load-distribution"
  template       = "CUSTOM"

  rules {
    rule_type = "FILTER"

    default_answer_data {
      answer_condition = "answer.name == 'default'"
    }

    cases {
      answer_data {
        answer_condition = "query.client.address in (10.0.0.0/8)"
        should_keep      = true
      }
      case_condition = "query.client.geoKey == 'us'"
    }
  }
}
```

---

## Hybrid Cloud Connectivity

### FastConnect

**Definition**: Dedicated, private connection between on-premises and OCI.

#### FastConnect Models

**1. FastConnect via Partner**:
```
On-Premises
    ↓
Partner Network (Equinix, Megaport, etc.)
    ↓
OCI Region
```

**Benefits**:
- Easier setup (partner handles circuit)
- Flexible bandwidth (1, 2, 5, 10 Gbps)
- Multiple locations

**2. FastConnect Direct**:
```
On-Premises
    ↓
Cross-Connect (Physical)
    ↓
OCI FastConnect Location
    ↓
OCI Region
```

**Benefits**:
- Lowest latency
- Maximum control
- Dedicated fiber
- Higher bandwidth options (up to 10 Gbps per connection)

#### FastConnect Architecture

**Private Peering** (VCN Access):
```
On-Premises Network
    ↓
FastConnect (Private Virtual Circuit)
    ↓
Dynamic Routing Gateway (DRG)
    ↓
VCN(s)
```

**Public Peering** (Oracle Services):
```
On-Premises Network
    ↓
FastConnect (Public Virtual Circuit)
    ↓
Oracle Public Services
├── Object Storage
├── Oracle Services Network
└── OCI APIs
```

**Configuration Steps**:

**1. Create DRG**:
```bash
oci network drg create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "fastconnect-drg"
```

**2. Attach DRG to VCN**:
```bash
oci network drg-attachment create \
  --drg-id ocid1.drg.oc1... \
  --vcn-id ocid1.vcn.oc1... \
  --display-name "vcn-attachment"
```

**3. Create FastConnect Virtual Circuit**:
```bash
oci network virtual-circuit create \
  --compartment-id ocid1.compartment.oc1... \
  --type PRIVATE \
  --bandwidth-shape-name "1 Gbps" \
  --display-name "private-vc" \
  --gateway-id ocid1.drg.oc1... \
  --provider-service-id ocid1.providerservice.oc1...
```

**4. Configure BGP**:
```
Customer BGP ASN: 65001
Oracle BGP ASN: 31898
Customer BGP IP: 192.168.1.1/30
Oracle BGP IP: 192.168.1.2/30
```

**5. Update Route Tables**:
```bash
# Add route to on-premises network
oci network route-table update \
  --rt-id ocid1.routetable.oc1... \
  --route-rules '[{
    "destination":"192.168.0.0/16",
    "destinationType":"CIDR_BLOCK",
    "networkEntityId":"ocid1.drg.oc1..."
  }]'
```

#### FastConnect Redundancy

**Dual Circuit HA**:
```
On-Premises
├── Router 1 → FastConnect 1 → DRG
└── Router 2 → FastConnect 2 → DRG
```

**Best Practices**:
- Use two virtual circuits
- Different FastConnect locations
- Different routers on-premises
- BGP routing for automatic failover
- Monitor circuit health

### Site-to-Site VPN

**IPSec VPN over Internet**:
```
On-Premises
    ↓
IPSec VPN Tunnel (Encrypted)
    ↓
DRG
    ↓
VCN
```

**Use Cases**:
- Backup for FastConnect
- Low-cost hybrid connectivity
- Temporary connections
- Branch office connectivity

#### VPN Configuration

**1. Create CPE (Customer-Premises Equipment)**:
```bash
oci network cpe create \
  --compartment-id ocid1.compartment.oc1... \
  --ip-address "203.0.113.10" \
  --display-name "onprem-cpe"
```

**2. Create IPSec Connection**:
```bash
oci network ip-sec-connection create \
  --compartment-id ocid1.compartment.oc1... \
  --cpe-id ocid1.cpe.oc1... \
  --drg-id ocid1.drg.oc1... \
  --static-routes '["192.168.0.0/16"]' \
  --display-name "site-to-site-vpn"
```

**3. Configure On-Premises Router**:
```
# Oracle provides configuration for major vendors
# Cisco, Palo Alto, Juniper, etc.

# Example Cisco IOS Configuration
crypto ikev2 proposal OCI-PROPOSAL
 encryption aes-cbc-256 aes-cbc-128
 integrity sha256
 group 14

crypto ikev2 policy OCI-POLICY
 proposal OCI-PROPOSAL

crypto ikev2 keyring OCI-KEYRING
 peer OCI
  address 129.213.1.1
  pre-shared-key <your-shared-secret>

crypto ikev2 profile OCI-PROFILE
 match identity remote address 129.213.1.1 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local OCI-KEYRING

crypto ipsec transform-set OCI-TRANSFORM-SET esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile OCI-IPSEC-PROFILE
 set transform-set OCI-TRANSFORM-SET
 set ikev2-profile OCI-PROFILE
```

#### VPN High Availability

**Multiple Tunnels**:
```
OCI provides 2 tunnels per IPSec connection:
├── Tunnel 1: 129.213.1.1
└── Tunnel 2: 129.213.1.2

On-Premises Router:
├── Configure both tunnels
├── BGP for automatic failover
└── Active-Passive or Active-Active
```

**Multiple CPEs**:
```
On-Premises
├── CPE 1 → IPSec-1 → DRG
└── CPE 2 → IPSec-2 → DRG
```

### Hybrid Connectivity Best Practices

**1. Redundancy**:
- Primary: FastConnect
- Backup: VPN
- Multiple circuits/tunnels
- Different paths

**2. Monitoring**:
```bash
# FastConnect health metrics
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_fastconnect" \
  --query-text 'ConnectionState[1m].mean()'

# VPN tunnel status
oci network ip-sec-connection get-status \
  --ipsc-id ocid1.ipsecconnection.oc1...
```

**3. Routing**:
- Use BGP for dynamic routing
- Configure proper AS-PATH
- Implement route filtering
- Monitor BGP sessions

**4. Security**:
- Enable MACsec on FastConnect (if available)
- Use strong VPN encryption (AES-256)
- Implement network segmentation
- Monitor traffic patterns

---

## Network Security

### Security Lists vs NSGs

**Security Lists** (Subnet-level):
```
Subnet: 10.0.1.0/24
└── Security List: web-sl
    ├── Ingress: 0.0.0.0/0:443 ALLOW
    ├── Ingress: 10.0.0.0/16:ALL ALLOW
    └── Egress: 0.0.0.0/0:ALL ALLOW
```

**Network Security Groups** (VNIC-level):
```
NSG: web-nsg
├── Ingress: source=lb-nsg, port=80
├── Ingress: source=0.0.0.0/0, port=443
└── Egress: destination=app-nsg, port=8080

Applied to:
├── Instance-1 VNIC
├── Instance-2 VNIC
└── Instance-3 VNIC
```

**Best Practices**:
- Use Security Lists for subnet-wide defaults
- Use NSGs for application-specific rules
- NSGs can reference other NSGs (micro-segmentation)
- Limit security list rules (max 25 stateful rules per direction)

### Advanced Security Architecture

**Zero Trust Network**:
```
Internet
    ↓
WAF → Load Balancer (Public)
    ↓
Bastion Service (No direct SSH)
    ↓
DMZ Subnet (NSG: dmz-nsg)
    ↓
App Subnet (NSG: app-nsg) ← Only from dmz-nsg
    ↓
DB Subnet (NSG: db-nsg) ← Only from app-nsg
```

**Micro-Segmentation with NSGs**:
```terraform
# Web tier NSG
resource "oci_core_network_security_group" "web_nsg" {
  compartment_id = var.compartment_id
  vcn_id         = oci_core_vcn.main.id
  display_name   = "web-nsg"
}

# App tier NSG
resource "oci_core_network_security_group" "app_nsg" {
  compartment_id = var.compartment_id
  vcn_id         = oci_core_vcn.main.id
  display_name   = "app-nsg"
}

# Web → App rule
resource "oci_core_network_security_group_security_rule" "web_to_app" {
  network_security_group_id = oci_core_network_security_group.app_nsg.id
  direction                 = "INGRESS"
  protocol                  = "6"  # TCP
  source_type               = "NETWORK_SECURITY_GROUP"
  source                    = oci_core_network_security_group.web_nsg.id
  tcp_options {
    destination_port_range {
      min = 8080
      max = 8080
    }
  }
}
```

---

## Exam Tips

### Critical Concepts

**Architecture**:
- Multi-AD for HA (survives AD failure)
- FDs provide additional redundancy within AD
- Regional subnets preferred (automatic AD distribution)
- DR strategies: Backup/Restore < Pilot Light < Warm < Hot
- RTO = recovery time, RPO = data loss tolerance

**Networking**:
- VCN is regional, subnets can be regional or AD-specific
- 5 reserved IPs per subnet
- Public subnet = IGW route + public IPs
- Private subnet = NAT Gateway for outbound
- Service Gateway = private access to Oracle Services

**Load Balancing**:
- Load Balancer = Layer 7, SSL termination, path routing
- Network LB = Layer 4, ultra-low latency, preserves source IP
- Regional service (spans ADs)
- Health checks remove unhealthy backends

**Hybrid Connectivity**:
- FastConnect = dedicated private connection (1-10 Gbps)
- VPN = encrypted over internet (backup option)
- DRG = hub for VCN peering, FastConnect, VPN
- Private peering (VCN access) vs Public peering (Oracle Services)

### Common Exam Scenarios

**Q: Application needs to survive AD failure?**
A: Deploy across multiple ADs with load balancer

**Q: Lowest latency to on-premises?**
A: FastConnect Direct

**Q: Private access to Object Storage without internet?**
A: Service Gateway

**Q: Need to preserve source IP at backend?**
A: Network Load Balancer

**Q: RTO 1 hour, RPO 15 minutes, minimize cost?**
A: Pilot Light DR strategy

**Q: Multi-tier app security design?**
A: Use NSGs for each tier, allow only necessary traffic between tiers

---

## Summary

**High Availability**:
- Multi-AD deployment
- Fault domain distribution
- Load balancers with health checks
- Database replication (Data Guard, RAC)

**Disaster Recovery**:
- Define RTO/RPO requirements
- Choose appropriate DR strategy
- Cross-region replication
- Regular DR testing

**Networking**:
- VCN design (hub-spoke, multi-tier)
- Load balancing strategies
- Hybrid connectivity (FastConnect, VPN)
- Network security (Security Lists, NSGs)

---

**Next Steps**: Study Compute and Storage Solutions
