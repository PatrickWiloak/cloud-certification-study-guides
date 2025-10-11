# Security Operations - GCP Professional Cloud Security Engineer

## Overview

Security monitoring, incident response, vulnerability management, and operational security practices for GCP environments. This comprehensive guide covers Cloud Logging, Security Command Center, Cloud Armor, incident response, threat detection, and compliance operations.

## Cloud Logging and Monitoring

### Cloud Audit Logs

GCP provides four types of audit logs:
- **Admin Activity**: API calls that modify configuration (always enabled, no charge)
- **Data Access**: API calls that read/write user data (disabled by default, charges apply)
- **System Event**: GCP-initiated actions (always enabled, no charge)
- **Policy Denied**: Access denied due to security policy violations (always enabled, no charge)

```bash
# Enable Data Access logs for a project
gcloud logging sinks create data-access-logs \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/audit_logs \
  --log-filter='logName:"cloudaudit.googleapis.com%2Fdata_access"'

# View Admin Activity logs
gcloud logging read "logName:cloudaudit.googleapis.com%2Factivity" \
  --limit 50 \
  --format json

# View Policy Denied logs
gcloud logging read "logName:cloudaudit.googleapis.com%2Fpolicy" \
  --project=PROJECT_ID \
  --freshness=7d

# Export specific audit logs to Cloud Storage
gcloud logging sinks create security-logs-bucket \
  storage.googleapis.com/security-audit-logs-bucket \
  --log-filter='protoPayload.@type="type.googleapis.com/google.cloud.audit.AuditLog" AND severity>=ERROR'

# Create log sink for BigQuery analysis
gcloud logging sinks create audit-logs-bq \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/security_logs \
  --log-filter='logName:("cloudaudit.googleapis.com" OR "securitycenter.googleapis.com")'
```

### Security Monitoring Queries

```sql
-- Detect privilege escalation attempts
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.resourceName,
  protoPayload.methodName,
  protoPayload.authorizationInfo,
  protoPayload.serviceData_v1_iam.policyDelta
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.methodName IN (
  "google.iam.admin.v1.SetIamPolicy",
  "SetIamPolicy",
  "google.iam.admin.v1.CreateRole",
  "google.iam.admin.v1.UpdateRole"
)
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
ORDER BY timestamp DESC;

-- Detect suspicious resource deletion
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  resource.type,
  protoPayload.resourceName,
  protoPayload.methodName,
  protoPayload.status.code,
  protoPayload.status.message
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.methodName LIKE "%.delete"
  OR protoPayload.methodName LIKE "%.remove"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;

-- Track service account key creation
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.request.name,
  protoPayload.methodName,
  protoPayload.response.keyType,
  protoPayload.response.keyAlgorithm
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.methodName = "google.iam.admin.v1.CreateServiceAccountKey"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
ORDER BY timestamp DESC;

-- Detect multiple failed authentication attempts
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  COUNT(*) as failed_attempts,
  ARRAY_AGG(protoPayload.methodName) as methods,
  ARRAY_AGG(DISTINCT httpRequest.remoteIp) as source_ips
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.status.code != 0
  AND protoPayload.status.message LIKE "%PERMISSION_DENIED%"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY
  timestamp,
  protoPayload.authenticationInfo.principalEmail
HAVING failed_attempts > 5
ORDER BY failed_attempts DESC;

-- Monitor VPC firewall rule changes
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.resourceName,
  protoPayload.methodName,
  JSON_EXTRACT(protoPayload.request, '$.allowed') as allowed_rules,
  JSON_EXTRACT(protoPayload.request, '$.sourceRanges') as source_ranges
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE resource.type = "gce_firewall_rule"
  AND protoPayload.methodName IN (
    "v1.compute.firewalls.insert",
    "v1.compute.firewalls.patch",
    "v1.compute.firewalls.update",
    "v1.compute.firewalls.delete"
  )
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;

-- Track Cloud Storage bucket permission changes
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.resourceName,
  protoPayload.methodName,
  JSON_EXTRACT(protoPayload.serviceData, '$.policyDelta.bindingDeltas') as changes
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE resource.type = "gcs_bucket"
  AND protoPayload.methodName = "storage.setIamPermissions"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;

-- Detect public bucket exposure
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.resourceName,
  JSON_EXTRACT_SCALAR(binding, '$.role') as role,
  JSON_EXTRACT_SCALAR(binding, '$.members') as members
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`,
  UNNEST(JSON_EXTRACT_ARRAY(protoPayload.serviceData, '$.policyDelta.bindingDeltas')) as binding
WHERE resource.type = "gcs_bucket"
  AND (
    JSON_EXTRACT_SCALAR(binding, '$.members') LIKE '%allUsers%'
    OR JSON_EXTRACT_SCALAR(binding, '$.members') LIKE '%allAuthenticatedUsers%'
  )
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
ORDER BY timestamp DESC;
```

### Log-Based Metrics

```bash
# Create log-based metric for failed SSH attempts
gcloud logging metrics create failed_ssh_attempts \
  --description="Count of failed SSH login attempts" \
  --log-filter='resource.type="gce_instance"
    AND logName="projects/PROJECT_ID/logs/syslog"
    AND textPayload=~"Failed password for"'

# Create metric for privilege escalation
gcloud logging metrics create privilege_escalation \
  --description="IAM policy changes granting admin roles" \
  --log-filter='protoPayload.methodName="SetIamPolicy"
    AND protoPayload.serviceData.policyDelta.bindingDeltas.role=~"roles/.*admin"
    AND protoPayload.serviceData.policyDelta.bindingDeltas.action="ADD"'

# Create metric for firewall rule changes
gcloud logging metrics create firewall_changes \
  --description="VPC firewall rule modifications" \
  --log-filter='resource.type="gce_firewall_rule"
    AND (protoPayload.methodName="v1.compute.firewalls.insert"
      OR protoPayload.methodName="v1.compute.firewalls.patch"
      OR protoPayload.methodName="v1.compute.firewalls.delete")'

# Create metric for sensitive data access
gcloud logging metrics create sensitive_data_access \
  --description="Access to buckets containing sensitive data" \
  --log-filter='resource.type="gcs_bucket"
    AND protoPayload.resourceName=~".*sensitive.*"
    AND protoPayload.methodName="storage.objects.get"'

# List all log-based metrics
gcloud logging metrics list

# Update an existing metric
gcloud logging metrics update failed_ssh_attempts \
  --description="Updated description for failed SSH attempts"

# Delete a metric
gcloud logging metrics delete old_metric_name
```

### Alerting Policies

```bash
# Create alerting policy for failed authentication
cat > failed-auth-policy.yaml <<EOF
displayName: "Multiple Failed Authentication Attempts"
conditions:
  - displayName: "Failed auth threshold exceeded"
    conditionThreshold:
      filter: 'metric.type="logging.googleapis.com/user/failed_ssh_attempts" resource.type="gce_instance"'
      comparison: COMPARISON_GT
      thresholdValue: 10
      duration: 300s
      aggregations:
        - alignmentPeriod: 60s
          perSeriesAligner: ALIGN_RATE
notificationChannels:
  - projects/PROJECT_ID/notificationChannels/CHANNEL_ID
alertStrategy:
  autoClose: 604800s
documentation:
  content: "Multiple failed SSH authentication attempts detected. Potential brute force attack. Investigate source IPs and consider blocking."
  mimeType: text/markdown
enabled: true
EOF

gcloud alpha monitoring policies create --policy-from-file=failed-auth-policy.yaml

# Create policy for privilege escalation
cat > privilege-escalation-policy.yaml <<EOF
displayName: "Privilege Escalation Detected"
conditions:
  - displayName: "Admin role granted"
    conditionThreshold:
      filter: 'metric.type="logging.googleapis.com/user/privilege_escalation" resource.type="audited_resource"'
      comparison: COMPARISON_GT
      thresholdValue: 0
      duration: 60s
notificationChannels:
  - projects/PROJECT_ID/notificationChannels/EMAIL_CHANNEL
  - projects/PROJECT_ID/notificationChannels/PAGERDUTY_CHANNEL
alertStrategy:
  autoClose: 86400s
  notificationRateLimit:
    period: 300s
documentation:
  content: |
    # Privilege Escalation Alert

    An admin role has been granted. Immediate investigation required:
    1. Verify the principal who made the change
    2. Check if the change was authorized
    3. Review the scope of permissions granted
    4. Consider revoking if unauthorized
  mimeType: text/markdown
severity: CRITICAL
enabled: true
EOF

gcloud alpha monitoring policies create --policy-from-file=privilege-escalation-policy.yaml

# Create notification channel
gcloud alpha monitoring channels create \
  --display-name="Security Team Email" \
  --type=email \
  --channel-labels=email_address=security@example.com

gcloud alpha monitoring channels create \
  --display-name="Security PagerDuty" \
  --type=pagerduty \
  --channel-labels=service_key=YOUR_PAGERDUTY_KEY

# List all alerting policies
gcloud alpha monitoring policies list

# Update alerting policy
gcloud alpha monitoring policies update POLICY_ID \
  --update-notification-channels=projects/PROJECT_ID/notificationChannels/NEW_CHANNEL

# Delete alerting policy
gcloud alpha monitoring policies delete POLICY_ID
```

### Log Sinks and Routing

```bash
# Create sink to route all audit logs to BigQuery
gcloud logging sinks create all-audit-logs-bq \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/audit_logs \
  --log-filter='logName:cloudaudit.googleapis.com'

# Create sink for security-critical logs to Cloud Storage
gcloud logging sinks create security-critical-logs \
  storage.googleapis.com/security-logs-archive \
  --log-filter='severity>=ERROR OR logName:cloudaudit.googleapis.com'

# Create sink to route logs to Pub/Sub for real-time processing
gcloud logging sinks create realtime-security-events \
  pubsub.googleapis.com/projects/PROJECT_ID/topics/security-events \
  --log-filter='protoPayload.methodName=~".*delete.*" OR severity=CRITICAL'

# Create organization-level sink
gcloud logging sinks create org-wide-security-logs \
  bigquery.googleapis.com/projects/CENTRAL_PROJECT/datasets/org_security_logs \
  --organization=ORG_ID \
  --include-children \
  --log-filter='logName:cloudaudit.googleapis.com OR logName:securitycenter.googleapis.com'

# Create folder-level sink
gcloud logging sinks create folder-audit-logs \
  storage.googleapis.com/folder-audit-logs \
  --folder=FOLDER_ID \
  --include-children \
  --log-filter='logName:cloudaudit.googleapis.com'

# Update sink filter
gcloud logging sinks update all-audit-logs-bq \
  --log-filter='logName:cloudaudit.googleapis.com AND severity>=WARNING'

# Grant sink service account permissions
# Get the sink's service account
SINK_SA=$(gcloud logging sinks describe all-audit-logs-bq --format='value(writerIdentity)')

# Grant BigQuery Data Editor role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="$SINK_SA" \
  --role="roles/bigquery.dataEditor"

# List all sinks
gcloud logging sinks list

# Delete sink
gcloud logging sinks delete old-sink-name
```

### Notification Channels

```bash
# Create email notification channel
gcloud alpha monitoring channels create \
  --display-name="Security Team" \
  --type=email \
  --channel-labels=email_address=security-team@example.com \
  --enabled

# Create Slack notification channel
gcloud alpha monitoring channels create \
  --display-name="Security Slack" \
  --type=slack \
  --channel-labels=url=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# Create PagerDuty notification channel
gcloud alpha monitoring channels create \
  --display-name="Security On-Call" \
  --type=pagerduty \
  --channel-labels=service_key=YOUR_PAGERDUTY_SERVICE_KEY

# Create webhook notification channel
gcloud alpha monitoring channels create \
  --display-name="Security Webhook" \
  --type=webhook_tokenauth \
  --channel-labels=url=https://your-webhook-endpoint.com/alerts

# Create SMS notification channel
gcloud alpha monitoring channels create \
  --display-name="Security SMS" \
  --type=sms \
  --channel-labels=number=+15551234567

# List all notification channels
gcloud alpha monitoring channels list

# Describe a specific channel
gcloud alpha monitoring channels describe CHANNEL_ID

# Update notification channel
gcloud alpha monitoring channels update CHANNEL_ID \
  --update-channel-labels=email_address=new-security@example.com

# Delete notification channel
gcloud alpha monitoring channels delete CHANNEL_ID
```

## Security Command Center (SCC)

### Overview

Security Command Center is GCP's centralized security and risk management platform. It provides asset discovery, vulnerability detection, threat detection, and security analytics.

**SCC Tiers:**
- **Standard (Free)**: Asset discovery, basic security findings
- **Premium**: Advanced threat detection, Event Threat Detection, Container Threat Detection, Security Health Analytics, compliance monitoring

### Asset Discovery and Management

```bash
# Enable Security Command Center API
gcloud services enable securitycenter.googleapis.com

# List all assets in organization
gcloud scc assets list ORGANIZATION_ID \
  --format="table(asset.securityCenterProperties.resourceName,asset.securityCenterProperties.resourceType,asset.createTime)"

# List assets of specific type
gcloud scc assets list ORGANIZATION_ID \
  --filter="securityCenterProperties.resourceType=\"google.compute.Instance\""

