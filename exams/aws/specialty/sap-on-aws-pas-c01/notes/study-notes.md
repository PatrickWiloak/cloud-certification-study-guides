# SAP on AWS Specialty - Comprehensive Study Notes

## Quick Reference Guide

### SAP-Certified EC2 Instance Types

#### HANA Database Instances (Memory-Optimized)
```bash
# High Memory instances for SAP HANA
X2idn: Up to 2 TB RAM, NVMe SSD, 100 Gbps network
X2iedn: Up to 4 TB RAM, NVMe SSD, 100 Gbps network
X2iezn: Up to 12 TB RAM, highest memory density
X1e: Up to 3.9 TB RAM, legacy but still supported
X1: Up to 1.9 TB RAM, legacy

# R6i family (newer generation)
r6i.32xlarge: 1 TB RAM, ideal for HANA < 1TB
r6i.metal: 1 TB RAM, bare metal performance
r7iz: Latest generation, up to 1 TB RAM

# R5 family (common for mid-size HANA)
r5.24xlarge: 768 GB RAM
r5.16xlarge: 512 GB RAM
r5.12xlarge: 384 GB RAM

# Certified instance check
aws ec2 describe-instance-types \
  --filters "Name=instance-type,Values=x2idn.32xlarge" \
  --query 'InstanceTypes[0].[InstanceType,MemoryInfo.SizeInMiB,VCpuInfo.DefaultVCpus]'
```

#### SAP Application Server Instances
```bash
# Compute-optimized (for CPU-intensive workloads)
C5: General purpose SAP app servers
C6i: Latest generation, better price-performance
c5.18xlarge: 72 vCPUs, 144 GB RAM
c6i.32xlarge: 128 vCPUs, 256 GB RAM

# Memory-optimized (for memory-intensive apps)
M5: Balanced compute and memory
M6i: Latest generation
m5.24xlarge: 96 vCPUs, 384 GB RAM
m6i.32xlarge: 128 vCPUs, 512 GB RAM

# General Purpose (for smaller dev/test)
T3: Burstable performance
t3.2xlarge: 8 vCPUs, 32 GB RAM (dev/test only)
```

### SAP HANA Sizing Calculations

#### SAPS to vCPU Conversion
```
SAPS = Standard Application Performance Benchmark
1 SAPS ≈ 100 fully processed order line items per hour

Common conversions:
- c5.18xlarge: ~145,000 SAPS
- m5.24xlarge: ~120,000 SAPS
- r5.24xlarge: ~100,000 SAPS

Sizing formula:
Required SAPS = Current workload SAPS × Growth factor
Memory = Database size × 1.5 (for data + delta + temp)
```

#### Memory Sizing for HANA
```bash
# Production sizing formula
Total Memory = (DB size × 1.2) + (Workload memory) + (20% buffer)

# Example: 500 GB database
DB in-memory: 500 GB × 1.2 = 600 GB
Workload: 100 GB
Buffer: 140 GB (20%)
Total: 840 GB → Choose x2idn.32xlarge (1 TB)

# Minimum requirements
Dev/Test: 32 GB RAM (r5.xlarge minimum)
Production: 128 GB RAM minimum
Scale-out: 256 GB per node minimum
```

### SAP on AWS CLI Commands

#### Launch SAP Instance with Best Practices
```bash
# Launch HANA instance with enhanced networking
aws ec2 run-instances \
  --image-id ami-xxxxxxxxx \
  --instance-type x2idn.32xlarge \
  --key-name my-sap-key \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --placement "GroupName=hana-cluster,Tenancy=default" \
  --block-device-mappings '[
    {"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":50,"VolumeType":"gp3"}},
    {"DeviceName":"/dev/sdb","Ebs":{"VolumeSize":512,"VolumeType":"io2","Iops":64000}},
    {"DeviceName":"/dev/sdc","Ebs":{"VolumeSize":512,"VolumeType":"io2","Iops":64000}},
    {"DeviceName":"/dev/sdd","Ebs":{"VolumeSize":512,"VolumeType":"io2","Iops":64000}}
  ]' \
  --iam-instance-profile Name=SAP-HANA-Instance-Profile \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=PRD-HANA-Primary},{Key=SID,Value=PRD},{Key=Environment,Value=Production}]'

# Enable enhanced networking (SR-IOV)
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxxx \
  --sriov-net-support simple
```

#### AWS Backint Agent Configuration
```bash
# Install AWS Backint Agent for HANA backup to S3
# Download from S3
aws s3 cp s3://awssap-backint-agent/aws-backint-agent-latest.rpm .

# Install on HANA instance
sudo rpm -ivh aws-backint-agent-latest.rpm

# Configure Backint parameters
cat > /usr/sap/PRD/SYS/global/hdb/opt/backint_config.json <<EOF
{
  "region": "us-east-1",
  "bucket": "sap-hana-backups-prd",
  "prefix": "PRD/HANA/",
  "maxParallel": 16,
  "encrypt": "AES256",
  "storageClass": "INTELLIGENT_TIERING"
}
EOF

# Test backup
/usr/sap/PRD/HDB00/exe/hdbsql -u SYSTEM -d SYSTEMDB \
  "BACKUP DATA USING BACKINT ('FULL_BACKUP')"

# List backups in S3
aws s3 ls s3://sap-hana-backups-prd/PRD/HANA/ --recursive
```

#### Systems Manager for SAP Management
```bash
# Register SAP instances with Systems Manager
aws ssm put-parameter \
  --name "/sap/prd/hana/sid" \
  --value "PRD" \
  --type "String" \
  --tags "Key=Application,Value=SAP"

# Run SAP-specific commands
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets "Key=tag:SID,Values=PRD" \
  --parameters 'commands=[
    "su - prdadm -c \"sapcontrol -nr 00 -function GetSystemInstanceList\"",
    "su - prdadm -c \"sapcontrol -nr 00 -function GetProcessList\""
  ]'

# Check SAP system status
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --instance-ids i-xxxxxxxxx \
  --parameters 'commands=[
    "su - prdadm -c \"HDB info\"",
    "su - prdadm -c \"HDBSettings.sh systemOverview.py\""
  ]'
```

