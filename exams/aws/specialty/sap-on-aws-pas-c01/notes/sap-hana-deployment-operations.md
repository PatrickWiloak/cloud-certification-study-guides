# SAP HANA Deployment and Operations

## SAP HANA Deployment Methods

### SAP HANA Installation

**Installation Options**
1. **Manual Installation**
- SAP Software Provisioning Manager (SWPM)
- Command-line hdblcm tool
- Full control over configuration
- Time-intensive

2. **AWS Launch Wizard for SAP**
- Automated deployment
- Best practices built-in
- CloudFormation-based
- Quick start templates

3. **SAP CAL (Cloud Appliance Library)**
- Pre-configured solutions
- Demo and development systems
- Pay-as-you-go
- Rapid deployment

**Prerequisites**
```bash
# OS requirements (SUSE/RHEL)
- Supported OS version (check SAP PAM)
- Kernel parameters tuned for SAP HANA
- Storage configured per SAP guidelines
- Network configuration complete

# Storage layout
/hana/data    - 4x RAM (gp3, 16K IOPS min)
/hana/log     - 512 GB min (gp3, high IOPS)
/hana/shared  - 1x RAM
/usr/sap      - 50 GB
/backup       - Database size × 1-2
```

### AWS Launch Wizard Deployment

**Step-by-Step Process**
```
1. Launch Wizard Selection
   - Choose "SAP HANA database"
   - Select deployment type (single-node, HA, scale-out)

2. Application Settings
   - SAP System ID (SID): 3 characters
   - Instance Number: 00-99
   - Master password
   - HANA version

3. Infrastructure Configuration
   - VPC and subnets
   - Instance type (r5.24xlarge, etc.)
   - Key pair for SSH access
   - Availability Zones for HA

4. Storage Configuration
   - EBS volume types and sizes
   - IOPS and throughput settings
   - Encryption (KMS keys)

5. High Availability (if selected)
   - Secondary instance configuration
   - Overlay IP address
   - Pacemaker cluster setup

6. Review and Deploy
   - Estimated cost
   - CloudFormation stack creation
   - 30-60 minutes deployment time
```

**CloudFormation Template (Simplified)**
```yaml
Parameters:
  SAPSID:
    Type: String
    Default: HDB
    Description: SAP System ID

  HANAInstanceNumber:
    Type: String
    Default: "00"

  MasterPassword:
    Type: String
    NoEcho: true

Resources:
  HANAPrimary:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: r5.24xlarge
      ImageId: !Ref SUSEHANAImageId
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          # Download HANA installation
          aws s3 cp s3://${InstallationBucket}/HANA2.0/ /hana/install/ --recursive

          # Install HANA
          /hana/install/HDB_SERVER_LINUX_X86_64/hdblcm \
            --action=install \
            --sid=${SAPSID} \
            --number=${HANAInstanceNumber} \
            --password=${MasterPassword} \
            --system_user_password=${MasterPassword} \
            --sapadm_password=${MasterPassword} \
            --batch

  DataVolume:
    Type: AWS::EC2::Volume
    Properties:
      Size: 3072
      VolumeType: gp3
      Iops: 16000
      Throughput: 1000
      Encrypted: true
      AvailabilityZone: !GetAtt HANAPrimary.AvailabilityZone

  VolumeAttachment:
    Type: AWS::EC2::VolumeAttachment
    Properties:
      Device: /dev/sdb
      InstanceId: !Ref HANAPrimary
      VolumeId: !Ref DataVolume
```

## SAP HANA High Availability

### HANA System Replication Setup

**Configure Primary System**
```bash
# Enable system replication
su - <sid>adm
hdbnsutil -sr_enable --name=PRIMARY

# Check replication status
hdbnsutil -sr_state
```

**Configure Secondary System**
```bash
# Stop HANA on secondary
sapcontrol -nr <instance> -function StopSystem HDB

# Register as replication target
hdbnsutil -sr_register \
  --name=SECONDARY \
  --remoteHost=primary-host \
  --remoteInstance=00 \
  --replicationMode=sync \
  --operationMode=logreplay \
  --online

# Start HANA
sapcontrol -nr <instance> -function StartSystem HDB
```

**Replication Modes**
| Mode | Description | RPO | Use Case |
|------|-------------|-----|----------|
| SYNC | Fully synchronous | 0 | Same AZ, HA |
| SYNCMEM | Sync memory, async disk | 0 | Cross-AZ, HA |
| ASYNC | Asynchronous | >0 | Cross-region, DR |

