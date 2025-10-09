# Implement Azure Security

## Overview
This domain covers implementing user authentication and authorization using Microsoft Identity Platform, and securing cloud solutions with Azure Key Vault and managed identities. It represents 20-25% of the exam and focuses on identity management and secrets protection.

## Key Topics

### User Authentication and Authorization
- Microsoft Identity Platform authentication and authorization
- Azure Active Directory integration for users and apps
- Shared Access Signature (SAS) creation and implementation
- Microsoft Graph API integration and usage

### Secure Cloud Solutions
- App Configuration and Azure Key Vault for configuration data
- Keys, secrets, and certificates management in Key Vault
- Managed Identities for Azure resources implementation

## Services Reference

### Core Services
- Microsoft Identity Platform
- Azure Active Directory (Azure AD)
- Azure Key Vault
- Azure App Configuration
- Microsoft Graph API

### Supporting Services
- Azure Managed Identity (System-assigned, User-assigned)
- Azure RBAC
- Azure AD B2C for customer identity
- Azure AD B2B for partner access

## Best Practices

### Identity and Access Management
- Use OAuth 2.0 and OpenID Connect for authentication
- Implement least privilege principle with RBAC
- Enable Multi-Factor Authentication (MFA)
- Use conditional access policies for risk-based access
- Implement proper token validation in applications

### Secrets Management
- Never store secrets in code or configuration files
- Use Key Vault for all secrets, keys, and certificates
- Implement managed identities to eliminate credentials in code
- Rotate secrets and keys regularly
- Enable soft delete and purge protection for Key Vault

### API Security
- Validate access tokens on every API call
- Use app roles for application-level authorization
- Implement API scopes for granular permissions
- Log authentication and authorization events

## Common Scenarios

### Authentication Flows
- Single-page application (SPA) authentication with MSAL.js
- Web application sign-in with OpenID Connect
- Service-to-service authentication with client credentials
- Mobile app authentication with MSAL libraries

### Secrets and Configuration Management
- Application retrieving database connection strings from Key Vault
- Certificate-based authentication for apps
- Managed identity accessing Key Vault without credentials
- Configuration updates without application redeployment

### Authorization Patterns
- Role-based access control (RBAC) implementation
- Claims-based authorization in applications
- API permission consent flow
- Microsoft Graph API calls for user data

## Study Tips

- Understand OAuth 2.0 and OpenID Connect flows thoroughly
- Practice implementing MSAL libraries in applications
- Learn the difference between delegated and application permissions
- Hands-on experience with Key Vault SDK operations
- Understand managed identity types and when to use each
- Study SAS token types and their use cases
- Practice Microsoft Graph API queries
- Learn token validation and claims extraction
- Understand Azure AD app registration configuration