### SAP HANA Storage Configuration

#### EBS Volume Layout for HANA
```bash
# Best practice volume configuration for 1TB HANA system

# Option 1: Multiple volumes with software RAID
# Root: 50 GB gp3
# /hana/data: 3×512 GB io2 (RAID 0) = 1.5 TB, 192K IOPS
# /hana/log: 2×256 GB io2 (RAID 0) = 512 GB, 128K IOPS
# /hana/shared: 1×512 GB gp3 = 512 GB
# /usr/sap: 1×50 GB gp3 = 50 GB
# /backup: EFS or S3 via Backint

# Create RAID 0 for data volumes
sudo mdadm --create /dev/md0 \
  --level=0 --raid-devices=3 \
  /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1

# Format and mount
sudo mkfs.xfs /dev/md0
sudo mkdir -p /hana/data
sudo mount /dev/md0 /hana/data

# Add to fstab
echo "/dev/md0 /hana/data xfs defaults,nofail 0 2" | sudo tee -a /etc/fstab

# Option 2: Single large volumes (simpler, for smaller systems)
# /hana/data: 1×1024 GB io2, 64K IOPS
# /hana/log: 1×256 GB io2, 32K IOPS
# /hana/shared: 1×512 GB gp3

# Volume performance calculation
# IOPS needed = (Throughput in MB/s ÷ IO size in KB) × 1024
# For HANA data: 400 MB/s ÷ 256 KB = 1,600 IOPS minimum
# Recommended: 5× minimum = 8,000 IOPS
```

#### EFS for SAP Transport Directory
```bash
# Create EFS for shared SAP transport
aws efs create-file-system \
  --creation-token sap-transport-prd \
  --performance-mode generalPurpose \
  --throughput-mode bursting \
  --encrypted \
  --tags Key=Name,Value=SAP-Transport-PRD

# Create mount targets in each AZ
aws efs create-mount-target \
  --file-system-id fs-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --security-groups sg-xxxxxxxxx

# Mount on all SAP instances
sudo mkdir -p /usr/sap/trans
sudo mount -t efs fs-xxxxxxxxx:/ /usr/sap/trans

# Add to fstab
echo "fs-xxxxxxxxx:/ /usr/sap/trans efs defaults,_netdev 0 0" \
  | sudo tee -a /etc/fstab

# Configure transport management (STMS)
# In SAP transaction STMS, configure:
# - Transport directory: /usr/sap/trans
# - Enable CTS+ (Change and Transport System)
```

### High Availability Configuration

#### HANA System Replication Setup
```bash
# On PRIMARY node
# Enable HANA System Replication
su - prdadm
hdbnsutil -sr_enable --name=PRIMARY

# Check status
hdbnsutil -sr_state
cdpy && python systemReplicationStatus.py

# On SECONDARY node
# Register secondary with primary
su - prdadm
hdbnsutil -sr_register \
  --remoteHost=hana-primary \
  --remoteInstance=00 \
  --replicationMode=sync \
  --operationMode=logreplay \
  --name=SECONDARY

# Start secondary in replication mode
HDB start

# Verify replication status
hdbnsutil -sr_state
cdpy && python systemReplicationStatus.py

# Replication modes:
# sync = Synchronous (RPO = 0, for same AZ)
# syncmem = Synchronous in-memory (fastest, same AZ)
# async = Asynchronous (for cross-region DR)
```

#### Pacemaker Cluster Configuration
```bash
# Install Pacemaker on both nodes (RHEL/SUSE)
sudo yum install -y pacemaker pcs fence-agents-aws

# Configure cluster authentication
echo "hacluster:MyPassword123!" | sudo chpasswd
sudo pcs cluster auth hana-primary hana-secondary

# Create cluster
sudo pcs cluster setup --name sap-hana-cluster \
  hana-primary hana-secondary

# Start cluster
sudo pcs cluster start --all
sudo pcs cluster enable --all

# Configure STONITH (fencing) with AWS
sudo pcs stonith create stonith-sbd fence_aws \
  region=us-east-1 \
  access_key=AKIAXXXXXXXX \
  secret_key=XXXXXXXX \
  pcmk_host_map="hana-primary:i-xxxxxx;hana-secondary:i-yyyyyy"

# Create overlay IP resource (VIP for SAP clients)
sudo pcs resource create vip IPaddr2 \
  ip=10.0.1.100 \
  cidr_netmask=24 \
  nic=eth0

# Create HANA resources
sudo pcs resource create SAPHana SAPHana \
  SID=PRD \
  InstanceNumber=00 \
  PREFER_SITE_TAKEOVER=true \
  AUTOMATED_REGISTER=true \
  master notify=true clone-max=2 interleave=true

# Create topology resource
sudo pcs resource create SAPHanaTopology SAPHanaTopology \
  SID=PRD \
  InstanceNumber=00 \
  clone clone-node-max=1 interleave=true

# Set resource constraints
sudo pcs constraint order SAPHanaTopology-clone then SAPHana-master-slave
sudo pcs constraint colocation add vip with master SAPHana-master-slave 2000

# Test failover
sudo pcs node standby hana-primary
# Verify VIP moved to secondary
sudo pcs node unstandby hana-primary
```