**Monitor Replication**
```sql
-- Check replication status
SELECT * FROM M_SERVICE_REPLICATION;
SELECT * FROM SYS.M_SYSTEM_REPLICATION;

-- Check replication lag
SELECT SECONDARY_ACTIVE_STATUS, REPLICATION_STATUS_DETAILS 
FROM M_SERVICE_REPLICATION;

-- Shipping delay
SELECT SHIPPED_LOG_POSITION_TIME, LAST_LOG_POSITION_TIME
FROM M_SERVICE_REPLICATION;
```

### Pacemaker Cluster Configuration

**Install Cluster Software (SUSE)**
```bash
# On both nodes
zypper install -y -t pattern ha_sles
zypper install -y SAPHanaSR SAPHanaSR-doc

# Initialize cluster (primary node only)
ha-cluster-init \
  --name=hana_cluster \
  --interface=eth0 \
  --unicast

# Join cluster (secondary node)
ha-cluster-join -c <primary-ip>
```

**Configure SAPHanaSR**
```bash
# Set cluster properties
crm configure property stonith-enabled="true"
crm configure property stonith-action="off"
crm configure rsc_defaults resource-stickiness="1000"
crm configure rsc_defaults migration-threshold="5000"

# Create SAPHana resource
crm configure primitive rsc_SAPHana_HDB_HDB00 ocf:suse:SAPHana \
  op start interval="0" timeout="3600" \
  op stop interval="0" timeout="3600" \
  op promote interval="0" timeout="3600" \
  op monitor interval="60" role="Master" timeout="700" \
  op monitor interval="61" role="Slave" timeout="700" \
  params SID="HDB" InstanceNumber="00" \
  PREFER_SITE_TAKEOVER="true" \
  DUPLICATE_PRIMARY_TIMEOUT="7200" \
  AUTOMATED_REGISTER="true"

# Create virtual IP resource
crm configure primitive rsc_ip_HDB_HDB00 IPaddr2 \
  params ip="10.0.20.100" \
  meta target-role="Started"

# Create cluster colocation
crm configure colocation col_saphana_ip_HDB_HDB00 2000: \
  rsc_ip_HDB_HDB00:Started \
  msl_SAPHana_HDB_HDB00:Master

# Commit configuration
crm configure commit
```

**STONITH with AWS**
```bash
# Install AWS CLI and configure
pip3 install awscli

# Create STONITH resource
crm configure primitive stonith_hana_primary stonith:external/ec2 \
  params tag=pacemaker \
  profile=cluster \
  pcmk_reboot_timeout=300 \
  op monitor interval=120 timeout=60

crm configure clone fencing_hana_primary stonith_hana_primary
```

**Test Failover**
```bash
# Crash test on primary
echo c > /proc/sysrq-trigger

# Verify automatic takeover
crm status
hdbnsutil -sr_state

# Manual takeover
crm resource move msl_SAPHana_HDB_HDB00 <secondary-node>
```

## Backup and Recovery

### AWS Backint Agent

**Benefits**
- Direct backup to S3
- Compression and encryption
- Lifecycle policies
- Cost-effective

**Installation**
```bash
# Download AWS Backint Agent
aws s3 cp s3://awssap-backint-agent/aws-backint-agent-latest.tar.gz /tmp/

# Extract
cd /tmp
tar -xvzf aws-backint-agent-latest.tar.gz

# Install
cd aws-backint-agent
./install.sh -i <HANA-SID>

# Configuration file: /opt/aws/backint/hdbbackint.cfg
```

**Configuration File**
```ini
[aws]
aws_access_key_id = <from IAM role>
aws_secret_access_key = <from IAM role>
region = us-east-1

[backup]
bucket = sap-hana-backups
prefix = HDB/backups/
max_channels = 4
compression_level = 6
encryption = SSE-S3

[log]
level = INFO
file = /var/log/aws-backint.log
```

**Backup Commands**
```sql
-- Full backup
BACKUP DATA USING BACKINT ('COMPLETE_BACKUP');

-- Differential backup
BACKUP DATA DIFFERENTIAL USING BACKINT ('DIFF_BACKUP');

-- Incremental backup  
BACKUP DATA INCREMENTAL USING BACKINT ('INCR_BACKUP');

-- Log backup (automatic, but can be triggered)
BACKUP LOG USING BACKINT;

-- Check backup status
SELECT * FROM M_BACKUP_CATALOG ORDER BY ENTRY_ID DESC;
```

