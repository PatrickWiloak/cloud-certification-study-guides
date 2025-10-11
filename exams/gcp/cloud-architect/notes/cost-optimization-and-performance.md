# Cost Optimization and Performance - GCP Professional Cloud Architect

## Overview

Comprehensive guide to cost optimization and performance tuning in Google Cloud Platform. This guide covers advanced strategies for resource optimization, pricing models, performance optimization, monitoring and observability, and cost-performance trade-offs essential for the Professional Cloud Architect certification. Understand how to design architectures that balance cost efficiency with performance requirements while maintaining reliability and scalability.

## Key Topics

1. **Cost Optimization Strategies** - Committed use discounts, sustained use discounts, preemptible/spot VMs, rightsizing, resource hierarchy
2. **Billing Architecture** - Billing accounts, projects, labels, budgets, alerts, cost allocation, BigQuery export
3. **Performance Optimization** - Compute optimization (machine types, GPUs, TPUs), storage optimization, network optimization
4. **Monitoring and Observability** - Cloud Monitoring architecture, SLIs/SLOs/SLAs, alerting policies, dashboards
5. **Database Performance** - Cloud SQL, Spanner, Bigtable, BigQuery optimization strategies
6. **Caching Strategies** - Cloud CDN, Memorystore, application-level caching patterns
7. **Cost/Performance Trade-offs** - Architecture decision frameworks balancing cost and performance
8. **Troubleshooting** - Common performance issues and resolution patterns
9. **Exam Focus** - Professional-level cost questions, optimization patterns, ROI calculations

## Cost Optimization Strategies

### Compute Cost Optimization

#### Committed Use Discounts (CUDs)
**Overview**: Purchase committed compute resources for 1-year or 3-year terms for significant discounts
- **Resource-based CUDs**: Commit to specific machine types in specific regions (up to 57% discount)
- **Spend-based CUDs**: Commit to spending amount, flexible across machine types and regions (up to 55% discount)
- **Automatic application**: Applied automatically to eligible usage across projects in billing account
- **Flexibility**: Can be shared across projects, regions (for spend-based)

**Discount Levels**:
| Commitment Type | 1-Year Discount | 3-Year Discount |
|----------------|-----------------|-----------------|
| General Purpose (N1, N2, N2D) | 37% | 55% |
| Compute Optimized (C2, C2D) | 37% | 55% |
| Memory Optimized (M1, M2) | 37% | 55% |
| Spend-based | 25% | 52% |

**Best Practices**:
- Start with spend-based CUDs for flexibility
- Analyze 30-60 day usage patterns before committing
- Reserve 60-70% of baseline usage, handle peaks with on-demand
- Use recommendations engine to identify CUD opportunities
- Consider regional vs global flexibility trade-offs

#### Sustained Use Discounts (SUDs)
**Automatic discounts**: Applied for consistent usage of Compute Engine resources
- Incremental discounts up to 30% for VMs running >25% of month
- Applied automatically, no commitment required
- Calculated per-second, applied monthly
- Combine with committed use discounts

**Discount Schedule**:
| Usage Percentage | Discount Applied |
|-----------------|------------------|
| 0-25% | 0% |
| 25-50% | 20% (incremental) |
| 50-75% | 40% (incremental) |
| 75-100% | 60% (incremental) |
| Effective maximum | ~30% |

**Important Notes**:
- Only applies to Compute Engine (not GKE, Dataproc on-demand)
- Combines with committed use discounts
- Applied after CUDs in discount hierarchy
- Calculated across all instances in same project/region

#### Preemptible and Spot VMs
**Preemptible VMs**: Short-lived VM instances at up to 80% discount
- Maximum runtime: 24 hours
- Can be preempted with 30-second warning
- No SLA, no live migration
- Fixed pricing (80% discount)

**Spot VMs**: Newer model replacing preemptible VMs
- No maximum runtime (until preempted)
- 60-91% discount vs on-demand
- Dynamic pricing based on supply/demand
- 30-second preemption notice
- Same machine types as regular VMs

**Comparison Table**:
| Feature | Preemptible | Spot | On-Demand |
|---------|------------|------|-----------|
| Discount | 80% | 60-91% | 0% |
| Max Runtime | 24 hours | Until preempted | Unlimited |
| Pricing | Fixed | Dynamic | Fixed |
| Preemption Notice | 30 seconds | 30 seconds | N/A |
| Live Migration | No | No | Yes |
| Cost (n1-standard-4) | $24.82/mo | $19.86-39.72/mo | $121.68/mo |

**Use Cases**:
- Batch processing jobs
- Fault-tolerant applications
- CI/CD pipelines
- Data processing (Dataflow, Dataproc)
- Rendering and transcoding
- Machine learning training

**Best Practices**:
- Implement checkpointing for long-running jobs
- Use managed instance groups with auto-healing
- Design for graceful shutdown handling
- Monitor preemption rates via Cloud Monitoring
- Combine with regular VMs for critical components
- Use shutdown scripts to handle preemption

#### Right-sizing and Custom Machine Types
**Right-sizing Process**:
1. Analyze VM utilization metrics (CPU, memory, disk)
2. Review recommendations from Google Cloud Console
3. Implement changes during maintenance windows
4. Monitor performance after changes
5. Iterate based on results

**Custom Machine Types**:
- Tailor CPU and memory to exact needs
- 1 vCPU to 96 vCPUs (N1), up to 128 vCPUs (N2)
- 0.9 GB memory per vCPU minimum
- 6.5 GB memory per vCPU maximum (extended memory: 8 GB/vCPU)
- Save 10-40% vs standard machine types

**Example Cost Comparison**:
```
Workload: 8 vCPU, 16 GB RAM needed
- n1-standard-8 (8 vCPU, 30 GB RAM): $190.68/month (wasted 14 GB RAM)
- n1-custom-8-16384 (8 vCPU, 16 GB RAM): $156.24/month (exact fit)
- Savings: $34.44/month (18% reduction)
```

**Machine Type Selection Guide**:
| Workload Type | Recommended Machine Family |
|--------------|---------------------------|
| General purpose web apps | N1, N2, N2D, E2 |
| High-performance computing | C2, C2D (compute optimized) |
| In-memory databases | M1, M2, M3 (memory optimized) |
| Scale-out workloads | E2 (cost-optimized) |
| ARM workloads | T2A (Tau T2A) |

#### Resource Scheduling
**Automated start/stop for non-production**:
- Use Cloud Scheduler + Cloud Functions to stop/start instances
- Instance schedules for predictable patterns
- 40-60% cost reduction for dev/test environments

**Implementation Options**:
1. **Instance Schedules** (native feature):
   - Define schedule in Cloud Console
   - Automatic start/stop based on timezone
   - Applied via labels or organizational policy

2. **Cloud Scheduler + Functions**:
   - More flexibility for complex schedules
   - Can handle multiple resources
   - Integrate with approval workflows

**Example Schedule**:
```yaml
# Dev environment: Off nights and weekends
Monday-Friday: 7 AM - 7 PM (12 hours)
Saturday-Sunday: OFF
Monthly hours: 260 hours (vs 730 always-on)
Cost reduction: 64%
```

#### Auto-scaling Strategies
**Managed Instance Groups (MIG)**:
- Scale based on CPU utilization, HTTP load balancing, Pub/Sub queue depth
- Minimum and maximum instance counts
- Cooldown periods to prevent flapping
- Predictive autoscaling for traffic patterns

**GKE Cluster Autoscaling**:
- Node pool autoscaling based on pod requests
- Cluster autoscaler adds/removes nodes
- Vertical Pod Autoscaler (VPA) for right-sizing
- Horizontal Pod Autoscaler (HPA) for replica count

