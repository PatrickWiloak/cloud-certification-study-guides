# Amazon GuardDuty

## Service Overview and Purpose

Amazon GuardDuty is a threat detection service that continuously monitors for malicious activity and unauthorized behavior to protect AWS accounts, workloads, and data. It uses machine learning, anomaly detection, and integrated threat intelligence to identify threats such as compromised instances, cryptocurrency mining, reconnaissance, and data exfiltration.

**Core Purpose:**
- Continuous threat detection and monitoring
- Automated security analysis using machine learning
- Integration with threat intelligence feeds
- Centralized security findings management
- Proactive threat hunting and incident response

## Key Features and Capabilities

### Core Detection Capabilities

1. **Threat Intelligence**
   - AWS threat intelligence feeds
   - Third-party commercial threat intelligence
   - Community-sourced threat intelligence
   - Custom threat intelligence integration
   - Real-time updates and analysis

2. **Machine Learning Analysis**
   - Anomaly detection for unusual behavior
   - Behavioral modeling for users and resources
   - Statistical analysis of network traffic
   - Pattern recognition for known attack vectors
   - Adaptive learning from environmental changes

3. **Data Sources**
   - VPC Flow Logs for network activity analysis
   - CloudTrail logs for API activity monitoring
   - DNS logs for domain resolution analysis
   - Kubernetes audit logs (EKS protection)
   - S3 data events for data access monitoring

### Advanced Features

1. **Malware Protection**
   - EBS volume scanning for malware
   - Runtime malware detection
   - File integrity monitoring
   - Custom malware signatures
   - Quarantine and remediation recommendations

2. **Multi-Account Management**
   - Centralized findings across accounts
   - Delegated administrator capabilities
   - Automated member account invitation
   - Unified threat intelligence sharing
   - Cross-account remediation workflows

3. **Finding Types**
   - Reconnaissance attacks
   - Instance compromise indicators
   - Data exfiltration attempts
   - Cryptocurrency mining detection
   - Backdoor and trojan detection
   - Command and control communication

## Use Cases and Scenarios

### Primary Use Cases

1. **Threat Detection**: Continuous monitoring for security threats
2. **Incident Response**: Automated threat detection and alerting
3. **Compliance Monitoring**: Security compliance and audit support
4. **Forensic Analysis**: Post-incident investigation and analysis
5. **Threat Hunting**: Proactive security threat identification
6. **Multi-Account Security**: Centralized security across AWS accounts
7. **DevSecOps Integration**: Security monitoring in CI/CD pipelines
8. **Cloud Migration Security**: Monitoring during cloud transitions
9. **Insider Threat Detection**: Unusual user behavior identification
10. **Supply Chain Security**: Third-party access monitoring

### Scenario Examples

- **Compromised EC2 Instance**: Detection of malware or unauthorized access
- **Data Exfiltration**: Unusual data transfer patterns and volumes
- **Cryptocurrency Mining**: Detection of unauthorized mining activities
- **Privileged Escalation**: Unusual IAM activity and privilege changes
- **DNS Tunneling**: Detection of covert communication channels

## Pricing Models and Cost Optimization

### Pricing Structure

1. **CloudTrail Event Analysis**
   - $4.00 per 1 million events analyzed
   - First 30 days free for new accounts
   - Based on actual CloudTrail events processed

2. **VPC Flow Log Analysis**
   - $1.00 per GB of flow log data analyzed
   - Based on actual flow log data processed
   - Regional data processing charges

3. **DNS Log Analysis**
   - $0.40 per 1 million DNS queries analyzed
   - Based on actual DNS queries processed
   - Cross-region query analysis

4. **S3 Data Event Analysis**
   - $0.80 per 1 million data events analyzed
   - Based on S3 object-level API calls
   - Data access pattern analysis

5. **Malware Protection**
   - $0.10 per GB of EBS volume data scanned
   - On-demand and scheduled scanning
   - Compressed data processing optimization

### Cost Optimization Strategies

1. **Selective Monitoring**: Enable only necessary data sources
2. **Regional Optimization**: Deploy in cost-effective regions
3. **Data Filtering**: Reduce unnecessary log volume
4. **Scheduled Scanning**: Optimize malware scanning frequency
5. **Finding Management**: Automated finding triage and cleanup
6. **Multi-Account Efficiency**: Centralized management to reduce overhead

## Configuration Details and Best Practices

### GuardDuty Configuration

