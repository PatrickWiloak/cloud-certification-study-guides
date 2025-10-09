# Amazon Inspector

## Service Overview and Purpose

Amazon Inspector is an automated vulnerability management service that continually scans AWS workloads for software vulnerabilities and unintended network exposure. Inspector automatically assesses applications for exposure, vulnerabilities, and deviations from best practices, providing detailed security findings with remediation guidance.

**Core Purpose:**
- Automated vulnerability scanning and assessment
- Continuous security monitoring for applications and infrastructure
- Software vulnerability detection and management
- Network reachability analysis
- Compliance and security best practice validation

## Key Features and Capabilities

### Core Scanning Capabilities

1. **Amazon EC2 Instance Scanning**
   - Operating system vulnerabilities
   - Application vulnerabilities
   - Network reachability analysis
   - Security configuration assessment
   - Runtime behavior analysis

2. **Container Image Scanning**
   - ECR container image vulnerability assessment
   - Software composition analysis
   - Base image and layer scanning
   - Continuous monitoring of image repositories
   - Integration with CI/CD pipelines

3. **Lambda Function Scanning**
   - Function code vulnerability assessment
   - Dependency vulnerability scanning
   - Runtime security analysis
   - Package vulnerability detection
   - Layer-by-layer analysis

### Advanced Features

1. **Vulnerability Intelligence**
   - CVE (Common Vulnerabilities and Exposures) database
   - Real-time vulnerability feed updates
   - CVSS scoring and prioritization
   - Exploit prediction scoring
   - Zero-day vulnerability detection

2. **Risk-Based Prioritization**
   - Inspector Risk Score calculation
   - Exploitability and impact assessment
   - Network exposure analysis
   - Business criticality consideration
   - Automated remediation recommendations

3. **Integration and Automation**
   - Security Hub integration
   - EventBridge event notifications
   - Systems Manager integration
   - Third-party tool integration
   - Automated reporting and dashboards

## Use Cases and Scenarios

### Primary Use Cases

1. **Vulnerability Management**: Continuous vulnerability scanning and remediation
2. **Compliance Assessment**: Meeting security compliance requirements
3. **DevSecOps Integration**: Security scanning in CI/CD pipelines
4. **Risk Assessment**: Understanding security risk posture
5. **Incident Response**: Post-incident vulnerability analysis
6. **Security Auditing**: Regular security assessment and reporting
7. **Container Security**: Secure container image deployment
8. **Serverless Security**: Lambda function security assessment
9. **Penetration Testing**: Automated security testing
10. **Regulatory Compliance**: Meeting industry security standards

### Scenario Examples

- **Application Deployment**: Pre-deployment vulnerability scanning
- **Production Monitoring**: Continuous runtime vulnerability monitoring
- **Container Pipeline**: Automated container image security validation
- **Compliance Audit**: Generating security compliance reports
- **Incident Investigation**: Post-breach vulnerability assessment

## Pricing Models and Cost Optimization

### Pricing Structure

1. **EC2 Instance Scanning**
   - $0.30 per instance per month for continuous assessment
   - No charges for stopped or terminated instances
   - Regional pricing variations apply

2. **ECR Container Image Scanning**
   - $0.09 per image scan for initial scan
   - $0.01 per image re-scan for updated images
   - Based on unique image digests

3. **Lambda Function Scanning**
   - $0.30 per function per month for continuous assessment
   - No charges for unused or deleted functions
   - Based on active function versions

### Cost Optimization Strategies

1. **Selective Scanning**: Target critical resources for scanning
2. **Scheduled Scanning**: Optimize scanning frequency for non-critical resources
3. **Image Lifecycle**: Manage container image lifecycle to reduce scan costs
4. **Resource Tagging**: Use tags for cost allocation and optimization
5. **Regional Deployment**: Deploy in cost-effective regions
6. **Integration Efficiency**: Minimize duplicate scanning through automation

## Configuration Details and Best Practices

### Inspector Configuration

