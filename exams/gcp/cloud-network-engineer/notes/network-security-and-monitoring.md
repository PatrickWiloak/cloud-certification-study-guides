# Network Security and Monitoring - GCP Professional Cloud Network Engineer

## Overview

Network security architecture, firewall management, Cloud Armor, network monitoring, troubleshooting, and security best practices.

## Firewall Management

### Hierarchical Firewall Policies
**Organization/Folder Level**:
```bash
# Create firewall policy
gcloud compute firewall-policies create corporate-policy \
  --organization=ORG_ID

# Add rule to deny egress to specific IPs
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=corporate-policy \
  --action=deny \
  --direction=egress \
  --dest-ranges=198.51.100.0/24 \
  --layer4-configs=all

# Associate with folder
gcloud compute firewall-policies associations create \
  --firewall-policy=corporate-policy \
  --folder=FOLDER_ID
```

### VPC Firewall Rules
**Best Practices**:
```bash
# Deny all ingress baseline
gcloud compute firewall-rules create deny-all-ingress \
  --network=vpc \
  --action=deny \
  --direction=ingress \
  --rules=all \
  --priority=65534

# Allow SSH from IAP
gcloud compute firewall-rules create allow-iap-ssh \
  --network=vpc \
  --allow=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --target-service-accounts=bastion-sa@project.iam.gserviceaccount.com

# Allow internal communication
gcloud compute firewall-rules create allow-internal \
  --network=vpc \
  --allow=tcp:0-65535,udp:0-65535,icmp \
  --source-ranges=10.0.0.0/8
```

### Network Tags vs Service Accounts
**Network Tags**: Legacy, less secure
**Service Accounts**: Recommended, more secure

```bash
# Using service account targeting
gcloud compute firewall-rules create allow-web \
  --network=vpc \
  --allow=tcp:80,tcp:443 \
  --target-service-accounts=web-sa@project.iam.gserviceaccount.com \
  --source-ranges=0.0.0.0/0
```

## Cloud Armor

### Security Policies
**Rate Limiting**:
```bash
gcloud compute security-policies create rate-limit-policy

# Rate limit rule
gcloud compute security-policies rules create 100 \
  --security-policy=rate-limit-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=1000 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-403
```

**Geo-blocking**:
```bash
gcloud compute security-policies rules create 200 \
  --security-policy=geo-policy \
  --expression="origin.region_code == 'CN' || origin.region_code == 'RU'" \
  --action=deny-403
```

**OWASP Protection**:
```bash
# SQL injection protection
gcloud compute security-policies rules create 300 \
  --security-policy=waf-policy \
  --expression="evaluatePreconfiguredExpr('sqli-v33-stable')" \
  --action=deny-403

# XSS protection
gcloud compute security-policies rules create 400 \
  --security-policy=waf-policy \
  --expression="evaluatePreconfiguredExpr('xss-v33-stable')" \
  --action=deny-403
```

### Custom Rules
```bash
# Block specific user agents
gcloud compute security-policies rules create 500 \
  --security-policy=custom-policy \
  --expression="request.headers['user-agent'].contains('badbot')" \
  --action=deny-403

# Whitelist specific IPs
gcloud compute security-policies rules create 50 \
  --security-policy=custom-policy \
  --expression="inIpRange(origin.ip, '203.0.113.0/24')" \
  --action=allow
```

## Network Monitoring

### VPC Flow Logs
**Configuration**:
```bash
# Enable flow logs
gcloud compute networks subnets update subnet-name \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-5-sec \
  --logging-flow-sampling=0.5 \
  --logging-metadata=include-all \
  --region=us-central1
```

**Analysis**:
```sql
-- Top talkers
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  SUM(CAST(jsonPayload.bytes_sent AS INT64)) as total_bytes
FROM `project.dataset.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY src_ip, dest_ip
ORDER BY total_bytes DESC
LIMIT 10;

-- Detect port scanning
SELECT
  jsonPayload.connection.src_ip,
  COUNT(DISTINCT jsonPayload.connection.dest_port) as unique_ports
FROM `project.dataset.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY src_ip
HAVING unique_ports > 100
ORDER BY unique_ports DESC;
```

### Packet Mirroring
**Use Case**: Deep packet inspection, IDS/IPS

