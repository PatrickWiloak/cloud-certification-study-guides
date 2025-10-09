# Load Balancing and Performance - GCP Professional Cloud Network Engineer

## Overview

Load balancing architectures, traffic management, CDN, network performance optimization, and global distribution strategies.

## Load Balancer Types

### Global Load Balancers
**HTTP(S) Load Balancer**:
- Layer 7, content-based routing
- Anycast IP (single IP globally)
- SSL/TLS termination
- Cloud CDN integration
- Cloud Armor support

**SSL Proxy**: Layer 4 SSL/TLS, global
**TCP Proxy**: Layer 4 TCP, global

### Regional Load Balancers
**Network Load Balancer**: Layer 4, pass-through, preserves client IP
**Internal Load Balancer**: Private, within VPC

## HTTP(S) Load Balancing Architecture

### Components
```
Global Forwarding Rule → Target HTTPS Proxy → URL Map → Backend Service → Instance Groups
```

**Implementation**:
```bash
# Health check
gcloud compute health-checks create http web-health \
  --port=80 \
  --request-path=/health \
  --check-interval=5s \
  --timeout=5s \
  --unhealthy-threshold=2 \
  --healthy-threshold=2

# Backend service
gcloud compute backend-services create web-backend \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=web-health \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --default-ttl=3600 \
  --global

# Add backends
gcloud compute backend-services add-backend web-backend \
  --instance-group=ig-us-central \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --global

gcloud compute backend-services add-backend web-backend \
  --instance-group=ig-europe-west \
  --instance-group-zone=europe-west1-b \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --global

# URL map
gcloud compute url-maps create web-map \
  --default-service=web-backend

# SSL certificate
gcloud compute ssl-certificates create web-cert \
  --domains=www.example.com \
  --global

# Target proxy
gcloud compute target-https-proxies create web-proxy \
  --url-map=web-map \
  --ssl-certificates=web-cert

# Forwarding rule
gcloud compute forwarding-rules create web-lb \
  --global \
  --target-https-proxy=web-proxy \
  --ports=443 \
  --address=RESERVED_IP
```

### URL Mapping and Routing
**Path-based**:
```yaml
# URL map with path matching
defaultService: default-backend
hostRules:
- hosts: ['example.com']
  pathMatcher: main
pathMatchers:
- name: main
  defaultService: default-backend
  pathRules:
  - service: api-backend
    paths: ['/api/*']
  - service: static-backend
    paths: ['/static/*']
```

**Header/Query-based**:
```yaml
routeRules:
- matchRules:
  - headerMatches:
    - headerName: version
      exactMatch: v2
  routeAction:
    weightedBackendServices:
    - backendService: backend-v2
      weight: 100
```

## Traffic Management

### Traffic Splitting
**Canary Deployment**:
```bash
# Split traffic 90/10
gcloud compute backend-services update web-backend \
  --global \
  --backend-service-policy='
    localityLbPolicy: ROUND_ROBIN
    weightedBackendServices:
    - backendService: projects/PROJECT/global/backendServices/stable-backend
      weight: 90
    - backendService: projects/PROJECT/global/backendServices/canary-backend
      weight: 10'
```

### Session Affinity
**Options**:
- NONE: No affinity
- CLIENT_IP: Based on source IP
- GENERATED_COOKIE: Load balancer-generated cookie
- HEADER_FIELD: Custom header
- HTTP_COOKIE: Application cookie

```bash
gcloud compute backend-services update web-backend \
  --session-affinity=CLIENT_IP \
  --affinity-cookie-ttl=3600 \
  --global
```

## Cloud CDN

### Configuration
```bash
# Enable CDN on backend
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --default-ttl=3600 \
  --max-ttl=86400 \
  --client-ttl=3600 \
  --global

# Cache key policy
gcloud compute backend-services update web-backend \
  --cache-key-include-host \
  --cache-key-include-protocol \
  --cache-key-include-query-string \
  --global
```

### Cache Control
**Response headers**:
```
Cache-Control: public, max-age=3600
Cache-Control: no-cache, no-store, must-revalidate
Vary: Accept-Encoding
```

**Cache Invalidation**:
```bash
gcloud compute url-maps invalidate-cdn-cache web-map \
  --path="/images/*" \
  --async
```