```json
{
  "AccountId": "123456789012",
  "Status": "ENABLED",
  "ResourceTypes": [
    "EC2",
    "ECR",
    "LAMBDA"
  ],
  "Ec2Configuration": {
    "ScanMode": "EC2_SSM_AGENT_BASED"
  },
  "EcrConfiguration": {
    "RescanDuration": "LIFETIME"
  }
}
```

### Best Practices

1. **Deployment Strategy**
   - Enable Inspector in all active regions
   - Configure for all relevant resource types
   - Implement centralized finding management
   - Establish baseline security posture
   - Regular configuration review and updates

2. **Finding Management**
   - Implement risk-based prioritization
   - Configure automated suppression rules
   - Establish SLA for finding remediation
   - Integrate with ticketing systems
   - Regular finding review and validation

3. **Integration Patterns**
   - Integrate with CI/CD pipelines for container scanning
   - Configure automated notifications for critical findings
   - Implement Security Hub for centralized management
   - Use Systems Manager for automated patching
   - Establish reporting and dashboard automation

4. **Compliance and Governance**
   - Map findings to compliance frameworks
   - Implement finding tracking and metrics
   - Regular security reporting and review
   - Establish remediation procedures and ownership
   - Continuous improvement of security practices

## Integration with Other AWS Services

### Core Integrations

1. **Security Hub**: Centralized security findings management
2. **Systems Manager**: Automated patching and remediation
3. **EventBridge**: Event-driven automation and notifications
4. **CloudWatch**: Monitoring and alerting for security metrics
5. **CloudFormation**: Infrastructure as code deployment
6. **Lambda**: Custom response and automation functions
7. **SNS**: Finding notifications and alerting
8. **ECR**: Container image vulnerability scanning
9. **CloudTrail**: API activity auditing
10. **Config**: Configuration compliance monitoring

### Advanced Integrations

1. **CodePipeline**: CI/CD security scanning integration
2. **CodeBuild**: Build-time security assessment
3. **Organizations**: Multi-account security management
4. **Trusted Advisor**: Security recommendations
5. **GuardDuty**: Threat detection correlation
6. **Macie**: Data security and classification
7. **Third-party SIEM**: External security tool integration
8. **Vulnerability Management**: External scanning tools

## Security Considerations

### Vulnerability Assessment Coverage

- **Operating System**: OS-level vulnerability detection
- **Application Software**: Application vulnerability scanning
- **Container Images**: Multi-layer container security assessment
- **Serverless Functions**: Lambda function vulnerability analysis
- **Network Exposure**: Reachability and exposure analysis

### Data Protection and Privacy

- **Scan Data Security**: Encrypted scanning and analysis
- **Finding Storage**: Secure storage of vulnerability findings
- **Access Control**: Restricted access to security information
- **Data Retention**: Configurable finding retention policies
- **Compliance**: Support for regulatory requirements

### Risk Management

- **CVSS Scoring**: Industry-standard vulnerability scoring
- **Exploit Prediction**: Likelihood of exploitation analysis
- **Environmental Context**: Network and system context consideration
- **Business Impact**: Criticality and impact assessment

## Monitoring and Troubleshooting

### CloudWatch Metrics

- **FindingsGenerated**: Number of new findings
- **CriticalFindings**: High-severity finding count
- **CoveredResources**: Number of resources being scanned
- **ScanDuration**: Time taken for scanning operations

### Common Issues and Solutions

1. **Missing Scan Coverage**
   - Verify SSM agent installation and connectivity
   - Check IAM permissions for Inspector service
   - Validate resource tags and filtering rules
   - Review regional service availability

2. **High False Positive Rate**
   - Review and adjust suppression rules
   - Validate vulnerability context and applicability
   - Configure finding severity thresholds
   - Implement whitelist for known acceptable findings

3. **Performance Impact**
   - Monitor resource utilization during scans
   - Adjust scanning schedules for non-critical resources
   - Optimize network bandwidth usage
   - Review scan configuration and frequency

