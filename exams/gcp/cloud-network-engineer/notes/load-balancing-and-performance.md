# Load Balancing and Performance - GCP Professional Cloud Network Engineer

## Overview

Load balancing architectures, traffic management, CDN, network performance optimization, and global distribution strategies for the Professional Cloud Network Engineer certification.

## Load Balancer Types Deep Dive

GCP offers seven distinct load balancer types. Understanding when to use each is critical for the exam.

### Complete Load Balancer Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────────┐
│ EXTERNAL LOAD BALANCERS (Internet-facing)                               │
├─────────────────────────────────────────────────────────────────────────┤
│ Application Load Balancer (External)                                    │
│ - Layer 7 (HTTP/HTTPS)                                                  │
│ - Global or Regional                                                    │
│ - Content-based routing (URL maps, headers, methods)                   │
│ - Cloud CDN, Cloud Armor                                                │
│ - Anycast IP (global), Regional IP (regional)                          │
│ - SSL/TLS termination                                                   │
│ - Use for: Web applications, APIs, microservices                       │
├─────────────────────────────────────────────────────────────────────────┤
│ Network Load Balancer (External)                                        │
│ - Layer 4 (TCP/UDP)                                                     │
│ - Regional only                                                         │
│ - Pass-through (preserves client IP)                                   │
│ - High performance, low latency                                         │
│ - No SSL termination                                                    │
│ - Use for: Gaming, IoT, non-HTTP protocols                             │
├─────────────────────────────────────────────────────────────────────────┤
│ Proxy Network Load Balancer (External)                                  │
│ - Layer 4 (TCP with SSL, TCP without SSL)                              │
│ - Global                                                                │
│ - TCP Proxy or SSL Proxy                                               │
│ - Anycast IP                                                            │
│ - SSL termination (SSL Proxy)                                          │
│ - Use for: Non-HTTP TCP traffic that needs global reach                │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│ INTERNAL LOAD BALANCERS (VPC-internal)                                  │
├─────────────────────────────────────────────────────────────────────────┤
│ Internal Application Load Balancer                                      │
│ - Layer 7 (HTTP/HTTPS)                                                  │
│ - Regional or Cross-region                                              │
│ - Content-based routing                                                 │
│ - Private traffic within VPC                                            │
│ - Cloud Armor support                                                   │
│ - Use for: Internal microservices, service mesh                        │
├─────────────────────────────────────────────────────────────────────────┤
│ Internal Network Load Balancer (passthrough)                            │
│ - Layer 4 (TCP/UDP)                                                     │
│ - Regional                                                              │
│ - Preserves client IP                                                   │
│ - High performance                                                      │
│ - Use for: Internal databases, message queues                          │
├─────────────────────────────────────────────────────────────────────────┤
│ Internal Network Load Balancer (proxy)                                  │
│ - Layer 4 (TCP)                                                         │
│ - Regional or Cross-region                                              │
│ - Does not preserve client IP                                           │
│ - Connection pooling                                                    │
│ - Use for: Internal services needing connection management             │
└─────────────────────────────────────────────────────────────────────────┘
```

### External Application Load Balancer (Layer 7)

The most feature-rich load balancer. Supports both global and regional configurations.

**Global External Application Load Balancer**:
```
Client → Anycast IP → Google Global Network → Regional Backend
Features:
- Single anycast IP serves globally
- Automatic routing to nearest healthy backend
- Cross-region failover
- Cloud CDN integration
- Cloud Armor security policies
- URL maps, header routing, traffic splitting
```

**Regional External Application Load Balancer**:
```
Client → Regional IP → Regional Backend
Features:
- Traffic stays within region
- Lower latency for regional workloads
- Cloud Armor support
- No Cloud CDN (use global for CDN)
- Lower cost than global
```

**Complete Global External Application LB Configuration**:
```bash
# 1. Create health check
gcloud compute health-checks create http web-http-health \
  --port=80 \
  --request-path=/healthz \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2 \
  --global

# 2. Create backend service
gcloud compute backend-services create web-backend-global \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=web-http-health \
  --timeout=30s \
  --connection-draining-timeout=300s \
  --enable-logging \
  --logging-sample-rate=1.0 \
  --global

# 3. Add backends with capacity scaling
gcloud compute backend-services add-backend web-backend-global \
  --instance-group=web-ig-us-central1 \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.80 \
  --capacity-scaler=1.0 \
  --global

gcloud compute backend-services add-backend web-backend-global \
  --instance-group=web-ig-europe-west1 \
  --instance-group-zone=europe-west1-b \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.80 \
  --capacity-scaler=1.0 \
  --global

gcloud compute backend-services add-backend web-backend-global \
  --instance-group=web-ig-asia-east1 \
  --instance-group-zone=asia-east1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.80 \
  --capacity-scaler=1.0 \
  --global

# 4. Create URL map
gcloud compute url-maps create web-url-map \
  --default-service=web-backend-global \
  --global

# 5. Reserve static IP
gcloud compute addresses create web-lb-ip \
  --ip-version=IPV4 \
  --global

# 6. Create SSL certificate (Google-managed)
gcloud compute ssl-certificates create web-ssl-cert \
  --domains=www.example.com,example.com \
  --global

# 7. Create target HTTPS proxy
gcloud compute target-https-proxies create web-https-proxy \
  --url-map=web-url-map \
  --ssl-certificates=web-ssl-cert \
  --global

# 8. Create global forwarding rule (frontend)
gcloud compute forwarding-rules create web-https-forwarding-rule \
  --address=web-lb-ip \
  --global \
  --target-https-proxy=web-https-proxy \
  --ports=443

# 9. Create HTTP to HTTPS redirect
gcloud compute url-maps import web-url-map \
  --source=redirect-config.yaml \
  --global

# redirect-config.yaml:
# defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/web-backend-global
# defaultUrlRedirect:
#   httpsRedirect: true
#   redirectResponseCode: MOVED_PERMANENTLY_DEFAULT
```

**Regional External Application LB Configuration**:
```bash
# Create regional health check
gcloud compute health-checks create http regional-web-health \
  --region=us-central1 \
  --port=80 \
  --request-path=/health

# Create regional backend service
gcloud compute backend-services create regional-web-backend \
  --protocol=HTTP \
  --health-checks=regional-web-health \
  --health-checks-region=us-central1 \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL_MANAGED

# Add regional backend
gcloud compute backend-services add-backend regional-web-backend \
  --instance-group=web-ig \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --region=us-central1

# Create regional URL map
gcloud compute url-maps create regional-url-map \
  --default-service=regional-web-backend \
  --region=us-central1

# Create regional target HTTP proxy
gcloud compute target-http-proxies create regional-http-proxy \
  --url-map=regional-url-map \
  --region=us-central1

# Create regional forwarding rule
gcloud compute forwarding-rules create regional-forwarding-rule \
  --load-balancing-scheme=EXTERNAL_MANAGED \
  --network-tier=PREMIUM \
  --address=regional-lb-ip \
  --target-http-proxy=regional-http-proxy \
  --ports=80 \
  --region=us-central1
```

### External Network Load Balancer (Layer 4 Passthrough)

Regional, high-performance TCP/UDP load balancer that preserves client IP addresses.

**Architecture**:
```
Client IP: 203.0.113.5
        ↓
External Network LB (Regional IP)
        ↓
Backend sees: 203.0.113.5 (original client IP)
```

**Complete Configuration**:
```bash
# 1. Create health check (for backend health)
gcloud compute health-checks create tcp tcp-health-check \
  --port=80 \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2 \
  --region=us-central1

# 2. Create backend service
gcloud compute backend-services create tcp-backend \
  --protocol=TCP \
  --health-checks=tcp-health-check \
  --health-checks-region=us-central1 \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL

# 3. Add instance groups
gcloud compute backend-services add-backend tcp-backend \
  --instance-group=tcp-ig \
  --instance-group-zone=us-central1-a \
  --region=us-central1

# 4. Reserve external IP
gcloud compute addresses create tcp-lb-ip \
  --region=us-central1

# 5. Create forwarding rule
gcloud compute forwarding-rules create tcp-forwarding-rule \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL \
  --address=tcp-lb-ip \
  --ip-protocol=TCP \
  --ports=80,443 \
  --backend-service=tcp-backend
```

**UDP Load Balancing**:
```bash
# For UDP (gaming, QUIC, DNS)
gcloud compute health-checks create tcp udp-health \
  --port=53 \
  --region=us-central1

gcloud compute backend-services create udp-backend \
  --protocol=UDP \
  --health-checks=udp-health \
  --health-checks-region=us-central1 \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL

gcloud compute forwarding-rules create udp-forwarding-rule \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL \
  --address=udp-lb-ip \
  --ip-protocol=UDP \
  --ports=53,5000-5100 \
  --backend-service=udp-backend
```

### Proxy Network Load Balancers (Global Layer 4)

Global TCP/SSL proxy for non-HTTP TCP traffic.

**TCP Proxy Load Balancer**:
```bash
# For non-SSL TCP traffic (e.g., database connections)
gcloud compute health-checks create tcp global-tcp-health \
  --port=3306 \
  --global

gcloud compute backend-services create mysql-proxy-backend \
  --protocol=TCP \
  --health-checks=global-tcp-health \
  --timeout=30s \
  --global

gcloud compute backend-services add-backend mysql-proxy-backend \
  --instance-group=mysql-ig-us \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --global

gcloud compute backend-services add-backend mysql-proxy-backend \
  --instance-group=mysql-ig-eu \
  --instance-group-zone=europe-west1-b \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --global

gcloud compute target-tcp-proxies create mysql-tcp-proxy \
  --backend-service=mysql-proxy-backend

gcloud compute forwarding-rules create mysql-forwarding-rule \
  --global \
  --target-tcp-proxy=mysql-tcp-proxy \
  --ports=3306 \
  --address=mysql-lb-ip
```

**SSL Proxy Load Balancer**:
```bash
# For SSL-encrypted TCP traffic
gcloud compute ssl-certificates create ssl-proxy-cert \
  --certificate=cert.pem \
  --private-key=key.pem \
  --global

gcloud compute backend-services create ssl-proxy-backend \
  --protocol=SSL \
  --health-checks=global-tcp-health \
  --global

gcloud compute backend-services add-backend ssl-proxy-backend \
  --instance-group=backend-ig \
  --instance-group-zone=us-central1-a \
  --global

gcloud compute target-ssl-proxies create ssl-target-proxy \
  --backend-service=ssl-proxy-backend \
  --ssl-certificates=ssl-proxy-cert \
  --proxy-header=PROXY_V1

gcloud compute forwarding-rules create ssl-forwarding-rule \
  --global \
  --target-ssl-proxy=ssl-target-proxy \
  --ports=443 \
  --address=ssl-lb-ip
