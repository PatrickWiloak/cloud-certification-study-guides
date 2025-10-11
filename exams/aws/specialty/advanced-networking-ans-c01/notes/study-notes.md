# AWS Advanced Networking Specialty - Comprehensive Study Notes

## Quick Reference Guide

### VPC Networking Essentials

#### VPC and Subnet Management
```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=prod-vpc}]'

# Create subnet
aws ec2 create-subnet --vpc-id vpc-xxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1a}]'

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute --subnet-id subnet-xxx \
  --map-public-ip-on-launch

# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=prod-igw}]'

# Attach IGW to VPC
aws ec2 attach-internet-gateway --vpc-id vpc-xxx --internet-gateway-id igw-xxx

# Create NAT Gateway
aws ec2 create-nat-gateway --subnet-id subnet-xxx \
  --allocation-id eipalloc-xxx \
  --tag-specifications 'ResourceType=nat-gateway,Tags=[{Key=Name,Value=nat-1a}]'
```

#### Route Table Management
```bash
# Create route table
aws ec2 create-route-table --vpc-id vpc-xxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]'

# Add route
aws ec2 create-route --route-table-id rtb-xxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxx

# Associate route table with subnet
aws ec2 associate-route-table --subnet-id subnet-xxx \
  --route-table-id rtb-xxx

# Create VPC endpoint route (for S3, DynamoDB)
aws ec2 create-route --route-table-id rtb-xxx \
  --destination-prefix-list-id pl-xxx \
  --vpc-endpoint-id vpce-xxx
```

#### Security Groups and NACLs
```bash
# Create security group
aws ec2 create-security-group --group-name web-sg \
  --description "Web servers security group" \
  --vpc-id vpc-xxx

# Add inbound rule
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Add outbound rule (needed for stateless NACLs)
aws ec2 authorize-security-group-egress --group-id sg-xxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Create NACL entry
aws ec2 create-network-acl-entry --network-acl-id acl-xxx \
  --rule-number 100 --protocol tcp --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 --egress --rule-action allow

# Create NACL for ephemeral ports (IMPORTANT!)
aws ec2 create-network-acl-entry --network-acl-id acl-xxx \
  --rule-number 200 --protocol tcp --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0 --egress --rule-action allow
```

### VPC Endpoints and PrivateLink

#### VPC Endpoints
```bash
# Create Gateway Endpoint (S3, DynamoDB)
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxx

# Create Interface Endpoint (most AWS services)
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --service-name com.amazonaws.us-east-1.ec2 \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-xxx subnet-yyy \
  --security-group-ids sg-xxx

# Enable private DNS for interface endpoint
aws ec2 modify-vpc-endpoint --vpc-endpoint-id vpce-xxx \
  --private-dns-enabled

# Create endpoint for S3 with policy
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --policy-document file://endpoint-policy.json
```

#### PrivateLink (Service Provider)
```bash
# Create endpoint service (for your application)
aws ec2 create-vpc-endpoint-service-configuration \
  --network-load-balancer-arns arn:aws:elasticloadbalancing:region:account:loadbalancer/net/my-nlb/xxx \
  --acceptance-required

# Allow specific AWS accounts/principals
aws ec2 modify-vpc-endpoint-service-permissions \
  --service-id vpce-svc-xxx \
  --add-allowed-principals arn:aws:iam::123456789012:root
```

### Transit Gateway

#### Transit Gateway Setup
```bash
# Create Transit Gateway
aws ec2 create-transit-gateway \
  --description "Regional hub TGW" \
  --options AmazonSideAsn=64512,DefaultRouteTableAssociation=enable,DefaultRouteTablePropagation=enable,DnsSupport=enable

# Create TGW attachment to VPC
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-xxx \
  --vpc-id vpc-xxx \
  --subnet-ids subnet-xxx subnet-yyy \
  --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=prod-vpc-attach}]'

# Create custom TGW route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-xxx \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=prod-rt}]'

# Associate attachment with route table
aws ec2 associate-transit-gateway-route-table \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --transit-gateway-attachment-id tgw-attach-xxx

# Create static route in TGW route table
aws ec2 create-transit-gateway-route \
  --destination-cidr-block 10.1.0.0/16 \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --transit-gateway-attachment-id tgw-attach-xxx

# Enable route propagation
aws ec2 enable-transit-gateway-route-table-propagation \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --transit-gateway-attachment-id tgw-attach-xxx
```

#### Transit Gateway Peering (Cross-Region)
```bash
# Create peering attachment (in primary region)
aws ec2 create-transit-gateway-peering-attachment \
  --transit-gateway-id tgw-xxx \
  --peer-transit-gateway-id tgw-yyy \
  --peer-account-id 123456789012 \
  --peer-region us-west-2

# Accept peering (in peer region)
aws ec2 accept-transit-gateway-peering-attachment \
  --transit-gateway-attachment-id tgw-attach-xxx
```

### Direct Connect

#### Virtual Interface Creation
```bash
# Create private VIF
aws directconnect create-private-virtual-interface \
  --connection-id dxcon-xxx \
  --new-private-virtual-interface \
    virtualInterfaceName=prod-private-vif,\
    vlan=100,\
    asn=65000,\
    authKey=secretkey123,\
    amazonAddress=169.254.255.1/30,\
    customerAddress=169.254.255.2/30,\
    addressFamily=ipv4,\
    directConnectGatewayId=dx-gw-xxx

# Create public VIF (for public AWS services)
aws directconnect create-public-virtual-interface \
  --connection-id dxcon-xxx \
  --new-public-virtual-interface \
    virtualInterfaceName=prod-public-vif,\
    vlan=101,\
    asn=65000,\
    authKey=secretkey456,\
    amazonAddress=169.254.255.5/30,\
    customerAddress=169.254.255.6/30,\
    addressFamily=ipv4,\
    routeFilterPrefixes=[{cidr=203.0.113.0/24}]

# Create transit VIF (for Direct Connect Gateway with TGW)
aws directconnect create-transit-virtual-interface \
  --connection-id dxcon-xxx \
  --new-transit-virtual-interface \
    virtualInterfaceName=prod-transit-vif,\
    vlan=102,\
    asn=65000,\
    directConnectGatewayId=dx-gw-xxx
```