# List assets with specific security marks
gcloud scc assets list ORGANIZATION_ID \
  --filter="securityMarks.marks.environment=\"production\""

# Search for assets by project
gcloud scc assets list ORGANIZATION_ID \
  --filter="securityCenterProperties.resourceParent=\"//cloudresourcemanager.googleapis.com/projects/PROJECT_NUMBER\""

# Export assets to BigQuery
gcloud scc assets list ORGANIZATION_ID \
  --format=json > assets-export.json

# Count assets by type
gcloud scc assets list ORGANIZATION_ID \
  --format="value(asset.securityCenterProperties.resourceType)" | \
  sort | uniq -c | sort -rn
```

### Security Marks

Security marks are custom key-value tags for assets and findings that help with organization, filtering, and automation.

```bash
# Add security marks to an asset
gcloud scc assets update-marks ASSET_NAME \
  --organization=ORGANIZATION_ID \
  --security-marks="environment=production,data-classification=confidential,owner=security-team"

# Add marks to a finding
gcloud scc findings update-marks FINDING_NAME \
  --organization=ORGANIZATION_ID \
  --security-marks="false-positive=true,ticket=JIRA-123"

# List assets with specific marks
gcloud scc assets list ORGANIZATION_ID \
  --filter="securityMarks.marks.environment=\"production\" AND securityMarks.marks.data-classification=\"confidential\""

# Remove specific security marks
gcloud scc assets update-marks ASSET_NAME \
  --organization=ORGANIZATION_ID \
  --update-mask="marks.environment" \
  --security-marks=""

# Bulk update security marks using script
for asset in $(gcloud scc assets list ORG_ID --filter="resourceType:Instance" --format="value(name)"); do
  gcloud scc assets update-marks $asset \
    --organization=ORG_ID \
    --security-marks="auto-tagged=true,scan-date=$(date +%Y-%m-%d)"
done
```

### Findings Management

```bash
# List all active findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\"" \
  --format="table(finding.category,finding.severity,finding.resourceName)"

# List high and critical severity findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\" AND (severity=\"HIGH\" OR severity=\"CRITICAL\")"

# List findings for specific category
gcloud scc findings list ORGANIZATION_ID \
  --filter="category=\"OPEN_FIREWALL\""

# List findings for specific project
gcloud scc findings list ORGANIZATION_ID \
  --filter="resourceName:\"projects/PROJECT_ID\""

# Update finding state
gcloud scc findings update FINDING_NAME \
  --organization=ORGANIZATION_ID \
  --state=INACTIVE

# Set finding as false positive
gcloud scc findings update-marks FINDING_NAME \
  --organization=ORGANIZATION_ID \
  --security-marks="false-positive=true,reviewed-by=analyst@example.com,review-date=$(date +%Y-%m-%d)"

# Create custom finding
gcloud scc findings create FINDING_ID \
  --organization=ORGANIZATION_ID \
  --source=SOURCE_NAME \
  --category="CUSTOM_SECURITY_ISSUE" \
  --state=ACTIVE \
  --resource-name="//compute.googleapis.com/projects/PROJECT/zones/us-central1-a/instances/instance-1" \
  --event-time="2024-01-01T12:00:00Z" \
  --severity=HIGH

# Export findings to file
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\"" \
  --format=json > active-findings.json

# Group findings by category
gcloud scc findings list ORGANIZATION_ID \
  --format="value(finding.category)" | \
  sort | uniq -c | sort -rn
```

### Security Health Analytics

Security Health Analytics automatically detects common misconfigurations and vulnerabilities.

**Common Finding Categories:**
- OPEN_FIREWALL: Overly permissive firewall rules
- PUBLIC_IP_ADDRESS: Resources with public IPs
- WEAK_SSL_POLICY: Weak SSL/TLS configurations
- ADMIN_SERVICE_ACCOUNT: Service accounts with excessive permissions
- PUBLIC_BUCKET_ACL: Publicly accessible storage buckets
- LEGACY_AUTHORIZATION_ENABLED: GKE clusters using legacy auth
- WEB_UI_ENABLED: GKE dashboard enabled
- OPEN_RDSP: RDP port (3389) exposed
- OPEN_SSH: SSH port (22) exposed to internet

```bash
# List all Security Health Analytics findings
gcloud scc findings list ORGANIZATION_ID \
  --source=ORGANIZATION_ID/sources/SECURITY_HEALTH_ANALYTICS_SOURCE_ID

# List open firewall findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="category=\"OPEN_FIREWALL\" AND state=\"ACTIVE\"" \
  --format="table(finding.resourceName,finding.externalUri,finding.category)"

# List publicly accessible resources
gcloud scc findings list ORGANIZATION_ID \
  --filter="(category=\"PUBLIC_BUCKET_ACL\" OR category=\"PUBLIC_IP_ADDRESS\") AND state=\"ACTIVE\""

# List weak SSL policy findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="category=\"WEAK_SSL_POLICY\" AND state=\"ACTIVE\""

# Enable custom modules (Premium tier)
gcloud scc custom-modules sha create \
  --organization=ORGANIZATION_ID \
  --display-name="Check for unencrypted disks" \
  --enablement-state=ENABLED \
  --custom-config-from-file=custom-config.yaml
```

### Event Threat Detection

Event Threat Detection (Premium tier) identifies threats and anomalies in Cloud Logging.

**Detection Categories:**
- Malware: Bad domain, bad IP
- Data exfiltration: Unusual data access patterns
- Brute force: Multiple failed login attempts
- Privilege escalation: Unauthorized permission grants
- Persistence: Backdoor creation attempts
- Cryptomining: Unauthorized cryptocurrency mining

```bash
# List Event Threat Detection findings
gcloud scc findings list ORGANIZATION_ID \
  --source=ORGANIZATION_ID/sources/EVENT_THREAT_DETECTION_SOURCE_ID

# List malware-related findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="category=\"Malware:*\" AND state=\"ACTIVE\"" \
  --format="table(finding.category,finding.resourceName,finding.severity)"

# List data exfiltration findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="category=\"Exfiltration:*\" AND state=\"ACTIVE\""

# List persistence mechanism findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="category=\"Persistence:*\" AND state=\"ACTIVE\""

# List findings by severity
gcloud scc findings list ORGANIZATION_ID \
  --filter="severity=\"CRITICAL\" AND state=\"ACTIVE\"" \
  --format="table(finding.category,finding.resourceName,finding.eventTime)"

# Configure Event Threat Detection settings
gcloud scc settings services enable \
  --organization=ORGANIZATION_ID \
  --service=EVENT_THREAT_DETECTION

# Disable specific detectors
gcloud scc settings services update \
  --organization=ORGANIZATION_ID \
  --service=EVENT_THREAT_DETECTION \
  --detector=MALWARE_BAD_DOMAIN \
  --enabled=false
```

### SCC Sources

```bash
# List all sources
gcloud scc sources list ORGANIZATION_ID

# Describe a specific source
gcloud scc sources describe SOURCE_NAME \
  --organization=ORGANIZATION_ID

# Create custom source
gcloud scc sources create \
  --organization=ORGANIZATION_ID \
  --display-name="Custom Security Scanner" \
  --description="Internal security scanning tool"

# Update source
gcloud scc sources update SOURCE_NAME \
  --organization=ORGANIZATION_ID \
  --display-name="Updated Scanner Name"

# Get IAM policy for source
gcloud scc sources get-iam-policy SOURCE_NAME \
  --organization=ORGANIZATION_ID

# Grant source editor role
gcloud scc sources add-iam-policy-binding SOURCE_NAME \
  --organization=ORGANIZATION_ID \
  --member="serviceAccount:scanner@project.iam.gserviceaccount.com" \
  --role="roles/securitycenter.findingsEditor"
```

### SCC Notifications

```bash
# Create notification config for critical findings
gcloud scc notifications create critical-findings-notif \
  --organization=ORGANIZATION_ID \
  --description="Notify on critical findings" \
  --pubsub-topic=projects/PROJECT_ID/topics/scc-critical-findings \
  --filter="severity=\"CRITICAL\" AND state=\"ACTIVE\""

# Create notification for specific category
gcloud scc notifications create firewall-notif \
  --organization=ORGANIZATION_ID \
  --description="Notify on open firewall findings" \
  --pubsub-topic=projects/PROJECT_ID/topics/scc-firewall \
  --filter="category=\"OPEN_FIREWALL\" AND state=\"ACTIVE\""

# List all notification configs
gcloud scc notifications list ORGANIZATION_ID

# Update notification config
gcloud scc notifications update critical-findings-notif \
  --organization=ORGANIZATION_ID \
  --filter="severity=\"CRITICAL\" OR severity=\"HIGH\""

# Delete notification config
gcloud scc notifications delete critical-findings-notif \
  --organization=ORGANIZATION_ID

# Test notification by creating test finding
gcloud scc findings create test-finding-$(date +%s) \
  --organization=ORGANIZATION_ID \
  --source=SOURCE_NAME \
  --category="TEST_FINDING" \
  --severity=CRITICAL \
  --state=ACTIVE
```

## Cloud Armor

### Overview

Cloud Armor provides DDoS protection, WAF capabilities, and edge security for applications running behind Google Cloud Load Balancers.

### Security Policies

```bash
# Create a basic security policy
gcloud compute security-policies create web-app-policy \
  --description="Security policy for web application"

# Create policy with default rule action
gcloud compute security-policies create block-by-default \
  --description="Default deny policy" \
  --default-rule-action=deny-403

# List all security policies
gcloud compute security-policies list

# Describe a security policy
gcloud compute security-policies describe web-app-policy

# Update policy description
gcloud compute security-policies update web-app-policy \
  --description="Updated security policy"

# Delete security policy
gcloud compute security-policies delete web-app-policy
```

### Preconfigured WAF Rules

```bash
# Add SQLi (SQL Injection) protection rule
gcloud compute security-policies rules create 1000 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('sqli-stable')" \
  --action=deny-403 \
  --description="Block SQL injection attempts"

# Add XSS (Cross-Site Scripting) protection
gcloud compute security-policies rules create 1001 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('xss-stable')" \
  --action=deny-403 \
  --description="Block XSS attempts"

# Add LFI (Local File Inclusion) protection
gcloud compute security-policies rules create 1002 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('lfi-stable')" \
  --action=deny-403 \
  --description="Block LFI attempts"

# Add RCE (Remote Code Execution) protection
gcloud compute security-policies rules create 1003 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('rce-stable')" \
  --action=deny-403 \
  --description="Block RCE attempts"

# Add RFI (Remote File Inclusion) protection
gcloud compute security-policies rules create 1004 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('rfi-stable')" \
  --action=deny-403 \
  --description="Block RFI attempts"

# Add method enforcement (block non-standard HTTP methods)
gcloud compute security-policies rules create 1005 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('methodenforcement-stable')" \
  --action=deny-403 \
  --description="Block non-standard HTTP methods"

# Add scanner detection
gcloud compute security-policies rules create 1006 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('scannerdetection-stable')" \
  --action=deny-403 \
  --description="Block security scanners"

# Add protocol attack protection
gcloud compute security-policies rules create 1007 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('protocolattack-stable')" \
  --action=deny-403 \
  --description="Block protocol attacks"

# Add PHP injection protection
gcloud compute security-policies rules create 1008 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('php-stable')" \
  --action=deny-403 \
  --description="Block PHP injection attempts"

# Add session fixation protection
gcloud compute security-policies rules create 1009 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('sessionfixation-stable')" \
  --action=deny-403 \
  --description="Block session fixation attempts"
```

### Custom Rules

```bash
# Block specific IP address
gcloud compute security-policies rules create 2000 \
  --security-policy=web-app-policy \
  --expression="origin.ip == '203.0.113.10'" \
  --action=deny-403 \
  --description="Block malicious IP"

# Block IP range
gcloud compute security-policies rules create 2001 \
  --security-policy=web-app-policy \
  --expression="inIpRange(origin.ip, '203.0.113.0/24')" \
  --action=deny-403 \
  --description="Block suspicious IP range"

# Allow only specific countries (geo-blocking)
gcloud compute security-policies rules create 2002 \
  --security-policy=web-app-policy \
  --expression="origin.region_code != 'US' && origin.region_code != 'CA'" \
  --action=deny-403 \
  --description="Allow only US and Canada"

# Block specific user agents
gcloud compute security-policies rules create 2003 \
  --security-policy=web-app-policy \
  --expression="has(request.headers['user-agent']) && request.headers['user-agent'].contains('BadBot')" \
  --action=deny-403 \
  --description="Block known bad bots"

# Block requests without referer header
gcloud compute security-policies rules create 2004 \
  --security-policy=web-app-policy \
  --expression="!has(request.headers['referer'])" \
  --action=deny-403 \
  --description="Block requests without referer"

# Allow only specific paths
gcloud compute security-policies rules create 2005 \
  --security-policy=web-app-policy \
  --expression="!request.path.matches('/api/.*') && !request.path.matches('/public/.*')" \
  --action=deny-403 \
  --description="Block access to non-public paths"

# Rate limit by IP
gcloud compute security-policies rules create 2006 \
  --security-policy=web-app-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP \
  --description="Rate limit: 100 requests per minute per IP"