```

### Internal Application Load Balancer

Layer 7 load balancer for private traffic within VPC.

**Regional Internal Application LB**:
```bash
# 1. Create subnet for load balancer (proxy-only subnet)
gcloud compute networks subnets create proxy-only-subnet \
  --purpose=REGIONAL_MANAGED_PROXY \
  --role=ACTIVE \
  --region=us-central1 \
  --network=default \
  --range=10.129.0.0/23

# 2. Create health check
gcloud compute health-checks create http internal-app-health \
  --region=us-central1 \
  --port=80 \
  --request-path=/healthz

# 3. Create backend service
gcloud compute backend-services create internal-app-backend \
  --load-balancing-scheme=INTERNAL_MANAGED \
  --protocol=HTTP \
  --health-checks=internal-app-health \
  --health-checks-region=us-central1 \
  --region=us-central1

# 4. Add backends
gcloud compute backend-services add-backend internal-app-backend \
  --instance-group=internal-ig \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --region=us-central1

# 5. Create URL map
gcloud compute url-maps create internal-url-map \
  --default-service=internal-app-backend \
  --region=us-central1

# 6. Create target HTTP proxy
gcloud compute target-http-proxies create internal-http-proxy \
  --url-map=internal-url-map \
  --region=us-central1

# 7. Reserve internal IP
gcloud compute addresses create internal-lb-ip \
  --region=us-central1 \
  --subnet=default \
  --purpose=SHARED_LOADBALANCER_VPC

# 8. Create forwarding rule
gcloud compute forwarding-rules create internal-forwarding-rule \
  --load-balancing-scheme=INTERNAL_MANAGED \
  --network=default \
  --subnet=default \
  --address=internal-lb-ip \
  --ports=80 \
  --region=us-central1 \
  --target-http-proxy=internal-http-proxy
```

**Cross-Region Internal Application LB**:
```bash
# For multi-region internal services
gcloud compute network-endpoint-groups create internal-neg-us \
  --network-endpoint-type=GCE_VM_IP \
  --zone=us-central1-a \
  --network=default \
  --subnet=default

gcloud compute network-endpoint-groups create internal-neg-eu \
  --network-endpoint-type=GCE_VM_IP \
  --zone=europe-west1-b \
  --network=default \
  --subnet=eu-subnet

# Create global backend service for internal cross-region
gcloud compute backend-services create cross-region-internal-backend \
  --load-balancing-scheme=INTERNAL_MANAGED \
  --protocol=HTTP \
  --health-checks=internal-app-health \
  --global

# Add backends from multiple regions
gcloud compute backend-services add-backend cross-region-internal-backend \
  --network-endpoint-group=internal-neg-us \
  --network-endpoint-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=100 \
  --global

gcloud compute backend-services add-backend cross-region-internal-backend \
  --network-endpoint-group=internal-neg-eu \
  --network-endpoint-group-zone=europe-west1-b \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=100 \
  --global
```

### Internal Network Load Balancer (Passthrough)

Layer 4 load balancer for internal TCP/UDP traffic. Preserves client IP.

**Configuration**:
```bash
# 1. Create health check
gcloud compute health-checks create tcp internal-tcp-health \
  --region=us-central1 \
  --port=3306

# 2. Create backend service
gcloud compute backend-services create internal-tcp-backend \
  --load-balancing-scheme=INTERNAL \
  --protocol=TCP \
  --health-checks=internal-tcp-health \
  --health-checks-region=us-central1 \
  --region=us-central1 \
  --connection-draining-timeout=300

# 3. Add instance groups
gcloud compute backend-services add-backend internal-tcp-backend \
  --instance-group=db-ig \
  --instance-group-zone=us-central1-a \
  --region=us-central1

# 4. Reserve internal IP
gcloud compute addresses create internal-tcp-ip \
  --region=us-central1 \
  --subnet=default \
  --purpose=SHARED_LOADBALANCER_VPC

# 5. Create forwarding rule
gcloud compute forwarding-rules create internal-tcp-forwarding-rule \
  --load-balancing-scheme=INTERNAL \
  --region=us-central1 \
  --address=internal-tcp-ip \
  --ip-protocol=TCP \
  --ports=3306 \
  --backend-service=internal-tcp-backend \
  --subnet=default \
  --network=default
```

**UDP Internal Load Balancer**:
```bash
gcloud compute backend-services create internal-udp-backend \
  --load-balancing-scheme=INTERNAL \
  --protocol=UDP \
  --health-checks=internal-tcp-health \
  --health-checks-region=us-central1 \
  --region=us-central1

gcloud compute forwarding-rules create internal-udp-forwarding-rule \
  --load-balancing-scheme=INTERNAL \
  --region=us-central1 \
  --address=internal-udp-ip \
  --ip-protocol=UDP \
  --ports=161,162 \
  --backend-service=internal-udp-backend \
  --subnet=default \
  --network=default
```

### Internal Network Load Balancer (Proxy)

Layer 4 proxy load balancer for internal traffic. Does not preserve client IP but provides connection pooling.

```bash
# Create proxy-only subnet first
gcloud compute networks subnets create internal-proxy-subnet \
  --purpose=REGIONAL_MANAGED_PROXY \
  --role=ACTIVE \
  --region=us-central1 \
  --network=default \
  --range=10.130.0.0/23

# Create backend service
gcloud compute backend-services create internal-proxy-backend \
  --load-balancing-scheme=INTERNAL_MANAGED \
  --protocol=TCP \
  --health-checks=internal-tcp-health \
  --health-checks-region=us-central1 \
  --region=us-central1

# Create target TCP proxy
gcloud compute target-tcp-proxies create internal-target-proxy \
  --backend-service=internal-proxy-backend \
  --proxy-header=PROXY_V1 \
  --region=us-central1

# Create forwarding rule
gcloud compute forwarding-rules create internal-proxy-forwarding-rule \
  --load-balancing-scheme=INTERNAL_MANAGED \
  --region=us-central1 \
  --address=internal-proxy-ip \
  --ports=3306 \
  --target-tcp-proxy=internal-target-proxy \
  --subnet=default \
  --network=default
```

## Global Load Balancing Deep Dive

### Anycast IP Addressing

Global load balancers use anycast IP addresses. The same IP is advertised from multiple Google locations worldwide.

**How Anycast Works**:
```
User in Tokyo (203.0.113.1) → Anycast IP: 34.107.200.1
                               ↓
                     Routes to asia-northeast1

User in London (198.51.100.1) → Same IP: 34.107.200.1
                                ↓
                     Routes to europe-west2

Benefits:
- Single IP for global service
- Automatic geo-routing
- DDoS protection (distributed attack surface)
- No DNS-based routing delays
```

**Reserve Anycast IP**:
```bash
# Global anycast IP
gcloud compute addresses create global-anycast-ip \
  --ip-version=IPV4 \
  --network-tier=PREMIUM \
  --global

# Get IP address
gcloud compute addresses describe global-anycast-ip \
  --global \
  --format="get(address)"

# IPv6 support
gcloud compute addresses create global-ipv6 \
  --ip-version=IPV6 \
  --network-tier=PREMIUM \
  --global
```

### Cross-Region Failover

Global load balancers automatically failover across regions when backends become unhealthy.

**Failover Architecture**:
```
Primary Region: us-central1 (100% traffic)
        ↓
    [All backends healthy]
        ↓
Backend Failure in us-central1
        ↓
Automatic failover → europe-west1 (nearest healthy region)
        ↓
Traffic reroutes in <1 second
```

**Configuration with Failover Policy**:
```bash
# Create backend service with failover configuration
gcloud compute backend-services create web-backend-failover \
  --protocol=HTTP \
  --health-checks=web-health \
  --connection-draining-timeout=300 \
  --enable-logging \
  --global

# Add primary backends (us-central1)
gcloud compute backend-services add-backend web-backend-failover \
  --instance-group=web-ig-us-central1-a \
  --instance-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-instance=1000 \
  --capacity-scaler=1.0 \
  --global

gcloud compute backend-services add-backend web-backend-failover \
  --instance-group=web-ig-us-central1-b \
  --instance-group-zone=us-central1-b \
  --balancing-mode=RATE \
  --max-rate-per-instance=1000 \
  --capacity-scaler=1.0 \
  --global

# Add failover backends (europe-west1)
gcloud compute backend-services add-backend web-backend-failover \
  --instance-group=web-ig-europe-west1 \
  --instance-group-zone=europe-west1-b \
  --balancing-mode=RATE \
  --max-rate-per-instance=1000 \
  --capacity-scaler=1.0 \
  --failover \
  --global

# Configure failover policy
gcloud compute backend-services update web-backend-failover \
  --failover-ratio=0.1 \
  --drop-traffic-if-unhealthy \
  --global
```

**Failover Ratio**: When primary capacity drops below threshold (e.g., 10%), failover backends activate.

### Backend Service Configuration

Complete backend service configuration options for the exam.

**Balancing Modes**:
```bash
# UTILIZATION - Based on CPU/RPS utilization
gcloud compute backend-services add-backend web-backend \
  --instance-group=web-ig \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.80 \
  --capacity-scaler=1.0 \
  --global

# RATE - Requests per second per instance
gcloud compute backend-services add-backend api-backend \
  --instance-group=api-ig \
  --instance-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-instance=100 \
  --capacity-scaler=1.0 \
  --global

# CONNECTION - Max concurrent connections
gcloud compute backend-services add-backend tcp-backend \
  --instance-group=tcp-ig \
  --instance-group-zone=us-central1-a \
  --balancing-mode=CONNECTION \
  --max-connections-per-instance=1000 \
  --global
```

**Capacity Scaler**: Scale backend capacity (0.0 to 1.0).
- 1.0 = 100% capacity (default)
- 0.5 = 50% capacity (useful for canary)
- 0.0 = drain traffic (maintenance)

```bash
# Reduce traffic to 50% for canary testing
gcloud compute backend-services update-backend web-backend \
  --instance-group=canary-ig \
  --instance-group-zone=us-central1-a \
  --capacity-scaler=0.5 \
  --global

# Drain backend for maintenance (no new connections)
gcloud compute backend-services update-backend web-backend \
  --instance-group=maint-ig \
  --instance-group-zone=us-central1-a \
  --capacity-scaler=0.0 \
  --global
```

**Connection Draining**: Gracefully close existing connections before removing backend.
```bash
gcloud compute backend-services update web-backend \
  --connection-draining-timeout=300 \
  --global

# During draining:
# - No new connections to backend
# - Existing connections have 300s to complete
# - After timeout, connections forcefully closed
```

**Backend Timeouts**:
```bash
gcloud compute backend-services update web-backend \
  --timeout=30s \
  --connection-draining-timeout=300s \
  --global

# Timeout applies to:
# - Backend response time
# - Does NOT include connection draining
```

### Health Checks Configuration

Comprehensive health check configuration for different protocols.

**HTTP Health Check**:
```bash
gcloud compute health-checks create http web-http-health \
  --port=80 \
  --request-path=/healthz \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2 \
  --proxy-header=NONE \
  --global