```bash
# Create packet mirroring policy
gcloud compute packet-mirrorings create mirror-policy \
  --region=us-central1 \
  --network=vpc \
  --collector-ilb=projects/PROJECT/regions/us-central1/forwardingRules/collector-ilb \
  --mirrored-subnets=subnet1,subnet2
```

### Network Intelligence Center
**Features**:
- Connectivity Tests
- Performance Dashboard
- Firewall Insights
- Network Topology

```bash
# Run connectivity test
gcloud network-management connectivity-tests create test-web-to-db \
  --source-instance=web-instance \
  --destination-instance=db-instance \
  --protocol=TCP \
  --destination-port=3306
```

### Cloud Monitoring for Networks
```python
from google.cloud import monitoring_v3

def create_network_alert(project_id):
    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    alert_policy = monitoring_v3.AlertPolicy(
        display_name="High Network Egress",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Network egress > 10 GB/hour",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="compute.googleapis.com/instance/network/sent_bytes_count"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=10 * 1024 * 1024 * 1024,  # 10 GB
                    duration={"seconds": 3600},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 300},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                        )
                    ],
                ),
            )
        ],
        notification_channels=[CHANNEL_ID],
    )

    policy = client.create_alert_policy(name=project_name, alert_policy=alert_policy)
    return policy
```

## Troubleshooting

### Connectivity Issues
**Tools**:
1. Connectivity Tests (Network Intelligence Center)
2. VPC Flow Logs
3. Firewall Insights
4. Cloud Logging

**Common Checks**:
```bash
# Check firewall rules
gcloud compute firewall-rules list --filter="network:vpc-name"

# Check routes
gcloud compute routes list --filter="network:vpc-name"

# Check instance network
gcloud compute instances describe INSTANCE --zone=ZONE \
  --format="get(networkInterfaces[0])"

# Test connectivity
gcloud compute ssh INSTANCE --zone=ZONE -- curl -v http://target:80
```

### Latency Issues
```sql
-- Analyze RTT from Flow Logs
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  AVG(CAST(jsonPayload.rtt_msec AS INT64)) as avg_rtt_ms,
  MAX(CAST(jsonPayload.rtt_msec AS INT64)) as max_rtt_ms
FROM `project.dataset.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY src_ip, dest_ip
HAVING avg_rtt_ms > 100
ORDER BY avg_rtt_ms DESC;
```

### Security Incidents
```bash
# Check Cloud Armor logs
gcloud logging read "resource.type=http_load_balancer AND
  jsonPayload.enforcedSecurityPolicy.name:*" \
  --limit=50 \
  --format=json

# Analyze blocked requests
SELECT
  httpRequest.remoteIp,
  COUNT(*) as blocked_count,
  jsonPayload.enforcedSecurityPolicy.name as policy
FROM `project.dataset.requests`
WHERE jsonPayload.enforcedSecurityPolicy.outcome = "DENY"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
GROUP BY remoteIp, policy
ORDER BY blocked_count DESC;
```

## Best Practices

### Security
1. Use hierarchical firewall policies
2. Implement least privilege firewall rules
3. Use service account targeting
4. Enable VPC Flow Logs
5. Configure Cloud Armor for public services
6. Regular firewall audits
7. Monitor for anomalies
8. Implement DDoS protection

### Monitoring
1. Enable comprehensive logging
2. Set up proactive alerts
3. Regular connectivity testing
4. Analyze traffic patterns
5. Monitor security events
6. Document baseline behavior
7. Automate incident response

### Performance
1. Monitor latency metrics
2. Analyze packet loss
3. Optimize routing
4. Use appropriate network tier
5. Regional placement for latency
6. Connection pooling
7. Load balancer health checks

## Study Tips

1. Practice firewall rule configuration
2. Implement Cloud Armor policies
3. Analyze VPC Flow Logs
4. Use Network Intelligence Center
5. Troubleshoot connectivity issues
6. Monitor network performance
7. Security incident response

## Additional Resources

- [VPC Firewall Rules](https://cloud.google.com/vpc/docs/firewalls)
- [Cloud Armor](https://cloud.google.com/armor/docs)
- [VPC Flow Logs](https://cloud.google.com/vpc/docs/flow-logs)
- [Network Intelligence Center](https://cloud.google.com/network-intelligence-center/docs)
- [Network Troubleshooting](https://cloud.google.com/architecture/best-practices-for-troubleshooting-networks)