# Combine multiple conditions
gcloud compute security-policies rules create 2007 \
  --security-policy=web-app-policy \
  --expression="origin.region_code == 'CN' && request.path.matches('/admin/.*')" \
  --action=deny-403 \
  --description="Block admin access from specific regions"

# Preview mode (log only, don't block)
gcloud compute security-policies rules create 2008 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('sqli-stable')" \
  --action=deny-403 \
  --preview \
  --description="SQLi protection in preview mode"
```

### Rate Limiting

```bash
# Rate limit by IP address
gcloud compute security-policies rules create 3000 \
  --security-policy=web-app-policy \
  --action=rate-based-ban \
  --rate-limit-threshold-count=1000 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP \
  --description="Limit to 1000 req/min per IP"

# Rate limit by HTTP header
gcloud compute security-policies rules create 3001 \
  --security-policy=web-app-policy \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=300 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=HTTP-HEADER \
  --enforce-on-key-name=X-API-Key \
  --description="Limit to 100 req/min per API key"

# Rate limit by user ID (from header)
gcloud compute security-policies rules create 3002 \
  --security-policy=web-app-policy \
  --action=rate-based-ban \
  --rate-limit-threshold-count=500 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=HTTP-HEADER \
  --enforce-on-key-name=X-User-ID \
  --description="Limit to 500 req/min per user"

# Rate limit with custom ban threshold
gcloud compute security-policies rules create 3003 \
  --security-policy=web-app-policy \
  --action=rate-based-ban \
  --rate-limit-threshold-count=50 \
  --rate-limit-threshold-interval-sec=10 \
  --ban-duration-sec=1800 \
  --ban-threshold-count=5 \
  --ban-threshold-interval-sec=60 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP \
  --description="Ban after 5 violations in 60 seconds"

# Global rate limit (all traffic)
gcloud compute security-policies rules create 3004 \
  --security-policy=web-app-policy \
  --action=rate-based-ban \
  --rate-limit-threshold-count=10000 \
  --rate-limit-threshold-interval-sec=60 \
  --conform-action=allow \
  --exceed-action=deny-503 \
  --enforce-on-key=ALL \
  --description="Global rate limit: 10k req/min"
```

### Adaptive Protection

```bash
# Enable Adaptive Protection (auto-deploy rules based on ML)
gcloud compute security-policies update web-app-policy \
  --enable-layer7-ddos-defense \
  --layer7-ddos-defense-rule-visibility=STANDARD

# Set Adaptive Protection to premium
gcloud compute security-policies update web-app-policy \
  --enable-layer7-ddos-defense \
  --layer7-ddos-defense-rule-visibility=PREMIUM

# Disable Adaptive Protection
gcloud compute security-policies update web-app-policy \
  --no-enable-layer7-ddos-defense

# Configure alert threshold
gcloud compute security-policies update web-app-policy \
  --layer7-ddos-defense-threshold-count=100 \
  --layer7-ddos-defense-threshold-interval-sec=60
```

### Bot Management

```bash
# Enable reCAPTCHA Enterprise integration
gcloud compute security-policies rules create 4000 \
  --security-policy=web-app-policy \
  --expression="!has(token.recaptcha_session) || token.recaptcha_session.score < 0.5" \
  --action=deny-403 \
  --description="Challenge low-score reCAPTCHA requests"

# Redirect to reCAPTCHA challenge
gcloud compute security-policies rules create 4001 \
  --security-policy=web-app-policy \
  --expression="evaluatePreconfiguredExpr('scannerdetection-stable')" \
  --action=redirect \
  --redirect-type=GOOGLE_RECAPTCHA \
  --description="Challenge suspected bots"

# Set reCAPTCHA site key
gcloud compute security-policies update web-app-policy \
  --recaptcha-site-key=SITE_KEY
```

### Attach Policy to Backend Service

```bash
# Attach security policy to backend service
gcloud compute backend-services update BACKEND_SERVICE \
  --security-policy=web-app-policy \
  --global

# Attach policy to backend bucket
gcloud compute backend-buckets update BACKEND_BUCKET \
  --security-policy=web-app-policy

# Remove security policy
gcloud compute backend-services update BACKEND_SERVICE \
  --security-policy="" \
  --global

# List backend services with security policies
gcloud compute backend-services list \
  --format="table(name,securityPolicy)"
```

### Rule Management

```bash
# List all rules in policy
gcloud compute security-policies rules describe 1000 \
  --security-policy=web-app-policy

# Update rule priority
gcloud compute security-policies rules update 1000 \
  --security-policy=web-app-policy \
  --priority=900

# Update rule expression
gcloud compute security-policies rules update 2000 \
  --security-policy=web-app-policy \
  --expression="origin.ip == '203.0.113.10' || origin.ip == '203.0.113.11'"

# Enable preview mode on rule
gcloud compute security-policies rules update 1000 \
  --security-policy=web-app-policy \
  --preview

# Disable preview mode
gcloud compute security-policies rules update 1000 \
  --security-policy=web-app-policy \
  --no-preview

# Delete rule
gcloud compute security-policies rules delete 1000 \
  --security-policy=web-app-policy

# Export policy configuration
gcloud compute security-policies export web-app-policy \
  --destination=policy-backup.yaml

# Import policy configuration
gcloud compute security-policies import web-app-policy \
  --source=policy-backup.yaml
```

## Incident Response

### Incident Response Framework

**NIST Incident Response Lifecycle:**
1. **Preparation**: Tools, processes, training
2. **Detection and Analysis**: Identify and investigate
3. **Containment, Eradication, and Recovery**: Stop, remove, restore
4. **Post-Incident Activity**: Lessons learned, improvements

### Detection Methods

```bash
# Monitor for security alerts in real-time
gcloud logging read "severity>=ERROR AND logName:cloudaudit" \
  --format=json \
  --freshness=5m

# Set up real-time log streaming
gcloud logging tail "severity>=WARNING" --format=json

# Query for suspicious activities
gcloud logging read '
  protoPayload.methodName=~".*delete.*" OR
  protoPayload.methodName="SetIamPolicy" OR
  protoPayload.methodName="CreateServiceAccountKey"
' --limit=100 --format=json

# Check Security Command Center for active findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\" AND severity=\"CRITICAL\""

# Monitor Cloud Armor logs for attacks
gcloud logging read 'resource.type="http_load_balancer" AND jsonPayload.enforcedSecurityPolicy.name!=""' \
  --limit=100 \
  --format=json
```

### Investigation with Cloud Logging

```bash
# Investigate user activity during incident window
gcloud logging read '
  protoPayload.authenticationInfo.principalEmail="suspicious-user@example.com"
  AND timestamp>="2024-01-01T10:00:00Z"
  AND timestamp<="2024-01-01T12:00:00Z"
' --format=json --limit=1000

# Trace specific resource changes
gcloud logging read '
  protoPayload.resourceName="//compute.googleapis.com/projects/PROJECT/zones/us-central1-a/instances/compromised-instance"
' --format=json

# Investigate IAM policy changes
gcloud logging read '
  protoPayload.methodName="SetIamPolicy"
  AND timestamp>="2024-01-01T00:00:00Z"
' --format=json

# Find all actions from specific IP
gcloud logging read '
  httpRequest.remoteIp="203.0.113.10"
  AND timestamp>="2024-01-01T00:00:00Z"
' --format=json

# Export comprehensive investigation logs
gcloud logging read '
  timestamp>="2024-01-01T00:00:00Z"
  AND timestamp<="2024-01-02T00:00:00Z"
  AND (severity>=WARNING OR logName:cloudaudit)
' --format=json > incident-investigation.json
```

### Containment Strategies

```bash
# 1. Isolate compromised instance - Remove from backend service
gcloud compute backend-services remove-backend BACKEND_SERVICE \
  --instance-group=INSTANCE_GROUP \
  --instance-group-zone=ZONE \
  --global

# 2. Apply emergency firewall rule to block traffic
gcloud compute firewall-rules create emergency-block-$(date +%s) \
  --direction=INGRESS \
  --priority=100 \
  --network=default \
  --action=DENY \
  --rules=all \
  --source-ranges=0.0.0.0/0 \
  --target-tags=compromised

# Tag the compromised instance
gcloud compute instances add-tags INSTANCE_NAME \
  --zone=ZONE \
  --tags=compromised

# 3. Disable compromised service account
gcloud iam service-accounts disable SERVICE_ACCOUNT_EMAIL

# 4. Delete service account keys
for key in $(gcloud iam service-accounts keys list --iam-account=SERVICE_ACCOUNT_EMAIL --format="value(name)"); do
  gcloud iam service-accounts keys delete $key --iam-account=SERVICE_ACCOUNT_EMAIL --quiet
done

# 5. Revoke IAM permissions immediately
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SERVICE_ACCOUNT_EMAIL" \
  --role="roles/editor"

# 6. Stop compromised instance (preserve for forensics)
gcloud compute instances stop INSTANCE_NAME --zone=ZONE

# 7. Remove external IP address
gcloud compute instances delete-access-config INSTANCE_NAME \
  --zone=ZONE \
  --access-config-name="external-nat"

# 8. Update VPC firewall to restrict access
gcloud compute firewall-rules update FIREWALL_RULE \
  --source-ranges=10.0.0.0/8 \
  --disabled

# 9. Suspend user account
gcloud identity users update USER_EMAIL --suspended

# 10. Revoke all user sessions
gcloud identity users signout USER_EMAIL
```

### Forensics and Evidence Collection

```bash
# Create disk snapshot for forensic analysis
gcloud compute disks snapshot DISK_NAME \
  --snapshot-names=forensic-snapshot-$(date +%Y%m%d-%H%M%S) \
  --zone=ZONE \
  --storage-location=US

# Create snapshot with forensic labels
gcloud compute disks snapshot DISK_NAME \
  --snapshot-names=forensic-snapshot-incident-12345 \
  --zone=ZONE \
  --labels=incident=12345,collected-by=security-team,date=$(date +%Y%m%d)

# Clone disk for offline analysis
gcloud compute disks create forensic-disk-copy \
  --source-snapshot=forensic-snapshot-incident-12345 \
  --zone=us-central1-a

# Create isolated forensic analysis instance
gcloud compute instances create forensic-workstation \
  --zone=us-central1-a \
  --machine-type=n2-standard-8 \
  --network=forensics-network \
  --no-address \
  --disk=name=forensic-disk-copy,mode=ro,boot=no

# Export memory dump (requires instance access)
gcloud compute ssh INSTANCE_NAME --zone=ZONE \
  --command="sudo dd if=/dev/mem of=/tmp/memory-dump.bin bs=1M"

# Export logs for specific time window
gcloud logging read "timestamp>=\"2024-01-01T00:00:00Z\" AND timestamp<=\"2024-01-02T00:00:00Z\"" \
  --format=json \
  --project=PROJECT_ID > forensic-logs-$(date +%Y%m%d).json

# Export audit logs to Cloud Storage for long-term retention
gcloud logging sinks create forensic-evidence-sink \
  storage.googleapis.com/forensic-evidence-bucket \
  --log-filter='timestamp>="2024-01-01T00:00:00Z" AND logName:cloudaudit'

# Capture network traffic logs (VPC Flow Logs)
gcloud logging read 'resource.type="gce_subnetwork" AND logName:vpc_flows' \
  --format=json > network-flows-$(date +%Y%m%d).json

# Export Security Command Center findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="eventTime>=\"2024-01-01T00:00:00Z\"" \
  --format=json > scc-findings-$(date +%Y%m%d).json

# Create forensic image export (requires Boot Disk)
gcloud compute images create forensic-image-$(date +%Y%m%d) \
  --source-disk=DISK_NAME \
  --source-disk-zone=ZONE \
  --family=forensic-images

# Export image to Cloud Storage
gcloud compute images export \
  --destination-uri=gs://forensic-evidence-bucket/forensic-image-$(date +%Y%m%d).tar.gz \
  --image=forensic-image-$(date +%Y%m%d)
```

### Recovery and Remediation

```bash
# 1. Deploy clean replacement instance from trusted image
gcloud compute instances create replacement-instance \
  --image=trusted-golden-image \
  --image-project=PROJECT_ID \
  --zone=us-central1-a

# 2. Restore from clean backup
gcloud compute disks create restored-disk \
  --source-snapshot=pre-incident-backup-snapshot \
  --zone=us-central1-a

# 3. Update firewall rules to normal state
gcloud compute firewall-rules update production-firewall \
  --enabled

gcloud compute firewall-rules delete emergency-block-*

# 4. Create new service account with least privilege
gcloud iam service-accounts create new-service-account \
  --display-name="Replacement Service Account"

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:new-service-account@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/viewer"

# 5. Rotate all potentially compromised credentials
gcloud iam service-accounts keys create new-key.json \
  --iam-account=SERVICE_ACCOUNT_EMAIL

# 6. Update application configuration
gcloud compute instances add-metadata INSTANCE_NAME \
  --zone=ZONE \
  --metadata-from-file=config=new-config.yaml

# 7. Verify security controls
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\" AND severity=\"HIGH\""

# 8. Re-enable monitoring and alerting
gcloud alpha monitoring policies update POLICY_ID --enabled

# 9. Document changes in Cloud Asset Inventory
gcloud asset search-all-resources \
  --query="createTime>2024-01-01T00:00:00Z" \
  --order-by="createTime DESC"
