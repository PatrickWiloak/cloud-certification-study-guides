# OCI Architect Associate - Database, Security, and Operations

## Table of Contents
- [Database Solutions](#database-solutions)
- [Security Architecture](#security-architecture)
- [Monitoring and Operations](#monitoring-and-operations)
- [Cost Management](#cost-management)
- [Backup and Recovery](#backup-and-recovery)
- [Exam Tips](#exam-tips)

---

## Database Solutions

### Autonomous Database Architecture

#### Workload-Specific Optimization

**ATP (Transaction Processing)**:
```
Characteristics:
- OLTP workloads
- High concurrency (thousands of concurrent users)
- Mixed workload (reads/writes)
- Short transactions
- Optimized for: Response time, throughput

Use Cases:
- E-commerce applications
- Banking systems
- SaaS applications
- Customer-facing apps
```

**ADW (Data Warehouse)**:
```
Characteristics:
- OLAP workloads
- Complex queries
- Large data scans
- Parallel processing
- Optimized for: Query performance, analytics

Use Cases:
- Business intelligence
- Data warehousing
- Analytics platforms
- Reporting systems
```

**APEX (Application Express)**:
```
Characteristics:
- Low-code development platform
- Integrated with Autonomous Database
- Rapid application development
- Web-based applications

Use Cases:
- Internal tools
- Dashboards
- Quick prototypes
- Business applications
```

#### Autonomous Database Features

**Auto-Scaling**:
```bash
oci db autonomous-database create \
  --db-name "proddb" \
  --cpu-core-count 2 \
  --is-auto-scaling-enabled true \
  --auto-scaling-for-storage-enabled true

# Auto-scaling behavior:
# CPU: Can scale up to 3x base OCPU count
# Storage: Automatically expands as needed
# No downtime during scaling
```

**Performance Tiers**:
```
Service Levels:
├── LOW: Lowest resource allocation, highest concurrency
├── MEDIUM: Balanced resources and concurrency
└── HIGH: Highest resources, lowest concurrency

Connection Strings:
proddb_high = (description=...)    # For critical queries
proddb_medium = (description=...)  # For standard queries
proddb_low = (description=...)     # For reporting, batch jobs
```

**Private Endpoint**:
```bash
# Create private endpoint for VCN-only access
oci db autonomous-database create \
  --db-name "securedb" \
  --cpu-core-count 2 \
  --subnet-id ocid1.subnet.oc1... \
  --nsg-ids '["ocid1.networksecuritygroup.oc1..."]' \
  --private-endpoint-label "proddb"

# Benefits:
# - No internet exposure
# - VCN-only access
# - Integrates with NSGs
# - Private IP address
```

**Access Control Lists (ACLs)**:
```bash
# Restrict access to specific IPs (public endpoint)
oci db autonomous-database update \
  --autonomous-database-id ocid1.autonomousdatabase.oc1... \
  --whitelisted-ips '["203.0.113.0/24","198.51.100.10/32"]'
```

#### Autonomous Database Operations

**Backup and Restore**:
```bash
# Manual backup
oci db autonomous-database create-backup \
  --autonomous-database-id ocid1.autonomousdatabase.oc1... \
  --display-name "pre-upgrade-backup"

# Restore to point-in-time
oci db autonomous-database restore \
  --autonomous-database-id ocid1.autonomousdatabase.oc1... \
  --timestamp "2024-10-10T14:30:00Z"

# Clone database
oci db autonomous-database create-clone \
  --source-id ocid1.autonomousdatabase.oc1... \
  --clone-type FULL \
  --display-name "proddb-test-clone"
```

**Maintenance Windows**:
```bash
# Set preferred maintenance window
oci db autonomous-database update \
  --autonomous-database-id ocid1.autonomousdatabase.oc1... \
  --preferred-maintenance-window '{
    "preference":"CUSTOM_PREFERENCE",
    "months":[{"name":"JANUARY"},{"name":"APRIL"}],
    "weeksOfMonth":[1,2],
    "daysOfWeek":[{"name":"SATURDAY"},{"name":"SUNDAY"}],
    "hoursOfDay":[2,3,4],
    "leadTimeInWeeks":2
  }'
```

### DB Systems Architecture

#### High Availability Options

**Data Guard Configuration**:
```
Primary DB System (AD-1)
    ↓
Standby DB System (AD-2)
    ↓ (optional)
Second Standby (AD-3)

Replication Modes:
├── MAXIMUM_AVAILABILITY: No data loss, slight performance impact
├── MAXIMUM_PERFORMANCE: Async, minimal performance impact
└── MAXIMUM_PROTECTION: Zero data loss, highest performance impact
```

**Creating Data Guard**:
```bash
oci db data-guard-association create \
  --database-id ocid1.database.oc1... \
  --creation-type NewDbSystem \
  --database-admin-password "Welcome123!" \
  --protection-mode MAXIMUM_PERFORMANCE \
  --transport-type ASYNC \
  --display-name "standby-db" \
  --availability-domain "Uocm:PHX-AD-2" \
  --subnet-id ocid1.subnet.oc1... \
  --shape "VM.Standard2.2"
```

**Switchover and Failover**:
```bash
# Planned switchover (no data loss)
oci db data-guard-association switchover \
  --database-id ocid1.database.oc1... \
  --data-guard-association-id ocid1.dataguardassociation.oc1... \
  --database-admin-password "Welcome123!"

# Emergency failover
oci db data-guard-association failover \
  --database-id ocid1.database.oc1... \
  --data-guard-association-id ocid1.dataguardassociation.oc1... \
  --database-admin-password "Welcome123!"
```

**RAC (Real Application Clusters)**:
```bash
# Launch 2-node RAC
oci db system launch \
  --compartment-id ocid1.compartment.oc1... \
  --availability-domain "Uocm:PHX-AD-1" \
  --subnet-id ocid1.subnet.oc1... \
  --shape "BM.DenseIO2.52" \
  --node-count 2 \
  --database-edition "ENTERPRISE_EDITION_EXTREME_PERFORMANCE" \
  --cluster-name "prodrac" \
  --db-name "proddb"

# Benefits:
# - Active-Active clustering
# - Automatic failover
# - Load balancing
# - Zero downtime patching
```

#### DB System Storage

**Storage Options**:
```
Logical Volume Manager (LVM):
├── Standard shapes (VM/BM)
├── Flexible storage sizing
└── 256 GB - 40 TB

Automatic Storage Management (ASM):
├── Exadata shapes only
├── Advanced storage management
├── Automatic rebalancing
└── High performance
```

**Expanding Storage**:
```bash
oci db system update \
  --db-system-id ocid1.dbsystem.oc1... \
  --data-storage-size-in-gbs 1024

# Note: Can only increase, not decrease
# No downtime required
```

### MySQL Database Service

**High Availability**:
```bash
oci mysql db-system create \
  --compartment-id ocid1.compartment.oc1... \
  --shape-name "MySQL.VM.Standard.E3.1.8GB" \
  --subnet-id ocid1.subnet.oc1... \
  --admin-username "admin" \
  --admin-password "Welcome123!" \
  --availability-domain "Uocm:PHX-AD-1" \
  --is-highly-available true

# HA Configuration:
# - 3 instances (primary + 2 replicas)
# - Automatic failover
# - Data replication across ADs
# - 99.95% SLA
```

**HeatWave Integration**:
```bash
# Enable HeatWave analytics engine
oci mysql db-system heatwave-cluster add \
  --db-system-id ocid1.mysqldbsystem.oc1... \
  --cluster-size 2 \
  --shape-name "MySQL.HeatWave.VM.Standard.E3"

# Benefits:
# - In-database analytics
# - 400x faster queries
# - No ETL needed
# - Same MySQL database
```

### NoSQL Database Cloud Service

**Use Cases**:
- IoT applications
- Mobile apps
- Gaming (leaderboards, player data)
- Real-time analytics
- Session storage

**Creating NoSQL Table**:
```bash
oci nosql table create \
  --compartment-id ocid1.compartment.oc1... \
  --name "UserSessions" \
  --ddl-statement "CREATE TABLE UserSessions (
    userId STRING,
    sessionId STRING,
    timestamp TIMESTAMP,
    data JSON,
    PRIMARY KEY(SHARD(userId), sessionId)
  )" \
  --table-limits '{
    "maxReadUnits":50,
    "maxWriteUnits":50,
    "maxStorageInGBs":25
  }'
```

---

## Security Architecture

### Defense in Depth

**Security Layers**:
```
Layer 7: Application Security
├── WAF (Web Application Firewall)
├── Input validation
└── HTTPS/TLS

Layer 6-5: Data Security
├── Encryption at rest (Vault)
├── Encryption in transit (TLS)
└── Data masking/redaction

Layer 4: Network Security
├── NSGs (Network Security Groups)
├── Security Lists
├── Private subnets
└── DDoS protection

Layer 3: Identity Security
├── IAM policies
├── MFA enforcement
├── Federation
└── Least privilege

Layer 2: Infrastructure Security
├── Isolated network virtualization
├── Dedicated hosts
└── Security Zones

Layer 1: Physical Security
├── Oracle data centers
├── Physical access controls
└── Hardware security modules
```

### IAM Advanced Patterns

#### Compartment Strategy

**Enterprise Compartment Design**:
```
Tenancy (root)
├── Security (Shared)
│   ├── Vault
│   ├── Keys
│   └── Secrets
├── Network (Shared)
│   ├── Hub-VCN
│   ├── Prod-VCN
│   └── NonProd-VCN
├── Production
│   ├── App-A-Prod
│   │   ├── Compute
│   │   ├── Database
│   │   └── Storage
│   └── App-B-Prod
│       ├── Compute
│       ├── Database
│       └── Storage
└── NonProduction
    ├── App-A-Dev
    ├── App-A-Test
    ├── App-B-Dev
    └── App-B-Test
```

**Cross-Compartment Access**:
```
# Policy in root compartment for cross-compartment access
Allow group AppA-Admins to manage instance-family in compartment App-A-Prod
Allow group AppA-Admins to use virtual-network-family in compartment Network:Prod-VCN

# Database admins access databases across compartments
Allow group DB-Admins to manage database-family in compartment Production
Allow group DB-Admins to manage autonomous-database-family in compartment Production
```

#### Advanced Policy Patterns

**Time-Based Access**:
```
# Allow access only during business hours
Allow group Contractors to use instance-family in compartment Development
  where request.utcTime.hour >= '09:00:00'
  and request.utcTime.hour <= '17:00:00'
```

**Tag-Based Access**:
```
# Allow access only to resources with specific tag
Allow group ProjectX-Team to manage all-resources in compartment Development
  where target.resource.tag.Project = 'ProjectX'

# Prevent deletion of protected resources
Allow group Operators to manage instances in compartment Production
  where target.resource.tag.Protected != 'true'
  or request.operation != 'DeleteInstance'
```

**Conditional Access by Resource State**:
```
# Allow stop/start but not terminate
Allow group Developers to use instances in compartment Development
Allow group Developers to manage instance-family in compartment Development
  where request.operation != 'TerminateInstance'
```

### Encryption Strategy

#### Data Encryption Architecture

**At Rest**:
```
Block Volumes → Vault (Customer-Managed Keys)
    ├── Production: prod-encryption-key
    └── Development: dev-encryption-key

Object Storage → Vault (Customer-Managed Keys)
    ├── Sensitive buckets: high-security-key
    └── Standard buckets: standard-key

Database → TDE (Transparent Data Encryption)
    ├── Autonomous: Always encrypted
    └── DB Systems: TDE with Vault keys
```

**In Transit**:
```
All OCI Traffic:
├── Inter-service: TLS 1.2+ (automatic)
├── Client to OCI: HTTPS/TLS
├── FastConnect: MACsec encryption (optional)
└── VPN: IPSec encryption
```

**Key Rotation**:
```bash
# Create new key version
oci kms management key create-key-version \
  --key-id ocid1.key.oc1... \
  --management-endpoint "https://..."

# Update resource to use new key version
oci bv volume update \
  --volume-id ocid1.volume.oc1... \
  --kms-key-id ocid1.key.oc1...

# Automatic rotation schedule
oci kms management key update \
  --key-id ocid1.key.oc1... \
  --management-endpoint "https://..." \
  --auto-rotation-period-in-days 90
```

### Cloud Guard Implementation

**Detector Recipes**:
```
Configuration Detectors:
├── Public Bucket Detection
├── Overly Permissive Security Lists
├── Instance With Public IP in Private Subnet
├── Unencrypted Block Volume
└── Database Without Backup

Activity Detectors:
├── Unusual API Activity
├── Bucket Deleted
├── IAM Policy Changes
├── Instance Stopped/Terminated
└── User Created/Deleted

Threat Detectors:
├── Suspicious IP Access
├── Potential Data Exfiltration
├── Crypto Mining Activity
└── Malware Detection
```

**Responder Recipes**:
```bash
# Example: Automatically disable public bucket access
{
  "responderRules": [{
    "responderRuleId": "MAKE_BUCKET_PRIVATE",
    "details": {
      "mode": "AUTOACTION"  # or "USERACTION" for notification only
    }
  }]
}
```

**Target Configuration**:
```bash
oci cloud-guard target create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "production-monitoring" \
  --target-resource-id ocid1.compartment.oc1.production \
  --target-resource-type COMPARTMENT \
  --target-detector-recipes '[{
    "detectorRecipeId":"ocid1.cloudguarddetectorrecipe.oc1..."
  }]' \
  --target-responder-recipes '[{
    "responderRecipeId":"ocid1.cloudguardresponderrecipe.oc1..."
  }]'
```

### Security Zones

**Maximum Security Recipe**:
```
Enforced Policies:
├── Encryption: All resources must use encryption
├── Public Access: No public Object Storage buckets
├── Logging: Audit logging must be enabled
├── Networking: Restrictive security rules only
├── Backups: Database backups required
└── Monitoring: Monitoring must be configured
```

**Moving Compartment to Security Zone**:
```bash
# Resources must comply before move
oci cloud-guard security-zone update \
  --security-zone-id ocid1.securityzone.oc1... \
  --security-zone-recipe-id ocid1.securityzonerecipe.oc1..maximum \
  --target-compartment-id ocid1.compartment.oc1.production
```

---

## Monitoring and Operations

### Comprehensive Monitoring

#### Monitoring Architecture

**Multi-Tier Application Monitoring**:
```
Application Layer:
├── APM (Application Performance Monitoring)
├── Custom application metrics
└── Log Analytics

Infrastructure Layer:
├── Compute metrics (CPU, Memory, Disk, Network)
├── Storage metrics (IOPS, Throughput)
├── Network metrics (Bandwidth, Packets)
└── Database metrics (Connections, Transactions)

Service Layer:
├── Load Balancer metrics
├── API Gateway metrics
└── Function metrics
```

**Alarm Strategy**:
```terraform
# Critical alarms (immediate action)
resource "oci_monitoring_alarm" "high_cpu" {
  compartment_id        = var.compartment_id
  display_name          = "CRITICAL: High CPU Usage"
  is_enabled            = true
  severity              = "CRITICAL"
  metric_compartment_id = var.compartment_id
  namespace             = "oci_computeagent"
  query                 = "CpuUtilization[1m].mean() > 90"
  destinations          = [oci_ons_topic.critical_alerts.id]

  message_format = "ONS_OPTIMIZED"
  repeat_notification_duration = "PT2H"
}

# Warning alarms (investigation needed)
resource "oci_monitoring_alarm" "elevated_cpu" {
  display_name = "WARNING: Elevated CPU Usage"
  severity     = "WARNING"
  query        = "CpuUtilization[5m].mean() > 75"
  destinations = [oci_ons_topic.warning_alerts.id]
}

# Info alarms (awareness)
resource "oci_monitoring_alarm" "backup_completion" {
  display_name = "INFO: Backup Completed"
  severity     = "INFO"
  query        = "BackupCompleted[1h].count() > 0"
  destinations = [oci_ons_topic.info_alerts.id]
}
```

#### Logging Strategy

**Centralized Logging**:
```
Log Groups by Environment:
├── production-logs
│   ├── vcn-flow-logs
│   ├── lb-access-logs
│   ├── lb-error-logs
│   ├── api-gateway-logs
│   └── application-logs
├── security-logs
│   ├── audit-logs (automatic)
│   ├── waf-logs
│   └── cloud-guard-logs
└── development-logs
    └── debug-logs
```

**Log Search Queries**:
```
# Find errors in last hour
search "compartment-ocid/production-logs" |
  where data.level='ERROR' |
  where datetime >= timestamp'2024-10-10 14:00:00'

# Find slow API responses
search "compartment-ocid/api-gateway-logs" |
  where data.responseTime > 5000 |
  sort by data.responseTime desc

# Find failed authentication attempts
search "compartment-ocid/audit-logs" |
  where data.message =~ 'authentication failed' |
  stats count() by data.sourceAddress
```

**Log Archival**:
```bash
# Configure log archival to Object Storage
oci logging log update \
  --log-id ocid1.log.oc1... \
  --configuration '{
    "archiving": {
      "isEnabled": true,
      "objectNamePrefix": "logs/",
      "bucketName": "log-archive"
    }
  }'
```

### Service Connector Hub

**Definition**: Route data between OCI services automatically.

**Common Patterns**:

**Logs to Object Storage**:
```
Logging Service
    ↓
Service Connector
    ↓
Object Storage (long-term archive)
```

**Logs to Streaming**:
```
Logging Service
    ↓
Service Connector
    ↓
Streaming
    ↓
External SIEM/Analytics
```

**Metrics to Monitoring**:
```
Custom Application
    ↓
Service Connector
    ↓
Monitoring Service
```

**Configuration**:
```bash
oci sch service-connector create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "logs-to-storage" \
  --source '{
    "kind":"logging",
    "logSources":[{
      "compartmentId":"ocid1.compartment.oc1...",
      "logGroupId":"ocid1.loggroup.oc1...",
      "logId":"ocid1.log.oc1..."
    }]
  }' \
  --target '{
    "kind":"objectStorage",
    "bucketName":"log-archive",
    "namespace":"namespace"
  }'
```

### Events and Automation

**Event-Driven Operations**:

**Auto-Remediation**:
```
Event: Instance Terminated
    ↓
Events Service
    ↓
Function: "RestartCriticalInstance"
    ↓
Launch replacement instance
```

**Notification Workflows**:
```
Event: High Cost Alert
    ↓
Events Service
    ↓
Notifications
    ↓
├── Email to Finance
├── Slack to Ops Team
└── PagerDuty to On-Call
```

**Event Rule Examples**:
```bash
# Alert on any IAM change
oci events rule create \
  --compartment-id ocid1.compartment.oc1... \
  --display-name "iam-changes-alert" \
  --is-enabled true \
  --condition '{
    "eventType":[
      "com.oraclecloud.identitycontrolplane.createpolicy",
      "com.oraclecloud.identitycontrolplane.updatepolicy",
      "com.oraclecloud.identitycontrolplane.deletepolicy"
    ]
  }' \
  --actions '{
    "actions":[{
      "actionType":"ONS",
      "isEnabled":true,
      "topicId":"ocid1.onstopic.oc1..."
    }]
  }'
```

---

## Cost Management

### Cost Optimization Strategies

#### Compute Cost Optimization

**Right-Sizing**:
```bash
# Analyze utilization
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1d].mean()' \
  --start-time "2024-09-10T00:00:00Z" \
  --end-time "2024-10-10T00:00:00Z"

# If average < 30%, consider downsizing
oci compute instance update \
  --instance-id ocid1.instance.oc1... \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus":1.0,"memoryInGBs":8.0}'
```

**Instance Scheduling**:
```python
# Stop non-production instances after hours
import oci
from datetime import datetime

config = oci.config.from_file()
compute = oci.core.ComputeClient(config)

# Define schedule
STOP_HOUR = 18  # 6 PM
START_HOUR = 8  # 8 AM

def schedule_instances():
    current_hour = datetime.now().hour

    # Get all dev instances
    instances = compute.list_instances(
        compartment_id=config["compartment_id"],
        lifecycle_state="RUNNING"
    ).data

    for instance in instances:
        if "dev-" in instance.display_name.lower():
            if current_hour >= STOP_HOUR or current_hour < START_HOUR:
                compute.instance_action(
                    instance.id,
                    "STOP"
                )
```

**Reserved Capacity**:
```bash
# Commit to capacity for discounts
oci compute capacity-reservation create \
  --availability-domain "Uocm:PHX-AD-1" \
  --compartment-id ocid1.compartment.oc1... \
  --instance-reservation-configs '[{
    "instanceShape":"VM.Standard.E4.Flex",
    "reservedCount":10
  }]'

# Launch instances using reservation
oci compute instance launch \
  --capacity-reservation-id ocid1.capacityreservation.oc1... \
  --shape "VM.Standard.E4.Flex"
```

#### Storage Cost Optimization

**Object Storage Tiering**:
```bash
# Enable Auto-Tiering
oci os bucket create \
  --name "application-data" \
  --storage-tier Standard \
  --auto-tiering Enabled

# Manual lifecycle policy
oci os object-lifecycle-policy put \
  --bucket-name "application-data" \
  --items '[{
    "name":"archive-old-data",
    "action":"ARCHIVE",
    "timeAmount":180,
    "timeUnit":"DAYS",
    "isEnabled":true
  }]'
```

**Block Volume Optimization**:
```bash
# Identify unused volumes
oci bv volume list \
  --compartment-id ocid1.compartment.oc1... \
  --lifecycle-state AVAILABLE \
  --query "data[?\"is-hydrated\"==\`true\`].{id:id, name:\"display-name\", size:\"size-in-gbs\"}"

# Delete or backup unused volumes
```

### Budgets and Cost Control

**Budget Configuration**:
```bash
# Monthly budget with alerts
oci budgets budget create \
  --amount 10000 \
  --reset-period MONTHLY \
  --target-type COMPARTMENT \
  --target-compartment-id ocid1.compartment.oc1... \
  --display-name "Production Monthly Budget" \
  --alert-rule-recipients '["finance@example.com"]'

# Alert rules
oci budgets alert-rule create \
  --budget-id ocid1.budget.oc1... \
  --type ACTUAL \
  --threshold 80 \
  --threshold-type PERCENTAGE \
  --recipients '["ops-team@example.com"]' \
  --message "Production spending at 80%"
```

**Cost Allocation Tags**:
```
Tagging Strategy:
├── Environment: Production, Development, Testing
├── CostCenter: Engineering, Marketing, Sales
├── Project: ProjectA, ProjectB, ProjectC
└── Owner: team-alpha, team-beta

Policy:
- Require tags on all resources
- Automated tagging via IaC
- Regular tag audits
```

---

## Backup and Recovery

### Backup Strategies

#### Comprehensive Backup Architecture

**3-2-1 Backup Rule**:
```
3 copies of data:
├── Production (primary)
├── Local backups (same region, different AD)
└── Remote backups (different region)

2 different media:
├── Block Volume backups
└── Object Storage (archived backups)

1 offsite copy:
└── Cross-region replicated backups
```

**Database Backup Strategy**:
```
Autonomous Database:
├── Automatic daily backups (60-day retention)
├── Manual backups (before major changes)
└── Clones for testing

DB Systems:
├── Automatic backups (via policy: Gold, Silver, Bronze)
├── Manual backups (pre-maintenance)
├── Cross-region backup copy
└── Data Guard for HA/DR
```

**Application Data Backup**:
```
Compute:
├── Boot volume backups (weekly)
├── Custom images (golden images)
└── User data backups (daily)

Block Volumes:
├── Volume group backups (consistent)
├── Policy-based automated backups
└── Cross-region copy

File Storage:
├── Snapshots (hourly/daily)
└── Snapshot lifecycle policy

Object Storage:
├── Versioning enabled
├── Cross-region replication
└── Archive tier for long-term
```

### Disaster Recovery Planning

**DR Runbook Template**:
```markdown
# DR Failover Procedure

## Pre-Requisites
- [ ] DR site is up-to-date (replicated)
- [ ] Network connectivity verified
- [ ] DNS records ready to update
- [ ] Team notified and assembled

## Failover Steps
1. Verify primary site is down
2. Activate DR database standby
3. Update DNS to DR region
4. Scale up DR compute resources
5. Update application configuration
6. Verify application functionality
7. Monitor for issues

## Rollback Plan
1. Verify primary site is recovered
2. Sync data from DR to primary
3. Update DNS back to primary
4. Scale down DR resources
```

**Testing Schedule**:
```
Monthly: Table-top exercises
Quarterly: Partial DR drill (single application)
Annually: Full DR drill (entire environment)
```

---

## Exam Tips

### Critical Concepts

**Database**:
- Autonomous: Self-managing, auto-scaling, 99.95% SLA
- ATP vs ADW: OLTP vs OLAP workloads
- Data Guard: Active-passive HA with standby
- RAC: Active-active clustering
- MySQL HeatWave: In-database analytics

**Security**:
- Defense in depth: Multiple security layers
- IAM policies: Least privilege, compartment-based
- Vault: Customer-managed encryption keys
- Cloud Guard: Detection and response
- Security Zones: Prevention (blocks violations)

**Monitoring**:
- Metrics: 1-minute granularity
- Alarms: Threshold-based alerts
- Logging: Centralized log management
- Events: Automated actions on state changes
- Service Connector: Route data between services

**Cost**:
- Right-sizing: Match resources to workload
- Commitment: Monthly/Annual Flex for discounts
- Scheduling: Stop non-production after hours
- Storage tiering: Auto-tier or lifecycle policies
- Budgets: Set alerts at 50%, 80%, 100%

### Common Exam Scenarios

**Q: Zero downtime database patching?**
A: Autonomous Database or RAC

**Q: Protect against accidental deletion in Object Storage?**
A: Enable versioning

**Q: Automatic remediation of security violations?**
A: Cloud Guard with responder recipes (autoaction mode)

**Q: Cost optimization for variable workload?**
A: Autoscaling + Right-sizing + Instance scheduling

**Q: Disaster recovery with 15-minute RPO?**
A: Data Guard with async replication or Pilot Light DR

---

## Summary

**Databases**: Autonomous (self-managing), DB Systems (flexible), MySQL (HeatWave), NoSQL (key-value)

**Security**: Defense in depth, IAM, Vault, Cloud Guard, Security Zones

**Operations**: Monitoring, logging, events, automation, Service Connector

**Cost Management**: Right-sizing, scheduling, tiering, budgets, tags

**DR**: Backups, replication, cross-region, testing

---

**Congratulations!** You've completed the OCI Architect Associate study materials!
