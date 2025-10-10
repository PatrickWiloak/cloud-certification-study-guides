# Azure Monitoring and Backup - AZ-104

## Azure Monitor

### Components
- **Metrics**: Numerical time-series data
- **Logs**: Text-based records
- **Alerts**: Notifications based on conditions
- **Dashboards**: Visual representation
- **Workbooks**: Interactive reports

### Metrics
- Collected automatically for Azure resources
- Near real-time (1-minute granularity)
- Retention: 93 days
- Metric Explorer for visualization
- No configuration needed

### Log Analytics
- Centralized log repository
- KQL (Kusto Query Language) for analysis
- Workspace: Container for logs
- Retention: 30-730 days (configurable)

**Data Sources**:
- VMs (agents required)
- Application Insights
- Diagnostic settings
- Azure Activity Log
- Custom logs

**Agents**:
- **Azure Monitor Agent (AMA)**: New, recommended
- **Log Analytics Agent**: Legacy (deprecated 2024)
- **Dependency Agent**: Service Map data

### Application Insights
- APM for web applications
- Auto-instrumentation or SDK
- Request/response times
- Dependency tracking
- Exception logging
- Live metrics
- Availability tests

### Alerts
**Alert Types**:
- Metric alerts
- Log query alerts
- Activity log alerts
- Smart detection (Application Insights)

**Components**:
- Alert rule: Condition to evaluate
- Action group: What to do (email, SMS, webhook, Logic App, Azure Function)
- Alert processing rule: Override or suppress

## Azure Backup

### Recovery Services Vault
- Container for backups
- Regional resource
- Storage replication: LRS, GRS (default)
- Soft delete: 14-day retention
- Cross Region Restore (CRR)

### Backup Solutions

**Azure VM Backup**:
- Agent-less (snapshot-based)
- Application-consistent backups
- Restore: Entire VM, disks, individual files
- Instant restore (snapshot tier)
- Retention: Up to 9999 days

**Backup Policies**:
- Schedule: Daily or weekly
- Retention: Daily, weekly, monthly, yearly
- Instant restore tier: 1-5 days

**Azure Files Backup**:
- Snapshot-based
- Up to 200 snapshots per share
- Retention: Up to 10 years

**SQL Server in Azure VM**:
- Database-level backups
- Transaction log backups (every 15 minutes)
- Point-in-time restore
- Long-term retention

### Azure Site Recovery (ASR)
- Disaster recovery solution
- Replicate VMs to another region
- Orchestrated failover/failback
- RPO: <5 minutes (async replication)
- RTO: Minutes (depends on failover)
- Test failover without impact

**Supported Scenarios**:
- Azure VM to Azure (region to region)
- On-premises VMware/Hyper-V to Azure
- Physical servers to Azure

## Diagnostic Settings
- Export platform logs and metrics
- Destinations:
  - Log Analytics workspace
  - Storage account (archival)
  - Event Hub (streaming)
  - Partner solutions

## Azure Advisor
- Personalized recommendations
- Categories:
  - Reliability
  - Security
  - Performance
  - Cost
  - Operational Excellence

## Azure Service Health
- Personalized view of Azure health
- Service issues affecting your resources
- Planned maintenance notifications
- Health advisories
- Set up alerts for proactive monitoring

## Exam Tips

### Common Scenarios
- **VM performance monitoring**: Install Azure Monitor Agent, send to Log Analytics
- **Application performance**: Application Insights
- **Alert on high CPU**: Metric alert + action group
- **Backup VMs**: Recovery Services Vault + backup policy
- **Disaster recovery**: Azure Site Recovery
- **Central log repository**: Log Analytics workspace
- **Archive logs long-term**: Diagnostic settings to Storage account
- **Query logs**: KQL in Log Analytics
- **Recommendations**: Azure Advisor

### Key Points
- Metrics automatically collected, logs need configuration
- Recovery Services Vault for backup and ASR
- Soft delete protects against accidental deletion (14 days)
- Application Insights for APM
- KQL for log queries
- Action groups define notification methods
- Diagnostic settings to export logs/metrics
