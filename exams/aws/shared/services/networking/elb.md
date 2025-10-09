# Elastic Load Balancing (ELB)

## Service Overview and Purpose

Elastic Load Balancing automatically distributes incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, IP addresses, Lambda functions, and virtual appliances. ELB provides fault tolerance for applications by seamlessly providing the required amount of load balancing capacity needed to route application traffic.

### Load Balancer Types

#### Application Load Balancer (ALB)
- **Layer 7**: HTTP/HTTPS traffic
- **Advanced Routing**: Path-based and host-based routing
- **Targets**: EC2 instances, IP addresses, Lambda functions, containers
- **Features**: SSL termination, WebSocket, HTTP/2, gRPC support

#### Network Load Balancer (NLB)
- **Layer 4**: TCP, UDP, TLS traffic
- **High Performance**: Millions of requests per second, ultra-low latency
- **Static IP**: Elastic IP address per AZ
- **Targets**: EC2 instances, IP addresses, ALB

#### Gateway Load Balancer (GWLB)
- **Layer 3**: Network layer gateway and load balancer
- **Third-Party Appliances**: Firewalls, intrusion detection systems
- **GENEVE Protocol**: Encapsulation and decapsulation
- **Transparent**: Preserves original packet headers

#### Classic Load Balancer (CLB)
- **Legacy**: Layer 4 and Layer 7 support
- **Simple**: Basic load balancing for EC2-Classic
- **Limited Features**: Compared to ALB/NLB
- **Not Recommended**: For new applications

## Configuration Examples

### Application Load Balancer Setup
```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name production-alb \
  --subnets subnet-12345678 subnet-87654321 \
  --security-groups sg-12345678 \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --tags Key=Environment,Value=Production

# Create target group
aws elbv2 create-target-group \
  --name web-servers \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-12345678 \
  --health-check-protocol HTTP \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --target-type instance

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:region:account:loadbalancer/app/production-alb/1234567890123456 \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:region:account:certificate/cert-id \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/web-servers/1234567890123456

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:region:account:targetgroup/web-servers/1234567890123456 \
  --targets Id=i-12345678,Port=80 Id=i-87654321,Port=80
```

### Advanced ALB Rules
```bash
# Path-based routing
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:region:account:listener/app/production-alb/1234567890123456/1234567890123456 \
  --priority 100 \
  --conditions Field=path-pattern,Values='/api/*' \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/api-servers/1234567890123456

# Host-based routing
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:region:account:listener/app/production-alb/1234567890123456/1234567890123456 \
  --priority 200 \
  --conditions Field=host-header,Values=admin.example.com \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/admin-servers/1234567890123456

# Lambda function target
aws elbv2 create-target-group \
  --name lambda-targets \
  --target-type lambda \
  --lambda-event-structure-version 2.0

aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:region:account:targetgroup/lambda-targets/1234567890123456 \
  --targets Id=arn:aws:lambda:region:account:function:my-function
```

### Network Load Balancer Setup
```bash
# Create NLB
aws elbv2 create-load-balancer \
  --name production-nlb \
  --scheme internet-facing \
  --type network \
  --subnets subnet-12345678 subnet-87654321 \
  --tags Key=Environment,Value=Production

# Create target group for NLB
aws elbv2 create-target-group \
  --name tcp-servers \
  --protocol TCP \
  --port 80 \
  --vpc-id vpc-12345678 \
  --health-check-protocol TCP \
  --health-check-port 80 \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 3 \
  --unhealthy-threshold-count 3 \
  --target-type instance

# Create listener for NLB
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:region:account:loadbalancer/net/production-nlb/1234567890123456 \
  --protocol TCP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/tcp-servers/1234567890123456
```

## Advanced Features

### SSL/TLS Configuration
```bash
# SSL termination on ALB
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:region:account:listener/app/production-alb/1234567890123456/1234567890123456 \
  --ssl-policy ELBSecurityPolicy-TLS-1-2-2017-01 \
  --certificates CertificateArn=arn:aws:acm:region:account:certificate/cert-id

# SSL passthrough on NLB
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:region:account:loadbalancer/net/production-nlb/1234567890123456 \
  --protocol TLS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:region:account:certificate/cert-id \
  --ssl-policy ELBSecurityPolicy-TLS-1-2-2017-01 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/tls-servers/1234567890123456
```

### Auto Scaling Integration
```bash
# Attach load balancer to Auto Scaling group
aws autoscaling attach-load-balancer-target-groups \
  --auto-scaling-group-name production-asg \
  --target-group-arns arn:aws:elasticloadbalancing:region:account:targetgroup/web-servers/1234567890123456

# Create Auto Scaling group with load balancer
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name web-asg \
  --launch-template LaunchTemplateName=web-template,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 4 \
  --target-group-arns arn:aws:elasticloadbalancing:region:account:targetgroup/web-servers/1234567890123456 \
  --vpc-zone-identifier subnet-12345678,subnet-87654321 \
  --health-check-type ELB \
  --health-check-grace-period 300
```

## Use Cases and Scenarios

