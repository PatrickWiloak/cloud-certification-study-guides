# OCI Foundations - One-Page Cram Sheet

## Architecture & Infrastructure

**Regions**
- 40+ worldwide | Multi-AD (3 ADs) or Single-AD (1 AD + 3 FDs) | Format: us-phoenix-1

**Availability Domains (ADs)**
- 1-3 per region | Physically separated data centers | AD# are customer-specific | For HA

**Fault Domains (FDs)**
- 3 per AD | Logical hardware groupings | Protection against hardware failure

**Tenancy & Compartments**
- Tenancy = root compartment | Max depth: 6 levels (root + 5) | Resources in 1 compartment, can move

## Core Services Matrix

| Service | Type | Use Case | Key Feature |
|---------|------|----------|-------------|
| **VM/BM** | Compute | Workloads | Flexible shapes |
| **OKE** | Container | Kubernetes | Managed K8s |
| **Functions** | Serverless | Event-driven | Pay-per-execution |
| **Block Volume** | Storage | Boot/data disks | Up to 32 TB, iSCSI |
| **Object Storage** | Storage | Unstructured data | 3 tiers, up to 10 TB/object |
| **File Storage** | Storage | Shared files | NFS v3 |
| **Archive** | Storage | Cold data | ~1 hr retrieval |
| **VCN** | Network | Private network | /16 to /30 CIDR |
| **Autonomous DB** | Database | Managed DB | ATP (OLTP), ADW (analytics) |

## Networking Quick Reference

**Gateways:**
- **IG** = Internet (bidirectional)
- **NAT** = Outbound only (private subnets)
- **SGW** = Oracle services (private, no internet)
- **DRG** = On-premises (VPN/FastConnect)

**Security:**
- **Security Lists** = Subnet-level, allow/deny, stateful
- **NSGs** = Resource-level, allow only, stateful (PREFERRED)

**Load Balancer:**
- Public or Private | Layer 4 (TCP) + Layer 7 (HTTP/HTTPS) | SSL termination | Health checks

## IAM Policy Syntax

```
Allow <subject> to <verb> <resource-type> in <location>
```

**Verbs (Permissions):** inspect < read < use < manage

**Examples:**
```
Allow group Developers to manage instance-family in compartment Dev
Allow group DBAdmins to use database-family in tenancy
Allow dynamic-group AppServers to read secret-family in compartment Prod
```

**Subjects:**
- Users/Groups = people
- Dynamic Groups = resources (instances, functions)

## Storage Decision Tree

```
Need block device for instance? → Block Volume
Need shared file system (NFS)? → File Storage
Unstructured data, frequent access? → Object Storage Standard
Unstructured data, occasional access? → Object Storage Infrequent Access
Long-term archive (10+ years)? → Archive Storage
```

**Object Storage Tiers:**
- Standard = hot, instant, $$
- Infrequent Access = cool, instant, $
- Archive = cold, ~1hr retrieval, ¢

## Database Quick Reference

**Autonomous Database:**
- **ATP** = Transaction processing (OLTP)
- **ADW** = Data warehouse (analytics/OLAP)
- **JSON** = JSON documents
- **Serverless** = shared infra, auto-scale
- **Dedicated** = isolated infra, more control

**DB Systems:**
- VM DB = virtual machines
- Bare Metal DB = physical servers
- Exadata DB = high-performance
- **RAC** = High Availability (clustering)
- **Data Guard** = Disaster Recovery (standby)

## Security Services

| Service | Purpose |
|---------|---------|
| **Vault** | Encryption keys + secrets storage |
| **Cloud Guard** | Automated threat detection |
| **Security Zones** | Enforce security policies (encryption required) |
| **WAF** | Web application firewall |

**Encryption:**
- All data encrypted at rest (default, Oracle-managed)
- In transit via TLS/SSL
- Customer-managed keys via Vault
- BYOK = Bring Your Own Key

## Observability

