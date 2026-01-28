# GCP Professional Cloud Security Engineer Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Configuring access within a cloud solution environment | 26% | 10 |
| Managing operations | 22% | 9 |
| Configuring network security | 20% | 8 |
| Ensuring compliance | 14% | 6 |
| Securing data | 18% | 7 |

---

## Domain 1: Configuring Access Within a Cloud Solution Environment (Questions 1-10)

### Question 1
**Scenario:** A company needs to implement a least privilege access model for their Google Cloud environment. They have 500 employees across multiple teams who need different levels of access to various projects. What approach should they use to manage IAM efficiently?

A. Assign IAM roles directly to individual user accounts
B. Use Google Groups for role assignments, mapping teams to groups with appropriate roles at folder/project level
C. Give all users Owner role and train them on responsibility
D. Create custom roles for each individual user

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Google Groups simplify IAM management at scale - adding/removing group membership automatically updates access. Groups can be mapped to organizational teams. Roles assigned to groups at folder level inherit to child projects. Individual assignments (A) become unmanageable at scale. Owner for all (C) violates least privilege. Per-user custom roles (D) create management overhead.

**Key Concept:** [IAM Best Practices](https://cloud.google.com/iam/docs/using-iam-securely)
</details>

### Question 2
**Scenario:** A development team needs temporary elevated access to production projects for troubleshooting. The access should expire automatically, require approval, and be fully audited. What solution provides this?

A. Permanently assign elevated roles to developers
B. Privileged Access Manager (PAM) with just-in-time access grants requiring approval and time-bound duration
C. Share service account keys when needed
D. Temporarily add developers to admin group

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Privileged Access Manager enables just-in-time, time-bound access with approval workflows. Access is automatically revoked after the specified duration. All grants are logged for audit. Permanent elevated access (A) violates least privilege. Service account keys (C) lack user attribution and are hard to revoke. Manual group membership (D) is error-prone and may not be revoked.

**Key Concept:** [Privileged Access Manager](https://cloud.google.com/iam/docs/privileged-access-manager)
</details>

### Question 3
**Scenario:** An application running on GKE needs to access Cloud Storage and Pub/Sub. The security team requires that no service account keys be used and that each workload has its own identity. How should this be configured?

A. Mount service account key files into pods
B. Use Workload Identity to bind Kubernetes service accounts to Google service accounts
C. Use the default compute service account for all pods
D. Hardcode service account credentials in container images

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Workload Identity provides keyless authentication by binding Kubernetes service accounts to Google Cloud service accounts. Each workload can have its own GSA with specific permissions. No key management required. Key files (A, D) risk exposure and require rotation. Default service account (C) violates least privilege and is overly permissioned.

**Key Concept:** [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
</details>

### Question 4
**Scenario:** A company uses multiple identity providers (corporate AD, partner IdPs) and needs users from all providers to access Google Cloud with appropriate roles. How should identity federation be configured?

A. Create Google accounts for all external users
B. Configure Workforce Identity Federation with pools for each IdP, mapping attributes to IAM roles
C. Share a single service account across organizations
D. Replicate user directories into Google Cloud

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Workforce Identity Federation allows external IdPs to authenticate users for Google Cloud access. Identity pools group users from external IdPs. Attribute mappings can translate IdP claims to IAM conditions for role assignment. Creating Google accounts (A) duplicates identity management. Shared service accounts (C) lack user attribution. Replication (D) creates sync overhead.

**Key Concept:** [Workforce Identity Federation](https://cloud.google.com/iam/docs/workforce-identity-federation)
</details>

### Question 5
**Scenario:** A CI/CD pipeline running in an external system (GitHub Actions) needs to deploy to Google Cloud. The security team prohibits long-lived service account keys. What authentication method should be used?

A. Store service account key in GitHub secrets
B. Configure Workload Identity Federation for GitHub Actions with short-lived token exchange
C. Manually approve each deployment
D. Use interactive user authentication

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Workload Identity Federation for external workloads allows GitHub Actions to exchange its OIDC token for short-lived Google Cloud credentials. No long-lived keys to manage or rotate. Service account key in secrets (A) uses long-lived credentials. Manual approval (C) slows automation. Interactive auth (D) doesn't work for automated pipelines.

**Key Concept:** [Workload Identity Federation for External Workloads](https://cloud.google.com/iam/docs/workload-identity-federation)
</details>

### Question 6
**Scenario:** A healthcare organization needs to ensure that only users accessing from their corporate network or managed devices can access sensitive patient data in BigQuery. How should this be enforced?

A. Use VPC Service Controls only
B. Configure context-aware access with access levels requiring corporate network IP or device trust, then apply in IAM conditions
C. Trust all authenticated users
D. Implement network firewall rules only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** BeyondCorp Enterprise context-aware access defines access levels based on device trust, network location, or other signals. These access levels can be used in IAM conditions to restrict BigQuery access. This implements zero trust - identity alone isn't sufficient. VPC controls (A) restrict network paths, not user context. Trusting all users (C) ignores device security. Firewall rules (D) don't restrict authenticated API access.

**Key Concept:** [Context-Aware Access](https://cloud.google.com/beyondcorp-enterprise/docs/securing-resources-with-access-levels)
</details>

### Question 7
**Scenario:** An organization wants to ensure that no user can grant themselves additional IAM permissions, even if they have IAM admin capabilities in their project. How can this be enforced?

A. Trust admins to not escalate privileges
B. Use Organization Policy constraints to restrict IAM role bindings, or implement custom constraints
C. Don't grant any IAM permissions
D. Review permissions quarterly

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Organization Policy can restrict which roles can be granted (e.g., deny granting Owner role) across the organization. Custom constraints can implement more specific restrictions like preventing self-assignment. This provides technical enforcement. Trust (A) is not verification. No permissions (C) prevents all work. Quarterly review (D) is reactive, not preventive.

**Key Concept:** [Custom Organization Policy Constraints](https://cloud.google.com/resource-manager/docs/organization-policy/creating-managing-custom-constraints)
</details>

### Question 8
**Scenario:** A company's security policy requires that service accounts used by production workloads cannot be impersonated by developers. Only automated systems should use production service accounts. How should this be enforced?

A. Don't share service account emails
B. Configure IAM deny policies to prevent developer principals from having iam.serviceAccounts.actAs permission on production service accounts
C. Verbal policy enforcement
D. Audit impersonation after the fact

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM deny policies take precedence over allow policies. A deny policy preventing developers from actAs permission on production service accounts cannot be overridden. This prevents impersonation even if a developer has broad permissions elsewhere. Obscurity (A) isn't security. Verbal policies (C) aren't enforceable. Post-facto audit (D) doesn't prevent the action.

**Key Concept:** [IAM Deny Policies](https://cloud.google.com/iam/docs/deny-overview)
</details>

### Question 9
**Scenario:** An application needs to access secrets stored in Secret Manager. Different environments (dev, staging, prod) have their own secrets, and the application should only access secrets for its environment. How should access be configured?

A. Grant access to all secrets and trust the application to access only appropriate ones
B. Use separate service accounts per environment with IAM permissions scoped to specific secret resource paths
C. Store all secrets in environment variables
D. Use a single secret with environment-specific JSON keys

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM permissions on secrets can be scoped to specific resources (e.g., `secretmanager.secretAccessor` on `projects/*/secrets/prod-*`). Each environment's service account gets permissions only for its secrets. Trust (A) relies on application behavior, not enforcement. Environment variables (C) expose secrets in process listings. Single secret (D) gives access to all environments.

**Key Concept:** [Secret Manager IAM](https://cloud.google.com/secret-manager/docs/access-control)
</details>

### Question 10
**Scenario:** A company needs to provision Google Cloud access for employees during onboarding and revoke access immediately upon termination. They use Okta as their IdP. What approach ensures timely provisioning and deprovisioning?

A. Manual account creation and deletion
B. Configure Cloud Identity or Workspace with Okta SCIM provisioning for automated lifecycle management
C. Create Google accounts separate from Okta
D. Weekly batch synchronization

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SCIM (System for Cross-domain Identity Management) automates user provisioning and deprovisioning. When a user is deactivated in Okta, Cloud Identity is immediately updated, revoking Google Cloud access. This ensures no orphaned access. Manual processes (A) are slow and error-prone. Separate accounts (C) require duplicate management. Weekly sync (D) leaves access active too long after termination.

**Key Concept:** [Cloud Identity Provisioning](https://cloud.google.com/architecture/identity/federating-gcp-with-okta)
</details>

---

## Domain 2: Managing Operations (Questions 11-19)

### Question 11
**Scenario:** A security team needs to detect suspicious activities across all Google Cloud projects in their organization, including unusual API access patterns, data exfiltration attempts, and potential compromised credentials. What service provides this capability?

A. Cloud Logging with custom queries
B. Security Command Center Premium with Event Threat Detection
C. Cloud Monitoring alerts
D. Manual log review

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Security Command Center (SCC) Premium includes Event Threat Detection which analyzes audit logs for threats including cryptomining, data exfiltration, IAM anomalies, and compromised credentials. It covers the entire organization with no setup per project. Custom logging queries (A) require building detection rules. Monitoring (C) is for metrics, not threat detection. Manual review (D) doesn't scale.

**Key Concept:** [Event Threat Detection](https://cloud.google.com/security-command-center/docs/concepts-event-threat-detection-overview)
</details>

### Question 12
**Scenario:** During a security incident, the team needs to investigate what a compromised service account did over the past week. They need to see all API calls made, resources accessed, and any errors. Where should they look?

A. Application logs
B. Cloud Audit Logs (Admin Activity and Data Access logs) filtered by the service account principal
C. Cloud Monitoring metrics
D. VPC Flow Logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Audit Logs capture API calls with principal identity, resource accessed, action taken, and result. Admin Activity logs (always on) capture config changes. Data Access logs (must be enabled) capture data reads/writes. Filter by `protoPayload.authenticationInfo.principalEmail` for the service account. Application logs (A) may not capture all API calls. Metrics (C) are aggregated. Flow logs (D) show network traffic, not API calls.

**Key Concept:** [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit)
</details>

### Question 13
**Scenario:** A company needs to ensure that their Google Cloud resources comply with CIS Benchmarks. They want continuous monitoring with automatic notification of compliance violations. What service provides this?

A. Manual compliance audits
B. Security Command Center with Security Health Analytics comparing against CIS Benchmarks
C. Penetration testing
D. Cloud Monitoring dashboards

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Security Health Analytics in SCC continuously scans for misconfigurations against security best practices including CIS Benchmarks. Findings are generated for violations with severity ratings. Notifications can be configured via Pub/Sub or email. Manual audits (A) are point-in-time. Pen testing (C) finds vulnerabilities, not compliance gaps. Monitoring (D) doesn't assess compliance.

**Key Concept:** [Security Health Analytics](https://cloud.google.com/security-command-center/docs/concepts-security-health-analytics-overview)
</details>

### Question 14
**Scenario:** A security analyst needs to investigate the attack path an adversary could take from an internet-exposed VM to sensitive data in BigQuery. What Security Command Center feature helps visualize this?

A. Vulnerability scanning
B. Attack Path Simulation showing possible paths from exposed assets to high-value resources
C. Asset inventory
D. Compliance reporting

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SCC Premium's Attack Path Simulation identifies paths attackers could take to reach high-value resources. It visualizes how an internet-exposed VM could potentially access BigQuery through IAM relationships, network paths, and service account chains. This enables proactive hardening. Vulnerability scanning (A) finds CVEs, not attack paths. Asset inventory (C) lists resources. Compliance (D) checks configurations.

**Key Concept:** [Attack Path Simulation](https://cloud.google.com/security-command-center/docs/concepts-attack-path-simulation)
</details>

### Question 15
**Scenario:** A company needs to ensure all access to production data in BigQuery is logged, including which datasets and tables were queried, by whom, and what data was returned. How should this be configured?

A. Enable Admin Activity audit logs only
B. Enable Data Access audit logs for BigQuery with ADMIN_READ and DATA_READ log types
C. Query BigQuery job history only
D. Application-level logging

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Data Access audit logs for BigQuery capture metadata reads (ADMIN_READ) and data reads (DATA_READ). They log query executions, datasets accessed, and caller identity. These logs are not enabled by default and must be explicitly configured. Admin Activity (A) only captures management operations. Job history (C) is limited and doesn't show all access patterns. Application logging (D) misses console/CLI queries.

**Key Concept:** [BigQuery Audit Logs](https://cloud.google.com/bigquery/docs/reference/auditlogs)
</details>

### Question 16
**Scenario:** An organization wants to ensure that when security findings are detected in SCC, relevant teams are automatically notified and tickets are created in their tracking system. How should this be automated?

A. Check SCC dashboard daily
B. Configure SCC notification config to Pub/Sub, triggering Cloud Functions to create tickets and send notifications
C. Email monthly reports
D. Manual finding review

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SCC notification configs export findings to Pub/Sub in real-time. Cloud Functions subscribed to Pub/Sub can filter findings, create tickets in Jira/ServiceNow, and send Slack/email notifications. This enables immediate response. Dashboard checking (A) introduces delay. Monthly reports (C) are too slow for security issues. Manual review (D) doesn't scale.

**Key Concept:** [SCC Notifications](https://cloud.google.com/security-command-center/docs/how-to-notifications)
</details>

### Question 17
**Scenario:** A company needs to detect if any of their GKE containers are running with known vulnerabilities. They want continuous scanning with integration into their security workflow. What should be enabled?

A. Manual container inspection
B. GKE Security Posture with container vulnerability scanning integrated with SCC
C. Network-level scanning only
D. Annual penetration tests

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GKE Security Posture provides continuous vulnerability scanning of container images. Findings appear in Security Command Center for unified security view. This covers both Artifact Registry images and running workloads. Manual inspection (A) doesn't scale. Network scanning (C) doesn't find container CVEs. Annual tests (D) miss vulnerabilities between tests.

**Key Concept:** [GKE Security Posture](https://cloud.google.com/kubernetes-engine/docs/concepts/security-posture)
</details>

### Question 18
**Scenario:** During incident response, the team needs to quickly identify all resources that could have been affected by a compromised service account. The account had Viewer role at the project level. What tool helps identify the blast radius?

A. List all resources manually
B. Use Policy Analyzer to show all resources the service account can access based on effective permissions
C. Check only the resources mentioned in logs
D. Assume all resources are affected

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Policy Analyzer evaluates effective permissions - given a principal and permission, it shows all resources they can access. This reveals the actual blast radius based on IAM inheritance and conditions. Manual listing (A) misses resources. Log-based (C) misses resources that weren't accessed but could have been. Assuming all (D) overestimates scope.

**Key Concept:** [Policy Analyzer](https://cloud.google.com/policy-intelligence/docs/analyze-iam-policies)
</details>

### Question 19
**Scenario:** A security team needs to review who has access to sensitive GCS buckets and whether any permissions are overly broad. They want recommendations for removing unused permissions. What tool provides this analysis?

A. Manual permission review
B. IAM Recommender analyzing actual usage patterns to suggest removing unused permissions
C. Grant minimum permissions and never review
D. Remove all permissions and add back as needed

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM Recommender analyzes actual permission usage over time and suggests removing permissions that haven't been used. This data-driven approach ensures least privilege based on real behavior. Manual review (A) is time-consuming and doesn't know actual usage. Static minimum (C) doesn't adapt to changing needs. Removing all (D) causes outages.

**Key Concept:** [IAM Recommender](https://cloud.google.com/iam/docs/recommender-overview)
</details>

---

## Domain 3: Configuring Network Security (Questions 20-27)

### Question 20
**Scenario:** A company wants to ensure that their Cloud Storage and BigQuery data cannot be exfiltrated to unauthorized projects, even by users with data access permissions. Data should only be accessible from within approved projects. What security control addresses this?

A. IAM permissions only
B. VPC Service Controls with a perimeter around sensitive projects blocking data egress
C. Firewall rules
D. Encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** VPC Service Controls create a security perimeter around Google Cloud resources. Data cannot be copied or accessed from outside the perimeter even with valid IAM permissions. This prevents exfiltration via `gsutil cp` to external buckets or BigQuery copies to unauthorized projects. IAM (A) controls who, not where. Firewall (C) is for VM networking. Encryption (D) protects data at rest, not exfiltration.

**Key Concept:** [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs/overview)
</details>

### Question 21
**Scenario:** An organization has a partner company that needs to access specific BigQuery datasets. Both companies have Google Cloud environments. The partner should not have access to any other resources. How should this cross-organization access be configured with VPC Service Controls?

A. Add partner project inside the perimeter
B. Configure an egress rule allowing access to specific BigQuery resources for the partner's perimeter, using an ingress rule on your perimeter
C. Disable VPC Service Controls
D. Copy data to partner's project

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** VPC Service Controls ingress/egress rules enable controlled cross-perimeter access. An ingress rule on your perimeter allows the partner's perimeter to access specified BigQuery datasets. An egress rule on the partner's perimeter allows their principals to access your resources. This maintains security while enabling collaboration. Adding to perimeter (A) grants too much access. Disabling (C) removes protection. Copying data (D) loses control.

**Key Concept:** [VPC-SC Ingress/Egress Rules](https://cloud.google.com/vpc-service-controls/docs/ingress-egress-rules)
</details>

### Question 22
**Scenario:** A web application running on GKE needs to be protected from common web attacks (SQL injection, XSS, OWASP Top 10). The application uses external HTTPS load balancing. What security control should be implemented?

A. Network-level firewalls only
B. Cloud Armor with WAF rules (OWASP ModSecurity Core Rule Set) attached to the load balancer
C. Application-level input validation only
D. VPC firewall rules for HTTP filtering

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Armor provides web application firewall (WAF) capabilities at the load balancer level. Pre-configured WAF rules based on OWASP ModSecurity CRS protect against SQL injection, XSS, and other OWASP Top 10 threats. Network firewalls (A, D) are layer 4 and can't inspect HTTP content. Application validation (C) is good defense-in-depth but Cloud Armor provides perimeter protection.

**Key Concept:** [Cloud Armor WAF](https://cloud.google.com/armor/docs/waf-rules)
</details>

### Question 23
**Scenario:** An organization wants to prevent users from creating public IP addresses on VMs and ensure all outbound traffic goes through a centralized firewall appliance. How should this be enforced?

A. Document the requirement in security policy
B. Organization Policy constraint preventing external IPs, with VPC routing all egress through Cloud NAT or firewall appliance
C. Review firewall logs for violations
D. Trust teams to follow guidelines

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Organization Policy `constraints/compute.vmExternalIpAccess` can deny all VMs external IPs. VPC routes can direct 0.0.0.0/0 traffic to a firewall appliance (Network Virtual Appliance) or through Cloud NAT for controlled egress. This provides technical enforcement. Documentation (A) and trust (D) aren't enforceable. Log review (C) is reactive.

**Key Concept:** [External IP Org Policy](https://cloud.google.com/resource-manager/docs/organization-policy/restricting-external-ips)
</details>

### Question 24
**Scenario:** A company needs to inspect all traffic between VPCs in their hub-and-spoke network architecture for threat detection and compliance logging. Traffic includes both north-south (internet) and east-west (VPC to VPC). What architecture supports this?

A. Firewall rules between VPCs
B. Cloud NGFW (Cloud Firewall Plus) with intrusion detection and traffic inspection at network choke points
C. VPC peering only
D. Network-level ACLs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud NGFW (next-generation firewall) provides deep packet inspection, intrusion detection (IDS/IPS), and TLS inspection. Deploying at network hub enables inspection of all traffic between spokes. This supports both security monitoring and compliance requirements. Basic firewall rules (A, D) are stateless/stateful but don't inspect payloads. Peering alone (C) doesn't provide inspection.

**Key Concept:** [Cloud NGFW](https://cloud.google.com/firewall/docs/about-firewalls)
</details>

### Question 25
**Scenario:** Developers need to access GKE cluster APIs for kubectl commands. The cluster should not have a public endpoint. Developers work remotely without VPN access to the VPC. How should secure access be provided?

A. Enable public cluster endpoint
B. Use Connect Gateway to access private GKE clusters through Cloud Identity authentication
C. VPN for all developers
D. SSH tunneling through a bastion

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Connect Gateway provides authenticated access to private GKE clusters without exposing cluster endpoints publicly or requiring VPN. Authentication uses Google Identity with IAM for authorization. This enables secure remote access with identity-based controls. Public endpoint (A) exposes the cluster. VPN (C) is complex to manage for all developers. Bastion SSH (D) is less direct and requires additional infrastructure.

**Key Concept:** [Connect Gateway](https://cloud.google.com/anthos/multicluster-management/gateway)
</details>

### Question 26
**Scenario:** A financial services company needs to ensure that API calls to Google Cloud services from their VPC never traverse the public internet, even for global services like BigQuery and Cloud Storage. How should this be configured?

A. Accept that some traffic goes over internet
B. Enable Private Google Access with restricted.googleapis.com VIP for all Google API traffic
C. Use VPN to Google Cloud
D. Proxy all API calls through a VM

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Private Google Access with the restricted.googleapis.com VIP (199.36.153.4/30) routes all Google API traffic through Google's private network, never touching the public internet. Combined with VPC Service Controls, this ensures complete private connectivity. Default googleapis.com uses public IPs. VPN (C) doesn't cover Google API traffic. Proxying (D) adds complexity and latency.

**Key Concept:** [Private Google Access](https://cloud.google.com/vpc/docs/configure-private-google-access)
</details>

### Question 27
**Scenario:** A company hosts customer-facing APIs on Cloud Run. They need to rate limit requests per customer, block requests from known bad actors, and protect against DDoS. What security controls should be implemented?

A. Application-level rate limiting only
B. Cloud Armor with rate limiting policies, IP deny lists, and DDoS protection in front of Cloud Run via load balancer
C. Network firewall rules
D. No protection - rely on Cloud Run scaling

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Armor provides rate limiting (per-client throttling), IP/geographic deny lists, and DDoS protection. For Cloud Run, this requires external HTTPS load balancer which Cloud Armor attaches to. This provides perimeter protection before traffic reaches the application. Application-level limiting (A) consumes resources before rejecting. Firewall rules (C) are layer 4. Relying on scaling (D) incurs cost from attack traffic.

**Key Concept:** [Cloud Armor for Cloud Run](https://cloud.google.com/run/docs/securing/cloud-armor)
</details>

---

## Domain 4: Ensuring Compliance (Questions 28-33)

### Question 28
**Scenario:** A healthcare organization needs to ensure their Google Cloud environment complies with HIPAA requirements. They need to sign a BAA with Google and ensure their configurations meet HIPAA controls. How should they approach this?

A. Assume Google Cloud is automatically HIPAA compliant
B. Sign Google Cloud BAA, use only HIPAA-eligible services, configure according to compliance guides, and enable Assured Workloads
C. Use any Google Cloud services
D. HIPAA only applies to on-premises

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** HIPAA compliance requires: signing Google's BAA, using only covered services, proper configuration (encryption, access controls, audit logging), and organizational controls. Assured Workloads can enforce compliant service/region restrictions. Simply using Google Cloud (A, C) isn't sufficient - configuration matters. HIPAA applies regardless of location (D).

**Key Concept:** [HIPAA on Google Cloud](https://cloud.google.com/security/compliance/hipaa)
</details>

### Question 29
**Scenario:** A company needs to restrict their Google Cloud resources to specific geographic regions for data sovereignty requirements. They cannot have any data processed or stored outside the EU. How should this be enforced?

A. Document regional requirements
B. Organization Policy constraints for resource locations combined with Assured Workloads for EU region compliance
C. Manually check resource locations
D. Trust teams to deploy in correct regions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Organization Policy `constraints/gcp.resourceLocations` restricts where resources can be created. Assured Workloads provides additional data residency controls and compliance guardrails for specific regimes. Together they technically enforce geographic restrictions. Documentation (A) and trust (D) aren't enforceable. Manual checks (C) are reactive and error-prone.

**Key Concept:** [Resource Location Constraint](https://cloud.google.com/resource-manager/docs/organization-policy/defining-locations)
</details>

### Question 30
**Scenario:** An organization needs to prove to auditors that their Google Cloud environment has maintained continuous compliance with SOC 2 controls over the past year. What evidence should they provide?

A. Current configuration screenshots
B. Google's SOC 2 report combined with organization's SCC compliance reports and audit logs showing continuous compliant state
C. Verbal assurance
D. Point-in-time assessment only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Google's SOC 2 Type II report covers Google's infrastructure controls. Organization's Security Command Center compliance reports show resource compliance against standards over time. Audit logs demonstrate continuous monitoring and control operation. Screenshots (A) and point-in-time (D) don't show continuity. Verbal assurance (C) isn't auditable evidence.

**Key Concept:** [Google Cloud Compliance](https://cloud.google.com/security/compliance)
</details>

### Question 31
**Scenario:** A company needs to implement data retention requirements - some data must be retained for 7 years for compliance, while other data must be deleted after 90 days for privacy. How should this be managed in Cloud Storage?

A. Manual deletion tracking
B. Object Lifecycle Management rules for automatic deletion, and Object Lock/Retention policies for mandatory retention periods
C. Never delete anything
D. Application-level deletion

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Lifecycle Management automatically deletes objects after specified age (90 days). Retention policies with Bucket Lock prevent deletion until retention period expires (7 years), meeting legal hold requirements. These are enforced by the platform. Manual tracking (A) is error-prone. Never deleting (C) violates privacy requirements. Application deletion (D) can be bypassed.

**Key Concept:** [Object Lifecycle Management](https://cloud.google.com/storage/docs/lifecycle)
</details>

### Question 32
**Scenario:** An organization needs to ensure that sensitive data classifications (PII, financial, health) are consistently applied across their data assets and that security controls match the classification. What approach should they use?

A. Informal data classification
B. Data Catalog with policy tags for classification, BigQuery column-level security enforcing access based on tags
C. Same security for all data
D. Spreadsheet tracking data classifications

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Data Catalog policy tags provide systematic classification. Policy tags can be applied to BigQuery columns, and column-level security enforces that only authorized roles can access tagged data. This connects classification to enforcement. Informal (A) and spreadsheet (D) classifications aren't enforced by technology. Same security for all (C) is either over-protective or under-protective.

**Key Concept:** [BigQuery Column-Level Security](https://cloud.google.com/bigquery/docs/column-level-security)
</details>

### Question 33
**Scenario:** A company receives a legal hold request requiring them to preserve all emails, documents, and cloud data related to a specific project. The data must not be modified or deleted until the hold is lifted. How should this be implemented?

A. Ask employees not to delete data
B. Google Vault for Gmail/Drive retention, Cloud Storage retention policy lock, and BigQuery table snapshots with restricted access
C. Back up data and allow normal operations
D. Disable all delete permissions broadly

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Google Vault provides legal hold for Gmail and Drive. Cloud Storage retention policy lock prevents object deletion until the policy period. BigQuery snapshots preserve point-in-time data. These technical controls ensure preservation regardless of user actions. Asking employees (A) is unenforceable. Backups (C) don't prevent modification of originals. Broad disable (D) affects all operations, not specific data.

**Key Concept:** [Google Vault](https://support.google.com/vault/answer/2462365)
</details>

---

## Domain 5: Securing Data (Questions 34-40)

### Question 34
**Scenario:** A company needs to encrypt data at rest with keys they control, rotate keys automatically, and ensure Google cannot access the plaintext encryption keys. The data is in Cloud Storage and BigQuery. What encryption approach meets these requirements?

A. Default Google-managed encryption
B. Customer-managed encryption keys (CMEK) in Cloud KMS with automatic rotation enabled
C. Client-side encryption only
D. Cloud HSM only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CMEK uses keys in Cloud KMS under customer control. Cloud KMS automatic rotation creates new key versions while maintaining access to data encrypted with previous versions. The envelope encryption model means Google never has access to the plaintext KEK. Default encryption (A) uses Google-managed keys. Client-side (C) works but adds application complexity. Cloud HSM (D) is for stricter requirements and is part of KMS.

**Key Concept:** [CMEK](https://cloud.google.com/kms/docs/cmek)
</details>

### Question 35
**Scenario:** A financial services company requires that their encryption keys are protected by FIPS 140-2 Level 3 certified hardware and that keys never exist in plaintext outside the HSM. Which Cloud KMS option meets this requirement?

A. Cloud KMS software keys
B. Cloud HSM (Hardware Security Module) keys in Cloud KMS
C. Cloud EKM (External Key Manager)
D. Default Google encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud HSM provides FIPS 140-2 Level 3 certified hardware protection. Key material never leaves the HSM in plaintext - all cryptographic operations happen within the HSM boundary. This meets strict financial services requirements. Software keys (A) aren't hardware-protected. EKM (C) uses external HSMs. Default encryption (D) doesn't provide customer control or HSM protection.

**Key Concept:** [Cloud HSM](https://cloud.google.com/kms/docs/hsm)
</details>

### Question 36
**Scenario:** A company has strict data sovereignty requirements mandating that encryption keys remain on-premises in their own data center. They still want to use Google Cloud services for compute and storage. What Cloud KMS option supports this?

A. Cloud KMS regional keys
B. Cloud EKM (External Key Manager) connecting to their on-premises key management system
C. Import keys into Cloud KMS
D. Cloud HSM in Google's data centers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud EKM allows Google Cloud services to use encryption keys stored and managed in a supported external key management system. Keys never leave the customer's on-premises environment. Google sends cryptographic requests to the external KMS. Imported keys (C) copy key material to Google. Cloud KMS/HSM keys (A, D) are stored in Google's infrastructure.

**Key Concept:** [Cloud EKM](https://cloud.google.com/kms/docs/ekm)
</details>

### Question 37
**Scenario:** A healthcare company needs to share de-identified patient data with researchers. The original data contains names, addresses, phone numbers, and medical record numbers. How should the data be de-identified while preserving research utility?

A. Delete all records with PII
B. Cloud DLP with de-identification transforms: masking for names/addresses, format-preserving encryption for medical record numbers, date shifting for dates
C. Manual redaction
D. Share original data with NDAs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud DLP identifies sensitive data and applies appropriate de-identification transforms. Masking replaces values with fixed characters. Format-preserving encryption maintains data format for referential integrity. Date shifting preserves temporal relationships while obscuring exact dates. This balances privacy with research utility. Deleting records (A) loses data. Manual redaction (C) doesn't scale. NDAs (D) don't technically protect data.

**Key Concept:** [Cloud DLP De-identification](https://cloud.google.com/dlp/docs/deidentify-sensitive-data)
</details>

### Question 38
**Scenario:** A company wants to ensure that sensitive data (credit card numbers, SSNs) is not accidentally stored in Cloud Storage or BigQuery. They want automatic detection and alerting when such data is discovered. What service provides this capability?

A. Manual data inspection
B. Cloud DLP inspection jobs with automated scanning of Cloud Storage and BigQuery, alerting via Pub/Sub/SCC
C. Keyword searching
D. Trust developers not to store sensitive data

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud DLP inspection jobs can automatically scan Cloud Storage objects and BigQuery tables for 150+ predefined detectors (credit cards, SSNs, etc.) or custom patterns. Findings can trigger Pub/Sub notifications or appear in Security Command Center. This provides continuous data discovery. Manual inspection (A) doesn't scale. Keyword search (C) misses variations. Trust (D) isn't verification.

**Key Concept:** [Cloud DLP Inspection](https://cloud.google.com/dlp/docs/inspecting-storage)
</details>

### Question 39
**Scenario:** A database application needs to encrypt specific columns containing sensitive data (SSN, credit card) so that database administrators cannot see the plaintext values. The application should transparently encrypt on write and decrypt on read. What approach provides this?

A. Database-level encryption (TDE)
B. Application-layer encryption using Cloud KMS tink library, storing ciphertext in database columns
C. Network encryption only
D. File-system encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application-layer encryption encrypts data before it reaches the database. Database admins see only ciphertext. Tink library with Cloud KMS integration provides easy-to-use envelope encryption. Only application service accounts with KMS access can decrypt. TDE (A) and filesystem encryption (D) protect at rest but DBAs can query plaintext. Network encryption (C) protects in transit only.

**Key Concept:** [Tink Cryptographic Library](https://developers.google.com/tink)
</details>

### Question 40
**Scenario:** A company uses BigQuery for analytics containing customer data. They need to ensure that analysts can run aggregate queries (counts, averages) without being able to see individual customer records. How should this be implemented?

A. Trust analysts not to query individual records
B. Authorized views returning only aggregated results, or BigQuery differential privacy for private aggregate queries
C. Remove detailed data from BigQuery
D. Encryption only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Authorized views can enforce that only aggregate queries are allowed, hiding underlying row-level data. BigQuery differential privacy provides mathematically guaranteed privacy for aggregate queries, preventing re-identification through query patterns. Trust (A) is unenforceable. Removing detailed data (C) loses analytical capability. Encryption (D) doesn't restrict query patterns.

**Key Concept:** [BigQuery Differential Privacy](https://cloud.google.com/bigquery/docs/differential-privacy)
</details>
