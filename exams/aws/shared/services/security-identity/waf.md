# AWS WAF (Web Application Firewall)

## Service Overview and Purpose

AWS WAF is a web application firewall that helps protect web applications and APIs from common web exploits and attacks that could affect application availability, compromise security, or consume excessive resources. WAF gives you control over how traffic reaches your applications by enabling you to create security rules that control bot traffic and block common attack vectors.

**Core Purpose:**
- Protect web applications from common web exploits
- Filter malicious traffic before it reaches applications
- Prevent DDoS attacks and bot traffic
- Control access based on geographic location
- Monitor and analyze web traffic patterns

## Key Features and Capabilities

### Core WAF Components

1. **Web ACLs (Access Control Lists)**
   - Primary WAF resource that contains rules
   - Associated with AWS resources (CloudFront, ALB, API Gateway)
   - Define default action (allow or block)
   - Process rules in priority order

2. **Rules and Rule Groups**
   - Individual conditions that inspect web requests
   - AWS Managed Rules (pre-configured rule sets)
   - Custom rules for specific application needs
   - Rate-based rules for DDoS protection
   - Geo-match rules for geographic filtering

3. **Conditions and Statements**
   - String match conditions
   - IP address conditions
   - Size constraint conditions
   - SQL injection detection
   - Cross-site scripting (XSS) detection

### Advanced Features

- **Bot Control**: Detect and manage bot traffic
- **Fraud Control**: Account takeover and fraud prevention
- **Rate Limiting**: Request rate controls
- **Geo Blocking**: Country-based access control
- **Custom Responses**: Custom error pages and responses
- **Logging**: Detailed request logging to CloudWatch, S3, or Kinesis
- **Real-time Metrics**: CloudWatch integration for monitoring
- **Captcha Integration**: Challenge suspicious requests

## Use Cases and Scenarios

### Primary Use Cases

1. **DDoS Protection**: Rate limiting and traffic filtering
2. **SQL Injection Prevention**: Detect and block SQL injection attempts
3. **XSS Protection**: Prevent cross-site scripting attacks
4. **Bot Management**: Control automated traffic and crawlers
5. **Geographic Restrictions**: Block traffic from specific countries
6. **API Protection**: Secure REST and GraphQL APIs
7. **Compliance**: Meet regulatory security requirements
8. **Content Protection**: Prevent unauthorized access to content
9. **Application Security**: Layer 7 security for web applications
10. **Traffic Analysis**: Monitor and analyze web request patterns

### Scenario Examples

- **E-commerce Platform**: Protect against scrapers and fraudulent transactions
- **API Gateway**: Rate limiting and authentication validation
- **Content Delivery**: Geographic content restrictions and bot blocking
- **Financial Services**: Enhanced security for online banking applications
- **Gaming Platform**: DDoS protection and anti-cheat mechanisms

## Pricing Models and Cost Optimization

### Pricing Structure

1. **Web ACLs**
   - $1.00 per Web ACL per month
   - $0.60 per million requests processed
   - Additional charges for specific rule types

2. **Rules and Conditions**
   - $1.00 per rule per month
   - Bot Control: $1.00 per million requests
   - Fraud Control: $1.00 per million requests
   - Captcha: $0.40 per 1,000 challenge attempts

3. **Logging and Storage**
   - CloudWatch Logs: Standard CloudWatch pricing
   - S3 Storage: Standard S3 pricing
   - Kinesis Data Firehose: Standard Kinesis pricing

4. **AWS Managed Rules**
   - $1.00 per rule group per month
   - Additional request charges may apply

### Cost Optimization Strategies

1. **Rule Optimization**: Use efficient rules and conditions
2. **Managed Rules**: Leverage AWS managed rules when appropriate
3. **Geographic Filtering**: Block unnecessary geographic regions early
4. **Rate Limiting**: Implement cost-effective rate controls
5. **Logging Strategy**: Optimize logging to essential data only
6. **Monitoring**: Regular review of traffic patterns and costs

## Configuration Details and Best Practices

### Web ACL Configuration

