# AWS Transit Gateway

## Service Overview and Purpose

AWS Transit Gateway is a service that enables customers to connect their Amazon Virtual Private Clouds (VPCs) and on-premises networks to a single gateway. It acts as a hub that controls how traffic is routed among all the connected networks which act like spokes.

### Key Characteristics
- **Centralized Hub**: Single point of connectivity
- **Scalable**: Support for thousands of VPCs
- **Multi-Region**: Global network connectivity
- **Route Control**: Flexible routing policies
- **Monitoring**: Comprehensive monitoring and logging

## Key Features and Capabilities

### Core Features
- **VPC Attachments**: Connect multiple VPCs
- **VPN Attachments**: Connect on-premises networks
- **Direct Connect Gateway**: Integration with Direct Connect
- **Peering Attachments**: Connect Transit Gateways across regions
- **Route Tables**: Control traffic flow between attachments

### Advanced Features
- **Multicast**: Support for multicast traffic
- **Security Group Referencing**: Reference security groups across VPCs
- **Network Manager**: Global network monitoring
- **Connect Attachments**: Third-party network appliance integration

## Configuration Examples

### Basic Transit Gateway Setup
```bash
# Create Transit Gateway
aws ec2 create-transit-gateway \
  --description "Main Transit Gateway" \
  --options DefaultRouteTableAssociation=enable,DefaultRouteTablePropagation=enable,DnsSupport=enable,MulticastSupport=enable \
  --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=Main-TGW},{Key=Environment,Value=Production}]'

# Create VPC attachment
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-12345678 \
  --vpc-id vpc-production \
  --subnet-ids subnet-12345678 subnet-87654321 \
  --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=Production-VPC-Attachment}]'

# Accept attachment (if cross-account)
aws ec2 accept-transit-gateway-vpc-attachment \
  --transit-gateway-attachment-id tgw-attach-12345678
```

### Advanced Routing Configuration
```bash
# Create custom route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Production-Routes}]'

# Associate VPC attachment with route table
aws ec2 associate-transit-gateway-route-table \
  --transit-gateway-attachment-id tgw-attach-production \
  --transit-gateway-route-table-id tgw-rtb-12345678

# Create static routes
aws ec2 create-route \
  --route-table-id tgw-rtb-12345678 \
  --destination-cidr-block 10.1.0.0/16 \
  --transit-gateway-attachment-id tgw-attach-development

# Enable route propagation
aws ec2 enable-transit-gateway-route-table-propagation \
  --transit-gateway-route-table-id tgw-rtb-12345678 \
  --transit-gateway-attachment-id tgw-attach-vpn
```

### Cross-Region Peering
```bash
# Create peering attachment (requester region)
aws ec2 create-transit-gateway-peering-attachment \
  --transit-gateway-id tgw-12345678 \
  --peer-transit-gateway-id tgw-87654321 \
  --peer-account-id 123456789012 \
  --peer-region us-east-1 \
  --tag-specifications 'ResourceType=transit-gateway-peering-attachment,Tags=[{Key=Name,Value=West-to-East-Peering}]'

# Accept peering attachment (accepter region)
aws ec2 accept-transit-gateway-peering-attachment \
  --transit-gateway-attachment-id tgw-attach-peering-12345678 \
  --region us-east-1

# Add routes for cross-region traffic
aws ec2 create-route \
  --route-table-id tgw-rtb-west \
  --destination-cidr-block 10.2.0.0/16 \
  --transit-gateway-attachment-id tgw-attach-peering-12345678
```

## Architecture Patterns

### Hub-and-Spoke Model
```bash
# Central Transit Gateway connecting multiple VPCs
# Production VPC attachment
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-12345678 \
  --vpc-id vpc-production \
  --subnet-ids subnet-prod1 subnet-prod2

# Development VPC attachment
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-12345678 \
  --vpc-id vpc-development \
  --subnet-ids subnet-dev1 subnet-dev2

# Shared Services VPC attachment
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-12345678 \
  --vpc-id vpc-shared-services \
  --subnet-ids subnet-shared1 subnet-shared2
```

### Segmented Networks
```bash
# Create separate route tables for network segmentation
# Production route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Production-RT}]'

# Development route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Development-RT}]'

# Shared Services route table
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Shared-Services-RT}]'

# Associate attachments with appropriate route tables
aws ec2 associate-transit-gateway-route-table \
  --transit-gateway-attachment-id tgw-attach-production \
  --transit-gateway-route-table-id tgw-rtb-production

# Create selective routes (Production can access Shared Services, but not Development)
aws ec2 create-route \
  --route-table-id tgw-rtb-production \
  --destination-cidr-block 10.100.0.0/16 \
  --transit-gateway-attachment-id tgw-attach-shared-services
```

## VPN Integration

### Site-to-Site VPN with Transit Gateway
```bash
# Create Customer Gateway
aws ec2 create-customer-gateway \
  --type ipsec.1 \
  --public-ip 203.0.113.12 \
  --bgp-asn 65000 \
  --tag-specifications 'ResourceType=customer-gateway,Tags=[{Key=Name,Value=Office-CGW}]'

# Create VPN connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-12345678 \
  --transit-gateway-id tgw-12345678 \
  --options StaticRoutesOnly=false \
  --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=Office-VPN}]'

# VPN attachment is automatically created
# Configure BGP on customer router for dynamic routing
```

