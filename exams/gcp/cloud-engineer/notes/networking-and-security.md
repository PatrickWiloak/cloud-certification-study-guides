# Networking and Security - GCP Associate Cloud Engineer

## Overview

This document covers Google Cloud networking and security fundamentals including VPC networks, firewall rules, load balancing, IAM, and security best practices. Understanding networking and security is critical for the Associate Cloud Engineer certification.

**[📖 Networking Overview](https://cloud.google.com/vpc/docs)** - Google Cloud Virtual Private Cloud networking

## Key Topics

### 1. Virtual Private Cloud (VPC)
- Software-defined networking in Google Cloud
- Subnets, routes, and firewall rules
- VPC peering and Shared VPC
- Private Google Access and Private Service Connect

**[📖 VPC Networks](https://cloud.google.com/vpc/docs/vpc)** - Virtual Private Cloud networks
**[📖 Firewall Rules](https://cloud.google.com/vpc/docs/firewalls)** - Control traffic to and from instances

### 2. Load Balancing
- Global and regional load balancers
- HTTP(S), TCP, and UDP load balancing
- Internal and external load balancing
- Health checks and backend services

**[📖 Load Balancing](https://cloud.google.com/load-balancing/docs)** - Distribute traffic across resources
**[📖 Choosing a Load Balancer](https://cloud.google.com/load-balancing/docs/choosing-load-balancer)** - Select the right load balancer

### 3. Cloud IAM
- Identity and Access Management
- Users, groups, service accounts
- Roles and permissions
- Policy hierarchy and inheritance

**[📖 IAM Documentation](https://cloud.google.com/iam/docs)** - Identity and Access Management
**[📖 IAM Roles](https://cloud.google.com/iam/docs/understanding-roles)** - Predefined and custom roles

### 4. Network Security
- Firewall rules and priorities
- Cloud Armor for DDoS protection
- VPN and Cloud Interconnect
- Security best practices

**[📖 Cloud Armor](https://cloud.google.com/armor/docs)** - DDoS protection and WAF
**[📖 Cloud VPN](https://cloud.google.com/network-connectivity/docs/vpn)** - Secure VPN connectivity

### 5. Hybrid Connectivity
- Cloud VPN for encrypted connectivity
- Cloud Interconnect for dedicated connections
- Cloud Router for dynamic routing
- Hybrid networking patterns

**[📖 Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect)** - Dedicated private connections
**[📖 Hybrid Connectivity](https://cloud.google.com/architecture/hybrid-and-multi-cloud-network-topologies)** - Network topology patterns

## GCP Services Reference

### VPC Networking
- **VPC Networks**: Global virtual networks
- **Subnets**: Regional IP address ranges
- **Routes**: Path definitions for network traffic
- **Firewall Rules**: Network security policies
- **VPC Peering**: Connect VPC networks privately
- **Shared VPC**: Share networks across projects
- **Private Google Access**: Access Google APIs from private IPs
- **Cloud NAT**: Outbound internet access for private instances

### Load Balancing
- **Global HTTP(S) Load Balancer**: Layer 7 global load balancing
- **Global SSL Proxy**: Layer 4 SSL/TLS load balancing
- **Global TCP Proxy**: Layer 4 TCP load balancing
- **Regional Network Load Balancer**: Layer 4 pass-through load balancing
- **Internal HTTP(S) Load Balancer**: Layer 7 internal load balancing
- **Internal TCP/UDP Load Balancer**: Layer 4 internal load balancing

### Cloud IAM
- **Members**: Users, groups, service accounts, domains
- **Roles**: Collections of permissions (basic, predefined, custom)
- **Policies**: Bindings of members to roles
- **Service Accounts**: Identity for applications and services
- **Workload Identity**: Kubernetes pod identity
- **Organization Policies**: Centralized constraints

### Security Services
- **Cloud Armor**: DDoS protection and WAF
- **Cloud KMS**: Encryption key management
- **Secret Manager**: Store and manage secrets
- **VPC Service Controls**: Perimeter security for APIs
- **Identity-Aware Proxy (IAP)**: Context-aware access control
- **Security Command Center**: Security posture management

## Best Practices

### VPC Networking Best Practices
1. **Subnet Design**: Plan IP ranges carefully to avoid overlap
2. **Custom Mode Networks**: Use custom mode for production (better control)
3. **Regional Subnets**: Design for regional availability
4. **Network Tags**: Use tags for firewall rule targeting
5. **Shared VPC**: Centralize network management for organizations
6. **Private Google Access**: Enable for instances without external IPs
7. **VPC Flow Logs**: Enable for network monitoring and troubleshooting
8. **Cloud NAT**: Use for secure outbound internet access

**[📖 VPC Best Practices](https://cloud.google.com/vpc/docs/best-practices)** - VPC network design best practices
**[📖 Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc)** - Centralized network management

### Firewall Rules Best Practices
1. **Deny-All Default**: Start with deny-all and explicitly allow required traffic
2. **Principle of Least Privilege**: Only open necessary ports
3. **Use Service Accounts**: Target rules by service account for granular control
4. **Priority Management**: Use priorities to control rule evaluation order
5. **Direction and Action**: Understand ingress/egress and allow/deny
6. **Logging**: Enable firewall logging for security analysis
7. **Hierarchical Policies**: Use for organization-wide enforcement
8. **Regular Audits**: Review and clean up unused rules

### Load Balancing Best Practices
1. **Health Checks**: Configure appropriate intervals and thresholds
2. **Session Affinity**: Use when client-server persistence is needed
3. **SSL Certificates**: Use managed certificates for automatic renewal
4. **CDN Integration**: Enable Cloud CDN for cacheable content
5. **Backend Configuration**: Set appropriate connection and request timeouts
6. **Capacity Planning**: Monitor and adjust backend capacity
7. **Logging**: Enable load balancer logging for analysis
8. **DDoS Protection**: Use Cloud Armor with global load balancers

### IAM Best Practices
1. **Least Privilege**: Grant minimum necessary permissions
2. **Groups Over Individuals**: Assign roles to groups, not individual users
3. **Service Accounts**: Use for application authentication
4. **Avoid Basic Roles**: Use predefined or custom roles instead of Owner/Editor/Viewer
5. **Organization Policies**: Implement organization-wide constraints
6. **Regular Audits**: Review IAM policies and remove unnecessary access
7. **Workload Identity**: Use for GKE pod authentication (not service account keys)
8. **Conditional Access**: Use IAM conditions for context-based access

### Security Best Practices
1. **Defense in Depth**: Implement multiple layers of security
2. **Encryption**: Use encryption at rest and in transit
3. **Secret Management**: Use Secret Manager, never hardcode secrets
4. **Network Segmentation**: Isolate workloads in separate networks
5. **Audit Logging**: Enable and monitor Cloud Audit Logs
6. **Vulnerability Scanning**: Use Container Analysis and Web Security Scanner
7. **Binary Authorization**: Enforce deployment policies for containers
8. **Security Command Center**: Monitor security posture continuously

## Common Scenarios

### Scenario 1: Three-Tier Web Application
**Requirement**: Deploy web, application, and database tiers securely
**Solution**:
- Create VPC with three subnets (web, app, database)
- Configure firewall rules: internet → web, web → app, app → database
- Use internal load balancer for app tier
- Use Cloud SQL with private IP for database
- Implement Cloud Armor for web tier DDoS protection

### Scenario 2: Hybrid Cloud Connectivity
**Requirement**: Connect on-premises network to Google Cloud
**Solution**:
- Set up Cloud VPN for encrypted connectivity over internet
- Configure Cloud Router for dynamic routing (BGP)
- Implement VPN tunnel redundancy for high availability
- Use Cloud Interconnect for dedicated, higher bandwidth connection
- Configure firewall rules to allow necessary traffic

### Scenario 3: Multi-Project Organization
**Requirement**: Manage multiple projects with shared networking
**Solution**:
- Create organization hierarchy with folders
- Implement Shared VPC in host project
- Attach service projects to shared VPC
- Use organization policies for security constraints
- Implement centralized logging and monitoring

### Scenario 4: Public API with DDoS Protection
**Requirement**: Expose REST API globally with security
**Solution**:
- Deploy API to Cloud Run or GKE
- Use global HTTP(S) load balancer
- Enable Cloud Armor for DDoS protection and WAF
- Configure rate limiting and geo-blocking
- Implement Cloud CDN for caching responses
- Use API keys or OAuth for authentication

### Scenario 5: Internal Application Access
**Requirement**: Provide secure access to internal application
**Solution**:
- Deploy application with internal load balancer
- Use Identity-Aware Proxy (IAP) for access control
- Configure IAM policies for authorized users
- Implement Cloud VPN for remote user access
- Enable audit logging for access monitoring

## Study Tips

### Hands-On Practice
1. **Create VPC Networks**: Practice creating custom mode networks with subnets
2. **Configure Firewall Rules**: Create allow/deny rules with different targets
3. **Set Up Load Balancers**: Deploy applications behind different load balancer types
4. **Manage IAM Policies**: Grant and revoke roles at different levels
5. **Implement VPN**: Set up Cloud VPN tunnel between two VPCs

### Key Concepts to Master
1. **VPC fundamentals**: Subnets, routes, firewall rules, peering
2. **Load balancer types**: When to use each type of load balancer
3. **IAM hierarchy**: Organization, folder, project, resource-level policies
4. **Service accounts**: Creation, management, and authentication
5. **Network security**: Firewalls, Cloud Armor, private connectivity

### Common Exam Topics
1. Designing VPC networks and subnet structures
2. Configuring firewall rules with appropriate priorities
3. Choosing the right load balancer for a scenario
4. Implementing IAM roles and policies
5. Setting up hybrid connectivity with VPN or Interconnect
6. Securing applications with Cloud Armor and IAP
7. Understanding Private Google Access and Cloud NAT

### gcloud Command Examples

```bash
# VPC Networks
gcloud compute networks create my-vpc --subnet-mode=custom
gcloud compute networks subnets create my-subnet --network=my-vpc --region=us-central1 --range=10.0.1.0/24
gcloud compute networks list
gcloud compute networks subnets list --network=my-vpc

# Enable Private Google Access
gcloud compute networks subnets update my-subnet --region=us-central1 --enable-private-ip-google-access

# Firewall Rules
gcloud compute firewall-rules create allow-ssh --network=my-vpc --allow=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create allow-http --network=my-vpc --allow=tcp:80 --target-tags=web-server
gcloud compute firewall-rules list
gcloud compute firewall-rules delete allow-ssh

# Routes
gcloud compute routes create my-route --network=my-vpc --destination-range=192.168.1.0/24 --next-hop-gateway=default-internet-gateway
gcloud compute routes list

# Load Balancer (Backend Service)
gcloud compute backend-services create my-backend --protocol=HTTP --health-checks=my-health-check --global
gcloud compute backend-services add-backend my-backend --instance-group=my-ig --instance-group-zone=us-central1-a --global

# Health Check
gcloud compute health-checks create http my-health-check --port=80 --request-path=/health

# IAM
gcloud projects add-iam-policy-binding PROJECT_ID --member=user:email@example.com --role=roles/compute.viewer
gcloud projects get-iam-policy PROJECT_ID
gcloud projects remove-iam-policy-binding PROJECT_ID --member=user:email@example.com --role=roles/compute.viewer

# Service Accounts
gcloud iam service-accounts create my-sa --display-name="My Service Account"
gcloud iam service-accounts list
gcloud iam service-accounts keys create key.json --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com

# Grant service account role
gcloud projects add-iam-policy-binding PROJECT_ID --member=serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com --role=roles/storage.objectViewer

# VPN
gcloud compute target-vpn-gateways create my-vpn-gateway --network=my-vpc --region=us-central1
gcloud compute forwarding-rules create my-vpn-rule --ip-protocol=ESP --target-vpn-gateway=my-vpn-gateway --region=us-central1

# Cloud Armor
gcloud compute security-policies create my-policy
gcloud compute security-policies rules create 1000 --security-policy=my-policy --action=deny-403 --src-ip-ranges=203.0.113.0/24
```

## Additional Resources

- [VPC Documentation](https://cloud.google.com/vpc/docs)
- [Load Balancing Documentation](https://cloud.google.com/load-balancing/docs)
- [Cloud IAM Documentation](https://cloud.google.com/iam/docs)
- [Cloud VPN Documentation](https://cloud.google.com/network-connectivity/docs/vpn)
- [Cloud Armor Documentation](https://cloud.google.com/armor/docs)
- [Networking Best Practices](https://cloud.google.com/architecture/best-practices-vpc-design)
