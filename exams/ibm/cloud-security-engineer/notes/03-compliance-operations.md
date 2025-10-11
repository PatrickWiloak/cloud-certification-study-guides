# IBM Cloud Security Engineer (C1000-178) - Compliance & Security Operations

## Table of Contents
- [Compliance Frameworks](#compliance-frameworks)
- [Security and Compliance Center](#security-and-compliance-center)
- [Audit Logging](#audit-logging)
- [Incident Response](#incident-response)
- [Security Scanning](#security-scanning)
- [Backup and Recovery](#backup-and-recovery)
- [Business Continuity](#business-continuity)

---

## Compliance Frameworks

### Supported Standards
- **ISO 27001/27017/27018**: Information security
- **SOC 1/2/3**: Service organization controls
- **PCI DSS**: Payment card data security
- **HIPAA**: Healthcare data protection
- **GDPR**: EU data protection
- **FedRAMP**: US federal government

### Compliance Features
```bash
# View compliance status
ibmcloud security-advisor findings list --provider-id compliance

# Generate compliance report
ibmcloud compliance report-generate \
  --profile-id ibm-cloud-framework \
  --attachment-id $ATTACHMENT_ID
```

---

## Security and Compliance Center

### Creating Profiles
```bash
# Create custom profile
ibmcloud scc profile-create \
  --name "Production Security Profile" \
  --description "Security controls for production" \
  --profile-type custom

# Attach profile to scope
ibmcloud scc attachment-create \
  --profile-id $PROFILE_ID \
  --name "Production Scope" \
  --scope-type account \
  --scope-id $ACCOUNT_ID

# Run scan
ibmcloud scc scan-create \
  --attachment-id $ATTACHMENT_ID

# View results
ibmcloud scc results-list \
  --attachment-id $ATTACHMENT_ID \
  --report-id $REPORT_ID
```

### Terraform for SCC
```hcl
# scc-configuration.tf
resource "ibm_scc_profile" "custom_profile" {
  name         = "Production Security Profile"
  description  = "Custom security controls"
  profile_type = "custom"
  
  controls {
    control_library_id = "ibm-cloud-framework"
    control_id         = "SC-7"
  }
}

resource "ibm_scc_profile_attachment" "prod_attachment" {
  profile_id  = ibm_scc_profile.custom_profile.id
  name        = "Production Environment"
  description = "Production scope attachment"
  
  scope {
    environment = "production"
    properties {
      name  = "resource_group_id"
      value = data.ibm_resource_group.prod.id
    }
  }

  schedule      = "daily"
  notifications = {
    enabled = true
    controls {
      threshold_limit = 15
      failed_control_ids = []
    }
  }
}
```

---

## Audit Logging

### Activity Tracker
```bash
# Create Activity Tracker instance
ibmcloud resource service-instance-create \
  production-at \
  logdnaat \
  7-day \
  us-south

# Configure event routing
ibmcloud atracker route create \
  --name production-route \
  --target-id $TARGET_ID \
  --locations us-south,us-east

# Query events
ibmcloud atracker event list \
  --from "2024-01-01T00:00:00Z" \
  --to "2024-01-31T23:59:59Z" \
  --action iam.policy.create
```

### Terraform for Activity Tracker
```hcl
# activity-tracker.tf
resource "ibm_resource_instance" "at_instance" {
  name     = "production-activity-tracker"
  service  = "logdnaat"
  plan     = "7-day"
  location = "us-south"
}

resource "ibm_atracker_target" "cos_target" {
  name        = "cos-target"
  target_type = "cloud_object_storage"
  cos_endpoint {
    endpoint           = "s3.us-south.cloud-object-storage.appdomain.cloud"
    target_crn         = ibm_resource_instance.cos.crn
    bucket             = ibm_cos_bucket.audit_logs.bucket_name
    api_key            = var.cos_api_key
  }
}

resource "ibm_atracker_route" "production_route" {
  name = "production-audit-route"
  rules {
    target_ids = [ibm_atracker_target.cos_target.id]
    locations  = ["us-south", "us-east"]
  }
}
```

---

## Incident Response

### Security Advisor
```bash
# View security findings
ibmcloud security-advisor findings list

# Create custom finding
ibmcloud security-advisor finding create \
  --provider-id security-ops \
  --finding-id incident-001 \
  --severity CRITICAL \
  --certainty HIGH \
  --context '{
    "region": "us-south",
    "resource_type": "compute",
    "resource_id": "vsi-123"
  }'

# Create occurrence
ibmcloud security-advisor occurrence-create \
  --provider-id security-ops \
  --note-name incident-001 \
  --occurrence-time "2024-01-15T10:30:00Z"
```

### Incident Response Playbook
```yaml
# incident-response.yaml
incident_types:
  - name: unauthorized_access
    severity: high
    steps:
      - action: isolate_resource
        commands:
          - "ibmcloud is instance-stop {instance_id}"
          - "ibmcloud is security-group-rule-delete-all {sg_id}"
      - action: collect_evidence
        commands:
          - "ibmcloud is instance-console-get {instance_id}"
          - "ibmcloud at event-list --from {incident_time}"
      - action: notify_team
        channels:
          - pagerduty
          - slack
      - action: initiate_forensics
        
  - name: data_breach
    severity: critical
    steps:
      - action: enable_lockdown
      - action: notify_legal
      - action: compliance_report
      - action: customer_notification
```

---

## Security Scanning

### Vulnerability Assessment
```bash
# Container vulnerability scan
ibmcloud cr vulnerability-assessment us.icr.io/namespace/app:latest

# View detailed vulnerabilities
ibmcloud cr vulnerability-assessment us.icr.io/namespace/app:latest \
  --output json | jq '.vulnerabilities'

# Set security policy
ibmcloud cr quota-set --vulnerability-check enabled

# Exemption for false positive
ibmcloud cr exemption-add \
  --scope us.icr.io/namespace/app \
  --issue-id CVE-2024-12345 \
  --reason "False positive - patched in base image"
```

### Code Scanning Integration
```yaml
# .gitlab-ci.yml
security_scan:
  stage: test
  script:
    - npm audit
    - npm audit --production
    - snyk test --severity-threshold=high
  allow_failure: false
  
container_scan:
  stage: build
  script:
    - docker build -t $IMAGE_NAME .
    - docker push $IMAGE_NAME
    - ibmcloud cr vulnerability-assessment $IMAGE_NAME
  only:
    - main
```

---

## Backup and Recovery

### Data Backup Strategy
```bash
# Create backup for Cloud Object Storage
ibmcloud cos bucket-versioning-put \
  --bucket audit-logs \
  --versioning-configuration Status=Enabled

# Enable replication
ibmcloud cos bucket-replication-put \
  --bucket audit-logs \
  --replication-configuration file://replication.json

# Database backup
ibmcloud cdb deployment-backup-create $DEPLOYMENT_ID

# List backups
ibmcloud cdb backups $DEPLOYMENT_ID

# Restore from backup
ibmcloud cdb deployment-pitr-create \
  --deployment-id $DEPLOYMENT_ID \
  --backup-id $BACKUP_ID \
  --target-deployment new-instance
```

### Terraform Backup Configuration
```hcl
# backup-configuration.tf
resource "ibm_cos_bucket" "backup_bucket" {
  bucket_name          = "production-backups"
  resource_instance_id = ibm_resource_instance.cos.id
  region_location      = "us-south"
  storage_class        = "standard"

  object_versioning {
    enable = true
  }

  retention_rule {
    default   = 30
    maximum   = 365
    minimum   = 1
    permanent = false
  }

  lifecycle_rule {
    id      = "archive-old-backups"
    enabled = true
    
    transition {
      days          = 90
      storage_class = "glacier"
    }
    
    expiration {
      days = 365
    }
  }
}

resource "ibm_database" "postgresql" {
  name              = "production-db"
  plan              = "standard"
  location          = "us-south"
  service           = "databases-for-postgresql"
  
  backup_schedule {
    start_time = "02:00"
    frequency  = "daily"
  }

  point_in_time_recovery {
    enabled = true
  }
}
```

---

## Business Continuity

### Disaster Recovery Plan
```bash
# Multi-zone deployment
for zone in us-south-1 us-south-2 us-south-3; do
  ibmcloud is instance-create \
    app-server-$zone \
    $VPC_ID \
    $zone \
    bx2-2x8 \
    $SUBNET_ID
done

# Cross-region replication
ibmcloud cos bucket-replication-put \
  --bucket production-data \
  --replication-configuration '{
    "Role": "arn:aws:iam:::role/s3crr_role",
    "Rules": [{
      "Status": "Enabled",
      "Priority": 1,
      "Destination": {
        "Bucket": "arn:aws:s3:::backup-eu-de"
      }
    }]
  }'

# Transit Gateway for DR
ibmcloud tg gateway-create dr-gateway \
  --location us-south \
  --routing global

ibmcloud tg connection-create $TG_ID \
  --name primary-vpc \
  --network-type vpc \
  --network-id $PRIMARY_VPC_CRN

ibmcloud tg connection-create $TG_ID \
  --name dr-vpc \
  --network-type vpc \
  --network-id $DR_VPC_CRN
```

### Terraform DR Configuration
```hcl
# disaster-recovery.tf
module "primary_region" {
  source = "./modules/infrastructure"
  
  region          = "us-south"
  environment     = "production"
  is_primary      = true
  dr_region_cidr  = "10.241.0.0/16"
}

module "dr_region" {
  source = "./modules/infrastructure"
  
  region          = "eu-de"
  environment     = "production-dr"
  is_primary      = false
  primary_region_cidr = "10.240.0.0/16"
}

resource "ibm_tg_gateway" "dr_transit" {
  name     = "disaster-recovery-transit"
  location = "us-south"
  global   = true
}

resource "ibm_tg_connection" "primary_vpc" {
  gateway      = ibm_tg_gateway.dr_transit.id
  network_type = "vpc"
  name         = "primary-vpc-connection"
  network_id   = module.primary_region.vpc_crn
}

resource "ibm_tg_connection" "dr_vpc" {
  gateway      = ibm_tg_gateway.dr_transit.id
  network_type = "vpc"
  name         = "dr-vpc-connection"
  network_id   = module.dr_region.vpc_crn
}

# Global Load Balancer for failover
resource "ibm_cis_global_load_balancer" "app" {
  cis_id           = ibm_cis.instance.id
  domain_id        = ibm_cis_domain.domain.id
  name             = "app.example.com"
  fallback_pool_id = ibm_cis_origin_pool.dr_pool.id
  default_pool_ids = [
    ibm_cis_origin_pool.primary_pool.id,
    ibm_cis_origin_pool.dr_pool.id
  ]
  description      = "Global load balancer with DR failover"
  proxied          = true
  session_affinity = "cookie"
}

resource "ibm_cis_origin_pool" "primary_pool" {
  cis_id = ibm_cis.instance.id
  name   = "primary-pool"

  origins {
    name    = "primary-origin"
    address = module.primary_region.load_balancer_ip
    enabled = true
  }

  monitor              = ibm_cis_healthcheck.app_health.id
  notification_email   = "ops@example.com"
  enabled              = true
  minimum_origins      = 1
}

resource "ibm_cis_origin_pool" "dr_pool" {
  cis_id = ibm_cis.instance.id
  name   = "dr-pool"

  origins {
    name    = "dr-origin"
    address = module.dr_region.load_balancer_ip
    enabled = true
  }

  monitor              = ibm_cis_healthcheck.app_health.id
  notification_email   = "ops@example.com"
  enabled              = true
  minimum_origins      = 1
}
```

### RTO/RPO Requirements
```yaml
# sla-requirements.yaml
disaster_recovery:
  tier_1_critical:
    rto: 1h          # Recovery Time Objective
    rpo: 15m         # Recovery Point Objective
    backup_frequency: continuous
    replication: synchronous
    
  tier_2_important:
    rto: 4h
    rpo: 1h
    backup_frequency: hourly
    replication: asynchronous
    
  tier_3_standard:
    rto: 24h
    rpo: 24h
    backup_frequency: daily
    replication: scheduled
```

---

## Security Operations Automation

### Automated Compliance Checks
```python
# automated-compliance.py
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_scc import SecurityAndComplianceCenterApiV3
import json
from datetime import datetime

class ComplianceAutomation:
    def __init__(self, api_key, instance_id):
        authenticator = IAMAuthenticator(api_key)
        self.scc_client = SecurityAndComplianceCenterApiV3(
            authenticator=authenticator
        )
        self.scc_client.set_service_url(
            'https://us-south.compliance.cloud.ibm.com/instances/{}'.format(instance_id)
        )

    def run_compliance_scan(self, profile_id, attachment_id):
        """Trigger compliance scan"""
        try:
            response = self.scc_client.create_scan(
                attachment_id=attachment_id
            )
            scan_id = response.get_result()['id']
            print(f"Scan initiated: {scan_id}")
            return scan_id
        except Exception as e:
            print(f"Scan failed: {str(e)}")
            return None

    def get_compliance_score(self, attachment_id, report_id):
        """Calculate compliance score"""
        try:
            results = self.scc_client.list_report_evaluations(
                report_id=report_id
            ).get_result()

            total_controls = len(results['evaluations'])
            passed_controls = sum(1 for e in results['evaluations']
                                 if e['status'] == 'pass')

            score = (passed_controls / total_controls) * 100

            return {
                'score': round(score, 2),
                'total_controls': total_controls,
                'passed': passed_controls,
                'failed': total_controls - passed_controls,
                'timestamp': datetime.now().isoformat()
            }
        except Exception as e:
            print(f"Error calculating score: {str(e)}")
            return None

    def export_compliance_report(self, report_id, output_file):
        """Export compliance report to JSON"""
        try:
            report = self.scc_client.get_report(
                report_id=report_id
            ).get_result()

            with open(output_file, 'w') as f:
                json.dump(report, f, indent=2)

            print(f"Report exported to {output_file}")
            return True
        except Exception as e:
            print(f"Export failed: {str(e)}")
            return False

    def check_critical_failures(self, report_id):
        """Identify critical control failures"""
        try:
            results = self.scc_client.list_report_evaluations(
                report_id=report_id
            ).get_result()

            critical_failures = []
            for evaluation in results['evaluations']:
                if (evaluation['status'] == 'fail' and
                    evaluation.get('severity') == 'critical'):
                    critical_failures.append({
                        'control_id': evaluation['control_id'],
                        'control_name': evaluation['control_name'],
                        'reason': evaluation.get('reason', 'N/A'),
                        'resource': evaluation.get('target', {})
                    })

            return critical_failures
        except Exception as e:
            print(f"Error checking failures: {str(e)}")
            return []

# Usage
if __name__ == "__main__":
    automation = ComplianceAutomation(
        api_key='your-api-key',
        instance_id='your-instance-id'
    )

    # Run scan
    scan_id = automation.run_compliance_scan(
        profile_id='profile-123',
        attachment_id='attachment-456'
    )

    # Get compliance score
    score = automation.get_compliance_score(
        attachment_id='attachment-456',
        report_id='report-789'
    )
    print(f"Compliance Score: {score['score']}%")

    # Check critical failures
    failures = automation.check_critical_failures('report-789')
    if failures:
        print(f"Critical Failures: {len(failures)}")
        for failure in failures:
            print(f"  - {failure['control_name']}: {failure['reason']}")
```

### Event-Driven Security Automation
```python
# security-event-handler.py
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator
from ibm_platform_services import IamAccessGroupsV2
import requests
import json

class SecurityEventHandler:
    def __init__(self, api_key):
        self.api_key = api_key
        authenticator = IAMAuthenticator(api_key)
        self.iam_client = IamAccessGroupsV2(authenticator=authenticator)

    def handle_unauthorized_access(self, event):
        """Respond to unauthorized access attempts"""
        user_id = event.get('initiator', {}).get('id')
        resource = event.get('target', {}).get('name')

        print(f"Unauthorized access detected: {user_id} -> {resource}")

        # Revoke user's API keys
        self.revoke_user_api_keys(user_id)

        # Send alert
        self.send_security_alert({
            'severity': 'HIGH',
            'event_type': 'unauthorized_access',
            'user': user_id,
            'resource': resource,
            'action': 'API keys revoked'
        })

        # Create incident ticket
        self.create_incident_ticket(event)

    def handle_policy_violation(self, event):
        """Respond to policy violations"""
        violation_type = event.get('outcome', {}).get('reason')
        resource_crn = event.get('target', {}).get('id')

        print(f"Policy violation: {violation_type}")

        # Apply remediation
        if violation_type == 'encryption_disabled':
            self.enable_encryption(resource_crn)
        elif violation_type == 'public_access_enabled':
            self.disable_public_access(resource_crn)
        elif violation_type == 'weak_password':
            self.enforce_password_policy()

    def handle_compliance_drift(self, event):
        """Respond to compliance drift"""
        control_id = event.get('control_id')
        resource_id = event.get('resource_id')

        print(f"Compliance drift detected: {control_id}")

        # Run targeted scan
        self.run_compliance_check(control_id, resource_id)

        # Notify compliance team
        self.send_compliance_notification({
            'control': control_id,
            'resource': resource_id,
            'status': 'drift_detected'
        })

    def revoke_user_api_keys(self, user_id):
        """Revoke all API keys for a user"""
        # Implementation would use IBM Cloud IAM API
        print(f"Revoking API keys for user: {user_id}")

    def send_security_alert(self, alert_data):
        """Send security alert to monitoring system"""
        # Integration with PagerDuty, Slack, etc.
        print(f"Security alert sent: {alert_data}")

    def create_incident_ticket(self, event):
        """Create incident management ticket"""
        # Integration with ServiceNow, Jira, etc.
        print(f"Incident ticket created for event: {event.get('id')}")

# Event processing function for IBM Cloud Functions
def main(args):
    """Process Activity Tracker events"""
    api_key = args.get('api_key')
    event = args.get('event')

    handler = SecurityEventHandler(api_key)

    action = event.get('action')
    outcome = event.get('outcome')

    # Route to appropriate handler
    if outcome == 'failure' and 'unauthorized' in action.lower():
        handler.handle_unauthorized_access(event)
    elif 'policy' in action.lower():
        handler.handle_policy_violation(event)
    elif 'compliance' in action.lower():
        handler.handle_compliance_drift(event)

    return {'status': 'processed', 'event_id': event.get('id')}
```

---

## Advanced Audit and Forensics

### Log Analysis and Correlation
```python
# log-analysis.py
from datetime import datetime, timedelta
import json
import re

class AuditLogAnalyzer:
    def __init__(self, activity_tracker_logs):
        self.logs = activity_tracker_logs

    def detect_suspicious_patterns(self):
        """Detect suspicious activity patterns"""
        patterns = {
            'brute_force': self.detect_brute_force(),
            'privilege_escalation': self.detect_privilege_escalation(),
            'data_exfiltration': self.detect_data_exfiltration(),
            'unusual_access': self.detect_unusual_access_patterns()
        }
        return patterns

    def detect_brute_force(self):
        """Detect brute force login attempts"""
        failed_logins = {}
        threshold = 5
        time_window = timedelta(minutes=10)

        for log in self.logs:
            if log.get('action') == 'iam.policy.authenticate':
                if log.get('outcome') == 'failure':
                    user = log.get('initiator', {}).get('id')
                    timestamp = datetime.fromisoformat(log.get('eventTime'))

                    if user not in failed_logins:
                        failed_logins[user] = []
                    failed_logins[user].append(timestamp)

        # Check for threshold breaches
        alerts = []
        for user, timestamps in failed_logins.items():
            timestamps.sort()
            for i in range(len(timestamps) - threshold + 1):
                window = timestamps[i:i+threshold]
                if window[-1] - window[0] <= time_window:
                    alerts.append({
                        'user': user,
                        'failed_attempts': threshold,
                        'time_range': f"{window[0]} to {window[-1]}",
                        'severity': 'HIGH'
                    })
                    break

        return alerts

    def detect_privilege_escalation(self):
        """Detect privilege escalation attempts"""
        escalation_events = []

        for log in self.logs:
            action = log.get('action', '')
            if any(keyword in action for keyword in [
                'iam.policy.create',
                'iam.role.assign',
                'iam.serviceId.create'
            ]):
                initiator = log.get('initiator', {}).get('id')
                target = log.get('target', {}).get('id')

                if initiator != target:  # Someone modifying another user
                    escalation_events.append({
                        'initiator': initiator,
                        'target': target,
                        'action': action,
                        'timestamp': log.get('eventTime'),
                        'outcome': log.get('outcome')
                    })

        return escalation_events

    def detect_data_exfiltration(self):
        """Detect potential data exfiltration"""
        download_events = []
        threshold_size = 1024 * 1024 * 100  # 100 MB

        for log in self.logs:
            if 'download' in log.get('action', '').lower():
                size = log.get('requestData', {}).get('size', 0)
                if size > threshold_size:
                    download_events.append({
                        'user': log.get('initiator', {}).get('id'),
                        'resource': log.get('target', {}).get('name'),
                        'size_mb': size / (1024 * 1024),
                        'timestamp': log.get('eventTime'),
                        'ip_address': log.get('initiator', {}).get('host', {}).get('address')
                    })

        return download_events

    def detect_unusual_access_patterns(self):
        """Detect unusual access patterns"""
        user_access = {}

        for log in self.logs:
            user = log.get('initiator', {}).get('id')
            timestamp = datetime.fromisoformat(log.get('eventTime'))
            hour = timestamp.hour

            if user not in user_access:
                user_access[user] = {'hours': [], 'locations': []}

            user_access[user]['hours'].append(hour)
            user_access[user]['locations'].append(
                log.get('initiator', {}).get('host', {}).get('address')
            )

        # Detect unusual access times (outside business hours)
        alerts = []
        for user, data in user_access.items():
            unusual_hours = [h for h in data['hours'] if h < 6 or h > 22]
            if len(unusual_hours) > 5:
                alerts.append({
                    'user': user,
                    'unusual_access_count': len(unusual_hours),
                    'pattern': 'after_hours_access',
                    'severity': 'MEDIUM'
                })

            # Detect access from multiple locations
            unique_locations = set(data['locations'])
            if len(unique_locations) > 3:
                alerts.append({
                    'user': user,
                    'location_count': len(unique_locations),
                    'pattern': 'multi_location_access',
                    'severity': 'MEDIUM'
                })

        return alerts

    def generate_forensic_report(self, incident_time, time_range_hours=24):
        """Generate forensic report for incident investigation"""
        incident_dt = datetime.fromisoformat(incident_time)
        start_time = incident_dt - timedelta(hours=time_range_hours)
        end_time = incident_dt + timedelta(hours=time_range_hours)

        relevant_logs = [
            log for log in self.logs
            if start_time <= datetime.fromisoformat(log.get('eventTime')) <= end_time
        ]

        report = {
            'incident_time': incident_time,
            'analysis_window': f"{start_time} to {end_time}",
            'total_events': len(relevant_logs),
            'event_breakdown': self._analyze_events(relevant_logs),
            'affected_resources': self._get_affected_resources(relevant_logs),
            'involved_users': self._get_involved_users(relevant_logs),
            'timeline': self._build_timeline(relevant_logs),
            'suspicious_patterns': self.detect_suspicious_patterns()
        }

        return report

    def _analyze_events(self, logs):
        """Analyze event types and outcomes"""
        event_types = {}
        outcomes = {'success': 0, 'failure': 0}

        for log in logs:
            action = log.get('action')
            outcome = log.get('outcome')

            event_types[action] = event_types.get(action, 0) + 1
            outcomes[outcome] = outcomes.get(outcome, 0) + 1

        return {
            'by_action': event_types,
            'by_outcome': outcomes
        }

    def _get_affected_resources(self, logs):
        """Extract affected resources"""
        resources = set()
        for log in logs:
            target = log.get('target', {}).get('name')
            if target:
                resources.add(target)
        return list(resources)

    def _get_involved_users(self, logs):
        """Extract involved users"""
        users = set()
        for log in logs:
            initiator = log.get('initiator', {}).get('id')
            if initiator:
                users.add(initiator)
        return list(users)

    def _build_timeline(self, logs):
        """Build event timeline"""
        timeline = sorted(logs, key=lambda x: x.get('eventTime'))
        return [
            {
                'time': log.get('eventTime'),
                'user': log.get('initiator', {}).get('id'),
                'action': log.get('action'),
                'resource': log.get('target', {}).get('name'),
                'outcome': log.get('outcome')
            }
            for log in timeline
        ]

# Usage example
if __name__ == "__main__":
    # Load logs from Activity Tracker
    with open('activity_tracker_logs.json', 'r') as f:
        logs = json.load(f)

    analyzer = AuditLogAnalyzer(logs)

    # Detect suspicious patterns
    patterns = analyzer.detect_suspicious_patterns()

    print("=== Brute Force Attempts ===")
    for alert in patterns['brute_force']:
        print(f"User: {alert['user']}, Attempts: {alert['failed_attempts']}")

    print("\n=== Privilege Escalation ===")
    for event in patterns['privilege_escalation']:
        print(f"{event['initiator']} -> {event['target']}: {event['action']}")

    print("\n=== Data Exfiltration ===")
    for event in patterns['data_exfiltration']:
        print(f"User: {event['user']}, Size: {event['size_mb']:.2f} MB")

    # Generate forensic report
    report = analyzer.generate_forensic_report('2024-01-15T14:30:00Z')
    with open('forensic_report.json', 'w') as f:
        json.dump(report, f, indent=2)
```

---

## Continuous Compliance Monitoring

### Terraform for Continuous Monitoring
```hcl
# continuous-monitoring.tf
resource "ibm_scc_profile_attachment" "continuous_monitoring" {
  profile_id  = ibm_scc_profile.production.id
  name        = "Continuous Compliance Monitoring"
  description = "24/7 compliance monitoring with automated remediation"

  scope {
    environment = "production"
    properties {
      name  = "scope_type"
      value = "account"
    }
  }

  schedule      = "every_hour"
  notifications {
    enabled = true
    controls {
      threshold_limit    = 5
      failed_control_ids = [
        "SC-7",   # Boundary Protection
        "AC-2",   # Account Management
        "AU-2",   # Audit Events
        "IA-2",   # Identification and Authentication
        "SC-13"   # Cryptographic Protection
      ]
    }
  }

  attachment_parameters {
    parameter_name         = "encryption_required"
    parameter_value        = "true"
    parameter_display_name = "Require Encryption"
    assessment_type        = "automated"
    assessment_id          = "rule-encryption-check"
  }

  attachment_parameters {
    parameter_name         = "mfa_required"
    parameter_value        = "true"
    parameter_display_name = "Require MFA"
    assessment_type        = "automated"
    assessment_id          = "rule-mfa-check"
  }
}

# Event Notifications for compliance alerts
resource "ibm_resource_instance" "event_notifications" {
  name              = "compliance-notifications"
  service           = "event-notifications"
  plan              = "standard"
  location          = "us-south"
  resource_group_id = data.ibm_resource_group.default.id
}

resource "ibm_en_destination" "slack_destination" {
  instance_guid = ibm_resource_instance.event_notifications.guid
  name          = "Slack Compliance Alerts"
  type          = "slack"

  config {
    params {
      url = var.slack_webhook_url
    }
  }
}

resource "ibm_en_subscription" "compliance_alerts" {
  instance_guid    = ibm_resource_instance.event_notifications.guid
  name             = "Compliance Alert Subscription"
  description      = "Send compliance alerts to Slack"
  destination_id   = ibm_en_destination.slack_destination.destination_id
  topic_id         = ibm_en_topic.compliance.topic_id

  attributes {
    signing_enabled = true
  }
}

# Cloud Functions for automated remediation
resource "ibm_function_namespace" "security_automation" {
  name              = "security-automation"
  resource_group_id = data.ibm_resource_group.default.id
}

resource "ibm_function_action" "auto_remediate" {
  name      = "${ibm_function_namespace.security_automation.name}/auto-remediate"
  namespace = ibm_function_namespace.security_automation.name

  exec {
    kind = "python:3.9"
    code = file("${path.module}/functions/auto_remediate.py")
  }

  limits {
    timeout = 300000  # 5 minutes
    memory  = 256
  }
}

resource "ibm_function_trigger" "compliance_failure" {
  name      = "${ibm_function_namespace.security_automation.name}/compliance-failure-trigger"
  namespace = ibm_function_namespace.security_automation.name

  feed {
    name       = "/whisk.system/alarms/alarm"
    parameters = jsonencode({
      cron = "*/5 * * * *"  # Every 5 minutes
    })
  }
}

resource "ibm_function_rule" "remediation_rule" {
  name         = "${ibm_function_namespace.security_automation.name}/remediation-rule"
  namespace    = ibm_function_namespace.security_automation.name
  trigger_name = ibm_function_trigger.compliance_failure.name
  action_name  = ibm_function_action.auto_remediate.name
}
```

---

## Exam Tips

1. **Compliance Frameworks**:
   - Know supported standards (ISO, SOC, PCI DSS, HIPAA, GDPR, FedRAMP)
   - Understand SCC profile creation and attachment
   - Master compliance scoring and reporting

2. **Audit Logging**:
   - Master Activity Tracker configuration
   - Understand event routing to COS, LogDNA
   - Know how to query and analyze events
   - Understand retention requirements

3. **Incident Response**:
   - Security Advisor findings and occurrences
   - Automated response playbooks
   - Event-driven security automation
   - Forensic analysis procedures

4. **Security Scanning**:
   - Container vulnerability assessment
   - Exemption management
   - CI/CD security integration
   - Automated scanning policies

5. **Backup and Recovery**:
   - Multi-zone deployments
   - Cross-region replication
   - Point-in-time recovery
   - Backup retention policies

6. **Business Continuity**:
   - RTO/RPO requirements
   - Transit Gateway for DR
   - Global Load Balancer failover
   - Multi-region architectures

7. **Automation**:
   - Python SDK for compliance automation
   - Cloud Functions for remediation
   - Event Notifications integration
   - Continuous monitoring patterns

### Common Exam Scenarios

**Scenario 1**: Implement GDPR compliance monitoring
- Create SCC profile with GDPR controls
- Configure encryption at rest and in transit
- Set up Activity Tracker with proper retention
- Implement data residency controls
- Configure automated compliance scanning
- Set up alerts for control failures

**Scenario 2**: Design DR solution with 4-hour RTO
- Deploy multi-zone primary infrastructure
- Configure async replication to DR region
- Set up Transit Gateway connectivity
- Implement Global Load Balancer with health checks
- Configure automated failover
- Document recovery procedures

**Scenario 3**: Respond to security incident
- Isolate affected resources immediately
- Collect evidence via Activity Tracker
- Create Security Advisor finding
- Execute incident response playbook
- Perform forensic analysis
- Generate compliance report
- Implement remediation

**Scenario 4**: Automate compliance remediation
- Set up continuous compliance scanning
- Configure Event Notifications for failures
- Implement Cloud Functions for auto-remediation
- Create remediation playbooks
- Test automated responses
- Monitor remediation effectiveness

**Scenario 5**: Implement container security
- Enable Vulnerability Advisor
- Set up automated scanning in CI/CD
- Configure security policies
- Manage exemptions appropriately
- Integrate with deployment pipeline
- Monitor for new vulnerabilities

### Key Terminology

- **RTO (Recovery Time Objective)**: Maximum acceptable downtime
- **RPO (Recovery Point Objective)**: Maximum acceptable data loss
- **SCC (Security and Compliance Center)**: Centralized compliance management
- **Activity Tracker**: Audit logging service
- **Security Advisor**: Security findings and threat detection
- **Vulnerability Advisor**: Container image scanning
- **CIS (Cloud Internet Services)**: DDoS protection, Global LB
- **Transit Gateway**: Multi-cloud/multi-region connectivity
- **SIEM**: Security Information and Event Management
- **SOAR**: Security Orchestration, Automation and Response

### Hands-On Practice

1. Create and attach SCC profiles
2. Configure Activity Tracker with COS archival
3. Set up Security Advisor custom findings
4. Implement automated compliance scanning
5. Build DR solution with failover testing
6. Create incident response automation
7. Perform vulnerability scanning and remediation
8. Configure backup and recovery procedures
