# AWS CLI Cheat Sheet

A comprehensive reference for AWS Command Line Interface commands covering essential services for cloud certification and daily operations.

## Table of Contents
- [Configuration & Setup](#configuration--setup)
- [EC2 (Elastic Compute Cloud)](#ec2-elastic-compute-cloud)
- [S3 (Simple Storage Service)](#s3-simple-storage-service)
- [IAM (Identity and Access Management)](#iam-identity-and-access-management)
- [VPC (Virtual Private Cloud)](#vpc-virtual-private-cloud)
- [Lambda](#lambda)
- [RDS (Relational Database Service)](#rds-relational-database-service)
- [DynamoDB](#dynamodb)
- [ECS (Elastic Container Service)](#ecs-elastic-container-service)
- [CloudFormation](#cloudformation)
- [CloudWatch](#cloudwatch)
- [Additional Services](#additional-services)

---

## Configuration & Setup

### Configure AWS CLI
```bash
# Configure default profile
aws configure

# Configure named profile
aws configure --profile production

# List configured profiles
aws configure list-profiles

# Set default region
aws configure set region us-west-2

# Set output format (json, yaml, text, table)
aws configure set output json

# View current configuration
aws configure list

# Get caller identity
aws sts get-caller-identity

# Assume role
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MyRole --role-session-name mysession
```

---

## EC2 (Elastic Compute Cloud)

### Instance Management
```bash
# List all instances
aws ec2 describe-instances

# List instances with specific state
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"

# List instances with specific tag
aws ec2 describe-instances --filters "Name=tag:Environment,Values=production"

# Get instance details by ID
aws ec2 describe-instances --instance-ids i-1234567890abcdef0

# Launch new instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Get instance console output
aws ec2 get-console-output --instance-id i-1234567890abcdef0

# Modify instance attribute
aws ec2 modify-instance-attribute --instance-id i-1234567890abcdef0 --instance-type t3.small

# Change instance type (stop required)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 modify-instance-attribute --instance-id i-1234567890abcdef0 --instance-type "{\"Value\":\"t3.medium\"}"
aws ec2 start-instances --instance-ids i-1234567890abcdef0
```

### AMI Management
```bash
# List AMIs owned by account
aws ec2 describe-images --owners self

# List public Amazon Linux AMIs
aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*"

# Create AMI from instance
aws ec2 create-image --instance-id i-1234567890abcdef0 --name "MyAMI" --description "My custom AMI"

# Copy AMI to another region
aws ec2 copy-image --source-region us-east-1 --source-image-id ami-0123456789abcdef0 --name "CopiedAMI"

# Deregister AMI
aws ec2 deregister-image --image-id ami-0123456789abcdef0

# Share AMI with another account
aws ec2 modify-image-attribute --image-id ami-0123456789abcdef0 --launch-permission "Add=[{UserId=123456789012}]"
```

### Security Groups
```bash
# List security groups
aws ec2 describe-security-groups

# Create security group
aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id vpc-0123456789abcdef0

# Add inbound rule (SSH)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Add inbound rule (HTTP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Add inbound rule from another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-9876543210fedcba0

# Remove inbound rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Add outbound rule
aws ec2 authorize-security-group-egress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Delete security group
aws ec2 delete-security-group --group-id sg-0123456789abcdef0
```

### Key Pairs
```bash
# List key pairs
aws ec2 describe-key-pairs

# Create key pair
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem

# Import key pair
aws ec2 import-key-pair --key-name MyImportedKey --public-key-material fileb://~/.ssh/id_rsa.pub

# Delete key pair
aws ec2 delete-key-pair --key-name MyKeyPair
```

### EBS Volumes
```bash
# List volumes
aws ec2 describe-volumes

# Create volume
aws ec2 create-volume --size 80 --region us-east-1 --availability-zone us-east-1a --volume-type gp3

# Attach volume to instance
aws ec2 attach-volume --volume-id vol-0123456789abcdef0 --instance-id i-1234567890abcdef0 --device /dev/sdf

# Detach volume
aws ec2 detach-volume --volume-id vol-0123456789abcdef0

# Delete volume
aws ec2 delete-volume --volume-id vol-0123456789abcdef0

# Create snapshot
aws ec2 create-snapshot --volume-id vol-0123456789abcdef0 --description "My snapshot"

# List snapshots
aws ec2 describe-snapshots --owner-ids self

# Copy snapshot to another region
aws ec2 copy-snapshot --source-region us-east-1 --source-snapshot-id snap-0123456789abcdef0 --destination-region us-west-2

# Create volume from snapshot
aws ec2 create-volume --snapshot-id snap-0123456789abcdef0 --availability-zone us-east-1a

# Delete snapshot
aws ec2 delete-snapshot --snapshot-id snap-0123456789abcdef0
```

### Elastic IPs
```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Associate Elastic IP with instance
aws ec2 associate-address --instance-id i-1234567890abcdef0 --allocation-id eipalloc-0123456789abcdef0

# Disassociate Elastic IP
aws ec2 disassociate-address --association-id eipassoc-0123456789abcdef0

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-0123456789abcdef0

# List Elastic IPs
aws ec2 describe-addresses
```

---

## S3 (Simple Storage Service)

### Bucket Management
```bash
# List all buckets
aws s3 ls

# Create bucket
aws s3 mb s3://my-unique-bucket-name

# Create bucket in specific region
aws s3 mb s3://my-unique-bucket-name --region us-west-2

# Remove empty bucket
aws s3 rb s3://my-unique-bucket-name

# Remove bucket and all contents
aws s3 rb s3://my-unique-bucket-name --force

# List bucket contents
aws s3 ls s3://my-bucket-name

# List bucket contents recursively
aws s3 ls s3://my-bucket-name --recursive

# List with human-readable sizes
aws s3 ls s3://my-bucket-name --human-readable --summarize
```

### File Operations
```bash
# Upload file
aws s3 cp myfile.txt s3://my-bucket-name/

# Upload with server-side encryption
aws s3 cp myfile.txt s3://my-bucket-name/ --sse AES256

# Upload directory recursively
aws s3 cp myfolder s3://my-bucket-name/myfolder --recursive

# Download file
aws s3 cp s3://my-bucket-name/myfile.txt .

# Download directory
aws s3 cp s3://my-bucket-name/myfolder . --recursive

# Sync local to S3
aws s3 sync myfolder s3://my-bucket-name/myfolder

# Sync S3 to local
aws s3 sync s3://my-bucket-name/myfolder myfolder

# Sync with delete (remove files not in source)
aws s3 sync myfolder s3://my-bucket-name/myfolder --delete

# Move file
aws s3 mv myfile.txt s3://my-bucket-name/

# Remove file
aws s3 rm s3://my-bucket-name/myfile.txt

# Remove all files in folder
aws s3 rm s3://my-bucket-name/myfolder --recursive
```

### Bucket Configuration
```bash
# Get bucket versioning
aws s3api get-bucket-versioning --bucket my-bucket-name

# Enable bucket versioning
aws s3api put-bucket-versioning --bucket my-bucket-name --versioning-configuration Status=Enabled

# Get bucket encryption
aws s3api get-bucket-encryption --bucket my-bucket-name

# Enable default encryption
aws s3api put-bucket-encryption --bucket my-bucket-name --server-side-encryption-configuration '{
  "Rules": [{
    "ApplyServerSideEncryptionByDefault": {
      "SSEAlgorithm": "AES256"
    }
  }]
}'

# Get bucket policy
aws s3api get-bucket-policy --bucket my-bucket-name

# Put bucket policy
aws s3api put-bucket-policy --bucket my-bucket-name --policy file://policy.json

# Delete bucket policy
aws s3api delete-bucket-policy --bucket my-bucket-name

# Get bucket ACL
aws s3api get-bucket-acl --bucket my-bucket-name

# Make bucket public (not recommended)
aws s3api put-bucket-acl --bucket my-bucket-name --acl public-read

# Block public access
aws s3api put-public-access-block --bucket my-bucket-name --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Get bucket lifecycle configuration
aws s3api get-bucket-lifecycle-configuration --bucket my-bucket-name

# Put bucket lifecycle configuration
aws s3api put-bucket-lifecycle-configuration --bucket my-bucket-name --lifecycle-configuration file://lifecycle.json

# Enable transfer acceleration
aws s3api put-bucket-accelerate-configuration --bucket my-bucket-name --accelerate-configuration Status=Enabled

# Enable bucket logging
aws s3api put-bucket-logging --bucket my-bucket-name --bucket-logging-status '{
  "LoggingEnabled": {
    "TargetBucket": "my-log-bucket",
    "TargetPrefix": "logs/"
  }
}'

# Get bucket replication
aws s3api get-bucket-replication --bucket my-bucket-name

# Enable static website hosting
aws s3api put-bucket-website --bucket my-bucket-name --website-configuration '{
  "IndexDocument": {"Suffix": "index.html"},
  "ErrorDocument": {"Key": "error.html"}
}'

# Get bucket location
aws s3api get-bucket-location --bucket my-bucket-name
```

### Object Operations
```bash
# Get object metadata
aws s3api head-object --bucket my-bucket-name --key myfile.txt

# Copy object within S3
aws s3api copy-object --bucket my-bucket-name --key newfile.txt --copy-source my-bucket-name/myfile.txt

# Generate presigned URL (valid for 1 hour)
aws s3 presign s3://my-bucket-name/myfile.txt --expires-in 3600

# Get object tagging
aws s3api get-object-tagging --bucket my-bucket-name --key myfile.txt

# Put object tagging
aws s3api put-object-tagging --bucket my-bucket-name --key myfile.txt --tagging 'TagSet=[{Key=Environment,Value=Production}]'

# List object versions
aws s3api list-object-versions --bucket my-bucket-name --prefix myfile.txt

# Restore from Glacier
aws s3api restore-object --bucket my-bucket-name --key myfile.txt --restore-request Days=7
```

---

## IAM (Identity and Access Management)

### User Management
```bash
# List users
aws iam list-users

# Create user
aws iam create-user --user-name john

# Get user details
aws iam get-user --user-name john

# Delete user
aws iam delete-user --user-name john

# Create access key for user
aws iam create-access-key --user-name john

# List access keys for user
aws iam list-access-keys --user-name john

# Delete access key
aws iam delete-access-key --user-name john --access-key-id AKIAIOSFODNN7EXAMPLE

# Update access key status
aws iam update-access-key --user-name john --access-key-id AKIAIOSFODNN7EXAMPLE --status Inactive

# Create login profile (console password)
aws iam create-login-profile --user-name john --password MyPassword123!

# Update login profile
aws iam update-login-profile --user-name john --password NewPassword456!

# Delete login profile
aws iam delete-login-profile --user-name john

# Add user to group
aws iam add-user-to-group --user-name john --group-name developers

# Remove user from group
aws iam remove-user-from-group --user-name john --group-name developers

# List groups for user
aws iam list-groups-for-user --user-name john
```

### Group Management
```bash
# List groups
aws iam list-groups

# Create group
aws iam create-group --group-name developers

# Get group details
aws iam get-group --group-name developers

# Delete group
aws iam delete-group --group-name developers

# List users in group
aws iam get-group --group-name developers

# Attach policy to group
aws iam attach-group-policy --group-name developers --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Detach policy from group
aws iam detach-group-policy --group-name developers --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# List attached group policies
aws iam list-attached-group-policies --group-name developers
```

### Role Management
```bash
# List roles
aws iam list-roles

# Create role
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust-policy.json

# Get role details
aws iam get-role --role-name MyRole

# Delete role
aws iam delete-role --role-name MyRole

# Attach policy to role
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Detach policy from role
aws iam detach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# List attached role policies
aws iam list-attached-role-policies --role-name MyRole

# Update assume role policy
aws iam update-assume-role-policy --role-name MyRole --policy-document file://new-trust-policy.json

# Create instance profile
aws iam create-instance-profile --instance-profile-name MyInstanceProfile

# Add role to instance profile
aws iam add-role-to-instance-profile --instance-profile-name MyInstanceProfile --role-name MyRole

# Remove role from instance profile
aws iam remove-role-from-instance-profile --instance-profile-name MyInstanceProfile --role-name MyRole

# Delete instance profile
aws iam delete-instance-profile --instance-profile-name MyInstanceProfile
```

### Policy Management
```bash
# List policies
aws iam list-policies

# List AWS managed policies
aws iam list-policies --scope AWS

# List customer managed policies
aws iam list-policies --scope Local

# Create policy
aws iam create-policy --policy-name MyPolicy --policy-document file://policy.json

# Get policy details
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Get policy version
aws iam get-policy-version --policy-arn arn:aws:iam::123456789012:policy/MyPolicy --version-id v1

# List policy versions
aws iam list-policy-versions --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Create policy version
aws iam create-policy-version --policy-arn arn:aws:iam::123456789012:policy/MyPolicy --policy-document file://new-policy.json --set-as-default

# Delete policy version
aws iam delete-policy-version --policy-arn arn:aws:iam::123456789012:policy/MyPolicy --version-id v1

# Delete policy
aws iam delete-policy --policy-arn arn:aws:iam::123456789012:policy/MyPolicy

# Attach user policy
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Detach user policy
aws iam detach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# List user policies
aws iam list-attached-user-policies --user-name john

# Put inline user policy
aws iam put-user-policy --user-name john --policy-name MyInlinePolicy --policy-document file://policy.json

# Get inline user policy
aws iam get-user-policy --user-name john --policy-name MyInlinePolicy

# Delete inline user policy
aws iam delete-user-policy --user-name john --policy-name MyInlinePolicy
```

---

## VPC (Virtual Private Cloud)

### VPC Management
```bash
# List VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-0123456789abcdef0

# Modify VPC attribute (enable DNS hostnames)
aws ec2 modify-vpc-attribute --vpc-id vpc-0123456789abcdef0 --enable-dns-hostnames

# Modify VPC attribute (enable DNS support)
aws ec2 modify-vpc-attribute --vpc-id vpc-0123456789abcdef0 --enable-dns-support

# Create tags for VPC
aws ec2 create-tags --resources vpc-0123456789abcdef0 --tags Key=Name,Value=MyVPC
```

### Subnet Management
```bash
# List subnets
aws ec2 describe-subnets

# Create subnet
aws ec2 create-subnet --vpc-id vpc-0123456789abcdef0 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a

# Delete subnet
aws ec2 delete-subnet --subnet-id subnet-0123456789abcdef0

# Modify subnet attribute (auto-assign public IP)
aws ec2 modify-subnet-attribute --subnet-id subnet-0123456789abcdef0 --map-public-ip-on-launch
```

### Internet Gateway
```bash
# List internet gateways
aws ec2 describe-internet-gateways

# Create internet gateway
aws ec2 create-internet-gateway

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway --internet-gateway-id igw-0123456789abcdef0 --vpc-id vpc-0123456789abcdef0

# Detach internet gateway
aws ec2 detach-internet-gateway --internet-gateway-id igw-0123456789abcdef0 --vpc-id vpc-0123456789abcdef0

# Delete internet gateway
aws ec2 delete-internet-gateway --internet-gateway-id igw-0123456789abcdef0
```

### Route Tables
```bash
# List route tables
aws ec2 describe-route-tables

# Create route table
aws ec2 create-route-table --vpc-id vpc-0123456789abcdef0

# Create route
aws ec2 create-route --route-table-id rtb-0123456789abcdef0 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0123456789abcdef0

# Delete route
aws ec2 delete-route --route-table-id rtb-0123456789abcdef0 --destination-cidr-block 0.0.0.0/0

# Associate route table with subnet
aws ec2 associate-route-table --route-table-id rtb-0123456789abcdef0 --subnet-id subnet-0123456789abcdef0

# Disassociate route table
aws ec2 disassociate-route-table --association-id rtbassoc-0123456789abcdef0

# Delete route table
aws ec2 delete-route-table --route-table-id rtb-0123456789abcdef0
```

### NAT Gateway
```bash
# Create NAT gateway
aws ec2 create-nat-gateway --subnet-id subnet-0123456789abcdef0 --allocation-id eipalloc-0123456789abcdef0

# List NAT gateways
aws ec2 describe-nat-gateways

# Delete NAT gateway
aws ec2 delete-nat-gateway --nat-gateway-id nat-0123456789abcdef0
```

### VPC Peering
```bash
# Create VPC peering connection
aws ec2 create-vpc-peering-connection --vpc-id vpc-0123456789abcdef0 --peer-vpc-id vpc-9876543210fedcba0

# Accept VPC peering connection
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-0123456789abcdef0

# Reject VPC peering connection
aws ec2 reject-vpc-peering-connection --vpc-peering-connection-id pcx-0123456789abcdef0

# Delete VPC peering connection
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id pcx-0123456789abcdef0

# List VPC peering connections
aws ec2 describe-vpc-peering-connections
```

### Network ACLs
```bash
# List network ACLs
aws ec2 describe-network-acls

# Create network ACL
aws ec2 create-network-acl --vpc-id vpc-0123456789abcdef0

# Create network ACL entry (allow)
aws ec2 create-network-acl-entry --network-acl-id acl-0123456789abcdef0 --rule-number 100 --protocol tcp --port-range From=80,To=80 --cidr-block 0.0.0.0/0 --rule-action allow --ingress

# Delete network ACL entry
aws ec2 delete-network-acl-entry --network-acl-id acl-0123456789abcdef0 --rule-number 100 --ingress

# Delete network ACL
aws ec2 delete-network-acl --network-acl-id acl-0123456789abcdef0
```

---

## Lambda

### Function Management
```bash
# List functions
aws lambda list-functions

# Create function
aws lambda create-function \
  --function-name MyFunction \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Update function code
aws lambda update-function-code --function-name MyFunction --zip-file fileb://function.zip

# Update function configuration
aws lambda update-function-configuration --function-name MyFunction --timeout 30 --memory-size 512

# Get function details
aws lambda get-function --function-name MyFunction

# Delete function
aws lambda delete-function --function-name MyFunction

# Invoke function
aws lambda invoke --function-name MyFunction --payload '{"key":"value"}' response.json

# Invoke function asynchronously
aws lambda invoke --function-name MyFunction --invocation-type Event --payload '{"key":"value"}' response.json

# Get function configuration
aws lambda get-function-configuration --function-name MyFunction

# Publish function version
aws lambda publish-version --function-name MyFunction

# List function versions
aws lambda list-versions-by-function --function-name MyFunction

# Create alias
aws lambda create-alias --function-name MyFunction --name prod --function-version 1

# Update alias
aws lambda update-alias --function-name MyFunction --name prod --function-version 2

# Delete alias
aws lambda delete-alias --function-name MyFunction --name prod

# List aliases
aws lambda list-aliases --function-name MyFunction
```

### Environment & Configuration
```bash
# Update environment variables
aws lambda update-function-configuration --function-name MyFunction \
  --environment "Variables={KEY1=value1,KEY2=value2}"

# Update VPC configuration
aws lambda update-function-configuration --function-name MyFunction \
  --vpc-config SubnetIds=subnet-0123456789abcdef0,SecurityGroupIds=sg-0123456789abcdef0

# Remove VPC configuration
aws lambda update-function-configuration --function-name MyFunction --vpc-config SubnetIds=[],SecurityGroupIds=[]

# Add layers
aws lambda update-function-configuration --function-name MyFunction \
  --layers arn:aws:lambda:us-east-1:123456789012:layer:my-layer:1

# Update dead letter queue
aws lambda update-function-configuration --function-name MyFunction \
  --dead-letter-config TargetArn=arn:aws:sqs:us-east-1:123456789012:my-dlq
```

### Event Source Mapping
```bash
# Create event source mapping (SQS)
aws lambda create-event-source-mapping \
  --function-name MyFunction \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue

# List event source mappings
aws lambda list-event-source-mappings --function-name MyFunction

# Delete event source mapping
aws lambda delete-event-source-mapping --uuid 12345678-1234-1234-1234-123456789012

# Update event source mapping
aws lambda update-event-source-mapping --uuid 12345678-1234-1234-1234-123456789012 --batch-size 5
```

### Permissions
```bash
# Add permission to function
aws lambda add-permission \
  --function-name MyFunction \
  --statement-id AllowS3Invoke \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-bucket

# Remove permission
aws lambda remove-permission --function-name MyFunction --statement-id AllowS3Invoke

# Get function policy
aws lambda get-policy --function-name MyFunction
```

---

## RDS (Relational Database Service)

### DB Instance Management
```bash
# List DB instances
aws rds describe-db-instances

# Create DB instance
aws rds create-db-instance \
  --db-instance-identifier mydbinstance \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password MyPassword123 \
  --allocated-storage 20

# Modify DB instance
aws rds modify-db-instance --db-instance-identifier mydbinstance --allocated-storage 30 --apply-immediately

# Delete DB instance
aws rds delete-db-instance --db-instance-identifier mydbinstance --skip-final-snapshot

# Delete with final snapshot
aws rds delete-db-instance --db-instance-identifier mydbinstance --final-db-snapshot-identifier myfinalsnapshot

# Start DB instance
aws rds start-db-instance --db-instance-identifier mydbinstance

# Stop DB instance
aws rds stop-db-instance --db-instance-identifier mydbinstance

# Reboot DB instance
aws rds reboot-db-instance --db-instance-identifier mydbinstance

# Get DB instance details
aws rds describe-db-instances --db-instance-identifier mydbinstance
```

### Snapshots
```bash
# Create snapshot
aws rds create-db-snapshot --db-snapshot-identifier mysnapshot --db-instance-identifier mydbinstance

# List snapshots
aws rds describe-db-snapshots

# List snapshots for specific instance
aws rds describe-db-snapshots --db-instance-identifier mydbinstance

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mynewdbinstance \
  --db-snapshot-identifier mysnapshot

# Copy snapshot
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier mysnapshot \
  --target-db-snapshot-identifier mysnapshot-copy

# Delete snapshot
aws rds delete-db-snapshot --db-snapshot-identifier mysnapshot

# Share snapshot
aws rds modify-db-snapshot-attribute \
  --db-snapshot-identifier mysnapshot \
  --attribute-name restore \
  --values-to-add 123456789012
```

### Parameter Groups
```bash
# List parameter groups
aws rds describe-db-parameter-groups

# Create parameter group
aws rds create-db-parameter-group \
  --db-parameter-group-name myparametergroup \
  --db-parameter-group-family mysql8.0 \
  --description "My parameter group"

# Modify parameter
aws rds modify-db-parameter-group \
  --db-parameter-group-name myparametergroup \
  --parameters "ParameterName=max_connections,ParameterValue=200,ApplyMethod=immediate"

# Describe parameters
aws rds describe-db-parameters --db-parameter-group-name myparametergroup

# Delete parameter group
aws rds delete-db-parameter-group --db-parameter-group-name myparametergroup
```

### Read Replicas
```bash
# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier myreadreplicadb \
  --source-db-instance-identifier mydbinstance

# Promote read replica
aws rds promote-read-replica --db-instance-identifier myreadreplicadb

# List read replicas
aws rds describe-db-instances --filters "Name=read-replica-source-db-instance-identifier,Values=mydbinstance"
```

---

## DynamoDB

### Table Management
```bash
# List tables
aws dynamodb list-tables

# Create table
aws dynamodb create-table \
  --table-name MyTable \
  --attribute-definitions AttributeName=Id,AttributeType=S \
  --key-schema AttributeName=Id,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5

# Describe table
aws dynamodb describe-table --table-name MyTable

# Update table
aws dynamodb update-table \
  --table-name MyTable \
  --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=10

# Delete table
aws dynamodb delete-table --table-name MyTable

# Wait for table to be active
aws dynamodb wait table-exists --table-name MyTable
```

### Item Operations
```bash
# Put item
aws dynamodb put-item \
  --table-name MyTable \
  --item '{"Id":{"S":"123"},"Name":{"S":"John"},"Age":{"N":"30"}}'

# Get item
aws dynamodb get-item \
  --table-name MyTable \
  --key '{"Id":{"S":"123"}}'

# Update item
aws dynamodb update-item \
  --table-name MyTable \
  --key '{"Id":{"S":"123"}}' \
  --update-expression "SET Age = :age" \
  --expression-attribute-values '{":age":{"N":"31"}}'

# Delete item
aws dynamodb delete-item \
  --table-name MyTable \
  --key '{"Id":{"S":"123"}}'

# Query items
aws dynamodb query \
  --table-name MyTable \
  --key-condition-expression "Id = :id" \
  --expression-attribute-values '{":id":{"S":"123"}}'

# Scan table
aws dynamodb scan --table-name MyTable

# Scan with filter
aws dynamodb scan \
  --table-name MyTable \
  --filter-expression "Age > :age" \
  --expression-attribute-values '{":age":{"N":"25"}}'

# Batch get items
aws dynamodb batch-get-item \
  --request-items '{"MyTable":{"Keys":[{"Id":{"S":"123"}},{"Id":{"S":"456"}}]}}'

# Batch write items
aws dynamodb batch-write-item \
  --request-items file://items.json
```

### Backup and Restore
```bash
# Create backup
aws dynamodb create-backup --table-name MyTable --backup-name MyBackup

# List backups
aws dynamodb list-backups --table-name MyTable

# Describe backup
aws dynamodb describe-backup --backup-arn arn:aws:dynamodb:us-east-1:123456789012:table/MyTable/backup/01234567890123-abcdef12

# Restore from backup
aws dynamodb restore-table-from-backup \
  --target-table-name MyRestoredTable \
  --backup-arn arn:aws:dynamodb:us-east-1:123456789012:table/MyTable/backup/01234567890123-abcdef12

# Delete backup
aws dynamodb delete-backup --backup-arn arn:aws:dynamodb:us-east-1:123456789012:table/MyTable/backup/01234567890123-abcdef12

# Enable point-in-time recovery
aws dynamodb update-continuous-backups \
  --table-name MyTable \
  --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true

# Restore to point in time
aws dynamodb restore-table-to-point-in-time \
  --source-table-name MyTable \
  --target-table-name MyRestoredTable \
  --restore-date-time 2024-01-01T00:00:00Z
```

---

## ECS (Elastic Container Service)

### Cluster Management
```bash
# List clusters
aws ecs list-clusters

# Create cluster
aws ecs create-cluster --cluster-name my-cluster

# Describe cluster
aws ecs describe-clusters --clusters my-cluster

# Delete cluster
aws ecs delete-cluster --cluster my-cluster
```

### Task Definitions
```bash
# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# List task definitions
aws ecs list-task-definitions

# Describe task definition
aws ecs describe-task-definition --task-definition my-task:1

# Deregister task definition
aws ecs deregister-task-definition --task-definition my-task:1
```

### Service Management
```bash
# Create service
aws ecs create-service \
  --cluster my-cluster \
  --service-name my-service \
  --task-definition my-task:1 \
  --desired-count 2

# List services
aws ecs list-services --cluster my-cluster

# Describe service
aws ecs describe-services --cluster my-cluster --services my-service

# Update service
aws ecs update-service --cluster my-cluster --service my-service --desired-count 3

# Delete service
aws ecs delete-service --cluster my-cluster --service my-service --force
```

### Task Management
```bash
# Run task
aws ecs run-task --cluster my-cluster --task-definition my-task:1 --count 1

# List tasks
aws ecs list-tasks --cluster my-cluster

# Describe task
aws ecs describe-tasks --cluster my-cluster --tasks arn:aws:ecs:us-east-1:123456789012:task/my-cluster/abc123

# Stop task
aws ecs stop-task --cluster my-cluster --task arn:aws:ecs:us-east-1:123456789012:task/my-cluster/abc123
```

---

## CloudFormation

### Stack Management
```bash
# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters file://parameters.json

# Create stack from S3
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-url https://s3.amazonaws.com/mybucket/template.yaml

# List stacks
aws cloudformation list-stacks

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Wait for stack creation
aws cloudformation wait stack-create-complete --stack-name my-stack

# Wait for stack update
aws cloudformation wait stack-update-complete --stack-name my-stack

# Wait for stack deletion
aws cloudformation wait stack-delete-complete --stack-name my-stack
```

### Stack Events and Resources
```bash
# Describe stack events
aws cloudformation describe-stack-events --stack-name my-stack

# Describe stack resources
aws cloudformation describe-stack-resources --stack-name my-stack

# List stack resources
aws cloudformation list-stack-resources --stack-name my-stack

# Get template
aws cloudformation get-template --stack-name my-stack
```

### Change Sets
```bash
# Create change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changeset \
  --template-body file://template.yaml

# Describe change set
aws cloudformation describe-change-set --stack-name my-stack --change-set-name my-changeset

# Execute change set
aws cloudformation execute-change-set --stack-name my-stack --change-set-name my-changeset

# Delete change set
aws cloudformation delete-change-set --stack-name my-stack --change-set-name my-changeset
```

### Stack Sets
```bash
# Create stack set
aws cloudformation create-stack-set \
  --stack-set-name my-stackset \
  --template-body file://template.yaml

# List stack sets
aws cloudformation list-stack-sets

# Create stack instances
aws cloudformation create-stack-instances \
  --stack-set-name my-stackset \
  --accounts 123456789012 \
  --regions us-east-1

# Delete stack instances
aws cloudformation delete-stack-instances \
  --stack-set-name my-stackset \
  --accounts 123456789012 \
  --regions us-east-1 \
  --no-retain-stacks

# Delete stack set
aws cloudformation delete-stack-set --stack-set-name my-stackset
```

---

## CloudWatch

### Logs
```bash
# List log groups
aws logs describe-log-groups

# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Delete log group
aws logs delete-log-group --log-group-name /aws/lambda/my-function

# List log streams
aws logs describe-log-streams --log-group-name /aws/lambda/my-function

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name 2024/01/01/[$LATEST]abc123

# Filter log events
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "ERROR"

# Filter with time range
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --start-time 1609459200000 \
  --end-time 1609545600000

# Tail logs
aws logs tail /aws/lambda/my-function --follow

# Put log events
aws logs put-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name mystream \
  --log-events file://events.json
```

### Metrics
```bash
# List metrics
aws cloudwatch list-metrics

# List metrics for namespace
aws cloudwatch list-metrics --namespace AWS/EC2

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 \
  --statistics Average

# Put metric data
aws cloudwatch put-metric-data \
  --namespace MyApp \
  --metric-name PageViews \
  --value 100 \
  --timestamp 2024-01-01T12:00:00Z
```

### Alarms
```bash
# List alarms
aws cloudwatch describe-alarms

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

# Delete alarm
aws cloudwatch delete-alarms --alarm-names high-cpu

# Describe alarm history
aws cloudwatch describe-alarm-history --alarm-name high-cpu

# Enable alarm actions
aws cloudwatch enable-alarm-actions --alarm-names high-cpu

# Disable alarm actions
aws cloudwatch disable-alarm-actions --alarm-names high-cpu

# Set alarm state
aws cloudwatch set-alarm-state \
  --alarm-name high-cpu \
  --state-value ALARM \
  --state-reason "Testing alarm"
```

---

## Additional Services

### ELB (Elastic Load Balancing)
```bash
# List load balancers (ALB/NLB)
aws elbv2 describe-load-balancers

# Create application load balancer
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-0123456789abcdef0 subnet-9876543210fedcba0 \
  --security-groups sg-0123456789abcdef0

# Delete load balancer
aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890abcdef

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-0123456789abcdef0

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/1234567890abcdef \
  --targets Id=i-1234567890abcdef0 Id=i-9876543210fedcba0

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/1234567890abcdef \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-targets/1234567890abcdef
```

### Auto Scaling
```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name my-template \
  --version-description "Version 1" \
  --launch-template-data file://template-data.json

# Create auto scaling group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template \
  --min-size 1 \
  --max-size 10 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-0123456789abcdef0,subnet-9876543210fedcba0"

# Update auto scaling group
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --min-size 2 \
  --max-size 20

# Delete auto scaling group
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name my-asg --force-delete

# Put scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name scale-up \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity
```

### SNS (Simple Notification Service)
```bash
# List topics
aws sns list-topics

# Create topic
aws sns create-topic --name my-topic

# Delete topic
aws sns delete-topic --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic

# Subscribe email
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic \
  --protocol email \
  --notification-endpoint user@example.com

# Publish message
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-topic \
  --message "Hello from SNS"
```

### SQS (Simple Queue Service)
```bash
# List queues
aws sqs list-queues

# Create queue
aws sqs create-queue --queue-name my-queue

# Get queue URL
aws sqs get-queue-url --queue-name my-queue

# Send message
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --message-body "Hello from SQS"

# Receive messages
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue

# Delete message
aws sqs delete-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue \
  --receipt-handle "AQEBxyz..."

# Delete queue
aws sqs delete-queue --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/my-queue
```

### Route 53
```bash
# List hosted zones
aws route53 list-hosted-zones

# Create hosted zone
aws route53 create-hosted-zone --name example.com --caller-reference $(date +%s)

# List resource record sets
aws route53 list-resource-record-sets --hosted-zone-id Z1234567890ABC

# Change resource record sets
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch file://changes.json
```

---

## Tips and Best Practices

### Query and Filter
```bash
# Use JMESPath queries
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table

# Filter by tag
aws ec2 describe-instances --filters "Name=tag:Environment,Values=production" --query 'Reservations[*].Instances[*].InstanceId' --output text

# Get specific attributes
aws s3api list-buckets --query 'Buckets[*].Name' --output text
```

### Output Formats
```bash
# JSON output (default)
aws ec2 describe-instances --output json

# Table output
aws ec2 describe-instances --output table

# Text output
aws ec2 describe-instances --output text

# YAML output
aws ec2 describe-instances --output yaml
```

### Pagination
```bash
# Use pagination for large result sets
aws s3api list-objects-v2 --bucket my-bucket --max-items 100

# Get next page
aws s3api list-objects-v2 --bucket my-bucket --max-items 100 --starting-token <token-from-previous-command>
```

### Dry Run
```bash
# Test command without executing
aws ec2 run-instances --dry-run --image-id ami-0abcdef1234567890 --instance-type t3.micro
```

---

This cheat sheet covers the most commonly used AWS CLI commands for certification preparation and daily operations. Practice these commands regularly to build muscle memory and confidence for exams and real-world scenarios.