#### AWS-Specific HA Components
```bash
# Configure Overlay IP using aws-vpc-move-ip
# Install overlay IP script
sudo wget https://github.com/aws/aws-ec2-net-utils/overlay-ip-agent.sh
sudo chmod +x overlay-ip-agent.sh

# Add secondary private IP to primary ENI
aws ec2 assign-private-ip-addresses \
  --network-interface-id eni-xxxxxxxxx \
  --private-ip-addresses 10.0.1.100 \
  --allow-reassignment

# Update route table for overlay IP (if needed)
aws ec2 create-route \
  --route-table-id rtb-xxxxxxxxx \
  --destination-cidr-block 10.0.1.100/32 \
  --network-interface-id eni-xxxxxxxxx

# Configure Route 53 health check for failover
aws route53 create-health-check \
  --health-check-config \
    IPAddress=10.0.1.50,Port=1128,Type=TCP,ResourcePath=/

# Create Route 53 failover records
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch file://failover-records.json
```

### SAP Migration Scenarios

#### Homogeneous System Copy (Same DB)
```bash
# Using SAP Software Provisioning Manager (SWPM)

# 1. Backup source system
su - prdadm
hdbsql -u SYSTEM -d SYSTEMDB "BACKUP DATA USING FILE ('FULL_BACKUP')"

# 2. Copy backup to AWS
aws s3 cp /backup/PRD/ s3://sap-migration-bucket/PRD/ --recursive

# 3. On target AWS instance, restore backup
aws s3 sync s3://sap-migration-bucket/PRD/ /backup/PRD/
hdbsql -u SYSTEM -d SYSTEMDB "RECOVER DATA USING FILE ('FULL_BACKUP')"

# 4. Run SWPM system copy
sudo /swpm/sapinst SAPINST_INPUT_PARAMETERS_URL=/tmp/inifile.params \
  SAPINST_EXECUTE_PRODUCT_ID=NW_ABAP_OneHost:S4HANA1909.CORE.HDB.ABAP \
  SAPINST_SKIP_DIALOGS=true

# 5. Post-copy activities
# - Update SAP profiles
# - Reconfigure RFC connections
# - Update logical system names
# - Test connectivity
```

#### Database Migration Option (DMO)
```bash
# Migrate from AnyDB to HANA during migration
# SUM (Software Update Manager) with DMO

# 1. Download SUM and DMO from SAP Support Portal
# 2. Prepare parameter file
cat > DMO_params.txt <<EOF
HDB_SCHEMA_PASSWORD=MyPassword123!
HDB_USERSTORE_KEY=SYSTEM
SOURCE_DB_TYPE=ORA
TARGET_DB_TYPE=HDB
PARALLEL_JOBS=12
EOF

# 3. Run DMO
./SUMSTART.sh -s /usr/sap/DMO -p DMO_params.txt

# 4. Monitor progress
tail -f /usr/sap/DMO/abap/log/UPGANA.LOG

# DMO benefits:
# - Combines upgrade + migration in one step
# - Unicode conversion if needed
# - Downtime optimization
# - Built-in validation
```

#### AWS Application Migration Service (MGN)
```bash
# For lift-and-shift SAP migration

# 1. Install replication agent on source
wget -O ./aws-replication-installer-init.py https://aws-application-migration-service-us-east-1.s3.amazonaws.com/latest/linux/aws-replication-installer-init.py
sudo python3 aws-replication-installer-init.py \
  --region us-east-1 \
  --aws-access-key-id AKIAXXXXXXXX \
  --aws-secret-access-key XXXXXXXX

# 2. Configure launch template
aws mgn update-launch-configuration \
  --source-server-id s-xxxxxxxxx \
  --target-instance-type x2idn.32xlarge \
  --copy-private-ip true \
  --copy-tags true

# 3. Test instance launch (non-disruptive)
aws mgn start-test --source-server-ids s-xxxxxxxxx

# 4. Cutover (final migration)
aws mgn start-cutover --source-server-ids s-xxxxxxxxx

# 5. Finalize cutover
aws mgn finalize-cutover --source-server-ids s-xxxxxxxxx
```

### Disaster Recovery Configuration

#### Cross-Region DR Setup
```bash
# Primary: us-east-1, DR: us-west-2

# 1. Configure async HANA System Replication
# On PRIMARY (us-east-1)
su - prdadm
hdbnsutil -sr_enable --name=PRIMARY_US_EAST

# On DR (us-west-2)
su - prdadm
hdbnsutil -sr_register \
  --remoteHost=hana-primary-east.company.com \
  --remoteInstance=00 \
  --replicationMode=async \
  --operationMode=logreplay \
  --name=DR_US_WEST

# 2. Configure VPC peering or Transit Gateway
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-xxxxxxxx \
  --peer-vpc-id vpc-yyyyyyyy \
  --peer-region us-west-2

# 3. Set up Route 53 health check and failover
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "sap-hana.company.com",
        "Type": "A",
        "SetIdentifier": "Primary",
        "Failover": "PRIMARY",
        "HealthCheckId": "xxxxxxxxx",
        "TTL": 60,
        "ResourceRecords": [{"Value": "10.0.1.100"}]
      }
    }]
  }'

# 4. Backup to S3 with cross-region replication
aws s3api put-bucket-replication \
  --bucket sap-backups-primary \
  --replication-configuration file://replication-config.json

# 5. Test DR failover procedure
# Document steps:
# a. Stop replication on primary
# b. Promote DR HANA to primary
# c. Update Route 53 manually if needed
# d. Start SAP application servers in DR
# e. Validate business processes
```

### Monitoring and Performance

