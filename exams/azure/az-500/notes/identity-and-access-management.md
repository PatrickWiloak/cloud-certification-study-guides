# Manage Identity and Access

## Overview
This domain covers configuring Azure Active Directory for workloads, implementing Privileged Identity Management, and managing Azure tenant security. It represents 25-30% of the exam and is fundamental to securing Azure resources and applications.

## Key Topics

### Azure Active Directory for Workloads
- App registration creation and configuration
- App registration permission scopes management
- Permission consent management
- Multi-Factor Authentication (MFA) configuration
- Azure AD directory groups and users management
- Azure AD Connect installation and configuration
- Authentication methods configuration
- Conditional Access policies implementation
- Azure AD Identity Protection

### Privileged Identity Management (PIM)
- Monitoring privileged access
- Access Reviews configuration
- PIM activation and approval workflows
- Just-in-time (JIT) access implementation
- Role assignment time-bound access

### Azure Tenant Security
- Subscription transfers between Azure AD tenants
- API access management to subscriptions and resources
- Azure RBAC configuration
- Custom role definitions

## Services Reference

### Core Services
- Azure Active Directory (Azure AD)
- Azure AD Connect
- Azure AD Privileged Identity Management (PIM)
- Azure AD Identity Protection
- Conditional Access

### Supporting Services
- Azure AD B2C (Customer identity)
- Azure AD B2B (Partner collaboration)
- Azure AD Domain Services
- Microsoft Defender for Identity

## Best Practices

### Identity Security
- Enable MFA for all users, especially administrators
- Implement passwordless authentication where possible
- Use managed identities for service-to-service authentication
- Follow least privilege principle for role assignments
- Regularly review and revoke unnecessary permissions

### Conditional Access
- Create policies based on user risk and sign-in risk
- Implement device compliance requirements
- Configure location-based access restrictions
- Block legacy authentication protocols
- Use report-only mode before enforcing policies

### Privileged Access Management
- Require approval for privileged role activation
- Set maximum activation duration appropriately
- Require justification for role activation
- Enable MFA for privileged role activation
- Conduct regular access reviews for privileged roles

### Application Security
- Use app roles for application-level permissions
- Implement admin consent workflow for apps
- Regularly audit app permissions and API access
- Revoke unused app registrations
- Use certificate credentials for production apps

## Common Scenarios

### Identity Management
- Hybrid identity with Azure AD Connect
- SSO implementation for SaaS applications
- Guest user access management (B2B)
- Customer identity and access management (B2C)
- Service principal management for automation

### Access Control
- Role-based access control (RBAC) implementation
- Just-in-time administrative access
- Emergency access account configuration
- Cross-tenant resource access
- Delegated administration with administrative units

### Security Monitoring
- Sign-in risk detection and response
- User risk detection and remediation
- Risky sign-in investigation
- Identity Protection policy configuration
- Security alerts and notifications

## Study Tips

- Understand Azure AD app registration components (App ID, Tenant ID, endpoints)
- Practice configuring Conditional Access policies with various conditions
- Learn PIM activation workflow and approval process
- Hands-on experience with Azure AD Connect configuration
- Study the difference between Azure RBAC and Azure AD roles
- Practice creating custom RBAC roles
- Understand identity protection risk levels and policies
- Learn MFA methods and their security levels
- Study OAuth 2.0 and OpenID Connect flows
- Practice Access Reviews configuration and approval
