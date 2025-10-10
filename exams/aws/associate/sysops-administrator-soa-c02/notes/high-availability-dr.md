# Reliability and Business Continuity (SOA-C02)

This domain covers 16% of the exam and focuses on implementing scalable, highly available, and resilient solutions with effective backup and disaster recovery strategies.

## Scalability and Elasticity

### Amazon EC2 Auto Scaling

**Auto Scaling Groups (ASG)**
- Minimum, desired, and maximum capacity
- Health check grace period (default: 300 seconds)
- Default termination policy (oldest launch template, closest to billing hour)
- Multiple AZ distribution for high availability

**Launch Templates vs Launch Configurations**
- Launch Templates: Versioning, multiple instance types, mix of on-demand and spot
- Launch Configurations: Legacy, single instance type, no versioning
- **Best Practice**: Always use launch templates

**Scaling Policies**

1. **Target Tracking**
   - Maintain metric at target value (e.g., CPU at 70%)
   - Simple to configure
   - Automatic scale-up and scale-down
   - Examples: Average CPU, Request count per target, Network bytes

2. **Step Scaling**
   - Scale based on alarm breach size
   - Different adjustments for different threshold ranges
   - More control than target tracking
   - Requires CloudWatch alarm

3. **Simple Scaling**
   - Single scaling adjustment
   - Cooldown period prevents rapid scaling
   - Less flexible than step scaling
   - Legacy approach (use step scaling instead)

4. **Scheduled Scaling**
   - Time-based scaling actions
   - Predictable traffic patterns
   - One-time or recurring schedules
   - Examples: Business hours, end-of-month processing

**Scaling Cooldowns**
- Default: 300 seconds
- Prevents rapid successive scaling
- Step scaling: Continuously evaluates (better than simple)
- Warm-up time: Instance initialization period

**Lifecycle Hooks**
- Pause instances during launch/termination
- Perform custom actions (logging, data backup)
- Timeout: 1 hour (default), max 48 hours
- Complete lifecycle action or timeout expires

**Health Checks**
- EC2 status checks (default)
- ELB health checks (recommended with load balancer)
- Custom health checks (via API)
- Grace period: Time before first health check

**Termination Policies**
- Default: Oldest launch configuration, then closest to billing hour
- OldestInstance: Terminate oldest instances
- NewestInstance: Terminate newest instances
- OldestLaunchConfiguration
- ClosestToNextInstanceHour
- Custom: Using termination protection

### Application Load Balancer (ALB)

**Features**
- Layer 7 (HTTP/HTTPS)
- Content-based routing (path, host, headers, query strings)
- WebSocket and HTTP/2 support
- Native IPv6 support
- Integration with WAF

**Target Types**
- EC2 instances
- IP addresses (on-premises, containers)
- Lambda functions
- Application Load Balancer (ALB chaining)

**Health Checks**
- Protocol: HTTP or HTTPS
- Path: Health check endpoint
- Port: Target port or override
- Interval: 5-300 seconds (default: 30)
- Timeout: 2-120 seconds (default: 5)
- Healthy/Unhealthy threshold: Consecutive checks (default: 5/2)
- Success codes: 200, or range (200-299)

**Sticky Sessions**
- Duration-based: 1 second to 7 days
- Application-controlled: Custom cookie
- Target group level configuration
- Use case: Maintain user session state

**Cross-Zone Load Balancing**
- Enabled by default for ALB
- Distribute traffic evenly across all AZs
- No additional charges for ALB