```json
{
  "Name": "MyWebACL",
  "Scope": "CLOUDFRONT",
  "DefaultAction": {
    "Allow": {}
  },
  "Rules": [
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 1,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      },
      "Action": {
        "Block": {}
      }
    },
    {
      "Name": "RateLimitRule",
      "Priority": 2,
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP"
        }
      },
      "Action": {
        "Block": {}
      }
    }
  ]
}
```

### Best Practices

1. **Layered Security**
   - Use WAF as part of defense-in-depth strategy
   - Combine with CloudFront security features
   - Integrate with AWS Shield for DDoS protection
   - Consider Network Load Balancer for additional protection

2. **Rule Management**
   - Start with AWS Managed Rules
   - Test rules in count mode before blocking
   - Regularly review and update custom rules
   - Use rule groups for organization
   - Monitor false positives and adjust accordingly

3. **Performance Optimization**
   - Order rules by priority and likelihood
   - Use geo-match rules for early filtering
   - Optimize string matching patterns
   - Consider rule evaluation costs
   - Monitor request latency impact

4. **Monitoring and Alerting**
   - Enable comprehensive logging
   - Set up CloudWatch alarms for security events
   - Regular review of blocked requests
   - Monitor for new attack patterns
   - Automated response to security incidents

## Integration with Other AWS Services

### Core Integrations

1. **CloudFront**: Protect content delivery networks
2. **Application Load Balancer**: Secure application traffic
3. **API Gateway**: Protect REST and WebSocket APIs
4. **AWS Shield**: Enhanced DDoS protection
5. **CloudWatch**: Monitoring and alerting
6. **CloudTrail**: API call auditing
7. **S3**: Store WAF logs and configurations
8. **Lambda**: Custom processing and responses
9. **Systems Manager**: Centralized configuration management
10. **Security Hub**: Centralized security findings

### Advanced Integrations

1. **GuardDuty**: Threat intelligence integration
2. **Inspector**: Application vulnerability assessment
3. **Config**: Configuration compliance monitoring
4. **Organizations**: Multi-account WAF management
5. **Firewall Manager**: Centralized firewall policies
6. **Route 53**: DNS-based protection
7. **Cognito**: User authentication integration

## Security Considerations

### Attack Protection

- **OWASP Top 10**: Protection against common web vulnerabilities
- **DDoS Mitigation**: Rate limiting and traffic shaping
- **Bot Management**: Distinguish between good and bad bots
- **Data Exfiltration**: Prevent unauthorized data access
- **API Security**: Protect against API-specific attacks

### Compliance and Governance

- **PCI DSS**: Payment card industry compliance
- **SOC**: Service organization control compliance
- **HIPAA**: Healthcare data protection
- **GDPR**: European privacy regulation compliance
- **Audit Logging**: Comprehensive security event logging

### Data Protection

- **Request Logging**: Secure storage of security logs
- **PII Handling**: Protect personally identifiable information
- **Encryption**: Secure transmission and storage
- **Access Control**: Restrict WAF configuration access

## Monitoring and Troubleshooting

### CloudWatch Metrics

- **AllowedRequests**: Number of allowed web requests
- **BlockedRequests**: Number of blocked web requests
- **CountedRequests**: Number of requests counted by rules
- **PassedRequests**: Number of requests that passed through
- **SampledRequests**: Sample of requests for analysis

### Common Issues and Solutions

1. **False Positives**
   - Review blocked legitimate requests
   - Adjust rule sensitivity and thresholds
   - Use count mode for testing
   - Implement whitelisting for known good sources

2. **Performance Impact**
   - Monitor request latency increase
   - Optimize rule ordering and complexity
   - Consider rule caching strategies
   - Review geographic distribution of rules

3. **Configuration Issues**
   - Verify Web ACL associations
   - Check rule priority ordering
   - Validate condition syntax
   - Test with sample requests

4. **Bypass Attempts**
   - Monitor for attack pattern evolution
   - Update rules for new threat vectors
   - Review logs for unusual patterns
   - Implement additional security layers

### Monitoring Tools

1. **WAF Dashboard**: Real-time security metrics
2. **CloudWatch Insights**: Log analysis and queries
3. **AWS Security Hub**: Centralized security findings
4. **Third-party SIEM**: Integration with security tools
5. **Custom Dashboards**: Application-specific monitoring

