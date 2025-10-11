# VPC and Connectivity - GCP Professional Cloud Network Engineer

## Overview

VPC network design, hybrid connectivity, routing, and network architecture for the Professional Cloud Network Engineer certification.

This comprehensive guide covers:
- VPC architecture and design patterns
- Hybrid connectivity options (VPN, Interconnect, Peering)
- Advanced routing and BGP configuration
- Private connectivity to Google services
- Firewall hierarchies and network security
- DNS architecture and policy-based routing
- Real-world network architecture scenarios
- Complete configuration examples and troubleshooting

## VPC Fundamentals

### Network Types and Architecture

**Auto Mode VPC**:
- Automatically creates one subnet per region
- Subnets use predefined IP ranges (10.128.0.0/20, 10.132.0.0/20, etc.)
- New regions get subnets automatically
- Simple to set up but less flexible
- **NOT recommended for production** - difficult to integrate with on-premises networks

**Custom Mode VPC** (Recommended):
- Full control over subnet creation and IP ranges
- No automatic subnet creation
- Better for hybrid cloud architectures
- Prevents IP overlap with on-premises networks
- Required for enterprise deployments

**Key VPC Characteristics**:
- VPC is global - spans all GCP regions
- Subnets are regional - can span multiple zones within a region
- Each VPC has its own routing table
- Firewall rules are applied at VPC level
- One default VPC per project (can be deleted)

### Subnet Design and IP Planning

**CIDR Planning Best Practices**:
- Primary range: /8 to /29 (minimum /29 = 8 IP addresses)
- GCP reserves first 2 and last 2 IPs in each subnet
- Secondary ranges: For GKE pods and services
- Plan for 3-5 years of growth
- Avoid RFC1918 overlaps with on-premises
- Document IP allocation in spreadsheet/IPAM tool

**IP Range Recommendations**:
```
Production VPCs:
- 10.0.0.0/8 (for large enterprises)
- 172.16.0.0/12 (medium organizations)
- 192.168.0.0/16 (small deployments)

Development/Test:
- Separate non-overlapping ranges
- Typically smaller subnets

GKE Considerations:
- Primary range: Node IPs (/24 = 251 usable IPs = ~250 nodes)
- Secondary range 1: Pod IPs (/14 = 262,144 IPs)
- Secondary range 2: Service IPs (/20 = 4,096 IPs)
```

**Comprehensive VPC Creation Example**:
```bash
# Create custom VPC with regional routing
gcloud compute networks create production-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional \
  --description="Production VPC for multi-tier application"

# Create subnets with different purposes
# Web tier - public subnet
gcloud compute networks subnets create web-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24 \
  --description="Web tier subnet" \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-5-sec \
  --logging-flow-sampling=0.5 \
  --logging-metadata=include-all

# Application tier - private subnet
gcloud compute networks subnets create app-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24 \
  --description="Application tier subnet" \
  --enable-private-ip-google-access \
  --enable-flow-logs

# Database tier - private subnet
gcloud compute networks subnets create db-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.3.0/24 \
  --description="Database tier subnet" \
  --enable-private-ip-google-access \
  --enable-flow-logs \
  --purpose=PRIVATE

# GKE cluster subnet with secondary ranges
gcloud compute networks subnets create gke-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.10.0/24 \
  --secondary-range pods=10.4.0.0/14 \
  --secondary-range services=10.8.0.0/20 \
  --enable-private-ip-google-access \
  --enable-flow-logs

# Management subnet for bastion hosts and operations
gcloud compute networks subnets create mgmt-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.100.0/24 \
  --description="Management and operations subnet"

# Multi-region subnets for DR
gcloud compute networks subnets create app-subnet-dr \
  --network=production-vpc \
  --region=us-east1 \
  --range=10.1.2.0/24 \
  --enable-private-ip-google-access \
  --enable-flow-logs
```

**Terraform Example for VPC**:
```hcl
# terraform/vpc.tf
resource "google_compute_network" "production_vpc" {
  name                    = "production-vpc"
  auto_create_subnetworks = false
  routing_mode            = "REGIONAL"
  description             = "Production VPC for multi-tier application"
}

resource "google_compute_subnetwork" "web_subnet" {
  name          = "web-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = "us-central1"
  network       = google_compute_network.production_vpc.id

  log_config {
    aggregation_interval = "INTERVAL_5_SEC"
    flow_sampling        = 0.5
    metadata             = "INCLUDE_ALL_METADATA"
  }
}

resource "google_compute_subnetwork" "app_subnet" {
  name          = "app-subnet"
  ip_cidr_range = "10.0.2.0/24"
  region        = "us-central1"
  network       = google_compute_network.production_vpc.id

  private_ip_google_access = true

  log_config {
    aggregation_interval = "INTERVAL_5_SEC"
    flow_sampling        = 0.5
    metadata             = "INCLUDE_ALL_METADATA"
  }
}

resource "google_compute_subnetwork" "gke_subnet" {
  name          = "gke-subnet"
  ip_cidr_range = "10.0.10.0/24"
  region        = "us-central1"
  network       = google_compute_network.production_vpc.id

  private_ip_google_access = true

  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = "10.4.0.0/14"
  }

  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = "10.8.0.0/20"
  }

  log_config {
    aggregation_interval = "INTERVAL_5_SEC"
    flow_sampling        = 0.5
    metadata             = "INCLUDE_ALL_METADATA"
  }
}
```

### Secondary IP Ranges for GKE

**Why Secondary Ranges**:
- GKE uses IP aliasing (VPC-native clusters)
- Pods get routable IPs from VPC
- Better network performance (no NAT for pod traffic)
- Integration with VPC firewall rules
- Supports VPC peering and Shared VPC

**Sizing Calculations**:
```
Node IP range sizing:
- /24 subnet = 251 usable IPs
- Allow 250 nodes max per cluster
- For autoscaling, plan for peak capacity

Pod IP range sizing:
- Default: 110 pods per node (configurable with --max-pods-per-node)
- Formula: (max nodes) × (max pods per node) × (growth factor)
- Example: 250 nodes × 110 pods × 1.5 = 41,250 IPs needed
- Recommendation: /14 (262,144 IPs) for large clusters

Service IP range sizing:
- One IP per Kubernetes Service
- Typical: 100-1,000 services
- Recommendation: /20 (4,096 IPs) for most clusters
```

**GKE Cluster Creation with IP Ranges**:
```bash
gcloud container clusters create production-cluster \
  --region=us-central1 \
  --network=production-vpc \
  --subnetwork=gke-subnet \
  --cluster-secondary-range-name=pods \
  --services-secondary-range-name=services \
  --enable-ip-alias \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.0.0/28 \
  --max-pods-per-node=110 \
  --num-nodes=3 \
  --enable-autoscaling \
  --min-nodes=3 \
  --max-nodes=100
```

## Firewall Architecture and Security

### Firewall Rule Hierarchy

**Evaluation Order**:
1. **Hierarchical Firewall Policies** (Organization/Folder level) - evaluated first
2. **VPC Firewall Rules** (Network level) - evaluated second
3. **Implicit Rules** - default deny ingress, default allow egress

**Key Concepts**:
- Firewall rules are stateful (return traffic is automatically allowed)
- Rules have priority 0-65535 (lower number = higher priority)
- First matching rule is applied (either allow or deny)
- Both allow and deny rules can be created
- Rules can target by tag, service account, or all instances

### Hierarchical Firewall Policies

**Organization/Folder Level Policies**:
- Centrally managed by security teams
- Inherited by all VPCs in organization/folder
- Cannot be overridden by VPC-level rules
- Best for compliance and baseline security

```bash
# Create hierarchical firewall policy at organization level
gcloud compute firewall-policies create security-baseline \
  --organization=ORGANIZATION_ID \
  --description="Organization-wide security baseline"

# Add rule to deny all RDP from internet (higher priority)
gcloud compute firewall-policies rules create 100 \
  --firewall-policy=security-baseline \
  --organization=ORGANIZATION_ID \
  --action=deny \
  --direction=ingress \
  --src-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:3389 \
  --description="Deny RDP from internet"

# Add rule to deny all SSH from internet except from corporate IPs
gcloud compute firewall-policies rules create 200 \
  --firewall-policy=security-baseline \
  --organization=ORGANIZATION_ID \
  --action=deny \
  --direction=ingress \
  --src-ip-ranges=0.0.0.0/0 \
  --layer4-configs=tcp:22 \
  --description="Deny SSH from non-corporate IPs"

# Allow SSH from corporate IP ranges
gcloud compute firewall-policies rules create 150 \
  --firewall-policy=security-baseline \
  --organization=ORGANIZATION_ID \
  --action=allow \
  --direction=ingress \
  --src-ip-ranges=203.0.113.0/24,198.51.100.0/24 \
  --layer4-configs=tcp:22 \
  --description="Allow SSH from corporate IPs"

# Associate policy with folder
gcloud compute firewall-policies associations create \
  --firewall-policy=security-baseline \
  --folder=FOLDER_ID \
  --organization=ORGANIZATION_ID
```

### VPC Firewall Rules

**Comprehensive Firewall Strategy**:
```bash
# 1. Baseline deny-all ingress (lowest priority)
gcloud compute firewall-rules create deny-all-ingress \
  --network=production-vpc \
  --action=deny \
  --direction=ingress \
  --rules=all \
  --priority=65534 \
  --description="Deny all ingress traffic by default"

# 2. Allow health checks from Google load balancers
gcloud compute firewall-rules create allow-health-checks \
  --network=production-vpc \
  --allow=tcp \
  --source-ranges=35.191.0.0/16,130.211.0.0/22 \
  --target-tags=lb-backend \
  --priority=500 \
  --description="Allow health checks from Google LB"

# 3. Allow HTTP/HTTPS to web tier
gcloud compute firewall-rules create allow-web-traffic \
  --network=production-vpc \
  --allow=tcp:80,tcp:443 \
  --target-tags=web-server \
  --source-ranges=0.0.0.0/0 \
  --priority=1000 \
  --enable-logging \
  --description="Allow HTTP/HTTPS to web servers"

# 4. Allow internal communication between tiers
gcloud compute firewall-rules create allow-web-to-app \
  --network=production-vpc \
  --allow=tcp:8080 \
  --target-tags=app-server \
  --source-tags=web-server \
  --priority=1100 \
  --description="Allow web tier to app tier"

gcloud compute firewall-rules create allow-app-to-db \
  --network=production-vpc \
  --allow=tcp:3306,tcp:5432 \
  --target-tags=db-server \
  --source-tags=app-server \
  --priority=1200 \
  --description="Allow app tier to database tier"

# 5. Allow internal subnet communication
gcloud compute firewall-rules create allow-internal \
  --network=production-vpc \
  --allow=tcp:0-65535,udp:0-65535,icmp \
  --source-ranges=10.0.0.0/16 \
  --priority=2000 \
  --description="Allow all internal VPC communication"

# 6. SSH access from bastion/IAP
gcloud compute firewall-rules create allow-ssh-from-iap \
  --network=production-vpc \
  --allow=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --priority=1500 \
  --description="Allow SSH from Identity-Aware Proxy"

# 7. Deny egress to specific destinations (optional)
gcloud compute firewall-rules create deny-egress-to-internet \
  --network=production-vpc \
  --action=deny \
  --direction=egress \
  --rules=all \
  --destination-ranges=0.0.0.0/0 \
  --priority=65000 \
  --target-tags=no-internet \
  --description="Deny internet access for sensitive instances"
```

