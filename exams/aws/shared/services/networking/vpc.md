# Amazon VPC (Virtual Private Cloud)

## Service Overview and Purpose

Amazon Virtual Private Cloud (VPC) lets you provision a logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of IP address ranges, creation of subnets, and configuration of route tables and network gateways.

### Key Characteristics
- **Network Isolation**: Logically isolated network environment
- **Complete Control**: Full control over networking configuration
- **Flexible Architecture**: Support for public, private, and hybrid connectivity
- **Security**: Multiple layers of security controls
- **Scalable**: Grow from single instance to enterprise-scale deployments

## Key Features and Capabilities

### Core Components

#### Subnets
- **Public Subnets**: Direct internet access via Internet Gateway
- **Private Subnets**: No direct internet access, use NAT for outbound
- **Availability Zone Placement**: Subnets are tied to specific AZs
- **CIDR Blocks**: IP address ranges for subnet resources

#### Route Tables
- **Traffic Routing**: Control traffic flow between subnets
- **Default Route Table**: Created automatically with VPC
- **Custom Route Tables**: Additional routing control
- **Route Priorities**: Longest prefix match rule

#### Internet Connectivity
- **Internet Gateway (IGW)**: Bidirectional internet access
- **NAT Gateway**: Managed outbound internet access for private subnets
- **NAT Instance**: EC2-based NAT solution (customer managed)
- **Egress-Only Internet Gateway**: IPv6 outbound-only access

#### Security
- **Security Groups**: Instance-level stateful firewall
- **Network ACLs**: Subnet-level stateless firewall
- **VPC Flow Logs**: Network traffic logging
- **VPC Endpoints**: Private connectivity to AWS services

### Advanced Features

#### VPC Peering
- **Cross-VPC Communication**: Connect VPCs privately
- **Transitive Routing**: Not supported (requires full mesh)
- **Cross-Region**: Supported across regions
- **Cross-Account**: Supported across AWS accounts

#### Transit Gateway
- **Central Hub**: Connect multiple VPCs and on-premises networks
- **Route Tables**: Control routing between attachments
- **Propagation**: Automatic route propagation
- **Multicast**: Support for multicast traffic

#### VPC Endpoints
- **Gateway Endpoints**: S3 and DynamoDB (route table based)
- **Interface Endpoints**: Other AWS services (ENI based)
- **PrivateLink**: Private connectivity to third-party services
- **Policy Control**: Resource and service access policies

## Use Cases and Scenarios

### Architecture Patterns

1. **Single-Tier Architecture**
   ```bash
   # Simple web application in public subnet
   aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=WebApp-VPC}]'

   aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.1.0/24 --availability-zone us-west-2a \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-Subnet}]'

   aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=WebApp-IGW}]'
   aws ec2 attach-internet-gateway --vpc-id vpc-12345678 --internet-gateway-id igw-12345678
   ```

2. **Multi-Tier Architecture**
   ```bash
   # Three-tier architecture: Web, App, Database
   # Public subnet for web tier
   aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.1.0/24 --availability-zone us-west-2a

   # Private subnet for application tier
   aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.2.0/24 --availability-zone us-west-2a

   # Private subnet for database tier
   aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.3.0/24 --availability-zone us-west-2a

   # NAT Gateway for private subnet internet access
   aws ec2 create-nat-gateway --subnet-id subnet-12345678 --allocation-id eipalloc-12345678
   ```

3. **Hybrid Cloud Architecture**
   ```bash
   # VPN Gateway for on-premises connectivity
   aws ec2 create-vpn-gateway --type ipsec.1 --amazon-side-asn 65000
   aws ec2 attach-vpn-gateway --vpn-gateway-id vgw-12345678 --vpc-id vpc-12345678

   # Customer Gateway
   aws ec2 create-customer-gateway --type ipsec.1 --public-ip 203.0.113.12 --bgp-asn 65001

   # VPN Connection
   aws ec2 create-vpn-connection --type ipsec.1 --customer-gateway-id cgw-12345678 --vpn-gateway-id vgw-12345678
   ```

## Configuration Details and Best Practices

### VPC Design Best Practices