4. **Integration Issues**
   - Verify EventBridge rule configuration
   - Check Security Hub integration settings
   - Validate Lambda function permissions
   - Test automated response workflows

### Monitoring Tools

1. **Inspector Dashboard**: Real-time vulnerability metrics
2. **Security Hub**: Centralized security posture management
3. **CloudWatch Dashboards**: Custom security monitoring
4. **Third-party Tools**: External vulnerability management integration

## Exam-Specific Tips and Common Scenarios

### Solutions Architect Associate (SAA-C03)

- **Security Architecture**: Implementing Inspector in security design
- **Vulnerability Management**: Automated vulnerability assessment strategies
- **Integration Patterns**: Combining with other security services
- **Cost Optimization**: Balancing security coverage with costs

### Solutions Architect Professional (SAP-C02)

- **Enterprise Security**: Large-scale vulnerability management
- **Compliance Architecture**: Meeting regulatory security requirements
- **Automation Strategy**: Advanced security automation workflows
- **Multi-Account Management**: Centralized vulnerability management

### Security Specialty (SCS-C02)

- **Advanced Configuration**: Custom vulnerability management workflows
- **Risk Assessment**: Comprehensive security risk evaluation
- **Incident Response**: Vulnerability-based incident investigation
- **Compliance Management**: Regulatory compliance automation

### Developer Associate (DVA-C02)

- **Application Security**: Integrating security scanning in development
- **Container Security**: Secure container development practices
- **CI/CD Integration**: Security scanning in deployment pipelines
- **Serverless Security**: Lambda function security best practices

### Common Exam Scenarios

1. **Scenario**: Need automated vulnerability scanning for EC2 instances
   **Solution**: Enable Inspector with SSM agent-based scanning

2. **Scenario**: Secure container deployment pipeline
   **Solution**: Integrate Inspector ECR scanning with CI/CD pipeline

3. **Scenario**: Compliance requirement for vulnerability assessment
   **Solution**: Configure Inspector with Security Hub for compliance reporting

4. **Scenario**: Automated response to critical vulnerabilities
   **Solution**: Use Inspector with EventBridge and Lambda for automated remediation

## Hands-on Examples and CLI Commands

### Inspector Setup and Configuration

```bash
# Enable Inspector
aws inspector2 enable \
  --account-ids 123456789012 \
  --resource-types EC2 ECR LAMBDA

# Get Inspector configuration
aws inspector2 describe-organization-configuration

# Update Inspector configuration
aws inspector2 update-organization-configuration \
  --auto-enable ec2=true,ecr=true,lambda=true

# Disable Inspector
aws inspector2 disable \
  --account-ids 123456789012 \
  --resource-types EC2 ECR LAMBDA

# Get account permissions
aws inspector2 get-permissions

# List supported regions
aws inspector2 list-account-permissions
```

### Finding Management

```bash
# List findings
aws inspector2 list-findings \
  --filter-criteria '{
    "severity": [{"comparison": "EQUALS", "value": "CRITICAL"}],
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_EC2_INSTANCE"}]
  }' \
  --max-results 100

# Get finding details
aws inspector2 get-findings \
  --finding-arns arn:aws:inspector2:us-east-1:123456789012:finding/12345678-1234-1234-1234-123456789012

# Create findings report
aws inspector2 create-findings-report \
  --report-format JSON \
  --s3-destination bucketName=security-reports,keyPrefix=inspector-findings/ \
  --filter-criteria '{
    "severity": [{"comparison": "EQUALS", "value": "HIGH"}]
  }'

# Get findings report status
aws inspector2 get-findings-report-status \
  --report-id 12345678-1234-1234-1234-123456789012

# List finding aggregations
aws inspector2 list-finding-aggregations \
  --aggregation-type TITLE \
  --aggregation-request '{
    "titleAggregation": {
      "resourceType": "AWS_EC2_INSTANCE",
      "sortBy": "CRITICAL",
      "sortOrder": "DESC"
    }
  }'
```