#### CloudWatch Custom Metrics for SAP
```bash
# Install SAP CloudWatch monitoring scripts
# AWS provides enhanced monitoring for SAP

# Create CloudWatch dashboard
aws cloudwatch put-dashboard \
  --dashboard-name SAP-HANA-PRD \
  --dashboard-body file://sap-dashboard.json

# Custom metric: HANA memory usage
#!/bin/bash
# /usr/local/bin/sap-cw-metrics.sh
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
HANA_MEM=$(su - prdadm -c "hdbcons 'hdbindexserver -v' | grep 'Total Memory' | awk '{print \$3}'")

aws cloudwatch put-metric-data \
  --namespace SAP/HANA \
  --metric-name MemoryUsage \
  --value $HANA_MEM \
  --unit Bytes \
  --dimensions InstanceId=$INSTANCE_ID,SID=PRD

# Schedule via cron
echo "*/5 * * * * /usr/local/bin/sap-cw-metrics.sh" | crontab -

# Create alarms
aws cloudwatch put-metric-alarm \
  --alarm-name SAP-HANA-High-Memory \
  --alarm-description "HANA memory usage above 90%" \
  --namespace SAP/HANA \
  --metric-name MemoryUsage \
  --statistic Average \
  --period 300 \
  --threshold 90 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:sap-alerts
```

#### SAP Solution Manager Integration
```bash
# Configure SAP Solution Manager to monitor AWS-based systems

# 1. Configure RFC connection from Solution Manager to AWS
# Transaction SM59: Create RFC destination
# Connection type: TCP/IP
# Target host: sap-app.company.com
# Use AWS Direct Connect or VPN for connectivity

# 2. Enable Solution Manager diagnostics agent
# On managed SAP instance
su - prdadm
cd /usr/sap/PRD/DVEBMGS00/work
./smdagent -start

# 3. Configure managed system in Solution Manager
# Transaction SOLMAN_SETUP
# Add AWS-based systems to monitoring

# 4. Enable Early Watch Alert
# Configure EWA reports to include AWS-specific metrics
# Monitor: CPU, memory, disk I/O, network throughput

# 5. Set up alerting
# Transaction RZ20: Configure alert thresholds
# Include AWS CloudWatch metrics via custom scripts
```

### AWS Launch Wizard for SAP

#### Automated SAP Deployment
```bash
# Launch Wizard deploys SAP using CloudFormation templates

# 1. Prepare input parameters
cat > launch-wizard-params.json <<EOF
{
  "SAPInstanceNumber": "00",
  "SAPSID": "PRD",
  "HANAMasterPassword": "MyPassword123!",
  "SAPInstanceType": "x2idn.32xlarge",
  "PrivateSubnetId": "subnet-xxxxxxxxx",
  "SecurityGroupId": "sg-xxxxxxxxx",
  "KeyName": "sap-key-pair",
  "HANADataVolSize": 1024,
  "HANALogVolSize": 256,
  "HANASharedVolSize": 512,
  "EnableHA": true,
  "EnableBackup": true,
  "BackupSchedule": "cron(0 2 * * ? *)"
}
EOF

# 2. Launch via AWS CLI (or use Console wizard)
# Note: Launch Wizard doesn't have direct CLI, use Console or CloudFormation

# 3. Monitor deployment
aws cloudformation describe-stacks \
  --stack-name SAP-HANA-PRD \
  --query 'Stacks[0].StackStatus'

# 4. Retrieve outputs
aws cloudformation describe-stacks \
  --stack-name SAP-HANA-PRD \
  --query 'Stacks[0].Outputs'

# Launch Wizard creates:
# - VPC and subnets (if needed)
# - EC2 instances with optimal configuration
# - EBS volumes with proper layout
# - Security groups
# - IAM roles
# - CloudWatch alarms
# - AWS Backup plans
# - HA configuration (if selected)
```

## Common Troubleshooting Scenarios

### Scenario 1: HANA System Replication Not Working

**Symptoms**: Secondary HANA not syncing, replication status shows ERROR

**Troubleshooting Steps**:
1. Check network connectivity between nodes
   ```bash
   # From primary, test connection to secondary
   telnet hana-secondary 30001
   telnet hana-secondary 30003
   telnet hana-secondary 30007

   # HANA ports:
   # 3XX00-3XX09: SQL/MDX/HTTP
   # 3XX01-3XX09: Internal communication
   # 3XX40: System replication
   ```

2. Verify HANA System Replication status
   ```bash
   su - prdadm
   hdbnsutil -sr_state
   cdpy && python systemReplicationStatus.py

   # Check for error messages
   cdtrace && tail -f nameserver_*.trc
   ```

3. Check security group rules
   ```bash
   # Must allow all HANA ports between nodes
   aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx \
     --query 'SecurityGroups[0].IpPermissions[?FromPort==`30001`]'

   # Add rule if missing
   aws ec2 authorize-security-group-ingress \
     --group-id sg-xxxxxxxxx \
     --protocol tcp \
     --port 30000-30099 \
     --source-group sg-xxxxxxxxx
   ```

4. Verify /etc/hosts and hostname resolution
   ```bash
   # On both nodes, verify entries
   cat /etc/hosts
   # Should have:
   # 10.0.1.10 hana-primary
   # 10.0.1.11 hana-secondary

   # Test DNS resolution
   nslookup hana-secondary
   ping hana-secondary
   ```

5. Check storage performance (slow I/O can cause replication lag)
   ```bash
   # Test disk performance
   sudo fio --name=random-write --ioengine=libaio --rw=randwrite \
     --bs=4k --size=1G --numjobs=4 --runtime=60 --group_reporting \
     --filename=/hana/data/testfile

   # Check for throttled IOPS
   aws cloudwatch get-metric-statistics \
     --namespace AWS/EBS \
     --metric-name BurstBalance \
     --dimensions Name=VolumeId,Value=vol-xxxxxxxxx \
     --start-time 2024-01-01T00:00:00Z \
     --end-time 2024-01-01T23:59:59Z \
     --period 300 --statistics Average
   ```

6. Review HANA trace files
   ```bash
   su - prdadm
   cdtrace
   grep -i "replication" *.trc | tail -50
   grep -i "error" *.trc | tail -50
   ```