#### Direct Connect Gateway
```bash
# Create Direct Connect Gateway
aws directconnect create-direct-connect-gateway \
  --direct-connect-gateway-name prod-dx-gw \
  --amazon-side-asn 64512

# Associate VGW with DX Gateway
aws directconnect create-direct-connect-gateway-association \
  --direct-connect-gateway-id dx-gw-xxx \
  --virtual-gateway-id vgw-xxx

# Associate TGW with DX Gateway
aws directconnect create-direct-connect-gateway-association \
  --direct-connect-gateway-id dx-gw-xxx \
  --gateway-id tgw-xxx \
  --add-allowed-prefixes-to-direct-connect-gateway cidr=10.0.0.0/8
```

#### Link Aggregation Group (LAG)
```bash
# Create LAG
aws directconnect create-lag \
  --location EqSV5 \
  --number-of-connections 2 \
  --lag-name prod-lag \
  --connections-bandwidth 10Gbps

# Associate connection to LAG
aws directconnect associate-connection-with-lag \
  --connection-id dxcon-xxx \
  --lag-id dxlag-xxx
```

### Site-to-Site VPN

#### VPN Connection Setup
```bash
# Create Customer Gateway
aws ec2 create-customer-gateway \
  --type ipsec.1 \
  --public-ip 203.0.113.1 \
  --bgp-asn 65000 \
  --tag-specifications 'ResourceType=customer-gateway,Tags=[{Key=Name,Value=onprem-cgw}]'

# Create Virtual Private Gateway
aws ec2 create-vpn-gateway --type ipsec.1 \
  --amazon-side-asn 64512 \
  --tag-specifications 'ResourceType=vpn-gateway,Tags=[{Key=Name,Value=prod-vgw}]'

# Attach VGW to VPC
aws ec2 attach-vpn-gateway --vpn-gateway-id vgw-xxx --vpc-id vpc-xxx

# Create VPN connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-xxx \
  --vpn-gateway-id vgw-xxx \
  --options StaticRoutesOnly=false,TunnelOptions=[{TunnelInsideCidr=169.254.10.0/30}]

# Enable route propagation for VPN
aws ec2 enable-vgw-route-propagation \
  --route-table-id rtb-xxx \
  --gateway-id vgw-xxx

# Create VPN with Transit Gateway
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-xxx \
  --transit-gateway-id tgw-xxx \
  --options StaticRoutesOnly=false
```

#### VPN Monitoring
```bash
# Monitor VPN tunnel status
aws ec2 describe-vpn-connections --vpn-connection-ids vpn-xxx \
  --query 'VpnConnections[0].VgwTelemetry'

# Get VPN configuration
aws ec2 describe-vpn-connections --vpn-connection-ids vpn-xxx \
  --query 'VpnConnections[0].CustomerGatewayConfiguration' \
  --output text > vpn-config.xml
```

### Route 53 Advanced Features

#### DNS Query Logging and DNSSEC
```bash
# Create query logging configuration
aws route53 create-query-logging-config \
  --hosted-zone-id Z1234567890ABC \
  --cloud-watch-logs-log-group-arn arn:aws:logs:us-east-1:123456789012:log-group:/aws/route53/example.com

# Enable DNSSEC signing
aws route53 enable-hosted-zone-dnssec \
  --hosted-zone-id Z1234567890ABC

# Create KMS key for DNSSEC
aws kms create-key --key-spec ECC_NIST_P256 \
  --key-usage SIGN_VERIFY \
  --customer-master-key-spec ECC_NIST_P256

# Create key-signing key (KSK)
aws route53 create-key-signing-key \
  --hosted-zone-id Z1234567890ABC \
  --key-management-service-arn arn:aws:kms:us-east-1:123456789012:key/xxx \
  --name prod-ksk \
  --status ACTIVE
```

#### Health Checks and Routing
```bash
# Create health check with string matching
aws route53 create-health-check \
  --health-check-config \
    IPAddress=203.0.113.1,\
    Port=443,\
    Type=HTTPS,\
    ResourcePath=/health,\
    RequestInterval=30,\
    FailureThreshold=3,\
    FullyQualifiedDomainName=example.com,\
    SearchString="healthy"

# Create calculated health check (AND logic)
aws route53 create-health-check \
  --health-check-config \
    Type=CALCULATED,\
    ChildHealthChecks=hc-xxx hc-yyy,\
    HealthThreshold=2

# Create CloudWatch alarm-based health check
aws route53 create-health-check \
  --health-check-config \
    Type=CLOUDWATCH_METRIC,\
    AlarmIdentifier=Region=us-east-1,Name=high-cpu-alarm,\
    InsufficientDataHealthStatus=Healthy
```

#### Traffic Policies (Advanced Routing)
```bash
# Create traffic policy with geolocation + latency routing
# (Use JSON file for complex policies)
aws route53 create-traffic-policy \
  --name multi-region-policy \
  --document file://traffic-policy.json

# Create policy instance
aws route53 create-traffic-policy-instance \
  --hosted-zone-id Z1234567890ABC \
  --name example.com \
  --ttl 300 \
  --traffic-policy-id xxx \
  --traffic-policy-version 1
```

### Route 53 Resolver (Hybrid DNS)

#### Resolver Endpoints
```bash
# Create inbound resolver endpoint (on-premises -> VPC)
aws route53resolver create-resolver-endpoint \
  --creator-request-id $(date +%s) \
  --name prod-inbound-endpoint \
  --direction INBOUND \
  --security-group-ids sg-xxx \
  --ip-addresses SubnetId=subnet-xxx SubnetId=subnet-yyy

# Create outbound resolver endpoint (VPC -> on-premises)
aws route53resolver create-resolver-endpoint \
  --creator-request-id $(date +%s) \
  --name prod-outbound-endpoint \
  --direction OUTBOUND \
  --security-group-ids sg-xxx \
  --ip-addresses SubnetId=subnet-xxx SubnetId=subnet-yyy

# Create resolver rule for conditional forwarding
aws route53resolver create-resolver-rule \
  --creator-request-id $(date +%s) \
  --name forward-to-onprem \
  --rule-type FORWARD \
  --domain-name onprem.example.com \
  --target-ips Ip=10.0.0.10,Port=53 Ip=10.0.0.11,Port=53 \
  --resolver-endpoint-id rslvr-out-xxx

# Associate resolver rule with VPC
aws route53resolver associate-resolver-rule \
  --resolver-rule-id rslvr-rr-xxx \
  --vpc-id vpc-xxx
```

### CloudFront and Global Accelerator

