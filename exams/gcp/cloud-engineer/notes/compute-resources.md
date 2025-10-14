# Compute Resources - GCP Associate Cloud Engineer

## Overview

This document covers compute services on Google Cloud Platform including Compute Engine, Google Kubernetes Engine (GKE), App Engine, Cloud Functions, and Cloud Run. Understanding when to use each service and how to manage them is crucial for the Associate Cloud Engineer certification.

**[📖 Compute Options Overview](https://cloud.google.com/docs/choosing-a-compute-option)** - Guide to choosing the right compute service for your workload

## Key Topics

### 1. Compute Engine
- Virtual machine instances in the cloud
- Customizable compute, memory, and disk configurations
- Multiple machine types and families
- Preemptible and spot VMs for cost savings

**[📖 Compute Engine Documentation](https://cloud.google.com/compute/docs)** - Complete guide to Google Compute Engine

### 2. Google Kubernetes Engine (GKE)
- Managed Kubernetes service
- Container orchestration at scale
- Node pools and cluster autoscaling
- Workload identity and security features

**[📖 GKE Documentation](https://cloud.google.com/kubernetes-engine/docs)** - Managed Kubernetes service documentation

### 3. App Engine
- Fully managed platform-as-a-service (PaaS)
- Automatic scaling and load balancing
- Standard vs. Flexible environments
- Support for multiple programming languages

**[📖 App Engine Documentation](https://cloud.google.com/appengine/docs)** - Platform-as-a-service documentation

### 4. Cloud Functions
- Event-driven serverless compute
- HTTP and event triggers
- Lightweight, single-purpose functions
- Automatic scaling based on load

**[📖 Cloud Functions Documentation](https://cloud.google.com/functions/docs)** - Event-driven serverless functions

### 5. Cloud Run
- Fully managed serverless for containers
- Run containers without managing infrastructure
- Automatic scaling including scale-to-zero
- HTTP request-based invocation

**[📖 Cloud Run Documentation](https://cloud.google.com/run/docs)** - Serverless containers platform

## GCP Services Reference

### Compute Engine Services
- **Instance Templates**: Define VM configuration for creating instances
- **Instance Groups**: Managed and unmanaged groups of VMs
- **Persistent Disks**: Durable block storage for VMs
- **Local SSDs**: High-performance local storage
- **Snapshots**: Point-in-time copies of persistent disks
- **Images**: Boot disk templates for VM instances
- **Machine Types**: Predefined and custom machine configurations

### GKE Services
- **Clusters**: Managed Kubernetes control plane and nodes
- **Node Pools**: Groups of nodes with same configuration
- **Workloads**: Deployments, StatefulSets, DaemonSets
- **Services**: Network access to pods
- **Ingress**: HTTP(S) load balancing for services
- **Config Connector**: Manage GCP resources from Kubernetes

## Best Practices

### Compute Engine Best Practices
1. **Right-sizing**: Use machine recommendations to optimize costs
2. **Preemptible VMs**: Use for fault-tolerant, batch workloads (up to 80% cost savings)
3. **Sustained Use Discounts**: Automatic discounts for running instances
4. **Committed Use Contracts**: 1-year or 3-year commitments for significant savings
5. **Custom Machine Types**: Create VMs that exactly match your needs
6. **Startup Scripts**: Automate instance configuration and software installation
7. **Instance Templates**: Use templates for consistent, repeatable deployments
8. **Health Checks**: Implement health checks for instance groups and load balancers

**[📖 Machine Types](https://cloud.google.com/compute/docs/machine-types)** - Guide to available machine types and families
**[📖 Preemptible VMs](https://cloud.google.com/compute/docs/instances/preemptible)** - Cost-effective compute for fault-tolerant workloads
**[📖 Instance Groups](https://cloud.google.com/compute/docs/instance-groups)** - Managing groups of VM instances

### GKE Best Practices
1. **Node Auto-provisioning**: Automatically add node pools based on workload needs
2. **Workload Identity**: Use GCP service accounts for pod authentication
3. **Binary Authorization**: Enforce deploy-time security controls
4. **Network Policies**: Control pod-to-pod communication
5. **Resource Limits**: Set CPU and memory limits for containers
6. **Cluster Autoscaler**: Automatically adjust cluster size based on demand
7. **Pod Security Policies**: Enforce security standards for pods
8. **Regional Clusters**: Use multi-zone clusters for high availability

**[📖 GKE Best Practices](https://cloud.google.com/kubernetes-engine/docs/best-practices)** - Production-ready GKE configurations
**[📖 Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)** - Secure pod authentication with GCP services
**[📖 Cluster Autoscaling](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)** - Automatic cluster scaling

### App Engine Best Practices
1. **Environment Selection**: Choose Standard for simple apps, Flexible for custom runtimes
2. **Automatic Scaling**: Configure scaling settings based on traffic patterns
3. **Traffic Splitting**: Use for canary deployments and A/B testing
4. **Memcache**: Implement caching for improved performance
5. **Task Queues**: Offload background work to asynchronous tasks
6. **Version Management**: Deploy new versions before deleting old ones
7. **Warmup Requests**: Pre-load application code before serving traffic

**[📖 App Engine Standard vs Flexible](https://cloud.google.com/appengine/docs/the-appengine-environments)** - Choosing the right environment
**[📖 Scaling in App Engine](https://cloud.google.com/appengine/docs/standard/python3/how-instances-are-managed)** - Instance management and scaling

### Cloud Functions Best Practices
1. **Idempotency**: Design functions to handle duplicate events
2. **Cold Start Optimization**: Minimize function initialization time
3. **Concurrency Settings**: Configure concurrent executions per instance
4. **Timeouts**: Set appropriate timeout values for function execution
5. **Environment Variables**: Use for configuration, not secrets (use Secret Manager)
6. **Dependencies**: Minimize dependencies to reduce deployment size
7. **Background Functions**: Use Pub/Sub for asynchronous processing

### Cloud Run Best Practices
1. **Container Optimization**: Build minimal container images
2. **Concurrency**: Configure concurrent requests per container instance
3. **Cold Start Reduction**: Use minimum instances to keep containers warm
4. **Request Timeout**: Set appropriate timeout for your application
5. **CPU Allocation**: Choose always-allocated for latency-sensitive apps
6. **Service Accounts**: Use least-privilege service accounts
7. **Cloud SQL Connections**: Use Cloud SQL Proxy for database connections

**[📖 Cloud Run Best Practices](https://cloud.google.com/run/docs/tips)** - Optimizing Cloud Run services

## Common Scenarios

### Scenario 1: Web Application Deployment
**Requirement**: Deploy a scalable web application with auto-scaling
**Solution**:
- Use Compute Engine with managed instance groups
- Configure autoscaling based on CPU utilization
- Implement health checks for instance monitoring
- Use HTTP(S) load balancer for traffic distribution

### Scenario 2: Containerized Microservices
**Requirement**: Deploy and manage containerized microservices
**Solution**:
- Use GKE for container orchestration
- Create separate deployments for each microservice
- Implement Kubernetes services for internal communication
- Use Ingress for external access with load balancing

### Scenario 3: Serverless API
**Requirement**: Build a serverless REST API with minimal management
**Solution**:
- Use Cloud Functions for simple endpoints or Cloud Run for complex APIs
- Trigger via HTTP requests
- Implement authentication with Cloud IAM or API Gateway
- Connect to Cloud SQL or Firestore for data storage

### Scenario 4: Batch Processing
**Requirement**: Process large datasets periodically
**Solution**:
- Use preemptible VMs with Compute Engine for cost-effective processing
- Implement checkpointing to handle preemption gracefully
- Use instance templates and managed instance groups
- Schedule jobs with Cloud Scheduler

### Scenario 5: Legacy Application Migration
**Requirement**: Migrate existing on-premises application to cloud
**Solution**:
- Start with Compute Engine VMs (lift-and-shift)
- Use machine images to clone existing environments
- Implement persistent disks for data storage
- Gradually modernize to containers or serverless

## Study Tips

### Hands-On Practice
1. **Create and manage VMs**: Practice creating instances with different configurations
2. **Deploy to GKE**: Create a cluster, deploy applications, expose services
3. **Use App Engine**: Deploy applications in both Standard and Flexible environments
4. **Build Cloud Functions**: Create functions triggered by HTTP and Pub/Sub
5. **Deploy to Cloud Run**: Containerize an app and deploy to Cloud Run

### Key Concepts to Master
1. **Instance lifecycle**: Starting, stopping, resetting, deleting instances
2. **Scaling patterns**: Manual, automatic, and scheduled scaling
3. **Networking**: VPC networks, firewall rules, load balancing
4. **Storage options**: Persistent disks, local SSDs, Cloud Storage integration
5. **Pricing models**: On-demand, preemptible, committed use, sustained use

### Common Exam Topics
1. Choosing the right compute service for a given scenario
2. Configuring autoscaling for managed instance groups
3. Deploying applications to GKE clusters
4. Understanding App Engine environment differences
5. Implementing serverless functions with Cloud Functions and Cloud Run
6. Managing VM lifecycle and configurations
7. Implementing cost optimization strategies

### gcloud Command Examples

```bash
# Compute Engine
gcloud compute instances create my-instance --zone=us-central1-a --machine-type=e2-medium
gcloud compute instances list
gcloud compute instances stop my-instance --zone=us-central1-a
gcloud compute instances delete my-instance --zone=us-central1-a

# Create instance from template
gcloud compute instances create my-instance --source-instance-template=my-template

# GKE
gcloud container clusters create my-cluster --num-nodes=3 --zone=us-central1-a
gcloud container clusters get-credentials my-cluster --zone=us-central1-a
gcloud container clusters resize my-cluster --num-nodes=5 --zone=us-central1-a
gcloud container clusters delete my-cluster --zone=us-central1-a

# App Engine
gcloud app deploy app.yaml
gcloud app browse
gcloud app versions list
gcloud app services set-traffic default --splits=v1=0.5,v2=0.5

# Cloud Functions
gcloud functions deploy my-function --runtime=python39 --trigger-http --allow-unauthenticated
gcloud functions call my-function --data='{"name":"World"}'
gcloud functions delete my-function

# Cloud Run
gcloud run deploy my-service --image=gcr.io/PROJECT_ID/my-image --platform=managed --region=us-central1
gcloud run services list
gcloud run services delete my-service --region=us-central1
```

## Additional Resources

- [Compute Engine Documentation](https://cloud.google.com/compute/docs)
- [GKE Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [App Engine Documentation](https://cloud.google.com/appengine/docs)
- [Cloud Functions Documentation](https://cloud.google.com/functions/docs)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Choosing a Compute Option](https://cloud.google.com/docs/choosing-a-compute-option)
