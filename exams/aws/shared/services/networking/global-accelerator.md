# AWS Global Accelerator

## Service Overview and Purpose

AWS Global Accelerator is a networking service that improves the performance of your users' traffic by up to 60% using Amazon Web Services' global network infrastructure. When the internet is congested, Global Accelerator's automatic routing optimizations will help keep your packet loss, jitter, and latency consistently low.

### Key Characteristics
- **Global Network**: Leverages AWS global infrastructure
- **Anycast IP**: Static IP addresses for global access
- **Health Monitoring**: Automatic health checks and failover
- **Traffic Optimization**: Intelligent routing to optimal endpoints
- **DDoS Protection**: Built-in DDoS protection with AWS Shield

## Key Features and Capabilities

### Core Components
- **Accelerator**: The main Global Accelerator resource
- **Listeners**: Define ports and protocols for traffic routing
- **Endpoint Groups**: Group endpoints by geographic region
- **Endpoints**: Application Load Balancers, Network Load Balancers, EC2 instances, or Elastic IPs

### Traffic Control
- **Traffic Dials**: Control percentage of traffic to endpoint groups
- **Weights**: Distribute traffic across endpoints within a group
- **Client Affinity**: Route requests from same client to same endpoint
- **Health Checks**: Monitor endpoint health and route accordingly

## Configuration Examples

### Basic Global Accelerator Setup
```bash
# Create Global Accelerator
aws globalaccelerator create-accelerator \
  --name "Production-Global-Accelerator" \
  --ip-address-type IPV4 \
  --enabled \
  --tags Key=Environment,Value=Production Key=Application,Value=WebApp

# Create listener
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::account:accelerator/12345678-1234-1234-1234-123456789012 \
  --listener-configuration '{
    "Protocol": "TCP",
    "PortRanges": [{"FromPort": 80, "ToPort": 80}]
  }'

# Create endpoint group
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::account:listener/12345678-1234-1234-1234-123456789012 \
  --endpoint-group-region us-west-2 \
  --endpoint-configurations '{
    "EndpointId": "arn:aws:elasticloadbalancing:us-west-2:account:loadbalancer/app/my-alb/1234567890123456",
    "Weight": 100,
    "ClientIPPreservationEnabled": true
  }' \
  --traffic-dial-percentage 100 \
  --health-check-interval-seconds 30 \
  --health-check-protocol HTTP \
  --health-check-path "/health" \
  --threshold-count 3
```

### Multi-Region Setup
```bash
# Create endpoint groups in multiple regions
# US West endpoint group
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::account:listener/12345678-1234-1234-1234-123456789012 \
  --endpoint-group-region us-west-2 \
  --endpoint-configurations '{
    "EndpointId": "arn:aws:elasticloadbalancing:us-west-2:account:loadbalancer/app/west-alb/1234567890123456",
    "Weight": 100
  }' \
  --traffic-dial-percentage 100

# US East endpoint group
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::account:listener/12345678-1234-1234-1234-123456789012 \
  --endpoint-group-region us-east-1 \
  --endpoint-configurations '{
    "EndpointId": "arn:aws:elasticloadbalancing:us-east-1:account:loadbalancer/app/east-alb/1234567890123456",
    "Weight": 100
  }' \
  --traffic-dial-percentage 100

# Europe endpoint group
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::account:listener/12345678-1234-1234-1234-123456789012 \
  --endpoint-group-region eu-west-1 \
  --endpoint-configurations '{
    "EndpointId": "arn:aws:elasticloadbalancing:eu-west-1:account:loadbalancer/app/europe-alb/1234567890123456",
    "Weight": 100
  }' \
  --traffic-dial-percentage 100
```

### Advanced Configuration
```bash
# Create accelerator with multiple listeners
# HTTP listener
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::account:accelerator/12345678-1234-1234-1234-123456789012 \
  --listener-configuration '{
    "Protocol": "TCP",
    "PortRanges": [{"FromPort": 80, "ToPort": 80}],
    "ClientAffinity": "SOURCE_IP"
  }'

# HTTPS listener
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::account:accelerator/12345678-1234-1234-1234-123456789012 \
  --listener-configuration '{
    "Protocol": "TCP",
    "PortRanges": [{"FromPort": 443, "ToPort": 443}],
    "ClientAffinity": "SOURCE_IP"
  }'

# UDP listener for gaming
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::account:accelerator/12345678-1234-1234-1234-123456789012 \
  --listener-configuration '{
    "Protocol": "UDP",
    "PortRanges": [{"FromPort": 3000, "ToPort": 3010}],
    "ClientAffinity": "SOURCE_IP"
  }'
```

## Use Cases and Scenarios