```json
{
  "DetectorId": "12abc34d567e8f90g1h234i56789j0k",
  "ServiceRole": "arn:aws:iam::123456789012:role/aws-guardduty-role",
  "Status": "ENABLED",
  "FindingPublishingFrequency": "FIFTEEN_MINUTES",
  "DataSources": {
    "S3Logs": {
      "Enable": true
    },
    "KubernetesAuditLogs": {
      "Enable": true
    },
    "MalwareProtection": {
      "ScanEc2InstanceWithFindings": {
        "EbsVolumes": true
      }
    }
  }
}
```

### Best Practices

1. **Initial Setup**
   - Enable GuardDuty in all active regions
   - Configure centralized management for multi-account environments
   - Set up automated finding notifications
   - Establish baseline behavior patterns
   - Configure threat intelligence feeds

2. **Multi-Account Management**
   - Designate a security account as administrator
   - Automate member account invitation and management
   - Implement consistent finding suppression rules
   - Centralize threat intelligence and custom rules
   - Establish cross-account remediation procedures

3. **Finding Management**
   - Implement automated finding triage
   - Configure appropriate suppression rules
   - Set up finding severity-based workflows
   - Establish escalation procedures
   - Regular review and cleanup of archived findings

4. **Integration and Automation**
   - Integrate with SIEM and security tools
   - Automate response to critical findings
   - Implement custom threat intelligence feeds
   - Configure automated remediation workflows
   - Establish threat hunting procedures

## Integration with Other AWS Services

### Core Integrations

1. **Security Hub**: Centralized security findings management
2. **CloudWatch Events/EventBridge**: Automated response triggers
3. **Lambda**: Custom response and remediation functions
4. **SNS**: Finding notifications and alerting
5. **CloudFormation**: Infrastructure as code deployment
6. **Systems Manager**: Automated incident response
7. **IAM**: Access control and service permissions
8. **CloudTrail**: API activity monitoring and analysis
9. **VPC**: Network traffic analysis and monitoring
10. **S3**: Data access monitoring and protection

### Advanced Integrations

1. **Inspector**: Vulnerability assessment correlation
2. **Macie**: Data classification and protection
3. **Config**: Configuration compliance monitoring
4. **Trusted Advisor**: Security recommendations
5. **Organizations**: Multi-account security management
6. **KMS**: Encryption and key management integration
7. **Detective**: Deep investigation and analysis
8. **Third-party SIEM**: External security tool integration

## Security Considerations

### Threat Detection Coverage

- **Network-based Attacks**: Traffic analysis and anomaly detection
- **Host-based Threats**: Instance behavior and malware detection
- **Identity-based Attacks**: Unusual user and role activity
- **Data Protection**: Unauthorized access and exfiltration detection
- **Infrastructure Threats**: Service abuse and resource compromise

### Data Privacy and Protection

- **Log Processing**: Secure analysis of sensitive log data
- **Finding Storage**: Encrypted storage of security findings
- **Access Control**: Restricted access to security information
- **Data Retention**: Configurable finding retention policies
- **Compliance**: Support for regulatory requirements

### Threat Intelligence

- **Real-time Updates**: Continuous threat intelligence refresh
- **Custom Feeds**: Integration with proprietary threat intelligence
- **Attribution Analysis**: Understanding attack sources and methods
- **False Positive Reduction**: ML-based accuracy improvements

## Monitoring and Troubleshooting

### CloudWatch Metrics

- **FindingCount**: Number of findings generated
- **ThreatIntelSetSize**: Size of threat intelligence sets
- **IPSetSize**: Size of trusted IP sets
- **DataSourceCost**: Cost of data source analysis

### Common Issues and Solutions

1. **High False Positive Rate**
   - Review and adjust threat intelligence feeds
   - Implement finding suppression rules
   - Configure trusted IP sets for known good sources
   - Adjust sensitivity settings for specific finding types

2. **Missing Expected Findings**
   - Verify data source configuration and enablement
   - Check service permissions and IAM roles
   - Validate CloudTrail and VPC Flow Logs configuration
   - Review finding suppression rules for over-filtering

3. **Cost Management**
   - Monitor data source usage and costs
   - Optimize log volume and frequency
   - Review and adjust scanning schedules
   - Implement cost alerts and budgets

4. **Integration Issues**
   - Verify EventBridge rule configuration
   - Check Lambda function permissions and triggers
   - Validate Security Hub integration settings
   - Test automated response workflows

### Monitoring Tools

1. **GuardDuty Console**: Real-time findings and analysis
2. **Security Hub Dashboard**: Centralized security posture
3. **CloudWatch Dashboards**: Custom metrics and monitoring
4. **Third-party SIEM**: External security tool integration

## Exam-Specific Tips and Common Scenarios

### Solutions Architect Associate (SAA-C03)