```bash
# Create production VPC with comprehensive configuration
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --enable-dns-hostnames \
  --enable-dns-support \
  --tag-specifications 'ResourceType=vpc,Tags=[
    {Key=Name,Value=Production-VPC},
    {Key=Environment,Value=Production},
    {Key=Project,Value=WebApplication}
  ]'

# Create subnets across multiple AZs for high availability
for az in a b c; do
  # Public subnets
  aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.$((10+az_num)).0/24 \
    --availability-zone us-west-2$az \
    --tag-specifications "ResourceType=subnet,Tags=[
      {Key=Name,Value=Public-Subnet-$az},
      {Key=Type,Value=Public}
    ]"

  # Private subnets for applications
  aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.$((20+az_num)).0/24 \
    --availability-zone us-west-2$az \
    --tag-specifications "ResourceType=subnet,Tags=[
      {Key=Name,Value=Private-App-Subnet-$az},
      {Key=Type,Value=Private}
    ]"

  # Private subnets for databases
  aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.$((30+az_num)).0/24 \
    --availability-zone us-west-2$az \
    --tag-specifications "ResourceType=subnet,Tags=[
      {Key=Name,Value=Private-DB-Subnet-$az},
      {Key=Type,Value=Database}
    ]"
done
```

### Security Configuration

```bash
# Create security groups with least privilege
# Web tier security group
aws ec2 create-security-group \
  --group-name web-tier-sg \
  --description "Security group for web tier" \
  --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
  --group-id sg-web123 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id sg-web123 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Application tier security group
aws ec2 create-security-group \
  --group-name app-tier-sg \
  --description "Security group for application tier" \
  --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
  --group-id sg-app456 \
  --protocol tcp \
  --port 8080 \
  --source-group sg-web123

# Database tier security group
aws ec2 create-security-group \
  --group-name db-tier-sg \
  --description "Security group for database tier" \
  --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
  --group-id sg-db789 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-app456
```

### Network ACLs Configuration

```bash
# Create custom Network ACL for additional security
aws ec2 create-network-acl \
  --vpc-id vpc-12345678 \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=Web-Tier-NACL}]'

# Allow HTTP inbound
aws ec2 create-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 100 \
  --protocol tcp \
  --rule-action allow \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0

# Allow HTTPS inbound
aws ec2 create-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 110 \
  --protocol tcp \
  --rule-action allow \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0

# Allow return traffic (ephemeral ports)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 120 \
  --protocol tcp \
  --rule-action allow \
  --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0

# Outbound rules
aws ec2 create-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 100 \
  --protocol tcp \
  --rule-action allow \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress
```

### VPC Endpoints Configuration

```bash
# Gateway endpoint for S3
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-west-2.s3 \
  --route-table-ids rtb-12345678 \
  --policy-document '{
    "Statement": [{
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }]
  }'

# Interface endpoint for EC2
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-west-2.ec2 \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-12345678 subnet-87654321 \
  --security-group-ids sg-endpoint123 \
  --private-dns-enabled
```

## Advanced Networking Features

### VPC Peering

```bash
# Create VPC peering connection
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-12345678 \
  --peer-vpc-id vpc-87654321 \
  --peer-region us-east-1 \
  --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=Prod-to-Dev-Peering}]'

# Accept peering connection (in peer region)
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-12345678 \
  --region us-east-1

# Update route tables
aws ec2 create-route \
  --route-table-id rtb-12345678 \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id pcx-12345678
```

### Transit Gateway

```bash
# Create Transit Gateway
aws ec2 create-transit-gateway \
  --description "Main Transit Gateway" \
  --options DefaultRouteTableAssociation=enable,DefaultRouteTablePropagation=enable \
  --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=Main-TGW}]'

# Attach VPCs to Transit Gateway
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-12345678 \
  --vpc-id vpc-12345678 \
  --subnet-ids subnet-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=Prod-VPC-Attachment}]'

# Create custom route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Prod-Routes}]'

# Create routes
aws ec2 create-route \
  --route-table-id tgw-rtb-12345678 \
  --destination-cidr-block 10.2.0.0/16 \
  --transit-gateway-attachment-id tgw-attach-87654321
```

## Monitoring and Troubleshooting

### VPC Flow Logs

```bash
# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name VPCFlowLogs \
  --deliver-logs-permission-arn arn:aws:iam::account:role/flowlogsRole \
  --tag-specifications 'ResourceType=vpc-flow-log,Tags=[{Key=Name,Value=VPC-Flow-Logs}]'

# Analyze flow logs with CloudWatch Insights
# Query example: Find top talkers
fields @timestamp, srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes
| filter action = "ACCEPT"
| stats sum(bytes) as total_bytes by srcaddr
| sort total_bytes desc
| limit 20
```