#### CloudFront Distribution
```bash
# Create distribution with custom origin
aws cloudfront create-distribution \
  --distribution-config file://distribution-config.json

# Create origin access identity (for S3)
aws cloudfront create-cloud-front-origin-access-identity \
  --cloud-front-origin-access-identity-config \
    CallerReference=$(date +%s),Comment="OAI for bucket"

# Invalidate cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"

# Create field-level encryption profile
aws cloudfront create-field-level-encryption-profile \
  --field-level-encryption-profile-config file://encryption-profile.json
```

#### Global Accelerator
```bash
# Create accelerator
aws globalaccelerator create-accelerator \
  --name prod-accelerator \
  --ip-address-type IPV4 \
  --enabled

# Create listener
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::123456789012:accelerator/xxx \
  --port-ranges FromPort=80,ToPort=80 FromPort=443,ToPort=443 \
  --protocol TCP

# Create endpoint group
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::123456789012:listener/xxx \
  --endpoint-group-region us-east-1 \
  --traffic-dial-percentage 100 \
  --health-check-interval-seconds 30

# Add endpoint (ALB, NLB, or EC2)
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::123456789012:listener/xxx \
  --endpoint-configurations \
    EndpointId=arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/xxx,Weight=128
```

### VPC Flow Logs Analysis

#### Flow Logs Setup
```bash
# Create flow logs for VPC
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs/prod \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole

# Create flow logs to S3
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxx \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::my-flow-logs-bucket/prefix/

# Custom format flow logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs/custom \
  --log-format '${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}'
```

#### Flow Logs Insights Queries
```sql
-- Find top talkers (most traffic)
fields @timestamp, srcaddr, dstaddr, bytes
| stats sum(bytes) as totalBytes by srcaddr
| sort totalBytes desc
| limit 20

-- Find rejected connections
fields @timestamp, srcaddr, dstaddr, srcport, dstport, action
| filter action = "REJECT"
| sort @timestamp desc
| limit 100

-- SSH brute force detection
fields @timestamp, srcaddr, dstport, action
| filter dstport = 22 and action = "REJECT"
| stats count() as attempts by srcaddr
| filter attempts > 100
| sort attempts desc

-- Data transfer by protocol
fields protocol, bytes
| stats sum(bytes) as totalBytes by protocol
| sort totalBytes desc

-- Outbound traffic analysis
fields @timestamp, dstaddr, dstport, bytes
| filter srcaddr like /^10\./
| stats sum(bytes) as totalBytes by dstaddr, dstport
| sort totalBytes desc
| limit 50
```

### Network Firewall

#### Firewall Deployment
```bash
# Create firewall policy
aws network-firewall create-firewall-policy \
  --firewall-policy-name prod-policy \
  --firewall-policy file://firewall-policy.json

# Create stateless rule group
aws network-firewall create-rule-group \
  --rule-group-name allow-ssh \
  --type STATELESS \
  --rule-group file://stateless-rules.json \
  --capacity 100

# Create stateful rule group (5-tuple)
aws network-firewall create-rule-group \
  --rule-group-name block-malicious-domains \
  --type STATEFUL \
  --rule-group file://stateful-rules.json \
  --capacity 1000

# Create firewall
aws network-firewall create-firewall \
  --firewall-name prod-firewall \
  --firewall-policy-arn arn:aws:network-firewall:us-east-1:123456789012:firewall-policy/prod-policy \
  --vpc-id vpc-xxx \
  --subnet-mappings SubnetId=subnet-xxx SubnetId=subnet-yyy

# Configure logging
aws network-firewall update-logging-configuration \
  --firewall-name prod-firewall \
  --logging-configuration file://logging-config.json
```

### AWS WAF

#### WAF Rules
```bash
# Create IP set
aws wafv2 create-ip-set \
  --name blocked-ips \
  --scope REGIONAL \
  --ip-address-version IPV4 \
  --addresses 203.0.113.0/24 198.51.100.0/24

# Create regex pattern set
aws wafv2 create-regex-pattern-set \
  --name sql-injection-patterns \
  --scope REGIONAL \
  --regular-expression-list RegexString='(union.*select|select.*from|insert.*into)'

# Create web ACL
aws wafv2 create-web-acl \
  --name prod-web-acl \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://waf-rules.json \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=prodWebACL

# Associate with ALB
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:regional/webacl/prod-web-acl/xxx \
  --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/xxx
```

## Common Troubleshooting Scenarios

### Scenario 1: VPN Tunnel Down

**Symptoms**: Site-to-Site VPN tunnel status shows DOWN, on-premises cannot reach AWS resources

**Troubleshooting Steps**:

1. **Check tunnel status and error messages**:
```bash
aws ec2 describe-vpn-connections --vpn-connection-ids vpn-xxx \
  --query 'VpnConnections[0].VgwTelemetry[*].[OutsideIpAddress,Status,StatusMessage]' \
  --output table
```

2. **Verify IKE Phase 1 (ISAKMP)**:
   - Customer Gateway public IP correct?
   - Pre-shared key matches on both sides?
   - IKE version compatible (IKEv1 or IKEv2)?
   - Encryption/hash algorithms match?
   - UDP 500 and 4500 allowed through firewalls?

3. **Verify IKE Phase 2 (IPsec)**:
   - Dead Peer Detection (DPD) enabled?
   - Perfect Forward Secrecy (PFS) configured?
   - SA lifetime values compatible?

4. **Check BGP (for dynamic routing)**:
```bash
# Verify BGP ASN numbers
aws ec2 describe-customer-gateways --customer-gateway-ids cgw-xxx \
  --query 'CustomerGateways[0].BgpAsn'

aws ec2 describe-vpn-gateways --vpn-gateway-ids vgw-xxx \
  --query 'VpnGateways[0].AmazonSideAsn'
```

5. **Review route propagation**:
```bash
# Check if VGW route propagation enabled
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxx \
  --query 'RouteTables[*].[RouteTableId,PropagatingVgws]'
```

6. **Check CloudWatch metrics**:
   - TunnelState (0 = DOWN, 1 = UP)
   - TunnelDataIn/TunnelDataOut
   - Use CloudWatch Logs for VPN logs (if configured)

**Common Issues**:
- NAT device between customer gateway and internet (breaks IPsec)
- Asymmetric routing (traffic entering through different tunnel)
- Maximum Transmission Unit (MTU) too large (use TCP MSS clamping: 1379 bytes)
- Only one tunnel configured (need redundancy)

### Scenario 2: Direct Connect Circuit Up but No Connectivity