## Exam-Specific Tips and Common Scenarios

### Solutions Architect Associate (SAA-C03)

- **Web Application Security**: Implementing WAF with CloudFront and ALB
- **DDoS Protection**: Combining WAF with AWS Shield
- **Cost Optimization**: Selecting appropriate WAF features
- **Geographic Restrictions**: Content delivery compliance

### Solutions Architect Professional (SAP-C02)

- **Enterprise Security**: Multi-account WAF strategies
- **Advanced Threat Protection**: Custom rules and integrations
- **Compliance Architecture**: Meeting regulatory requirements
- **Performance Optimization**: Large-scale WAF deployments

### Security Specialty (SCS-C02)

- **Advanced WAF Features**: Bot control and fraud detection
- **Threat Intelligence**: Integrating external threat feeds
- **Incident Response**: Automated security responses
- **Compliance Monitoring**: Continuous security assessment

### Developer Associate (DVA-C02)

- **API Protection**: Securing API Gateway with WAF
- **Application Integration**: WAF in application architecture
- **Custom Rules**: Developing application-specific protections
- **Monitoring**: Implementing security monitoring

### Common Exam Scenarios

1. **Scenario**: Web application experiencing DDoS attacks
   **Solution**: Implement WAF with rate-based rules and AWS Shield

2. **Scenario**: Need to block traffic from specific countries
   **Solution**: Configure geo-match conditions in WAF rules

3. **Scenario**: API receiving malicious SQL injection attempts
   **Solution**: Use AWS Managed Rules for SQL injection protection

4. **Scenario**: High false positive rate from WAF rules
   **Solution**: Use count mode for testing and whitelist legitimate sources

## Hands-on Examples and CLI Commands

### Web ACL Management

```bash
# Create Web ACL
aws wafv2 create-web-acl \
  --scope CLOUDFRONT \
  --default-action Allow={} \
  --name MyWebACL \
  --description "My Web Application Firewall" \
  --rules '[
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 1,
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      },
      "Action": {"Block": {}},
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "AWSManagedRulesCommonRuleSetMetric"
      }
    }
  ]'

# List Web ACLs
aws wafv2 list-web-acls --scope CLOUDFRONT

# Get Web ACL
aws wafv2 get-web-acl \
  --scope CLOUDFRONT \
  --id 12345678-1234-1234-1234-123456789012 \
  --name MyWebACL

# Update Web ACL
aws wafv2 update-web-acl \
  --scope CLOUDFRONT \
  --id 12345678-1234-1234-1234-123456789012 \
  --name MyWebACL \
  --default-action Block={} \
  --lock-token 12345678-1234-1234-1234-123456789012

# Delete Web ACL
aws wafv2 delete-web-acl \
  --scope CLOUDFRONT \
  --id 12345678-1234-1234-1234-123456789012 \
  --lock-token 12345678-1234-1234-1234-123456789012
```

### Rule Management

```bash
# Create IP Set
aws wafv2 create-ip-set \
  --scope CLOUDFRONT \
  --name BlockedIPs \
  --description "Blocked IP addresses" \
  --ip-address-version IPV4 \
  --addresses 192.0.2.0/24,203.0.113.0/24

# Create Regex Pattern Set
aws wafv2 create-regex-pattern-set \
  --scope CLOUDFRONT \
  --name MaliciousPatterns \
  --description "Malicious request patterns" \
  --regular-expression-list '[
    {"RegexString": "(?i)(union|select|insert|delete|drop|create|alter|exec)"}
  ]'

# Create Rate-based Rule
aws wafv2 create-rule-group \
  --scope CLOUDFRONT \
  --name RateLimitRuleGroup \
  --capacity 100 \
  --description "Rate limiting rules" \
  --rules '[
    {
      "Name": "RateLimitRule",
      "Priority": 1,
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP"
        }
      },
      "Action": {"Block": {}},
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "RateLimitRule"
      }
    }
  ]'

# List managed rule groups
aws wafv2 list-available-managed-rule-groups --scope CLOUDFRONT

# Describe managed rule group
aws wafv2 describe-managed-rule-group \
  --vendor-name AWS \
  --name AWSManagedRulesCommonRuleSet \
  --scope CLOUDFRONT
```