# With custom headers
gcloud compute health-checks create http web-http-health-custom \
  --port=8080 \
  --request-path=/api/health \
  --response="OK" \
  --check-interval=5s \
  --timeout=3s \
  --unhealthy-threshold=2 \
  --healthy-threshold=2 \
  --global
```

**HTTPS Health Check**:
```bash
gcloud compute health-checks create https web-https-health \
  --port=443 \
  --request-path=/healthz \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2 \
  --global
```

**HTTP/2 Health Check**:
```bash
gcloud compute health-checks create http2 web-http2-health \
  --port=443 \
  --request-path=/health \
  --check-interval=10s \
  --timeout=5s \
  --global
```

**TCP Health Check**:
```bash
gcloud compute health-checks create tcp tcp-health \
  --port=3306 \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2 \
  --proxy-header=NONE \
  --region=us-central1
```

**SSL Health Check**:
```bash
gcloud compute health-checks create ssl ssl-health \
  --port=443 \
  --check-interval=10s \
  --timeout=5s \
  --global
```

**gRPC Health Check**:
```bash
gcloud compute health-checks create grpc grpc-health \
  --port=50051 \
  --check-interval=10s \
  --timeout=5s \
  --global
```

**Health Check Best Practices**:
```
Interval: 5-10 seconds (balance between responsiveness and overhead)
Timeout: 3-5 seconds (< interval)
Unhealthy Threshold: 2-3 (avoid false positives)
Healthy Threshold: 2 (faster recovery)

Calculation:
Time to mark unhealthy = interval × unhealthy_threshold
Example: 10s × 3 = 30s

Time to mark healthy = interval × healthy_threshold
Example: 10s × 2 = 20s
```

**Monitoring Health Status**:
```bash
# Get health status of backends
gcloud compute backend-services get-health web-backend \
  --global

# Output shows:
# - Backend instance/group
# - Health state (HEALTHY/UNHEALTHY)
# - Health check details

# Example output:
# ---
# backend: https://www.googleapis.com/compute/v1/projects/PROJECT/zones/us-central1-a/instanceGroups/web-ig
# status:
#   healthStatus:
#   - healthState: HEALTHY
#     instance: https://www.googleapis.com/compute/v1/projects/PROJECT/zones/us-central1-a/instances/web-vm-1
#     ipAddress: 10.128.0.2
#     port: 80
```

### Session Affinity

Control how clients are routed to backends across multiple requests.

**Session Affinity Options**:
```bash
# 1. NONE - No affinity, round-robin distribution
gcloud compute backend-services update web-backend \
  --session-affinity=NONE \
  --global

# 2. CLIENT_IP - Based on client IP address
gcloud compute backend-services update web-backend \
  --session-affinity=CLIENT_IP \
  --global

# 3. CLIENT_IP_PROTO - Client IP + protocol
gcloud compute backend-services update web-backend \
  --session-affinity=CLIENT_IP_PROTO \
  --global

# 4. CLIENT_IP_PORT_PROTO - Client IP + port + protocol
gcloud compute backend-services update web-backend \
  --session-affinity=CLIENT_IP_PORT_PROTO \
  --global

# 5. GENERATED_COOKIE - LB generates cookie (HTTP(S) only)
gcloud compute backend-services update web-backend \
  --session-affinity=GENERATED_COOKIE \
  --affinity-cookie-ttl=3600 \
  --global

# 6. HEADER_FIELD - Custom header (HTTP(S) only)
gcloud compute backend-services update web-backend \
  --session-affinity=HEADER_FIELD \
  --custom-request-header="X-User-ID" \
  --global

# 7. HTTP_COOKIE - Application cookie (HTTP(S) only)
gcloud compute backend-services update web-backend \
  --session-affinity=HTTP_COOKIE \
  --affinity-cookie-ttl=7200 \
  --global
```

**Use Cases**:
```
CLIENT_IP:
- Simple stateful applications
- WebSocket connections
- Applications without cookie support

GENERATED_COOKIE:
- Shopping carts
- User sessions
- Multi-step forms

HTTP_COOKIE:
- Applications with existing session cookies
- Custom session management

HEADER_FIELD:
- API routing by user ID
- Tenant-based routing
- Custom affinity logic
```

**Important Notes**:
- Session affinity is best-effort, not guaranteed
- Backends going unhealthy breaks affinity
- Strong affinity can create hot spots
- Consider stateless design with external session store

## Cloud CDN (Content Delivery Network)

Cloud CDN caches content at Google's globally distributed edge points of presence (PoPs).

### Cache Modes

```bash
# 1. CACHE_ALL_STATIC (default)
# Caches static content automatically
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --global

# 2. USE_ORIGIN_HEADERS
# Respects Cache-Control headers from origin
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=USE_ORIGIN_HEADERS \
  --global

# 3. FORCE_CACHE_ALL
# Caches everything regardless of headers
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=FORCE_CACHE_ALL \
  --default-ttl=3600 \
  --max-ttl=86400 \
  --client-ttl=3600 \
  --global
```

**Cache Mode Comparison**:
```
CACHE_ALL_STATIC:
- Automatically caches common static file types
- Images: .jpg, .png, .gif, .webp
- Scripts: .js, .css
- Documents: .pdf, .xml
- Videos: .mp4, .webm
- No configuration needed

USE_ORIGIN_HEADERS:
- Full control via Cache-Control headers
- Requires backend configuration
- Most flexible
- Best for dynamic content with caching

FORCE_CACHE_ALL:
- Caches all responses
- Ignores Cache-Control headers
- Set TTLs at CDN level
- Use with caution (can cache errors)
```

### TTL Configuration

```bash
# Configure all TTL values
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=FORCE_CACHE_ALL \
  --default-ttl=3600 \
  --max-ttl=86400 \
  --client-ttl=7200 \
  --serve-while-stale=86400 \
  --global
```

**TTL Types**:
```
default-ttl (1 hour):
- Used when origin doesn't specify Cache-Control
- Fallback TTL

max-ttl (24 hours):
- Maximum cache time
- Overrides longer Cache-Control values
- Prevents stale content

client-ttl (2 hours):
- TTL in Cache-Control header to client
- Can differ from CDN cache time

serve-while-stale (24 hours):
- Serve stale content if origin is unavailable
- CDN revalidates in background
- Improves availability
```

### Cache Keys

Cache keys determine what makes a request unique for caching purposes.

```bash
# Include query string in cache key
gcloud compute backend-services update web-backend \
  --cache-key-include-protocol \
  --cache-key-include-host \
  --cache-key-include-query-string \
  --global

# Whitelist specific query parameters
gcloud compute backend-services update web-backend \
  --cache-key-include-protocol \
  --cache-key-include-host \
  --cache-key-include-query-string \
  --cache-key-query-string-whitelist=version,locale \
  --global

# Blacklist query parameters (ignore for caching)
gcloud compute backend-services update web-backend \
  --cache-key-include-protocol \
  --cache-key-include-host \
  --cache-key-include-query-string \
  --cache-key-query-string-blacklist=utm_source,utm_campaign \
  --global

# Include HTTP headers in cache key
gcloud compute backend-services update web-backend \
  --cache-key-include-protocol \
  --cache-key-include-host \
  --cache-key-include-named-cookies=session_id,user_pref \
  --global
```

**Cache Key Examples**:
```
Basic cache key:
https://example.com/image.jpg
→ Cache key: https|example.com|/image.jpg

With query strings:
https://example.com/image.jpg?size=large&format=webp
→ Cache key: https|example.com|/image.jpg|size=large&format=webp

With cookie:
https://example.com/page.html + Cookie: session_id=abc123
→ Cache key: https|example.com|/page.html|session_id=abc123

Analytics parameters ignored:
https://example.com/page?utm_source=email&id=5
→ Cache key: https|example.com|/page|id=5
(utm_source blacklisted)
```

### Signed URLs and Signed Cookies

Protect cached content with time-limited signed URLs or cookies.

**Create Signing Key**:
```bash
# Generate key
head -c 16 /dev/urandom | base64 | tr +/ -_ | tr -d '=' > key.txt

# Add key to backend
gcloud compute backend-services add-signed-url-key web-backend \
  --key-name=my-key \
  --key-file=key.txt \
  --global
```

**Generate Signed URL (Python)**:
```python
import base64
import hashlib
import hmac
import time
from datetime import datetime, timedelta
from urllib.parse import urlparse

def sign_url(url, key_name, key, expiration_time=3600):
    """Generate signed URL for Cloud CDN"""
    # Parse URL
    parsed = urlparse(url)

    # Calculate expiration timestamp
    expires = int(time.time()) + expiration_time

    # Create signature base
    url_to_sign = f"{parsed.path}?Expires={expires}&KeyName={key_name}"

    # Generate signature
    signature = base64.urlsafe_b64encode(
        hmac.new(
            base64.urlsafe_b64decode(key),
            url_to_sign.encode('utf-8'),
            hashlib.sha1
        ).digest()
    ).decode('utf-8')

    # Remove padding
    signature = signature.rstrip('=')

    # Build signed URL
    signed_url = f"{url}?Expires={expires}&KeyName={key_name}&Signature={signature}"

    return signed_url

# Usage
url = "https://cdn.example.com/videos/private-video.mp4"
key_name = "my-key"
key = "YOUR_BASE64_KEY"

signed_url = sign_url(url, key_name, key, expiration_time=3600)
print(f"Signed URL: {signed_url}")
```

**Signed Cookies**:
```python
def sign_cookie(url_prefix, key_name, key, expiration_time=3600):
    """Generate signed cookie for Cloud CDN"""
    expires = int(time.time()) + expiration_time

    # URLPrefix must be base64url encoded
    encoded_url_prefix = base64.urlsafe_b64encode(
        url_prefix.encode('utf-8')
    ).decode('utf-8').rstrip('=')

    # Create signature base
    policy = f"URLPrefix={encoded_url_prefix}:Expires={expires}:KeyName={key_name}"

    # Generate signature
    signature = base64.urlsafe_b64encode(
        hmac.new(
            base64.urlsafe_b64decode(key),
            policy.encode('utf-8'),
            hashlib.sha1
        ).digest()
    ).decode('utf-8').rstrip('=')

    # Return cookie values
    return {
        'Cloud-CDN-Cookie': f"URLPrefix={encoded_url_prefix}:Expires={expires}:KeyName={key_name}:Signature={signature}"
    }

# Usage
cookie = sign_cookie("https://cdn.example.com/private/", "my-key", key)
# Set-Cookie: Cloud-CDN-Cookie=URLPrefix=...:Expires=...:KeyName=...:Signature=...
```

### Cache Invalidation

```bash
# Invalidate specific path
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --path="/images/logo.png" \
  --global

# Invalidate directory
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --path="/images/*" \
  --global

# Invalidate multiple paths
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --path="/page1.html" \
  --path="/page2.html" \
  --path="/css/style.css" \
  --global

# Invalidate entire site (use carefully)
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --path="/*" \
  --global