**Symptoms**: Physical Direct Connect shows UP, but cannot access VPC resources

**Troubleshooting Steps**:

1. **Verify Virtual Interface status**:
```bash
aws directconnect describe-virtual-interfaces \
  --query 'virtualInterfaces[*].[virtualInterfaceId,virtualInterfaceState,bgpPeers[0].bgpStatus]' \
  --output table
```

2. **Check BGP peering**:
   - BGP peer status should be "available"
   - Verify ASN numbers match (customer ASN and Amazon ASN)
   - Check BGP routes advertised from on-premises
   - Verify AWS is advertising VPC CIDR blocks

3. **Validate VLAN tagging**:
   - VLAN ID must match on both sides
   - Correct 802.1Q tagging on customer router
   - No VLAN conflicts with other traffic

4. **Verify IP addressing**:
   - Point-to-point /30 or /31 addresses configured correctly
   - Amazon address and customer address on same subnet
   - No IP conflicts

5. **Check route tables in VPC**:
```bash
# Verify routes to on-premises CIDRs exist
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxx \
  --query 'RouteTables[*].Routes[?GatewayId!=`local`]' \
  --output table
```

6. **Verify Direct Connect Gateway associations**:
```bash
aws directconnect describe-direct-connect-gateway-associations \
  --direct-connect-gateway-id dx-gw-xxx
```

7. **Test connectivity with MTR/traceroute**:
   - Path should show Direct Connect router hops
   - Check for packet loss or high latency

8. **Review CloudWatch metrics**:
   - ConnectionState (should be "available")
   - ConnectionBpsEgress/ConnectionBpsIngress
   - ConnectionLightLevelTx/ConnectionLightLevelRx (for fiber health)

**Common Issues**:
- BGP routes not being advertised (check route filters)
- VGW not attached to VPC or TGW
- Overlapping CIDR blocks between VPCs
- Direct Connect Gateway allowed prefixes too restrictive
- Maximum prefix limit reached (100 prefixes per private VIF)

### Scenario 3: Transit Gateway Routing Not Working

**Symptoms**: VPCs attached to Transit Gateway cannot communicate with each other

**Troubleshooting Steps**:

1. **Verify TGW attachments are active**:
```bash
aws ec2 describe-transit-gateway-attachments \
  --filters Name=transit-gateway-id,Values=tgw-xxx \
  --query 'TransitGatewayAttachments[*].[TransitGatewayAttachmentId,State,ResourceId]' \
  --output table
```

2. **Check route table associations**:
```bash
# Each attachment should be associated with a route table
aws ec2 get-transit-gateway-attachment-associations \
  --transit-gateway-attachment-id tgw-attach-xxx

# Check route table associations
aws ec2 describe-transit-gateway-route-tables \
  --transit-gateway-route-table-ids tgw-rtb-xxx \
  --query 'TransitGatewayRouteTables[0].Associations'
```

3. **Verify route propagation**:
```bash
# Check if routes are being propagated
aws ec2 get-transit-gateway-route-table-propagations \
  --transit-gateway-route-table-id tgw-rtb-xxx

# Search for specific route
aws ec2 search-transit-gateway-routes \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --filters "Name=type,Values=static,propagated"
```

4. **Check VPC route tables point to TGW**:
```bash
# Verify each VPC subnet route table has route to TGW
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxx \
  --query 'RouteTables[*].Routes[?TransitGatewayId!=`null`]'
```

5. **Verify no CIDR overlaps**:
   - Overlapping CIDRs will cause routing ambiguity
   - Check all attached VPC CIDRs are unique

6. **Check security groups and NACLs**:
   - Security groups must allow traffic from other VPC CIDRs
   - NACLs must allow bidirectional traffic

7. **Test using VPC Reachability Analyzer**:
```bash
aws ec2 create-network-insights-path \
  --source i-source-xxx \
  --destination i-dest-yyy \
  --protocol tcp \
  --destination-port 443

aws ec2 start-network-insights-analysis \
  --network-insights-path-id nip-xxx
```

**Common Issues**:
- Attachment not associated with correct route table
- Route propagation not enabled
- Static routes missing or incorrect
- Route table blackhole routes (deleted attachments)
- VPC route tables not updated to point to TGW
- Attachment in different TGW route domain (isolated routing)

### Scenario 4: DNS Resolution Failure in Hybrid Environment

**Symptoms**: Cannot resolve on-premises hostnames from VPC, or cannot resolve VPC hostnames from on-premises

**Troubleshooting Steps**:

1. **Verify Route 53 Resolver endpoints**:
```bash
# Check resolver endpoints exist and are available
aws route53resolver list-resolver-endpoints \
  --query 'ResolverEndpoints[*].[Id,Direction,Status]'

# Get endpoint IP addresses
aws route53resolver list-resolver-endpoint-ip-addresses \
  --resolver-endpoint-id rslvr-in-xxx
```

2. **Check resolver rules** (for outbound queries):
```bash
# List resolver rules
aws route53resolver list-resolver-rules

# Verify rule association with VPC
aws route53resolver list-resolver-rule-associations \
  --filters Name=ResolverRuleId,Values=rslvr-rr-xxx
```

3. **Verify security groups**:
   - Inbound endpoint: Allow UDP/TCP 53 from on-premises CIDR
   - Outbound endpoint: Allow UDP/TCP 53 to on-premises DNS servers

4. **Check Route 53 Private Hosted Zone associations**:
```bash
# List PHZ associations
aws route53 list-vpc-association-authorizations \
  --hosted-zone-id Z1234567890ABC
```

5. **Test DNS resolution**:
```bash
# From EC2 instance in VPC
dig @resolver-ip-address onprem.example.com
nslookup vpc-hostname.internal resolver-ip-address

# Check VPC DHCP options set
aws ec2 describe-vpcs --vpc-ids vpc-xxx \
  --query 'Vpcs[0].DhcpOptionsId'

aws ec2 describe-dhcp-options --dhcp-options-ids dopt-xxx
```

6. **Review CloudWatch Logs for Resolver query logging**:
```bash
# Enable query logging if not already enabled
aws route53resolver create-resolver-query-log-config \
  --name prod-query-logs \
  --destination-arn arn:aws:logs:us-east-1:123456789012:log-group:/aws/route53/resolver
```