**Service Account Based Targeting** (Recommended):
```bash
# More secure than network tags - uses IAM identity
gcloud compute firewall-rules create allow-app-to-db-sa \
  --network=production-vpc \
  --allow=tcp:3306 \
  --target-service-accounts=db-sa@project.iam.gserviceaccount.com \
  --source-service-accounts=app-sa@project.iam.gserviceaccount.com \
  --priority=1000 \
  --description="Allow app service account to database service account"
```

**Terraform Example for Firewall Rules**:
```hcl
# terraform/firewall.tf
resource "google_compute_firewall" "deny_all_ingress" {
  name    = "deny-all-ingress"
  network = google_compute_network.production_vpc.name

  deny {
    protocol = "all"
  }

  direction   = "INGRESS"
  priority    = 65534
  description = "Deny all ingress traffic by default"
}

resource "google_compute_firewall" "allow_web_traffic" {
  name    = "allow-web-traffic"
  network = google_compute_network.production_vpc.name

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  direction     = "INGRESS"
  priority      = 1000
  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]

  log_config {
    metadata = "INCLUDE_ALL_METADATA"
  }
}

resource "google_compute_firewall" "allow_internal" {
  name    = "allow-internal"
  network = google_compute_network.production_vpc.name

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "icmp"
  }

  direction     = "INGRESS"
  priority      = 2000
  source_ranges = ["10.0.0.0/16"]
}
```

### Firewall Insights and Optimization

**Firewall Insights** - Identifies unused/redundant rules:
```bash
# Enable Firewall Insights
gcloud compute networks update production-vpc \
  --enable-firewall-insights

# List shadowed rules (rules that will never be hit)
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=global \
  --recommender=google.compute.firewall.Recommender \
  --filter="recommenderSubtype=SHADOWED_RULE"

# List overly permissive rules
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=global \
  --recommender=google.compute.firewall.Recommender \
  --filter="recommenderSubtype=ALLOW_RULE_TOO_PERMISSIVE"
```

**Firewall Logging and Analysis**:
```bash
# Enable logging on firewall rule
gcloud compute firewall-rules update allow-web-traffic \
  --enable-logging \
  --logging-metadata=include-all

# Query firewall logs in Cloud Logging
gcloud logging read "resource.type=gce_subnetwork \
  AND logName:compute.googleapis.com%2Ffirewall \
  AND jsonPayload.disposition=DENIED" \
  --limit=50 \
  --format=json
```

## Routing Architecture and Cloud Router

### Route Types and Priority

**Route Types in Order of Evaluation**:
1. **Subnet routes** - Auto-created for each subnet (priority N/A, cannot be changed)
2. **Static routes** - Manually created custom routes
3. **Dynamic routes** - Learned via BGP through Cloud Router
4. **Default route** - 0.0.0.0/0 to internet gateway (priority 1000)

**Route Priority**:
- Priority range: 0-65535 (lower number = higher priority)
- Default route priority: 1000
- Same destination, different priority: higher priority wins
- Same destination, same priority: ECMP load balancing across routes

### Dynamic Routing Modes

**Regional vs Global Dynamic Routing**:

**Regional Mode** (default):
- Cloud Router learns on-premises routes via BGP
- Routes only propagated to subnets in same region as Cloud Router
- More isolated, better for security
- Lower costs (no cross-region traffic charges)
- Recommended for most deployments

**Global Mode**:
- Routes propagated to all subnets in all regions
- Enables cross-region connectivity through on-premises
- Required for multi-region hybrid connectivity
- Higher costs for cross-region traffic
- Use case: Multi-region DR with on-premises backhaul

```bash
# Set VPC to regional dynamic routing (default)
gcloud compute networks update production-vpc \
  --bgp-routing-mode=regional

# Set VPC to global dynamic routing
gcloud compute networks update production-vpc \
  --bgp-routing-mode=global

# View current routing mode
gcloud compute networks describe production-vpc \
  --format="get(routingConfig.routingMode)"
```

### Static Routes

**Custom Static Route Configuration**:
```bash
# Route to on-premises via VPN tunnel
gcloud compute routes create route-to-onprem-dc1 \
  --network=production-vpc \
  --destination-range=192.168.0.0/16 \
  --next-hop-vpn-tunnel=vpn-tunnel-1 \
  --next-hop-vpn-tunnel-region=us-central1 \
  --priority=100 \
  --description="Route to on-premises DC1"

# Route via instance (virtual appliance)
gcloud compute routes create route-to-dmz \
  --network=production-vpc \
  --destination-range=172.16.0.0/16 \
  --next-hop-instance=firewall-instance \
  --next-hop-instance-zone=us-central1-a \
  --priority=200

# Route via internal load balancer (ILB as next hop)
gcloud compute routes create route-to-nva \
  --network=production-vpc \
  --destination-range=10.10.0.0/16 \
  --next-hop-ilb=nva-ilb \
  --next-hop-ilb-region=us-central1 \
  --priority=300

# Blackhole route (drop traffic)
gcloud compute routes create blackhole-route \
  --network=production-vpc \
  --destination-range=10.255.255.0/24 \
  --priority=1000

# Tag-based routing (route applies only to tagged instances)
gcloud compute routes create route-for-proxies \
  --network=production-vpc \
  --destination-range=0.0.0.0/0 \
  --next-hop-instance=proxy-instance \
  --next-hop-instance-zone=us-central1-a \
  --tags=use-proxy \
  --priority=500
```

**Route Troubleshooting**:
```bash
# List all routes in VPC
gcloud compute routes list \
  --filter="network:production-vpc" \
  --sort-by=priority

# Describe specific route
gcloud compute routes describe route-to-onprem-dc1

# List routes with specific next-hop
gcloud compute routes list \
  --filter="nextHopVpnTunnel:vpn-tunnel-1"

# View effective routes for an instance
gcloud compute instances describe INSTANCE_NAME \
  --zone=ZONE \
  --format="get(networkInterfaces[0].network)"
```

### Cloud Router and BGP Configuration

**Cloud Router Overview**:
- Enables dynamic routing using BGP
- Exchanges routes with on-premises routers
- Supports multiple BGP sessions
- Automatic failover and load balancing
- Required for HA VPN and Cloud Interconnect

**Cloud Router Creation and Configuration**:
```bash
# Create Cloud Router with private ASN
gcloud compute routers create production-router \
  --network=production-vpc \
  --region=us-central1 \
  --asn=65001 \
  --description="Production Cloud Router for hybrid connectivity"

# Create Cloud Router with custom advertisement
gcloud compute routers create production-router-custom \
  --network=production-vpc \
  --region=us-central1 \
  --asn=65001 \
  --advertisement-mode=custom \
  --set-advertisement-ranges=10.0.0.0/16,10.1.0.0/16 \
  --set-advertisement-groups=all_subnets

# Update router to advertise custom ranges
gcloud compute routers update production-router \
  --region=us-central1 \
  --advertisement-mode=custom \
  --set-advertisement-ranges=10.0.0.0/16,10.4.0.0/14 \
  --set-advertisement-groups=all_subnets

# Add BGP peer for VPN tunnel
gcloud compute routers add-bgp-peer production-router \
  --peer-name=onprem-peer-1 \
  --interface=if-tunnel-1 \
  --peer-ip-address=169.254.1.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --advertised-route-priority=100

# Add BGP peer with BFD (Bidirectional Forwarding Detection)
gcloud compute routers add-bgp-peer production-router \
  --peer-name=onprem-peer-2 \
  --interface=if-tunnel-2 \
  --peer-ip-address=169.254.2.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --enable-bfd \
  --bfd-session-initialization-mode=ACTIVE \
  --bfd-min-transmit-interval=1000 \
  --bfd-min-receive-interval=1000 \
  --bfd-multiplier=5
```

**BGP Configuration Details**:

**ASN (Autonomous System Number)**:
- Private ASN range: 64512-65534, 4200000000-4294967294
- Must be unique per Cloud Router
- Must not conflict with on-premises ASN
- Cannot be changed after creation

**Route Advertisement Modes**:
```bash
# Default mode - advertises all subnets in VPC
gcloud compute routers update production-router \
  --region=us-central1 \
  --advertisement-mode=default

# Custom mode - advertise specific ranges
gcloud compute routers update production-router \
  --region=us-central1 \
  --advertisement-mode=custom \
  --set-advertisement-ranges=10.0.0.0/16 \
  --set-advertisement-groups=all_subnets

# Advertisement groups options:
# - all_subnets: All subnet ranges
# - all_vpc_subnets: All VPC subnets (including peered)
# - all_peer_vpc_subnets: Subnets from peered VPCs
```

**BGP Route Priority and MED**:
```bash
# Set BGP route priority (affects inbound traffic)
# Lower priority value = more preferred
gcloud compute routers add-bgp-peer production-router \
  --peer-name=primary-peer \
  --interface=if-tunnel-1 \
  --peer-ip-address=169.254.1.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --advertised-route-priority=100

gcloud compute routers add-bgp-peer production-router \
  --peer-name=backup-peer \
  --interface=if-tunnel-2 \
  --peer-ip-address=169.254.2.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --advertised-route-priority=200
```

**BGP Monitoring and Troubleshooting**:
```bash
# Get BGP peer status
gcloud compute routers get-status production-router \
  --region=us-central1 \
  --format="get(result.bgpPeerStatus)"

# View advertised routes
gcloud compute routers get-status production-router \
  --region=us-central1 \
  --format="table(
    result.bestRoutes.destRange,
    result.bestRoutes.nextHopIp,
    result.bestRoutes.priority
  )"

# View learned routes
gcloud compute routers get-status production-router \
  --region=us-central1 \
  --format="table(
    result.bestRoutesForRouter.destRange,
    result.bestRoutesForRouter.nextHopIp
  )"
```

**Terraform Example for Cloud Router**:
```hcl
# terraform/router.tf
resource "google_compute_router" "production_router" {
  name    = "production-router"
  network = google_compute_network.production_vpc.id
  region  = "us-central1"

  bgp {
    asn               = 65001
    advertise_mode    = "CUSTOM"
    advertised_groups = ["ALL_SUBNETS"]

    advertised_ip_ranges {
      range = "10.0.0.0/16"
    }

    advertised_ip_ranges {
      range = "10.4.0.0/14"
      description = "GKE pod IP range"
    }
  }
}

resource "google_compute_router_peer" "onprem_peer_1" {
  name                      = "onprem-peer-1"
  router                    = google_compute_router.production_router.name
  region                    = google_compute_router.production_router.region
  peer_ip_address          = "169.254.1.2"
  peer_asn                 = 65002
  advertised_route_priority = 100
  interface                = google_compute_router_interface.tunnel_1.name

  bfd {
    session_initialization_mode = "ACTIVE"
    min_transmit_interval       = 1000
    min_receive_interval        = 1000
    multiplier                  = 5
  }
}
```

## VPC Connectivity and Peering

### VPC Peering