# Check invalidation status
gcloud compute operations describe OPERATION_ID --global
```

**Invalidation Considerations**:
```
- Propagation time: ~5 minutes globally
- Cost: First 1000 invalidations/month free, then charged
- Rate limits: Max 1000 invalidations per request
- Best practice: Use versioned URLs instead of invalidation
  - Example: /images/logo.v2.png instead of invalidating /images/logo.png
```

### Negative Caching

Cache error responses to reduce load on origin.

```bash
# Enable negative caching
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --negative-caching \
  --global

# Configure negative cache TTLs for specific codes
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --negative-caching \
  --negative-caching-policy=404=60,403=120,500=5 \
  --global
```

**Negative Caching Policy**:
```
404 Not Found: 60s
- Cache missing resources briefly
- Reduces repeated lookups

403 Forbidden: 120s
- Cache authorization failures
- Prevents auth system overload

500 Internal Server Error: 5s
- Short TTL for server errors
- Allows quick recovery

502 Bad Gateway: 10s
- Brief cache for backend issues
- Enables fast failover
```

### Custom Origins

CDN can cache content from custom origins outside GCP.

```bash
# Create NEG for custom origin
gcloud compute network-endpoint-groups create custom-origin-neg \
  --network-endpoint-type=INTERNET_FQDN_PORT \
  --global

# Add custom origin endpoint
gcloud compute network-endpoint-groups update custom-origin-neg \
  --add-endpoint="fqdn=origin.example.com,port=443" \
  --global

# Create backend service with custom origin
gcloud compute backend-services create custom-origin-backend \
  --protocol=HTTPS \
  --global

# Add custom origin NEG to backend
gcloud compute backend-services add-backend custom-origin-backend \
  --network-endpoint-group=custom-origin-neg \
  --global-network-endpoint-group \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=1000 \
  --global

# Enable CDN
gcloud compute backend-services update custom-origin-backend \
  --enable-cdn \
  --cache-mode=USE_ORIGIN_HEADERS \
  --global
```

**Custom Origin Use Cases**:
- Migrate to GCP gradually (CDN first, then origin)
- Multi-cloud architecture
- Legacy systems that can't move to GCP
- Third-party content aggregation

### CDN Monitoring

```bash
# View cache hit/miss metrics
gcloud monitoring time-series list \
  --filter='metric.type="loadbalancing.googleapis.com/https/backend_request_count"' \
  --format=json

# Cache hit ratio query (Cloud Monitoring)
# Go to Monitoring > Metrics Explorer
# Metric: Cloud CDN → cache_hit_ratio
# Resource: Backend service

# Log CDN requests
gcloud compute backend-services update web-backend \
  --enable-logging \
  --logging-sample-rate=1.0 \
  --global

# Query CDN logs
gcloud logging read 'resource.type="http_load_balancer"
  jsonPayload.cacheHit=true' \
  --limit=50 \
  --format=json
```

## Cloud Armor (Security Policies)

Cloud Armor provides DDoS protection, WAF rules, and bot management for load balancers.

### Security Policy Basics

```bash
# Create security policy
gcloud compute security-policies create web-security-policy \
  --description="Security policy for web application"

# Attach to backend service
gcloud compute backend-services update web-backend \
  --security-policy=web-security-policy \
  --global

# Default rule (always exists)
gcloud compute security-policies rules update 2147483647 \
  --security-policy=web-security-policy \
  --action=deny-403
```

### Preconfigured WAF Rules

```bash
# Block SQL injection attacks
gcloud compute security-policies rules create 1000 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('sqli-v33-stable')" \
  --action=deny-403 \
  --description="Block SQL injection"

# Block XSS attacks
gcloud compute security-policies rules create 1100 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('xss-v33-stable')" \
  --action=deny-403 \
  --description="Block XSS"

# Block local file inclusion
gcloud compute security-policies rules create 1200 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('lfi-v33-stable')" \
  --action=deny-403 \
  --description="Block LFI"

# Block remote code execution
gcloud compute security-policies rules create 1300 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('rce-v33-stable')" \
  --action=deny-403 \
  --description="Block RCE"

# Block remote file inclusion
gcloud compute security-policies rules create 1400 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('rfi-v33-stable')" \
  --action=deny-403 \
  --description="Block RFI"

# Block scanner detection
gcloud compute security-policies rules create 1500 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('scannerdetection-v33-stable')" \
  --action=deny-403 \
  --description="Block scanners"

# Block protocol attacks
gcloud compute security-policies rules create 1600 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('protocolattack-v33-stable')" \
  --action=deny-403 \
  --description="Block protocol attacks"

# Block PHP injection
gcloud compute security-policies rules create 1700 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('php-v33-stable')" \
  --action=deny-403 \
  --description="Block PHP injection"

# Block session fixation
gcloud compute security-policies rules create 1800 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('sessionfixation-v33-stable')" \
  --action=deny-403 \
  --description="Block session fixation"
```

**Sensitivity Levels**:
```bash
# High sensitivity (more false positives, better protection)
gcloud compute security-policies rules create 1000 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('sqli-v33-stable', {'sensitivity': 1})" \
  --action=deny-403

# Medium sensitivity (default)
gcloud compute security-policies rules create 1000 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('sqli-v33-stable', {'sensitivity': 2})" \
  --action=deny-403

# Low sensitivity (fewer false positives)
gcloud compute security-policies rules create 1000 \
  --security-policy=web-security-policy \
  --expression="evaluatePreconfiguredWaf('sqli-v33-stable', {'sensitivity': 3})" \
  --action=deny-403
```

### Custom Rules

```bash
# Block specific IP addresses
gcloud compute security-policies rules create 100 \
  --security-policy=web-security-policy \
  --expression="inIpRange(origin.ip, '203.0.113.0/24')" \
  --action=deny-403 \
  --description="Block malicious IP range"

# Allow only specific countries (geo-blocking)
gcloud compute security-policies rules create 200 \
  --security-policy=web-security-policy \
  --expression="origin.region_code != 'US' && origin.region_code != 'CA'" \
  --action=deny-403 \
  --description="Block non-US/CA traffic"

# Block specific user agents
gcloud compute security-policies rules create 300 \
  --security-policy=web-security-policy \
  --expression="has(request.headers['user-agent']) && request.headers['user-agent'].contains('BadBot')" \
  --action=deny-403 \
  --description="Block bad bots"

# Protect admin paths
gcloud compute security-policies rules create 400 \
  --security-policy=web-security-policy \
  --expression="request.path.matches('/admin/.*') && !inIpRange(origin.ip, '10.0.0.0/8')" \
  --action=deny-403 \
  --description="Restrict admin access"

# Block specific HTTP methods
gcloud compute security-policies rules create 500 \
  --security-policy=web-security-policy \
  --expression="request.method == 'DELETE' || request.method == 'PATCH'" \
  --action=deny-403 \
  --description="Block certain HTTP methods"

# Allow only HTTPS
gcloud compute security-policies rules create 600 \
  --security-policy=web-security-policy \
  --expression="request.scheme != 'https'" \
  --action=deny-403 \
  --description="Enforce HTTPS"

# Block based on header
gcloud compute security-policies rules create 700 \
  --security-policy=web-security-policy \
  --expression="!has(request.headers['x-api-key'])" \
  --action=deny-401 \
  --description="Require API key"
```

### Rate Limiting

```bash
# Create rate limit rule (100 requests per minute per IP)
gcloud compute security-policies rules create 2000 \
  --security-policy=web-security-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP \
  --description="Rate limit per IP"

# Rate limit per user (via cookie)
gcloud compute security-policies rules create 2100 \
  --security-policy=web-security-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=1000 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=300 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=HTTP-COOKIE \
  --enforce-on-key-name=session_id \
  --description="Rate limit per user session"

# Rate limit specific path
gcloud compute security-policies rules create 2200 \
  --security-policy=web-security-policy \
  --expression="request.path.matches('/api/.*')" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=50 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=120 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP \
  --description="Rate limit API endpoints"

# Throttle instead of ban
gcloud compute security-policies rules create 2300 \
  --security-policy=web-security-policy \
  --expression="true" \
  --action=throttle \
  --rate-limit-threshold-count=200 \
  --rate-limit-threshold-interval-sec=60 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP \
  --description="Throttle excessive requests"
```

**Enforce-On-Key Options**:
```
IP: Per source IP address
ALL: Global rate limit
HTTP-HEADER: Based on HTTP header value
XFF-IP: X-Forwarded-For IP
HTTP-COOKIE: Based on cookie value
HTTP-PATH: Based on request path
SNI: Based on TLS SNI
```

### Adaptive Protection

Machine learning-based DDoS protection that adapts to traffic patterns.

```bash
# Enable Adaptive Protection
gcloud compute security-policies update web-security-policy \
  --enable-layer7-ddos-defense \
  --layer7-ddos-defense-auto-deploy-confidence-threshold=0.8 \
  --layer7-ddos-defense-auto-deploy-impacted-baseline-threshold=0.1 \
  --layer7-ddos-defense-auto-deploy-expiration-sec=3600

# Alert only mode (for testing)
gcloud compute security-policies update web-security-policy \
  --enable-layer7-ddos-defense \
  --layer7-ddos-defense-enable=false \
  --layer7-ddos-defense-rule-visibility=STANDARD
```

**Adaptive Protection Parameters**:
```
confidence-threshold (0.0-1.0):
- 0.8 = High confidence (fewer false positives)
- 0.5 = Medium confidence
- Lower values = more aggressive

impacted-baseline-threshold (0.0-1.0):
- 0.1 = 10% deviation from baseline triggers protection
- Higher values = less sensitive

expiration-sec:
- How long auto-deployed rules remain active
- Default: 3600s (1 hour)
```

### Bot Management

```bash
# Enable reCAPTCHA Enterprise integration
gcloud compute security-policies update web-security-policy \
  --recaptcha-redirect-site-key=YOUR_SITE_KEY

# Challenge suspected bots
gcloud compute security-policies rules create 3000 \
  --security-policy=web-security-policy \
  --expression="origin.user_agent.matches('.*bot.*')" \
  --action=redirect \
  --redirect-type=google-recaptcha \
  --description="Challenge bot traffic"

# Block low-score reCAPTCHA responses
gcloud compute security-policies rules create 3100 \
  --security-policy=web-security-policy \
  --expression="token.recaptcha_session.score < 0.4" \
  --action=deny-403 \
  --description="Block low reCAPTCHA scores"

# Allow verified bots (Googlebot, etc.)
gcloud compute security-policies rules create 3200 \
  --security-policy=web-security-policy \
  --expression="origin.user_agent.matches('.*(Googlebot|bingbot).*')" \
  --action=allow \
  --description="Allow search engine bots"
```

### Security Policy Logging

```bash
# Enable verbose logging
gcloud compute security-policies update web-security-policy \
  --log-level=VERBOSE

# Query security policy logs
gcloud logging read 'resource.type="http_load_balancer"
  jsonPayload.enforcedSecurityPolicy.name="web-security-policy"
  jsonPayload.enforcedSecurityPolicy.outcome="DENY"' \
  --limit=50 \
  --format=json