**Common Issues**:
- Resolver endpoint IPs not added to on-premises DNS forwarders
- Conditional forwarding rules missing or misconfigured
- Security groups blocking DNS traffic (UDP/TCP 53)
- Private Hosted Zone not associated with VPC
- DHCP options set pointing to wrong DNS server
- DNS firewall rules blocking queries
- Route 53 Resolver not available in specific subnets (need endpoint ENIs in each AZ)

### Scenario 5: High Latency or Packet Loss

**Symptoms**: Application experiencing slow response times, intermittent connectivity, packet loss

**Troubleshooting Steps**:

1. **Identify where latency occurs**:
```bash
# MTR (My Traceroute) from source to destination
mtr -r -c 100 destination-ip

# Check for packet loss and high latency hops
# Look for asymmetric paths (different path each way)
```

2. **Review CloudWatch metrics**:
   - Direct Connect: ConnectionBpsIngress/Egress (approaching limits?)
   - VPN: TunnelDataIn/Out, TunnelState
   - Transit Gateway: BytesIn/Out, PacketDropCountBlackhole, PacketsDropCountNoRoute
   - VPC Flow Logs: Check for packet drops

3. **Check MTU and fragmentation**:
   - VPN: MTU should be 1399 or less (overhead for IPsec)
   - Direct Connect: 1500 (standard) or 9001 (jumbo frames if supported)
   - Enable TCP MSS clamping on routers
   - Test with specific packet sizes:
```bash
# Don't fragment test
ping -M do -s 1400 destination-ip
```

4. **Verify Enhanced Networking enabled** (for EC2):
```bash
aws ec2 describe-instances --instance-ids i-xxx \
  --query 'Reservations[0].Instances[0].[EnaSupport,SriovNetSupport]'

# Check network performance in instance type specifications
# ena = Elastic Network Adapter (up to 100 Gbps)
```

5. **Check placement groups** (for low-latency requirements):
```bash
aws ec2 describe-placement-groups --group-names cluster-pg \
  --query 'PlacementGroups[0].Strategy'
# cluster = low latency within same AZ
# partition = spread across logical partitions
# spread = each instance on different hardware
```

6. **Review load balancer health checks**:
   - Unhealthy targets cause routing delays
   - Check health check interval and timeout
   - Review target response times

7. **Analyze VPC Flow Logs for retransmissions**:
```sql
-- High retransmission ratio indicates network issues
fields @timestamp, srcaddr, dstaddr, packets, bytes
| stats sum(packets) as totalPackets, sum(bytes) as totalBytes by srcaddr, dstaddr
| sort totalPackets desc
```

8. **Check for bandwidth saturation**:
   - Direct Connect: Upgrade connection speed or add LAG
   - Transit Gateway: Each attachment has 50 Gbps bandwidth limit
   - VPN: Single tunnel limited to 1.25 Gbps
   - NAT Gateway: Scales to 45 Gbps per AZ

**Common Issues**:
- MTU mismatch causing fragmentation
- Direct Connect or VPN bandwidth saturated
- Path MTU Discovery (PMTUD) blocked by firewall (ICMP type 3 code 4)
- Suboptimal routing (hairpinning traffic)
- Insufficient instance network performance (wrong instance type)
- Cross-region traffic not using TGW peering
- NAT Gateway bottleneck (need one per AZ for high availability)

### Scenario 6: CloudFront Not Caching or Slow Performance

**Symptoms**: CloudFront not improving performance, cache hit ratio low, slow content delivery

**Troubleshooting Steps**:

1. **Check cache behavior settings**:
```bash
# Get distribution configuration
aws cloudfront get-distribution-config --id E1234567890ABC > dist-config.json

# Review cache behaviors (TTL settings, query string forwarding)
cat dist-config.json | jq '.DistributionConfig.CacheBehaviors'
```

2. **Review cache statistics**:
   - CloudWatch Metrics:
     - CacheHitRate (should be > 80% for static content)
     - OriginLatency (time to fetch from origin)
     - Requests (total requests)
     - BytesDownloaded
     - 4xxErrorRate, 5xxErrorRate

3. **Check Cache-Control headers from origin**:
```bash
# Test origin response headers
curl -I https://origin.example.com/path

# Look for:
# Cache-Control: max-age=3600
# Cache-Control: no-cache (prevents caching!)
# Expires: <future-date>
```

4. **Verify object versions are correct**:
   - Versioned URLs (/v1/file.js, /v2/file.js) cache separately
   - Query string parameters included in cache key if configured
   - Invalidate cache if stale content:
```bash
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*" "/images/*" "/css/*"
```

5. **Check origin connection issues**:
   - Origin timeout settings (default 30s)
   - Origin connection timeout (default 10s)
   - Review CloudWatch Logs for origin errors
   - Check origin security groups allow CloudFront IPs (use prefix list)

6. **Analyze real-time logs**:
```bash
# Enable real-time logs for detailed analysis
aws cloudfront create-realtime-log-config \
  --name prod-realtime-logs \
  --end-points file://kinesis-endpoint.json \
  --fields cs-method cs-uri-stem sc-status time-taken

# Query logs for patterns (cache misses, slow requests)
```

7. **Check regional edge cache effectiveness**:
   - Regional Edge Caches sit between edge locations and origin
   - Auto-enabled for custom origins
   - Check RegionalEdgeCacheHitRate metric

8. **Verify compression enabled**:
   - CloudFront can compress objects (gzip, brotli)
   - Reduces transfer size and improves performance
   - Check if "Compress Objects Automatically" enabled

**Common Issues**:
- Origin sets Cache-Control: no-cache or max-age=0
- Query strings included in cache key unnecessarily (each unique query string = separate cache object)
- Cookies forwarded to origin (reduces cache efficiency)
- TTL too low (frequent origin fetches)
- Origin slow response times
- Distribution not using optimal price class (consider "Use All Edge Locations")
- Custom SSL certificate causing latency (use SNI)

## Exam Tips and Common Traps

### VPC Design Patterns

**Multi-Tier Architecture**:
```
Public Subnet:
- ALB/NLB
- NAT Gateway
- Bastion hosts (or use Systems Manager Session Manager!)

Private Subnet (Application):
- Application servers
- Auto Scaling Group
- No direct internet access

Private Subnet (Data):
- RDS Multi-AZ
- ElastiCache
- Isolated from internet completely
```

**Common Trap**:
> Q: Application in private subnet cannot reach internet for software updates
>
> **WRONG**: Add Internet Gateway to private subnet route table
>
> **CORRECT**: Route 0.0.0.0/0 to NAT Gateway in public subnet. NAT GW needs public subnet + IGW.