**Monitoring** = Metrics + Alarms (FREE)
**Logging** = Audit + Service + Custom logs
**Notifications** = Topics + Subscriptions (email, SMS, webhook)
**Events** = State changes → trigger actions
**Tags** = Cost tracking + organization

## Pricing Models

| Model | Commitment | Discount | Flexibility |
|-------|------------|----------|-------------|
| **PAYG** | None | 0% | Full |
| **Monthly Flex** | Monthly | Moderate | High |
| **Annual Flex** | 1 year | 33%+ | High |
| **BYOL** | License | 50%+ | Specific |

**Universal Credits** = Use for any OCI service

## Always Free Tier

**Compute:** 2 AMD VMs (1/8 OCPU, 1GB RAM) + 4 ARM cores (24GB RAM)
**Storage:** 200GB Block + 10GB Object + 10GB Archive
**Database:** 2 Autonomous DBs (20GB each)
**Networking:** Load Balancer (10 Mbps), VCN, Monitoring, Logging

**Trial:** $300 credits, 30 days

## High Availability Patterns

**Single-AD Region:**
```
AD-1
├── FD-1: Instance-1
├── FD-2: Instance-2
└── FD-3: Instance-3
```

**Multi-AD Region:**
```
AD-1: Web-1, App-1, DB-Primary
AD-2: Web-2, App-2, DB-Standby
AD-3: Web-3, App-3, DB-Backup
```

## Exam Quick Facts

**Numbers to Remember:**
- 60 questions, 90 minutes, 68% passing
- Max compartment depth = 6 levels
- Multi-AD region = 3 ADs
- Single-AD region = 3 FDs
- Block Volume max = 32 TB
- Object max size = 10 TB
- VCN CIDR = /16 to /30
- Always Free = 2 VMs, 4 ARM cores, 2 DBs

**Architecture:**
- ADs = physically separated (different buildings)
- FDs = logically separated (different racks)
- AD numbers randomized per customer
- Compartments = organization + access control + billing

**Keywords in Questions:**
- "High availability" → Multi-AD or Multi-FD
- "Most cost-effective" → Always Free, Object Storage, Serverless
- "Lowest latency" → Region closest to users
- "Secure" → Private subnets, NSGs, Vault
- "On-premises" → DRG + FastConnect/VPN
- "Unstructured data" → Object Storage
- "Shared file system" → File Storage
- "Long-term archive" → Archive Storage

## Common Scenarios & Answers

**Q: Design HA web app**
A: Multi-AD deployment + Load Balancer + DB replication

**Q: Secure database credentials**
A: Store in Vault, use IAM policy to control access

**Q: Private access to Object Storage**
A: Service Gateway (no internet required)

**Q: Organize multi-project resources**
A: Compartments by project, then by environment (Dev/Test/Prod)

**Q: Outbound internet for private subnet**
A: NAT Gateway

**Q: Allow developers to manage VMs in Dev**
A: `Allow group Developers to manage instance-family in compartment Dev`

**Q: Long-term log retention**
A: Archive Storage

**Q: Frequently accessed user uploads**
A: Object Storage Standard tier

**Q: Detect security threats automatically**
A: Cloud Guard

**Q: Best DB for analytics workload**
A: Autonomous Data Warehouse (ADW)

## Critical "Don't Forget" Items

✅ AD numbers are customer-specific (not the same for all)
✅ Compartments inherit policies from parents
✅ Security Lists = subnet, NSGs = resource
✅ NAT = outbound only (no inbound)
✅ Service Gateway = private (no internet needed)
✅ Dynamic Groups = resources, not users
✅ Archive Storage = ~1 hour retrieval time
✅ Autonomous DB has auto-scaling
✅ BYOL = Bring Your Own (Oracle) License
✅ Home region cannot be changed

---

**Last-Minute Tips:**
1. Read questions carefully - look for keywords
2. Eliminate obviously wrong answers first
3. Consider cost, security, and HA in scenarios
4. Watch for "MOST correct" vs just "correct"
5. Time: 1.5 minutes per question average
6. Flag uncertain questions, return later

**Good luck!** 🍀