# View blocked requests
gcloud logging read 'resource.type="http_load_balancer"
  jsonPayload.enforcedSecurityPolicy.outcome="DENY"
  jsonPayload.statusDetails="denied_by_security_policy"' \
  --limit=50 \
  --format=json
```

## SSL/TLS Configuration

### SSL Certificate Types

**1. Google-managed SSL Certificates**:
```bash
# Automatically provisioned and renewed
gcloud compute ssl-certificates create web-cert-managed \
  --domains=www.example.com,example.com,api.example.com \
  --global

# Check provisioning status
gcloud compute ssl-certificates describe web-cert-managed \
  --global \
  --format="get(managed.status)"

# Status values:
# - PROVISIONING: Certificate being issued
# - ACTIVE: Certificate ready
# - RENEWAL_FAILED: Check DNS
```

**2. Self-managed SSL Certificates**:
```bash
# Upload your own certificate
gcloud compute ssl-certificates create web-cert-self-managed \
  --certificate=cert.pem \
  --private-key=key.pem \
  --global

# Regional certificate (for regional LB)
gcloud compute ssl-certificates create regional-cert \
  --certificate=cert.pem \
  --private-key=key.pem \
  --region=us-central1

# Certificate chain (intermediate + root CAs)
gcloud compute ssl-certificates create web-cert-chain \
  --certificate=fullchain.pem \
  --private-key=key.pem \
  --global
```

**3. Certificate Manager (Recommended)**:
```bash
# Create certificate map
gcloud certificate-manager maps create web-cert-map \
  --description="Certificate map for web application"

# Create DNS authorization
gcloud certificate-manager dns-authorizations create dns-auth \
  --domain=example.com

# Create certificate
gcloud certificate-manager certificates create web-cert \
  --domains=www.example.com,example.com \
  --dns-authorizations=dns-auth

# Create certificate map entry
gcloud certificate-manager maps entries create web-entry \
  --map=web-cert-map \
  --certificates=web-cert \
  --hostname=www.example.com

# Attach to target proxy
gcloud compute target-https-proxies update web-https-proxy \
  --certificate-map=web-cert-map \
  --global
```

### SSL Policies

Control TLS versions and cipher suites.

```bash
# Create custom SSL policy
gcloud compute ssl-policies create modern-ssl-policy \
  --profile=MODERN \
  --min-tls-version=1.2 \
  --global

# Apply to target proxy
gcloud compute target-https-proxies update web-https-proxy \
  --ssl-policy=modern-ssl-policy \
  --global

# SSL policy profiles:
# - COMPATIBLE: TLS 1.0+ (legacy compatibility)
# - MODERN: TLS 1.2+ (recommended)
# - RESTRICTED: TLS 1.2+ with restricted ciphers
# - CUSTOM: Define your own cipher list
```

**Custom SSL Policy with Specific Ciphers**:
```bash
gcloud compute ssl-policies create custom-ssl-policy \
  --profile=CUSTOM \
  --min-tls-version=1.2 \
  --custom-features=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 \
  --global
```

**SSL Policy Comparison**:
```
MODERN (Recommended):
- TLS 1.2 and 1.3
- Strong cipher suites
- Forward secrecy
- Suitable for most applications

RESTRICTED:
- TLS 1.2 and 1.3 only
- PCI DSS compliant ciphers
- Use for compliance requirements

COMPATIBLE:
- TLS 1.0, 1.1, 1.2, 1.3
- Legacy browser support
- Less secure (avoid if possible)
```

### HTTPS Redirects

```bash
# Method 1: URL map redirect
gcloud compute url-maps import web-url-map \
  --source=url-map-redirect.yaml \
  --global
```

**url-map-redirect.yaml**:
```yaml
name: web-url-map
defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/web-backend
hostRules:
- hosts:
  - example.com
  - www.example.com
  pathMatcher: main

