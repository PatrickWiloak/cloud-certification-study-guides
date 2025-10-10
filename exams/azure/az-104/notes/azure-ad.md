# Azure Active Directory - AZ-104 (Supplement to identity-governance.md)

## Azure AD Concepts
- Cloud-based identity and access management
- Different from on-premises AD
- Flat structure (no OUs, GPOs)

## Users and Groups
**User Types**:
- Member: Native Azure AD user
- Guest: B2B collaboration (external user)

**Group Types**:
- Security: Access management
- Microsoft 365: Collaboration

**Assignment Types**:
- Assigned: Manual membership
- Dynamic user: Rule-based user membership
- Dynamic device: Rule-based device membership

## Azure AD Connect
- Sync on-premises AD to Azure AD
- Password hash synchronization
- Pass-through authentication
- Federation with AD FS

## Conditional Access
- If-then policy statements
- Conditions: User, location, device, app, risk
- Controls: MFA, block, require compliant device
- Requires Azure AD Premium P1

## Self-Service Password Reset (SSPR)
- Users reset own passwords
- Verification methods: Email, SMS, security questions
- Writeback to on-premises AD (with Azure AD Connect)

## Azure AD Roles
- Global Administrator: Full access
- User Administrator: Manage users and groups
- Password Administrator: Reset passwords
- Custom roles (P1/P2)

## Exam Tips
- Azure AD Connect for hybrid identity
- SSPR reduces helpdesk calls
- Conditional Access for zero-trust security
- Dynamic groups for automatic membership
- Guest users for B2B collaboration
- Managed identities for Azure resources
