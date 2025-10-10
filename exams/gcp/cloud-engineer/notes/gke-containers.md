# Google Kubernetes Engine (GKE) and Containers - Cloud Engineer

## Google Kubernetes Engine (GKE)

### Cluster Types

**Standard Cluster**:
- Full control over cluster configuration
- Manual management of node pools
- Configure networking, security, add-ons
- More flexibility and control

**Autopilot Cluster**:
- Fully managed, hands-off operation
- Google manages nodes, networking, security
- Pod-based pricing (no node charges)
- Optimized configuration
- Best for most use cases

### Cluster Architecture

**Control Plane**:
- Managed by Google (always free)
- API server, scheduler, controller manager
- etcd for cluster state
- Regional or zonal

**Nodes**:
- VMs running containerized workloads
- Part of managed instance groups
- Different machine types available
- Preemptible nodes for cost savings

**Node Pools**:
- Group of nodes with same configuration
- Multiple pools per cluster
- Different machine types, OS, node version
- Auto-scaling per pool
- Auto-repair and auto-upgrade

### Cluster Modes

**Zonal Cluster**:
- Single zone master
- Nodes in single or multiple zones
- Lower cost
- No SLA for control plane

**Regional Cluster**:
- Multi-zonal control plane (3 replicas)
- Nodes across zones in region
- 99.95% SLA for control plane
- Higher availability

**Private Cluster**:
- Nodes have only private IPs
- Control plane accessible via private endpoint
- Enhanced security
- VPC-native required

### Networking

**VPC-Native (Recommended)**:
- Uses alias IP ranges
- Pod IPs routable in VPC
- Better integration with GCP services
- Supports Network Policy

**Routes-Based**:
- Legacy mode
- Uses routes for pod networking
- Limited scalability

**Network Policy**:
- Calico-based pod-to-pod firewall
- Define ingress and egress rules
- Kubernetes NetworkPolicy resources

### Workload Identity
- Kubernetes service accounts → Google service accounts
- Fine-grained IAM permissions for pods
- No need for service account keys
- Best practice for pod authentication

### Auto-Scaling

**Cluster Autoscaler**:
- Automatically add/remove nodes
- Based on pod resource requests
- Respects node pool constraints

**Horizontal Pod Autoscaler (HPA)**:
- Scale pods based on CPU, memory, or custom metrics
- Kubernetes native feature

**Vertical Pod Autoscaler (VPA)**:
- Automatically adjust CPU and memory requests
- Right-size pod resources

### Node Auto-Provisioning
- Automatically create new node pools
- Based on unschedulable pod requirements
- Dynamic resource optimization

### Release Channels

**Rapid**:
- Latest Kubernetes features
- Frequent updates
- For testing and early adoption

**Regular** (Default):
- Balanced updates
- Recommended for most workloads
- 2-3 months behind Rapid

**Stable**:
- Conservative updates
- For production workloads requiring stability
- 2-3 months behind Regular

### Node Auto-Upgrade
- Automatically upgrade node versions
- Controlled by release channel
- Maintenance windows configurable
- Surge upgrades for minimal disruption

### Node Auto-Repair
- Automatically repairs unhealthy nodes
- Monitors node health checks
- Drains and recreates failed nodes
- Enabled by default

## Container Registry

### Artifact Registry (Recommended)
- Next-generation container registry
- Multi-format support (Docker, Maven, npm)
- Regional and multi-regional repositories
- Vulnerability scanning
- IAM integration
- Encryption at rest

### Container Registry (Legacy)
- Docker-only
- Storage in Cloud Storage buckets
- Being replaced by Artifact Registry
- Still supported but migrate to Artifact Registry

### Image Security
- Vulnerability scanning (automatic)
- Binary Authorization (deployment policy)
- Image signing and verification
- Continuous validation

## Cloud Run

### Features
- Serverless containers
- Fully managed, no cluster management
- Auto-scaling to zero
- Pay per use (CPU and memory)
- HTTPS endpoints automatically

### Deployment
- Container images from Artifact Registry or Container Registry
- Revision-based deployments
- Traffic splitting between revisions
- Rollback to previous revisions

### Concurrency
- Requests per container instance (1-1000)
- Default: 80 concurrent requests
- CPU allocated only during request handling

### Networking
- Public endpoints by default
- VPC connector for private resources
- Cloud Load Balancing integration
- Custom domains with SSL