**Overview**:
- Private connectivity between VPC networks
- Works across projects and organizations
- No bandwidth charges for traffic
- Low latency (same as intra-VPC)
- Useful for shared services and multi-tenant architectures

**VPC Peering Configuration**:
```bash
# Create peering from VPC1 to VPC2
gcloud compute networks peerings create peer-vpc1-to-vpc2 \
  --network=vpc1 \
  --peer-project=project-2 \
  --peer-network=vpc2 \
  --auto-create-routes \
  --import-custom-routes \
  --export-custom-routes

# Create reverse peering (required for bidirectional connectivity)
gcloud compute networks peerings create peer-vpc2-to-vpc1 \
  --network=vpc2 \
  --peer-project=project-1 \
  --peer-network=vpc1 \
  --auto-create-routes \
  --import-custom-routes \
  --export-custom-routes

# Update peering to exchange subnet routes
gcloud compute networks peerings update peer-vpc1-to-vpc2 \
  --network=vpc1 \
  --import-subnet-routes-with-public-ip \
  --export-subnet-routes-with-public-ip

# List peerings
gcloud compute networks peerings list \
  --network=vpc1

# Delete peering
gcloud compute networks peerings delete peer-vpc1-to-vpc2 \
  --network=vpc1
```

**VPC Peering Capabilities**:

**Custom Route Exchange**:
```bash
# Enable custom route import/export (for static routes)
gcloud compute networks peerings update peer-vpc1-to-vpc2 \
  --network=vpc1 \
  --import-custom-routes \
  --export-custom-routes

# Disable custom route exchange
gcloud compute networks peerings update peer-vpc1-to-vpc2 \
  --network=vpc1 \
  --no-import-custom-routes \
  --no-export-custom-routes
```

**Terraform Example for VPC Peering**:
```hcl
# terraform/vpc_peering.tf
resource "google_compute_network_peering" "peering1" {
  name         = "peer-vpc1-to-vpc2"
  network      = google_compute_network.vpc1.self_link
  peer_network = google_compute_network.vpc2.self_link

  import_custom_routes = true
  export_custom_routes = true

  import_subnet_routes_with_public_ip = true
  export_subnet_routes_with_public_ip = true
}

resource "google_compute_network_peering" "peering2" {
  name         = "peer-vpc2-to-vpc1"
  network      = google_compute_network.vpc2.self_link
  peer_network = google_compute_network.vpc1.self_link

  import_custom_routes = true
  export_custom_routes = true
}
```

**VPC Peering Limitations** (IMPORTANT for exam):

1. **No Transitive Peering**:
   - If VPC-A peers with VPC-B, and VPC-B peers with VPC-C
   - VPC-A cannot reach VPC-C through VPC-B
   - Must create direct peering from VPC-A to VPC-C
   - Workaround: Use Shared VPC or multi-NIC instances

2. **IP Address Overlap**:
   - Subnet IP ranges cannot overlap between peered VPCs
   - Primary and secondary ranges both checked
   - Peering will fail if overlap detected

3. **Peering Limits**:
   - Max 25 peerings per VPC network
   - For more connectivity, use Shared VPC

4. **Routing Limitations**:
   - Only subnet routes automatically exchanged
   - Static routes exchanged only if custom route exchange enabled
   - Dynamic routes (BGP) NOT exchanged through peering
   - Cannot export/import default route (0.0.0.0/0)

5. **Firewall Rules**:
   - Each VPC maintains its own firewall rules
   - Must configure firewall rules on both sides
   - Use source IP ranges from peered VPC

**VPC Peering Use Cases**:
- Microservices across multiple VPCs
- Shared services (DNS, monitoring, logging)
- Multi-tenant architectures
- Organization boundary separation
- Dev/Staging/Prod environment isolation with shared services

### Shared VPC

**Architecture Overview**:
- **Host Project**: Contains the VPC network and subnets
- **Service Projects**: Attached projects that use host VPC
- Centralized network management
- Decentralized application/resource management
- Supports multiple service projects per host

**Shared VPC Benefits**:
- Centralized network administration (network admins)
- Separated permissions (app owners vs network owners)
- Shared networking resources (firewall rules, routes)
- Better IP address management
- Cost optimization (shared Cloud NAT, VPN, Interconnect)

**Shared VPC Configuration**:
```bash
# Enable Shared VPC on host project
gcloud compute shared-vpc enable host-project-id

# Attach service project to host project
gcloud compute shared-vpc associated-projects add service-project-id \
  --host-project=host-project-id

# List associated service projects
gcloud compute shared-vpc associated-projects list host-project-id

# Detach service project
gcloud compute shared-vpc associated-projects remove service-project-id \
  --host-project=host-project-id

# Disable Shared VPC (must detach all service projects first)
gcloud compute shared-vpc disable host-project-id
```

**IAM Roles for Shared VPC**:

**Host Project Roles**:
```bash
# Grant Shared VPC Admin role (can enable/manage Shared VPC)
gcloud organizations add-iam-policy-binding ORG_ID \
  --member='user:network-admin@example.com' \
  --role='roles/compute.xpnAdmin'

# Grant Network Admin role (can manage VPC networks)
gcloud projects add-iam-policy-binding host-project-id \
  --member='user:network-admin@example.com' \
  --role='roles/compute.networkAdmin'
```

**Service Project Roles**:
```bash
# Grant Network User role at subnet level (recommended - least privilege)
gcloud compute networks subnets add-iam-policy-binding app-subnet \
  --member='user:app-developer@example.com' \
  --role='roles/compute.networkUser' \
  --region=us-central1 \
  --project=host-project-id

# Grant Network User role at host project level (broader access)
gcloud projects add-iam-policy-binding host-project-id \
  --member='serviceAccount:PROJECT_NUMBER@cloudservices.gserviceaccount.com' \
  --role='roles/compute.networkUser'

# Grant Network User role to service account (for GKE, GCE, etc.)
gcloud compute networks subnets add-iam-policy-binding gke-subnet \
  --member='serviceAccount:service-PROJECT_NUMBER@container-engine-robot.iam.gserviceaccount.com' \
  --role='roles/compute.networkUser' \
  --region=us-central1 \
  --project=host-project-id

# Grant Security Admin role (for firewall rule management from service project)
gcloud projects add-iam-policy-binding host-project-id \
  --member='user:security-admin@example.com' \
  --role='roles/compute.securityAdmin'
```

**Complete Shared VPC Setup Example**:
```bash
# Step 1: Enable Shared VPC on host project
gcloud compute shared-vpc enable corp-networking-host

# Step 2: Create VPC in host project
gcloud compute networks create shared-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional \
  --project=corp-networking-host

# Step 3: Create subnets for different teams
gcloud compute networks subnets create finance-subnet \
  --network=shared-vpc \
  --region=us-central1 \
  --range=10.10.0.0/24 \
  --enable-private-ip-google-access \
  --project=corp-networking-host

gcloud compute networks subnets create marketing-subnet \
  --network=shared-vpc \
  --region=us-central1 \
  --range=10.11.0.0/24 \
  --enable-private-ip-google-access \
  --project=corp-networking-host

# Step 4: Attach service projects
gcloud compute shared-vpc associated-projects add finance-app-project \
  --host-project=corp-networking-host

gcloud compute shared-vpc associated-projects add marketing-app-project \
  --host-project=corp-networking-host

# Step 5: Grant subnet-level permissions to service projects
gcloud compute networks subnets add-iam-policy-binding finance-subnet \
  --member='serviceAccount:PROJECT_NUMBER@cloudservices.gserviceaccount.com' \
  --role='roles/compute.networkUser' \
  --region=us-central1 \
  --project=corp-networking-host

gcloud compute networks subnets add-iam-policy-binding marketing-subnet \
  --member='serviceAccount:PROJECT_NUMBER@cloudservices.gserviceaccount.com' \
  --role='roles/compute.networkUser' \
  --region=us-central1 \
  --project=corp-networking-host
```

**Terraform Example for Shared VPC**:
```hcl
# terraform/shared_vpc.tf

# Enable Shared VPC on host project
resource "google_compute_shared_vpc_host_project" "host" {
  project = "corp-networking-host"
}

# Attach service projects
resource "google_compute_shared_vpc_service_project" "finance" {
  host_project    = google_compute_shared_vpc_host_project.host.project
  service_project = "finance-app-project"
}

resource "google_compute_shared_vpc_service_project" "marketing" {
  host_project    = google_compute_shared_vpc_host_project.host.project
  service_project = "marketing-app-project"
}

# Grant subnet IAM permissions
resource "google_compute_subnetwork_iam_member" "finance_subnet_user" {
  project    = "corp-networking-host"
  region     = "us-central1"
  subnetwork = google_compute_subnetwork.finance_subnet.name
  role       = "roles/compute.networkUser"
  member     = "serviceAccount:${var.finance_project_number}@cloudservices.gserviceaccount.com"
}
```

**Shared VPC Design Patterns**:

**Pattern 1: Team-Based Isolation**:
- One subnet per team/department
- Subnet-level IAM permissions
- Team autonomy within their subnet
- Centralized network policies

**Pattern 2: Environment-Based Isolation**:
- Separate subnets for dev/staging/prod
- Different service projects per environment
- Consistent networking across environments
- Environment-specific firewall rules

**Pattern 3: Application-Based Isolation**:
- Subnets per application tier (web/app/db)
- Multiple service projects sharing same subnets
- Fine-grained access control
- Shared infrastructure components

**Shared VPC vs VPC Peering**:

| Feature | Shared VPC | VPC Peering |
|---------|-----------|-------------|
| Management | Centralized | Distributed |
| Projects | One host, multiple service | Independent |
| IP overlap | Not allowed | Not allowed |
| Routing | Shared routing table | Separate routing |
| Firewall | Centralized | Independent |
| IAM control | Granular (subnet-level) | Network-level |
| Use case | Organization with central IT | Cross-org/independent teams |
| Billing | Separate per project | Separate per project |
| Max connections | Unlimited service projects | Max 25 peerings |

## Hybrid Connectivity

### Hybrid Connectivity Options Comparison

| Option | Bandwidth | SLA | Latency | Encryption | Cost | Use Case |
|--------|-----------|-----|---------|------------|------|----------|
| **HA VPN** | 1.5-3 Gbps per tunnel | 99.99% | Higher | IPSec (built-in) | Low | General hybrid connectivity |
| **Classic VPN** | 1.5-3 Gbps | 99.9% | Higher | IPSec (built-in) | Low | Legacy, migrate to HA VPN |
| **Dedicated Interconnect** | 10/100 Gbps | 99.9-99.99% | Low | No (use MACsec) | High | High bandwidth, low latency |
| **Partner Interconnect** | 50 Mbps - 50 Gbps | Varies | Low | No (use VPN over Interconnect) | Medium | Flexible bandwidth |
| **Direct Peering** | 10 Gbps min | No SLA | Low | No | Low | Google services (not VPC) |
| **Carrier Peering** | Varies | No SLA | Medium | No | Varies | Google services via ISP |

### Cloud VPN

**VPN Types**:
- **HA VPN**: Recommended, 99.99% SLA, requires two tunnels
- **Classic VPN**: Legacy, 99.9% SLA, being deprecated

### HA VPN (High Availability VPN)