7. Re-register secondary if needed
   ```bash
   # On SECONDARY node
   su - prdadm
   HDB stop
   hdbnsutil -sr_unregister
   hdbnsutil -sr_register \
     --remoteHost=hana-primary \
     --remoteInstance=00 \
     --replicationMode=sync \
     --operationMode=logreplay \
     --name=SECONDARY
   HDB start
   ```

### Scenario 2: Pacemaker Cluster Split-Brain

**Symptoms**: Both nodes think they are primary, VIP assigned to both

**Root Cause**: Network partition or STONITH failure

**Resolution Steps**:
```bash
# 1. Check cluster status on both nodes
sudo pcs status

# 2. If split-brain detected, stop cluster on secondary
sudo pcs cluster stop hana-secondary

# 3. Verify primary is healthy
sudo pcs status | grep -i "Online:"

# 4. Check STONITH configuration
sudo pcs stonith show stonith-sbd
sudo pcs stonith test stonith-sbd hana-secondary

# 5. Clean up resources
sudo pcs resource cleanup

# 6. Start secondary back
sudo pcs cluster start hana-secondary

# 7. Verify cluster converged
sudo pcs status

# Prevention:
# - Enable STONITH (fencing)
# - Use dedicated heartbeat network
# - Configure proper quorum settings
sudo pcs quorum expected-votes 2
sudo pcs quorum unblock
```

### Scenario 3: SAP Application Server Can't Connect to HANA

**Symptoms**: ST22 dumps, connection errors in sm21

**Troubleshooting**:
```bash
# 1. Test connectivity from app server
telnet hana-primary 30015
# HANA SQL port = 3XX15 (where XX = instance number)

# 2. Check SAP profile parameters
su - prdadm
cdpro
grep "dbs/hdb/dbname" DEFAULT.PFL
grep "SAPDBHOST" DEFAULT.PFL

# 3. Verify HANA is listening
netstat -tlnp | grep 30015

# 4. Check security group from app server to HANA
aws ec2 describe-security-groups --group-ids sg-appserver \
  --query 'SecurityGroups[0].IpPermissionsEgress'

# 5. Test with hdbsql
/usr/sap/hdbclient/hdbsql -n hana-primary:30015 -u SYSTEM -p Password

# 6. Check R3trans connection
export DBHOST=hana-primary
R3trans -d

# 7. Verify hdbuserstore keys
hdbuserstore list
# If missing, create:
hdbuserstore set DEFAULT hana-primary:30015 SAPSR3 password

# 8. Review SAP connection trace
su - prdadm
cd /usr/sap/PRD/DVEBMGS00/work
tail -f dev_w0
```

### Scenario 4: AWS Backint Backup Failing

**Symptoms**: HANA backup completes with errors, files not in S3

**Troubleshooting**:
```bash
# 1. Check IAM role permissions
aws iam get-role --role-name SAP-HANA-Instance-Role
aws iam list-attached-role-policies --role-name SAP-HANA-Instance-Role

# Required permissions:
# - s3:PutObject
# - s3:GetObject
# - s3:DeleteObject
# - s3:ListBucket

# 2. Verify Backint configuration
cat /usr/sap/PRD/SYS/global/hdb/opt/backint_config.json

# 3. Test S3 access from instance
aws s3 ls s3://sap-hana-backups-prd/
aws s3 cp testfile.txt s3://sap-hana-backups-prd/test/

# 4. Check Backint logs
cat /usr/sap/PRD/HDB00/backup.log
grep -i "error" /usr/sap/PRD/HDB00/backup.log

# 5. Run manual test backup
su - prdadm
hdbsql -u SYSTEM -d SYSTEMDB \
  "BACKUP DATA USING BACKINT ('TEST_BACKUP')"

# 6. Verify files in S3
aws s3 ls s3://sap-hana-backups-prd/PRD/HANA/ --recursive

# 7. Check S3 bucket policy
aws s3api get-bucket-policy --bucket sap-hana-backups-prd

# 8. Review VPC endpoint (if private subnet)
aws ec2 describe-vpc-endpoints --filters "Name=service-name,Values=com.amazonaws.us-east-1.s3"
```

### Scenario 5: High Latency Between App Server and HANA

**Symptoms**: Slow response times, users complaining, ST03 shows high DB time

**Troubleshooting**:
```bash
# 1. Check placement groups
aws ec2 describe-instances --instance-ids i-xxxxxxxxx \
  --query 'Reservations[0].Instances[0].Placement'

# Should use cluster placement group for low latency
aws ec2 create-placement-group \
  --group-name sap-cluster \
  --strategy cluster

# 2. Verify enhanced networking enabled
aws ec2 describe-instance-attribute \
  --instance-id i-xxxxxxxxx \
  --attribute sriovNetSupport

# 3. Check network performance metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name NetworkPacketsIn \
  --dimensions Name=InstanceId,Value=i-xxxxxxxxx \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 300 --statistics Average

# 4. Test network latency
# From app server to HANA
ping -c 100 hana-primary | tail -1
# Should be < 1 ms in same AZ

# 5. Check for network throttling
# Exceeded network bandwidth for instance type?
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name NetworkOut \
  --dimensions Name=InstanceId,Value=i-xxxxxxxxx \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 60 --statistics Maximum

# 6. Review HANA performance
su - prdadm
hdbcons "indexserver perftrace on"
# Run workload
hdbcons "indexserver perftrace off"
# Analyze trace files

# 7. Check SAP ABAP performance
# Transaction ST03 - Workload analysis
# Transaction ST12 - ABAP trace
# Transaction DB02 - Database statistics

# 8. Verify no cross-AZ traffic (costs + latency)
# App servers should be in same AZ as active HANA node
```

### Scenario 6: SAP Migration Cutover Taking Too Long

**Symptoms**: Downtime exceeding planned window