- **Threat Detection Architecture**: Implementing GuardDuty in security design
- **Multi-Account Strategy**: Centralized security monitoring approach
- **Cost Considerations**: Balancing security coverage with costs
- **Integration Patterns**: Combining with other security services

### Solutions Architect Professional (SAP-C02)

- **Enterprise Security**: Large-scale GuardDuty deployment strategies
- **Advanced Threat Hunting**: Proactive security monitoring approaches
- **Compliance Architecture**: Meeting regulatory security requirements
- **Incident Response**: Automated response and remediation workflows

### Security Specialty (SCS-C02)

- **Advanced Configuration**: Custom threat intelligence and finding management
- **Threat Analysis**: Deep dive into finding types and investigation
- **Integration Strategy**: Comprehensive security architecture with GuardDuty
- **Incident Response**: Advanced response and forensic capabilities

### Developer Associate (DVA-C02)

- **Application Security**: Monitoring application-specific threats
- **API Security**: Detecting API abuse and attacks
- **DevSecOps Integration**: Security monitoring in development workflows
- **Automated Response**: Building security automation with Lambda

### Common Exam Scenarios

1. **Scenario**: Need automated threat detection for AWS environment
   **Solution**: Enable GuardDuty with multi-account management and automated response

2. **Scenario**: Detecting data exfiltration from S3 buckets
   **Solution**: Enable S3 data event monitoring in GuardDuty

3. **Scenario**: Centralized security monitoring across organization
   **Solution**: Configure GuardDuty with Organizations integration

4. **Scenario**: Automated incident response to security threats
   **Solution**: Use GuardDuty with EventBridge and Lambda for automated response

## Hands-on Examples and CLI Commands

### GuardDuty Setup and Management

```bash
# Enable GuardDuty
aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES

# Get detector information
aws guardduty get-detector --detector-id 12abc34d567e8f90g1h234i56789j0k

# Update detector configuration
aws guardduty update-detector \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --enable \
  --finding-publishing-frequency SIX_HOURS \
  --data-sources '{
    "S3Logs": {"Enable": true},
    "KubernetesAuditLogs": {"Enable": true},
    "MalwareProtection": {
      "ScanEc2InstanceWithFindings": {"EbsVolumes": true}
    }
  }'

# List detectors
aws guardduty list-detectors

# Disable GuardDuty
aws guardduty delete-detector --detector-id 12abc34d567e8f90g1h234i56789j0k
```

### Multi-Account Management

```bash
# Create GuardDuty administrator account
aws guardduty enable-organization-admin-account \
  --admin-account-id 123456789012

# Invite member account
aws guardduty create-members \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --account-details '[
    {
      "AccountId": "123456789013",
      "Email": "security@member.com"
    }
  ]'

# Accept invitation (from member account)
aws guardduty accept-invitation \
  --detector-id 23bcd45e678f9g01h2i345j67890k1l \
  --master-id 123456789012 \
  --invitation-id 84b097ab026d4ad5adbaabc4ba51beb4

# List members
aws guardduty list-members \
  --detector-id 12abc34d567e8f90g1h234i56789j0k

# Get member details
aws guardduty get-members \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --account-ids 123456789013

# Disassociate member
aws guardduty disassociate-members \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --account-ids 123456789013
```

### Finding Management

```bash
# List findings
aws guardduty list-findings \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --finding-criteria '{
    "Criterion": {
      "severity": {
        "Gte": 7.0
      },
      "type": {
        "Eq": ["Recon:EC2/PortProbeUnprotectedPort"]
      }
    }
  }' \
  --max-results 50

# Get finding details
aws guardduty get-findings \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --finding-ids 34c567890def1234567890abcdef1234

# Archive findings
aws guardduty archive-findings \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --finding-ids 34c567890def1234567890abcdef1234

# Unarchive findings
aws guardduty unarchive-findings \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --finding-ids 34c567890def1234567890abcdef1234

# Update finding feedback
aws guardduty update-findings-feedback \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --finding-ids 34c567890def1234567890abcdef1234 \
  --feedback USEFUL
```

### Threat Intelligence Management

