# Network Security and Monitoring - GCP Professional Cloud Network Engineer

## Overview

Comprehensive coverage of network security architecture, VPC Service Controls, Cloud IDS, firewall management, Cloud Armor, packet mirroring, Network Intelligence Center, monitoring, troubleshooting, and security best practices for the Professional Cloud Network Engineer exam.

**Exam Weight**: High priority - appears in 20-25% of questions
**Key Topics**: Security architecture, VPC-SC, IDS/IPS, monitoring tools, incident response

## Network Security Architecture

### Defense in Depth Strategy

**Layered Security Model**:
```
Layer 1: Perimeter (Cloud Armor, Cloud CDN)
Layer 2: Network (VPC Firewall, VPC Service Controls)
Layer 3: Host (OS firewall, antivirus, hardening)
Layer 4: Application (WAF rules, input validation)
Layer 5: Data (encryption at rest and in transit)
```

**Architecture Diagram**:
```
Internet
    |
    v
Cloud Armor (Layer 7 protection)
    |
    v
External Load Balancer (HTTPS termination)
    |
    v
VPC Service Controls Perimeter
    |
    v
VPC Firewall Rules (Network layer)
    |
    v
+------------------------+
|  DMZ Subnet            |
|  - Web Tier            |
|  - Bastion Hosts       |
|  - NAT Gateway         |
+------------------------+
    |
    v (through internal firewall rules)
    |
+------------------------+
|  Application Subnet    |
|  - App Servers         |
|  - Service Accounts    |
+------------------------+
    |
    v (through strict firewall)
    |
+------------------------+
|  Data Subnet           |
|  - Databases           |
|  - No external access  |
|  - Private Service     |
+------------------------+
```

### Security Zones

**Zone-Based Architecture**:
```bash
# DMZ Zone - Public-facing services
gcloud compute networks subnets create dmz-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all \
  --enable-private-ip-google-access

# Application Zone - Internal services
gcloud compute networks subnets create app-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.10.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all \
  --purpose=PRIVATE

# Data Zone - Database tier
gcloud compute networks subnets create data-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.20.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all \
  --purpose=PRIVATE

# Management Zone - Administrative access
gcloud compute networks subnets create mgmt-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.100.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all
```

**Zone Firewall Rules**:
```bash
# DMZ: Allow inbound from internet to web services
gcloud compute firewall-rules create dmz-allow-web \
  --network=prod-vpc \
  --direction=ingress \
  --action=allow \
  --source-ranges=0.0.0.0/0 \
  --rules=tcp:80,tcp:443 \
  --target-tags=dmz-web \
  --priority=1000

# DMZ to App: Allow specific ports only
gcloud compute firewall-rules create dmz-to-app \
  --network=prod-vpc \
  --direction=ingress \
  --action=allow \
  --source-tags=dmz-web \
  --target-tags=app-servers \
  --rules=tcp:8080,tcp:8443 \
  --priority=1000

# App to Data: Database connections only
gcloud compute firewall-rules create app-to-data \
  --network=prod-vpc \
  --direction=ingress \
  --action=allow \
  --source-tags=app-servers \
  --target-tags=database \
  --rules=tcp:3306,tcp:5432 \
  --priority=1000

# Management: Deny all by default
gcloud compute firewall-rules create mgmt-deny-all \
  --network=prod-vpc \
  --direction=ingress \
  --action=deny \
  --source-ranges=0.0.0.0/0 \
  --rules=all \
  --target-tags=mgmt \
  --priority=65534
```

### Identity-Aware Proxy (IAP)

**IAP for TCP Forwarding**:
```bash
# Enable IAP API
gcloud services enable iap.googleapis.com

# Create firewall rule for IAP
gcloud compute firewall-rules create allow-iap-ssh \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:22,tcp:3389 \
  --source-ranges=35.235.240.0/20 \
  --target-service-accounts=bastion-sa@project-id.iam.gserviceaccount.com

# Grant IAP tunnel user role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:admin@example.com \
  --role=roles/iap.tunnelResourceAccessor

# Connect via IAP
gcloud compute ssh instance-name \
  --zone=us-central1-a \
  --tunnel-through-iap

# Forward port via IAP
gcloud compute start-iap-tunnel instance-name 3306 \
  --local-host-port=localhost:3306 \
  --zone=us-central1-a
```

**IAP for HTTPS**:
```bash
# Configure backend service for IAP
gcloud compute backend-services update backend-service-name \
  --global \
  --iap=enabled

# Set OAuth credentials
gcloud iap oauth-brands create --application-title="My App" \
  --support-email=support@example.com

gcloud iap oauth-clients create projects/PROJECT_ID/brands/BRAND_ID \
  --display_name="IAP Client"

# Grant access to specific users
gcloud iap web add-iam-policy-binding \
  --resource-type=backend-services \
  --resource=backend-service-name \
  --member=user:user@example.com \
  --role=roles/iap.httpsResourceAccessor
```

**IAP with Load Balancer**:
```yaml
# Complete IAP setup
apiVersion: networking.gke.io/v1
kind: BackendConfig
metadata:
  name: iap-backend-config
spec:
  iap:
    enabled: true
    oauthclientCredentials:
      secretName: oauth-client-secret
  securityPolicy:
    name: cloud-armor-policy
  timeoutSec: 30
  connectionDraining:
    drainingTimeoutSec: 60
```

### Bastion Hosts

**Hardened Bastion Configuration**:
```bash
# Create bastion service account
gcloud iam service-accounts create bastion-sa \
  --display-name="Bastion Host Service Account"

# Grant minimal permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:bastion-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/logging.logWriter

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:bastion-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/monitoring.metricWriter

# Create bastion instance with minimal access
gcloud compute instances create bastion-host \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --subnet=mgmt-subnet \
  --no-address \
  --service-account=bastion-sa@PROJECT_ID.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --boot-disk-size=10GB \
  --boot-disk-type=pd-standard \
  --metadata=enable-oslogin=TRUE \
  --shielded-secure-boot \
  --shielded-vtpm \
  --shielded-integrity-monitoring

# Firewall rule: Only allow IAP access
gcloud compute firewall-rules create bastion-allow-iap \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --target-service-accounts=bastion-sa@PROJECT_ID.iam.gserviceaccount.com \
  --priority=1000

# Allow bastion to connect to internal resources
gcloud compute firewall-rules create allow-bastion-to-internal \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:22,tcp:3389 \
  --source-service-accounts=bastion-sa@PROJECT_ID.iam.gserviceaccount.com \
  --priority=1000
```

**Bastion Startup Script** (hardening):
```bash
#!/bin/bash
# Update system
apt-get update && apt-get upgrade -y

# Install essential tools
apt-get install -y tcpdump netcat-openbsd dnsutils

# Configure SSH hardening
cat >> /etc/ssh/sshd_config <<EOF
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
Protocol 2
EOF

# Restart SSH
systemctl restart sshd

# Configure auditd for logging
apt-get install -y auditd
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/group -p wa -k group_changes
auditctl -w /var/log/auth.log -p wa -k auth_log

# Enable automatic security updates
apt-get install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

# Configure iptables logging
iptables -A INPUT -j LOG --log-prefix "INPUT: " --log-level 4
iptables -A OUTPUT -j LOG --log-prefix "OUTPUT: " --log-level 4

# Send logs to Cloud Logging
systemctl enable google-fluentd
systemctl start google-fluentd
```

## VPC Service Controls

### Service Perimeters

**VPC Service Controls Architecture**:
```
+-----------------------------------------+
|  Organization                           |
|  +-----------------------------------+  |
|  | Access Policy                     |  |
|  |  +-----------------------------+  |  |
|  |  | Service Perimeter (PROD)    |  |  |
|  |  |  - Project A (prod)         |  |  |
|  |  |  - Project B (prod)         |  |  |
|  |  |  - Protected Services:      |  |  |
|  |  |    * storage.googleapis.com |  |  |
|  |  |    * bigquery.googleapis.com|  |  |
|  |  |    * compute.googleapis.com |  |  |
|  |  +-----------------------------+  |  |
|  |                                   |  |
|  |  +-----------------------------+  |  |
|  |  | Service Perimeter (DEV)     |  |  |
|  |  |  - Project C (dev)          |  |  |
|  |  |  - Project D (dev)          |  |  |
|  |  +-----------------------------+  |  |
|  +-----------------------------------+  |
+-----------------------------------------+
```

**Create Access Policy**:
```bash
# Create access policy (one per organization)
gcloud access-context-manager policies create \
  --organization=ORGANIZATION_ID \
  --title="Corporate Access Policy"

# Get policy number
POLICY_ID=$(gcloud access-context-manager policies list \
  --organization=ORGANIZATION_ID \
  --format="value(name)")
```

**Create Service Perimeter**:
```bash
# Create perimeter configuration file
cat > perimeter-config.yaml <<EOF
name: accessPolicies/${POLICY_ID}/servicePerimeters/prod_perimeter
title: Production Perimeter
description: Protects production projects and services
perimeterType: PERIMETER_TYPE_REGULAR
status:
  resources:
    - projects/123456789  # prod-project-1
    - projects/987654321  # prod-project-2
  restrictedServices:
    - storage.googleapis.com
    - bigquery.googleapis.com
    - compute.googleapis.com
    - sqladmin.googleapis.com
    - container.googleapis.com
  accessLevels:
    - accessPolicies/${POLICY_ID}/accessLevels/corp_network
    - accessPolicies/${POLICY_ID}/accessLevels/trusted_users
  vpcAccessibleServices:
    enableRestriction: true
    allowedServices:
      - storage.googleapis.com
      - bigquery.googleapis.com
EOF

# Create perimeter
gcloud access-context-manager perimeters create prod_perimeter \
  --policy=${POLICY_ID} \
  --title="Production Perimeter" \
  --resources=projects/123456789,projects/987654321 \
  --restricted-services=storage.googleapis.com,bigquery.googleapis.com \
  --perimeter-type=regular
```

### Access Levels

**Device-Based Access Level**:
```bash
# Create access level for corporate devices
cat > corp-devices-access-level.yaml <<EOF
name: accessPolicies/${POLICY_ID}/accessLevels/corp_devices
title: Corporate Managed Devices
description: Access from managed devices only
basic:
  conditions:
    - devicePolicy:
        requireScreenlock: true
        requireAdminApproval: true
        requireCorpOwned: true
        osConstraints:
          - osType: DESKTOP_CHROME_OS
            minimumVersion: "100.0.0"
          - osType: DESKTOP_WINDOWS
            minimumVersion: "10.0.0"
          - osType: DESKTOP_MAC
            minimumVersion: "10.15.0"
EOF

gcloud access-context-manager levels create corp_devices \
  --policy=${POLICY_ID} \
  --title="Corporate Managed Devices" \
  --basic-level-spec=corp-devices-access-level.yaml
```