```

### Post-Incident Review

```bash
# Generate incident timeline from logs
gcloud logging read '
  timestamp>="2024-01-01T10:00:00Z"
  AND timestamp<="2024-01-01T12:00:00Z"
  AND (severity>=ERROR OR logName:cloudaudit)
' --format="table(timestamp,resource.type,protoPayload.methodName,protoPayload.authenticationInfo.principalEmail)" \
  --limit=1000 > incident-timeline.txt

# Analyze asset changes during incident
gcloud asset search-all-resources \
  --query="updateTime>=2024-01-01T10:00:00Z AND updateTime<=2024-01-01T12:00:00Z" \
  --format=json > incident-asset-changes.json

# Review IAM changes
gcloud logging read '
  protoPayload.methodName="SetIamPolicy"
  AND timestamp>="2024-01-01T00:00:00Z"
' --format="table(timestamp,protoPayload.authenticationInfo.principalEmail,protoPayload.resourceName)"

# Document lessons learned
# Create incident report with:
# - Root cause analysis
# - Timeline of events
# - Actions taken
# - Gaps identified
# - Recommendations for improvement
```

## Vulnerability Management

### Container Analysis and Vulnerability Scanning

```bash
# Enable Container Analysis API
gcloud services enable containeranalysis.googleapis.com containerscanning.googleapis.com

# Push image to trigger automatic scanning
docker tag myapp:latest gcr.io/PROJECT_ID/myapp:v1.0
docker push gcr.io/PROJECT_ID/myapp:v1.0

# Manual scan trigger
gcloud container images scan gcr.io/PROJECT_ID/myapp:v1.0

# List vulnerabilities for specific image
gcloud container images describe gcr.io/PROJECT_ID/myapp:v1.0 \
  --show-package-vulnerability \
  --format=json

# List vulnerabilities with specific severity
gcloud container images describe gcr.io/PROJECT_ID/myapp:v1.0 \
  --show-package-vulnerability \
  --format=json | jq '.vulnerability.packageIssue[] | select(.severity | IN("CRITICAL", "HIGH"))'

# Get vulnerability occurrences
gcloud container images list-tags gcr.io/PROJECT_ID/myapp \
  --format='get(digest)' \
  --filter='tags:v1.0' | \
  xargs -I {} gcloud container images describe gcr.io/PROJECT_ID/myapp@{} \
  --show-package-vulnerability

# List all images with vulnerabilities
gcloud container images list --repository=gcr.io/PROJECT_ID \
  --format="table(name,tags)"

# Export vulnerability scan results
gcloud container images describe gcr.io/PROJECT_ID/myapp:v1.0 \
  --show-package-vulnerability \
  --format=json > vulnerability-report.json

# Check for specific CVEs
gcloud container images describe gcr.io/PROJECT_ID/myapp:v1.0 \
  --show-package-vulnerability \
  --format=json | jq '.vulnerability.packageIssue[] | select(.cve | contains("CVE-2024-"))'

# Set up continuous scanning
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Docker repository with scanning"

# Enable vulnerability scanning on Artifact Registry
gcloud artifacts repositories update my-repo \
  --location=us-central1 \
  --enable-vulnerability-scanning
```

### Binary Authorization

```bash
# Enable Binary Authorization API
gcloud services enable binaryauthorization.googleapis.com

# Create attestation note in Container Analysis
cat > note.json <<EOF
{
  "name": "projects/PROJECT_ID/notes/prod-attestor-note",
  "attestation": {
    "hint": {
      "human_readable_name": "Production attestation"
    }
  }
}
EOF

curl -X POST \
  "https://containeranalysis.googleapis.com/v1/projects/PROJECT_ID/notes?noteId=prod-attestor-note" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json" \
  -d @note.json

# Create attestor
gcloud container binauthz attestors create prod-attestor \
  --attestation-authority-note=prod-attestor-note \
  --attestation-authority-note-project=PROJECT_ID

# Add public key to attestor
gcloud container binauthz attestors public-keys add \
  --attestor=prod-attestor \
  --pgp-public-key-file=pubkey.asc

