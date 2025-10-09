# Amazon Route 53

## Service Overview and Purpose

Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. Route 53 connects user requests to infrastructure running in AWS and can also route users to infrastructure outside of AWS.

### Key Characteristics
- **100% Uptime SLA**: Highly available DNS service
- **Global Anycast Network**: 200+ edge locations worldwide
- **DNS and Domain Registration**: Complete DNS solution
- **Health Checks**: Monitor endpoint health and route accordingly
- **Traffic Flow**: Visual traffic management

## Key Features and Capabilities

### DNS Record Types
- **A Record**: IPv4 address mapping
- **AAAA Record**: IPv6 address mapping
- **CNAME Record**: Canonical name mapping
- **MX Record**: Mail exchange routing
- **TXT Record**: Text information
- **SRV Record**: Service location
- **NS Record**: Name server delegation
- **PTR Record**: Reverse DNS lookup

### Routing Policies
1. **Simple Routing**: Single resource record
2. **Weighted Routing**: Distribute traffic by weight
3. **Latency-Based Routing**: Route to lowest latency region
4. **Failover Routing**: Active-passive failover
5. **Geolocation Routing**: Route based on user location
6. **Geoproximity Routing**: Route based on geographic proximity
7. **Multivalue Answer Routing**: Return multiple healthy endpoints

### Health Checks
- **HTTP/HTTPS Checks**: Monitor web endpoints
- **TCP Checks**: Monitor TCP services
- **Calculated Health Checks**: Combine multiple checks
- **CloudWatch Alarm Checks**: Monitor based on metrics

## Configuration Examples

### Basic DNS Setup
```bash
# Create hosted zone
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference $(date +%s) \
  --hosted-zone-config PrivateZone=false,Comment="Production domain"

# Create A record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "192.0.2.1"}]
      }
    }]
  }'

# Create CNAME record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "blog.example.com",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [{"Value": "www.example.com"}]
      }
    }]
  }'
```

### Alias Records for AWS Resources
```bash
# Alias record for Application Load Balancer
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.example.com",
        "Type": "A",
        "AliasTarget": {
          "DNSName": "my-load-balancer-1234567890.us-west-2.elb.amazonaws.com",
          "EvaluateTargetHealth": true,
          "HostedZoneId": "Z1D633PJN98FT9"
        }
      }
    }]
  }'

# Alias record for CloudFront distribution
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "cdn.example.com",
        "Type": "A",
        "AliasTarget": {
          "DNSName": "d123456789.cloudfront.net",
          "EvaluateTargetHealth": false,
          "HostedZoneId": "Z2FDTNDATAQYW2"
        }
      }
    }]
  }'
```

### Advanced Routing Policies

#### Weighted Routing
```bash
# Weighted routing for blue-green deployment
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Blue",
          "Weight": 90,
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.1"}]
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Green",
          "Weight": 10,
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.2"}]
        }
      }
    ]
  }'
```

#### Latency-Based Routing
```bash
# Latency-based routing for global application
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "global.example.com",
          "Type": "A",
          "SetIdentifier": "US-West",
          "Region": "us-west-2",
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.1"}]
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "global.example.com",
          "Type": "A",
          "SetIdentifier": "EU-West",
          "Region": "eu-west-1",
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.2"}]
        }
      }
    ]
  }'
```

#### Geolocation Routing
```bash
# Geolocation routing for compliance
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "US-Users",
          "GeoLocation": {"CountryCode": "US"},
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.1"}]
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "EU-Users",
          "GeoLocation": {"ContinentCode": "EU"},
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.2"}]
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Default",
          "GeoLocation": {"CountryCode": "*"},
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.3"}]
        }
      }
    ]
  }'
```

### Health Checks

```bash
# Create HTTP health check
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "Type": "HTTP",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "api.example.com",
    "Port": 80,
    "RequestInterval": 30,
    "FailureThreshold": 3
  }' \
  --tags ResourceType=healthcheck,Tags='[{Key=Name,Value=API-Health-Check}]'

# Create calculated health check
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "Type": "CALCULATED",
    "ChildHealthChecks": ["health-check-1", "health-check-2"],
    "ChildHealthCheckCount": 1,
    "Inverted": false
  }'

# Failover routing with health checks
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Primary",
          "Failover": "PRIMARY",
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.1"}],
          "HealthCheckId": "health-check-primary"
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "app.example.com",
          "Type": "A",
          "SetIdentifier": "Secondary",
          "Failover": "SECONDARY",
          "TTL": 60,
          "ResourceRecords": [{"Value": "192.0.2.2"}]
        }
      }
    ]
  }'
```

## Private Hosted Zones

```bash
# Create private hosted zone
aws route53 create-hosted-zone \
  --name internal.company.com \
  --caller-reference $(date +%s) \
  --vpc VPCRegion=us-west-2,VPCId=vpc-12345678 \
  --hosted-zone-config PrivateZone=true,Comment="Internal company DNS"

# Associate additional VPC
aws route53 associate-vpc-with-hosted-zone \
  --hosted-zone-id Z987654321 \
  --vpc VPCRegion=us-east-1,VPCId=vpc-87654321

# Create internal records
aws route53 change-resource-record-sets \
  --hosted-zone-id Z987654321 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "database.internal.company.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "10.0.1.100"}]
      }
    }]
  }'
```

## Integration with AWS Services

