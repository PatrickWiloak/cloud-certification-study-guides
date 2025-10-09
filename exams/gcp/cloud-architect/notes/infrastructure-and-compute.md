# Infrastructure and Compute - GCP Professional Cloud Architect

## Overview

Comprehensive guide to infrastructure and compute architecture decisions, covering compute services, storage strategies, network topologies, and infrastructure management for the Professional Cloud Architect certification.

## Key Topics

1. **Compute Service Selection** - Choosing between Compute Engine, GKE, App Engine, Cloud Run, Cloud Functions
2. **Storage Architecture** - Cloud Storage, persistent disks, filestore, data lifecycle management
3. **Network Topology** - VPC design, hybrid connectivity, global load balancing
4. **Infrastructure as Code** - Terraform, Deployment Manager, configuration management
5. **Capacity Planning** - Resource sizing, scaling strategies, performance optimization

## Compute Service Decision Matrix

### Compute Engine
**Use When**:
- Full control over OS and environment
- Legacy application migration
- Specific licensing requirements
- GPU/TPU workloads
- Custom networking requirements

**Considerations**:
- Manual scaling configuration
- OS patch management
- Higher operational overhead
- Most flexible option

### Google Kubernetes Engine (GKE)
**Use When**:
- Container-based microservices
- Multi-cloud portability needed
- Complex application orchestration
- Service mesh requirements
- Need for Kubernetes ecosystem

**Considerations**:
- Kubernetes learning curve
- Managed control plane
- Auto-scaling capabilities
- Binary Authorization support

### App Engine
**Use When**:
- Simple web applications
- Rapid development/deployment
- Automatic scaling needed
- Minimal operational overhead
- Standard runtime languages

**Considerations**:
- Limited customization
- Platform lock-in
- Traffic splitting for versions
- Standard vs. Flexible choice

### Cloud Run
**Use When**:
- Stateless HTTP services
- Container-based applications
- Pay-per-request desired
- Automatic scaling including to zero
- Minimal configuration

**Considerations**:
- Cold start latency
- Request timeout limits
- Container requirements
- Regional or multi-region

### Cloud Functions
**Use When**:
- Event-driven workloads
- Simple, single-purpose functions
- Integration with GCP services
- Rapid development
- Minimal code deployment

**Considerations**:
- Execution time limits
- Language runtime support
- Cold start considerations
- Event source integration

## Storage Architecture Patterns

### Cloud Storage Patterns

**Data Lake Pattern**:
- Raw data in Standard class
- Processed data in separate buckets
- Lifecycle policies for archival
- IAM and ACLs for access control
- Regional for performance, Multi-Regional for availability

**Content Delivery Pattern**:
- Cloud Storage for origin
- Cloud CDN for global distribution
- Signed URLs for private content
- Load balancing for high availability
- Versioning for rollback

**Backup and Archive Pattern**:
- Nearline for monthly access
- Coldline for quarterly access
- Archive for long-term retention
- Object lifecycle management
- Retention policies for compliance

### Persistent Disk Patterns

**Database Storage**:
- SSD for high IOPS requirements
- Standard for sequential workloads
- Snapshots for backup
- Regional PD for HA
- Sizing for performance needs

**Application Storage**:
- Boot disk optimization
- Separate data disks
- Snapshot scheduling
- Local SSD for extreme performance
- Read-only disk sharing

## Network Architecture

### VPC Design Patterns

**Hub-and-Spoke**:
- Central hub VPC for shared services
- Spoke VPCs for workloads
- VPC Peering or VPN connectivity
- Centralized network management
- Reduced complexity

**Mesh Network**:
- Full VPC Peering between all VPCs
- Direct communication paths
- Higher complexity
- Maximum performance
- More interconnections to manage

**Shared VPC**:
- Host project with VPC
- Service projects attached
- Centralized network admin
- Project-level isolation
- Organization-level management

### Load Balancing Architecture