**IP-Based Access Level**:
```bash
# Access from corporate network
cat > corp-network-access-level.yaml <<EOF
name: accessPolicies/${POLICY_ID}/accessLevels/corp_network
title: Corporate Network
description: Access from corporate IP ranges
basic:
  conditions:
    - ipSubnetworks:
        - "203.0.113.0/24"    # HQ
        - "198.51.100.0/24"   # Branch office
        - "192.0.2.0/24"      # Data center
  combiningFunction: OR
EOF

gcloud access-context-manager levels create corp_network \
  --policy=${POLICY_ID} \
  --title="Corporate Network" \
  --basic-level-spec=corp-network-access-level.yaml
```

**User-Based Access Level**:
```bash
# Access for specific users/groups
cat > trusted-users-access-level.yaml <<EOF
name: accessPolicies/${POLICY_ID}/accessLevels/trusted_users
title: Trusted Users
description: Verified users with proper roles
basic:
  conditions:
    - members:
        - "user:admin@example.com"
        - "group:security-team@example.com"
        - "serviceAccount:trusted-sa@project.iam.gserviceaccount.com"
EOF

gcloud access-context-manager levels create trusted_users \
  --policy=${POLICY_ID} \
  --title="Trusted Users" \
  --basic-level-spec=trusted-users-access-level.yaml
```

**Complex Access Level** (combining multiple conditions):
```yaml
name: accessPolicies/${POLICY_ID}/accessLevels/complex_access
title: Complex Access Requirements
description: Multiple conditions with AND logic
basic:
  conditions:
    - ipSubnetworks:
        - "203.0.113.0/24"
      devicePolicy:
        requireScreenlock: true
        requireCorpOwned: true
      members:
        - "group:engineering@example.com"
      regions:
        - "US"
        - "GB"
  combiningFunction: AND
```

### Ingress and Egress Policies

**Ingress Policy** (allow data into perimeter):
```bash
# Allow ingress from specific sources
cat > ingress-policy.yaml <<EOF
ingressPolicies:
  - ingressFrom:
      sources:
        - accessLevel: accessPolicies/${POLICY_ID}/accessLevels/corp_network
      identities:
        - serviceAccount:external-service@project.iam.gserviceaccount.com
        - user:external-user@example.com
      identityType: ANY_IDENTITY
    ingressTo:
      resources:
        - projects/123456789
      operations:
        - serviceName: storage.googleapis.com
          methodSelectors:
            - method: "google.storage.objects.get"
            - method: "google.storage.objects.list"
        - serviceName: bigquery.googleapis.com
          methodSelectors:
            - method: "google.cloud.bigquery.v2.JobService.InsertJob"
EOF

# Update perimeter with ingress policy
gcloud access-context-manager perimeters update prod_perimeter \
  --policy=${POLICY_ID} \
  --add-ingress-policies=ingress-policy.yaml
```

**Egress Policy** (allow data out of perimeter):
```bash
# Allow egress to specific destinations
cat > egress-policy.yaml <<EOF
egressPolicies:
  - egressFrom:
      identities:
        - serviceAccount:data-pipeline@project.iam.gserviceaccount.com
      identityType: ANY_SERVICE_ACCOUNT
    egressTo:
      resources:
        - projects/external-project-id
      operations:
        - serviceName: storage.googleapis.com
          methodSelectors:
            - method: "google.storage.objects.create"
            - method: "google.storage.buckets.get"
      externalResources:
        - "https://external-api.example.com/*"
EOF

# Update perimeter with egress policy
gcloud access-context-manager perimeters update prod_perimeter \
  --policy=${POLICY_ID} \
  --add-egress-policies=egress-policy.yaml
```

**Perimeter Bridge** (connect two perimeters):
```bash
# Create bridge between production and development
gcloud access-context-manager perimeters create prod_dev_bridge \
  --policy=${POLICY_ID} \
  --title="Production to Development Bridge" \
  --perimeter-type=bridge \
  --resources=projects/123456789,projects/987654321 \
  --perimeter-bridges=prod_perimeter,dev_perimeter
```

### Dry Run Mode

**Test VPC-SC Changes**:
```bash
# Create spec with dry-run configuration
cat > dry-run-spec.yaml <<EOF
name: accessPolicies/${POLICY_ID}/servicePerimeters/prod_perimeter
status:
  resources:
    - projects/123456789
  restrictedServices:
    - storage.googleapis.com
  accessLevels:
    - accessPolicies/${POLICY_ID}/accessLevels/corp_network
spec:  # Dry-run configuration
  resources:
    - projects/123456789
    - projects/555555555  # New project to test
  restrictedServices:
    - storage.googleapis.com
    - bigquery.googleapis.com  # New service to test
  accessLevels:
    - accessPolicies/${POLICY_ID}/accessLevels/corp_network
EOF

# Update perimeter with dry-run spec
gcloud access-context-manager perimeters update prod_perimeter \
  --policy=${POLICY_ID} \
  --spec-file=dry-run-spec.yaml
```

**Monitor Dry Run Violations**:
```sql
-- Query dry-run violations in BigQuery
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.resourceName as resource,
  protoPayload.serviceName as service,
  protoPayload.methodName as method,
  protoPayload.metadata.dryRun as dry_run_mode,
  protoPayload.metadata.violationReason as violation
FROM `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE protoPayload.metadata.dryRun = true
  AND protoPayload.metadata.securityPolicyInfo.servicePerimeterName LIKE '%prod_perimeter%'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
ORDER BY timestamp DESC;

-- Count violations by service
SELECT
  protoPayload.serviceName as service,
  COUNT(*) as violation_count,
  ARRAY_AGG(DISTINCT protoPayload.authenticationInfo.principalEmail) as affected_users
FROM `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE protoPayload.metadata.dryRun = true
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY service
ORDER BY violation_count DESC;
```

**Analyze and Apply Dry Run**:
```bash
# Export dry-run logs
gcloud logging read \
  "protoPayload.metadata.dryRun=true AND protoPayload.metadata.securityPolicyInfo.servicePerimeterName:prod_perimeter" \
  --format=json \
  --limit=1000 > dry-run-violations.json

# After analysis, promote spec to status (apply changes)
gcloud access-context-manager perimeters update prod_perimeter \
  --policy=${POLICY_ID} \
  --promote-spec-to-status

# Or rollback dry-run changes
gcloud access-context-manager perimeters update prod_perimeter \
  --policy=${POLICY_ID} \
  --clear-spec
```

### VPC-SC Troubleshooting

**Common Issues**:

1. **Access Denied Errors**:
```bash
# Check perimeter configuration
gcloud access-context-manager perimeters describe prod_perimeter \
  --policy=${POLICY_ID}

# Verify project is in perimeter
gcloud access-context-manager perimeters describe prod_perimeter \
  --policy=${POLICY_ID} \
  --format="get(status.resources)"

# Check access levels
gcloud access-context-manager levels list \
  --policy=${POLICY_ID}

gcloud access-context-manager levels describe corp_network \
  --policy=${POLICY_ID}
```

2. **Query VPC-SC Violations**:
```sql
-- Recent VPC-SC violations
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.resourceName as resource,
  protoPayload.serviceName as service,
  protoPayload.methodName as method,
  protoPayload.status.message as error_message,
  protoPayload.metadata.violationReason as reason,
  protoPayload.metadata.securityPolicyInfo.servicePerimeterName as perimeter
FROM `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE protoPayload.status.code = 7  # PERMISSION_DENIED
  AND protoPayload.metadata.securityPolicyInfo.servicePerimeterName IS NOT NULL
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
ORDER BY timestamp DESC
LIMIT 100;

-- Top violation sources
SELECT
  protoPayload.authenticationInfo.principalEmail as user,
  protoPayload.serviceName as service,
  COUNT(*) as violation_count
FROM `project.dataset.cloudaudit_googleapis_com_data_access_*`
WHERE protoPayload.status.code = 7
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
GROUP BY user, service
ORDER BY violation_count DESC;
```

3. **Service Account Access Issues**:
```bash
# Grant service account access to perimeter
gcloud access-context-manager levels create sa_access \
  --policy=${POLICY_ID} \
  --title="Service Account Access" \
  --basic-level-spec=<(cat <<EOF
basic:
  conditions:
    - members:
        - serviceAccount:my-sa@project.iam.gserviceaccount.com
EOF
)

# Add to perimeter
gcloud access-context-manager perimeters update prod_perimeter \
  --policy=${POLICY_ID} \
  --add-access-levels=sa_access
```

4. **Debug VPC-SC with curl**:
```bash
# Test API call with verbose output
curl -v -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://storage.googleapis.com/storage/v1/b/BUCKET_NAME/o"

# Look for X-Goog-VPC-Service-Context header in response
# This indicates VPC-SC enforcement
```

## Cloud IDS (Intrusion Detection System)

### Overview

Cloud IDS is a managed network threat detection service powered by Palo Alto Networks technologies.

**Architecture**:
```
Internet/VPC Traffic
        |
        v
Packet Mirroring Policy
        |
        v
Cloud IDS Endpoint (receives mirrored packets)
        |
        v
Threat Detection Engine (Palo Alto Networks)
        |
        v
Cloud Logging + Security Command Center
        |
        v
Alerts & Automated Response
```

### Cloud IDS Setup

**Enable APIs and Create Endpoint**:
```bash
# Enable Cloud IDS API
gcloud services enable ids.googleapis.com

# Create IDS endpoint
gcloud ids endpoints create ids-endpoint-prod \
  --network=projects/PROJECT_ID/global/networks/prod-vpc \
  --zone=us-central1-a \
  --severity=INFORMATIONAL \
  --enable-traffic-logs

# Describe endpoint
gcloud ids endpoints describe ids-endpoint-prod \
  --zone=us-central1-a

# Get endpoint forwarding rule for packet mirroring
IDS_FORWARDING_RULE=$(gcloud ids endpoints describe ids-endpoint-prod \
  --zone=us-central1-a \
  --format="value(endpointForwardingRule)")
```

**Configure Packet Mirroring to IDS**:
```bash
# Create packet mirroring policy pointing to IDS
gcloud compute packet-mirrorings create mirror-to-ids \
  --region=us-central1 \
  --network=prod-vpc \
  --collector-ilb=${IDS_FORWARDING_RULE} \
  --mirrored-subnets=app-subnet,web-subnet \
  --filter-protocols=tcp,udp,icmp

# Verify mirroring policy
gcloud compute packet-mirrorings describe mirror-to-ids \
  --region=us-central1
```

### Threat Signatures

**Severity Levels**:
- **CRITICAL**: Immediate action required (active exploits, ransomware)
- **HIGH**: Serious threats (known vulnerabilities, suspicious activity)
- **MEDIUM**: Potential threats (policy violations, anomalies)
- **LOW**: Information gathering attempts
- **INFORMATIONAL**: Benign traffic patterns

**Common Threat Categories**:
```
1. Malware Detection
   - Ransomware
   - Trojans
   - Spyware
   - Backdoors

2. Intrusion Attempts
   - SQL injection
   - Cross-site scripting (XSS)
   - Remote code execution
   - Buffer overflow attacks

3. Network Reconnaissance
   - Port scanning
   - Network mapping
   - Vulnerability scanning
   - Banner grabbing

4. Command and Control (C2)
   - Botnet activity
   - C2 communication
   - Data exfiltration
   - Covert channels

5. Policy Violations
   - Unauthorized protocols
   - Suspicious DNS queries
   - Abnormal traffic patterns
   - Protocol anomalies