**Shared Services VPC Pattern**:
- Central VPC with shared services (DNS, Active Directory, monitoring)
- Multiple spoke VPCs for applications
- Connected via Transit Gateway or VPC peering
- Use PrivateLink for private connectivity to shared services

### Transit Gateway vs VPC Peering

| Feature | Transit Gateway | VPC Peering |
|---------|----------------|-------------|
| Topology | Hub-and-spoke | Point-to-point |
| Transitive Routing | YES | NO |
| Cross-Region | Yes (TGW peering) | Yes |
| Cross-Account | Yes | Yes |
| VPN/DX Connectivity | Yes | No (VPN/DX only to VGW) |
| Cost | $0.05/hour + $0.02/GB | Free + data transfer |
| Use Case | Enterprise, many VPCs | Simple, few VPCs |

**Common Trap**:
> Q: Company has 50 VPCs that need to communicate with each other
>
> **WRONG**: Create VPC peering between all VPCs (requires 1,225 peering connections!)
>
> **CORRECT**: Use Transit Gateway (hub-and-spoke model)

### Direct Connect vs VPN

**When to use Direct Connect**:
- Consistent, predictable bandwidth requirements (1 Gbps, 10 Gbps, 100 Gbps)
- Latency-sensitive applications
- Large data transfer (cost-effective for > 500 GB/month)
- Regulatory requirements for private connectivity
- Hybrid cloud architecture with always-on connectivity

**When to use VPN**:
- Quick setup (minutes vs weeks for DX)
- Lower bandwidth requirements (< 1 Gbps)
- Budget constraints (no port hour charges)
- Backup for Direct Connect (redundancy)
- Temporary/project-based connectivity

**Resilient Hybrid Architecture**:
```
Primary: Two Direct Connect connections (different locations)
Backup: Site-to-Site VPN
Result: Four tunnels total for maximum redundancy
```

**Common Trap**:
> Q: Need to establish hybrid connectivity within 1 hour
>
> **WRONG**: Request Direct Connect connection
>
> **CORRECT**: Use Site-to-Site VPN (can be set up in minutes). Order DX for long-term, use VPN as backup.

### BGP Path Selection

**BGP Attributes (in order of preference)**:
1. **Weight** (Cisco-specific, higher is better)
2. **Local Preference** (within AS, higher is better)
3. **AS Path Length** (shorter is better)
4. **Origin Type** (IGP > EGP > Incomplete)
5. **MED (Multi-Exit Discriminator)** (lower is better, used between ASes)