**Overview**:
- 99.99% SLA when configured correctly
- Regional service (two interfaces in same region)
- Requires two tunnels to on-premises
- Supports BGP for dynamic routing
- Redundant tunnel to redundant on-premises devices
- Each tunnel: 1.5-3 Gbps (up to 6 Gbps total)

**HA VPN Topologies**:

**Topology 1: HA VPN to Two On-Premises VPN Devices**:
```
GCP HA VPN Gateway          On-Premises
Interface 0 ----tunnel-1----> Device 1
Interface 1 ----tunnel-2----> Device 2

Provides 99.99% SLA (recommended)
```

**Topology 2: HA VPN to One On-Premises VPN Device**:
```
GCP HA VPN Gateway          On-Premises
Interface 0 ----tunnel-1----> Device 1 (Interface 0)
Interface 1 ----tunnel-2----> Device 1 (Interface 1)

Provides 99.99% SLA if on-prem device has redundant interfaces
```

**Complete HA VPN Configuration**:

```bash
# 1. Create HA VPN gateway
gcloud compute vpn-gateways create ha-vpn-gateway \
  --network=production-vpc \
  --region=us-central1

# 2. Get HA VPN gateway IP addresses
gcloud compute vpn-gateways describe ha-vpn-gateway \
  --region=us-central1 \
  --format="get(vpnInterfaces)"

# Output example:
# interface 0: 203.0.113.1
# interface 1: 203.0.113.2

# 3. Create Cloud Router
gcloud compute routers create ha-vpn-router \
  --network=production-vpc \
  --region=us-central1 \
  --asn=65001

# 4. Create external VPN gateway (represents on-premises device)
gcloud compute external-vpn-gateways create onprem-gateway \
  --interfaces=0=198.51.100.1,1=198.51.100.2

# 5. Create VPN tunnel 1 (interface 0 to on-prem device 1)
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-external-gateway=onprem-gateway \
  --peer-external-gateway-interface=0 \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SECRET_1 \
  --router=ha-vpn-router \
  --vpn-gateway=ha-vpn-gateway \
  --interface=0

# 6. Create VPN tunnel 2 (interface 1 to on-prem device 2)
gcloud compute vpn-tunnels create tunnel-2 \
  --peer-external-gateway=onprem-gateway \
  --peer-external-gateway-interface=1 \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SECRET_2 \
  --router=ha-vpn-router \
  --vpn-gateway=ha-vpn-gateway \
  --interface=1

# 7. Create router interface for tunnel 1
gcloud compute routers add-interface ha-vpn-router \
  --interface-name=if-tunnel-1 \
  --ip-address=169.254.0.1 \
  --mask-length=30 \
  --vpn-tunnel=tunnel-1 \
  --region=us-central1

# 8. Create router interface for tunnel 2
gcloud compute routers add-interface ha-vpn-router \
  --interface-name=if-tunnel-2 \
  --ip-address=169.254.1.1 \
  --mask-length=30 \
  --vpn-tunnel=tunnel-2 \
  --region=us-central1

# 9. Add BGP peer for tunnel 1
gcloud compute routers add-bgp-peer ha-vpn-router \
  --peer-name=bgp-peer-1 \
  --interface=if-tunnel-1 \
  --peer-ip-address=169.254.0.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --advertised-route-priority=100

# 10. Add BGP peer for tunnel 2
gcloud compute routers add-bgp-peer ha-vpn-router \
  --peer-name=bgp-peer-2 \
  --interface=if-tunnel-2 \
  --peer-ip-address=169.254.1.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --advertised-route-priority=100
```

**Terraform Example for HA VPN**:
```hcl
# terraform/ha_vpn.tf

# HA VPN Gateway
resource "google_compute_ha_vpn_gateway" "ha_gateway" {
  name    = "ha-vpn-gateway"
  network = google_compute_network.production_vpc.id
  region  = "us-central1"
}

# External VPN Gateway (on-premises)
resource "google_compute_external_vpn_gateway" "onprem_gateway" {
  name            = "onprem-gateway"
  redundancy_type = "TWO_IPS_REDUNDANCY"

  interface {
    id         = 0
    ip_address = "198.51.100.1"
  }

  interface {
    id         = 1
    ip_address = "198.51.100.2"
  }
}

# Cloud Router
resource "google_compute_router" "ha_vpn_router" {
  name    = "ha-vpn-router"
  network = google_compute_network.production_vpc.id
  region  = "us-central1"

  bgp {
    asn = 65001
  }
}

# VPN Tunnel 1
resource "google_compute_vpn_tunnel" "tunnel1" {
  name                            = "tunnel-1"
  region                          = "us-central1"
  vpn_gateway                     = google_compute_ha_vpn_gateway.ha_gateway.id
  peer_external_gateway           = google_compute_external_vpn_gateway.onprem_gateway.id
  peer_external_gateway_interface = 0
  shared_secret                   = var.shared_secret_1
  router                          = google_compute_router.ha_vpn_router.id
  vpn_gateway_interface           = 0
}

# VPN Tunnel 2
resource "google_compute_vpn_tunnel" "tunnel2" {
  name                            = "tunnel-2"
  region                          = "us-central1"
  vpn_gateway                     = google_compute_ha_vpn_gateway.ha_gateway.id
  peer_external_gateway           = google_compute_external_vpn_gateway.onprem_gateway.id
  peer_external_gateway_interface = 1
  shared_secret                   = var.shared_secret_2
  router                          = google_compute_router.ha_vpn_router.id
  vpn_gateway_interface           = 1
}

# Router Interface 1
resource "google_compute_router_interface" "if_tunnel1" {
  name       = "if-tunnel-1"
  router     = google_compute_router.ha_vpn_router.name
  region     = "us-central1"
  ip_range   = "169.254.0.1/30"
  vpn_tunnel = google_compute_vpn_tunnel.tunnel1.name
}

# Router Interface 2
resource "google_compute_router_interface" "if_tunnel2" {
  name       = "if-tunnel-2"
  router     = google_compute_router.ha_vpn_router.name
  region     = "us-central1"
  ip_range   = "169.254.1.1/30"
  vpn_tunnel = google_compute_vpn_tunnel.tunnel2.name
}

# BGP Peer 1
resource "google_compute_router_peer" "bgp_peer1" {
  name                      = "bgp-peer-1"
  router                    = google_compute_router.ha_vpn_router.name
  region                    = "us-central1"
  peer_ip_address          = "169.254.0.2"
  peer_asn                 = 65002
  advertised_route_priority = 100
  interface                = google_compute_router_interface.if_tunnel1.name
}

# BGP Peer 2
resource "google_compute_router_peer" "bgp_peer2" {
  name                      = "bgp-peer-2"
  router                    = google_compute_router.ha_vpn_router.name
  region                    = "us-central1"
  peer_ip_address          = "169.254.1.2"
  peer_asn                 = 65002
  advertised_route_priority = 100
  interface                = google_compute_router_interface.if_tunnel2.name
}
```

**VPN Monitoring and Troubleshooting**:
```bash
# Check VPN tunnel status
gcloud compute vpn-tunnels describe tunnel-1 \
  --region=us-central1 \
  --format="get(status,detailedStatus)"

# View VPN gateway details
gcloud compute vpn-gateways describe ha-vpn-gateway \
  --region=us-central1

# Check BGP session status
gcloud compute routers get-status ha-vpn-router \
  --region=us-central1 \
  --format="table(result.bgpPeerStatus[].name,result.bgpPeerStatus[].status,result.bgpPeerStatus[].state)"

# View learned routes from on-premises
gcloud compute routers get-status ha-vpn-router \
  --region=us-central1 \
  --format="table(result.bestRoutesForRouter[].destRange,result.bestRoutesForRouter[].nextHopIp)"

# Monitor VPN logs
gcloud logging read "resource.type=vpn_gateway \
  AND resource.labels.gateway_name=ha-vpn-gateway" \
  --limit=50 \
  --format=json

# Verify tunnel health
gcloud compute operations list \
  --filter="operationType:compute.vpnTunnels.*"
```

### Classic VPN (Legacy)

**Classic VPN vs HA VPN**:
- 99.9% SLA (vs 99.99% for HA VPN)
- Single external IP address
- Static or dynamic routing (BGP)
- Being deprecated - migrate to HA VPN

```bash
# Create Classic VPN gateway (legacy - for reference only)
gcloud compute target-vpn-gateways create classic-vpn-gateway \
  --network=production-vpc \
  --region=us-central1

# Reserve static IP
gcloud compute addresses create vpn-ip \
  --region=us-central1

# Create forwarding rules (ESP, UDP 500, UDP 4500)
gcloud compute forwarding-rules create vpn-rule-esp \
  --region=us-central1 \
  --address=vpn-ip \
  --ip-protocol=ESP \
  --target-vpn-gateway=classic-vpn-gateway

# Note: Migrate to HA VPN for production workloads
```

### Cloud Interconnect

**Cloud Interconnect Overview**:
- Extends on-premises network to GCP VPC
- Lower latency than VPN
- Higher bandwidth (10 Gbps to 100 Gbps)
- Not encrypted by default (use MACsec or VPN over Interconnect)
- Two types: Dedicated Interconnect and Partner Interconnect

### Dedicated Interconnect

**Overview**:
- Direct physical connection to Google network
- 10 Gbps or 100 Gbps connections
- Requires colocation facility near Google Point of Presence (PoP)
- 99.9% or 99.99% SLA (with redundant connections)
- Best for large enterprises with high bandwidth needs

**Dedicated Interconnect Configuration**:
```bash
# 1. Order Dedicated Interconnect (done through Console/sales)
# 2. Receive Letter of Authorization (LOA) from Google
# 3. Provide LOA to colocation provider
# 4. After physical connection is established, create VLAN attachment

# Create VLAN attachment for Dedicated Interconnect
gcloud compute interconnects attachments create vlan-attachment-1 \
  --interconnect=my-interconnect \
  --router=interconnect-router \
  --region=us-central1 \
  --vlan=100 \
  --candidate-subnets=169.254.100.0/29

# Create Cloud Router for Interconnect
gcloud compute routers create interconnect-router \
  --network=production-vpc \
  --region=us-central1 \
  --asn=65001

# Add BGP peer for VLAN attachment
gcloud compute routers add-bgp-peer interconnect-router \
  --peer-name=onprem-interconnect-peer \
  --interface=if-vlan-100 \
  --peer-ip-address=169.254.100.2 \
  --peer-asn=65002 \
  --region=us-central1 \
  --advertised-route-priority=100
```

### Partner Interconnect

**Overview**:
- Connect to GCP through supported service provider
- 50 Mbps to 50 Gbps per connection
- No need for colocation facility
- More flexible capacity (can start small, scale up)
- Good for organizations without direct Google PoP access

**Partner Interconnect Configuration**:
```bash
# 1. Create VLAN attachment with service provider
gcloud compute interconnects attachments create partner-attachment-1 \
  --router=interconnect-router \
  --region=us-central1 \
  --edge-availability-domain=availability-domain-1 \
  --admin-enabled

# 2. Get pairing key
gcloud compute interconnects attachments describe partner-attachment-1 \
  --region=us-central1 \
  --format="get(pairingKey)"

# 3. Provide pairing key to service provider
# 4. After provider activates connection, create BGP session

# Create router interface for VLAN attachment
gcloud compute routers add-interface interconnect-router \
  --interface-name=if-partner-1 \
  --interconnect-attachment=partner-attachment-1 \
  --region=us-central1

# Add BGP peer
gcloud compute routers add-bgp-peer interconnect-router \
  --peer-name=partner-peer-1 \
  --interface=if-partner-1 \
  --peer-ip-address=169.254.200.2 \
  --peer-asn=65003 \
  --region=us-central1
```

