# Amazon CloudFront

## Service Overview and Purpose

Amazon CloudFront is a content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds. CloudFront integrates with other AWS services to give developers and businesses an easy way to accelerate content to end users.

### Key Characteristics
- **Global Edge Network**: 400+ edge locations worldwide
- **Low Latency**: Content cached closer to users
- **High Performance**: Optimized for speed and availability
- **Security**: DDoS protection, SSL/TLS, and access controls
- **Cost Effective**: Pay for what you use with volume discounts

## Key Features and Capabilities

### Distribution Types
- **Web Distribution**: For websites, APIs, and general content
- **RTMP Distribution**: For media streaming (deprecated)

### Cache Behaviors
- **Path Patterns**: Route requests based on URL patterns
- **TTL Settings**: Control cache duration
- **Query String Forwarding**: Cache based on query parameters
- **Header Forwarding**: Pass specific headers to origin

### Origins
- **S3 Buckets**: Static content delivery
- **Custom Origins**: EC2, Load Balancers, on-premises servers
- **Origin Groups**: Failover support with multiple origins

### Security Features
- **AWS WAF Integration**: Web application firewall
- **AWS Shield**: DDoS protection
- **SSL/TLS**: HTTPS encryption
- **Signed URLs/Cookies**: Restricted content access
- **Origin Access Control**: Secure S3 access

## Configuration Examples

### Basic Distribution Setup
```bash
# Create CloudFront distribution with S3 origin
aws cloudfront create-distribution \
  --distribution-config '{
    "CallerReference": "my-distribution-'$(date +%s)'",
    "Comment": "Production website distribution",
    "DefaultRootObject": "index.html",
    "Origins": {
      "Quantity": 1,
      "Items": [{
        "Id": "S3-my-website-bucket",
        "DomainName": "my-website-bucket.s3.amazonaws.com",
        "S3OriginConfig": {
          "OriginAccessIdentity": "origin-access-identity/cloudfront/ABCDEFG1234567"
        }
      }]
    },
    "DefaultCacheBehavior": {
      "TargetOriginId": "S3-my-website-bucket",
      "ViewerProtocolPolicy": "redirect-to-https",
      "MinTTL": 0,
      "ForwardedValues": {
        "QueryString": false,
        "Cookies": {"Forward": "none"}
      }
    },
    "Enabled": true,
    "PriceClass": "PriceClass_100"
  }'
```

### Advanced Cache Behaviors
```json
{
  "CacheBehaviors": {
    "Quantity": 2,
    "Items": [
      {
        "PathPattern": "/api/*",
        "TargetOriginId": "API-Origin",
        "ViewerProtocolPolicy": "https-only",
        "MinTTL": 0,
        "DefaultTTL": 0,
        "MaxTTL": 0,
        "ForwardedValues": {
          "QueryString": true,
          "Headers": {
            "Quantity": 3,
            "Items": ["Authorization", "Content-Type", "User-Agent"]
          }
        }
      },
      {
        "PathPattern": "/images/*",
        "TargetOriginId": "S3-Images",
        "ViewerProtocolPolicy": "redirect-to-https",
        "MinTTL": 86400,
        "DefaultTTL": 86400,
        "MaxTTL": 31536000,
        "Compress": true
      }
    ]
  }
}
```

## Use Cases and Scenarios

### Primary Use Cases
1. **Website Acceleration**: Static and dynamic content delivery
2. **API Acceleration**: Reduce API latency globally
3. **Video Streaming**: On-demand and live video delivery
4. **Software Distribution**: Download acceleration
5. **Security**: DDoS protection and WAF integration

### Architecture Patterns

#### Multi-Origin Setup
```python
# Lambda@Edge function for dynamic origin selection
def lambda_handler(event, context):
    request = event['Records'][0]['cf']['request']

    # Route based on request path
    if request['uri'].startswith('/api/'):
        request['origin'] = {
            'custom': {
                'domainName': 'api.example.com',
                'port': 443,
                'protocol': 'https',
                'path': '/v1'
            }
        }
    elif request['uri'].startswith('/static/'):
        request['origin'] = {
            's3': {
                'domainName': 'static-assets.s3.amazonaws.com',
                'region': 'us-east-1',
                'authMethod': 'origin-access-identity',
                'oaiId': 'ABCDEFG1234567'
            }
        }

    return request
```

