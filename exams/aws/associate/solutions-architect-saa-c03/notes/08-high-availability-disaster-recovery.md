# High Availability and Disaster Recovery - SAA-C03

## High Availability Fundamentals

### Availability Zones (AZs)
- **Multiple AZs per Region**: Typically 3-6
- **Isolated**: Separate power, networking, facilities
- **Low Latency**: High-speed private links
- **Fault Isolation**: Failure in one AZ doesn't affect others
- **Best Practice**: Deploy across multiple AZs

### Regions
- **Geographic Areas**: 30+ AWS Regions globally
- **Multiple AZs**: Each region has >= 3 AZs
- **Data Residency**: Data stays within region unless explicitly replicated
- **Service Availability**: Not all services in all regions
- **Latency Optimization**: Choose region close to users

### Edge Locations
- CloudFront Points of Presence (400+)
- Lower latency for content delivery
- Not the same as Regions/AZs

## RTO and RPO

### Recovery Time Objective (RTO)
- Maximum acceptable downtime
- Time to restore service after disaster
- Measured in hours, minutes, or seconds
- Lower RTO = Higher cost

### Recovery Point Objective (RPO)
- Maximum acceptable data loss
- Point in time to which data must be recovered
- Measured in hours, minutes, or seconds
- Lower RPO = More frequent backups

**Example**:
- RTO = 4 hours: Service must be restored within 4 hours
- RPO = 1 hour: Can lose at most 1 hour of data

## Disaster Recovery Strategies

### 1. Backup and Restore (Lowest Cost, Highest RTO/RPO)
- **RPO**: Hours
- **RTO**: Hours to days
- **Method**: Regular backups to S3, restore when needed
- **Cost**: Very low (storage only)
- **Use Case**: Non-critical applications

**Implementation**:
- AWS Backup for automated backups
- EBS snapshots to S3
- RDS automated backups
- S3 versioning and lifecycle policies
- AWS Storage Gateway for on-premises

### 2. Pilot Light (Lower Cost, Medium RTO/RPO)
- **RPO**: Minutes to hours
- **RTO**: Hours
- **Method**: Minimal version always running, scale up during disaster
- **Core Services**: Database replicated, ready to scale
- **Cost**: Low (minimal resources running)
- **Use Case**: Business-critical applications

**Implementation**:
- RDS with cross-region read replica
- Data replication to secondary region
- AMIs and scripts ready for quick launch
- Keep application code deployed but scaled to zero
- Quickly provision resources when needed

### 3. Warm Standby (Medium Cost, Lower RTO/RPO)
- **RPO**: Minutes
- **RTO**: Minutes to hours
- **Method**: Scaled-down version always running
- **Infrastructure**: Fully functional but smaller scale
- **Cost**: Medium (always running)
- **Use Case**: Production workloads

**Implementation**:
- Scaled-down EC2 instances in secondary region
- RDS Multi-AZ or cross-region read replica
- Auto Scaling to full capacity during failover
- Route 53 health checks for automatic failover
- Continuous data replication

### 4. Multi-Site / Active-Active (Highest Cost, Lowest RTO/RPO)
- **RPO**: Near zero
- **RTO**: Near zero (automatic failover)
- **Method**: Full production environment in multiple sites
- **Traffic**: Active in all sites simultaneously
- **Cost**: Highest (duplicate infrastructure)
- **Use Case**: Mission-critical applications

**Implementation**:
- Full deployment in multiple regions
- Route 53 with latency or geolocation routing
- DynamoDB Global Tables
- Aurora Global Database
- S3 Cross-Region Replication
- CloudFront for global distribution

## Multi-AZ Deployments

### EC2 Multi-AZ
- Deploy instances in multiple AZs
- Use Auto Scaling Groups spanning AZs
- Attach to Multi-AZ load balancer
- EBS snapshots for backup

### RDS Multi-AZ
- **Synchronous Replication**: Primary to standby
- **Automatic Failover**: 1-2 minutes
- **Same Region**: Different AZs only
- **Single Endpoint**: DNS updated during failover
- **Use Case**: High availability, not disaster recovery

### ELB Multi-AZ
- Automatically distributes across enabled AZs
- Health checks detect failed instances
- Cross-zone load balancing for even distribution
- **ALB**: Enabled by default (free)
- **NLB**: Disabled by default (charged)

### Aurora Multi-AZ
- Storage automatically replicated across 3 AZs
- 6 copies of data (2 per AZ)
- Self-healing storage
- Up to 15 read replicas across AZs
- Automatic failover to replica

## Cross-Region Disaster Recovery

### S3 Cross-Region Replication (CRR)
- **Asynchronous Replication**: Objects to different region
- **Versioning Required**: Both source and destination
- **Replication Rule**: Specify what to replicate
- **Use Cases**: Compliance, lower latency, disaster recovery
- **Replication Time Control (RTC)**: 99.99% within 15 minutes (SLA)

### DynamoDB Global Tables
- **Multi-Region, Multi-Active**: Read and write to any region
- **Replication**: <1 second lag
- **Last Writer Wins**: Conflict resolution
- **Use Case**: Globally distributed applications

### Aurora Global Database
- **Primary Region**: Read and write
- **Secondary Regions** (up to 5): Read-only replicas
- **Replication Lag**: <1 second
- **RTO**: <1 minute for failover
- **Use Case**: Global applications, disaster recovery

### RDS Cross-Region Read Replicas
- Asynchronous replication
- Can be promoted to standalone
- Use for disaster recovery or read scaling
- Cross-region network charges apply