### Association Management

```bash
# Associate Web ACL with CloudFront distribution
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:global/webacl/MyWebACL/12345678-1234-1234-1234-123456789012 \
  --resource-arn arn:aws:cloudfront::123456789012:distribution/E1234567890123

# Associate Web ACL with Application Load Balancer
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:regional/webacl/MyWebACL/12345678-1234-1234-1234-123456789012 \
  --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890123456

# List resources associated with Web ACL
aws wafv2 list-resources-for-web-acl \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:global/webacl/MyWebACL/12345678-1234-1234-1234-123456789012

# Disassociate Web ACL from resource
aws wafv2 disassociate-web-acl \
  --resource-arn arn:aws:cloudfront::123456789012:distribution/E1234567890123
```

### Monitoring and Logging

```bash
# Put logging configuration
aws wafv2 put-logging-configuration \
  --logging-configuration '{
    "ResourceArn": "arn:aws:wafv2:us-east-1:123456789012:global/webacl/MyWebACL/12345678-1234-1234-1234-123456789012",
    "LogDestinationConfigs": [
      "arn:aws:logs:us-east-1:123456789012:log-group:aws-waf-logs-MyWebACL"
    ],
    "RedactedFields": [
      {"SingleHeader": {"Name": "authorization"}}
    ]
  }'

# Get logging configuration
aws wafv2 get-logging-configuration \
  --resource-arn arn:aws:wafv2:us-east-1:123456789012:global/webacl/MyWebACL/12345678-1234-1234-1234-123456789012

# Get sampled requests
aws wafv2 get-sampled-requests \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:global/webacl/MyWebACL/12345678-1234-1234-1234-123456789012 \
  --rule-metric-name AWSManagedRulesCommonRuleSetMetric \
  --scope CLOUDFRONT \
  --time-window StartTime=2023-01-01T00:00:00Z,EndTime=2023-01-01T01:00:00Z \
  --max-items 100

# Check capacity usage
aws wafv2 check-capacity \
  --scope CLOUDFRONT \
  --rules '[
    {
      "Name": "TestRule",
      "Priority": 1,
      "Statement": {
        "ByteMatchStatement": {
          "SearchString": "attack",
          "FieldToMatch": {"Body": {}},
          "TextTransformations": [{"Priority": 0, "Type": "LOWERCASE"}],
          "PositionalConstraint": "CONTAINS"
        }
      },
      "Action": {"Block": {}},
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "TestRule"
      }
    }
  ]'
```

### Testing and Validation

```bash
# Test SQL injection rule
curl -X POST https://example.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"username": "admin'\'' OR 1=1--", "password": "password"}'

# Test rate limiting
for i in {1..3000}; do
  curl -s https://example.com/ > /dev/null &
done

# Test geographic blocking (requires VPN or proxy)
curl -H "CF-IPCountry: CN" https://example.com/

# Monitor CloudWatch metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/WAFV2 \
  --metric-name BlockedRequests \
  --dimensions Name=WebACL,Value=MyWebACL Name=Region,Value=CloudFront \
  --statistics Sum \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T01:00:00Z \
  --period 3600
```

### Custom Rule Examples

```bash
# Create custom rule for SQL injection protection
aws wafv2 update-web-acl \
  --scope CLOUDFRONT \
  --id 12345678-1234-1234-1234-123456789012 \
  --name MyWebACL \
  --default-action Allow={} \
  --rules '[
    {
      "Name": "CustomSQLiRule",
      "Priority": 1,
      "Statement": {
        "ByteMatchStatement": {
          "SearchString": "union select",
          "FieldToMatch": {"AllQueryArguments": {}},
          "TextTransformations": [
            {"Priority": 0, "Type": "URL_DECODE"},
            {"Priority": 1, "Type": "LOWERCASE"}
          ],
          "PositionalConstraint": "CONTAINS"
        }
      },
      "Action": {"Block": {}},
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CustomSQLiRule"
      }
    }
  ]' \
  --lock-token 12345678-1234-1234-1234-123456789012
```

This comprehensive AWS WAF documentation covers all essential security features and implementation details needed for AWS certification preparation and real-world application protection.