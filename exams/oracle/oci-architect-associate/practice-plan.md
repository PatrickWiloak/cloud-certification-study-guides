# OCI Architect Associate - Practice Plan

A comprehensive 8-10 week study plan to prepare for the Oracle Cloud Infrastructure Architect Associate certification (1Z0-1072-24).

## Prerequisites
- [ ] OCI Foundations certification completed (recommended)
- [ ] Active OCI Free Tier account + some paid credits for advanced features
- [ ] Basic understanding of cloud architecture

## Week 1-2: Foundation Review and Hybrid Architecture

### Week 1: Architecture Foundations
**Day 1-2: Review Core Concepts**
- [ ] Review OCI Foundations materials (compartments, IAM, regions, ADs)
- [ ] Study architecture design principles
- [ ] Understand well-architected framework pillars
- [ ] **Hands-on**: Review existing OCI console and services

**Day 3-4: Hybrid Cloud Architecture**
- [ ] Study FastConnect (1 Gbps, 10 Gbps options)
- [ ] Understand Site-to-Site VPN
- [ ] Learn Dynamic Routing Gateway (DRG)
- [ ] Study hybrid connectivity patterns
- [ ] **Hands-on**: Configure Site-to-Site VPN
- [ ] **Practice**: 20 hybrid connectivity questions

**Day 5-7: Hybrid Integration Patterns**
- [ ] Study Oracle Cloud VMware Solution
- [ ] Understand database migration strategies
- [ ] Learn DNS integration patterns
- [ ] Review multi-cloud scenarios
- [ ] **Hands-on**: Design hybrid architecture diagram
- [ ] **Practice**: 25 hybrid architecture scenarios

### Week 2: Compute and Storage Architecture
**Day 1-3: Compute Design**
- [ ] Study shape selection (VM.Standard, VM.DenseIO, BM, GPU)
- [ ] Understand dedicated VM hosts
- [ ] Learn instance pools and autoscaling
- [ ] Study placement groups
- [ ] **Hands-on**: Deploy instance pool with autoscaling
- [ ] **Hands-on**: Test different compute shapes
- [ ] **Practice**: 25 compute architecture questions

**Day 4-7: Storage Architecture**
- [ ] Study block volume performance tiers
- [ ] Understand Object Storage tier selection
- [ ] Learn File Storage use cases
- [ ] Study backup strategies and retention
- [ ] **Hands-on**: Configure block volume with high performance tier
- [ ] **Hands-on**: Set up Object Storage lifecycle policies
- [ ] **Hands-on**: Implement cross-region replication
- [ ] **Practice**: 30 storage architecture scenarios

## Week 3-4: Networking Architecture

### Week 3: VCN Design
**Day 1-3: VCN Architecture**
- [ ] Study CIDR planning and subnet design
- [ ] Understand public vs private subnets
- [ ] Learn hub-and-spoke topology with DRG
- [ ] Study VCN peering (local and remote)
- [ ] **Hands-on**: Design and deploy hub-and-spoke network
- [ ] **Hands-on**: Configure VCN peering
- [ ] **Practice**: 25 VCN design questions

**Day 4-7: Gateways and Security**
- [ ] Study all gateway types (IG, NAT, SG, DRG)
- [ ] Understand route table design
- [ ] Learn security lists vs NSGs (when to use each)
- [ ] Study defense in depth approach
- [ ] **Hands-on**: Configure all gateway types
- [ ] **Hands-on**: Implement microsegmentation with NSGs
- [ ] **Practice**: 30 networking security scenarios

### Week 4: Load Balancing and Advanced Networking
**Day 1-3: Load Balancing**
- [ ] Study load balancer types (flexible, network)
- [ ] Understand backend set design
- [ ] Learn SSL/TLS termination
- [ ] Study health check configuration
- [ ] **Hands-on**: Deploy load balancer with SSL termination
- [ ] **Hands-on**: Configure backend sets and health checks
- [ ] **Practice**: 20 load balancing questions

**Day 4-7: Advanced Networking**
- [ ] Study Traffic Management steering policies
- [ ] Understand DNS architecture (public/private zones)
- [ ] Learn private endpoints for PaaS
- [ ] Review all networking concepts
- [ ] **Hands-on**: Configure DNS failover
- [ ] **Hands-on**: Set up private endpoint
- [ ] **Practice**: 25 advanced networking scenarios