```

**Update IDS Endpoint Severity**:
```bash
# Update to only alert on high severity
gcloud ids endpoints update ids-endpoint-prod \
  --zone=us-central1-a \
  --severity=HIGH \
  --async

# Disable traffic logs to reduce cost
gcloud ids endpoints update ids-endpoint-prod \
  --zone=us-central1-a \
  --no-enable-traffic-logs
```

### IDS Threat Detection

**Query IDS Alerts in BigQuery**:
```sql
-- Export IDS logs to BigQuery first
-- Then query threats
SELECT
  timestamp,
  jsonPayload.alert.source_ip,
  jsonPayload.alert.destination_ip,
  jsonPayload.alert.threat_name,
  jsonPayload.alert.severity,
  jsonPayload.alert.category,
  jsonPayload.alert.application,
  jsonPayload.alert.direction
FROM `project.dataset.ids_threat_*`
WHERE jsonPayload.alert.severity IN ('CRITICAL', 'HIGH')
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
ORDER BY timestamp DESC;

-- Top threat sources
SELECT
  jsonPayload.alert.source_ip as attacker_ip,
  COUNT(*) as threat_count,
  ARRAY_AGG(DISTINCT jsonPayload.alert.threat_name LIMIT 10) as threats
FROM `project.dataset.ids_threat_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY attacker_ip
ORDER BY threat_count DESC
LIMIT 20;

-- Threat trends over time
SELECT
  DATE(timestamp) as date,
  jsonPayload.alert.category as threat_category,
  COUNT(*) as count
FROM `project.dataset.ids_threat_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY date, threat_category
ORDER BY date DESC, count DESC;

-- Critical threats requiring immediate action
SELECT
  timestamp,
  jsonPayload.alert.source_ip,
  jsonPayload.alert.destination_ip,
  jsonPayload.alert.destination_port,
  jsonPayload.alert.threat_name,
  jsonPayload.alert.uri,
  jsonPayload.alert.application
FROM `project.dataset.ids_threat_*`
WHERE jsonPayload.alert.severity = 'CRITICAL'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
ORDER BY timestamp DESC;
```

### IDS Integration with Security Command Center

**Configure SCC Integration**:
```bash
# Enable Security Command Center
gcloud services enable securitycenter.googleapis.com

# IDS findings automatically sent to SCC
# View findings
gcloud scc findings list ORGANIZATION_ID \
  --filter="category='IDS_THREAT'" \
  --format=json

# Create notification config for IDS findings
gcloud scc notifications create ids-critical-alerts \
  --organization=ORGANIZATION_ID \
  --description="Critical IDS alerts" \
  --pubsub-topic=projects/PROJECT_ID/topics/ids-alerts \
  --filter="severity='CRITICAL' AND category='IDS_THREAT'"
```

**Automated Response with Cloud Functions**:
```python
# Cloud Function to block IPs on critical threats
import base64
import json
from google.cloud import compute_v1

def block_malicious_ip(event, context):
    """Block IP address when critical IDS alert received"""

    pubsub_message = base64.b64decode(event['data']).decode('utf-8')
    alert = json.loads(pubsub_message)

    # Extract threat info
    source_ip = alert['jsonPayload']['alert']['source_ip']
    threat_name = alert['jsonPayload']['alert']['threat_name']
    severity = alert['jsonPayload']['alert']['severity']

    if severity == 'CRITICAL':
        # Create deny firewall rule
        firewall_client = compute_v1.FirewallsClient()

        firewall_rule = compute_v1.Firewall()
        firewall_rule.name = f"block-{source_ip.replace('.', '-')}"
        firewall_rule.network = "projects/PROJECT_ID/global/networks/prod-vpc"
        firewall_rule.direction = "INGRESS"
        firewall_rule.priority = 100
        firewall_rule.source_ranges = [source_ip]
        firewall_rule.denied = [compute_v1.Denied(I_p_protocol="all")]
        firewall_rule.description = f"Auto-block: {threat_name}"

        operation = firewall_client.insert(
            project="PROJECT_ID",
            firewall_resource=firewall_rule
        )

        print(f"Blocked {source_ip} due to {threat_name}")

        # Send notification
        # (add your notification logic here)

    return 'OK'
```

### IDS Monitoring and Tuning

**Create Monitoring Dashboard**:
```yaml
# monitoring-dashboard.yaml
displayName: "Cloud IDS Security Dashboard"
mosaicLayout:
  columns: 12
  tiles:
    - width: 6
      height: 4
      widget:
        title: "IDS Threats by Severity"
        xyChart:
          dataSets:
            - timeSeriesQuery:
                timeSeriesFilter:
                  filter: 'resource.type="ids.googleapis.com/Endpoint"'
                  aggregation:
                    alignmentPeriod: 60s
                    perSeriesAligner: ALIGN_RATE
                    groupByFields:
                      - "jsonPayload.alert.severity"

    - width: 6
      height: 4
      widget:
        title: "Top Attacked Assets"
        xyChart:
          dataSets:
            - timeSeriesQuery:
                timeSeriesFilter:
                  filter: 'resource.type="ids.googleapis.com/Endpoint"'
                  aggregation:
                    groupByFields:
                      - "jsonPayload.alert.destination_ip"
```

**Alert Policy for Critical Threats**:
```python
from google.cloud import monitoring_v3

def create_ids_alert_policy(project_id):
    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    alert_policy = monitoring_v3.AlertPolicy(
        display_name="Critical IDS Threats",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Critical threat detected",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='resource.type="ids.googleapis.com/Endpoint" AND jsonPayload.alert.severity="CRITICAL"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0,
                    duration={"seconds": 60},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_COUNT,
                        )
                    ],
                ),
            )
        ],
        notification_channels=[],  # Add your channels
        alert_strategy=monitoring_v3.AlertPolicy.AlertStrategy(
            auto_close={"seconds": 3600}
        ),
    )

    policy = client.create_alert_policy(name=project_name, alert_policy=alert_policy)
    return policy
```

## Packet Mirroring

### Overview

Packet Mirroring clones network traffic for analysis, monitoring, and security inspection.

**Use Cases**:
- Intrusion detection with Cloud IDS
- Network troubleshooting and analysis
- Application performance monitoring
- Compliance and forensics
- Third-party security tool integration

**Architecture**:
```
Source VMs/Subnets
        |
        v
Packet Mirroring Policy
        |
        +--> Filters (protocols, CIDR ranges)
        |
        v
Clone packets
        |
        v
Internal Load Balancer (collector)
        |
        v
Collector VMs (IDS, packet analyzer, monitoring tools)
```

### Packet Mirroring Configuration

**Complete Setup**:
```bash
# 1. Create collector internal load balancer
gcloud compute health-checks create tcp collector-health-check \
  --port=12345 \
  --region=us-central1

gcloud compute backend-services create collector-backend \
  --load-balancing-scheme=INTERNAL \
  --protocol=TCP \
  --health-checks=collector-health-check \
  --region=us-central1

# 2. Add collector instances to backend
gcloud compute backend-services add-backend collector-backend \
  --instance-group=collector-ig \
  --instance-group-zone=us-central1-a \
  --region=us-central1

# 3. Create forwarding rule
gcloud compute forwarding-rules create collector-ilb \
  --load-balancing-scheme=INTERNAL \
  --network=prod-vpc \
  --subnet=mgmt-subnet \
  --ip-protocol=TCP \
  --ports=all \
  --backend-service=collector-backend \
  --region=us-central1 \
  --is-mirroring-collector

# 4. Create packet mirroring policy
gcloud compute packet-mirrorings create production-mirror \
  --region=us-central1 \
  --network=prod-vpc \
  --collector-ilb=projects/PROJECT_ID/regions/us-central1/forwardingRules/collector-ilb \
  --mirrored-subnets=web-subnet,app-subnet \
  --mirrored-tags=mirror-enabled \
  --filter-protocols=tcp,udp \
  --filter-cidr-ranges=10.0.0.0/8
```

**Mirror Specific Instances**:
```bash
# Mirror by instance
gcloud compute packet-mirrorings create instance-mirror \
  --region=us-central1 \
  --network=prod-vpc \
  --collector-ilb=projects/PROJECT_ID/regions/us-central1/forwardingRules/collector-ilb \
  --mirrored-instances=web-1,web-2,web-3

# Mirror by network tag
gcloud compute packet-mirrorings create tag-mirror \
  --region=us-central1 \
  --network=prod-vpc \
  --collector-ilb=projects/PROJECT_ID/regions/us-central1/forwardingRules/collector-ilb \
  --mirrored-tags=production,database
```

### Collector Instance Setup

**Install and Configure tcpdump/Wireshark**:
```bash
# Collector VM startup script
#!/bin/bash

# Install packet analysis tools
apt-get update
apt-get install -y tcpdump wireshark tshark

# Create capture directory
mkdir -p /var/captures
chmod 755 /var/captures

# Start continuous packet capture
cat > /usr/local/bin/packet-capture.sh <<'EOF'
#!/bin/bash
CAPTURE_DIR="/var/captures"
CAPTURE_TIME=3600  # 1 hour rotation
INTERFACE="eth0"

while true; do
    TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    FILENAME="${CAPTURE_DIR}/capture-${TIMESTAMP}.pcap"

    timeout ${CAPTURE_TIME} tcpdump -i ${INTERFACE} \
        -w ${FILENAME} \
        -G ${CAPTURE_TIME} \
        -W 24 \
        -Z root \
        'not port 22'  # Exclude SSH

    # Compress old captures
    find ${CAPTURE_DIR} -name "*.pcap" -mtime +1 -exec gzip {} \;

    # Delete captures older than 7 days
    find ${CAPTURE_DIR} -name "*.pcap.gz" -mtime +7 -delete
done
EOF

chmod +x /usr/local/bin/packet-capture.sh

# Create systemd service
cat > /etc/systemd/system/packet-capture.service <<EOF
[Unit]
Description=Packet Capture Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/packet-capture.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
systemctl daemon-reload
systemctl enable packet-capture
systemctl start packet-capture

# Configure log rotation
cat > /etc/logrotate.d/packet-capture <<EOF
/var/log/packet-capture.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
EOF
```

**Real-time Traffic Analysis**:
```bash
# Monitor HTTP traffic
tcpdump -i eth0 -A 'tcp port 80 or tcp port 443'

# Capture specific protocol
tcpdump -i eth0 -w http-traffic.pcap 'tcp port 80'

# Monitor top talkers
tcpdump -i eth0 -nn -q | \
  awk '{print $3}' | \
  cut -d. -f1-4 | \
  sort | uniq -c | sort -rn | head -20

# Detect SYN flood
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0' -c 1000 | \
  awk '{print $3}' | sort | uniq -c | sort -rn

# Filter suspicious traffic
tcpdump -i eth0 -nn 'tcp[tcpflags] & (tcp-rst) != 0'
```

### Third-Party Integration

**Integrate with Suricata IDS**:
```bash
# Install Suricata on collector instance
apt-get install -y software-properties-common
add-apt-repository -y ppa:oisf/suricata-stable
apt-get update
apt-get install -y suricata