## Backup Strategies

### AWS Backup
- **Centralized Backup**: Across AWS services
- **Backup Plans**: Schedule, retention, lifecycle
- **Backup Vault**: Encrypted storage
- **Cross-Region Backup**: Automatic copy to another region
- **Cross-Account Backup**: Backup to different account
- **Supported Services**: EC2, EBS, RDS, DynamoDB, EFS, FSx, Storage Gateway

**Backup Plan Components**:
- **Schedule**: Cron or rate expression
- **Lifecycle**: Transition to cold storage, delete
- **Backup Vault**: Storage location
- **Resource Assignment**: Tags or ARNs

### EBS Snapshots
- **Incremental**: Only changed blocks
- **Stored in S3**: Cross-AZ automatically
- **Copy**: Across regions
- **Encryption**: Encrypted snapshots from encrypted volumes
- **Lifecycle Manager**: Automate creation and deletion

### RDS Snapshots
- **Automated Backups**: Daily full + transaction logs
- **Manual Snapshots**: User-initiated
- **Retention**: 1-35 days (automated), indefinite (manual)
- **Cross-Region Copy**: For disaster recovery
- **Restored**: Creates new RDS instance

## Auto Scaling for High Availability

### Auto Scaling Groups (ASG)
- **Desired Capacity**: Target number of instances
- **Minimum**: Never scale below
- **Maximum**: Never scale above
- **Health Checks**: EC2 status, ELB health
- **Multi-AZ**: Distribute instances evenly

**Scaling Policies**:
- **Target Tracking**: Maintain metric (CPU at 50%)
- **Step Scaling**: Add/remove based on metric breaches
- **Simple Scaling**: One adjustment at a time
- **Scheduled**: Based on known patterns
- **Predictive**: ML-based forecasting

### Application Auto Scaling
- DynamoDB, ECS, Aurora, AppStream, Lambda
- Same scaling policy types as ASG
- Use Case: Right-size and optimize costs

## Route 53 for High Availability

### Health Checks
- **Endpoint Monitoring**: HTTP, HTTPS, TCP
- **Calculated Health Checks**: Combine multiple checks (AND, OR, NOT)
- **CloudWatch Alarm**: Monitor CloudWatch metric
- **Interval**: 30 seconds (standard) or 10 seconds (fast)
- **Failover**: Remove unhealthy resources from DNS

### Failover Routing
- Active-passive failover
- Primary and secondary resources
- Health check on primary
- Automatic DNS update to secondary

### Multi-Value Answer Routing
- Return multiple IP addresses
- Health checks on each value
- Client-side load balancing
- Not a replacement for ELB

## Resilience Best Practices

### Design Patterns

**Retry with Exponential Backoff**:
- Retry failed requests
- Increasing delay between retries
- Implemented in AWS SDKs
- Use for transient failures

**Circuit Breaker**:
- Detect failures quickly
- Prevent cascading failures
- Fallback to default response
- Automatic recovery attempts

**Bulkhead**:
- Isolate critical resources
- Prevent resource exhaustion
- Limit concurrent requests
- Use separate thread pools

### Chaos Engineering
- Intentionally inject failures
- Test system resilience
- Validate recovery procedures
- AWS Fault Injection Simulator (FIS)

### Testing Strategies
1. **Failure Injection**: Simulate component failures
2. **Load Testing**: Stress test under peak load
3. **Chaos Monkey**: Random instance termination
4. **DR Drills**: Regular failover testing
5. **Game Days**: Simulated disaster scenarios

## Elasticity vs Scalability

### Scalability
- **Vertical (Scale Up)**: Larger instance type
- **Horizontal (Scale Out)**: More instances
- **Purpose**: Handle increased load

### Elasticity
- **Auto Scaling**: Dynamic adjustment
- **Purpose**: Match capacity to demand
- **Cost Optimization**: Pay only for what you need

### High Availability
- **Purpose**: Minimize downtime
- **Implementation**: Multi-AZ, redundancy
- **Disaster Recovery**: Part of HA strategy

## Exam Tips

### Common Scenarios
- **Multi-AZ for HA**: RDS Multi-AZ, ALB across AZs, ASG spanning AZs
- **Cross-Region for DR**: S3 CRR, Aurora Global Database, DynamoDB Global Tables
- **Lowest RTO/RPO**: Multi-site active-active
- **Cost-effective DR**: Backup and restore or pilot light
- **Automatic failover**: Route 53 health checks + failover routing
- **Data replication**: DynamoDB Streams, Aurora Global Database
- **Centralized backup**: AWS Backup
- **Scale based on demand**: Auto Scaling with CloudWatch

### Decision Framework

**Choose Multi-AZ when**:
- Need high availability within region
- Automatic failover required
- Cannot tolerate single AZ failure

**Choose Cross-Region when**:
- Disaster recovery requirement
- Compliance/data residency needs
- Global user base (low latency)
- Regional failover capability

**RTO/RPO Trade-offs**:
- Lower RTO/RPO = Higher cost
- Multi-site > Warm standby > Pilot light > Backup/restore
- Choose based on business requirements and budget

### Best Practices Checklist
- [ ] Deploy across multiple AZs
- [ ] Implement Auto Scaling
- [ ] Use health checks and automatic failover
- [ ] Regular backups with tested restore procedures
- [ ] Document and test DR plan
- [ ] Monitor and alert on failures
- [ ] Implement retry logic with exponential backoff
- [ ] Use managed services for automatic HA
- [ ] Practice chaos engineering
- [ ] Keep AMIs and scripts ready