**Common Causes & Solutions**:

1. **Database backup/restore slow**
   ```bash
   # Use multiple parallel streams
   hdbsql -u SYSTEM "BACKUP DATA USING FILE ('FULL') WITH THREADS 16"

   # Use AWS Backint instead of file backup
   hdbsql -u SYSTEM "BACKUP DATA USING BACKINT ('FULL')"

   # Optimize network bandwidth
   # Use Direct Connect or VPN with provisioned bandwidth
   aws directconnect describe-connections
   ```

2. **Large number of transport requests**
   ```bash
   # Pre-load transports before cutover
   # Use tp import with multiple processes
   tp import all PRD pf=/usr/sap/trans/bin/TP_DOMAIN_PRD.PFL \
     client=100 U126 -Dforce_utf8
   ```

3. **Unicode conversion in DMO**
   ```bash
   # Increase parallel jobs in DMO
   # Edit SAPup_add.par
   parallel_jobs = 20

   # Use SSD-backed storage for temp files
   # Mount /usr/sap/DMO on io2 volumes
   ```

4. **Network bandwidth insufficient**
   ```bash
   # Check current bandwidth
   iperf3 -c aws-endpoint -P 10

   # Use multiple network streams
   # Configure AWS DataSync for large file transfers
   aws datasync create-task \
     --source-location-arn arn:aws:datasync:...:location/source \
     --destination-location-arn arn:aws:datasync:...:location/dest \
     --options OverwriteMode=ALWAYS,TransferMode=ALL
   ```

## Exam Tips and Common Traps

### SAP HANA Sizing Traps

**Common Mistake**: Choosing instance based only on database size
```
WRONG: 500 GB database → r5.12xlarge (384 GB) thinking compression
CORRECT: 500 GB database → x2idn.32xlarge (1 TB)
Reason: Need 1.5-2× DB size for in-memory operations
```

**Exam Tip**: Always add 20-50% buffer for HANA memory sizing

**Instance Selection Matrix**:
```
< 128 GB HANA: r5.4xlarge (Dev/Test only)
128-512 GB: r5.12xlarge to r5.24xlarge
512 GB-1 TB: x2idn.32xlarge (preferred) or r5.metal
1-2 TB: X2idn.16xlarge to X2idn.32xlarge
2-4 TB: X2iedn family
4-12 TB: X2iezn family
> 12 TB: Scale-out cluster (multiple nodes)
```

### High Availability Gotchas

**STONITH is Critical**:
- **Exam Scenario**: "Pacemaker cluster without STONITH - what's the risk?"
- **Answer**: Split-brain scenario - both nodes could become primary
- **Solution**: Always configure AWS fence agent

**Overlay IP vs Elastic IP**:
```
Overlay IP (Secondary Private IP):
✓ Fast failover (seconds)
✓ No AWS API calls needed
✓ Works within VPC
✗ Requires custom scripting

Elastic IP:
✓ Works across internet
✓ Simple to understand
✗ Slower failover (API call required)
✗ Not recommended for production HA
```

**Common Trap**:
> Q: Why is application server still connecting to old HANA after failover?
>
> **WRONG**: Security group issue
>
> **CORRECT**: Application profile still has old hostname; needs virtual hostname or overlay IP

### Storage Configuration Traps

**RAID Misconceptions**:
```
Q: Should I use RAID 10 for HANA data volumes?

WRONG: Yes, for redundancy
CORRECT: No, use RAID 0 only

Reason:
- EBS volumes are already redundant (replicated in AZ)
- RAID 0 provides maximum performance
- RAID 10 adds unnecessary overhead
- Use HANA System Replication for data protection
```

**Volume Type Selection**:
```
For /hana/data: io2 (provisioned IOPS)
For /hana/log: io2 (high IOPS critical)
For /hana/shared: gp3 (lower IOPS okay)
For /backup: gp3 or S3 via Backint

WRONG: Use gp3 for everything (cost optimization)
CORRECT: Use io2 for data/log (performance critical)
```

### Backup Strategy Traps

**Exam Scenario**: "Fastest restore time objective for SAP HANA"
```
Options:
A) Daily file-based backup to EBS
B) Backint backup to S3
C) HANA System Replication to standby
D) EBS snapshots

CORRECT: C - HANA System Replication
Reason: Immediate failover (minutes), not restore (hours)

For backup/restore specifically:
Best: Backint to S3 (parallel restore, high bandwidth)
Second: File-based to EFS (network limitations)
Avoid: EBS snapshots (not HANA-consistent without quiesce)
```

**Backup Retention Trap**:
```
Q: Keep daily backups for 1 year compliance

WRONG: Keep all backups in S3 Standard
CORRECT: Use S3 Lifecycle policies
- 0-7 days: S3 Standard (fast restore)
- 7-30 days: S3 Standard-IA
- 30-90 days: S3 Glacier Instant Retrieval
- 90+ days: S3 Glacier Deep Archive
```

### Migration Strategy Selection

**When to Use Which Method**:
```
Lift-and-Shift (MGN):
✓ Fastest to AWS
✓ No SAP downtime for replication
✗ No upgrade/optimization
✗ Still on old DB/OS
Use: Quick migration, optimize later

Homogeneous System Copy:
✓ Proven SAP procedure
✓ Control over process
✗ Requires SAP skills
✗ Manual steps
Use: Standard migration, same DB

DMO (Database Migration Option):
✓ Upgrade + migrate together
✓ AnyDB → HANA
✓ Unicode conversion
✗ Longer downtime
✗ Complex process
Use: Technical debt reduction

Greenfield S/4HANA:
✓ Clean slate
✓ Business process redesign
✗ Longest project
✗ Data migration required
Use: Major transformation
```