### Lambda Integration for Dynamic DNS
```python
import boto3
import json

def lambda_handler(event, context):
    route53 = boto3.client('route53')
    ec2 = boto3.client('ec2')

    # Get instance information from event
    instance_id = event['detail']['instance-id']
    state = event['detail']['state']

    if state == 'running':
        # Get instance details
        response = ec2.describe_instances(InstanceIds=[instance_id])
        instance = response['Reservations'][0]['Instances'][0]

        private_ip = instance.get('PrivateIpAddress')
        instance_name = next(
            (tag['Value'] for tag in instance.get('Tags', []) if tag['Key'] == 'Name'),
            instance_id
        )

        # Update DNS record
        route53.change_resource_record_sets(
            HostedZoneId='Z123456789',
            ChangeBatch={
                'Changes': [{
                    'Action': 'UPSERT',
                    'ResourceRecordSet': {
                        'Name': f'{instance_name}.internal.company.com',
                        'Type': 'A',
                        'TTL': 60,
                        'ResourceRecords': [{'Value': private_ip}]
                    }
                }]
            }
        )

    elif state == 'terminated':
        # Remove DNS record
        route53.change_resource_record_sets(
            HostedZoneId='Z123456789',
            ChangeBatch={
                'Changes': [{
                    'Action': 'DELETE',
                    'ResourceRecordSet': {
                        'Name': f'{instance_name}.internal.company.com',
                        'Type': 'A',
                        'TTL': 60,
                        'ResourceRecords': [{'Value': private_ip}]
                    }
                }]
            }
        )

    return {'statusCode': 200}
```

## Monitoring and Troubleshooting

### CloudWatch Metrics
```bash
# Monitor health check status
aws cloudwatch get-metric-statistics \
  --namespace AWS/Route53 \
  --metric-name HealthCheckStatus \
  --dimensions Name=HealthCheckId,Value=health-check-id \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Average

# Monitor query volume
aws cloudwatch get-metric-statistics \
  --namespace AWS/Route53 \
  --metric-name QueryCount \
  --dimensions Name=HostedZoneId,Value=Z123456789 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 3600 \
  --statistics Sum
```

### DNS Troubleshooting
```bash
# Test DNS resolution
dig @8.8.8.8 example.com A
nslookup example.com 8.8.8.8

# Check Route 53 resolver logs (if enabled)
aws logs filter-log-events \
  --log-group-name /aws/route53/resolver \
  --start-time 1640995200000 \
  --filter-pattern "{ $.query_name = \"example.com\" }"

# Validate health check
aws route53 get-health-check \
  --health-check-id health-check-id

aws route53 get-health-check-status \
  --health-check-id health-check-id
```

## Use Cases and Scenarios

### Primary Use Cases
1. **Domain Registration and Management**: Register and manage domains
2. **DNS Hosting**: Authoritative DNS for domains
3. **Traffic Management**: Intelligent routing based on various factors
4. **Health Monitoring**: Monitor endpoint health and route accordingly
5. **Hybrid DNS**: Bridge on-premises and cloud DNS

### Architecture Patterns

#### Multi-Region Active-Active
```bash
# Health checks for each region
aws route53 create-health-check \
  --caller-reference us-west-health-$(date +%s) \
  --health-check-config '{
    "Type": "HTTP",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "us-west.api.example.com",
    "Port": 443,
    "RequestInterval": 30
  }'

# Latency-based routing with health checks
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456789 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.example.com",
        "Type": "A",
        "SetIdentifier": "US-West-API",
        "Region": "us-west-2",
        "TTL": 60,
        "ResourceRecords": [{"Value": "192.0.2.1"}],
        "HealthCheckId": "us-west-health-check"
      }
    }]
  }'
```

#### Blue-Green Deployment
```python
# Gradual traffic shifting script
import boto3
import time

def shift_traffic(hosted_zone_id, record_name, blue_weight, green_weight):
    route53 = boto3.client('route53')

    route53.change_resource_record_sets(
        HostedZoneId=hosted_zone_id,
        ChangeBatch={
            'Changes': [
                {
                    'Action': 'UPSERT',
                    'ResourceRecordSet': {
                        'Name': record_name,
                        'Type': 'A',
                        'SetIdentifier': 'Blue',
                        'Weight': blue_weight,
                        'TTL': 60,
                        'ResourceRecords': [{'Value': '192.0.2.1'}]
                    }
                },
                {
                    'Action': 'UPSERT',
                    'ResourceRecordSet': {
                        'Name': record_name,
                        'Type': 'A',
                        'SetIdentifier': 'Green',
                        'Weight': green_weight,
                        'TTL': 60,
                        'ResourceRecords': [{'Value': '192.0.2.2'}]
                    }
                }
            ]
        }
    )

# Gradual shift from blue to green
weights = [(90, 10), (70, 30), (50, 50), (30, 70), (10, 90), (0, 100)]
for blue, green in weights:
    shift_traffic('Z123456789', 'app.example.com', blue, green)
    time.sleep(300)  # Wait 5 minutes between shifts
```

## Exam-Specific Tips

### Key Exam Topics
1. **Routing Policies**: Understand all routing policies and their use cases
2. **Health Checks**: Types of health checks and failover scenarios
3. **Alias Records**: When to use alias vs CNAME records
4. **Private Hosted Zones**: Internal DNS for VPC resources
5. **Integration**: How Route 53 integrates with other AWS services

### Common Exam Scenarios
1. **Global Application**: Use latency-based routing for performance
2. **Disaster Recovery**: Implement failover routing with health checks
3. **Blue-Green Deployment**: Use weighted routing for traffic shifting
4. **Compliance**: Use geolocation routing for data residency
5. **Microservices**: Use service discovery with private hosted zones

### Exam Tips
- **Know routing policy differences** and when to use each
- **Understand health check types** and their limitations
- **Remember alias record benefits** over CNAME records
- **Know private hosted zone** use cases and VPC association
- **Understand DNS propagation** and TTL implications