**Backup Schedule**
```bash
# Create backup script
cat > /usr/sap/HDB/backup/full_backup.sh << 'EOF'
#!/bin/bash
source /usr/sap/HDB/home/.sapenv.sh
hdbsql -u SYSTEM -p <password> -d SYSTEMDB \
  "BACKUP DATA USING BACKINT ('FULL_$(date +%Y%m%d_%H%M%S)')"
EOF

# Add to crontab
crontab -e -u <sid>adm
# Daily full backup at 2 AM
0 2 * * * /usr/sap/HDB/backup/full_backup.sh
```

### Recovery Procedures

**Point-in-Time Recovery**
```sql
-- Stop HANA
sapcontrol -nr 00 -function StopSystem HDB

-- Recover to specific time
RECOVER DATA USING BACKINT 
  UNTIL TIMESTAMP '2024-01-15 14:30:00' 
  CLEAR LOG 
  CHECK ACCESS USING FILE;

-- Start HANA
sapcontrol -nr 00 -function StartSystem HDB
```

**Recovery from S3 Backups**
```bash
# List available backups
aws s3 ls s3://sap-hana-backups/HDB/backups/

# Restore specific backup (if needed manually)
aws s3 cp s3://sap-hana-backups/HDB/backups/COMPLETE_BACKUP_databackup_0_1 \
  /hana/backup/data/

# HANA will use Backint automatically during recovery
hdbcons "recover data"
```

## Performance Monitoring

### HANA Monitoring Tools

**SAP HANA Studio**
- Graphical interface
- Real-time monitoring
- SQL console
- Administration tasks

**SAP HANA Cockpit**
- Web-based interface
- Multi-system monitoring
- Alerts and diagnostics
- Resource utilization

**SQL Queries for Monitoring**
```sql
-- CPU usage
SELECT HOST, USAGE_PCT 
FROM M_HOST_RESOURCE_UTILIZATION 
WHERE MEASURE_NAME = 'CPU';

-- Memory usage
SELECT HOST, USED_PHYSICAL_MEMORY, FREE_PHYSICAL_MEMORY, TOTAL_MEMORY
FROM M_HOST_RESOURCE_UTILIZATION;

-- Disk I/O
SELECT HOST, SUM(TOTAL_READS), SUM(TOTAL_WRITES)
FROM M_VOLUME_IO_TOTAL_STATISTICS
GROUP BY HOST;

-- Long-running queries
SELECT STATEMENT_STRING, EXECUTION_TIME, CPU_TIME
FROM M_ACTIVE_STATEMENTS
WHERE EXECUTION_TIME > 60000
ORDER BY EXECUTION_TIME DESC;

-- Table sizes
SELECT TABLE_NAME, DISK_SIZE, MEMORY_SIZE_IN_TOTAL
FROM M_TABLE_PERSISTENCE_STATISTICS
ORDER BY MEMORY_SIZE_IN_TOTAL DESC
LIMIT 20;
```

### CloudWatch Integration

**HANA CloudWatch Metrics**
```python
import boto3
import subprocess

cloudwatch = boto3.client('cloudwatch')

def publish_hana_metrics():
    # Get HANA metrics via SQL
    cpu_usage = get_hana_cpu()
    memory_usage = get_hana_memory()
    
    # Publish to CloudWatch
    cloudwatch.put_metric_data(
        Namespace='SAP/HANA',
        MetricData=[
            {
                'MetricName': 'CPUUtilization',
                'Value': cpu_usage,
                'Unit': 'Percent',
                'Dimensions': [
                    {'Name': 'SID', 'Value': 'HDB'},
                    {'Name': 'Environment', 'Value': 'Production'}
                ]
            },
            {
                'MetricName': 'MemoryUtilization',
                'Value': memory_usage,
                'Unit': 'Percent',
                'Dimensions': [
                    {'Name': 'SID', 'Value': 'HDB'},
                    {'Name': 'Environment', 'Value': 'Production'}
                ]
            }
        ]
    )

# Schedule with cron
# */5 * * * * python3 /scripts/publish_metrics.py
```

**CloudWatch Alarms**
```yaml
Resources:
  HANAHighCPUAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: HANA-High-CPU
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      Dimensions:
        - Name: InstanceId
          Value: !Ref HANAInstance
      AlarmActions:
        - !Ref SNSTopic

  HANAHighMemoryAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: HANA-High-Memory
      MetricName: MemoryUtilization
      Namespace: CWAgent
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 85
      ComparisonOperator: GreaterThanThreshold
```

## Patching and Maintenance

### HANA Patching Strategy

**Patch Types**
1. **Support Package Stack (SPS)**
- Major updates (annually)
- New features
- Requires downtime
- Test thoroughly