**Terraform Example for Partner Interconnect**:
```hcl
# terraform/partner_interconnect.tf

resource "google_compute_router" "interconnect_router" {
  name    = "interconnect-router"
  network = google_compute_network.production_vpc.id
  region  = "us-central1"

  bgp {
    asn = 65001
  }
}

resource "google_compute_interconnect_attachment" "partner_attachment" {
  name                     = "partner-attachment-1"
  edge_availability_domain = "AVAILABILITY_DOMAIN_1"
  type                     = "PARTNER"
  router                   = google_compute_router.interconnect_router.id
  region                   = "us-central1"
  admin_enabled            = true
}

resource "google_compute_router_interface" "partner_interface" {
  name                    = "if-partner-1"
  router                  = google_compute_router.interconnect_router.name
  region                  = "us-central1"
  interconnect_attachment = google_compute_interconnect_attachment.partner_attachment.id
}

resource "google_compute_router_peer" "partner_peer" {
  name                      = "partner-peer-1"
  router                    = google_compute_router.interconnect_router.name
  region                    = "us-central1"
  peer_ip_address          = "169.254.200.2"
  peer_asn                 = 65003
  advertised_route_priority = 100
  interface                = google_compute_router_interface.partner_interface.name
}
```

### Direct Peering and Carrier Peering

**Direct Peering**:
- Exchange BGP routes directly with Google
- Access Google Workspace, YouTube, Google APIs (NOT VPC resources)
- Requires ASN and colocation at Google peering facility
- No SLA, no Google Cloud support
- Free (no egress charges for peered traffic)

**Carrier Peering**:
- Connect through ISP that peers with Google
- Access Google services (NOT VPC resources)
- No SLA
- Easier than Direct Peering (no colocation required)

**Key Difference**: Peering is for Google services, NOT for VPC connectivity. Use VPN or Interconnect for VPC access.

### Interconnect Monitoring
```bash
# View interconnect status
gcloud compute interconnects describe my-interconnect

# View VLAN attachment status
gcloud compute interconnects attachments describe vlan-attachment-1 \
  --region=us-central1

# Check BGP session for interconnect
gcloud compute routers get-status interconnect-router \
  --region=us-central1 \
  --format="table(result.bgpPeerStatus[].name,result.bgpPeerStatus[].status)"

# Monitor interconnect logs
gcloud logging read "resource.type=interconnect" \
  --limit=50 \
  --format=json
```

## Private Google Access and Private Service Connect

### Private Google Access