### Coverage and Resource Management

```bash
# List coverage
aws inspector2 list-coverage \
  --filter-criteria '{
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_EC2_INSTANCE"}],
    "scanStatusCode": [{"comparison": "EQUALS", "value": "ACTIVE"}]
  }' \
  --max-results 100

# Get coverage statistics
aws inspector2 get-coverage-statistics \
  --filter-criteria '{
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_ECR_CONTAINER_IMAGE"}]
  }'

# List covered resources
aws inspector2 list-covered-resources \
  --filter-criteria '{
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_LAMBDA_FUNCTION"}]
  }'

# Search vulnerabilities
aws inspector2 search-vulnerabilities \
  --filter-criteria '{
    "vulnerabilityId": [{"comparison": "EQUALS", "value": "CVE-2021-44228"}]
  }'
```

### ECR Container Scanning

```bash
# Scan ECR repository
aws inspector2 batch-get-account-status \
  --account-ids 123456789012

# List repository scan results
aws inspector2 list-findings \
  --filter-criteria '{
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_ECR_CONTAINER_IMAGE"}],
    "ecrImageRepositoryName": [{"comparison": "EQUALS", "value": "my-app"}]
  }'

# Get image scan details
aws inspector2 get-findings \
  --finding-arns arn:aws:inspector2:us-east-1:123456789012:finding/12345678-1234-1234-1234-123456789012

# Configure ECR scan settings
aws inspector2 update-configuration \
  --ecr-configuration rescanDuration=LIFETIME
```

### Lambda Function Scanning

```bash
# List Lambda function findings
aws inspector2 list-findings \
  --filter-criteria '{
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_LAMBDA_FUNCTION"}],
    "severity": [{"comparison": "EQUALS", "value": "HIGH"}]
  }'

# Get Lambda function coverage
aws inspector2 list-coverage \
  --filter-criteria '{
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_LAMBDA_FUNCTION"}]
  }'

# Lambda vulnerability search
aws inspector2 search-vulnerabilities \
  --filter-criteria '{
    "vulnerabilitySource": [{"comparison": "EQUALS", "value": "NVD"}]
  }'
```

### Suppression Rules

```bash
# Create suppression rule
aws inspector2 create-filter \
  --action SUPPRESS \
  --filter-criteria '{
    "vulnerabilityId": [{"comparison": "EQUALS", "value": "CVE-2021-12345"}],
    "resourceType": [{"comparison": "EQUALS", "value": "AWS_EC2_INSTANCE"}]
  }' \
  --name "Suppress known false positive" \
  --description "Suppress CVE-2021-12345 for EC2 instances"

# List suppression rules
aws inspector2 list-filters

# Update suppression rule
aws inspector2 update-filter \
  --filter-arn arn:aws:inspector2:us-east-1:123456789012:filter/12345678-1234-1234-1234-123456789012 \
  --action SUPPRESS \
  --filter-criteria '{
    "vulnerabilityId": [{"comparison": "EQUALS", "value": "CVE-2021-12345"}]
  }'

# Delete suppression rule
aws inspector2 delete-filter \
  --filter-arn arn:aws:inspector2:us-east-1:123456789012:filter/12345678-1234-1234-1234-123456789012
```

### Automated Response Integration