### Network Troubleshooting

```bash
# VPC Reachability Analyzer
aws ec2 create-network-insights-path \
  --source sg-12345678 \
  --destination sg-87654321 \
  --protocol tcp \
  --destination-port 80 \
  --tag-specifications 'ResourceType=network-insights-path,Tags=[{Key=Name,Value=Web-to-App-Path}]'

aws ec2 start-network-insights-analysis \
  --network-insights-path-id nip-12345678

# Common troubleshooting commands
# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-12345678"

# Check security groups
aws ec2 describe-security-groups --filters "Name=vpc-id,Values=vpc-12345678"

# Check NACLs
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=vpc-12345678"

# Check VPC endpoints
aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=vpc-12345678"
```

## Security Best Practices

### Network Segmentation

```bash
# Implement network segmentation with multiple subnets
# DMZ subnet for public-facing resources
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.1.0/24 \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=DMZ-Subnet},{Key=Tier,Value=DMZ}]'

# Application subnet with no direct internet access
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.10.0/24 \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=App-Subnet},{Key=Tier,Value=Application}]'

# Database subnet in isolated network
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.20.0/24 \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=DB-Subnet},{Key=Tier,Value=Database}]'
```

### Defense in Depth

```python
# Example of layered security implementation
security_layers = {
    "edge": {
        "cloudfront": "DDoS protection and geographic restrictions",
        "waf": "Application layer filtering",
        "shield": "DDoS protection"
    },
    "network": {
        "nacl": "Subnet-level stateless filtering",
        "security_groups": "Instance-level stateful filtering",
        "vpc_endpoints": "Private service access"
    },
    "host": {
        "os_hardening": "Operating system security",
        "antivirus": "Malware protection",
        "intrusion_detection": "Host-based monitoring"
    },
    "application": {
        "authentication": "User verification",
        "authorization": "Access control",
        "encryption": "Data protection"
    }
}
```

## Cost Optimization

### VPC Cost Factors

```bash
# Monitor VPC-related costs
# NAT Gateway data processing charges
aws cloudwatch get-metric-statistics \
  --namespace AWS/NATGateway \
  --metric-name BytesOutToDestination \
  --dimensions Name=NatGatewayId,Value=nat-12345678 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-31T23:59:59Z \
  --period 86400 \
  --statistics Sum

# VPC Endpoint cost optimization
# Use Gateway endpoints (free) instead of Interface endpoints when possible
# Monitor Interface endpoint usage
aws ec2 describe-vpc-endpoint-connections \
  --filters "Name=vpc-endpoint-state,Values=available"

# Data transfer costs
# Cross-AZ data transfer charges
# Internet egress charges
# Consider using CloudFront for static content delivery
```

## Exam-Specific Tips and Common Scenarios

### Key Exam Topics

1. **VPC Components**
   - Understand subnets, route tables, gateways
   - Know security groups vs NACLs differences
   - Understand VPC endpoints and their types
   - Know peering and Transit Gateway capabilities

2. **Security Models**
   - Layered security approach
   - Stateful vs stateless filtering
   - Network segmentation strategies
   - VPC Flow Logs for monitoring

3. **Connectivity Options**
   - Internet Gateway for public access
   - NAT Gateway/Instance for private subnet internet access
   - VPN and Direct Connect for hybrid connectivity
   - VPC Endpoints for private AWS service access

### Common Exam Scenarios

1. **Multi-Tier Web Application**
   - Design VPC with public and private subnets
   - Configure security groups for each tier
   - Implement proper routing and NAT
   - Ensure high availability across AZs

2. **Hybrid Cloud Connectivity**
   - Connect on-premises to AWS
   - Choose between VPN and Direct Connect
   - Configure routing between networks
   - Implement secure connectivity

3. **Microservices Architecture**
   - Multiple VPCs for service isolation
   - Use Transit Gateway for connectivity
   - Implement service mesh networking
   - Container networking with EKS/ECS

### Exam Tips

- **Know the differences** between security groups and NACLs
- **Understand routing** and how route tables work
- **Remember VPC limits** and how to work around them
- **Know connectivity options** for different scenarios
- **Understand cost implications** of different networking choices

This comprehensive VPC documentation covers all essential networking concepts needed for AWS certification exams, including practical implementation examples and real-world scenarios.