**Exam Question Pattern**:
> Q: Company wants to migrate SAP ECC on Oracle to S/4HANA on AWS HANA with minimal downtime
>
> **Key Words**: "S/4HANA" (not just HANA), "minimal downtime"
>
> **Answer**: SUM/DMO with downtime optimization techniques (parallel export, table splitting)
>
> **NOT**: Lift-and-shift (doesn't convert to S/4HANA)

### Network Design Patterns

**Direct Connect Sizing**:
```
Minimum for SAP:
- Dev/Test: 1 Gbps
- Production: 10 Gbps (preferably redundant)
- High-volume (>10 TB data): 100 Gbps

Exam Trap:
Q: 1 Gbps Direct Connect for 5 TB database migration

Wrong: Sufficient
Correct: Insufficient; would take 11+ hours assuming 100% utilization
Solution: Use Snowball for initial data, DX for sync
```

**VPC Architecture**:
```
Best Practice Multi-Account Setup:

Network Account (Hub):
- Transit Gateway
- Direct Connect Gateway
- Shared Services (AD, DNS)

SAP Production Account:
- Production VPC
- PRD, QAS systems
- Private subnets only

SAP Non-Production Account:
- Dev/Test VPC
- DEV, SBX systems
- Can have internet access

Exam Focus:
- Understand VPC peering vs Transit Gateway
- Know subnet sizing (plan for growth)
- Security group strategy (minimize open rules)
```

### Cost Optimization Strategies

**Instance Purchase Options**:
```
On-Demand:
- Development systems
- Short-term projects
- Testing

Reserved Instances (1-3 year):
✓ Production HANA (up to 72% savings)
✓ Application servers running 24/7
✗ Don't over-commit (changing instance types)

Savings Plans:
✓ Flexible across instance families
✓ Good for mixed workloads
- Commit based on usage patterns

Exam Scenario: "Most cost-effective for SAP DEV"
Answer: Scheduled scaling (off during nights/weekends)
Savings: ~70% vs 24/7 on-demand
```

**Storage Optimization**:
```bash
# Delete old HANA backups
# Lifecycle policy for S3
aws s3api put-bucket-lifecycle-configuration \
  --bucket sap-backups \
  --lifecycle-configuration '{
    "Rules": [{
      "Id": "ArchiveOldBackups",
      "Status": "Enabled",
      "Transitions": [
        {"Days": 7, "StorageClass": "STANDARD_IA"},
        {"Days": 30, "StorageClass": "GLACIER_IR"}
      ],
      "Expiration": {"Days": 365}
    }]
  }'

# EBS volume optimization
# Convert gp2 to gp3 (20% cheaper, better performance)
aws ec2 modify-volume --volume-id vol-xxx --volume-type gp3
```

### Security Best Practices

**SAP-Specific Security**:
```
Common Mistakes:
- Opening port 3200-3299 to 0.0.0.0/0
- Using default SAP passwords (DDIC, SAP*)
- No encryption for HANA backups
- Shared IAM keys on instances

Best Practices:
- Limit SAP ports to known IP ranges
- Use Secrets Manager for SAP passwords
- Enable S3 encryption (KMS) for backups
- Use IAM roles, not access keys
- Enable CloudTrail for audit
- Use PrivateLink for AWS service access

Exam Question:
Q: How to securely store SAP HANA SYSTEM password for automation?

Wrong: Parameter Store String
Correct: Secrets Manager with automatic rotation
Benefit: Automatic rotation, audit trail, fine-grained access
```

### SAP Notes to Remember

**Critical SAP Notes for Exam**:
- **1656250**: SAP on AWS - Support prerequisites
- **2015404**: SAP HANA on AWS - Installation and operation
- **2456406**: AWS Launch Wizard for SAP
- **2927211**: AWS Backint Agent for SAP HANA
- **2369910**: SAP Software on AWS - General information
- **2235581**: SAP HANA Support on AWS
- **3108316**: SAP HANA High Availability across AZs

**Exam Tip**: Questions often reference "as per SAP Note" - know the high-level content, not the exact note number.

## Pre-Exam Checklist

### Core SAP on AWS Concepts to Master

- ✅ **SAP HANA Instance Types**: X2, R5/R6i families, memory sizing (1.5-2× DB)
- ✅ **Storage Layouts**: RAID 0 for data/log, io2 vs gp3, EBS volume sizing
- ✅ **High Availability**: HSR (sync/async), Pacemaker, overlay IP, STONITH
- ✅ **Backup/Restore**: Backint agent, S3 lifecycle, restore procedures
- ✅ **Migration Strategies**: Lift-and-shift, homogeneous, DMO, greenfield
- ✅ **Network Design**: Direct Connect sizing, VPC architecture, Transit Gateway
- ✅ **Monitoring**: CloudWatch metrics, SAP Solution Manager integration
- ✅ **Security**: IAM roles, KMS encryption, security groups for SAP ports
- ✅ **Launch Wizard**: Automated deployment, CloudFormation templates
- ✅ **DR Strategies**: Cross-region HSR async, RPO/RTO objectives

### SAP Architecture Patterns

**Three-Tier Architecture**:
```
Load Balancer (ALB/NLB)
    ↓
SAP Web Dispatcher (Presentation Layer)
    ↓
Multiple App Servers (Application Layer)
- Central Services (ASCS/ERS)
- Dialog instances (PAS/AAS)
- In multiple AZs for HA
    ↓
HANA Database (Database Layer)
- Primary + Secondary (HSR)
- Pacemaker cluster
- Overlay IP for VIP
```

**Multi-System Landscape**:
```
Shared Services VPC:
- SAP Router
- SAP Solution Manager
- Transport Directory (EFS)
- Backup (S3 buckets)

Production VPC:
- PRD system (highly available)
- QAS system (pre-production testing)

Non-Production VPC:
- DEV system
- SBX system (sandbox/training)

Connectivity:
- Transit Gateway connects all VPCs
- Direct Connect to on-premises
- VPC endpoints for AWS services
```

### Troubleshooting Checklist

**HANA Issues**:
- [ ] Check HANA system status: `HDB info`
- [ ] Verify memory usage: `hdbcons "mm info"`
- [ ] Review trace files: `/usr/sap/PRD/HDB00/*/trace/`
- [ ] Check table distribution: `SELECT * FROM M_CS_TABLES`
- [ ] Validate backups: Check S3 or backup catalog
- [ ] Test HSR status: `hdbnsutil -sr_state`

**Application Server Issues**:
- [ ] Check SAP processes: `sapcontrol -nr 00 -function GetProcessList`
- [ ] Review work process status: Transaction SM50
- [ ] Check system log: Transaction SM21
- [ ] Verify database connection: R3trans -d
- [ ] Check update status: Transaction SM13
- [ ] Review short dumps: Transaction ST22

**Infrastructure Issues**:
- [ ] Verify EC2 instance status and health checks
- [ ] Check security groups (3000-3099, 5XX00-5XX99)
- [ ] Review VPC Flow Logs for dropped traffic
- [ ] Validate IAM role permissions
- [ ] Check EBS volume performance (IOPS, throughput)
- [ ] Review CloudWatch alarms and metrics
- [ ] Verify DNS resolution (Route 53 or /etc/hosts)
- [ ] Test network connectivity (ping, telnet)

### Exam Day Strategy

**Time Management**:
- 170 minutes / 65 questions = ~2.6 minutes per question
- Budget 2 minutes per question, leaves 30 minutes for review
- Flag difficult questions, return at end
- Don't second-guess - first instinct usually correct

**Question Analysis**:
```
Key Phrases and Their Meanings:

"Most cost-effective" → Think RI/Savings Plans, right-sizing, lifecycle policies
"Minimal downtime" → HA setup, replication, automated failover
"Least operational overhead" → Managed services (Launch Wizard), automation
"Best performance" → Instance types, placement groups, enhanced networking
"Highest availability" → Multi-AZ, HSR sync, Pacemaker, multiple app servers
"Compliance requirement" → Encryption (KMS), CloudTrail, backup retention
"Disaster recovery" → Cross-region HSR async, backup replication, DR testing
"Migration with upgrade" → DMO (AnyDB to HANA), SUM for S/4HANA
```

**Elimination Strategy**:
- Eliminate obviously wrong answers first
- Watch for "always/never" statements (usually wrong)
- If two answers seem correct, choose the AWS-native solution
- Remember SAP best practices trump general AWS practices

**SAP-Specific Focus Areas**:
```
Heavy Weight (expect 30-40% of questions):
- HANA sizing and instance selection
- High availability setup (HSR + Pacemaker)
- Migration strategies and tools
- Backup/restore procedures

Medium Weight (expect 20-30%):
- Storage configuration and performance
- Network design (DX, VPC, security groups)
- Monitoring and operations
- Cost optimization

Light Weight (expect 10-20%):
- Launch Wizard automation
- Security and compliance
- Integration with other AWS services
```

### Final Preparation

**Week Before Exam**:
- ✅ Take full practice exam (Whizlabs, TutorialsDojo, official AWS)
- ✅ Review all incorrect answers and understand why
- ✅ Re-read AWS SAP Lens (Well-Architected Framework)
- ✅ Skim through key SAP Notes (1656250, 2015404, 2456406)
- ✅ Review your own documentation from hands-on practice

**Day Before Exam**:
- ✅ Light review only (don't cram new material)
- ✅ Verify exam location/setup (online or test center)
- ✅ Prepare valid ID (two forms for test center)
- ✅ Get good sleep (8+ hours)
- ✅ Avoid alcohol and late-night studying

**Exam Day**:
- ✅ Eat a good breakfast (protein + complex carbs)
- ✅ Arrive 30 minutes early (test center) or start 15 min early (online)
- ✅ Use restroom before starting (no breaks during exam)
- ✅ Take deep breaths if anxious
- ✅ Read each question twice before answering
- ✅ Trust your preparation

### Post-Exam Actions

**Pass or Fail**:
- Score report available immediately (pass/fail)
- Detailed score by domain available in 5 business days
- Certificate available in AWS Certification account
- Digital badge can be shared on LinkedIn
- Update resume/CV with certification

**If You Pass**:
- Celebrate! This is a challenging exam.
- Update LinkedIn with AWS certification badge
- Consider next certification (DevOps Pro, Security Specialty)
- Stay updated with new SAP on AWS features
- Join AWS SAP community forums

**If You Don't Pass**:
- Review domain-level scores (shows weak areas)
- Wait 14 days before retaking
- Focus study on weakest domains
- Get more hands-on practice
- Consider AWS training courses or labs
- Don't give up - many pass on second attempt

### Key Resources Summary

**Official AWS Documentation**:
- SAP on AWS Implementation Guides
- AWS Launch Wizard for SAP documentation
- EC2 SAP instance types list
- AWS Backint Agent for SAP HANA guide
- SAP on AWS Quick Start reference architectures

**SAP Resources**:
- SAP Note 1656250 (master note for AWS)
- SAP HANA Administration Guide
- SAP Installation guides for AWS
- SAP Community forums

**Practice and Training**:
- AWS Skill Builder: SAP on AWS courses
- Whizlabs PAS-C01 practice exams
- TutorialsDojo PAS-C01 practice tests
- Linux Academy / A Cloud Guru courses
- Hands-on labs in AWS account

**AWS Whitepapers**:
- SAP HANA on AWS Operations Guide
- SAP on AWS Technical Overview
- SAP Migration to AWS Best Practices
- AWS Well-Architected Framework: SAP Lens

Good luck with your AWS Certified SAP on AWS - Specialty exam!