```bash
# Create EventBridge rule for Inspector findings
aws events put-rule \
  --name "InspectorCriticalFindingRule" \
  --event-pattern '{
    "source": ["aws.inspector2"],
    "detail-type": ["Inspector2 Finding"],
    "detail": {
      "severity": ["CRITICAL", "HIGH"]
    }
  }'

# Add Lambda target for automated response
aws events put-targets \
  --rule "InspectorCriticalFindingRule" \
  --targets '[
    {
      "Id": "1",
      "Arn": "arn:aws:lambda:us-east-1:123456789012:function:InspectorResponse"
    }
  ]'

# Create CloudWatch alarm for finding count
aws cloudwatch put-metric-alarm \
  --alarm-name "InspectorCriticalFindings" \
  --alarm-description "Alert on critical Inspector findings" \
  --metric-name "FindingCount" \
  --namespace "AWS/Inspector" \
  --statistic Sum \
  --period 3600 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1
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
    Automated response to Inspector findings
    """

    # Parse Inspector finding
    detail = event['detail']
    finding_arn = detail['findingArn']
    severity = detail['severity']
    resource_type = detail['resources'][0]['type']

    # Initialize AWS clients
    ssm = boto3.client('ssm')
    sns = boto3.client('sns')
    inspector = boto3.client('inspector2')

    logger.info(f"Processing finding: {finding_arn} with severity {severity}")

    # Automated response based on severity and resource type
    if severity in ['CRITICAL', 'HIGH']:
        if resource_type == 'AWS_EC2_INSTANCE':
            # Trigger automated patching
            trigger_automated_patching(detail, ssm)

        elif resource_type == 'AWS_ECR_CONTAINER_IMAGE':
            # Block vulnerable image deployment
            block_vulnerable_image(detail)

        elif resource_type == 'AWS_LAMBDA_FUNCTION':
            # Update Lambda function
            update_lambda_function(detail)

        # Send high priority notification
        send_security_alert(detail, sns)

    return {
        'statusCode': 200,
        'body': json.dumps('Inspector finding processed successfully')
    }

def trigger_automated_patching(detail, ssm):
    """Trigger automated patching for EC2 instances"""
    instance_id = detail['resources'][0]['id']

    # Run patch baseline
    response = ssm.send_command(
        InstanceIds=[instance_id],
        DocumentName='AWS-RunPatchBaseline',
        Parameters={
            'Operation': ['Install'],
            'RebootOption': ['RebootIfNeeded']
        }
    )

    logger.info(f"Triggered patching for instance {instance_id}: {response['Command']['CommandId']}")

def block_vulnerable_image(detail):
    """Block deployment of vulnerable container image"""
    logger.info("Implementing container image deployment block")
    # Implementation for blocking vulnerable image deployment

def update_lambda_function(detail):
    """Update Lambda function with security patches"""
    logger.info("Triggering Lambda function update")
    # Implementation for Lambda function security updates

def send_security_alert(detail, sns):
    """Send security alert notification"""
    message = f"""
    SECURITY ALERT: Inspector Finding

    Severity: {detail['severity']}
    Resource: {detail['resources'][0]['id']}
    Type: {detail['resources'][0]['type']}
    CVE: {detail.get('packageVulnerabilityDetails', {}).get('vulnerabilityId', 'N/A')}

    Finding ARN: {detail['findingArn']}

    Please investigate and remediate immediately.
    """

    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:inspector-alerts',
        Subject='Critical Inspector Vulnerability Finding',
        Message=message
    )
```

### Compliance Reporting

```bash
# Generate compliance report
aws inspector2 create-findings-report \
  --report-format CSV \
  --s3-destination bucketName=compliance-reports,keyPrefix=inspector/ \
  --filter-criteria '{
    "severity": [
      {"comparison": "EQUALS", "value": "CRITICAL"},
      {"comparison": "EQUALS", "value": "HIGH"}
    ],
    "findingStatus": [{"comparison": "EQUALS", "value": "ACTIVE"}]
  }'

# Get vulnerability statistics for compliance
aws inspector2 get-findings-report-status \
  --report-id 12345678-1234-1234-1234-123456789012

# List findings by compliance framework
aws inspector2 list-findings \
  --filter-criteria '{
    "inspectorScore": [{"lowerInclusive": 7.0, "upperInclusive": 10.0}]
  }' \
  --sort-criteria '{
    "field": "INSPECTOR_SCORE",
    "sortOrder": "DESC"
  }'
```

This comprehensive Inspector documentation provides detailed coverage of vulnerability management capabilities essential for AWS certification preparation and real-world security operations.