## Week 5-6: Database and Container Architecture

### Week 5: Database Solutions
**Day 1-3: Autonomous Database**
- [ ] Study serverless vs dedicated infrastructure
- [ ] Understand ATP vs ADW use cases
- [ ] Learn auto-scaling and performance tuning
- [ ] Study Autonomous Data Guard for DR
- [ ] **Hands-on**: Deploy Autonomous Database (serverless and dedicated)
- [ ] **Hands-on**: Configure Autonomous Data Guard
- [ ] **Practice**: 25 Autonomous Database questions

**Day 4-7: DB Systems and Data Management**
- [ ] Study VM DB vs Bare Metal vs Exadata
- [ ] Understand RAC for high availability
- [ ] Learn Data Guard for disaster recovery
- [ ] Study backup and patching strategies
- [ ] **Hands-on**: Deploy DB System with RAC
- [ ] **Hands-on**: Configure Data Guard
- [ ] **Practice**: 30 database architecture scenarios

### Week 6: Container Architecture
**Day 1-4: OKE Architecture**
- [ ] Study OKE cluster design (managed, virtual, self-managed nodes)
- [ ] Understand OKE networking modes (VCN-native, flannel)
- [ ] Learn persistent storage with OCI Block Volumes
- [ ] Study OKE autoscaling (HPA, cluster autoscaler)
- [ ] **Hands-on**: Deploy OKE cluster with VCN-native networking
- [ ] **Hands-on**: Configure autoscaling for OKE
- [ ] **Practice**: 25 OKE architecture questions

**Day 5-7: Container Registry and Serverless**
- [ ] Study OCIR for image management
- [ ] Understand Functions architecture
- [ ] Learn API Gateway integration
- [ ] Study Container Instances use cases
- [ ] **Hands-on**: Push images to OCIR
- [ ] **Hands-on**: Deploy serverless function with API Gateway
- [ ] **Practice**: 20 container/serverless scenarios

## Week 7-8: Security, HA, and Observability

### Week 7: Security and Compliance
**Day 1-3: IAM Architecture**
- [ ] Study compartment strategy design
- [ ] Understand IAM policy design (least privilege)
- [ ] Learn federation with external IdPs
- [ ] Study dynamic groups for resource policies
- [ ] **Hands-on**: Design and implement compartment hierarchy
- [ ] **Hands-on**: Configure federation with SAML
- [ ] **Practice**: 25 IAM architecture questions

**Day 4-7: Data Security and Compliance**
- [ ] Study Vault for key and secrets management
- [ ] Understand encryption (at rest and in transit)
- [ ] Learn Cloud Guard configuration
- [ ] Study Security Zones and compliance
- [ ] **Hands-on**: Implement encryption with customer-managed keys
- [ ] **Hands-on**: Configure Cloud Guard and Security Zones
- [ ] **Practice**: 30 security architecture scenarios

### Week 8: High Availability and Observability
**Day 1-3: HA and DR Design**
- [ ] Study multi-AD deployment strategies
- [ ] Understand fault domain distribution
- [ ] Learn DR strategies (pilot light, warm standby, active-active)
- [ ] Study RTO and RPO requirements
- [ ] **Hands-on**: Design multi-AD architecture
- [ ] **Hands-on**: Implement cross-region DR solution
- [ ] **Practice**: 25 HA/DR scenarios

**Day 4-7: Monitoring and Observability**
- [ ] Study monitoring architecture (metrics, alarms)
- [ ] Understand logging strategy
- [ ] Learn Log Analytics for centralized logging
- [ ] Study APM for application performance
- [ ] **Hands-on**: Configure comprehensive monitoring solution
- [ ] **Hands-on**: Set up Log Analytics
- [ ] **Practice**: 20 observability questions

## Week 9-10: Practice and Final Review

### Week 9: Architecture Scenarios and Case Studies
**Day 1-3: Practice Scenarios**
- [ ] Complete 10 multi-tier application designs
- [ ] Practice 10 hybrid cloud scenarios
- [ ] Design 5 high availability solutions
- [ ] Practice 5 disaster recovery plans
- [ ] **Review**: Document design decisions and trade-offs

