# IBM Cloud Security Engineer (C1000-178) - Network & Infrastructure Security

## Table of Contents
- [VPC Security](#vpc-security)
- [Network Access Control Lists](#network-access-control-lists)
- [Security Groups](#security-groups)
- [Virtual Private Networks](#virtual-private-networks)
- [Direct Link and Transit Gateway](#direct-link-and-transit-gateway)
- [Load Balancer Security](#load-balancer-security)
- [DDoS Protection](#ddos-protection)
- [Web Application Firewall](#web-application-firewall)
- [Container Security](#container-security)
- [Encryption](#encryption)

---

## VPC Security

### VPC Architecture

```
┌─────────────────────────────────────────────────┐
│                IBM Cloud VPC                     │
│                                                  │
│  ┌──────────────────────────────────────────┐  │
│  │            Zone 1                        │  │
│  │  ┌────────────────────────────────┐     │  │
│  │  │  Subnet (10.240.0.0/24)        │     │  │
│  │  │  - ACL: default-acl            │     │  │
│  │  │  - Public Gateway              │     │  │
│  │  │                                 │     │  │
│  │  │  ┌──────────────────────┐     │     │  │
│  │  │  │  VSI (Web Server)    │     │     │  │
│  │  │  │  - Security Group    │     │     │  │
│  │  │  │  - Floating IP       │     │     │  │
│  │  │  └──────────────────────┘     │     │  │
│  │  └────────────────────────────────┘     │  │
│  └──────────────────────────────────────────┘  │
│                                                  │
│  ┌──────────────────────────────────────────┐  │
│  │            Zone 2                        │  │
│  │  ┌────────────────────────────────┐     │  │
│  │  │  Subnet (10.240.64.0/24)       │     │  │
│  │  │  - ACL: default-acl            │     │  │
│  │  │  - No Public Gateway           │     │  │
│  │  │                                 │     │  │
│  │  │  ┌──────────────────────┐     │     │  │
│  │  │  │  VSI (App Server)    │     │     │  │
│  │  │  │  - Security Group    │     │     │  │
│  │  │  │  - Private IP only   │     │     │  │
│  │  │  └──────────────────────┘     │     │  │
│  │  └────────────────────────────────┘     │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### Creating Secure VPC

```bash
# Create VPC
ibmcloud is vpc-create my-secure-vpc \
  --resource-group-name production \
  --address-prefix-management manual

# Add address prefixes
ibmcloud is vpc-address-prefix-create 10.240.0.0/16 my-secure-vpc us-south-1
ibmcloud is vpc-address-prefix-create 10.241.0.0/16 my-secure-vpc us-south-2
ibmcloud is vpc-address-prefix-create 10.242.0.0/16 my-secure-vpc us-south-3

# Create subnets
ibmcloud is subnet-create web-tier-subnet my-secure-vpc us-south-1 \
  --ipv4-cidr-block 10.240.0.0/24

ibmcloud is subnet-create app-tier-subnet my-secure-vpc us-south-1 \
  --ipv4-cidr-block 10.240.1.0/24

ibmcloud is subnet-create data-tier-subnet my-secure-vpc us-south-1 \
  --ipv4-cidr-block 10.240.2.0/24

# Create public gateway (only for web tier)
ibmcloud is public-gateway-create web-gateway my-secure-vpc us-south-1

# Attach public gateway to subnet
ibmcloud is subnet-update web-tier-subnet \
  --public-gateway-id $GATEWAY_ID
```

### Terraform for VPC

```hcl
# vpc-security.tf
resource "ibm_is_vpc" "secure_vpc" {
  name                      = "secure-vpc"
  resource_group            = data.ibm_resource_group.prod.id
  address_prefix_management = "manual"
  tags                      = ["production", "security"]
}

resource "ibm_is_vpc_address_prefix" "zone1_prefix" {
  vpc  = ibm_is_vpc.secure_vpc.id
  zone = "us-south-1"
  cidr = "10.240.0.0/16"
}

# Web tier subnet (public)
resource "ibm_is_subnet" "web_tier" {
  name            = "web-tier-subnet"
  vpc             = ibm_is_vpc.secure_vpc.id
  zone            = "us-south-1"
  ipv4_cidr_block = "10.240.0.0/24"
  public_gateway  = ibm_is_public_gateway.web_gateway.id

  # Disable auto-assign public IP
  resource_group = data.ibm_resource_group.prod.id
}

# Application tier subnet (private)
resource "ibm_is_subnet" "app_tier" {
  name            = "app-tier-subnet"
  vpc             = ibm_is_vpc.secure_vpc.id
  zone            = "us-south-1"
  ipv4_cidr_block = "10.240.1.0/24"
  resource_group  = data.ibm_resource_group.prod.id
}

# Data tier subnet (isolated)
resource "ibm_is_subnet" "data_tier" {
  name            = "data-tier-subnet"
  vpc             = ibm_is_vpc.secure_vpc.id
  zone            = "us-south-1"
  ipv4_cidr_block = "10.240.2.0/24"
  resource_group  = data.ibm_resource_group.prod.id
}

resource "ibm_is_public_gateway" "web_gateway" {
  name           = "web-gateway"
  vpc            = ibm_is_vpc.secure_vpc.id
  zone           = "us-south-1"
  resource_group = data.ibm_resource_group.prod.id
}
```

---

## Network Access Control Lists

### ACL Rules

```bash
# Create custom ACL
ibmcloud is network-acl-create web-acl my-secure-vpc

# Allow inbound HTTPS
ibmcloud is network-acl-rule-add web-acl allow inbound tcp \
  --source-address 0.0.0.0/0 \
  --destination-address 10.240.0.0/24 \
  --destination-port-min 443 \
  --destination-port-max 443 \
  --priority 10

# Allow inbound HTTP (redirect to HTTPS)
ibmcloud is network-acl-rule-add web-acl allow inbound tcp \
  --source-address 0.0.0.0/0 \
  --destination-address 10.240.0.0/24 \
  --destination-port-min 80 \
  --destination-port-max 80 \
  --priority 20

# Allow inbound SSH from specific IP
ibmcloud is network-acl-rule-add web-acl allow inbound tcp \
  --source-address 203.0.113.0/24 \
  --destination-address 10.240.0.0/24 \
  --destination-port-min 22 \
  --destination-port-max 22 \
  --priority 30

# Allow outbound traffic
ibmcloud is network-acl-rule-add web-acl allow outbound all \
  --source-address 10.240.0.0/24 \
  --destination-address 0.0.0.0/0 \
  --priority 10

# Deny all other inbound (explicit deny)
ibmcloud is network-acl-rule-add web-acl deny inbound all \
  --source-address 0.0.0.0/0 \
  --destination-address 10.240.0.0/24 \
  --priority 1000

# Attach ACL to subnet
ibmcloud is subnet-update web-tier-subnet --network-acl web-acl
```

### Terraform for ACLs

```hcl
# network-acls.tf
resource "ibm_is_network_acl" "web_tier_acl" {
  name = "web-tier-acl"
  vpc  = ibm_is_vpc.secure_vpc.id

  # Inbound rules
  rules {
    name        = "allow-https-inbound"
    action      = "allow"
    direction   = "inbound"
    source      = "0.0.0.0/0"
    destination = "10.240.0.0/24"
    tcp {
      port_min = 443
      port_max = 443
    }
  }

  rules {
    name        = "allow-http-inbound"
    action      = "allow"
    direction   = "inbound"
    source      = "0.0.0.0/0"
    destination = "10.240.0.0/24"
    tcp {
      port_min = 80
      port_max = 80
    }
  }

  rules {
    name        = "allow-ssh-from-office"
    action      = "allow"
    direction   = "inbound"
    source      = "203.0.113.0/24"
    destination = "10.240.0.0/24"
    tcp {
      port_min = 22
      port_max = 22
    }
  }

  # Outbound rules
  rules {
    name        = "allow-all-outbound"
    action      = "allow"
    direction   = "outbound"
    source      = "10.240.0.0/24"
    destination = "0.0.0.0/0"
  }
}

resource "ibm_is_network_acl" "app_tier_acl" {
  name = "app-tier-acl"
  vpc  = ibm_is_vpc.secure_vpc.id

  # Allow inbound from web tier only
  rules {
    name        = "allow-from-web-tier"
    action      = "allow"
    direction   = "inbound"
    source      = "10.240.0.0/24"
    destination = "10.240.1.0/24"
    tcp {
      port_min = 8080
      port_max = 8080
    }
  }

  # Allow outbound to data tier
  rules {
    name        = "allow-to-data-tier"
    action      = "allow"
    direction   = "outbound"
    source      = "10.240.1.0/24"
    destination = "10.240.2.0/24"
    tcp {
      port_min = 5432
      port_max = 5432
    }
  }

  # Allow outbound HTTPS for external APIs
  rules {
    name        = "allow-https-outbound"
    action      = "allow"
    direction   = "outbound"
    source      = "10.240.1.0/24"
    destination = "0.0.0.0/0"
    tcp {
      port_min = 443
      port_max = 443
    }
  }
}

resource "ibm_is_network_acl" "data_tier_acl" {
  name = "data-tier-acl"
  vpc  = ibm_is_vpc.secure_vpc.id

  # Allow inbound from app tier only
  rules {
    name        = "allow-from-app-tier"
    action      = "allow"
    direction   = "inbound"
    source      = "10.240.1.0/24"
    destination = "10.240.2.0/24"
    tcp {
      port_min = 5432
      port_max = 5432
    }
  }

  # Deny all outbound (data tier should not initiate connections)
  rules {
    name        = "deny-all-outbound"
    action      = "deny"
    direction   = "outbound"
    source      = "10.240.2.0/24"
    destination = "0.0.0.0/0"
  }
}
```

---

## Security Groups

### Security Group Rules

```bash
# Create security group for web servers
ibmcloud is security-group-create web-sg my-secure-vpc \
  --resource-group-name production

# Allow HTTPS inbound
ibmcloud is security-group-rule-add web-sg inbound tcp \
  --port-min 443 --port-max 443 \
  --remote 0.0.0.0/0

# Allow HTTP inbound
ibmcloud is security-group-rule-add web-sg inbound tcp \
  --port-min 80 --port-max 80 \
  --remote 0.0.0.0/0

# Allow SSH from bastion
ibmcloud is security-group-rule-add web-sg inbound tcp \
  --port-min 22 --port-max 22 \
  --remote $BASTION_SG_ID

# Create security group for app servers
ibmcloud is security-group-create app-sg my-secure-vpc

# Allow traffic from web tier
ibmcloud is security-group-rule-add app-sg inbound tcp \
  --port-min 8080 --port-max 8080 \
  --remote $WEB_SG_ID

# Create security group for database
ibmcloud is security-group-create db-sg my-secure-vpc

# Allow PostgreSQL from app tier
ibmcloud is security-group-rule-add db-sg inbound tcp \
  --port-min 5432 --port-max 5432 \
  --remote $APP_SG_ID

# List security groups
ibmcloud is security-groups

# List rules in security group
ibmcloud is security-group-rules web-sg
```

### Terraform for Security Groups

```hcl
# security-groups.tf
resource "ibm_is_security_group" "bastion_sg" {
  name = "bastion-sg"
  vpc  = ibm_is_vpc.secure_vpc.id
}

resource "ibm_is_security_group_rule" "bastion_ssh_inbound" {
  group     = ibm_is_security_group.bastion_sg.id
  direction = "inbound"
  remote    = "203.0.113.0/24"  # Office IP range

  tcp {
    port_min = 22
    port_max = 22
  }
}

resource "ibm_is_security_group_rule" "bastion_all_outbound" {
  group     = ibm_is_security_group.bastion_sg.id
  direction = "outbound"
  remote    = "0.0.0.0/0"
}

resource "ibm_is_security_group" "web_sg" {
  name = "web-sg"
  vpc  = ibm_is_vpc.secure_vpc.id
}

resource "ibm_is_security_group_rule" "web_https_inbound" {
  group     = ibm_is_security_group.web_sg.id
  direction = "inbound"
  remote    = "0.0.0.0/0"

  tcp {
    port_min = 443
    port_max = 443
  }
}

resource "ibm_is_security_group_rule" "web_http_inbound" {
  group     = ibm_is_security_group.web_sg.id
  direction = "inbound"
  remote    = "0.0.0.0/0"

  tcp {
    port_min = 80
    port_max = 80
  }
}

resource "ibm_is_security_group_rule" "web_ssh_from_bastion" {
  group     = ibm_is_security_group.web_sg.id
  direction = "inbound"
  remote    = ibm_is_security_group.bastion_sg.id

  tcp {
    port_min = 22
    port_max = 22
  }
}

resource "ibm_is_security_group_rule" "web_all_outbound" {
  group     = ibm_is_security_group.web_sg.id
  direction = "outbound"
  remote    = "0.0.0.0/0"
}

resource "ibm_is_security_group" "app_sg" {
  name = "app-sg"
  vpc  = ibm_is_vpc.secure_vpc.id
}

resource "ibm_is_security_group_rule" "app_from_web" {
  group     = ibm_is_security_group.app_sg.id
  direction = "inbound"
  remote    = ibm_is_security_group.web_sg.id

  tcp {
    port_min = 8080
    port_max = 8080
  }
}

resource "ibm_is_security_group_rule" "app_ssh_from_bastion" {
  group     = ibm_is_security_group.app_sg.id
  direction = "inbound"
  remote    = ibm_is_security_group.bastion_sg.id

  tcp {
    port_min = 22
    port_max = 22
  }
}

resource "ibm_is_security_group_rule" "app_all_outbound" {
  group     = ibm_is_security_group.app_sg.id
  direction = "outbound"
  remote    = "0.0.0.0/0"
}

resource "ibm_is_security_group" "db_sg" {
  name = "db-sg"
  vpc  = ibm_is_vpc.secure_vpc.id
}

resource "ibm_is_security_group_rule" "db_from_app" {
  group     = ibm_is_security_group.db_sg.id
  direction = "inbound"
  remote    = ibm_is_security_group.app_sg.id

  tcp {
    port_min = 5432
    port_max = 5432
  }
}

resource "ibm_is_security_group_rule" "db_ssh_from_bastion" {
  group     = ibm_is_security_group.db_sg.id
  direction = "inbound"
  remote    = ibm_is_security_group.bastion_sg.id

  tcp {
    port_min = 22
    port_max = 22
  }
}

# No outbound rule for database (deny by default)
```

---

## Virtual Private Networks

### VPN for VPC

```bash
# Create VPN gateway
ibmcloud is vpn-gateway-create my-vpn-gateway \
  --subnet $SUBNET_ID \
  --mode policy

# Create VPN connection
ibmcloud is vpn-gateway-connection-create connection-1 $VPN_GATEWAY_ID \
  --peer-address 203.0.113.50 \
  --preshared-key "super-secret-key" \
  --local-cidrs 10.240.0.0/16 \
  --peer-cidrs 192.168.0.0/16

# Create IKE policy
ibmcloud is ike-policy-create my-ike-policy \
  --authentication-algorithm sha256 \
  --encryption-algorithm aes256 \
  --dh-group 14 \
  --ike-version 2

# Create IPsec policy
ibmcloud is ipsec-policy-create my-ipsec-policy \
  --authentication-algorithm sha256 \
  --encryption-algorithm aes256 \
  --pfs 14

# Update VPN connection with policies
ibmcloud is vpn-gateway-connection-update $VPN_GATEWAY_ID connection-1 \
  --ike-policy $IKE_POLICY_ID \
  --ipsec-policy $IPSEC_POLICY_ID

# Check VPN status
ibmcloud is vpn-gateway-connection $VPN_GATEWAY_ID connection-1
```

### Terraform for VPN

```hcl
# vpn.tf
resource "ibm_is_vpn_gateway" "vpn" {
  name   = "my-vpn-gateway"
  subnet = ibm_is_subnet.web_tier.id
  mode   = "policy"
}

resource "ibm_is_ike_policy" "ike_policy" {
  name                     = "my-ike-policy"
  authentication_algorithm = "sha256"
  encryption_algorithm     = "aes256"
  dh_group                 = 14
  ike_version              = 2
  key_lifetime             = 28800
}

resource "ibm_is_ipsec_policy" "ipsec_policy" {
  name                     = "my-ipsec-policy"
  authentication_algorithm = "sha256"
  encryption_algorithm     = "aes256"
  pfs                      = "group_14"
  key_lifetime             = 3600
}

resource "ibm_is_vpn_gateway_connection" "vpn_connection" {
  name           = "onprem-connection"
  vpn_gateway    = ibm_is_vpn_gateway.vpn.id
  peer_address   = "203.0.113.50"
  preshared_key  = var.vpn_preshared_key
  local_cidrs    = ["10.240.0.0/16"]
  peer_cidrs     = ["192.168.0.0/16"]
  ike_policy     = ibm_is_ike_policy.ike_policy.id
  ipsec_policy   = ibm_is_ipsec_policy.ipsec_policy.id
  admin_state_up = true
}
```

### Client VPN

```bash
# Create VPN server
ibmcloud is vpn-server-create my-vpn-server \
  --subnet $SUBNET_ID \
  --certificate-crn $CERT_CRN \
  --client-ip-pool 172.16.0.0/16 \
  --client-dns-server-ips 8.8.8.8,8.8.4.4 \
  --port 443

# Add route for VPN server
ibmcloud is vpn-server-route-create $VPN_SERVER_ID \
  --destination 10.240.0.0/16 \
  --action translate

# Create client authentication
ibmcloud is vpn-server-client-create $VPN_SERVER_ID user1 \
  --client-authentication certificate

# Get VPN client configuration
ibmcloud is vpn-server-client-configuration $VPN_SERVER_ID
```

---

## Direct Link and Transit Gateway

### Direct Link

```bash
# Create Direct Link connection
ibmcloud dl gateway-create \
  --name my-direct-link \
  --type dedicated \
  --speed-mbps 1000 \
  --bgp-asn 64999 \
  --bgp-base-cidr 10.254.0.0/16 \
  --metered true \
  --carrier-name "Network Provider" \
  --customer-name "My Company" \
  --location dal10

# Create virtual connection to VPC
ibmcloud dl gateway-vc-create $GATEWAY_ID \
  --name vpc-connection \
  --type vpc \
  --network-id $VPC_CRN

# List Direct Link gateways
ibmcloud dl gateways

# View BGP status
ibmcloud dl gateway-status $GATEWAY_ID
```

### Transit Gateway

```bash
# Create Transit Gateway
ibmcloud tg gateway-create \
  --name my-transit-gateway \
  --location us-south \
  --routing global

# Add VPC connection
ibmcloud tg connection-create $TG_ID \
  --name vpc1-connection \
  --network-type vpc \
  --network-id $VPC1_CRN

# Add another VPC
ibmcloud tg connection-create $TG_ID \
  --name vpc2-connection \
  --network-type vpc \
  --network-id $VPC2_CRN

# Add Direct Link connection
ibmcloud tg connection-create $TG_ID \
  --name directlink-connection \
  --network-type directlink \
  --network-id $DL_ID

# List connections
ibmcloud tg connections $TG_ID
```

### Terraform for Transit Gateway

```hcl
# transit-gateway.tf
resource "ibm_tg_gateway" "transit_gateway" {
  name     = "my-transit-gateway"
  location = "us-south"
  global   = true
}

resource "ibm_tg_connection" "vpc1_connection" {
  gateway      = ibm_tg_gateway.transit_gateway.id
  network_type = "vpc"
  name         = "vpc1-connection"
  network_id   = ibm_is_vpc.vpc1.crn
}

resource "ibm_tg_connection" "vpc2_connection" {
  gateway      = ibm_tg_gateway.transit_gateway.id
  network_type = "vpc"
  name         = "vpc2-connection"
  network_id   = ibm_is_vpc.vpc2.crn
}

resource "ibm_tg_connection" "classic_connection" {
  gateway      = ibm_tg_gateway.transit_gateway.id
  network_type = "classic"
  name         = "classic-connection"
}
```

---

## Load Balancer Security

### Application Load Balancer

```bash
# Create load balancer
ibmcloud is load-balancer-create alb \
  --subnet $SUBNET_ID \
  --family application \
  --logging-datapath-active true

# Create backend pool
ibmcloud is load-balancer-pool-create web-pool $LB_ID \
  --algorithm round_robin \
  --protocol https \
  --health-delay 5 \
  --health-retries 2 \
  --health-timeout 2 \
  --health-type https \
  --health-monitor-url /health \
  --session-persistence cookie

# Add members to pool
ibmcloud is load-balancer-pool-member-create $LB_ID web-pool \
  --port 443 \
  --target $VSI_ID

# Create HTTPS listener with SSL termination
ibmcloud is load-balancer-listener-create $LB_ID \
  --port 443 \
  --protocol https \
  --default-pool web-pool \
  --certificate-instance $CERT_CRN \
  --connection-limit 2000

# Create HTTP listener (redirect to HTTPS)
ibmcloud is load-balancer-listener-create $LB_ID \
  --port 80 \
  --protocol http \
  --default-pool web-pool
```

### Terraform for Load Balancer

```hcl
# load-balancer.tf
resource "ibm_is_lb" "alb" {
  name    = "application-lb"
  subnets = [ibm_is_subnet.web_tier.id]
  type    = "public"
  profile = "network-fixed"

  logging {
    datapath = true
  }
}

resource "ibm_is_lb_pool" "web_pool" {
  lb                 = ibm_is_lb.alb.id
  name               = "web-pool"
  protocol           = "https"
  algorithm          = "round_robin"
  health_delay       = 5
  health_retries     = 2
  health_timeout     = 2
  health_type        = "https"
  health_monitor_url = "/health"

  session_persistence_type = "app_cookie"
  session_persistence_cookie_name = "LB_SESSION"
}

resource "ibm_is_lb_pool_member" "web_members" {
  count          = 3
  lb             = ibm_is_lb.alb.id
  pool           = ibm_is_lb_pool.web_pool.id
  port           = 443
  target_address = ibm_is_instance.web[count.index].primary_network_interface[0].primary_ipv4_address
}

resource "ibm_is_lb_listener" "https_listener" {
  lb                   = ibm_is_lb.alb.id
  port                 = 443
  protocol             = "https"
  certificate_instance = ibm_sm_imported_certificate.ssl_cert.crn
  default_pool         = ibm_is_lb_pool.web_pool.id
  connection_limit     = 2000

  https_redirect {
    http_status_code = 301
    listener         = ibm_is_lb_listener.http_listener.id
    uri              = "/"
  }
}

resource "ibm_is_lb_listener" "http_listener" {
  lb               = ibm_is_lb.alb.id
  port             = 80
  protocol         = "http"
  default_pool     = ibm_is_lb_pool.web_pool.id
  connection_limit = 2000
}

# Listener policy for path-based routing
resource "ibm_is_lb_listener_policy" "api_policy" {
  lb       = ibm_is_lb.alb.id
  listener = ibm_is_lb_listener.https_listener.id
  action   = "forward"
  priority = 1
  name     = "api-routing"
  target   = ibm_is_lb_pool.api_pool.id

  rules {
    condition = "contains"
    type      = "path"
    value     = "/api"
  }
}
```

---

## DDoS Protection

### Cloud Internet Services (CIS)

```bash
# Create CIS instance
ibmcloud resource service-instance-create \
  my-cis \
  internet-svcs \
  enterprise \
  global

# Add domain to CIS
ibmcloud cis domain-add example.com \
  --instance my-cis

# Enable DDoS protection
ibmcloud cis firewall-set example.com \
  --mode high

# Configure rate limiting
ibmcloud cis rate-limit-create example.com \
  --match '{"request":{"methods":["POST"],"schemes":["HTTPS"],"url":"*/login"}}' \
  --threshold 10 \
  --period 60 \
  --action challenge
```

### Terraform for CIS

```hcl
# cis-security.tf
resource "ibm_cis" "cis_instance" {
  name     = "my-cis"
  plan     = "enterprise-usage"
  location = "global"
}

resource "ibm_cis_domain" "domain" {
  cis_id = ibm_cis.cis_instance.id
  domain = "example.com"
}

# DDoS settings
resource "ibm_cis_firewall" "firewall" {
  cis_id    = ibm_cis.cis_instance.id
  domain_id = ibm_cis_domain.domain.id
  mode      = "high"
}

# Rate limiting
resource "ibm_cis_rate_limit" "login_rate_limit" {
  cis_id    = ibm_cis.cis_instance.id
  domain_id = ibm_cis_domain.domain.id

  threshold = 10
  period    = 60
  action {
    mode    = "challenge"
    timeout = 86400
  }

  match {
    request {
      url     = "*.example.com/login"
      schemes = ["HTTPS"]
      methods = ["POST"]
    }
  }

  description = "Rate limit login attempts"
}

# IP firewall rules
resource "ibm_cis_firewall_rules" "firewall_rules" {
  cis_id    = ibm_cis.cis_instance.id
  domain_id = ibm_cis_domain.domain.id

  firewall_rules {
    priority    = 1
    action      = "block"
    description = "Block malicious IPs"
    filter {
      expression = "(ip.src in {1.2.3.4 5.6.7.8})"
    }
  }

  firewall_rules {
    priority    = 2
    action      = "challenge"
    description = "Challenge suspicious requests"
    filter {
      expression = "(cf.threat_score > 50)"
    }
  }
}
```

---

## Web Application Firewall

### CIS WAF Configuration

```bash
# Enable WAF
ibmcloud cis waf-set example.com on

# Configure WAF rules
ibmcloud cis waf-group-set example.com OWASP on

# Set sensitivity
ibmcloud cis waf-package-set example.com owasp-modsecurity-crs \
  --sensitivity high

# Create custom WAF rule
ibmcloud cis waf-rule-create example.com \
  --expression '(http.request.uri.path contains "admin")' \
  --action block
```

### Terraform for WAF

```hcl
# waf.tf
resource "ibm_cis_waf_rule" "custom_rule" {
  cis_id    = ibm_cis.cis_instance.id
  domain_id = ibm_cis_domain.domain.id

  mode = "on"

  filter {
    expression = "(http.request.uri.path contains \"/admin\")"
  }

  action = "block"
}

resource "ibm_cis_waf_package" "owasp" {
  cis_id    = ibm_cis.cis_instance.id
  domain_id = ibm_cis_domain.domain.id
  package_id = "a25a9a7e9c00afc1fb2e0245519d725b"

  sensitivity = "high"
  action_mode = "block"
}
```

---

## Container Security

### Container Registry Security

```bash
# Enable Vulnerability Advisor
ibmcloud cr va-enable

# Scan image
ibmcloud cr vulnerability-assessment us.icr.io/namespace/myapp:latest

# View vulnerabilities
ibmcloud cr vulnerability-assessment us.icr.io/namespace/myapp:latest --output json

# Set security policy
ibmcloud cr image-prune --restrict

# Enable image signing (Notary)
ibmcloud cr image-sign us.icr.io/namespace/myapp:latest
```

### Kubernetes Pod Security

```yaml
# pod-security-policy.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
```

---

## Encryption

### Encryption at Rest

```bash
# Create Key Protect instance
ibmcloud resource service-instance-create \
  my-key-protect \
  kms \
  tiered-pricing \
  us-south

# Create root key
ibmcloud kp key-create my-root-key \
  --instance-id $INSTANCE_ID \
  --key-material $(openssl rand -base64 32)

# Enable encryption for COS bucket
ibmcloud cos bucket-kp-enable my-bucket \
  --crn $KEY_CRN
```

### Encryption in Transit

```bash
# Force HTTPS on load balancer
# All traffic between load balancer and clients uses TLS 1.2+

# Enable mutual TLS
kubectl create secret tls mtls-cert \
  --cert=client-cert.pem \
  --key=client-key.pem
```

### Exam Tips

1. **VPC Security**: Know ACLs vs Security Groups, subnet isolation
2. **Network Security**: Understand VPN configurations, Direct Link
3. **Load Balancer**: Master SSL termination, health checks
4. **DDoS/WAF**: Know CIS capabilities, rate limiting
5. **Container Security**: Vulnerability scanning, pod security
6. **Encryption**: Key Protect, BYOK, KYOK concepts