2. **Revision Updates**
- Security patches
- Bug fixes
- Monthly/quarterly
- Minimal downtime

3. **Hot fixes**
- Critical issues
- Applied as needed

**Patching Process**
```bash
# 1. Download patch from SAP Support Portal
# 2. Upload to S3
aws s3 cp SAP_HANA_DATABASE100_122_45_Linux_on_x86_64.SAR \
  s3://sap-software/patches/

# 3. Download to HANA server
aws s3 cp s3://sap-software/patches/SAP_HANA_DATABASE100_122_45_Linux_on_x86_64.SAR \
  /hana/patches/

# 4. Extract SAR file
SAPCAR -xvf SAP_HANA_DATABASE100_122_45_Linux_on_x86_64.SAR

# 5. Run hdblcm as sidadm
su - hdbadm
cd /hana/patches/SAP_HANA_DATABASE
./hdblcm --action=update --sid=HDB

# 6. Verify version
HDB version

# 7. For HA, repeat on secondary after replication sync
```

### Maintenance Windows

**Plan Maintenance**
```bash
# Create maintenance plan script
cat > /scripts/maintenance.sh << 'EOF'
#!/bin/bash
# Pre-maintenance checks
echo "Starting maintenance at $(date)"

# 1. Notify users (via email/slack)
/scripts/notify_users.sh "Maintenance starting in 30 minutes"

# 2. Stop all application servers
for server in $(cat /config/app_servers.txt); do
  ssh $server "su - <sid>adm -c 'sapcontrol -nr 00 -function Stop'"
done

# 3. Backup HANA
su - hdbadm -c "hdbsql -u SYSTEM -p <pwd> 'BACKUP DATA USING BACKINT'"

# 4. Perform maintenance (patch, reboot, etc.)
# ...

# 5. Start HANA
su - hdbadm -c "HDB start"

# 6. Start application servers
for server in $(cat /config/app_servers.txt); do
  ssh $server "su - <sid>adm -c 'sapcontrol -nr 00 -function Start'"
done

# 7. Verify systems
/scripts/health_check.sh

echo "Maintenance completed at $(date)"
EOF
```

## Migration to AWS

### SAP Migration Options

**1. Homogeneous Migration (Same OS/DB)**
- Backup/restore
- Storage replication
- Minimal changes

**2. Heterogeneous Migration (Different OS/DB)**
- SAP DMO (Database Migration Option)
- Combines migration and upgrade
- Zero downtime options

**3. SAP S/4HANA Conversion**
- Convert ECC to S/4HANA
- Use DMO with system conversion
- Data cleanup during migration

**Migration Tools**
- **SAP Software Update Manager (SUM/DMO)**
- **AWS Application Migration Service (MGN)**
- **CloudEndure Migration**
- **SAP LT Replication Server**

**Migration Steps**
```
1. Assessment
   - Current landscape documentation
   - Sizing for AWS
   - Cost analysis

2. Planning
   - Migration strategy selection
   - Cutover planning
   - Rollback procedures

3. Preparation
   - AWS account setup
   - VPC and networking
   - Install AWS tools
   - Create S3 buckets

4. Test Migration
   - Migrate to test environment
   - Validate functionality
   - Performance testing
   - User acceptance testing

5. Production Migration
   - Final sync
   - Cutover window
   - Production go-live
   - Hypercare support

6. Post-Migration
   - Decommission source
   - Optimize AWS resources
   - Documentation update
```

## Exam Tips

### HANA Deployment
- Launch Wizard: Fastest deployment method
- Manual: Full control, time-intensive
- Prerequisites: OS tuning, storage layout
- Installation: hdblcm or SWPM

### High Availability
- HSR modes: SYNC (HA), ASYNC (DR)
- Pacemaker: Automated failover
- Overlay IP: Virtual IP for failover
- Test failover regularly

### Backup & Recovery
- Backint Agent: Direct S3 backups
- Backup types: Full, differential, incremental, log
- Recovery: Point-in-time using Backint
- S3 lifecycle policies for cost optimization

### Operations
- Monitoring: HANA Studio, Cockpit, CloudWatch
- Patching: SPS, revisions, hot fixes
- Maintenance: Plan windows, backup first
- Migration: DMO for database change, backup/restore for homogeneous

### Common Scenarios
- **HA Setup**: SYNC HSR + Pacemaker + Overlay IP
- **DR Setup**: ASYNC HSR to different region
- **Backup**: Daily full to S3 with Backint
- **Performance**: Monitor CloudWatch, optimize queries
- **Patching**: Test on non-prod, patch secondary first in HA
