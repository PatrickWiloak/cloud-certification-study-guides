# IBM Cloud Security Engineer (C1000-178) - Practice Plan

## Exam Overview
**Duration**: 90 minutes | **Questions**: 60-70 | **Passing**: 70% | **Format**: Multiple choice/select

### Exam Sections
1. **IAM & Access Control** (30%) - Users, policies, service IDs, trusted profiles
2. **Network Security** (25%) - VPC, ACLs, security groups, VPN, Direct Link
3. **Data Protection** (20%) - Encryption, Key Protect, Secrets Manager
4. **Compliance & Auditing** (15%) - SCC, Activity Tracker, compliance frameworks
5. **Security Operations** (10%) - Incident response, vulnerability management

## 8-Week Study Schedule

### Week 1-2: IAM & Access Management
- Day 1-3: IAM fundamentals, roles, policies
- Day 4-5: Access groups, dynamic rules
- Day 6-7: Service IDs, API keys
- Day 8-10: Trusted profiles, context-based restrictions
- **Lab**: Implement least-privilege access for multi-tier application

### Week 3-4: Network Security
- Day 1-3: VPC security architecture, subnets
- Day 4-5: ACLs vs Security Groups
- Day 6-7: VPN configurations, Direct Link
- Day 8-10: Load balancers, DDoS protection, WAF
- **Lab**: Build secure multi-zone VPC with proper segmentation

### Week 5: Data Protection & Encryption
- Day 1-2: Key Protect, KYOK, BYOK
- Day 3-4: Secrets Manager integration
- Day 5: Certificate Manager
- Day 6-7: Encryption at rest and in transit
- **Lab**: Implement end-to-end encryption for application

### Week 6: Compliance & Governance
- Day 1-3: Security and Compliance Center
- Day 4-5: Activity Tracker, audit logging
- Day 6-7: Compliance frameworks (ISO, SOC, PCI DSS)
- **Lab**: Set up compliance monitoring and reporting

### Week 7: Security Operations
- Day 1-2: Vulnerability assessment, container security
- Day 3-4: Incident response, Security Advisor
- Day 5: Backup and disaster recovery
- Day 6-7: Business continuity planning
- **Lab**: Create incident response playbook and DR solution

### Week 8: Review & Practice
- Day 1-3: Review all notes, focus on weak areas
- Day 4-5: Complete practice exams
- Day 6: Final review, exam strategies
- Day 7: Exam day

## Key Commands

### IAM
```bash
ibmcloud iam user-policy-create user@example.com --roles Editor --service-name containers-kubernetes
ibmcloud iam access-group-create developers
ibmcloud iam service-id-create app-service-id
ibmcloud iam trusted-profile-create app-profile
```

### Network Security
```bash
ibmcloud is vpc-create secure-vpc
ibmcloud is security-group-create web-sg
ibmcloud is network-acl-rule-add web-acl allow inbound tcp
ibmcloud is vpn-gateway-create my-vpn --subnet $SUBNET_ID
```

### Data Protection
```bash
ibmcloud kp key-create root-key --instance-id $INSTANCE_ID
ibmcloud secrets-manager secret-create --secret-type arbitrary
ibmcloud certificate-manager certificate-import
```

### Compliance
```bash
ibmcloud scc profile-create --name "Custom Profile"
ibmcloud atracker route create --name audit-route
ibmcloud security-advisor findings list
```

## Practice Questions

1. Which IAM feature allows compute resources to authenticate without API keys?
   - A) Service IDs
   - B) Trusted Profiles (CORRECT)
   - C) Access Groups
   - D) API Keys

2. What is the difference between ACLs and Security Groups?
   - ACLs: Stateless, subnet-level, numbered rules
   - Security Groups: Stateful, instance-level, allow rules only

3. Which service provides centralized compliance monitoring?
   - A) Activity Tracker
   - B) Security and Compliance Center (CORRECT)
   - C) Security Advisor
   - D) Key Protect

## Success Tips
- Master IAM policy syntax and conditions
- Understand VPC security layer differences
- Know encryption key hierarchy (root, standard, data)
- Practice with Terraform for infrastructure as code
- Study real-world incident response scenarios
- Focus on IBM Cloud-specific security features

## Resources
- IBM Cloud Security Architecture Center
- IBM Cloud VPC documentation
- Key Protect and Secrets Manager guides
- Security and Compliance Center tutorials
- IBM Cloud CLI reference