### Multiple VPN Connections
```bash
# Primary VPN connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-primary \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=Primary-VPN}]'

# Backup VPN connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id cgw-backup \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=Backup-VPN}]'

# BGP AS-PATH prepending for primary/backup preference
# Configure on customer routers
```

## Direct Connect Integration

### Direct Connect Gateway with Transit Gateway
```bash
# Create Direct Connect Gateway
aws directconnect create-direct-connect-gateway \
  --name "Global-DX-Gateway" \
  --amazon-side-asn 64512

# Create Transit VIF
aws directconnect create-transit-virtual-interface \
  --connection-id dxcon-12345678 \
  --new-transit-virtual-interface '{
    "virtualInterfaceName": "Transit-VIF",
    "vlan": 100,
    "asn": 65000,
    "mtu": 1500,
    "directConnectGatewayId": "dx-gw-12345678"
  }'

# Associate DX Gateway with Transit Gateway
aws directconnect create-direct-connect-gateway-association \
  --direct-connect-gateway-id dx-gw-12345678 \
  --gateway-id tgw-12345678 \
  --association-name "DX-TGW-Association"
```

## Monitoring and Troubleshooting

### CloudWatch Metrics
```bash
# Monitor Transit Gateway metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/TransitGateway \
  --metric-name BytesIn \
  --dimensions Name=TransitGateway,Value=tgw-12345678 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Sum

# Monitor attachment metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/TransitGateway \
  --metric-name PacketDropCount \
  --dimensions Name=TransitGateway,Value=tgw-12345678 Name=Attachment,Value=tgw-attach-12345678 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Sum
```

### VPC Flow Logs with Transit Gateway
```bash
# Enable Flow Logs for Transit Gateway
aws ec2 create-flow-logs \
  --resource-type TransitGateway \
  --resource-ids tgw-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name TransitGatewayFlowLogs \
  --deliver-logs-permission-arn arn:aws:iam::account:role/flowlogsRole

# Query flow logs for troubleshooting
aws logs filter-log-events \
  --log-group-name TransitGatewayFlowLogs \
  --filter-pattern "{ $.action = \"REJECT\" }" \
  --start-time 1640995200000
```

### Route Analysis
```bash
# Check route tables
aws ec2 describe-transit-gateway-route-tables \
  --transit-gateway-route-table-ids tgw-rtb-12345678

# Search routes
aws ec2 search-transit-gateway-routes \
  --transit-gateway-route-table-id tgw-rtb-12345678 \
  --filters Prefix=10.0.0.0/16

# Check route propagation
aws ec2 get-transit-gateway-route-table-propagations \
  --transit-gateway-route-table-id tgw-rtb-12345678
```

## Security Considerations

### Network Segmentation
```bash
# Create isolated route tables for security zones
# DMZ route table (limited connectivity)
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=DMZ-RT},{Key=SecurityZone,Value=DMZ}]'

# Internal route table (full connectivity)
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-12345678 \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Internal-RT},{Key=SecurityZone,Value=Internal}]'

# Restricted routes for DMZ
aws ec2 create-route \
  --route-table-id tgw-rtb-dmz \
  --destination-cidr-block 10.100.0.0/16 \
  --transit-gateway-attachment-id tgw-attach-shared-services
# No route to internal networks from DMZ
```

### Access Control
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:DescribeTransitGateways",
      "ec2:DescribeTransitGatewayAttachments",
      "ec2:DescribeTransitGatewayRouteTables"
    ],
    "Resource": "*"
  }, {
    "Effect": "Allow",
    "Action": [
      "ec2:CreateTransitGatewayVpcAttachment"
    ],
    "Resource": "*",
    "Condition": {
      "StringEquals": {
        "aws:RequestedRegion": ["us-west-2", "us-east-1"]
      }
    }
  }]
}
```

## Cost Optimization

### Pricing Components
- **Transit Gateway Hours**: $0.05 per hour per Transit Gateway
- **Data Processing**: $0.02 per GB processed
- **No Charges**: For VPC-to-VPC traffic within same AZ

### Cost Optimization Strategies
```bash
# Monitor data processing charges
aws cloudwatch get-metric-statistics \
  --namespace AWS/TransitGateway \
  --metric-name BytesIn \
  --dimensions Name=TransitGateway,Value=tgw-12345678 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-31T23:59:59Z \
  --period 86400 \
  --statistics Sum

# Optimize by reducing cross-AZ traffic
# Use VPC peering for high-bandwidth VPC-to-VPC communication
# Implement data caching to reduce redundant transfers
```

## Exam-Specific Tips

### Key Exam Topics
1. **Centralized Connectivity**: Hub-and-spoke model benefits
2. **Route Tables**: Custom routing and network segmentation
3. **Cross-Region**: Peering attachments for global connectivity
4. **VPN Integration**: Site-to-Site VPN with dynamic routing
5. **Direct Connect**: Integration with DX Gateway

### Common Exam Scenarios
1. **Multi-VPC Connectivity**: Replace complex VPC peering with Transit Gateway
2. **Hybrid Networking**: Connect on-premises to multiple VPCs
3. **Network Segmentation**: Use route tables for security isolation
4. **Global Networks**: Cross-region connectivity with peering
5. **Migration**: Transition from VPC peering to Transit Gateway

### Exam Tips
- **Know the hub-and-spoke model** and its benefits over VPC peering
- **Understand route table** association and propagation
- **Remember cross-region peering** capabilities
- **Know integration options** with VPN and Direct Connect
- **Understand cost implications** of data processing charges