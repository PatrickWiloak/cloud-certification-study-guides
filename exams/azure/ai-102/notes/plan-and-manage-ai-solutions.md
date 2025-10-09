# Plan and Manage an Azure AI Solution

## Overview
This domain covers selecting appropriate AI services, planning security configurations, and creating and managing AI service resources. It represents 15-20% of the exam and focuses on foundational decisions for AI solution architecture.

## Key Topics

### Select Appropriate AI Service
- Computer vision service selection
- Natural language processing service selection
- Speech service selection
- Generative AI service selection
- Understanding service capabilities and limitations
- Choosing between custom and pre-built models

### Plan and Configure Security
- AI service security requirements planning
- AI service security configuration
- Access key and endpoint management
- Diagnostic logging configuration
- Network security for AI services
- Data privacy and compliance considerations

### Create and Configure AI Service
- AI service resource creation
- Endpoint configuration
- Keys and endpoint URL management
- Cost management and optimization
- AI service monitoring and metrics
- Resource scaling and performance tuning

## Services Reference

### Core AI Services
- Azure AI Vision (Computer Vision)
- Azure AI Language
- Azure AI Speech
- Azure OpenAI Service
- Azure AI Document Intelligence
- Azure Cognitive Search

### Supporting Services
- Azure Key Vault (for key management)
- Azure Monitor (for monitoring)
- Log Analytics
- Azure Virtual Networks
- Azure Private Link
- Managed Identity

## Best Practices

### Service Selection
- Use pre-built models when possible for faster deployment
- Choose custom models when domain-specific accuracy is needed
- Consider latency requirements for real-time scenarios
- Evaluate cost vs. performance trade-offs
- Plan for scalability and growth
- Consider regional availability of services

### Security Implementation
- Store API keys in Azure Key Vault
- Use managed identities instead of keys when possible
- Implement network restrictions with Private Link
- Enable diagnostic logging for audit trails
- Apply least privilege access principles
- Rotate keys regularly

### Resource Management
- Use appropriate pricing tiers for workload
- Implement cost alerts and budgets
- Monitor usage patterns and optimize
- Use single-region vs multi-region strategically
- Tag resources for cost tracking
- Plan for disaster recovery

### Performance Optimization
- Choose regions close to users for low latency
- Use appropriate batch sizes for throughput
- Implement caching for repeated requests
- Monitor throttling and adjust capacity
- Use async operations for better scalability
- Optimize request payloads

## Common Scenarios

### Service Deployment
- Multi-region deployment for high availability
- Development, test, and production environment setup
- Shared services across multiple applications
- Isolated services for compliance requirements
- Hybrid on-premises and cloud deployments

### Security Configuration
- Private endpoint setup for enhanced security
- Managed identity for secure service-to-service calls
- Key rotation without application downtime
- Audit logging for compliance requirements
- Data residency compliance for regulated industries

### Cost Optimization
- Choosing right pricing tier based on usage
- Implementing caching to reduce API calls
- Batching requests to improve efficiency
- Using commitment tiers for predictable workloads
- Monitoring and alerting on cost anomalies

### Monitoring and Troubleshooting
- Setting up Application Insights integration
- Creating custom metrics for business KPIs
- Configuring alerts for service health
- Analyzing usage patterns and trends
- Troubleshooting API errors and latency issues

## Study Tips

- Understand capabilities and limitations of each AI service
- Learn when to use custom vs pre-built models
- Practice creating AI services via Portal, CLI, and SDKs
- Study security best practices for AI services
- Understand pricing models and cost optimization strategies
- Learn about managed identities and their benefits
- Practice configuring diagnostic settings
- Study Private Link and VNet integration
- Understand regional availability and data residency
- Learn monitoring and alerting configuration