**Day 4-7: Practice Exams**
- [ ] Take practice exam #1 (timed, 60 questions)
- [ ] Review all incorrect answers thoroughly
- [ ] Take practice exam #2 (timed, 60 questions)
- [ ] Identify weak areas
- [ ] **Focus**: Deep dive on weak topics

### Week 10: Final Review and Preparation
**Day 1-3: Comprehensive Review**
- [ ] Review all study notes from weeks 1-9
- [ ] Revisit OCI Architecture Center reference architectures
- [ ] Review all hands-on labs documentation
- [ ] **Practice**: 50 mixed topic questions

**Day 4-5: Final Practice**
- [ ] Take practice exam #3 (timed, 60 questions)
- [ ] Achieve 80%+ score
- [ ] Review any remaining weak areas
- [ ] **Hands-on**: Build end-to-end solution (VCN, compute, database, HA, monitoring)

**Day 6-7: Exam Preparation**
- [ ] Light review of key concepts
- [ ] Review exam tips and strategies
- [ ] Prepare exam environment (if online)
- [ ] Rest and relax
- [ ] **Exam Day**: Stay confident!

## Critical Hands-on Labs

Must complete before exam:

- [ ] Design and implement hub-and-spoke network with DRG
- [ ] Deploy highly available web app across multiple ADs
- [ ] Configure FastConnect or Site-to-Site VPN (simulator OK)
- [ ] Deploy Autonomous Database with Autonomous Data Guard
- [ ] Implement comprehensive IAM with compartments and policies
- [ ] Deploy OKE cluster with autoscaling
- [ ] Configure monitoring, logging, and alarms
- [ ] Implement DR solution with cross-region replication
- [ ] Design and deploy multi-tier application architecture
- [ ] Configure Cloud Guard and Security Zones

## Practice Question Breakdown

Aim for 500+ practice questions total:

- [ ] 100 questions: Hybrid cloud and networking (20%)
- [ ] 125 questions: Compute and storage design (25%)
- [ ] 150 questions: Networking architecture (30%)
- [ ] 100 questions: Database solutions (20%)
- [ ] 75 questions: Security and compliance (15%)
- [ ] 75 questions: HA and DR (15%)
- [ ] 50 questions: Observability (10%)

## Key Architecture Patterns to Master

**Multi-Tier Application**:
- Web tier: Load balancer + instances in public subnet
- App tier: Instances in private subnet
- Database tier: Autonomous DB in private subnet
- Bastion for admin access

**Hub-and-Spoke Network**:
- Hub VCN: Central routing with DRG
- Spoke VCNs: Application workloads
- On-premises: FastConnect or VPN

**High Availability Pattern**:
- Deploy across 2+ ADs
- Distribute across fault domains
- Use regional services
- Implement health checks

**Disaster Recovery Pattern**:
- Primary region: Active workloads
- Secondary region: Standby resources
- Cross-region replication
- DNS failover

## Study Resources Checklist

- [ ] OCI Architect Associate Learning Path (mylearn.oracle.com) - completed
- [ ] OCI Architecture Center - all reference architectures reviewed
- [ ] OCI Documentation - key services reviewed
- [ ] Practice exams - minimum 3 full exams completed
- [ ] Hands-on labs - all critical labs completed
- [ ] Study notes - organized and reviewed

## Exam Strategy

**Time Management** (105 min / 60 questions):
- First pass: 60 minutes (1 min/question)
- Review flagged: 30 minutes
- Final review: 15 minutes

**Question Approach**:
- Read scenario carefully, identify requirements
- Consider cost, performance, security, HA
- Eliminate obviously wrong answers
- Choose BEST option (not just correct)
- Flag and move on if unsure

**Common Question Keywords**:
- "Most cost-effective" → Consider pricing models
- "Highest security" → Look for KYOK, Security Zones
- "Best performance" → Consider dedicated resources
- "High availability" → Multi-AD, fault domains
- "Disaster recovery" → Cross-region, backups

## Week Before Exam

- [ ] Take final practice exam (90%+ target score)
- [ ] Review all architecture patterns
- [ ] Quick review of all services and use cases
- [ ] Prepare exam day logistics
- [ ] Get adequate sleep (7-8 hours)

---

**Success Formula**: 40% Hands-on Labs + 40% Practice Questions + 20% Theory

**Remember**: Architecture is about trade-offs. Always consider cost, performance, security, and availability in your designs!

Good luck! 🏗️