### Primary Use Cases
1. **High Availability**: Distribute traffic across multiple targets
2. **Scalability**: Handle varying traffic loads
3. **Security**: SSL termination and WAF integration
4. **Microservices**: Route requests to different services
5. **Blue-Green Deployments**: Traffic shifting between environments

### Architecture Patterns

#### Multi-Tier Application
```bash
# Internet-facing ALB for web tier
# Internal ALB for application tier
# NLB for database connections

# Web tier ALB (public)
aws elbv2 create-load-balancer \
  --name web-tier-alb \
  --scheme internet-facing \
  --type application \
  --subnets subnet-public1 subnet-public2 \
  --security-groups sg-web-alb

# App tier ALB (internal)
aws elbv2 create-load-balancer \
  --name app-tier-alb \
  --scheme internal \
  --type application \
  --subnets subnet-private1 subnet-private2 \
  --security-groups sg-app-alb

# Database NLB (internal)
aws elbv2 create-load-balancer \
  --name db-tier-nlb \
  --scheme internal \
  --type network \
  --subnets subnet-db1 subnet-db2
```

#### Container Applications
```bash
# ALB with dynamic port mapping for ECS
aws elbv2 create-target-group \
  --name ecs-services \
  --protocol HTTP \
  --vpc-id vpc-12345678 \
  --health-check-path /health \
  --target-type ip \
  --health-check-protocol HTTP

# ECS service with ALB integration
aws ecs create-service \
  --cluster production-cluster \
  --service-name web-service \
  --task-definition web-task:1 \
  --desired-count 3 \
  --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/ecs-services/1234567890123456,containerName=web-container,containerPort=80
```

## Monitoring and Troubleshooting

### CloudWatch Metrics
```bash
# Monitor ALB performance
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name RequestCount \
  --dimensions Name=LoadBalancer,Value=app/production-alb/1234567890123456 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Sum

# Monitor target health
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name HealthyHostCount \
  --dimensions Name=TargetGroup,Value=targetgroup/web-servers/1234567890123456 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Average

# Monitor response times
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name TargetResponseTime \
  --dimensions Name=LoadBalancer,Value=app/production-alb/1234567890123456 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-01T23:59:59Z \
  --period 300 \
  --statistics Average
```

### Access Logs
```bash
# Enable access logs
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:aws:elasticloadbalancing:region:account:loadbalancer/app/production-alb/1234567890123456 \
  --attributes Key=access_logs.s3.enabled,Value=true Key=access_logs.s3.bucket,Value=my-alb-logs Key=access_logs.s3.prefix,Value=production-alb

# Analyze access logs with Athena
CREATE EXTERNAL TABLE alb_logs (
  type string,
  time string,
  elb string,
  client_ip string,
  client_port int,
  target_ip string,
  target_port int,
  request_processing_time double,
  target_processing_time double,
  response_processing_time double,
  elb_status_code string,
  target_status_code string,
  received_bytes bigint,
  sent_bytes bigint,
  request_verb string,
  request_url string,
  request_proto string,
  user_agent string,
  ssl_cipher string,
  ssl_protocol string,
  target_group_arn string,
  trace_id string,
  domain_name string,
  chosen_cert_arn string,
  matched_rule_priority string,
  request_creation_time string,
  actions_executed string,
  redirect_url string,
  lambda_error_reason string,
  target_port_list string,
  target_status_code_list string,
  classification string,
  classification_reason string
)
PARTITIONED BY(year string, month string, day string)
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://my-alb-logs/production-alb/AWSLogs/account-id/elasticloadbalancing/region/';
```

## Security Best Practices

### Security Groups
```bash
# ALB security group
aws ec2 create-security-group \
  --group-name alb-security-group \
  --description "Security group for ALB" \
  --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
  --group-id sg-alb123 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id sg-alb123 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Backend security group (only allow traffic from ALB)
aws ec2 create-security-group \
  --group-name backend-security-group \
  --description "Security group for backend instances" \
  --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
  --group-id sg-backend456 \
  --protocol tcp \
  --port 80 \
  --source-group sg-alb123
```

### WAF Integration
```bash
# Associate WAF with ALB
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:region:account:global/webacl/name/id \
  --resource-arn arn:aws:elasticloadbalancing:region:account:loadbalancer/app/production-alb/1234567890123456
```

## Exam-Specific Tips

### Key Exam Topics
1. **Load Balancer Types**: ALB vs NLB vs GWLB vs CLB
2. **Target Types**: EC2, IP, Lambda functions
3. **Health Checks**: Configuration and troubleshooting
4. **SSL/TLS**: Termination vs passthrough
5. **Advanced Routing**: Path-based, host-based, weighted

### Common Exam Scenarios
1. **Microservices Architecture**: Use ALB with path-based routing
2. **High-Performance Applications**: Use NLB for layer 4 load balancing
3. **SSL Termination**: Use ALB to offload SSL processing
4. **Blue-Green Deployment**: Use weighted target groups
5. **Container Applications**: Use ALB with dynamic port mapping

### Exam Tips
- **Know the differences** between load balancer types
- **Understand target types** and their use cases
- **Remember health check** configuration options
- **Know SSL/TLS options** for each load balancer type
- **Understand integration** with Auto Scaling and ECS