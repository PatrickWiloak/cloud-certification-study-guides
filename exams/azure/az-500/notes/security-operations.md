# Manage Security Operations

## Overview
This domain covers configuring centralized policy management, managing threat protection, and implementing security monitoring and automation. It represents 25-30% of the exam and focuses on operational security and incident response.

## Key Topics

### Centralized Policy Management
- Custom security policy configuration
- Policy initiative creation
- Security settings and auditing with Azure Policy
- Regulatory compliance monitoring
- Policy remediation and enforcement

### Threat Protection
- Microsoft Defender for Servers configuration
- Vulnerability scan evaluation from Defender
- Microsoft Defender for SQL configuration
- Microsoft Threat Modeling Tool usage
- Security alerts investigation and response

### Security Monitoring and Automation
- Security event monitoring with Azure Monitor
- Data retention policy configuration
- Microsoft Sentinel configuration and management
- Cloud workload protection assessment
- Workflow automation design and configuration
- Security playbook implementation

## Services Reference

### Core Services
- Azure Policy
- Microsoft Defender for Cloud
- Microsoft Sentinel
- Azure Monitor
- Log Analytics

### Supporting Services
- Microsoft Defender for Servers
- Microsoft Defender for SQL
- Microsoft Defender for Storage
- Microsoft Defender for Containers
- Azure Logic Apps (for automation)

## Best Practices

### Policy Management
- Use built-in policies before creating custom ones
- Group related policies into initiatives
- Test policies in audit mode before enforcement
- Document policy intent and parameters
- Use policy exemptions judiciously

### Threat Detection
- Enable Microsoft Defender for Cloud on all subscriptions
- Configure security contacts for alert notifications
- Regularly review security recommendations
- Prioritize high-severity alerts and vulnerabilities
- Integrate Defender alerts with SIEM/SOAR

### Security Monitoring
- Centralize logs in Log Analytics workspace
- Configure appropriate log retention periods
- Create custom queries for security investigations
- Set up alerts for suspicious activities
- Use workbooks for security dashboard visualization

### Incident Response
- Create playbooks for common security scenarios
- Automate response to known threat patterns
- Document incident response procedures
- Conduct regular security drills
- Use Sentinel incidents for case management

## Common Scenarios

### Policy Enforcement
- Enforce resource tagging for cost allocation
- Require encryption for storage accounts
- Block public network access to PaaS services
- Enforce allowed VM SKUs for cost control
- Require diagnostic settings on resources

### Threat Management
- SQL injection attack detection and response
- Brute force attack mitigation
- Malware detection on VMs
- Suspicious PowerShell execution investigation
- Data exfiltration attempt detection

### Security Automation
- Automated vulnerability remediation
- Security alert enrichment with threat intelligence
- Automated incident creation and assignment
- Compliance reporting automation
- Security baseline enforcement

### Monitoring and Investigation
- User behavior analytics for insider threats
- Cross-resource security investigation
- Threat hunting with KQL queries
- Security posture assessment
- Compliance dashboard creation

## Study Tips

- Practice writing Azure Policy definitions in JSON
- Learn Kusto Query Language (KQL) for Log Analytics
- Hands-on experience with Microsoft Sentinel deployment
- Study Defender for Cloud security controls and recommendations
- Practice creating Logic Apps for security automation
- Understand the MITRE ATT&CK framework
- Learn incident investigation workflow in Sentinel
- Practice configuring data connectors in Sentinel
- Study workbook creation for security visualization
- Understand security alert severity levels and response priorities
