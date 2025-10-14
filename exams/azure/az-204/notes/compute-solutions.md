# Develop Azure Compute Solutions

## Overview
This domain covers the implementation of Infrastructure as a Service (IaaS) solutions, Azure App Service web apps, and Azure Functions. It represents 25-30% of the exam and focuses on deploying and managing compute resources for cloud applications.

**[📖 AZ-204 Exam Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-204)** - Official exam study guide from Microsoft

## Key Topics

### IaaS Solutions
- Provisioning and configuring virtual machines
- ARM template configuration, validation, and deployment
- Container image configuration for solutions
- Azure Container Registry operations
- Azure Container Instance deployment and management

**[📖 Azure Virtual Machines Documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/)** - Comprehensive guide to Azure VMs
**[📖 ARM Templates Overview](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)** - Azure Resource Manager template documentation
**[📖 Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/)** - ACR documentation and best practices

### Azure App Service Web Apps
- Creating and configuring App Service web apps
- Diagnostic logging enablement and analysis
- Code deployment strategies and methods
- Web app settings configuration (SSL, API, connection strings)
- Autoscaling rules implementation (scheduled and metric-based)

**[📖 Azure App Service Documentation](https://learn.microsoft.com/en-us/azure/app-service/)** - Complete App Service documentation
**[📖 App Service Deployment Best Practices](https://learn.microsoft.com/en-us/azure/app-service/deploy-best-practices)** - Deployment strategies and recommendations
**[📖 App Service Autoscaling](https://learn.microsoft.com/en-us/azure/app-service/manage-scale-up)** - Scaling and autoscale configuration

### Azure Functions
- Function app creation and deployment
- Input and output bindings configuration
- Trigger implementation (data operations, timers, webhooks)
- Azure Durable Functions patterns
- Custom handlers implementation

**[📖 Azure Functions Documentation](https://learn.microsoft.com/en-us/azure/azure-functions/)** - Complete guide to Azure Functions
**[📖 Azure Functions Triggers and Bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings)** - Comprehensive bindings reference
**[📖 Azure Durable Functions](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview)** - Stateful functions and orchestration patterns

## Services Reference

### Core Services
- Azure Virtual Machines
- Azure Container Registry (ACR)
- Azure Container Instances (ACI)
- Azure App Service
- Azure Functions
- Azure Kubernetes Service (AKS)

**[📖 Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/)** - Serverless container deployment documentation
**[📖 Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/aks/)** - Managed Kubernetes cluster documentation

### Supporting Services
- Azure Resource Manager (ARM)
- Azure CLI and PowerShell
- Azure DevOps / GitHub Actions
- Application Insights

## Best Practices

### VM and Container Management
- Use managed disks for reliability
- Implement proper tagging strategy for resources
- Leverage Azure Container Registry for private container storage
- Use staging slots for web app deployments

### App Service Configuration
- Store connection strings and secrets in Key Vault
- Enable Always On for production apps
- Configure health checks for reliability
- Implement deployment slots for zero-downtime deployments

### Azure Functions Development
- Choose appropriate hosting plan (Consumption, Premium, Dedicated)
- Implement proper error handling and retry logic
- Use durable functions for stateful workflows
- Monitor function execution and performance

## Common Scenarios

### Deployment Scenarios
- Blue-green deployments using App Service slots
- Containerized application deployment to ACI or AKS
- Serverless event-driven processing with Functions
- Microservices architecture using containers

### Scaling Scenarios
- Horizontal scaling with App Service autoscale rules
- Function app scaling based on queue depth
- Container orchestration with AKS
- Performance optimization for high-traffic applications

## Study Tips

- Practice deploying applications using multiple methods (Portal, CLI, ARM templates)
- Understand the differences between hosting plans and when to use each
- Hands-on experience with ARM template creation and validation is crucial
- Learn function bindings and triggers thoroughly
- Practice implementing autoscaling rules with different metrics
- Understand container image workflows from build to deployment