pathMatchers:
- name: main
  defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/web-backend
  pathRules:
  - paths:
    - /*
    urlRedirect:
      httpsRedirect: true
      redirectResponseCode: MOVED_PERMANENTLY_DEFAULT
```

**Method 2: Separate HTTP forwarding rule**:
```bash
# Create HTTP target proxy with redirect
gcloud compute target-http-proxies create web-http-redirect \
  --url-map=web-url-map \
  --global

# Create HTTP forwarding rule (port 80)
gcloud compute forwarding-rules create http-redirect-rule \
  --global \
  --target-http-proxy=web-http-redirect \
  --ports=80 \
  --address=web-lb-ip
```

### Mutual TLS (mTLS)

Client certificate authentication for enhanced security.

```bash
# Create trust config with client CA
gcloud certificate-manager trust-configs create mtls-trust \
  --trust-stores=trust-store.yaml

# trust-store.yaml:
# trustAnchors:
# - pemCertificate: |
#     -----BEGIN CERTIFICATE-----
#     <client-ca-certificate>
#     -----END CERTIFICATE-----

# Create server TLS policy
gcloud network-security server-tls-policies create mtls-policy \
  --mtls-policy=mtls-config.yaml \
  --location=global

# mtls-config.yaml:
# clientValidationMode: REJECT_INVALID
# clientValidationTrustConfig: projects/PROJECT/locations/global/trustConfigs/mtls-trust

# Attach to target HTTPS proxy
gcloud compute target-https-proxies update web-https-proxy \
  --server-tls-policy=mtls-policy \
  --global
```

**mTLS Validation Modes**:
```
REJECT_INVALID:
- Require valid client certificate
- Reject if cert invalid or missing
- Strict mode

ALLOW_INVALID_OR_MISSING_CLIENT_CERT:
- Accept requests without cert
- Pass cert info to backend if present
- Backend validates

ALLOW_INVALID:
- Accept any cert or no cert
- Pass cert info to backend
- Use with caution
```

## Traffic Management

### URL Mapping and Routing

**Path-based Routing**:
```bash
# Create multiple backend services
gcloud compute backend-services create web-frontend \
  --protocol=HTTP \
  --health-checks=http-health \
  --global

gcloud compute backend-services create api-backend \
  --protocol=HTTP \
  --health-checks=http-health \
  --global

gcloud compute backend-services create static-backend \
  --protocol=HTTP \
  --health-checks=http-health \
  --enable-cdn \
  --global

# Import URL map with path rules
gcloud compute url-maps import web-url-map \
  --source=url-map-paths.yaml \
  --global
```

**url-map-paths.yaml**:
```yaml
name: web-url-map
defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/web-frontend

hostRules:
- hosts:
  - example.com
  - www.example.com
  pathMatcher: main-matcher

pathMatchers:
- name: main-matcher
  defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/web-frontend

  pathRules:
  - paths:
    - /api/*
    - /api/v1/*
    service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/api-backend

  - paths:
    - /static/*
    - /images/*
    - /css/*
    - /js/*
    service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/static-backend
```

**Host-based Routing**:
```yaml
name: multi-host-url-map
defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/default-backend

hostRules:
- hosts:
  - www.example.com
  pathMatcher: web-matcher

- hosts:
  - api.example.com
  pathMatcher: api-matcher

- hosts:
  - admin.example.com
  pathMatcher: admin-matcher

pathMatchers:
- name: web-matcher
  defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/web-backend

- name: api-matcher
  defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/api-backend

- name: admin-matcher
  defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/admin-backend
```

**Header-based Routing**:
```yaml
name: header-routing-map
defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/default-backend

hostRules:
- hosts: ['api.example.com']
  pathMatcher: api-matcher

pathMatchers:
- name: api-matcher
  defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/api-v1-backend

  routeRules:
  # Route based on API version header
  - priority: 1
    matchRules:
    - headerMatches:
      - headerName: X-API-Version
        exactMatch: "v2"
    service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/api-v2-backend

  # Route based on user agent
  - priority: 2
    matchRules:
    - headerMatches:
      - headerName: User-Agent
        regexMatch: ".*Mobile.*"
    service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/mobile-backend

  # Route based on authorization
  - priority: 3
    matchRules:
    - headerMatches:
      - headerName: Authorization
        prefixMatch: "Bearer premium-"
    service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/premium-backend
```

**Query Parameter Routing**:
```yaml
routeRules:
- priority: 1
  matchRules:
  - queryParameterMatches:
    - name: version
      exactMatch: "beta"
  service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/beta-backend

- priority: 2
  matchRules:
  - queryParameterMatches:
    - name: locale
      presentMatch: true
  service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/i18n-backend
```

### Traffic Splitting (Weighted Routing)

**Canary Deployment**:
```yaml
name: canary-url-map
defaultService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/stable-backend

hostRules:
- hosts: ['example.com']
  pathMatcher: canary-matcher

pathMatchers:
- name: canary-matcher
  routeRules:
  - priority: 1
    matchRules:
    - prefixMatch: /
    routeAction:
      weightedBackendServices:
      - backendService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/stable-backend
        weight: 95
      - backendService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/canary-backend
        weight: 5
```

**A/B Testing**:
```yaml
routeRules:
- priority: 1
  matchRules:
  - prefixMatch: /
  routeAction:
    weightedBackendServices:
    - backendService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/variant-a
      weight: 50
      headerAction:
        responseHeadersToAdd:
        - headerName: X-Variant
          headerValue: A
    - backendService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/variant-b
      weight: 50
      headerAction:
        responseHeadersToAdd:
        - headerName: X-Variant
          headerValue: B
```

**Blue-Green Deployment**:
```bash
# Initially: 100% blue
gcloud compute url-maps import web-url-map \
  --source=blue-green-100-0.yaml \
  --global

# Shift to 50/50
gcloud compute url-maps import web-url-map \
  --source=blue-green-50-50.yaml \
  --global

# Complete migration: 100% green
gcloud compute url-maps import web-url-map \
  --source=blue-green-0-100.yaml \
  --global
```

### Traffic Mirroring

Send copy of traffic to another backend for testing without affecting production.

```yaml
routeRules:
- priority: 1
  matchRules:
  - prefixMatch: /api/
  service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/prod-backend
  routeAction:
    requestMirrorPolicy:
      backendService: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/shadow-backend
      # Mirror 10% of production traffic
      weight: 0.1
```

**Mirroring Use Cases**:
```
- Test new backend version with production traffic
- Debug production issues in isolated environment
- Performance testing with real traffic patterns
- Security testing without user impact
- Response from mirrored backend is discarded
```

### Request/Response Transformation

```yaml
routeRules:
- priority: 1
  matchRules:
  - prefixMatch: /api/
  service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/api-backend
  routeAction:
    # Add request headers
    requestHeadersToAdd:
    - headerName: X-Custom-Header
      headerValue: custom-value
      replace: true

    # Remove request headers
    requestHeadersToRemove:
    - X-Internal-Header

    # Add response headers
    responseHeadersToAdd:
    - headerName: X-Frame-Options
      headerValue: DENY
      replace: false

    # Remove response headers
    responseHeadersToRemove:
    - X-Powered-By

    # URL rewrite
    urlRewrite:
      pathPrefixRewrite: /v2/api/
      hostRewrite: internal-api.example.com
```

### Timeouts and Retries

```yaml
routeRules:
- priority: 1
  matchRules:
  - prefixMatch: /api/
  service: https://www.googleapis.com/compute/v1/projects/PROJECT/global/backendServices/api-backend
  routeAction:
    # Request timeout
    timeout: 30s

    # Retry policy
    retryPolicy:
      numRetries: 3
      retryConditions:
      - 5xx
      - refused-stream
      - cancelled
      perTryTimeout: 10s

    # Fault injection (for testing)
    faultInjectionPolicy:
      abort:
        httpStatus: 503
        percentage: 1.0  # 1% of requests
      delay:
        fixedDelay: 5s
        percentage: 5.0  # 5% of requests
```

## Performance Optimization

### Premium vs Standard Network Tier

**Premium Tier**:
```bash
# Create address with Premium tier (default)
gcloud compute addresses create premium-ip \
  --network-tier=PREMIUM \
  --global

# Premium tier features:
# - Google's global network (low latency)
# - Anycast IP addresses
# - Global load balancing
# - Lower latency worldwide
# - Higher cost
```

**Standard Tier**:
```bash
# Create address with Standard tier
gcloud compute addresses create standard-ip \
  --network-tier=STANDARD \
  --region=us-central1

# Standard tier features:
# - Regional routing only
# - Internet routing to region
# - Regional load balancing only
# - Higher latency from distant locations
# - Lower cost (~40% cheaper)
```

**Network Tier Comparison**:
```
                       Premium                    Standard
Traffic Path:      Client → Google PoP        Client → ISP
                   → Google Network           → Internet
                   → GCP Region               → GCP Region

Global LB:         Yes                        No (regional only)
Anycast IP:        Yes                        No
Latency:           Low (optimized routing)    Variable
Cost:              Higher                     Lower
Use Case:          Global applications        Regional applications
                   Latency-sensitive          Cost-sensitive
```

**Exam Tip**: Premium tier traffic enters Google's network at the nearest PoP. Standard tier traffic traverses the internet until it reaches the destination region.

### Cloud Interconnect for Low Latency

**Dedicated Interconnect**:
```
Direct physical connection: 10 Gbps or 100 Gbps
Latency: <2ms (typical within region)
Use: Hybrid load balancing, on-premises to GCP
```

**Partner Interconnect**:
```
Via service provider: 50 Mbps to 50 Gbps
Latency: <5ms (typical)
Use: Smaller bandwidth requirements
```

**Hybrid Load Balancing Architecture**:
```
                Internet Users
                      ↓
           External Global LB (Anycast IP)
                      ↓
            ┌─────────┴─────────┐
            ↓                   ↓
      GCP Backends        On-Premises Backends
      (us-central1)       (via Interconnect)
            ↓                   ↓
      Instance Groups     Internal LB → Servers
```

**Configuration**:
```bash
# Create NEG for on-premises endpoints
gcloud compute network-endpoint-groups create onprem-neg \
  --network-endpoint-type=NON_GCP_PRIVATE_IP_PORT \
  --zone=us-central1-a \
  --network=default \
  --subnet=default

# Add on-premises endpoints
gcloud compute network-endpoint-groups update onprem-neg \
  --add-endpoint="ip=192.168.1.10,port=80" \
  --add-endpoint="ip=192.168.1.11,port=80" \
  --zone=us-central1-a

# Add to backend service
gcloud compute backend-services add-backend hybrid-backend \
  --network-endpoint-group=onprem-neg \
  --network-endpoint-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=100 \
  --global
```

### TCP Optimization

```bash
# TCP proxy load balancers automatically provide:
# - TCP connection pooling
# - TCP window scaling
# - TCP Fast Open (TFO)
# - Congestion control optimization

# Enable TCP proxy for global TCP load balancing
gcloud compute backend-services create tcp-optimized \
  --protocol=TCP \
  --health-checks=tcp-health \
  --timeout=30s \
  --connection-draining-timeout=300s \
  --global

gcloud compute target-tcp-proxies create tcp-proxy \
  --backend-service=tcp-optimized \
  --proxy-header=PROXY_V1

# Proxy header forwards original client IP to backend
```

**TCP Optimization Benefits**:
```
Connection Pooling:
- Reuse TCP connections to backends
- Reduce connection establishment overhead
- Lower latency for subsequent requests

TCP Window Scaling:
- Larger TCP windows
- Better throughput on high-latency links
- Improved performance for large transfers

TCP Fast Open:
- Data in SYN packet
- Reduce handshake latency
- 1 RTT saved per connection
```

### Connection Pooling and Keep-Alive

```bash
# Backend service connection settings
gcloud compute backend-services update web-backend \
  --connection-draining-timeout=300 \
  --timeout=30s \
  --global

# For HTTP/2: connection pooling is automatic
# For HTTP/1.1: configure keep-alive on backends

# Backend nginx configuration:
# keepalive_timeout 75s;
# keepalive_requests 1000;
```

**HTTP/2 Benefits**:
```
- Multiplexing: Multiple requests per connection
- Header compression: Reduced overhead
- Server push: Proactive resource delivery
- Binary protocol: More efficient parsing
```

### Network Endpoint Groups (NEGs)

NEGs provide fine-grained control over load balancing endpoints.

**Zonal NEG (GCE_VM_IP_PORT)**:
```bash
# Create zonal NEG
gcloud compute network-endpoint-groups create web-neg \
  --network-endpoint-type=GCE_VM_IP_PORT \
  --zone=us-central1-a \
  --network=default \
  --subnet=default

# Add endpoints (VM instance + port)
gcloud compute network-endpoint-groups update web-neg \
  --add-endpoint="instance=web-vm-1,port=8080" \
  --add-endpoint="instance=web-vm-2,port=8080" \
  --zone=us-central1-a

# Add NEG to backend service
gcloud compute backend-services add-backend web-backend \
  --network-endpoint-group=web-neg \
  --network-endpoint-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=100 \
  --global
```

**Serverless NEG (Cloud Run, Cloud Functions, App Engine)**:
```bash
# Cloud Run NEG
gcloud compute network-endpoint-groups create run-neg \
  --region=us-central1 \
  --network-endpoint-type=SERVERLESS \
  --cloud-run-service=my-service

# Cloud Functions NEG
gcloud compute network-endpoint-groups create function-neg \
  --region=us-central1 \
  --network-endpoint-type=SERVERLESS \
  --cloud-function-name=my-function

# App Engine NEG
gcloud compute network-endpoint-groups create appengine-neg \
  --region=us-central1 \
  --network-endpoint-type=SERVERLESS \
  --app-engine-app \
  --app-engine-service=default \
  --app-engine-version=v1

# Add serverless NEG to backend
gcloud compute backend-services add-backend web-backend \
  --network-endpoint-group=run-neg \
  --network-endpoint-group-region=us-central1 \
  --global
```

**Internet NEG (External Endpoints)**:
```bash
# For external services outside GCP
gcloud compute network-endpoint-groups create external-neg \
  --network-endpoint-type=INTERNET_FQDN_PORT \
  --global

# Add external endpoints
gcloud compute network-endpoint-groups update external-neg \
  --add-endpoint="fqdn=api.example.com,port=443" \
  --global

gcloud compute backend-services add-backend external-backend \
  --network-endpoint-group=external-neg \
  --global-network-endpoint-group \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=1000 \
  --global
```

**Hybrid Connectivity NEG (On-Premises)**:
```bash
# For on-premises endpoints via VPN/Interconnect
gcloud compute network-endpoint-groups create hybrid-neg \
  --network-endpoint-type=NON_GCP_PRIVATE_IP_PORT \
  --zone=us-central1-a \
  --network=default \
  --subnet=default

# Add on-premises IPs
gcloud compute network-endpoint-groups update hybrid-neg \
  --add-endpoint="ip=10.1.0.5,port=80" \
  --add-endpoint="ip=10.1.0.6,port=80" \
  --zone=us-central1-a
```

**NEG Use Cases**:
```
Zonal NEG (GCE_VM_IP_PORT):
- Container workloads (GKE pods)
- Multiple services per VM (different ports)
- Fine-grained endpoint control

Serverless NEG:
- Cloud Run services
- Cloud Functions
- App Engine
- Unified load balancing across compute types

Internet NEG:
- Multi-cloud backends
- SaaS endpoints
- CDN origins
- Legacy systems

Hybrid NEG:
- On-premises servers
- Datacenter migration
- Hybrid cloud deployments
```

## Load Balancing Scenarios

### Scenario 1: Global E-Commerce Website

**Requirements**:
- Global users, latency <100ms
- Static content (images, CSS, JS)
- Dynamic API backend
- SSL/TLS encryption
- DDoS protection
- 99.99% availability

**Solution**:
```bash
# 1. Create health check
gcloud compute health-checks create http ecommerce-health \
  --port=80 \
  --request-path=/healthz \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2 \
  --global

# 2. Create backend services
# Static content backend with CDN
gcloud compute backend-services create static-backend \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=ecommerce-health \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --default-ttl=86400 \
  --max-ttl=604800 \
  --client-ttl=86400 \
  --global

# API backend (no CDN)
gcloud compute backend-services create api-backend \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=ecommerce-health \
  --session-affinity=GENERATED_COOKIE \
  --affinity-cookie-ttl=3600 \
  --global

# 3. Add multi-region backends
for region in us-central1 europe-west1 asia-east1; do
  gcloud compute backend-services add-backend static-backend \
    --instance-group=static-ig-$region \
    --instance-group-zone=${region}-a \
    --balancing-mode=UTILIZATION \
    --max-utilization=0.8 \
    --global

  gcloud compute backend-services add-backend api-backend \
    --instance-group=api-ig-$region \
    --instance-group-zone=${region}-a \
    --balancing-mode=RATE \
    --max-rate-per-instance=100 \
    --global
done

# 4. Create security policy
gcloud compute security-policies create ecommerce-security \
  --description="E-commerce security policy"

# Add WAF rules
gcloud compute security-policies rules create 1000 \
  --security-policy=ecommerce-security \
  --expression="evaluatePreconfiguredWaf('sqli-v33-stable')" \
  --action=deny-403

gcloud compute security-policies rules create 1100 \
  --security-policy=ecommerce-security \
  --expression="evaluatePreconfiguredWaf('xss-v33-stable')" \
  --action=deny-403

# Rate limiting
gcloud compute security-policies rules create 2000 \
  --security-policy=ecommerce-security \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=200 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP

# Apply to backend
gcloud compute backend-services update api-backend \
  --security-policy=ecommerce-security \
  --global

# 5. Create URL map with path routing
gcloud compute url-maps import ecommerce-url-map \
  --source=ecommerce-url-map.yaml \
  --global

# 6. SSL certificate
gcloud compute ssl-certificates create ecommerce-cert \
  --domains=www.example.com,example.com \
  --global

# 7. Target HTTPS proxy
gcloud compute target-https-proxies create ecommerce-https-proxy \
  --url-map=ecommerce-url-map \
  --ssl-certificates=ecommerce-cert \
  --global

# 8. Global forwarding rule
gcloud compute addresses create ecommerce-ip \
  --network-tier=PREMIUM \
  --global

gcloud compute forwarding-rules create ecommerce-https-rule \
  --address=ecommerce-ip \
  --global \
  --target-https-proxy=ecommerce-https-proxy \
  --ports=443
```

### Scenario 2: Internal Microservices Architecture

**Requirements**:
- Private internal services
- Service mesh communication
- Multiple regions
- No internet exposure
- Circuit breaking

**Solution**:
```bash
# 1. Create proxy-only subnet
gcloud compute networks subnets create proxy-subnet \
  --purpose=REGIONAL_MANAGED_PROXY \
  --role=ACTIVE \
  --region=us-central1 \
  --network=default \
  --range=10.129.0.0/23

# 2. Create internal health check
gcloud compute health-checks create http internal-health \
  --region=us-central1 \
  --port=8080 \
  --request-path=/healthz

# 3. Create internal backend services per microservice
for service in user order payment inventory; do
  gcloud compute backend-services create ${service}-backend \
    --load-balancing-scheme=INTERNAL_MANAGED \
    --protocol=HTTP \
    --health-checks=internal-health \
    --health-checks-region=us-central1 \
    --region=us-central1

  # Add NEG (for GKE pods)
  gcloud compute backend-services add-backend ${service}-backend \
    --network-endpoint-group=${service}-neg \
    --network-endpoint-group-zone=us-central1-a \
    --balancing-mode=RATE \
    --max-rate-per-endpoint=100 \
    --region=us-central1
done

# 4. Create URL map for service routing
gcloud compute url-maps import microservices-url-map \
  --source=microservices-routing.yaml \
  --region=us-central1

# 5. Target HTTP proxy
gcloud compute target-http-proxies create internal-proxy \
  --url-map=microservices-url-map \
  --region=us-central1

# 6. Internal forwarding rule
gcloud compute addresses create internal-lb-ip \
  --region=us-central1 \
  --subnet=default \
  --purpose=SHARED_LOADBALANCER_VPC

gcloud compute forwarding-rules create internal-lb-rule \
  --load-balancing-scheme=INTERNAL_MANAGED \
  --network=default \
  --subnet=default \
  --address=internal-lb-ip \
  --ports=80 \
  --region=us-central1 \
  --target-http-proxy=internal-proxy
```

### Scenario 3: Gaming Application (UDP)

**Requirements**:
- UDP protocol (game servers)
- Client IP preservation
- Regional deployment
- Session persistence
- Fast failover

**Solution**:
```bash
# 1. Create UDP health check
gcloud compute health-checks create tcp game-health \
  --port=7777 \
  --check-interval=5s \
  --timeout=3s \
  --unhealthy-threshold=2 \
  --healthy-threshold=2 \
  --region=us-central1

# 2. Create backend service
gcloud compute backend-services create game-backend \
  --protocol=UDP \
  --health-checks=game-health \
  --health-checks-region=us-central1 \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL \
  --session-affinity=CLIENT_IP

# 3. Add game server instance groups
gcloud compute backend-services add-backend game-backend \
  --instance-group=game-servers-ig \
  --instance-group-zone=us-central1-a \
  --region=us-central1

# 4. Create forwarding rule
gcloud compute addresses create game-lb-ip \
  --region=us-central1

gcloud compute forwarding-rules create game-lb-rule \
  --region=us-central1 \
  --load-balancing-scheme=EXTERNAL \
  --address=game-lb-ip \
  --ip-protocol=UDP \
  --ports=7777-7877 \
  --backend-service=game-backend
```

### Scenario 4: Hybrid Cloud Migration

**Requirements**:
- Gradual migration from on-premises to GCP
- On-premises via Interconnect
- Traffic splitting (canary)
- Zero downtime
- Maintain existing IPs

**Solution**:
```bash
# 1. Create NEGs for both environments
# GCP NEG
gcloud compute network-endpoint-groups create gcp-neg \
  --network-endpoint-type=GCE_VM_IP_PORT \
  --zone=us-central1-a \
  --network=default \
  --subnet=default

# On-premises NEG
gcloud compute network-endpoint-groups create onprem-neg \
  --network-endpoint-type=NON_GCP_PRIVATE_IP_PORT \
  --zone=us-central1-a \
  --network=default \
  --subnet=default

# Add on-premises endpoints
gcloud compute network-endpoint-groups update onprem-neg \
  --add-endpoint="ip=10.1.0.10,port=80" \
  --add-endpoint="ip=10.1.0.11,port=80" \
  --zone=us-central1-a

# 2. Create backend services
gcloud compute backend-services create gcp-backend \
  --protocol=HTTP \
  --health-checks=hybrid-health \
  --global

gcloud compute backend-services create onprem-backend \
  --protocol=HTTP \
  --health-checks=hybrid-health \
  --global

# 3. Add NEGs to backends
gcloud compute backend-services add-backend gcp-backend \
  --network-endpoint-group=gcp-neg \
  --network-endpoint-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=100 \
  --global

gcloud compute backend-services add-backend onprem-backend \
  --network-endpoint-group=onprem-neg \
  --network-endpoint-group-zone=us-central1-a \
  --balancing-mode=RATE \
  --max-rate-per-endpoint=50 \
  --global

# 4. Create URL map with weighted traffic (10% to GCP)
gcloud compute url-maps import migration-url-map \
  --source=migration-canary.yaml \
  --global

# migration-canary.yaml:
# routeRules:
# - priority: 1
#   matchRules:
#   - prefixMatch: /
#   routeAction:
#     weightedBackendServices:
#     - backendService: projects/PROJECT/global/backendServices/onprem-backend
#       weight: 90
#     - backendService: projects/PROJECT/global/backendServices/gcp-backend
#       weight: 10

# Gradually increase GCP weight: 10% → 25% → 50% → 75% → 100%
```

### Scenario 5: API with Rate Limiting and Authentication

**Requirements**:
- REST API
- API key authentication
- Rate limiting per user
- Geographic restrictions
- Logging

**Solution**:
```bash
# 1. Create security policy
gcloud compute security-policies create api-security \
  --description="API security policy"

# Require API key
gcloud compute security-policies rules create 100 \
  --security-policy=api-security \
  --expression="!has(request.headers['x-api-key'])" \
  --action=deny-401 \
  --description="Require API key"

# Geographic restriction
gcloud compute security-policies rules create 200 \
  --security-policy=api-security \
  --expression="origin.region_code != 'US' && origin.region_code != 'CA' && origin.region_code != 'GB'" \
  --action=deny-403 \
  --description="Allow US, CA, GB only"

# Rate limiting per API key
gcloud compute security-policies rules create 1000 \
  --security-policy=api-security \
  --expression="has(request.headers['x-api-key'])" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=1000 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=300 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=HTTP-HEADER \
  --enforce-on-key-name=X-API-Key \
  --description="Rate limit per API key"

# 2. Create backend with security policy
gcloud compute backend-services create api-backend \
  --protocol=HTTP \
  --health-checks=api-health \
  --security-policy=api-security \
  --enable-logging \
  --logging-sample-rate=1.0 \
  --global
```

## Troubleshooting Load Balancers

### Common Issues and Resolution

**503 Service Unavailable**:
```bash
# Check backend health
gcloud compute backend-services get-health web-backend --global

# Common causes:
# 1. All backends unhealthy
# 2. Health check misconfiguration
# 3. Firewall blocking health check probes
# 4. Backend capacity exceeded

# Solution: Fix health check or add capacity
# Health check firewall rule
gcloud compute firewall-rules create allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=35.191.0.0/16,130.211.0.0/22 \
  --rules=tcp:80,tcp:443
```

**502 Bad Gateway**:
```bash
# Causes:
# - Backend returning invalid HTTP response
# - Backend timeout
# - Connection refused by backend

# Check backend logs
gcloud logging read 'resource.type="http_load_balancer"
  httpRequest.status=502' \
  --limit=50 \
  --format=json

# Solution: Check backend application, increase timeout
gcloud compute backend-services update web-backend \
  --timeout=60s \
  --global
```

**High Latency**:
```bash
# Check latency metrics
gcloud monitoring time-series list \
  --filter='metric.type="loadbalancing.googleapis.com/https/backend_latencies"' \
  --format=json

# Possible causes:
# - Wrong network tier (Standard instead of Premium)
# - Distant backend location
# - Backend performance issues
# - No CDN for static content

# Enable CDN
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --global

# Check network tier
gcloud compute addresses describe lb-ip --global --format="get(networkTier)"
```

**SSL Certificate Provisioning Failed**:
```bash
# Check certificate status
gcloud compute ssl-certificates describe cert-name \
  --global \
  --format=yaml

# Common issues:
# 1. DNS not pointing to load balancer IP
# 2. CAA record blocking Google CA
# 3. Domain validation timeout

# Verify DNS
dig example.com

# Solution: Update DNS A record to LB IP
# Wait 10-60 minutes for provisioning
```

**Session Affinity Not Working**:
```bash
# Check session affinity configuration
gcloud compute backend-services describe web-backend \
  --global \
  --format="get(sessionAffinity)"

# Causes:
# - Backend becoming unhealthy breaks affinity
# - CDN caching interfering
# - Client IP changing (mobile networks)

# Solution: Use GENERATED_COOKIE instead of CLIENT_IP
gcloud compute backend-services update web-backend \
  --session-affinity=GENERATED_COOKIE \
  --affinity-cookie-ttl=3600 \
  --global
```

**CDN Not Caching Content**:
```bash
# Check CDN status
gcloud compute backend-services describe web-backend \
  --global \
  --format="get(cdnPolicy)"

# Test cache hit/miss
curl -I https://example.com/image.jpg | grep -i "cache"
# Look for: x-cache: HIT or MISS

# Causes:
# - Cache-Control: no-cache headers from origin
# - Query strings not included in cache key
# - Cookies in request

# Solution: Configure cache key
gcloud compute backend-services update web-backend \
  --cache-key-include-query-string \
  --cache-key-query-string-whitelist=version \
  --global
```

**Uneven Backend Traffic Distribution**:
```bash
# Check backend metrics
gcloud monitoring time-series list \
  --filter='metric.type="loadbalancing.googleapis.com/https/request_count"' \
  --format=json

# Causes:
# - Session affinity creating hot spots
# - Capacity scaler misconfigured
# - One backend has lower max-utilization

# Solution: Adjust balancing configuration
gcloud compute backend-services update-backend web-backend \
  --instance-group=ig-name \
  --instance-group-zone=us-central1-a \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --capacity-scaler=1.0 \
  --global
```

### Logging and Monitoring

**Enable Load Balancer Logging**:
```bash
# Enable HTTP(S) LB logging
gcloud compute backend-services update web-backend \
  --enable-logging \
  --logging-sample-rate=1.0 \
  --global

# Query logs
gcloud logging read 'resource.type="http_load_balancer"
  httpRequest.status>=400' \
  --limit=50 \
  --format=json

# Useful log fields:
# - httpRequest.requestUrl
# - httpRequest.status
# - httpRequest.latency
# - jsonPayload.cacheHit (CDN)
# - jsonPayload.statusDetails
```

**Key Metrics to Monitor**:
```bash
# Request count
# Metric: loadbalancing.googleapis.com/https/request_count

# Backend latency
# Metric: loadbalancing.googleapis.com/https/backend_latencies

# Request bytes
# Metric: loadbalancing.googleapis.com/https/request_bytes_count

# Response bytes
# Metric: loadbalancing.googleapis.com/https/response_bytes_count

# Backend health
# Metric: loadbalancing.googleapis.com/https/backend_health

# CDN cache hit ratio
# Metric: loadbalancing.googleapis.com/https/cache_hit_ratio

# Create alert policy
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High 5xx rate" \
  --condition-display-name="5xx errors >5%" \
  --condition-threshold-value=0.05 \
  --condition-threshold-duration=300s \
  --condition-filter='resource.type="http_load_balancer"
    AND metric.type="loadbalancing.googleapis.com/https/request_count"
    AND metric.label.response_code_class="500"'
```

## Professional Network Engineer Exam Tips

### Load Balancer Selection Decision Tree

```
Question: Which load balancer should I use?

Is traffic HTTP/HTTPS?
  Yes → Is it internal or external?
    External → Need global distribution?
      Yes → External Application LB (Global) + CDN
      No → External Application LB (Regional)
    Internal → Need Layer 7 routing?
      Yes → Internal Application LB
      No → Internal Network LB (Passthrough)

  No → Is it TCP or UDP?
    TCP → Is it internal or external?
      External → Need global distribution?
        Yes → TCP Proxy LB (Global)
        No → External Network LB (Regional)
      Internal → Need connection pooling?
        Yes → Internal Network LB (Proxy)
        No → Internal Network LB (Passthrough)

    UDP → Must be External or Internal Network LB (Passthrough)
          (Regional only, preserves client IP)
```

### Key Exam Topics

**Load Balancer Components**:
```
Always remember the flow:
Forwarding Rule → Target Proxy → URL Map → Backend Service → Backends

Forwarding Rule:
- IP address + port
- Entry point for traffic
- Global or regional

Target Proxy:
- HTTP, HTTPS, TCP, SSL
- SSL termination here
- Associates with URL map

URL Map:
- Routing rules (paths, hosts, headers)
- Traffic splitting
- Redirects

Backend Service:
- Health checks
- Balancing mode
- Session affinity
- CDN configuration
- Security policies

Backends:
- Instance groups
- Network endpoint groups (NEGs)
- Capacity and utilization settings
```

**Common Exam Scenarios**:

1. **Choose between Premium and Standard tier**:
   - Premium: Global reach, anycast IP, lower latency, higher cost
   - Standard: Regional only, no anycast, higher latency, lower cost
   - Exam tip: Premium for global apps, Standard for regional cost optimization

2. **When to use Cloud Armor**:
   - DDoS protection (always mention for public-facing apps)
   - WAF rules (SQL injection, XSS)
   - Rate limiting
   - Geographic restrictions
   - Only works with External Application LB and Internal Application LB

3. **CDN configuration**:
   - Enable CDN on backend service
   - Choose cache mode (CACHE_ALL_STATIC vs USE_ORIGIN_HEADERS)
   - Configure cache keys (query strings, cookies)
   - Signed URLs for private content
   - Cache invalidation vs versioned URLs

4. **Session affinity requirements**:
   - CLIENT_IP: Simple, but can create hot spots
   - GENERATED_COOKIE: Better distribution, requires HTTP/HTTPS
   - Stateless design with external session store is preferred

5. **Health check configuration**:
   - Must create firewall rule for health check ranges: 35.191.0.0/16, 130.211.0.0/22
   - Interval × unhealthy_threshold = time to mark unhealthy
   - Configure appropriate thresholds to avoid false positives

6. **SSL certificate management**:
   - Google-managed: Auto-renewal, requires DNS pointing to LB
   - Self-managed: You manage renewal
   - Certificate Manager: Multiple certs, certificate maps
   - SSL policies: MODERN (TLS 1.2+) recommended

7. **Traffic splitting for deployments**:
   - Canary: 95% stable, 5% new
   - Blue-green: Gradual shift 100→50→0
   - A/B testing: 50/50 with different backends

8. **Internal vs External LBs**:
   - Internal: Private IPs, within VPC, no internet exposure
   - External: Public IPs, internet-facing
   - Requires proxy-only subnet for Internal Application LB

9. **NEG use cases**:
   - Serverless NEG: Cloud Run, Functions, App Engine
   - Zonal NEG: GKE pods, multiple ports per VM
   - Internet NEG: External endpoints, multi-cloud
   - Hybrid NEG: On-premises via VPN/Interconnect

10. **Cross-region failover**:
    - Global LB automatically routes to nearest healthy backend
    - Configure failover backends with --failover flag
    - Set failover-ratio for capacity threshold
    - Health checks determine backend availability

### Exam Question Patterns

**Pattern 1: Best Load Balancer for Scenario**:
```
Question: "Company needs to load balance HTTP traffic across VMs in
multiple regions with a single anycast IP address. Which LB?"

Answer: External Application LB (Global)
- HTTP traffic → Application LB
- Multiple regions + anycast IP → Global
- Alternative regional LB doesn't support anycast
```

**Pattern 2: Troubleshooting**:
```
Question: "Users report 503 errors. Health checks pass. What's the issue?"

Common causes:
- Firewall blocking LB to backend traffic
- Backend capacity exhausted (check max-utilization)
- Connection draining in progress
- Security policy blocking traffic

Check: Backend service get-health, firewall rules, capacity scaler
```

**Pattern 3: Security Requirements**:
```
Question: "Need to protect API from DDoS and limit to 100 req/min per IP"

Answer: Cloud Armor security policy
- Create security policy
- Add rate limiting rule (100 req/60s, enforce-on-key=IP)
- Attach to backend service
- Enable Adaptive Protection for DDoS
```

**Pattern 4: Performance Optimization**:
```
Question: "Global users experience high latency for static content"

Answer: Enable Cloud CDN
- Static content should be cached
- Enable CDN on backend service
- Set appropriate TTLs
- Configure cache keys to exclude tracking parameters
- Use Premium tier for best performance
```

**Pattern 5: Migration Strategy**:
```
Question: "Migrate on-premises app to GCP with zero downtime"

Answer: Hybrid load balancing with traffic splitting
- Create NEGs for both on-prem and GCP
- On-prem NEG via VPN/Interconnect
- Use weighted routing: Start 100% on-prem
- Gradually shift: 90/10 → 70/30 → 50/50 → 100% GCP
- Monitor and rollback if issues
```

### Important Limits and Quotas

```
Global External Application LB:
- Backends per backend service: 50 instance groups or 500 NEG endpoints
- URL maps per project: 50
- SSL certificates per target proxy: 15
- Forwarding rules per project: 50 (global)

Backend Service:
- Max backend timeout: 86400s (24 hours)
- Max connection draining timeout: 3600s (1 hour)
- Health check interval: 1-300s

Cloud Armor:
- Security policies per project: 100
- Rules per policy: 200
- Rate limiting rules: 20 per policy

Cloud CDN:
- Cache invalidation: 1000 free per month
- Max cache entry size: 10 MB (default), 50 MB (configurable)
- Signed URL key: 128-bit base64url encoded
```

### Best Practices for Exam

1. **Always consider global vs regional first**
   - Global: Multi-region, anycast IP, worldwide users
   - Regional: Single region, regional IP, lower cost

2. **Layer 4 vs Layer 7**
   - Layer 7 (HTTP/HTTPS): Content-based routing, URL maps, headers
   - Layer 4 (TCP/UDP): Port-based, faster, simpler

3. **Internal vs External**
   - External: Internet-facing, public IP
   - Internal: VPC-internal, private IP

4. **Enable security and monitoring**
   - Cloud Armor for DDoS/WAF
   - Logging for troubleshooting
   - Alerts for SLOs

5. **Use managed services**
   - Google-managed SSL certificates over self-managed
   - Adaptive Protection for DDoS
   - Serverless NEGs for Cloud Run/Functions

6. **High availability design**
   - Multi-region backends
   - Health checks configured properly
   - Connection draining enabled
   - Failover backends for critical apps

7. **Cost optimization**
   - Standard tier for regional workloads
   - CDN cache to reduce origin traffic
   - Right-size backend capacity
   - Use committed use discounts

## Additional Resources

- [Load Balancing Documentation](https://cloud.google.com/load-balancing/docs)
- [Cloud CDN Documentation](https://cloud.google.com/cdn/docs)
- [Cloud Armor Documentation](https://cloud.google.com/armor/docs)
- [Network Performance Guide](https://cloud.google.com/architecture/best-practices-for-optimizing-network-performance)
- [SSL Certificates Documentation](https://cloud.google.com/load-balancing/docs/ssl-certificates)
- [Network Intelligence Center](https://cloud.google.com/network-intelligence-center/docs)

## Summary

This comprehensive guide covers all load balancing and performance topics for the Professional Cloud Network Engineer exam:

1. **Load Balancer Types**: Seven distinct types with complete decision matrix
2. **Global Load Balancing**: Anycast IPs, cross-region failover, backend configuration
3. **Cloud CDN**: Cache modes, cache keys, signed URLs, invalidation, custom origins
4. **Cloud Armor**: Security policies, WAF rules, rate limiting, Adaptive Protection, bot management
5. **SSL/TLS**: Certificate types, SSL policies, HTTPS redirects, mutual TLS
6. **Traffic Management**: URL maps, routing rules, traffic splitting, mirroring, transformations
7. **Performance**: Premium/Standard tiers, Cloud Interconnect, TCP optimization, NEGs
8. **Real-World Scenarios**: E-commerce, microservices, gaming, hybrid cloud, API security
9. **Troubleshooting**: Common issues, logging, monitoring, health checks
10. **Exam Tips**: Decision trees, question patterns, limits, best practices

Key takeaways for the exam:
- Understand the load balancer selection criteria (Layer 4 vs 7, internal vs external, global vs regional)
- Know when to use Cloud Armor and Cloud CDN
- Master NEG types and use cases
- Understand health check ranges and firewall requirements (35.191.0.0/16, 130.211.0.0/22)
- Remember the component flow: Forwarding Rule → Target Proxy → URL Map → Backend Service → Backends
- Know the difference between Premium and Standard network tiers
- Understand traffic splitting for canary deployments and blue-green migrations
- Master security policy configuration and rate limiting

Practice creating complete load balancer configurations with all components for different scenarios. Good luck on your Professional Cloud Network Engineer exam!