#### Content Security and Access Control
```bash
# Create signed URLs for private content
aws cloudfront sign \
  --url https://d123456789.cloudfront.net/private-video.mp4 \
  --private-key-file private_key.pem \
  --key-pair-id APKAEIBAERJR2EXAMPLE \
  --date-less-than 2023-12-31T23:59:59Z
```

## Performance Optimization

### Cache Optimization
```json
{
  "CacheBehavior": {
    "CachePolicyId": "custom-cache-policy",
    "OriginRequestPolicyId": "custom-origin-policy",
    "ResponseHeadersPolicyId": "custom-headers-policy",
    "Compress": true,
    "ViewerProtocolPolicy": "redirect-to-https"
  }
}
```

### Lambda@Edge Functions
```python
# Viewer request function for A/B testing
def lambda_handler(event, context):
    request = event['Records'][0]['cf']['request']

    # Implement A/B testing logic
    import random
    if random.random() < 0.5:
        request['uri'] = '/version-a' + request['uri']
    else:
        request['uri'] = '/version-b' + request['uri']

    return request

# Origin response function for custom headers
def lambda_handler(event, context):
    response = event['Records'][0]['cf']['response']

    # Add security headers
    response['headers']['strict-transport-security'] = [{
        'key': 'Strict-Transport-Security',
        'value': 'max-age=31536000; includeSubdomains; preload'
    }]

    response['headers']['x-content-type-options'] = [{
        'key': 'X-Content-Type-Options',
        'value': 'nosniff'
    }]

    return response
```

## Monitoring and Analytics

### CloudWatch Metrics
```bash
# Monitor CloudFront performance
aws cloudwatch get-metric-statistics \
  --namespace AWS/CloudFront \
  --metric-name Requests \
  --dimensions Name=DistributionId,Value=E1234567890123 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 3600 \
  --statistics Sum

# Cache hit ratio monitoring
aws cloudwatch get-metric-statistics \
  --namespace AWS/CloudFront \
  --metric-name CacheHitRate \
  --dimensions Name=DistributionId,Value=E1234567890123 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 3600 \
  --statistics Average
```

### Real-Time Logs
```bash
# Enable real-time logs
aws cloudfront create-realtime-log-config \
  --name "production-realtime-logs" \
  --end-points StreamType=Kinesis,KinesisStreamConfig='{
    "RoleArn": "arn:aws:iam::account:role/CloudFrontRealtimeLogRole",
    "StreamArn": "arn:aws:kinesis:us-east-1:account:stream/cloudfront-logs"
  }' \
  --fields timestamp,c-ip,sc-status,cs-method,cs-uri-stem,cs-bytes,time-taken
```

## Security Configuration

### SSL/TLS Setup
```bash
# Use ACM certificate for custom domain
aws cloudfront update-distribution \
  --id E1234567890123 \
  --distribution-config '{
    "ViewerCertificate": {
      "ACMCertificateArn": "arn:aws:acm:us-east-1:account:certificate/cert-id",
      "SSLSupportMethod": "sni-only",
      "MinimumProtocolVersion": "TLSv1.2_2021"
    },
    "Aliases": {
      "Quantity": 1,
      "Items": ["www.example.com"]
    }
  }'
```

### WAF Integration
```bash
# Associate WAF with CloudFront
aws cloudfront update-distribution \
  --id E1234567890123 \
  --distribution-config '{
    "WebACLId": "arn:aws:wafv2:us-east-1:account:global/webacl/name/id"
  }'
```

## Exam-Specific Tips

### Key Exam Topics
1. **Cache Behaviors**: Path patterns, TTL settings, header forwarding
2. **Origins**: S3 vs custom origins, origin groups, failover
3. **Security**: SSL/TLS, signed URLs, OAI, WAF integration
4. **Performance**: Edge locations, cache optimization, compression
5. **Lambda@Edge**: Use cases and event types

### Common Scenarios
1. **Global Web Application**: Use CloudFront to reduce latency
2. **API Acceleration**: Cache static responses, forward dynamic requests
3. **Media Streaming**: Deliver video content globally
4. **Security Enhancement**: DDoS protection and content filtering

### Exam Tips
- **Know cache behavior precedence** and path pattern matching
- **Understand origin types** and when to use each
- **Remember security features** like OAI and signed URLs
- **Know Lambda@Edge event types** and their use cases
- **Understand cost factors** and price classes