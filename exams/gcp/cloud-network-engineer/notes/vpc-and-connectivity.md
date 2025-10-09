# VPC and Connectivity - GCP Professional Cloud Network Engineer

## Overview

VPC network design, hybrid connectivity, routing, and network architecture for the Professional Cloud Network Engineer certification.

## VPC Fundamentals

### Network Types
**Auto Mode**: Automatic subnet creation in each region
**Custom Mode**: Manual subnet creation (recommended for production)

### Subnet Design
**CIDR Planning**:
- Primary range: /8 to /29
- Secondary ranges: For pods (GKE) and services
- Avoid overlap with on-premises
- Plan for growth

**Example**:
```bash
# Create custom VPC
gcloud compute networks create production-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional

# Create subnets
gcloud compute networks subnets create web-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24

gcloud compute networks subnets create app-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24

gcloud compute networks subnets create db-subnet \
  --network=production-vpc \
  --region=us-central1 \
  --range=10.0.3.0/24 \
  --enable-private-ip-google-access
```

### Firewall Rules
**Hierarchy**:
1. Hierarchical firewall policies (org/folder)
2. VPC firewall rules (network)
3. Implicit rules

**Best Practices**:
```bash
# Baseline deny-all
gcloud compute firewall-rules create deny-all-ingress \
  --network=production-vpc \
  --action=deny \
  --direction=ingress \
  --rules=all \
  --priority=65534

# Allow specific traffic
gcloud compute firewall-rules create allow-web-traffic \
  --network=production-vpc \
  --allow=tcp:80,tcp:443 \
  --target-tags=web-server \
  --source-ranges=0.0.0.0/0 \
  --priority=1000

# Allow internal communication
gcloud compute firewall-rules create allow-internal \
  --network=production-vpc \
  --allow=tcp:0-65535,udp:0-65535,icmp \
  --source-ranges=10.0.0.0/16 \
  --priority=2000
```

### Routes
**Types**:
- System-generated routes
- Custom static routes
- Dynamic routes (Cloud Router)

```bash
# Create static route
gcloud compute routes create route-to-onprem \
  --network=production-vpc \
  --destination-range=192.168.0.0/16 \
  --next-hop-vpn-tunnel=my-vpn-tunnel \
  --priority=100
```

## VPC Connectivity

### VPC Peering
**Use Case**: Connect VPCs privately

```bash
# Create peering from VPC1 to VPC2
gcloud compute networks peerings create peer-vpc1-to-vpc2 \
  --network=vpc1 \
  --peer-network=vpc2 \
  --peer-project=PROJECT2

# Create reverse peering
gcloud compute networks peerings create peer-vpc2-to-vpc1 \
  --network=vpc2 \
  --peer-network=vpc1 \
  --peer-project=PROJECT1
```

**Limitations**:
- No transitive peering
- Overlapping IPs not allowed
- Max 25 peerings per VPC

### Shared VPC
**Architecture**:
- Host project: Contains VPC network
- Service projects: Attached to host project

```bash
# Enable Shared VPC
gcloud compute shared-vpc enable HOST_PROJECT

# Attach service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT \
  --host-project=HOST_PROJECT

# Grant subnet IAM permissions
gcloud compute networks subnets add-iam-policy-binding SUBNET \
  --member='serviceAccount:SERVICE_ACCOUNT' \
  --role='roles/compute.networkUser' \
  --region=REGION \
  --project=HOST_PROJECT
```

## Hybrid Connectivity

### Cloud VPN
**HA VPN** (recommended):
- 99.99% SLA
- Two tunnels required
- Supports BGP

```bash
# Create HA VPN gateway
gcloud compute vpn-gateways create ha-vpn-gw \
  --network=production-vpc \
  --region=us-central1

# Create Cloud Router
gcloud compute routers create cloud-router \
  --network=production-vpc \
  --region=us-central1 \
  --asn=65001

# Create VPN tunnels
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-gcp-gateway=ON_PREM_GW \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SHARED_SECRET \
  --router=cloud-router \
  --vpn-gateway=ha-vpn-gw \
  --interface=0

# Configure BGP
gcloud compute routers add-bgp-peer cloud-router \
  --peer-name=onprem-peer-1 \
  --interface=if-tunnel-1 \
  --peer-ip-address=169.254.1.2 \
  --peer-asn=65002 \
  --region=us-central1
```

### Cloud Interconnect
**Dedicated Interconnect**:
- 10 Gbps or 100 Gbps connections
- Direct connection to Google
- Low latency, high throughput