**Global HTTP(S) Load Balancer**:
- Anycast IP for global distribution
- URL-based routing
- CDN integration
- Cloud Armor for security
- SSL certificate management

**Multi-Region Design**:
- Backend services in multiple regions
- Health checks for failover
- Session affinity considerations
- Capacity balancing
- Disaster recovery

**Internal Load Balancing**:
- Private load balancing within VPC
- Regional or cross-regional
- Service discovery for microservices
- Reduced latency
- Enhanced security

## Best Practices

### Compute Best Practices
1. **Right-sizing**: Use machine type recommendations
2. **Preemptible VMs**: Use for batch and fault-tolerant workloads
3. **Committed Use**: Purchase for predictable workloads
4. **Auto-scaling**: Configure based on actual metrics
5. **Regional Distribution**: Deploy across zones for HA
6. **Health Checks**: Implement for all production workloads
7. **Instance Templates**: Use for consistent deployments
8. **Startup Scripts**: Automate instance configuration

### Storage Best Practices
1. **Storage Class Selection**: Match to access patterns
2. **Lifecycle Policies**: Automate class transitions
3. **Regional Placement**: Consider data gravity
4. **Encryption**: Use CMEK for sensitive data
5. **Access Control**: Apply principle of least privilege
6. **Retention Policies**: Implement for compliance
7. **Monitoring**: Track storage usage and costs
8. **Versioning**: Enable for critical data

### Network Best Practices
1. **Subnet Planning**: Adequate IP space, avoid overlap
2. **Firewall Hierarchy**: Use tags and service accounts
3. **Private Google Access**: Enable for secure API access
4. **Cloud NAT**: For private instance internet access
5. **VPC Flow Logs**: Enable for troubleshooting
6. **Load Balancer Health Checks**: Configure appropriately
7. **SSL Certificates**: Use managed certificates
8. **Global Distribution**: Use for worldwide applications

## Common Exam Scenarios

**Scenario**: High-traffic web application with global users
**Solution**: Global HTTP(S) LB, Cloud CDN, multi-region GKE, Cloud SQL read replicas, Memorystore

**Scenario**: Batch data processing with cost optimization
**Solution**: Preemptible VMs, managed instance groups, Cloud Storage, scheduled scaling

**Scenario**: Hybrid architecture with on-premises integration
**Solution**: Cloud Interconnect, Shared VPC, Private Google Access, hybrid load balancing

**Scenario**: Microservices platform with service mesh
**Solution**: GKE with Istio, Anthos Service Mesh, internal load balancing, Cloud Trace

## gcloud Command Reference

```bash
# Compute Engine
gcloud compute instances create INSTANCE --machine-type=n1-standard-4 --preemptible
gcloud compute instance-templates create TEMPLATE --machine-type=e2-medium
gcloud compute instance-groups managed create GROUP --template=TEMPLATE --size=3

# GKE
gcloud container clusters create CLUSTER --num-nodes=3 --enable-autoscaling --min-nodes=1 --max-nodes=10
gcloud container node-pools create POOL --cluster=CLUSTER --machine-type=n1-standard-4

# Storage
gsutil mb -c STANDARD -l us-central1 gs://BUCKET
gsutil lifecycle set lifecycle.json gs://BUCKET

# Networking
gcloud compute networks create VPC --subnet-mode=custom
gcloud compute networks subnets create SUBNET --network=VPC --region=us-central1 --range=10.0.1.0/24
gcloud compute firewall-rules create RULE --network=VPC --allow=tcp:80,tcp:443
```

## Additional Resources

- [Compute Options Comparison](https://cloud.google.com/docs/choosing-a-compute-option)
- [Storage Decision Tree](https://cloud.google.com/products/storage)
- [Network Architecture Best Practices](https://cloud.google.com/architecture/best-practices-vpc-design)
- [Infrastructure as Code on GCP](https://cloud.google.com/docs/terraform)
