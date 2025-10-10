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

### Exam Tips

1. **Compliance**: Know major frameworks, SCC capabilities
2. **Audit Logging**: Master Activity Tracker, event routing
3. **Incident Response**: Understand Security Advisor, playbooks
4. **Scanning**: Know VA, exemptions, CI/CD integration
5. **Backup/DR**: Multi-zone, cross-region, RTO/RPO concepts
6. **Business Continuity**: Transit Gateway, Global LB, failover strategies

### Common Scenarios

**Scenario 1**: Implement GDPR compliance monitoring
- Use SCC with GDPR profile
- Configure data encryption
- Set up audit logging
- Implement data retention policies

**Scenario 2**: Design DR solution with 4-hour RTO
- Multi-zone primary deployment
- Async replication to DR region
- Transit Gateway connectivity
- Global Load Balancer with health checks

**Scenario 3**: Respond to security incident
- Isolate affected resources
- Collect evidence via Activity Tracker
- Create Security Advisor finding
- Execute incident response playbook
- Generate compliance report
