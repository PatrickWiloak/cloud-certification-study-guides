# SQL Server and Azure SQL Deployment and Management

## Table of Contents
1. [SQL Server Deployment Options](#sql-server-deployment-options)
2. [Azure SQL Database Deployment](#azure-sql-database-deployment)
3. [Azure SQL Managed Instance](#azure-sql-managed-instance)
4. [SQL Server on Azure VMs](#sql-server-on-azure-vms)
5. [High Availability and Disaster Recovery](#high-availability-and-disaster-recovery)
6. [Security Implementation](#security-implementation)
7. [Backup and Restore Strategies](#backup-and-restore-strategies)
8. [Migration Strategies](#migration-strategies)
9. [Exam Tips](#exam-tips)

---

## SQL Server Deployment Options

### Comparison of Azure SQL Deployment Options

#### Azure SQL Database
**Type:** Platform as a Service (PaaS)

**Characteristics:**
- Fully managed
- Database-level deployment
- Latest SQL Server features
- Automatic backups and patching
- Built-in high availability (99.99% SLA)
- Serverless and Hyperscale options

**Best For:**
- Modern cloud applications
- SaaS applications
- Applications needing elastic scale
- Minimal administration

#### Azure SQL Managed Instance
**Type:** Platform as a Service (PaaS)

**Characteristics:**
- Near 100% SQL Server compatibility
- Instance-level features
- VNet integration
- SQL Agent support
- Cross-database queries
- Linked servers

**Best For:**
- Lift-and-shift migrations
- Applications using instance-level features
- Multi-database applications
- Existing SQL Server workloads

#### SQL Server on Azure VMs
**Type:** Infrastructure as a Service (IaaS)

**Characteristics:**
- Full SQL Server control
- OS-level access
- Custom configurations
- Legacy version support
- SSIS, SSRS, SSAS support

**Best For:**
- Custom configurations required
- OS-level dependencies
- Specific SQL Server versions
- Complete control needed

### Decision Matrix

| Requirement | SQL Database | Managed Instance | SQL on VM |
|-------------|--------------|------------------|-----------|
| Minimal administration | Yes | Moderate | No |
| PaaS benefits | Yes | Yes | No |
| Full SQL Server compatibility | Partial | Near 100% | 100% |
| Instance-level features | No | Yes | Yes |
| OS access | No | No | Yes |
| Cost (relative) | Low | Medium | Variable |
| SQL Agent | Limited | Yes | Yes |
| Cross-database queries | Elastic query | Yes | Yes |
| Linked servers | No | Yes | Yes |

---

## Azure SQL Database Deployment

### Service Tiers

#### DTU-Based Model
**Basic:**
- 5 DTUs
- 2 GB max size
- Development and test
- Non-critical workloads

**Standard (S0-S12):**
- 10-3000 DTUs
- Up to 1 TB
- Production workloads
- Predictable performance

**Premium (P1-P15):**
- 125-4000 DTUs
- Up to 4 TB
- Business-critical workloads
- Lowest latency
- Read replicas

#### vCore-Based Model

**General Purpose:**
- Budget-oriented
- Balanced compute and storage
- Remote storage (Azure Premium Storage)
- 5-10ms latency
- Up to 80 vCores
- Up to 4 TB (Hyperscale up to 100 TB)

**Business Critical:**
- Low-latency requirements
- Local SSD storage
- Built-in read replicas
- 1-2ms latency
- 99.995% SLA
- In-memory OLTP

**Hyperscale:**
- Rapid scale (up to 100 TB)
- Independent compute and storage scaling
- Fast backups and restores
- Multiple read replicas
- Optimal for large databases

### Serverless Compute Tier
**Features:**
- Auto-pause during inactive periods
- Auto-resume on connection
- Per-second billing
- Adjustable compute range

**Best For:**
- Intermittent, unpredictable usage
- Development and test
- New applications
- Cost optimization

**Configuration:**
- Minimum vCores (0.5-40)
- Maximum vCores (0.5-40)
- Auto-pause delay (60 min - 7 days)

### Deployment Methods

#### Azure Portal
1. Create SQL Database resource
2. Select server (existing or new)
3. Choose pricing tier
4. Configure networking
5. Set additional settings
6. Review and create

#### Azure CLI
```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create logical server
az sql server create \
    --name myserver \
    --resource-group myResourceGroup \
    --location eastus \
    --admin-user sqladmin \
    --admin-password <password>

# Create database
az sql db create \
    --resource-group myResourceGroup \
    --server myserver \
    --name myDatabase \
    --service-objective S0
```

#### PowerShell
```powershell
# Create resource group
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"

# Create server
New-AzSqlServer `
    -ResourceGroupName "myResourceGroup" `
    -ServerName "myserver" `
    -Location "EastUS" `
    -SqlAdministratorCredentials (Get-Credential)

# Create database
New-AzSqlDatabase `
    -ResourceGroupName "myResourceGroup" `
    -ServerName "myserver" `
    -DatabaseName "myDatabase" `
    -RequestedServiceObjectiveName "S0"
```

#### ARM Templates
```json
{
  "type": "Microsoft.Sql/servers/databases",
  "apiVersion": "2021-02-01-preview",
  "name": "[concat(parameters('serverName'), '/', parameters('databaseName'))]",
  "location": "[parameters('location')]",
  "sku": {
    "name": "S0",
    "tier": "Standard"
  },
  "properties": {
    "collation": "SQL_Latin1_General_CP1_CI_AS",
    "maxSizeBytes": 268435456000
  }
}
```

### Elastic Pools
Share resources among multiple databases.

**Benefits:**
- Cost optimization
- Automatic resource allocation
- Performance isolation
- Simplified management

**When to Use:**
- Multiple databases with varying usage
- Unpredictable resource demands
- SaaS applications
- Multi-tenant scenarios

**Sizing:**
- Based on aggregate DTU/vCore needs
- Monitor pool utilization
- Add/remove databases dynamically

**Configuration:**
- Pool size (DTU/vCore)
- Per-database min/max limits
- Storage limits

---

## Azure SQL Managed Instance

### Key Features

#### Instance-Level Capabilities
- SQL Server Agent
- Service Broker
- Database Mail
- Cross-database queries
- Linked servers
- CLR (Common Language Runtime)
- Distributed transactions (preview)
- Change Data Capture (CDC)

#### Network Isolation
- Deployed in VNet
- Private IP addresses
- On-premises connectivity (VPN/ExpressRoute)
- Private endpoints
- Service endpoints

### Service Tiers

#### General Purpose
- Remote storage
- Standard availability (99.99%)
- 4-80 vCores
- Up to 8 TB storage
- Cost-effective

#### Business Critical
- Local SSD storage
- High availability (99.99%)
- Built-in read replicas
- 4-80 vCores
- Up to 4 TB storage
- Zone redundancy

### Deployment Process

#### Prerequisites
1. VNet with dedicated subnet
2. Route table configuration
3. Network Security Group (NSG)
4. Sufficient IP addresses (/27 or larger)

#### Deployment Steps
1. Prepare VNet and subnet
2. Create Managed Instance (4-6 hours initial)
3. Configure connectivity
4. Restore databases or migrate

#### PowerShell Example
```powershell
# Create Managed Instance
New-AzSqlInstance `
    -Name "myManagedInstance" `
    -ResourceGroupName "myResourceGroup" `
    -Location "EastUS" `
    -SubnetId "/subscriptions/.../subnets/ManagedInstance" `
    -AdministratorCredential (Get-Credential) `
    -StorageSizeInGB 256 `
    -VCore 8 `
    -Edition "GeneralPurpose" `
    -ComputeGeneration "Gen5"
```

### Instance Pools
Deploy multiple small instances cost-effectively.

**Benefits:**
- Lower compute costs
- Faster deployment
- Shared infrastructure
- Ideal for small instances

**Requirements:**
- General Purpose tier only
- Minimum 8 vCores per pool
- Same VNet and subnet

### Connectivity Options

#### Public Endpoint
- Internet-accessible
- 3342 port
- TLS encryption
- Must be explicitly enabled

#### Private Endpoint
- VNet connectivity
- No public IP
- Azure Private Link
- On-premises access via VPN/ExpressRoute

---

## SQL Server on Azure VMs

### VM Images
**Marketplace Images:**
- Pre-configured SQL Server
- Various SQL Server versions
- Optimized settings
- Pay-as-you-go or BYOL (Bring Your Own License)

**Versions Available:**
- SQL Server 2012-2022
- Developer, Express, Web, Standard, Enterprise
- Windows or Linux OS

### Storage Configuration

#### Best Practices
1. **Premium SSDs** for data and log files
2. **Separate disks** for data, logs, tempdb
3. **Storage Spaces** for increased throughput
4. **Read caching** for data files
5. **No caching** for log files

#### Disk Types
- **Premium SSD** - Production workloads
- **Standard SSD** - Dev/test
- **Ultra Disk** - Mission-critical, low-latency

#### Example Configuration
- OS Disk: Premium SSD
- Data Files: Premium SSD with read caching
- Log Files: Premium SSD, no caching
- TempDB: Local SSD (D: drive) or separate disk

### VM Sizing

#### General Purpose (D-series)
- Balanced CPU-to-memory
- Small to medium databases
- Development and testing

#### Memory Optimized (E-series, M-series)
- High memory-to-CPU ratio
- Large databases
- In-memory workloads
- OLTP and DW

#### Storage Optimized (L-series)
- High disk throughput
- Large data warehouses
- Big data applications

### High Availability Options

#### Always On Availability Groups
- Multi-replica synchronization
- Automatic failover
- Read-only replicas
- Up to 9 replicas

**Requirements:**
- Windows Server Failover Cluster (WSFC)
- Domain-joined VMs
- Shared nothing architecture

#### Always On Failover Cluster Instances (FCI)
- Shared storage
- Automatic instance failover
- Single database copy

**Requirements:**
- WSFC
- Shared storage (S2D, Azure Shared Disks, SIOS)

#### Azure Site Recovery
- Disaster recovery solution
- Replicate VMs to secondary region
- Automated failover and failback

### Performance Optimization

#### SQL Server Settings
- Max server memory
- Cost threshold for parallelism
- Max degree of parallelism
- Optimize for ad hoc workloads
- Database file settings

#### VM Settings
- Enable accelerated networking
- Adjust power plan (High Performance)
- Configure storage spaces
- Enable lock pages in memory
- Instant file initialization

### Licensing

#### Pay-As-You-Go
- SQL Server included in hourly rate
- No upfront costs
- Flexible scaling

#### Azure Hybrid Benefit (AHBB)
- Use existing SQL Server licenses
- Software Assurance required
- Up to 55% savings
- Switch between licensing models

#### License Mobility
- Bring licenses to Azure
- Software Assurance required
- Migrate between on-premises and Azure

---

## High Availability and Disaster Recovery

### High Availability (HA)

#### Active Geo-Replication (SQL Database)
**Features:**
- Up to 4 readable secondaries
- Any Azure region
- Asynchronous replication
- Manual failover
- Independent scaling

**Use Cases:**
- Disaster recovery
- Read scale-out
- Application relocation
- Regional outage protection

#### Auto-Failover Groups
**Features:**
- Automatic failover
- Group of databases
- Read-write and read-only listeners
- Failover policy configuration

**Configuration:**
- Primary server/managed instance
- Secondary server/managed instance (different region)
- Databases to replicate
- Failover policy (automatic or manual)

#### Zone Redundancy
**Features:**
- Multiple availability zones
- Synchronous replication
- Automatic failover
- No additional cost for Premium/Business Critical

**Availability:**
- SQL Database (Premium, Business Critical)
- Managed Instance (Business Critical)

### Disaster Recovery (DR)

#### Backup-Based Recovery
**Capabilities:**
- Point-in-time restore (PITR)
- Geo-restore
- Long-term retention (LTR)

**Recovery Objectives:**
- RPO (Recovery Point Objective): 5-10 minutes
- RTO (Recovery Time Objective): Varies by size

#### Active-Passive DR
- Primary region (active)
- Secondary region (standby)
- Geo-replication or backup
- Manual or automatic failover

#### Active-Active DR
- Multiple active regions
- Load balancing
- Complex configuration
- Higher cost

### Backup Strategies

#### Automated Backups (SQL Database/Managed Instance)
**Full Backups:** Weekly
**Differential Backups:** Every 12-24 hours
**Transaction Log Backups:** Every 5-10 minutes

**Retention:**
- Default: 7 days
- Configurable: 1-35 days
- Long-term: Up to 10 years

#### Manual Backups (SQL on VMs)
**Types:**
- Full backup
- Differential backup
- Transaction log backup

**Azure Backup Integration:**
- Automated scheduling
- 15-minute RPO
- Streaming to Azure Storage
- Long-term retention

### Recovery Scenarios

#### Point-in-Time Restore (PITR)
```sql
-- Azure Portal, CLI, or PowerShell
-- Restore to any point within retention period
```

**Use Cases:**
- User error (deleted data)
- Application error
- Testing and development

#### Geo-Restore
```powershell
# Restore from geo-redundant backup
Restore-AzSqlDatabase `
    -FromGeoBackup `
    -ResourceGroupName "TargetRG" `
    -ServerName "targetserver" `
    -TargetDatabaseName "RestoredDB" `
    -ResourceId "/subscriptions/.../backups/..."
```

**Use Cases:**
- Regional outage
- Disaster recovery
- Database migration

#### Long-Term Retention (LTR)
**Configuration:**
- Weekly backups: Up to 5 years
- Monthly backups: Up to 10 years
- Yearly backups: Up to 10 years

**Restore:**
```powershell
Get-AzSqlDatabaseLongTermRetentionBackup `
    -Location "EastUS" `
    -ServerName "myserver" `
    -DatabaseName "mydb"
```

---

## Security Implementation

### Authentication

#### SQL Authentication
- SQL Server logins
- Username and password
- Simple to implement

**Best Practices:**
- Strong passwords
- Limit login attempts
- Regular password rotation

#### Azure Active Directory (Azure AD)
- Cloud-based identity
- MFA support
- Managed identities
- Single sign-on

**Benefits:**
- Centralized management
- Enhanced security
- Integration with Azure services

**Setup:**
```sql
-- Set Azure AD admin
-- Azure Portal or PowerShell
-- Then connect using Azure AD authentication
```

#### Managed Identity
- System-assigned or user-assigned
- No credentials in code
- Automatic rotation
- Azure services only

### Authorization

#### Role-Based Access Control (RBAC)
**Control Plane (Azure):**
- SQL Server Contributor
- SQL DB Contributor
- SQL Managed Instance Contributor
- SQL Security Manager

**Data Plane (SQL):**
- Database roles
- Server roles (Managed Instance/VM)
- Custom roles

#### Row-Level Security (RLS)
Filter rows based on user context.

```sql
-- Create security policy
CREATE SECURITY POLICY SalesFilter
ADD FILTER PREDICATE dbo.fn_securitypredicate(SalesPersonID)
ON dbo.Sales
WITH (STATE = ON);

-- Create predicate function
CREATE FUNCTION dbo.fn_securitypredicate(@SalesPersonID int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS fn_securitypredicate_result
WHERE @SalesPersonID = USER_NAME() OR USER_NAME() = 'Manager';
```

#### Dynamic Data Masking (DDM)
Obfuscate sensitive data.

```sql
-- Apply masking
ALTER TABLE Customers
ALTER COLUMN Phone ADD MASKED WITH (FUNCTION = 'partial(1,"XXX-XXX-",4)');

ALTER TABLE Customers
ALTER COLUMN Email ADD MASKED WITH (FUNCTION = 'email()');

-- Grant unmask permission
GRANT UNMASK TO [User];
```

### Encryption

#### Transparent Data Encryption (TDE)
- Encryption at rest
- Automatic in Azure SQL
- Service-managed or customer-managed keys
- No application changes

**Enable TDE (if disabled):**
```sql
-- Typically enabled by default in Azure SQL
ALTER DATABASE [DatabaseName]
SET ENCRYPTION ON;
```

#### Always Encrypted
- Encryption in use
- Client-side encryption
- Protect sensitive columns
- No access to encrypted data on server

**Types:**
- **Deterministic** - Same plaintext → same ciphertext (equality operations)
- **Randomized** - Different ciphertext each time (highest security)

```sql
CREATE COLUMN MASTER KEY MyCMK
WITH (
    KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
    KEY_PATH = 'https://myvault.vault.azure.net/keys/mykey'
);

CREATE COLUMN ENCRYPTION KEY MyCEK
WITH VALUES (
    COLUMN_MASTER_KEY = MyCMK,
    ALGORITHM = 'RSA_OAEP'
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    SSN CHAR(11) ENCRYPTED WITH (
        COLUMN_ENCRYPTION_KEY = MyCEK,
        ENCRYPTION_TYPE = Randomized,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
    )
);
```

#### Transport Layer Security (TLS)
- Encrypt data in transit
- Enforced by default
- Minimum TLS 1.2

**Enforce encryption:**
```sql
-- Connection string
Server=myserver.database.windows.net;
Database=mydb;
Encrypt=True;
TrustServerCertificate=False;
```

### Advanced Threat Protection (ATP)

#### Features
- Vulnerability assessment
- Threat detection
- Anomalous activities
- SQL injection attempts
- Unusual access patterns

#### Vulnerability Assessment
- Scan for security issues
- Best practice rules
- Baseline configuration
- Remediation recommendations

**Schedule:**
- Recurring scans
- Email notifications
- Comparison to baseline

#### Threat Detection
**Alerts for:**
- SQL injection
- Potential data infiltration
- Brute force attacks
- Anomalous client locations
- Potentially harmful applications

### Network Security

#### Firewall Rules
**Server-Level:**
```sql
-- Add IP rule
EXECUTE sp_set_firewall_rule
    @name = N'AllowMyIP',
    @start_ip_address = '203.0.113.10',
    @end_ip_address = '203.0.113.10';
```

**Database-Level (SQL Database):**
```sql
EXECUTE sp_set_database_firewall_rule
    @name = N'AllowSpecificDB',
    @start_ip_address = '203.0.113.0',
    @end_ip_address = '203.0.113.255';
```

#### Virtual Network Rules
- Allow specific VNets/subnets
- Service endpoints
- Enhanced security

#### Private Endpoint
- Private IP in your VNet
- No public endpoint needed
- Azure Private Link
- On-premises access via VPN/ExpressRoute

**Benefits:**
- Network isolation
- No internet exposure
- Eliminate data exfiltration risks

---

## Backup and Restore Strategies

### Automated Backups

#### SQL Database and Managed Instance
**Backup Schedule:**
- Full: Weekly
- Differential: 12-24 hours
- Log: 5-10 minutes

**Retention:**
- Short-term: 1-35 days (configurable)
- Long-term: Up to 10 years

**Storage:**
- Geo-redundant by default
- Configurable: LRS, ZRS, GRS

#### Configure Backup Retention
```powershell
# Set short-term retention
Set-AzSqlDatabaseBackupShortTermRetentionPolicy `
    -ResourceGroupName "myRG" `
    -ServerName "myserver" `
    -DatabaseName "mydb" `
    -RetentionDays 14

# Set long-term retention
Set-AzSqlDatabaseBackupLongTermRetentionPolicy `
    -ResourceGroupName "myRG" `
    -ServerName "myserver" `
    -DatabaseName "mydb" `
    -WeeklyRetention "P4W" `
    -MonthlyRetention "P12M" `
    -YearlyRetention "P5Y" `
    -WeekOfYear 1
```

### Manual Backups (SQL on VMs)

#### Azure Backup for SQL Server
**Features:**
- 15-minute RPO
- Up to 99 years retention
- No backup infrastructure
- Automatic backup scheduling

**Setup:**
1. Create Recovery Services vault
2. Install backup extension on VM
3. Discover SQL Server instances
4. Configure backup policy
5. Enable backup

#### Native SQL Server Backups
```sql
-- Full backup to Azure Storage
BACKUP DATABASE [MyDB]
TO URL = 'https://mystorageaccount.blob.core.windows.net/backups/MyDB.bak'
WITH CREDENTIAL = 'MyCredential',
     COMPRESSION,
     STATS = 10;

-- Differential backup
BACKUP DATABASE [MyDB]
TO URL = 'https://mystorageaccount.blob.core.windows.net/backups/MyDB_diff.bak'
WITH DIFFERENTIAL,
     CREDENTIAL = 'MyCredential',
     COMPRESSION;

-- Transaction log backup
BACKUP LOG [MyDB]
TO URL = 'https://mystorageaccount.blob.core.windows.net/backups/MyDB_log.trn'
WITH CREDENTIAL = 'MyCredential',
     COMPRESSION;
```

### Restore Operations

#### Point-in-Time Restore (SQL Database)
```powershell
Restore-AzSqlDatabase `
    -FromPointInTimeBackup `
    -PointInTime (Get-Date).AddHours(-2) `
    -ResourceGroupName "myRG" `
    -ServerName "myserver" `
    -TargetDatabaseName "MyDB_Restored" `
    -ResourceId "/subscriptions/.../databases/MyDB"
```

#### Copy-Only Backups
```sql
-- Won't disrupt backup chain
BACKUP DATABASE [MyDB]
TO DISK = 'C:\Backups\MyDB_CopyOnly.bak'
WITH COPY_ONLY, COMPRESSION;
```

### Backup Best Practices

1. **Test Restores Regularly**
   - Verify backup integrity
   - Practice recovery procedures
   - Validate RTOs

2. **Monitor Backup Success**
   - Check backup history
   - Set up alerts
   - Review error logs

3. **Secure Backup Data**
   - Encrypt backups
   - Control access
   - Geo-redundant storage for DR

4. **Document Procedures**
   - Recovery runbooks
   - Contact lists
   - Escalation paths

5. **Consider Retention Requirements**
   - Legal/compliance needs
   - Business requirements
   - Cost implications

---

## Migration Strategies

### Assessment

#### Azure Migrate
- Discover on-premises SQL Servers
- Assess readiness
- Right-size recommendations
- Cost estimates

#### Data Migration Assistant (DMA)
- Compatibility assessment
- Feature parity check
- Performance recommendations
- Migration blocking issues

**Run Assessment:**
1. Install DMA
2. Create new assessment project
3. Connect to source SQL Server
4. Select target Azure SQL platform
5. Review recommendations

### Migration Methods

#### Online Migration (Minimal Downtime)

**Azure Database Migration Service (DMS):**
- Continuous data sync
- Minimal downtime
- Automated cutover option

**Process:**
1. Create DMS instance
2. Create migration project
3. Configure source and target
4. Run full data migration
5. Continuous sync
6. Cutover when ready

**Supported Scenarios:**
- SQL Server → SQL Database
- SQL Server → Managed Instance
- SQL Server → SQL on VM

#### Offline Migration

**Backup/Restore:**
- Native SQL Server backups
- Restore to Azure SQL
- Downtime during migration

**Steps:**
1. Full backup of source
2. Copy backup to Azure Storage
3. Restore to target
4. Apply differential/log backups
5. Cutover

**BACPAC Export/Import:**
- Logical export of database
- Import to Azure SQL Database

```powershell
# Export
New-AzSqlDatabaseExport `
    -ResourceGroupName "myRG" `
    -ServerName "myserver" `
    -DatabaseName "mydb" `
    -StorageKeyType "StorageAccessKey" `
    -StorageKey $storageKey `
    -StorageUri "https://mystorageaccount.blob.core.windows.net/bacpac/mydb.bacpac" `
    -AdministratorLogin $adminLogin `
    -AdministratorLoginPassword $adminPassword

# Import
New-AzSqlDatabaseImport `
    -ResourceGroupName "myRG" `
    -ServerName "targetserver" `
    -DatabaseName "mydb" `
    -StorageKeyType "StorageAccessKey" `
    -StorageKey $storageKey `
    -StorageUri "https://mystorageaccount.blob.core.windows.net/bacpac/mydb.bacpac" `
    -AdministratorLogin $adminLogin `
    -AdministratorLoginPassword $adminPassword `
    -Edition "Standard" `
    -ServiceObjectiveName "S0"
```

#### Transactional Replication
- Continuous data sync
- Publisher (source) → Subscriber (Azure)
- One-way replication
- Minimal downtime

**Supported:**
- To SQL Database (as subscriber)
- To Managed Instance
- To SQL on VM

### Post-Migration

#### Validation
- Verify data integrity
- Test application connectivity
- Performance testing
- User acceptance testing

#### Optimization
- Update statistics
- Rebuild indexes
- Configure query store
- Adjust compatibility level
- Monitor performance

#### Cutover Checklist
- [ ] Verify all data migrated
- [ ] Test application connectivity
- [ ] Update connection strings
- [ ] Configure security
- [ ] Set up monitoring
- [ ] Update DNS/networking
- [ ] Communication plan
- [ ] Rollback plan ready

---

## Exam Tips

### Key Concepts

**Deployment Options:**
- SQL Database: PaaS, database-level, modern apps
- Managed Instance: PaaS, instance-level, lift-and-shift
- SQL on VM: IaaS, full control, custom configurations

**High Availability:**
- Auto-failover groups: Automatic, multiple databases
- Active geo-replication: Manual, up to 4 secondaries
- Zone redundancy: Availability zone protection
- Always On (VMs): Multi-replica synchronization

**Security:**
- TDE: Encryption at rest (automatic)
- Always Encrypted: Encryption in use (client-side)
- RLS: Row-level filtering
- DDM: Data masking
- ATP: Threat detection and vulnerability assessment

**Backup:**
- Automated: 7-35 days retention
- LTR: Up to 10 years
- PITR: Point-in-time restore within retention
- Geo-restore: Restore from geo-redundant backup

### Common Scenarios

**High Availability Requirement:**
- Use auto-failover groups for automatic failover
- Zone redundancy for datacenter failures
- Active geo-replication for read scale-out

**Minimal Administration:**
- Choose SQL Database or Managed Instance
- Automated backups and patching
- Built-in high availability

**Lift-and-Shift Migration:**
- Managed Instance for instance-level features
- SQL on VM for complete control
- DMS for minimal downtime migration

**Cost Optimization:**
- Serverless for intermittent usage
- Elastic pools for multiple databases
- Azure Hybrid Benefit for license savings
- Reserved capacity for predictable workloads

### Important Limits

| Service | Max Database Size | Max vCores | Backup Retention |
|---------|------------------|------------|------------------|
| SQL Database (vCore) | 4 TB (100 TB Hyperscale) | 80 | 1-35 days, LTR up to 10 years |
| Managed Instance | 8 TB (GP), 4 TB (BC) | 80 | 1-35 days, LTR up to 10 years |
| SQL on VM | OS limits | VM limits | Manual configuration |

### Quick Reference

**Choose SQL Database when:**
- Modern cloud-native application
- Need elastic scale
- Minimal administration
- Database-level deployment

**Choose Managed Instance when:**
- Instance-level features required
- Lift-and-shift from SQL Server
- Cross-database queries needed
- SQL Agent jobs required

**Choose SQL on VM when:**
- Full SQL Server control needed
- OS-level dependencies
- Specific SQL Server version required
- Custom configurations

**High Availability Solutions:**
- Auto-failover groups: Automatic, group-level
- Active geo-replication: Manual, database-level
- Zone redundancy: Automatic, same region
- Always On AG: VMs, multi-replica

**Security Layers:**
1. Network: Firewall, VNet rules, Private Endpoint
2. Access: Azure AD, RBAC, SQL authentication
3. Data: TDE, Always Encrypted, RLS, DDM
4. Monitoring: ATP, auditing, threat detection

### Study Focus

1. **Understand deployment options** and when to use each
2. **Know HA/DR configurations** and their differences
3. **Master security features** and implementation
4. **Understand backup strategies** and restore scenarios
5. **Know migration methods** and tools
6. **Practice Azure CLI/PowerShell** commands
7. **Understand performance tiers** and scaling options

### Final Checklist

- [ ] Deployment option decision matrix
- [ ] Service tier characteristics
- [ ] HA/DR configuration methods
- [ ] Security implementation layers
- [ ] Backup and restore procedures
- [ ] Migration assessment and execution
- [ ] Monitoring and maintenance tasks
- [ ] Cost optimization strategies