**Partner Interconnect**:
- 50 Mbps to 50 Gbps
- Through service provider
- Flexible capacity

```bash
# Create VLAN attachment
gcloud compute interconnects attachments create my-attachment \
  --region=us-central1 \
  --router=cloud-router \
  --interconnect=INTERCONNECT_NAME \
  --vlan=100
```

### Cloud Router (BGP)
**Features**:
- Dynamic route exchange
- Automatic failover
- Custom route advertisements

```bash
# Advertise custom routes
gcloud compute routers update cloud-router \
  --region=us-central1 \
  --advertisement-mode=custom \
  --set-advertisement-ranges=10.0.0.0/16,10.1.0.0/16
```

## Load Balancing

### Global Load Balancers
**HTTP(S) Load Balancer**:
- Layer 7 load balancing
- URL-based routing
- SSL/TLS termination
- Cloud CDN integration

```bash
# Create backend service
gcloud compute backend-services create web-backend \
  --protocol=HTTP \
  --health-checks=http-health-check \
  --global

# Add backend
gcloud compute backend-services add-backend web-backend \
  --instance-group=web-ig \
  --instance-group-zone=us-central1-a \
  --global

# Create URL map
gcloud compute url-maps create web-map \
  --default-service=web-backend

# Create target proxy
gcloud compute target-http-proxies create web-proxy \
  --url-map=web-map

# Create forwarding rule
gcloud compute forwarding-rules create web-lb \
  --global \
  --target-http-proxy=web-proxy \
  --ports=80
```

### Regional Load Balancers
**Internal TCP/UDP Load Balancer**:
- Private load balancing
- Regional service
- Pass-through

## Network Services

### Cloud DNS
**Private Zone**:
```bash
# Create private zone
gcloud dns managed-zones create private-zone \
  --dns-name=internal.example.com. \
  --networks=production-vpc \
  --visibility=private

# Add record
gcloud dns record-sets create db.internal.example.com. \
  --zone=private-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=10.0.3.10
```

### Cloud CDN
```bash
# Enable CDN on backend service
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --global
```

### Cloud NAT
```bash
# Create Cloud NAT
gcloud compute routers nats create my-nat \
  --router=cloud-router \
  --region=us-central1 \
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips
```

## Best Practices

### Network Design
1. Use custom mode VPCs
2. Plan IP addressing carefully
3. Implement network segmentation
4. Use hierarchical firewall policies
5. Enable VPC Flow Logs
6. Private Google Access for internal resources

### Hybrid Connectivity
1. Use HA VPN for production
2. Implement redundant connections
3. BGP for dynamic routing
4. Monitor tunnel health
5. Test failover scenarios
6. Document network topology

### Security
1. Principle of least privilege for firewall rules
2. Use service accounts for targeting
3. Implement Cloud Armor for public services
4. Private Service Connect for SaaS
5. VPC Service Controls for sensitive data

## Common Scenarios

**Scenario**: Multi-tier application with hybrid connectivity
**Solution**: Shared VPC with subnets per tier, HA VPN to on-premises, Cloud Router for BGP, internal load balancing

**Scenario**: Global application with low latency
**Solution**: Multi-region deployment, global HTTP(S) LB, Cloud CDN, Premium network tier

**Scenario**: Secure internal services
**Solution**: Private subnets, Cloud NAT for outbound, Identity-Aware Proxy for access, no external IPs

## Study Tips

1. Practice VPC and subnet creation
2. Configure hybrid connectivity (VPN, Interconnect)
3. Implement load balancing scenarios
4. Design firewall rule hierarchies
5. Work with Cloud Router and BGP
6. Understand network troubleshooting
7. IP addressing and CIDR planning

## Key Commands

```bash
# VPC
gcloud compute networks create/list/describe/delete

# Subnets
gcloud compute networks subnets create/list/describe/update

# Firewall
gcloud compute firewall-rules create/list/describe/delete

# VPN
gcloud compute vpn-gateways/vpn-tunnels create/list/describe

# Load Balancing
gcloud compute backend-services/forwarding-rules create/list

# Cloud Router
gcloud compute routers create/update/add-bgp-peer
```

## Additional Resources

- [VPC Documentation](https://cloud.google.com/vpc/docs)
- [Hybrid Connectivity](https://cloud.google.com/network-connectivity/docs)
- [Load Balancing](https://cloud.google.com/load-balancing/docs)
- [Network Design Best Practices](https://cloud.google.com/architecture/best-practices-vpc-design)