```bash
# Create threat intelligence set
aws guardduty create-threat-intel-set \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --name "MaliciousIPs" \
  --format TXT \
  --location s3://my-threat-intel/malicious-ips.txt \
  --activate

# Create IP set (trusted IPs)
aws guardduty create-ip-set \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --name "TrustedIPs" \
  --format TXT \
  --location s3://my-threat-intel/trusted-ips.txt \
  --activate

# List threat intelligence sets
aws guardduty list-threat-intel-sets \
  --detector-id 12abc34d567e8f90g1h234i56789j0k

# List IP sets
aws guardduty list-ip-sets \
  --detector-id 12abc34d567e8f90g1h234i56789j0k

# Update threat intelligence set
aws guardduty update-threat-intel-set \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --threat-intel-set-id 45d678901ef2345678901bcdef234567 \
  --location s3://my-threat-intel/updated-malicious-ips.txt \
  --activate

# Delete threat intelligence set
aws guardduty delete-threat-intel-set \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --threat-intel-set-id 45d678901ef2345678901bcdef234567
```

### Malware Protection

```bash
# Get malware scan settings
aws guardduty get-malware-scan-settings \
  --detector-id 12abc34d567e8f90g1h234i56789j0k

# Update malware scan settings
aws guardduty update-malware-scan-settings \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --scan-resource-criteria '{
    "Include": {
      "EC2_INSTANCE_TAG": {
        "Key": "Environment",
        "Value": "Production"
      }
    }
  }' \
  --ebs-snapshot-preservation RETENTION_WITH_FINDING

# Describe malware scans
aws guardduty describe-malware-scans \
  --detector-id 12abc34d567e8f90g1h234i56789j0k \
  --max-results 50

# Start on-demand malware scan
aws guardduty start-malware-scan \
  --resource-arn arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0
```

### Automated Response Integration

```bash
# Create EventBridge rule for GuardDuty findings
aws events put-rule \
  --name "GuardDutyFindingRule" \
  --event-pattern '{
    "source": ["aws.guardduty"],
    "detail-type": ["GuardDuty Finding"],
    "detail": {
      "severity": [7.0, 8.0, 8.9]
    }
  }'

# Add Lambda target to EventBridge rule
aws events put-targets \
  --rule "GuardDutyFindingRule" \
  --targets '[
    {
      "Id": "1",
      "Arn": "arn:aws:lambda:us-east-1:123456789012:function:GuardDutyResponse"
    }
  ]'

# Create SNS notification for findings
aws sns create-topic --name guardduty-alerts

aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:guardduty-alerts \
  --protocol email \
  --notification-endpoint security@example.com
```

### Custom Response Lambda Function

```python
import boto3
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """
    Automated response to GuardDuty findings
    """

    # Parse GuardDuty finding
    detail = event['detail']
    finding_type = detail['type']
    severity = detail['severity']

    # Initialize AWS clients
    ec2 = boto3.client('ec2')
    sns = boto3.client('sns')

    logger.info(f"Processing finding: {finding_type} with severity {severity}")

    # Automated response based on finding type
    if finding_type.startswith('UnauthorizedAPICall'):
        # Disable compromised IAM credentials
        handle_compromised_credentials(detail)

    elif finding_type.startswith('CryptoCurrency'):
        # Isolate compromised instance
        handle_crypto_mining(detail, ec2)

    elif finding_type.startswith('Trojan'):
        # Quarantine infected instance
        handle_malware_infection(detail, ec2)

    # Send notification for high severity findings
    if severity >= 7.0:
        send_security_alert(detail, sns)

    return {
        'statusCode': 200,
        'body': json.dumps('GuardDuty finding processed successfully')
    }

def handle_compromised_credentials(detail):
    """Handle compromised IAM credentials"""
    logger.info("Handling compromised credentials")
    # Implementation for credential response

def handle_crypto_mining(detail, ec2):
    """Handle cryptocurrency mining detection"""
    if 'instanceDetails' in detail['service']:
        instance_id = detail['service']['instanceDetails']['instanceId']

        # Create isolation security group
        response = ec2.create_security_group(
            GroupName=f'isolation-{instance_id}',
            Description='Isolation security group for compromised instance'
        )

        # Replace instance security groups
        ec2.modify_instance_attribute(
            InstanceId=instance_id,
            Groups=[response['GroupId']]
        )

        logger.info(f"Isolated instance {instance_id}")

def handle_malware_infection(detail, ec2):
    """Handle malware infection"""
    logger.info("Handling malware infection")
    # Implementation for malware response

def send_security_alert(detail, sns):
    """Send security alert notification"""
    message = f"""
    SECURITY ALERT: GuardDuty Finding

    Type: {detail['type']}
    Severity: {detail['severity']}
    Description: {detail['description']}
    Account: {detail['accountId']}
    Region: {detail['region']}

    Please investigate immediately.
    """

    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:guardduty-alerts',
        Subject='High Severity GuardDuty Finding',
        Message=message
    )
```

This comprehensive GuardDuty documentation provides detailed coverage of threat detection capabilities and security monitoring essential for AWS certification preparation and real-world security operations.