**Routing Rules**
- Host-based: api.example.com vs www.example.com
- Path-based: /api/* vs /images/*
- HTTP header-based: User-Agent, custom headers
- Query string parameters
- Source IP address

### Network Load Balancer (NLB)

**Features**
- Layer 4 (TCP, UDP, TLS)
- Ultra-low latency (millions of requests per second)
- Static IP per AZ (Elastic IP support)
- Preserve source IP address
- Connection-based routing

**Use Cases**
- High performance requirements
- Static IP needed (whitelisting)
- TCP/UDP protocols
- Gaming applications
- IoT workloads

**Health Checks**
- TCP, HTTP, HTTPS protocols
- Fast failover (10 seconds)
- Connection-based health monitoring

**Cross-Zone Load Balancing**
- Disabled by default for NLB
- Charges apply when enabled
- Reduces AZ failure impact

### Gateway Load Balancer (GWLB)

**Purpose**
- Deploy, scale, and manage third-party virtual appliances
- Firewalls, intrusion detection, deep packet inspection
- Single entry/exit point for traffic

**Architecture**
- GENEVE protocol on port 6081
- Transparent network gateway
- Integrated health checks

## High Availability Architectures

### Multi-AZ Deployments

**Amazon RDS Multi-AZ**
- Synchronous replication to standby
- Automatic failover (60-120 seconds)
- No manual intervention required
- Backups from standby (no performance impact)
- Maintenance on standby first
- Endpoint remains same after failover

**Failover Scenarios**
- Primary DB instance failure
- AZ outage
- Instance type change
- OS patching
- Manual failover (reboot with failover)

**Aurora Multi-AZ**
- Shared storage across 3 AZs (6 copies)
- Up to 15 read replicas
- Failover in less than 30 seconds
- Storage auto-scaling (10GB to 128TB)
- Continuous backup to S3

**Aurora Global Database**
- Cross-region disaster recovery
- <1 second replication lag
- Up to 5 secondary regions
- RTO < 1 minute for disaster recovery

**ElastiCache Multi-AZ**
- Redis: Multi-AZ with automatic failover
- Memcached: Partitioning across nodes (no replication)
- Redis cluster mode: Sharding + replication

### Multi-Region Architectures

**Route 53 Routing Policies**

1. **Failover Routing**
   - Primary and secondary resources
   - Health checks on primary
   - Automatic failover to secondary
   - Active-passive DR pattern

2. **Geolocation Routing**
   - Route based on user location
   - Content localization
   - License restrictions by geography
   - Default location for non-matching queries

3. **Geoproximity Routing**
   - Route based on resource and user location
   - Bias: Shift traffic toward or away from resources
   - Traffic flow visualization

4. **Latency-Based Routing**
   - Route to lowest latency endpoint
   - Measured AWS network latency
   - Improved user experience

5. **Weighted Routing**
   - Distribute traffic proportionally
   - A/B testing
   - Blue/green deployments
   - Gradual migration

6. **Multivalue Answer Routing**
   - Return multiple IP addresses
   - Health checks on each
   - Client-side load balancing
   - Up to 8 healthy records

**Health Checks**
- Endpoint monitoring (IP or domain)
- Monitor CloudWatch alarm
- Calculated health check (combine multiple checks)
- HTTP, HTTPS, TCP protocols
- String matching in response body
- Fast (30s) or standard (10s) intervals

**S3 Cross-Region Replication (CRR)**
- Asynchronous replication
- Requires versioning on source and destination
- Different AWS accounts supported
- Selective replication (prefix, tags)
- Delete marker replication (optional)
- Replication Time Control (RTC): 15-minute SLA

**CloudFront for Global Distribution**
- Edge locations worldwide
- Low latency content delivery
- DDoS protection with Shield
- Origin failover (primary and secondary origins)
- Geo-restriction capabilities

## Backup and Restore Strategies

### AWS Backup

**Centralized Backup Management**
- Cross-service backup policies
- Automated backup schedules
- Lifecycle management
- Cross-region and cross-account backup
- Compliance reporting

**Supported Services**
- EC2, EBS, EFS, FSx
- RDS, Aurora, DynamoDB
- DocumentDB, Neptune
- Storage Gateway, S3

**Backup Plans**
- Backup rules: Schedule, retention, lifecycle
- Resource assignment: Tags, resource IDs
- Backup vault: Encrypted storage location
- Vault lock: WORM (Write Once Read Many) compliance

**Backup Features**
- Point-in-time recovery (PITR)
- Incremental backups (except first)
- Encryption at rest (KMS)
- Cross-region copy
- Lifecycle to cold storage

### EBS Snapshots

**Snapshot Characteristics**
- Incremental backups
- Stored in S3 (hidden from your account)
- Regional resource (copy cross-region for DR)
- Can create AMI from snapshot
- Encryption inherited from volume

**Snapshot Lifecycle**
- Create manually or via Data Lifecycle Manager
- Copy across regions
- Share with other accounts
- Archive tier for cost savings (75% cost reduction)
- Fast snapshot restore (FSR) for quick recovery

**Data Lifecycle Manager (DLM)**
- Automated snapshot creation and deletion
- Tag-based policies
- Schedule: Hourly, daily, weekly, monthly
- Retention rules
- Cross-account snapshot sharing

### RDS Backups

**Automated Backups**
- Daily full backup during backup window
- Transaction logs every 5 minutes
- Point-in-time recovery (PITR): Any time up to 5 minutes ago
- Retention: 0-35 days (0 = disabled)
- Deleted when DB instance deleted (optional retain)

**Manual Snapshots**
- User-initiated
- Never expire automatically
- Retained even after DB instance deletion
- Can share with other accounts
- Can copy to other regions

**Backup Best Practices**
- Enable automated backups
- Set appropriate retention period
- Test restore procedures
- Use Multi-AZ for production
- Take manual snapshot before major changes

### DynamoDB Backups

**On-Demand Backups**
- Full table backup
- No performance impact
- Retained until explicitly deleted
- Same region as source table
- Restore to new table only

**Point-in-Time Recovery (PITR)**
- Continuous backups for 35 days
- Restore to any second in recovery window
- No performance impact
- Must be explicitly enabled
- Minimal RPO (seconds)

**Global Tables**
- Multi-region, multi-active replication
- <1 second replication
- Automatic conflict resolution (last write wins)
- Disaster recovery and high availability

### S3 Backup Strategies

**Versioning**
- Preserve all versions of objects
- Protection against accidental deletion
- Recover previous versions
- Additional storage cost
- MFA delete for added protection

**S3 Lifecycle Policies**
- Transition to cheaper storage classes
- Expire old versions
- Delete incomplete multipart uploads
- Tag-based policies

**S3 Replication**
- Same-region replication (SRR): Compliance, log aggregation
- Cross-region replication (CRR): Disaster recovery, latency
- Requires versioning
- Optional delete marker replication
- Replication metrics and notifications

## Disaster Recovery Strategies

### DR Objectives

**RTO (Recovery Time Objective)**
- Target time to restore operations
- How long can business tolerate downtime?
- Influences DR strategy choice
- Measured in seconds to days

**RPO (Recovery Point Objective)**
- Maximum acceptable data loss
- Point in time to which data must be recovered
- Determines backup frequency
- Measured in seconds to hours

### DR Strategies (Lowest to Highest Cost)

**1. Backup and Restore**
- **RTO**: Hours to days
- **RPO**: Hours
- **Cost**: Lowest
- Data backed up to S3/Glacier
- Restore when disaster occurs
- Good for non-critical workloads

**Implementation**
- Regular snapshots to S3
- CloudFormation templates for infrastructure
- Automated restore procedures
- Test restore regularly

**2. Pilot Light**
- **RTO**: 10s of minutes to hours
- **RPO**: Minutes
- **Cost**: Low to medium
- Minimal version always running (database)
- Scale up when needed
- Core infrastructure pre-deployed

**Implementation**
- RDS replica in DR region (off or small instance)
- Data replication (CRR, database replication)
- AMIs and CloudFormation templates ready
- Increase capacity during DR event

**3. Warm Standby**
- **RTO**: Minutes
- **RPO**: Seconds to minutes
- **Cost**: Medium to high
- Scaled-down but fully functional system
- Can handle some production load
- Quick scale-up capability

**Implementation**
- Smaller EC2 instance types
- Lower Auto Scaling minimums
- Read replicas for databases
- Route 53 health checks for failover
- Active/passive configuration

**4. Multi-Site Active-Active**
- **RTO**: Near zero
- **RPO**: Near zero
- **Cost**: Highest
- Full production environment in multiple regions
- Active traffic in both regions
- Maximum availability

**Implementation**
- DynamoDB Global Tables
- Aurora Global Database
- Route 53 weighted routing
- CloudFront for global delivery
- Bi-directional data replication

### Disaster Recovery Best Practices

**Planning**
1. Define RTO and RPO requirements
2. Choose appropriate DR strategy
3. Document DR procedures
4. Identify critical applications
5. Prioritize recovery order

**Testing**
1. Regular DR drills (quarterly minimum)
2. Test full recovery procedures
3. Measure actual RTO and RPO
4. Update documentation based on learnings
5. Train team on DR procedures

**Automation**
1. Automate failover procedures
2. Infrastructure as code for rebuilding
3. Automated data replication
4. Monitoring and alerting
5. Automated backup verification

**Data Protection**
1. Multiple backup copies
2. Cross-region replication
3. Versioning enabled
4. Encryption at rest and in transit
5. Backup retention aligned with compliance

## Best Practices

### High Availability Design

1. **Eliminate Single Points of Failure**
   - Multi-AZ deployments
   - Multiple instances behind load balancer
   - Database replicas

2. **Design for Failure**
   - Expect components to fail
   - Automated recovery
   - Graceful degradation

3. **Implement Health Checks**
   - ELB health checks
   - Route 53 health checks
   - Application-level health endpoints
   - Auto Scaling health checks

4. **Use Managed Services**
   - RDS instead of self-managed databases
   - DynamoDB for serverless NoSQL
   - S3 for durable object storage
   - Reduced operational burden

5. **Automate Everything**
   - Infrastructure as code
   - Automated deployments
   - Self-healing systems
   - Automated scaling

### Resilience Patterns

**Circuit Breaker**
- Prevent cascading failures
- Fail fast when downstream service unavailable
- Automatic retry with exponential backoff

**Bulkhead**
- Isolate critical resources
- Separate thread pools/connections
- Limit blast radius of failures

**Retry Logic**
- Exponential backoff
- Jitter to prevent thundering herd
- Maximum retry limits
- Idempotent operations

**Timeout**
- Set appropriate timeouts
- Fail fast on unresponsive services
- Prevent resource exhaustion

## Exam Tips

### Key Concepts to Remember

- Auto Scaling: Target tracking simplest, step scaling more flexible
- ALB: Layer 7 (HTTP/HTTPS), NLB: Layer 4 (TCP/UDP), ultra-high performance
- RDS Multi-AZ: Synchronous, automatic failover, same endpoint
- Aurora: 6 copies across 3 AZs, 15 read replicas, <30s failover
- Route 53 Failover: Active-passive DR, health checks required
- AWS Backup: Centralized, cross-service, lifecycle policies
- EBS Snapshots: Incremental, S3-stored, cross-region copy
- RDS Backups: Automated (0-35 days), manual (never expire)
- DynamoDB PITR: 35 days, restore to any second
- DR Strategies: Backup/Restore < Pilot Light < Warm Standby < Multi-Site

### Common Scenarios

**Scenario**: Web application needs auto-scaling based on CPU
**Solution**: ASG with target tracking policy (70% CPU), ALB health checks

**Scenario**: Database requires automatic failover
**Solution**: RDS Multi-AZ deployment, Aurora for faster failover

**Scenario**: Multi-region DR with <1 minute failover
**Solution**: Aurora Global Database, Route 53 health checks, warm standby

**Scenario**: Backup all EC2 instances daily, retain 7 days
**Solution**: AWS Backup with daily schedule, 7-day retention policy

**Scenario**: Distribute traffic globally with low latency
**Solution**: CloudFront + Route 53 latency-based routing + multi-region deployment

### Troubleshooting

**Issue**: Auto Scaling not working
- Check scaling policies and CloudWatch alarms
- Verify health check configuration
- Check service quotas (maximum capacity)
- Review cooldown periods

**Issue**: ELB health checks failing
- Verify security group allows health check port
- Check application health endpoint
- Increase timeout or interval
- Review success codes configuration

**Issue**: RDS failover taking too long
- Ensure Multi-AZ enabled
- Check for long-running transactions
- Verify application connection timeout
- Review DB instance size (larger = faster failover)

**Issue**: Backup restore fails
- Verify IAM permissions
- Check encryption key access
- Ensure target region supports service
- Review resource quotas in target region