# Create Binary Authorization policy
cat > policy.yaml <<EOF
globalPolicyEvaluationMode: ENABLE
defaultAdmissionRule:
  requireAttestationsBy:
    - projects/PROJECT_ID/attestors/prod-attestor
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: REQUIRE_ATTESTATION
clusterAdmissionRules:
  us-central1-a.prod-cluster:
    requireAttestationsBy:
      - projects/PROJECT_ID/attestors/prod-attestor
      - projects/PROJECT_ID/attestors/security-attestor
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    evaluationMode: REQUIRE_ATTESTATION
admissionWhitelistPatterns:
  - namePattern: gcr.io/PROJECT_ID/trusted-base-images/*
EOF

# Import policy
gcloud container binauthz policy import policy.yaml

# View current policy
gcloud container binauthz policy export

# Create attestation for image
gcloud container binauthz attestations sign-and-create \
  --artifact-url=gcr.io/PROJECT_ID/myapp@sha256:abc123... \
  --attestor=prod-attestor \
  --attestor-project=PROJECT_ID \
  --pgp-key-fingerprint=ABCD1234... \
  --pgp-secret-key-file=privkey.asc

# List attestations for image
gcloud container binauthz attestations list \
  --artifact-url=gcr.io/PROJECT_ID/myapp@sha256:abc123... \
  --attestor=prod-attestor \
  --attestor-project=PROJECT_ID

# Dry run to test policy
gcloud container binauthz attestations create \
  --artifact-url=gcr.io/PROJECT_ID/myapp:v1.0 \
  --attestor=prod-attestor \
  --attestor-project=PROJECT_ID \
  --dry-run

# Update policy enforcement mode
cat > updated-policy.yaml <<EOF
globalPolicyEvaluationMode: ENABLE
defaultAdmissionRule:
  requireAttestationsBy:
    - projects/PROJECT_ID/attestors/prod-attestor
  enforcementMode: DRYRUN_AUDIT_LOG_ONLY
  evaluationMode: REQUIRE_ATTESTATION
EOF

gcloud container binauthz policy import updated-policy.yaml
```

### OS Patch Management (VM Manager)

```bash
# Enable OS Config API
gcloud services enable osconfig.googleapis.com

# Create patch deployment for immediate patching
gcloud compute os-config patch-deployments create emergency-patch \
  --file=patch-deployment.yaml

cat > patch-deployment.yaml <<EOF
patchConfig:
  rebootConfig: ALWAYS
  apt:
    type: DIST
    excludes:
      - kernel*
  yum:
    security: true
    minimal: true
instanceFilter:
  all: false
  zones:
    - us-central1-a
  labels:
    - env: prod
duration: 3600s
EOF

# Create recurring patch schedule
gcloud compute os-config patch-deployments create weekly-patches \
  --file=weekly-patch-deployment.yaml

cat > weekly-patch-deployment.yaml <<EOF
patchConfig:
  rebootConfig: DEFAULT
  apt:
    type: DIST
  yum:
    security: true
    minimal: true
instanceFilter:
  groupLabels:
    - labels:
        env: prod
        os: linux
recurringSchedule:
  timeZone:
    id: America/New_York
  timeOfDay:
    hours: 2
    minutes: 0
  frequency: WEEKLY
  weekly:
    dayOfWeek: SUNDAY
duration: 7200s
EOF

# List all patch deployments
gcloud compute os-config patch-deployments list

# Execute patch job immediately
gcloud compute os-config patch-jobs execute \
  --instance-filter-labels=env=prod \
  --async

# List patch jobs
gcloud compute os-config patch-jobs list

# Describe patch job status
gcloud compute os-config patch-jobs describe PATCH_JOB_ID

# List instance patch states
gcloud compute os-config patch-jobs list-instance-details PATCH_JOB_ID

# Cancel running patch job
gcloud compute os-config patch-jobs cancel PATCH_JOB_ID

# Get patch compliance report
gcloud compute instances os-inventory describe INSTANCE_NAME \
  --zone=us-central1-a \
  --view=FULL
```

### Web Security Scanner

```bash
# Enable Web Security Scanner API
gcloud services enable websecurityscanner.googleapis.com

# Create scan configuration
cat > scan-config.yaml <<EOF
displayName: "Production Web App Scan"
startingUrls:
  - "https://example.com"
targetPlatforms:
  - APP_ENGINE
  - COMPUTE
userAgent: CHROME_LINUX
maxQps: 5
authentication:
  googleAccount:
    username: "scanner@example.com"
    password: "secure-password"
schedule:
  scheduleTime: "2024-01-01T02:00:00Z"
  intervalDurationDays: 7
EOF

# Create scan configuration
gcloud web-security-scanner scan-configs create \
  --scan-config-from-file=scan-config.yaml \
  --project=PROJECT_ID

# Start scan run
gcloud web-security-scanner scan-runs start SCAN_CONFIG_NAME

# List scan runs
gcloud web-security-scanner scan-runs list SCAN_CONFIG_NAME

# Describe scan run
gcloud web-security-scanner scan-runs describe SCAN_RUN_NAME

# List findings
gcloud web-security-scanner findings list SCAN_RUN_NAME \
  --format="table(name,category,severity,trackingId)"

# List high severity findings
gcloud web-security-scanner findings list SCAN_RUN_NAME \
  --filter="severity=HIGH OR severity=CRITICAL"

# Export findings
gcloud web-security-scanner findings list SCAN_RUN_NAME \
  --format=json > scan-findings.json

# Stop scan run
gcloud web-security-scanner scan-runs stop SCAN_RUN_NAME

# Delete scan configuration
gcloud web-security-scanner scan-configs delete SCAN_CONFIG_NAME
```

### On-Demand Security Scanning

```bash
# Enable Cloud Asset Inventory
gcloud services enable cloudasset.googleapis.com

# Scan for public IPs
gcloud compute instances list \
  --format="table(name,zone,networkInterfaces[0].accessConfigs[0].natIP)" \
  --filter="networkInterfaces.accessConfigs.natIP:*"

# Find instances without Shielded VM features
gcloud compute instances list \
  --format="table(name,zone)" \
  --filter="NOT shieldedInstanceConfig.enableSecureBoot=true"

# Scan for unencrypted disks
gcloud compute disks list \
  --format="table(name,zone,diskEncryptionKey.sha256)" \
  --filter="NOT diskEncryptionKey.sha256:*"

# Find buckets with public access
gsutil ls -L -b gs://** 2>/dev/null | grep -A 5 "allUsers\|allAuthenticatedUsers"

# Scan for service accounts with keys
gcloud iam service-accounts list --format="value(email)" | while read sa; do
  echo "Service Account: $sa"
  gcloud iam service-accounts keys list --iam-account=$sa
done

# Find overly permissive firewall rules
gcloud compute firewall-rules list \
  --filter="sourceRanges:(0.0.0.0/0) AND allowed[].ports:(22 OR 3389)" \
  --format="table(name,sourceRanges,allowed)"

# Scan for legacy object ACLs on buckets
gsutil uniformbucketlevelaccess get gs://BUCKET_NAME

# Find SQL instances without SSL
gcloud sql instances list \
  --filter="settings.ipConfiguration.requireSsl=false" \
  --format="table(name,settings.ipConfiguration.requireSsl)"
```

## Threat Detection and Response

### Cloud IDS (Intrusion Detection System)

```bash
# Enable Cloud IDS API
gcloud services enable ids.googleapis.com

# Create IDS endpoint
gcloud ids endpoints create production-ids \
  --network=projects/PROJECT_ID/global/networks/prod-vpc \
  --zone=us-central1-a \
  --severity=INFORMATIONAL \
  --enable-traffic-logs

# List IDS endpoints
gcloud ids endpoints list --zone=us-central1-a

# Describe IDS endpoint
gcloud ids endpoints describe production-ids --zone=us-central1-a

# Update IDS endpoint severity threshold
gcloud ids endpoints update production-ids \
  --zone=us-central1-a \
  --severity=MEDIUM

# View IDS threat detections
gcloud logging read 'resource.type="ids.googleapis.com/Endpoint"' \
  --limit=100 \
  --format=json

# Query for specific threat types
gcloud logging read '
  resource.type="ids.googleapis.com/Endpoint"
  AND jsonPayload.alert.severity="HIGH"
  AND jsonPayload.alert.category="malware"
' --limit=50 --format=json

# Export IDS logs to BigQuery for analysis
gcloud logging sinks create ids-threat-logs \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/threat_intelligence \
  --log-filter='resource.type="ids.googleapis.com/Endpoint"'
```

### Chronicle SIEM Integration

```bash
# Configure log export to Chronicle
gcloud logging sinks create chronicle-security-logs \
  pubsub.googleapis.com/projects/PROJECT_ID/topics/chronicle-ingestion \
  --log-filter='
    logName:cloudaudit.googleapis.com OR
    resource.type="gce_firewall_rule" OR
    resource.type="ids.googleapis.com/Endpoint" OR
    logName:securitycenter.googleapis.com
  '

# Grant Chronicle service account permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:chronicle-ingestion@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/pubsub.subscriber"

# Create Pub/Sub subscription for Chronicle
gcloud pubsub subscriptions create chronicle-subscription \
  --topic=chronicle-ingestion \
  --push-endpoint=https://chronicle-forwarder.googleapis.com/v1/projects/PROJECT_ID/logs:write

# Export Security Command Center findings to Chronicle
gcloud scc notifications create chronicle-scc-export \
  --organization=ORGANIZATION_ID \
  --description="Export SCC findings to Chronicle" \
  --pubsub-topic=projects/PROJECT_ID/topics/chronicle-ingestion \
  --filter="state=\"ACTIVE\""
```

### Anomaly Detection Queries

```sql
-- Detect unusual data access patterns
SELECT
  protoPayload.authenticationInfo.principalEmail,
  resource.type,
  COUNT(*) as access_count,
  ARRAY_AGG(DISTINCT protoPayload.resourceName LIMIT 10) as resources_accessed
FROM `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY
  protoPayload.authenticationInfo.principalEmail,
  resource.type
HAVING access_count > 1000
ORDER BY access_count DESC;

-- Detect access from unusual locations
SELECT
  protoPayload.authenticationInfo.principalEmail,
  httpRequest.remoteIp,
  COUNT(*) as request_count,
  ARRAY_AGG(DISTINCT protoPayload.methodName LIMIT 10) as methods
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND NOT REGEXP_CONTAINS(httpRequest.remoteIp, r'^10\.|^172\.16\.|^192\.168\.')
GROUP BY
  protoPayload.authenticationInfo.principalEmail,
  httpRequest.remoteIp
HAVING request_count > 100
ORDER BY request_count DESC;

-- Detect after-hours administrative activity
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.methodName,
  protoPayload.resourceName
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE EXTRACT(HOUR FROM timestamp) NOT BETWEEN 6 AND 18
  AND EXTRACT(DAYOFWEEK FROM timestamp) BETWEEN 2 AND 6
  AND protoPayload.methodName IN (
    "SetIamPolicy",
    "CreateServiceAccountKey",
    "v1.compute.firewalls.insert",
    "v1.compute.firewalls.patch"
  )
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;

-- Detect rapid credential creation
SELECT
  protoPayload.authenticationInfo.principalEmail,
  COUNT(*) as keys_created,
  ARRAY_AGG(protoPayload.request.name) as service_accounts
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.methodName = "google.iam.admin.v1.CreateServiceAccountKey"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY protoPayload.authenticationInfo.principalEmail
HAVING keys_created > 5
ORDER BY keys_created DESC;
```

## Troubleshooting Scenarios

### Scenario 1: False Positive - SSH Port Detected as Open

**Situation**: Security Command Center reports OPEN_SSH finding for production instances that should have SSH restricted.

**Step-by-step Investigation:**

1. Verify the finding details
```bash
# Get finding details
gcloud scc findings list ORGANIZATION_ID \
  --filter='category="OPEN_SSH" AND state="ACTIVE"' \
  --format=json > ssh-findings.json

# Review affected resources
cat ssh-findings.json | jq '.[] | {resource: .finding.resourceName, severity: .finding.severity}'
```

2. Check actual firewall configuration
```bash
# List firewall rules allowing SSH
gcloud compute firewall-rules list \
  --filter="allowed[].ports:22" \
  --format="table(name,sourceRanges,targetTags,allowed)"

# Check if rules are actually restricted
gcloud compute firewall-rules describe ssh-allow-rule
```

3. Verify the discrepancy
```bash
# Check if instance has the expected network tags
gcloud compute instances describe INSTANCE_NAME \
  --zone=ZONE \
  --format="get(tags.items)"

# Verify firewall rule targets
gcloud compute firewall-rules describe ssh-allow-rule \
  --format="get(targetTags,sourceTags,sourceRanges)"
```

4. Resolution - Finding indicates rule allows 0.0.0.0/0 but only for specific tags
```bash
# Mark finding as false positive if rules are actually restricted
gcloud scc findings update-marks FINDING_NAME \
  --organization=ORGANIZATION_ID \
  --security-marks="false-positive=true,reason=restricted-by-tags,reviewed-by=security-team,review-date=$(date +%Y-%m-%d)"

# Set finding to inactive
gcloud scc findings update FINDING_NAME \
  --organization=ORGANIZATION_ID \
  --state=INACTIVE

# Document exception in security policy
# Consider using Cloud Armor or VPC Service Controls for additional protection
```

### Scenario 2: Alert Fatigue - Too Many Low-Severity Findings

**Situation**: Security team overwhelmed with hundreds of low-severity SCC findings.

**Step-by-step Investigation:**

1. Analyze finding distribution
```bash
# Count findings by category and severity
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\"" \
  --format="value(finding.category,finding.severity)" | \
  sort | uniq -c | sort -rn > finding-distribution.txt

# Identify top noisy findings
cat finding-distribution.txt | head -20
```

2. Review specific noisy category
```bash
# Example: Too many PUBLIC_IP_ADDRESS findings
gcloud scc findings list ORGANIZATION_ID \
  --filter='category="PUBLIC_IP_ADDRESS" AND state="ACTIVE"' \
  --format="table(finding.resourceName,finding.severity)" \
  --limit=100
```

3. Implement filtering strategy
```bash
# Option A: Update notification config to filter out low-severity
gcloud scc notifications update security-alerts \
  --organization=ORGANIZATION_ID \
  --filter='state="ACTIVE" AND (severity="HIGH" OR severity="CRITICAL")'

# Option B: Use security marks to track exceptions
for finding in $(gcloud scc findings list ORG_ID --filter='category="PUBLIC_IP_ADDRESS" AND severity="LOW"' --format="value(name)"); do
  gcloud scc findings update-marks $finding \
    --organization=ORG_ID \
    --security-marks="suppress=true,reason=expected-public-ips"
done

# Option C: Create separate notification channels by severity
gcloud scc notifications create critical-findings \
  --organization=ORGANIZATION_ID \
  --description="Critical findings only" \
  --pubsub-topic=projects/PROJECT_ID/topics/critical-security \
  --filter='state="ACTIVE" AND severity="CRITICAL"'

gcloud scc notifications create high-findings \
  --organization=ORGANIZATION_ID \
  --description="High severity findings" \
  --pubsub-topic=projects/PROJECT_ID/topics/high-security \
  --filter='state="ACTIVE" AND severity="HIGH"'
```

4. Automate finding triage
```python
# Create Cloud Function to auto-triage findings
def triage_finding(event, context):
    from google.cloud import securitycenter

    finding_name = event['attributes']['finding']
    client = securitycenter.SecurityCenterClient()

    finding = client.get_finding(name=finding_name)

    # Auto-mark known false positives
    if finding.category == "PUBLIC_IP_ADDRESS":
        if "bastion" in finding.resource_name or "lb-" in finding.resource_name:
            marks = {"expected": "true", "auto-triaged": "true"}
            client.update_security_marks(
                security_marks={"name": f"{finding_name}/securityMarks", "marks": marks}
            )
```

### Scenario 3: Missed Detection - Data Exfiltration Not Alerted

**Situation**: Post-incident review reveals large data export wasn't detected by monitoring.

**Step-by-step Investigation:**

1. Analyze what happened
```bash
# Search for large data transfers in the incident timeframe
gcloud logging read '
  resource.type="gcs_bucket"
  AND protoPayload.methodName="storage.objects.get"
  AND timestamp>="2024-01-01T10:00:00Z"
  AND timestamp<="2024-01-01T12:00:00Z"
' --format=json > data-access-incident.json

# Analyze transfer volumes
cat data-access-incident.json | jq -r '
  .[] |
  select(.protoPayload.response.size) |
  {
    user: .protoPayload.authenticationInfo.principalEmail,
    file: .protoPayload.resourceName,
    size: .protoPayload.response.size,
    time: .timestamp
  }
' | jq -s 'group_by(.user) | map({user: .[0].user, total_bytes: map(.size | tonumber) | add})'
```

2. Identify detection gaps
```bash
# Check if Data Access logs were enabled
gcloud logging read 'logName:cloudaudit.googleapis.com%2Fdata_access' \
  --limit=1 \
  --format=json

# Verify log-based metrics exist
gcloud logging metrics list | grep -i "data\|exfil"

# Check alerting policies
gcloud alpha monitoring policies list \
  --format="table(displayName,conditions[].displayName)"
```

3. Implement proper detection
```bash
# Create log-based metric for large data access
gcloud logging metrics create large_data_access \
  --description="Tracks large data downloads from GCS" \
  --log-filter='
    resource.type="gcs_bucket"
    AND protoPayload.methodName="storage.objects.get"
    AND protoPayload.status.code=0
  ' \
  --value-extractor='EXTRACT(protoPayload.response.size)'

# Create alerting policy for unusual data access
cat > data-exfiltration-alert.yaml <<EOF
displayName: "Potential Data Exfiltration"
conditions:
  - displayName: "Large volume of data accessed"
    conditionThreshold:
      filter: 'metric.type="logging.googleapis.com/user/large_data_access" resource.type="gcs_bucket"'
      comparison: COMPARISON_GT
      thresholdValue: 10737418240  # 10GB in bytes
      duration: 300s
      aggregations:
        - alignmentPeriod: 300s
          perSeriesAligner: ALIGN_SUM
          crossSeriesReducer: REDUCE_SUM
          groupByFields:
            - resource.bucket_name
notificationChannels:
  - projects/PROJECT_ID/notificationChannels/SECURITY_CHANNEL
alertStrategy:
  autoClose: 86400s
documentation:
  content: |
    # Potential Data Exfiltration Detected

    Unusual volume of data accessed from GCS bucket.

    Investigation steps:
    1. Identify the user/service account
    2. Verify the access pattern is authorized
    3. Check source IP addresses
    4. Review destination of data transfer
    5. Contain if unauthorized
  mimeType: text/markdown
severity: CRITICAL
enabled: true
EOF

gcloud alpha monitoring policies create --policy-from-file=data-exfiltration-alert.yaml

# Enable VPC Flow Logs for network-level visibility
gcloud compute networks subnets update SUBNET_NAME \
  --region=REGION \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-5-sec \
  --logging-flow-sampling=1.0
```

4. Enhance monitoring
```bash
# Create DLP job to scan for sensitive data
gcloud dlp jobs create --locations=us \
  --template=projects/PROJECT_ID/locations/us/inspectTemplates/TEMPLATE_ID \
  --actions='[{"saveFindings": {"outputConfig": {"table": {"projectId": "PROJECT_ID", "datasetId": "dlp_findings", "tableId": "scan_results"}}}}]'

# Set up Chronicle SOAR integration for automated response
gcloud logging sinks create chronicle-data-access \
  pubsub.googleapis.com/projects/PROJECT_ID/topics/chronicle-alerts \
  --log-filter='
    resource.type="gcs_bucket"
    AND protoPayload.methodName="storage.objects.get"
    AND protoPayload.response.size > 1073741824
  '
```

### Scenario 4: Cloud Armor Rule Causing Legitimate Traffic Blocks

**Situation**: Users reporting access issues after deploying new Cloud Armor WAF rules.

**Step-by-step Investigation:**

1. Identify blocked requests
```bash
# Query Cloud Armor logs for denied requests
gcloud logging read '
  resource.type="http_load_balancer"
  AND jsonPayload.enforcedSecurityPolicy.name!=""
  AND jsonPayload.enforcedSecurityPolicy.outcome="DENY"
' --limit=100 --format=json > armor-denies.json

# Analyze which rules are blocking
cat armor-denies.json | jq -r '.[] | {
  time: .timestamp,
  ip: .httpRequest.remoteIp,
  url: .httpRequest.requestUrl,
  rule: .jsonPayload.enforcedSecurityPolicy.configuredAction,
  priority: .jsonPayload.enforcedSecurityPolicy.priority
}'
```

2. Review specific rule causing blocks
```bash
# Get rule details
gcloud compute security-policies rules describe 1000 \
  --security-policy=web-app-policy

# Check rule expression and action
gcloud compute security-policies describe web-app-policy \
  --format=json | jq '.rules[] | select(.priority == 1000)'
```

3. Test and adjust rule
```bash
# Put rule in preview mode (log but don't block)
gcloud compute security-policies rules update 1000 \
  --security-policy=web-app-policy \
  --preview

# Monitor logs for would-be blocks
gcloud logging read '
  resource.type="http_load_balancer"
  AND jsonPayload.enforcedSecurityPolicy.name!=""
  AND jsonPayload.enforcedSecurityPolicy.preconfiguredExprIds:*
' --limit=50 --format=json

# Analyze false positive rate
cat armor-preview.json | jq -r '
  group_by(.httpRequest.remoteIp) |
  map({ip: .[0].httpRequest.remoteIp, count: length}) |
  sort_by(.count) | reverse
'
```

4. Resolution - Whitelist legitimate traffic
```bash
# Create exemption rule with higher priority
gcloud compute security-policies rules create 500 \
  --security-policy=web-app-policy \
  --expression='origin.ip == "203.0.113.50" || origin.ip == "203.0.113.51"' \
  --action=allow \
  --description="Whitelist known good IPs"

# OR adjust existing rule to be more specific
gcloud compute security-policies rules update 1000 \
  --security-policy=web-app-policy \
  --expression='evaluatePreconfiguredExpr("sqli-stable") && !inIpRange(origin.ip, "203.0.113.0/24")'

# Re-enable rule after validation
gcloud compute security-policies rules update 1000 \
  --security-policy=web-app-policy \
  --no-preview

# Document exception
gcloud compute security-policies rules update 500 \
  --security-policy=web-app-policy \
  --description="Whitelist partner IPs - Ticket SEC-12345"
```

### Scenario 5: Cryptomining Detection Investigation

**Situation**: Event Threat Detection alerts on potential cryptomining activity.

**Step-by-step Investigation:**

1. Review the finding
```bash
# Get cryptomining finding details
gcloud scc findings list ORGANIZATION_ID \
  --filter='category="Cryptomining" AND state="ACTIVE"' \
  --format=json > cryptomining-finding.json

# Extract key information
cat cryptomining-finding.json | jq -r '.[] | {
  resource: .finding.resourceName,
  category: .finding.category,
  sourceIp: .finding.sourceProperties.sourceId.customerOrganization,
  eventTime: .finding.eventTime
}'
```

2. Investigate affected instance
```bash
# Get instance details
INSTANCE_NAME=$(cat cryptomining-finding.json | jq -r '.[0].finding.resourceName' | grep -oP 'instances/\K[^/]+')
ZONE=$(cat cryptomining-finding.json | jq -r '.[0].finding.resourceName' | grep -oP 'zones/\K[^/]+')

# Check instance metadata and configuration
gcloud compute instances describe $INSTANCE_NAME --zone=$ZONE

# Review recent changes to instance
gcloud logging read "
  resource.type=\"gce_instance\"
  AND resource.labels.instance_id=\"$INSTANCE_NAME\"
  AND protoPayload.methodName=~\"compute.instances.*\"
" --limit=50 --format=json

# Check for unauthorized access
gcloud logging read "
  resource.type=\"gce_instance\"
  AND resource.labels.instance_id=\"$INSTANCE_NAME\"
  AND (textPayload=~\"Accepted publickey\" OR textPayload=~\"session opened\")
" --limit=50 --format=json
```

3. Analyze network activity
```bash
# Check VPC Flow Logs for unusual connections
gcloud logging read "
  resource.type=\"gce_subnetwork\"
  AND resource.labels.subnetwork_name=\"$SUBNET_NAME\"
  AND jsonPayload.connection.src_instance=\"$INSTANCE_NAME\"
" --limit=100 --format=json > flow-logs.json

# Look for connections to known mining pools
cat flow-logs.json | jq -r '.[] | {
  dest_ip: .jsonPayload.connection.dest_ip,
  dest_port: .jsonPayload.connection.dest_port,
  bytes: .jsonPayload.bytes_sent
}' | sort | uniq -c
```

4. Containment and remediation
```bash
# 1. Immediate isolation
gcloud compute instances stop $INSTANCE_NAME --zone=$ZONE

# 2. Create forensic snapshot
gcloud compute disks snapshot ${INSTANCE_NAME}-disk \
  --snapshot-names=cryptomining-forensic-$(date +%Y%m%d) \
  --zone=$ZONE

# 3. Block malicious IPs at firewall level
MALICIOUS_IPS=$(cat flow-logs.json | jq -r '.[] | .jsonPayload.connection.dest_ip' | sort -u)
gcloud compute firewall-rules create block-mining-pools \
  --direction=EGRESS \
  --priority=100 \
  --network=default \
  --action=DENY \
  --rules=all \
  --destination-ranges=$MALICIOUS_IPS

# 4. Terminate instance and deploy clean replacement
gcloud compute instances delete $INSTANCE_NAME --zone=$ZONE
gcloud compute instances create ${INSTANCE_NAME}-clean \
  --image=trusted-base-image \
  --zone=$ZONE

# 5. Review IAM permissions
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:*compute*"

# 6. Rotate all potentially compromised credentials
gcloud iam service-accounts keys list --iam-account=SUSPECTED_SA
gcloud iam service-accounts keys delete KEY_ID --iam-account=SUSPECTED_SA
```

### Scenario 6: Binary Authorization Blocking Legitimate Deployment

**Situation**: GKE deployment failing due to Binary Authorization policy rejecting container image.

**Step-by-step Investigation:**

1. Review deployment failure
```bash
# Check GKE audit logs for Binary Authorization denials
gcloud logging read '
  resource.type="k8s_cluster"
  AND protoPayload.status.message=~".*VIOLATES_POLICY.*"
' --limit=10 --format=json > binauthz-denials.json

# Get specific rejection details
cat binauthz-denials.json | jq -r '.[] | {
  time: .timestamp,
  image: .protoPayload.request.spec.containers[].image,
  reason: .protoPayload.status.message
}'
```

2. Check Binary Authorization policy
```bash
# Export current policy
gcloud container binauthz policy export > current-policy.yaml

# Review policy requirements
cat current-policy.yaml

# Check which attestors are required
cat current-policy.yaml | grep -A 5 "requireAttestationsBy"
```

3. Verify image attestations
```bash
# Get image digest
IMAGE_DIGEST=$(gcloud container images describe gcr.io/PROJECT_ID/myapp:v1.0 --format='get(image_summary.digest)')

# Check for attestations
gcloud container binauthz attestations list \
  --artifact-url="gcr.io/PROJECT_ID/myapp@${IMAGE_DIGEST}" \
  --attestor=prod-attestor \
  --attestor-project=PROJECT_ID
```

4. Resolution options
```bash
# Option A: Create required attestation
gcloud container binauthz attestations sign-and-create \
  --artifact-url="gcr.io/PROJECT_ID/myapp@${IMAGE_DIGEST}" \
  --attestor=prod-attestor \
  --attestor-project=PROJECT_ID \
  --pgp-key-fingerprint=YOUR_KEY_FINGERPRINT

# Option B: Temporarily add image to whitelist
cat > temp-policy.yaml <<EOF
globalPolicyEvaluationMode: ENABLE
admissionWhitelistPatterns:
  - namePattern: gcr.io/PROJECT_ID/myapp:v1.0
defaultAdmissionRule:
  requireAttestationsBy:
    - projects/PROJECT_ID/attestors/prod-attestor
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: REQUIRE_ATTESTATION
EOF

gcloud container binauthz policy import temp-policy.yaml

# Option C: Switch to dry-run mode for testing
cat > dryrun-policy.yaml <<EOF
globalPolicyEvaluationMode: ENABLE
defaultAdmissionRule:
  requireAttestationsBy:
    - projects/PROJECT_ID/attestors/prod-attestor
  enforcementMode: DRYRUN_AUDIT_LOG_ONLY
  evaluationMode: REQUIRE_ATTESTATION
EOF

gcloud container binauthz policy import dryrun-policy.yaml

# Verify deployment succeeds
kubectl apply -f deployment.yaml

# Monitor audit logs to ensure no legitimate blocks
gcloud logging read '
  resource.type="k8s_cluster"
  AND protoPayload.response.status.message=~".*DRYRUN.*"
' --limit=10 --format=json
```

### Scenario 7: Compliance Violation - PCI-DSS Audit Failure

**Situation**: PCI-DSS audit reveals security logging gaps and configuration issues.

**Step-by-step Investigation:**

1. Audit current logging configuration
```bash
# Check if all audit log types are enabled
gcloud logging read "logName:cloudaudit.googleapis.com" --limit=10 --format=json

# Verify data access logs are enabled
gcloud projects get-iam-policy PROJECT_ID --format=json | \
  jq '.auditConfigs'

# Check log retention
gcloud logging sinks list --format="table(name,destination)"
```

2. Enable required PCI-DSS logging
```bash
# Enable Data Access logs for all services
cat > audit-config.yaml <<EOF
auditConfigs:
  - service: allServices
    auditLogConfigs:
      - logType: ADMIN_READ
      - logType: DATA_READ
      - logType: DATA_WRITE
EOF

gcloud projects set-iam-policy PROJECT_ID audit-config.yaml

# Create long-term storage sink for compliance
gcloud logging sinks create pci-dss-audit-logs \
  storage.googleapis.com/pci-compliance-logs-$(date +%Y) \
  --log-filter='logName:cloudaudit.googleapis.com OR severity>=WARNING'

# Set bucket retention policy (1 year for PCI-DSS)
gsutil retention set 1y gs://pci-compliance-logs-$(date +%Y)
gsutil retention lock gs://pci-compliance-logs-$(date +%Y)

# Enable bucket versioning
gsutil versioning set on gs://pci-compliance-logs-$(date +%Y)
```

3. Review security findings relevant to PCI-DSS
```bash
# Check for unencrypted data
gcloud scc findings list ORGANIZATION_ID \
  --filter='category="ENCRYPTION" AND state="ACTIVE"' \
  --format=json

# Check for public exposure
gcloud scc findings list ORGANIZATION_ID \
  --filter='(category="PUBLIC_BUCKET_ACL" OR category="PUBLIC_IP_ADDRESS") AND state="ACTIVE"' \
  --format=json

# Check firewall configurations
gcloud scc findings list ORGANIZATION_ID \
  --filter='category="OPEN_FIREWALL" AND state="ACTIVE"' \
  --format=json
```

4. Implement PCI-DSS controls
```bash
# Ensure encryption at rest
gcloud compute disks list --filter="NOT diskEncryptionKey:*" \
  --format="table(name,zone)"

# Enable CMEK for existing disks
gcloud compute disks add-resource-policies DISK_NAME \
  --resource-policies=disk-encryption-policy \
  --zone=ZONE

# Require SSL for Cloud SQL
for instance in $(gcloud sql instances list --format="value(name)"); do
  gcloud sql instances patch $instance --require-ssl
done

# Implement network segmentation
gcloud compute networks create pci-cardholder-data \
  --subnet-mode=custom

gcloud compute networks subnets create pci-subnet \
  --network=pci-cardholder-data \
  --region=us-central1 \
  --range=10.10.0.0/24 \
  --enable-flow-logs \
  --enable-private-ip-google-access

# Create restrictive firewall rules
gcloud compute firewall-rules create pci-deny-all \
  --network=pci-cardholder-data \
  --direction=INGRESS \
  --priority=65534 \
  --action=DENY \
  --rules=all \
  --source-ranges=0.0.0.0/0

gcloud compute firewall-rules create pci-allow-internal \
  --network=pci-cardholder-data \
  --direction=INGRESS \
  --priority=1000 \
  --action=ALLOW \
  --rules=all \
  --source-ranges=10.10.0.0/24

# Set up quarterly vulnerability scanning
gcloud web-security-scanner scan-configs create pci-quarterly-scan \
  --scan-config-from-file=pci-scan-config.yaml
```

5. Generate compliance report
```bash
# Export Security Command Center findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\"" \
  --format=json > compliance-findings-$(date +%Y%m%d).json

# Generate access review report
gcloud logging read '
  protoPayload.methodName="SetIamPolicy"
  AND timestamp>="'$(date -d '90 days ago' '+%Y-%m-%dT00:00:00Z')'"
' --format=json > iam-changes-90days.json

# Document all findings and remediation
echo "PCI-DSS Compliance Report - $(date)" > pci-compliance-report.md
echo "================================================" >> pci-compliance-report.md
echo "" >> pci-compliance-report.md
echo "## Security Findings Summary" >> pci-compliance-report.md
jq -r 'group_by(.finding.severity) | map({severity: .[0].finding.severity, count: length})' \
  compliance-findings-$(date +%Y%m%d).json >> pci-compliance-report.md
```

### Scenario 8: Privilege Escalation Detection and Response

**Situation**: Alert triggered for unauthorized privilege escalation attempt.

**Step-by-step Investigation:**

1. Analyze the privilege escalation event
```bash
# Get privilege escalation events
gcloud logging read '
  protoPayload.methodName="SetIamPolicy"
  AND protoPayload.serviceData.policyDelta.bindingDeltas.action="ADD"
  AND protoPayload.serviceData.policyDelta.bindingDeltas.role=~"roles/.*admin"
' --limit=20 --format=json > privesc-events.json

# Review specific escalation
cat privesc-events.json | jq -r '.[] | {
  time: .timestamp,
  actor: .protoPayload.authenticationInfo.principalEmail,
  resource: .protoPayload.resourceName,
  role_granted: .protoPayload.serviceData.policyDelta.bindingDeltas[].role,
  member: .protoPayload.serviceData.policyDelta.bindingDeltas[].member
}'
```

2. Determine if escalation was authorized
```bash
# Check if change ticket exists
ACTOR=$(cat privesc-events.json | jq -r '.[0].protoPayload.authenticationInfo.principalEmail')

# Review actor's recent activities
gcloud logging read "
  protoPayload.authenticationInfo.principalEmail=\"$ACTOR\"
  AND timestamp>=\"$(date -d '1 hour ago' '+%Y-%m-%dT%H:%M:%SZ')\"
" --limit=50 --format=json > actor-activity.json

# Check if actor typically performs admin actions
gcloud logging read "
  protoPayload.authenticationInfo.principalEmail=\"$ACTOR\"
  AND protoPayload.methodName=\"SetIamPolicy\"
  AND timestamp>=\"$(date -d '30 days ago' '+%Y-%m-%dT%H:%M:%SZ')\"
" --format=json | jq length
```

3. Immediate containment if unauthorized
```bash
# Suspend the account
gcloud identity users update $ACTOR --suspended

# Revoke the escalated permission
RESOURCE=$(cat privesc-events.json | jq -r '.[0].protoPayload.resourceName')
ROLE=$(cat privesc-events.json | jq -r '.[0].protoPayload.serviceData.policyDelta.bindingDeltas[0].role')
MEMBER=$(cat privesc-events.json | jq -r '.[0].protoPayload.serviceData.policyDelta.bindingDeltas[0].member')

gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="$MEMBER" \
  --role="$ROLE"

# Terminate active sessions
gcloud logging read "
  protoPayload.authenticationInfo.principalEmail=\"$ACTOR\"
  AND timestamp>=\"$(date -d '4 hours ago' '+%Y-%m-%dT%H:%M:%SZ')\"
  AND httpRequest.remoteIp:*
" --format=json | jq -r '.[].httpRequest.remoteIp' | sort -u > active-ips.txt

# Block source IPs if necessary
while read ip; do
  gcloud compute firewall-rules create block-$ip \
    --direction=INGRESS \
    --priority=100 \
    --network=default \
    --action=DENY \
    --rules=all \
    --source-ranges=$ip
done < active-ips.txt
```

4. Post-incident hardening
```bash
# Enable Organization Policy constraints
cat > restrict-admin-roles.yaml <<EOF
name: organizations/ORGANIZATION_ID/policies/iam.allowedPolicyMemberDomains
spec:
  rules:
    - values:
        allowedValues:
          - "C0abc123"  # Your organization ID
EOF

gcloud org-policies set-policy restrict-admin-roles.yaml

# Require approval for sensitive role changes
cat > approval-policy.yaml <<EOF
privilegedAccess:
  gcpIamPolicy:
    allowedServices:
      - "cloudresourcemanager.googleapis.com"
    allowedResourceTypes:
      - "cloudresourcemanager.googleapis.com/Project"
    maxRequestDuration: "3600s"
    approverPools:
      - users:
          - security-admin@example.com
          - director@example.com
EOF

# Enhance monitoring
gcloud logging metrics create high_privilege_grants \
  --description="Track high-privilege role grants" \
  --log-filter='
    protoPayload.methodName="SetIamPolicy"
    AND protoPayload.serviceData.policyDelta.bindingDeltas.action="ADD"
    AND protoPayload.serviceData.policyDelta.bindingDeltas.role=~"roles/(.*admin|owner|editor)"
  '

# Create zero-tolerance alert
cat > privesc-alert.yaml <<EOF
displayName: "Privilege Escalation - Immediate Response Required"
conditions:
  - displayName: "Admin role granted"
    conditionThreshold:
      filter: 'metric.type="logging.googleapis.com/user/high_privilege_grants"'
      comparison: COMPARISON_GT
      thresholdValue: 0
      duration: 0s
notificationChannels:
  - projects/PROJECT_ID/notificationChannels/SECURITY_PAGERDUTY
  - projects/PROJECT_ID/notificationChannels/SECURITY_EMAIL
alertStrategy:
  autoClose: 86400s
  notificationRateLimit:
    period: 60s
documentation:
  content: |
    # CRITICAL: Privilege Escalation Detected

    Admin role has been granted. IMMEDIATE investigation required.

    1. Verify authorization (check change ticket)
    2. If unauthorized, suspend account immediately
    3. Revoke escalated permissions
    4. Block source IP addresses
    5. Initiate incident response procedure
    6. Page security on-call
  mimeType: text/markdown
severity: CRITICAL
enabled: true
EOF

gcloud alpha monitoring policies create --policy-from-file=privesc-alert.yaml
```

## Compliance Monitoring

### Organization Policy Compliance
```bash
# List policy violations
gcloud asset search-all-resources \
  --scope=organizations/ORG_ID \
  --query="state:ACTIVE AND -labels.compliant:true"

# Automated compliance checks
for project in $(gcloud projects list --format="value(projectId)"); do
  echo "Checking $project..."
  gcloud sql instances list --project=$project --filter="settings.ipConfiguration.requireSsl=false"
done

# Check for Organization Policy violations
gcloud resource-manager org-policies describe constraints/iam.allowedPolicyMemberDomains \
  --effective \
  --project=PROJECT_ID

# Export compliance posture
gcloud asset analyze-iam-policy \
  --scope=organizations/ORG_ID \
  --full-resource-name=//cloudresourcemanager.googleapis.com/organizations/ORG_ID \
  --format=json > org-iam-analysis.json
```

### PCI-DSS Compliance Checks

```bash
# Check encryption requirements
# Requirement 3.4: Encryption at rest
gcloud compute disks list \
  --filter="NOT diskEncryptionKey:*" \
  --format="table(name,zone,sizeGb)" > unencrypted-disks.txt

# Requirement 1.3: Network segmentation
gcloud compute firewall-rules list \
  --filter="sourceRanges:(0.0.0.0/0)" \
  --format="table(name,sourceRanges,allowed)" > permissive-firewall-rules.txt

# Requirement 10.2: Audit logging
gcloud logging sinks list --format="table(name,destination)" > logging-sinks.txt

# Requirement 8.7: Access control
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --format="table(bindings.role,bindings.members)" > iam-permissions.txt
```

### HIPAA Compliance Checks

```bash
# Check BAA compliance
# Access logs for PHI data
gcloud logging read 'resource.type="gcs_bucket" AND resource.labels.bucket_name=~".*phi.*"' \
  --limit=100 --format=json > phi-access-logs.json

# Verify encryption in transit
gcloud sql instances list \
  --filter="settings.ipConfiguration.requireSsl=false" \
  --format="table(name,settings.ipConfiguration.requireSsl)"

# Check for VPC Service Controls
gcloud access-context-manager perimeters list \
  --policy=ACCESS_POLICY_NAME

# Verify audit log retention
gsutil lifecycle get gs://hipaa-audit-logs
```

### SOC 2 Compliance Checks

```bash
# CC6.1: Logical access controls
gcloud projects get-iam-policy PROJECT_ID \
  --format=json | jq '.bindings[] | select(.role | contains("admin") or contains("owner"))'

# CC6.6: Encryption
gcloud compute disks list --filter="NOT diskEncryptionKey:*"
gcloud sql instances list --filter="settings.ipConfiguration.requireSsl=false"

# CC7.2: System monitoring
gcloud alpha monitoring policies list --format="table(displayName,enabled)"

# CC7.3: Threat detection
gcloud scc findings list ORGANIZATION_ID \
  --filter="state=\"ACTIVE\"" \
  --format="table(finding.category,finding.severity)"

# Generate SOC 2 compliance report
cat > soc2-compliance-check.sh <<'EOF'
#!/bin/bash
echo "SOC 2 Compliance Report - $(date)" > soc2-report.txt
echo "======================================" >> soc2-report.txt

echo -e "\n## CC6.1 - Logical Access Controls" >> soc2-report.txt
gcloud projects get-iam-policy PROJECT_ID --flatten="bindings[].members" \
  --filter="bindings.role:(roles/owner OR roles/editor)" \
  --format="table(bindings.role,bindings.members)" >> soc2-report.txt

echo -e "\n## CC6.6 - Encryption" >> soc2-report.txt
echo "Unencrypted disks:" >> soc2-report.txt
gcloud compute disks list --filter="NOT diskEncryptionKey:*" --format="value(name)" >> soc2-report.txt

echo -e "\n## CC7.2 - System Monitoring" >> soc2-report.txt
gcloud alpha monitoring policies list --format="table(displayName,enabled)" >> soc2-report.txt

echo -e "\n## CC7.3 - Active Security Findings" >> soc2-report.txt
gcloud scc findings list ORGANIZATION_ID --filter="state=\"ACTIVE\"" \
  --format="table(finding.category,finding.severity)" >> soc2-report.txt

cat soc2-report.txt
EOF
chmod +x soc2-compliance-check.sh
./soc2-compliance-check.sh
```

### Access Reviews
```python
def audit_service_account_keys():
    """Find old service account keys"""
    from google.cloud import iam_admin_v1
    from datetime import datetime, timedelta

    client = iam_admin_v1.IAMClient()
    project = "projects/PROJECT_ID"

    old_keys = []
    for sa in client.list_service_accounts(name=project):
        for key in client.list_service_account_keys(name=sa.name):
            age = datetime.now() - key.valid_after_time
            if age > timedelta(days=90):
                old_keys.append((sa.email, key.name, age.days))

    return old_keys
```

## Best Practices

### Monitoring
1. **Enable all audit log types**: Admin Activity, Data Access, System Event, Policy Denied
2. **Real-time alerting**: Configure Cloud Logging metrics and alerting policies for critical security events
3. **Log aggregation**: Use BigQuery sinks for long-term analysis and compliance
4. **Regular SCC review**: Daily review of active findings, especially CRITICAL and HIGH severity
5. **Automated compliance**: Schedule regular compliance checks using Cloud Scheduler + Cloud Functions
6. **Baseline normal behavior**: Establish baselines for data access, IAM changes, and network patterns
7. **Threat intelligence integration**: Export logs to Chronicle SIEM for advanced threat correlation
8. **Multi-channel notifications**: Use Email, Slack, PagerDuty for severity-appropriate routing
9. **Log retention**: Maintain audit logs for 1+ year for compliance (PCI-DSS, HIPAA, SOC 2)
10. **VPC Flow Logs**: Enable for network visibility and anomaly detection

### Incident Response
1. **Documented IR procedures**: Maintain runbooks for common scenarios (privilege escalation, data exfiltration, cryptomining)
2. **Regular tabletop exercises**: Quarterly IR drills to test procedures and team readiness
3. **Forensic readiness**: Automated disk snapshotting, log preservation, and isolated forensic networks
4. **Communication plan**: Clear escalation paths, stakeholder notification templates, external communication procedures
5. **Evidence preservation**: Immutable log sinks, snapshot retention policies, chain of custody documentation
6. **Post-incident reviews**: Blameless postmortems focusing on process improvement
7. **Continuous improvement**: Update detection rules, containment procedures, and monitoring thresholds based on lessons learned
8. **Automated containment**: Use Cloud Functions triggered by SCC notifications for rapid response
9. **Integration with SOAR**: Connect Chronicle or third-party SOAR platforms for orchestrated response
10. **Legal and compliance**: Understand breach notification requirements (GDPR, state laws)

### Vulnerability Management
1. **Continuous scanning**: Automatic Container Analysis for all images pushed to GCR/Artifact Registry
2. **Patch management**: Weekly OS patching schedules using VM Manager, emergency patches within 24 hours
3. **Binary Authorization**: Enforce attestation requirements for production GKE clusters
4. **Regular penetration testing**: Quarterly external pen tests, annual red team exercises
5. **Bug bounty program**: Leverage Google VRP or establish private program
6. **Vulnerability disclosure process**: Clear process for accepting and triaging external reports
7. **Risk-based prioritization**: Focus on CRITICAL/HIGH vulnerabilities in internet-facing systems first
8. **Vulnerability SLAs**: CRITICAL - 24 hours, HIGH - 7 days, MEDIUM - 30 days
9. **Web Security Scanner**: Monthly automated scans of web applications
10. **Third-party dependencies**: Monitor and update container base images, language libraries

### Cloud Armor
1. **Start with preview mode**: Test WAF rules in preview before enforcing to avoid false positives
2. **Layer defenses**: Use multiple rule types - preconfigured WAF, rate limiting, geo-blocking
3. **Adaptive Protection**: Enable ML-based Layer 7 DDoS detection for dynamic attack mitigation
4. **Rule priority**: Order rules correctly - whitelists at low priority, blocks at higher priority
5. **Regular tuning**: Review denied request logs weekly, adjust rules based on false positives
6. **Bot management**: Integrate reCAPTCHA Enterprise for sophisticated bot detection
7. **Logging and monitoring**: Export Cloud Armor logs to BigQuery for trend analysis
8. **Geographic restrictions**: Use geo-blocking for regions with no legitimate business need
9. **Rate limiting**: Implement progressive rate limits - per IP, per session, global
10. **Policy versioning**: Export policies to version control before making changes

### Security Command Center
1. **Premium tier for production**: Standard tier lacks critical features (ETD, Container Threat Detection)
2. **Security marks**: Use consistently for asset classification, false positive tracking, exception management
3. **Custom findings**: Create findings for organization-specific security controls
4. **Notification routing**: Separate channels by severity and category for appropriate response
5. **Finding workflow**: ACTIVE -> investigate -> mark false positive or INACTIVE after remediation
6. **Integration with SIEM**: Export to Chronicle or third-party SIEM for advanced analytics
7. **Automated remediation**: Trigger Cloud Functions based on SCC notifications to auto-fix issues
8. **Regular audits**: Monthly review of all active findings, quarterly validation of security marks
9. **Compliance dashboards**: Use Data Studio to visualize findings by compliance framework
10. **Asset inventory**: Leverage SCC for real-time asset discovery and change tracking

## Exam Tips

### Common Security Operations Traps

1. **Log Types Confusion**
   - TRAP: Thinking Admin Activity logs include data reads
   - REALITY: Data Access logs (disabled by default) required for read/write operations
   - EXAM TIP: Admin Activity = configuration changes, Data Access = data operations

2. **SCC Standard vs Premium**
   - TRAP: Assuming Standard tier includes Event Threat Detection
   - REALITY: ETD, Container Threat Detection, Security Health Analytics require Premium
   - EXAM TIP: If question mentions malware detection, cryptomining, or container threats = Premium tier

3. **Cloud Armor Scope**
   - TRAP: Thinking Cloud Armor protects all GCP resources
   - REALITY: Only protects resources behind Global/External HTTP(S) Load Balancers
   - EXAM TIP: Cloud Armor cannot protect Cloud SQL, GCS direct access, or internal resources

4. **Binary Authorization Enforcement**
   - TRAP: Believing Binary Authorization applies to all container deployments
   - REALITY: Only enforces on GKE clusters with policy enabled
   - EXAM TIP: Cloud Run and GCE containers are NOT covered by Binary Authorization

5. **Log Sink Permissions**
   - TRAP: Creating sink without granting destination permissions
   - REALITY: Sink service account needs Writer role on destination (BigQuery Data Editor, Storage Object Creator)
   - EXAM TIP: Questions about "logs not appearing" often involve missing sink SA permissions

6. **Security Marks vs Labels**
   - TRAP: Using resource labels for SCC filtering
   - REALITY: Security marks are SCC-specific, separate from resource labels
   - EXAM TIP: Security marks can be added to findings AND assets, labels only on resources

7. **Chronicle vs SCC**
   - TRAP: Thinking Chronicle replaces Security Command Center
   - REALITY: Chronicle is SIEM (log analysis, threat hunting), SCC is CSPM (asset inventory, vulnerability detection)
   - EXAM TIP: Use Chronicle for log correlation, SCC for configuration/vulnerability findings

8. **VPC Flow Logs Cost**
   - TRAP: Enabling 100% sampling on all subnets
   - REALITY: High sampling rates generate massive log volumes and costs
   - EXAM TIP: Start with 10% sampling, use aggregation intervals of 5-10 seconds for production

9. **Incident Response Order**
   - TRAP: Deleting compromised instances before collecting forensics
   - REALITY: Always snapshot disks BEFORE termination for evidence preservation
   - EXAM TIP: Containment order: snapshot -> isolate -> investigate -> eradicate -> recover

10. **Compliance Requirements**
    - TRAP: Assuming audit logs alone satisfy compliance
    - REALITY: Compliance requires logs + retention + encryption + access controls + monitoring
    - EXAM TIP: PCI-DSS questions involve multiple controls, not just one technology

### SCC vs Chronicle vs Cloud Armor Decision Matrix

| Scenario | Solution | Why |
|----------|----------|-----|
| Detect open firewall rules | Security Command Center | Configuration vulnerability detection |
| Correlate logs from multiple sources | Chronicle | SIEM for log aggregation and threat hunting |
| Block SQL injection attacks | Cloud Armor | WAF at application edge |
| Find unencrypted GCS buckets | Security Command Center | Asset inventory and misconfiguration detection |
| Detect brute force login attempts | Event Threat Detection (SCC Premium) | Behavioral threat detection from Cloud Logging |
| Rate limit API endpoint | Cloud Armor | Layer 7 rate limiting and DDoS protection |
| Hunt for APT indicators | Chronicle | Advanced threat hunting with YARA-L rules |
| Detect container vulnerabilities | Container Threat Detection (SCC Premium) | Runtime container security monitoring |
| Block traffic from specific countries | Cloud Armor | Geo-blocking at edge |
| Investigate security incident timeline | Cloud Logging + Chronicle | Log analysis and correlation |
| Compliance reporting (PCI-DSS) | Security Command Center | Finding aggregation by compliance standard |
| Protect against Layer 7 DDoS | Cloud Armor Adaptive Protection | ML-based attack mitigation |

### When to Use Each Tool

**Security Command Center:**
- Asset discovery and inventory management
- Configuration vulnerability scanning
- Compliance posture management
- Security health analytics
- Managing findings across organization
- Automated finding creation from custom detectors

**Chronicle:**
- Advanced threat hunting across all logs
- Correlation of security events from multiple sources
- Behavioral analysis and anomaly detection
- Investigation of complex attack chains
- Threat intelligence integration
- Long-term log retention (years) for forensics

**Cloud Armor:**
- DDoS protection (Layer 3/4 and Layer 7)
- WAF protection against OWASP Top 10
- Rate limiting and bot management
- Geo-blocking and IP allow/deny lists
- Protection of public-facing web applications
- Adaptive ML-based attack mitigation

**Cloud Logging:**
- Real-time log streaming and analysis
- Short-to-medium term log storage (30 days default)
- Log-based metrics creation
- Alerting policy triggers
- Audit trail for API activity
- Integration point for SIEM export

### Compliance Framework Requirements

**PCI-DSS:**
- Requirement 10: Enable all audit logs, 1-year retention minimum
- Requirement 1: Network segmentation with restrictive firewall rules
- Requirement 3: Encryption at rest for cardholder data (CMEK)
- Requirement 4: Encryption in transit (SSL/TLS required)
- Requirement 8: Strong access control, MFA, regular access reviews
- Requirement 11: Quarterly vulnerability scans
- **EXAM TIP**: PCI questions emphasize logging, encryption, and network segmentation

**HIPAA:**
- Access logs for all PHI access (Data Access logs required)
- Encryption at rest and in transit
- VPC Service Controls to prevent data exfiltration
- BAA (Business Associate Agreement) with Google
- Audit log retention for 6 years
- **EXAM TIP**: HIPAA questions focus on data protection and access controls

**SOC 2:**
- CC6.1: Logical access controls (IAM policies, least privilege)
- CC6.6: Encryption of sensitive data
- CC7.2: System monitoring (Cloud Monitoring, alerting policies)
- CC7.3: Threat detection (Security Command Center, ETD)
- CC7.4: Incident response procedures
- **EXAM TIP**: SOC 2 questions are control-focused, emphasizing process over specific tech

### Quick Command Reference for Exam

```bash
# Most common security operations commands

# View recent security events
gcloud logging read "logName:cloudaudit.googleapis.com" --limit=50

# List active SCC findings
gcloud scc findings list ORG_ID --filter="state=\"ACTIVE\""

# Create log-based metric
gcloud logging metrics create METRIC_NAME --log-filter="FILTER"

# Create alerting policy from file
gcloud alpha monitoring policies create --policy-from-file=policy.yaml

# Create Cloud Armor rule
gcloud compute security-policies rules create PRIORITY \
  --security-policy=POLICY --expression="EXPR" --action=ACTION

# Snapshot disk for forensics
gcloud compute disks snapshot DISK --zone=ZONE

# List container vulnerabilities
gcloud container images describe IMAGE --show-package-vulnerability

# Export Binary Authorization policy
gcloud container binauthz policy export

# Create log sink
gcloud logging sinks create SINK_NAME DESTINATION --log-filter="FILTER"

# Create SCC notification
gcloud scc notifications create NOTIF_NAME --organization=ORG \
  --pubsub-topic=TOPIC --filter="FILTER"
```

### Scenario-Based Exam Strategy

When you see scenario questions:
1. **Identify the security domain**: Is it detection, investigation, containment, or prevention?
2. **Check for keywords**: "real-time" = Cloud Logging, "vulnerability" = SCC, "block" = Cloud Armor
3. **Consider scope**: Organization-level = SCC/Chronicle, Application-level = Cloud Armor
4. **Think defense in depth**: Correct answers often involve multiple layers
5. **Compliance requirements**: If mentioned, all controls must align with framework requirements
6. **Cost optimization**: If two solutions work, choose simpler/cheaper (SCC Standard vs Premium)
7. **Operational overhead**: Prefer managed solutions over custom implementations

### Red Flags in Answer Choices

Answers are WRONG if they suggest:
- Using Cloud Armor for internal resources (only works with external LBs)
- Relying on Standard SCC for threat detection (needs Premium for ETD)
- Storing audit logs only in Cloud Logging (retention too short for compliance)
- Implementing WAF rules without testing in preview mode first
- Using Binary Authorization without attestor configuration
- Enabling 100% VPC Flow Logs sampling without cost consideration
- Deleting resources before forensic evidence collection
- Using resource labels instead of security marks for SCC filtering
- Exporting logs without granting sink service account permissions
- Assuming Data Access logs are enabled by default

## Study Tips

### Hands-On Practice

1. **Set up Security Command Center**
   - Enable SCC Premium trial
   - Review all finding categories
   - Practice using security marks
   - Create custom sources and findings
   - Set up notification configurations

2. **Configure comprehensive logging**
   - Enable all four audit log types
   - Create log sinks to BigQuery, GCS, and Pub/Sub
   - Build custom log-based metrics
   - Set up alerting policies with multiple conditions
   - Test notification channels

3. **Deploy Cloud Armor policies**
   - Create policies with preconfigured WAF rules
   - Test rate limiting configurations
   - Practice geo-blocking
   - Use preview mode extensively
   - Analyze blocked request logs

4. **Implement Binary Authorization**
   - Create attestors and notes
   - Sign container images
   - Test cluster-specific policies
   - Use dry-run mode for validation
   - Create exception patterns

5. **Conduct incident response drills**
   - Practice snapshot creation
   - Export logs for specific time windows
   - Revoke IAM permissions quickly
   - Apply emergency firewall rules
   - Document investigation findings

6. **Vulnerability management**
   - Push container images and review scan results
   - Set up OS patch management schedules
   - Run Web Security Scanner
   - Check for common misconfigurations
   - Generate compliance reports

7. **Practice BigQuery log analysis**
   - Write queries for common security scenarios
   - Detect privilege escalation patterns
   - Find unusual data access
   - Identify failed authentication attempts
   - Correlate events across log types

### Key Concepts to Master

1. **Four types of audit logs**: Admin Activity, Data Access, System Event, Policy Denied
2. **SCC Premium features**: Event Threat Detection, Container Threat Detection, Security Health Analytics
3. **Cloud Armor rule evaluation order**: Lowest priority number evaluated first
4. **Binary Authorization enforcement modes**: ENFORCED_BLOCK_AND_AUDIT_LOG, DRYRUN_AUDIT_LOG_ONLY
5. **Log sink destinations**: Cloud Storage, BigQuery, Pub/Sub, Cloud Logging bucket
6. **Security marks**: Custom key-value pairs for SCC assets and findings
7. **Incident response phases**: Preparation, Detection, Containment, Eradication, Recovery, Post-Incident
8. **Chronicle vs SCC**: SIEM vs CSPM, log analysis vs configuration/vulnerability scanning
9. **Compliance log retention**: PCI-DSS (1 year), HIPAA (6 years), SOC 2 (varies)
10. **VPC Flow Logs components**: source IP/port, dest IP/port, protocol, bytes, packets

### Exam Focus Areas

The exam heavily tests:
- **Incident response procedures** (20-25% of questions)
- **Cloud Logging configuration and analysis** (20% of questions)
- **Security Command Center operations** (15-20% of questions)
- **Cloud Armor WAF configuration** (10-15% of questions)
- **Vulnerability and patch management** (10-15% of questions)
- **Compliance monitoring** (10% of questions)
- **Threat detection and response** (10% of questions)

### Common Question Patterns

1. **"Company needs to detect..."** → Identify correct detection tool (SCC, Chronicle, Cloud Logging)
2. **"Company needs to block..."** → Cloud Armor configuration
3. **"Security incident occurred..."** → Incident response steps in correct order
4. **"Compliance audit requires..."** → Enable appropriate logs and retention
5. **"Container deployment failing..."** → Binary Authorization troubleshooting
6. **"Too many false positive alerts..."** → Alert tuning with security marks or filtering
7. **"Investigate suspicious activity..."** → Cloud Logging queries or Chronicle hunting
8. **"Automate security response..."** → Cloud Functions triggered by SCC notifications

### Resources

- [Security Command Center Documentation](https://cloud.google.com/security-command-center/docs)
- [Cloud Logging Documentation](https://cloud.google.com/logging/docs)
- [Cloud Armor Documentation](https://cloud.google.com/armor/docs)
- [Binary Authorization Documentation](https://cloud.google.com/binary-authorization/docs)
- [Chronicle Documentation](https://cloud.google.com/chronicle/docs)
- [GCP Incident Response Guide](https://cloud.google.com/architecture/incident-response)
- [Container Security Best Practices](https://cloud.google.com/architecture/best-practices-for-operating-containers)
- [Security Operations Framework](https://cloud.google.com/architecture/framework/security)
- [PCI-DSS on GCP](https://cloud.google.com/security/compliance/pci-dss)
- [HIPAA on GCP](https://cloud.google.com/security/compliance/hipaa)

### Final Exam Tips

1. **Read scenarios carefully**: Identify requirements, constraints, and current state before looking at answers
2. **Eliminate wrong answers**: Often 2 answers are clearly wrong, choose between remaining 2
3. **Watch for "MOST" and "LEAST"**: Question asks for optimal solution, not just a working solution
4. **Time management**: Don't spend >2 minutes per question, flag and return to difficult ones
5. **Hands-on experience wins**: If you've actually configured it, you'll recognize correct answer pattern
6. **Multiple correct approaches**: Choose simplest or most cost-effective if multiple solutions work
7. **Security best practices**: When in doubt, choose defense-in-depth, least privilege, and automation
8. **Compliance trumps convenience**: If compliance mentioned, solution must fully satisfy requirements