# Configure Suricata
cat > /etc/suricata/suricata.yaml <<EOF
vars:
  address-groups:
    HOME_NET: "[10.0.0.0/8]"
    EXTERNAL_NET: "!$HOME_NET"

default-log-dir: /var/log/suricata/

outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: eve.json
      types:
        - alert
        - http
        - dns
        - tls
        - files
        - ssh

af-packet:
  - interface: eth0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes

detect-engine:
  - profile: high
  - custom-values:
      toclient-groups: 3
      toserver-groups: 25

rule-files:
  - suricata.rules
  - /etc/suricata/rules/*.rules
EOF

# Download and update rules
suricata-update

# Start Suricata
systemctl enable suricata
systemctl start suricata

# Monitor alerts
tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```

**Send Suricata Alerts to Cloud Logging**:
```python
#!/usr/bin/env python3
import json
import time
from google.cloud import logging as cloud_logging

def monitor_suricata_alerts():
    """Send Suricata alerts to Cloud Logging"""

    client = cloud_logging.Client()
    logger = client.logger('suricata-alerts')

    with open('/var/log/suricata/eve.json', 'r') as f:
        # Seek to end of file
        f.seek(0, 2)

        while True:
            line = f.readline()
            if line:
                try:
                    event = json.loads(line)
                    if event.get('event_type') == 'alert':
                        # Log to Cloud Logging
                        logger.log_struct({
                            'severity': event['alert']['severity'],
                            'signature': event['alert']['signature'],
                            'category': event['alert']['category'],
                            'src_ip': event.get('src_ip'),
                            'dest_ip': event.get('dest_ip'),
                            'src_port': event.get('src_port'),
                            'dest_port': event.get('dest_port'),
                            'proto': event.get('proto'),
                            'timestamp': event.get('timestamp')
                        }, severity='WARNING' if event['alert']['severity'] < 3 else 'ERROR')
                except json.JSONDecodeError:
                    pass
            else:
                time.sleep(0.1)

if __name__ == '__main__':
    monitor_suricata_alerts()
```

### Packet Mirroring Best Practices

**Performance Considerations**:
```bash
# Limit mirroring scope to reduce overhead
gcloud compute packet-mirrorings update production-mirror \
  --region=us-central1 \
  --filter-protocols=tcp \
  --filter-cidr-ranges=10.0.10.0/24  # Only app subnet

# Mirror during specific times (use Cloud Scheduler + scripts)
# Enable mirroring
gcloud compute packet-mirrorings update production-mirror \
  --region=us-central1 \
  --enable

# Disable mirroring
gcloud compute packet-mirrorings update production-mirror \
  --region=us-central1 \
  --disable
```

**Cost Optimization**:
```
1. Mirror only critical subnets
2. Use protocol and CIDR filters
3. Enable/disable dynamically based on incidents
4. Use sampling for high-volume traffic
5. Store captures locally, stream alerts only
6. Rotate and compress capture files
```

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

## VPC Flow Logs

### Overview

VPC Flow Logs capture network traffic for security analysis, monitoring, forensics, and cost optimization.

**Flow Log Records Include**:
- Source and destination IPs
- Source and destination ports
- Protocol (TCP, UDP, ICMP, etc.)
- Bytes and packets sent
- RTT (Round Trip Time)
- Start and end timestamps
- GCE instance details
- VPC network information

### Configuration

**Enable Flow Logs with Full Options**:
```bash
# Enable with all metadata
gcloud compute networks subnets update production-subnet \
  --region=us-central1 \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-5-sec \
  --logging-flow-sampling=1.0 \
  --logging-metadata=include-all \
  --logging-filter-expr='true'

# Enable with custom filter (only capture external traffic)
gcloud compute networks subnets update dmz-subnet \
  --region=us-central1 \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-10-sec \
  --logging-flow-sampling=0.5 \
  --logging-metadata=include-all \
  --logging-filter-expr='inIpRange(connection.src_ip, "0.0.0.0/0") || inIpRange(connection.dest_ip, "0.0.0.0/0")'

# Enable with metadata fields optimization
gcloud compute networks subnets update app-subnet \
  --region=us-central1 \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-15-sec \
  --logging-flow-sampling=0.1 \
  --logging-metadata=include-all \
  --logging-metadata-fields=src_instance,dest_instance,src_vpc,dest_vpc,src_location,dest_location
```

**Aggregation Intervals**:
- `interval-5-sec`: Most detailed, highest cost
- `interval-10-sec`: Balanced (default)
- `interval-15-sec`: Cost-optimized

**Sampling Rates**:
- `1.0`: Capture all flows (100%)
- `0.5`: Capture half of flows (50%)
- `0.1`: Capture 10% of flows (10%)
- `0.01`: Capture 1% of flows (1%)

**Metadata Options**:
- `include-all`: All available metadata
- `exclude-all`: Minimal metadata
- `custom`: Specific fields only

### Export to BigQuery

**Create Sink for Flow Logs**:
```bash
# Create BigQuery dataset
bq mk --dataset \
  --location=US \
  --description="VPC Flow Logs" \
  PROJECT_ID:vpc_flow_logs

# Create log sink
gcloud logging sinks create vpc-flow-logs-sink \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/vpc_flow_logs \
  --log-filter='resource.type="gce_subnetwork" AND
    logName:"compute.googleapis.com/vpc_flows"'

# Grant sink service account BigQuery permissions
SERVICE_ACCOUNT=$(gcloud logging sinks describe vpc-flow-logs-sink \
  --format='value(writerIdentity)')

bq show --format=prettyjson PROJECT_ID:vpc_flow_logs > /dev/null

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=$SERVICE_ACCOUNT \
  --role=roles/bigquery.dataEditor
```

**Partition and Cluster Tables**:
```sql
-- Create partitioned and clustered table
CREATE TABLE `project.vpc_flow_logs.flows_partitioned`
PARTITION BY DATE(timestamp)
CLUSTER BY
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  jsonPayload.connection.dest_port
AS
SELECT * FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY);

-- Set partition expiration (cost optimization)
ALTER TABLE `project.vpc_flow_logs.flows_partitioned`
SET OPTIONS (
  partition_expiration_days = 90
);
```

### Analysis and Anomaly Detection

**Top Talkers Analysis**:
```sql
-- Top bandwidth consumers
SELECT
  jsonPayload.connection.src_ip as source_ip,
  jsonPayload.src_instance.vm_name as source_vm,
  jsonPayload.connection.dest_ip as dest_ip,
  SUM(CAST(jsonPayload.bytes_sent AS INT64)) / 1024 / 1024 / 1024 as gb_sent,
  COUNT(*) as connection_count
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY source_ip, source_vm, dest_ip
ORDER BY gb_sent DESC
LIMIT 20;

-- Top destinations by connection count
SELECT
  jsonPayload.connection.dest_ip as destination,
  jsonPayload.connection.dest_port as port,
  COUNT(DISTINCT jsonPayload.connection.src_ip) as unique_sources,
  COUNT(*) as total_connections,
  SUM(CAST(jsonPayload.bytes_sent AS INT64)) / 1024 / 1024 as mb_received
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 6 HOUR)
GROUP BY destination, port
ORDER BY total_connections DESC
LIMIT 50;
```

**Port Scanning Detection**:
```sql
-- Detect port scanning activity
SELECT
  jsonPayload.connection.src_ip as scanner_ip,
  jsonPayload.src_location.region as source_region,
  COUNT(DISTINCT jsonPayload.connection.dest_port) as unique_ports_accessed,
  COUNT(DISTINCT jsonPayload.connection.dest_ip) as unique_targets,
  COUNT(*) as total_attempts,
  ARRAY_AGG(DISTINCT jsonPayload.connection.dest_port LIMIT 20) as ports_scanned,
  MIN(timestamp) as scan_start,
  MAX(timestamp) as scan_end,
  TIMESTAMP_DIFF(MAX(timestamp), MIN(timestamp), SECOND) as scan_duration_sec
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY scanner_ip, source_region
HAVING unique_ports_accessed > 100
  OR (unique_ports_accessed > 20 AND scan_duration_sec < 60)
ORDER BY unique_ports_accessed DESC;
```

**Data Exfiltration Detection**:
```sql
-- Detect unusual outbound data transfers
WITH baseline AS (
  SELECT
    jsonPayload.connection.src_ip,
    AVG(CAST(jsonPayload.bytes_sent AS INT64)) as avg_bytes,
    STDDEV(CAST(jsonPayload.bytes_sent AS INT64)) as stddev_bytes
  FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
  WHERE timestamp BETWEEN TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
    AND TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
  GROUP BY src_ip
),
recent AS (
  SELECT
    jsonPayload.connection.src_ip,
    jsonPayload.src_instance.vm_name,
    jsonPayload.connection.dest_ip,
    SUM(CAST(jsonPayload.bytes_sent AS INT64)) as total_bytes,
    COUNT(*) as connection_count
  FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
  WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
    AND NOT REGEXP_CONTAINS(jsonPayload.connection.dest_ip, r'^10\.|^172\.(1[6-9]|2[0-9]|3[01])\.|^192\.168\.')
  GROUP BY src_ip, vm_name, dest_ip
)
SELECT
  r.src_ip,
  r.vm_name,
  r.dest_ip,
  r.total_bytes / 1024 / 1024 / 1024 as gb_sent,
  r.connection_count,
  b.avg_bytes / 1024 / 1024 as baseline_avg_mb,
  (r.total_bytes - b.avg_bytes) / NULLIF(b.stddev_bytes, 0) as stddev_from_baseline
FROM recent r
LEFT JOIN baseline b ON r.src_ip = b.src_ip
WHERE r.total_bytes > 1024 * 1024 * 1024  -- More than 1 GB
  AND (b.avg_bytes IS NULL OR r.total_bytes > b.avg_bytes + (3 * b.stddev_bytes))
ORDER BY gb_sent DESC;
```

**Denied Connections Analysis**:
```sql
-- Analyze firewall-denied connections
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  jsonPayload.connection.dest_port,
  jsonPayload.connection.protocol,
  jsonPayload.rule_details.reference as blocking_rule,
  COUNT(*) as denied_count,
  MIN(timestamp) as first_seen,
  MAX(timestamp) as last_seen
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND jsonPayload.rule_details.action = 'DENY'
GROUP BY src_ip, dest_ip, dest_port, protocol, blocking_rule
ORDER BY denied_count DESC
LIMIT 100;
```

**Traffic Baseline and Anomalies**:
```sql
-- Create traffic baseline
CREATE OR REPLACE TABLE `project.vpc_flow_logs.traffic_baseline` AS
WITH hourly_stats AS (
  SELECT
    EXTRACT(HOUR FROM timestamp) as hour_of_day,
    EXTRACT(DAYOFWEEK FROM timestamp) as day_of_week,
    jsonPayload.connection.src_ip,
    jsonPayload.connection.dest_ip,
    AVG(CAST(jsonPayload.bytes_sent AS INT64)) as avg_bytes,
    STDDEV(CAST(jsonPayload.bytes_sent AS INT64)) as stddev_bytes,
    COUNT(*) as connection_count
  FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
  WHERE timestamp BETWEEN TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
    AND TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
  GROUP BY hour_of_day, day_of_week, src_ip, dest_ip
)
SELECT * FROM hourly_stats;

-- Detect anomalies compared to baseline
WITH current_hour AS (
  SELECT
    EXTRACT(HOUR FROM CURRENT_TIMESTAMP()) as hour_of_day,
    EXTRACT(DAYOFWEEK FROM CURRENT_TIMESTAMP()) as day_of_week,
    jsonPayload.connection.src_ip,
    jsonPayload.connection.dest_ip,
    SUM(CAST(jsonPayload.bytes_sent AS INT64)) as total_bytes,
    COUNT(*) as connection_count
  FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
  WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
  GROUP BY src_ip, dest_ip
)
SELECT
  c.src_ip,
  c.dest_ip,
  c.total_bytes / 1024 / 1024 as current_mb,
  b.avg_bytes / 1024 / 1024 as baseline_avg_mb,
  c.connection_count as current_connections,
  b.connection_count as baseline_connections,
  (c.total_bytes - b.avg_bytes) / NULLIF(b.stddev_bytes, 0) as bytes_stddev_diff
FROM current_hour c
LEFT JOIN `project.vpc_flow_logs.traffic_baseline` b
  ON c.hour_of_day = b.hour_of_day
  AND c.day_of_week = b.day_of_week
  AND c.src_ip = b.src_ip
  AND c.dest_ip = b.dest_ip
WHERE ABS((c.total_bytes - b.avg_bytes) / NULLIF(b.stddev_bytes, 0)) > 3  -- 3 sigma
ORDER BY ABS(bytes_stddev_diff) DESC;
```

**Geographic Analysis**:
```sql
-- Traffic by geography
SELECT
  jsonPayload.src_location.country as source_country,
  jsonPayload.src_location.region as source_region,
  jsonPayload.dest_location.country as dest_country,
  jsonPayload.dest_location.region as dest_region,
  COUNT(*) as connection_count,
  SUM(CAST(jsonPayload.bytes_sent AS INT64)) / 1024 / 1024 / 1024 as gb_transferred,
  AVG(CAST(jsonPayload.rtt_msec AS INT64)) as avg_latency_ms
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND jsonPayload.src_location.country IS NOT NULL
  AND jsonPayload.dest_location.country IS NOT NULL
GROUP BY source_country, source_region, dest_country, dest_region
ORDER BY gb_transferred DESC;

-- Unexpected geographic sources
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.src_location.country,
  jsonPayload.src_location.city,
  jsonPayload.connection.dest_ip,
  jsonPayload.connection.dest_port,
  COUNT(*) as attempts
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
  AND jsonPayload.src_location.country NOT IN ('US', 'CA', 'GB', 'DE')  -- Expected countries
  AND NOT REGEXP_CONTAINS(jsonPayload.connection.src_ip, r'^10\.|^172\.(1[6-9]|2[0-9]|3[01])\.|^192\.168\.')
GROUP BY src_ip, country, city, dest_ip, dest_port
ORDER BY attempts DESC;
```

**Protocol Analysis**:
```sql
-- Protocol distribution
SELECT
  jsonPayload.connection.protocol,
  COUNT(*) as flow_count,
  SUM(CAST(jsonPayload.bytes_sent AS INT64)) / 1024 / 1024 / 1024 as gb_transferred,
  AVG(CAST(jsonPayload.rtt_msec AS INT64)) as avg_rtt_ms,
  COUNT(DISTINCT jsonPayload.connection.src_ip) as unique_sources,
  COUNT(DISTINCT jsonPayload.connection.dest_ip) as unique_destinations
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY protocol
ORDER BY gb_transferred DESC;

-- Unusual protocols
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  jsonPayload.connection.protocol,
  jsonPayload.connection.dest_port,
  COUNT(*) as connection_count
FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
  AND jsonPayload.connection.protocol NOT IN (6, 17, 1)  -- Not TCP, UDP, or ICMP
GROUP BY src_ip, dest_ip, protocol, dest_port
ORDER BY connection_count DESC;
```

### Automated Alerting

**Create Alert Policy for Anomalies**:
```python
from google.cloud import monitoring_v3
from google.cloud import bigquery

def create_flow_log_alert(project_id):
    """Create alert for flow log anomalies"""

    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    # Alert on high denied connection rate
    alert_policy = monitoring_v3.AlertPolicy(
        display_name="High Firewall Deny Rate",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Denied connections > 1000/min",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='resource.type="gce_subnetwork" AND metric.type="logging.googleapis.com/user/vpc_flow_denied"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=1000,
                    duration={"seconds": 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                        )
                    ],
                ),
            )
        ],
        documentation=monitoring_v3.AlertPolicy.Documentation(
            content="High rate of firewall-denied connections detected. Possible attack or misconfiguration."
        ),
    )

    policy = client.create_alert_policy(name=project_name, alert_policy=alert_policy)
    return policy

def scheduled_anomaly_detection():
    """Run scheduled anomaly detection on flow logs"""

    bq_client = bigquery.Client()

    query = """
    -- Detect port scanning
    SELECT
      jsonPayload.connection.src_ip as scanner_ip,
      COUNT(DISTINCT jsonPayload.connection.dest_port) as ports_scanned
    FROM `project.vpc_flow_logs.compute_googleapis_com_vpc_flows_*`
    WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 15 MINUTE)
    GROUP BY scanner_ip
    HAVING ports_scanned > 50
    """

    results = bq_client.query(query)

    for row in results:
        # Send alert or create firewall rule
        print(f"Port scan detected from {row.scanner_ip}: {row.ports_scanned} ports")
        # Add alerting logic here

if __name__ == '__main__':
    create_flow_log_alert('PROJECT_ID')
```

## Network Monitoring and Observability

### Cloud Monitoring for Networks

**Network Metrics**:
```python
from google.cloud import monitoring_v3
import time

def create_network_monitoring_dashboard(project_id):
    """Create comprehensive network monitoring dashboard"""

    client = monitoring_v3.DashboardsServiceClient()
    project_name = f"projects/{project_id}"

    dashboard = monitoring_v3.Dashboard(
        display_name="Network Security Dashboard",
        mosaic_layout=monitoring_v3.MosaicLayout(
            columns=12,
            tiles=[
                # Tile 1: Network egress traffic
                monitoring_v3.MosaicLayout.Tile(
                    width=6,
                    height=4,
                    widget=monitoring_v3.Widget(
                        title="Network Egress (GB/hour)",
                        xy_chart=monitoring_v3.XyChart(
                            data_sets=[
                                monitoring_v3.XyChart.DataSet(
                                    time_series_query=monitoring_v3.TimeSeriesQuery(
                                        time_series_filter=monitoring_v3.TimeSeriesFilter(
                                            filter='metric.type="compute.googleapis.com/instance/network/sent_bytes_count" resource.type="gce_instance"',
                                            aggregation=monitoring_v3.Aggregation(
                                                alignment_period={"seconds": 3600},
                                                per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                                                cross_series_reducer=monitoring_v3.Aggregation.Reducer.REDUCE_SUM,
                                            ),
                                        )
                                    )
                                )
                            ]
                        )
                    )
                ),

                # Tile 2: Firewall denied connections
                monitoring_v3.MosaicLayout.Tile(
                    width=6,
                    height=4,
                    widget=monitoring_v3.Widget(
                        title="Firewall Denied Connections",
                        xy_chart=monitoring_v3.XyChart(
                            data_sets=[
                                monitoring_v3.XyChart.DataSet(
                                    time_series_query=monitoring_v3.TimeSeriesQuery(
                                        time_series_filter=monitoring_v3.TimeSeriesFilter(
                                            filter='metric.type="logging.googleapis.com/user/firewall_denied" resource.type="gce_subnetwork"',
                                            aggregation=monitoring_v3.Aggregation(
                                                alignment_period={"seconds": 60},
                                                per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                                            ),
                                        )
                                    )
                                )
                            ]
                        )
                    )
                ),

                # Tile 3: Network latency
                monitoring_v3.MosaicLayout.Tile(
                    width=6,
                    height=4,
                    widget=monitoring_v3.Widget(
                        title="Network RTT (ms)",
                        xy_chart=monitoring_v3.XyChart(
                            data_sets=[
                                monitoring_v3.XyChart.DataSet(
                                    time_series_query=monitoring_v3.TimeSeriesQuery(
                                        time_series_filter=monitoring_v3.TimeSeriesFilter(
                                            filter='metric.type="compute.googleapis.com/instance/network/tcp_rtt_latencies" resource.type="gce_instance"',
                                            aggregation=monitoring_v3.Aggregation(
                                                alignment_period={"seconds": 60},
                                                per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_DELTA,
                                            ),
                                        )
                                    )
                                )
                            ]
                        )
                    )
                ),

                # Tile 4: Load balancer health
                monitoring_v3.MosaicLayout.Tile(
                    width=6,
                    height=4,
                    widget=monitoring_v3.Widget(
                        title="Load Balancer Backend Health",
                        xy_chart=monitoring_v3.XyChart(
                            data_sets=[
                                monitoring_v3.XyChart.DataSet(
                                    time_series_query=monitoring_v3.TimeSeriesQuery(
                                        time_series_filter=monitoring_v3.TimeSeriesFilter(
                                            filter='metric.type="loadbalancing.googleapis.com/https/backend_request_count" resource.type="https_lb_rule"',
                                            aggregation=monitoring_v3.Aggregation(
                                                alignment_period={"seconds": 60},
                                                per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                                            ),
                                        )
                                    )
                                )
                            ]
                        )
                    )
                ),
            ]
        )
    )

    response = client.create_dashboard(
        name=project_name,
        dashboard=dashboard
    )
    return response

def create_network_alert_policies(project_id):
    """Create comprehensive network alert policies"""

    client = monitoring_v3.AlertPolicyServiceClient()
    project_name = f"projects/{project_id}"

    alerts = []

    # Alert 1: High network egress
    alerts.append(monitoring_v3.AlertPolicy(
        display_name="High Network Egress (> 100 GB/hour)",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Egress threshold exceeded",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="compute.googleapis.com/instance/network/sent_bytes_count" resource.type="gce_instance"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=100 * 1024 * 1024 * 1024,  # 100 GB
                    duration={"seconds": 3600},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 3600},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                            cross_series_reducer=monitoring_v3.Aggregation.Reducer.REDUCE_SUM,
                        )
                    ],
                ),
            )
        ],
        documentation=monitoring_v3.AlertPolicy.Documentation(
            content="Network egress exceeds 100 GB/hour. Check for data exfiltration or misconfigured applications."
        ),
    ))

    # Alert 2: High firewall deny rate
    alerts.append(monitoring_v3.AlertPolicy(
        display_name="High Firewall Deny Rate",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Denied connections > 1000/min",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="logging.googleapis.com/user/firewall_denied"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=1000,
                    duration={"seconds": 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_RATE,
                        )
                    ],
                ),
            )
        ],
    ))

    # Alert 3: VPN tunnel down
    alerts.append(monitoring_v3.AlertPolicy(
        display_name="VPN Tunnel Down",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="VPN tunnel status not established",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter='metric.type="vpn.googleapis.com/gateway/tunnel_established" resource.type="vpn_gateway"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_LT,
                    threshold_value=1,
                    duration={"seconds": 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_MEAN,
                        )
                    ],
                ),
            )
        ],
    ))

    # Create all alerts
    created_alerts = []
    for alert in alerts:
        policy = client.create_alert_policy(name=project_name, alert_policy=alert)
        created_alerts.append(policy)

    return created_alerts
```

### Uptime Checks

**Configure Uptime Monitoring**:
```python
from google.cloud import monitoring_v3

def create_uptime_check(project_id, display_name, host, path='/'):
    """Create uptime check for network services"""

    client = monitoring_v3.UptimeCheckServiceClient()
    project_name = f"projects/{project_id}"

    # HTTPS uptime check
    config = monitoring_v3.UptimeCheckConfig(
        display_name=display_name,
        monitored_resource=monitoring_v3.MonitoredResource(
            type="uptime_url",
            labels={
                "project_id": project_id,
                "host": host,
            }
        ),
        http_check=monitoring_v3.UptimeCheckConfig.HttpCheck(
            request_method=monitoring_v3.UptimeCheckConfig.HttpCheck.RequestMethod.GET,
            path=path,
            port=443,
            use_ssl=True,
            validate_ssl=True,
        ),
        period={"seconds": 60},
        timeout={"seconds": 10},
        selected_regions=[
            monitoring_v3.UptimeCheckRegion.USA,
            monitoring_v3.UptimeCheckRegion.EUROPE,
            monitoring_v3.UptimeCheckRegion.ASIA_PACIFIC,
        ],
    )

    uptime_check = client.create_uptime_check_config(
        parent=project_name,
        uptime_check_config=config
    )

    # Create alert for uptime check failures
    alert_client = monitoring_v3.AlertPolicyServiceClient()

    alert_policy = monitoring_v3.AlertPolicy(
        display_name=f"Uptime Check Failed: {display_name}",
        conditions=[
            monitoring_v3.AlertPolicy.Condition(
                display_name="Uptime check failed",
                condition_threshold=monitoring_v3.AlertPolicy.Condition.MetricThreshold(
                    filter=f'metric.type="monitoring.googleapis.com/uptime_check/check_passed" AND resource.label.check_id="{uptime_check.uptime_check_id}"',
                    comparison=monitoring_v3.ComparisonType.COMPARISON_GT,
                    threshold_value=0.1,  # 10% failure rate
                    duration={"seconds": 300},
                    aggregations=[
                        monitoring_v3.Aggregation(
                            alignment_period={"seconds": 60},
                            per_series_aligner=monitoring_v3.Aggregation.Aligner.ALIGN_FRACTION_TRUE,
                        )
                    ],
                ),
            )
        ],
    )

    alert_client.create_alert_policy(name=project_name, alert_policy=alert_policy)

    return uptime_check

# Create uptime checks for critical services
create_uptime_check('PROJECT_ID', 'Web Service', 'example.com', '/')
create_uptime_check('PROJECT_ID', 'API Service', 'api.example.com', '/health')
create_uptime_check('PROJECT_ID', 'Admin Portal', 'admin.example.com', '/login')
```

### SLIs and SLOs

**Define Network SLIs/SLOs**:
```python
from google.cloud import monitoring_v3

def create_network_slo(project_id, service_name):
    """Create SLO for network availability"""

    client = monitoring_v3.ServiceMonitoringServiceClient()
    project_name = f"projects/{project_id}"

    # Create service
    service = monitoring_v3.Service(
        display_name=service_name,
        custom=monitoring_v3.Service.Custom(),
    )

    service = client.create_service(
        parent=project_name,
        service=service
    )

    # Create availability SLO (99.9% uptime)
    slo = monitoring_v3.ServiceLevelObjective(
        display_name=f"{service_name} Availability SLO",
        goal=0.999,  # 99.9%
        rolling_period={"seconds": 2592000},  # 30 days
        service_level_indicator=monitoring_v3.ServiceLevelIndicator(
            request_based=monitoring_v3.ServiceLevelIndicator.RequestBased(
                good_total_ratio=monitoring_v3.TimeSeriesRatio(
                    good_service_filter='metric.type="loadbalancing.googleapis.com/https/request_count" AND metric.label.response_code_class="200"',
                    total_service_filter='metric.type="loadbalancing.googleapis.com/https/request_count"',
                )
            )
        ),
    )

    slo = client.create_service_level_objective(
        parent=service.name,
        service_level_objective=slo
    )

    # Create latency SLO (95% of requests < 200ms)
    latency_slo = monitoring_v3.ServiceLevelObjective(
        display_name=f"{service_name} Latency SLO",
        goal=0.95,  # 95%
        rolling_period={"seconds": 2592000},  # 30 days
        service_level_indicator=monitoring_v3.ServiceLevelIndicator(
            request_based=monitoring_v3.ServiceLevelIndicator.RequestBased(
                distribution_cut=monitoring_v3.ServiceLevelIndicator.RequestBased.DistributionCut(
                    distribution_filter='metric.type="loadbalancing.googleapis.com/https/backend_latencies"',
                    range=monitoring_v3.Range(
                        min=0,
                        max=200,  # 200ms
                    ),
                )
            )
        ),
    )

    latency_slo = client.create_service_level_objective(
        parent=service.name,
        service_level_objective=latency_slo
    )

    return service, slo, latency_slo

# Create SLOs for services
create_network_slo('PROJECT_ID', 'Web Application')
create_network_slo('PROJECT_ID', 'API Gateway')
```

## Security Scenarios and Solutions

### Scenario 1: Multi-Tier Application with Defense in Depth

**Requirements**:
- Public web tier
- Private application tier
- Isolated database tier
- Bastion host for administration
- Comprehensive logging and monitoring

**Solution**:
```bash
# 1. Create VPC and subnets
gcloud compute networks create prod-vpc --subnet-mode=custom

# Web tier subnet (DMZ)
gcloud compute networks subnets create web-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all

# Application tier subnet
gcloud compute networks subnets create app-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.10.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all \
  --enable-private-ip-google-access

# Database tier subnet
gcloud compute networks subnets create db-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.20.0/24 \
  --enable-flow-logs \
  --logging-metadata=include-all \
  --enable-private-ip-google-access

# Management subnet
gcloud compute networks subnets create mgmt-subnet \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.0.100.0/24 \
  --enable-flow-logs

# 2. Create hierarchical firewall policy (organization level)
gcloud compute firewall-policies create prod-baseline-policy \
  --organization=ORG_ID

# Deny all egress to known malicious IPs
gcloud compute firewall-policies rules create 100 \
  --firewall-policy=prod-baseline-policy \
  --action=deny \
  --direction=egress \
  --dest-ranges=MALICIOUS_IP_LIST \
  --layer4-configs=all

# Deny egress to non-approved countries
gcloud compute firewall-policies rules create 200 \
  --firewall-policy=prod-baseline-policy \
  --action=deny \
  --direction=egress \
  --dest-threat-intelligence=iplist-known-malicious-ips \
  --layer4-configs=all

# Associate with folder
gcloud compute firewall-policies associations create \
  --firewall-policy=prod-baseline-policy \
  --folder=PROD_FOLDER_ID

# 3. VPC-level firewall rules
# Allow IAP for SSH
gcloud compute firewall-rules create allow-iap-ssh \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --priority=1000 \
  --enable-logging

# Web tier: Allow HTTPS from internet
gcloud compute firewall-rules create web-allow-https \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server \
  --priority=1000 \
  --enable-logging

# Web to App: Allow specific ports
gcloud compute firewall-rules create web-to-app \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:8080 \
  --source-tags=web-server \
  --target-tags=app-server \
  --priority=1000 \
  --enable-logging

# App to DB: Database connections only
gcloud compute firewall-rules create app-to-db \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:5432 \
  --source-tags=app-server \
  --target-tags=database \
  --priority=1000 \
  --enable-logging

# Management: Bastion to all tiers
gcloud compute firewall-rules create bastion-to-internal \
  --network=prod-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:22,tcp:3389 \
  --source-tags=bastion \
  --priority=1000 \
  --enable-logging

# Default deny all ingress
gcloud compute firewall-rules create deny-all-ingress \
  --network=prod-vpc \
  --action=deny \
  --direction=ingress \
  --rules=all \
  --source-ranges=0.0.0.0/0 \
  --priority=65534 \
  --enable-logging

# 4. Cloud Armor for web tier
gcloud compute security-policies create web-protection \
  --description="Web tier protection"

# Rate limiting
gcloud compute security-policies rules create 100 \
  --security-policy=web-protection \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429

# SQL injection protection
gcloud compute security-policies rules create 200 \
  --security-policy=web-protection \
  --expression="evaluatePreconfiguredExpr('sqli-v33-stable')" \
  --action=deny-403

# XSS protection
gcloud compute security-policies rules create 300 \
  --security-policy=web-protection \
  --expression="evaluatePreconfiguredExpr('xss-v33-stable')" \
  --action=deny-403

# Attach to backend service
gcloud compute backend-services update web-backend \
  --security-policy=web-protection \
  --global

# 5. Setup packet mirroring for IDS
# Create Cloud IDS endpoint
gcloud ids endpoints create prod-ids \
  --network=projects/PROJECT_ID/global/networks/prod-vpc \
  --zone=us-central1-a \
  --severity=INFORMATIONAL

# Mirror DMZ traffic to IDS
IDS_FORWARDING_RULE=$(gcloud ids endpoints describe prod-ids \
  --zone=us-central1-a \
  --format="value(endpointForwardingRule)")

gcloud compute packet-mirrorings create dmz-mirror \
  --region=us-central1 \
  --network=prod-vpc \
  --collector-ilb=${IDS_FORWARDING_RULE} \
  --mirrored-subnets=web-subnet

# 6. Create log sink for SIEM
gcloud logging sinks create security-logs-sink \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/security_logs \
  --log-filter='resource.type="gce_subnetwork" OR
    protoPayload.serviceName="ids.googleapis.com" OR
    jsonPayload.enforcedSecurityPolicy.name:*'
```

**Architecture Diagram**:
```
Internet
    |
    v
[Cloud Armor]
    |
    v
[External HTTPS LB]
    |
    v
+-------------------+
| Web Tier (DMZ)    |
| - Cloud Armor     |
| - Flow Logs ON    |
| - Public IPs      |
| - Tag: web-server |
+-------------------+
    | (fw: web-to-app tcp:8080)
    v
+-------------------+
| App Tier          |
| - Private IPs     |
| - Flow Logs ON    |
| - Tag: app-server |
+-------------------+
    | (fw: app-to-db tcp:5432)
    v
+-------------------+
| DB Tier           |
| - Private IPs     |
| - Flow Logs ON    |
| - No external     |
| - Tag: database   |
+-------------------+

+-------------------+
| Management        |
| - Bastion Host    |
| - IAP Access Only |
| - Audit Logging   |
| - Tag: bastion    |
+-------------------+
    |
    +--> (Can SSH to all tiers)

[Packet Mirroring] -> [Cloud IDS] -> [Security Command Center]
```

### Scenario 2: Secure API Gateway with Rate Limiting

**Requirements**:
- Public API endpoint
- Authentication via IAP
- Rate limiting per client
- DDoS protection
- API security (OWASP)

**Solution**:
```bash
# 1. Create backend service with IAP
gcloud compute backend-services create api-backend \
  --protocol=HTTPS \
  --health-checks=api-health-check \
  --global \
  --iap=enabled,oauth2-client-id=CLIENT_ID,oauth2-client-secret=CLIENT_SECRET

# 2. Create Cloud Armor policy for API protection
gcloud compute security-policies create api-protection \
  --description="API Gateway protection"

# Whitelist known API clients
gcloud compute security-policies rules create 50 \
  --security-policy=api-protection \
  --expression="inIpRange(origin.ip, '203.0.113.0/24')" \
  --action=allow \
  --description="Trusted API clients"

# Rate limit by IP
gcloud compute security-policies rules create 100 \
  --security-policy=api-protection \
  --expression="true" \
  --action=throttle \
  --rate-limit-threshold-count=1000 \
  --rate-limit-threshold-interval-sec=60 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP

# Rate limit by header (API key)
gcloud compute security-policies rules create 150 \
  --security-policy=api-protection \
  --expression="has(request.headers['x-api-key'])" \
  --action=throttle \
  --rate-limit-threshold-count=10000 \
  --rate-limit-threshold-interval-sec=60 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=HTTP_HEADER \
  --enforce-on-key-name=x-api-key

# Block common attacks
gcloud compute security-policies rules create 200 \
  --security-policy=api-protection \
  --expression="evaluatePreconfiguredExpr('sqli-v33-stable') || evaluatePreconfiguredExpr('xss-v33-stable')" \
  --action=deny-403

# Attach to backend
gcloud compute backend-services update api-backend \
  --security-policy=api-protection \
  --global

# 3. Setup logging and monitoring
# Create log-based metric for API errors
gcloud logging metrics create api_error_rate \
  --description="API 5xx error rate" \
  --log-filter='resource.type="http_load_balancer" AND
    httpRequest.status>=500'

# Create alert
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High API Error Rate" \
  --condition-display-name="5xx errors > 10/min" \
  --condition-threshold-value=10 \
  --condition-threshold-duration=60s \
  --condition-expression='resource.type="http_load_balancer" AND
    metric.type="logging.googleapis.com/user/api_error_rate"'
```

### Scenario 3: Zero Trust Network with VPC Service Controls

**Requirements**:
- Protect sensitive data in BigQuery and Cloud Storage
- Restrict access to corporate network
- Allow specific service accounts
- Dry-run mode before enforcement

**Solution**:
```bash
# 1. Create VPC-SC access policy
POLICY_ID=$(gcloud access-context-manager policies list \
  --organization=ORG_ID \
  --format="value(name)")

# 2. Create access levels
# Corporate network
cat > corp-network.yaml <<EOF
name: accessPolicies/${POLICY_ID}/accessLevels/corp_network
title: Corporate Network Access
basic:
  conditions:
    - ipSubnetworks:
        - "203.0.113.0/24"  # HQ
        - "198.51.100.0/24"  # Remote office
EOF

gcloud access-context-manager levels create corp_network \
  --policy=${POLICY_ID} \
  --basic-level-spec=corp-network.yaml

# Managed devices
cat > managed-devices.yaml <<EOF
name: accessPolicies/${POLICY_ID}/accessLevels/managed_devices
title: Managed Devices
basic:
  conditions:
    - devicePolicy:
        requireScreenlock: true
        requireCorpOwned: true
        osConstraints:
          - osType: DESKTOP_WINDOWS
            minimumVersion: "10.0.19041"
          - osType: DESKTOP_MAC
            minimumVersion: "11.0.0"
EOF

gcloud access-context-manager levels create managed_devices \
  --policy=${POLICY_ID} \
  --basic-level-spec=managed-devices.yaml

# 3. Create service perimeter with dry-run
cat > perimeter.yaml <<EOF
name: accessPolicies/${POLICY_ID}/servicePerimeters/data_perimeter
title: Sensitive Data Perimeter
perimeterType: PERIMETER_TYPE_REGULAR
status:
  resources:
    - projects/123456789  # Data project
  restrictedServices:
    - storage.googleapis.com
    - bigquery.googleapis.com
  accessLevels:
    - accessPolicies/${POLICY_ID}/accessLevels/corp_network
    - accessPolicies/${POLICY_ID}/accessLevels/managed_devices
spec:  # Dry-run configuration
  resources:
    - projects/123456789
    - projects/987654321  # Additional project to test
  restrictedServices:
    - storage.googleapis.com
    - bigquery.googleapis.com
    - pubsub.googleapis.com  # Additional service to test
  accessLevels:
    - accessPolicies/${POLICY_ID}/accessLevels/corp_network
    - accessPolicies/${POLICY_ID}/accessLevels/managed_devices
  ingressPolicies:
    - ingressFrom:
        sources:
          - accessLevel: accessPolicies/${POLICY_ID}/accessLevels/corp_network
        identities:
          - serviceAccount:data-pipeline@project.iam.gserviceaccount.com
      ingressTo:
        resources:
          - projects/123456789
        operations:
          - serviceName: bigquery.googleapis.com
            methodSelectors:
              - method: "*"
EOF

gcloud access-context-manager perimeters create data_perimeter \
  --policy=${POLICY_ID} \
  --title="Sensitive Data Perimeter" \
  --perimeter-type=regular \
  --config-file=perimeter.yaml

# 4. Monitor dry-run violations for 2 weeks
# Query violations
cat > check-violations.sql <<EOF
SELECT
  DATE(timestamp) as date,
  protoPayload.serviceName as service,
  protoPayload.authenticationInfo.principalEmail as user,
  COUNT(*) as violations
FROM \`project.dataset.cloudaudit_googleapis_com_data_access_*\`
WHERE protoPayload.metadata.dryRun = true
  AND protoPayload.metadata.securityPolicyInfo.servicePerimeterName LIKE '%data_perimeter%'
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 14 DAY)
GROUP BY date, service, user
ORDER BY date DESC, violations DESC
EOF

# 5. After validation, promote to enforcement
gcloud access-context-manager perimeters update data_perimeter \
  --policy=${POLICY_ID} \
  --promote-spec-to-status
```

### Scenario 4: Hybrid Connectivity with Secure Access

**Requirements**:
- Cloud VPN to on-premises
- HA VPN for redundancy
- Private Google Access
- Cloud NAT for outbound traffic
- Monitoring and alerting

**Solution**:
```bash
# 1. Create HA VPN gateway
gcloud compute vpn-gateways create ha-vpn-gw \
  --network=prod-vpc \
  --region=us-central1

# 2. Create Cloud Router
gcloud compute routers create cloud-router \
  --region=us-central1 \
  --network=prod-vpc \
  --asn=65001 \
  --advertisement-mode=CUSTOM \
  --set-advertisement-groups=ALL_SUBNETS \
  --set-advertisement-ranges=199.36.153.4/30

# 3. Create VPN tunnels
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-gcp-gateway=on-prem-vpn-gw \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SHARED_SECRET \
  --router=cloud-router \
  --vpn-gateway=ha-vpn-gw \
  --interface=0

gcloud compute vpn-tunnels create tunnel-2 \
  --peer-gcp-gateway=on-prem-vpn-gw \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SHARED_SECRET_2 \
  --router=cloud-router \
  --vpn-gateway=ha-vpn-gw \
  --interface=1

# 4. Create BGP sessions
gcloud compute routers add-bgp-peer cloud-router \
  --peer-name=on-prem-peer-1 \
  --interface=tunnel-1-interface \
  --peer-ip-address=169.254.0.1 \
  --peer-asn=65002 \
  --region=us-central1

gcloud compute routers add-bgp-peer cloud-router \
  --peer-name=on-prem-peer-2 \
  --interface=tunnel-2-interface \
  --peer-ip-address=169.254.1.1 \
  --peer-asn=65002 \
  --region=us-central1

# 5. Setup Cloud NAT
gcloud compute routers nats create cloud-nat \
  --router=cloud-router \
  --region=us-central1 \
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips \
  --enable-logging

# 6. Configure Private Google Access
gcloud compute networks subnets update app-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access

# 7. Create monitoring and alerts
# VPN tunnel status alert
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="VPN Tunnel Down" \
  --condition-threshold-value=1 \
  --condition-threshold-duration=300s \
  --condition-expression='metric.type="vpn.googleapis.com/gateway/tunnel_established"
    resource.type="vpn_gateway"' \
  --condition-threshold-comparison=COMPARISON_LT

# VPN bandwidth alert
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High VPN Bandwidth Usage" \
  --condition-threshold-value=900000000 \
  --condition-threshold-duration=300s \
  --condition-expression='metric.type="vpn.googleapis.com/network/sent_bytes_count"
    resource.type="vpn_gateway"' \
  --condition-threshold-comparison=COMPARISON_GT
```

## Troubleshooting Methodology

### Systematic Network Troubleshooting

**Troubleshooting Flowchart**:
```
Issue Reported
    |
    v
1. Define the Problem
   - What is failing?
   - When did it start?
   - What changed?
    |
    v
2. Check Connectivity
   - Can source reach destination?
   - Use Connectivity Tests
    |
    +---> [REACHABLE] ---> Check Application Layer
    |
    +---> [UNREACHABLE] ---> Continue below
    |
    v
3. Check Firewall Rules
   - Are rules allowing traffic?
   - Check priority and matching
   - Enable logging
    |
    +---> [RULES ALLOW] ---> Continue below
    |
    +---> [RULES DENY] ---> Fix firewall rules
    |
    v
4. Check Routes
   - Does route exist?
   - Check route priority
   - Check next hop
    |
    +---> [ROUTE EXISTS] ---> Continue below
    |
    +---> [NO ROUTE] ---> Add route
    |
    v
5. Check Instance Config
   - Instance running?
   - Network interface configured?
   - OS firewall rules?
    |
    +---> [CONFIG OK] ---> Continue below
    |
    +---> [CONFIG ISSUE] ---> Fix instance config
    |
    v
6. Check VPC-SC/IAP
   - Is VPC-SC blocking?
   - Is IAP required?
   - Check access levels
    |
    v
7. Analyze Logs
   - VPC Flow Logs
   - Firewall logs
   - Cloud Audit Logs
    |
    v
8. Performance Issues
   - Check latency (RTT)
   - Check packet loss
   - Check bandwidth
    |
    v
Problem Identified and Resolved
```

### Troubleshooting Commands

**Complete Troubleshooting Script**:
```bash
#!/bin/bash
# Network troubleshooting script for GCP

PROJECT_ID="your-project-id"
NETWORK="prod-vpc"
SOURCE_INSTANCE="web-server"
SOURCE_ZONE="us-central1-a"
DEST_IP="10.0.20.5"
DEST_PORT="5432"

echo "=== GCP Network Troubleshooting ==="
echo "Source: $SOURCE_INSTANCE ($SOURCE_ZONE)"
echo "Destination: $DEST_IP:$DEST_PORT"
echo ""

# 1. Check instance status
echo "1. Checking instance status..."
gcloud compute instances describe $SOURCE_INSTANCE \
  --zone=$SOURCE_ZONE \
  --format="table(name,status,networkInterfaces[0].networkIP)"

# 2. Check firewall rules
echo ""
echo "2. Checking firewall rules..."
gcloud compute firewall-rules list \
  --filter="network:$NETWORK AND (sourceRanges.list():* OR sourceTags.list():*)" \
  --format="table(name,direction,priority,sourceRanges.list():label=SRC_RANGES,allowed[].map().firewall_rule().list():label=ALLOW)"

# 3. Check routes
echo ""
echo "3. Checking routes..."
gcloud compute routes list \
  --filter="network:$NETWORK" \
  --format="table(name,destRange,nextHopGateway,nextHopIp,priority)"

# 4. Run connectivity test
echo ""
echo "4. Running connectivity test..."
gcloud network-management connectivity-tests create troubleshoot-test \
  --source-instance=projects/$PROJECT_ID/zones/$SOURCE_ZONE/instances/$SOURCE_INSTANCE \
  --destination-ip-address=$DEST_IP \
  --protocol=TCP \
  --destination-port=$DEST_PORT

# Wait for test to complete
sleep 10

# Get test results
gcloud network-management connectivity-tests describe troubleshoot-test \
  --format=json | jq '.reachabilityDetails'

# 5. Check VPC Flow Logs
echo ""
echo "5. Checking recent flow logs..."
gcloud logging read "resource.type=gce_subnetwork AND
  jsonPayload.connection.dest_ip=\"$DEST_IP\"" \
  --limit=10 \
  --format=json | jq '.[] | {
    timestamp: .timestamp,
    src: .jsonPayload.connection.src_ip,
    dest: .jsonPayload.connection.dest_ip,
    port: .jsonPayload.connection.dest_port,
    action: .jsonPayload.rule_details.action
  }'

# 6. Check firewall logs
echo ""
echo "6. Checking firewall deny logs..."
gcloud logging read "resource.type=gce_subnetwork AND
  jsonPayload.rule_details.action=\"DENY\" AND
  jsonPayload.connection.dest_ip=\"$DEST_IP\"" \
  --limit=5 \
  --format=json | jq '.[] | {
    timestamp: .timestamp,
    src: .jsonPayload.connection.src_ip,
    dest: .jsonPayload.connection.dest_ip,
    port: .jsonPayload.connection.dest_port,
    rule: .jsonPayload.rule_details.reference,
    direction: .jsonPayload.rule_details.direction
  }'

# 7. SSH to instance and test connectivity
echo ""
echo "7. Testing from source instance..."
gcloud compute ssh $SOURCE_INSTANCE --zone=$SOURCE_ZONE --command="
  echo 'Testing connectivity from instance...'
  echo ''
  echo 'Ping test:'
  ping -c 3 $DEST_IP
  echo ''
  echo 'TCP port test:'
  nc -zv $DEST_IP $DEST_PORT
  echo ''
  echo 'Route to destination:'
  ip route get $DEST_IP
  echo ''
  echo 'DNS resolution:'
  nslookup google.com
"

# 8. Check Cloud NAT (if used)
echo ""
echo "8. Checking Cloud NAT status..."
gcloud compute routers get-status cloud-router \
  --region=us-central1 \
  --format=json | jq '.result.natStatus'

# Cleanup
echo ""
echo "Cleaning up test resources..."
gcloud network-management connectivity-tests delete troubleshoot-test --quiet

echo ""
echo "=== Troubleshooting Complete ==="
```

### Common Issues and Solutions

**Issue 1: Cannot connect to instance**
```bash
# Check 1: Instance status
gcloud compute instances describe INSTANCE --zone=ZONE

# Check 2: Firewall rules allow traffic
gcloud compute firewall-rules list --filter="network:VPC_NAME"

# Check 3: Run connectivity test
gcloud network-management connectivity-tests create test \
  --source-instance=SOURCE \
  --destination-instance=DEST

# Solution: Add missing firewall rule
gcloud compute firewall-rules create allow-required-traffic \
  --network=VPC_NAME \
  --allow=tcp:PORT \
  --source-ranges=SOURCE_IP
```

**Issue 2: High latency**
```sql
-- Query Flow Logs for RTT
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  AVG(CAST(jsonPayload.rtt_msec AS INT64)) as avg_rtt,
  APPROX_QUANTILES(CAST(jsonPayload.rtt_msec AS INT64), 100)[OFFSET(95)] as p95_rtt
FROM `project.dataset.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY src_ip, dest_ip
HAVING avg_rtt > 100
ORDER BY avg_rtt DESC;
```

**Issue 3: VPC-SC blocking access**
```bash
# Check perimeter configuration
gcloud access-context-manager perimeters describe PERIMETER \
  --policy=POLICY_ID

# Check dry-run violations
gcloud logging read \
  "protoPayload.metadata.dryRun=true" \
  --limit=50 \
  --format=json

# Solution: Add access level or ingress policy
gcloud access-context-manager perimeters update PERIMETER \
  --add-access-levels=NEW_ACCESS_LEVEL \
  --policy=POLICY_ID
```

## Exam Tips for Professional Cloud Network Engineer

### Network Security Questions

**Common Exam Patterns**:

1. **VPC Service Controls**:
   - Understand perimeter types (regular vs bridge)
   - Know when to use dry-run mode
   - Ingress/egress policy configuration
   - Access level combinations (AND vs OR)

2. **Cloud Armor**:
   - Layer 7 DDoS protection
   - Preconfigured WAF rules (SQL injection, XSS)
   - Rate limiting strategies (per IP, per header, per cookie)
   - Custom CEL expressions

3. **Cloud IDS**:
   - Integration with packet mirroring
   - Threat severity levels
   - Security Command Center integration
   - Cost optimization (sampling, severity filtering)

4. **IAP**:
   - TCP forwarding use cases
   - IAP IP range (35.235.240.0/20)
   - Integration with load balancers
   - OAuth configuration

### Monitoring and Troubleshooting Tips

**High-Probability Exam Topics**:

1. **VPC Flow Logs**:
   - Know sampling rates and aggregation intervals
   - BigQuery analysis patterns
   - Cost optimization strategies
   - Anomaly detection queries

2. **Network Intelligence Center**:
   - Connectivity Tests for troubleshooting
   - Firewall Insights types
   - Performance Dashboard metrics
   - Network Topology visualization

3. **Cloud Monitoring**:
   - Uptime checks configuration
   - Alert policies for network metrics
   - SLI/SLO definitions
   - Log-based metrics

### Decision-Making Framework

**When to use what**:

| Requirement | Solution |
|-------------|----------|
| Layer 3/4 firewall | VPC Firewall Rules |
| Layer 7 protection | Cloud Armor |
| DDoS protection | Cloud Armor + Autoscaling |
| Data perimeter | VPC Service Controls |
| IDS/IPS | Cloud IDS + Packet Mirroring |
| Network monitoring | VPC Flow Logs + Network Intelligence Center |
| Secure admin access | IAP for TCP forwarding |
| API rate limiting | Cloud Armor (throttle action) |
| Zero-trust access | VPC-SC + IAP + BeyondCorp |
| Hybrid connectivity | Cloud VPN or Cloud Interconnect |

### Key Exam Strategies

1. **Read Carefully**: Distinguish between Layer 3/4 (firewall rules) and Layer 7 (Cloud Armor)

2. **Cost Optimization**: Always consider sampling rates, aggregation intervals, and log retention

3. **Security Layers**: Questions often require multiple security controls (defense in depth)

4. **VPC-SC Scenarios**: Understand dry-run mode for testing before enforcement

5. **Monitoring**: Know when to use Flow Logs vs Packet Mirroring vs Network Intelligence Center

6. **Troubleshooting**: Follow systematic approach (Connectivity Tests -> Firewall -> Routes -> Instance)

### Important Metrics and Limits

```
VPC Firewall Rules:
- Maximum rules per VPC: 15,000 (including hierarchical)
- Rule evaluation: Lowest priority number wins
- Logging impact: ~10% performance overhead

Cloud Armor:
- Rate limiting: Based on IP, cookie, header, region
- CEL expression limit: 1024 characters
- Preconfigured rules: SQL injection, XSS, LFI, RCE, etc.

VPC Service Controls:
- Projects per perimeter: 1,000
- Access levels per perimeter: 200
- Dry-run period: Minimum 2 weeks recommended

VPC Flow Logs:
- Sampling rates: 0.01 to 1.0 (1% to 100%)
- Aggregation: 5-sec, 10-sec (default), 15-sec
- Retention: 30 days default (configurable)

Packet Mirroring:
- Supported sources: Subnets, instances, network tags
- Protocols: TCP, UDP, ICMP, or all
- Cost: ~$0.10 per GB mirrored
```

### Sample Exam Questions

**Question 1**: Your company requires that all access to BigQuery datasets in a production project must originate from corporate IP addresses or managed devices. The solution should be tested before enforcement. What should you do?

**Answer**: Create a VPC Service Controls perimeter with access levels for corporate IPs and device policies. Configure the perimeter with both status (current) and spec (dry-run) configurations. Monitor dry-run violations for 2-3 weeks, then promote spec to status.

**Question 2**: You need to protect your web application from SQL injection and DDoS attacks while allowing legitimate traffic. What is the most appropriate solution?

**Answer**: Implement Cloud Armor security policy with preconfigured WAF rules for SQL injection (sqli-v33-stable) and rate limiting (throttle or rate-based-ban action) attached to the load balancer backend service.

**Question 3**: Your security team needs to analyze network traffic for potential threats without impacting application performance. What should you implement?

**Answer**: Configure packet mirroring to send traffic to a Cloud IDS endpoint. Set appropriate filters (subnets, protocols) and adjust IDS severity level to balance detection and cost.

---

**Document Version**: 2.0 (Enhanced for Professional Cloud Network Engineer Exam)
**Last Updated**: 2025-10
**Lines**: Comprehensive coverage with 3100+ lines
**Coverage**: Network Security Architecture, VPC-SC, Cloud IDS, Packet Mirroring, Network Intelligence Center, VPC Flow Logs, Monitoring, Troubleshooting, Security Scenarios, Exam Tips

## Additional Resources

- [VPC Firewall Rules](https://cloud.google.com/vpc/docs/firewalls)
- [Cloud Armor Documentation](https://cloud.google.com/armor/docs)
- [VPC Flow Logs](https://cloud.google.com/vpc/docs/flow-logs)
- [Network Intelligence Center](https://cloud.google.com/network-intelligence-center/docs)
- [Cloud IDS Documentation](https://cloud.google.com/intrusion-detection-system/docs)
- [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
- [Identity-Aware Proxy](https://cloud.google.com/iap/docs)
- [Packet Mirroring](https://cloud.google.com/vpc/docs/packet-mirroring)
- [Network Troubleshooting Guide](https://cloud.google.com/architecture/best-practices-for-troubleshooting-networks)
- [Security Best Practices](https://cloud.google.com/security/best-practices)
- [Cloud Monitoring for Networks](https://cloud.google.com/monitoring/docs)
