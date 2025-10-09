# Security Operations - GCP Professional Cloud Security Engineer

## Overview

Security monitoring, incident response, vulnerability management, and operational security practices for GCP environments.

## Security Monitoring

### Cloud Logging for Security
```sql
-- Detect privilege escalation
SELECT
  timestamp,
  protoPayload.authenticationInfo.principalEmail,
  protoPayload.authorizationInfo
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.methodName LIKE "%setIamPolicy%"
  AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY);

-- Suspicious resource deletion
SELECT timestamp, principalEmail, resourceName, methodName
FROM `project.dataset.cloudaudit_googleapis_com_activity_*`
WHERE protoPayload.methodName LIKE "%.delete"
  AND severity = "ERROR";
```

### Security Command Center
- Asset Discovery
- Security Health Analytics
- Web Security Scanner
- Event Threat Detection
- Container Threat Detection

### Alerting
```yaml
displayName: "Suspicious IAM Activity"
conditions:
  - displayName: "Multiple failed auth attempts"
    conditionThreshold:
      filter: 'metric.type="logging.googleapis.com/user/failed_auth" resource.type="global"'
      comparison: COMPARISON_GT
      thresholdValue: 5
      duration: 300s
notificationChannels: ["projects/PROJECT/notificationChannels/CHANNEL"]
documentation:
  content: "Multiple failed authentication attempts detected. Investigate for potential breach."
```

## Incident Response

### Response Process
1. **Detection**: Automated alerts, SCC findings
2. **Containment**: Isolate affected resources
3. **Eradication**: Remove threat, patch vulnerabilities
4. **Recovery**: Restore services
5. **Post-Incident**: Review, improve

### Forensics
```bash
# Capture instance for forensics
gcloud compute disks snapshot DISK_NAME --snapshot-names=forensic-snapshot-$(date +%Y%m%d)

# Create forensic instance
gcloud compute instances create forensic-instance \
  --source-snapshot=forensic-snapshot-20240101 \
  --no-address

# Export logs for analysis
gcloud logging read "timestamp>=\"2024-01-01T00:00:00Z\"" \
  --format=json > investigation-logs.json
```

## Vulnerability Management

### Container Scanning
```bash
# Enable Container Analysis API
gcloud services enable containeranalysis.googleapis.com

# Scan image
gcloud container images scan gcr.io/PROJECT/IMAGE:TAG

# View vulnerabilities
gcloud container images list-tags gcr.io/PROJECT/IMAGE \
  --filter="tags:TAG" \
  --format="get(digest)" | \
  xargs -I {} gcloud container images describe gcr.io/PROJECT/IMAGE@{} \
  --show-package-vulnerability
```

### Binary Authorization
```bash
# Create attestor
gcloud beta container binauthz attestors create security-reviewer \
  --attestation-authority-note=projects/PROJECT/notes/security-note \
  --attestation-authority-note-project=PROJECT

# Create policy requiring attestations
cat > policy.yaml <<EOF
defaultAdmissionRule:
  requireAttestationsBy:
    - projects/PROJECT/attestors/security-reviewer
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
EOF

gcloud beta container binauthz policy import policy.yaml
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
1. Enable all audit log types
2. Real-time alerting for security events
3. Log aggregation and analysis
4. Regular SCC findings review
5. Automated compliance checks
6. Baseline normal behavior
7. Threat intelligence integration

### Incident Response
1. Documented IR procedures
2. Regular tabletop exercises
3. Forensic readiness
4. Communication plan
5. Evidence preservation
6. Post-incident reviews
7. Continuous improvement

### Vulnerability Management
1. Continuous scanning
2. Patch management process
3. Binary Authorization for containers
4. Regular penetration testing
5. Bug bounty program
6. Vulnerability disclosure process
7. Risk-based prioritization

## Study Tips

1. Practice security monitoring queries
2. Set up Security Command Center
3. Configure alerting policies
4. Implement Binary Authorization
5. Conduct incident response drills
6. Understand compliance requirements
7. Vulnerability scanning and remediation

## Additional Resources

- [Security Command Center](https://cloud.google.com/security-command-center)
- [Incident Response](https://cloud.google.com/architecture/incident-response)
- [Container Security](https://cloud.google.com/container-analysis/docs)
- [Binary Authorization](https://cloud.google.com/binary-authorization/docs)