### Authentication
- Allow unauthenticated (public)
- Require authentication (IAM)
- Service-to-service authentication

## Cloud Build

### Features
- Serverless CI/CD platform
- Build, test, deploy containers
- Native Docker and Buildpacks support
- Integration with GitHub, Bitbucket, Cloud Source Repositories

### Build Configuration
- cloudbuild.yaml or JSON
- Steps: Build, test, deploy
- Substitution variables
- Concurrent steps

**Example cloudbuild.yaml**:
```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp']
  - name: 'gcr.io/cloud-builders/gke-deploy'
    args:
      - run
      - --filename=k8s/
      - --cluster=my-cluster
      - --location=us-central1
```

### Triggers
- Automated builds on code changes
- GitHub, Bitbucket, Cloud Source Repositories
- Branch, tag, or pull request filters
- Webhook triggers

## kubectl Commands

### Cluster Management
```bash
# Get cluster credentials
gcloud container clusters get-credentials CLUSTER_NAME --zone=ZONE

# View cluster info
kubectl cluster-info
kubectl get nodes
```

### Workload Management
```bash
# Create deployment
kubectl create deployment nginx --image=nginx

# Expose as service
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Scale deployment
kubectl scale deployment nginx --replicas=3

# Update image
kubectl set image deployment/nginx nginx=nginx:1.19
```

### Monitoring and Debugging
```bash
# View pods
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME

# Execute in pod
kubectl exec -it POD_NAME -- /bin/bash

# View events
kubectl get events
```

## Best Practices

### GKE Configuration
1. Use Autopilot for standard workloads
2. Enable Workload Identity (disable node service accounts)
3. Use regional clusters for production
4. Enable auto-upgrade and auto-repair
5. Use VPC-native networking
6. Implement Network Policies
7. Use Binary Authorization for image validation

### Resource Management
1. Set resource requests and limits
2. Use Horizontal Pod Autoscaler
3. Implement pod disruption budgets
4. Use node affinity and taints/tolerations
5. Separate workloads with namespaces

### Security
1. Enable Workload Identity
2. Use private clusters when possible
3. Implement least privilege RBAC
4. Scan images for vulnerabilities
5. Use secrets for sensitive data (not environment variables)
6. Enable GKE security features (Shielded nodes, Binary Authorization)

### Cost Optimization
1. Use preemptible nodes (up to 80% savings)
2. Enable cluster autoscaler
3. Right-size node pools
4. Use committed use discounts
5. Autopilot for hands-off optimization
6. Delete unused clusters and resources

## Exam Tips

### Common Scenarios
- **Managed Kubernetes**: GKE (Standard or Autopilot)
- **Serverless containers**: Cloud Run
- **Full control over nodes**: GKE Standard
- **Hands-off management**: GKE Autopilot
- **Store container images**: Artifact Registry
- **CI/CD for containers**: Cloud Build
- **High availability**: Regional GKE cluster
- **Cost optimization**: Preemptible nodes, cluster autoscaler
- **Private cluster**: Nodes with private IPs only
- **Pod authentication**: Workload Identity

### Key Differences
- **GKE Standard vs Autopilot**: Standard = more control, Autopilot = fully managed
- **GKE vs Cloud Run**: GKE = orchestration at scale, Cloud Run = simpler serverless
- **Zonal vs Regional**: Zonal = single zone control plane, Regional = multi-zone HA
- **VPC-native vs Routes-based**: VPC-native recommended (alias IPs, better integration)
- **Artifact Registry vs Container Registry**: Artifact Registry is next-gen (multi-format)

### Important Commands
```bash
# Create GKE cluster
gcloud container clusters create CLUSTER_NAME \
  --zone=us-central1-a \
  --num-nodes=3

# Get credentials
gcloud container clusters get-credentials CLUSTER_NAME

# Deploy to GKE
kubectl apply -f deployment.yaml

# Deploy Cloud Run
gcloud run deploy SERVICE_NAME \
  --image=gcr.io/PROJECT/IMAGE \
  --platform=managed \
  --region=us-central1
```

### Quick Reference
- Control plane: Always managed by Google (free)
- Node pools: Groups of nodes with same config
- Workload Identity: Pods → Google service accounts
- Auto-scaling: Cluster (nodes), HPA (pods), VPA (resources)
- Release channels: Rapid, Regular, Stable
- Cloud Run: Fully managed, serverless containers
- Artifact Registry: Store and manage container images