**Influencing Inbound Traffic (to AWS)**:
- Use AS Path Prepending (make path longer so it's less preferred)
- Adjust MED (only works between same neighboring AS)

**Influencing Outbound Traffic (from AWS)**:
- Use Local Preference on customer gateway
- Adjust AS Path (shorter path preferred by AWS)

**Common Trap**:
> Q: How to prefer Direct Connect over VPN for outbound traffic from AWS?
>
> **CORRECT**: Configure longer AS path on VPN BGP advertisements, or use more specific prefixes on Direct Connect

### Route Table Priority

**Route Selection Order** (most specific wins):
1. **Longest prefix match**: 10.0.1.0/24 preferred over 10.0.0.0/16
2. **Local routes** (within VPC): Always highest priority
3. **Propagated routes** (from VGW/TGW): Dynamic
4. **Static routes**: Manually configured

**Prefix Priority** (when same prefix length):
1. Local routes
2. Longest prefix match
3. Static routes
4. Propagated routes from Direct Connect
5. Propagated routes from VPN
6. Propagated routes from TGW

**Common Trap**:
> Q: Have both Direct Connect and VPN advertising 10.0.0.0/16. Which route is preferred?
>
> **ANSWER**: Direct Connect (preferred over VPN in propagated routes)

### Route 53 Routing Policies

**Simple Routing**:
- Single resource
- Multiple IP addresses return in random order
- No health checks

**Weighted Routing**:
- Distribute traffic based on weights (0-255)
- Use case: Blue/green deployments, A/B testing
- Can have health checks

**Latency Routing**:
- Route to lowest latency region for user
- Based on AWS measurements between regions
- Best for global applications

**Failover Routing**:
- Active-passive setup
- Primary and secondary resources
- Requires health check on primary
- Automatic failover when primary unhealthy

**Geolocation Routing**:
- Route based on user's location (continent, country, state)
- Use case: Content localization, compliance (data residency)
- Must have default location

**Geoproximity Routing** (Traffic Flow only):
- Route based on geographic location of users and resources
- Bias value to expand/shrink geographic region
- Can use AWS resources or custom coordinates

**Multi-Value Answer Routing**:
- Return multiple IP addresses (up to 8)
- Health checks for each record
- Not a substitute for load balancer (client-side selection)

**Common Trap**:
> Q: Need to route EU users to EU region due to GDPR compliance
>
> **WRONG**: Use latency-based routing
>
> **CORRECT**: Use geolocation routing (ensures EU users always go to EU region)

### PrivateLink vs VPC Peering vs TGW

**Use PrivateLink when**:
- Providing SaaS service to customers
- Shared services consumed by many VPCs
- Don't want to expose entire VPC network
- Fine-grained access control needed
- One-way traffic (consumer to provider)

**Use VPC Peering when**:
- Small number of VPCs (< 10)
- Full mesh connectivity not needed
- Cost optimization (no hourly charges)
- Simple bidirectional connectivity

**Use Transit Gateway when**:
- Large number of VPCs (10+)
- Transitive routing required
- Centralized routing management
- Connect to on-premises via VPN/Direct Connect
- Network segmentation with route tables

### Security Group vs NACL Best Practices

**Security Groups** (Stateful):
```
Use for:
- Instance-level protection
- Application-level rules
- Allow rules only
- Reference other security groups (micro-segmentation)

Example:
- Web tier SG: Allow 443 from ALB SG
- App tier SG: Allow 8080 from Web tier SG
- DB tier SG: Allow 3306 from App tier SG
```

**NACLs** (Stateless):
```
Use for:
- Subnet-level protection (additional layer)
- Explicit deny rules
- Block specific IP addresses
- Compliance requirements

Remember:
- Must allow ephemeral ports (1024-65535) for return traffic!
- Rules evaluated in order (lowest number first)
- Explicit deny overrides allows
```

**Common Trap**:
> Q: Added security group rule to allow port 443 inbound, but still cannot connect
>
> **CHECK**: NACL might be blocking traffic. NACLs require both inbound AND outbound rules.

### CloudFront vs Global Accelerator

| Feature | CloudFront | Global Accelerator |
|---------|-----------|-------------------|
| Use Case | Static content, videos, dynamic content | TCP/UDP applications, gaming, IoT, VoIP |
| Protocol | HTTP/HTTPS | TCP/UDP (Layer 4) |
| Caching | YES | NO |
| Static IPs | No (DNS-based) | YES (2 static anycast IPs) |
| Web Security | WAF, Shield | Shield only |
| Origin | S3, HTTP/HTTPS origins | ALB, NLB, EC2, EIP |
| Price | Pay for data transfer | Pay for accelerator + data transfer |

**Common Trap**:
> Q: Need static IP addresses for whitelisting firewall rules for global application
>
> **WRONG**: Use CloudFront
>
> **CORRECT**: Use Global Accelerator (provides 2 static anycast IP addresses)

### VPC Endpoint Types

**Gateway Endpoints** (S3, DynamoDB):
- Free (no hourly charge)
- Target in route table
- Prefix list (pl-xxx)
- VPC-level resource
- No security groups needed
- Same region only

**Interface Endpoints** (Most AWS services):
- ENI in your VPC subnet
- Private IP address
- Uses PrivateLink
- Charged per hour + data processed
- Security groups apply
- Can be cross-region (via inter-region VPC peering)
- Private DNS option (use service's DNS name)

**Common Trap**:
> Q: Application in private subnet getting charged for NAT Gateway data processing to access S3
>
> **OPTIMIZATION**: Create S3 Gateway Endpoint (free, no data processing charges through NAT)

## Network Design Patterns and Best Practices

### Hub-and-Spoke with Transit Gateway

**Architecture**:
```
                  Transit Gateway (Hub)
                         |
        +----------------+----------------+
        |                |                |
   Prod VPC         Dev VPC          Shared Services VPC
  (10.1.0.0/16)   (10.2.0.0/16)     (10.0.0.0/16)
                                     - AD, DNS, Monitoring
                         |
                  +------+------+
                  |             |
           Direct Connect     VPN
          (On-Premises)    (Backup)
```

**Best Practices**:
1. Use separate TGW route tables for network segmentation:
   - Production route table (prod VPCs + shared services)
   - Development route table (dev VPCs + shared services)
   - Shared services route table (access all)

2. Enable auto-accept for VPC attachments (same account)
3. Use TGW peering for inter-region connectivity
4. Monitor with CloudWatch: BytesIn/Out, PacketDropCount, PacketsIn/Out
5. Enable VPC Flow Logs on TGW network interfaces

### Multi-Region Active-Active

**Architecture**:
```
                 Route 53 (Latency-based routing)
                            |
           +----------------+----------------+
           |                                 |
      US-EAST-1                         EU-WEST-1
    - ALB + EC2 ASG                   - ALB + EC2 ASG
    - Aurora Global (Primary)         - Aurora Global (Secondary)
    - DynamoDB Global Table          - DynamoDB Global Table
    - S3 (CRR to EU)                 - S3 (CRR to US)

      Transit Gateway                  Transit Gateway
           |                                 |
    Direct Connect                     Direct Connect
           |                                 |
      On-Premises (US)                On-Premises (EU)
```

**Best Practices**:
1. Use Aurora Global Database for low-latency reads (< 1s replication lag)
2. DynamoDB Global Tables for multi-region read/write
3. S3 Cross-Region Replication (CRR) for object replication
4. Global Accelerator for static IP addresses and optimal routing
5. Route 53 health checks for automatic failover
6. CloudFront with regional edge caches

### Centralized Egress (Outbound Internet)

**Architecture**:
```
Spoke VPCs (10.1.x.x, 10.2.x.x, 10.3.x.x)
           |
    Transit Gateway
           |
    Egress VPC (10.0.0.0/16)
           |
  +--------+--------+
  |                 |
NAT GW (AZ-1)   NAT GW (AZ-2)
  |                 |
IGW (Internet Gateway)
```

**Benefits**:
- Centralized internet egress control
- Single point for security inspection (add Network Firewall)
- Reduced NAT Gateway costs (only in egress VPC)
- Simplified security policies

**Implementation**:
1. Create egress VPC with public and private subnets
2. Deploy NAT Gateways in public subnets (one per AZ)
3. Attach egress VPC to Transit Gateway
4. Create TGW route table for internet traffic (0.0.0.0/0 → egress VPC)
5. Associate spoke VPCs with internet route table
6. Optional: Add AWS Network Firewall for inspection

### Zero-Trust Network with PrivateLink

**Architecture**:
```
Service Provider VPC:
  - Application (ALB/NLB)
  - VPC Endpoint Service (PrivateLink)

Service Consumer VPCs (10.1.x.x, 10.2.x.x):
  - VPC Interface Endpoints
  - No routing between consumer VPCs
  - No VPC peering required
```

**Best Practices**:
1. Use Network Load Balancer for PrivateLink endpoint service
2. Enable acceptance required for manual approval
3. Use endpoint policies to restrict access
4. Monitor connections with VPC Flow Logs
5. Use security groups on interface endpoints
6. Consider AWS PrivateLink for third-party SaaS integrations

### Global Content Delivery

**Architecture**:
```
              Route 53 DNS
                    |
        +-----------+-----------+
        |                       |
   CloudFront               CloudFront
   (US Viewers)            (EU Viewers)
        |                       |
   +----+----+             +----+----+
   |         |             |         |
S3 (US) ALB (US)      S3 (EU)  ALB (EU)
```

**Best Practices**:
1. Use CloudFront for both static and dynamic content
2. Enable Origin Shield for reduced origin load
3. Use Lambda@Edge for request/response manipulation
4. Enable compression (gzip/brotli)
5. Set appropriate Cache-Control headers
6. Use versioned URLs for cache busting
7. Enable HTTPS with custom SSL certificate (via ACM)
8. Use Origin Access Identity (OAI) for S3 origins
9. Implement WAF rules for security
10. Monitor with CloudWatch and real-time logs

### Network Segmentation for Compliance

**Architecture**:
```
Transit Gateway with Multiple Route Tables:

PCI Route Table:
  - PCI VPC (isolated workloads)
  - Direct Connect only (no internet access)
  - Restricted to other PCI resources

General Route Table:
  - Application VPCs
  - Internet access via Egress VPC
  - Access to shared services

Management Route Table:
  - Shared Services VPC (AD, DNS, Monitoring)
  - Access to all route tables
  - Centralized logging
```

**Best Practices**:
1. Use separate AWS accounts for different security zones (via AWS Organizations)
2. TGW route tables for network isolation
3. Security groups as defense in depth
4. VPC Flow Logs to S3 with AWS KMS encryption
5. Centralized logging to Security Account
6. AWS Network Firewall for traffic inspection
7. DNS Firewall for malicious domain blocking
8. No IGW attached to compliance VPCs

## Pre-Exam Checklist

### Core Networking Services to Master
- VPC design and implementation (subnets, route tables, gateways)
- Transit Gateway architecture and routing
- Direct Connect (VIFs, LAG, DX Gateway, redundancy)
- Site-to-Site VPN (BGP, redundancy, monitoring)
- VPC Peering (limitations, use cases)
- VPC Endpoints (Gateway vs Interface)
- PrivateLink architecture
- Elastic Load Balancers (ALB, NLB, GWLB)
- Route 53 (all routing policies, health checks, Resolver)
- CloudFront (caching, origins, security)
- Global Accelerator (anycast, use cases)
- AWS Network Firewall (stateful/stateless rules)
- AWS WAF (rule types, managed rules)
- VPC Flow Logs (analysis, troubleshooting)

### BGP and Routing Concepts
- BGP path selection criteria
- AS Path prepending for traffic engineering
- Local Preference vs MED
- Route propagation priorities (DX > VPN)
- Route table longest prefix match
- Transitive routing (TGW yes, peering no)
- Route summarization and aggregation
- Asymmetric routing detection

### Hybrid Connectivity Design
- Resilient Direct Connect (multi-location)
- VPN backup for Direct Connect
- Direct Connect Gateway use cases
- Transit VIF for TGW integration
- Site-to-Site VPN with Transit Gateway
- Customer Gateway configuration
- Virtual Private Gateway vs Transit Gateway
- BGP routing preferences

### DNS Architecture
- Route 53 Resolver endpoints (inbound/outbound)
- Conditional forwarding rules
- Private Hosted Zones
- DNSSEC signing
- Split-view DNS
- Hybrid DNS integration
- DNS Firewall rules
- Query logging and analysis

### Security and Compliance
- Defense in depth (multiple security layers)
- Security Groups vs NACLs (stateful vs stateless)
- PrivateLink for private connectivity
- AWS WAF rule types and managed rules
- AWS Shield (Standard vs Advanced)
- Network Firewall deployment patterns
- TLS/SSL termination strategies
- Encryption in transit (IPsec, MACsec, TLS)
- VPC isolation and segmentation
- Compliance frameworks (PCI-DSS, HIPAA, FedRAMP)

### Monitoring and Troubleshooting
- VPC Flow Logs analysis (CloudWatch Insights queries)
- CloudWatch metrics for network services
- VPC Reachability Analyzer
- Network Access Analyzer
- Traffic Mirroring for packet inspection
- Direct Connect CloudWatch metrics
- VPN tunnel monitoring
- Route 53 query logging
- CloudFront access logs and real-time logs

### Performance Optimization
- Enhanced networking (ENA vs SR-IOV)
- Placement groups (cluster, partition, spread)
- MTU optimization and Path MTU Discovery
- TCP MSS clamping for VPN
- Jumbo frames for Direct Connect
- CloudFront caching strategies
- Global Accelerator for TCP/UDP optimization
- Network Load Balancer for low latency
- Direct Connect LAG for increased bandwidth

### Design Scenarios Practice
- Multi-VPC connectivity (TGW vs Peering)
- Hybrid cloud with Direct Connect + VPN backup
- Multi-region active-active architecture
- Centralized egress for internet traffic
- PrivateLink for shared services
- Global content delivery with CloudFront
- Zero-trust network architecture
- Network segmentation for compliance
- Disaster recovery with failover
- Bandwidth optimization and cost reduction

### Cost Optimization
- Gateway Endpoints vs Interface Endpoints (S3, DynamoDB)
- NAT Gateway vs NAT Instance (cost vs management)
- Direct Connect vs VPN (data transfer volume)
- VPC Peering vs Transit Gateway (number of VPCs)
- CloudFront vs S3 Transfer Acceleration
- Reserved capacity for Direct Connect
- Right-sizing bandwidth requirements

## Final Exam Tips

### Time Management
- 170 minutes, 65 questions = 2.6 minutes per question
- Flag difficult scenarios, return later
- Read entire question and all answers before selecting
- Watch for keywords: "MOST cost-effective", "LEAST operational overhead", "highest availability"

### Common Question Patterns
1. **"Most secure solution"**: Private connectivity (PrivateLink, VPC Endpoints), no internet exposure, encryption
2. **"Highest availability"**: Multi-AZ, redundant connections, automatic failover, health checks
3. **"Lowest latency"**: Direct Connect, placement groups, enhanced networking, CloudFront, Global Accelerator
4. **"Hybrid connectivity with redundancy"**: Two Direct Connect (different locations) + VPN backup
5. **"Centralized management"**: Transit Gateway, AWS Network Manager, Resource Access Manager
6. **"Compliance requirement"**: Network isolation, VPC Flow Logs, encryption in transit, no internet access
7. **"Cost optimization"**: Gateway Endpoints (free), VPC Peering (vs TGW), VPN (vs DX for low bandwidth)

### Elimination Strategy
- Remove obviously wrong answers first
- Watch for anti-patterns (NAT in private subnet, IGW for private subnet)
- Be suspicious of "always" and "never" statements
- Choose AWS managed services over manual solutions when possible
- Prefer simpler solutions unless complexity is justified

### AWS Networking Anti-Patterns (Wrong Answers)
- NAT Gateway in private subnet
- Internet Gateway attached to private subnet route table
- Transitive routing through VPC peering
- Single Direct Connect without backup
- VPN without redundant tunnels
- CloudFront for real-time bidirectional TCP applications
- Security group deny rules (only allow rules exist)
- Overlapping CIDR blocks in connected VPCs

### Before Exam Day
- Review AWS Networking whitepapers
- Hands-on practice: Create VPC, TGW, DX Gateway, VPN, Route 53 Resolver
- Practice BGP configuration and troubleshooting
- Understand VPC Flow Logs analysis
- Take multiple practice exams (target 85%+ passing)
- Review missed questions to understand why
- Sleep well, arrive early, stay calm

Good luck with your AWS Advanced Networking Specialty certification!
