# OCI Operations Associate - Monitoring and Troubleshooting

## Table of Contents
- [Observability Platform](#observability-platform)
- [Monitoring Service](#monitoring-service)
- [Logging Service](#logging-service)
- [Troubleshooting](#troubleshooting)
- [Exam Tips](#exam-tips)

---

## Observability Platform

### Architecture Overview

```
Observability Stack:
├── Monitoring (Metrics & Alarms)
├── Logging (Service & Custom Logs)
├── Notifications (Alert delivery)
├── Events (State change triggers)
└── Service Connector Hub (Data routing)
```

---

## Monitoring Service

### Metrics

**Viewing Metrics**:
```bash
# List available metrics
oci monitoring metric list \
  --compartment-id $COMPARTMENT_ID \
  --list-metrics-details '{
    "namespace":"oci_computeagent",
    "name":"CpuUtilization"
  }'

# Query metric data
oci monitoring metric-data summarize-metrics-data \
  --compartment-id $COMPARTMENT_ID \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1m]{resourceId="ocid1.instance..."}.mean()' \
  --start-time "2024-10-10T00:00:00Z" \
  --end-time "2024-10-10T23:59:59Z"
```

**Common Metrics by Service**:

**Compute** (oci_computeagent):
```
CpuUtilization - CPU usage percentage
MemoryUtilization - Memory usage percentage
DiskBytesRead - Disk read bytes
DiskBytesWritten - Disk write bytes
NetworkBytesIn - Network ingress
NetworkBytesOut - Network egress
```

**Block Volume** (oci_blockstore):
```
VolumeReadThroughput - Read throughput (bytes/sec)
VolumeWriteThroughput - Write throughput (bytes/sec)
VolumeReadOps - Read IOPS
VolumeWriteOps - Write IOPS
```

**Load Balancer** (oci_lbaas):
```
HttpRequests - Total HTTP requests
ActiveConnections - Current active connections
UnhealthyBackendServers - Number of unhealthy backends
HttpResponseTime - Response time (ms)
```

**Database** (oci_database):
```
CpuUtilization - Database CPU usage
StorageUtilization - Storage used percentage
ActiveSessions - Active database sessions
```

### Alarms

**Creating Alarms**:
```bash
# High CPU alarm
oci monitoring alarm create \
  --compartment-id $COMPARTMENT_ID \
  --display-name "High-CPU-Alarm" \
  --metric-compartment-id $COMPARTMENT_ID \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1m]{resourceId="ocid1.instance..."}.mean() > 80' \
  --severity "CRITICAL" \
  --destinations "[$ONS_TOPIC_ID]" \
  --is-enabled true \
  --repeat-notification-duration "PT2H"
```

**Alarm States**:
- **OK**: Metric below threshold
- **FIRING**: Alarm triggered (threshold exceeded)
- **RESET**: Was firing, now OK

**Alarm Query Syntax**:
```
MetricName[interval]{dimensions}.statistic() comparison value

Examples:
CpuUtilization[5m].mean() > 75
MemoryUtilization[1m].max() > 90
DiskBytesRead[10m].sum() < 1000
```

**Alarm Best Practices**:
```terraform
# Critical alarm - immediate action
resource "oci_monitoring_alarm" "critical" {
  compartment_id        = var.compartment_id
  display_name          = "CRITICAL-High-CPU"
  metric_compartment_id = var.compartment_id
  namespace             = "oci_computeagent"
  query                 = "CpuUtilization[1m].mean() > 90"
  severity              = "CRITICAL"
  destinations          = [oci_ons_topic.critical.id]

  # Suppress repeating notifications
  repeat_notification_duration = "PT1H"

  # Notification format
  message_format = "ONS_OPTIMIZED"

  # Body message
  body = "CRITICAL: CPU usage above 90% on ${resource_name}"
}

# Warning alarm - investigation needed
resource "oci_monitoring_alarm" "warning" {
  display_name = "WARNING-Elevated-CPU"
  query        = "CpuUtilization[5m].mean() > 75"
  severity     = "WARNING"
  destinations = [oci_ons_topic.warnings.id]

  repeat_notification_duration = "PT2H"
}
```

### Custom Metrics

**Publishing Custom Metrics**:
```python
import oci
from datetime import datetime

monitoring_client = oci.monitoring.MonitoringClient(config)

def publish_custom_metric(metric_name, value, dimensions={}):
    post_metric_data_response = monitoring_client.post_metric_data(
        post_metric_data_details=oci.monitoring.models.PostMetricDataDetails(
            metric_data=[
                oci.monitoring.models.MetricDataDetails(
                    namespace="custom_metrics",
                    compartment_id=config["compartment_id"],
                    name=metric_name,
                    dimensions=dimensions,
                    datapoints=[
                        oci.monitoring.models.Datapoint(
                            timestamp=datetime.utcnow(),
                            value=value
                        )
                    ]
                )
            ]
        )
    )

# Example usage
publish_custom_metric(
    "application_errors",
    25,
    dimensions={
        "app": "web-app",
        "environment": "production"
    }
)

publish_custom_metric(
    "queue_length",
    150,
    dimensions={"queue_name": "tasks"}
)
```

**Querying Custom Metrics**:
```bash
oci monitoring metric-data summarize-metrics-data \
  --compartment-id $COMPARTMENT_ID \
  --namespace "custom_metrics" \
  --query-text 'application_errors[5m]{app="web-app"}.sum()'
```

---

## Logging Service

### Service Logs

**Enabling VCN Flow Logs**:
```bash
oci logging log create \
  --log-group-id $LOG_GROUP_ID \
  --display-name "vcn-flow-logs" \
  --log-type SERVICE \
  --configuration '{
    "source": {
      "sourceType":"OCISERVICE",
      "service":"flowlogs",
      "resource":"'$SUBNET_ID'",
      "category":"all"
    },
    "compartmentId":"'$COMPARTMENT_ID'"
  }' \
  --is-enabled true \
  --retention-duration 30
```

**Enabling Load Balancer Logs**:
```bash
# Access logs
oci logging log create \
  --log-group-id $LOG_GROUP_ID \
  --display-name "lb-access-logs" \
  --log-type SERVICE \
  --configuration '{
    "source": {
      "sourceType":"OCISERVICE",
      "service":"loadbalancer",
      "resource":"'$LB_ID'",
      "category":"access"
    },
    "compartmentId":"'$COMPARTMENT_ID'"
  }'

# Error logs
oci logging log create \
  --log-group-id $LOG_GROUP_ID \
  --display-name "lb-error-logs" \
  --log-type SERVICE \
  --configuration '{
    "source": {
      "sourceType":"OCISERVICE",
      "service":"loadbalancer",
      "resource":"'$LB_ID'",
      "category":"error"
    },
    "compartmentId":"'$COMPARTMENT_ID'"
  }'
```

### Log Search

**Basic Queries**:
```
# Find errors in last hour
search "compartment-ocid/log-group-name" |
where level = 'ERROR' |
where datetime >= now() - 1h

# Find specific IP address
search "compartment-ocid/vcn-flow-logs" |
where data.sourceAddress = '203.0.113.10' |
sort by datetime desc

# Count errors by type
search "compartment-ocid/application-logs" |
where level = 'ERROR' |
stats count() by error_type

# Find slow requests
search "compartment-ocid/lb-access-logs" |
where data.responseTime > 5000 |
sort by data.responseTime desc |
limit 20
```

**Advanced Queries**:
```
# Failed authentication attempts
search "compartment-ocid/audit-logs" |
where data.message =~ 'authentication failed' |
stats count() as failures by data.sourceAddress |
where failures > 5 |
sort by failures desc

# Network traffic analysis
search "compartment-ocid/vcn-flow-logs" |
eval totalBytes = data.bytesIn + data.bytesOut |
stats sum(totalBytes) as traffic by data.sourceAddress, data.destinationAddress |
where traffic > 1000000000 |
sort by traffic desc

# API error rate
search "compartment-ocid/api-gateway-logs" |
stats count() as total, count_if(data.statusCode >= 400) as errors by minute(datetime) |
eval error_rate = (errors / total) * 100 |
where error_rate > 5
```

### Log Analytics

**Log Group Management**:
```bash
# Create log group
oci logging log-group create \
  --compartment-id $COMPARTMENT_ID \
  --display-name "production-logs" \
  --description "Production environment logs"

# List log groups
oci logging log-group list \
  --compartment-id $COMPARTMENT_ID

# Update retention
oci logging log update \
  --log-id $LOG_ID \
  --retention-duration 90
```

**Log Archival**:
```bash
# Archive logs to Object Storage
oci sch service-connector create \
  --compartment-id $COMPARTMENT_ID \
  --display-name "logs-to-storage" \
  --source '{
    "kind":"logging",
    "logSources":[{
      "compartmentId":"'$COMPARTMENT_ID'",
      "logGroupId":"'$LOG_GROUP_ID'"
    }]
  }' \
  --target '{
    "kind":"objectStorage",
    "bucketName":"log-archive",
    "namespace":"'$NAMESPACE'",
    "objectNamePrefix":"logs/",
    "batchRolloverSizeInMBs":100,
    "batchRolloverTimeInMs":600000
  }'
```

---

## Troubleshooting

### Instance Troubleshooting

**Instance Not Starting**:
```bash
# Check instance state
oci compute instance get \
  --instance-id $INSTANCE_ID \
  --query 'data.lifecycle-state'

# Check console history
oci compute instance get-console-history \
  --instance-id $INSTANCE_ID

# Check instance console
oci compute instance-console-connection create \
  --instance-id $INSTANCE_ID \
  --ssh-public-key-file ~/.ssh/id_rsa.pub

# Connect to console
ssh -o ProxyCommand='ssh -i ~/.ssh/id_rsa -W %h:%p -p 443 \
  ocid1.instanceconsoleconnection...@instance-console.region.oraclecloud.com' \
  ocid1.instance...
```

**Network Connectivity Issues**:
```bash
# Check VCN flow logs
search "compartment-ocid/vcn-flow-logs" |
where data.destinationAddress = 'instance-private-ip' |
where data.action = 'REJECT' |
sort by datetime desc |
limit 100

# Check security lists
oci network security-list get \
  --security-list-id $SECURITY_LIST_ID

# Check NSG rules
oci network nsg list-network-security-group-security-rules \
  --network-security-group-id $NSG_ID

# Test from instance (via console connection)
ping 8.8.8.8
telnet api.example.com 443
curl -v https://objectstorage.region.oraclecloud.com
```

**High CPU Investigation**:
```bash
# Check metrics
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_computeagent" \
  --query-text 'CpuUtilization[1m]{resourceId="'$INSTANCE_ID'"}.mean()' \
  --start-time "2024-10-10T14:00:00Z" \
  --end-time "2024-10-10T15:00:00Z"

# SSH to instance and investigate
ssh instance-ip
top -c  # Check processes
htop    # Better process viewer
iostat -x 1  # Check I/O
vmstat 1     # Check memory
```

### Storage Troubleshooting

**Volume Not Accessible**:
```bash
# Check volume state
oci bv volume get \
  --volume-id $VOLUME_ID \
  --query 'data.lifecycle-state'

# Check attachment
oci compute volume-attachment list \
  --compartment-id $COMPARTMENT_ID \
  --instance-id $INSTANCE_ID

# Reattach if needed
oci compute volume-attachment attach \
  --instance-id $INSTANCE_ID \
  --volume-id $VOLUME_ID \
  --type iscsi

# On instance, reconnect iSCSI
sudo iscsiadm -m node -T <iqn> -p <ip>:3260 -l
```

**Performance Issues**:
```bash
# Check volume metrics
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_blockstore" \
  --query-text 'VolumeReadOps[1m]{resourceId="'$VOLUME_ID'"}.sum()'

# Check VPUs
oci bv volume get \
  --volume-id $VOLUME_ID \
  --query 'data."vpus-per-gb"'

# Increase performance if needed
oci bv volume update \
  --volume-id $VOLUME_ID \
  --vpus-per-gb 20
```

### Network Troubleshooting

**Load Balancer Issues**:
```bash
# Check health status
oci lb backend-health get \
  --load-balancer-id $LB_ID \
  --backend-set-name "backend-set" \
  --backend-name "10.0.1.5:80"

# Check access logs
search "compartment-ocid/lb-access-logs" |
where data.statusCode >= 500 |
sort by datetime desc |
limit 100

# Check backend connectivity
# From LB subnet, test backend
telnet backend-ip backend-port
curl -v http://backend-ip:backend-port/health
```

**VPN Connectivity**:
```bash
# Check IPSec status
oci network ip-sec-connection get-status \
  --ipsc-id $IPSEC_ID

# Check tunnel status
oci network ip-sec-connection-tunnel get \
  --ipsc-id $IPSEC_ID \
  --tunnel-id $TUNNEL_ID

# View VPN metrics
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_vpn" \
  --query-text 'ConnectionState[1m]{resourceId="'$IPSEC_ID'"}.mean()'
```

### Database Troubleshooting

**Connection Issues**:
```bash
# Check database state
oci db database get \
  --database-id $DATABASE_ID \
  --query 'data.lifecycle-state'

# Test connectivity (from app server)
tnsping database-host:1521/service_name
sqlplus user/password@database-host:1521/service_name

# Check database metrics
oci monitoring metric-data summarize-metrics-data \
  --namespace "oci_database" \
  --query-text 'ActiveSessions[1m]{resourceId="'$DATABASE_ID'"}.mean()'
```

**Performance Issues**:
```sql
-- Check active sessions
SELECT sid, serial#, username, status, sql_id
FROM v$session
WHERE status = 'ACTIVE'
AND username IS NOT NULL;

-- Check long-running queries
SELECT sql_id, elapsed_time/1000000 AS elapsed_seconds,
       cpu_time/1000000 AS cpu_seconds,
       executions
FROM v$sql
WHERE elapsed_time > 60000000  -- More than 60 seconds
ORDER BY elapsed_time DESC;

-- Check wait events
SELECT event, total_waits, time_waited
FROM v$system_event
WHERE wait_class != 'Idle'
ORDER BY time_waited DESC
FETCH FIRST 20 ROWS ONLY;
```

---

## Exam Tips

### Critical Concepts

**Monitoring**:
- Metrics: Time-series data (1-min granularity)
- Alarms: Threshold-based alerts
- Namespaces: oci_computeagent, oci_blockstore, oci_lbaas
- States: OK, FIRING, RESET

**Logging**:
- Service logs: Automatic (VCN flow, LB, audit)
- Custom logs: Application-generated
- Retention: 30 days default, configurable
- Search: Query language for analysis

**Troubleshooting**:
- Instance: Console history, console connection
- Network: Flow logs, security lists/NSGs
- Storage: Check state, metrics, reattach
- Database: Connection test, performance queries

### Common Scenarios

**Q: Alert when CPU > 80%?**
A: Create alarm with query: CpuUtilization[1m].mean() > 80

**Q: Find failed SSH attempts?**
A: Search VCN flow logs for destination port 22 with REJECT action

**Q: Instance not accessible via SSH?**
A: Check security lists/NSGs, check instance console, check route tables

**Q: Slow database queries?**
A: Check Active Sessions metric, query v$sql for long-running queries

**Q: Archive logs for 1 year?**
A: Service Connector Hub to Object Storage, set lifecycle policy

### Quick Reference

**Metric Query Format**:
```
MetricName[interval]{dimensions}.statistic()

Intervals: 1m, 5m, 1h
Statistics: mean(), max(), min(), sum(), count()
Dimensions: resourceId, availabilityDomain, etc.
```

**Alarm Severity Levels**:
- CRITICAL: Immediate action
- ERROR: Urgent attention
- WARNING: Investigation needed
- INFO: Informational

**Troubleshooting Checklist**:
- [ ] Check resource state
- [ ] Check metrics (last 1 hour)
- [ ] Check logs (last 1 hour)
- [ ] Check connectivity (network path)
- [ ] Check security rules
- [ ] Test from different location

---

## Summary

**Monitoring**: Metrics, alarms, custom metrics, dashboards

**Logging**: Service logs, custom logs, search, archive

**Troubleshooting**: Systematic approach, logs/metrics, connectivity tests

**Tools**: OCI Console, CLI, API, Log Analytics

---

**Next**: Security Operations and Cost Management