### Primary Use Cases
1. **Global Applications**: Improve performance for worldwide users
2. **Gaming**: Reduce latency for real-time gaming applications
3. **IoT**: Optimize connectivity for IoT devices
4. **Media Streaming**: Improve streaming quality and reduce buffering
5. **VoIP**: Enhance voice and video communication quality

### Architecture Patterns

#### Global Web Application
```bash
# Multi-region web application with Global Accelerator
# Frontend: Global Accelerator -> ALB -> EC2/Fargate
# Backend: Regional services in each region

# Create accelerator for web traffic
aws globalaccelerator create-accelerator \
  --name "Global-Web-App" \
  --ip-address-type IPV4 \
  --enabled

# Add ALBs from multiple regions as endpoints
# Traffic automatically routed to closest healthy region
```

#### Blue-Green Deployment
```bash
# Use traffic dials for blue-green deployments
# Green environment (new version)
aws globalaccelerator update-endpoint-group \
  --endpoint-group-arn arn:aws:globalaccelerator::account:endpoint-group/12345678-1234-1234-1234-123456789012 \
  --traffic-dial-percentage 10  # Start with 10% traffic

# Gradually increase traffic to green environment
aws globalaccelerator update-endpoint-group \
  --endpoint-group-arn arn:aws:globalaccelerator::account:endpoint-group/12345678-1234-1234-1234-123456789012 \
  --traffic-dial-percentage 50  # 50% traffic

# Full cutover to green
aws globalaccelerator update-endpoint-group \
  --endpoint-group-arn arn:aws:globalaccelerator::account:endpoint-group/12345678-1234-1234-1234-123456789012 \
  --traffic-dial-percentage 100  # 100% traffic
```

#### Disaster Recovery
```bash
# Primary region with 100% traffic
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::account:listener/12345678-1234-1234-1234-123456789012 \
  --endpoint-group-region us-west-2 \
  --traffic-dial-percentage 100

# DR region with 0% traffic (standby)
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::account:listener/12345678-1234-1234-1234-123456789012 \
  --endpoint-group-region us-east-1 \
  --traffic-dial-percentage 0

# In case of disaster, shift traffic to DR region
aws globalaccelerator update-endpoint-group \
  --endpoint-group-arn arn:aws:globalaccelerator::account:endpoint-group/dr-12345678 \
  --traffic-dial-percentage 100
```

## Client IP Preservation

### Client IP Configuration
```bash
# Enable client IP preservation (ALB/NLB endpoints)
aws globalaccelerator update-endpoint-group \
  --endpoint-group-arn arn:aws:globalaccelerator::account:endpoint-group/12345678-1234-1234-1234-123456789012 \
  --endpoint-configurations '{
    "EndpointId": "arn:aws:elasticloadbalancing:us-west-2:account:loadbalancer/app/my-alb/1234567890123456",
    "Weight": 100,
    "ClientIPPreservationEnabled": true
  }'

# For EC2 endpoints, client IP is always preserved
# Original client IP available in X-Forwarded-For header for ALB
```

### Application Integration
```python
# Python Flask application to handle client IP
from flask import Flask, request
import json

app = Flask(__name__)

@app.route('/api/user-info')
def get_user_info():
    # Get client IP from various sources
    client_ip = request.headers.get('X-Forwarded-For', request.remote_addr)

    # Global Accelerator adds custom headers
    ga_edge_location = request.headers.get('X-Amzn-Ga-Edge-Location')
    ga_request_id = request.headers.get('X-Amzn-Ga-Request-Id')

    return json.dumps({
        'client_ip': client_ip,
        'edge_location': ga_edge_location,
        'request_id': ga_request_id,
        'user_agent': request.headers.get('User-Agent')
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

## Monitoring and Troubleshooting

### CloudWatch Metrics
```bash
# Monitor Global Accelerator performance
aws cloudwatch get-metric-statistics \
  --namespace AWS/GlobalAccelerator \
  --metric-name NewFlowCount \
  --dimensions Name=Accelerator,Value=12345678-1234-1234-1234-123456789012 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Sum

# Monitor processed bytes
aws cloudwatch get-metric-statistics \
  --namespace AWS/GlobalAccelerator \
  --metric-name ProcessedBytesIn \
  --dimensions Name=Accelerator,Value=12345678-1234-1234-1234-123456789012 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Sum

# Monitor endpoint health
aws cloudwatch get-metric-statistics \
  --namespace AWS/GlobalAccelerator \
  --metric-name HealthyEndpointCount \
  --dimensions Name=EndpointGroup,Value=12345678-1234-1234-1234-123456789012 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Average
```

### Flow Logs
```bash
# Enable Flow Logs for Global Accelerator
aws globalaccelerator update-accelerator \
  --accelerator-arn arn:aws:globalaccelerator::account:accelerator/12345678-1234-1234-1234-123456789012 \
  --flow-logs-enabled \
  --flow-logs-s3-bucket my-ga-flow-logs \
  --flow-logs-s3-prefix production/