**Overview**:
- Allows instances without external IPs to access Google APIs
- Enables access to Cloud Storage, BigQuery, and other Google services
- Traffic stays within Google network (doesn't traverse internet)
- Configured per subnet
- No additional cost

**Enable Private Google Access**:
```bash
# Enable on existing subnet
gcloud compute networks subnets update app-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access

# Create subnet with Private Google Access enabled
gcloud compute networks subnets create private-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.5.0/24 \
  --enable-private-ip-google-access

# Verify Private Google Access status
gcloud compute networks subnets describe app-subnet \
  --region=us-central1 \
  --format="get(privateIpGoogleAccess)"
```

**Private Google Access Domain Names**:
```
*.googleapis.com (restricted.googleapis.com)
- 199.36.153.8/30 (VIP range)
- Used for most Google APIs

private.googleapis.com
- 199.36.153.4/30
- Recommended for Private Service Connect
```

**DNS Configuration for Private Google Access**:
```bash
# Create DNS zone for googleapis.com
gcloud dns managed-zones create google-apis \
  --dns-name=googleapis.com. \
  --networks=production-vpc \
  --visibility=private \
  --description="Private zone for Google APIs"

# Add A record for restricted.googleapis.com
gcloud dns record-sets create restricted.googleapis.com. \
  --zone=google-apis \
  --type=A \
  --ttl=300 \
  --rrdatas=199.36.153.8,199.36.153.9,199.36.153.10,199.36.153.11

# Add CNAME for wildcard
gcloud dns record-sets create *.googleapis.com. \
  --zone=google-apis \
  --type=CNAME \
  --ttl=300 \
  --rrdatas=restricted.googleapis.com.
```

### Private Service Connect

**Private Service Connect Overview**:
- Access Google services and third-party services using private IP addresses
- More secure than Private Google Access
- Service-specific endpoints
- Supports VPC Service Controls
- Two types: For Google APIs, For service producers

**Private Service Connect for Google APIs**:
```bash
# Create Private Service Connect endpoint for all Google APIs
gcloud compute addresses create psc-endpoint \
  --global \
  --purpose=PRIVATE_SERVICE_CONNECT \
  --addresses=10.0.254.1 \
  --network=production-vpc

gcloud compute forwarding-rules create psc-all-apis \
  --global \
  --network=production-vpc \
  --address=psc-endpoint \
  --target-google-apis-bundle=all-apis

# Create endpoint for specific API (e.g., Cloud Storage)
gcloud compute forwarding-rules create psc-storage \
  --global \
  --network=production-vpc \
  --address=10.0.254.2 \
  --target-google-apis-bundle=storage-api

# List available API bundles
gcloud compute forwarding-rules create --help | grep target-google-apis-bundle
```

**Private Service Connect for Service Producers**:
```bash
# Consumer side: Create endpoint to connect to producer service
gcloud compute addresses create psc-consumer-endpoint \
  --region=us-central1 \
  --purpose=GCE_ENDPOINT \
  --subnet=app-subnet \
  --addresses=10.0.2.100

gcloud compute forwarding-rules create psc-consumer-connection \
  --region=us-central1 \
  --network=production-vpc \
  --address=psc-consumer-endpoint \
  --target-service-attachment=projects/PRODUCER_PROJECT/regions/us-central1/serviceAttachments/ATTACHMENT_NAME

# Producer side: Create service attachment
gcloud compute service-attachments create my-service-attachment \
  --region=us-central1 \
  --producer-forwarding-rule=my-ilb \
  --connection-preference=ACCEPT_AUTOMATIC \
  --nat-subnets=nat-subnet \
  --consumer-reject-list=PROJECT_ID1,PROJECT_ID2
```

**Terraform Example for Private Service Connect**:
```hcl
# terraform/private_service_connect.tf

resource "google_compute_global_address" "psc_endpoint" {
  name          = "psc-endpoint"
  purpose       = "PRIVATE_SERVICE_CONNECT"
  address_type  = "INTERNAL"
  address       = "10.0.254.1"
  network       = google_compute_network.production_vpc.id
}

resource "google_compute_global_forwarding_rule" "psc_all_apis" {
  name                  = "psc-all-apis"
  target                = "all-apis"
  network               = google_compute_network.production_vpc.id
  ip_address            = google_compute_global_address.psc_endpoint.id
  load_balancing_scheme = ""
}
```

## Cloud NAT (Network Address Translation)

**Overview**:
- Provides outbound internet access for instances without external IPs
- Regional resource (serves instances in same region)
- Supports static or automatically allocated external IPs
- Port allocation per VM
- Logging and monitoring support
- Manual or automatic NAT IP address allocation

**Cloud NAT Configuration**:
```bash
# Create Cloud Router (required for Cloud NAT)
gcloud compute routers create nat-router \
  --network=production-vpc \
  --region=us-central1

# Create Cloud NAT with automatic IP allocation
gcloud compute routers nats create my-nat \
  --router=nat-router \
  --region=us-central1 \
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips

# Create Cloud NAT with manual IP allocation
gcloud compute addresses create nat-ip-1 --region=us-central1
gcloud compute addresses create nat-ip-2 --region=us-central1

gcloud compute routers nats create my-nat-manual \
  --router=nat-router \
  --region=us-central1 \
  --nat-all-subnet-ip-ranges \
  --nat-external-ip-pool=nat-ip-1,nat-ip-2

# Create Cloud NAT for specific subnets only
gcloud compute routers nats create my-nat-selective \
  --router=nat-router \
  --region=us-central1 \
  --nat-custom-subnet-ip-ranges=app-subnet,db-subnet \
  --auto-allocate-nat-external-ips

# Enable Cloud NAT logging
gcloud compute routers nats update my-nat \
  --router=nat-router \
  --region=us-central1 \
  --enable-logging \
  --log-filter=ALL
```

**Cloud NAT Port Allocation**:
```bash
# Configure minimum ports per VM
gcloud compute routers nats update my-nat \
  --router=nat-router \
  --region=us-central1 \
  --min-ports-per-vm=128 \
  --max-ports-per-vm=65536

# Enable dynamic port allocation
gcloud compute routers nats update my-nat \
  --router=nat-router \
  --region=us-central1 \
  --enable-dynamic-port-allocation \
  --min-ports-per-vm=32 \
  --max-ports-per-vm=65536
```

**Port Allocation Calculation**:
```
Default: 64 ports per VM (static allocation)
Min: 32 ports per VM (dynamic allocation)
Max: 65536 ports per VM

Ports per IP: 64,512 (65,536 - 1,024 reserved)

Example:
- 100 VMs × 64 ports = 6,400 ports needed
- 6,400 ÷ 64,512 = 0.1 NAT IPs (round up to 1 IP)

For high-connection workloads:
- 100 VMs × 1,024 ports = 102,400 ports
- 102,400 ÷ 64,512 = 1.59 NAT IPs (round up to 2 IPs)
```

**Terraform Example for Cloud NAT**:
```hcl
# terraform/cloud_nat.tf

resource "google_compute_router" "nat_router" {
  name    = "nat-router"
  network = google_compute_network.production_vpc.id
  region  = "us-central1"
}

resource "google_compute_router_nat" "nat" {
  name                               = "my-nat"
  router                             = google_compute_router.nat_router.name
  region                             = google_compute_router.nat_router.region
  nat_ip_allocate_option             = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"

  log_config {
    enable = true
    filter = "ERRORS_ONLY"
  }

  min_ports_per_vm = 128
  max_ports_per_vm = 65536

  enable_dynamic_port_allocation      = true
  enable_endpoint_independent_mapping = false
}

# Cloud NAT with manual IP allocation
resource "google_compute_address" "nat_ips" {
  count  = 2
  name   = "nat-ip-${count.index + 1}"
  region = "us-central1"
}

resource "google_compute_router_nat" "nat_manual" {
  name   = "my-nat-manual"
  router = google_compute_router.nat_router.name
  region = google_compute_router.nat_router.region

  nat_ip_allocate_option = "MANUAL_ONLY"
  nat_ips                = google_compute_address.nat_ips[*].self_link

  source_subnetwork_ip_ranges_to_nat = "LIST_OF_SUBNETWORKS"

  subnetwork {
    name                    = google_compute_subnetwork.app_subnet.id
    source_ip_ranges_to_nat = ["ALL_IP_RANGES"]
  }

  log_config {
    enable = true
    filter = "ALL"
  }
}
```

**Cloud NAT Monitoring**:
```bash
# View Cloud NAT configuration
gcloud compute routers nats describe my-nat \
  --router=nat-router \
  --region=us-central1

# Query Cloud NAT logs
gcloud logging read "resource.type=nat_gateway \
  AND resource.labels.router_id=nat-router" \
  --limit=50 \
  --format=json

# Monitor NAT port usage (Cloud Monitoring)
gcloud monitoring time-series list \
  --filter='metric.type="router.googleapis.com/nat/allocated_ports"' \
  --format=json
```

## Cloud DNS

### DNS Zone Types

**Public Zones**:
- DNS records visible on public internet
- Requires domain ownership verification
- Supports DNSSEC
- Anycast from Google's global network

**Private Zones**:
- DNS records only visible within specified VPCs
- Internal name resolution
- No domain ownership required
- Can override public DNS

### Cloud DNS Configuration

**Public DNS Zone**:
```bash
# Create public DNS zone
gcloud dns managed-zones create public-zone \
  --dns-name=example.com. \
  --description="Public DNS zone for example.com"

# Add A record
gcloud dns record-sets create www.example.com. \
  --zone=public-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=203.0.113.1

# Add MX record
gcloud dns record-sets create example.com. \
  --zone=public-zone \
  --type=MX \
  --ttl=3600 \
  --rrdatas="10 mail.example.com."

# Add CNAME record
gcloud dns record-sets create blog.example.com. \
  --zone=public-zone \
  --type=CNAME \
  --ttl=300 \
  --rrdatas=www.example.com.
```

**Private DNS Zone**:
```bash
# Create private DNS zone
gcloud dns managed-zones create private-zone \
  --dns-name=internal.example.com. \
  --networks=production-vpc,development-vpc \
  --visibility=private \
  --description="Private DNS zone for internal resources"

# Add A record
gcloud dns record-sets create db.internal.example.com. \
  --zone=private-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=10.0.3.10

# Add SRV record (for service discovery)
gcloud dns record-sets create _http._tcp.internal.example.com. \
  --zone=private-zone \
  --type=SRV \
  --ttl=300 \
  --rrdatas="10 10 80 web1.internal.example.com.,10 10 80 web2.internal.example.com."
```

### DNS Peering

**Overview**:
- Allows one VPC to resolve DNS queries using another VPC's private zones
- Cross-project DNS resolution
- Useful for shared services architecture

```bash
# Create DNS peering zone in consumer VPC
gcloud dns managed-zones create peering-zone \
  --dns-name=shared.internal. \
  --networks=consumer-vpc \
  --visibility=private \
  --target-network=projects/producer-project/global/networks/producer-vpc \
  --description="DNS peering to producer VPC"
```

### DNS Forwarding

**Overview**:
- Forward DNS queries to alternative name servers
- Integrate with on-premises DNS
- Selective forwarding based on domain

**Inbound DNS Forwarding** (on-prem queries GCP DNS):
```bash
# Create inbound DNS policy
gcloud dns policies create inbound-policy \
  --networks=production-vpc \
  --enable-inbound-forwarding \
  --description="Allow on-premises to query GCP DNS"

# Inbound forwarder addresses (auto-assigned)
# 10.0.0.2 (first IP in subnet range + 2)
```

**Outbound DNS Forwarding** (GCP queries on-prem DNS):
```bash
# Create forwarding zone
gcloud dns managed-zones create onprem-forward-zone \
  --dns-name=corp.example.com. \
  --networks=production-vpc \
  --visibility=private \
  --forwarding-targets=192.168.1.10,192.168.1.11 \
  --description="Forward corporate domain queries to on-premises"

# Create DNS server policy with alternative name servers
gcloud dns policies create outbound-policy \
  --networks=production-vpc \
  --alternative-name-servers=192.168.1.10,192.168.1.11 \
  --description="Forward DNS queries to on-premises DNS"
```

### DNSSEC

**Enable DNSSEC**:
```bash
# Enable DNSSEC on zone
gcloud dns managed-zones update public-zone \
  --dnssec-state=on

# Get DS records for domain registrar
gcloud dns managed-zones describe public-zone \
  --format="get(dnssecConfig.dsRecords)"
```

**Terraform Example for DNS**:
```hcl
# terraform/dns.tf

# Private DNS zone
resource "google_dns_managed_zone" "private_zone" {
  name        = "private-zone"
  dns_name    = "internal.example.com."
  visibility  = "private"
  description = "Private DNS zone"

  private_visibility_config {
    networks {
      network_url = google_compute_network.production_vpc.id
    }
  }
}

# DNS A record
resource "google_dns_record_set" "database" {
  managed_zone = google_dns_managed_zone.private_zone.name
  name         = "db.internal.example.com."
  type         = "A"
  ttl          = 300
  rrdatas      = ["10.0.3.10"]
}

# DNS forwarding zone
resource "google_dns_managed_zone" "forward_zone" {
  name        = "onprem-forward-zone"
  dns_name    = "corp.example.com."
  visibility  = "private"
  description = "Forward queries to on-premises DNS"

  private_visibility_config {
    networks {
      network_url = google_compute_network.production_vpc.id
    }
  }

  forwarding_config {
    target_name_servers {
      ipv4_address = "192.168.1.10"
    }
    target_name_servers {
      ipv4_address = "192.168.1.11"
    }
  }
}

# DNS policy for inbound forwarding
resource "google_dns_policy" "inbound_policy" {
  name                      = "inbound-policy"
  enable_inbound_forwarding = true

  networks {
    network_url = google_compute_network.production_vpc.id
  }
}
```

## Real-World Network Architecture Scenarios

### Scenario 1: Enterprise Hybrid Cloud with Multi-Tier Application

**Requirements**:
- On-premises datacenter connectivity
- Three-tier application (web, app, database)
- High availability (99.99% SLA)
- Private communication between tiers
- Secure access to Google APIs
- Centralized network management

**Architecture**:
```
On-Premises DC (192.168.0.0/16)
    |
    | Redundant VPN Tunnels
    |
GCP VPC (10.0.0.0/16) - Custom Mode, Regional Routing
    |
    +-- Web Subnet (10.0.1.0/24) - us-central1
    |   - External HTTP(S) Load Balancer
    |   - Cloud Armor for DDoS protection
    |   - Instances with external IPs
    |
    +-- App Subnet (10.0.2.0/24) - us-central1
    |   - Internal TCP/UDP Load Balancer
    |   - No external IPs
    |   - Private Google Access enabled
    |   - Cloud NAT for outbound
    |
    +-- DB Subnet (10.0.3.0/24) - us-central1
    |   - Cloud SQL with private IP
    |   - No external IPs
    |   - Restricted firewall rules
    |
    +-- Management Subnet (10.0.100.0/24) - us-central1
        - Bastion host with Identity-Aware Proxy
        - Cloud Operations monitoring
```

**Complete Implementation**:
```bash
# 1. Create VPC
gcloud compute networks create enterprise-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional

# 2. Create subnets
gcloud compute networks subnets create web-subnet \
  --network=enterprise-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24 \
  --enable-flow-logs

gcloud compute networks subnets create app-subnet \
  --network=enterprise-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24 \
  --enable-private-ip-google-access \
  --enable-flow-logs

gcloud compute networks subnets create db-subnet \
  --network=enterprise-vpc \
  --region=us-central1 \
  --range=10.0.3.0/24 \
  --enable-private-ip-google-access \
  --enable-flow-logs

gcloud compute networks subnets create mgmt-subnet \
  --network=enterprise-vpc \
  --region=us-central1 \
  --range=10.0.100.0/24

# 3. Configure HA VPN
gcloud compute vpn-gateways create enterprise-vpn \
  --network=enterprise-vpc \
  --region=us-central1

gcloud compute routers create enterprise-router \
  --network=enterprise-vpc \
  --region=us-central1 \
  --asn=65001

gcloud compute external-vpn-gateways create onprem-gateway \
  --interfaces=0=198.51.100.1,1=198.51.100.2

# Create tunnels and BGP peers (abbreviated)
# ... tunnel creation commands from HA VPN section ...

# 4. Configure Cloud NAT for app tier
gcloud compute routers nats create app-nat \
  --router=enterprise-router \
  --region=us-central1 \
  --nat-custom-subnet-ip-ranges=app-subnet \
  --auto-allocate-nat-external-ips

# 5. Create firewall rules
# Deny all ingress (baseline)
gcloud compute firewall-rules create deny-all-ingress \
  --network=enterprise-vpc \
  --action=deny \
  --direction=ingress \
  --rules=all \
  --priority=65534

# Allow HTTP/HTTPS to web tier
gcloud compute firewall-rules create allow-web \
  --network=enterprise-vpc \
  --allow=tcp:80,tcp:443 \
  --target-tags=web-server \
  --source-ranges=0.0.0.0/0 \
  --priority=1000

# Allow web to app tier
gcloud compute firewall-rules create allow-web-to-app \
  --network=enterprise-vpc \
  --allow=tcp:8080 \
  --target-tags=app-server \
  --source-tags=web-server \
  --priority=1100

# Allow app to database tier
gcloud compute firewall-rules create allow-app-to-db \
  --network=enterprise-vpc \
  --allow=tcp:3306 \
  --target-tags=db-server \
  --source-tags=app-server \
  --priority=1200

# Allow on-premises to all tiers
gcloud compute firewall-rules create allow-onprem \
  --network=enterprise-vpc \
  --allow=tcp,udp,icmp \
  --source-ranges=192.168.0.0/16 \
  --priority=900

# Allow IAP for SSH
gcloud compute firewall-rules create allow-iap-ssh \
  --network=enterprise-vpc \
  --allow=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --priority=1500

# 6. Configure Private DNS
gcloud dns managed-zones create enterprise-private \
  --dns-name=enterprise.internal. \
  --networks=enterprise-vpc \
  --visibility=private

gcloud dns record-sets create db.enterprise.internal. \
  --zone=enterprise-private \
  --type=A \
  --ttl=300 \
  --rrdatas=10.0.3.10

# 7. Configure DNS forwarding to on-premises
gcloud dns managed-zones create onprem-forward \
  --dns-name=corp.example.com. \
  --networks=enterprise-vpc \
  --visibility=private \
  --forwarding-targets=192.168.1.10,192.168.1.11

gcloud dns policies create inbound-policy \
  --networks=enterprise-vpc \
  --enable-inbound-forwarding
```

### Scenario 2: Multi-Region GKE Deployment with Global Load Balancing

**Requirements**:
- GKE clusters in us-central1 and europe-west1
- Global HTTP(S) load balancer
- Private GKE nodes (no external IPs)
- Shared VPC architecture
- Cross-region disaster recovery

**Architecture**:
```
Global HTTP(S) Load Balancer (Anycast IP)
    |
    +-- Backend Service (Multi-region)
        |
        +-- US Region NEG (us-central1)
        |   - GKE Cluster 1
        |   - Pods: 10.4.0.0/14
        |   - Services: 10.8.0.0/20
        |
        +-- EU Region NEG (europe-west1)
            - GKE Cluster 2
            - Pods: 10.12.0.0/14
            - Services: 10.16.0.0/20
```

**Implementation**:
```bash
# 1. Create Shared VPC in host project
gcloud compute shared-vpc enable host-project

gcloud compute networks create shared-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=global \
  --project=host-project

# 2. Create GKE subnets in both regions
gcloud compute networks subnets create gke-us-subnet \
  --network=shared-vpc \
  --region=us-central1 \
  --range=10.0.10.0/24 \
  --secondary-range pods-us=10.4.0.0/14 \
  --secondary-range services-us=10.8.0.0/20 \
  --enable-private-ip-google-access \
  --project=host-project

gcloud compute networks subnets create gke-eu-subnet \
  --network=shared-vpc \
  --region=europe-west1 \
  --range=10.1.10.0/24 \
  --secondary-range pods-eu=10.12.0.0/14 \
  --secondary-range services-eu=10.16.0.0/20 \
  --enable-private-ip-google-access \
  --project=host-project

# 3. Attach service project
gcloud compute shared-vpc associated-projects add service-project \
  --host-project=host-project

# 4. Grant IAM permissions
gcloud compute networks subnets add-iam-policy-binding gke-us-subnet \
  --member='serviceAccount:service-PROJECT_NUM@container-engine-robot.iam.gserviceaccount.com' \
  --role='roles/compute.networkUser' \
  --region=us-central1 \
  --project=host-project

# 5. Create GKE clusters
gcloud container clusters create gke-us-cluster \
  --region=us-central1 \
  --network=projects/host-project/global/networks/shared-vpc \
  --subnetwork=projects/host-project/regions/us-central1/subnetworks/gke-us-subnet \
  --cluster-secondary-range-name=pods-us \
  --services-secondary-range-name=services-us \
  --enable-ip-alias \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.0.0/28 \
  --enable-autoscaling \
  --min-nodes=3 \
  --max-nodes=10 \
  --project=service-project

gcloud container clusters create gke-eu-cluster \
  --region=europe-west1 \
  --network=projects/host-project/global/networks/shared-vpc \
  --subnetwork=projects/host-project/regions/europe-west1/subnetworks/gke-eu-subnet \
  --cluster-secondary-range-name=pods-eu \
  --services-secondary-range-name=services-eu \
  --enable-ip-alias \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.1.0/28 \
  --enable-autoscaling \
  --min-nodes=3 \
  --max-nodes=10 \
  --project=service-project

# 6. Configure Cloud NAT for both regions
gcloud compute routers create nat-router-us \
  --network=shared-vpc \
  --region=us-central1 \
  --project=host-project

gcloud compute routers nats create nat-us \
  --router=nat-router-us \
  --region=us-central1 \
  --nat-custom-subnet-ip-ranges=gke-us-subnet \
  --auto-allocate-nat-external-ips \
  --project=host-project

# Repeat for europe-west1...
```

### Scenario 3: Multi-VPC Architecture with Hub-and-Spoke Topology

**Requirements**:
- Central hub VPC for shared services
- Multiple spoke VPCs for different business units
- Centralized egress through hub
- Private connectivity to on-premises
- Network appliances (firewall, IDS/IPS) in hub

**Architecture**:
```
                  Hub VPC (10.0.0.0/16)
                  - Shared Services
                  - Network Virtual Appliances
                  - VPN to On-Premises
                        |
        +---------------+---------------+
        |               |               |
    Spoke VPC 1     Spoke VPC 2     Spoke VPC 3
    (10.1.0.0/16)   (10.2.0.0/16)   (10.3.0.0/16)
    Finance         Marketing        Engineering
```

**Implementation**:
```bash
# 1. Create Hub VPC
gcloud compute networks create hub-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=global

gcloud compute networks subnets create hub-subnet \
  --network=hub-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24

gcloud compute networks subnets create nva-subnet \
  --network=hub-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24

# 2. Create Spoke VPCs
gcloud compute networks create spoke-finance-vpc \
  --subnet-mode=custom

gcloud compute networks subnets create finance-subnet \
  --network=spoke-finance-vpc \
  --region=us-central1 \
  --range=10.1.1.0/24 \
  --enable-private-ip-google-access

gcloud compute networks create spoke-marketing-vpc \
  --subnet-mode=custom

gcloud compute networks subnets create marketing-subnet \
  --network=spoke-marketing-vpc \
  --region=us-central1 \
  --range=10.2.1.0/24 \
  --enable-private-ip-google-access

# 3. Configure VPC Peering (Hub-to-Spoke)
gcloud compute networks peerings create hub-to-finance \
  --network=hub-vpc \
  --peer-network=spoke-finance-vpc \
  --export-custom-routes \
  --import-custom-routes

gcloud compute networks peerings create finance-to-hub \
  --network=spoke-finance-vpc \
  --peer-network=hub-vpc \
  --export-custom-routes \
  --import-custom-routes

# Repeat for other spokes...

# 4. Deploy Network Virtual Appliance (NVA)
gcloud compute instances create nva-instance \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --subnet=nva-subnet \
  --can-ip-forward \
  --tags=nva \
  --image-family=ubuntu-2004-lts \
  --image-project=ubuntu-os-cloud

# 5. Create ILB as Next-Hop for NVA
gcloud compute health-checks create tcp nva-health \
  --port=80

gcloud compute backend-services create nva-backend \
  --load-balancing-scheme=INTERNAL \
  --health-checks=nva-health \
  --region=us-central1 \
  --protocol=TCP

gcloud compute backend-services add-backend nva-backend \
  --instance-group=nva-ig \
  --instance-group-zone=us-central1-a \
  --region=us-central1

gcloud compute forwarding-rules create nva-ilb \
  --load-balancing-scheme=INTERNAL \
  --backend-service=nva-backend \
  --region=us-central1 \
  --subnet=nva-subnet \
  --ip-protocol=TCP \
  --ports=ALL

# 6. Create custom routes for internet via NVA
gcloud compute routes create route-to-internet \
  --network=hub-vpc \
  --destination-range=0.0.0.0/0 \
  --next-hop-ilb=nva-ilb \
  --next-hop-ilb-region=us-central1 \
  --priority=100
```

### Scenario 4: Private Service Connect Architecture for SaaS

**Requirements**:
- Publish internal service to multiple consumer VPCs
- Consumer isolation
- Private IP addressing
- Accept connections only from approved projects

**Implementation**:
```bash
# Producer Side (Service Provider)

# 1. Create producer VPC and subnet
gcloud compute networks create producer-vpc \
  --subnet-mode=custom

gcloud compute networks subnets create service-subnet \
  --network=producer-vpc \
  --region=us-central1 \
  --range=10.100.1.0/24

# 2. Create NAT subnet for PSC
gcloud compute networks subnets create psc-nat-subnet \
  --network=producer-vpc \
  --region=us-central1 \
  --range=10.100.200.0/24 \
  --purpose=PRIVATE_SERVICE_CONNECT

# 3. Create internal load balancer for service
gcloud compute forwarding-rules create service-ilb \
  --load-balancing-scheme=INTERNAL \
  --backend-service=my-backend-service \
  --region=us-central1 \
  --subnet=service-subnet \
  --ip-protocol=TCP \
  --ports=443

# 4. Create service attachment
gcloud compute service-attachments create my-service \
  --region=us-central1 \
  --producer-forwarding-rule=service-ilb \
  --connection-preference=ACCEPT_MANUAL \
  --nat-subnets=psc-nat-subnet \
  --consumer-accept-list=PROJECT_ID_1:10,PROJECT_ID_2:5

# Consumer Side

# 5. Create PSC endpoint in consumer VPC
gcloud compute addresses create psc-endpoint \
  --region=us-central1 \
  --subnet=consumer-subnet \
  --addresses=10.0.5.100

gcloud compute forwarding-rules create psc-connection \
  --region=us-central1 \
  --network=consumer-vpc \
  --address=psc-endpoint \
  --target-service-attachment=projects/PRODUCER_PROJECT/regions/us-central1/serviceAttachments/my-service

# 6. Verify connection
gcloud compute forwarding-rules describe psc-connection \
  --region=us-central1 \
  --format="get(pscConnectionStatus)"
```

## Network Troubleshooting and Monitoring

### Common Issues and Solutions

**Issue 1: VPN Tunnel Down**
```bash
# Check tunnel status
gcloud compute vpn-tunnels describe TUNNEL_NAME \
  --region=REGION \
  --format="get(status,detailedStatus)"

# Check BGP session
gcloud compute routers get-status ROUTER_NAME \
  --region=REGION \
  --format="table(result.bgpPeerStatus[].name,result.bgpPeerStatus[].status)"

# Common causes:
# - Incorrect shared secret
# - Firewall blocking UDP 500, 4500, or ESP protocol
# - Incorrect peer IP or ASN
# - BGP session timeout

# Solution: Verify configuration
gcloud compute vpn-tunnels describe TUNNEL_NAME --region=REGION
```

**Issue 2: Instances Cannot Reach Internet**
```bash
# Check if instance has external IP
gcloud compute instances describe INSTANCE --zone=ZONE \
  --format="get(networkInterfaces[0].accessConfigs[0].natIP)"

# Check Cloud NAT configuration
gcloud compute routers nats describe NAT_NAME \
  --router=ROUTER_NAME \
  --region=REGION

# Check routes
gcloud compute routes list \
  --filter="network:VPC_NAME AND destRange:0.0.0.0/0"

# Check firewall egress rules
gcloud compute firewall-rules list \
  --filter="network:VPC_NAME AND direction:EGRESS"

# Common causes:
# - No external IP and no Cloud NAT
# - Egress firewall rule blocking traffic
# - No default route to internet gateway
```

**Issue 3: Cannot Access Google APIs from Private Instances**
```bash
# Check Private Google Access
gcloud compute networks subnets describe SUBNET \
  --region=REGION \
  --format="get(privateIpGoogleAccess)"

# Enable if disabled
gcloud compute networks subnets update SUBNET \
  --region=REGION \
  --enable-private-ip-google-access

# Check DNS resolution
# From instance:
dig restricted.googleapis.com
# Should resolve to 199.36.153.8/30 range

# Check firewall allows HTTPS to 199.36.153.8/30
gcloud compute firewall-rules list \
  --filter="network:VPC_NAME AND allowed.IPProtocol=tcp AND allowed.ports:443"
```

**Issue 4: High Latency Between Regions**
```bash
# Check routing mode (regional vs global)
gcloud compute networks describe VPC_NAME \
  --format="get(routingConfig.routingMode)"

# Verify network tier (Premium vs Standard)
gcloud compute addresses describe ADDRESS_NAME \
  --global \
  --format="get(networkTier)"

# Use Performance Dashboard
# Console -> Network Intelligence Center -> Performance Dashboard

# Perform connectivity test
gcloud network-management connectivity-tests create latency-test \
  --source-instance=projects/PROJECT/zones/us-central1-a/instances/instance1 \
  --destination-instance=projects/PROJECT/zones/europe-west1-b/instances/instance2 \
  --protocol=TCP \
  --destination-port=443
```

### Network Intelligence Center Tools

**Connectivity Tests**:
```bash
# Test connectivity between two endpoints
gcloud network-management connectivity-tests create test1 \
  --source-instance=projects/PROJECT/zones/ZONE/instances/SOURCE \
  --destination-ip-address=DEST_IP \
  --destination-port=443 \
  --protocol=TCP

# View test results
gcloud network-management connectivity-tests describe test1 \
  --format=json
```

**Performance Dashboard**:
```bash
# View packet loss metrics
gcloud monitoring time-series list \
  --filter='metric.type="networking.googleapis.com/vm_flow/rtt"' \
  --format=json

# View throughput metrics
gcloud monitoring time-series list \
  --filter='metric.type="networking.googleapis.com/vm_flow/egress_bytes_count"' \
  --format=json
```

**Firewall Insights**:
```bash
# Enable Firewall Insights
gcloud compute networks update VPC_NAME \
  --enable-firewall-insights

# View shadowed rules
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=global \
  --recommender=google.compute.firewall.Recommender \
  --filter="recommenderSubtype=SHADOWED_RULE"

# View overly permissive rules
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=global \
  --recommender=google.compute.firewall.Recommender \
  --filter="recommenderSubtype=ALLOW_RULE_TOO_PERMISSIVE"
```

## Professional Network Engineer Exam Tips

### Key Concepts to Master

**1. VPC Peering Limitations** (HIGH PRIORITY):
- NO transitive peering
- NO exchange of default routes (0.0.0.0/0)
- BGP routes NOT exchanged through peering
- Max 25 peerings per VPC
- No IP address overlap allowed

**2. Routing Decision Order**:
```
1. Subnet routes (highest priority - cannot be overridden)
2. Static routes (by priority value)
3. Dynamic routes (by priority value)
4. Default route (priority 1000)

Same destination + same priority = ECMP (load balancing)
```

**3. HA VPN vs Classic VPN**:
- HA VPN: 99.99% SLA (requires 2 tunnels to 2 endpoints)
- Classic VPN: 99.9% SLA (deprecated)
- HA VPN required for production (exam preference)

**4. Hybrid Connectivity Selection**:
```
Use HA VPN when:
- Need 99.99% SLA
- Bandwidth up to 6 Gbps sufficient
- Encryption required
- Lower cost acceptable

Use Dedicated Interconnect when:
- Need 10+ Gbps bandwidth
- Lowest latency required
- Have colocation facility near Google PoP
- Higher cost acceptable

Use Partner Interconnect when:
- Need flexible bandwidth (50 Mbps - 50 Gbps)
- No colocation facility available
- Lower latency than VPN
```

**5. Shared VPC vs VPC Peering**:
```
Shared VPC:
- Centralized network administration
- Same organization
- Subnet-level IAM control
- Shared routing table
- Unlimited service projects

VPC Peering:
- Distributed management
- Works across organizations
- Independent routing
- Network-level access
- Max 25 peerings
```

**6. Private Google Access Scenarios**:
- Instances WITHOUT external IP need Private Google Access
- Configured per subnet
- Requires route to 199.36.153.8/30 (restricted.googleapis.com)
- DNS must resolve *.googleapis.com to restricted.googleapis.com

**7. Cloud NAT Port Calculations** (EXAM MATH):
```
Ports per IP: 64,512 (65,536 - 1,024 reserved)
Default allocation: 64 ports per VM

Formula: (Number of VMs × Ports per VM) ÷ 64,512 = NAT IPs needed

Example:
- 200 VMs × 128 ports = 25,600 ports
- 25,600 ÷ 64,512 = 0.4 → Need 1 NAT IP
```

**8. BGP Route Priority**:
- Lower priority value = MORE preferred
- Primary path: priority 100
- Backup path: priority 200
- Default: priority 100 (if not specified)

**9. DNS Resolution Order**:
```
1. Private zones (if name matches)
2. Forwarding zones (if name matches)
3. Peering zones (if name matches)
4. Public zones
5. Internet DNS
```

**10. Firewall Rule Evaluation**:
- Lowest priority number evaluated first
- First match wins (allow or deny)
- Hierarchical policies evaluated before VPC rules
- Implicit deny all ingress (priority 65535)
- Implicit allow all egress (priority 65535)

### Exam Question Patterns

**Pattern 1: "Which connectivity option should be used?"**
- Look for: SLA requirements, bandwidth needs, latency sensitivity
- Answer: HA VPN for standard needs, Interconnect for high bandwidth/low latency

**Pattern 2: "VPC A can't reach VPC C through VPC B"**
- Answer: VPC Peering is NOT transitive - must create direct peering

**Pattern 3: "Instances can't reach Google APIs"**
- Check: Private Google Access enabled, correct routes, DNS resolution

**Pattern 4: "How to centralize network management?"**
- Answer: Use Shared VPC for centralized control, VPC Peering for distributed

**Pattern 5: "Optimize firewall rules"**
- Answer: Use Firewall Insights to identify shadowed/overly permissive rules

**Pattern 6: "Route not being used despite lower priority"**
- Check: Subnet routes take precedence over custom routes

**Pattern 7: "High Cloud NAT port exhaustion"**
- Answer: Increase min-ports-per-vm or add more NAT IPs

**Pattern 8: "Cross-region traffic routing through on-premises"**
- Check: Dynamic routing mode (regional blocks this, global allows it)

**Pattern 9: "How to implement network segmentation?"**
- Answer: Separate subnets + firewall rules OR separate VPCs with peering

**Pattern 10: "Private connectivity for managed services"**
- Answer: Private Service Connect for GCP services, VPC Service Controls for perimeter

### Practice Scenarios for Exam

**Scenario A**: Company needs to connect 5 VPCs with centralized egress control
- **Answer**: Hub-and-spoke with central hub VPC, custom routes via ILB next-hop

**Scenario B**: On-premises needs to resolve GCP private DNS zones
- **Answer**: Inbound DNS forwarding policy, configure on-prem DNS to forward to GCP

**Scenario C**: GKE pods need internet access but nodes should be private
- **Answer**: Private GKE cluster + Cloud NAT on GKE subnet

**Scenario D**: Multi-region app needs active-active with health-based routing
- **Answer**: Global HTTP(S) Load Balancer with backend services in multiple regions

**Scenario E**: Require 15 Gbps bandwidth to on-premises with encryption
- **Answer**: Dedicated Interconnect + VPN over Interconnect for encryption

## Best Practices Summary

### Network Design
1. Always use custom mode VPCs for production
2. Plan IP addressing for 3-5 years growth
3. Use /24 subnets for compute, /14-/16 for GKE pods
4. Document all IP allocations in IPAM tool
5. Enable VPC Flow Logs for troubleshooting
6. Use descriptive names for all network resources

### Security
1. Implement defense in depth: hierarchical + VPC firewall rules
2. Use service accounts for firewall targeting (not tags)
3. Enable Private Google Access on private subnets
4. Use Cloud NAT instead of external IPs when possible
5. Implement Cloud Armor for internet-facing services
6. Enable firewall logging for security analysis
7. Regular firewall rule audits with Firewall Insights

### High Availability
1. Use HA VPN with two tunnels for 99.99% SLA
2. Deploy resources across multiple zones
3. Use regional managed instance groups
4. Configure health checks for all load balancers
5. Implement redundant Cloud Routers
6. Use global load balancing for multi-region deployments

### Hybrid Connectivity
1. HA VPN for most hybrid scenarios (cost-effective, encrypted)
2. Dedicated/Partner Interconnect for high bandwidth needs
3. Always use BGP for dynamic routing
4. Set up redundant connections for critical workloads
5. Monitor BGP sessions and tunnel status
6. Document all on-premises network dependencies

### Monitoring and Operations
1. Enable VPC Flow Logs (with sampling for cost)
2. Use Network Intelligence Center for visibility
3. Set up alerts for VPN tunnel status
4. Monitor Cloud NAT port allocation
5. Use Connectivity Tests for troubleshooting
6. Regular review of firewall rules and routes

### Cost Optimization
1. Use standard network tier for non-latency-sensitive workloads
2. Implement Cloud NAT instead of external IPs
3. Use VPC Flow Logs sampling (not 100%)
4. Consolidate resources in same region/zone when possible
5. Right-size Cloud NAT IPs based on actual port usage
6. Use Shared VPC to share networking costs

## Key Commands Reference

```bash
# VPC Operations
gcloud compute networks create/list/describe/delete/update

# Subnet Operations
gcloud compute networks subnets create/list/describe/update/delete
gcloud compute networks subnets add-iam-policy-binding  # For Shared VPC

# Firewall Rules
gcloud compute firewall-rules create/list/describe/update/delete
gcloud compute firewall-policies create/update  # Hierarchical policies

# Routes
gcloud compute routes create/list/describe/delete

# VPN
gcloud compute vpn-gateways create/list/describe  # HA VPN
gcloud compute vpn-tunnels create/list/describe
gcloud compute external-vpn-gateways create/describe

# Cloud Router
gcloud compute routers create/update/list/describe/delete
gcloud compute routers add-interface/add-bgp-peer
gcloud compute routers get-status  # View BGP status and routes

# Cloud NAT
gcloud compute routers nats create/update/describe/delete

# Interconnect
gcloud compute interconnects attachments create/describe
gcloud compute interconnects describe/list

# VPC Peering
gcloud compute networks peerings create/list/update/delete

# Shared VPC
gcloud compute shared-vpc enable/disable
gcloud compute shared-vpc associated-projects add/remove

# DNS
gcloud dns managed-zones create/list/describe/delete
gcloud dns record-sets create/list/update/delete
gcloud dns policies create/update/describe

# Private Service Connect
gcloud compute forwarding-rules create  # For PSC endpoints
gcloud compute service-attachments create  # For PSC producers

# Network Intelligence
gcloud network-management connectivity-tests create/describe
gcloud recommender recommendations list  # Firewall Insights

# Monitoring
gcloud logging read "resource.type=..."
gcloud monitoring time-series list
```

## Additional Resources

### Official Documentation
- [VPC Documentation](https://cloud.google.com/vpc/docs)
- [Hybrid Connectivity Options](https://cloud.google.com/network-connectivity/docs)
- [Cloud Router and BGP](https://cloud.google.com/network-connectivity/docs/router)
- [VPC Network Peering](https://cloud.google.com/vpc/docs/vpc-peering)
- [Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc)
- [Private Google Access](https://cloud.google.com/vpc/docs/private-google-access)
- [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)
- [Cloud NAT](https://cloud.google.com/nat/docs/overview)
- [Cloud DNS](https://cloud.google.com/dns/docs)
- [Network Intelligence Center](https://cloud.google.com/network-intelligence-center/docs)

### Architecture Guides
- [VPC Design Best Practices](https://cloud.google.com/architecture/best-practices-vpc-design)
- [Hybrid and Multi-cloud Architecture](https://cloud.google.com/solutions/hybrid-and-multi-cloud-architecture-patterns)
- [Network Architecture Patterns](https://cloud.google.com/architecture/networking-patterns)
- [Hub-and-Spoke Network Architecture](https://cloud.google.com/architecture/hub-and-spoke-network-architecture)

### Exam Preparation
- [Professional Cloud Network Engineer Exam Guide](https://cloud.google.com/certification/guides/cloud-network-engineer)
- [Practice Exam](https://cloud.google.com/certification/practice-exam/cloud-network-engineer)
- [Network Engineer Learning Path](https://cloud.google.com/training/networking)
