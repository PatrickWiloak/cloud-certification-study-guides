# Implement Platform Protection

## Overview
This domain covers implementing advanced network security and configuring security for compute resources. It represents 35-40% of the exam and is the largest domain, focusing on defense-in-depth strategies for Azure infrastructure.

## Key Topics

### Advanced Network Security
- Hybrid network connectivity security
- Virtual network connectivity security
- Azure Firewall creation and configuration
- Azure Firewall Manager implementation
- Azure Application Gateway configuration
- Azure Front Door setup and configuration
- Web Application Firewall (WAF) implementation
- Resource firewall configuration
- Network Security Groups (NSGs) management
- Azure Service Endpoints implementation
- Azure Private Endpoints configuration
- Azure Private Links setup
- Azure DDoS Protection implementation

### Advanced Security for Compute
- Azure Endpoint Protection for VMs
- Security update management for VMs
- Container security configuration
- Vulnerability management for VMs and containers
- AKS isolation configuration
- Serverless compute security
- Azure App Service security
- Encryption at rest configuration
- Encryption in transit implementation

## Services Reference

### Network Security Services
- Azure Firewall
- Azure Firewall Manager
- Azure Application Gateway
- Azure Front Door
- Web Application Firewall (WAF)
- Network Security Groups (NSGs)
- Azure DDoS Protection
- Azure Bastion

### Compute Security Services
- Microsoft Defender for Servers
- Microsoft Defender for Containers
- Azure Disk Encryption
- Azure Key Vault
- Azure Kubernetes Service (AKS)
- Azure Update Management

## Best Practices

### Network Segmentation
- Use NSGs to control traffic between subnets
- Implement hub-and-spoke network topology
- Use Azure Firewall for centralized network filtering
- Deploy Application Gateway with WAF for web applications
- Implement zero-trust network architecture

### Perimeter Security
- Enable DDoS Protection Standard for public resources
- Use WAF to protect against OWASP Top 10 vulnerabilities
- Configure geo-filtering for content access control
- Implement rate limiting at application gateway
- Use Azure Front Door for global load balancing with security

### Private Connectivity
- Use Private Endpoints for PaaS services
- Disable public access when possible
- Implement Private Link for secure cross-subscription access
- Use service endpoints as cost-effective alternative
- Configure DNS for private endpoint resolution

### VM and Container Security
- Enable Microsoft Defender for Servers on all VMs
- Implement disk encryption for all OS and data disks
- Use managed identities instead of service principals
- Apply security baselines to VM configurations
- Scan container images for vulnerabilities before deployment

### AKS Security
- Enable Azure Policy for Kubernetes
- Implement network policies for pod isolation
- Use Azure AD integration for cluster access
- Enable pod security standards
- Scan container images in Azure Container Registry

## Common Scenarios

### Network Security
- Hub-and-spoke topology with Azure Firewall
- Application protection with WAF and DDoS Protection
- Secure hybrid connectivity with VPN or ExpressRoute
- Micro-segmentation with NSGs and ASGs
- Zero-trust network with Private Endpoints

### Compute Protection
- VM vulnerability management and patching
- Container security scanning and runtime protection
- Serverless function security hardening
- Disk encryption for compliance requirements
- Secure container orchestration with AKS

### Application Security
- Web application protection with Application Gateway WAF
- API security with API Management and WAF
- Multi-region application protection with Front Door
- Certificate management for SSL/TLS
- Secret management for applications

## Study Tips

- Understand NSG rule evaluation order and precedence
- Practice configuring Azure Firewall rules (application, network, NAT)
- Learn WAF rule sets and custom rule creation
- Hands-on experience with Private Endpoint configuration
- Study the difference between Service Endpoints and Private Endpoints
- Practice VM disk encryption with Key Vault
- Understand AKS security features and Azure Policy integration
- Learn container scanning and vulnerability management
- Study DDoS Protection tiers and capabilities
- Practice configuring Azure Bastion for secure VM access