# Analyze flow logs with Athena
CREATE EXTERNAL TABLE ga_flow_logs (
  version string,
  account_id string,
  accelerator_id string,
  client_ip string,
  client_port int,
  accelerator_ip string,
  accelerator_port int,
  endpoint_ip string,
  endpoint_port int,
  protocol string,
  ip_address_type string,
  packets bigint,
  bytes bigint,
  start_time bigint,
  end_time bigint,
  action string
)
PARTITIONED BY(year string, month string, day string)
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://my-ga-flow-logs/production/';
```

### Health Check Monitoring
```bash
# Check endpoint health status
aws globalaccelerator describe-endpoint-group \
  --endpoint-group-arn arn:aws:globalaccelerator::account:endpoint-group/12345678-1234-1234-1234-123456789012 \
  --query 'EndpointGroup.EndpointDescriptions[*].[EndpointId,HealthState,HealthReason]'

# Monitor health check failures
aws cloudwatch put-metric-alarm \
  --alarm-name "GA-Unhealthy-Endpoints" \
  --alarm-description "Global Accelerator unhealthy endpoints" \
  --metric-name HealthyEndpointCount \
  --namespace AWS/GlobalAccelerator \
  --statistic Average \
  --period 300 \
  --threshold 1 \
  --comparison-operator LessThanThreshold \
  --dimensions Name=EndpointGroup,Value=12345678-1234-1234-1234-123456789012
```

## Security Considerations

### DDoS Protection
- **AWS Shield Standard**: Automatic protection included
- **AWS Shield Advanced**: Enhanced protection (optional)
- **Rate Limiting**: Implement at application level
- **WAF Integration**: Use with ALB endpoints

### Access Control
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "globalaccelerator:Describe*",
      "globalaccelerator:List*"
    ],
    "Resource": "*"
  }, {
    "Effect": "Allow",
    "Action": [
      "globalaccelerator:UpdateAccelerator",
      "globalaccelerator:UpdateEndpointGroup"
    ],
    "Resource": "arn:aws:globalaccelerator::account:accelerator/12345678-*"
  }]
}
```

### Network Security
```bash
# Security group rules for ALB endpoints
aws ec2 authorize-security-group-ingress \
  --group-id sg-alb123 \
  --protocol tcp \
  --port 80 \
  --source-group sg-ga-prefix-list

# Use Global Accelerator IP ranges for security groups
# Download AWS IP ranges and filter for Global Accelerator
curl -s https://ip-ranges.amazonaws.com/ip-ranges.json | \
  jq -r '.prefixes[] | select(.service=="GLOBALACCELERATOR") | .ip_prefix'
```

## Cost Optimization

### Pricing Components
- **Fixed Fee**: $0.025 per hour per accelerator
- **Data Transfer Premium**: $0.015 per GB for first 10TB/month
- **Dominion-Specific**: Additional costs for some regions

### Cost Optimization Strategies
```bash
# Monitor data transfer costs
aws cloudwatch get-metric-statistics \
  --namespace AWS/GlobalAccelerator \
  --metric-name ProcessedBytesOut \
  --dimensions Name=Accelerator,Value=12345678-1234-1234-1234-123456789012 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-31T23:59:59Z \
  --period 86400 \
  --statistics Sum

# Optimize by:
# 1. Using CloudFront for static content
# 2. Implementing efficient caching
# 3. Compressing data at application level
# 4. Regional content optimization
```

## Exam-Specific Tips

### Key Exam Topics
1. **Global Performance**: How Global Accelerator improves performance
2. **Anycast IPs**: Static IP addresses for global access
3. **Traffic Control**: Traffic dials and endpoint weights
4. **Health Checks**: Automatic failover and health monitoring
5. **Client IP Preservation**: When and how it works

### Common Exam Scenarios
1. **Global Application**: Improve performance for worldwide users
2. **Gaming Application**: Reduce latency for real-time applications
3. **Disaster Recovery**: Use traffic dials for failover
4. **Blue-Green Deployment**: Gradual traffic shifting
5. **Multi-Region Setup**: Distribute traffic across regions

### Exam Tips
- **Know the difference** between Global Accelerator and CloudFront
- **Understand anycast IPs** and their benefits
- **Remember traffic dial** functionality for deployments
- **Know client IP preservation** capabilities and limitations
- **Understand health check** behavior and failover

### Global Accelerator vs CloudFront
| Feature | Global Accelerator | CloudFront |
|---------|-------------------|------------|
| Use Case | Dynamic content, TCP/UDP | Static content, HTTP/HTTPS |
| Protocol | TCP, UDP | HTTP/HTTPS |
| Caching | No caching | Edge caching |
| IP Addresses | Static anycast IPs | Dynamic edge IPs |
| Client IP | Preserved | Via headers |