**Best Practices**:
- Set appropriate scaling metrics (don't rely solely on CPU)
- Configure minimum instances for availability
- Use multiple metrics for scaling decisions
- Implement warm-up periods for instance readiness
- Monitor scaling events and adjust thresholds

### Storage Cost Optimization

#### Cloud Storage Classes and Pricing
**Storage Class Selection Guide**:
| Storage Class | Use Case | Retrieval Cost | Min Storage | Monthly Cost/GB |
|--------------|----------|----------------|-------------|-----------------|
| Standard | Frequently accessed | None | None | $0.020-0.023 |
| Nearline | <1/month access | $0.01/GB | 30 days | $0.010 |
| Coldline | <1/quarter access | $0.02/GB | 90 days | $0.004 |
| Archive | <1/year access | $0.05/GB | 365 days | $0.0012 |

**Cost Optimization Strategy**:
```
Example: 100 TB storage with varying access patterns
- Hot data (10%): Standard storage = 10 TB × $20 = $200/month
- Warm data (30%): Nearline storage = 30 TB × $10 = $300/month
- Cold data (40%): Coldline storage = 40 TB × $4 = $160/month
- Archive (20%): Archive storage = 20 TB × $1.20 = $24/month
Total: $684/month vs $2,000/month (all Standard) = 66% savings
```

#### Lifecycle Management Policies
**Automated transitions and deletions**:
- Move objects to cheaper storage classes based on age
- Delete temporary files after retention period
- Delete old object versions
- Transition based on creation date or modification time

**Example Lifecycle Policy**:
```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30, "matchesPrefix": ["logs/"]}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90, "matchesPrefix": ["logs/"]}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 365, "matchesPrefix": ["logs/"]}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"numNewerVersions": 3, "isLive": false}
      }
    ]
  }
}
```

**Lifecycle Cost Impact**:
```
Scenario: 50 TB of logs per month
Without lifecycle: 50 TB × 12 months × $20 = $12,000/year (Standard)
With lifecycle:
  - Month 1: 50 TB × $20 = $1,000 (Standard)
  - Month 2-3: 100 TB × $10 = $1,000 (Nearline)
  - Month 4-12: 450 TB × $4 = $1,800 (Coldline)
  - Total: $3,800/year
Savings: $8,200/year (68% reduction)
```

#### Regional vs Multi-Regional Storage
**Location Selection Trade-offs**:
| Location Type | Use Case | Cost/GB | Availability SLA |
|--------------|----------|---------|------------------|
| Multi-region | Global access, HA | $0.026 | 99.95% |
| Dual-region | Regional redundancy | $0.020 | 99.95% |
| Regional | Single region | $0.020 | 99.90% |

**Best Practices**:
- Use multi-region for globally accessed content (websites, media)
- Use regional for compute-local data (reduces egress costs)
- Use dual-region for regional redundancy without multi-region cost
- Consider egress costs when choosing location

#### Persistent Disk Optimization
**Disk Type Selection**:
| Disk Type | Use Case | IOPS/GB | Throughput/GB | Cost/GB/month |
|-----------|----------|---------|---------------|---------------|
| Standard PD | Sequential, archival | 0.75 | 0.12 MB/s | $0.040 |
| Balanced PD | General purpose | 6 | 0.28 MB/s | $0.100 |
| SSD PD | Random I/O, databases | 30 | 0.48 MB/s | $0.170 |
| Extreme PD | Highest performance | Configurable | Configurable | $0.125 + IOPS |

**Performance-Cost Optimization**:
```
Workload: Database requiring 10,000 IOPS
Option 1: SSD PD (30 IOPS/GB) = 334 GB × $0.17 = $56.78/month
Option 2: Extreme PD = 10 GB × $0.125 + 10,000 IOPS × $0.065 = $651.25/month
Option 3: Balanced PD (6 IOPS/GB) = 1,667 GB × $0.10 = $166.70/month

Best choice: SSD PD for this workload (lowest cost, meets requirements)
```

**Snapshot Management**:
- Incremental snapshots reduce costs (only changes stored)
- Schedule automated snapshots for disaster recovery
- Delete old snapshots based on retention policy
- Use snapshot scheduling to automate lifecycle

**Snapshot Cost Example**:
```
500 GB disk, 5% daily change rate, 30-day retention
- Initial snapshot: 500 GB
- Daily incremental: 25 GB × 29 days = 725 GB
- Total storage: 1,225 GB × $0.026 = $31.85/month
```

#### Object Versioning Cost Management
**Versioning strategies**:
- Limit number of versions retained (noncurrent versions)
- Delete old versions via lifecycle policies
- Consider version storage costs (each version billed)
- Use object holds for compliance, not retention

**Version Cost Impact**:
```
Bucket with 10 TB, 5 versions per object
Without lifecycle: 10 TB × 5 versions × $20 = $1,000/month
With lifecycle (keep 3 versions): 10 TB × 3 versions × $20 = $600/month
Savings: $400/month (40% reduction)
```

### Network Cost Optimization

#### Understanding Network Pricing
**Egress Pricing Tiers** (per GB):
| Destination | Premium Tier | Standard Tier |
|------------|--------------|---------------|
| Same zone | Free | Free |
| Same region (different zone) | $0.01 | $0.01 |
| Inter-region (US) | $0.01 | $0.01 |
| Inter-region (intercontinental) | $0.05-0.08 | $0.05-0.08 |
| Internet (0-1 TB/month) | $0.12 | $0.085 |
| Internet (1-10 TB/month) | $0.11 | $0.085 |
| Internet (10+ TB/month) | $0.08 | $0.085 |
| China/Australia internet | $0.23 | $0.16 |

**Ingress**: Generally free (except for load balancer processing)

#### Network Tier Selection
**Premium Tier**:
- Google's global network for all routing
- Lower latency, better performance
- Single global IP address
- Higher cost for egress
- Best for: Global applications, real-time services, low-latency requirements

**Standard Tier**:
- Public internet routing outside Google network
- Regional IP addresses only
- 15-30% cost savings on egress
- Higher latency, variable performance
- Best for: Cost-sensitive workloads, regional applications, bulk data transfer

**Cost Comparison Example**:
```
Application: 10 TB/month egress to internet
Premium Tier: 10,000 GB × $0.11 = $1,100/month
Standard Tier: 10,000 GB × $0.085 = $850/month
Savings: $250/month (23% reduction)
```

#### Egress Reduction Strategies
**1. Regional Architecture**:
- Colocate resources in same region/zone
- Use regional load balancers for regional traffic
- Store data close to compute resources
- Minimize cross-region data replication

**2. Cloud CDN**:
- Cache static content at edge locations
- Reduce origin server load and egress
- Cache hit ratio typically 70-90%
- Pay only for cache egress (lower than origin egress)

**CDN Cost Savings Example**:
```
Website: 50 TB/month total traffic, 80% cacheable
Without CDN: 50 TB × $0.11 = $5,500/month (origin egress)
With CDN:
  - Cache hits (40 TB): 40 TB × $0.08 = $3,200 (CDN egress)
  - Cache misses (10 TB): 10 TB × $0.11 = $1,100 (origin egress)
  - Total: $4,300/month
Savings: $1,200/month (22% reduction)
```

**3. Cloud Interconnect**:
- Dedicated or Partner Interconnect for hybrid workloads
- Lower cost than internet egress for >1 TB/month
- 50-75% cost reduction for high-volume egress
- More predictable performance

**Interconnect Cost Analysis**:
```
Hybrid cloud: 20 TB/month to on-premises
Internet egress: 20 TB × $0.11 = $2,200/month
Dedicated Interconnect:
  - 10 Gbps port: $1,750/month flat fee
  - Egress: 20 TB × $0.035 = $700/month
  - Total: $2,450/month
  - Break-even: ~15 TB/month
Partner Interconnect:
  - 10 Gbps: $900/month + $700 egress = $1,600/month
Savings with Partner: $600/month (27% reduction)
```

#### Load Balancer Cost Optimization
**Load Balancer Pricing**:
| Type | Use Case | Hourly Rate | Per GB Processed |
|------|----------|-------------|------------------|
| Global HTTPS/SSL | Global web apps | $0.025 | $0.008-0.010 |
| Global TCP/SSL Proxy | Global TCP apps | $0.025 | $0.008-0.010 |
| Regional Network | Regional L4 | $0.025 | $0.008 |
| Regional Internal | Private services | $0.025 | $0.008 |
| Classic Network | Legacy | Free | $0.008 |

**Optimization Strategies**:
- Use internal load balancers for private traffic (no egress charges)
- Combine multiple services on single load balancer
- Use connection pooling to reduce connection overhead
- Consider Cloud Run/Cloud Functions for automatic scaling to zero

#### VPC Design for Cost Optimization
**Best Practices**:
- Use VPC peering for inter-project communication (free within region)
- Implement Cloud NAT for shared egress gateway
- Use Private Google Access to avoid egress charges for GCP services
- Design subnet CIDR ranges to minimize IP waste
- Use shared VPC for centralized network management

**Cloud NAT Cost Optimization**:
```
Scenario: 100 VMs needing internet access
Without Cloud NAT: 100 VMs × 100 GB egress × $0.12 = $1,200/month
With Cloud NAT:
  - NAT gateway: $0.044/hour × 730 hours = $32.12/month
  - NAT processing: 10 TB × $0.045 = $450/month
  - Egress: 10 TB × $0.12 = $1,200/month
  - Total: $1,682/month
Note: Cost benefit comes from centralized management and security, not egress savings
```

## Billing Architecture and Cost Management

### Billing Account Hierarchy
**Organization Structure**:
```
Organization (optional)
└── Billing Account(s)
    ├── Project A
    ├── Project B
    └── Folder
        ├── Project C
        └── Project D
```

**Billing Account Types**:
1. **Self-serve/Online**: Credit card or bank account, monthly invoicing
2. **Invoiced**: Monthly or quarterly invoicing, credit checks required, $2,500+ commitment
3. **Reseller**: Managed by Google Cloud reseller partner

**Key Concepts**:
- Billing accounts can link to multiple projects
- Projects can only have one billing account
- Billing IAM roles: Billing Account Administrator, Billing Account User, Billing Account Viewer
- Organization-level billing for consolidated reporting

### Resource Hierarchy for Cost Allocation
**Labels and Tags**:
- Apply labels to resources for cost tracking
- Standard labels: environment, team, cost-center, application
- Labels appear in billing exports for analysis
- Maximum 64 labels per resource

**Example Label Strategy**:
```yaml
environment: production | staging | development
team: backend | frontend | data | infrastructure
cost-center: engineering | marketing | operations
application: web-app | api | analytics
owner: team-email@company.com
```

**Project Organization**:
- Separate projects by environment (prod, staging, dev)
- Separate by team or application
- Use folders for grouping related projects
- Apply organization policies at folder/org level

**Cost Allocation Example**:
```
Organization: company.com
├── Production Folder
│   ├── prod-web-app (labels: env=prod, team=backend)
│   ├── prod-api (labels: env=prod, team=backend)
│   └── prod-analytics (labels: env=prod, team=data)
├── Staging Folder
│   └── staging-all (labels: env=staging)
└── Development Folder
    ├── dev-team-a (labels: env=dev, team=backend)
    └── dev-team-b (labels: env=dev, team=frontend)

Monthly costs by label:
- env=prod: $45,000
- env=staging: $8,000
- env=dev: $5,000
- team=backend: $32,000
- team=data: $18,000
```

### Budgets and Alerts
**Budget Configuration**:
- Set at billing account or project level
- Budget amount: Fixed or based on previous period
- Alert thresholds: 50%, 90%, 100%, custom
- Notification channels: Email, Pub/Sub, Monitoring
- Forecast-based alerts: Predict overspend before month end

**Budget Alert Examples**:
```
Production Budget: $50,000/month
Alert thresholds:
- 50% ($25,000): Email to team lead
- 75% ($37,500): Email to manager, Pub/Sub notification
- 90% ($45,000): Email to director, PagerDuty alert
- 100% ($50,000): Email to exec team, freeze non-critical deployments
- 110% forecast: Early warning of projected overspend
```

**Programmatic Budget Actions**:
- Pub/Sub triggers Cloud Functions for automated actions
- Disable billing on project (stops all resources)
- Send alerts to monitoring systems
- Create tickets in JIRA/ServiceNow
- Stop non-production instances
- Reduce instance sizes or scale down

### Cost Breakdown and Analysis
**Billing Reports**:
- View costs by project, service, SKU, location
- Filter by time range, tags, folders
- Compare periods (month-over-month, year-over-year)
- Export to CSV for offline analysis

**Cost Breakdown Dimensions**:
- **By Service**: Compute Engine, Cloud Storage, BigQuery, etc.
- **By Project**: Individual project costs
- **By SKU**: Specific resource types (n1-standard-4, SSD storage)
- **By Location**: us-central1, europe-west1, etc.
- **By Label**: Custom cost allocation tags
- **By Invoice**: Monthly or quarterly invoices

### BigQuery Export for Advanced Analysis
**Setup**:
1. Enable billing export to BigQuery dataset
2. Create dataset in specific project
3. Export includes: Standard usage costs, pricing, credits, adjustments
4. Daily export with previous day's data
5. Partitioned by date for query efficiency

**Schema Overview**:
```sql
-- Key fields in billing export
billing_account_id      -- Billing account
project.id             -- Project ID
project.name           -- Project name
service.description    -- Service name (Compute Engine, Cloud Storage)
sku.description       -- Specific SKU (N1 Predefined Instance Core)
usage_start_time      -- When usage occurred
usage_end_time        -- When usage ended
cost                  -- Cost in billing currency
currency              -- Billing currency (USD, EUR)
credits               -- Promotional credits, committed use discounts
labels                -- Resource labels as array
location.region       -- Region where resource used
```

**Example Analysis Queries**:

**Monthly cost by service**:
```sql
SELECT
  service.description AS service,
  SUM(cost) AS total_cost,
  SUM(cost) / SUM(SUM(cost)) OVER() * 100 AS percent_of_total
FROM `project.dataset.gcp_billing_export_v1_XXXXXX`
WHERE DATE(usage_start_time) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY service
ORDER BY total_cost DESC
LIMIT 10;
```

**Cost by label (team/environment)**:
```sql
SELECT
  label.value AS team,
  SUM(cost) AS total_cost
FROM `project.dataset.gcp_billing_export_v1_XXXXXX`,
  UNNEST(labels) AS label
WHERE label.key = 'team'
  AND DATE(usage_start_time) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY team
ORDER BY total_cost DESC;
```

**Daily cost trend with forecast**:
```sql
SELECT
  DATE(usage_start_time) AS usage_date,
  SUM(cost) AS daily_cost,
  AVG(SUM(cost)) OVER(ORDER BY DATE(usage_start_time) ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7day
FROM `project.dataset.gcp_billing_export_v1_XXXXXX`
WHERE DATE(usage_start_time) >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)
GROUP BY usage_date
ORDER BY usage_date;
```

**Committed use discount utilization**:
```sql
SELECT
  DATE_TRUNC(DATE(usage_start_time), MONTH) AS month,
  SUM(IF(credit.type = 'COMMITTED_USAGE_DISCOUNT', credit.amount, 0)) AS cud_savings,
  SUM(cost) AS total_cost,
  SUM(IF(credit.type = 'COMMITTED_USAGE_DISCOUNT', credit.amount, 0)) / SUM(cost) * 100 AS cud_savings_percent
FROM `project.dataset.gcp_billing_export_v1_XXXXXX`,
  UNNEST(credits) AS credit
WHERE DATE(usage_start_time) >= DATE_SUB(CURRENT_DATE(), INTERVAL 12 MONTH)
GROUP BY month
ORDER BY month DESC;
```

### Cost Optimization Recommendations
**Built-in Recommendations**:
- Active Assist provides automated recommendations
- Machine type right-sizing suggestions
- Idle VM detection
- Unattached disk identification
- Snapshot scheduling recommendations
- Committed use discount opportunities

**Recommender API**:
- Programmatic access to recommendations
- Integrate with automation tools
- Track recommendation adoption
- Calculate potential savings

**Recommendation Types and Savings**:
| Recommendation Type | Typical Savings | Implementation Complexity |
|--------------------|----------------|---------------------------|
| Stop idle VMs | 100% of VM cost | Low |
| Resize overprovisioned VMs | 30-50% | Medium |
| Delete unattached disks | 100% of disk cost | Low |
| Committed use discounts | 37-57% | Low |
| Storage class transitions | 50-95% | Low |
| Delete old snapshots | 100% of snapshot cost | Low |

### Showback and Chargeback
**Showback**: Reporting costs to teams without billing them
- Build dashboards showing team/project costs
- Regular cost reviews with stakeholders
- Trend analysis and forecasting
- Educate teams on cloud costs

**Chargeback**: Actual billing of costs to departments
- Allocate costs based on resource usage
- Use labels for precise attribution
- Monthly or quarterly invoicing to departments
- Incentivize cost optimization

**Implementation Approaches**:
1. **Direct Project Allocation**: Each team has own projects, costs directly attributed
2. **Label-based Allocation**: Shared resources, costs split by labels
3. **Usage-based Allocation**: Split shared service costs by usage metrics
4. **Hybrid**: Combination of above methods

**Example Chargeback Model**:
```
Shared Infrastructure Costs: $10,000/month
- Networking: $3,000
- Monitoring: $2,000
- Shared services: $5,000

Allocation by team based on compute usage:
- Team A (40% of compute): $4,000
- Team B (35% of compute): $3,500
- Team C (25% of compute): $2,500

Direct costs added:
- Team A total: $4,000 + $15,000 = $19,000
- Team B total: $3,500 + $12,000 = $15,500
- Team C total: $2,500 + $8,000 = $10,500
```

### Database Cost Optimization

#### Cloud SQL Optimization
**Right-sizing Instances**: Match resources to workload
- Monitor CPU, memory, disk I/O utilization
- Use performance insights to identify bottlenecks
- Start small and scale up based on metrics
- Consider high availability (HA) costs (2x instance cost)

**Cloud SQL Pricing Components**:
| Component | Pricing Factor | Optimization Strategy |
|-----------|---------------|----------------------|
| Instance (vCPU/RAM) | Per-second billing | Right-size, use shared-core for dev |
| Storage | Per GB/month | Start with minimum, autoresize enabled |
| Backups | Per GB stored | Retention policy, avoid unnecessary backups |
| Network egress | Per GB | Keep clients in same region |
| HA configuration | 2x instance cost | Only for production |

**Cloud SQL Cost Example**:
```
Workload: MySQL database
Option 1: db-n1-standard-4 (4 vCPU, 15 GB RAM) = $177.56/month
Option 2: db-n1-standard-2 (2 vCPU, 7.5 GB RAM) = $88.78/month
Option 3: db-custom-2-8192 (2 vCPU, 8 GB RAM) = $73.12/month

Storage: 100 GB SSD = $17/month
Backups: 100 GB × 7 days = $6.80/month

Total (Option 3 + storage + backups): $96.92/month
```

**Read Replicas**: Use for read scaling, not HA if not needed
- Read replicas cost same as primary instance
- Use for read-heavy workloads
- Replicas can be smaller than primary
- Consider application-level caching before adding replicas

**Connection Pooling**: Reduce connection overhead
- Use connection poolers (PgBouncer, ProxySQL)
- Reduce database connection overhead
- Lower memory usage on database
- Better handling of connection spikes

#### BigQuery Optimization
**Cost Control**:
- Partition tables by date for query pruning
- Cluster tables on filter/join columns
- Use query validators to estimate costs before running
- Implement slot reservations for predictable costs
- Use materialized views for repeated queries

**BigQuery Pricing Models**:
| Model | Use Case | Pricing | Best For |
|-------|----------|---------|----------|
| On-Demand | Variable workload | $6.25/TB scanned | Ad-hoc queries, unpredictable |
| Flat-rate | Consistent workload | $2,000+/month | 300+ TB/month scanned |
| Storage | Active/long-term | $0.020/$0.010 per GB | All workloads |

**Optimization Techniques**:
```
Query: SELECT * FROM large_table WHERE date = '2024-01-15'

Without partition: Scans entire table (10 TB) = $62.50
With date partition: Scans only 1 day (10 GB) = $0.0625
Savings: 99.9%
```

#### Cloud Spanner Optimization
**Node Configuration**:
- Minimum 3 nodes for production (1 for dev)
- Add nodes for every 10,000 QPS or 2 TB storage
- Regional: $0.90/node/hour ($657/node/month)
- Multi-regional: $3.00/node/hour ($2,190/node/month)

**Cost Optimization**:
```
Workload: 15,000 QPS, 5 TB storage, multi-regional
Nodes required: MAX(15,000/10,000, 5/2) = 3 nodes (minimum)
Cost: 3 nodes × $2,190 = $6,570/month

Alternative: Regional configuration
Cost: 3 nodes × $657 = $1,971/month
Savings: $4,599/month (70% reduction) - if regional latency acceptable
```

**Performance optimization**:
- Use interleaved tables for parent-child relationships
- Create secondary indexes for query patterns
- Batch writes for throughput
- Use mutations API for bulk operations
- Monitor CPU utilization, keep under 65% for headroom

#### Bigtable Optimization
**Node Sizing**:
- 1 node: Up to 10,000 QPS, 2.5 TB storage
- Add nodes for higher QPS or storage
- SSD: $0.65/hour/node ($474/month/node)
- HDD: $0.44/hour/node ($321/month/node)

**Storage Selection**:
```
Workload A: High QPS, low latency, 1 TB = Use SSD
Workload B: Batch analytics, 10 TB = Use HDD
Cost difference: SSD $474/node vs HDD $321/node (32% savings)
```

**Performance optimization**:
- Design row keys to distribute workload evenly
- Avoid sequential keys (timestamps) - use reversed timestamps or add prefix
- Pre-split tables for initial performance
- Monitor CPU, keep nodes under 70% utilization
- Use replication for HA, not just reads

## Performance Optimization

### Compute Performance

#### Machine Type Selection
**Machine Family Characteristics**:
| Family | vCPU | Memory/vCPU | Best For | Performance Notes |
|--------|------|-------------|----------|-------------------|
| E2 | 2-32 | 0.5-8 GB | Cost-optimized, scale-out | Variable CPU platform |
| N1 | 1-96 | 3.75 GB (std) | General purpose | Mature, widely available |
| N2 | 2-128 | 4 GB (std) | General purpose | 20% better price/perf than N1 |
| N2D | 2-224 | 4 GB (std) | General purpose | AMD EPYC, best price/perf |
| C2 | 4-60 | 4 GB/vCPU | Compute-intensive | 3.4 GHz turbo, high IPC |
| C2D | 2-112 | 4 GB/vCPU | Compute-intensive | AMD EPYC, excellent price/perf |
| M1 | 40-160 | 28.5 GB/vCPU | Memory-intensive | Up to 4 TB RAM |
| M2 | 208-416 | 28 GB/vCPU | Memory-intensive | Up to 12 TB RAM |
| A2 | 12-96 | Variable | GPU workloads | NVIDIA A100 GPUs |

**Performance Selection Guide**:
```
Web Application (mixed workload):
- Dev/Test: E2-medium (2 vCPU, 4 GB) - $30/month
- Production: N2-standard-4 (4 vCPU, 16 GB) - $146/month
- High traffic: C2-standard-8 (8 vCPU, 32 GB) - $281/month

Database (memory-intensive):
- Small: N2-highmem-4 (4 vCPU, 32 GB) - $220/month
- Large: M1-megamem-96 (96 vCPU, 1.4 TB) - $10,974/month

Batch Processing (compute-intensive):
- Standard: C2-standard-16 (16 vCPU, 64 GB) - $562/month
- Cost-optimized: Spot C2 (16 vCPU, 64 GB) - $84/month (85% discount)
```

#### Local SSD Performance
**Characteristics**:
- 375 GB per device, up to 24 devices (9 TB total)
- Very high IOPS: 680,000 read / 360,000 write (per VM)
- Low latency: <1 ms
- Ephemeral: Data lost when VM stops
- Cost: $0.080/GB/month (included in instance hours)

**Use Cases**:
- Scratch space for data processing
- Cache for frequently accessed data
- Temporary staging for ETL pipelines
- High-performance databases (if data replicated elsewhere)

**Performance Comparison**:
| Storage Type | IOPS (Read) | IOPS (Write) | Latency | Cost/GB |
|--------------|-------------|--------------|---------|---------|
| Standard PD | 7,500 | 15,000 | ~5 ms | $0.040 |
| SSD PD | 100,000 | 50,000 | ~1 ms | $0.170 |
| Local SSD | 680,000 | 360,000 | <1 ms | $0.080 |

#### GPU and TPU Optimization
**GPU Selection** (NVIDIA):
| GPU Model | Memory | Use Case | Price/hour |
|-----------|--------|----------|-----------|
| T4 | 16 GB | Inference, light training | $0.35 |
| V100 | 16 GB | Training, inference | $2.48 |
| A100 (40GB) | 40 GB | Large-scale training | $3.67 |
| A100 (80GB) | 80 GB | Largest models | $4.89 |

**TPU (Tensor Processing Units)**:
- Optimized for TensorFlow workloads
- v2: $4.50/hour (8 cores), 64 GB HBM
- v3: $8.00/hour (8 cores), 128 GB HBM
- v4: Best performance, contact sales
- Use TPU for large-scale training, GPU for inference

**Cost Optimization**:
```
ML Training Job: 100 hours compute needed
- V100 GPU: 100 hours × $2.48 = $248
- V100 Preemptible: 100 hours × $0.74 = $74 (70% savings)
- A100 (faster, 50 hours): 50 hours × $3.67 = $183.50
- TPU v3 (fastest, 30 hours): 30 hours × $8.00 = $240

Best choice: Preemptible V100 if time flexible, A100 if time critical
```

#### CPU Platform Selection
**Available Platforms**:
- Intel Cascade Lake (newest N1, C2)
- Intel Ice Lake (N2, C2)
- AMD EPYC Rome (N2D)
- AMD EPYC Milan (C2D, N2D)

**Selection Strategy**:
- Let GCP auto-select for best price/performance
- Specify platform for consistency (CI/CD)
- Newer platforms: Better performance, same cost
- AMD platforms: 10-15% better price/performance

#### Network Performance Optimization
**Network Throughput by Machine Type**:
| Machine Size | Network Bandwidth | Notes |
|--------------|------------------|-------|
| <8 vCPUs | 2 Gbps/vCPU | Up to 10 Gbps |
| 8-16 vCPUs | ~16 Gbps | Typical |
| 16-32 vCPUs | 32 Gbps | Tier 1 networking |
| 32+ vCPUs | 50-100 Gbps | Highest performance |

**Optimization Techniques**:
- Use larger instances for network-intensive workloads
- Enable gVNIC (Google Virtual NIC) for better performance
- Use Premium tier for lower latency
- Colocate communicating instances in same zone
- Use internal IPs for intra-VPC communication (free)

**Network Performance Testing**:
```bash
# Install iperf3
sudo apt-get install iperf3

# Server (receiving instance)
iperf3 -s

# Client (sending instance)
iperf3 -c SERVER_INTERNAL_IP -t 30 -P 4

# Expected results:
# Same zone: 20-100 Gbps (depending on instance size)
# Same region: 10-50 Gbps
# Cross-region: 5-20 Gbps (depends on regions)
```

### Storage Performance

#### Persistent Disk Performance Scaling
**Performance Characteristics**:
- IOPS and throughput scale with disk size
- Also scale with number of vCPUs on VM (up to limit)
- Read IOPS scale higher than write IOPS
- Performance caps based on disk type and VM size

**Performance Formulas**:

**Standard PD**:
- Read IOPS: 0.75 × GB (max 7,500)
- Write IOPS: 1.5 × GB (max 15,000)
- Throughput: 0.12 MB/s × GB (max 1,200 MB/s read, 400 MB/s write)

**Balanced PD**:
- Read/Write IOPS: 6 × GB (max 80,000)
- Throughput: 0.28 MB/s × GB (max 1,200 MB/s)

**SSD PD**:
- Read IOPS: 30 × GB (max 100,000)
- Write IOPS: 30 × GB (max 100,000)
- Throughput: 0.48 MB/s × GB (max 1,200 MB/s)

**Performance Examples**:
```
Application needs: 20,000 IOPS for database

Option 1: SSD PD
  Required size: 20,000 / 30 = 667 GB
  Cost: 667 GB × $0.17 = $113.39/month

Option 2: Balanced PD
  Required size: 20,000 / 6 = 3,334 GB
  Cost: 3,334 GB × $0.10 = $333.40/month

Option 3: Local SSD (if ephemeral acceptable)
  Required size: 375 GB minimum
  IOPS available: 680,000 (far exceeds requirement)
  Cost: 375 GB × $0.08 = $30/month

Best choice: SSD PD (best balance of performance, cost, durability)
```

#### Cloud Storage Performance
**Transfer Performance**:
- Use gsutil -m for parallel transfers (multiprocessing)
- Composite uploads for large files (>150 MB)
- Adjust chunk size for network conditions
- Use Cloud Storage FUSE for file system access (with limitations)

**Performance Tips**:
```bash
# Standard upload (single threaded)
gsutil cp large_file.zip gs://bucket/  # ~100 MB/s

# Parallel upload (multi-threaded)
gsutil -m cp -r directory/ gs://bucket/  # ~500-1000 MB/s

# Composite upload for large files
gsutil -o GSUtil:parallel_composite_upload_threshold=150M cp large_file.zip gs://bucket/

# Transfer from another region/cloud
gsutil -m rsync -r s3://source-bucket gs://destination-bucket
```

**Request Performance**:
- Standard: 5,000 writes/s, 5,000 reads/s per prefix
- Multi-region: Higher throughput for reads globally
- Use multiple prefixes to scale beyond limits
- Batch operations when possible

#### Filestore Performance
**Performance Tiers**:
| Tier | Capacity | Throughput/TB | IOPS/TB | Cost/GB |
|------|----------|--------------|---------|---------|
| Basic HDD | 1-63.9 TB | 100 MB/s | 1,000 | $0.20 |
| Basic SSD | 2.5-63.9 TB | 350 MB/s | 5,000 | $0.20 |
| High Scale SSD | 10-100 TB | 1,200 MB/s | 100,000 | $0.30 |
| Enterprise | 1-10 TB | 1,050 MB/s | 30,000 | $0.35 |

**Use Cases and Performance**:
```
Application: ML training with 500 GB of data files

Basic SSD: 2.5 TB × $0.20 = $500/month
  Performance: 2.5 TB × 350 MB/s = 875 MB/s (sufficient)

High Scale SSD: 10 TB × $0.30 = $3,000/month
  Performance: 10 TB × 1,200 MB/s = 12,000 MB/s (overkill)

Recommendation: Basic SSD tier for this workload
```

## Monitoring and Observability

### Cloud Monitoring Architecture
**Core Components**:
1. **Metrics**: Time-series data for resources and applications
2. **Logs**: Application and system logs via Cloud Logging
3. **Traces**: Distributed traces via Cloud Trace
4. **Profiler**: CPU and memory profiling via Cloud Profiler
5. **Dashboards**: Visualization and monitoring
6. **Alerting**: Notifications based on metric thresholds

**Metrics Collection**:
- Automatic collection for GCP services
- Custom metrics via Monitoring API
- OpenTelemetry for application metrics
- Prometheus integration for Kubernetes
- Third-party integrations (Datadog, New Relic)

**Data Retention**:
| Metric Resolution | Retention Period |
|------------------|------------------|
| 1-minute intervals | 6 weeks |
| 1-hour intervals | 1 year |
| 1-day intervals | 2 years |

### SLIs, SLOs, and SLAs

#### Service Level Indicators (SLIs)
**Common SLIs**:
- **Availability**: Uptime / (Uptime + Downtime)
- **Latency**: Percentage of requests < threshold (e.g., 95th percentile < 200ms)
- **Error Rate**: Errors / Total Requests
- **Throughput**: Requests per second
- **Durability**: Data integrity percentage

**Example SLI Definitions**:
```
API Service SLIs:
1. Availability: % of successful requests (non-5xx responses)
2. Latency: % of requests completed in <500ms (p99)
3. Error Rate: % of requests with 4xx/5xx errors
4. Correctness: % of responses with valid data schema
```

#### Service Level Objectives (SLOs)
**SLO Definition Framework**:
- Target percentage for SLI over time window
- Should be achievable but challenging
- Based on user expectations and business requirements
- Leave error budget for changes and incidents

**Example SLOs**:
```
API Service SLOs (30-day window):
1. Availability: 99.9% (error budget: 0.1% = 43 minutes/month)
2. Latency: 99% of requests < 500ms
3. Error Rate: <0.5% of all requests
4. Throughput: Handle 10,000 req/s sustained

Database SLOs:
1. Availability: 99.95% (error budget: 21 minutes/month)
2. Query Latency: 95% < 100ms, 99% < 500ms
3. Write Success: 99.99% of writes successful
```

**Error Budget Calculation**:
```
SLO: 99.9% availability over 30 days
Total time: 30 days × 24 hours × 60 minutes = 43,200 minutes
Allowed downtime: 43,200 × 0.001 = 43.2 minutes/month

Error budget remaining = 43.2 minutes - (actual downtime this month)

If error budget exhausted: Freeze non-critical deployments
```

#### Service Level Agreements (SLAs)
**GCP Service SLAs (examples)**:
| Service | SLA | Monthly Credit |
|---------|-----|----------------|
| Compute Engine | 99.99% (regional MIG) | 10-50% based on downtime |
| Cloud Storage | 99.95% (multi-region) | 10-25% |
| Cloud SQL | 99.95% (HA) | 10-50% |
| Cloud Spanner | 99.999% (multi-region) | 10-50% |
| BigQuery | 99.99% | 10-50% |

**SLA vs SLO Relationship**:
- SLA: External commitment with financial penalties
- SLO: Internal target, typically stricter than SLA
- SLI: Actual measured performance

### Alerting Policies

#### Alert Policy Configuration
**Components**:
1. **Conditions**: Metric threshold, aggregation, duration
2. **Notification Channels**: Email, SMS, PagerDuty, Slack, webhooks
3. **Documentation**: Playbook for responders
4. **Filters**: Resource labels, projects, regions

**Example Alerting Policies**:
```yaml
# High CPU Alert
Name: High CPU Utilization
Condition:
  Metric: compute.googleapis.com/instance/cpu/utilization
  Threshold: > 0.80 (80%)
  Duration: 5 minutes
  Aggregation: Mean across all instances
Notification: Email ops-team, PagerDuty
Documentation: "Check for runaway processes, consider scaling"

# High Error Rate Alert
Name: API Error Rate High
Condition:
  Metric: Custom metric - error_rate
  Threshold: > 0.05 (5%)
  Duration: 2 minutes
  Aggregation: Rate over 1-minute window
Notification: Email dev-team, Slack #incidents
Documentation: "Check logs, review recent deployments, rollback if needed"

# Budget Alert
Name: Monthly Budget 90% Reached
Condition:
  Budget threshold: 90% of $10,000
  Forecast: Enabled
Notification: Email finance-team, Pub/Sub topic
Documentation: "Review spend by project, consider cost optimization"
```

#### Alert Fatigue Prevention
**Best Practices**:
- Set appropriate thresholds (avoid too sensitive)
- Use duration windows (transient spikes)
- Implement alert grouping
- Use different channels for different severities
- Regular review and tuning of alerts
- Auto-resolve when condition clears

### Dashboards and Visualization

#### Dashboard Types
**1. Golden Signals Dashboard** (Latency, Traffic, Errors, Saturation):
```
Panels:
- Request Rate (requests/s over time)
- Error Rate (% errors over time)
- Request Latency (p50, p95, p99)
- Resource Saturation (CPU, memory, disk)
```

**2. Infrastructure Dashboard**:
```
Panels:
- Compute: CPU, memory, disk utilization by instance
- Network: Ingress/egress bandwidth, packets dropped
- Storage: IOPS, throughput, latency
- Database: Connections, query latency, replication lag
```

**3. Cost Dashboard**:
```
Panels:
- Daily spend trend
- Spend by service (pie chart)
- Spend by project (bar chart)
- Budget vs actual (gauge)
- Forecast for month-end
```

**4. Business Metrics Dashboard**:
```
Panels:
- Active users (from application metrics)
- Transactions per minute
- Revenue (if applicable)
- Conversion rate
```

### Cloud Trace and Cloud Profiler

#### Cloud Trace
**Purpose**: Distributed tracing for request latency analysis
- Trace spans across microservices
- Identify slow components
- Analyze request flow
- Detect bottlenecks

**Integration**:
```python
# Python example
from google.cloud import trace_v1
from google.cloud.trace_v1 import enums

tracer = trace_v1.TraceServiceClient()

# Create span
span = tracer.create_span(
    name="process-request",
    span_kind=enums.Span.SpanKind.SERVER,
    start_time=start_time,
    end_time=end_time
)
```

**Analysis**:
- View trace timeline
- Identify longest spans
- Compare similar requests
- Find RPC call latencies

#### Cloud Profiler
**Purpose**: Continuous CPU and memory profiling
- Identify performance bottlenecks in code
- Low overhead (<5% performance impact)
- Production-safe profiling
- Multi-language support (Java, Go, Python, Node.js)

**Use Cases**:
- High CPU usage investigation
- Memory leak detection
- Optimize hot code paths
- Resource usage by function

**Example Analysis**:
```
Application showing high CPU usage

Profiler reveals:
- 45% CPU: JSON parsing function
- 25% CPU: Database query execution
- 15% CPU: Business logic
- 15% CPU: Other

Actions:
1. Optimize JSON parsing (use faster library, reduce payload size)
2. Add database query caching
3. Profile again after changes to verify improvement
```

### Log-Based Metrics

**Creating Log-Based Metrics**:
```
# Example: Count 404 errors
Metric Name: http_404_errors
Filter: resource.type="http_load_balancer" AND httpRequest.status=404
Metric Type: Counter
Labels: httpRequest.requestUrl, httpRequest.requestMethod

# Example: Calculate request latency
Metric Name: request_latency_ms
Filter: resource.type="cloud_run_revision"
Metric Type: Distribution
Value: httpRequest.latency
Labels: resource.labels.service_name
```

**Use Cases**:
- Track custom application events
- Create alerts from log patterns
- Monitor security events
- Business metrics from logs

## Caching Strategies

### Cloud CDN (Content Delivery Network)

#### Overview and Configuration
**Cloud CDN**:
- Global edge caching for HTTP(S) Load Balancer
- 200+ edge locations worldwide
- Automatic cache invalidation
- Custom TTL settings
- Signed URLs for private content

**Pricing**:
| Region | Cache Egress | Cache Fill (Origin) |
|--------|-------------|---------------------|
| North America | $0.08/GB | $0.01/GB |
| Europe | $0.08/GB | $0.01/GB |
| Asia | $0.14/GB | $0.01/GB |
| Australia | $0.19/GB | $0.01/GB |

**Cache Hit Ratio Impact**:
```
Website: 100 TB/month total traffic, 80% cache hit ratio

Without CDN:
Origin egress: 100 TB × $0.11 = $11,000/month

With CDN (80% cache hit):
CDN egress (cache hits): 80 TB × $0.08 = $6,400
Origin egress (cache misses): 20 TB × $0.11 = $2,200
Cache fill: 20 TB × $0.01 = $200
Total: $8,800/month
Savings: $2,200/month (20% reduction)

Performance benefits:
- 50-80% lower latency for cached content
- Reduced origin server load
- Better global user experience
```

#### Cache Configuration Best Practices
**Cache Keys**:
- Default: Full URL including query strings
- Custom: Include/exclude specific headers, query parameters
- Consider user location, device type for personalization

**TTL Settings**:
```
Content Type → Recommended TTL:
- Static assets (images, CSS, JS): 1 year (31,536,000s)
- HTML pages: 5-10 minutes (300-600s)
- API responses: 1-60 seconds (or no-cache)
- Frequently changing content: 1-5 minutes (60-300s)
```

**Cache Control Headers**:
```http
# Maximum caching
Cache-Control: public, max-age=31536000, immutable

# Short-term caching with revalidation
Cache-Control: public, max-age=300, must-revalidate

# No caching
Cache-Control: private, no-cache, no-store, must-revalidate
```

### Memorystore (Redis and Memcached)

#### Redis vs Memcached Comparison
| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Rich (strings, lists, sets, hashes) | Key-value only |
| Persistence | Optional (RDB, AOF) | None |
| Replication | Yes (HA) | No |
| Max memory | 300 GB | 5 GB per node |
| Use case | Complex caching, sessions | Simple key-value cache |
| Cost (5 GB, Basic) | $82/month | $82/month |
| Cost (5 GB, Standard HA) | $164/month | N/A |

#### Redis Tiers and Pricing
**Basic Tier** (development, testing):
- Single zone, no replication
- Automatic failover within zone
- 1-300 GB capacity

**Standard Tier** (production):
- Cross-zone replication
- Automatic failover
- 99.9% availability SLA
- 5-300 GB capacity

**Pricing Example**:
```
Redis Basic M1 (1 GB): $29/month
Redis Basic M4 (5 GB): $82/month
Redis Standard M4 (5 GB, HA): $164/month
Redis Standard M5 (16 GB, HA): $475/month
```

#### Application-Level Caching Patterns

**1. Cache-Aside (Lazy Loading)**:
```python
def get_user_data(user_id):
    # Try cache first
    cached = redis.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)

    # Cache miss: fetch from database
    user_data = database.query(f"SELECT * FROM users WHERE id = {user_id}")

    # Store in cache with TTL
    redis.setex(f"user:{user_id}", 3600, json.dumps(user_data))

    return user_data
```

**2. Write-Through Cache**:
```python
def update_user_data(user_id, data):
    # Write to database
    database.update(f"UPDATE users SET ... WHERE id = {user_id}")

    # Update cache immediately
    redis.setex(f"user:{user_id}", 3600, json.dumps(data))
```

**3. Write-Behind Cache** (asynchronous):
```python
def update_user_data(user_id, data):
    # Write to cache immediately
    redis.setex(f"user:{user_id}", 3600, json.dumps(data))

    # Queue database update for async processing
    pubsub.publish("db_updates", {"user_id": user_id, "data": data})
```

#### Caching Cost/Performance Impact
**Scenario: E-commerce product catalog**:
```
Without caching:
- 10,000 req/s to database
- Database: Cloud SQL db-n1-standard-8 = $354/month
- Database needs: 4 read replicas = $1,416/month
- Total: $1,770/month

With Redis caching (90% hit rate):
- 1,000 req/s to database (10% miss rate)
- Database: Cloud SQL db-n1-standard-2 = $177/month
- Redis: Standard M5 (16 GB) = $475/month
- Total: $652/month
- Savings: $1,118/month (63% reduction)

Performance improvement:
- Cache hit latency: <1ms
- Database query latency: 10-50ms
- Average latency reduction: 85%
```

### Multi-Layer Caching Strategy

**Layered Approach**:
1. **Browser Cache**: Cache-Control headers (free, fastest)
2. **CDN Cache**: Cloud CDN for static assets (low cost, global)
3. **Application Cache**: Redis/Memcached for hot data (moderate cost, fast)
4. **Database Cache**: Query cache, materialized views (included, moderate)
5. **Disk Cache**: SSD for frequently accessed data (moderate cost, slower)

**Example Architecture**:
```
Request flow:
1. Browser cache (static assets) → 50% hit rate
2. CDN cache (static content) → 30% hit rate (of remaining)
3. Redis cache (dynamic data) → 15% hit rate (of remaining)
4. Database (cache miss) → 5% of total requests

Effective cache hit rate: 95%
Database load reduction: 20x
Cost savings: 60-70%
```

### Cache Invalidation Strategies

**Time-Based (TTL)**:
- Set expiration time on cache entries
- Simple, but may serve stale data
- Good for data with predictable freshness requirements

**Event-Based**:
- Invalidate cache when data changes
- Pub/Sub or database triggers to notify cache
- Ensures data freshness, more complex

**Version-Based**:
- Include version number in cache key
- Increment version on update
- Old cached data naturally expires

**Tag-Based**:
- Group related cache entries with tags
- Invalidate all entries with specific tag
- Useful for complex relationships

**Example Invalidation**:
```python
# Time-based
redis.setex("product:123", ttl=300, value=data)

# Event-based
def on_product_update(product_id):
    redis.delete(f"product:{product_id}")
    redis.delete(f"product_list:category:{category_id}")

# Version-based
version = redis.incr(f"product:{product_id}:version")
redis.setex(f"product:{product_id}:v{version}", 3600, data)

# Tag-based (using Redis sets)
redis.sadd("tag:electronics", "product:123")
# Invalidate all electronics
for key in redis.smembers("tag:electronics"):
    redis.delete(key)
```

## Cost Optimization Scenarios

### Scenario 1: High-Traffic E-Commerce Website

**Current Architecture**:
```
- 50 n1-standard-4 VMs (always on) = $6,084/month
- Cloud SQL MySQL (db-n1-standard-16, HA) = $1,417/month
- 100 TB Cloud Storage (Standard) = $2,000/month
- 30 TB egress = $3,300/month
- Load Balancer = $36/month
Total: $12,837/month
```

**Problems**:
- Overprovisioned compute (40% average utilization)
- Non-production environments running 24/7
- No caching layer
- All storage in Standard class
- No CDN for static assets

**Optimized Solution**:
```
Compute:
- 20 n2d-standard-4 VMs (production, with CUD) = $1,374/month (57% discount)
- 15 e2-standard-4 VMs (autoscaling peak) = $411/month
- 15 e2-standard-4 VMs (dev/staging, scheduled 40 hrs/wk) = $73/month
Subtotal: $1,858/month

Database:
- Cloud SQL MySQL (db-n1-standard-8, HA) = $709/month
- Memorystore Redis (16 GB, Standard) = $475/month
Subtotal: $1,184/month

Storage:
- 20 TB Cloud Storage (Standard, hot data) = $400/month
- 50 TB Cloud Storage (Nearline, warm data) = $500/month
- 30 TB Cloud Storage (Coldline, archives) = $120/month
Subtotal: $1,020/month

Network:
- Cloud CDN (80% hit rate): $2,720/month
- Origin egress (20% of 30 TB): $660/month
- Load Balancer: $36/month
Subtotal: $3,416/month

Total: $7,478/month
Savings: $5,359/month (42% reduction)
```

**Key Optimizations**:
1. Right-sized compute with CUDs (37% compute savings)
2. Implemented autoscaling (reduced baseline instances)
3. Resource scheduling for non-production (88% savings)
4. Added Redis caching layer (reduced database size by 50%)
5. Storage lifecycle policies (67% storage savings)
6. Cloud CDN implementation (27% network savings)

### Scenario 2: Data Analytics Platform

**Current Architecture**:
```
- BigQuery on-demand: 500 TB scanned/month = $3,125/month
- Dataflow: 20 n1-standard-4 VMs × 730 hours = $2,434/month
- Cloud Storage: 200 TB Standard = $4,000/month
- Dataproc: 10 n1-standard-4 VMs × 730 hours = $1,217/month
Total: $10,776/month
```

**Problems**:
- Inefficient BigQuery queries scanning full tables
- Dataflow jobs running continuously with variable load
- No storage lifecycle management
- Dataproc cluster always on for sporadic jobs

**Optimized Solution**:
```
BigQuery:
- Partitioned/clustered tables: 50 TB scanned/month = $312/month
- Materialized views for common queries = $50/month
Subtotal: $362/month

Dataflow:
- Flex templates with autoscaling: 8 e2-standard-4 VMs avg = $584/month
- Preemptible workers (80% of workload) = $117/month
Subtotal: $701/month

Storage:
- 50 TB Standard (active data) = $1,000/month
- 100 TB Nearline (monthly access) = $1,000/month
- 50 TB Coldline (quarterly access) = $200/month
Subtotal: $2,200/month

Dataproc:
- Ephemeral clusters (job-specific) = $122/month (10% of previous)
- Preemptible workers = $24/month
Subtotal: $146/month

Total: $3,409/month
Savings: $7,367/month (68% reduction)
```

**Key Optimizations**:
1. BigQuery partitioning/clustering (90% query cost reduction)
2. Dataflow autoscaling and preemptible workers (71% savings)
3. Storage lifecycle policies (45% storage savings)
4. Ephemeral Dataproc clusters (88% savings)
5. Preemptible VMs for fault-tolerant workloads

### Scenario 3: Global SaaS Application

**Current Architecture**:
```
- 30 n1-standard-8 VMs across 3 regions = $5,720/month
- Cloud Spanner multi-region (9 nodes) = $19,710/month
- Load Balancer (global) = $36/month
- Network egress: 20 TB = $2,200/month
Total: $27,666/month
```

**Problems**:
- Overprovisioned Spanner (30% CPU utilization)
- No caching layer
- Regional instances not optimized
- High cross-region traffic

**Optimized Solution**:
```
Compute:
- 20 n2-standard-8 VMs (3-year CUD) = $1,836/month (55% discount)
- GKE Autopilot for microservices = $800/month (avg)
Subtotal: $2,636/month

Database:
- Cloud Spanner multi-region (5 nodes) = $10,950/month
- Memorystore Redis (3 × 16 GB, Standard) = $1,425/month
Subtotal: $12,375/month

Network:
- Premium tier with Cloud CDN = $1,540/month
- Optimized regional routing = $660/month
- Load Balancer = $36/month
Subtotal: $2,236/month

Total: $17,247/month
Savings: $10,419/month (38% reduction)
```

**Key Optimizations**:
1. 3-year CUDs for predictable workload (55% savings)
2. Right-sized Spanner based on actual QPS (44% savings)
3. Redis caching reduces Spanner load
4. GKE Autopilot for efficient resource utilization
5. Optimized network routing reduces cross-region traffic

### Scenario 4: Machine Learning Training Workloads

**Current Architecture**:
```
- 10 n1-standard-32 with V100 GPUs × 730 hrs = $18,324/month
- 50 TB Cloud Storage Standard = $1,000/month
- Filestore High Scale (10 TB) = $3,000/month
Total: $22,324/month
```

**Problems**:
- GPU instances running continuously but training sporadic
- Expensive Filestore tier for infrequent access
- Not using preemptible instances
- No automated shutdown

**Optimized Solution**:
```
Compute:
- 10 n1-standard-32 with V100 Spot GPUs × 300 hrs = $2,220/month
- Automated start/stop for training jobs
Subtotal: $2,220/month

Storage:
- 10 TB Filestore Basic SSD = $2,000/month
- 50 TB Cloud Storage (Standard for datasets) = $1,000/month
- 20 TB Cloud Storage (Nearline for old models) = $200/month
Subtotal: $3,200/month

Total: $5,420/month
Savings: $16,904/month (76% reduction)
```

**Key Optimizations**:
1. Spot VMs with checkpointing (88% GPU savings)
2. Automated job scheduling and shutdown
3. Right-sized Filestore tier (33% savings)
4. Storage lifecycle for model artifacts
5. Training orchestration to maximize GPU utilization

### Scenario 5: Development and Testing Environment

**Current Architecture**:
```
- 40 n1-standard-4 VMs (24/7) = $4,867/month
- Cloud SQL PostgreSQL (db-n1-standard-4, HA) × 3 = $1,063/month
- 20 TB Cloud Storage Standard = $400/month
Total: $6,330/month
```

**Problems**:
- Dev/test running 24/7
- Production-like HA not needed for testing
- Overprovisioned instances
- No resource quotas or policies

**Optimized Solution**:
```
Compute:
- 25 e2-standard-4 VMs (Mon-Fri, 8am-6pm) = $273/month
- 5 n1-standard-4 VMs (always-on shared services) = $608/month
Subtotal: $881/month

Database:
- Cloud SQL PostgreSQL (db-n1-standard-2, no HA) × 3 = $266/month
Subtotal: $266/month

Storage:
- 20 TB Cloud Storage Standard = $400/month
Subtotal: $400/month

Total: $1,547/month
Savings: $4,783/month (76% reduction)
```

**Key Optimizations**:
1. Resource scheduling (82% compute savings)
2. Right-sized instances (E2 for dev workloads)
3. Removed unnecessary HA (75% database savings)
4. Organization policies to prevent overprovisioning
5. Automated environment creation/deletion

### Scenario 6: Microservices on Kubernetes

**Current Architecture**:
```
- GKE Standard: 30 n1-standard-8 nodes = $5,720/month
- Cloud SQL (3 databases, HA) = $2,126/month
- Load Balancer = $36/month
- 10 TB egress = $1,100/month
Total: $8,982/month
```

**Problems**:
- Overprovisioned nodes (avg 40% utilization)
- Always-on nodes for variable workload
- No caching or optimization
- Standard tier GKE management overhead

**Optimized Solution**:
```
Compute:
- GKE Autopilot (pay per pod): $2,100/month avg
- Spot pods for batch workloads: $315/month
Subtotal: $2,415/month

Database:
- Cloud SQL (1 shared, db-n1-standard-4, HA) = $709/month
- 2 Cloud SQL (smaller, db-n1-standard-2) = $354/month
- Memorystore Redis (8 GB) = $246/month
Subtotal: $1,309/month

Network:
- Internal load balancers = $36/month
- Optimized egress: 5 TB = $550/month
Subtotal: $586/month

Total: $4,310/month
Savings: $4,672/month (52% reduction)
```

**Key Optimizations**:
1. GKE Autopilot for efficient resource allocation (58% compute savings)
2. Spot pods for fault-tolerant workloads
3. Database consolidation and right-sizing (38% savings)
4. Redis caching reduces database load
5. Optimized microservices communication (50% egress reduction)

### Scenario 7: Media Streaming Platform

**Current Architecture**:
```
- 100 c2-standard-8 VMs (transcoding) = $28,068/month
- 500 TB Cloud Storage Standard = $10,000/month
- 200 TB egress = $22,000/month
- Load Balancer = $36/month
Total: $60,104/month
```

**Problems**:
- Expensive compute for sporadic transcoding
- All content in Standard storage
- No CDN for content delivery
- High egress costs

**Optimized Solution**:
```
Compute:
- 20 c2-standard-8 VMs (baseline, CUD) = $2,420/month
- 80 c2-standard-8 Spot VMs (transcoding) = $4,214/month
Subtotal: $6,634/month

Storage:
- 100 TB Standard (new/popular content) = $2,000/month
- 200 TB Nearline (recent content) = $2,000/month
- 200 TB Coldline (archive content) = $800/month
Subtotal: $4,800/month

Network:
- Cloud CDN (90% hit rate): $14,400/month
- Origin egress (10% of 200 TB): $2,200/month
- Media CDN for video optimization: included
- Load Balancer: $36/month
Subtotal: $16,636/month

Total: $28,070/month
Savings: $32,034/month (53% reduction)
```

**Key Optimizations**:
1. Spot VMs for transcoding (76% compute savings)
2. Storage lifecycle based on content popularity (52% savings)
3. Cloud CDN/Media CDN (27% network savings)
4. Committed use for baseline capacity
5. Optimized video encoding settings

### Scenario 8: Batch Processing Pipeline

**Current Architecture**:
```
- 50 n1-standard-16 VMs (24/7) = $12,168/month
- 100 TB Cloud Storage Standard = $2,000/month
- Cloud Pub/Sub = $80/month
- Cloud Functions = $120/month
Total: $14,368/month
```

**Problems**:
- Batch jobs only run 8 hours/day
- VMs idle 16 hours/day
- Expensive VM types for batch work
- No use of managed services

**Optimized Solution**:
```
Compute:
- Dataflow (autoscaling, preemptible): $1,460/month
- Cloud Composer (workflow orchestration): $300/month
- 10 e2-standard-16 Spot VMs (custom jobs): $292/month
Subtotal: $2,052/month

Storage:
- 100 TB Cloud Storage Standard = $2,000/month
Subtotal: $2,000/month

Orchestration:
- Cloud Pub/Sub = $80/month
- Cloud Functions = $120/month
- Cloud Scheduler = $5/month
Subtotal: $205/month

Total: $4,257/month
Savings: $10,111/month (70% reduction)
```

**Key Optimizations**:
1. Dataflow for managed batch processing (83% compute savings)
2. Preemptible workers for fault-tolerant jobs
3. Automated orchestration with Cloud Composer
4. Spot VMs for remaining custom workloads
5. Pay-per-use model instead of always-on infrastructure

## Performance Troubleshooting

### Common Performance Issues and Resolutions

#### High Latency Issues
**Symptoms**: Slow response times, timeouts, poor user experience

**Common Causes and Solutions**:
1. **Database bottleneck**:
   - Add read replicas for read-heavy workloads
   - Implement Redis/Memcached caching
   - Optimize slow queries (use Query Insights)
   - Add appropriate indexes
   - Consider connection pooling

2. **Network latency**:
   - Use Premium tier for lower latency
   - Deploy resources closer to users (multi-region)
   - Implement Cloud CDN for static content
   - Use Cloud Interconnect for hybrid connectivity
   - Optimize cross-region data transfer

3. **Compute resource saturation**:
   - Scale up (larger machine type) or scale out (more instances)
   - Enable autoscaling
   - Use load balancing for distribution
   - Identify hot spots with Cloud Profiler

4. **Application-level issues**:
   - Profile code with Cloud Profiler
   - Optimize algorithmic complexity
   - Implement async processing for long tasks
   - Use Cloud Trace to identify slow spans

#### High Error Rates
**Symptoms**: 5xx errors, failed requests, service unavailability

**Common Causes and Solutions**:
1. **Resource exhaustion**:
   - Increase quotas
   - Scale resources
   - Implement rate limiting
   - Add circuit breakers

2. **Configuration issues**:
   - Review recent changes
   - Check firewall rules
   - Verify IAM permissions
   - Validate service configurations

3. **Dependency failures**:
   - Implement retry logic with exponential backoff
   - Use Circuit Breaker pattern
   - Add health checks
   - Implement fallback mechanisms

#### Resource Utilization Issues
**Symptoms**: High CPU, memory exhaustion, disk I/O saturation

**Solutions by Resource Type**:

**CPU**:
- Identify hot code paths with Cloud Profiler
- Optimize algorithms
- Scale horizontally or vertically
- Use more compute-optimized machine types (C2, C2D)

**Memory**:
- Identify memory leaks
- Optimize data structures
- Use memory-optimized machine types (M1, M2, M3)
- Implement object pooling

**Disk I/O**:
- Use SSD persistent disks instead of Standard
- Increase disk size for more IOPS
- Use Local SSD for extreme performance
- Implement caching layer

**Network**:
- Use larger machine types (better network bandwidth)
- Optimize data transfer patterns
- Implement compression
- Use internal IPs for intra-VPC communication

## Cost/Performance Trade-off Decision Framework

### Decision Matrix
| Requirement | Cost-Optimized Solution | Balanced Solution | Performance-Optimized Solution |
|-------------|------------------------|-------------------|-------------------------------|
| Compute | E2, Spot VMs, Scheduled | N2, N2D with CUDs | C2, C2D, On-demand |
| Storage | Coldline/Archive, HDD | Nearline, Balanced PD | Standard, SSD PD, Local SSD |
| Database | Shared-core, scheduled | Standard, read replicas | High-mem, multiple replicas, caching |
| Network | Standard tier, regional | Premium tier, CDN | Premium tier, CDN, Interconnect |
| Caching | Minimal, TTL-based | Redis Basic, CDN | Redis Standard HA, multi-layer |

### When to Prioritize Cost
- Development and testing environments
- Batch processing with flexible timelines
- Archive and backup storage
- Non-critical applications with lenient SLAs
- Startups with limited budgets
- Proof-of-concept projects

**Strategies**:
- Use Spot/Preemptible VMs
- Implement resource scheduling
- Use cheaper storage classes
- Standard network tier
- Minimal redundancy for non-production
- On-demand or serverless where possible

### When to Prioritize Performance
- Customer-facing applications
- Real-time processing requirements
- Financial transactions
- Gaming platforms
- Video streaming
- High-traffic e-commerce during peak seasons

**Strategies**:
- Premium compute tiers (C2, M2)
- SSD or Local SSD storage
- Premium network tier
- Multi-region deployment
- Advanced caching strategies
- Over-provision for headroom

### Balancing Cost and Performance
**Steps to Find Optimal Balance**:
1. **Define SLOs**: Establish clear performance targets
2. **Baseline**: Measure current performance and costs
3. **Identify bottlenecks**: Use monitoring and profiling
4. **Incremental optimization**: Start with highest ROI optimizations
5. **Monitor and iterate**: Continuously track metrics
6. **Error budget**: Use SLO error budget to guide trade-offs

**Example Decision Process**:
```
API Service Requirements:
- p99 latency < 500ms (SLO)
- 99.9% availability (SLO)
- 10,000 req/s average, 20,000 req/s peak

Current: n1-standard-8 with Cloud SQL and no caching
Cost: $5,000/month, p99 latency: 800ms (violates SLO)

Option A: Add Redis caching
Additional cost: $500/month
Expected p99 latency: 200ms (meets SLO)
Decision: Implement (required to meet SLO)

Option B: Upgrade to C2-standard-8
Additional cost: $1,200/month
Expected p99 latency: 300ms (meets SLO)
Decision: Skip (caching is more cost-effective)

Option C: Add read replicas
Additional cost: $1,200/month
Expected p99 latency: 400ms (meets SLO)
Decision: Implement if caching insufficient
```

## Exam Tips and Focus Areas

### Professional-Level Cost Optimization Questions

**Common Question Patterns**:
1. **Scenario-based optimization**: Given architecture, identify cost savings opportunities
2. **Trade-off analysis**: Balance cost, performance, and reliability
3. **Pricing model selection**: Choose appropriate pricing for workload characteristics
4. **ROI calculations**: Calculate savings from optimizations
5. **Service selection**: Choose most cost-effective service for requirement

**Key Concepts to Master**:
- Committed use discounts vs sustained use discounts (when to use each)
- Spot/Preemptible VMs (use cases and limitations)
- Storage class selection based on access patterns
- Network egress costs and optimization strategies
- BigQuery pricing (on-demand vs flat-rate, partitioning impact)
- Database sizing and read replica considerations

**Example Exam Questions**:

**Q1**: "A company runs batch processing jobs that process 100 TB of data daily. Jobs take 8 hours and can tolerate interruptions. Current cost is $15,000/month using on-demand VMs. What optimization reduces cost most?"

**Answer**: Use Spot VMs with checkpointing (80-91% discount). Implement job orchestration to handle preemption. Expected cost: $1,500-3,000/month (80-90% savings).

**Q2**: "An application stores 500 TB of compliance data. Data must be retained for 7 years but accessed less than once per year. Retrieval can take hours. Current cost: $10,000/month in Standard storage. What storage class is most cost-effective?"

**Answer**: Archive storage ($0.0012/GB vs $0.020/GB). Cost: $600/month (94% savings). Meets compliance and access requirements.

**Q3**: "A global application serves 100 TB/month with 70% cacheable content. Users are worldwide. What architecture minimizes cost while maintaining performance?"

**Answer**: Implement Cloud CDN (cache 70 TB at edge). Use multi-region storage for origin. Reduce origin egress by 70%. Estimated savings: 25-30% on network costs.

### Performance Optimization Patterns for Exam

**Key Performance Concepts**:
1. **Caching layers**: Browser → CDN → Application → Database
2. **Database optimization**: Indexes, query optimization, read replicas, caching
3. **Compute selection**: Machine families for different workloads
4. **Storage performance**: IOPS/GB calculations, disk sizing
5. **Network optimization**: Colocation, Premium tier, CDN
6. **Monitoring**: SLIs, SLOs, SLAs, error budgets

**Performance Troubleshooting Patterns**:
- High latency → Add caching, optimize queries, check network
- High error rate → Check resources, review recent changes, verify dependencies
- Resource saturation → Scale up/out, optimize application, use better instance types
- Variable performance → Implement autoscaling, use load balancing

### ROI Calculation Patterns

**Formula**: ROI = (Savings - Implementation Cost) / Implementation Cost × 100%

**Example 1: Adding Redis Cache**:
```
Current: 4 Cloud SQL read replicas = $1,416/month
Implementation: Redis Standard (16 GB) = $475/month + setup time
After: 1 primary database + Redis = $354 + $475 = $829/month

Monthly savings: $1,416 - $829 = $587
Annual savings: $7,044
Implementation cost: $475 + 40 hours × $100 = $4,475
First-year ROI: ($7,044 - $4,475) / $4,475 = 57%
Payback period: 7.6 months
```

**Example 2: Committed Use Discounts**:
```
Current: 20 n1-standard-8 on-demand = $3,813/month
Implementation: 3-year CUD = $1,719/month (55% discount)

Monthly savings: $2,094
Annual savings: $25,128
Implementation cost: $0 (just commit)
ROI: Infinite (no implementation cost)
Risk: Locked in for 3 years, architecture changes may reduce value
```

### Quick Reference for Exam

**Cost Optimization Priorities (highest to lowest impact)**:
1. Committed use discounts for baseline workloads (37-57% savings)
2. Spot/Preemptible VMs for fault-tolerant workloads (60-91% savings)
3. Storage lifecycle policies (50-95% savings on aged data)
4. Resource scheduling for non-production (60-80% savings)
5. Right-sizing based on utilization (20-40% savings)
6. Cloud CDN for high-egress workloads (20-30% network savings)
7. Network tier optimization (15-30% egress savings)

**Performance Optimization Priorities**:
1. Caching (Redis/CDN) - 70-90% latency reduction
2. Database optimization (indexes, query tuning) - 50-80% improvement
3. Compute right-sizing (machine type selection) - 20-50% improvement
4. Network optimization (colocation, Premium tier) - 30-60% improvement
5. Application profiling and optimization - Variable impact

**Service Selection Decision Tree**:
```
Need relational database?
├─ Yes → Multi-region? → Yes → Cloud Spanner
│        └─ No → Need scale? → Yes → Cloud Spanner
│              └─ No → Cloud SQL
└─ No → Key-value? → Yes → Bigtable (high throughput) or Memorystore (caching)
      └─ Analytics? → BigQuery
```

**Remember for Exam**:
- Sustained use discounts are automatic (no action needed)
- Committed use discounts require commitment (1 or 3 years)
- Preemptible/Spot VMs: 24-hour max (preemptible), no max (Spot)
- Storage classes: Standard > Nearline (30 days) > Coldline (90 days) > Archive (365 days)
- Network: Ingress free (mostly), egress charged, Premium > Standard tier
- BigQuery: $6.25/TB scanned, partition/cluster to reduce scanning
- Cloud Spanner: 3 nodes minimum (production), 1 node (dev/test)

## gcloud Commands for Cost and Performance Management

```bash
# === Cost Management ===

# List all recommenders available
gcloud recommender recommenders list

# View machine type recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1 \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --format="table(name,primaryImpact.costProjection.cost.units,state)"

# View idle VM recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1 \
  --recommender=google.compute.instance.IdleResourceRecommender

# View committed use discount recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1 \
  --recommender=google.compute.commitment.UsageCommitmentRecommender

# View disk recommendations (idle, snapshot)
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1 \
  --recommender=google.compute.disk.IdleResourceRecommender

# View billing accounts
gcloud billing accounts list

# Link project to billing account
gcloud billing projects link PROJECT_ID \
  --billing-account=BILLING_ACCOUNT_ID

# View project billing information
gcloud billing projects describe PROJECT_ID

# View quotas for project
gcloud compute project-info describe --project=PROJECT_ID

# View region-specific quotas
gcloud compute regions describe us-central1 --project=PROJECT_ID

# === Instance Management ===

# Stop instances (cost optimization)
gcloud compute instances stop INSTANCE_NAME --zone=ZONE

# Start instances
gcloud compute instances start INSTANCE_NAME --zone=ZONE

# Create spot/preemptible instance
gcloud compute instances create INSTANCE_NAME \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# Create instance with committed use discount
gcloud compute instances create INSTANCE_NAME \
  --zone=us-central1-a \
  --machine-type=n2-standard-4

# Create custom machine type
gcloud compute instances create INSTANCE_NAME \
  --zone=us-central1-a \
  --custom-cpu=8 \
  --custom-memory=16GB

# === Monitoring and Performance ===

# Create dashboard
gcloud monitoring dashboards create --config-from-file=dashboard.json

# List dashboards
gcloud monitoring dashboards list

# Create alert policy
gcloud alpha monitoring policies create --policy-from-file=policy.yaml

# List alert policies
gcloud alpha monitoring policies list

# Describe metrics
gcloud monitoring metrics-descriptors list \
  --filter="metric.type:compute.googleapis.com"

# === Storage Management ===

# Set lifecycle policy on bucket
gsutil lifecycle set lifecycle.json gs://BUCKET_NAME

# View lifecycle policy
gsutil lifecycle get gs://BUCKET_NAME

# View storage usage
gsutil du -sh gs://BUCKET_NAME

# Set storage class
gsutil rewrite -s NEARLINE gs://BUCKET_NAME/**

# === BigQuery Optimization ===

# Estimate query cost (dry run)
bq query --use_legacy_sql=false --dry_run 'SELECT * FROM dataset.table'

# View table details (partitioning, clustering)
bq show --format=prettyjson PROJECT:DATASET.TABLE

# Create partitioned table
bq mk --table \
  --time_partitioning_field timestamp \
  --time_partitioning_type DAY \
  PROJECT:DATASET.TABLE \
  schema.json

# Export billing data to BigQuery (API call, not gcloud)
# Done via Console or API

# === Performance Testing ===

# SSH into instance for performance testing
gcloud compute ssh INSTANCE_NAME --zone=ZONE

# View instance details
gcloud compute instances describe INSTANCE_NAME --zone=ZONE

# List machine types
gcloud compute machine-types list --filter="zone:us-central1-a"

# View disk performance limits
gcloud compute disk-types describe pd-ssd --zone=us-central1-a
```

## Terraform Examples for Cost-Optimized Architectures

```hcl
# Example 1: Spot VM with autoscaling
resource "google_compute_instance_template" "spot_template" {
  name         = "spot-worker-template"
  machine_type = "n2-standard-4"

  disk {
    source_image = "debian-cloud/debian-11"
    auto_delete  = true
    boot         = true
  }

  scheduling {
    preemptible         = true
    automatic_restart   = false
    provisioning_model  = "SPOT"
  }

  network_interface {
    network = "default"
  }
}

# Example 2: Storage with lifecycle policy
resource "google_storage_bucket" "optimized_bucket" {
  name     = "cost-optimized-bucket"
  location = "US"

  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type          = "SetStorageClass"
      storage_class = "NEARLINE"
    }
  }

  lifecycle_rule {
    condition {
      age = 90
    }
    action {
      type          = "SetStorageClass"
      storage_class = "COLDLINE"
    }
  }

  lifecycle_rule {
    condition {
      age = 365
    }
    action {
      type = "Delete"
    }
  }
}

# Example 3: Budget alert
resource "google_billing_budget" "project_budget" {
  billing_account = "BILLING_ACCOUNT_ID"
  display_name    = "Project Monthly Budget"

  budget_filter {
    projects = ["projects/${var.project_id}"]
  }

  amount {
    specified_amount {
      currency_code = "USD"
      units         = "10000"
    }
  }

  threshold_rules {
    threshold_percent = 0.5
  }

  threshold_rules {
    threshold_percent = 0.9
  }

  threshold_rules {
    threshold_percent = 1.0
  }

  all_updates_rule {
    monitoring_notification_channels = [
      google_monitoring_notification_channel.email.id
    ]
  }
}

# Example 4: Committed use discount (requires reservation)
resource "google_compute_reservation" "cpu_reservation" {
  name = "cpu-reservation"
  zone = "us-central1-a"

  specific_reservation {
    count = 10
    instance_properties {
      machine_type = "n2-standard-4"
    }
  }

  specific_reservation_required = false
}
```

## Additional Resources

### Official Google Cloud Documentation
- [Cost Optimization Best Practices](https://cloud.google.com/architecture/best-practices-for-cost-optimization)
- [GCP Pricing Calculator](https://cloud.google.com/products/calculator)
- [Performance Optimization Guide](https://cloud.google.com/architecture/framework/performance-optimization)
- [Billing Documentation](https://cloud.google.com/billing/docs)
- [Resource Hierarchy for Cost Management](https://cloud.google.com/billing/docs/onboarding-checklist)
- [Committed Use Discounts](https://cloud.google.com/compute/docs/instances/committed-use-discounts-overview)
- [Spot VMs Documentation](https://cloud.google.com/compute/docs/instances/spot)
- [Cloud Monitoring Documentation](https://cloud.google.com/monitoring/docs)
- [SRE Book - Implementing SLOs](https://sre.google/workbook/implementing-slos/)

### Pricing and Cost Resources
- [All GCP Pricing](https://cloud.google.com/pricing)
- [Compute Engine Pricing](https://cloud.google.com/compute/all-pricing)
- [Cloud Storage Pricing](https://cloud.google.com/storage/pricing)
- [BigQuery Pricing](https://cloud.google.com/bigquery/pricing)
- [Cloud SQL Pricing](https://cloud.google.com/sql/pricing)
- [Network Pricing](https://cloud.google.com/vpc/network-pricing)

### Performance and Optimization
- [Cloud Architecture Center](https://cloud.google.com/architecture)
- [Performance Testing Guide](https://cloud.google.com/architecture/framework/performance-optimization/test-performance)
- [Database Best Practices](https://cloud.google.com/sql/docs/postgres/best-practices)
- [BigQuery Best Practices](https://cloud.google.com/bigquery/docs/best-practices-performance-overview)
- [Compute Engine Best Practices](https://cloud.google.com/compute/docs/tutorials)

### Training and Certification
- [Professional Cloud Architect Exam Guide](https://cloud.google.com/certification/cloud-architect)
- [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)
- [Architecture Framework](https://cloud.google.com/architecture/framework)
- [Case Studies](https://cloud.google.com/customers)

### Tools and Calculators
- [Pricing Calculator](https://cloud.google.com/products/calculator)
- [TCO Calculator](https://cloud.google.com/tco)
- [Carbon Footprint Dashboard](https://cloud.google.com/carbon-footprint)
- [Active Assist](https://cloud.google.com/active-assist)
- [Recommender API](https://cloud.google.com/recommender/docs)
