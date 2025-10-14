# Azure Active Directory - AZ-104 (Supplement to identity-governance.md)

**[📖 Azure Active Directory Documentation](https://learn.microsoft.com/en-us/azure/active-directory/)** - Cloud-based identity and access management service

## Azure AD Concepts

**[📖 What is Azure Active Directory?](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis)** - Introduction to Azure AD fundamentals
- Cloud-based identity and access management
- Different from on-premises AD
- Flat structure (no OUs, GPOs)

## Users and Groups

**[📖 Add or Delete Users](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/add-users-azure-active-directory)** - Manage Azure AD user accounts

**User Types**:
- Member: Native Azure AD user
- Guest: B2B collaboration (external user)

**Group Types**:

**[📖 Create a Basic Group and Add Members](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal)** - Manage groups in Azure AD

- Security: Access management
- Microsoft 365: Collaboration

**Assignment Types**:

**[📖 Dynamic Membership Rules](https://learn.microsoft.com/en-us/azure/active-directory/enterprise-users/groups-dynamic-membership)** - Automate group membership with rules

- Assigned: Manual membership
- Dynamic user: Rule-based user membership
- Dynamic device: Rule-based device membership

## Azure AD Connect

**[📖 Azure AD Connect Sync](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect)** - Integrate on-premises directories with Azure AD

- Sync on-premises AD to Azure AD
- Password hash synchronization
- Pass-through authentication
- Federation with AD FS

## Conditional Access

**[📖 What is Conditional Access?](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/overview)** - Implement zero trust security policies

- If-then policy statements
- Conditions: User, location, device, app, risk
- Controls: MFA, block, require compliant device
- Requires Azure AD Premium P1

## Self-Service Password Reset (SSPR)

**[📖 Plan SSPR Deployment](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-sspr-deployment)** - Enable users to reset their own passwords

- Users reset own passwords
- Verification methods: Email, SMS, security questions
- Writeback to on-premises AD (with Azure AD Connect)

## Azure AD Roles

**[📖 Azure AD Built-in Roles](https://learn.microsoft.com/en-us/azure/active-directory/roles/permissions-reference)** - Understand Azure AD administrator roles

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