### Signed URLs/Cookies
```python
import base64
import hashlib
import hmac
from datetime import datetime, timedelta

def sign_url(url, key_name, key, expiration_time):
    """Generate signed URL for CDN"""
    encoded_url = base64.urlsafe_b64encode(url.encode()).decode()
    expiration = int((datetime.utcnow() + timedelta(hours=1)).timestamp())

    signature_input = f"URLPrefix={encoded_url}&Expires={expiration}&KeyName={key_name}"
    signature = base64.urlsafe_b64encode(
        hmac.new(key.encode(), signature_input.encode(), hashlib.sha1).digest()
    ).decode()

    return f"{url}?{signature_input}&Signature={signature}"
```

## Performance Optimization

### Premium vs Standard Tier
**Premium**:
- Google's global network
- Lower latency
- Higher cost
- Global anycast IPs

**Standard**:
- Regional routing
- Higher latency
- Lower cost
- Regional IPs

### Connection Optimization
**TCP Optimization**:
- TCP window scaling
- TCP Fast Open
- QUIC protocol support

**HTTP/2 and HTTP/3**:
- Multiplexing
- Server push
- Header compression

### Network Endpoint Groups (NEGs)
**Types**:
- Zonal NEG: VM instance endpoints
- Serverless NEG: Cloud Run, Cloud Functions, App Engine
- Internet NEG: External endpoints
- Hybrid NEG: On-premises endpoints

```bash
# Create serverless NEG
gcloud compute network-endpoint-groups create cloud-run-neg \
  --region=us-central1 \
  --network-endpoint-type=serverless \
  --cloud-run-service=my-service

# Add to backend
gcloud compute backend-services add-backend web-backend \
  --network-endpoint-group=cloud-run-neg \
  --network-endpoint-group-region=us-central1 \
  --global
```

## Troubleshooting

### Health Check Issues
```bash
# View health status
gcloud compute backend-services get-health web-backend --global

# Test health check
curl -v http://BACKEND_IP/health
```

### Latency Analysis
**Tools**:
- VPC Flow Logs
- Cloud Trace
- Cloud Monitoring
- Network Intelligence Center

```bash
# Enable VPC Flow Logs
gcloud compute networks subnets update SUBNET \
  --enable-flow-logs \
  --region=REGION

# Query flow logs
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  AVG(CAST(jsonPayload.rtt_msec AS INT64)) as avg_latency_ms
FROM `project.dataset.compute_googleapis_com_vpc_flows_*`
WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY src_ip, dest_ip
ORDER BY avg_latency_ms DESC;
```

## Best Practices

### Load Balancing
1. Use health checks with appropriate intervals
2. Configure session affinity when needed
3. Implement gradual traffic migration
4. Monitor backend capacity
5. Use Cloud CDN for static content
6. Configure appropriate timeouts
7. Enable connection draining

### CDN
1. Cache static assets
2. Set appropriate TTL values
3. Use cache-control headers
4. Implement cache invalidation
5. Use signed URLs for private content
6. Monitor cache hit ratio
7. Compress responses

### Performance
1. Use Premium tier for latency-sensitive apps
2. Deploy in multiple regions
3. Implement proper caching
4. Optimize backend response time
5. Monitor latency metrics
6. Use HTTP/2 or HTTP/3
7. Configure connection pooling

## Common Scenarios

**Scenario**: Global web application with CDN
**Solution**: Global HTTP(S) LB, Cloud CDN, multi-region backends, managed SSL certificates, Cloud Armor

**Scenario**: Internal microservices
**Solution**: Internal HTTP(S) LB or TCP/UDP LB, NEGs for service discovery, VPC-native

**Scenario**: API with rate limiting
**Solution**: Global HTTP(S) LB, Cloud Armor rate limiting, backend API Gateway, monitoring

## Study Tips

1. Practice load balancer configuration
2. Understand LB type selection
3. Implement traffic splitting
4. Configure Cloud CDN
5. Troubleshoot health checks
6. Network performance analysis
7. NEG use cases

## Additional Resources

- [Load Balancing Documentation](https://cloud.google.com/load-balancing/docs)
- [Cloud CDN Documentation](https://cloud.google.com/cdn/docs)
- [Network Performance Guide](https://cloud.google.com/architecture/best-practices-for-optimizing-network-performance)
